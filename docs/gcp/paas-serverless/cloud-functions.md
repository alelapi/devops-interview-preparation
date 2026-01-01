# Cloud Functions

## Core Concepts

Cloud Functions is a serverless execution environment for single-purpose functions triggered by events. Deploy code without infrastructure management.

**Key Principle**: Event-driven, single-purpose functions; use for glue code and event processing, not full applications.

## Generations Comparison

| Feature | 1st Gen | 2nd Gen (Recommended) |
|---------|---------|----------------------|
| **Runtime** | Cloud Functions runtime | Cloud Run infrastructure |
| **Timeout** | 9 minutes | 60 minutes |
| **Instances** | Max 3000 | Max 1000 per region |
| **Concurrency** | 1 request/instance | Up to 1000/instance |
| **Min instances** | 0 only | 0-1000 |
| **Traffic splitting** | No | Yes |
| **Use case** | Simple events | Production workloads |

**Recommendation**: Use 2nd gen for new projects

## When to Use Cloud Functions

### ✅ Use When

- Event-driven automation (Pub/Sub, Storage, Firestore)
- Lightweight API endpoints
- Webhooks and callbacks
- Data transformation pipelines
- Scheduled tasks (simple cron)
- Glue code between services

### ❌ Don't Use When

- Complex applications → Cloud Run
- Long-running tasks (>60m) → Compute Engine
- Stateful processing → Cloud Run + storage
- High concurrency needed → Cloud Run (better concurrency)
- Need HTTP routing → Cloud Run

## Event Sources

### HTTP Functions

**Use cases**: Webhooks, API endpoints, HTTP callbacks

**Invocation**: Direct HTTP request

### Background Functions

**Pub/Sub**:

- Message-driven processing
- Async workflows
- Fan-out patterns

**Cloud Storage**:

- File upload processing
- Image transformation
- Data validation

**Firestore**:

- Document create/update/delete triggers
- Data validation
- Derived data updates

**Firebase**:

- Authentication events
- Realtime Database triggers
- Analytics events

**Cloud Logging**:

- Log-based triggers
- Custom alerting

## Architecture Patterns

### ETL Pipeline

```
Cloud Storage upload → Cloud Function → Transform → BigQuery
```

### Image Processing

```
Storage (upload) → Function (resize) → Storage (thumbnails)
```

### Pub/Sub Fan-Out

```
Pub/Sub message → Multiple Cloud Functions (parallel processing)
```

### API Gateway

```
Client → Cloud Function → Backend services
```

### Serverless Cron

```
Cloud Scheduler → Pub/Sub → Cloud Function
```

## Configuration

### Timeout

- 1st gen: Max 9 minutes
- 2nd gen: Max 60 minutes
- Default: 60 seconds

**Decision**: Set based on expected execution time

### Memory

- Range: 128 MB to 32 GB
- CPU scales with memory
- More memory = higher cost

**Decision**: Start low, increase if needed

### Concurrency (2nd gen only)

- Max 1000 concurrent requests per instance
- More concurrency = fewer cold starts
- Less concurrency = more isolation

### Min Instances

- 0: Scale to zero (cost-effective)
- >0: Reduce cold starts (always warm)

**Cost**: Charged for min instances even when idle

## Cold Starts

**Typical Duration**: 1-2 seconds (varies by runtime and dependencies)

**Mitigation**:

- Min instances > 0
- Lighter dependencies
- Keep functions warm (scheduled ping)
- Use 2nd gen (better concurrency)

**Architecture Decision**: Accept cold starts or pay for min instances

## Networking

**Default**: Public internet egress

**VPC Connector**: Access private resources (Cloud SQL, Memorystore)

**Serverless VPC Access**: Connect to VPC without public IPs

## Security

### IAM

- Function-level permissions
- Invoker role for who can call
- Service account for what function can access

### Authentication

**HTTP Functions**:

- Require authentication (default)
- Allow unauthenticated (`allUsers` invoker)

**Background Functions**: Triggered by events (auth via service)

### Secret Manager

**Integration**: Environment variables from secrets

**Pattern**: Store API keys, credentials securely

## Cost Model

**Pricing** (approximate):

- Invocations: $0.40 per million
- Compute time: Based on memory allocation
- Networking: Egress charged

**Free tier**:

- 2M invocations/month
- 400k GB-seconds/month
- 200k GHz-seconds/month

**Optimization**:

- Right-size memory
- Minimize dependencies (faster startup)
- Scale to zero when idle
- Use 2nd gen (better concurrency)

## Limitations

### 1st Gen

- 1 concurrent request per instance
- 9-minute timeout
- No traffic splitting
- Slower cold starts

### 2nd Gen

- 60-minute timeout
- 1000 instances per region max
- 32 GB memory max
- 4 vCPU max

**Both**:

- Stateless (ephemeral disk)
- No direct inbound connections
- Limited execution environment

## Common Patterns

### Async Processing

```
App Engine/Cloud Run → Pub/Sub → Cloud Function (background work)
```

### File Processing

```
Upload to GCS → Cloud Function → Process → Store result
```

### Database Triggers

```
Firestore write → Cloud Function → Update derived data
```

### Scheduled Jobs

```
Cloud Scheduler → Pub/Sub → Cloud Function → Task execution
```

### Webhook Handler

```
External service → Cloud Function (HTTP) → Process event
```

## Migration to Cloud Run

**When to migrate**:

- Need longer timeout (>60m)
- Complex routing required
- Better concurrency needed
- Multi-region deployment
- Traffic splitting

**How**: Package function in container, deploy to Cloud Run

## Best Practices

### Function Design

- Single purpose per function
- Idempotent (can retry safely)
- Stateless
- Fast execution
- Minimal dependencies

### Error Handling

- Background functions: Throw exception to retry
- HTTP functions: Return appropriate status codes
- Implement retry logic with backoff
- Use dead letter queues (Pub/Sub)

### Monitoring

- Cloud Logging for logs
- Cloud Monitoring for metrics
- Set up alerts for errors
- Track cold start frequency

## Exam Focus

### Design Decisions

- Cloud Functions vs Cloud Run
- 1st gen vs 2nd gen
- Event source selection
- When to use functions vs full app

### Architecture

- Event-driven patterns
- Function chaining
- Pub/Sub fan-out
- VPC integration

### Configuration

- Timeout selection
- Memory allocation
- Concurrency settings
- Min instances trade-off

### Limitations

- Timeout constraints
- Concurrency limits
- Stateless requirement
- Cold start impact

### Cost Optimization

- Right-sizing
- Scale to zero
- 2nd gen efficiency
- Minimize dependencies
