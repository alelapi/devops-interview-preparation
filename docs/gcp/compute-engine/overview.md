# Compute Engine Overview

## Description

Google Compute Engine (GCE) provides Infrastructure as a Service (IaaS) with virtual machines running in Google's data centers. As an architect, understanding when to choose Compute Engine versus managed services is critical for designing scalable, cost-effective, and maintainable solutions.

**Key Architectural Consideration**: Compute Engine offers maximum control but requires operational overhead. Evaluate managed alternatives (Cloud Run, GKE, App Engine, Cloud SQL) before defaulting to VMs.

## Core Concepts

### Virtual Machine Model

- **Ephemeral vs Persistent**: VM instance is ephemeral; only persistent disks survive deletion
- **Live Migration**: VMs migrate between hosts transparently during maintenance
- **Preemptible/Spot VMs**: Up to 91% discount but can be terminated any time (24-hour max runtime)
- **Regional Persistent Disks**: Synchronous replication across two zones for HA

### Machine Families

**Decision Framework**:

- **E2 (Cost-Optimized)**: Development, testing, low-traffic applications
- **N2/N2D (General Purpose)**: Balanced price/performance, most workloads
- **C2/C2D/H3 (Compute-Optimized)**: CPU-intensive, HPC, high-performance computing
- **M1/M2/M3 (Memory-Optimized)**: In-memory databases, SAP HANA, large caches
- **A2/A3 (GPU-Accelerated)**: ML training, inference, scientific computing

**Custom Machine Types**: Create VMs with specific vCPU-to-memory ratios when predefined types don't fit

## Architectural Patterns

### High Availability

**Single-Zone Deployment**:

- Simplest architecture
- No SLA for VM availability
- Acceptable for dev/test, non-critical workloads
- Use Managed Instance Groups (MIGs) for autohealing

**Multi-Zone Deployment**:

- Regional MIG distributes VMs across zones
- 99.99% availability (with MIG and load balancer)
- Protects against zone failures
- Recommended for production

**Multi-Region Deployment**:

- Global load balancer distributes traffic across regions
- Protects against regional failures
- Higher cost (cross-region egress)
- Required for global low-latency services

### Disaster Recovery

**RPO/RTO Targets**:

- **Snapshots**: RPO 24 hours (daily), RTO 30-60 minutes
- **Machine Images**: RPO 7 days (weekly), RTO 15-30 minutes
- **Regional Persistent Disks**: RPO near-zero, RTO minutes
- **Cross-Region Replication**: Manual via snapshots/images

**DR Strategies**:

- **Backup and Restore**: Snapshots/images, slowest RTO, lowest cost
- **Pilot Light**: Minimal infrastructure in DR region, scale on failover
- **Warm Standby**: Running infrastructure at reduced capacity
- **Hot Standby**: Full capacity active-active, highest cost

## Important Limits (Architecture Impact)

| Limit | Value | Architectural Impact |
|-------|-------|---------------------|
| **VMs per project** | 24 (default) per region | Request increases proactively; use multiple projects |
| **CPUs per project** | 24 (default) per region | Affects scalability; plan quotas |
| **Persistent disks per VM** | 128 | Consider object storage for > 128 data sources |
| **Network interfaces per VM** | 8 | Limits multi-VPC connectivity options |
| **Local SSD per VM** | 24 devices (9 TB) | Ephemeral; design for data loss |
| **Snapshots per project** | 10,000 | Implement retention policies |

## Decision Criteria: Compute Engine vs Alternatives

### Use Compute Engine When:

**Full OS Control Required**:

- Custom kernel modules
- Specific OS versions/configurations
- Legacy applications requiring full VM
- Windows Server workloads

**Lift-and-Shift Migrations**:

- Rehosting existing on-premises VMs
- Minimal application changes acceptable
- Time-to-cloud more important than optimization

**Hybrid Cloud**:

- Consistent VM experience on-premises and cloud
- VPN/Interconnect integration
- Active Directory integration

**Stateful Applications**:

- Self-managed databases (when Cloud SQL doesn't fit)
- Traditional enterprise applications
- Licensing constraints (BYOL)

### Don't Use Compute Engine When:

**Managed Services Available**:

- Databases → Cloud SQL, Spanner, Firestore
- Containers → GKE, Cloud Run
- Batch processing → Dataflow, Dataproc
- Message queues → Pub/Sub

**Serverless Fits**:

- HTTP-only services → Cloud Run
- Event-driven functions → Cloud Functions
- Stateless applications with variable load
- Want zero operational overhead

**Simple Requirements**:

- Static websites → Cloud Storage + CDN
- Simple APIs → Cloud Run
- No persistent state needed

## Cost Optimization Strategies

### Pricing Models

**On-Demand**:

- Per-second billing (1-minute minimum)
- Sustained use discounts (automatic 20-30%)
- Most flexible, no commitment

**Spot VMs (Preemptible)**:

- Up to 91% discount
- Can terminate any time (30-second warning)
- Use for: batch jobs, CI/CD, fault-tolerant workloads
- Not for: databases, stateful apps, critical services

**Committed Use Discounts (CUD)**:

- 1-year or 3-year commitments
- Up to 57% discount
- Resource-based or spend-based
- Use for: predictable, long-running workloads

**Sole-Tenant Nodes**:

- Dedicated physical servers
- Per-node pricing
- Use for: compliance, licensing, isolation requirements

### Cost Architecture Decisions

**Right-Sizing**:

- Use Recommender API for optimization suggestions
- Start small, scale up based on metrics
- Custom machine types for exact fit
- Avoid over-provisioning

**Scheduling**:

- Stop VMs during off-hours (dev/test)
- Only pay for disk storage when stopped
- Use Instance Schedules or Cloud Scheduler
- 60-70% savings for non-24/7 workloads

**Storage Optimization**:

- Use standard disks for non-critical data
- Delete unused snapshots/images
- Regional vs multi-regional storage
- Consider object storage for large datasets

## Networking Considerations

### VPC Integration

**Network Design**:

- VMs exist in VPC subnets
- Each VM can have up to 8 network interfaces
- Internal IP (private) required, external IP (public) optional
- Use Cloud NAT for outbound internet without public IPs

**Connectivity Options**:

- **Internal**: VPC, VPC Peering, Shared VPC
- **Hybrid**: Cloud VPN, Cloud Interconnect
- **Internet**: Cloud NAT, External IPs, Load Balancers

### Load Balancing Architecture

**External Application Load Balancer**:

- Global HTTP(S) load balancing
- SSL termination
- URL-based routing
- Cloud CDN integration

**Internal Load Balancer**:

- Private load balancing within VPC
- TCP/UDP support
- Regional or cross-region

**Network Load Balancer**:

- Layer 4 (TCP/UDP)
- Preserves client IP
- Regional or global

## Security Architecture

### Security Layers

**VM-Level Security**:

- OS hardening and patching
- OS Login (IAM-based SSH)
- Shielded VMs (Secure Boot, vTPM, integrity monitoring)
- Confidential VMs (memory encryption)

**Network Security**:

- VPC firewall rules (stateful)
- Hierarchical firewall policies
- Private Google Access
- VPC Service Controls

**Identity and Access**:

- Service accounts with least privilege
- Workload Identity for GKE integration
- IAM roles for VM management
- Separate admin duties

**Data Protection**:

- Persistent disks encrypted by default
- Customer-managed encryption keys (CMEK)
- Snapshots encrypted automatically
- Regional disks for data residency

## Monitoring and Operations

### Observability Strategy

**Metrics**:

- CPU, memory, disk, network utilization
- Custom metrics via monitoring agent
- Application-level metrics
- Uptime checks for availability

**Logging**:

- System logs, application logs
- Serial console output
- Audit logs for compliance
- Centralized logging in Cloud Logging

**Alerting**:

- Proactive notification of issues
- Threshold-based alerts
- Log-based metrics
- Integration with incident management

### Operational Patterns

**Managed Instance Groups (MIGs)**:

- Autoscaling based on CPU, LB utilization, custom metrics
- Autohealing with health checks
- Rolling updates with surge/unavailable controls
- Regional MIGs for zone distribution

**Instance Templates**:

- Standardized VM configurations
- Version control for infrastructure
- Foundation for MIGs
- Consistent deployments

**Startup/Shutdown Scripts**:

- Automated configuration
- State preservation
- Graceful shutdown handling
- Integration with external systems

## Compliance and Governance

### Compliance Considerations

**Data Residency**:

- Regional resources stay in region
- Snapshots stored in specified locations
- Machine images location-controlled
- Logs can be restricted to regions

**Regulatory Requirements**:

- HIPAA: Shielded VMs, encryption, audit logs
- PCI-DSS: Network isolation, encryption, logging
- SOC 2: Access controls, change management, monitoring
- GDPR: Data location, encryption, deletion capabilities

**Organizational Controls**:

- Organization policies (constraints)
- Resource labels for tracking
- IAM hierarchy (org → folder → project)
- Separate projects for environments

## Integration Patterns

### With Other GCP Services

**Data Services**:

- Cloud SQL (managed MySQL, PostgreSQL, SQL Server)
- Cloud Storage (object storage, backups, static content)
- Persistent Disk (block storage)
- Filestore (NFS file storage)

**Container Services**:

- GKE for containerized workloads alongside VMs
- Hybrid architectures with VMs + containers
- Migration path: VMs → Containers → Serverless

**Analytics and ML**:

- BigQuery for data warehousing
- Dataflow for stream/batch processing
- Vertex AI for ML workloads
- Data transfer from VMs to analytics services

**Management**:

- Cloud Monitoring and Logging
- Cloud Deployment Manager / Terraform
- Cloud Build for CI/CD
- Artifact Registry for custom images

## Best Practices Summary

### Architecture Design

1. **Use managed services first**: Only use VMs when necessary
2. **Design for failure**: Assume VMs can fail, use MIGs and health checks
3. **Implement monitoring**: Proactive alerting before users notice issues
4. **Plan for scale**: Use autoscaling, not manual intervention
5. **Optimize costs**: Right-size, use Spot VMs, implement schedules
6. **Secure by default**: Least privilege, private IPs, OS hardening
7. **Document decisions**: Architecture decision records (ADRs)

### Operational Excellence

1. **Infrastructure as Code**: Terraform, Deployment Manager
2. **Immutable infrastructure**: Replace, don't modify
3. **Blue-green deployments**: Zero-downtime updates
4. **Disaster recovery plan**: Test regularly, document procedures
5. **Capacity planning**: Quota management, growth projections
6. **Cost monitoring**: Budgets, alerts, regular reviews

## Exam Focus Areas

### Key Topics for Professional Architect

**Design Decisions**:

- When to use Compute Engine vs managed services
- Machine type selection criteria
- High availability architecture patterns
- Disaster recovery strategies
- Cost optimization approaches

**Scaling Patterns**:

- Horizontal vs vertical scaling trade-offs
- Autoscaling configuration considerations
- Multi-region deployment strategies
- Load balancing options

**Security**:

- Network isolation strategies
- IAM role design
- Encryption options
- Compliance requirements

**Operations**:

- Monitoring and alerting strategy
- Update and patch management
- Backup and restore procedures
- Incident response planning

**Integration**:

- Hybrid cloud connectivity
- Service integration patterns
- Data flow architecture
- Migration strategies
