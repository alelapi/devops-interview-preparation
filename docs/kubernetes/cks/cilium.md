# Cilium

Cilium is an open-source software for providing, securing, and observing network connectivity between container workloads. It's designed specifically for Kubernetes and leverages a powerful Linux kernel technology called eBPF (extended Berkeley Packet Filter) to deliver its features with high performance and minimal overhead.

Cilium operates at Layer 3/4 (IP/TCP) as well as Layer 7 (HTTP, gRPC, Kafka), allowing it to secure modern API-driven microservices while also supporting traditional network security approaches.

## Core Concepts

### eBPF

eBPF is a revolutionary technology that allows programs to run in the Linux kernel without changing kernel source code or loading kernel modules. Cilium leverages eBPF to:

- Intercept and control network traffic at the kernel level
- Apply security policies with minimal performance overhead
- Provide deep visibility into network and application behavior
- Dynamically program network flows without service disruption

Unlike traditional networking solutions that use iptables, eBPF programs are JIT-compiled and highly efficient, making Cilium particularly well-suited for large-scale environments.

### Identity-Based Security

Cilium introduces the concept of security identities, which are derived from Kubernetes labels. This enables:

- Security policies based on service identities rather than IP addresses
- Persistent security even when pods are rescheduled to different nodes
- Simplified policy management that aligns with Kubernetes native concepts

### Network Policy

Cilium extends the Kubernetes NetworkPolicy API with CiliumNetworkPolicy, which adds:

- Layer 7 (application protocol) visibility and filtering
- Support for cluster-wide policies with ClusterwideCiliumNetworkPolicy
- FQDN/DNS based filtering for external services
- Deny policies and more advanced rule semantics

## Key Features

### Networking

- **CNI Plugin**: Implements the Container Network Interface for Kubernetes
- **Multi-cluster Routing**: Connect multiple Kubernetes clusters
- **IPv4/IPv6 Support**: Dual-stack networking capabilities
- **VXLAN, Geneve, or Direct Routing**: Flexible overlay or native routing options
- **Bandwidth Management**: QoS and rate limiting capabilities

### Security

- **Network Policies**: Layer 3-4 (IP/ports) filtering
- **Application Policies**: Layer 7 filtering for HTTP, gRPC, Kafka, etc.
- **Transparent Encryption**: IPsec or WireGuard for node-to-node traffic
- **Service Authorization**: Control access to services
- **DNS Security**: Filter outbound connections based on DNS names

### Observability

- **Hubble**: Dedicated observability platform built into Cilium
- **Flow Logs**: Detailed network flow information
- **Service Maps**: Visual representation of service dependencies
- **Policy Verdicts**: See which policies accept or deny connections
- **Metrics**: Prometheus integration for monitoring

### Load Balancing

- **Kubernetes Services**: Implementation of kube-proxy functionality
- **Direct Server Return (DSR)**: Optimized load balancing path
- **Session Affinity**: Consistent hashing for stable connections
- **Global Services**: Services spanning multiple clusters
- **XDP Acceleration**: High-performance packet processing

## Basic Configuration

### Network Policies

Cilium extends Kubernetes NetworkPolicy with additional features. Here's an example of a basic CiliumNetworkPolicy:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "allow-web-from-frontend"
spec:
  endpointSelector:
    matchLabels:
      app: web
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/.*"
```

This policy allows frontend pods to make HTTP GET requests to web pods on port 80, specifically for paths matching `/api/v1/.*`.

### Service Mesh

Cilium can be used as a service mesh alternative by enabling the following features:

```bash
helm upgrade cilium cilium/cilium --namespace kube-system \
  --reuse-values \
  --set hubble.enabled=true \
  --set hubble.metrics.enabled="{dns,drop,tcp,flow,port-distribution,icmp,http}"
```

This provides many service mesh features without a sidecar proxy, including:
- Service-to-service communication security
- Traffic monitoring and metrics
- L7 visibility

### Cluster Mesh

To connect multiple Kubernetes clusters with Cilium:

1. Enable Cluster Mesh on each cluster:

```bash
cilium clustermesh enable
```

2. Connect the clusters:

```bash
cilium clustermesh connect --destination-context=cluster2
```

## Hubble: Observability Platform

### Installing Hubble

Hubble is Cilium's observability platform:

```bash
# Enable Hubble with UI
cilium hubble enable --ui

# Verify Hubble is properly installed
cilium hubble status
```

### Using Hubble UI

Access the Hubble UI by port-forwarding:

```bash
cilium hubble ui
```

This will open a browser window with the Hubble UI, displaying:
- Service dependency map
- Real-time network flows
- HTTP, DNS, and TCP metrics
- Detailed flow information

### Hubble CLI

Hubble CLI provides command-line access to observability data:

```bash
# Install Hubble CLI
export HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
curl -L --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-amd64.tar.gz
sudo tar xzvfC hubble-linux-amd64.tar.gz /usr/local/bin

# Set up Hubble CLI
cilium hubble port-forward&

# Observe flows in real-time
hubble observe --follow

# Filter flows for specific pods
hubble observe --pod frontend

# Show HTTP metrics
hubble observe --protocol http
```

## Advanced Features

### Transparent Encryption

Enable transparent encryption between nodes using IPsec or WireGuard:

```bash
# Using Helm with IPsec
helm upgrade cilium cilium/cilium --namespace kube-system \
  --reuse-values \
  --set encryption.enabled=true \
  --set encryption.type=ipsec

# Using Cilium CLI with WireGuard
cilium config set encryption.enabled=true
cilium config set encryption.type=wireguard
```

### Host Firewall

Protect the Kubernetes nodes themselves with Cilium's host firewall:

```bash
helm upgrade cilium cilium/cilium --namespace kube-system \
  --reuse-values \
  --set hostFirewall.enabled=true
```

Example policy to protect hosts:

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumClusterwideNetworkPolicy
metadata:
  name: "host-policy"
spec:
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/control-plane: ""
  ingress:
  - fromEntities:
    - cluster
    toPorts:
    - ports:
      - port: "6443"
        protocol: TCP
```

### Kubernetes Without kube-proxy

Cilium can replace kube-proxy for better performance:

```bash
helm upgrade cilium cilium/cilium --namespace kube-system \
  --reuse-values \
  --set kubeProxyReplacement=strict \
  --set k8sServiceHost=<API_SERVER_IP> \
  --set k8sServicePort=<API_SERVER_PORT>
```

Benefits include:
- eBPF-based service implementation
- Lower latency
- Better scalability
- Support for DSR (Direct Server Return)

### Multi-Cluster Connectivity

Create global services spanning multiple clusters:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: global-service
  annotations:
    service.cilium.io/global: "true"
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
```

## Performance Considerations

Cilium is designed for high performance:

- **eBPF vs iptables**: Significant performance improvements, especially at scale
- **Kernel Bypass**: XDP acceleration for load balancing
- **Direct Routing**: Improved performance compared to overlay networks
- **Hubble Overhead**: Minimal when sampling is configured appropriately

Optimizing Cilium for performance:

```bash
# Direct routing for better performance
helm upgrade cilium cilium/cilium --namespace kube-system \
  --reuse-values \
  --set tunnel=disabled \
  --set autoDirectNodeRoutes=true

# Enable XDP acceleration for service load balancing
helm upgrade cilium cilium/cilium --namespace kube-system \
  --reuse-values \
  --set loadBalancer.acceleration=native
```

## Troubleshooting

Common troubleshooting commands:

```bash
# Check Cilium status
cilium status

# Validate Cilium installation
cilium connectivity test

# Troubleshoot a specific endpoint
cilium endpoint get <endpoint-id>

# Check Cilium agent logs
kubectl -n kube-system logs -l k8s-app=cilium

# Restart Cilium on a node
kubectl -n kube-system delete pod -l k8s-app=cilium -l kubernetes.io/hostname=<node-name>

# Verify network policies
cilium policy get
```


