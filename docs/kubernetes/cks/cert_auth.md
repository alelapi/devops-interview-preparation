# Create User Certificate Authentication

How to create and use client certificates for authenticating users to a Kubernetes cluster. Certificate-based authentication is one of the standard methods for controlling access to your Kubernetes API server.

## Prerequisites

- Access to a running Kubernetes cluster
- `kubectl` command-line tool installed
- `openssl` command-line tool installed
- Admin access to the Kubernetes cluster (to approve certificate signing requests)

## Step 1: Create a Private Key for the User

First, create a private key for the user using OpenSSL:

```bash
openssl genrsa -out jane.key 2048
```

This command generates a 2048-bit RSA key and saves it to a file named `jane.key`.

## Step 2: Create a Certificate Signing Request (CSR)

Create a certificate signing request using the private key:

```bash
openssl req -new -key jane.key -out jane.csr -subj "/CN=jane/O=engineering"
```

In this command:
- `/CN=jane` specifies the username
- `/O=engineering` specifies the group the user belongs to (you can specify multiple groups using multiple `/O=` entries)

## Step 3: Encode the CSR in Base64

Encode the CSR file in base64 format:

```bash
cat jane.csr | base64 | tr -d '\n'
```

Copy the output for use in the next step.

## Step 4: Create a CertificateSigningRequest Object in Kubernetes

Create a file named `jane-csr.yaml` with the following content:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # 24 hours
  usages:
  - client auth
```

Replace `<base64-encoded-csr>` with the output from Step 3.

Apply this configuration to your cluster:

```bash
kubectl apply -f jane-csr.yaml
```

## Step 5: Approve the Certificate Signing Request

As a cluster administrator, approve the CSR:

```bash
kubectl certificate approve jane
```

## Step 6: Retrieve the Signed Certificate

Retrieve the approved certificate:

```bash
kubectl get csr jane -o jsonpath='{.status.certificate}' | base64 --decode > jane.crt
```

This command extracts the signed certificate from the CSR object and decodes it into a file named `jane.crt`.

## Step 7: Create a kubeconfig File for the User

Now you need to create a kubeconfig file that the user can use for authentication:

```bash
kubectl config set-credentials jane --client-certificate=<client-cert> --client-key=<client-key> --embed-certs=true
kubectl config set-context <context-name> --cluster=<cluster-name> --user=jane
kubectl config use-context jane@<cluster-name>
```

## Step 8: Set Appropriate Permissions with RBAC

The user now has a valid certificate, but they need to be granted permissions to perform actions in the cluster. Create a Role or ClusterRole and a corresponding RoleBinding or ClusterRoleBinding:

Example RoleBinding (for namespace-specific access):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jane-engineering-rolebinding
  namespace: development
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

Example ClusterRoleBinding (for cluster-wide access):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jane-engineering-clusterrolebinding
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: engineering
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

Apply the binding using kubectl:

```bash
kubectl apply -f jane-rolebinding.yaml
```
