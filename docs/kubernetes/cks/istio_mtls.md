# mTLS Pod-to-Pod Communication with Istio

## Introduction

Mutual TLS (mTLS) is one of the most powerful security features offered by Istio service mesh. Unlike regular TLS where only the server authenticates itself to the client, in mutual TLS both parties verify each other's identity. 

## Understanding Istio mTLS

### How mTLS Works in Istio

Istio implements mTLS through its sidecar proxies (Envoy). When a service with an Istio sidecar communicates with another service in the mesh, the following happens:

1. The client-side sidecar proxy initiates a connection to the server
2. The client-side proxy performs a TLS handshake with the server-side proxy, exchanging certificates
3. The client-side proxy verifies the server's identity
4. The client and server establish a mutual TLS connection
5. The client proxy forwards the request to the server through the encrypted channel
6. The server-side proxy forwards the request to the application container

This entire process happens transparently to your application.

### mTLS Modes in Istio

Istio offers three modes for mTLS:

1. **PERMISSIVE**: Accepts both plaintext and mTLS traffic (default)
2. **STRICT**: Only accepts mTLS traffic
3. **DISABLED**: Only accepts plaintext traffic

## Implementing mTLS Between Pods

### Enable Strict mTLS

Enable strict mTLS for the namespaces using PeerAuthentication:

```yaml
# save this as peer-authentication.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo
spec:
  mtls:
    mode: STRICT
---
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: bar
spec:
  mtls:
    mode: STRICT
```

Apply the configuration:

```bash
kubectl apply -f peer-authentication.yaml
```

### Configure Destination Rules

Destination rules define how traffic is routed to a service after virtual service routing rules are evaluated. 
When you think about Istio's traffic management, it's important to understand the sequence:

- First, traffic is routed (often using VirtualService)
- Then, DestinationRule policies are applied to that routed traffic

Create destination rules to enforce mTLS for outbound traffic:

```yaml
# save this as destination-rule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: httpbin-foo
  namespace: foo
spec:
  host: "httpbin.foo.svc.cluster.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: httpbin-bar
  namespace: bar
spec:
  host: "httpbin.bar.svc.cluster.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```

Apply the configuration:

```bash
kubectl apply -f destination-rule.yaml
```

## Handling Non-Mesh Services

For services outside the mesh or services that don't support mTLS, you can configure exceptions to the mTLS policy.

### Workload-Specific mTLS Policies

Create workload-specific PeerAuthentication policies to override namespace or mesh-wide policies:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: workload-policy
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  mtls:
    mode: PERMISSIVE
```

### Configure Destination Rules for External Services

For external services, set up a destination rule to disable mTLS:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: external-service
spec:
  host: external-service.example.com
  trafficPolicy:
    tls:
      mode: DISABLE  # or SIMPLE for regular TLS
```

## Istio Sidecar Injection

Istio sidecar injection is the process of automatically adding an Envoy proxy container (the "sidecar") to Kubernetes pods. This sidecar proxy is what enables Istio's service mesh functionality, as it intercepts and manages all network traffic going in and out of your application pods.

When you deploy applications in a Kubernetes cluster with Istio, each pod gets its own sidecar proxy that handles all the networking concerns like traffic routing, load balancing, circuit breaking, and implementing security policies.

## How Sidecar Injection Works

Istio uses a Kubernetes feature called "mutating webhook admission controller" to modify pod specifications at creation time. When the webhook is enabled and a namespace is properly labeled, any new pods created in that namespace will automatically have the Istio sidecar proxy container added to them.

The injection can happen in two ways:

1. **Automatic injection**: Configured at the namespace level
2. **Manual injection**: Applied directly to deployments using the `istioctl` command

## How to Enable Sidecar Injection

### Method 1: Automatic Injection (Namespace-Level)

This is the most common approach:

```bash
# Label the namespace to enable automatic injection
kubectl label namespace <your-namespace> istio-injection=enabled
```

Once you apply this label, all new workloads deployed in the namespace will automatically have the Istio sidecar injected. Note that existing workloads won't be affected—you'll need to redeploy them to get the sidecar.

You can verify the label is applied with:

```bash
kubectl get namespace <your-namespace> --show-labels
```

### Method 2: Manual Injection (Deployment-Level)

If you prefer to inject sidecars on specific deployments:

```bash
# Manually inject the sidecar into a deployment
istioctl kube-inject -f your-deployment.yaml | kubectl apply -f -
```

This approach modifies the deployment YAML directly, adding the sidecar configuration before applying it to the cluster.

## How to Verify Injection Worked

You can verify that a pod has the sidecar injected by checking the READY column:

```bash
kubectl get pods
```

A pod with the sidecar injected will show `2/2` in the READY column, indicating that there are two containers running (your application and the Istio proxy).

You can also describe the pod to see the `istio-proxy` container:

```bash
kubectl describe pod <pod-name>
```

Or check for the presence of the sidecar container directly:

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'
```

## Disabling Sidecar Injection

If you need to exclude a specific workload from injection, you can add an annotation to that workload:

```yaml
metadata:
  annotations:
    sidecar.istio.io/inject: "false"
```

Alternatively, the newer approach uses a label instead of an annotation:

```yaml
metadata:
  labels:
    sidecar.istio.io/inject: "false"
```

This is particularly useful for jobs, CronJobs, or any workload that shouldn't be part of the service mesh.