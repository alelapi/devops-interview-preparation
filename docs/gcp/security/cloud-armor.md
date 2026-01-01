# Cloud Armor

## Core Concepts

Cloud Armor is a DDoS protection and Web Application Firewall (WAF) service that defends applications from attacks at the edge of Google's network.

**Key Principle**: Layer 7 (application) and Layer 3/4 (network) defense at scale.

## Protection Types

| Type | Protection | Use Case |
|------|------------|----------|
| **Standard** | L3/4 DDoS (network/transport) | Volumetric attacks |
| **Advanced** | L7 DDoS (application) | Application-layer attacks |
| **WAF Rules** | OWASP Top 10, custom rules | SQLi, XSS, etc. |
| **Rate Limiting** | Request throttling | API protection, abuse prevention |

## When to Use Cloud Armor

### ✅ Use When

- Public-facing applications need DDoS protection
- WAF rules required (OWASP protection)
- Geographic restrictions needed (geo-blocking)
- Rate limiting per client IP
- Bot management required
- Adaptive protection desired

### ❌ Don't Use When

- Internal applications only → VPC firewall rules
- No load balancer → Cloud Armor requires LB
- Simple firewall rules → VPC firewall sufficient
- Cost-sensitive small apps → Basic DDoS included free

## Architecture

**Deployment**:

```
Internet → Cloud Armor (at edge) → Load Balancer → Backend
```

**Attachment**: Cloud Armor policies attach to backend services

**Enforcement**: At Google's edge network (blocks before reaching LB)

## Security Policies

### Allow/Deny Rules

**Priority-based evaluation**: Lower number = higher priority

**Actions**:

- **Allow**: Permit request
- **Deny (403)**: Block with HTTP 403
- **Deny (404)**: Block with HTTP 404 (hide resource)
- **Deny (502)**: Block with HTTP 502
- **Rate-based ban**: Temporarily ban IP

**Conditions**:

- Source IP/CIDR
- Geographic location (country/region)
- Request headers
- Request method
- Custom expressions (CEL)

### Example Rules

**Geo-blocking**:

```
Priority 1000: Deny traffic from specific countries
Priority 2000: Allow all other traffic
```

**Rate limiting**:

```
Priority 500: Deny if >100 requests/minute from single IP
```

**OWASP protection**:

```
Priority 100: Deny SQLi patterns
Priority 200: Deny XSS patterns
```

## Preconfigured WAF Rules

**ModSecurity Core Rule Set**:

- SQL injection (SQLi)
- Cross-site scripting (XSS)
- Local file inclusion (LFI)
- Remote file inclusion (RFI)
- Remote code execution (RCE)
- Protocol attacks
- Session fixation

**Sensitivity Levels**: 0-4 (0 = most sensitive, more false positives)

**Tuning**: Adjust sensitivity or create exceptions

## Adaptive Protection

**Purpose**: Automatic detection and mitigation of L7 DDoS

**How it works**:

1. Machine learning baseline normal traffic
2. Detect anomalies (traffic spikes, patterns)
3. Auto-generate security rules
4. Apply rules to mitigate attack

**Use case**: Unknown attack patterns, zero-day protection

**Recommendation**: Enable for production applications

## Rate Limiting

**Types**:

- **Rate-based ban**: Ban IP after threshold exceeded
- **Rate limiting**: Throttle requests (return 429)

**Granularity**:

- Per client IP
- Per user (if authenticated)
- Global across all IPs

**Use cases**:

- API protection (prevent abuse)
- Login protection (brute force)
- Resource exhaustion prevention

**Example**: 100 requests/minute per IP

## Geographic Restrictions

**Allow/Deny by**:

- Country
- Region (within country)

**Use cases**:

- Compliance (GDPR, data residency)
- Reduce attack surface
- Licensing restrictions

**Considerations**: VPN/proxy may circumvent

## Named IP Lists

**Purpose**: Reusable IP allowlists/denylists

**Use cases**:

- Corporate office IPs (allowlist)
- Known malicious IPs (denylist)
- Partner/vendor IPs
- Third-party threat intel

**Management**: Centrally managed, referenced in policies

## Integration with Cloud Monitoring

**Metrics**:

- Requests allowed/denied
- Requests by country
- Rate limiting events
- WAF rule matches
- Adaptive protection alerts

**Logging**: Request logs include Cloud Armor decision

**Alerting**: Configure alerts on attack patterns

## Security Layers

**Defense in Depth**:

```
Layer 1: Cloud Armor (DDoS, WAF, geo-blocking)
Layer 2: Load Balancer (SSL termination, health checks)
Layer 3: IAP (authentication) or API keys
Layer 4: Application (authorization, validation)
```

## Cost Model

**Standard tier**: $0.75/policy/month + $0.0075/1M requests

**Advanced DDoS protection**: Additional cost for L7 adaptive protection

**Optimization**: Consolidate rules, use one policy per backend

## Supported Services

**Works with**:

- Global HTTP(S) Load Balancer
- Global SSL Proxy Load Balancer
- Global TCP Proxy Load Balancer

**Does NOT work with**:

- Regional load balancers
- Internal load balancers
- Network load balancers
- Services without load balancers

## Architecture Patterns

### Public Web Application

```
Users → Cloud Armor (WAF + DDoS) → Global LB → App Engine/Cloud Run
```

**Protection**: SQLi, XSS, DDoS, geo-blocking

### API Gateway

```
Clients → Cloud Armor (rate limiting) → Global LB → Cloud Endpoints → Backend
```

**Protection**: Rate limiting, geographic restrictions

### Multi-Region HA

```
Global LB with Cloud Armor → Backend (us-central1)
                           → Backend (europe-west1)
```

**Benefits**: DDoS at edge, backend protected

## Cloud Armor vs Alternatives

| Need | Solution |
|------|----------|
| L7 DDoS + WAF | Cloud Armor |
| L3/4 only | Standard DDoS (free) |
| Internal traffic | VPC firewall rules |
| VM-specific rules | VPC firewall |
| Network-level blocking | VPC firewall |

## Common Patterns

### OWASP Protection

**Enable**: Preconfigured WAF rules
**Tune**: Adjust sensitivity (start at 2-3)
**Monitor**: Review false positives
**Exceptions**: Create allow rules for known safe patterns

### Rate Limiting

**Configure**: Threshold per use case (API: 1000/min, Login: 10/min)
**Action**: Rate-based ban or throttle (429)
**Monitoring**: Track banned IPs

### Geo-Fencing

**Allow**: Specific countries only
**Deny**: High-risk countries
**Compliance**: Data residency requirements

## Limitations

- Global load balancers only (not regional)
- HTTP/HTTPS traffic only
- Max 200 rules per policy
- CEL expressions have complexity limits
- Real-time monitoring, not prevention (some delay)

## Best Practices

### Layered Security

- Cloud Armor (edge protection)
- IAP (authentication)
- Application validation (input sanitization)
- Database parameterization (SQLi prevention)

### Monitoring

- Enable request logging
- Set up alerts for attacks
- Review denied requests regularly
- Tune rules based on false positives

### Testing

- Test rules in staging first
- Start with logging mode (no blocking)
- Monitor before enforcing
- Have rollback plan

## Exam Focus

### Core Concepts

- DDoS protection (L3/4 standard, L7 advanced)
- WAF rules (OWASP, custom)
- Edge protection (blocks at Google edge)
- Requires load balancer

### Use Cases

- Public application protection
- OWASP Top 10 defense
- Geographic restrictions
- Rate limiting (API, brute force)

### Architecture

- Attachment to backend services
- Works with global LB only
- Defense in depth layers
- Multi-region protection

### Features

- Preconfigured WAF rules
- Adaptive protection (ML-based)
- Rate limiting per IP
- Named IP lists
- CEL expressions

### Limitations

- Global LB only (not regional/internal)
- HTTP/HTTPS only
- Max rules per policy
- Cannot protect services without LB
