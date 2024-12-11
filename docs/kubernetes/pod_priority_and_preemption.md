# Implementing Pod Priority and Preemption in Kubernetes

**Pod Priority and Preemption** is a feature in Kubernetes that allows you to assign different levels of importance to Pods. Higher-priority Pods can preempt (evict) lower-priority Pods to make room for critical workloads when cluster resources are scarce.

---

## Steps to Implement Pod Priority and Preemption

### Step 1: Enable Priority and Preemption

Pod Priority and Preemption are enabled by default in Kubernetes (v1.14+). Ensure it is not disabled in your cluster by checking the `--enable-admission-plugins` flag on the API server. The `Priority` admission plugin must be enabled.

```bash
kubectl get podsecuritypolicy
# Verify the admission plugins include "Priority".
```

---

### Step 2: Create PriorityClasses

PriorityClasses define the priority level for Pods. A higher `value` indicates higher priority.

#### Example YAML for PriorityClasses:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "This priority is for critical Pods."
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 500
globalDefault: false
description: "This priority is for less important Pods."
```

Apply the PriorityClasses:

```bash
kubectl apply -f priorityclasses.yaml
```

---

### Step 3: Assign Priority to Pods

Use the `priorityClassName` field in the Pod specification to assign a priority to your Pods.

#### Example: High-Priority Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority-pod
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx
  priorityClassName: high-priority
```

#### Example: Low-Priority Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-priority-pod
  namespace: default
spec:
  containers:
    - name: nginx
      image: nginx
  priorityClassName: low-priority
```

Apply the Pod definitions:

```bash
kubectl apply -f high-priority-pod.yaml
kubectl apply -f low-priority-pod.yaml
```

---

### Step 4: Test Preemption

1. Simulate a resource-scarce scenario by scheduling multiple low-priority Pods to consume available resources.
2. Schedule a high-priority Pod. Kubernetes will preempt (evict) the low-priority Pods if necessary to make room for the high-priority Pod.

#### Verify Preemption:

Check the status of the evicted Pods:

```bash
kubectl get pods -o wide
```

Evicted Pods will show a status of `Evicted` or `Pending`.

---

## Additional Considerations

1. **Preemption Delay**:

   - Preemption is not immediate. Kubernetes waits for evicted Pods to terminate before scheduling high-priority Pods.

2. **Avoid Overuse of High Priority**:

   - Overusing high-priority Pods can lead to instability by preempting essential workloads.

3. **Graceful Eviction**:

   - Kubernetes respects the `terminationGracePeriodSeconds` of evicted Pods to allow graceful termination.

4. **Default Priority**:
   - You can define a `globalDefault: true` PriorityClass, which will be used for Pods without an explicit `priorityClassName`.

---

## Benefits

- Ensures critical workloads are prioritized during resource contention.
- Helps maintain cluster reliability by protecting important services.

## Use Cases

- Assigning higher priority to system Pods (e.g., DNS, monitoring).
- Ensuring critical workloads are scheduled even in overloaded clusters.
- Preempting non-essential workloads for disaster recovery operations.

By carefully designing your PriorityClasses and assigning them appropriately, you can efficiently manage resource allocation in your Kubernetes cluster.

# `system-cluster-critical` PriorityClass

`system-cluster-critical` is a predefined **PriorityClass** in Kubernetes. It is used for system-critical Pods that are essential for the overall functionality of the cluster. This PriorityClass ensures that critical system Pods have the highest priority and can preempt less critical workloads to maintain cluster health.

---

## Key Features

1. **High Priority**:

   - `system-cluster-critical` is one of the highest priority levels in Kubernetes, ensuring that critical system Pods can always run, even under resource contention.

2. **Reserved for System Pods**:

   - Intended for Pods required for cluster management, such as DNS, network plugins, or monitoring systems.

3. **Preemption**:
   - Pods with this PriorityClass can preempt lower-priority Pods to free up resources.

---
