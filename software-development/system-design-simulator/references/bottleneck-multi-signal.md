# Multi-Signal Bottleneck Detection

## Overview

Bottleneck detection uses multiple signals (not just utilization) and provides specific recommendations.

## Detection Signals (Priority Order)

1. **Utilization > 95%** → critical
2. **Error rate > 10%** → critical  
3. **P99 latency > 4× baseline** → critical
4. **Utilization > 80%** → warning
5. **Error rate > 5%** → warning
6. **P99 latency > 2.5× baseline** → warning
7. **Queue depth > 50** → warning
8. **SPOF (single instance, no autoscale)** → warning
9. **Database with replicationFactor=1** → warning

## Event Types Generated

During simulation (JS fallback), these events are generated:
- `high_latency` — when p99 > 2.5× config baseline
- `high_error_rate` — when error rate > 8%
- `high_utilization` — when utilization > 85%
- `utilization_spike` — when utilization > 95%
- `rps_overflow` — when currentRps > 1.1× maxRps
- `spof_detected` — single instance, no autoscale
- `connection_pool_full` — pool usage > 90%
- `memory_pressure` — memory < 4GB
- `consumer_lag` — queue depth > 50
- `node_degraded` — status set to degraded
- `cascading_failure` — downstream failure affecting upstream

## MetricsPanel Implementation

- `getBottleneckReason(node)` — returns { reason, rec, severity } or null
- `getSPOFBottlenecks(nodes, edges)` — checks for single instances, no replicas
- Sorted by severity (critical > warning > info), then by utilization
- Deduplicated by label, keeping highest severity
