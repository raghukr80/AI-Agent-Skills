# System Design Tools Infrastructure

## Routing: Hash-Based Page Navigation (NOT Modals)

Tools use **URL hash routing** instead of modal windows:
- `#tools` — Tools library page (category grid, search, recent/favorites)
- `#tools/:toolId` — Individual tool workspace page

**Files**:
```
src/
├── types/tools.ts              # Tool types, categories, inputs/outputs
├── stores/toolsStore.ts        # Zustand store with persist middleware
├── components/tools/
│   ├── ToolsMenu.tsx           # Toolbar button + ⌘K shortcut → sets window.location.hash
│   ├── toolRegistry.ts         # Tool definitions (19 tools, 5 categories)
│   └── index.ts                # Barrel export
└── pages/tools/
    ├── ToolRouter.tsx          # Reads hash, renders ToolsPage or ToolPage
    ├── ToolsPage.tsx           # Full-page library with search, tabs, 2-col grid
    └── ToolPage.tsx            # Full-page workspace with split inputs/outputs
```

**Removed files**: `ToolModal.tsx` and `ToolsPanel.tsx` — deleted after page-routing switch. Only `ToolsMenu.tsx`, `toolRegistry.ts`, and `index.ts` remain in `components/tools/`.

**App routing** (`main.tsx`):
```tsx
function App() {
  const [hash, setHash] = useState(() => window.location.hash);
  useEffect(() => {
    const handler = () => setHash(window.location.hash);
    window.addEventListener('hashchange', handler);
    return () => window.removeEventListener('hashchange', handler);
  }, []);
  
  if (hash.startsWith('#tools')) return <ToolRouter onBack={() => { window.location.hash = '' }} />;
  return <SimulatorCanvas />;
}
```

**ToolRouter** (`pages/tools/ToolRouter.tsx`):
```tsx
function readHash(): Page | null {
  const hash = window.location.hash.slice(1);
  if (!hash.startsWith('tools')) return null;
  const parts = hash.split('/');
  if (parts.length >= 2) return { page: 'tool', toolId: parts.slice(1).join('/') };
  return { page: 'tools' };
}
```

**ToolsMenu** (toolbar button):
```tsx
// Click → window.location.hash = 'tools'
// ⌘K/Ctrl+K shortcut toggles between canvas (#) and tools (#tools)
```

**Don't use react-router** — hash-based routing keeps it dependency-free and simple.

## Tool Page Layout

- **Header**: Back button (←), tool icon, name, difficulty badge, fav/copy/export actions
- **Split**: Inputs (380px left) | Outputs (flex-1 right) with border separator
- **Inputs panel**: Each input renders based on type (number/select/text/checkbox)
- **Outputs panel**: Results computed via `tool.compute(inputs)` with useMemo
- **History**: Expandable section in inputs panel (last 5 runs, clickable to restore)
- **Footer**: Time estimate, input/output counts

## Current Tool Registry (19 tools, 5 categories)

### Planning & Analysis (3)
| Tool ID | Name | Difficulty | Type |
|---------|------|-----------|------|
| `system-design-calculator` | System Design Calculator | beginner | Numeric |
| `capacity-planning` | Capacity Planning Tool | intermediate | Numeric |
| `cloud-cost-estimator` | Cloud Cost Estimator | intermediate | Numeric |

### Performance & Optimization (4)
| Tool ID | Name | Difficulty | Type |
|---------|------|-----------|------|
| `bandwidth-calculator` | Bandwidth Calculator | beginner | Numeric |
| `latency-simulator` | Latency Simulator | intermediate | Queueing |
| `cache-simulator` | Cache Strategy Simulator | intermediate | Simulator |
| `load-balancer-visualizer` | Load Balancer Visualizer | intermediate | Visualizer |

### Data & Storage (3)
| Tool ID | Name | Difficulty | Type |
|---------|------|-----------|------|
| `database-sizing` | Database Sizing Calculator | intermediate | Numeric |
| `database-selection` | Database Selection Tool | intermediate | Select |
| `sharding-visualizer` | Database Sharding Visualizer | advanced | Visualizer |

### Architecture & Design (6)
| Tool ID | Name | Difficulty | Type |
|---------|------|-----------|------|
| `adr-builder` | ADR Builder | intermediate | Builder |
| `api-design-builder` | API Design Builder | intermediate | Builder |
| `message-queue-designer` | Message Queue Designer | intermediate | Builder |
| `scalability-planner` | Scalability Planning Tool | advanced | Planner |
| `consistency-model-explorer` | Consistency Model Explorer | advanced | Explorer |
| `microservices-decomposer` | Microservices Decomposer | advanced | Decomposer |

### Reliability & Resilience (3)
| Tool ID | Name | Difficulty | Type |
|---------|------|-----------|------|
| `reliability-calculator` | System Reliability Calculator | intermediate | Numeric |
| `rate-limit-tester` | Rate Limit Tester | intermediate | Tester |
| `circuit-breaker-simulator` | Circuit Breaker Simulator | intermediate | Simulator |

## Implementation Phases Completed

- **Phase 1**: Types, store, ToolsMenu, hash routing, ToolsPage, ToolPage, ToolRouter
- **Phase 2**: 7 numeric/select calculators (System Design, Capacity, Cost, Bandwidth, Latency, Database Sizing, Database Selection, Reliability)
- **Phase 3**: 4 builders/selectors (ADR Builder, API Design Builder, Message Queue Designer, Scalability Planner)
- **Phase 4**: 5 simulators/visualizers (Cache Simulator, Load Balancer, Rate Limit Tester, Circuit Breaker, Sharding Visualizer)
- **Phase 5**: 2 advanced tools (Consistency Model Explorer, Microservices Decomposer). Skipped Architecture Diagram Builder per user request.
- **Phase 6** (remaining): 2 guides — Capacity Planning Examples, Load Testing Guide

## Tool Input Types

- `number` — numeric input with min/max/step
- `select` — dropdown (needs `options` array with `{ value, label }` entries)
- `text` — free text, multiline JSON snippets
- `checkbox` — boolean toggle
- `multiselect` — multi-select (TBD)
- `slider` — range slider (TBD)

## Select-type tools vs Numeric tools

Select-type tools use dropdown/checkbox inputs. Their `compute()` returns text-based outputs (`recommended`, `alternatives`, `reasoning`, `topologyDiagram`) rather than numeric values with units. The compute function is a decision matrix or lookup table based on selected dropdown values.

Numeric tools use number/text inputs and return numeric outputs with optional `unit` labels (Mbps, GB, $, %, etc.). Outputs are formatted with `toLocaleString()` for display.

Builder tools generate structured text (ADR markdown, OpenAPI spec JSON, Mermaid diagrams, service lists).

Simulator tools run iterative loops (for-loops over seconds/requests) and produce timelines, distribution visualizations, and statistical outputs.

## Persisted State (localStorage, Zustand persist)

- `recentTools`: last 10 tool IDs opened
- `favoriteTools`: user-starred tool IDs
- `toolHistory`: per-tool history (inputs/outputs/timestamp, max 20)
- `selectedCategory`: last category tab
- `searchQuery`: last search

## Keyboard Shortcut

- `⌘K` (macOS) / `Ctrl+K` (any) — toggles between canvas and tools page
- If on tool sub-page, goes back to tools index

## Adding a New Tool

1. Add definition to `TOOL_REGISTRY` array in `toolRegistry.ts`:
   - `id`, `name`, `shortName`, `description`
   - `category` (ToolCategory), `difficulty`, `estimatedTimeMinutes`
   - `icon` (emoji), `color` (tailwind text class)
   - `inputs: ToolInput[]` — each with key, label, type, defaultValue, min/max/step, required, placeholder
   - `outputs: ToolOutput[]` — each with key, label, type, unit
   - `compute: (inputs) => outputs` — pure function
   - Optional: `tags`
2. Tool auto-appears in ToolsPage (category grouping, search, recents)
3. No route/path registration needed — hash `#tools/${toolId}` works automatically

## Category Colors (UI)

- Planning: blue (`text-blue-400`)
- Performance: yellow (`text-yellow-400`)
- Data: green (`text-green-400`)
- Architecture: purple (`text-purple-400`)
- Reliability: red (`text-red-400`)