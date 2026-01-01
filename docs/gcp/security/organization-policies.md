# Organization Policies

## Core Concepts

Organization Policies set constraints on resources across the entire organization. Guardrails that prevent misconfigurations and enforce compliance.

**Key Principle**: Centralized governance; restrictions that override individual permissions.

## Organization Policies vs IAM

| Feature | Organization Policies | IAM |
|---------|----------------------|-----|
| **Purpose** | What can/cannot be done | Who can do what |
| **Type** | Constraints/restrictions | Permissions/allowances |
| **Example** | "VMs must be in us-central1" | "User can create VMs" |
| **Precedence** | Overrides IAM | Evaluated after org policies |

**Relationship**: Org policies restrict, IAM permits within those restrictions

## Policy Types

### List Constraints

**Allow or deny specific values**

**Examples**:

- Allowed VM instance types: `constraints/compute.vmExternalIpAccess`
- Allowed resource locations: `constraints/gcp.resourceLocations`
- Allowed service accounts: `constraints/iam.allowedPolicyMemberDomains`

**Modes**:

- **Allow list**: Only specified values permitted
- **Deny list**: Specified values prohibited
- **Allow all**: No restrictions
- **Deny all**: Completely prohibited

### Boolean Constraints

**Enable or disable features**

**Examples**:

- `constraints/compute.disableSerialPortAccess`: Disable serial port
- `constraints/sql.restrictPublicIp`: Prevent Cloud SQL public IPs
- `constraints/iam.disableServiceAccountKeyCreation`: No service account keys

**Values**: True (enforced) or False (not enforced)

## Common Organization Policies

### Resource Locations

**Constraint**: `constraints/gcp.resourceLocations`

**Purpose**: Restrict where resources can be created

**Use case**: GDPR compliance (EU only), data residency

**Example**: Only allow `in:us-locations`

### Disable VM External IPs

**Constraint**: `constraints/compute.vmExternalIpAccess`

**Purpose**: Force VMs to use Cloud NAT or private IPs

**Use case**: Security (no public VMs), centralized egress

**Example**: Deny all external IPs

### Restrict Service Account Key Creation

**Constraint**: `constraints/iam.disableServiceAccountKeyCreation`

**Purpose**: Prevent user-managed service account keys

**Use case**: Security (keys can be stolen), force Workload Identity

**Example**: Enforce true

### Allowed Policy Member Domains

**Constraint**: `constraints/iam.allowedPolicyMemberDomains`

**Purpose**: Only allow IAM members from specific domains

**Use case**: Prevent external sharing

**Example**: Only allow `@company.com` and `@company.gserviceaccount.com`

### Require OS Login

**Constraint**: `constraints/compute.requireOsLogin`

**Purpose**: Enforce OS Login for SSH (IAM-based)

**Use case**: Centralized access control, no SSH keys

### Shielded VMs

**Constraint**: `constraints/compute.requireShieldedVm`

**Purpose**: Require Shielded VMs (Secure Boot, vTPM)

**Use case**: Security baseline, compliance

### Uniform Bucket-Level Access

**Constraint**: `constraints/storage.uniformBucketLevelAccess`

**Purpose**: Require uniform bucket-level access (IAM only, no ACLs)

**Use case**: Simplified permissions, better security

### Disable Service Account Key Upload

**Constraint**: `constraints/iam.disableServiceAccountKeyUpload`

**Purpose**: Prevent external service account key upload

**Use case**: Security (prevent stolen keys from other orgs)

## Policy Hierarchy

```
Organization (broadest)
├── Folder
│   └── Project (narrowest)
```

**Inheritance**: Child inherits parent policies

**Merging**:

- List constraints: Union of allowed values (most restrictive wins)
- Boolean constraints: Cannot override parent (parent true = child must be true)

**Example**:

```
Org: Allow locations = [us-*, eu-*]
Folder: Allow locations = [us-central1, us-east1]
Result: Only us-central1, us-east1 (intersection)
```

## Policy Evaluation

**Order**:

1. Check organization policy constraints
2. If allowed by org policy, check IAM permissions
3. If both pass, action permitted

**Example**:

```
User has IAM permission to create VM
Org policy restricts locations to us-central1
User tries to create VM in europe-west1
Result: DENIED (org policy blocks)
```

## When to Use

### ✅ Use Organization Policies When

- Need to enforce governance across org
- Compliance requirements (data residency, security)
- Prevent costly mistakes (expensive resources)
- Security baselines (Shielded VMs, no public IPs)
- Consistent standards across projects
- Prevent shadow IT

### ❌ Don't Use When

- Need per-resource control → IAM
- Need to grant permissions → IAM
- Temporary exceptions common → Difficult with org policies

## Exceptions

### Tags

**Purpose**: Allow exceptions via tags

**Pattern**:

1. Define org policy with tag condition
2. Tag resources that need exception
3. Policy doesn't apply to tagged resources

**Use case**: "No external IPs" except for bastion hosts (tagged)

### Policy Override

**Carefully consider**: Exceptions reduce effectiveness

**Better**: Rethink policy or architecture

## Implementation Strategy

### Gradual Rollout

**Phase 1**: Audit mode (log violations, don't block)
**Phase 2**: Enforce on new resources
**Phase 3**: Remediate existing violations
**Phase 4**: Full enforcement

### Testing

**Dry run**: Test policy on test folder/project first

**Monitor**: Check for blocked legitimate actions

### Communication

**Announce**: Warn teams before enforcement

**Document**: Explain rationale and exceptions

## Common Patterns

### Data Residency (GDPR)

```
Org policy: Only EU locations
Result: All resources must be in EU
Compliance: GDPR data residency
```

### Security Baseline

```
Policies:

- Require Shielded VMs
- Disable service account keys
- Require OS Login
- No public IPs on VMs
Result: Enforced security minimum
```

### Cost Control

```
Policies:

- Allowed VM types (no expensive ones)
- Allowed GKE node types
- Disable APIs (prevent usage)
Result: Control cloud spend
```

### Shadow IT Prevention

```
Policy: Only allow @company.com domain members
Result: Can't share with external users
```

## Monitoring and Compliance

### Policy Analyzer

**Purpose**: See which policies apply to resources

**Use case**: Troubleshooting, compliance verification

### Compliance Reports

**Integration**: Security Command Center

**Shows**: Violations, compliance status

### Audit Logs

**Logs**: Policy changes, constraint evaluations

**Use for**: Security audits, compliance evidence

## Best Practices

### Start Broad, Get Specific

**Phase 1**: Organization-level policies (baseline)
**Phase 2**: Folder-level policies (teams/divisions)
**Phase 3**: Project-level only if necessary

### Document Rationale

**Each policy**: Why it exists, what it prevents

**Exceptions**: Document and regularly review

### Regular Reviews

**Quarterly**: Review policies, remove obsolete

**Annual**: Comprehensive security review

### Least Restrictive

**Balance**: Security vs developer productivity

**Avoid**: Overly restrictive policies that block legitimate work

## Troubleshooting

### "Permission Denied" Despite IAM

**Check**: Organization policies may be blocking

**Tool**: Policy Analyzer to see constraints

### Policy Not Applying

**Check**: Inheritance, policy propagation delay

**Verify**: Resource hierarchy

### Conflicts

**Understand**: Most restrictive wins

**Resolution**: Review parent policies

## Limitations

- Max 20 boolean constraints per resource
- Max 20 list constraints per resource
- Propagation delay (up to 15 minutes)
- Cannot disable for single resource (use tags)
- Some services don't support all constraints

## Organization Policies vs VPC Service Controls

| Feature | Organization Policies | VPC Service Controls |
|---------|----------------------|---------------------|
| **Scope** | Resource constraints | API perimeter security |
| **Purpose** | Prevent misconfig | Prevent data exfil |
| **Example** | "No public IPs" | "Data can't leave VPC" |
| **Granularity** | Resource types | API calls |

**Use both**: Complementary security layers

## Exam Focus

### Core Concepts

- Constraints vs permissions
- List vs boolean constraints
- Policy hierarchy and inheritance
- Org policies override IAM

### Common Policies

- Resource location restrictions
- VM external IP disable
- Service account key restrictions
- Domain restrictions
- Shielded VM requirements

### Use Cases

- Data residency (GDPR)
- Security baselines
- Cost control
- Shadow IT prevention
- Compliance enforcement

### Architecture

- Hierarchy (org → folder → project)
- Policy inheritance
- Exception handling (tags)
- Gradual rollout strategy

### Integration

- IAM relationship (restrictive, not permissive)
- VPC Service Controls (complementary)
- Security Command Center
- Policy Analyzer for troubleshooting

### Best Practices

- Start at org level
- Document rationale
- Test before full enforcement
- Regular reviews
- Balance security and productivity
