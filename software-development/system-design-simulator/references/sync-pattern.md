# ReactFlow ↔ Zustand Sync: The Definitive Pattern

## The Problem

ReactFlow maintains internal state via `useNodesState`/`useEdgesState`. Zustand holds the simulation source of truth. These are ALWAYS separate. Syncing them incorrectly causes:

1. **Infinite loops**: `onNodesChange` → `store.setNodes()` → `store.nodes` changes → `useEffect` fires → `setNodes()` → re-render → repeat
2. **setState during render**: Calling `store.setNodes()` inside `requestAnimationFrame` or `setTimeout(..., 0)` from event handlers triggers "Cannot update a component while rendering" and silently drops the update
3. **Stale closures**: `store.selectedNodeIds` inside `useCallback([store])` reads the value at callback creation time, not the live value
4. **Selection desync**: Clearing Zustand selection doesn't clear ReactFlow's internal selection, preventing re-selection

## The Solution: syncToStore Pattern

```tsx
const syncingFromStore = useRef(false)

// Simulation → ReactFlow: polling interval
useEffect(() => {
  const interval = setInterval(() => {
    if (syncingFromStore.current) return
    if (store.simState !== 'running') return
    setNodes(currentNodes => {
      let changed = false
      const updated = currentNodes.map(cfNode => {
        const storeNode = store.nodes.find(sn => sn.id === cfNode.id)
        if (!storeNode) return cfNode
        // Only update metrics/status, never add/remove nodes
        if (metricsChanged || statusChanged) {
          changed = true
          return { ...cfNode, data: { ...cfNode.data, metrics: storeNode.data.metrics, status: storeNode.data.status } }
        }
        return cfNode
      })
      return changed ? updated : currentNodes
    })
  }, 300)
  return () => clearInterval(interval)
}, [store.nodes, store.simState, setNodes])

// User actions → store: sync via rAF with guard flag
const syncToStore = useCallback(() => {
  syncingFromStore.current = true
  store.setNodes(nodes.map(n => ({ ...simNode })))
  store.setEdges(edges.map(e => ({ ...simEdge })))
  setTimeout(() => { syncingFromStore.current = false }, 50)
}, [nodes, edges, store])

// In event handlers:
const onNodesChange = useCallback((changes) => {
  onNodesChangeRaw(changes)
  if (syncingFromStore.current) return
  requestAnimationFrame(syncToStore)
}, [onNodesChangeRaw, syncToStore])
```

## Key Rules

1. **Never use `useEffect` depending on `store.nodes` that calls `setNodes()`** — causes infinite loop
2. **Never call Zustand setters from `requestAnimationFrame` or `setTimeout(..., 0)` in event handlers** — runs during commit phase, silently dropped
3. **Use `useRef` guard flags** to break echo loops between the two sync directions
4. **Simulation polling only updates existing nodes** — never adds or removes
5. **Selection changes must use `useRef` comparison** — never read Zustand state in callbacks for comparison
6. **Deselect must clear BOTH Zustand AND ReactFlow internal selection** — use a `deselectVersion` counter + `useEffect`
7. **Clear must clear BOTH Zustand AND ReactFlow local state** — use a `clearVersion` counter + `useEffect`
8. **PropertiesPanel must use dual-write for color/tag/label** — pass `setNodes` directly, update both store and ReactFlow. See `references/editable-nodes.md`
