# PropertiesPanel Config State Persistence Fix

## Problem

When changing configuration values (e.g., Max RPS) in the PropertiesPanel and then selecting a different node, the config would revert to the previous node's values or defaults. The state was not persisting correctly when switching selection.

## Root Cause

The PropertiesPanel reads `selectedNode` from the Zustand store, but ReactFlow maintains its own `nodes` state. When `store.updateNodeConfig()` updates the store, the ReactFlow `nodes` state is NOT updated. Later, when `syncToStore()` runs (triggered by any node change), it copies the stale ReactFlow node data back to the store, overwriting the config changes.

## Fix

In `PropertiesPanel.tsx`, the `updateConfig` function must update BOTH:
1. The Zustand store (for simulation engine)
2. The ReactFlow nodes state (to prevent syncToStore from overwriting)

```typescript
const updateConfig = (key: keyof ComponentConfig, value: number | string | boolean) => {
  store.updateNodeConfig(selectedNode.id, { [key]: value })
  // Also update ReactFlow nodes state to keep it in sync
  setNodes(currentNodes =>
    currentNodes.map(n =>
      n.id === selectedNode.id
        ? { ...n, data: { ...n.data, config: { ...(n.data.config as Record<string, any>), [key]: value } } }
        : n
    )
  )
}
```

## TypeScript Note

The config spread requires a type cast because `n.data.config` may have an incomplete type at the ReactFlow node level:

```typescript
config: { ...(n.data.config as Record<string, any>), [key]: value }
```

Also ensure the `config` constant is explicitly typed:

```typescript
const config = selectedNode.data.config as ComponentConfig
```

## When This Pattern Applies

Any time you update node data that must survive:
- Node selection changes
- The `syncToStore()` operation
- ReactFlow's internal state management

Other similar cases that need dual updates:
- `updateNodeLabel`
- `updateNodeTag` 
- `updateNodeColor`
- `updateNodeIcon`

The `updateNodeData` function already handles this correctly by calling both `store.updateNodeX()` AND `setNodes()`.

## Verification

After the fix:
1. Select node A
2. Change Max RPS from 2000 to 5000
3. Select node B
4. Select node A again
5. Max RPS should still show 5000 (not revert to 2000)