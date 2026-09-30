# Multi-Cloud Cost Estimation with Provider Tabs

## Overview
Extended CostEstimate and CostPanel to show costs for AWS, Azure, and GCP simultaneously with a tab switcher.

## Type Changes (src/types/index.ts)

### CostEstimate Interface Extended
```typescript
export interface CostEstimate {
  compute: number
  storage: number
  networking: number
  requests: number
  total: number
  // Multi-cloud totals
  awsTotal: number
  azureTotal: number
  gcpTotal: number
}
```

### ComponentMeta Extended
```typescript
awsCostPerMonth: number
azureCostPerMonth: number
gcpCostPerMonth: number
```

All 47 components now have all three base costs populated.

## CostPanel Implementation (src/components/toolbar/CostPanel.tsx)

### Multi-Cloud Totals Computed Per Tick
```typescript
// Inside useMemo
let awsTotal = 0, azureTotal = 0, gcpTotal = 0
for (const node of store.nodes) {
  const meta = getComponentMeta(node.data.componentType)
  const rps = node.data.metrics?.currentRps || 0
  const rpsCost = (COST_PER_RPS[type] || 0) * rps * 2592000
  
  awsTotal += (meta.awsCostPerMonth || 0) + rpsCost
  azureTotal += (meta.azureCostPerMonth || 0) + rpsCost
  gcpTotal += (meta.gcpCostPerMonth || 0) + rpsCost
}
```

### Provider Tabs
```tsx
<div className="flex border-b border-border bg-bg/50">
  {(['AWS', 'Azure', 'GCP'] as const).map(cloud => (
    <button
      onClick={() => setSelectedCloud(cloud)}
      className={selectedCloud === cloud 
        ? 'flex-1 px-4 py-2 text-xs font-medium text-accent border-b-2 border-accent'
        : 'flex-1 px-4 py-2 text-xs font-medium text-text-dim hover:text-text'
      }
    >
      {cloud}
    </button>
  ))}
</div>
```

### Dynamic Content Based on Selected Cloud
- Total monthly cost: uses `awsTotal` / `azureTotal` / `gcpTotal`
- Annual projection: uses selected total × 12
- Reserved savings: uses selected total × 12 × 0.6
- Breakdown: uses `meta.cloudEquivalents[selectedCloud.toLowerCase()]` for service names

### CostBreakdown Category Totals
Still computed from AWS base (for simplicity) since RPS variable cost is shared across clouds. Only base monthly costs differ by provider.

## Files Modified
- `src/types/index.ts` — CostEstimate + ComponentMeta extensions
- `src/types/components.ts` — All 47 components get azureCostPerMonth, gcpCostPerMonth
- `src/components/toolbar/CostPanel.tsx` — Full multi-cloud UI with tabs

## Lessons
- Single source of truth: ComponentMeta.cloudEquivalents drives both PropertiesPanel service dropdown AND CostPanel service names
- Variable RPS cost shared across providers (assumes similar pricing), only base cost differs
- Tab switcher is cheap (no recomputation) — just reads different field from memoized costEstimate
- OSS not included in cost tabs (no pricing model) — only AWS/Azure/GCP