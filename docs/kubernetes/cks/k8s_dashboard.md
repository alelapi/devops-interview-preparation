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
