# Admission Controller Configuration

This document provides the necessary configuration files for setting up the ImagePolicyWebhook admission controller in Kubernetes. The ImagePolicyWebhook allows you to enforce policies on which container images can be used in your cluster by validating them against an external webhook service.

## 1. Admission Configuration File

Create a file named `admission-configuration.yaml`:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/image-policy-webhook/kubeconfig.yaml
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false
```

Save this file to `/etc/kubernetes/admission-configuration.yaml` on all control plane nodes.

## 2. Webhook KubeConfig File

Create a file named `kubeconfig.yaml` with your webhook service configuration:

```yaml
apiVersion: v1
kind: Config
# clusters refers to the remote service
clusters:
- name: image-policy-webhook
  cluster:
    certificate-authority: /etc/kubernetes/image-policy-webhook/ca.crt
    server: https://image-policy-webhook.example.com/image-policy
# users refers to the API server's webhook configuration
users:
- name: api-server
  user:
    client-certificate: /etc/kubernetes/image-policy-webhook/apiserver-client.crt
    client-key: /etc/kubernetes/image-policy-webhook/apiserver-client.key
# kubeconfig files require a context, current-context refers to the context to use
current-context: webhook
contexts:
- context:
    cluster: image-policy-webhook
    user: api-server
  name: webhook
```

Save this file to `/etc/kubernetes/image-policy-webhook/kubeconfig.yaml` on all control plane nodes.

## 3. Certificate Setup

You'll need to set up TLS certificates for secure communication between the API server and your webhook service:

1. Create a directory for certificates:
```bash
mkdir -p /etc/kubernetes/image-policy-webhook
```

2. Generate a CA certificate (if you don't have one already):
```bash
openssl genrsa -out /etc/kubernetes/image-policy-webhook/ca.key 2048
openssl req -x509 -new -nodes -key /etc/kubernetes/image-policy-webhook/ca.key \
  -subj "/CN=image-policy-webhook-ca" \
  -days 3650 -out /etc/kubernetes/image-policy-webhook/ca.crt
```

3. Generate client certificates for the API server:
```bash
openssl genrsa -out /etc/kubernetes/image-policy-webhook/apiserver-client.key 2048
openssl req -new -key /etc/kubernetes/image-policy-webhook/apiserver-client.key \
  -subj "/CN=api-server" \
  -out /etc/kubernetes/image-policy-webhook/apiserver-client.csr
openssl x509 -req -in /etc/kubernetes/image-policy-webhook/apiserver-client.csr \
  -CA /etc/kubernetes/image-policy-webhook/ca.crt \
  -CAkey /etc/kubernetes/image-policy-webhook/ca.key \
  -CAcreateserial \
  -days 3650 \
  -out /etc/kubernetes/image-policy-webhook/apiserver-client.crt
```

4. Set appropriate permissions:
```bash
chmod 600 /etc/kubernetes/image-policy-webhook/*.key
```

## 4. API Server Configuration

Enable the ImagePolicyWebhook admission controller in your API server configuration:

For kubeadm installations, edit `/etc/kubernetes/manifests/kube-apiserver.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.23.0
    command:
    - kube-apiserver
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
    - --admission-control-config-file=/etc/kubernetes/admission-configuration.yaml
    # Other API server flags here...
    volumeMounts:
    - mountPath: /etc/kubernetes/admission-configuration.yaml
      name: admission-configuration
      readOnly: true
    - mountPath: /etc/kubernetes/image-policy-webhook
      name: image-policy-webhook
      readOnly: true
  volumes:
  - hostPath:
      path: /etc/kubernetes/admission-configuration.yaml
      type: File
    name: admission-configuration
  - hostPath:
      path: /etc/kubernetes/image-policy-webhook
      type: Directory
    name: image-policy-webhook
```

For non-kubeadm installations, update the API server flags in the appropriate systemd service file:

```
--enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
--admission-control-config-file=/etc/kubernetes/admission-configuration.yaml
```

## 5. Webhook Service Implementation

Your webhook service should implement the admission review API and respond with an admission review response. Here's a sample response format from your webhook service:

```json
{
  "apiVersion": "imagepolicy.k8s.io/v1alpha1",
  "kind": "ImageReview",
  "status": {
    "allowed": true,
    "reason": "Image is from an allowed registry"
  }
}
```

Or to deny an image:

```json
{
  "apiVersion": "imagepolicy.k8s.io/v1alpha1",
  "kind": "ImageReview",
  "status": {
    "allowed": false,
    "reason": "Image from untrusted registry"
  }
}
```

## 6. Understanding Configuration Options

The ImagePolicyWebhook configuration has several important options:

- **kubeConfigFile**: Path to the kubeconfig file for connecting to the webhook
- **allowTTL**: Duration in seconds to cache 'allow' responses
- **denyTTL**: Duration in seconds to cache 'deny' responses
- **retryBackoff**: Duration in milliseconds to wait between retries
- **defaultAllow**: Whether to allow all images if the webhook service is unavailable
  - `true`: All images are allowed when the webhook is unavailable
  - `false`: All images are denied when the webhook is unavailable (more secure)

