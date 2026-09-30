# Blueprint Connector Routing

## Problem
Default blueprint edges connect `top→top` for all nodes, creating a flat ladder-like diagram. Complex architectures need connections from all 4 sides of nodes to produce clean, readable layouts.

## Solution Architecture

### 1. Type Extensions (`types/index.ts`)

```ts
// SimNode.data — added connectors field
export interface SimNode {
  data: {
    connectors?: ('top' | 'bottom' | 'left' | 'right')[]
  }
}

// SimEdge — added 4 fields (sides + handles)
export interface SimEdge {
  sourceSide?: 'top' | 'bottom' | 'left' | 'right'
  targetSide?: 'top' | 'bottom' | 'left' | 'right'
  sourceHandle?: string
  targetHandle?: string
}
```

**Why both side + handle fields?** `sourceSide`/`targetSide` are semantic (which side conceptually connects). `sourceHandle`/`targetHandle` are ReactFlow integration (which DOM handle to route to). The `ce()` helper sets all four so the ReactFlow conversion just copies them through.

### 2. Connector Defaults (Permissive)

Most component types accept all 4 sides for blueprint flexibility:
- `client`, `dns`, `cdn` → `['top']` (ingress only)
- `load_balancer` → `['left', 'right', 'bottom']`
- `api_gateway` → `['top', 'left', 'right', 'bottom']` (hub role)
- `web_server`, `serverless`, `container_cluster`, `database` → `['top', 'bottom', 'left', 'right']` (all sides)
- `cache` → `['left', 'right', 'bottom']`
- `event_bus`, `message_queue` → `['left', 'right']`

### 3. Blueprint Helper `ce()` Sets All 4 Fields

```ts
function ce(id, src, tgt, srcSide?, tgtSide?): SimEdge {
  const s = srcSide || src.data.connectors?.[0] || 'top'
  const t = tgtSide || tgt.data.connectors?.[0] || 'top'
  return {
    id, source: src.id, target: tgt.id, type: 'simConnection',
    sourceSide: s, targetSide: t, sourceHandle: s, targetHandle: t,
  }
}
```

### 4. ReactFlow Sync

Initial sync (store → ReactFlow): copy `sourceHandle`/`targetHandle` directly.
Reverse sync (ReactFlow → store): convert `null` → `undefined` via `?? undefined`.

### 5. Pitfalls

1. **`string | null` vs `string | undefined`**: ReactFlow `Edge.sourceHandle` is `string | null`. SimEdge's is `string | undefined`. Use `e.sourceHandle ?? undefined` when syncing ReactFlow → store.
2. **Function signature changes break callers**: Changing required param count cascades TS errors. Fix all callers before rebuilding.
3. **connectors is 6th arg**: `cn('id', 'type', 'label', x, y, connectors)`. Passing connectors as part of config causes silent type errors.
4. **sourceHandle must match Handle id**: Edge `sourceHandle="right"` requires `<Handle id="right">` on the node.
5. **Preserve all exports**: When refactoring blueprints.ts, never truncate to a single blueprint — users notice immediately.

### 6. Files
- `src/types/index.ts` — SimNode and SimEdge types
- `src/types/blueprints.ts` — Blueprint definitions with cn()/ce() helpers
- `src/components/canvas/ComponentNode.tsx` — Handle definitions (top, left, right, bottom)
- `src/components/canvas/SimulatorCanvas.tsx` — ReactFlow↔Store sync
