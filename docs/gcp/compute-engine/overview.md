# Compute Engine Overview

## Description

Google Compute Engine (GCE) is Infrastructure as a Service (IaaS) that provides virtual machines running in Google's data centers. It offers high-performance, scalable VMs with flexible configurations, per-second billing, and deep integration with other Google Cloud services.

**Architecture**: Virtual machines running on Google's infrastructure with customizable CPU, memory, storage, and networking configurations.

## Key Features

### Virtual Machine Options

- **Predefined Machine Types**: Pre-configured VM shapes optimized for common workloads
- **Custom Machine Types**: Create VMs with custom CPU and memory configurations
- **Machine Families**: General-purpose, compute-optimized, memory-optimized, accelerator-optimized
- **Spot VMs**: Preemptible instances at up to 91% discount
- **Confidential VMs**: VMs with encrypted memory and CPU
- **Sole-Tenant Nodes**: Dedicated physical servers for compliance

### Operating System Support

- **Linux**: Multiple distributions (Debian, Ubuntu, CentOS, RHEL, SUSE, etc.)
- **Windows Server**: Full Windows Server support with license included or BYOL
- **Container-Optimized OS**: Optimized for running containers
- **Custom Images**: Bring your own OS images
- **Shielded VMs**: Verified boot, vTPM, and integrity monitoring

### Performance and Scaling

- **Live Migration**: VMs migrate between hosts with no downtime
- **Per-Second Billing**: Pay only for compute time used
- **Sustained Use Discounts**: Automatic discounts for running VMs
- **Committed Use Discounts**: 1-year or 3-year commitments for up to 57% discount
- **Autoscaling**: Managed Instance Groups with automatic scaling
- **Load Balancing**: Integrated with Cloud Load Balancing

### Storage Options

- **Persistent Disks**: Durable block storage (standard, balanced, SSD, extreme)
- **Local SSDs**: High-performance local storage (up to 9 TB per VM)
- **Cloud Storage Buckets**: Object storage integration
- **Filestore**: Managed NFS file storage
- **Boot Disks**: System disk with OS and bootloader

### Networking

- **VPC Integration**: Private networking with VPC
- **Multiple Network Interfaces**: Up to 8 network interfaces per VM
- **Internal and External IPs**: Public and private IP addressing
- **Cloud NAT**: NAT gateway for outbound internet access
- **Cloud VPN**: Hybrid connectivity to on-premises
- **Cloud Interconnect**: Dedicated network connections

### Management Features

- **Metadata Server**: Instance metadata and startup scripts
- **OS Login**: Manage SSH access via IAM
- **OS Patch Management**: Automated OS patching
- **Cloud Monitoring**: Native integration with Cloud Operations
- **Serial Console**: Emergency access to VMs
- **Instance Templates**: Reusable VM configurations

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **VMs per project** | 24 (default) per region | Soft limit, can be increased |
| **CPUs per project** | 24 (default) per region | Varies by machine type |
| **Persistent disks per project** | 500 per zone | Can be increased |
| **Snapshots per project** | 10,000 | Global limit |
| **Images per project** | 1,000 | Custom images |
| **Network interfaces per VM** | 8 | Maximum |
| **Persistent disks per VM** | 128 | Including boot disk |
| **Local SSDs per VM** | 24 (9 TB total) | Instance-dependent |
| **VM metadata size** | 512 KB | Key-value pairs |

## Machine Families

### General Purpose (E2, N2, N2D, N1)

**Best For**: Balanced workloads

- **E2**: Cost-optimized, shared-core and standard
- **N2**: Balanced price/performance, Intel or AMD
- **N2D**: AMD-based, cost-effective
- **N1**: Previous generation, still widely used

### Compute Optimized (C2, C2D, H3)

**Best For**: Compute-intensive workloads

- **C2**: Ultra-high performance per core
- **C2D**: AMD-based compute-optimized
- **H3**: Latest Intel, highest performance

### Memory Optimized (M1, M2, M3)

**Best For**: Memory-intensive workloads

- **M1**: Up to 4 TB RAM
- **M2**: Up to 12 TB RAM
- **M3**: Latest generation, up to 4 TB RAM

### Accelerator Optimized (A2, A3, G2)

**Best For**: GPU workloads

- **A2**: NVIDIA A100 GPUs for ML/AI
- **A3**: NVIDIA H100 GPUs for AI
- **G2**: NVIDIA L4 GPUs for inference and graphics

## When to Use Compute Engine

### ✅ Use Compute Engine When:

1. **Full OS Control Needed**

   - Need root/administrator access
   - Custom kernel modules or drivers
   - Specific OS configurations
   - Legacy applications requiring full VM

2. **Lift-and-Shift Migrations**

   - Migrating existing on-premises VMs
   - Rehosting applications without modification
   - Maintaining existing architecture
   - Compliance requires VM isolation

3. **Stateful Applications**

   - Traditional databases (if not using Cloud SQL)
   - Applications with persistent local state
   - File servers and NAS
   - Legacy enterprise applications

4. **Windows Workloads**

   - Windows Server applications
   - Active Directory domain controllers
   - .NET Framework applications
   - Microsoft SQL Server

5. **High-Performance Computing**

   - Scientific computing
   - Rendering and simulation
   - Financial modeling
   - Big data processing

6. **Hybrid Cloud Architecture**

   - Consistent with on-premises VMs
   - Gradual cloud migration
   - Burst capacity to cloud
   - Disaster recovery site

### ❌ Don't Use Compute Engine When:

1. **Serverless Better Suited**

   - Stateless HTTP services (use Cloud Run)
   - Event-driven functions (use Cloud Functions)
   - No need for OS management
   - Sporadic or unpredictable workloads

2. **Managed Services Available**

   - Relational databases (use Cloud SQL)
   - NoSQL databases (use Firestore, Bigtable)
   - Data warehousing (use BigQuery)
   - Container orchestration (use GKE)

3. **Simple Static Hosting**

   - Static websites (use Cloud Storage + CDN)
   - Single-page applications (use Firebase Hosting)
   - No server-side processing needed

4. **Very Small Workloads**

   - Minimal resource requirements
   - Cost of VM overhead not justified
   - App Engine or Cloud Run more economical

## Pricing Considerations

### Pricing Models

**On-Demand (Pay-as-you-go)**

- Per-second billing (1-minute minimum)
- Sustained use discounts (automatic)
- No upfront costs
- Most flexible

**Spot VMs (Preemptible)**

- Up to 91% discount
- Can be terminated any time
- 24-hour maximum runtime
- No SLA

**Committed Use Discounts**

- 1-year or 3-year commitments
- Up to 57% discount
- Resource-based or spend-based
- Regional commitment

**Sole-Tenant Nodes**

- Dedicated physical servers
- Node licensing costs
- Compliance use cases
- Pay for entire node

### Cost Optimization Tips

1. **Right-Size VMs**

   - Use recommender API for sizing
   - Start small, scale up as needed
   - Monitor utilization metrics
   - Use custom machine types

2. **Use Spot VMs for Fault-Tolerant Workloads**

   - Batch processing
   - CI/CD pipelines
   - Development/testing
   - Rendering farms

3. **Leverage Discounts**

   - Sustained use (automatic)
   - Committed use (for stable workloads)
   - Spot VMs (up to 91% off)

4. **Schedule VMs**

   - Stop VMs during off-hours
   - Use instance schedules
   - Automate with Cloud Scheduler
   - Only pay for disk storage when stopped

5. **Optimize Storage**

   - Use standard disks for non-critical data
   - Delete unused snapshots
   - Use regional instead of multi-regional
   - Clean up orphaned disks

## Common Use Cases

### Web Application Hosting

```
Load Balancer
    │
    ├─── Managed Instance Group (us-central1)
    │       └── VMs: n2-standard-4 (autoscaling 2-10)
    ├─── Managed Instance Group (europe-west1)
    │       └── VMs: n2-standard-4 (autoscaling 2-10)
    └─── Cloud CDN (static content)

Persistent Disks: Balanced PD for OS and application
Cloud SQL: Managed database backend
Cloud Storage: User uploads and assets
```

### Development/Testing Environments

```
Development:

  - Spot VMs (e2-medium)
  - Stopped during off-hours
  - Snapshots for backup

Testing/Staging:

  - Standard VMs (e2-standard-2)
  - Automated provisioning
  - Instance templates

Production:

  - Regional MIGs
  - Committed use discounts
  - High availability setup
```

### Data Processing Pipeline

```
Ingestion: n2-highmem-8 (data collection)
Processing: c2-standard-60 (compute-intensive)
Storage: Persistent SSD disks + Cloud Storage
Orchestration: Cloud Composer (managed Airflow)
Monitoring: Cloud Monitoring + Logging
```

## Getting Started

### Create a VM Instance

```bash
# Basic VM creation
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud

# Production VM with all options
gcloud compute instances create prod-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-balanced \
  --network=my-vpc \
  --subnet=my-subnet \
  --no-address \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx' \
  --tags=http-server,https-server \
  --labels=env=production,team=platform

# Spot VM for cost savings
gcloud compute instances create spot-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --preemptible \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

### List and Manage VMs

```bash
# List all instances
gcloud compute instances list

# Describe specific instance
gcloud compute instances describe my-vm --zone=us-central1-a

# SSH into instance
gcloud compute ssh my-vm --zone=us-central1-a

# Stop instance
gcloud compute instances stop my-vm --zone=us-central1-a

# Start instance
gcloud compute instances start my-vm --zone=us-central1-a

# Delete instance
gcloud compute instances delete my-vm --zone=us-central1-a
```

## Best Practices

### 1. Security

- **Use OS Login**: IAM-based SSH access instead of SSH keys
- **Shielded VMs**: Enable for production workloads
- **Service Accounts**: Use least-privilege service accounts
- **Firewall Rules**: Restrict access to necessary ports only
- **No Public IPs**: Use Cloud NAT for outbound access
- **VPC Service Controls**: Create security perimeters

### 2. High Availability

- **Regional Resources**: Use regional persistent disks and MIGs
- **Multiple Zones**: Distribute VMs across zones
- **Health Checks**: Configure for load balancing and autohealing
- **Managed Instance Groups**: Use for automatic scaling and healing
- **Snapshots**: Regular backups of critical data

### 3. Performance

- **Right Machine Type**: Match workload characteristics
- **SSD Disks**: Use for I/O-intensive workloads
- **Local SSDs**: For highest performance (ephemeral)
- **Network Tier**: Premium tier for global applications
- **Placement Policies**: Co-locate related VMs for low latency

### 4. Cost Management

- **Instance Schedules**: Stop VMs during off-hours
- **Committed Use**: For long-running production workloads
- **Spot VMs**: For fault-tolerant batch processing
- **Right-Sizing**: Use recommender API
- **Clean Up**: Delete unused disks and snapshots

### 5. Management

- **Instance Templates**: Standardize VM configurations
- **Metadata**: Use for configuration and automation
- **Labels**: Organize and track resources
- **Startup Scripts**: Automate configuration
- **OS Patch Management**: Automate patching

### 6. Monitoring

- **Cloud Monitoring**: Enable monitoring agent
- **Cloud Logging**: Centralize logs
- **Alerting**: Set up alerts for critical metrics
- **Dashboards**: Create custom dashboards
- **Uptime Checks**: Monitor service availability

## Compute Engine vs Other Services

### Compute Engine vs Cloud Run

- **Compute Engine**: Full OS control, stateful, any protocol
- **Cloud Run**: Serverless, stateless, HTTP only, easier scaling

### Compute Engine vs GKE

- **Compute Engine**: Traditional VMs, full control, any workload
- **GKE**: Container orchestration, microservices, Kubernetes

### Compute Engine vs App Engine

- **Compute Engine**: Infrastructure-level control, any language/runtime
- **App Engine**: Platform-level, limited languages, automatic scaling

### Compute Engine vs Cloud Functions

- **Compute Engine**: Long-running processes, complex applications
- **Cloud Functions**: Event-driven, short-lived, single-purpose

## Related Services

- **Cloud Load Balancing**: Distribute traffic across VMs
- **Cloud DNS**: Managed DNS service
- **Cloud NAT**: NAT gateway for VMs without public IPs
- **Cloud VPN**: Hybrid connectivity
- **Cloud Interconnect**: Dedicated connections
- **Cloud Storage**: Object storage for backups and data
- **Persistent Disks**: Block storage for VMs
- **Managed Instance Groups**: Autoscaling and autohealing
