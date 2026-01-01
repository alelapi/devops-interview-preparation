# Cloud IAM

## Core Concepts

IAM (Identity and Access Management) controls who can do what on which resources. Foundation of GCP security through policy-based access control.

**Key Principle**: Grant least privilege; use predefined roles when possible; organize with resource hierarchy.

## IAM Model

**Who (Identity) + Can do what (Role) + On which resource (Resource)**

```
Principal + Role + Resource = IAM Policy Binding
```

## Identities (Who)

### Types

**Google Account**: Individual user (alice@gmail.com)
**Service Account**: Application/service identity (sa@project.iam.gserviceaccount.com)
**Google Group**: Collection of users (team@example.com)
**Google Workspace domain**: All users in domain (example.com)
**Cloud Identity domain**: Domain without Google Workspace

**allUsers**: Anyone on internet (public)
**allAuthenticatedUsers**: Any authenticated Google account

### Service Accounts

**Types**:

- **User-managed**: Created by you, fully controlled
- **Default**: Auto-created (Compute, App Engine)
- **Google-managed**: Used by Google services

**Keys**:

- Google-managed (recommended): Automatic rotation
- User-managed: Manual creation, rotation responsibility

**Best Practice**: Use Workload Identity (GKE), avoid keys when possible

## Roles (What)

### Role Types

| Type | Granularity | Use Case | Example |
|------|-------------|----------|---------|
| **Basic** | Project-level | Legacy, avoid | Owner, Editor, Viewer |
| **Predefined** | Service-level | Standard use | Storage Admin, Compute Admin |
| **Custom** | Fine-grained | Specific needs | Custom limited permissions |

### Basic Roles (Legacy - Avoid)

- **Owner**: Full control + billing
- **Editor**: Modify resources
- **Viewer**: Read-only

**Problem**: Too broad, violates least privilege

**Recommendation**: Use predefined or custom roles instead

### Predefined Roles

**Examples**:

- `roles/storage.objectViewer`: Read objects in Cloud Storage
- `roles/compute.instanceAdmin.v1`: Manage Compute Engine instances
- `roles/bigquery.dataEditor`: Edit BigQuery data

**Format**: `roles/SERVICE.ROLE`

**Best Practice**: Use predefined when available, narrowest scope needed

### Custom Roles

**When to use**:

- Predefined too broad
- Specific permission combination needed
- Compliance requirements

**Limitations**:

- Project or organization level only (not folder)
- More maintenance overhead
- Can become outdated with API changes

## Resource Hierarchy

```
Organization
├── Folder (Department)
│   ├── Folder (Team)
│   │   └── Project
│   └── Project
└── Project
```

**Policy Inheritance**: Child inherits parent's policies (additive, cannot restrict)

**Best Practice**: Grant at highest appropriate level

## IAM Policy

### Policy Structure

```json
{
  "bindings": [
    {
      "role": "roles/storage.objectViewer",
      "members": [
        "user:alice@example.com",
        "serviceAccount:sa@project.iam.gserviceaccount.com"
      ]
    }
  ]
}
```

### Policy Evaluation

1. Deny policies (if exists) - takes precedence
2. Organization policy constraints
3. IAM allow policies (union of all)
4. Result: Allow if any allow, deny if no allow or explicit deny

**Key**: IAM is additive (cannot remove inherited permissions at lower level)

## Conditions

**Purpose**: Time-based or attribute-based access

**Examples**:

```yaml
condition:
  title: "Expires in 2024"
  expression: "request.time < timestamp('2024-12-31T23:59:59Z')"
```

```yaml
condition:
  title: "Only from corporate network"
  expression: "origin.ip in ['203.0.113.0/24']"
```

**Use cases**: Temporary access, resource tagging, IP restrictions

## Organization Policies

**Purpose**: Set guardrails across organization (restrictions)

**Examples**:

- Disable service account key creation
- Require Shielded VMs
- Restrict resource locations
- Disable public IP on VMs

**Difference from IAM**: Constraints (what cannot be done) vs permissions (what can be done)

## Best Practices

### Principle of Least Privilege

- Grant minimum permissions needed
- Use predefined roles over basic
- Regular permission audits
- Remove unused permissions

### Resource Hierarchy

- Use folders for teams/environments
- Grant at highest appropriate level
- Separate projects for isolation
- Consistent naming conventions

### Service Accounts

- One service account per application/function
- Use Workload Identity (GKE)
- Avoid user-managed keys
- Never commit service account keys
- Rotate keys regularly if must use

### Groups for Users

- Manage users via Google Groups
- Grant roles to groups, not individuals
- Easier permission management
- Better auditability

### Separation of Duties

- Split admin roles (compute admin ≠ network admin)
- No single person has complete access
- Require approval for sensitive operations
- Multiple administrators

## Common Patterns

### Multi-Project Setup

```
Shared VPC Project: Network admins
Dev Project: Developers (editor)
Staging Project: Developers (viewer), CI/CD (editor)
Prod Project: Ops (admin), CI/CD (deployer)
```

### Service-to-Service Auth

```
Service A (SA-A) → Service B
Service B: IAM policy grants SA-A required role
```

**No API keys**: Use service account authentication

### Workload Identity (GKE)

```
Kubernetes SA → Google SA → GCP services
```

**Benefit**: No service account keys in pods

## IAM for Specific Services

### Compute Engine

- `compute.instanceAdmin`: Manage instances
- `compute.networkAdmin`: Manage networks
- `compute.securityAdmin`: Manage firewall rules

### Cloud Storage

- `storage.objectViewer`: Read objects
- `storage.objectCreator`: Write objects
- `storage.objectAdmin`: Full object control
- `storage.admin`: Bucket + object control

### BigQuery

- `bigquery.dataViewer`: Read data
- `bigquery.dataEditor`: Edit data
- `bigquery.admin`: Full control

**Dataset-level**: More granular than project

### GKE

- `container.admin`: Full cluster control
- `container.developer`: Deploy workloads
- `container.viewer`: Read-only

## Troubleshooting Access

### Policy Analyzer

**Tool**: IAM Policy Analyzer in Console

**Capabilities**:

- Why does user have access?
- What can user access?
- Policy troubleshooting

### Policy Simulator

**Purpose**: Test policy changes before applying

**Use case**: Verify least privilege, prevent breaking changes

### Audit Logs

**Admin Activity**: Who changed what
**Data Access**: Who accessed what data (must enable)
**System Events**: Automated actions

**Best Practice**: Enable data access logs for sensitive resources

## Security Considerations

### Avoid Basic Roles

**Problem**: Too permissive

**Solution**: Use predefined or custom roles

### Service Account Keys

**Risk**: Can be stolen, no expiration

**Mitigation**:

- Use Workload Identity
- Use Google-managed keys
- Rotate user-managed keys
- Disable key creation (org policy)

### Public Access

**Risk**: Data exposure

**Mitigation**:

- Avoid `allUsers`, `allAuthenticatedUsers`
- Use signed URLs for temporary access
- Public access prevention org policy
- Regular access reviews

### Over-Privileged Service Accounts

**Risk**: Compromised service = broad access

**Mitigation**:

- One SA per application
- Minimum permissions
- Regular permission audits
- Service account impersonation for debugging

## IAM vs ACLs

| Feature | IAM | ACLs |
|---------|-----|------|
| **Granularity** | Bucket or project | Object-level |
| **Recommended** | Yes (uniform bucket-level) | No (legacy) |
| **Complexity** | Lower | Higher |
| **Audit** | Easier | Harder |

**Recommendation**: Use uniform bucket-level access (IAM only)

## Exam Focus

### Fundamentals

- Who (principal) + What (role) + Where (resource)
- Policy inheritance (additive)
- Basic vs predefined vs custom roles
- Service account types and uses

### Best Practices

- Least privilege principle
- Resource hierarchy design
- Service account management
- Separation of duties

### Common Patterns

- Multi-project organization
- Service-to-service auth
- Workload Identity
- Temporary access (conditions)

### Security

- Avoid basic roles
- Key management
- Public access risks
- Organization policies

### Troubleshooting

- Policy Analyzer usage
- Audit logs
- Policy simulation
- Access debugging
