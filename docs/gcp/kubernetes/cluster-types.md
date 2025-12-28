# GKE Cluster Types

## Description

GKE offers two distinct cluster modes of operation: **Standard** and **Autopilot**. Each provides different levels of control, management responsibility, and pricing models. Understanding the differences is crucial for choosing the right cluster type for your workload.

## Standard GKE Clusters

### Description

Standard GKE clusters give you complete control over cluster configuration and node management. You're responsible for configuring node pools, managing scaling, security, and updates while Google manages the control plane.

**Model**: You manage nodes, Google manages control plane.

### Key Features

#### Full Node Control

- **Custom Machine Types**: Choose any Compute Engine machine type
- **Node Customization**: Configure boot disk, local SSDs, GPUs, taints, labels
- **SSH Access**: Direct SSH access to nodes for debugging
- **Custom Images**: Use custom node images if needed
- **DaemonSets**: Run privileged pods on all nodes

#### Flexible Node Pools

- **Multiple Node Pools**: Create pools with different machine types
- **Node Taints and Labels**: Control pod scheduling
- **Spot VMs**: Use preemptible/spot VMs for cost savings
- **Node Pool Management**: Manual or automated scaling per pool

#### Networking Options

- **Routes-Based**: Traditional Kubernetes networking
- **VPC-Native**: Alias IP ranges (recommended)
- **Network Policies**: Calico or GKE Dataplane V2
- **Private Clusters**: No public IPs on nodes

#### Advanced Features

- **Windows Node Pools**: Run Windows containers
- **Multi-Cluster Ingress**: Share load balancers across clusters
- **Config Connector**: Manage GCP resources as Kubernetes objects
- **Istio/ASM**: Full service mesh capabilities

### Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Nodes per cluster** | 15,000 | Across all node pools |
| **Node pools per cluster** | 1,000 | Different configurations |
| **Pods per node** | 110 (default), 256 (max) | Via --max-pods-per-node |
| **Pods per cluster** | 200,000 | Theoretical maximum |
| **Services per cluster** | 10,000 | LoadBalancer type limited |

### When to Use Standard GKE

✅ **Use Standard When:**

1. **Custom Infrastructure Requirements**

   - Need specific machine types or custom configurations
   - Require GPU or TPU nodes
   - Need local SSDs for high-performance storage
   - Custom kernel modules or system-level modifications

2. **Full Control Over Nodes**

   - Need SSH access to nodes for debugging
   - Want to run privileged pods or DaemonSets
   - Require custom node images
   - Need to configure node-level security

3. **Windows Workloads**

   - Running Windows containers
   - Mixed Linux/Windows workloads
   - .NET Framework applications

4. **Cost Optimization with Spot VMs**

   - Fault-tolerant batch workloads
   - CI/CD pipelines
   - Development/testing environments
   - Up to 91% cost savings acceptable with interruptions

5. **Complex Networking Requirements**

   - Multiple network interfaces
   - Custom CNI plugins
   - Advanced network policies
   - Specific IP address management

❌ **Don't Use Standard When:**

1. **Want Minimal Management**

   - Team lacks Kubernetes operations expertise
   - Prefer hands-off infrastructure management
   - Don't want to manage node scaling/updates

2. **Unpredictable Workloads**

   - Highly variable traffic patterns
   - Sporadic batch jobs
   - Cost efficiency more important than control

3. **Simplicity is Priority**

   - Small team without dedicated platform engineers
   - Rapid prototyping and development
   - Quick time-to-market needed

### Pricing

**Standard GKE Costs:**

- **Control Plane**: 
  - Zonal clusters: Free
  - Regional clusters: $0.10/hour

- **Nodes**: Standard Compute Engine pricing
  - e2-medium: ~$0.03/hour
  - Spot VMs: ~$0.008/hour (up to 91% discount)

- **Networking**: Egress charges apply

**Cost Optimization:**

```bash
# Use Spot VMs (preemptible) for cost savings
gcloud container node-pools create spot-pool \
  --cluster=my-cluster \
  --machine-type=e2-medium \
  --spot \
  --num-nodes=3

# Enable cluster autoscaling
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10
```

---

## Autopilot GKE Clusters

### Description

Autopilot is a fully managed GKE mode where Google manages the entire cluster infrastructure including nodes, node pools, scaling, security, and networking. You only configure and deploy your pods.

**Model**: Google manages everything, you manage workloads.

### Key Features

#### Fully Managed Infrastructure

- **No Node Management**: Google provisions and scales nodes automatically
- **Automatic Scaling**: Nodes scale based on pod resource requests
- **Hardened Security**: Security best practices enforced by default
- **Automatic Updates**: Both control plane and nodes updated automatically
- **Optimized Configuration**: Google-optimized settings for performance and cost

#### Simplified Operations

- **No Node Pools**: Infrastructure abstracted away
- **Per-Pod Billing**: Pay only for CPU and memory requested by pods
- **Resource-Based Scaling**: Nodes added/removed based on pod requests
- **Hands-Off Upgrades**: No maintenance windows or disruption management

#### Built-In Security

- **Workload Identity**: Enabled by default
- **Shielded Nodes**: All nodes use shielded GKE nodes
- **Secure by Default**: Security best practices enforced
- **No SSH Access**: Nodes are not directly accessible (improved security)

#### Restrictions (for Security and Optimization)

- **No Privileged Pods**: Cannot run privileged containers
- **No Host Network**: Pods can't use host networking
- **No DaemonSets**: With node selectors (some exceptions apply)
- **Predefined Pod Resources**: Must specify CPU/memory requests
- **No Windows Nodes**: Linux only

### Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Pods per cluster** | Auto-managed | Scales based on demand |
| **Pod CPU request** | 0.25 to 32 vCPU | In 0.25 vCPU increments |
| **Pod memory request** | 0.5 to 128 GB | Specific ratios to CPU |
| **Ephemeral storage** | Up to 10 GB included | Per pod |
| **Persistent volumes** | Unlimited (quota-based) | Standard limits apply |
| **Services (LoadBalancer)** | Auto-managed | Google handles capacity |

### Pod Resource Classes

Autopilot uses predefined CPU-to-memory ratios:

| Class | CPU:Memory Ratio | Example |
|-------|------------------|---------|
| **General Purpose** | 1:4 GB | 1 vCPU : 4 GB RAM |
| **Scale-Out** | 1:1 GB | 1 vCPU : 1 GB RAM |
| **Balanced** | 1:2 GB | 1 vCPU : 2 GB RAM |
| **Memory-Optimized** | 1:6.5 GB | 1 vCPU : 6.5 GB RAM |

### When to Use Autopilot GKE

✅ **Use Autopilot When:**

1. **Minimal Operational Overhead**

   - Small team or no dedicated platform engineers
   - Want Google to handle all infrastructure decisions
   - Prefer hands-off cluster management
   - Focus on application deployment, not infrastructure

2. **Unpredictable or Variable Workloads**

   - Traffic patterns vary significantly
   - Batch jobs with sporadic execution
   - Development and testing environments
   - Cost efficiency through automatic scaling

3. **Security is Critical**

   - Want hardened defaults
   - Need compliance with security baselines
   - Prefer least-privilege by default
   - Don't need privileged containers

4. **Optimal Cost Management**

   - Pay only for what you use (per-pod resources)
   - No over-provisioning nodes
   - Automatic right-sizing
   - Scale-to-zero capability

5. **Standard Kubernetes Workloads**

   - Stateless applications
   - Microservices
   - HTTP services and APIs
   - Standard containerized applications

❌ **Don't Use Autopilot When:**

1. **Need Privileged Access**

   - Running privileged containers
   - DaemonSets with node selectors
   - Host networking required
   - SSH access to nodes needed

2. **Custom Node Configuration**

   - Specific machine types required
   - GPUs or TPUs needed
   - Local SSDs for storage
   - Custom node images

3. **Windows Workloads**

   - Running Windows containers
   - .NET Framework applications
   - Windows-specific requirements

4. **Special Network Requirements**

   - Custom CNI plugins
   - Multiple network interfaces
   - Non-standard networking configurations

5. **Very Stable, Predictable Workloads**

   - 24/7 steady-state traffic
   - Reserved capacity might be cheaper
   - Committed use discounts apply (Standard with CUDs may be cheaper)

### Pricing

**Autopilot GKE Costs:**

- **vCPU**: $0.0445/hour per vCPU requested
- **Memory**: $0.00488/hour per GB requested
- **Control Plane**: Included in pod pricing
- **Networking**: Egress charges apply

**Example Cost Calculation:**

```
Pod Request: 1 vCPU, 4 GB RAM
Hourly Cost: (1 × $0.0445) + (4 × $0.00488) = $0.064/hour
Monthly Cost: $0.064 × 730 hours = ~$46.72/month per pod
```

**Cost Optimization:**

```yaml
# Right-size pod resources - you pay for requests
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:

  - name: app
    image: my-app:latest
    resources:
      requests:
        cpu: "250m"      # 0.25 vCPU
        memory: "512Mi"  # 0.5 GB
      limits:
        cpu: "1000m"
        memory: "2Gi"
```

---

## Comparison Matrix

| Feature | Standard GKE | Autopilot GKE |
|---------|--------------|---------------|
| **Node Management** | Manual configuration | Fully automated |
| **Pricing** | Per node-hour | Per pod resource request |
| **Scaling** | Configure autoscaling | Automatic |
| **Node Pools** | Manual creation/config | Not applicable |
| **Machine Types** | Full choice | Google-optimized |
| **SSH to Nodes** | Yes | No |
| **Privileged Pods** | Yes | No |
| **DaemonSets** | Yes (unrestricted) | Limited |
| **GPU/TPU** | Yes | Limited GPU support |
| **Windows Nodes** | Yes | No |
| **Local SSDs** | Yes | No |
| **Control Plane Cost** | $0.10/hr (regional) | Included |
| **Security Baseline** | Manual config | Hardened by default |
| **Best For** | Custom requirements | Simplicity, cost efficiency |

---

## Choosing Between Standard and Autopilot

### Decision Framework

```
Start Here: Do you need Windows nodes, GPUs, or privileged containers?
    │
    ├─ Yes → Standard GKE
    │
    └─ No
        │
        Do you have dedicated platform/ops team?
        │
        ├─ Yes → Do you need custom node configuration?
        │   │
        │   ├─ Yes → Standard GKE
        │   └─ No → Autopilot GKE (less operational overhead)
        │
        └─ No → Autopilot GKE (hands-off management)
```

### When to Choose Standard

- Full control over infrastructure
- Custom hardware requirements (GPU, TPU, local SSD)
- Windows workloads
- Privileged containers or DaemonSets
- Spot VMs for cost optimization
- Experienced Kubernetes operations team

### When to Choose Autopilot

- Minimal operational overhead
- Pay-per-pod cost model
- Unpredictable or variable workloads
- Security hardening by default
- Small team or no dedicated ops
- Standard containerized applications

---

## Migration Considerations

### Standard to Autopilot

**Potential Issues:**

- Privileged pods will be rejected
- DaemonSets with node selectors may not work
- Need to define resource requests/limits
- Custom node configurations lost

**Migration Path:**

1. Audit existing workloads for incompatibilities
2. Add resource requests/limits to all pods
3. Remove privileged security contexts
4. Test in new Autopilot cluster
5. Migrate workloads gradually

### Autopilot to Standard

**Reasons to Switch:**

- Need GPU/TPU support
- Require Windows nodes
- Want Spot VM cost savings
- Need privileged containers

**Migration Path:**

1. Create Standard cluster with similar configuration
2. Configure node pools and autoscaling
3. Migrate workloads
4. Optimize node pool configuration

---

## Regional vs Zonal Clusters

Both Standard and Autopilot support regional and zonal deployments:

### Zonal Clusters

**Characteristics:**

- Control plane in single zone
- Nodes in single zone (Standard) or multi-zone (Standard with manual pools)
- Lower cost (free control plane for Standard)
- Lower SLA (no SLA for zonal)

**Use Cases:**

- Development and testing
- Non-critical workloads
- Cost-sensitive applications

### Regional Clusters

**Characteristics:**

- Control plane replicated across 3 zones
- Nodes distributed across zones
- Higher availability (99.95% SLA)
- Higher cost ($0.10/hour for Standard, included for Autopilot)

**Use Cases:**

- Production workloads
- High-availability requirements
- Mission-critical applications

---

## Best Practices

### For Standard Clusters

1. **Use Multiple Node Pools**

   - Separate pools for different workload types
   - Production vs. batch workloads
   - Different machine types for different needs

2. **Enable Autoscaling**

   - Configure cluster autoscaler
   - Set appropriate min/max nodes
   - Use node affinity for workload placement

3. **Use Spot VMs Wisely**

   - Only for fault-tolerant workloads
   - Not for critical services
   - Implement pod disruption budgets

4. **Right-Size Nodes**

   - Don't over-provision nodes
   - Use smaller nodes for better bin packing
   - Monitor utilization and adjust

### For Autopilot Clusters

1. **Define Resource Requests**

   - Required for all pods
   - Directly impacts cost
   - Use VPA for recommendations

2. **Optimize Pod Resources**

   - Right-size requests to actual usage
   - Avoid over-requesting resources
   - Use Vertical Pod Autoscaler

3. **Understand Restrictions**

   - No privileged pods
   - Limited DaemonSet capabilities
   - Plan accordingly

4. **Leverage Auto-Scaling**

   - HPA for pod replicas
   - Let Autopilot handle nodes
   - No need to configure cluster autoscaler

---

## Related Resources

- [GKE Overview](gke-overview.md)
- [GKE Deployments](gke-deployments.md)
- [GKE Scaling](gke-scaling.md)
- [kubectl](kubectl.md)
