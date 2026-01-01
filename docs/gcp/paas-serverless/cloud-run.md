# Cloud Run

## Core Concepts

Cloud Run is a fully managed serverless platform for running stateless containers. Deploy any containerized application that responds to HTTP requests without managing infrastructure.

**Key Principle**: Containers that scale to zero, pay per request, any language/library.

## Cloud Run vs Alternatives

| Feature | Cloud Run | App Engine Standard | Cloud Functions | GKE Autopilot |
|---------|-----------|---------------------|-----------------|---------------|
| **Unit** | Container | Runtime sandbox | Function | Pod |
| **Languages** | Any (container) | Specific versions | Specific versions | Any |
| **Scale to zero** | Yes | Yes | Yes | No (min 1) |
| **Cold start** | ~1s | ~100ms | ~1s | N/A |
| **Multi-region** | Yes (managed) | No | No | Manual |
| **Control** | Container-level | Runtime-level | Function-level | Full K8s |

## When to Use Cloud Run

### ✅ Use When

- Stateless HTTP workloads (APIs, web apps)
- Any language/framework (containerized)
- Multi-region deployment needed
- Scale to zero desired
- Pay-per-request model preferred
- Existing containers to deploy

### ❌ Don't Use When

- Long-running background jobs → Cloud Tasks + Cloud Run or Compute Engine
- WebSocket/streaming needed → GKE
- Stateful applications → GKE + StatefulSets
- Sub-100ms latency critical → Keep warm instances

## Architecture Patterns

### Multi-Region Deployment

**Global Load Balancer**:

```
Global LB → Cloud Run (us-central1)
         → Cloud Run (europe-west1)
         → Cloud Run (asia-east1)
```

**Benefits**: Low latency globally, high availability, automatic failover

### Service-to-Service Communication

**Pattern**: Cloud Run services calling each other

**Authentication**: Service accounts + IAM, no public access

### Event-Driven Architecture

**Triggers**:

- Pub/Sub push subscriptions
- Cloud Storage events (via Eventarc)
- Cloud Scheduler (cron)
- Direct HTTP invocation

## Key Features

### Autoscaling

**Configuration**:

- Min instances: 0-1000 (0 for scale to zero)
- Max instances: 1-1000
- Concurrency: 1-1000 requests per instance

**Strategy**: Min 0 for cost, min >0 for latency

### Request Timeout

- Default: 5 minutes
- Max: 60 minutes
- Configurable per service

### CPU Allocation

**Always allocated** (default): CPU always available, faster response
**Allocated during request**: CPU only during request, cheaper

**Decision**: Always allocated for low latency, request-only for cost

### Networking

**Ingress**:

- All: Public internet
- Internal: VPC only
- Internal + Cloud Load Balancing: VPC + global LB

**Egress**: Via VPC connector for private access

## Concurrency Model

**Default**: 80 concurrent requests per instance

**High concurrency** (80-1000): Fewer instances, lower cost, shared memory
**Low concurrency** (1-10): More instances, isolation, predictable performance

**Decision**: High for stateless, low for resource-intensive

## Security

**Default**:

- HTTPS only, managed certificates
- Require authentication by default
- Per-service IAM permissions

**Allow unauthenticated**: `allUsers` invoker role (public APIs)

**VPC Service Controls**: Perimeter-based security

## Cost Model

**Pricing**:

- Request: $0.40 per million
- CPU time: $0.00002400 per vCPU-second
- Memory: $0.00000250 per GiB-second
- Free tier: 2M requests, 360k GiB-seconds/month

**Optimization**:

- Scale to zero when idle
- Right-size CPU/memory
- CPU allocation during request only
- High concurrency

## Cloud Run Jobs

**Purpose**: Run containers to completion (batch, data processing)

**Differences from Services**:

- No HTTP endpoint
- Run to completion, then stop
- Parallel execution support
- Scheduled via Cloud Scheduler

**Use cases**: ETL, batch processing, scheduled tasks

## Integration Patterns

**With Cloud Services**:

- **Cloud SQL**: Direct connection via Unix socket
- **Firestore**: Native client libraries
- **Pub/Sub**: Push subscriptions trigger Cloud Run
- **Cloud Storage**: Eventarc for bucket events
- **Secret Manager**: Environment variables from secrets

**Service Mesh**: Cloud Run integrates with Anthos Service Mesh (advanced)

## Common Patterns

### API Gateway Pattern

```
API Gateway (Cloud Endpoints) → Cloud Run (backend)
```

### Fan-Out Pattern

```
Cloud Run (coordinator) → Pub/Sub → Multiple Cloud Run services
```

### Async Processing

```
Cloud Run (web) → Pub/Sub → Cloud Run (worker)
```

## Limitations

- 60m max request timeout
- 32 GiB max memory per instance
- 4 vCPU max per instance
- Stateless (local disk ephemeral)
- Cold starts (~1s typical)

## Migration Paths

**From App Engine**: Containerize app, deploy to Cloud Run
**From GKE**: Extract stateless services to Cloud Run
**From Compute Engine**: Containerize, remove state

## Exam Focus

### Design Decisions

- Cloud Run vs App Engine vs Cloud Functions
- When to use multi-region
- Concurrency configuration
- CPU allocation strategy

### Architecture

- Service-to-service auth
- Event-driven patterns
- Global deployment
- VPC integration

### Cost Optimization

- Scale to zero
- CPU allocation modes
- Right-sizing
- Concurrency tuning

### Security

- IAM authentication
- Public vs private services
- VPC Service Controls
- Secret management

### Limitations

- Request timeout
- Stateless requirement
- Cold start consideration
- Resource limits
