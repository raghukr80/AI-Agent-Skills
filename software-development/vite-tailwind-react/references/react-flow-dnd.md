# React Flow External Drag-and-Drop Reference

## The Pattern

When building a node palette that drops onto a React Flow canvas, you cannot use React Flow's (non-existent) `onDrop`/`onDragOver` props. Use native event listeners instead.

## Working Canvas Component Template

```tsx
import { useCallback, useRef, useEffect } from 'react';
import {
  ReactFlow, Background, Controls, MiniMap,
  type OnNodesChange, type OnEdgesChange, type OnConnect,
  type Node, type Edge, type Connection, BackgroundVariant,
  useReactFlow,
} from '@xyflow/react';
import '@xyflow/react/dist/style.css';

/** Renders null -- sets up native drag/drop listeners on the parent container */
function ExternalDropHandler() {
  const { screenToFlowPosition } = useReactFlow();
  const addNodeFnRef = useRef(addNode);
  addNodeFnRef.current = addNode;
  const screenPosRef = useRef(screenToFlowPosition);
  screenPosRef.current = screenToFlowPosition;

  useEffect(() => {
    const container = document.querySelector('[data-external-drop="true"]');
    if (!container) return;

    const onDragOver = (e: Event) => {
      e.preventDefault();
      if ((e as DragEvent).dataTransfer)
        (e as DragEvent).dataTransfer!.dropEffect = 'move';
    };

    const onDrop = (e: Event) => {
      const de = e as DragEvent;
      const type = de.dataTransfer?.getData('application/reactflow');
      if (!type) return; // internal React Flow drag, ignore
      e.preventDefault();
      e.stopPropagation();
      const position = screenPosRef.current({ x: de.clientX, y: de.clientY });
      addNodeFnRef.current(type as any, position);
    };

    container.addEventListener('dragover', onDragOver);
    container.addEventListener('drop', onDrop);
    return () => {
      container.removeEventListener('dragover', onDragOver);
      container.removeEventListener('drop', onDrop);
    };
  }, []);

  return null;
}

// In your Canvas component's JSX:
<div data-external-drop="true" className="flex-1 h-full relative">
  <ReactFlow nodes={rfNodes} edges={rfEdges} nodeTypes={nodeTypes} ...>
    <Background ... />
    <Controls />
    <MiniMap ... />
    <ExternalDropHandler />
  </ReactFlow>
</div>
```

## Why Not an Overlay Div?

An alternative approach uses a full-size overlay div with high z-index to catch drops. This has two problems:

1. **Blocks all interaction** -- clicking nodes, dragging edges, panning the canvas all fail when the overlay is active
2. **Complex toggle logic** -- you need `isDragging` state, `dragEnter`/`dragLeave` detection, and edge cases for leaving the container

The native listener approach avoids both: no DOM elements, no blocked events, `useReactFlow()` gives correct coordinates.

## Why `screenToFlowPosition`?

Using raw `e.clientX - rect.left` for position calculation doesn't account for React Flow's pan/zoom transform. `screenToFlowPosition` converts screen coordinates to flow coordinates correctly regardless of viewport state.

## TypeScript Gotcha

Native `Element.addEventListener('dragover', handler)` expects `Event` type, NOT `DragEvent`. The `DragEvent` type exists on the React side but not in the DOM lib's `addEventListener` overloads. Cast inside the handler:

```ts
const onDrop = (e: Event) => {
  const de = e as DragEvent;
  // now use de.dataTransfer, de.clientX, etc.
};
```

## Debugging: Drag-and-Drop Still Doesn't Work

If the native listener approach works in automated tests but not in a specific browser:

1. **Verify drag events fire at all** -- Add a native capture-phase listener on the draggable element:
   ```js
   document.querySelector('[draggable="true"]').addEventListener('dragstart', function(e) {
     console.log('[DRAGSTART] types:', Array.from(e.dataTransfer.types));
   }, true);
   ```
   If this doesn't fire, the browser may be blocking drag events (extensions, security settings).

2. **Verify the drop event reaches the container** -- Add `console.log` to the native `onDrop` handler. If it doesn't fire, React Flow's internal pane may be calling `stopPropagation()`.

3. **Check for browser extensions** -- Ad blockers, privacy extensions, and security tools can interfere with drag-and-drop. Try incognito/private mode.

4. **Verify the MIME type matches exactly** -- The palette's `dragstart` must use `'application/reactflow'` and the drop handler must check for the same string. A mismatch means the drop handler silently ignores the event.

5. **Test with a programmatic button first** -- Add a toolbar button that calls `addNode(type, {x, y})` directly. If this works (node appears on canvas), the rendering pipeline is fine and the issue is purely in drag-and-drop event delivery.

6. **Check browser console for extension errors** -- Errors like "A listener indicated an asynchronous response by returning true, but the message channel closed" indicate browser extension interference, not application bugs.

## `screenToFlowPosition` Coordinate Issue with Native Events

**Symptom:** `screenToFlowPosition` works correctly with React Flow's synthetic events but produces wrong coordinates (e.g. negative Y values like `y: -133`) when used with native `DragEvent` from `addEventListener`. Nodes are added to the store and config panel opens, but the node appears off-screen.

**Root cause:** `screenToFlowPosition` may misinterpret native drag event coordinates in some browser/event combinations, especially when the event target is the container div rather than an internal React Flow element.

**Fix:** Calculate position manually from the pane's bounding rect as a fallback:

```tsx
const onDrop = (e: Event) => {
  const de = e as DragEvent;
  const type = de.dataTransfer?.getData('application/reactflow');
  if (!type) return;
  e.preventDefault();
  e.stopPropagation();

  // Calculate position from pane bounding rect (works reliably across browsers)
  const pane = container.querySelector('.react-flow__pane');
  if (!pane) return;
  const rect = pane.getBoundingClientRect();
  const position = { x: de.clientX - rect.left, y: de.clientY - rect.top };

  addNodeFnRef.current(type as any, position);
};
```

**Note:** The bounding rect approach does NOT account for React Flow's pan/zoom transform. If the user has panned or zoomed, use `screenToFlowPosition` but verify the result — if coordinates are negative or wildly off, fall back to the bounding rect method.

## Edges Not Rendering

**Symptom:** The `<div class="react-flow__edges">` layer exists but is empty. No `.react-flow__edge` or `.react-flow__edge-path` elements appear even after connecting nodes.

**Diagnostic steps:**

1. **Check if `onConnect` fires** -- Add `console.log('[CONNECT]', params)` to the `onConnect` handler. If it doesn't fire, the connection drag isn't completing.

2. **Verify edge data structure** -- Edges need at minimum: `{ id, source, target }`. Optional: `sourceHandle`, `targetHandle`, `type`.

3. **Check for CSS issues** -- Verify `.react-flow__edge-path` has `stroke` and `stroke-width` set.

4. **Handle `sourceHandle`/`targetHandle` matching** -- If edges specify `sourceHandle: "bottom"` but the source node's handle has `id: "bottom"`, they must match exactly. Mismatched handle IDs cause edges to not render.

5. **Check React Flow edges array** -- The `edges` prop passed to `<ReactFlow>` must contain valid edge objects. Console.log `rfEdges` before passing to verify.

6. **React Flow v12 `connectionMode`** -- Ensure `connectionMode={ConnectionMode.Loose}` is set on `<ReactFlow>` if you want to connect any handle to any handle without strict source/target type matching.
