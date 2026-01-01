# Identity-Aware Proxy (IAP)

## Core Concepts

IAP provides zero-trust access control for applications without VPN. Authenticate users via Google identity before allowing access to applications.

**Key Principle**: Context-aware access based on user identity and device, not network location.

## How IAP Works

```
User → IAP (authenticate + authorize) → Application
```

**Authentication**: Google Account, Workspace, Cloud Identity
**Authorization**: IAM permissions check
**Result**: Signed headers to application (user identity)

## When to Use IAP

### ✅ Use When

- Protect internal apps without VPN
- Google-based authentication needed
- Context-aware access control
- Zero-trust security model
- Centralized access management
- Need user identity in application

### ❌ Don't Use When

- Public applications (no auth needed)
- Non-HTTP protocols → Cloud VPN
- Need OAuth/SAML flexibility → Custom auth
- API authentication → API keys, service accounts
- Machine-to-machine → Service accounts

## Supported Resources

**App Engine**: Native integration
**Compute Engine**: Via load balancer
**GKE**: Via Ingress with BackendConfig
**Cloud Run**: Not supported (use Cloud Run auth instead)
**Cloud Functions**: Not supported (use function auth)

## Architecture Patterns

### Internal Web App

```
Employees → IAP → GKE (internal dashboard)
```

**Benefits**: No VPN, Google identity, simple

### Multi-Tier Application

```
Users → IAP → App Engine (frontend) → Backend (internal, no IAP)
```

**Security**: Only frontend exposed via IAP

### Admin Access

```
Admins → IAP (conditional access) → VM instances (SSH/RDP)
```

**Features**: TCP forwarding for SSH/RDP without public IPs

## Access Levels (Context-Aware)

**Conditions**:

- IP address range
- Device policy (managed, encrypted)
- Geographic location
- Time of day

**Use case**: Only allow from corporate network + managed devices

**Example**: Access allowed if:

```
IP in corporate range AND device managed AND location = US
```

## IAM Integration

**Roles**:

- `roles/iap.httpsResourceAccessor`: Access web apps
- `roles/iap.tunnelResourceAccessor`: SSH/RDP via IAP

**Granularity**: Project, service, or URL path

**Best Practice**: Use Google Groups for user management

## TCP Forwarding

**Purpose**: SSH/RDP to VMs without public IPs or bastion hosts

**Architecture**:

```
User → IAP tunnel → Private VM (no external IP)
```

**Command**: `gcloud compute ssh` or `gcloud compute start-iap-tunnel`

**Benefits**: No bastion host, no firewall rules for SSH, audit logging

## Signed Headers

**Purpose**: Application receives authenticated user identity

**Headers Provided**:

- `X-Goog-Authenticated-User-Email`: User email
- `X-Goog-Authenticated-User-ID`: User ID

**Use case**: Application-level authorization, logging

**Security**: Verify header signature (prevent spoofing)

## Security Considerations

### Header Verification

**Risk**: Malicious users could spoof headers

**Mitigation**: Verify JWT signature, only trust IAP-verified headers

### Defense in Depth

**Pattern**: IAP + Application-level auth

**Reason**: IAP handles authentication, app handles fine-grained authorization

### Disable Direct Access

**Problem**: Users could bypass IAP via VM external IP

**Solution**:

- Remove external IPs (use IAP tunnel)
- Firewall rules block direct access
- VPC Service Controls

## Cost

**Pricing**: Free (no additional charge for IAP itself)

**Related costs**: Load balancer, compute resources

## Limitations

- HTTP/HTTPS only (TCP forwarding for SSH/RDP)
- Requires Google identity
- Limited to GCP and on-prem (not other clouds)
- Some latency overhead (auth check)

## IAP vs Alternatives

| Need | Solution |
|------|----------|
| Web app auth | IAP |
| VPN access | Cloud VPN |
| API auth | API keys, OAuth |
| Service-to-service | Service accounts |
| Custom auth | Cloud Endpoints, custom code |

## Exam Focus

### Core Concepts

- Zero-trust access control
- Google identity integration
- No VPN needed
- Context-aware access

### Use Cases

- Internal web applications
- SSH/RDP without bastion
- Admin access control
- Replace VPN for web apps

### Architecture

- Supported services (App Engine, GCE, GKE)
- TCP forwarding for SSH/RDP
- Signed headers for user identity
- Access levels (conditional)

### Security

- Defense in depth (IAP + app auth)
- Header verification
- Disable direct access
- IAM role management

### Limitations

- HTTP/HTTPS only
- Google identity required
- Not for Cloud Run/Functions
- No support for non-Google clouds
