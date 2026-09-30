---
name: vite-tailwind-react
description: Scaffold, verify, and debug Vite + React + Tailwind CSS projects. Covers project creation, required config files, common setup gotchas (especially PostCSS), and browser-based UI verification.
---

# Vite + React + Tailwind CSS

Scaffold and verify modern frontend projects using Vite, React, and Tailwind CSS.

## Required Config Files

A working Vite + Tailwind + React project needs ALL of these:

| File | Purpose |
|------|---------|
| `vite.config.ts` | Vite config with `@vitejs/plugin-react` |
| `tailwind.config.js` | Tailwind theme + content paths |
| `postcss.config.js` | **Critical** — PostCSS plugins (tailwindcss + autoprefixer) |
| `tsconfig.json` | TypeScript config |
| `index.html` | Entry HTML with `#root` div |
| `src/main.tsx` | React entry point |
| `src/index.css` | `@tailwind base/components/utilities` |

## The #1 Gotcha: Missing postcss.config.js

**Symptom:** UI renders but all Tailwind classes (`flex-1`, `h-full`, `bg-*`, `border-*`, etc.) have zero effect. Everything looks unstyled or collapsed. React components mount but layout is broken.

**Root cause:** `src/index.css` contains `@tailwind` directives, but without `postcss.config.js`, PostCSS doesn't know to run the `tailwindcss` plugin. Tailwind utilities are never generated. Vite processes the CSS file but leaves `@tailwind` directives as-is (which browsers ignore).

**Fix:** Create `postcss.config.js` at project root:

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

Then restart the dev server (Vite doesn't hot-reload PostCSS config changes).

**Verify:** In browser console, test that Tailwind classes produce computed styles:
```javascript
const el = document.createElement('div');
el.className = 'flex-1';
document.body.appendChild(el);
getComputedStyle(el).flexGrow; // should be "1", not "0"
document.body.removeChild(el);
```

## Project Scaffold Commands

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p   # generates tailwind.config.js + postcss.config.js
```

**Note:** `npx tailwindcss init -p` generates both config files. If you ran `npx tailwindcss init` (without `-p`), you MUST create `postcss.config.js` manually.

## Tailwind Config

`tailwind.config.js` must include all source files in `content`:

```js
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        // custom colors here
      },
    },
  },
  plugins: [],
};
```

## React `React.StrictMode` Requires Explicit Import

**Symptom**: Black/blank page, no visible components, no console error visible. In some browsers the error is silently swallowed.

**Root cause**: Vite uses the new JSX transform (React 17+) where `React` is NOT automatically in scope. Using `React.StrictMode` without `import React from 'react'` causes `ReferenceError: React is not defined` that crashes the ENTIRE render tree. Unlike a component-level error, this kills the root `createRoot().render()` call so nothing mounts — not even an error boundary can catch it.

**Fix**: Always add the explicit import when using StrictMode:
```tsx
import React from 'react'  // REQUIRED for React.StrictMode in Vite
import ReactDOM from 'react-dom/client'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

**Why this is especially dangerous**:
- No error boundary can catch it (it happens at the root render call)
- Some browsers swallow the error silently — you just see a blank page
- `tsc --noEmit` passes fine (TypeScript resolves the type, but at runtime the value is undefined)
- The page `<title>` still renders (it's in HTML, not React), making it look like the app loaded

**Quick test**: If `document.title` is correct but `document.getElementById('root').innerHTML === ''`, suspect a root-level render crash. Manually test in console:
```javascript
import('/src/main.tsx').catch(e => console.error(e))
```

## Full-Height Layout Pattern

For a full-viewport app (like a workflow designer or dashboard):

```css
/* index.css */
html, body, #root {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
```

```tsx
// App.tsx
<div className="h-full flex flex-col">
  <Toolbar />
  <div className="flex-1 flex overflow-hidden">
    <Sidebar />
    <main className="flex-1 h-full">
      {/* content */}
    </main>
  </div>
</div>
```

**Important:** `flex-1` only works on children of a `display: flex` parent that has a defined height. If a `flex-1` child renders with height 0, trace up the DOM to find where the height chain breaks.

## Verification Checklist

After scaffolding or when debugging a broken UI:

1. **Build compiles:** `npx tsc --noEmit` — zero errors
2. **Vite builds:** `npx vite build` — no warnings
3. **Dev server serves HTML:** `curl -s http://localhost:5173/` returns 200 with HTML
4. **Tailwind works:** Browser console test above shows `flex-grow: 1`
5. **Canvas/layout elements sized:** Check `.react-flow` or main content div has non-zero width AND height
6. **No JS errors:** Browser console has zero uncaught exceptions

## Debugging: UI Renders but Looks Broken

1. Check if Tailwind CSS is loaded (test `flex-1` in console)
2. If not, verify `postcss.config.js` exists and restart dev server
3. If Tailwind works but layout is wrong, inspect element heights up the DOM tree
4. Check for React Flow / canvas library warnings about container dimensions
5. Verify `html, body, #root` all have `height: 100%`
6. **Nodes exist in DOM but invisible on canvas:** Check viewport transform — if `scale` is high (e.g. 2x) and `translate` values are large negatives, `fitView` is pushing nodes off-screen. Replace with `defaultViewport={{ x: 0, y: 0, zoom: 1 }}`

## Tailwind `border-color/opacity` Shorthand Gotcha

**Symptom:** Custom nodes (e.g. React Flow) render in the DOM with correct positions but are invisible -- no border, no background.

**Root cause:** Classes like `border-green-500/40` set only the border **color**, not the border **width**. Without the `border` class, `border-width` stays at 0px and the element has no visible border.

**Fix:** Always pair color shorthands with `border`:
```tsx
// Before (invisible):
<div className="border-green-500/40">

// After (visible):
<div className="border border-green-500/40 bg-surface rounded-lg">
```

**General rule:** `border-{color}` only sets `border-color`. You still need `border` (or `border-2`, etc.) for width. This applies to all color/opacity shorthands: `border-red-500/30`, `border-amber-400`, etc.

Also add `bg-*` to inner node divs so they have a visible surface, especially on dark backgrounds.

## React Flow Custom Node Visibility

When using `@xyflow/react` custom nodes, the node component's root div needs explicit styling:

1. **Border:** `border border-{color}/opacity` (both width and color)
2. **Background:** `bg-surface` or similar (otherwise transparent on dark themes)
3. **Border radius:** `rounded-lg` for visual polish

Without these, React Flow positions the node correctly (check `translate(x, y)` in DOM), but the node container is invisible -- users see an empty canvas even though `document.querySelectorAll('.react-flow__node')` returns elements.

**Quick diagnose:** In browser console:
```javascript
// Check if nodes exist but are invisible
const nodes = document.querySelectorAll('.react-flow__node');
for (const n of nodes) {
  const inner = n.querySelector('div');
  const s = getComputedStyle(inner);
  console.log({
    id: n.getAttribute('data-id'),
    borderWidth: s.borderWidth,
    backgroundColor: s.backgroundColor,
    w: n.offsetWidth,
    h: n.offsetHeight
  });
}
```

## React Flow v12 `visibility: hidden` on Nodes AND Handles

**Symptom:** Nodes exist in the DOM (`.react-flow__node` selectors find them), have correct `transform: translate(x, y)` positions, non-zero width/height, but are completely invisible on the canvas. No console errors. **Same issue affects handles** — connector points don't appear even though `.react-flow__handle` elements exist in the DOM.

**Root cause:** `@xyflow/react` v12 sets `visibility: hidden` as an inline style on `.react-flow__node` elements (and sometimes handles) during its internal measurement/layout cycle. This normally gets flipped to `visible` after measurement completes, but with custom nodes/handles the visibility sometimes never updates.

**Diagnose:**
```javascript
const n = document.querySelector('.react-flow__node');
n.style.visibility; // returns "hidden" even though node should be visible

const h = document.querySelector('.react-flow__handle');
getComputedStyle(h).visibility; // returns "hidden" for handles too
```

**Fix:** Override in CSS with `!important` to beat the inline style. Apply to BOTH nodes AND handles:
```css
/* index.css */
.react-flow__node {
  /* ... other node styles ... */
  visibility: visible !important;
}

.react-flow__handle {
  width: 16px !important;
  height: 16px !important;
  background: #3b82f6 !important;
  border: 3px solid #0f172a !important;
  border-radius: 50% !important;
  cursor: crosshair !important;
  z-index: 10 !important;
}

.react-flow__handle:hover {
  background: #60a5fa !important;
  border-color: #3b82f6 !important;
  width: 22px !important;
  height: 22px !important;
  box-shadow: 0 0 6px rgba(59, 130, 246, 0.6) !important;
}
```

**Important:** The `!important` is necessary because React Flow sets `visibility: hidden` as an inline style, which has higher specificity than regular CSS class rules. Apply `!important` to ALL handle properties since React Flow may also set inline width/background/border on handles.

**CRITICAL: Do NOT pass `style` prop to `<Handle>` components.** If you write `<Handle style={{ width: 10, height: 10 }} />`, those inline styles will override your CSS rules (even with `!important` in some browsers). Let CSS control all handle sizing:
```tsx
// WRONG — inline style overrides CSS
<Handle type="source" position={Position.Top} style={{ width: 10, height: 10 }} />

// CORRECT — let CSS handle sizing
<Handle type="source" position={Position.Top} />
```

## React Flow Handles (Connectors) — Complete Guide

### Making Handles Visible

1. **CSS with `!important`** — required to override React Flow inline styles (see above)
2. **No inline `style` prop** on `<Handle>` — if you pass `style={{ width: 10, height: 10 }}`, those inline styles will override your CSS. Remove the `style` prop and let CSS control sizing.
3. **Minimum 14-16px size** — default 8-10px handles are too small to see easily

### All-Sides Handles Pattern (connect any side to any side)

For maximum flexibility, place handles on all 4 sides of every node. Use the same `id` for both source and target on a given side — React Flow uses internal `data-id` to distinguish them:

```tsx
import { Handle, Position } from '@xyflow/react';

function AllSidesHandles() {
  return (
    <>
      <Handle type="source" position={Position.Top} id="top" />
      <Handle type="source" position={Position.Right} id="right" />
      <Handle type="source" position={Position.Bottom} id="bottom" />
      <Handle type="source" position={Position.Left} id="left" />
      <Handle type="target" position={Position.Top} id="top" />
      <Handle type="target" position={Position.Right} id="right" />
      <Handle type="target" position={Position.Bottom} id="bottom" />
      <Handle type="target" position={Position.Left} id="left" />
    </>
  );
}

export const MyNode = memo((props: NodeProps) => (
  <div className="border border-color/40 bg-surface rounded-lg">
    <AllSidesHandles />
    <NodeCard data={props.data} />
  </div>
));
```

This gives 8 handles per node (4 source + 4 target), enabling connections from any edge to any edge.

### Diagnosing Handle Issues

```javascript
const handles = document.querySelectorAll('.react-flow__handle');
console.log('Total handles:', handles.length);
handles.forEach(h => {
  const r = h.getBoundingClientRect();
  const s = getComputedStyle(h);
  console.log({
    pos: h.getAttribute('data-handlepos'),
    type: h.classList.contains('source') ? 'source' : 'target',
    x: Math.round(r.x), y: Math.round(r.y),
    w: Math.round(r.width), h: Math.round(r.height),
    visibility: s.visibility,
    display: s.display
  });
});
```

### Common Handle Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Handles in DOM but invisible | `visibility: hidden` inline style | Add `visibility: visible !important` to CSS |
| Handles too small | Default 8-10px or inline style override | Set `width: 16px !important; height: 16px !important` |
| Handles not `cursor: crosshair` | CSS not applied | Add `cursor: crosshair !important` |
| Handles don't grow on hover | Hover CSS overridden by inline styles | Use `!important` on hover state too |
| Can't connect | Handles missing `type` or wrong `position` | Ensure both `source` and `target` handles exist |
| Can connect but edge doesn't render | Strict connection mode | Add `connectionMode={ConnectionMode.Loose}` to `<ReactFlow>` |

### Connections Not Working?

If handles are visible but dragging from source to target doesn't create an edge:

1. **`connectionMode`** -- By default React Flow uses `Strict` mode where sources can only connect to targets of the same handle type. Use `Loose` mode for maximum flexibility:
   ```tsx
   import { ConnectionMode } from '@xyflow/react';
   <ReactFlow connectionMode={ConnectionMode.Loose} ...>
   ```

2. **Both source AND target handles required** -- Every handle position needs BOTH a `type="source"` and `type="target"` handle. A source-only handle can't receive connections.

3. **Edge data format** -- The `onConnect` handler must create edges with valid `source` and `target` node IDs:
   ```tsx
   const onConnect = useCallback((params: Connection) => {
     setEdges(eds => [...eds, {
       id: `edge-${params.source}-${params.target}-${Date.now()}`,
       source: params.source,
       target: params.target,
       sourceHandle: params.sourceHandle,
       targetHandle: params.targetHandle,
     }]);
   }, [setEdges]);
   ```

## React Flow Drag-and-Drop from External Palette

**Symptom:** Dragging a node from an external palette onto the React Flow canvas fires the drop handler (store updates, config panel opens) but the node never appears on the canvas. Or: the drop handler doesn't fire at all.

**Root cause:** `@xyflow/react` v12 does NOT expose `onDrop` or `onDragOver` props. React Flow's internal pane intercepts these events and stops propagation. Putting `onDragOver`/`onDrop` on a wrapper div doesn't work because React Flow's internal elements swallow the events before they reach the wrapper.

**Fix:** Use native `addEventListener` (not React synthetic events) on the container div, inside a child component that has access to `useReactFlow()`:

```tsx
import { useReactFlow } from '@xyflow/react';

function ExternalDropHandler() {
  const { screenToFlowPosition } = useReactFlow();
  const addNode = useWorkflowStore((s) => s.addNode);
  const addNodeRef = useRef(addNode);
  addNodeRef.current = addNode;
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
      addNodeRef.current(type as any, position);
    };

    container.addEventListener('dragover', onDragOver);
    container.addEventListener('drop', onDrop);
    return () => {
      container.removeEventListener('dragover', onDragOver);
      container.removeEventListener('drop', onDrop);
    };
  }, []);

  return null; // renders nothing, no DOM elements to block interaction
}
```

**Key details:**
- `ExternalDropHandler` renders `null` — no overlay div that would block pointer events
- Uses `useReactFlow()` to get `screenToFlowPosition` for correct coordinate conversion (accounts for pan/zoom)
- Checks for `application/reactflow` data to distinguish palette drops from internal node drags
- The container div needs a `data-external-drop="true"` attribute for the selector
- TypeScript: native `addEventListener` expects `Event` not `DragEvent` — cast inside the handler

**Palette drag start** must set the correct MIME type:
```tsx
const handleDragStart = (e: React.DragEvent, type: NodeType) => {
  e.dataTransfer.setData('application/reactflow', type);
  e.dataTransfer.effectAllowed = 'move';
};
```

**Coordinate issue — `screenToFlowPosition` can give wrong values for native events:** In some browsers, `screenToFlowPosition` misinterprets native drag event coordinates, producing positions with negative Y values (e.g. `y: -133`). If this happens, calculate position manually from the pane's bounding rect:

```tsx
const onDrop = (e: Event) => {
  const de = e as DragEvent;
  const type = de.dataTransfer?.getData('application/reactflow');
  if (!type) return;
  e.preventDefault();
  e.stopPropagation();

  // Fallback: calculate position from pane bounding rect
  const pane = container.querySelector('.react-flow__pane');
  if (!pane) return;
  const rect = pane.getBoundingClientRect();
  const position = { x: de.clientX - rect.left, y: de.clientY - rect.top };

  addNodeRef.current(type as any, position);
};
```

**Verify it works:** After dropping, check:
```javascript
document.querySelectorAll('.react-flow__node').length // should be > 0
document.querySelector('.react-flow__node').style.visibility // should not be "hidden"
```

For the full working code template, see [`references/react-flow-dnd.md`](references/react-flow-dnd.md).

## CSS Overflow Clipping vs Tooltip/Popover Visibility

**Symptom:** Tooltips or popovers that use `absolute` positioning inside a scrollable container (e.g. `overflow-y-auto`) are invisible or clipped.

**Root cause:** `overflow-hidden` on any ancestor clips all absolutely-positioned children, including popovers that need to extend beyond the container.

**Fix pattern (3 steps):**
1. Add `relative` to the direct parent of the tooltip — required for `absolute` children
2. Use `overflow-x-visible` on the scrollable inner div (not the outermost container)
3. Keep `overflow-hidden` on the outermost flex container for vertical scroll

```tsx
// Outer: overflow-hidden for vertical scroll
<div className="flex-1 flex flex-col overflow-hidden">
  // Inner: overflow-y-auto for scroll, overflow-x-visible for tooltip
  <div className="flex-1 overflow-y-auto py-1">
    {items.map(item => (
      // Item: relative for absolute tooltip child
      <div key={item.id} className="relative group">
        <span>{item.label}</span>
        {/* Tooltip: absolute, extends right */}
        <div className="absolute left-full top-0 ml-2 hidden group-hover:block z-50">
          <div className="bg-surface border border-border p-3 rounded-lg">
            {item.tooltip}
          </div>
        </div>
      </div>
    ))}
  </div>
</div>
```

**Pitfall:** Removing `overflow-hidden` from the scroll container breaks vertical scrolling — users can't scroll to see all content.

## React Flow Viewport Transform (DOM Fallback)

When `useReactFlow().screenToFlowPosition()` crashes (missing `ReactFlowProvider`) or `.project()` doesn't exist (v12), read the viewport transform from the DOM:

```ts
const pane = document.querySelector('.react-flow__viewport') as HTMLElement | null
let viewportX = 0, viewportY = 0, zoom = 1
if (pane) {
  const style = window.getComputedStyle(pane)
  const matrix = new DOMMatrix(style.transform)
  viewportX = matrix.m41  // translateX
  viewportY = matrix.m42  // translateY
  zoom = matrix.a         // scale
}
const position = {
  x: (e.clientX - bounds.left - viewportX) / zoom,
  y: (e.clientY - bounds.top - viewportY) / zoom,
}
```

**Why:** `useReactFlow()` hooks require `<ReactFlowProvider>` ancestor. If you call `screenToFlowPosition()` without the provider, it throws and the entire page goes black. The DOM approach is safe because it reads computed CSS transforms directly.

## React Flow `fitView` Gotcha

**Symptom:** Nodes are added to the store and exist in the DOM, but are not visible on the canvas. The viewport is zoomed in aggressively (e.g. `scale(2)`) and panned to extreme offsets (e.g. `translate(-636px, -307.5px)`), pushing nodes outside the visible area.

**Root cause:** The `fitView` prop on `<ReactFlow>` recalculates the viewport on every render. With few nodes (especially a single node), it zooms in too aggressively and the node ends up off-screen. This is especially confusing because `document.querySelectorAll('.react-flow__node')` returns elements and they have correct `translate(x, y)` transforms — they're just outside the viewport.

**Fix:** Replace `fitView` with an explicit default viewport:
```tsx
<ReactFlow
  defaultViewport={{ x: 0, y: 0, zoom: 1 }}
  minZoom={0.2}
  maxZoom={2}
  ...
>
```

Users can still click the "Fit View" button in React Flow's `<Controls />` when they want auto-fit behavior.

**Diagnose:** In browser console:
```javascript
// Check viewport transform
document.querySelector('.react-flow__viewport').style.transform
// If scale > 1.5 or translate values are large negative numbers, fitView is the culprit
```
