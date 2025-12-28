# GKE Service Mesh - Istio

## Description

A service mesh is an infrastructure layer that manages service-to-service communication in microservices architectures. On GKE, you can use Anthos Service Mesh (ASM), Google's managed Istio distribution, or open-source Istio. The service mesh provides traffic management, security, and observability without changing application code.

**Concept**: Transparent proxy layer (sidecar) injected into pods to control, secure, and observe service communication.

## Service Mesh Options on GKE

### Anthos Service Mesh (ASM) - Recommended

- **Managed by Google**: Google handles control plane
- **Automatic Updates**: Managed upgrades and patches
- **Google Cloud Integration**: Deep integration with Cloud Ops
- **Support**: Enterprise support from Google
- **Certificate Management**: Automatic with Certificate Authority Service

### Open Source Istio

- **Self-Managed**: You manage control plane
- **Latest Features**: Access to newest Istio features
- **Full Control**: Complete configuration control
- **Community Support**: Istio community forums

### Anthos Service Mesh vs Istio

| Feature | Anthos Service Mesh | Open Source Istio |
|---------|-------------------|-------------------|
| **Management** | Fully managed by Google | Self-managed |
| **Updates** | Automatic (managed) | Manual |
| **Support** | Google Cloud Support | Community |
| **Pricing** | Included with GKE (vCPU hours) | Free (infrastructure costs) |
| **Control Plane** | Managed (in-cluster or managed) | Self-hosted in-cluster |
| **Best For** | Production, enterprise | Advanced users, latest features |

## Key Features

### Traffic Management

- **Intelligent Routing**: Route requests based on headers, paths, weights
- **Load Balancing**: Advanced algorithms (round-robin, least request, random)
- **Traffic Splitting**: A/B testing, canary deployments
- **Circuit Breaking**: Prevent cascading failures
- **Retries and Timeouts**: Automatic retry logic
- **Fault Injection**: Test resilience by injecting failures

### Security

- **Mutual TLS (mTLS)**: Automatic encryption of service-to-service traffic
- **Authentication**: Verify service identity
- **Authorization**: Fine-grained access control policies
- **Certificate Management**: Automatic certificate rotation
- **Security Policies**: Namespace and workload-level policies

### Observability

- **Distributed Tracing**: Request flow across services (Cloud Trace integration)
- **Metrics Collection**: Automatic metrics for all service traffic
- **Access Logs**: Detailed logs of service communication
- **Service Graph**: Visualize service dependencies
- **Dashboards**: Pre-built monitoring dashboards

### Resilience

- **Circuit Breakers**: Prevent overwhelming failing services
- **Outlier Detection**: Remove unhealthy instances from load balancing
- **Request Timeouts**: Prevent indefinite waits
- **Retries**: Automatic retry with exponential backoff
- **Connection Pooling**: Limit connections to downstream services

## Architecture

### Components

**Data Plane:**

- **Envoy Proxy**: Sidecar container injected into each pod
- **Intercepts Traffic**: All inbound/outbound traffic goes through Envoy
- **Policy Enforcement**: Applies routing, security, observability

**Control Plane:**

- **istiod**: Single binary consolidating Pilot, Citadel, Galley
- **Configuration Management**: Distributes configuration to Envoy proxies
- **Certificate Authority**: Issues and rotates certificates for mTLS
- **Service Discovery**: Tracks services and endpoints

```
┌─────────────────────────────────────────┐
│         Control Plane (istiod)          │
│  - Configuration Distribution           │
│  - Certificate Authority (CA)           │
│  - Service Discovery                    │
└─────────────────────────────────────────┘
            │
            │ Configuration & Certs
            ▼
┌─────────────────────────────────────────┐
│              Data Plane                 │
│                                         │
│  ┌─────────────────┐ ┌──────────────┐  │
│  │  Pod A          │ │  Pod B       │  │
│  │  ┌───────────┐  │ │ ┌──────────┐ │  │
│  │  │ Container │  │ │ │Container │ │  │
│  │  └───────────┘  │ │ └──────────┘ │  │
│  │  ┌───────────┐  │ │ ┌──────────┐ │  │
│  │  │Envoy Proxy│◄─┼─┼►│Envoy Prox││  │
│  │  └───────────┘  │ │ └──────────┘ │  │
│  └─────────────────┘ └──────────────┘  │
└─────────────────────────────────────────┘
```

## Installation

### Install Anthos Service Mesh (Recommended)

```bash
# Download asmcli
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.18 > asmcli
chmod +x asmcli

# Install ASM (managed control plane)
./asmcli install \
  --project_id PROJECT_ID \
  --cluster_name CLUSTER_NAME \
  --cluster_location CLUSTER_LOCATION \
  --fleet_id PROJECT_ID \
  --output_dir ./asm-output \
  --enable_all \
  --option managed \
  --ca mesh_ca

# Verify installation
kubectl get pods -n istio-system
```

### Install Open Source Istio

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Install Istio with default profile
istioctl install --set profile=default -y

# Verify installation
kubectl get pods -n istio-system

# Enable sidecar injection for namespace
kubectl label namespace default istio-injection=enabled
```

## Sidecar Injection

### Automatic Sidecar Injection

Enable for entire namespace:

```bash
# Label namespace for auto-injection
kubectl label namespace default istio-injection=enabled

# Verify label
kubectl get namespace default --show-labels

# Deploy application - sidecars injected automatically
kubectl apply -f deployment.yaml
```

### Manual Sidecar Injection

Inject sidecar into specific deployment:

```bash
# Inject sidecar into YAML
istioctl kube-inject -f deployment.yaml | kubectl apply -f -

# Or inject and save to file
istioctl kube-inject -f deployment.yaml > deployment-injected.yaml
kubectl apply -f deployment-injected.yaml
```

### Verify Sidecar Injection

```bash
# Check pod has 2 containers (app + envoy)
kubectl get pods

# Should show 2/2 ready
# Describe pod to see containers
kubectl describe pod <pod-name>
```

## Traffic Management

### VirtualService

Define routing rules for a service:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  
  - reviews
  
  http:
  
  # Route 90% traffic to v1

  - match:
    
    - headers:
        end-user:
          exact: jason
    route:
    
    - destination:
        host: reviews
        subset: v2
  
  # Default route for other users

  - route:
    
    - destination:
        host: reviews
        subset: v1
      weight: 90
    
    - destination:
        host: reviews
        subset: v2
      weight: 10
```

### DestinationRule

Define subsets and traffic policies:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  
  trafficPolicy:
    loadBalancer:
      simple: LEAST_REQUEST
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  
  subsets:
  
  - name: v1
    labels:
      version: v1
  
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
```

### Gateway

Expose services outside the mesh:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    
    - "bookinfo.example.com"
  
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: bookinfo-cert
    hosts:
    
    - "bookinfo.example.com"
```

### Traffic Splitting (Canary Deployment)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: canary-rollout
spec:
  hosts:
  
  - myapp
  
  http:
  
  - route:
    
    - destination:
        host: myapp
        subset: stable
      weight: 90
    
    - destination:
        host: myapp
        subset: canary
      weight: 10
```

## Security

### Mutual TLS (mTLS)

Enable mTLS for entire mesh:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT  # STRICT, PERMISSIVE, or DISABLE
```

Enable mTLS for specific namespace:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

### Authorization Policies

Deny all traffic by default:

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  {}  # Empty spec denies all
```

Allow specific traffic:

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend
  namespace: default
spec:
  selector:
    matchLabels:
      app: backend
  
  action: ALLOW
  
  rules:
  
  - from:
    
    - source:
        principals:
        
        - "cluster.local/ns/default/sa/frontend"
    to:
    
    - operation:
        methods:
        
        - "GET"
        - "POST"
        paths:
        
        - "/api/*"
```

HTTP-based authorization:

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: httpbin-viewer
spec:
  selector:
    matchLabels:
      app: httpbin
  
  action: ALLOW
  
  rules:
  
  - from:
    
    - source:
        requestPrincipals:
        
        - "*"
    
    when:
    
    - key: request.auth.claims[groups]
      values:
      
      - "viewer"
```

### Request Authentication

Validate JWT tokens:

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: default
spec:
  selector:
    matchLabels:
      app: httpbin
  
  jwtRules:
  
  - issuer: "https://accounts.google.com"
    jwksUri: "https://www.googleapis.com/oauth2/v3/certs"
    audiences:
    
    - "my-app"
```

## Observability

### Distributed Tracing

View traces in Cloud Trace:

```bash
# Enable Cloud Trace integration (ASM)
# Already integrated by default with ASM

# View traces in Cloud Console
# Navigation: Operations → Trace → Trace List
```

### Metrics

View metrics in Cloud Monitoring:

```bash
# ASM metrics automatically exported to Cloud Monitoring
# Navigate to Cloud Console → Monitoring → Metrics Explorer

# Query Istio metrics
# Metric type: istio.io/service/server/request_count
```

### Service Dashboard

Access Istio dashboards:

```bash
# Install Kiali (service mesh dashboard)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml

# Port forward to access Kiali
kubectl port-forward -n istio-system svc/kiali 20001:20001

# Open browser to http://localhost:20001
```

### Access Logs

Enable access logs:

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-default
  namespace: istio-system
spec:
  accessLogging:
  
  - providers:
    
    - name: envoy
```

View logs:

```bash
# View Envoy proxy logs
kubectl logs <pod-name> -c istio-proxy

# Stream access logs
kubectl logs -f <pod-name> -c istio-proxy
```

## Advanced Features

### Circuit Breaking

Prevent overwhelming failing services:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: circuit-breaker
spec:
  host: myservice
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 1m
      maxEjectionPercent: 50
```

### Fault Injection

Test resilience by injecting failures:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: fault-injection
spec:
  hosts:
  
  - ratings
  
  http:
  
  - fault:
      delay:
        percentage:
          value: 10
        fixedDelay: 5s
      abort:
        percentage:
          value: 10
        httpStatus: 500
    route:
    
    - destination:
        host: ratings
```

### Retry Logic

Automatic retries with backoff:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: retry-policy
spec:
  hosts:
  
  - myservice
  
  http:
  
  - route:
    
    - destination:
        host: myservice
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure,refused-stream
```

### Request Timeouts

Set global timeout:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: timeout-policy
spec:
  hosts:
  
  - myservice
  
  http:
  
  - route:
    
    - destination:
        host: myservice
    timeout: 10s
```

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Services per mesh** | 1000s | Based on cluster capacity |
| **Virtual Services** | 1000s | No hard limit |
| **Gateways** | Depends on ingress capacity | |
| **Sidecar overhead** | ~50-100MB RAM | Per pod |
| **CPU overhead** | ~0.1-0.5 vCPU | Varies with traffic |
| **Latency overhead** | 1-10ms | P99 typically <5ms |

## When to Use

### ✅ Use Service Mesh When:

1. **Microservices Architecture**

   - Multiple services communicating
   - Need traffic management between services
   - Service-to-service security required

2. **Advanced Traffic Management**

   - Canary deployments
   - A/B testing
   - Traffic splitting and routing
   - Blue-green deployments

3. **Security Requirements**

   - Need mTLS for service communication
   - Fine-grained authorization policies
   - Compliance requirements for encrypted traffic

4. **Observability Needed**

   - Distributed tracing across services
   - Service-level metrics
   - Service dependency visualization

5. **Resilience Patterns**

   - Circuit breaking
   - Retry logic
   - Timeout management
   - Fault injection for testing

### ❌ Don't Use Service Mesh When:

1. **Simple Architecture**

   - Monolithic application
   - Few services (< 5)
   - Complexity not justified

2. **Performance Critical**

   - Cannot tolerate 1-10ms latency overhead
   - Extremely high throughput requirements
   - Resource-constrained environment

3. **Limited Team Expertise**

   - Small team without service mesh experience
   - Cannot invest in learning curve
   - Lack operational expertise

4. **Resource Constraints**

   - Very small cluster
   - Cannot afford sidecar overhead (memory/CPU)
   - Budget constraints

## Best Practices

### 1. Start with Permissive mTLS

```yaml
# Begin with PERMISSIVE mode
spec:
  mtls:
    mode: PERMISSIVE

# Migrate to STRICT after all services have sidecars
spec:
  mtls:
    mode: STRICT
```

### 2. Use Namespace-Level Policies

```yaml
# Apply policies at namespace level
metadata:
  namespace: production
```

### 3. Monitor Sidecar Resource Usage

```bash
# Check sidecar memory/CPU
kubectl top pods

# Set resource limits for sidecars
# In ASM, configured via MeshConfig
```

### 4. Enable Access Logging Selectively

```yaml
# Enable only for specific services
spec:
  selector:
    matchLabels:
      app: critical-service
```

### 5. Test with Fault Injection

```yaml
# Test resilience before production
fault:
  delay:
    percentage:
      value: 10
    fixedDelay: 3s
```

### 6. Use Circuit Breakers

```yaml
# Prevent cascading failures
outlierDetection:
  consecutiveErrors: 5
  interval: 30s
  baseEjectionTime: 30s
```

### 7. Implement Gradual Rollouts

```yaml
# Start with small percentage

- destination:
    host: myapp
    subset: v2
  weight: 10  # 10% canary traffic
```

## Troubleshooting

### Sidecar Not Injecting

```bash
# Check namespace label
kubectl get namespace default --show-labels

# Verify webhook exists
kubectl get mutatingwebhookconfigurations

# Check injection status
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].name}'
```

### mTLS Connection Issues

```bash
# Check PeerAuthentication
kubectl get peerauthentication -A

# Verify certificates
istioctl proxy-config secret <pod-name>

# Check mutual TLS status
istioctl authn tls-check <pod-name>
```

### Traffic Not Routing Correctly

```bash
# Validate configuration
istioctl analyze

# Check VirtualService
kubectl get virtualservice <name> -o yaml

# View Envoy configuration
istioctl proxy-config routes <pod-name>
```

### High Latency

```bash
# Check Envoy stats
kubectl exec <pod-name> -c istio-proxy -- curl localhost:15000/stats

# View traces
# Navigate to Cloud Trace in Console

# Analyze metrics
kubectl top pods
```

## Related Resources

- [GKE Overview](gke-overview.md) - GKE fundamentals
- [GKE Deployments](gke-deployments.md) - Application deployment
- [GKE Scaling](gke-scaling.md) - Scaling considerations
- [kubectl](kubectl.md) - Command-line operations
