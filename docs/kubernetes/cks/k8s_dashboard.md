# Kubernetes Dashboard

The Kubernetes Dashboard is a web-based UI for Kubernetes clusters that allows users to manage and troubleshoot applications. While convenient, it requires proper security measures to prevent unauthorized access.

## 1. Use RBAC (Role-Based Access Control)

RBAC is critical for limiting dashboard access to authorized users only:

```yaml
# Example: Create a restricted dashboard role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: kubernetes-dashboard-restricted
  namespace: kubernetes-dashboard
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```

Link roles to users with RoleBindings:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dashboard-user-binding
  namespace: kubernetes-dashboard
subjects:
- kind: User
  name: dashboard-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: kubernetes-dashboard-restricted
  apiGroup: rbac.authorization.k8s.io
```

## 2. Never Expose Dashboard Publicly

Never expose your dashboard directly to the internet. Use secure access methods:

- **Kubectl Proxy**:
  ```bash
  kubectl proxy
  # Access at http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
  ```

- **Port Forwarding**:
  ```bash
  kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8443:443
  # Access at https://localhost:8443
  ```

## 3. Use Token Authentication

Create service accounts with limited permissions for dashboard access:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-viewer
  namespace: kubernetes-dashboard
```

Bind appropriate roles:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-viewer
subjects:
- kind: ServiceAccount
  name: dashboard-viewer
  namespace: kubernetes-dashboard
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

Generate and retrieve tokens:

```bash
# For Kubernetes v1.24+
kubectl create token dashboard-viewer -n kubernetes-dashboard

# For older versions
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep dashboard-viewer | awk '{print $1}')
```

## 4. Enable HTTPS/TLS

Always use HTTPS with valid certificates:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  template:
    spec:
      containers:
      - name: kubernetes-dashboard
        args:
        - --auto-generate-certificates
        - --namespace=kubernetes-dashboard
```

## 5. Implement Network Policies

Restrict network access to the dashboard:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dashboard-network-policy
  namespace: kubernetes-dashboard
spec:
  podSelector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: dashboard
```

## 6. Use an Authentication Proxy

Consider implementing an authentication proxy like OAuth2 Proxy to add another layer of authentication:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oauth2-proxy
spec:
  template:
    spec:
      containers:
      - name: oauth2-proxy
        image: quay.io/oauth2-proxy/oauth2-proxy:latest
        args:
        - --provider=github
        - --email-domain=*
        - --upstream=https://kubernetes-dashboard.kubernetes-dashboard.svc.cluster.local
        - --cookie-secret=...
        - --client-id=...
        - --client-secret=...
```

## 7. Regular Auditing

Implement and review audit logs:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["pods", "services"]
```

## 8. Keep Dashboard Updated

Always run the latest version of Kubernetes Dashboard to benefit from security fixes:

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo update
helm upgrade kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard
```

## 9. Use Pod Security Standards

Apply Pod Security Standards to the dashboard namespace:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard
  labels:
    pod-security.kubernetes.io/enforce: restricted
```

## 10. Consider a Service Mesh

For advanced scenarios, implement a service mesh like Istio to manage dashboard access with mutual TLS:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: kubernetes-dashboard
spec:
  hosts:
  - "dashboard.example.com"
  gateways:
  - admin-gateway
  http:
  - route:
    - destination:
        host: kubernetes-dashboard.kubernetes-dashboard.svc.cluster.local
        port:
          number: 443
```

## Conclusion

Security is a multi-layered approach. By implementing these best practices, you can significantly reduce the risk of unauthorized access to your Kubernetes Dashboard. Always follow the principle of least privilege and regularly review your security configuration.
