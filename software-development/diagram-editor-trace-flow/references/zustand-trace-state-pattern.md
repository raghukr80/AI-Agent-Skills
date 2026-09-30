# Zustand Trace State Pattern

## Store Shape

```typescript
interface DiagramStore {
  // Trace state
  traceMode: boolean
  tracePath: string[]              // Visited node IDs in order
  traceEdgeNumbers: Record<string, number>  // edgeId → step number
  traceEdges: Record<string, number>        // Alias for UI convenience
  traceStepCounter: number         // Highest step assigned

  // Actions
  setTraceMode: (mode: boolean) => void
  addTraceNode: (nodeId: string) => void
  clearTrace: () => void
  autoPopulateTrace: () => void
}
```

## Implementation Pattern

### setTraceMode with Auto-Populate

```typescript
setTraceMode: (mode: boolean) => {
  if (mode) {
    set({
      traceMode: true,
      tracePath: [],
      traceEdgeNumbers: {},
      traceEdges: {},
      traceStepCounter: 0
    })
    // Defer to next tick so state is committed before autoPopulate reads it
    setTimeout(() => get().autoPopulateTrace(), 0)
  } else {
    set({
      traceMode: false,
      tracePath: [],
      traceEdgeNumbers: {},
      traceEdges: {},
      traceStepCounter: 0
    })
  }
}
```

### clearTrace

```typescript
clearTrace: () => set({
  tracePath: [],
  traceEdgeNumbers: {},
  traceEdges: {},
  traceStepCounter: 0
})
```

### addTraceNode (Manual Mode)

```typescript
addTraceNode: (nodeId: string) => {
  const path = [...get().tracePath]
  if (path.length === 0) {
    path.push(nodeId)
    set({ tracePath: path })
    return
  }
  const lastNode = path[path.length - 1]
  if (nodeId === lastNode) return
  
  const edge = get().edges.find((e: any) =>
    (e.source === lastNode && e.target === nodeId) ||
    (e.source === nodeId && e.target === lastNode)
  )
  if (!edge) return
  
  path.push(nodeId)
  const numbers = { ...get().traceEdgeNumbers }
  const step = get().traceStepCounter + 1
  numbers[edge.id] = step
  set({ tracePath: path, traceEdgeNumbers: numbers, traceStepCounter: step })
}
```

## UI Consumption

```tsx
const tracePath = useDiagramStore(s => s.tracePath)
const traceEdgeNumbers = useDiagramStore(s => s.traceEdgeNumbers)
const edges = useDiagramStore(s => s.edges)

// Find edge step between consecutive nodes
tracePath.map((nodeId, i) => {
  let edgeStep = null
  if (i < tracePath.length - 1) {
    const next = tracePath[i + 1]
    const edge = edges.find(e => 
      (e.source === nodeId && e.target === next) ||
      (e.source === next && e.target === nodeId)
    )
    edgeStep = edge ? traceEdgeNumbers[edge.id] : null
  }
  return <NodeWithEdgeStep key={i} nodeId={nodeId} edgeStep={edgeStep} />
})
```

## TypeScript Safety

Ensure interface includes all actions — missing `autoPopulateTrace` causes TS2552:

```typescript
// Interface MUST include:
autoPopulateTrace: () => void
```

## Key Invariants

1. **`traceEdges` mirrors `traceEdgeNumbers`** for UI convenience
2. **`traceStepCounter` tracks max assigned step** for manual `addTraceNode`
3. **All state cleared together** on `clearTrace()` and `setTraceMode(false)`
4. **Auto-populate uses setTimeout(..., 0)** to read committed state