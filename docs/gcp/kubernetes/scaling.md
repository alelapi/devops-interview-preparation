# GKE Scaling

## Description

Scaling in GKE encompasses multiple dimensions: horizontal pod scaling (adding more pod replicas), vertical pod scaling (increasing pod resources), and cluster scaling (adding more nodes). GKE provides automated scaling mechanisms to handle variable workloads efficiently while optimizing costs.

**Concept**: Automatically adjust resources (pods and nodes) based on demand to maintain performance while optimizing costs.

## Types of Scaling

### Horizontal Pod Autoscaler (HPA)

Scales the number of pod replicas based on observed metrics.

### Vertical Pod Autoscaler (VPA)

Adjusts CPU and memory requests/limits for containers.

### Cluster Autoscaler

Adds or removes nodes based on pod resource requirements.

### Multidimensional Pod Autoscaler (MPA)

Scales both pod replicas and resources (GKE Autopilot feature).

## Horizontal Pod Autoscaler (HPA)

### Description

HPA automatically scales the number of pods in a deployment, replica set, or stateful set based on observed CPU utilization, memory usage, or custom metrics.

### Key Features

- **CPU-based Scaling**: Scale based on CPU utilization (most common)
- **Memory-based Scaling**: Scale based on memory usage
- **Custom Metrics**: Scale on application-specific metrics (Pub/Sub queue length, HTTP requests/sec)
- **External Metrics**: Scale based on metrics from external systems
- **Multiple Metrics**: Combine different metrics for scaling decisions
- **Configurable Behavior**: Set min/max replicas, scaling velocity

### When to Use HPA

✅ **Use HPA When:**

- Traffic varies throughout the day
- Want automatic scaling based on load
- Cost optimization through dynamic scaling
- Stateless applications that can scale horizontally

❌ **Don't Use HPA When:**

- Stateful applications that can't easily add replicas
- Applications with long startup times (scale-up lag)
- Need vertical scaling (use VPA instead)

---

## Vertical Pod Autoscaler (VPA)

### Description

VPA automatically adjusts CPU and memory requests and limits for containers based on historical usage patterns.

### Key Features

- **Right-Sizing**: Automatically set appropriate resource requests
- **Historical Analysis**: Based on actual resource usage patterns
- **Update Modes**: Recommend, auto-update, or initial-only
- **Container-Level**: Can configure per-container in a pod

### VPA Update Modes

**Off (Recommendation Only):**

```yaml
updatePolicy:
  updateMode: "Off"
```

- VPA only generates recommendations
- No automatic updates
- Use for analysis before implementing

**Initial:**

```yaml
updatePolicy:
  updateMode: "Initial"
```

- Set resources only when pods are created
- No updates to running pods
- Good for stateful workloads

**Auto:**

```yaml
updatePolicy:
  updateMode: "Auto"
```

- Automatically update running pods
- Evicts and recreates pods to apply new resources
- Best for stateless workloads

**Recreate:**

```yaml
updatePolicy:
  updateMode: "Recreate"
```

- Similar to Auto but always recreates pods
- More disruptive

### Important Considerations

**VPA Limitations:**

- Cannot be used with HPA on same CPU/memory metrics
- Requires pod eviction to apply changes (except Initial mode)
- May cause brief downtime during updates

### When to Use VPA

✅ **Use VPA When:**

- Resource requests are incorrect or unknown
- Want automatic right-sizing based on usage
- Applications with varying resource needs over time
- Optimizing cost by eliminating over-provisioning

❌ **Don't Use VPA When:**

- Already using HPA on CPU/memory
- Cannot tolerate pod evictions
- Resource requirements are well-known and stable
- Application startup time is very long

---

## Cluster Autoscaler

### Description

Cluster Autoscaler automatically adjusts the number of nodes in a cluster based on pod resource requests that cannot be scheduled on existing nodes.

### How It Works

1. Pods are unschedulable due to insufficient resources
2. Cluster Autoscaler detects pending pods
3. New nodes added to accommodate pods
4. When nodes are underutilized, they're removed

### Key Features

- **Automatic Scale-Up**: Add nodes when pods can't be scheduled
- **Automatic Scale-Down**: Remove nodes when underutilized
- **Node Pool Awareness**: Scale specific node pools
- **Cost Optimization**: Reduce costs by removing unused nodes
- **Configurable Behavior**: Set min/max nodes, scale-down delays

### Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Min nodes per pool** | 0 | Can scale to zero |
| **Max nodes per pool** | 1000 | Configurable |
| **Max nodes per cluster** | 15,000 | Total limit |
| **Scale-up time** | ~2-5 minutes | Node provisioning time |
| **Scale-down time** | 10 minutes (default) | Configurable delay |

### When to Use Cluster Autoscaler

✅ **Use Cluster Autoscaler When:**

- Workload varies significantly
- Want automatic infrastructure scaling
- Cost optimization important
- Using HPA (complement to pod autoscaling)

❌ **Don't Use Cluster Autoscaler When:**

- Workload is stable and predictable
- Cannot tolerate 2-5 minute scale-up delay
- Using Autopilot (handles this automatically)

---

## Autopilot Scaling (GKE Autopilot)

### Description

In Autopilot mode, Google manages all scaling automatically. No cluster autoscaler configuration needed.

### How It Works

- Nodes provisioned automatically based on pod requests
- Scales to zero when no workloads running
- Right-sized nodes for pod requirements
- No over-provisioning

### Autopilot Scaling Features

**Automatic:**

- Node provisioning and removal
- Perfect bin-packing
- Cost optimization
- No configuration needed

**You Still Configure:**

- HPA for pod replica scaling
- VPA for resource recommendations (optional)
- Pod resource requests (required)

**Set up Alerts:**

- HPA at max replicas
- Cluster at max nodes
- High pod pending time
- Scaling failures
