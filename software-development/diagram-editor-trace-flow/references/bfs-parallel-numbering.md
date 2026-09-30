# BFS Parallel Edge Numbering Reference

## Algorithm Overview

The key insight: **edges at the same BFS depth level execute in parallel**. This matches how request flows work in distributed systems — a load balancer fans out to multiple app servers simultaneously.

## Pseudocode

```
function autoPopulateTrace(nodes, edges):
    targetIds = Set(edges.map(e => e.target))
    entryNodes = nodes.filter(n => !targetIds.has(n.id))
    seeds = entryNodes if entryNodes.length > 0 else [nodes[0]]
    
    queue = seeds.map(s => {nodeId: s.id, depth: 0})
    visited = Set()
    tracePath = []
    traceEdgeNumbers = {}
    stepCounter = 0
    
    while queue not empty:
        currentDepth = queue[0].depth
        batch = queue.filter(item => item.depth == currentDepth)
        queue = queue.filter(item => item.depth > currentDepth)  // remove batch
        
        step = stepCounter + 1
        
        for item in batch:
            if item.nodeId in visited: continue
            visited.add(item.nodeId)
            tracePath.append(item.nodeId)
            
            outEdges = edges.filter(e => e.source == item.nodeId)
            for edge in outEdges:
                if edge.id not in traceEdgeNumbers:
                    traceEdgeNumbers[edge.id] = step  // SAME step for all parallel edges
                    stepCounter = step
                    queue.append({nodeId: edge.target, depth: currentDepth + 1})
    
    return {tracePath, traceEdgeNumbers, traceStepCounter: stepCounter}
```

## Example Traces

### Fan-out (Parallel)
```
LB → App1
LB → App2
LB → App3

Step 1: LB→App1, LB→App2, LB→App3 (all same step)
```

### Sequential Chain
```
App → Cache → DB

Step 1: App→Cache
Step 2: Cache→DB
```

### Mixed
```
LB → App1, App2
App1 → Cache
App2 → Cache
Cache → DB

Step 1: LB→App1, LB→App2
Step 2: App1→Cache, App2→Cache
Step 3: Cache→DB
```

## TypeScript Types

```typescript
interface TraceState {
  tracePath: string[]              // Node visitation order
  traceEdgeNumbers: Record<string, number>  // edgeId → step
  traceStepCounter: number         // Max step assigned
}
```