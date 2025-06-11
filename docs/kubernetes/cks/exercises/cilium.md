# Cilium Network Policies Certification Guide

## Layer 3 Network Policies

Layer 3 policies control traffic based on IP addresses and CIDR blocks.

### Basic L3 Policy - Allow Specific IP Ranges

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l3-allow-specific-ips
  namespace: production
spec:
  endpointSelector:
    matchLabels:
      app: web-server
  ingress:
  - fromCIDR:
    - "10.0.1.0/24"    # Allow from specific subnet
    - "192.168.1.100/32" # Allow from specific IP
  egress:
  - toCIDR:
    - "10.0.2.0/24"    # Allow to database subnet
  - toFQDNs:
    - matchName: "api.external-service.com"
```

### L3 Policy - Block External Traffic

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l3-block-external
  namespace: secure-zone
spec:
  endpointSelector:
    matchLabels:
      security: high
  ingress:
  - fromCIDR:
    - "10.0.0.0/8"     # Only allow internal traffic
    - "172.16.0.0/12"
    - "192.168.0.0/16"
  egress:
  - toCIDR:
    - "10.0.0.0/8"     # Only allow internal egress
    - "172.16.0.0/12"
    - "192.168.0.0/16"
```

### L3 Policy with Node Selection

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l3-node-based
spec:
  endpointSelector:
    matchLabels:
      app: monitoring
  ingress:
  - fromEntities:
    - "host"           # Allow from node
  - fromNodes:
    - matchLabels:
        node-role: "worker"
```

---

## Layer 4 Network Policies

Layer 4 policies control traffic based on ports and protocols.

### Basic L4 Policy - Port-specific Access

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l4-web-server-ports
  namespace: web-app
spec:
  endpointSelector:
    matchLabels:
      app: nginx
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      - port: "443"
        protocol: TCP
  - fromEndpoints:
    - matchLabels:
        app: monitoring
    toPorts:
    - ports:
      - port: "9090"    # Metrics port
        protocol: TCP
```

### L4 Policy - Database Access Control

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l4-database-access
  namespace: database
spec:
  endpointSelector:
    matchLabels:
      app: mysql
  ingress:
  - fromEndpoints:
    - matchLabels:
        tier: backend
    toPorts:
    - ports:
      - port: "3306"
        protocol: TCP
  # Deny all other ingress traffic (implicit)
  egress:
  - {} # Allow all egress for database operations
```

### L4 Policy - Multi-Protocol Support

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l4-multi-protocol
spec:
  endpointSelector:
    matchLabels:
      app: dns-server
  ingress:
  - fromEndpoints:
    - matchLabels:
        role: client
    toPorts:
    - ports:
      - port: "53"
        protocol: TCP
      - port: "53"
        protocol: UDP
    - ports:
      - port: "853"      # DNS over TLS
        protocol: TCP
```

---

## ICMP Policies

ICMP policies control ping, traceroute, and other ICMP traffic.

### Allow ICMP for Monitoring

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: icmp-monitoring
  namespace: infrastructure
spec:
  endpointSelector:
    matchLabels:
      role: server
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: monitoring
    icmps:
    - fields:
      - type: 8          # Echo Request (ping)
        code: 0
  - fromEndpoints:
    - matchLabels:
        app: network-tools
    icmps:
    - fields:
      - type: 8          # Echo Request
      - type: 11         # Time Exceeded (traceroute)
```

### Block ICMP from External Sources

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: icmp-block-external
spec:
  endpointSelector:
    matchLabels:
      exposure: internal
  ingress:
  - fromCIDR:
    - "10.0.0.0/8"
    icmps:
    - fields:
      - type: 8          # Allow internal ping
  # ICMP from external sources blocked (no rule = deny)
```

### ICMP Troubleshooting Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: icmp-troubleshooting
spec:
  endpointSelector:
    matchLabels:
      debug: enabled
  ingress:
  - fromEndpoints:
    - matchLabels:
        role: admin
    icmps:
    - fields:
      - type: 8          # Echo Request
      - type: 0          # Echo Reply
      - type: 3          # Destination Unreachable
      - type: 11         # Time Exceeded
      - type: 12         # Parameter Problem
```

---

## Layer 7 (Application Layer) Policies

Layer 7 policies provide HTTP/gRPC/Kafka protocol-aware filtering.

### HTTP-based L7 Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-http-api-access
  namespace: api-gateway
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
        - method: "GET"
          path: "/api/v1/health"
  - fromEndpoints:
    - matchLabels:
        app: admin-panel
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/.*"    # Regex for any v1 endpoint
        - method: "POST"
          path: "/api/v1/.*"
        - method: "PUT"
          path: "/api/v1/.*"
        - method: "DELETE"
          path: "/api/v1/.*"
```

### L7 Policy with Headers

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-http-headers
spec:
  endpointSelector:
    matchLabels:
      app: auth-service
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: web-app
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "POST"
          path: "/auth/login"
          headers:
          - "Content-Type: application/json"
        - method: "GET"
          path: "/auth/verify"
          headers:
          - "Authorization: Bearer .*"
```

### L7 gRPC Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-grpc-service
spec:
  endpointSelector:
    matchLabels:
      app: grpc-service
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
        - service: "user.UserService"
          method: "GetUser"
        - service: "user.UserService"
          method: "ListUsers"
```

### L7 DNS Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-dns-filtering
spec:
  endpointSelector:
    matchLabels:
      app: web-crawler
  egress:
  - toEndpoints:
    - matchLabels:
        k8s:io.kubernetes.pod.namespace: kube-system
        k8s:app: kube-dns
    toPorts:
    - ports:
      - port: "53"
        protocol: UDP
      rules:
        dns:
        - matchPattern: "*.example.com"
        - matchPattern: "api.allowed-service.org"
        - matchName: "specific-host.com"
```

---

## Deny Policies

Cilium supports explicit deny policies for enhanced security.

### Explicit Deny Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: deny-suspicious-traffic
spec:
  endpointSelector:
    matchLabels:
      security: strict
  ingressDeny:
  - fromCIDR:
    - "192.168.100.0/24"  # Block specific subnet
  - fromEndpoints:
    - matchLabels:
        reputation: suspicious
  egressDeny:
  - toPorts:
    - ports:
      - port: "22"        # Block SSH
        protocol: TCP
      - port: "3389"      # Block RDP
        protocol: TCP
```

### Deny with L7 Rules

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: deny-admin-endpoints
spec:
  endpointSelector:
    matchLabels:
      app: web-server
  ingressDeny:
  - fromEndpoints:
    - matchLabels:
        role: user
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/admin/.*"
        - method: "POST"
          path: "/admin/.*"
```

### Time-based Deny (Advanced)

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: deny-after-hours
spec:
  endpointSelector:
    matchLabels:
      access: business-hours
  ingressDeny:
  - fromEndpoints:
    - matchLabels:
        role: employee
    toPorts:
    - ports:
      - port: "443"
        protocol: TCP
      rules:
        http:
        - method: ".*"
          path: "/.*"
          headers:
          - "X-Time-Restriction: after-hours"
```

---

## Allow Policies

Allow policies explicitly permit traffic and can work with deny policies.

### Basic Allow Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-essential-services
spec:
  endpointSelector:
    matchLabels:
      app: microservice
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: gateway
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
  egress:
  - toEndpoints:
    - matchLabels:
        app: database
    toPorts:
    - ports:
      - port: "5432"
        protocol: TCP
  - toFQDNs:
    - matchName: "logging.external.com"
```

### Allow with Service Mesh Integration

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-istio-sidecar
spec:
  endpointSelector:
    matchLabels:
      app: bookinfo
  ingress:
  - fromEndpoints:
    - matchLabels:
        "k8s:io.kubernetes.pod.namespace": istio-system
  - fromEndpoints:
    - {} # Allow from any pod in same namespace
    toPorts:
    - ports:
      - port: "15001"     # Istio proxy
        protocol: TCP
```

---

## Advanced Scenarios

### Multi-tier Application Security

```yaml
# Frontend tier
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: frontend-policy
  namespace: web-app
spec:
  endpointSelector:
    matchLabels:
      tier: frontend
  ingress:
  - fromCIDR:
    - "0.0.0.0/0"        # Allow from internet
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      - port: "443"
        protocol: TCP
  egress:
  - toEndpoints:
    - matchLabels:
        tier: backend
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/.*"
        - method: "POST"
          path: "/api/.*"

---
# Backend tier
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: backend-policy
  namespace: web-app
spec:
  endpointSelector:
    matchLabels:
      tier: backend
  ingress:
  - fromEndpoints:
    - matchLabels:
        tier: frontend
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
  egress:
  - toEndpoints:
    - matchLabels:
        tier: database
    toPorts:
    - ports:
      - port: "5432"
        protocol: TCP

---
# Database tier
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: database-policy
  namespace: web-app
spec:
  endpointSelector:
    matchLabels:
      tier: database
  ingress:
  - fromEndpoints:
    - matchLabels:
        tier: backend
    toPorts:
    - ports:
      - port: "5432"
        protocol: TCP
  # No egress rules = deny all egress
```

### Cross-Namespace Communication

```yaml
apiVersion: cilium.io/v2
kind: CiliumClusterwideNetworkPolicy
metadata:
  name: cross-namespace-api
spec:
  endpointSelector:
    matchLabels:
      app: shared-api
  ingress:
  - fromEndpoints:
    - matchLabels:
        "k8s:io.kubernetes.pod.namespace": "frontend"
        app: web-app
    - matchLabels:
        "k8s:io.kubernetes.pod.namespace": "mobile"
        app: mobile-api
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/shared/.*"
```

### Zero Trust Network

```yaml
apiVersion: cilium.io/v2
kind: CiliumClusterwideNetworkPolicy
metadata:
  name: zero-trust-default-deny
spec:
  endpointSelector: {}   # Apply to all pods
  ingressDeny:
  - fromEntities:
    - "all"
  egressDeny:
  - toEntities:
    - "all"

---
# Explicit allow for essential services
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-dns
  namespace: kube-system
spec:
  endpointSelector:
    matchLabels:
      k8s-app: kube-dns
  ingress:
  - fromEntities:
    - "cluster"
    toPorts:
    - ports:
      - port: "53"
        protocol: UDP
      - port: "53"
        protocol: TCP
```