# Cloud SQL Proxy

## Core Concepts

Cloud SQL Proxy provides secure access to Cloud SQL instances without whitelisting IPs or managing SSL certificates. Encrypted connection with automatic authentication.

**Key Principle**: Secure database access via authenticated proxy, not public IPs.

## How It Works

```
Application → Cloud SQL Proxy (local) → Encrypted tunnel → Cloud SQL
```

**Authentication**: Uses service account or user credentials
**Encryption**: Automatic TLS encryption
**Connection**: Via Cloud SQL Admin API, not direct TCP

## When to Use

### ✅ Use When

- Connecting from Compute Engine, GKE, Cloud Run
- Development from local machine
- Secure connection without public IP
- Don't want to manage SSL certificates
- Need automatic credential rotation
- Connecting from App Engine Standard (required)

### ❌ Don't Use When

- Cloud SQL Auth Proxy alternative exists (Cloud Run, Cloud Functions native support)
- Performance critical (slight latency overhead) → Private IP
- Serverless VPC Access better fit (Cloud Functions to private Cloud SQL)

## Connection Methods

### Private IP (Recommended for GCP)

**Architecture**:

```
VM/GKE (same VPC) → Private IP → Cloud SQL
```

**Benefits**:

- No internet egress
- Lower latency
- No proxy needed (direct connection)
- Better performance

**Requirements**: VPC peering, same region

### Public IP + Cloud SQL Proxy

**Architecture**:

```
Application → Cloud SQL Proxy → Public IP → Cloud SQL
```

**Benefits**:

- Works from anywhere
- No VPC peering needed
- Automatic encryption

**Use when**: External connections, multi-region, development

### Public IP + Authorized Networks

**Architecture**:

```
Application (whitelisted IP) → Public IP → Cloud SQL
```

**Not recommended**: Manual IP management, less secure

## Deployment Patterns

### Compute Engine / VM

**Sidecar pattern**:

```bash
# Proxy runs on VM
cloud-sql-proxy <INSTANCE_CONNECTION_NAME>
# App connects to localhost:3306
```

### GKE Sidecar

**Pattern**: Proxy container in same pod as application

```yaml
containers:

- name: app
  image: myapp

- name: cloud-sql-proxy
  image: gcr.io/cloud-sql-connectors/cloud-sql-proxy
  command: ["/cloud-sql-proxy", "PROJECT:REGION:INSTANCE"]
```

**Benefits**: One proxy per pod, scales with app

### GKE Workload Identity

**Best practice**: Use Workload Identity instead of service account keys

```
K8s Service Account → Google Service Account → Cloud SQL
```

### App Engine Standard

**Built-in**: No proxy needed, automatic connection

**Configuration**: Specify instance in `app.yaml`

### Cloud Run

**Native support**: Use Unix socket connection

**Configuration**: Environment variable with connection name

### Cloud Functions

**Serverless VPC Access**: Connect via VPC to private IP

**Or**: Cloud SQL Proxy in application code (2nd gen only)

## Authentication

### Service Account (Production)

**Requirements**:

- Service account with Cloud SQL Client role
- Credentials available to proxy

**Best practice**: Workload Identity (GKE), not keys

### User Credentials (Development)

**Method**: `gcloud auth application-default login`

**Use for**: Local development only, not production

## Security Benefits

### No IP Whitelisting

**Problem with authorized networks**: IP management overhead, security risk

**Solution**: Proxy uses IAM authentication, no IP restrictions

### Automatic Encryption

**All connections encrypted**: No manual SSL certificate management

**TLS version**: Automatically upgraded

### Credential Management

**Automatic rotation**: Service accounts rotate automatically

**No hardcoded passwords**: Uses Google credentials

## Performance Considerations

### Latency

**Overhead**: ~5-10ms additional latency (proxy hop)

**Mitigation**: Use private IP for latency-sensitive workloads

### Connection Pooling

**Proxy handles**: Multiple connections to database

**Application**: Still needs connection pooling

**Best practice**: Configure app connection pool appropriately

### Resource Usage

**Lightweight**: Minimal CPU/memory overhead

**Scaling**: One proxy can handle many connections

## Cost

**Proxy**: Free (no additional charge)

**Data transfer**:

- Private IP: No egress charges (same region)
- Public IP: Standard egress charges apply

**Cloud SQL**: Normal instance charges

## Cloud SQL Proxy vs Alternatives

| Method | Security | Performance | Complexity | Cost |
|--------|----------|-------------|------------|------|
| **Private IP** | High | Best | Medium | Low |
| **Cloud SQL Proxy** | High | Good | Low | Medium |
| **Authorized Networks** | Medium | Good | Low | Medium |
| **Direct (no security)** | Low | Best | Lowest | Low |

**Recommendation**: Private IP for production, Proxy for development/external

## Common Patterns

### Development Setup

```
Developer laptop → Cloud SQL Proxy → Cloud SQL (dev instance)
```

**Benefit**: Secure access without VPN

### Multi-Environment

```
Dev: Cloud SQL Proxy (public IP)
Staging: Private IP (VPC peering)
Production: Private IP (VPC peering)
```

### Hybrid Cloud

```
On-premises app → Cloud SQL Proxy → Cloud Interconnect → Cloud SQL
```

### Migration

```
Legacy app (on-prem) → Cloud SQL Proxy → Cloud SQL
```

**Use case**: Gradual migration, hybrid connectivity

## Troubleshooting

### Connection Failures

**Check**:

- Service account has Cloud SQL Client role
- Instance connection name correct
- Network connectivity (firewall rules)
- Cloud SQL Admin API enabled

### Performance Issues

**Diagnose**:

- Proxy latency vs network latency
- Connection pool configuration
- Database query performance
- Consider private IP for better performance

### Authentication Errors

**Common causes**:

- Missing IAM permissions
- Expired credentials
- Service account key not found
- Wrong instance connection name

## Best Practices

### Use Private IP When Possible

**Production**: Private IP for best performance/security

**Proxy**: Development, external access, migration

### Workload Identity for GKE

**Avoid**: Service account key files in pods

**Use**: Workload Identity for automatic credential management

### Connection Pooling

**Application-level**: Configure connection pool

**Proxy**: Not a connection pool replacement

### Monitoring

**Metrics**: Connection count, latency, errors

**Logging**: Enable Cloud SQL logs

## Limitations

- Slight latency overhead (vs direct connection)
- Requires Cloud SQL Admin API enabled
- One proxy instance per application (typically)
- Not for extremely high-performance scenarios (use private IP)

## Exam Focus

### Core Concepts

- Secure access without IP whitelisting
- Automatic encryption (TLS)
- IAM-based authentication
- Proxy architecture (local proxy to Cloud SQL)

### Use Cases

- Secure database access
- Development from local machine
- No public IP on Cloud SQL
- Avoid SSL certificate management
- App Engine Standard (required)

### Architecture

- Sidecar pattern (GKE)
- Private IP vs Proxy decision
- Workload Identity integration
- Multi-environment strategy

### Security

- No hardcoded credentials
- Automatic encryption
- IAM authentication
- Better than authorized networks

### Alternatives

- Private IP (best for production in GCP)
- Serverless VPC Access (Cloud Functions)
- Native support (Cloud Run, App Engine)
