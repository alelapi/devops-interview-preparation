# Cloud Endpoints

## Core Concepts

Cloud Endpoints is an API management system for developing, deploying, and managing APIs. Provides authentication, monitoring, and API key management for your services.

**Key Principle**: API gateway and management layer; use for API security, monitoring, and developer experience.

## Endpoints vs Alternatives

| Feature | Cloud Endpoints | Apigee | API Gateway | Cloud Armor |
|---------|----------------|--------|-------------|-------------|
| **Purpose** | API management | Full API platform | Managed gateway | DDoS protection |
| **Backends** | GCP services | Any | GCP/on-prem | Load balanced |
| **Cost** | Low (free tier) | High | Medium | DDoS protection |
| **Complexity** | Low-Medium | High | Low | Low |
| **Use case** | GCP APIs | Enterprise APIs | Simple routing | Security only |

## Supported Backends

### Cloud Endpoints for OpenAPI

**Backends**:

- App Engine Standard
- Cloud Functions
- Cloud Run
- Compute Engine
- GKE

**Specification**: OpenAPI (Swagger) 2.0

**Use case**: RESTful HTTP APIs

### Cloud Endpoints for gRPC

**Backends**:

- GKE
- Compute Engine
- Any gRPC service

**Specification**: gRPC with protocol buffers

**Use case**: High-performance microservices

## When to Use

### ✅ Use Cloud Endpoints When

- Need API key management
- Require API monitoring/logging
- Multiple API versions
- Rate limiting needed
- Developer portal desired
- JWT validation required

### ✅ Use Apigee When

- Enterprise API management
- Monetization required
- Complex transformations
- Legacy system integration
- Multi-cloud/hybrid APIs
- Advanced analytics

### ❌ Don't Use When

- Simple internal APIs (IAM sufficient)
- No API key management needed
- Just need DDoS protection → Cloud Armor
- Just need authentication → IAP

## Key Features

### Authentication

**API Keys**:

- Simple credential
- Per-project or per-application
- Rate limiting per key
- Revocable

**JWT (JSON Web Tokens)**:

- Service account authentication
- OAuth 2.0 integration
- Firebase Auth integration
- Custom issuers

**Decision**: API keys for simple, JWT for enterprise

### Rate Limiting

**Quota Configuration**:

```yaml
quota:
  limits:

    - name: "read-limit"
      metric: "read-requests"
      values:
        STANDARD: 10000  # per day

    - name: "write-limit"
      metric: "write-requests"
      values:
        STANDARD: 1000
```

**Per-User Quotas**: Prevent abuse, fair usage

### Monitoring

**Automatic Metrics**:

- Request count
- Latency
- Error rate
- API key usage

**Integration**: Cloud Monitoring dashboards

### Developer Portal

**Features**:

- API documentation (from OpenAPI spec)
- Try the API interface
- API key management
- Usage statistics

**Generation**: Automatic from spec

## Architecture Patterns

### API Gateway Pattern

```
Client → Cloud Endpoints → Backend (App Engine/Cloud Run/GKE)
```

**Benefits**: Centralized auth, monitoring, rate limiting

### Multi-Version API

```
Endpoints:
  /v1 → Backend v1
  /v2 → Backend v2
```

**Use case**: Backward compatibility, gradual migration

### Microservices Gateway

```
Endpoints → Service A (Cloud Run)
         → Service B (Cloud Functions)
         → Service C (GKE)
```

**Benefits**: Unified API surface, per-service auth

## Deployment Models

### Extensible Service Proxy (ESP)

**Architecture**: Nginx-based proxy in front of backend

**Deployment**:

- Sidecar container (GKE)
- Separate proxy (Compute Engine)

**Limitations**: Legacy, being replaced by ESPv2

### Extensible Service Proxy v2 (ESPv2)

**Architecture**: Envoy-based proxy (modern)

**Benefits**:

- Better performance
- More features
- Active development

**Recommendation**: Use ESPv2 for new deployments

### Cloud Endpoints on Cloud Run

**Pattern**: Deploy ESP as container alongside backend

```
Cloud Run Service = ESP container + Backend logic
```

**Benefits**: Serverless, auto-scaling, simple deployment

## Configuration

### OpenAPI Specification

**Required fields**:

```yaml
swagger: "2.0"
host: "api.example.com"
x-google-endpoints:

  - name: "api.example.com"
    target: "SERVICE_NAME"
securityDefinitions:
  api_key:
    type: "apiKey"
    name: "key"
    in: "query"
```

### Security Schemes

**API Key**:

```yaml
securityDefinitions:
  api_key:
    type: apiKey
    name: key
    in: query
```

**JWT**:

```yaml
securityDefinitions:
  jwt_auth:
    authorizationUrl: ""
    flow: "implicit"
    type: "oauth2"
    x-google-issuer: "issuer-url"
    x-google-jwks_uri: "jwks-url"
```

## Cost Model

**Free tier**: 2M API calls/month

**Pricing** (beyond free tier):

- $3 per million calls
- No infrastructure costs (serverless)

**Backend costs**: Separate (App Engine, Cloud Run, etc.)

## Limitations

- HTTP/REST and gRPC only
- OpenAPI 2.0 (not 3.0 yet)
- Regional deployment (backend determines)
- Limited transformation capabilities vs Apigee

## Apigee Comparison

### Use Cloud Endpoints When

- Simple API management
- GCP-native backends
- Cost-sensitive
- Fast setup needed
- Internal/partner APIs

### Use Apigee When

- Enterprise-grade API platform
- API monetization
- Complex mediation/transformation
- Hybrid/multi-cloud
- Advanced analytics/ML
- API product management

**Cost**: Apigee 10-100x more expensive, more features

## Security Best Practices

### API Key Management

- Restrict keys to specific APIs
- Rotate keys regularly
- Use per-app keys
- Monitor key usage

### JWT Validation

- Verify issuer and audience
- Check expiration
- Use HTTPS only
- Validate signatures

### Network Security

- VPC Service Controls
- Cloud Armor for DDoS
- HTTPS enforcement
- Rate limiting

## Integration

**With Cloud Services**:

- **Identity Platform**: User authentication
- **Cloud Monitoring**: Metrics and alerting
- **Cloud Logging**: Request logging
- **Cloud Trace**: Distributed tracing

## Common Patterns

### Mobile Backend

```
Mobile App → Endpoints (API key) → Cloud Functions/Run
```

### Partner API

```
Partner → Endpoints (JWT) → Backend services
```

### Public API

```
Developer → Endpoints (API key + rate limit) → Your API
```

## Exam Focus

### Design Decisions

- Endpoints vs Apigee
- When to use API gateway
- Authentication method selection
- Backend service choice

### Architecture

- API gateway pattern
- Multi-version APIs
- Microservices gateway
- Security layers

### Features

- API key management
- Rate limiting
- JWT authentication
- Monitoring capabilities

### Deployment

- ESP vs ESPv2
- Integration with backends
- Cloud Run deployment
- GKE sidecar pattern
