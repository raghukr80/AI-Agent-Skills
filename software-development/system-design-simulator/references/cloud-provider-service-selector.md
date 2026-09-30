# Cloud Provider & Service Selector Pattern

## Overview

Each component can have a `cloudProvider` (aws/azure/gcp/oss) and `cloudService` (specific service name like "NGINX"). The cloudService is shown on the canvas node.

## Data Model

```typescript
// types/index.ts
export interface SimNode {
  data: {
    cloudProvider?: 'aws' | 'azure' | 'gcp' | 'oss'
    cloudService?: string  // e.g., "NGINX", "Cloud Run"
    // ... other fields
  }
}
```

## Component Meta (all 47 components)

```typescript
// types/components.ts
{
  type: 'load_balancer',
  awsCostPerMonth: 22,
  azureCostPerMonth: 22,
  gcpCostPerMonth: 22,
  cloudEquivalents: {
    aws: 'ALB / NLB / CLB',          // Slash-separated
    azure: 'Azure Load Balancer / Application Gateway',  // Slash-separated
    gcp: 'Cloud Load Balancing',
    oss: 'HAProxy, NGINX, Traefik, Envoy'  // Comma-separated
  }
}
```

## Dropdown Split Logic

Both `/` and `,` are used as separators in cloudEquivalents:
```typescript
const services = cloudEquivalents[provider].split(/[,\/]/)
// "ALB / NLB / CLB" → ["ALB", " NLB", " CLB"]
// "HAProxy, NGINX, Traefik" → ["HAProxy", " NGINX", " Traefik"]
```

## PropertiesPanel Implementation

1. **Provider selector**: Four toggle buttons (AWS/Azure/GCP/OSS)
2. **Service dropdown**: Populated from `cloudEquivalents[provider].split(/[,\/]/)`
3. When switching provider, clear `cloudService: undefined`

## Canvas Node Display

Shows `data.cloudService` if set, otherwise first item from the split list:
```typescript
const providerService = data.cloudService || allServices.split(/[,\/]/)[0]?.trim() || ''
```

## Store Action

```typescript
updateCloudProvider: (nodeId, provider) => {
  set({
    nodes: get().nodes.map(n =>
      n.id === nodeId ? { ...n, data: { ...n.data, cloudProvider: provider } } : n
    ),
  })
}
```
