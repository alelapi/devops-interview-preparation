# PodDisruptionBudget (PDB) in Kubernetes

A **PodDisruptionBudget (PDB)** is a Kubernetes resource that helps ensure a certain number or percentage of Pods remain available during **voluntary disruptions**. These disruptions can include node maintenance, cluster scaling, or rolling updates.

---

## **Purpose of PodDisruptionBudget**

- To protect application availability during planned events.
- To enforce a minimum number of Pods running or restrict the maximum number of Pods disrupted simultaneously.
- To balance the needs of system administrators and application reliability.

---

## **Key Features**

1. **Voluntary Disruptions**:

   - PDB applies only to voluntary disruptions, such as:
     - Node draining for maintenance.
     - Rolling updates.
     - Scaling events.

2. **Minimum Availability**:

   - Ensures that a certain number of Pods remain available during disruptions.

3. **Maximum Disruption**:

   - Restricts the maximum number of Pods that can be disrupted simultaneously.

4. **Integration with Controllers**:
   - Works with Deployments, StatefulSets, ReplicaSets, and other controllers.

---

## **How PodDisruptionBudget Works**

- **`minAvailable`**:

  - Specifies the minimum number of Pods that must remain available during disruptions.

- **`maxUnavailable`**:

  - Specifies the maximum number of Pods that can be disrupted simultaneously.

- **Scope**:
  - PDB is applied to a group of Pods matching the specified label selector.

---

## **Example PodDisruptionBudget**

### **1. Minimum Available Pods**

This PDB ensures at least 2 Pods are always running during voluntary disruptions.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
  namespace: my-namespace
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

### **2. Maximum Unavailable Pods**

This PDB ensures that no more than 1 Pod can be disrupted at any time.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
  namespace: my-namespace
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: my-app
```

---

## **Use Cases**

1. **High Availability**:

   - Ensures critical applications remain operational during cluster maintenance.

2. **Rolling Updates**:

   - Controls the pace of Pod evictions to prevent service downtime.

3. **Stateful Applications**:
   - Protects databases or StatefulSets that require a specific number of Pods for consistency.

---

## **Best Practices**

1. **Plan for Downtime**:

   - Use `minAvailable` or `maxUnavailable` based on the application's availability requirements.

2. **Label Pods Consistently**:

   - Ensure Pods have appropriate labels to match the PDB's selector.

3. **Combine with Monitoring**:

   - Use monitoring tools to track PDB effectiveness during disruptions.

4. **Test Scenarios**:
   - Simulate node drains and rolling updates to verify PDB behavior.

---

## **Limitations**

1. **Voluntary Disruptions Only**:

   - PDB does not apply to involuntary disruptions, such as crashes or node failures.

2. **No Guarantee of Scheduling**:
   - PDB ensures Pods are not evicted below the threshold but does not guarantee new Pods can be scheduled.

---

## **Conclusion**

PodDisruptionBudget is a vital tool in Kubernetes for ensuring application availability during planned events like maintenance or updates. By setting appropriate thresholds with `minAvailable` or `maxUnavailable`, you can balance operational flexibility with application reliability.
