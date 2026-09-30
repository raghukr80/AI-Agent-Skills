---
name: diagram-editor-trace-flow
category: software-development
description: Implement request trace/flow visualization in ReactFlow-based diagram editors with graph traversal algorithms (BFS/DFS), parallel vs sequential edge numbering, and auto-population from entry nodes.
trigger_conditions:
  - Building trace/flow visualization features in diagram editors
  - Need to auto-populate execution paths from entry points
  - Modeling parallel vs sequential execution in UI
  - ReactFlow + Zustand state management for trace state
key_patterns:
  - BFS from entry nodes (nodes with no incoming edges) for auto-population
  - Same BFS level = parallel edges → same step number
  - Sequential edges → incrementing step numbers
  - Auto-populate on mode activation via setTimeout(() => get().autoPopulate(), 0)
  - Edge-based step display in breadcrumb (not node-based)
  - Zustand store with traceEdges: Record<edgeId, stepNumber> and traceStepCounter
pitfalls:
  - Don't use node index for step numbers — use edge traversal order
  - Entry nodes = never a target in edges; fallback to first node if cyclic
  - Clear all trace state on deactivation (tracePath, traceEdgeNumbers, traceEdges, traceStepCounter)
  - TypeScript: ensure interface includes all new actions (autoPopulateTrace)
  - BFS queue: process by depth level batches for correct parallel numbering
verification:
  - yarn workspace web build (must pass tsc --noEmit + vite build)
  - npx tsc --noEmit in apps/web (no type errors)
  - Manual test: click trace button → auto-populates with correct parallel/sequential numbers
references:
  - references/bfs-parallel-numbering.md
  - references/zustand-trace-state-pattern.md
---

# Diagram Editor Trace Flow Implementation

## Overview

This skill covers implementing request trace/flow visualization features in ReactFlow-based diagram editors. The core challenge is modeling **parallel execution** (multiple edges from one node at same time) vs **sequential execution** (chained edges) with correct step numbering.

## Core Algorithm: BFS with Level-Based Numbering

```typescript
autoPopulateTrace: () => {
  const { nodes, edges } = get()
  // 1. Find entry nodes (no incoming edges)
  const targetIds = new Set(edges.map(e => e.target))
  const entryNodes = nodes.filter(n => !targetIds.has(n.id))
  const seeds = entryNodes.length > 0 ? entryNodes : [nodes[0]]

  // 2. BFS with depth tracking
  const queue = seeds.map(s => ({ nodeId: s.id, depth: 0 }))
  const visited = new Set<string>()
  const tracePath: string[] = []
  const traceEdgeNumbers: Record<string, number> = {}
  let stepCounter = 0

  while (queue.length > 0) {
    const currentDepth = queue[0].depth
    const batch = queue.filter(item => item.depth === currentDepth)
    queue.splice(0, batch.length)

    const step = stepCounter + 1

    for (const item of batch) {
      if (visited.has(item.nodeId)) continue
      visited.add(item.nodeId)
      tracePath.push(item.nodeId)

      const outEdges = edges.filter(e => e.source === item.nodeId)
      for (const edge of outEdges) {
        if (!(edge.id in traceEdgeNumbers)) {
          traceEdgeNumbers[edge.id] = step  // Same step for parallel edges
          stepCounter = step
          queue.push({ nodeId: edge.target, depth: currentDepth + 1 })
        }
      }
    }
  }

  set({ tracePath, traceEdgeNumbers, traceEdges: {...traceEdgeNumbers}, traceStepCounter })
}
```

## Key Design Decisions

| Aspect | Decision | Rationale |
|--------|----------|-----------|
| Numbering basis | Edge-based (not node-based) | Edges represent execution steps; parallel fan-out = same step |
| Parallel detection | Same BFS depth level | All edges discovered at depth N execute in parallel |
| Auto-trigger | On trace mode activation | Zero-click UX; trace appears instantly |
| State shape | `traceEdges: Record<edgeId, step>` + `traceStepCounter` | O(1) lookup for edge step numbers in UI |

## Zustand Store Pattern

```typescript
interface DiagramStore {
  traceMode: boolean
  tracePath: string[]           // Node visitation order
  traceEdgeNumbers: Record<string, number>  // edgeId → step
  traceEdges: Record<string, number>        // alias for UI
  traceStepCounter: number      // Max step assigned

  setTraceMode: (mode: boolean) => void
  addTraceNode: (nodeId: string) => void
  clearTrace: () => void
  autoPopulateTrace: () => void
}
```

## UI Display Pattern

Show edge step numbers **between nodes** in breadcrumb:

```tsx
{store.tracePath.map((nodeId, i) => {
  let edgeStep = null
  if (i < store.tracePath.length - 1) {
    const next = store.tracePath[i + 1]
    const edge = store.edges.find(e => 
      (e.source === nodeId && e.target === next) ||
      (e.source === next && e.target === nodeId)
    )
    edgeStep = edge ? store.traceEdgeNumbers[edge.id] : null
  }
  return (
    <div key={i}>
      <span>{i + 1}</span>           {/* Node sequence */}
      <span>{label}</span>
      {edgeStep !== null && (
        <span className="badge">{edgeStep}</span>  {/* Edge step */}
      )}
    </div>
  )
})}
```

## Common Pitfalls

1. **Node index vs edge step**: Using `i + 1` for steps shows sequential only. Must use edge traversal.
2. **Cyclic graphs**: Entry node detection fails → fallback to first node.
3. **Stale TypeScript interface**: Adding `autoPopulateTrace` to implementation but not interface causes TS2552.
4. **Missing state reset**: Clear `traceEdges` and `traceStepCounter` on `clearTrace()` and `setTraceMode(false)`.
5. **Queue mutation during iteration**: Use `splice` to remove batch, not `filter` which creates new array.

## Verification Checklist

- [ ] `yarn workspace web build` passes (tsc + vite)
- [ ] `npx tsc --noEmit` in apps/web shows no errors
- [ ] Click trace button → auto-populates full flow
- [ ] Parallel fan-out (LB → App1, App2, App3) all show Step 1
- [ ] Sequential chain (App → Cache → DB) shows Step 2, Step 3
- [ ] Click trace again → clears cleanly