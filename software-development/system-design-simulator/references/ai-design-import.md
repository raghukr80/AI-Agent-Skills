# AI Design: Prompt Generation + JSON Import

## Feature Overview

The AI Design feature lets users generate a structured prompt from their current canvas state, paste it into any AI chatbot (ChatGPT, Claude, etc.), and import the AI-generated JSON design back into the app.

## Architecture

```
AIDesignModal (AIDesign.tsx)
├── Generate Prompt Tab
│   ├── Auto-generated prompt from canvas state
│   ├── Copy button (clipboard API)
│   └── Usage instructions
└── Import from AI Tab
    ├── Textarea for JSON paste
    ├── Validation engine
    └── Load into canvas via store.loadDiagram()
```

## Generate Prompt Structure

The prompt describes the current architecture in structured format:
- Node list with types, labels, configs
- Edge connections (source → target)
- Available component types catalog
- Expected JSON response format
- Rules for generation

When canvas is empty, the prompt asks the AI to design from scratch.

## JSON Validation Rules

- `nodes` array required, each node needs `id`, `type`, `label`, `x`, `y`
- `edges` array required, each edge needs `id`, `source`, `target`
- Node `type` must be in the valid component catalog
- Edge `source`/`target` must reference existing node IDs
- Auto-generate missing fields: random positions, default labels

## Store Actions

```typescript
// Add to DiagramStore interface
loadDiagram: (nodes: any[], edges: any[]) => void

// Implementation
loadDiagram: (nodes, edges) => {
  set({ nodes, edges, selectedNodeIds: [], selectedEdgeIds: [] })
  get().pushHistory()
}
```

## Component Catalog for Validation

```typescript
function getAvailableTypes(): string[] {
  return [
    'load_balancer', 'api_gateway', 'api_management', 'cdn', 'dns', 'waf',
    'web_server', 'microservice', 'serverless', 'container_cluster',
    'database', 'cache', 'search_engine', 'graph_database', 'time_series_db',
    'message_queue', 'event_bus', 'notification_service',
    'identity_provider', 'secrets_manager', 'certificate_manager',
    'monitoring', 'logging', 'tracing',
    'ml_model', 'ml_training', 'feature_store',
    'third_party_api', 'client', 'custom_component',
    // ... full list in AIDesign.tsx
  ]
}
```

## Key Files

- `apps/web/src/components/canvas/AIDesign.tsx` — Modal with two tabs
- `apps/web/src/stores/diagramStore.ts` — `loadDiagram` action
- `apps/web/src/components/canvas/CanvasToolbar.tsx` — AI button triggers modal
