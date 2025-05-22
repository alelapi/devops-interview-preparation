# Cilium Exam Exercises by ISO Layer

## Overview
This document contains comprehensive exam exercises for Cilium, organized by ISO network layers. Each exercise tests different aspects of Cilium's functionality and capabilities.

---

## Layer 1 - Physical Layer

### Exercise 1.1: Node Connectivity and Agent Status
**Scenario**: You have a 3-node Kubernetes cluster with Cilium installed. One node appears to have connectivity issues.

**Tasks**:
1. Check the status of Cilium agents on all nodes
2. Verify that Cilium can detect all network interfaces
3. Troubleshoot a node where the Cilium agent is not starting properly
4. Examine Cilium's view of the physical network topology

**Commands to use**:
```bash
cilium status
cilium node list
kubectl get nodes -o wide
cilium connectivity test
```

**Expected outcomes**:
- All Cilium agents should be running and healthy
- Network interfaces should be properly detected
- Node-to-node connectivity should be established

---

## Layer 2 - Data Link Layer

### Exercise 2.1: MAC Address Learning and ARP Table Management
**Scenario**: Investigate how Cilium handles Layer 2 operations in a cluster with multiple subnets.

**Tasks**:
1. Examine Cilium's ARP table entries
2. Verify MAC address learning for endpoints
3. Configure and test VXLAN encapsulation
4. Troubleshoot Layer 2 connectivity between pods on different nodes

**Commands to use**:
```bash
cilium bpf tunnel list
cilium map get cilium_lxc
cilium endpoint list
cilium bpf lb list
```

**Configuration example**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cilium-config
  namespace: kube-system
data:
  tunnel: "vxlan"
  auto-direct-node-routes: "false"
```

**Expected outcomes**:
- Proper VXLAN tunnel establishment
- Correct MAC address resolution
- Successful Layer 2 forwarding between nodes

---

## Layer 3 - Network Layer

### Exercise 3.1: IP Routing and IPAM Configuration
**Scenario**: Configure and troubleshoot Cilium's IP Address Management (IPAM) and routing policies.

**Tasks**:
1. Configure custom IPAM pools for different node groups
2. Implement IP routing policies between different subnets
3. Set up BGP peering with external routers (if using Cilium BGP)
4. Troubleshoot IP routing issues between pods

**IPAM Configuration**:
```yaml
apiVersion: "cilium.io/v2alpha1"
kind: CiliumNode
metadata:
  name: worker-node-1
spec:
  ipam:
    podCIDRs:
    - 10.244.1.0/24
    pools:
      allocated:
      - cidrs:
        - 10.244.1.0/26
        pool: default
```

**BGP Configuration**:
```yaml
apiVersion: "cilium.io/v2alpha1"
kind: CiliumBGPPeeringPolicy
metadata:
  name: bgp-peering-policy
spec:
  nodeSelector:
    matchLabels:
      kubernetes.io/hostname: worker-node-1
  virtualRouters:
  - localASN: 65001
    exportPodCIDR: true
    neighbors:
    - peerAddress: "192.168.1.1"
      peerASN: 65000
```

**Expected outcomes**:
- Proper IP allocation from custom pools
- Successful BGP route advertisement
- Correct routing table entries

---

## Layer 4 - Transport Layer

### Exercise 4.1: Load Balancing and Service Mesh Configuration
**Scenario**: Configure advanced Layer 4 load balancing and troubleshoot service connectivity issues.

**Tasks**:
1. Configure different load balancing algorithms for services
2. Set up session affinity for stateful applications
3. Implement health checking for backend endpoints
4. Troubleshoot connection timeout issues

**Service Configuration**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  annotations:
    service.cilium.io/lb-algorithm: "maglev"
    service.cilium.io/health-check-nodeport: "31234"
spec:
  type: LoadBalancer
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: web-app
```

**Health Check Configuration**:
```yaml
apiVersion: cilium.io/v2
kind: CiliumEndpoint
metadata:
  name: web-endpoint
spec:
  addressing:
    ipv4: 10.244.1.100
  networking:
    node: worker-node-1
  health:
    tcp:
      port: 8080
```

**Commands to use**:
```bash
cilium service list
cilium bpf lb list
cilium connectivity test --test-namespace default
```

**Expected outcomes**:
- Services accessible with proper load balancing
- Health checks functioning correctly
- Session affinity working as configured

---

## Layer 5 - Session Layer

### Exercise 5.1: Connection Tracking and Session Management
**Scenario**: Implement and troubleshoot connection tracking for long-lived sessions.

**Tasks**:
1. Configure connection tracking timeouts for different protocols
2. Monitor active connections and session state
3. Implement session persistence for database connections
4. Troubleshoot session timeout issues

**Connection Tracking Configuration**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cilium-config
  namespace: kube-system
data:
  ct-global-max-entries-tcp: "1000000"
  ct-global-max-entries-other: "500000"
  conntrack-gc-interval: "60s"
```

**Commands to use**:
```bash
cilium bpf ct list global
cilium monitor --type trace
cilium endpoint get <endpoint-id>
```

**Expected outcomes**:
- Proper connection tracking entries
- Sessions maintained for configured duration
- Efficient garbage collection of expired connections

---

## Layer 6 - Presentation Layer

### Exercise 6.1: TLS Termination and Certificate Management
**Scenario**: Configure TLS termination at the ingress level and manage certificates.

**Tasks**:
1. Configure Cilium Ingress Controller with TLS termination
2. Implement automatic certificate rotation
3. Set up mutual TLS (mTLS) between services
4. Monitor certificate expiration and renewal

**Ingress Configuration**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    ingress.cilium.io/tls-passthrough: "false"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: cilium
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

**mTLS Configuration**:
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: mtls-policy
spec:
  endpointSelector:
    matchLabels:
      app: secure-app
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: client-app
    toPorts:
    - ports:
      - port: "443"
        protocol: TCP
      rules:
        tls:
        - secret:
            name: mtls-secret
```

**Expected outcomes**:
- TLS termination working correctly
- Certificates automatically renewed
- mTLS authentication successful

---

## Layer 7 - Application Layer

### Exercise 7.1: HTTP/gRPC Policy Enforcement and Observability
**Scenario**: Implement comprehensive Layer 7 policies and monitoring for HTTP and gRPC traffic.

**Tasks**:
1. Create HTTP-aware network policies with path-based routing
2. Implement rate limiting for specific API endpoints
3. Set up gRPC load balancing with health checking
4. Configure comprehensive Layer 7 observability

**HTTP Network Policy**:
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-access-policy
spec:
  endpointSelector:
    matchLabels:
      app: api-server
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/users"
        - method: "POST"
          path: "/api/v1/users"
          headers:
          - "Content-Type: application/json"
  - fromEndpoints:
    - matchLabels:
        app: admin-panel
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET|POST|PUT|DELETE"
          path: "/api/v1/.*"
```

**Rate Limiting Configuration**:
```yaml
apiVersion: cilium.io/v2
kind: CiliumLocalRedirectPolicy
metadata:
  name: rate-limit-policy
spec:
  redirectFrontend:
    serviceMatcher:
      serviceName: api-service
      namespace: default
  redirectBackend:
    localEndpointSelector:
      matchLabels:
        app: rate-limiter
    toPorts:
    - port: 8081
      name: rate-limit
      protocol: TCP
```

**gRPC Service Configuration**:
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: grpc-policy
spec:
  endpointSelector:
    matchLabels:
      app: grpc-server
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: grpc-client
    toPorts:
    - ports:
      - port: "9090"
        protocol: TCP
      rules:
        grpc:
        - service: "UserService"
          method: "GetUser"
        - service: "UserService"
          method: "CreateUser"
```

**Observability Configuration**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cilium-config
  namespace: kube-system
data:
  enable-l7-proxy: "true"
  hubble-listen-address: ":4244"
  hubble-disable-tls: "false"
  hubble-tls-cert-file: "/var/lib/cilium/tls/hubble-server.crt"
  hubble-tls-key-file: "/var/lib/cilium/tls/hubble-server.key"
```

**Commands to use**:
```bash
hubble observe --protocol http
hubble observe --protocol grpc
cilium monitor --type l7
hubble status
```

**Expected outcomes**:
- HTTP policies enforced correctly
- Rate limiting functional
- gRPC traffic properly load balanced
- Comprehensive Layer 7 observability

---

## Comprehensive Multi-Layer Exercise

### Exercise: End-to-End Microservices Security and Observability
**Scenario**: Deploy a complete microservices application with Cilium providing security and observability across all network layers.

**Architecture**:
- Frontend (React app)
- API Gateway (Envoy)
- User Service (gRPC)
- Order Service (REST API)
- Database (PostgreSQL)
- Message Queue (Redis)

**Tasks**:
1. Implement network policies for each service
2. Configure Layer 7 routing and load balancing
3. Set up mTLS between all services
4. Implement rate limiting and DDoS protection
5. Configure comprehensive monitoring and alerting
6. Perform security testing and validation
