# Virtual Machines (VMs)

## Core Concepts

A Compute Engine VM instance is a virtualized server with customizable CPU, memory, storage, and networking resources. Understanding VM characteristics and lifecycle is essential for architectural decisions.

**Key Principle**: VMs are ephemeral; design for failure and use persistent storage for stateful data.

## Machine Types Selection

### Predefined vs Custom

**Predefined Machine Types**:

- Standard configurations (e.g., n2-standard-4: 4 vCPUs, 16 GB RAM)
- Optimized for common workload patterns
- Simpler to manage and plan
- Use for: Most workloads with standard requirements

**Custom Machine Types**:

- Specify exact vCPU and memory (0.9-8 GB RAM per vCPU)
- Cost optimization for specific ratios
- Extended memory available (up to 624 GB)
- Use for: Workloads with non-standard resource ratios

### Machine Family Decision Matrix

| Workload Type | Recommended Family | Reasoning |
|---------------|-------------------|-----------|
| Web servers, APIs | N2, N2D | Balanced, cost-effective |
| Databases (small-medium) | N2, N2D | Good memory/CPU ratio |
| Large databases | M-series | Memory-optimized |
| Batch processing | E2, Spot VMs | Cost-optimized |
| HPC, gaming servers | C2, C2D, H3 | Highest CPU performance |
| ML training | A2, A3 | GPU-accelerated |
| Development/testing | E2, Spot VMs | Lowest cost |

## VM Lifecycle and States

### State Transitions

- **PROVISIONING** → **STAGING** → **RUNNING**: Normal startup
- **RUNNING** → **STOPPING** → **TERMINATED**: Clean shutdown
- **TERMINATED** → **RUNNING**: Start operation
- **RUNNING** → **SUSPENDING** → **SUSPENDED** (Beta): Save state to disk

**Architectural Impact**:

- Billing stops in TERMINATED state (disk charges remain)
- No SLA for VM availability without MIG
- Live migration during maintenance (RUNNING → RUNNING transparently)
- Preemptible VMs: 30-second termination warning, 24-hour maximum

## Spot VMs (Preemptible)

### Cost vs Availability Trade-off

**Benefits**:

- Up to 91% discount
- Same performance as regular VMs
- Significant cost savings at scale

**Limitations**:

- Can be terminated at any time
- 30-second warning via shutdown script
- No live migration
- No SLA
- 24-hour maximum runtime

### Architectural Use Cases

**Appropriate for**:

- Batch processing jobs
- CI/CD pipelines
- Data processing (MapReduce-style)
- Rendering farms
- Fault-tolerant distributed systems
- Development/testing environments

**Not appropriate for**:

- Databases (unless replicated)
- Stateful applications without external state
- Long-running single processes
- Applications requiring guaranteed uptime
- Services without checkpointing

**Design Pattern**: Combine regular + Spot VMs in MIG for cost optimization with availability

## Boot Disk Configuration

### Disk Type Selection

| Disk Type | IOPS | Throughput | Use Case | Cost |
|-----------|------|------------|----------|------|
| pd-standard | Low | Low | Dev/test, backup target | Lowest |
| pd-balanced | Medium | Medium | General purpose (recommended) | Medium |
| pd-ssd | High | High | Databases, I/O intensive | High |
| pd-extreme | Custom | Very high | Mission-critical databases | Highest |

**Decision Criteria**:

- Boot disk I/O rarely bottleneck (use pd-balanced)
- Data disks: Match I/O requirements
- Performance scales with disk size
- Consider Local SSD for highest performance (ephemeral)

### Size Considerations

**Minimum Recommendations**:

- Linux: 20 GB (OS + updates + applications)
- Windows: 50 GB (OS + updates)
- Production: Add 30-50% buffer

**Performance Impact**:

- Larger disks = higher IOPS/throughput
- 100 GB pd-balanced: 600 IOPS
- 1000 GB pd-balanced: 6000 IOPS
- Maximum depends on VM machine type

## Security Options

### Shielded VMs

**Components**:

- **Secure Boot**: Verify bootloader and kernel integrity
- **vTPM**: Virtual Trusted Platform Module for key management
- **Integrity Monitoring**: Alert on boot sequence changes

**When to Enable**:

- Production workloads (should be default)
- Compliance requirements (PCI-DSS, HIPAA)
- Protection against rootkits and bootkits
- Minimal performance impact

### Confidential VMs

**Capabilities**:

- Memory encryption using AMD SEV
- Isolates VM memory from Google and other VMs
- Hardware-based protection

**Trade-offs**:

- Limited machine types (N2D, C2D)
- Slight performance overhead
- Higher cost
- Use for: Highly sensitive workloads, regulatory requirements

### Sole-Tenant Nodes

**Use Cases**:

- Licensing requirements (per-socket, per-core)
- Compliance requirements (physical isolation)
- Performance isolation
- Workload separation for security

**Considerations**:

- Pay for entire node (all vCPUs), not per VM
- More expensive than shared tenancy
- Requires capacity planning
- Suitable for large, stable workloads

## Network Configuration

### IP Addressing

**Internal IP (Required)**:

- Private RFC 1918 address
- Assigned from subnet range
- Used for VPC communication
- Can be ephemeral or static

**External IP (Optional)**:

- Public internet-routable address
- Ephemeral (changes on stop/start) or static (reserved)
- Billed when reserved but not attached
- Alternative: Cloud NAT for outbound without public IP

### Multiple Network Interfaces

**Capabilities**:

- Up to 8 NICs per VM
- Each in different VPC network
- Each with own routing table
- Each can have internal and external IPs

**Use Cases**:

- Network appliances (firewall, router)
- DMZ architectures
- Separate management network
- Traffic separation for security

**Limitations**:

- Cannot be in same VPC
- No automatic traffic routing between interfaces
- Requires application-level routing

## Service Accounts

### Architecture Patterns

**Default Compute Engine Service Account**:

- Auto-created per project
- Has Editor role by default (too permissive)
- Used if no SA specified
- **Don't use for production**

**Custom Service Account (Recommended)**:

- Create per application/workload
- Grant minimal necessary permissions
- Principle of least privilege
- Easier to audit and rotate

**Scopes**:

- Legacy mechanism (being replaced by IAM)
- Limit API access even with SA permissions
- Use `cloud-platform` scope + IAM roles (modern approach)
- Scopes cannot grant more access than IAM roles

## Managed Instance Groups (MIGs)

### Benefits

**Availability**:

- Autohealing with health checks
- Automatic replacement of unhealthy instances
- Distribution across zones (regional MIG)
- No single point of failure

**Scalability**:

- Autoscaling based on metrics
- Horizontal scaling (add/remove instances)
- Integration with load balancing
- Handle traffic spikes automatically

**Management**:

- Rolling updates with version control
- Canary deployments
- Consistent configuration via templates
- Automated operations

### Autoscaling Configuration

**Scaling Policies**:

- CPU utilization (most common)
- HTTP load balancing utilization
- Cloud Pub/Sub queue size
- Custom metrics (Cloud Monitoring)
- Multiple policies (scales on any)

**Considerations**:

- **Cool-down period**: Prevent thrashing
- **Min/max instances**: Set boundaries
- **Scale-in controls**: Prevent rapid scale-down
- **Target utilization**: 60-70% for headroom

**Architecture Pattern**:

- Use MIG for all production workloads
- Even single-instance (autohealing benefit)
- Regional MIG for high availability
- Combine with load balancer for best results

## Metadata and Startup Scripts

### Metadata Server

**Purpose**:

- Provide instance and project metadata
- Service account tokens
- Startup/shutdown scripts
- Custom key-value pairs

**Security Considerations**:

- Accessible from within VM without authentication
- Service account tokens available
- Use Workload Identity for GKE instead
- Restrict metadata access if needed

### Startup Scripts

**Use Cases**:

- Software installation and configuration
- Join domain or cluster
- Register with external services
- Application deployment

**Best Practices**:

- Idempotent execution
- Log to serial console
- Handle failures gracefully
- Keep minimal (use configuration management)
- Consider cloud-init or Ansible

## High Availability Patterns

### Single-Zone Architecture

**Characteristics**:

- All VMs in one zone
- Simplest design
- Lower cost (no cross-zone traffic)
- No zone failure protection

**Appropriate for**:

- Development/testing
- Non-critical workloads
- Stateful services with external replication
- When zone failure is acceptable

### Multi-Zone (Regional MIG)

**Characteristics**:

- VMs distributed across 3+ zones
- Zone failure protection
- Slightly higher cost (cross-zone traffic)
- 99.99% availability with load balancer

**Design Considerations**:

- Even distribution vs BALANCED (Google decides)
- Health check requirements
- Data replication strategy
- Shared storage (Filestore, Cloud SQL)

### Multi-Region

**Characteristics**:

- VMs in multiple regions
- Regional failure protection
- Global load balancing
- Higher cost (cross-region egress)

**When Required**:

- Global user base (low latency)
- Disaster recovery (regional failure)
- Compliance (data residency)
- Highest availability requirements

## Performance Considerations

### CPU Platform

**Options**:

- Intel Cascade Lake
- Intel Ice Lake
- Intel Sapphire Rapids
- AMD Milan, Genoa

**Impact**:

- Newer platforms: Better performance
- Varies by region/zone
- Can specify minimum platform
- Automatic upgrades over time

### CPU Overcommitment

**Shared-Core Machine Types**:

- E2-micro, E2-small, E2-medium
- Burst capability above baseline
- Suitable for low-utilization workloads
- Cost-effective for development

**Standard and Higher**:

- Dedicated vCPU allocation
- Consistent performance
- Production workloads
- Predictable behavior

## Compliance and Governance

### Data Residency

**Control Mechanisms**:

- Choose region for VM placement
- Boot disk in same region
- Snapshots can specify location
- Organization policy constraints

### Labeling Strategy

**Purpose**:

- Cost allocation and tracking
- Resource organization
- Automation selection
- Compliance tagging

**Recommended Labels**:

- environment (prod, staging, dev)
- team/cost-center
- application/service
- compliance-level
- data-classification

## Exam Focus Areas

### Design Decisions

- Machine type selection criteria
- When to use Spot VMs
- HA architecture patterns (single/multi-zone/region)
- Network design (internal vs external IPs)

### Cost Optimization

- Right-sizing strategies
- Committed use vs on-demand
- Spot VM use cases and limitations
- Instance scheduling

### Security

- Service account best practices
- Shielded VMs vs Confidential VMs
- Network isolation patterns
- IAM role design

### Scalability

- MIG configuration
- Autoscaling policies
- Load balancing integration
- Regional distribution

### Operations

- Update strategies (rolling, canary)
- Health check design
- Monitoring and alerting
- Startup/shutdown scripts
