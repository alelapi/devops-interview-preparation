# Secret Manager

## Core Concepts

Secret Manager stores and manages sensitive data like API keys, passwords, and certificates. Centralized secret storage with access control, versioning, and audit logging.

**Key Principle**: Never hardcode secrets in code or config files; store centrally with access control.

## When to Use Secret Manager

### ✅ Use When

- Need to store API keys, passwords, tokens
- Application needs secrets at runtime
- Want centralized secret management
- Need secret versioning and rotation
- Audit logging required
- Multiple services share secrets
- CI/CD pipelines need secrets

### ❌ Don't Use When

- Public configuration values → Environment variables
- Large files (>64 KB) → Cloud Storage
- Database passwords with Cloud SQL → Cloud SQL auth
- Service account authentication → Workload Identity

## Secret Manager vs Alternatives

| Need | Solution |
|------|----------|
| API keys, passwords | Secret Manager |
| Service account auth | Workload Identity (no keys) |
| Large files/certs | Cloud Storage with CMEK |
| Config values (non-secret) | Environment variables |
| Database passwords | Secret Manager or Cloud SQL auth |

## Architecture

### Secret Structure

```
Secret (API key name)
├── Version 1 (enabled)
├── Version 2 (enabled) ← Latest
└── Version 3 (disabled)
```

**Secret**: Container (metadata, IAM policies)
**Version**: Actual secret value (immutable)

### Versioning

**Immutable versions**: Cannot modify, only create new

**States**:

- **Enabled**: Can be accessed
- **Disabled**: Cannot be accessed (soft delete)
- **Destroyed**: Permanently deleted (after disabled)

**Use case**: Rotate secrets without downtime

## Access Control

### IAM Roles

**Secret Manager Admin** (`roles/secretmanager.admin`):

- Full control (create, delete, manage IAM)

**Secret Manager Secret Accessor** (`roles/secretmanager.secretAccessor`):

- Read secret values (most common for apps)

**Secret Manager Secret Version Manager** (`roles/secretmanager.secretVersionManager`):

- Create/enable/disable versions

**Secret Manager Viewer** (`roles/secretmanager.viewer`):

- View metadata only (not values)

### Best Practices

- **One secret per service/app**: Granular access control
- **Grant accessor role**: Only to services that need it
- **Use service accounts**: For applications
- **Separate secrets**: Dev, staging, prod

## Integration Patterns

### Cloud Run / Cloud Functions

**Environment variable**:

```
Declare secret in config → Mounted as env var or file
```

**No code changes**: Secret available as environment variable

**Auto-refresh**: Updates when secret rotated (Cloud Run 2nd gen)

### GKE

**Kubernetes Secret**:

```
External Secrets Operator → Secret Manager → K8s Secret
```

**Or CSI Driver**: Mount secrets as volumes

### Compute Engine

**Access via API**:

```python
from google.cloud import secretmanager
client = secretmanager.SecretManagerServiceClient()
name = "projects/PROJECT/secrets/SECRET/versions/latest"
response = client.access_secret_version(request={"name": name})
secret = response.payload.data.decode("UTF-8")
```

**Service account**: VM needs accessor role

### CI/CD

**Cloud Build**:

```yaml
availableSecrets:
  secretManager:

  - versionName: projects/PROJECT/secrets/SECRET/versions/latest
    env: 'API_KEY'
```

**Use for**: Deploy keys, API tokens

## Secret Rotation

### Manual Rotation

**Process**:

1. Create new secret version
2. Update applications to use latest
3. Disable old version
4. After grace period, destroy old version

### Automatic Rotation

**Pattern**: Cloud Function/Cloud Scheduler triggers rotation

**Steps**:

1. Scheduler triggers function
2. Function generates new secret
3. Function creates new version
4. Function updates dependent services
5. Function disables old version

**Use for**: Regularly rotated passwords, API keys

### Rotation Strategy

**API Keys**: 90-day rotation
**Passwords**: 60-day rotation
**Certificates**: Before expiration
**Service accounts**: Prefer Workload Identity (no keys to rotate)

## Replication

### Automatic Replication

**Default**: Secret replicated across multiple regions within geography

**Benefit**: High availability, disaster recovery

### User-Managed Replication

**Specify regions**: Control exact locations

**Use case**: Data residency requirements, compliance

**Example**: Only replicate in EU regions (GDPR)

## Encryption

### Google-Managed Keys (Default)

**Automatic**: Secrets encrypted at rest

**No config**: Works out of the box

### Customer-Managed Encryption Keys (CMEK)

**Cloud KMS integration**: Your keys, your control

**Use case**: Compliance requires customer-managed keys

**Benefit**: Can revoke access by disabling key

## Monitoring and Audit

### Audit Logs

**Logged actions**:

- Secret access (who accessed what when)
- Secret creation/deletion
- Version changes
- IAM policy changes

**Use for**: Compliance, security investigations

### Monitoring

**Metrics**:

- Access count
- Access latency
- Version count

**Alerts**: Unusual access patterns, access failures

## Cost

**Pricing**:

- Active secret versions: $0.06/version/month
- Access operations: $0.03 per 10,000 accesses
- Replication: Included (no extra charge)

**Optimization**:

- Delete unused versions
- Consolidate secrets (don't over-segment)
- Rotate only when necessary

## Common Patterns

### Database Password

**Store**: Database password in Secret Manager
**Access**: Application reads on startup
**Rotate**: Quarterly, update via function

### API Keys

**Store**: Third-party API keys
**Access**: Applications read latest version
**Benefit**: Centralized management, easy rotation

### TLS Certificates

**Store**: Private keys, certificates (if <64 KB)
**Access**: Load balancers, applications
**Rotate**: Before expiration

### Multi-Environment

**Pattern**: Separate secrets for each environment

```
db-password-dev
db-password-staging
db-password-prod
```

**Benefit**: Clear separation, different access controls

## Security Best Practices

### Never Hardcode Secrets

**Bad**: `API_KEY = "abc123"` in code

**Good**: `API_KEY = secret_manager.get("api-key")`

### Minimal Access

**Principle**: Grant accessor role only to services that need it

**Avoid**: Overly broad permissions

### Separate Environments

**Pattern**: Different secrets for dev/staging/prod

**Benefit**: Compromise in dev doesn't affect prod

### Regular Rotation

**Schedule**: Rotate secrets regularly (90 days typical)

**Automation**: Use Cloud Functions for automatic rotation

### Audit Regularly

**Review**: Access logs monthly

**Alert**: Unusual access patterns

### Use Latest Version

**Pattern**: Applications reference "latest" version

**Benefit**: Automatic pickup of rotated secrets

## Secret Manager vs Environment Variables

**Environment Variables**:

- ✅ Fast access
- ✅ Simple
- ❌ No central management
- ❌ No versioning
- ❌ No audit logs
- ❌ Visible in config

**Secret Manager**:

- ✅ Centralized
- ✅ Versioning
- ✅ Audit logs
- ✅ Access control
- ❌ Slight latency
- ❌ More complex

**Decision**: Secret Manager for sensitive data, env vars for non-secrets

## Integration with Other Services

### Cloud Build

**Mount secrets**: Available as environment variables in build steps

### Cloud Functions

**Automatic**: Declare in config, available as env var

### Cloud Run

**Automatic**: Mount as env var or file volume

### GKE

**External Secrets Operator**: Sync to K8s Secrets

### App Engine

**Access via client library**: Read in application code

## Limitations

- Max 64 KB per secret version
- Max 100 versions per secret
- Secret names: 255 characters max
- No automatic rotation (must implement)
- Cannot modify version (create new)

## Troubleshooting

### Access Denied

**Check**:

- Service account has accessor role
- Secret exists and is enabled
- Secret Manager API enabled
- Correct project/secret name

### Secret Not Updated

**Issue**: Application using cached value

**Solution**: Restart application, use latest version alias

### High Costs

**Cause**: Too many versions

**Solution**: Delete old disabled versions

## Exam Focus

### Core Concepts

- Centralized secret storage
- Versioning (immutable versions)
- IAM-based access control
- Audit logging

### Use Cases

- API keys, passwords, tokens
- Database credentials
- TLS certificates (<64 KB)
- CI/CD secrets
- Multi-environment secrets

### Architecture

- Secret versioning strategy
- Rotation patterns
- Integration with Cloud Run/Functions/GKE
- Replication (automatic vs user-managed)

### Security

- Never hardcode secrets
- Minimal access (accessor role)
- Separate environments
- Regular rotation
- Audit logging

### Integration

- Cloud Run/Functions (env vars)
- GKE (External Secrets Operator)
- Cloud Build (secret mounting)
- Compute Engine (client library)

### Best Practices

- One secret per service
- Use "latest" version alias
- Regular rotation (automated)
- Delete old versions
- Monitor access patterns
