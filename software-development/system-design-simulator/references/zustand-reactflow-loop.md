# Zustand + ReactFlow Infinite Loop Debug Session

## Error Transcript

```
Warning: Maximum update depth exceeded. This can happen when a component calls setState inside useEffect, but either doesn't have a dependency array, or one of the dependencies changes on every render.
    at SelectionListenerInner (@xyflow_react.js?v=94b3acb4:5591:35)
    at SelectionListener (@xyflow_react.js?v=94b3acb4:5602:30)
    at BatchProvider (@xyflow_react.js?v=94b3acb4:6103:36)
    at ReactFlowProvider (@xyflow_react.js?v=94b3acb4:8277:44)
    at ReactFlow (@xyflow_react.js?v=94b3acb4:8309:22)
    at SimulatorCanvas (SimulatorCanvas.tsx:26:17)
```

## Loop Mechanism

```
onSelectionChange (ReactFlow internal)
  → calls store.setSelectedNodes(ids)
    → Zustand handleStoreChange
      → forceStoreRerender
        → ReactFlow SelectionListenerInner re-renders
          → fires onSelectionChange again (same selection)
            → store.setSelectedNodes(ids)  ← infinite loop
```

## Root Cause

`onSelectionChange` was unconditionally calling `store.setSelectedNodes()`:

```tsx
// BEFORE (broken)
const onSelectionChange = useCallback(
  ({ nodes: selNodes, edges: selEdges }) => {
    store.setSelectedNodes(selNodes.map(n => n.id))
    store.setSelectedEdges(selEdges.map(e => e.id))
  },
  [store],
)
```

ReactFlow fires `onSelectionChange` on **every internal re-render** (even when selection hasn't changed), causing an infinite loop.

## Additional Subtle Bug: Stale Closure

`store.selectedNodeIds` inside `useCallback([store])` **always reads `[]`** (the initial value).

In Zustand v5, `useDiagramStore()` returns a stable object reference, so `useCallback([store])` creates the callback **once** and `store.selectedNodeIds` inside forever reads the initial empty array. This means you CANNOT compare `newIds !== store.selectedNodeIds` to detect changes — it's always `[]`.

**Fix**: Use a `useRef` to track the last selection:

```tsx
const lastSelectionRef = useRef<{ ids: string[]; edgeIds: string[] }>({ ids: [], edgeIds: [] })
const onSelectionChange = useCallback(
  ({ nodes: selNodes, edges: selEdges }: { nodes: Node[]; edges: Edge[] }) => {
    const newIds = selNodes.map(n => n.id)
    const newEdgeIds = selEdges.map(e => e.id)
    const last = lastSelectionRef.current
    const idsChanged = newIds.length !== last.ids.length || newIds.some((id, i) => id !== last.ids[i])
    const edgeIdsChanged = newEdgeIds.length !== last.edgeIds.length || newEdgeIds.some((id, i) => id !== last.edgeIds[i])
    if (idsChanged || edgeIdsChanged) {
      lastSelectionRef.current = { ids: newIds, edgeIds: newEdgeIds }
      store.setSelectedNodes(newIds)
      store.setSelectedEdges(newEdgeIds)
    }
  },
  [store],
)
```

## Why This Breaks the Loop

After the first selection change sets `store.setSelectedNodes(["node1"])`, ReactFlow re-renders and fires `onSelectionChange` again with `["node1"]`. Now `newIds = ["node1"]` and `last.ids = ["node1"]` — `idsChanged = false` → no Zustand update → no re-render → loop broken.

Never call Zustand setters unconditionally inside ReactFlow event callbacks.

**⚠️ Do NOT use `requestAnimationFrame()` as a workaround** — it also runs during the commit phase and triggers the same "setState during render" error. Use the version counter + `useEffect` pattern documented in the SKILL.md instead.

---

## Bug #2: Simulation Shows All Zeros (ReactFlow + Zustand Dual-State Desync)

### Symptom
Simulation runs (no errors), but all component metrics (RPS, P99, Error Rate, Utilization) stay at 0. Properties panel shows empty values. The canvas works fine — nodes drop, connections work.

### Root Cause
ReactFlow maintains its own internal node/edge arrays via `useNodesState`/`useEdgesState`. The Zustand store is a **completely separate** data store. The simulation engine reads from `store.nodes` and `store.edges` (Zustand). But `onDrop` and `onConnect` only update ReactFlow's local state:

```tsx
// onDrop only does this:
setNodes(nds => [...nds, newNode])  // ← only ReactFlow local state, Zustand stays empty

// onConnect only does this:
setEdges(eds => addEdge(edge, eds))  // ← only ReactFlow local state
```

When Play is clicked, the simulation reads `store.nodes` → sees empty array → produces all zeros.

### Fix: Sync Both Stores on Every Mutation

```tsx
// onDrop — after adding to ReactFlow, also sync to Zustand
setNodes(nds => [...nds, newNode])
setTimeout(() => {
  setNodes(currentNodes => {
    const simNodes: SimNode[] = currentNodes.map(n => ({
      id: n.id,
      type: 'simComponent' as const,
      position: n.position,
      data: n.data as SimNode['data'],
    }))
    store.setNodes(simNodes)
    return currentNodes
  })
}, 0)

// onConnect — after adding edge to ReactFlow, also sync to Zustand
setEdges(eds => addEdge(edge, eds))
setTimeout(() => {
  setEdges(currentEdges => {
    const simEdges: SimEdge[] = currentEdges.map(e => ({
      id: e.id,
      source: e.source,
      target: e.target,
      type: 'simConnection' as const,
    }))
    store.setEdges(simEdges)
    return currentEdges
  })
}, 0)
```

The `setTimeout(..., 0)` is critical because `setNodes`/`setEdges` are functional updaters — they take a callback and pass the current state. On the next tick, `currentNodes`/`currentEdges` contains the fully-updated ReactFlow state, which we convert to the Zustand format and push to the store.

**⚠️ UPDATE: `setTimeout(..., 0)` also fails.** It runs during React's commit phase, causing "Cannot update a component while rendering" and silently dropping the Zustand update. `requestAnimationFrame()` has the same problem. The correct approach is the **version counter + `useEffect` pattern** — see the SKILL.md "ReactFlow ↔ Zustand dual-state sync" section for the full pattern.

### The Reverse Sync (Store → ReactFlow)

The `useEffect` + JSON comparison approach for syncing Zustand state changes back to ReactFlow local state is correct and necessary (metrics display on nodes). The `lastStoreVersion` ref prevents infinite loops. But it's only half the picture — you must also sync ReactFlow mutations TO the store, otherwise the simulation has no data to run on.
