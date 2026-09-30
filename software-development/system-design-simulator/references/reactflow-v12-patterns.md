# ReactFlow v12 Integration Patterns

Patterns discovered while building the System Design Simulator at `/home/raghu/syssim/`.

## NodeProps Typing

ReactFlow v12 exports `NodeProps<NodeType extends Node = Node>`. Your custom node data type must extend the `Node` interface from `@xyflow/react`, which requires fields like `position`, `measured`, etc. Custom types like `SimNode` (which only have `id`, `type`, `position`, `data`) often fail the constraint.

**Fix**: Use `any` and destructure manually:

```tsx
function ComponentNodeComponent(props: any) {
  const { data, id, selected } = props as {
    data: SimNode['data']
    id: string
    selected?: boolean
  }
  // ...
}
export const ComponentNode = memo(ComponentNodeComponent)
```

Do NOT use `NodeProps<NodeProps<T>>` (double nesting) or `NodeProps<SimNode>` (constraint error).

## Zustand v5 Store Typing

`ReturnType<typeof useStore>` returns `unknown` in Zustand v5. Helper functions that accept the store as a parameter should use `any`:

```tsx
// GOOD — use `any` for store parameters
function getNodeStatusColor(nodeId: string, store: any): string {
  const node = store.nodes.find((n: any) => n.id === nodeId)
}
```

For inline usage, `useDiagramStore.getState()` works fine and returns correctly-typed state.

## Syncing ReactFlow Internal State with Zustand

Keep ReactFlow's internal `useNodesState`/`useEdgesState` separate from Zustand. Sync patterns:

```tsx
// Sync RF → Zustand on drag stop
const onNodeDragStop = useCallback(() => {
  syncToStore()
}, [syncToStore])

// Sync Zustand → RF on store changes (e.g., undo/redo)
useEffect(() => {
  const unsub = useDiagramStore.subscribe((state) => {
    // Update RF nodes/edges from store
  })
  return unsub
}, [])

// Use setTimeout to avoid stale closures in event handlers
setTimeout(syncToStore, 0)
```

## Keyboard Shortcuts

Wire in a single `useEffect` on `window.addEventListener('keydown', ...)`:

```tsx
useEffect(() => {
  const handler = (e: KeyboardEvent) => {
    const tag = (e.target as HTMLElement).tagName
    if (tag === 'INPUT' || tag === 'TEXTAREA' || tag === 'SELECT') return

    if ((e.key === 'Delete' || e.key === 'Backspace') && store.selectedNodeIds.length > 0) {
      store.removeNodes(store.selectedNodeIds)
      setNodes(nds => nds.filter(n => !store.selectedNodeIds.includes(n.id)))
      setEdges(eds => eds.filter(e => !store.selectedNodeIds.includes(e.source) && !store.selectedNodeIds.includes(e.target)))
    }
    if ((e.ctrlKey || e.metaKey) && e.key === 'z' && !e.shiftKey) {
      e.preventDefault()
      store.undo()
    }
    if ((e.ctrlKey || e.metaKey) && (e.key === 'y' || (e.key === 'z' && e.shiftKey))) {
      e.preventDefault()
      store.redo()
    }
    if (e.key === 'Escape') {
      store.setSelectedNodes([])
      store.setSelectedEdges([])
    }
  }
  window.addEventListener('keydown', handler)
  return () => window.removeEventListener('keydown', handler)
}, [store, setNodes, setEdges])
```

## Edge Connections (Any Side to Any Side)

Each node has 8 handles (target+source × 4 sides). For proper any-side-to-any-side connections:

1. **Give each handle a unique ID**: `id="target-top"`, `id="source-top"`, `id="target-left"`, `id="source-left"`, etc.
2. **Make handles visible**: 3×3px with 2px border — `className="w-3 h-3 bg-accent border-2 border-surface"`
3. **Explicitly pass handle IDs in `onConnect`** — do NOT use `...params` spread:
   ```tsx
   const onConnect: OnConnect = useCallback((params) => {
     const edge: Edge = {
       id: `edge_${params.source}_${params.target}_${Date.now()}`,
       source: params.source!,
       target: params.target!,
       sourceHandle: params.sourceHandle,  // e.g. "source-right"
       targetHandle: params.targetHandle,  // e.g. "target-left"
       type: 'smoothstep',
       ...
     }
     setEdges(eds => addEdge(edge, eds))
   }, [])
   ```
4. **Do NOT use `edgesSelectable` prop** — it doesn't exist in ReactFlow v12. Edges are selectable by default.
5. **Do NOT use `connectionMode="loose"`** — not a valid prop in ReactFlow v12.

## Native Drop Event Position

`screenToFlowPosition` gives wrong coordinates for native drag-and-drop events. Compute manually:

```tsx
const onDrop = useCallback((e: React.DragEvent) => {
  e.preventDefault()
  const componentType = e.dataTransfer.getData('application/syssim-component')
  const bounds = reactFlowWrapper.current?.getBoundingClientRect()
  if (!bounds) return
  const position = {
    x: e.clientX - bounds.left - 75,
    y: e.clientY - bounds.top - 25,
  }
  // ...create node at position
}, [])
```

## Particle Canvas Viewport Transform

To correctly position particles over ReactFlow edges during pan/zoom:

```tsx
function getViewportTransform() {
  const pane = document.querySelector('.react-flow__viewport') as HTMLElement | null
  if (!pane) return { x: 0, y: 0, zoom: 1 }
  const transform = window.getComputedStyle(pane).transform
  if (!transform || transform === 'none') return { x: 0, y: 0, zoom: 1 }
  const matrix = new DOMMatrix(transform)
  return { x: matrix.m41, y: matrix.m42, zoom: matrix.a }
}

// Transform flow coords → screen coords
const sx = pt.x * tZoom + tx
const sy = pt.y * tZoom + ty
```

Query edge SVG paths by ReactFlow's `data-id` attribute:
```tsx
const pathEl = document.querySelector(`[data-id="${edgeId}"] .react-flow__edge-path`) as SVGPathElement
const pt = pathEl.getPointAtLength(progress * pathEl.getTotalLength())
```
