# Bottleneck Detection & Recommendations System

## Overview
Enhanced the bottleneck detection from single-signal (utilization > 80%) to multi-signal with actionable recommendations per bottleneck.

## MetricsPanel Changes (src/components/metrics/MetricsPanel.tsx)

### Before
- Single bottleneck threshold: utilization > 60% (table) vs > 80% (metric card count) — inconsistent
- Only showed label, utilization, RPS, P99
- No recommendations

### After
- Consistent 80% threshold for both count and table
- Multi-signal detection:
  - **Utilization > 95%** → critical
  - **Error rate > 10%** → critical
  - **P99 latency 4× baseline** → critical
  - **Utilization > 80%** → warning
  - **Error rate > 5%** → warning
  - **P99 latency 2.5× baseline** → warning
  - **Queue depth > 50** → warning
- **SPOF detection**: Single instance no autoscale (compute), Database replicationFactor=1
- **Recommendations column**: Each bottleneck shows specific actionable advice
- **Severity badges**: critical (red), warning (amber), info (accent)

### Detection Logic (`getBottleneckReason`)
```typescript
function getBottleneckReason(node): { reason, rec, severity } | null {
  const m = node.data.metrics
  const config = node.data.config
  
  if (m.utilization > 0.95) 
    return { reason: 'Severe overload >95%', rec: 'Increase maxRps or add replicas. Enable autoscaling.', severity: 'critical' }
  if (m.errorRate > 0.1)
    return { reason: `High error rate ${(m.errorRate*100).toFixed(1)}%`, rec: 'Check logs, add circuit breaker, rollback if recent deploy.', severity: 'critical' }
  if (m.p99Latency > config.latencyP99 * 4)
    return { reason: `P99 ${(m.p99Latency/config.latencyP99).toFixed(1)}× baseline`, rec: 'Profile slow queries, add caching, or scale vertically.', severity: 'critical' }
  // ... warning thresholds
  if (!config.autoScale && config.minInstances <= 1 && isComputeType)
    return { reason: 'Single instance — no autoscaling (SPOF)', rec: 'Enable autoscaling with minInstances ≥ 2. Add redundancy.', severity: 'warning' }
  if (node.data.componentType === 'database' && (config.replicationFactor || 1) <= 1)
    return { reason: 'Database with no replicas (SPOF)', rec: 'Increase replicationFactor to ≥ 2. Add read replicas.', severity: 'warning' }
  return null
}
```

### SPOF Detection
- **Compute nodes**: `autoScale === false && minInstances <= 1` + compute type (web_server, microservice, etc.)
- **Database**: `replicationFactor <= 1`
- Runs each render, no persistent state needed

### UI
- Card per bottleneck with colored border (red/amber/accent)
- Left: label + severity badge + reason
- Right: utilization, RPS, P99, error rate (if >1%)
- Bottom: green arrow + recommendation text

## SimulationControls (src/components/toolbar/SimulationControls.tsx)

### New Event Types Generated
| Event Type | Trigger | Severity |
|------------|---------|----------|
| `spof_detected` | Single instance compute, DB no replicas | warning |
| `connection_pool_full` | Current RPS > 90% connectionLimit | warning/critical |
| `memory_pressure` | Config memoryGb < 4GB | warning/critical |
| `consumer_lag` | Queue depth > 50 (messaging) | warning/critical |
| `node_degraded` | Node status === 'degraded' | warning |
| `cascading_failure` | Downstream failed → upstream affected | critical |

### Cascading Failure Detection
- Runs AFTER node metrics computed
- For each `failed` node, find upstream edges
- If upstream not failed, generate `cascading_failure` event
- Throttled: max 1 per upstream per 5 ticks

### Event → Recommendation Mapping
All new event types have entries in `getRecommendation()` map in diagramStore.ts with specific remediation advice.

## Key Files Modified
1. `src/components/metrics/MetricsPanel.tsx` — Complete rewrite with multi-signal + SPOF + recommendations
2. `src/components/toolbar/SimulationControls.tsx` — Added 6 new event types + cascading failure logic
3. `src/stores/diagramStore.ts` — Extended `getRecommendation` map with 21 event types

## Lessons
- Bottleneck count in summary card must match table threshold (both 80%)
- SPOF detection is architecture-aware, not just metric-aware
- Cascading failure detection needs topology (edges) not just node state
- Recommendations must be specific to the event type, not generic