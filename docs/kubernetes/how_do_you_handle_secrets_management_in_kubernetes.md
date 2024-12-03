
# How do you handle secrets management in Kubernetes to ensure security and confidentiality?

## Answer

## Answer

### Managing Kubernetes Resource Allocation
1. **Resource Requests and Limits**:
   - Define resource requests (minimum) and limits (maximum) for CPU and memory in pod specifications.
   - Example:
     ```yaml
     resources:
       requests:
         memory: "64Mi"
         cpu: "250m"
       limits:
         memory: "128Mi"
         cpu: "500m"
     ```

2. **Vertical Pod Autoscaler (VPA)**:
   - Dynamically adjusts pod resources based on usage.

3. **Horizontal Pod Autoscaler (HPA)**:
   - Scales pods based on metrics like CPU or custom metrics.

4. **Namespace Quotas**:
   - Restrict resource consumption at the namespace level using `ResourceQuota`.

### Tools:
- Monitoring: Prometheus, Grafana
- Kubernetes Autoscalers: HPA, VPA
