# Cloud Equivalents Reference

## Overview

Every component in the simulator includes a `cloudEquivalents` field mapping to the equivalent managed service across the four major cloud providers plus open-source alternatives. This enables cross-cloud architecture design and vendor-agnostic system design.

## Structure

```typescript
interface CloudEquivalents {
  aws: string;      // AWS managed service(s)
  azure: string;    // Azure managed service(s)
  gcp: string;      // GCP managed service(s)
  oss: string;      // Open-source / self-hosted alternatives
}
```

## Display in UI

When hovering over a component in the palette (ComponentPalette.tsx), a tooltip appears showing:
- Component label + description
- "CLOUD EQUIVALENTS" section with four columns:
  - **AWS** (accent color)
  - **Azure** (blue-400)
  - **GCP** (orange-400)
  - **OSS** (green-400)

## Example: Microservice

```typescript
cloudEquivalents: {
  aws: 'ECS / EKS',
  azure: 'Container Apps / AKS',
  gcp: 'Cloud Run / GKE',
  oss: 'Spring Boot, Express, FastAPI, Flask'
}
```

## Implementation Notes

- **Tooltip positioning**: Uses `absolute left-full top-0 ml-2` on the item with `group relative` so it appears to the right of the palette
- **Tooltip visibility**: `hidden group-hover:block z-50` pattern for CSS-only hover
- **Width**: Fixed `w-72` (288px) to accommodate all four providers
- **Scrolling fix**: Parent container must have `overflow-x-visible` (not `overflow-hidden`) and `overflow-y-auto` for the items list
- **Container overflow**: ComponentPalette root and ComponentsTab container must NOT have `overflow-hidden`

## Data Coverage

All 53+ components have cloudEquivalents populated. Categories covered:

| Category | Components | Example |
|----------|------------|---------|
| Networking | 12 | Load Balancer, API Gateway, CDN, DNS, WAF, VPC, NAT Gateway, Service Mesh, API Management, Sidecar Proxy, Rate Limiter, Circuit Breaker |
| Compute | 8 | Web Server, Serverless, Container Cluster, Microservice, GraphQL, WebSocket, Worker, Cron Job |
| Data | 10 | Database, Cache, Storage, Search Engine, Graph DB, Time Series DB, Document Store, Key-Value Store, Data Warehouse, Data Lake |
| Messaging | 5 | Message Queue, Event Bus, Notification Service, Email Service, SMS Service |
| Security | 4 | Identity Provider, Secrets Manager, Certificate Manager, DDoS Protection |
| Observability | 4 | Monitoring, Logging, Tracing, Alerting |
| ML/AI | 5 | ML Model, ML Training, Feature Store, Vector Search, Recommendation Engine |
| Custom | 1 | Custom Component |
| External | 2 | Third-Party API, Client |

## Adding New Components

When adding a new component type, always include `cloudEquivalents` in its COMPONENT_META entry:

```typescript
{
  type: 'new_component',
  label: 'New Component',
  category: 'networking',
  description: '...',
  icon: '🔧',
  defaultConfig: { ... },
  awsService: 'AWS Service Name',
  awsCostPerMonth: 50,
  cloudEquivalents: {
    aws: 'AWS Equivalent',
    azure: 'Azure Equivalent',
    gcp: 'GCP Equivalent',
    oss: 'Open Source Alternative'
  }
}
```

## Design Rationale

The simulator is designed for **system design** (not AWS-specific architecture). Real-world architectures span multiple clouds and on-premises OSS. The cloudEquivalents field makes it easy to:
1. Design vendor-agnostic architectures
2. Compare cloud provider offerings side-by-side
3. Plan multi-cloud or hybrid deployments
4. Understand the OSS technology underlying each managed service