# Component Catalog Reference

Complete list of all 50+ components available in the simulator, organized by category.

## Networking (12)
- `load_balancer` - ALB/NLB/CLB, distributes requests, configurable algorithm/health check/SSL. **Cloud Equivalents**: AWS: ALB / NLB / CLB | Azure: Azure Load Balancer / Application Gateway | GCP: Cloud Load Balancing | OSS: HAProxy, NGINX, Traefik, Envoy
- `api_gateway` - Rate limiting, auth, request routing, API versioning. **Cloud Equivalents**: AWS: API Gateway | Azure: API Management | GCP: API Gateway / Cloud Endpoints | OSS: Kong, Traefik, Tyk, KrakenD
- `api_management` - Full lifecycle API management with developer portal. **Cloud Equivalents**: AWS: API Gateway (Enterprise) | Azure: API Management (Premium) | GCP: Apigee | OSS: Kong Enterprise, Tyk, WSO2
- `cdn` - Edge caching (CloudFront), configurable edge locations/TTL/origin shield. **Cloud Equivalents**: AWS: CloudFront | Azure: Azure CDN / Front Door | GCP: Cloud CDN | OSS: Varnish, NGINX, Caddy, Cloudflare (self-hosted)
- `dns` - Domain resolution (Route 53). **Cloud Equivalents**: AWS: Route 53 | Azure: Azure DNS | GCP: Cloud DNS | OSS: CoreDNS, PowerDNS, BIND, Unbound
- `waf` - Web Application Firewall, SQL injection/XSS/bot protection. **Cloud Equivalents**: AWS: WAF | Azure: Azure WAF / Front Door WAF | GCP: Cloud Armor | OSS: ModSecurity, NAXSI, CrowdSec, Coraza
- `vpc` - Virtual Private Cloud, network isolation. **Cloud Equivalents**: AWS: VPC | Azure: Virtual Network (VNet) | GCP: VPC Network | OSS: OpenVPN, WireGuard, Tailscale, Calico
- `nat_gateway` - NAT for private subnet internet access. **Cloud Equivalents**: AWS: NAT Gateway | Azure: NAT Gateway | GCP: Cloud NAT | OSS: iptables/nftables, NAT instance, FRR
- `service_mesh` - mTLS, traffic management, observability (App Mesh). **Cloud Equivalents**: AWS: App Mesh | Azure: Open Service Mesh / Azure Service Mesh | GCP: Traffic Director / Anthos Service Mesh | OSS: Istio, Linkerd, Consul Connect, Kuma
- `sidecar_proxy` - Sidecar proxy for mTLS, traffic splitting, resilience (Envoy/Linkerd). **Cloud Equivalents**: AWS: App Mesh (Envoy) | Azure: Open Service Mesh (Envoy) | GCP: Anthos Service Mesh (Envoy) | OSS: Envoy, Linkerd2-proxy, Traefik Mesh
- `rate_limiter` - Request rate limiting with multiple algorithms and burst handling. **Cloud Equivalents**: AWS: API Gateway Usage Plans / Lambda@Edge | Azure: API Management Rate Limits / Front Door Rules | GCP: API Gateway Quotas / Cloud Armor Rate Limiting | OSS: Envoy Rate Limit, NGINX limit_req, Kong Rate Limiting, Traefik RateLimit
- `circuit_breaker` - Automatic failure detection and circuit breaking for downstream services. **Cloud Equivalents**: AWS: App Mesh / API Gateway | Azure: Open Service Mesh / API Management Policies | GCP: Traffic Director / Cloud Load Balancing | OSS: Istio DestinationRule, Hystrix, Resilience4j, GoBreaker, PyBreaker

## Compute (8)
- `web_server` - App Server (EC2 / Azure App Service / Compute Engine), auto-scaling, CPU/memory config. **Cloud Equivalents**: AWS: EC2 / Elastic Beanstalk | Azure: Azure App Service / VMSS | GCP: Compute Engine / Cloud Run | OSS: Apache, NGINX, Express, Flask, FastAPI
- `microservice` - Single-purpose service with own data store. **Cloud Equivalents**: AWS: ECS / EKS | Azure: Container Apps / AKS | GCP: Cloud Run / GKE | OSS: Spring Boot, Express, FastAPI, Flask
- `serverless` - Lambda/Azure Functions/Cloud Functions, pay-per-use, cold start, concurrency limits. **Cloud Equivalents**: AWS: Lambda | Azure: Azure Functions | GCP: Cloud Functions | OSS: OpenFaaS, Kubeless, Fn Project, Knative
- `container_cluster` - Kubernetes-style (EKS/AKS/GKE), auto-scaling. **Cloud Equivalents**: AWS: EKS / Fargate | Azure: AKS / Container Apps | GCP: GKE / Cloud Run | OSS: Kubernetes, Docker Swarm, Nomad, OpenShift
- `graphql` - Flexible query API with schema stitching (AppSync/Apollo). **Cloud Equivalents**: AWS: AppSync | Azure: API Management / Apollo | GCP: Apollo GraphQL / Cloud Run | OSS: Apollo Server, GraphQL Yoga, Hasura, Mercurius
- `websocket` - Real-time bidirectional communication. **Cloud Equivalents**: AWS: API Gateway (WebSocket) / IoT | Azure: Azure Web PubSub / SignalR | GCP: Cloud Pub/Sub + Cloud Run | OSS: Socket.io, uWebSockets, Gorilla WebSocket, Centrifugo, Phoenix Channels
- `worker` - Background job processor consuming from queues. **Cloud Equivalents**: AWS: EC2 / ECS Worker | Azure: Container Apps / Functions (Worker) | GCP: Cloud Run Jobs / Cloud Tasks | OSS: Sidekiq, Celery, BullMQ, Laravel Queue, Hangfire
- `cron_job` - Scheduled task runner (EventBridge/Logic Apps/Cloud Scheduler). **Cloud Equivalents**: AWS: EventBridge Scheduler / Lambda | Azure: Azure Logic Apps / Functions Cron | GCP: Cloud Scheduler / Cloud Functions | OSS: Linux cron, Kubernetes CronJob, Airflow DAG, Cavalcade

## Data & Storage (10)
- `database` - PostgreSQL (RDS), replication, consistency, IOPS
- `cache` - Redis (ElastiCache), hit ratio, TTL, memory
- `storage` - Object Storage (S3), storage class, versioning, encryption
- `search_engine` - Full-text search (Elasticsearch/OpenSearch)
- `graph_database` - Relationship-focused (Neo4j/Neptune)
- `time_series_db` - Time-stamped metrics (InfluxDB/Timestream)
- `document_store` - Schema-flexible (MongoDB/DocumentDB)
- `key_value_store` - Ultra-low latency (DynamoDB)
- `data_warehouse` - Analytical queries (Redshift)
- `data_lake` - Raw data storage (S3 + Athena)

## Messaging (5)
- `message_queue` - Kafka (MSK), partitioning, retention, delivery guarantee
- `event_bus` - Pub/sub routing (EventBridge)
- `notification_service` - Push notifications (SNS)
- `email_service` - Transactional email (SES)
- `sms_service` - SMS/OTP delivery

## Security (4)
- `identity_provider` - Auth (Cognito, OAuth2/OIDC/SAML)
- `secrets_manager` - API keys, passwords, certificates
- `certificate_manager` - TLS certificate provisioning (ACM)
- `ddos_protection` - DDoS mitigation (Shield Advanced)

## Observability (4)
- `monitoring` - Metrics, dashboards (CloudWatch)
- `logging` - Log aggregation (CloudWatch Logs)
- `tracing` - Distributed tracing (X-Ray)
- `alerting` - Incident alerting (PagerDuty)

## ML/AI (5)
- `ml_model` - Model inference endpoint (SageMaker)
- `ml_training` - Batch training pipeline with GPU
- `feature_store` - Feature management (SageMaker Feature Store)
- `vector_search` - Similarity search for embeddings/RAG
- `recommendation_engine` - Personalized recommendations

## Custom (1)
- `custom_component` - Fully customizable with ALL config fields

## External (2)
- `third_party_api` - External service dependency
- `client` - End user/client application
