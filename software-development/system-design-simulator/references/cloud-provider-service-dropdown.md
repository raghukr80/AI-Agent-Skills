# Cloud Provider & Service Selection Dropdown

## Overview
Each component node can now be configured for a specific cloud provider (AWS, Azure, GCP, OSS) and a specific service within that provider. This replaces the hardcoded AWS-only service names.

## Data Model Changes

### SimNode (types/index.ts)
```typescript
export interface SimNode {
  // ...existing fields
  data: {
    // ...existing fields
    cloudProvider?: 'aws' | 'azure' | 'gcp' | 'oss'
    cloudService?: string  // e.g., "HAProxy", "Spring Boot", "Cloud Run"
  }
}
```

### ComponentMeta (types/index.ts)
```typescript
export interface ComponentMeta {
  // ...existing fields
  awsService: string          // kept for backward compat (legacy default)
  awsCostPerMonth: number
  azureCostPerMonth: number   // NEW
  gcpCostPerMonth: number     // NEW
  cloudEquivalents: {         // NEW - comma-separated service names per provider
    aws: string
    azure: string
    gcp: string
    oss: string
  }
}
```

All 47+ components now have `azureCostPerMonth`, `gcpCostPerMonth`, and `cloudEquivalents` populated.

## Palette Tooltip
- Hover on any palette item → shows tooltip with all 4 providers' service names
- Uses `cloudEquivalents` object
- No `awsService` subtitle shown in the palette item itself (replaced with description)

## PropertiesPanel - Cloud Provider Selector
```tsx
// 4 buttons: AWS / Azure / GCP / OSS
// Clicking one sets: cloudProvider, resets cloudService to undefined
```

## PropertiesPanel - Cloud Service Dropdown
```tsx
// Dropdown populated by: meta.cloudEquivalents[provider].split(',').map(s => s.trim())
// User selects ONE service (e.g., "HAProxy" from "HAProxy, NGINX, Traefik, Envoy")
// Selection stored in: cloudService
// Displayed on node under the label
```

## Node Canvas Display
```tsx
// ComponentNode.tsx reads: data.cloudProvider || 'aws'
// Gets meta.cloudEquivalents[provider]
// Shows providerService under the node label (text-[9px] text-text-dim)
```

## Cost Calculation
```tsx
// CostPanel.tsx
const baseCost = meta?.awsCostPerMonth || 0 // for AWS
// or
const baseCost = meta?.azureCostPerMonth || 0 // for Azure
// or
const baseCost = meta?.gcpCostPerMonth || 0 // for GCP
// Selected per-node via cloudProvider
```

## Key Files Modified
1. `src/types/index.ts` - SimNode, ComponentMeta interfaces
2. `src/types/components.ts` - COMPONENT_META with new fields for all components
3. `src/components/palette/ComponentPalette.tsx` - Tooltip + removed awsService subtitle
4. `src/components/properties/PropertiesPanel.tsx` - Cloud Provider buttons + Cloud Service dropdown
5. `src/components/canvas/ComponentNode.tsx` - Display selected cloudService
6. `src/components/toolbar/CostPanel.tsx` - Uses cloudProvider for per-node cost

## UX Flow
1. User drags component from palette (tooltip shows all providers)
2. User selects node → PropertiesPanel opens
3. User clicks cloud provider (AWS/Azure/GCP/OSS)
4. User opens Cloud Service dropdown → sees comma-separated list from cloudEquivalents
5. User selects one service → stored in node.data.cloudService
6. Node on canvas updates to show selected service name
7. Cost panel uses provider-specific base cost