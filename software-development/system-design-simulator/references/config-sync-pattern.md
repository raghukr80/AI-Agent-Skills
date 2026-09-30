# Dual-State Config Sync Pattern (PropertiesPanel.tsx)

## Problem
When editing component config (sliders, selects, toggles), changes persisted to Zustand store but **not** to ReactFlow's local `nodes` state. When switching to another node and back, the ReactFlow state had stale config, and the next `syncToStore()` would overwrite Zustand with stale values.

## Root Cause
Two state sources for node data:
1. **Zustand store** (`diagramStore.nodes`) — source of truth for simulation, export, history
2. **ReactFlow state** (`useNodesState`) — source of truth for canvas rendering

`syncToStore()` copies ReactFlow → Zustand on every user action (drag, connect, etc.). If ReactFlow state is stale, it overwrites Zustand with old data.

## Fix Pattern

### In PropertiesPanel.tsx
```tsx
const updateConfig = (key: keyof ComponentConfig, value: number | string | boolean) => {
  // 1. Update Zustand (source of truth)
  store.updateNodeConfig(selectedNode.id, { [key]: value })
  
  // 2. ALSO update ReactFlow nodes state to keep it fresh
  setNodes(currentNodes =>
    currentNodes.map(n =>
      n.id === selectedNode.id
        ? { ...n, data: { ...n.data, config: { ...(n.data.config as Record<string, any>), [key]: value } } }
        : n
    )
  )
}
```

### Key Points
- **Always update both** when user changes config
- Use type assertion `(n.data.config as Record<string, any>)` because `SimNode['data']['config']` is `ComponentConfig` (interface) which TypeScript doesn't allow spreading directly
- The `setNodes` callback form ensures we're working with latest ReactFlow state
- This pattern applies to any user-driven property change: label, tag, color, icon, config

## Files Using This Pattern
- `apps/web/src/components/properties/PropertiesPanel.tsx` — `updateConfig` function (lines 63-74)

## Anti-Pattern to Avoid
```tsx
// WRONG: Only updates Zustand
const updateConfig = (key, value) => {
  store.updateNodeConfig(selectedNode.id, { [key]: value })
  // Missing setNodes call — config will be lost on next syncToStore()
}
```

## Why Not Single Source of Truth?
ReactFlow's `useNodesState` is optimized for canvas rendering (viewport transforms, selection, drag). Zustand is optimized for app logic (simulation, history, export, persistence). Keeping both synchronized is the pragmatic choice for this architecture.

## Testing Checklist
- [ ] Change config (e.g., Max RPS slider)
- [ ] Click different node in canvas
- [ ] Click back to original node
- [ ] Config should show updated value (not reset to default)
- [ ] Run simulation — metrics should use new config
- [ ] Export JSON — config should be in exported file