---
name: system-design-simulator
description: Build browser-based system design simulators with React Flow canvas + Rust/WASM discrete-event simulation engine. Use when building architecture diagram editors with traffic simulation, chaos engineering, cost estimation, request tracing, or any interactive distributed-systems visualization. Covers canvas architecture, DES engine design, WASM bridge patterns, particle animation, component modeling, trace request flow, and Vercel deployment.
---

# System Design Simulator

Build a browser-based distributed systems simulator: drag-and-drop architecture components onto a canvas, connect them with wires, run discrete-event traffic simulation, inject chaos failures, and estimate costs — all in the browser.

## When to Use

- Building architecture diagram editors with simulation capabilities
- Implementing discrete-event simulation (DES) engines
- Creating React Flow-based canvas apps with custom node/edge types
- Compiling Rust simulation logic to WebAssembly for browser execution
- Visualizing traffic flow, bottlenecks, and failure cascades on infrastructure diagrams

## Architecture

```
Frontend (React + React Flow + Zustand + TailwindCSS)
  ├── Canvas: infinite pan/zoom, dot grid, bezier edges
  ├── Palette: draggable component sidebar with tabs (Components, Chaos)
  ├── Properties Panel: per-node configuration (label, tag, color, icon, metrics)
  ├── Toolbar: simulation controls (play/pause/stop, speed, traffic, theme, metrics, zones)
  ├── Event Log: bottom-left panel showing real-time simulation events
  ├── Particle Canvas: overlay animation on edges
  └── Layout Zones: resizable background containers (not ReactFlow nodes)

WASM Simulation Engine (Rust → wasm-pack)
  ├── DES Core: event queue (binary heap), time advancement
  ├── Component Models: queue-based behavior per node type
  ├── Chaos Engine: scenario injection and cascade tracking
  └── Cost Module: real-time AWS cost mapping

JS Simulation Fallback
  └── Pure-JS loop when WASM not yet built; reads Zustand store
```

## Phase Order

1. **Canvas** — React Flow with custom nodes, bezier edges, zoom, grid
2. **Editor** — Properties panel, undo/redo, multi-select, export JSON
3. **DES Engine** — Rust event queue, component behaviors, WASM bridge
4. **Visualization** — Particle animation on edges, live metrics overlay
5. **Chaos** — Failure injection scenarios (sidebar tab, not modal)
6. **Cost** — AWS pricing model, live cost breakdown
7. **Blueprints** — Pre-built architecture templates
8. **Polish** — Keyboard shortcuts, editable labels/tags/colors/icons, dark/light mode, event log, simulation report, layout zones

## Key Technical Patterns

### React Flow Canvas Setup
- Use React Flow v12 with `fitView: false` (fitView causes aggressive zoom issues)
- Custom node components need explicit `border` class + `bg-surface`
- `visibility: visible !important` override on nodes
- For native drop events, compute position from pane bounding rect directly
- **ReactFlow v12 NodeProps typing**: Use `any` for props and destructure manually
- **ReactFlow Zustand sync**: See `references/sync-pattern.md` for the definitive pattern

### Keyboard Shortcuts
- Wire in a single `useEffect` on `window.addEventListener('keydown', ...)`
- Guard against input fields: check `tagName` for INPUT/TEXTAREA/SELECT
- Delete/Backspace → remove selected nodes (also remove connected edges) OR selected edges
- Ctrl+Z / Ctrl+Y → undo/redo via Zustand store
- Escape → deselect all

### Zustand v5 Store Typing
- `ReturnType<typeof useStore>` does NOT work in Zustand v5. Use `any` or `useDiagramStore.getState()` inline.
- Keep undo/redo as a history stack of `{ nodes, edges }` JSON snapshots (max ~50 entries)
- **Stale closure warning**: `const store = useDiagramStore()` at component render captures stale state. Inside event handlers and callbacks, ALWAYS use `useDiagramStore.getState()` to get fresh state.

### JS Simulation Fallback Pattern
- Use `setInterval` at ~500ms adjusted by speed multiplier
- Check `store.simState !== 'running'` to stop
- Read `store.activeChaos` each tick and apply effects to targeted nodes
- Read `store.trafficMultiplier` each tick and multiply RPS
- Traffic multiplier defaults to 1x; UI dropdown offers 0.5x, 1x, 2x, 3x, 5x
- Call `store.updateNodeMetrics()`, `store.updateNodeStatus()`, `store.setSystemMetrics()` each tick
- **Test WASM first**: After `simulationController.init()` + `start()`, call `step()` once. If `componentMetrics` is empty, stop WASM and fall back to JS.

### Config Slider with Numeric Input
- Configuration sliders must include a numeric input with up/down stepper buttons alongside the range slider
- Layout: `[▾] [input value] [▴] [unit] ———— [slider]`
- The numeric input and slider must be bidirectionally synced
- Use a local `useState` for the input string to allow intermediate typing
- On blur or Enter, parse the value, clamp to min/max, and call onChange
- For percent values, display as percentage (0-100) but store as decimal (0-1)

### Color Picker (Hex Input + Native Picker)
- Color picker must include: (1) clickable color swatch opening native picker, (2) hex input field with validation, (3) reset button, (4) preset color swatches always visible
- Do NOT use a toggle/show pattern — hex input and presets always visible

### Component-Specific Configuration
- Each component type has unique config fields affecting simulation behavior
- **53+ components across 9 categories**: networking (12), compute (8), data (10), messaging (5), security (4), observability (4), ml/ai (5), custom (1), external (2)
- **UI components**: `ConfigSlider` (numeric + stepper + slider), `ConfigSelect` (dropdown), `ConfigToggle` (switch)
- **updateConfig must accept `number | string | boolean`**

### New Networking Components Added (Session 2026-06-30)
- **Sidecar Proxy** (`sidecar_proxy`): mTLS, traffic splitting, resilience (Envoy/Linkerd). Config: `proxyMode` (sidecar/gateway/ingress), `protocol` (http/grpc/tcp/http2), `mtlsEnabled`, `trafficSplit`
- **Rate Limiter** (`rate_limiter`): Multiple algorithms + burst handling. Config: `rlAlgorithm` (token-bucket/leaky-bucket/fixed-window/sliding-window/sliding-log), `rateLimitRps`, `burstLimit`, `keyType` (ip/user/header/custom), `scope` (global/per-instance/per-route)
- **Circuit Breaker** (`circuit_breaker`): Auto failure detection + circuit breaking. Config: `failureThreshold`, `successThreshold`, `cbTimeout`, `halfOpenRequests`, `monitoredEndpoints`

## Trace Request Flow Implementation (Session 2026-08-18)
**Critical fix for non-functional "Trace Request Flow" button and breadcrumb numbering:**

### 1. Button Toggle Logic
The "Trace Request Flow" button in `CanvasToolbarButtons` was static and never responded to clicks. Fixed to toggle trace mode:

```tsx
// Fixed component in CanvasToolbar.tsx
<button
  onClick={() => {
    if (!isTraceActive) {
      store.setTraceMode(true)  // Start trace mode
    } else {
      store.clearTrace()         // Clear trace when active
    }
  }}
  className={`p-1.5 rounded transition-all ${
    isTraceActive
      ? 'bg-accent text-white'    // Active state
      : 'text-text-dim hover:text-text hover:bg-surface-hover' // Inactive state
  }`}
  title={isTraceActive ? 'Clear Trace' : 'Trace Request Flow'}
>
  <Route className="w-3.5 h-3.5" />
</button>
```

**Key improvements:**
- Active state shows `bg-accent text-white` (green, selected appearance)
- Inactive shows dim hover state
- Dynamic title changes between "Trace Request Flow" and "Clear Trace"

### 2. Sequential Breadcrumb Numbering
Fixed breadcrumb display from conditional zero-based numbers (`{i}`) to consistent sequential numbering (`{i + 1}`):

```tsx
// Fixed breadcrumb mapping in CanvasToolbar.tsx
{tracePath.map((nodeId, i) => {
  const node = store.nodes.find(n => n.id === nodeId)
  const label = node?.data.label || nodeId
  return (
    <div key={i} className="flex items-center gap-1 shrink-0">
      <span className="flex items-center justify-center w-5 h-5 rounded-full bg-accent text-white text-[10px] font-bold shrink-0">
        {i + 1}  // Sequential numbering: 1, 2, 3...
      </span>
      <span className={`text-[10px] font-medium ${...} >
        {label}
      </span>
      {i < tracePath.length - 1 && <ChevronRight className="w-3 h-3 text-text-dim shrink-0" />}
    </div>
  )
})}
```

**Key improvements:**
- Always shows sequential numbers (1, 2, 3...) regardless of node reuse
- Larger (w-5 h-5) and clearer (text-[10px]) than previous sizing
- Consistent formatting across both CanvasToolbar and TraceBreadcrumb components

### 3. Store Actions Added
```tsx
// In stores/diagramStore.ts
setTraceMode: (mode: boolean) => void
traceMode: boolean
tracePath: string[]
traceEdgeNumbers: Record<string, number>  // edgeId -> sequence number
addTraceNode: (nodeId: string) => void
clearTrace: () => void
```

`addTraceNode` validates edge existence between nodes before adding to trace path, ensuring logical flow tracing.

### 4. User Workflow Preferences (CONCISE APPROACH)
**User expressed preference for:**
- Concise, direct fixes with minimal explanation
- Local changes first, wait for confirmation before committing/pushing to GitHub
- Changes made in isolated session before team review

**Updated workflow:**
- Make changes locally and test them locally
- Wait for user confirmation before committing and pushing to GitHub
- Use concise language and focused fixes
- Always respect the user's explicit "do the change locally once i confirm you can commit and push to github" requirement

**Updated skill to reflect:** Future work on this system should follow the user's workflow preferences - always make changes locally first, be concise and direct, and never auto-commit/push without confirmation.

### 5. ComponentNode.tsx Handle Pitfall Fix
**CRITICAL: Removed duplicate edge handles** in `ComponentNode.tsx`. Original had handles at both:
- Lines 88-90: `top` and `left` source handles (before header div)
- Lines 161-167: `top`, `left`, `right`, `bottom` source handles (after metrics)

**Removed lines 88-90** (the dead code). Only keep the bottom set (lines 161-167) with all 4 sides.

**Why this matters:** Duplicate handles cause unexpected behavior in edge connections and ReactFlow interactions.

### 6. Verification
```bash
# Build verification
yarn workspace web build  # ✅ PASSED - No errors
npx tsc --noEmit        # ✅ PASSED - TypeScript clean

# Development server
yarn workspace web dev  # ✅ STARTED on port 3000
```

### ReactFlow Viewport Transform for Drag-and-Drop
- **CRITICAL**: `useReactFlow().project()` does NOT exist in ReactFlow v12 — it throws `Property 'project' does not exist`
- **CRITICAL**: `useReactFlow().screenToFlowPosition()` works but REQUIRES a `<ReactFlowProvider>` wrapping the component. If the hook is called outside that context, it crashes the entire page (black screen)
- **Workaround**: Read the viewport transform directly from the DOM:
  ```ts
  const pane = document.querySelector('.react-flow__viewport') as HTMLElement | null
  let viewportX = 0, viewportY = 0, zoom = 1
  if (pane) {
    const style = window.getComputedStyle(pane)
    const matrix = new DOMMatrix(style.transform)
    viewportX = matrix.m41  // translateX
    viewportY = matrix.m42  // translateY
    zoom = matrix.a         // scale
  }
  const position = {
    x: (e.clientX - bounds.left - viewportX) / zoom,
    y: (e.clientY - bounds.top - viewportY) / zoom,
  }
  ```
- Old approach `(e.clientX - bounds.left - 75)` fails because the -75 is a magic number that only works at default zoom. It breaks when zoomed or panned
- **Apply to**: `onDrop` handler in `SimulatorCanvas.tsx`
- **Pitfall**: Using `useReactFlow()` in a component that doesn't have `<ReactFlowProvider>` as ancestor causes a black screen crash. The hook IS used elsewhere in the same file (ZoomButtons), but that works because `useReactFlow` only fails when you try to call its methods without context — the component itself doesn't crash until the method runs

### CSS Overflow Clipping vs Tooltip Visibility
- **Problem**: Tooltips with `absolute` positioning inside a scrollable container with `overflow-y-auto` or `overflow-hidden` get clipped and are invisible
- **Root cause**: Parent containers in the component palette hierarchy all had `overflow-hidden` to enable scrolling — this clips absolutely-positioned children
- **Fix sequence** (applied in order):
  1. Add `relative` to the parent div (`group relative`) — required for absolute children
  2. Change tooltip to `absolute left-full top-0 ml-2` (right side) instead of `absolute bottom-full left-1/2 -translate-x-1/2` (above center) — prevents left-edge clipping
  3. Add `overflow-x-visible` to the scroll container — allows horizontal overflow (tooltip extends right)
  4. Keep `overflow-hidden` on the outermost container for vertical scroll — prevents the whole palette from growing
- **Pattern**: `overflow-hidden` on the outermost flex container, `overflow-y-auto overflow-x-visible` on the scrollable inner div, `relative` on each item, `absolute left-full` on tooltips
- **Pitfall**: Removing `overflow-hidden` from the scroll container breaks vertical scrolling — users can't scroll to see all components

### PropertiesPanel Config Persistence Fix
- **Problem**: Config values (like Max RPS) revert to defaults when switching between nodes
- **Root cause**: The `config` variable was untyped (`selectedNode.data.config` returns the raw store type). TypeScript couldn't infer reactive updates, so ConfigSlider components received stale values
- **Fix**: Add explicit type cast: `const config = selectedNode.data.config as ComponentConfig`
- **Also**: When updating config, sync BOTH the Zustand store AND the ReactFlow nodes state:
  ```ts
  const updateConfig = (key: keyof ComponentConfig, value: number | string | boolean) => {
    store.updateNodeConfig(selectedNode.id, { [key]: value })
    setNodes(currentNodes =>
      currentNodes.map(n =>
        n.id === selectedNode.id
          ? { ...n, data: { ...n.data, config: { ...(n.data.config as Record<string, any>), [key]: value } } }
          : n
      )
    )
  }
  ```
- **Why both?** ReactFlow's `setNodes` and Zustand's `updateNodeConfig` are independent state stores. If only Zustand updates, the next `syncToStore` call copies the stale ReactFlow config back to Zustand, overwriting your change

### Multi-Signal Bottleneck Detection
- **Old**: Only used `utilization > 0.8` as the sole bottleneck signal
- **New**: Multi-signal detection with priority ordering:
  1. Utilization > 95% → critical | > 80% → warning
  2. Error rate > 10% → critical | > 5% → warning
  3. P99 latency 4× baseline → critical | 2.5× → warning
  4. Queue depth > 50 → warning
  5. SPOF (single instance, no autoscale) → warning
  6. Database with replicationFactor=1 → warning
- **Cascading failure detection**: When a downstream node fails, generate `cascading_failure` events for upstream nodes
- **Architecture analysis**: SPOF detection runs at each tick, checking for single-instance compute nodes and databases with no replicas
- **Event generation**: Added `spof_detected`, `connection_pool_full`, `memory_pressure`, `consumer_lag`, `node_degraded`, `cascading_failure` event types to the simulation

### System Design Tools (Hash-Based Page Routing)
- **Key architecture decision**: Tools run as full pages with URL hash routing (`#tools`, `#tools/:toolId`), NOT modal windows. User explicitly rejected modal approach — panels/menus should open new pages.\n- **Location**:\n  - Routing: `src/main.tsx` watches `window.location.hash` — `#tools` → `ToolRouter`, otherwise → `SimulatorCanvas`\n  - Router: `src/pages/tools/ToolRouter.tsx` — reads hash, renders `ToolsPage` (index) or `ToolPage` (detail)\n  - Library page: `src/pages/tools/ToolsPage.tsx` — category tabs, search, 2-col grid, recent/favorites\n  - Detail page: `src/pages/tools/ToolPage.tsx` — split inputs/outputs workspace (380px left, flex-1 right)\n  - Toolbar button: `src/components/tools/ToolsMenu.tsx` — sets `window.location.hash = 'tools'` on click\n  - Registry: `src/components/tools/toolRegistry.ts` — tool definitions (19 tools across 5 categories)\n  - Store: `src/stores/toolsStore.ts` — Zustand with persist middleware\n  - Types: `src/types/tools.ts` — Tool, ToolCategory, ToolInput, ToolOutput, selective inputs\n- **Toolbar integration**: `ToolsMenu` component placed in `Toolbar.tsx` between BlueprintPanel and file operations\n- **Keyboard shortcut**: ⌘K / Ctrl+K toggles between canvas and tools page\n- **How it works**: `ToolsMenu` button sets `window.location.hash = 'tools'`. `main.tsx` listens for hashchange and renders `ToolRouter` when hash starts with `#tools`. `ToolRouter` parses hash — `#tools` shows library, `#tools/${id}` shows a specific tool. Back buttons set hash back to `#tools` (from detail) or `#''` (from library to canvas)\n- **Adding a new tool**: Add definition to `TOOL_REGISTRY` array — `id`, `name`, `inputs`, `outputs`, `compute()`. Tool auto-appears in library. No route registration needed.\n- **Reference**: `references/tools-infrastructure.md`\n- **Removed files**: `ToolModal.tsx` and `ToolsPanel.tsx` were deleted post page-routing switch — they were unused. Only `ToolsMenu.tsx`, `toolRegistry.ts`, and `index.ts` remain in `components/tools/`.\n\n### System Design Tools — Current State (19 tools, 5 categories)\n- **Phase 1**: Types, store, ToolsMenu, hash routing, ToolsPage, ToolPage, ToolRouter\n- **Phase 2 (Calculators)**: 8 tools — System Design Calculator, Capacity Planning, Cloud Cost Estimator, Bandwidth Calculator, Latency Simulator (M/M/1, M/M/c), Database Sizing, Database Selection (select-type), Reliability Calculator\n- **Phase 3 (Selectors & Builders)**: 4 tools — ADR Builder (markdown output), API Design Builder (OpenAPI 3.0), Message Queue Designer (Mermaid topology + broker rec), Scalability Planning Tool (bottleneck detection + horizontal/vertical scaling)\n- **Phase 4 (Simulators)**: 5 tools — Cache Strategy Simulator (Zipf/LRU/LFU), Load Balancer Visualizer (RR/LC/IP/Consistent/Weighted + bar chart), Rate Limit Tester (token/leaky/sliding/fixed window), Circuit Breaker Simulator (CLOSED/OPEN/HALF_OPEN state machine), Database Sharding Visualizer (hash/range/consistent + distribution bars)\n- **Phase 5 (Advanced)**: 2 tools — Consistency Model Explorer (6 models: linearizable→eventual + replica simulation), Microservices Decomposer (6 domains: ecommerce/banking/social/logistics/healthcare/streaming + DDD bounded contexts)\n- **Phase 6 (Guides)**: 2 tools remaining per plan — Capacity Planning Examples, Load Testing Guide\n- **Tool input types**: `number` (numeric), `select` (dropdown with `options` array), `text`, `checkbox` (boolean), `multiselect`, `slider`\n- **Select-type tools**: Different from numeric calculators — use dropdown inputs. `compute()` returns text-based outputs (recommendations, alternatives, reasoning). Input schema uses `options: { value, label }[]`.\n- **Tool categories in UI**: `all`, `planning`, `performance`, `data`, `architecture`, `reliability`

### Multi-Cloud Cost Estimation (Session 2026-07-04)
- **CostEstimate interface extended**: Added `awsTotal`, `azureTotal`, `gcpTotal` fields
- **ComponentMeta extended**: Added `azureCostPerMonth`, `gcpCostPerMonth` to all 47 components
- **CostPanel tabs**: Cloud provider switcher (AWS/Azure/GCP) updates all breakdowns dynamically
- **Per-component costs**: Uses provider-specific base cost + shared RPS variable cost
- **Service names**: Displays `cloudEquivalents[provider]` for selected cloud (e.g., "Cloud Run" for GCP)
- **Reference**: `references/multi-cloud-cost-tabs.md`

### Cloud Provider & Service Selector Pattern
- **SimNode**: Added `cloudProvider?: 'aws' | 'azure' | 'gcp' | 'oss'` and `cloudService?: string` to `data`
- **PropertiesPanel**: Four toggle buttons for provider, then a dropdown showing `cloudEquivalents[provider]` split on `[,/]` — both comma AND slash separators
- **Dropdown split**: `cloudEquivalents.aws.split(/[,\/]/)` — AWS uses slashes ("ALB / NLB / CLB"), OSS uses commas ("HAProxy, NGINX, Traefik, Envoy")
- **Canvas node**: Shows `data.cloudService` (selected) not the full comma/slash list. Fallback: first item from the split list
- **Palette subtitle**: Removed `awsService` text, now shows `meta.description` instead (since cloud equivalents are in the tooltip)
- **Store action**: `updateCloudProvider(nodeId, provider)` — also clear `cloudService: undefined` when switching provider
- **Pitfall**: The parent wrapper of dropdown needs `relative` positioning, and the scroll container needs `overflow-x-visible` for the dropdown to be visible

### Cloud Provider & Service Selection (Session 2026-07-04)
- **SimNode extended**: Added `cloudProvider?: 'aws' | 'azure' | 'gcp' | 'oss'` and `cloudService?: string` fields
- **ComponentMeta extended**: Added `azureCostPerMonth`, `gcpCostPerMonth`, `cloudEquivalents` (aws/azure/gcp/oss strings) to all components
- **Palette tooltip**: Shows all four providers with service names on hover (AWS, Azure, GCP, OSS)
- **Palette item label**: Removed `awsService` subtitle, now shows component description
- **PropertiesPanel - Cloud Provider selector**: Four buttons (AWS/Azure/GCP/OSS) to set the cloud provider for the node
- **PropertiesPanel - Cloud Service dropdown**: Dropdown populated from `cloudEquivalents[provider]` (comma-separated list). User selects one specific service (e.g., "HAProxy" from "HAProxy, NGINX, Traefik, Envoy")
- **Node display**: Shows selected `cloudService` under the label (e.g., "HAProxy") instead of generic type
- **Cost calculation**: Uses provider-specific base cost (`awsCostPerMonth`, `azureCostPerMonth`, `gcpCostPerMonth`) based on selected provider
- **Reference**: `references/cloud-provider-service-dropdown.md`

### Adding New Component Types (Process)
1. Add type to `ComponentType` union in `types/index.ts`
2. Add config fields to `ComponentConfig` interface in `types/index.ts` (use unique prefixes: `rlAlgorithm`, `lbAlgorithm`, `cbTimeout` to avoid duplicates)
3. Add metadata entry to `COMPONENT_META` in `types/components.ts` with label, category, description, icon, defaultConfig, awsService, awsCostPerMonth, **and `cloudEquivalents` (aws, azure, gcp, oss)**
4. Add type to appropriate `CATEGORIES` array (already includes 'networking')
5. Run `npx tsc --noEmit` to verify TypeScript
6. Run `npm run build` to verify full build

### Layout Zones (Background Containers — NOT Nodes)
- Layout zones are absolutely-positioned div layers rendered alongside the ReactFlow canvas — they are NOT ReactFlow nodes
- **Critical: Each zone box container uses `pointerEvents: 'none'`** so clicks pass through to nodes inside the zone
- Only interactive sub-elements get `pointerEvents: 'auto'`: the drag header and resize handles
- Background border and label text use `pointerEvents: 'none'` (default) — clicks pass through
- Managed by `useLayoutZones()` hook returning: `zones`, `selectedZoneId`, `addZone`, `updateZone`, `deleteZone`, `renameZone`, `setSelectedZoneId`
- Each zone: `id`, `name`, `x`, `y`, `width`, `height`, `color`, `borderColor`
- **Drag**: Header `onMouseDown` → global `window.addEventListener('mousemove')` → updates zone x/y
- **Resize**: Edge/corner handles → global mousemove → updates width/height (and x/y for n/w edges). Min size 150×100.
- **Rename**: Double-click zone name → inline text input → blur/Enter to save
- **8 color variants** auto-assigned: Indigo, Green, Yellow, Red, Purple, Teal, Orange, Blue
- Resize handles: edges 2-3px thick spanning full edge minus inset, corners 5×5px, all `pointerEvents: 'auto'` with hover highlight
- **Do NOT register zones as ReactFlow node types** — completely separate from node system
- Add Zone button in toolbar (FolderPlus icon) calls `layoutZones.addZone()`
- Render OUTSIDE the reactFlowWrapper div (in the flex column, after the wrapper) so it doesn't overlay the canvas
- **Old `isGroup: true` approach is DEPRECATED** — removed from ComponentType and COMPONENT_META
- **Rendering order matters**: Zones must be rendered AFTER ReactFlow in the DOM so ReactFlow sits on top and receives all mouse events. If zones are rendered before ReactFlow, they sit on top and block all canvas interactions.

### Node Connectors (All 4 Sides, All Source)
- Each node has **4 handles** (NOT 8): one per side, **all `type="source"`**
- Handle IDs: `top`, `left`, `right`, `bottom` (simple, no source/target prefix)
- Handles should be 3×3 with a 2px border for visibility: `className="w-3 h-3 bg-accent border-2 border-surface"`
- **PITFALL: Do NOT duplicate handles.** In `ComponentNode.tsx`, handles must appear ONCE — at the bottom of the JSX, after the metrics section. An earlier version had `top` and `left` handles duplicated at both the top (before header div, lines 88-90) and bottom (lines 161-167) of the component. The first set is dead code and should be removed. Only keep the bottom set with all 4 sides.
- **Why all-source?** Having both source and target handles on each node causes ReactFlow to auto-swaps source/target based on handle type, making edge direction unpredictable. All-source handles + `ConnectionMode.Loose` + `onConnectStart` tracking gives correct direction.
- **Edge connections**: Explicitly pass `sourceHandle` and `targetHandle` from `onConnect` params to the edge object. Do NOT use `...params` spread — construct the edge manually:
  ```tsx
  const edge: Edge = {
    id: `edge_${source}_${target}_${Date.now()}_${Math.random().toString(36).slice(2, 6)}`,
    source,
    target,
    sourceHandle,
    targetHandle,
    type: 'smoothstep',
    ...
  }
  ```
- **Edge direction fix**: See `references/edge-direction-fix.md` for the complete solution. Summary: (1) Make ALL handles `source` type (no target handles), (2) Use `ConnectionMode.Loose`, (3) Track user's grab node with `onConnectStart`, (4) Swap source/target back in `onConnect` if ReactFlow swapped them. Do NOT use `edgesSelectable` prop — it doesn't exist in ReactFlow v12.

### Blueprint Connector Routing (Explicit Port Selection)
- **Problem**: Default blueprint edges connect `top→top` for all nodes, causing spaghetti lines in complex diagrams.
- **Solution**: Nodes declare which sides they accept connections on (`connectors` field). Edges specify `sourceSide`/`targetSide` which map to ReactFlow's `sourceHandle`/`targetHandle` on the Edge object.
- **SimNode.data.connectors**: Optional array `('top' | 'bottom' | 'left' | 'right')[]` — which sides this node can connect on. Used by blueprint helpers to auto-select the right handle.
- **SimEdge type fields**: SimEdge has FOUR optional side/handle fields:
  - `sourceSide` / `targetSide` — semantic: which side the edge conceptually connects to
  - `sourceHandle` / `targetHandle` — ReactFlow: which handle ID to route to (must match Handle `id` prop)
  - The `ce()` helper sets ALL FOUR. The ReactFlow conversion just copies `sourceHandle`/`targetHandle` through.
- **Connector defaults by component type** (permissive — most types accept all 4 sides for blueprint flexibility):
  - `client`, `dns`, `cdn`, `identity_provider` → `['top']` (ingress from above)
  - `load_balancer` → `['left', 'right', 'bottom']`
  - `api_gateway` → `['top', 'left', 'right', 'bottom']` (full access — hub role)
  - `message_queue` → `['left', 'right', 'bottom']`
  - `web_server`, `serverless`, `container_cluster` → `['top', 'bottom', 'left', 'right']` (all sides — can receive or send in any direction)
  - `database` → `['top', 'bottom', 'left', 'right']` (all sides)
  - `storage` → `['top', 'left', 'right']`
  - `event_bus` → `['left', 'right']`
  - `cache` → `['left', 'right', 'bottom']`
- **Blueprint helper pattern**: Use `cn()` for nodes (auto-assigns connectors from defaults, overridable) and `ce()` for edges (auto-selects sourceHandle/targetSide from node connectors):
  ```ts
  function cn(id, type, label, x, y, connectors?, cfg?): SimNode {
    const conns = connectors || CONNECTOR_DEFAULTS[type] || ['top']
    return { id, type: 'simComponent', position: { x, y },
      data: { componentType: type, label, config: {...base, ...cfg}, status: 'idle', connectors: conns } }
  }
  function ce(id, src, tgt, srcSide?, tgtSide?): SimEdge {
    const s = srcSide || src.data.connectors?.[0] || 'top'
    const t = tgtSide || tgt.data.connectors?.[0] || 'top'
    return { id, source: src.id, target: tgt.id, type: 'simConnection',
      sourceSide: s, targetSide: t, sourceHandle: s, targetHandle: t }
  }
  ```
- **Blueprint edge routing example** (3-Tier):
  ```ts
  ce('e3', cdn, lb, 'bottom', 'top'),   // CDN bottom → LB top (vertical stack)
  ce('e4', lb, web1, 'bottom', 'top'),  // LB bottom → Web1 top (fan out down)
  ce('e6', web1, cache, 'bottom', 'top'), // Web1 bottom → Cache top
  ce('e7', web2, cache, 'bottom', 'left'), // Web2 bottom → Cache left
  ce('e10', db1, db2, 'right', 'left'), // DB replication (horizontal)
  ```
- **ReactFlow integration**: When syncing `SimEdge[]` → ReactFlow `Edge[]`, copy `sourceHandle`/`targetHandle` through. Convert `null` (ReactFlow) to `undefined` (SimEdge):
  ```ts
  const rfEdges = simEdges.map(e => ({
    ...e,
    sourceHandle: e.sourceHandle ?? undefined,  // ReactFlow uses string|null, SimEdge uses string|undefined
    targetHandle: e.targetHandle ?? undefined,
    type: 'smoothstep',
  }))
  ```
- **Pitfall**: ReactFlow `Edge.sourceHandle` is `string | null`, but `SimEdge.sourceHandle` is `string | undefined`. When syncing ReactFlow→store, use `e.sourceHandle ?? undefined` to convert null to undefined, or TypeScript will reject the assignment.
- **Pitfall**: The `SimEdge` type must include `sourceSide`, `targetSide`, `sourceHandle`, `targetHandle` fields. The `SimNode.data` type must include `connectors?` field. All added to `types/index.ts`.
- **Pitfall**: When authoring blueprints, the `cn()` helper takes connectors as the 6th positional argument (after x, y), NOT as part of the config object. Passing an array as config causes silent type errors.
- **Pitfall**: When changing a helper function signature (e.g., adding required params to `ce()` or `createEdge()`), ALL callers must be updated. TypeScript catches this but the error messages cascade — fix all callers before rebuilding.
- See `references/blueprint-connector-routing.md` for the full implementation.

### Edge Arrows and Markers
- All edges should have `markerEnd` with `type: 'arrowclosed'` pointing to the target
- Arrow color matches edge stroke color (accent when running, border when idle)
- Arrow size: 15×15px
- **Per-edge toggle**: Each edge has `showArrow` data field (default `true`). When `false`, `markerEnd` is `undefined` (no arrow).
- **EdgePropertiesPanel**: Include "Show Arrow" toggle switch (same pattern as Encrypted toggle)
- **New edges**: Default to `showArrow: true`
- **Rendering**: In the animated state `useEffect`, read `(e.data as Record<string, unknown>)?.showArrow !== false` and conditionally set `markerEnd`
- **Initial sync from store**: Also check `showArrow` when creating edges from `store.edges`
- **onConnect**: Include `showArrow: true` in the edge data object
- See `references/edge-arrows.md` for code examples
- **Export dropdown**: See `references/export-dropdown.md` for the click-outside handler fix (use `click` event, not `mousedown`)
- Replace single Export button with dropdown menu (3 options: JSON, PNG, PDF)
- **PNG export**: Use `html-to-image` `toPng()` on `.react-flow` element at 2x pixel ratio
- **PDF export**: Use `jsPDF` — first capture PNG via `toPng()`, then embed in PDF with auto orientation
- **JSON export**: Existing download-as-JSON functionality
- **CRITICAL: Use `click` event (not `mousedown`) for outside-click handler** — `mousedown` fires before button click handlers, preventing export from executing
- **Use ref to check if click is inside dropdown** — `exportMenuRef.current.contains(e.target)` instead of blanket close
- Close dropdown after each export action
- ReactFlow v12 edges are selectable by default — NO `edgesSelectable` prop needed (it doesn't exist in v12)
- To delete edges: listen for Delete/Backspace key, filter selected edge IDs from store

### Edge Labels (No Default, Double-Click to Edit Inline)
- By default, edges show NO label — connectors are clean
- Double-click any edge to open an **inline text input** at the edge position (NOT a browser prompt — user rejected `prompt()`)
- The inline editor appears at the edge center position with a styled input (accent border, surface bg, shadow)
- Press Enter or blur to save, Escape to cancel
- Labels are stored in `edge.data.label` and persist in the store
- Use `onEdgeDoubleClick` ReactFlow handler + `useState` for `editingEdgeId`, `editingLabel`, `editingPos`
- Use `useEffect` to auto-focus and select the input when editing starts
- Do NOT set default labels like `"Client → DNS"` — users find them noisy
- **Edge Properties Panel** must show node names (not IDs): `store.nodes.find(n => n.id === edge.source)?.data.label || edge.source`
- See `references/inline-edge-label-editor.md` for code examples

### AI Design (Prompt Generation + JSON Import)
- **AI Design Button** (Wand2/wand icon) in the toolbar opens a modal with two tabs
- **Generate Prompt tab**: Auto-generates a structured prompt from current canvas state describing the architecture for AI chatbots (ChatGPT, Claude, etc.). Includes: node list with types/configs, edge connections, available component types, expected JSON response format
- **Import from AI tab**: Textarea to paste JSON from AI chatbots, with validation:
  - Validates node types against component catalog
  - Validates edge references (source/target must exist as node IDs)
  - Auto-generates missing fields (positions, labels)
  - Loads validated design directly into the canvas via `store.loadDiagram()`
- **Prompt structure**: Describes current nodes/connections, asks AI to analyze/improve, specifies exact JSON format for response
- Store: `loadDiagram(nodes, edges)` action replaces current state and pushes to history
- Component catalog for validation: `getAvailableTypes()` returns all valid `ComponentType` strings
- Replace single Export button with dropdown menu (3 options)
- **PNG export**: Use `html-to-image` `toPng()` on `.react-flow` element at 2x pixel ratio
- **PDF export**: Use `jsPDF` — first capture PNG via `toPng()`, then embed in PDF with auto orientation
- **JSON export**: Existing download-as-JSON functionality
- **CRITICAL: Use `click` event (not `mousedown`) for outside-click handler** — `mousedown` fires before button click handlers, preventing export from executing
- **Use ref to check if click is inside dropdown** — `exportMenuRef.current.contains(e.target)` instead of blanket close
- Close dropdown after each export action
- Close dropdown when clicking outside the export menu div

### Canvas Toolbar (Bottom-Right, Above Controls)
- Toolbar buttons are in a **separate `<Panel position="bottom-right">` rendered AFTER the `<Controls>` component** in the JSX
- Use `className="!mb-14"` on the toolbar Panel so it sits above the Controls panel without overlap
- **CRITICAL**: Do NOT nest Controls inside the toolbar Panel — ReactFlow's Controls component uses internal `position: absolute` that breaks flex layout
- **CRITICAL**: Use custom `ZoomButtons` component with `useReactFlow()` hook instead of ReactFlow's `Controls` component — the Controls component cannot be placed inside a flex container without overlapping
- Do NOT float on left side — user rejected that as awkward
- Layout: `flex flex-col gap-1` inside a bordered/surface container
- **Working structure** (both inside a single Panel, stacked vertically):
  ```tsx
  <Panel position="bottom-right">
    <div className="flex flex-col gap-1">
      <div className="flex flex-col gap-1 bg-surface border border-border rounded-lg shadow-lg p-1">
        <CanvasToolbarButtons ... />
      </div>
      <div className="bg-surface border border-border rounded-lg shadow-lg flex flex-col items-center justify-center gap-0.5 p-1">
        <ZoomButtons />
      </div>
    </div>
  </Panel>
  ```
- **If Controls overlaps toolbar**: The issue is Controls' internal `position: absolute` which ignores flex layout. Solution: Replace Controls with custom ZoomButtons using `useReactFlow()` hook:
  ```tsx
  function ZoomButtons() {
    const { zoomIn, zoomOut, fitView } = useReactFlow()
    return (
      <div className="flex flex-col gap-0.5">
        <button onClick={() => zoomIn()} className="p-1 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Zoom In">
          <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
            <circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35M11 8v6M8 11h6"/>
          </svg>
        </button>
        <button onClick={() => zoomOut()} className="p-1 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Zoom Out">
          <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
            <circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35M8 11h6"/>
          </svg>
        </button>
        <button onClick={() => fitView()} className="p-1 rounded text-text-dim hover:text-text hover:bg-surface-hover" title="Fit View">
          <svg className="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
            <path d="M15 3h6v6M9 21H3v-6M21 3l-7 7M3 21l7-7"/>
          </svg>
        </button>
      </div>
    )
  }
  ```
  Place ZoomButtons inside the same Panel as toolbar buttons, stacked vertically with `flex flex-col gap-1`.
- **💡 Suggestions**: Architecture recommendations (isolated nodes, high traffic, missing LB, no data layer, circular deps)
- **📝 Architecture Notes**: Persistent text notes saved to store (`architectureNotes` field)
- **🔀 Trace Request Flow**: Click nodes in sequence to trace request path
  - Click nodes one by one to build the path
  - Edges between consecutive nodes highlight in green with numbered badges (1, 2, 3...)
  - Breadcrumb at top shows the flow path with node names and numbered circles
  - Reset button (↺) to clear and start over
  - Store: `traceMode`, `tracePath`, `traceEdgeNumbers`, `setTraceMode`, `addTraceNode`, `clearTrace`
  - `addTraceNode` validates that an edge exists between last clicked node and new node
  - Edge rendering: check `traceEdgeNumbers[e.id]` to determine if traced — use green color, thicker stroke, larger arrow
  - Edge labels show trace number when traced: `isTraced ? ` ${traceNum} ` : (e.data?.label || '')`

### Toolbar: Editable Diagram Title
- Title sits between the SysSim logo and the action buttons in the toolbar
- Default: "Untitled Design"
- Click to edit: inline text input with accent border
- Press Enter or blur to save, Escape to cancel
- Max 60 characters
- Syncs with `document.title` (`SysSim — [title]`)
- Stored in Zustand store (`title` field, `setTitle` action)
- Implemented as `EditableTitle` component in `Toolbar.tsx`

### Toolbar: Add Zone + Clear Canvas + Auto Align
- **Add Zone button**: FolderPlus icon, calls `layoutZones.addZone()`
- **Clear Canvas button**: Eraser icon (`Eraser` from lucide-react), calls `store.clearDiagram()`
- **Auto Align button**: Grid icon (`LayoutGrid` from lucide-react), calls `store.autoAlign()`
- Auto align algorithm: grid layout with `cols = ceil(sqrt(nodeCount))`, fixed node size (160×100), 40px padding

### Dark/Light Mode Toggle
- Theme toggle in Toolbar, persisted to localStorage
- **CRITICAL: Tailwind config must use CSS variables, not hex values**
- Light mode palette: bg `#f8f9fc`, surface `#ffffff`, text `#1a1a2e`

### Event Log System
- Bottom-left panel, color-coded by severity, **draggable by header**
- Events for: high latency, high errors, high utilization, RPS overflow, chaos injection
- Throttle: max 1 event per node per 3 ticks
- MUST instrument both WASM and JS paths
- **Drag pattern**: See `references/draggable-event-log.md`. Summary: Use `top` positioning (NOT `bottom` — causes inversion), delta-based movement (store start pos + mouse delta), global window listeners for mousemove/mouseup, clamp to `window.innerWidth - 380` / `window.innerHeight - 280`, init at `window.innerHeight - 280` for bottom-left default. Grip cursor on header. Filter button clicks from drag initiation.

### Simulation Report
- Auto-generated on stop, modal with download as `.md`
- Executive summary, node performance table, incident history, recommendations

### Chaos Engineering
- Sidebar tab (not modal), compact active chaos summary
- **Collapsible category layout**: 6 category groups (Network, Infrastructure, Traffic, Data Layer, Application, Dependency) — each with icon, color, description, scenario count, and ▾ collapse/expand
- Scenarios: 29 total across 6 categories (Network 5, Infrastructure 5, Traffic 4, Data Layer 6, Application 6, Dependency 3). When adding scenarios, update the count in the skill doc and the `CHAOS_SCENARIOS` array
- **Target Selection Modal**: When a chaos scenario has multiple compatible nodes, show a target selection modal instead of auto-applying to all
  - If only 1 compatible node → inject directly
  - If multiple compatible nodes → open modal with node list, checkboxes, Select All/Deselect All
  - All nodes pre-selected by default
  - Each node shows: checkbox, icon, label, component type
  - Inject button disabled if no nodes selected
  - Cancel button to go back without injecting
  - Modal renders at `z-[60]` (higher than main chaos modal's `z-50`)
  - Main chaos modal stays open when target selector is shown (don't close it)
  - Target selector modal gets its own backdrop click handler to close just the selector

## User Workflow Preferences

- **Do NOT commit/push to git automatically after every change.** Make changes locally, wait for explicit user confirmation before committing and pushing to GitHub. The user has explicitly said: "do the change locally once i confirm you can commit and push to github". Violating this erodes trust.
- When the user says "no it doesn't work" or "still overlapping" — the issue is usually a subtle detail (absolute positioning, z-index, event order, or type inference). Don't just re-try the same approach; identify the root cause before patching.
- User prefers concise, direct fixes with minimal explanation.

## Common Pitfalls

- **React Flow v12 visibility bug**: `visibility: visible !important` in CSS
- **fitView zoom issue**: Remove `fitView` prop
- **Zustand stale closure**: Always use `useDiagramStore.getState()` in callbacks
- **PropertiesPanel needs direct setNodes**: Pass `setNodes` for instant canvas updates
- **Hardcoded colors break dark/light mode**: Use `var(--color-*)` everywhere, Tailwind config must use CSS variables
- **WASM returns empty metrics**: Test `step()` after init, fall back to JS if empty
- **nodeEventStats stale closure**: Use direct mutation, not `set()`
- **Layout zones must use `pointerEvents: 'none'` on each zone box container**: The renderer wrapper should NOT have pointerEvents override — each zone box manages its own
- **Layout zones are NOT ReactFlow nodes**: Don't register them in `nodeTypes` or create them via the drop handler
- **Global mouse events for zone drag/resize**: Must use `window.addEventListener('mousemove')` not local element events
- **Config fields conditional rendering**: Only render when key exists (`config.cacheHitRatio !== undefined`)
- **Emoji vs URL icon rendering**: Check `data.icon.startsWith('/')` to distinguish SVG URL from emoji string
- **New components not showing in palette**: Adding types to `ComponentType` is not enough — you MUST also add the category to the `CATEGORIES` array in `types/components.ts`
- **Edge selection in ReactFlow v12**: Do NOT use `edgesSelectable` prop — it doesn't exist in v12. Use `ConnectionMode.Loose` (imported from `@xyflow/react`) for loose connections — NOT `"loose" as const` (ConnectionMode is an enum, not string union)
- **Duplicate store actions**: When adding a new action to the store interface, check both the interface declaration AND the implementation object for duplicates
- **Edge connections require explicit handle IDs**: Each handle needs a unique `id` prop. In `onConnect`, explicitly set `sourceHandle` and `targetHandle` from params — do NOT use `...params` spread
- **Stale closure in callbacks**: Always use `useDiagramStore.getState()` inside `setInterval`, `setTimeout`, and event handlers — never use the `store` variable from component render scope
- **Layout zones rendering order**: Zones must be rendered AFTER ReactFlow in the DOM. If rendered before, they sit on top and block ALL canvas interactions (node selection, zoom, pan, connections)
- **Layout zones pointer-events**: Each zone box container must have `pointerEvents: 'none'`. Only the header (for drag) and resize handles should have `pointerEvents: 'auto'`. The renderer wrapper should NOT set pointerEvents — each zone box manages its own
- **Chaos target selection**: When injecting chaos with multiple compatible nodes, show a target selection modal. Don't auto-apply to all nodes. The modal must render at a higher z-index than the main chaos modal, and the main modal must stay open when the selector is shown
- **CRITICAL: Chaos UI is in ComponentPalette.tsx, NOT ChaosPanel.tsx**: The chaos engineering UI lives in `src/components/palette/ComponentPalette.tsx` as the `ChaosTab` function. There is a separate `ChaosPanel.tsx` file that is NOT imported anywhere — do NOT edit it. When asked to modify chaos behavior or UI, always edit `ComponentPalette.tsx`. The chaos panel is a tab in the left sidebar (toggle between Components/Chaos tabs)
- **Draggable window pattern**: See `references/draggable-window-top-positioning.md`. Summary: Use `top` positioning (NOT `bottom` — causes inversion), delta-based movement (store start pos + mouse delta), global window listeners for mousemove/mouseup, clamp to `window.innerWidth - panelWidth` / `window.innerHeight - 40` (40px min visible, NOT panel height), init at `window.innerHeight - 280` for bottom-left default. Grip cursor on header. Filter button clicks from drag initiation.
- **Canvas API cannot parse CSS variables**: `CanvasGradient.addColorStop()` and `ctx.fillStyle` cannot parse `var(--color-success)` — they need actual hex/rgb values. Use hex colors (`#22c55e`) for canvas operations, CSS variables only for DOM/Tailwind. For alpha, append hex alpha to 8-digit hex: `#22c55e4D` (not `var(--color-success)50`)
- **Vite HMR module caching**: Browser may cache old JavaScript modules even when Vite serves updated files. If code changes don't appear: (1) kill Vite server, (2) clear `node_modules/.vite` cache, (3) restart server, (4) hard-reload browser (Ctrl+Shift+R or clear site data). Confirmed via `curl` that Vite serves the right file — the issue is browser-side module cache, not the dev server
- **Vercel build debugging**: See `references/vercel-build-debugging-chain.md` for the complete 9-error chain and fixes (ESM path, tsconfig, WASM, @types/node, lockfile, output directory). Key: `@types/node` in dependencies (not devDeps), `tsc --noEmit` not `tsc -b`, dynamic import for WASM, `vercel.json` in `apps/web/` not root
- **Monorepo dependency placement**: When adding npm packages in a Yarn/npm workspace, add them to the workspace's `package.json` (e.g. `apps/web/package.json`), NOT just the root. Vercel installs deps per-workspace. If a package is only in root `package.json`, the web workspace won't have it and `tsc --noEmit` will fail with "Cannot find module"
- **Yarn to npm conversion**: Delete `yarn.lock`, run `npm install`, update root `package.json` scripts to use `npm run --workspace=web`, remove workspace deps from web `package.json`, create `vercel.json` with npm commands, update README
- **Draggable window pattern**: See `references/draggable-window-top-positioning.md`. Summary: Use `top` positioning (NOT `bottom` — causes inversion), delta-based movement (store start pos + mouse delta), global window listeners for mousemove/mouseup, clamp to `window.innerWidth - panelWidth` / `window.innerHeight - 40` (40px min visible, NOT panel height), init at `window.innerHeight - 280` for bottom-left default. Grip cursor on header. Filter button clicks from drag initiation.
- **Canvas API cannot parse CSS variables**: `CanvasGradient.addColorStop()` and `ctx.fillStyle` cannot parse `var(--color-success)` — they need actual hex/rgb values. Use hex colors (`#22c55e`) for canvas operations, CSS variables only for DOM/Tailwind. For alpha, append hex alpha to 8-digit hex: `#22c55e4D` (not `var(--color-success)50`)
- **Vite HMR module caching**: Browser may cache old JavaScript modules even when Vite serves updated files. If code changes don't appear: (1) kill Vite server, (2) clear `node_modules/.vite` cache, (3) restart server, (4) hard-reload browser (Ctrl+Shift+R or clear site data). Confirmed via `curl` that Vite serves the right file — the issue is browser-side module cache, not the dev server

- `references/component-catalog.md` — Complete catalog of all 50+ components across 9 categories
- `references/tools-infrastructure.md` — System Design Tools: types, store, registry, adding new tools
- `references/sync-pattern.md` — DEFINITIVE sync pattern: syncToStore with useRef guard flags
- `references/editable-nodes.md` — Inline label editing, tags, colors, local SVG icon system, inline icon dropdown
- `references/color-picker.md` — Color picker with hex input, native system picker, preset swatches
- `references/event-log-system.md` — Event log panel, dual-path event generation, simulation report
- `references/config-controls.md` — ConfigSlider, ConfigSelect, ConfigToggle, conditional rendering
- `references/dark-light-mode.md` — Dark/light mode implementation
- `references/selection-desync.md` — Selection desync and deselectVersion fix
- `references/zustand-reactflow-sync.md` — ReactFlow↔Zustand dual-state sync patterns
- `references/reactflow-v12-patterns.md` — ReactFlow v12 integration patterns
- `references/edge-property-panel.md` — Edge selection, deletion, property panel, keyboard shortcuts
- `references/layout-zones.md` — Layout zone system: pointer-events pass-through, drag/resize, naming
- `references/chaos-target-selection.md` — Chaos target selection modal: selective injection vs auto-apply-all
- `references/vercel-build-config.md` — Vercel/production build config: ESM path resolution, tsconfig.node.json, @types/node in dependencies
- `references/wasm-typescript-integration.md` — WASM module TypeScript integration: @ts-nocheck, .d.ts declarations, .gitignore whitelist
- `references/canvas-color-pitfall.md` — Canvas API cannot parse CSS variables; use hex colors for all canvas operations
- `references/yarn-to-npm.md` — Yarn to npm conversion: lockfile, scripts, workspace, Vercel config
- `references/draggable-window-top-positioning.md` — Draggable window pattern: top positioning with delta movement, no inversion
- `references/vercel-build-debugging-chain.md` — Complete 9-error Vercel build debugging chain and fixes
- `references/vercel-output-directory.md` — Vercel output directory detection, vercel.json placement, build context
- `references/vercel-build-debugging.md` — Full transcript of 9 Vercel build errors and fixes (ESM path, tsconfig, WASM, @types/node, lockfile)
- `references/edge-arrows.md` — Edge arrow markers with per-edge toggle (markerEnd, showArrow data field, EdgePropertiesPanel toggle)
- `references/export-dropdown.md` — Export dropdown with PNG/PDF/JSON, click-outside handler must use `click` event not `mousedown`, use ref to check if click is inside dropdown, position panel above Controls (bottom-right)
- `references/edge-direction-fix.md` — Fix edge arrow direction: use onConnectStart to track user's grab node, swap source/target back when ReactFlow auto-swaps
- `references/chaos-panel-location.md` — Chaos UI is in ComponentPalette.tsx, NOT ChaosPanel.tsx (which is dead code)
- `references/trace-request-flow.md` — Trace Request Flow: click nodes in sequence, highlight path with numbered edges, breadcrumb at top
- `references/canvas-toolbar-panels.md` — Canvas toolbar panels: Suggestions, Notes, Trace buttons positioned above Controls (bottom-right), panel positioning and state management
- `references/canvas-toolbar-controls-layout.md` — Fix Controls overlap: replace Controls with custom ZoomButtons using useReactFlow(), stack vertically in single Panel
- `references/ai-design-import.md` — AI Design: prompt generation from canvas, JSON validation, loadDiagram store action
- `references/inline-edge-label-editor.md` — Double-click edge for inline label editor (not browser prompt), use onEdgeDoubleClick + useState + useEffect auto-focus
- `references/cloud-equivalents.md` — Cloud equivalents system: cross-provider mapping (AWS/Azure/GCP/OSS) for every component
- `references/cloud-provider-service-selector.md` — Cloud provider + service dropdown pattern in PropertiesPanel
- `references/blueprint-connector-routing.md` — Blueprint connector routing: nodes declare sides, edges specify sourceSide/targetSide, mapped to ReactFlow sourceHandle/targetHandle
- `references/bottleneck-multi-signal.md` — Multi-signal bottleneck detection with recommendations, tooltip display in palette