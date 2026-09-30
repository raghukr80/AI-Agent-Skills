# SysSimulator Reference Analysis

Source: https://syssimulator.com and https://syssimulator.com/docs (accessed 2026-06-14)

## Product Overview

Browser-based distributed systems simulator. Design architectures on a canvas, run real traffic through them, inject failures, and estimate AWS costs — all locally in the browser. No signup required.

**Key stats**: 18 component types, 28 chaos scenarios, 57 blueprints, 28 system design guides.

## Feature Breakdown

### 1. Visual Diagram Editor
- Infinite canvas with dot grid snapping
- Zoom range: 0.1x–3x
- 18 draggable component types in sidebar palette, grouped by category
- Bezier wire connections with animated packet flow
- Multi-select (click+shift, box select), duplicate (Ctrl+D)
- Undo/redo with 50-entry stack
- Auto-layout: tiers components by category; optional layout zones (named columns)
- Export: JSON, PNG
- Share links: short URLs backed by time-limited server storage (~7 days)
- LocalStorage persistence (local-first)

### 2. Simulation Engine
- **Type**: Discrete Event Simulation (DES)
- **Implementation**: Rust compiled to WebAssembly
- **Execution**: Entirely in browser, no server round-trips
- **Traffic range**: 10–100,000 RPS with configurable speed multipliers
- **Visualization**: Colored particles flow along wires
  - Green = success
  - Yellow = degraded
  - Red = failed
- **Per-component metrics**: Update in real-time during simulation

### 3. Live Metrics
- **Status bar** (during simulation): RPS, system p99, error share, bottleneck count
- **System metrics analysis page**: totals, latency chart (~60 samples), bottleneck list, glossary
- **Properties panel**: Selected components show live metrics during simulation

### 4. Chaos Engineering (28 scenarios, 7 themes)

| Theme | Scenarios |
|-------|-----------|
| Network | Latency injection, partition, packet loss, bandwidth throttle |
| Infrastructure | Node failure, disk full, CPU spike, memory pressure |
| Traffic | Request spike, payload bloat, slow clients, thundering herd |
| Data Layer | Database crash, replication lag, cache stampede, connection pool exhaustion |
| Application | Memory leak, thread pool exhaustion, deadlock, cascading failure |
| Dependency | Third-party timeout, degraded response, error response, rate limit hit |
| MCP/Agents | Tool timeouts, token budget pressure, policy denials, vector index staleness |

MCP/Agent scenarios are blueprint-gated (only unlock on MCP agent blueprints).

### 5. AWS Cost Estimation
- Every component maps to real AWS pricing
- Updates live as you design and adjust traffic
- Breakdown by: Compute, Storage, Networking, Requests

### 6. Blueprints (57 total)
Categories include: E-commerce, Chat, IoT, MCP Agents, Social Media, Video Streaming, URL Shortener, etc.

### 7. System Design Guides (28 total)
- In-depth guides with back-of-envelope estimation
- Failure narration scripts
- Each guide has a matching blueprint you can simulate

## Component Reference (18 types, 5 categories)

### Networking
- API Gateway — rate limiting, routing, auth
- Load Balancer — round-robin, least-connections, weighted
- CDN — edge caching, origin fetch
- DNS — resolution, TTL, round-robin

### Compute
- Web Server — thread pool, request queue
- Serverless — cold start, concurrency limits, per-invocation billing
- Container Cluster — orchestration, scaling
- Agent Runtime — MCP agent execution
- MCP Tool Server — tool serving for agents

### Data
- Database (Postgres) — connection pool, read/write split, replication
- Cache (Redis) — hit/miss ratio, eviction, TTL
- Storage (S3) — read/write latency, eventual consistency
- Vector Store — embedding search, index staleness
- Tool Registry — MCP tool discovery

### Messaging
- Message Queue (Kafka) — topic partitioning, consumer groups, backlog
- Event Bus — pub/sub, event routing

### External
- Third-Party API — external dependency simulation
- Client/User — traffic source

## Tech Stack (from docs)
- **Frontend**: Likely Flutter Web or React (docs mention "Rust + WASM + Flutter Stack" as a deep-dive topic)
- **Simulation Engine**: Rust → wasm-pack → WebAssembly
- **Canvas**: Custom canvas or React Flow-like library
- **Backend** (minimal): Serverless endpoints for share links and optional feedback form
- **Storage**: localStorage (primary), serverless KV for share links

## Comparison to Alternatives

| Feature | SysSimulator | Lucidchart | draw.io | Excalidraw |
|---------|-------------|------------|---------|------------|
| Real-time traffic simulation | Yes | No | No | No |
| Chaos engineering | Yes | No | No | No |
| AWS cost estimation | Yes | No | No | No |
| Runs in browser | Yes | Yes | Yes | Yes |
| Free | Yes | Freemium | Yes | Yes |
| Pre-built blueprints | 57 | Limited | No | No |
| Per-component metrics | Yes | No | No | No |
| No sign-up required | Yes | No | Yes | Yes |

## Architecture Notes

- The WASM engine is the core differentiator — DES running at 100K RPS in-browser
- Particle animation is a canvas overlay on top of the diagram editor
- State is split: diagram state in JS, simulation state in WASM
- The bridge between them translates topology → simulation model on "Play"
- Chaos scenarios are injected as events into the DES queue
- Cost estimation is derived from component configs + simulation throughput data
