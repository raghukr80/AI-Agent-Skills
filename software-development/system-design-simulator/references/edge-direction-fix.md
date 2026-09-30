# Edge Direction Fix: Arrow Always Follows User's Drag Direction

## Status: PARTIALLY WORKING — Still needs refinement

The all-source + onConnectStart approach was tested but users reported it still doesn't work correctly in all cases. The core issue is that ReactFlow's internal connection logic may still override the direction in certain scenarios.

## What Was Tried
1. All 4 handles as `type="source"` — eliminates handle-type-based swapping
2. `ConnectionMode.Loose` — allows any-to-any connections
3. `onConnectStart` tracking — stores `nodeId` of the node the user grabbed
4. Swap detection in `onConnect` — if `connectingNodeRef !== params.source`, swap back

## What Still Fails
Users reported that dragging from Node A's right handle to Node B's top/bottom handle still results in reversed direction (B→A instead of A→B).

## Possible Next Steps to Try
1. **Use `onConnect` with `isSource` param**: ReactFlow v12 may pass an `isSource` boolean in the connection params — check if available
2. **Custom edge rendering**: Instead of relying on ReactFlow's source/target, render labels manually based on `onConnectStart` data
3. **Reverse all edges post-connection**: After creating all edges, check if the direction matches the user's intent based on a separate tracking mechanism
4. **Use `onConnectEnd` instead**: `onConnectEnd` may provide more info about which handle was actually grabbed/released

---

## Problem
ReactFlow auto-swaps source/target in `onConnect` so `source` is always the node with a `source`-type handle. When user drags from a `target` handle on Node A to a `source` handle on Node B, ReactFlow sets `source=B, target=A`, reversing the intended direction. The arrow points B→A instead of A→B.

## Root Cause
ReactFlow's internal logic ensures `source` always corresponds to a `source`-type handle. With both `source` and `target` handles on each node (8 handles total), the direction becomes ambiguous.

## Solution: All Source Handles + onConnectStart Tracking

### Step 1: Make ALL handles `source` type
```tsx
// ComponentNode.tsx — 4 handles, all type="source"
<Handle id="top" type="source" position={Position.Top} className="w-3 h-3 bg-accent border-2 border-surface" />
<Handle id="left" type="source" position={Position.Left} className="w-3 h-3 bg-accent border-2 border-surface" />
<Handle id="right" type="source" position={Position.Right} className="w-3 h-3 bg-accent border-2 border-surface" />
<Handle id="bottom" type="source" position={Position.Bottom} className="w-3 h-3 bg-accent border-2 border-surface" />
```

### Step 2: Use ConnectionMode.Loose
```tsx
import { ConnectionMode } from '@xyflow/react'
// In ReactFlow component:
connectionMode={ConnectionMode.Loose}
```

### Step 3: Track user's grab node with onConnectStart
```tsx
const connectingNodeRef = useRef<string | null>(null)

const onConnectStart = useCallback((_: any) => {
  connectingNodeRef.current = _.nodeId
}, [])

const onConnectEnd = useCallback(() => {
  connectingNodeRef.current = null
}, [])
```

### Step 4: Swap back in onConnect if needed
```tsx
const onConnect: OnConnect = useCallback((params) => {
  let source = params.source!
  let target = params.target!
  let sourceHandle = params.sourceHandle
  let targetHandle = params.targetHandle

  if (connectingNodeRef.current && connectingNodeRef.current !== source) {
    // ReactFlow swapped source/target — reverse back
    const tmpNode = source
    source = target
    target = tmpNode
    const tmpH = sourceHandle
    sourceHandle = targetHandle
    targetHandle = tmpH
  }

  const edge: Edge = {
    id: `edge_${source}_${target}_${Date.now()}_${Math.random().toString(36).slice(2, 6)}`,
    source, target, sourceHandle, targetHandle,
    type: 'smoothstep',
    animated: store.simState === 'running',
    style: { stroke: 'var(--color-accent)', strokeWidth: 2 },
    markerEnd: { type: 'arrowclosed' as const, color: 'var(--color-accent)', width: 15, height: 15 },
    data: { label: '', protocol: 'HTTP', bandwidthMbps: 1000, latencyMs: 0, encrypted: false, showArrow: true },
  }
  setEdges(eds => addEdge(edge, eds))
  requestAnimationFrame(syncToStore)
}, [setEdges, store.simState, syncToStore])
```

### Step 5: Wire up onConnectStart/End to ReactFlow
```tsx
<ReactFlow
  onConnect={onConnect}
  onConnectStart={onConnectStart}
  onConnectEnd={onConnectEnd}
  connectionMode={ConnectionMode.Loose}
  // ... other props
/>
```

## Key Insight
`onConnectStart` fires BEFORE ReactFlow processes the connection, so `_.nodeId` reflects the actual node the user grabbed. Compare this with `params.source` in `onConnect` to detect if ReactFlow swapped the direction.

## Common Mistakes
- Using `"loose" as const` — `ConnectionMode` is an enum, not a string union. Must use `ConnectionMode.Loose`
- Using both source AND target handles — this causes the swapping confusion. All-source handles + Loose mode eliminates ambiguity
- Forgetting to pass `sourceHandle`/`targetHandle` explicitly — do NOT use `...params` spread
