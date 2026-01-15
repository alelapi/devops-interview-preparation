# Compute Engine Essentials

## 🎯 Heavy Hitters (High Frequency)

### 1. **Machine Types & Families**
Know when to use each family - this comes up frequently!

| Family | Use Case | Key Features |
|--------|----------|--------------|
| **E2** | Cost-optimized, general purpose | Cheapest, shared-core, no local SSD |
| **N2/N2D** | Balanced price/performance | General workloads, good all-around |
| **C2/C2D** | Compute-optimized | High CPU, gaming servers, HPC |
| **M2/M3** | Memory-optimized | Large in-memory databases, SAP HANA |
| **A2** | GPU workloads | ML training/inference, 12-16 GPUs |
| **T2D/T2A** | Scale-out workloads | ARM-based, web servers, microservices |

**Exam Clue Keywords:**
- "Cost-effective, general purpose" → **E2**
- "High CPU, compute-intensive" → **C2/C2D**
- "Large in-memory database, SAP HANA" → **M2/M3**
- "ML training, GPU workload" → **A2** (with GPUs)
- "ARM-based, containerized apps" → **T2A**

### 2. **Instance Groups**
- **Managed Instance Group (MIG)**: Auto-healing, auto-scaling, rolling updates
  - **Stateless**: Web servers, microservices (use with load balancer)
  - **Stateful**: Databases, game servers (preserves instance state)
  - **Regional MIG**: Instances across multiple zones (99.99% SLA)
  - **Zonal MIG**: Instances in single zone
  - **Exam Clue**: "Auto-scaling, high availability" → Regional MIG
  
- **Unmanaged Instance Group**: Manual management, no auto-scaling
  - **When**: Heterogeneous instances, legacy applications
  - **Exam Clue**: "Different instance types in group" → Unmanaged

### 3. **Preemptible vs Spot VMs**
- **Preemptible VMs**: Up to 80% discount, max 24 hours, 30-second shutdown warning
  - **Use**: Batch jobs, fault-tolerant workloads
  - **Not for**: Production databases, always-on services
  
- **Spot VMs**: Up to 91% discount, can run > 24 hours, same 30-second warning
  - **Better than Preemptible**: No 24-hour limit, more predictable
  - **Exam Clue**: "Fault-tolerant batch job, cost savings" → Spot VMs

---

## 🖥️ Machine Types & Sizing

### **Standard vs Custom vs Predefined**
- **Predefined**: e2-standard-4, n2-standard-8 (fixed CPU/memory ratios)
- **Custom**: Choose exact vCPU + memory (fine-tune for workload)
  - **When**: Need specific CPU/memory ratio not in predefined
  - **Cost**: Slightly more expensive than predefined
  - **Limits**: 1-96 vCPUs (N2), 0.9-8 GB RAM per vCPU

### **Machine Family Selection**
```
General purpose, cost-effective        → E2 (shared-core available)
Balanced workload                      → N2 or N2D (AMD)
Compute-intensive (HPC, gaming)        → C2 or C2D
Memory-intensive (large DB)            → M2 or M3 (up to 12 TB RAM!)
GPU workloads (ML, rendering)          → A2 + GPUs
Scale-out, containerized               → T2A (ARM, cost-effective)
```

### **Shared-Core vs Standard**
- **Shared-core**: e2-micro, e2-small, e2-medium (< 1 vCPU)
  - **Use**: Low-traffic web servers, development
  - **Cost**: Cheapest option
  - **Free tier**: f1-micro, e2-micro
  
- **Standard**: 1+ vCPU, dedicated CPU time
  - **Use**: Production workloads

---

## 💾 Disks & Storage

### **Disk Types**
| Type | Use Case | Performance | Cost |
|------|----------|-------------|------|
| **Persistent Disk (PD) Standard** | Sequential I/O, big data | 0.75 IOPS/GB | Cheapest |
| **PD Balanced** | General purpose, balance cost/performance | 6 IOPS/GB | Mid-range |
| **PD SSD** | High random IOPS, databases | 30 IOPS/GB | More expensive |
| **PD Extreme** | Highest IOPS/throughput | 100,000+ IOPS | Most expensive |
| **Local SSD** | Temporary, ultra-high performance | 680,000 IOPS | Lost on stop/restart |
| **Hyperdisk** | Next-gen, flexible provisioning | Up to 1.2M IOPS | Premium, newest |

**Exam Clues:**
- "Boot disk, general purpose" → **PD Balanced**
- "High-performance database" → **PD SSD** or **PD Extreme**
- "Need 100,000+ IOPS" → **PD Extreme** or **Hyperdisk**
- "Scratch disk, cache, temporary data" → **Local SSD**
- "Big data, Hadoop" → **PD Standard** (cost-effective)

### **Disk Sizing & Limits**
- **Boot disk**: 10 GB - 64 TB
- **PD Standard/Balanced/SSD**: Up to 64 TB per disk
- **PD Extreme**: Up to 64 TB per disk
- **Local SSD**: 375 GB per disk (max 24 disks = 9 TB per instance)
- **Max disks per instance**: 128 (PD), 24 (Local SSD)

### **Disk Snapshots**
- **What**: Point-in-time backup of persistent disk
- **Incremental**: Only changed blocks after first snapshot
- **Regional or Multi-regional**: Store in different location for DR
- **Restore**: Create new disk from snapshot
- **Schedule**: Automated snapshot schedules (daily, weekly, etc.)
- **Cost**: Pay for storage used
- **Exam Tip**: Use for backup, DR, and disk cloning

### **Local SSD vs Persistent Disk**
- **Local SSD**: 
  - ✅ Ultra-high performance (680,000 IOPS)
  - ✅ Low latency (< 1ms)
  - ❌ Data lost on stop, restart, or instance termination
  - ❌ No snapshots
  - **Use**: Caching, scratch disk, temporary processing
  
- **Persistent Disk**:
  - ✅ Data persists independently of instance
  - ✅ Snapshots, replication, encryption
  - ✅ Can detach and attach to different instances
  - ❌ Lower performance than Local SSD
  - **Use**: Boot disk, databases, persistent storage

---

## 🔄 Instance Groups & Autoscaling

### **Managed Instance Group (MIG)**
- **What**: Collection of identical VM instances managed as single entity
- **Key Features**:
  - **Auto-healing**: Recreate unhealthy instances (health checks)
  - **Auto-scaling**: Add/remove instances based on load
  - **Rolling updates**: Gradual update of instance template
  - **Load balancing**: Distribute traffic across instances
  - **Regional MIG**: Multi-zone (HA), 99.99% SLA
  - **Zonal MIG**: Single zone

### **Stateless vs Stateful MIG**
- **Stateless MIG**: 
  - Instances interchangeable
  - Use with load balancers
  - **Use**: Web servers, API servers, microservices
  
- **Stateful MIG**:
  - Preserves instance name, attached disks, metadata
  - **Use**: Databases, game servers, stateful apps
  - **Note**: Can't auto-scale (manual scaling only)

### **Autoscaling Policies**
Autoscaling triggers (can combine multiple):
- **CPU utilization**: Scale at X% CPU usage
- **HTTP(S) load balancing utilization**: Scale based on requests
- **Cloud Monitoring metrics**: Custom metrics (e.g., queue depth)
- **Schedule-based**: Scale at specific times (predictable traffic)

**Configuration:**
- **Min instances**: Always running (avoid zero for production)
- **Max instances**: Cost protection
- **Cooldown period**: Wait before scaling again (default 60s)
- **Scale-in controls**: Prevent aggressive scale-down

**Exam Tips:**
- Use multiple metrics for better scaling decisions
- Set reasonable cooldown periods
- Use predictive autoscaling for predictable patterns

### **Rolling Updates**
Update instance template with zero downtime:
- **Max surge**: How many instances can be added during update
- **Max unavailable**: How many can be down during update
- **Update type**:
  - **Proactive**: Update all instances immediately
  - **Opportunistic**: Update instances when they're recreated
- **Canary update**: Test on subset before full rollout
- **Exam Clue**: "Update instances with zero downtime" → Rolling update on MIG

---

## 🚀 High Availability & Fault Tolerance

### **Regional MIG (Recommended for HA)**
- Distributes instances across multiple zones in region
- **Benefits**:
  - Survives zone failures
  - 99.99% SLA (vs 99.5% single zone)
  - Automatic rebalancing
- **Exam Clue**: "High availability across zones" → Regional MIG

### **Instance Templates**
- **What**: Configuration blueprint for VM instances
- **Includes**: Machine type, disk, network, metadata, service account
- **Immutable**: Can't edit, must create new version
- **Versioning**: Use for rolling updates
- **Best Practice**: Store common configs in templates

### **Health Checks**
- **Purpose**: Determine if instance is healthy
- **Types**: HTTP, HTTPS, TCP, SSL
- **Used by**: 
  - Load balancers (route traffic to healthy instances)
  - MIG auto-healing (recreate unhealthy instances)
- **Configuration**:
  - **Check interval**: How often to check (default 5s)
  - **Timeout**: Max time for response (default 5s)
  - **Healthy threshold**: Consecutive successes to mark healthy (default 2)
  - **Unhealthy threshold**: Consecutive failures to mark unhealthy (default 2)

### **Live Migration**
- **What**: Move running instance to different host without downtime
- **When**: Host maintenance, hardware failure
- **Supported**: Most machine types (except GPU, preemptible, A2)
- **Configuration**: Can set maintenance behavior:
  - **Migrate** (default): Live migrate during maintenance
  - **Terminate**: Stop instance during maintenance
- **Exam Tip**: Live migration = zero downtime for most instances

---

## 💰 Cost Optimization

### **Committed Use Discounts (CUD)**
- **1-year commitment**: 25-37% discount
- **3-year commitment**: 52-55% discount
- **Types**:
  - **Resource-based**: Commit to specific machine type in region
  - **Spend-based**: Commit to minimum spend (flexible across types/regions)
- **Exam Clue**: "Predictable workload, long-term" → CUD

### **Sustained Use Discounts (Automatic)**
- Automatic discount for running instances > 25% of month
- Up to 30% discount for continuous use
- **No commitment needed**: Automatically applied

### **Spot VMs (Formerly Preemptible)**
- **Discount**: Up to 91% off
- **Use**: Batch jobs, fault-tolerant workloads, rendering
- **Limitations**: 
  - Can be terminated anytime (30-second warning)
  - No SLA
  - May not always be available
- **Best Practices**:
  - Handle 30-second shutdown script
  - Checkpoint work frequently
  - Use with MIG for automatic recreation
- **Exam Clue**: "Cost-effective batch processing" → Spot VMs

### **Rightsizing Recommendations**
- GCP analyzes utilization and recommends smaller machine types
- **Use**: Cloud Console → Compute Engine → Recommendations
- **Can save**: 20-60% on underutilized instances
- **Exam Tip**: Mention rightsizing for cost optimization questions

### **Cost Optimization Strategies**
```
Predictable workload                  → CUD (committed use)
Batch jobs, fault-tolerant            → Spot VMs (up to 91% off)
Short-lived, < 1 hour                 → Spot VMs
Steady-state workload                 → Regular instances (sustained use discount)
Varying workload                      → Autoscaling + E2 family
Dev/test environments                 → Schedule start/stop, Spot VMs
```

---

## 🔐 Security & Access

### **Service Accounts**
- **What**: Identity for instances to access GCP services
- **Default service account**: Created automatically (don't use in production!)
- **Custom service account**: Create with minimum permissions (recommended)
- **Best Practices**:
  - One service account per application/function
  - Use IAM roles, not access scopes
  - Never use default service account in production

### **Access Scopes (Legacy)**
- **What**: Permissions for instances to access Google Cloud APIs
- **Limitation**: Can only restrict, not grant permissions
- **Modern approach**: Use IAM roles instead
- **Exam Tip**: If question mentions access scopes, recommend IAM roles

### **OS Login**
- **What**: Manage SSH access using IAM
- **Benefits**:
  - Centralized SSH key management
  - IAM-based access control
  - Audit logging (who accessed when)
- **Roles**: 
  - `compute.osLogin`: SSH access
  - `compute.osAdminLogin`: SSH + sudo access
- **Enable**: Project or instance metadata
- **Exam Clue**: "Manage SSH access with IAM" → OS Login

### **Shielded VMs**
- **What**: Hardened VM with security features
- **Components**:
  - **Secure Boot**: Verify boot components
  - **vTPM**: Virtual Trusted Platform Module
  - **Integrity Monitoring**: Detect boot-time changes
- **Use**: Compliance, high-security workloads
- **Exam Tip**: Recommended for production workloads

### **Confidential VMs**
- **What**: Encrypt data in use (memory encryption)
- **Use**: Highly sensitive data (PII, medical records)
- **Technology**: AMD SEV (Secure Encrypted Virtualization)
- **Supported**: N2D, C2D machine types
- **Exam Clue**: "Encrypt data in memory" → Confidential VMs

---

## 🛠️ Operations & Management

### **Startup Scripts & Metadata**
- **Startup script**: Runs when instance starts
  - **Use**: Install software, configure instance, fetch configs
  - **Can be**: Local file or Cloud Storage URL
  - **Runs as**: root user
  
- **Metadata**: Key-value pairs stored with instance
  - **Access**: From instance via metadata server (169.254.169.254)
  - **Use**: Configuration, secrets (use Secret Manager instead!)
  - **Examples**: startup-script, ssh-keys, enable-oslogin

**Metadata Server:**
```bash
# Query metadata from instance
curl "http://metadata.google.internal/computeMetadata/v1/instance/hostname" -H "Metadata-Flavor: Google"
```

### **Images & Image Families**
- **Image**: Bootable disk snapshot
- **Image family**: Group of images (latest automatically selected)
- **Types**:
  - **Public images**: Google-provided (Debian, Ubuntu, CentOS)
  - **Custom images**: Your own images
  - **Shared images**: From another project
- **Best Practice**: 
  - Create "golden images" with pre-installed software
  - Use image families for versioning

### **Instance Cloning**
- **Create similar instance**: Clone existing instance configuration
- **Note**: Creates new instance, doesn't copy data
- **Use**: Quickly create similar instances

---

## 📦 GPU & Specialized Hardware

### **GPUs**
- **Use**: ML training/inference, rendering, HPC
- **Machine types**: N1, A2, G2
- **GPU types**:
  - **NVIDIA T4**: Inference, low cost
  - **NVIDIA V100**: Training, high performance
  - **NVIDIA A100**: Latest, highest performance
  - **NVIDIA L4**: Latest, inference + training
- **Limits**: 
  - Not available in all zones
  - Can't live migrate
  - No preemptible (but can use Spot)
- **Exam Clue**: "ML training workload" → A2 + A100 GPUs

### **TPUs (Tensor Processing Units)**
- **What**: Google's custom ASIC for ML workloads
- **Use**: Large-scale ML training (TensorFlow, PyTorch)
- **Performance**: 10-100x faster than GPUs for specific workloads
- **Types**: TPU v2, v3, v4 (Pods for large scale)
- **Exam Clue**: "Fastest ML training, TensorFlow" → TPUs

### **Sole-Tenant Nodes**
- **What**: Physical Compute Engine server dedicated to your project
- **Use**: Compliance (BYOL), performance, security
- **Benefits**:
  - Meet compliance requirements
  - Avoid noisy neighbors
  - BYOL (bring your own license)
- **Cost**: More expensive (pay for entire host)
- **Exam Clue**: "Compliance requires dedicated hardware" → Sole-tenant

---

## 🔄 Migration to GCP

### **Migrate for Compute Engine (M4CE)**
- **What**: Migrate VMs from on-prem or other clouds to GCP
- **Supported sources**:
  - VMware (vSphere)
  - AWS
  - Azure
  - Physical servers
- **Process**:
  1. **Deploy replication agents**: On source VMs
  2. **Continuous replication**: Stream data to GCP
  3. **Test clones**: Validate migration
  4. **Cutover**: Switch to GCP
- **Benefits**:
  - Minimal downtime (continuous replication)
  - Test before cutover
  - Rollback capability
- **Exam Clue**: "Migrate VMs from VMware/AWS with minimal downtime" → M4CE

### **Migration Strategies**
```
VMware VMs → GCP                      → Migrate for Compute Engine
AWS EC2 → GCP                         → Migrate for Compute Engine
Lift-and-shift → Optimize later      → Migrate for Compute Engine
Containerize workloads                → Migrate to GKE (not Compute Engine)
Rewrite for serverless                → Cloud Run / Cloud Functions
```

---

## 🎓 Exam Decision Trees

### **Machine Type Selection**
```
Cost-effective, general purpose       → E2
Balanced workload                     → N2 or N2D
High CPU, compute-intensive           → C2 or C2D
Large memory (SAP HANA, in-memory DB) → M2 or M3
GPU workload (ML, rendering)          → A2 + GPUs
ARM-based, containerized              → T2A (cost-effective)
```

### **Disk Type Selection**
```
Boot disk                             → PD Balanced
High-performance database             → PD SSD or PD Extreme
Need 100,000+ IOPS                    → PD Extreme or Hyperdisk
Temporary data, cache                 → Local SSD (data lost on stop!)
Big data, Hadoop                      → PD Standard (cost-effective)
```

### **High Availability**
```
Single zone application               → Zonal MIG
Multi-zone HA (99.99% SLA)            → Regional MIG
Stateless web servers                 → Regional MIG + load balancer
Stateful applications                 → Stateful MIG
Need custom instance types            → Unmanaged instance group (rare)
```

### **Cost Optimization**
```
Predictable, long-term workload       → CUD (1-year or 3-year)
Batch jobs, fault-tolerant            → Spot VMs (up to 91% off)
Dev/test environments                 → Schedule start/stop + Spot VMs
Auto-scaling workload                 → MIG + E2 family
Continuous usage                      → Sustained use discount (automatic)
```

### **Security**
```
Manage SSH access with IAM            → OS Login
Encrypt data in memory                → Confidential VMs
Compliance workload                   → Shielded VMs
Service account access                → Custom service account + IAM roles
Dedicated hardware for compliance     → Sole-tenant nodes
```

---

## ⚡ Quick Reminders

### **Important Limits**
- **Instances per project**: 2000 (can request increase)
- **vCPUs per region**: Quota-based (request increase)
- **Persistent disks per instance**: 128
- **Local SSDs per instance**: 24 (9 TB total)
- **Max disk size**: 64 TB
- **Max memory**: 12 TB (M3 mega-memory)

### **SLA & Availability**
- **Single instance**: 99.5% (multi-zone PDs)
- **Regional MIG**: 99.99%
- **Zonal MIG**: 99.5%
- **No SLA**: Preemptible/Spot VMs

### **Performance**
- **PD Standard**: 0.75 IOPS/GB (seq I/O)
- **PD Balanced**: 6 IOPS/GB (general purpose)
- **PD SSD**: 30 IOPS/GB (high performance)
- **PD Extreme**: 100,000+ IOPS
- **Local SSD**: 680,000 read IOPS, 360,000 write IOPS

### **Common Gotchas**
- **Local SSD**: Data lost on stop/restart (ephemeral!)
- **Preemptible/Spot**: Can be terminated anytime (30s warning)
- **Live migration**: Not supported for GPU, preemptible, A2 instances
- **Instance templates**: Immutable (can't edit, must create new)
- **Access scopes**: Legacy, use IAM roles instead
- **Default service account**: Don't use in production!

---

## 🔍 Troubleshooting Quick Checks

**Instance won't start:**
- ✅ Check quota limits (vCPUs, disk space)
- ✅ Verify boot disk is valid
- ✅ Check zone has capacity
- ✅ Review serial console logs
- ✅ Verify service account has necessary permissions

**Can't SSH to instance:**
- ✅ Check firewall allows port 22
- ✅ Verify instance has external IP (or use IAP tunnel)
- ✅ Check SSH keys are configured correctly
- ✅ Review OS Login settings
- ✅ Check metadata for startup-script errors

**Instance performance issues:**
- ✅ Check CPU, memory, disk utilization in Cloud Monitoring
- ✅ Review disk throughput (might need PD SSD)
- ✅ Check for IOPS throttling
- ✅ Consider rightsizing (might need larger machine type)
- ✅ Review sustained use patterns

**Autoscaling not working:**
- ✅ Check autoscaling policy configuration
- ✅ Verify health checks are passing
- ✅ Review cooldown period (might be too long)
- ✅ Check min/max instance limits
- ✅ Verify metrics are being collected

**High costs:**
- ✅ Identify idle instances (stop or delete)
- ✅ Review machine types (rightsizing recommendations)
- ✅ Consider CUD for long-running instances
- ✅ Use Spot VMs for batch jobs
- ✅ Check for orphaned persistent disks
- ✅ Review snapshot storage costs

---

## 📚 Pro Tips for Exam

1. **Read carefully**: Look for keywords like "cost-effective", "high availability", "GPU", "fault-tolerant"
2. **Machine families**: Match workload to right family (E2=cheap, C2=compute, M2=memory)
3. **Regional MIG**: Default choice for HA (99.99% SLA)
4. **Spot VMs**: For batch jobs and cost savings (up to 91% off)
5. **Local SSD**: Fast but ephemeral (data lost on stop!)
6. **OS Login**: Modern way to manage SSH access with IAM
7. **Service accounts**: Always use custom, not default
8. **Live migration**: Remember GPU instances can't live migrate
9. **CUD**: Committed use discounts for predictable workloads
10. **M4CE**: Migrate for Compute Engine for VM migrations

---

## 🎯 Memorization Shortcuts

**Cost-effective general purpose:**
E2 (cheapest option)

**High CPU compute workload:**
C2/C2D (compute-optimized)

**Large in-memory database:**
M2/M3 (up to 12 TB RAM)

**ML training workload:**
A2 + A100 GPUs (or TPUs for TensorFlow)

**High availability across zones:**
Regional MIG + Load Balancer

**Cost-effective batch jobs:**
Spot VMs (up to 91% off)

**Fast temporary storage:**
Local SSD (but data lost on stop!)

**High-performance database disk:**
PD SSD or PD Extreme

**Manage SSH access with IAM:**
OS Login (modern approach)

**Encrypt data in memory:**
Confidential VMs (N2D, C2D)

**Migrate VMs with minimal downtime:**
Migrate for Compute Engine (M4CE)

**Zero-downtime updates:**
Rolling update on MIG

---

## 🧪 Scenario-Based Examples

**Scenario 1**: "Web application needs to auto-scale based on traffic, survive zone failures, and minimize costs."
- **Answer**: Regional MIG with autoscaling + E2 instances + Spot VMs for burst capacity

**Scenario 2**: "Gaming company needs high CPU performance for game servers with predictable workload."
- **Answer**: C2 or C2D instances + 3-year CUD for maximum discount

**Scenario 3**: "Data science team needs to run ML training jobs overnight, budget is limited."
- **Answer**: A2 instances with A100 GPUs + Spot VMs (91% discount)

**Scenario 4**: "Financial company needs to encrypt sensitive data in memory, comply with regulations."
- **Answer**: Confidential VMs (N2D) + Shielded VMs + CMEK encryption

**Scenario 5**: "Update 500 web server instances with new application version without downtime."
- **Answer**: Create new instance template → Rolling update on MIG (configure max surge/unavailable)

**Scenario 6**: "Database needs ultra-high IOPS (150,000) with persistent storage."
- **Answer**: PD Extreme disk (can provision up to 120,000 IOPS per disk)

**Scenario 7**: "Migrate 200 VMware VMs to GCP with minimal downtime and testing capability."
- **Answer**: Migrate for Compute Engine (M4CE) with continuous replication and test clones

**Scenario 8**: "Startup needs to manage SSH access for 100 developers across 500 instances using IAM."
- **Answer**: Enable OS Login + assign compute.osLogin or compute.osAdminLogin IAM roles

**Scenario 9**: "Video rendering farm needs fast temporary storage, data doesn't need to persist."
- **Answer**: Instances with Local SSD (680,000 IOPS, ephemeral storage is OK for temp data)

**Scenario 10**: "E-commerce site has predictable traffic spike every weekend, normal traffic weekdays."
- **Answer**: MIG with schedule-based autoscaling + min instances for weekday + scale up on weekends