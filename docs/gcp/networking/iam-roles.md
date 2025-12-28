# IAM Roles for Networking

## Description

IAM (Identity and Access Management) controls who can perform what actions on which GCP networking resources. Proper IAM configuration is critical for security, compliance, and operational efficiency. This guide covers the key predefined roles for networking and when to use them.

## Role Hierarchy

### Primitive Roles (Avoid for Production)

- **Owner**: Full access (too permissive)
- **Editor**: Modify resources (too permissive)
- **Viewer**: Read-only access (acceptable for read operations)

**Recommendation**: Use **predefined** or **custom** roles instead of primitive roles for networking.

---

## Core Networking Roles

### Compute Network Admin (`roles/compute.networkAdmin`)

**Permissions**: Full control over networking resources

**Can**:

- Create, modify, delete VPC networks, subnets, routes
- Configure firewall rules, firewall policies
- Manage VPN gateways, tunnels, Cloud Routers
- Configure load balancers and SSL certificates
- Create and delete Private Service Connect endpoints
- Manage network peering connections
- Configure Cloud NAT
- Administer Private Google Access settings

**Cannot**:

- Create instances (requires `compute.instanceAdmin`)
- Modify organization policies
- Manage billing
- Create Shared VPC host projects (requires `compute.xpnAdmin`)

**Use Case**:

- Network administrators managing VPC infrastructure
- Platform teams responsible for network design and implementation
- DevOps engineers managing cloud networking

**Scope**: Project-level (can also grant at folder/org level)

---

### Compute Network User (`roles/compute.networkUser`)

**Permissions**: Use existing network resources but cannot create or modify them

**Can**:

- Use VPC subnets to create instances
- Attach instances to existing subnets
- View network, subnet, route information (read-only)
- Use shared VPC subnets (critical for service projects)

**Cannot**:

- Create or delete networks, subnets
- Modify firewall rules
- Create VPN or load balancers
- Change network configurations

**Use Case**:

- Application teams creating VMs in service projects
- Developers deploying workloads to existing infrastructure
- Service accounts for GKE, Cloud Run needing network access
- Users in Shared VPC service projects

**Scope**: Project-level or subnet-level (for granular access)

**Important**: In Shared VPC, grant this role **at subnet level** in host project to service project service accounts.

---

### Compute Network Viewer (`roles/compute.networkViewer`)

**Permissions**: Read-only access to networking resources

**Can**:

- View VPC networks, subnets, routes
- View firewall rules and policies
- View VPN gateways, Cloud Routers, load balancers
- View network topology and configuration
- Access networking metrics and logs (if logging viewer role also granted)

**Cannot**:

- Create, modify, or delete any resources
- Use network resources to create instances

**Use Case**:

- Auditors reviewing network configurations
- Read-only access for troubleshooting
- Security teams reviewing firewall rules
- Junior team members learning infrastructure

**Scope**: Project, folder, or organization level

---

### Compute Security Admin (`roles/compute.securityAdmin`)

**Permissions**: Manage security-related networking resources

**Can**:

- Create, modify, delete firewall rules
- Manage SSL certificates and SSL policies
- Configure Cloud Armor security policies
- Manage organization firewall policies (if org-level)
- View network resources (read-only)

**Cannot**:

- Create or modify VPC networks, subnets
- Create VPN or load balancers (only security configs)
- Modify routes

**Use Case**:

- Security teams managing firewall rules
- Compliance teams enforcing security policies
- Separation of duties (security vs networking)

**Scope**: Project, folder, or organization level

---

## Shared VPC Specific Roles

### Compute Shared VPC Admin (`roles/compute.xpnAdmin`)

**Permissions**: Enable and manage Shared VPC configuration

**Can**:

- Enable/disable a project as Shared VPC host
- Attach/detach service projects to host projects
- View Shared VPC configuration

**Cannot**:

- Create networks or subnets (requires `networkAdmin`)
- Grant networkUser role (requires IAM admin permissions)

**Use Case**:

- Organization admins setting up Shared VPC
- Platform teams managing multi-project architectures

**Scope**: **Organization or folder level only** (cannot grant at project level)

**Important**: Typically paired with `compute.networkAdmin` for full Shared VPC management.

---

### Service Project Admin (No specific role)

**Pattern**: Combine multiple roles for service project administrators

**Typical Combination**:

- `roles/compute.instanceAdmin`: Create VMs
- `roles/compute.networkUser`: Use shared subnets (granted on host project subnets)
- `roles/iam.serviceAccountUser`: Attach service accounts

**Use Case**: Users who manage resources in service projects of Shared VPC

---

## Specialized Networking Roles

### Compute Load Balancer Admin (`roles/compute.loadBalancerAdmin`)

**Permissions**: Manage load balancing resources

**Can**:

- Create, modify, delete load balancers
- Configure backend services, health checks
- Manage SSL certificates for load balancers
- Configure URL maps, target proxies
- Manage network endpoint groups (NEGs)

**Cannot**:

- Modify VPC networks or subnets
- Create instances (backend resources)

**Use Case**:

- Application teams managing their own load balancers
- Separation of duties for load balancer management

---

### Compute Public IP Admin (`roles/compute.publicIpAdmin`)

**Permissions**: Manage external IP addresses

**Can**:

- Reserve and release external IP addresses
- Promote ephemeral IPs to static
- View IP address usage

**Cannot**:

- Attach IPs to instances (requires instance admin)
- Modify networks or subnets

**Use Case**:

- IP address management teams
- Resource optimization (releasing unused IPs)

---

### Service Networking Admin (`roles/servicenetworking.networksAdmin`)

**Permissions**: Manage private service connections

**Can**:

- Create Private Service Connect endpoints
- Configure private IP allocations for Google services
- Manage peered VPC connections for managed services (e.g., Cloud SQL)
- Configure Private Service Connect forwarding rules

**Cannot**:

- Modify VPC networks (requires `networkAdmin`)

**Use Case**:

- Platform teams configuring private access to Google services
- Database administrators setting up Cloud SQL private IP

---

### DNS Administrator (`roles/dns.admin`)

**Permissions**: Manage Cloud DNS resources

**Can**:

- Create, modify, delete DNS zones
- Manage DNS records
- Configure DNS peering
- Manage DNS policies (split horizon, inbound/outbound forwarding)

**Cannot**:

- Modify VPC networks directly (but can peer DNS zones)

**Use Case**:

- DNS administrators managing internal and external DNS
- Multi-cloud DNS integration

---

## Custom Roles

### When to Create Custom Roles

✅ **Use Custom Roles When**:

- Predefined roles are too permissive
- Need specific combination of permissions
- Compliance requires fine-grained access control
- Implementing least-privilege security model

❌ **Avoid Custom Roles When**:

- Predefined roles meet requirements
- Organization lacks resources to maintain custom roles
- Complexity outweighs security benefits

### Example Custom Role: VPN Operator

**Use Case**: Allow VPN tunnel management without full network admin access

```yaml
title: "VPN Operator"
description: "Manage VPN tunnels and Cloud Routers"
includedPermissions:

  - compute.vpnGateways.*
  - compute.vpnTunnels.*
  - compute.routers.*
  - compute.routes.get
  - compute.routes.list
  - compute.networks.get
  - compute.networks.list
  - compute.regions.get
  - compute.regions.list
```

### Example Custom Role: Firewall Rule Manager

**Use Case**: Manage firewall rules without network creation permissions

```yaml
title: "Firewall Rule Manager"
description: "Create and modify firewall rules only"
includedPermissions:

  - compute.firewalls.*
  - compute.networks.get
  - compute.networks.list
  - compute.networks.updatePolicy
```

---

## IAM Best Practices for Networking

### 1. Principle of Least Privilege

- Grant minimum permissions needed
- Use predefined roles when possible
- Create custom roles for specific needs
- Avoid primitive roles (Owner, Editor) in production

### 2. Use Groups, Not Individual Users
```bash
# Good: Grant role to group
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:network-admins@example.com" \
  --role="roles/compute.networkAdmin"

# Avoid: Grant role to individual users
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/compute.networkAdmin"
```

### 3. Scope Roles Appropriately

**Organization Level**: Broad permissions across all projects
```bash
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="group:network-admins@example.com" \
  --role="roles/compute.networkViewer"
```

**Folder Level**: Specific business unit or environment
```bash
gcloud resource-manager folders add-iam-policy-binding FOLDER_ID \
  --member="group:prod-network-admins@example.com" \
  --role="roles/compute.networkAdmin"
```

**Project Level**: Individual project permissions
```bash
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:dev-team@example.com" \
  --role="roles/compute.networkUser"
```

**Subnet Level** (Shared VPC): Granular service project access
```bash
gcloud compute networks subnets add-iam-policy-binding SUBNET_NAME \
  --member="serviceAccount:SERVICE_PROJECT_ID@cloudservices.gserviceaccount.com" \
  --role="roles/compute.networkUser" \
  --region=REGION
```

### 4. Service Accounts for Automation

**Pattern**: Use dedicated service accounts with minimal permissions

```bash
# Create service account for Terraform
gcloud iam service-accounts create terraform-network-sa \
  --display-name="Terraform Network Automation"

# Grant specific network admin permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:terraform-network-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/compute.networkAdmin"
```

### 5. Audit and Review

- Regularly review IAM bindings
- Use Cloud Asset Inventory for IAM audits
- Enable audit logging for IAM changes
- Implement periodic access reviews
- Remove unused or stale permissions

### 6. Separation of Duties

**Example**: Separate networking and security management

```
Network Admins Group:

  - roles/compute.networkAdmin (VPC, subnets, routes, VPN)

Security Admins Group:

  - roles/compute.securityAdmin (firewall rules, SSL policies)

Shared VPC Admins Group:

  - roles/compute.xpnAdmin (Shared VPC management)
  - roles/compute.networkAdmin (network resources)
```

---

## Common IAM Patterns

### Pattern 1: Shared VPC Setup

**Host Project**:
```bash
# Enable Shared VPC (requires xpnAdmin at org/folder level)
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Grant networkAdmin to network team
gcloud projects add-iam-policy-binding HOST_PROJECT_ID \
  --member="group:network-admins@example.com" \
  --role="roles/compute.networkAdmin"
```

**Service Project**:
```bash
# Attach service project (requires xpnAdmin)
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# Grant networkUser on specific subnet to service project
gcloud compute networks subnets add-iam-policy-binding SUBNET_NAME \
  --member="serviceAccount:SERVICE_PROJECT_NUMBER@cloudservices.gserviceaccount.com" \
  --role="roles/compute.networkUser" \
  --region=REGION

# Grant instance admin to application team in service project
gcloud projects add-iam-policy-binding SERVICE_PROJECT_ID \
  --member="group:app-team@example.com" \
  --role="roles/compute.instanceAdmin"
```

### Pattern 2: Multi-Environment Access Control

**Production**:
```bash
# Strict permissions, network changes require approval
gcloud projects add-iam-policy-binding PROD_PROJECT_ID \
  --member="group:prod-network-admins@example.com" \
  --role="roles/compute.networkAdmin"

gcloud projects add-iam-policy-binding PROD_PROJECT_ID \
  --member="group:developers@example.com" \
  --role="roles/compute.networkViewer"
```

**Development**:
```bash
# More permissive for faster iteration
gcloud projects add-iam-policy-binding DEV_PROJECT_ID \
  --member="group:developers@example.com" \
  --role="roles/compute.networkAdmin"
```

### Pattern 3: Read-Only Network Auditor

```bash
# Grant read-only access for auditing
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="group:network-auditors@example.com" \
  --role="roles/compute.networkViewer"

# Also grant logging viewer for complete audit trail
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="group:network-auditors@example.com" \
  --role="roles/logging.viewer"
```

---

## Troubleshooting IAM Issues

### "Permission Denied" Errors

**Symptoms**: User cannot create VMs in Shared VPC subnet

**Common Cause**: Missing `compute.networkUser` role on subnet

**Solution**:
```bash
# Grant networkUser on specific subnet
gcloud compute networks subnets add-iam-policy-binding SUBNET_NAME \
  --member="serviceAccount:SERVICE_ACCOUNT@example.com" \
  --role="roles/compute.networkUser" \
  --region=REGION
```

### Cannot Attach Service Project to Shared VPC

**Common Cause**: Missing `compute.xpnAdmin` role or granted at project level

**Solution**:
```bash
# Grant xpnAdmin at organization or folder level
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="user:admin@example.com" \
  --role="roles/compute.xpnAdmin"
```

### Service Account Cannot Create Resources

**Common Cause**: Impersonation or missing role on service account

**Solution**:
```bash
# Grant service account user role
gcloud iam service-accounts add-iam-policy-binding SERVICE_ACCOUNT@PROJECT.iam.gserviceaccount.com \
  --member="user:admin@example.com" \
  --role="roles/iam.serviceAccountUser"

# OR grant directly to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SERVICE_ACCOUNT@PROJECT.iam.gserviceaccount.com" \
  --role="roles/compute.networkAdmin"
```

---

## Security Considerations

### 1. Avoid Over-Permissive Roles
❌ **Bad**: Granting `roles/owner` for network tasks
✅ **Good**: Granting `roles/compute.networkAdmin`

### 2. Protect Sensitive Resources

- Use organizational policies to enforce controls
- Require approval for firewall rule changes
- Implement four-eyes principle for production changes

### 3. Monitor IAM Changes
```bash
# Set up log-based alert for IAM policy changes
gcloud logging sinks create iam-policy-changes \
  storage.googleapis.com/iam-audit-logs-bucket \
  --log-filter='protoPayload.methodName="SetIamPolicy"'
```

### 4. Rotate Service Account Keys

- Use short-lived credentials when possible
- Rotate long-lived keys regularly
- Use Workload Identity for GKE instead of service account keys

### 5. Conditional IAM (Context-Aware Access)
```yaml
# Example: Grant networkAdmin only from corporate network
bindings:

  - members:
      - group:network-admins@example.com
    role: roles/compute.networkAdmin
    condition:
      expression: "request.auth.claims.iss == 'https://accounts.google.com' && 
                   origin.ip in ['203.0.113.0/24']"
      title: "Allow only from corporate network"
```

---

## Related Documentation

- **IAM Overview**: https://cloud.google.com/iam/docs
- **Predefined Roles**: https://cloud.google.com/iam/docs/understanding-roles
- **Custom Roles**: https://cloud.google.com/iam/docs/creating-custom-roles
- **Shared VPC IAM**: https://cloud.google.com/vpc/docs/shared-vpc#iam_in_shared_vpc
- **Best Practices**: https://cloud.google.com/iam/docs/best-practices
