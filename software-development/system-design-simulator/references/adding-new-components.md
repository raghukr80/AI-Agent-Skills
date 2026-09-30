# Adding New Component Types

## Complete Process (from adding Sidecar Proxy, Rate Limiter, Circuit Breaker)

### 1. Add to ComponentType Union (src/types/index.ts)
```typescript
export type ComponentType =
  // Networking
  | 'load_balancer'
  | 'api_gateway'
  | 'cdn'
  | 'dns'
  | 'waf'
  | 'vpc'
  | 'nat_gateway'
  | 'service_mesh'
  | 'api_management'
  | 'sidecar_proxy'      // NEW
  | 'rate_limiter'       // NEW
  | 'circuit_breaker'    // NEW
  // ...
```

### 2. Add Config Fields to ComponentConfig (src/types/index.ts)
```typescript
export interface ComponentConfig {
  // Common fields
  maxRps: number
  latencyP50: number
  // ...
  
  // Sidecar Proxy-specific (prefix with unique abbreviations)
  proxyMode?: 'sidecar' | 'gateway' | 'ingress'
  protocol?: 'http' | 'grpc' | 'tcp' | 'http2'
  mtlsEnabled?: boolean
  trafficSplit?: Record<string, number>
  
  // Rate Limiter-specific
  rlAlgorithm?: 'token-bucket' | 'leaky-bucket' | 'fixed-window' | 'sliding-window' | 'sliding-log'
  rateLimitRps?: number
  burstLimit?: number
  keyType?: 'ip' | 'user' | 'header' | 'custom'
  scope?: 'global' | 'per-instance' | 'per-route'
  
  // Circuit Breaker-specific
  failureThreshold?: number
  successThreshold?: number
  cbTimeout?: number
  halfOpenRequests?: number
  monitoredEndpoints?: string[]
  
  // ... other type-specific fields
}
```
**Critical**: Use unique prefixes (`rlAlgorithm`, `lbAlgorithm`, `cbTimeout`) to avoid duplicate property names in TypeScript.

### 3. Add Metadata Entry (src/types/components.ts)
```typescript
{
  type: 'sidecar_proxy',
  label: 'Sidecar Proxy',
  category: 'networking',
  description: 'Sidecar proxy for mTLS, traffic splitting, and resilience patterns (Envoy/Linkerd)',
  icon: '🔀',
  defaultConfig: {
    maxRps: 10000, latencyP50: 2, latencyP95: 5, latencyP99: 20,
    connectionLimit: 10000, failureRate: 0.001,
    proxyMode: 'sidecar', protocol: 'http', mtlsEnabled: true,
    trafficSplit: {},
  },
  awsService: 'App Mesh (Envoy)',
  awsCostPerMonth: 0,
  azureCostPerMonth: 0,
  gcpCostPerMonth: 0,
  cloudEquivalents: {
    aws: 'App Mesh (Envoy)',
    azure: 'Open Service Mesh (Envoy)',
    gcp: 'Anthos Service Mesh (Envoy)',
    oss: 'Envoy, Linkerd2-proxy, Traefik Mesh'
  }
}
```
**Required fields**: `type`, `label`, `category`, `description`, `icon`, `defaultConfig`, `awsService`, `awsCostPerMonth`, `azureCostPerMonth`, `gcpCostPerMonth`, `cloudEquivalents`

### 4. Verify Category Array Includes Type
Check `CATEGORIES` array in `types/components.ts` has `'networking'` — it already does for these.

### 5. Add to Simulation Engine (Rust — if new behavior needed)
- `packages/sim-engine/src/components/mod.rs` — add ComponentType variant
- `packages/sim-engine/src/lib.rs` — add match arm in `add_component`
- Run `wasm-pack build --target web` in `packages/sim-engine`

### 6. Verify
```bash
cd apps/web && npx tsc --noEmit  # TypeScript check
npm run build  # Full build
```

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Duplicate property in ComponentConfig | Use unique prefixes: `rlAlgorithm`, `lbAlgorithm`, `cbTimeout` |
| TypeScript "property missing" on ComponentMeta | Add `azureCostPerMonth`, `gcpCostPerMonth`, `cloudEquivalents` |
| Not showing in palette | Ensure category key matches CATEGORIES array |
| WASM component not recognized | Add to lib.rs match arms + rebuild wasm-pack |
| Config slider doesn't appear | Only renders when `config.fieldName !== undefined` |

## New Networking Components Added (2026-06-30)

| Component | Type | Category | Key Config |
|-----------|------|----------|------------|
| Sidecar Proxy | `sidecar_proxy` | networking | `proxyMode`, `protocol`, `mtlsEnabled`, `trafficSplit` |
| Rate Limiter | `rate_limiter` | networking | `rlAlgorithm`, `rateLimitRps`, `burstLimit`, `keyType`, `scope` |
| Circuit Breaker | `circuit_breaker` | networking | `failureThreshold`, `successThreshold`, `cbTimeout`, `halfOpenRequests`, `monitoredEndpoints` |

All three have full cloud equivalents and cost data.