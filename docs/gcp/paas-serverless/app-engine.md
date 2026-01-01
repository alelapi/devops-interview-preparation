# App Engine

## Core Concepts

App Engine is a fully managed PaaS for building web applications and APIs without infrastructure management. Two environments: Standard (sandboxed, instant scaling) and Flexible (containers, more control).

**Key Decision**: Use Standard for simple web apps with variable traffic; Flexible for custom runtimes or background processing.

## Standard vs Flexible

| Feature | Standard | Flexible |
|---------|----------|----------|
| **Scaling** | 0 to N instances (instant) | Min 1 instance (gradual) |
| **Runtime** | Specific language versions | Any Docker container |
| **Startup** | Milliseconds | Minutes |
| **Pricing** | Instance hours ($0.05/hr) | vCPU/memory hours |
| **Free Tier** | 28 instance hours/day | None |
| **SSH** | No | Yes |
| **Timeout** | 60s (10m tasks) | 60m |

## When to Use

### Standard Environment

✅ **Use when**:

- Simple web apps, REST APIs
- Unpredictable traffic (scale to zero)
- Budget-conscious (free tier)
- Supported runtime sufficient
- Fast startup critical

❌ **Don't use when**:

- Need custom runtime/libraries
- Background processing required
- Write to local disk needed
- Need SSH access

### Flexible Environment

✅ **Use when**:

- Custom runtime/dependencies
- Native code libraries
- Background workers
- Existing Docker containers

❌ **Don't use when**:

- Can use Standard (cheaper)
- Need multi-region → Cloud Run
- Need K8s control → GKE

## Architecture Patterns

### Services (Microservices)

```
app.yaml           → default service (frontend)
api.yaml           → api service (backend)
worker.yaml        → worker service (async tasks)
```

**Benefits**: Independent scaling, separate deploys, logical separation

### Traffic Splitting

**Use cases**: Canary (5% new version), A/B testing (50/50), blue-green

**Methods**: IP, cookie, random

### Scaling Strategies

**Automatic** (most common): Scale based on load
**Basic** (Standard only): Fixed instances, idle shutdown
**Manual**: Fixed instance count

## Key Limitations

**Standard**:

- 60s request timeout
- Read-only filesystem (except /tmp)
- Outbound HTTP/S only
- No background threads

**Flexible**:

- Always ≥1 instance (cost)
- Slower cold start
- No free tier

**Both**:

- Single region deployment
- No global load balancing

## Integration

**Native**:

- Cloud Datastore/Firestore (automatic indexing)
- Cloud Tasks (async work)
- Cloud Storage (static files)
- Cloud SQL (via proxy)

**Identity-Aware Proxy**: Zero-trust authentication without code

## Cost Optimization

**Standard**: Scale to zero, free tier, instance hours
**Flexible**: Always running, higher base cost

**Strategy**: Use Standard unless Flexible required

## Exam Focus

- Standard vs Flexible decision criteria
- When App Engine vs Cloud Run vs GKE
- Scaling configurations
- Service architecture
- Request timeout constraints
- Regional limitation
