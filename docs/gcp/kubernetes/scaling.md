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

### HPA Configuration

**Basic CPU-based HPA:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Advanced HPA with Multiple Metrics:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: advanced-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 50
  
  # Scaling behavior configuration
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
      policies:
      
      - type: Percent
        value: 50
        periodSeconds: 60  # Scale down max 50% per minute
      
      - type: Pods
        value: 2
        periodSeconds: 60  # Or max 2 pods per minute
      selectPolicy: Min  # Use most conservative policy
    
    scaleUp:
      stabilizationWindowSeconds: 0  # Scale up immediately
      policies:
      
      - type: Percent
        value: 100
        periodSeconds: 30  # Double pods every 30 seconds
      
      - type: Pods
        value: 5
        periodSeconds: 30  # Or add 5 pods every 30 seconds
      selectPolicy: Max  # Use most aggressive policy
  
  metrics:
  
  # CPU utilization

  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  
  # Memory utilization

  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  
  # Custom metric (e.g., HTTP requests per second)

  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

### HPA with Custom Metrics

**Using Pub/Sub Queue Depth:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: pubsub-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pubsub-worker
  minReplicas: 1
  maxReplicas: 50
  metrics:
  
  - type: External
    external:
      metric:
        name: pubsub.googleapis.com|subscription|num_undelivered_messages
        selector:
          matchLabels:
            resource.labels.subscription_id: my-subscription
      target:
        type: AverageValue
        averageValue: "30"  # Scale to maintain 30 messages per pod
```

### Create HPA via kubectl

```bash
# CPU-based autoscaling
kubectl autoscale deployment web-app \
  --cpu-percent=70 \
  --min=2 \
  --max=10

# View HPA status
kubectl get hpa

# Describe HPA
kubectl describe hpa web-app-hpa

# Delete HPA
kubectl delete hpa web-app-hpa
```

### Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Min replicas** | 1 | Can be 0 with specific setup |
| **Max replicas** | Cluster capacity | Limited by resources |
| **Metrics evaluation** | Every 15 seconds | Default, configurable |
| **Scaling cooldown** | 3 min (down), 0 (up) | Configurable via behavior |
| **Target metrics** | Up to 10 | Multiple metrics supported |

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

### VPA Configuration

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  
  updatePolicy:
    updateMode: "Auto"  # Auto, Recreate, Initial, or Off
  
  resourcePolicy:
    containerPolicies:
    
    - containerName: web-app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2000m
        memory: 2Gi
      controlledResources:
      
      - cpu
      - memory
```

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

### Install VPA

```bash
# VPA not installed by default, install manually
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh

# Verify installation
kubectl get vpa -A
```

### View VPA Recommendations

```bash
# Get VPA status
kubectl get vpa

# Describe VPA (see recommendations)
kubectl describe vpa web-app-vpa

# View recommendations
kubectl get vpa web-app-vpa -o yaml
```

### Important Considerations

**VPA Limitations:**

- Cannot be used with HPA on same CPU/memory metrics
- Requires pod eviction to apply changes (except Initial mode)
- May cause brief downtime during updates

**VPA + HPA Together:**

```yaml
# HPA on custom metric
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  metrics:
  
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"

---
# VPA for right-sizing
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-app-vpa
spec:
  updatePolicy:
    updateMode: "Auto"
  # HPA handles CPU scaling, VPA handles right-sizing
```

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

### Enable Cluster Autoscaler (Standard GKE)

```bash
# Enable on existing cluster
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --zone=us-central1-a

# Enable on specific node pool
gcloud container node-pools update default-pool \
  --cluster=my-cluster \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --zone=us-central1-a

# Create cluster with autoscaling
gcloud container clusters create my-cluster \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --zone=us-central1-a
```

### Cluster Autoscaler Configuration

**Node Pool Settings:**

```bash
# Set autoscaling limits
gcloud container node-pools update pool-name \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=20

# Location policy for multi-zone
gcloud container node-pools update pool-name \
  --location-policy=BALANCED  # or ANY
```

**Advanced Configuration via ConfigMap:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-config
  namespace: kube-system
data:
  scale-down-delay-after-add: "10m"
  scale-down-unneeded-time: "10m"
  scale-down-utilization-threshold: "0.5"
  skip-nodes-with-local-storage: "false"
  skip-nodes-with-system-pods: "false"
```

### Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Min nodes per pool** | 0 | Can scale to zero |
| **Max nodes per pool** | 1000 | Configurable |
| **Max nodes per cluster** | 15,000 | Total limit |
| **Scale-up time** | ~2-5 minutes | Node provisioning time |
| **Scale-down time** | 10 minutes (default) | Configurable delay |

### Preventing Scale-Down

**Safe-to-Evict Annotation:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
```

**PodDisruptionBudget:**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

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

```yaml
# Autopilot pod with HPA
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3  # Initial, HPA will adjust
  template:
    spec:
      containers:
      
      - name: web-app
        resources:
          requests:  # Required for Autopilot
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1000m"
            memory: "2Gi"

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 50
  metrics:
  
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Scaling Best Practices

### 1. Always Set Resource Requests

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1000m"
    memory: "1Gi"
```

**Why:**

- Required for HPA and cluster autoscaler
- Proper pod scheduling
- Cost calculation (Autopilot)

### 2. Use Appropriate Min/Max Replicas

```yaml
spec:
  minReplicas: 3  # Enough for availability
  maxReplicas: 50  # Reasonable upper bound
```

**Why:**

- Prevent scaling to zero (availability)
- Prevent runaway scaling (cost control)
- Faster response to traffic spikes

### 3. Configure Scaling Behavior

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # Prevent flapping
  scaleUp:
    stabilizationWindowSeconds: 0  # React quickly
```

**Why:**

- Prevent rapid scaling up/down
- Reduce pod churn
- Smoother scaling behavior

### 4. Use Pod Disruption Budgets

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
```

**Why:**

- Maintain availability during scaling
- Prevent all pods being removed
- Safe cluster operations

### 5. Monitor Scaling Metrics

```bash
# Watch HPA status
kubectl get hpa -w

# Check cluster autoscaler events
kubectl get events -A | grep cluster-autoscaler

# Monitor node count
kubectl get nodes -w
```

**Set up Alerts:**

- HPA at max replicas
- Cluster at max nodes
- High pod pending time
- Scaling failures

### 6. Test Scaling Behavior

```bash
# Generate load for testing
kubectl run -i --tty load-generator \
  --rm --image=busybox \
  --restart=Never \
  -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://web-app; done"

# Watch scaling
kubectl get hpa -w
kubectl get deployment web-app -w
```

### 7. Use VPA for Right-Sizing

```yaml
# Start with VPA in recommendation mode
updatePolicy:
  updateMode: "Off"

# Analyze recommendations, then switch to Auto
updatePolicy:
  updateMode: "Auto"
```

### 8. Combine Scaling Strategies

**Pattern:**

- HPA: Scale pod replicas (horizontal)
- VPA: Right-size pod resources (vertical)
- Cluster Autoscaler: Scale nodes (infrastructure)

**Example:**

```yaml
# HPA on custom metric

- Scale replicas based on request rate

# VPA for resource optimization

- Right-size CPU/memory requests

# Cluster Autoscaler

- Add nodes when pods can't schedule
```

---

## Troubleshooting Scaling

### HPA Not Scaling

**Check metrics availability:**

```bash
kubectl get hpa
kubectl describe hpa <hpa-name>

# Check metrics server
kubectl get apiservice v1beta1.metrics.k8s.io
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
```

**Common Issues:**

- Metrics server not running
- Resource requests not set
- Target already at min/max
- Insufficient metrics data

### Cluster Not Scaling Up

```bash
# Check pending pods
kubectl get pods --field-selector=status.phase=Pending

# Check node pool autoscaling
gcloud container node-pools describe pool-name \
  --cluster=cluster-name

# View cluster autoscaler logs
kubectl logs -n kube-system deployment/cluster-autoscaler
```

**Common Issues:**

- Max nodes reached
- Resource quotas exceeded
- Pod resource requests too large
- Node taints/tolerations mismatch

### Nodes Not Scaling Down

**Check:**

- Pods with safe-to-evict=false
- Pods with local storage
- System pods on nodes
- PodDisruptionBudgets

```bash
# Check node utilization
kubectl top nodes

# Check what's preventing scale-down
kubectl describe node <node-name>
```

---

## Related Resources

- [GKE Deployments](gke-deployments.md) - Managing workloads
- [GKE Pods](gke-pods.md) - Pod resource configuration
- [GKE Cluster Types](gke-cluster-types.md) - Standard vs Autopilot
- [kubectl](kubectl.md) - Scaling commands
