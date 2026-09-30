# ReactFlow <-> Zustand Dual-State Sync

## Anti-Pattern: setTimeout/requestAnimationFrame in event handlers

Both run during React's commit phase, causing "Cannot update a component while rendering" and silently dropping state updates. NEVER use these to sync to Zustand from ReactFlow event handlers.

## Solution: Version Counter + useEffect

Use a local state counter (nodesVersion). Event handlers increment it via bumpVersion(). A useEffect([nodesVersion]) syncs to Zustand AFTER render completes (safe to call setState). A separate useEffect([store.nodes]) syncs simulation updates back to ReactFlow.

See the "ReactFlow <-> Zustand dual-state sync" pitfall in the main SKILL.md for full code.

## Clear Diagram

store.clearDiagram() must also clear ReactFlow local state. Add a clearVersion counter to the store, increment it in clearDiagram(), and add a useEffect([store.clearVersion]) in SimulatorCanvas that calls setNodes([]) and setEdges([]).

## Debugging (simulation shows all zeros)

1. Does nodesVersion increment after drop?
2. Does useEffect([nodesVersion]) fire and store.nodes become non-empty?
3. Does simulation tick read store.nodes.length > 0?
4. Does store->ReactFlow sync push metrics back?
