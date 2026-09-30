# ReactFlow ↔ Zustand Selection Desync

## Problem

After closing the PropertiesPanel (which calls `store.setSelectedNodes([])`), clicking the same node again doesn't reopen the panel. The node appears visually selected (highlighted) but `onSelectionChange` never fires.

## Root Cause

- `store.setSelectedNodes([])` clears Zustand's `selectedNodeIds` → PropertiesPanel hides (conditional render)
- But ReactFlow's **internal** selection state still has the node marked as `selected: true`
- Clicking the same node again → ReactFlow sees no selection change → doesn't fire `onSelectionChange`
- Without `onSelectionChange`, Zustand's `selectedNodeIds` stays empty → PropertiesPanel stays hidden

## Fix

### 1. Store: Add `deselectVersion` and `deselectAll` action

```typescript
// In DiagramStore interface
deselectVersion: number
deselectAll: () => void

// In initial state
deselectVersion: 0

// Action
deselectAll: () => set({
  selectedNodeIds: [],
  selectedEdgeIds: [],
  deselectVersion: get().deselectVersion + 1,
}),
```

### 2. SimulatorCanvas: Watch `deselectVersion` and clear ReactFlow internal selection

```typescript
useEffect(() => {
  if (store.deselectVersion > 0) {
    setNodes(currentNodes => currentNodes.map(n => ({ ...n, selected: false })))
    setEdges(currentEdges => currentEdges.map(e => ({ ...e, selected: false })))
    lastSelectionRef.current = { ids: [], edgeIds: [] }
  }
}, [store.deselectVersion, setNodes, setEdges])
```

### 3. PropertiesPanel: Call `deselectAll()` on close

```typescript
const handleClose = () => {
  store.deselectAll()
}
```

## Pattern: External Store + ReactFlow Selection

This is a specific case of the general pattern: ReactFlow maintains its own internal selection state that is **separate** from any external store. When you need to programmatically change selection from outside ReactFlow:

1. Update the external store (Zustand) for application logic
2. Update ReactFlow's internal state imperatively via `setNodes`/`setEdges`
3. Track selection changes via `onSelectionChange` callback
4. Never assume ReactFlow's internal selection matches your external store

## Related Patterns

- `clearVersion` for clearing all nodes/edges (similar pattern, different trigger)
- `lastSelectionRef` guard to prevent `onSelectionChange` → Zustand infinite loops
