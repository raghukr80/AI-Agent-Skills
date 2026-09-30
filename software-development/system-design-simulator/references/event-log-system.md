# Event Log & Simulation Report System

## Event Log Component (`EventLog.tsx`)

Bottom-left collapsible panel on the canvas. Shows real-time simulation events.

### Features
- Color-coded by severity: info (blue), warning (yellow), critical (red)
- Auto-scrolls to latest event; detects manual scroll to pause auto-scroll
- Badge counts for critical/warning events
- Minimizes to small button when collapsed
- Clear button to reset all events

### Store State
```
events: SimEvent[]           // ring buffer, max 500
simTick: number              // increments each simulation tick
showEventLog: boolean        // panel visibility
nodeEventStats: Record<string, NodeEventStats>
showReport: boolean          // report modal visibility
lastReport: SimulationReport | null
```

## Event Generation

Both WASM polling and JS simulation paths must generate events. Event detection (throttled to 1 per node per 3 ticks):

| Condition | Event Type | Severity |
|-----------|-----------|----------|
| P99 > 2.5x baseline | high_latency | warning (critical if >4x) |
| Error rate > 8% | high_error_rate | warning (critical if >20%) |
| Utilization > 85% | high_utilization | warning (critical if >95%) |
| RPS > 110% capacity | rps_overflow | warning (critical if >150%) |
| Chaos injected | chaos_injected | warning |

### Critical: Dual-Path Instrumentation

Both startWASMPolling() and startJSSimulation() MUST:
1. Extract per-node metrics (currentRps, p99Latency, errorRate, utilization)
2. Track nodeEventStats[nodeId] with running totals
3. Check event conditions (with 3-tick throttle)
4. Call store.addEvent()
5. Call store.incrementTick()

The WASM path uses snake_case (current_rps, p99_latency_ms, error_rate, utilization).
The JS path uses camelCase. Handle both.

## Simulation Report

Auto-generated on stop. Contains: executive summary, node performance table, incident history, engineering recommendations.

### Download Format (.md)
Matches syssimulator.com report format: Executive Summary, Node Performance table, Incident History table, numbered Engineering Recommendations.

## Pitfall: Empty Event Log / No Live Metrics

**Symptoms:** StatusBar shows 0 RPS, component nodes show no metrics, event log stays empty.

**Root causes (check in order):**

1. **No nodes on canvas** — Simulation can't run without components. Check `store.nodes.length > 0` before starting.

2. **Stale closure in handlePlay/handleStop** — `const store = useDiagramStore()` at component render time captures stale state. Inside handlers, ALWAYS use `useDiagramStore.getState()`:
   ```
   // ✅ Correct
   const handlePlay = async () => {
     const currentState = useDiagramStore.getState()
     if (currentState.simState === 'paused') { ... }
     const diagram = { nodes: currentState.nodes, edges: currentState.edges }
     if (currentState.nodes.length === 0) { return }
   }
   
   // ❌ Wrong — stale closure
   const store = useDiagramStore()  // captured at render
   const handlePlay = async () => {
     if (store.simState === 'paused') { ... }  // stale!
   }
   ```

3. **WASM returns empty componentMetrics** — The WASM `step()` may succeed but return `{ componentMetrics: {} }` if the Rust serde format doesn't match. Test before committing:
   ```
   const testResult = simulationController.step()
   if (testResult?.systemMetrics && Object.keys(testResult.systemMetrics.componentMetrics || {}).length > 0) {
     startWASMPolling()
   } else {
     simulationController.stop()
     startJSSimulation()  // fallback
   }
   ```

3. **WASM returns empty componentMetrics** — The WASM `step()` may succeed but return `{ componentMetrics: {} }` if the Rust serde format doesn't match the JS parser. The `parseResult()` function expects specific snake_case fields (`total_rps`, `p99_latency_ms`, `error_rate`, `bottleneck_count`, `component_metrics`). If the Rust struct uses different field names, the parsed result will have all zeros/empty objects.

   **Fix: Test WASM output before committing to it:**
   ```typescript
   const testResult = simulationController.step()
   if (testResult?.systemMetrics && Object.keys(testResult.systemMetrics.componentMetrics || {}).length > 0) {
     useWASM = true
     startWASMPolling()
   } else {
     simulationController.stop()
     startJSSimulation()  // JS fallback — always works
   }
   ```
   Add `console.log('[Sim] WASM active')` or `console.log('[Sim] JS fallback')` to indicate which path is active.

4. **WASM polling path doesn't generate events** — Both paths need identical event generation logic. Copy the same event detection (high latency, error rate, utilization, RPS overflow) and stats tracking into both `startWASMPolling()` and `startJSSimulation()`.

## Pitfall: nodeEventStats Not Updating in Intervals

Inside `setInterval` callbacks, you can't use Zustand's `set()` (it's not a React hook). Use direct mutation:

```
// ✅ Correct
const store = useDiagramStore.getState()
store.nodeEventStats[nodeId] = newNs

// ❌ Wrong — set() not available outside React
set(state => ({ nodeEventStats: { ...state.nodeEventStats, [nodeId]: newNs } }))
```
