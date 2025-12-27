# Shared VPC

## Description

Shared VPC allows an organization to connect resources from multiple service projects to a common VPC network in a host project. This enables centralized network administration while maintaining project-level billing and resource organization. Network admins maintain control over the network resources while service project admins can create resources that use the shared network.

**Architecture**: Centralized VPC network (host project) with attached service projects consuming the network resources.

## Key Features

### Organizational
- **Centralized Network Administration**: Network team manages all network resources in one place
- **Separation of Concerns**: IAM separation between network admins and resource admins
- **Project-Level Billing**: Service projects maintain separate billing despite using shared network
- **Cross-Project Resource Creation**: VMs in service projects use subnets from host project

### Network Capabilities
- **Shared Subnets**: Service projects can use host project subnets
- **Private IP Address Management**: Centralized IP address space planning
- **Firewall Rules**: Centralized or delegated firewall rule management
- **VPC Network Peering**: Host project can peer with other VPCs, connectivity extends to service projects
- **Hybrid Connectivity**: Single point for VPN/Interconnect setup benefiting all service projects

### Security & Access Control
- **Subnet-Level IAM**: Granular permissions at subnet level
- **Service Project Admins**: Limited to resource creation, not network changes
- **Organizational Policy**: Can enforce Shared VPC usage across organization
- **Audit Logging**: Centralized network change tracking

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Service projects per host** | 100 | Default, can be increased |
| **Host projects per organization** | No limit | Can have multiple host projects |
| **Shared VPC Admin role** | Organization or folder level | Cannot be granted at project level |
| **Subnet sharing** | Must explicitly share subnets | Subnets not automatically shared |
| **VPC Peering** | Only host project can peer | Service projects cannot create peering |
| **Cloud VPN/Interconnect** | Only in host project | Service projects cannot create these |
| **Network tags** | Created in service projects | But firewall rules in host project can use them |

## IAM Roles

### Host Project Roles
- **`roles/compute.xpnAdmin`**: Shared VPC Admin (org/folder level) - enables/disables host projects
- **`roles/compute.networkAdmin`**: Network Admin - manages network resources in host project
- **`roles/compute.securityAdmin`**: Security Admin - manages firewall rules

### Service Project Roles
- **`roles/compute.networkUser`**: Allows creating resources using shared subnets
- **`roles/compute.instanceAdmin`**: Create instances but needs networkUser for Shared VPC subnets

## When to Use

### ✅ Use Shared VPC When:

1. **Centralized Network Management**
   - Single network team managing infrastructure across multiple teams/projects
   - Standardized network policies and security controls
   - Consistent IP address management across organization

2. **Hub and Spoke Architecture**
   - Central hub (host project) with multiple spokes (service projects)
   - Shared services (DNS, NTP, monitoring) accessed by all projects
   - Transitive connectivity requirements between spokes

3. **Enterprise Organizations**
   - Multiple business units or teams with separate projects
   - Compliance requirements for network oversight
   - Cost allocation per team while sharing network infrastructure

4. **Hybrid Cloud Deployments**
   - Single VPN/Interconnect connection serving multiple projects
   - On-premises integration for all cloud projects
   - Centralized egress/ingress control

5. **Multi-Tier Applications**
   - Frontend, backend, and database in separate projects
   - Private communication between tiers
   - Different teams managing different tiers

### ❌ Don't Use Shared VPC When:

1. **Complete Project Isolation Needed**
   - Projects must be fully network-isolated
   - Regulatory requirements prevent network sharing
   - Use separate VPCs with VPC Peering if limited connectivity needed

2. **Small Organizations with Single Team**
   - Overhead of Shared VPC not justified
   - Single project VPC is simpler
   - Team has both network and resource admin responsibilities

3. **Cross-Organization Scenarios**
   - Shared VPC only works within a single organization
   - Use VPC Peering for cross-organization connectivity

4. **Frequently Changing Network Topology**
   - Service projects constantly added/removed
   - Network architecture not yet stable
   - Consider starting simple, migrate to Shared VPC later

## Common Architectures

### Hub and Spoke with Shared VPC
```
Host Project (Hub)
├── Shared VPC Network
│   ├── Subnet A (us-central1)
│   ├── Subnet B (europe-west1)
│   └── Subnet C (asia-east1)
├── Cloud VPN/Interconnect (on-premises)
└── VPC Peering (to partner networks)

Service Project 1 (Spoke) - Production
├── Uses Subnet A
└── Compute Engine VMs

Service Project 2 (Spoke) - Development  
├── Uses Subnet B
└── GKE Clusters

Service Project 3 (Spoke) - Data
├── Uses Subnet C
└── Cloud SQL, BigQuery
```

### Multi-Environment Setup
```
Shared Services Host Project
├── Shared VPC Network
│   ├── Monitoring/Logging Subnet
│   ├── Artifact Registry Subnet
│   └── DNS Subnet

Production Host Project
├── Shared VPC Network
│   └── Production Subnets
└── Service Projects: App1, App2, App3

Non-Production Host Project
├── Shared VPC Network
│   └── Dev/Staging Subnets
└── Service Projects: Dev-App1, Dev-App2
```

## Setup Process

1. **Enable Host Project**
   ```bash
   gcloud compute shared-vpc enable HOST_PROJECT_ID
   ```

2. **Attach Service Project**
   ```bash
   gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
       --host-project=HOST_PROJECT_ID
   ```

3. **Grant IAM Permissions**
   ```bash
   # Grant network user on specific subnet
   gcloud compute networks subnets add-iam-policy-binding SUBNET_NAME \
       --member=serviceAccount:SERVICE_PROJECT_NUMBER@cloudservices.gserviceaccount.com \
       --role=roles/compute.networkUser \
       --region=REGION
   ```

4. **Create Resources in Service Project**
   - Resources can now reference host project subnets
   - Use `--subnet=projects/HOST_PROJECT/regions/REGION/subnetworks/SUBNET_NAME`

## Configuration Best Practices

1. **IP Address Planning**
   - Plan comprehensive IP address scheme before implementation
   - Leave room for growth in subnet ranges
   - Document subnet allocation per service project

2. **IAM Management**
   - Use groups for role assignments, not individual users
   - Grant minimal necessary permissions
   - Use subnet-level IAM for fine-grained control

3. **Firewall Strategy**
   - Use hierarchical firewall policies for organization-wide rules
   - Delegate specific firewall rules where appropriate
   - Leverage network tags for flexible rule application

4. **Monitoring & Logging**
   - Enable VPC Flow Logs on shared subnets
   - Set up Cloud Monitoring for network metrics
   - Centralize logging in host project or separate logging project

5. **Service Project Management**
   - Document which service projects use which subnets
   - Regular audits of attached service projects
   - Establish process for onboarding new service projects

## Troubleshooting Common Issues

**Permission Denied Creating Resources**
- Verify service project is attached to host project
- Check IAM permissions on specific subnet
- Ensure service account has `compute.networkUser` role

**Cannot See Shared Subnets**
- Confirm subnet is in same region as resource
- Verify IAM permissions at subnet level
- Check if using correct subnet reference format

**Firewall Rules Not Working**
- Verify target tags exist on resources in service projects
- Check if hierarchical firewall policies apply
- Confirm source/destination ranges include service project IPs

## Related Services

- **VPC Peering**: Host project can peer with other VPCs
- **Private Service Connect**: Access managed services privately
- **Cloud VPN/Interconnect**: Hybrid connectivity through host project
- **Hierarchical Firewall Policies**: Organization-wide firewall management
- **VPC Service Controls**: Enhanced security perimeter around resources
