# Cilium

## Introduction

Cilium is a cloud-native networking, observability, and security solution built on top of eBPF (extended Berkeley Packet Filter). It provides advanced networking capabilities for containerized applications, particularly in Kubernetes environments, while offering enhanced security and observability features.

## Core Components

### 1. Cilium Agent
- **Role**: Primary component running as a DaemonSet on each Kubernetes node
- **Responsibilities**:
  - Manages eBPF programs on the node
  - Handles network policy enforcement
  - Maintains local identity and endpoint management
  - Communicates with Cilium Operator and etcd/Kubernetes API
- **Location**: Runs in userspace but interacts heavily with kernel space

### 2. Cilium Operator
- **Role**: Cluster-wide management component
- **Responsibilities**:
  - Manages cluster-wide resources (CiliumNetworkPolicy, CiliumClusterwideNetworkPolicy, Cluster Mesh)
  - Handles IPAM (IP Address Management) coordination
  - Manages CRD lifecycle and validation
  - Coordinates with cloud provider APIs for advanced features
- **Deployment**: Typically runs as a Deployment with 1-2 replicas

### 3. Envoy Proxy
- **Role**: Manage L7 traffic
- **Responsibilities**:
  - Manage L7 Network Policies
- **Location**: Part of Clilum Agent or standalone pod (daemon)

### 4. Cilium CNI Plugin (CRD)
- **Role**: Container Network Interface implementation
- **Responsibilities**:
  - Pod network setup and teardown
  - IP address assignment
  - Network namespace configuration
  - Integration with container runtime
- **Location**: Binary installed on each node

### 5. Hubble
- **Role**: Network observability and security monitoring
- **Components**:
  - **Hubble Server**: Runs alongside Cilium agent, exposes gRPC API and collects flows and visibility data using eBPF
  - **Hubble Relay**: Cluster-wide aggregation service
  - **Hubble UI**: Web-based interface for network visualization
  - **Hubble CLI**: Command-line tool for querying network flows throught thw Hubble relay

### 6. eBPF Programs
- **Role**: Manage L3/L4 traffic
- **Responsibilities**:
  - Manage L3/L4 Network Policies
- **Location**: Loaded into that node's kernel

## eBPF Foundation

### What is eBPF?
- Extended Berkeley Packet Filter - a kernel technology
- Allows running sandboxed programs in kernel space without changing kernel source
- Provides high-performance, programmable packet processing
- Enables advanced networking, security, and observability features

### Cilium's eBPF Usage

#### Network Datapath
- **TC (Traffic Control) Programs**: Attached to network interfaces for ingress/egress processing
- **XDP (eXpress Data Path) Programs**: Ultra-fast packet processing at driver level
- **Socket Operations**: Intercept and redirect socket operations
- **Kernel Tracing**: Monitor system calls and kernel functions

#### Key eBPF Maps
- **Identity Map**: Maps security identities to numeric IDs
- **Policy Map**: Stores network policy decisions
- **Endpoint Map**: Tracks local endpoints and their properties
- **Service Map**: Load balancing and service discovery information

## Cilium Features

### IPAM

Two available modes:
- Kubernetes Host Scope: kube-controllermanager assign PodCIDR to each nodes. Set the resource Node
```
ipam:
  mode: "kubernetes"
k8s:
  requireIPv4PodCIDR: true
  requireIPv6PodCIDR: true
```

- Cluster Scope (default): Cilium Operator assign PodCIDR. Use the resource CiliumNode
```
ipam:
  mode: "cluster-pool"
k8s:
  clusterPoolIPv4PodCIDRList: ["10.0.0.0/8"]
  clusterPoolIPv4MaskSize: 24
  clusterPoolIPv6PodCIDRList: ["fd00::/104"]
  clusterPoolIPv6MaskSize: 120
```

To get all assigned CIDRs:

```
cilium-dbg status --all-addresses
```

Do not change IPAM mode on live clusters. Instead, deploy a new cluster with desired IPAM mode and migrate workloads.

### Routing modes

Cilium supports following routing modes that determine how packets are forwarded between pods across nodes:
1. Native Routing
Uses the host's existing routing table and network stack
```
routingMode: "native"
ipV4NativeRoutingCIDR: "10.244.0.0/16"
ipV6NativeRoutingCIDR: ""
```

2. Tunnel - Encapsulating (Default)
Encapsulates pod traffic in VXLAN or Geneve tunnels
Overhead: ~50 bytes per packet (VXLAN + UDP + IP headers)
```
routingMode: "tunnel"

tunnelProtocol: "vxlan" # default
# tunnelProtocol: "geneve"

tunnelPort: 8472 # vxlan
# tunnelPort: 6081 # geneve
```

### Kube-Proxy Replacement

Cilium can completely replace kube-proxy with a more efficient eBPF-based implementation for Kubernetes Service load balancing.
Kube-proxy can degrades with large numbers of services/endpoints. eBPF-based has linear scaling regardless of service count.
```
kubeProxyReplacement: false # kube-proxy
---
kubeProxyReplacement: true # cilium
k8sServiceHost: "host-ip-control-plane"
k8sServicePort: "6443"
```

Check current config:
```
cilium-dbg status
```

### Cilium Ingress
Cilium provides L7 HTTP/HTTPS ingress capabilities without needing a separate ingress controller.
```
nodePort:
  enabled: true
ingressController:
  enabled: true
  default: true
  loadBalancerMode: dedicated # or "shared"
```
Ingress mode:

- Dedicated: one loadbalancer for each ingress
- Shared: one single loadbalancer for all ingress

Ingress components:
- GatewayClass: created by Infra team
- Gateway: created by cluster operator
- HTTPRoute (TCP, GRPC): created by developer

## Encryption
If enabled, traffic between clusters will be encryted, traffic within cluster not encrypted.
- IPSec
```
encryption:
  enabled: true
  type: ipsec
```
- WireGuard
```
encryption:
  enabled: true
  type: wireguard
```

check encryption status:
```
cilium-dbg encryption status
```

## mTLS
Applied with SPIFFE, implemented by Spire.
- A spire-server will be deployed in the cluster
- A spire-agent in every node

Encryption must be enabled.
```
encryption:
  enabled: true
  type: ipsec # or wireguard

authentication:
  enabled: true
  mutual:
    spire:
      enabled: true
      install:
        enabled: true
```

In the CiliumNetworkPolicy:
```
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
 name: frontend-policy
 namespace: default
spec:
 endpointSelector:
   matchLabels:
     app: frontend
 ingress:
 - fromEndpoints:
   - matchLabels:
       app: gateway
   toPorts:
   - ports:
     - port: "8080"
       protocol: TCP
     authentication: # <---
       mode: "required"
```

## Cluster Mesh

### Prerequisites
- Clusters must have same routing mode
- Non overlapping Pod CIDRs

```
cilium clustermesh enable --context $CLUSTER1 # Enable, for every cluster
cilium clustermesh connect --context $CLUSTER1 --destination-context $CLUSTER2 # connect clusters
cilium clustermesh status # check status

```

### KVStoreMesh
Cross-cluster networking without requiring a shared key-value store (etcd).
Replaces shared etcd with direct cluster-to-cluster communication. Clusters exchange identity and service information directly.
No central store: Eliminates shared etcd dependency

### Global Services
Global Services enable cross-cluster load balancing, a single service that distributes traffic across pods in multiple clusters.
Service exists in multiple clusters with same name/namespace
Cilium merges endpoints from all clusters into one global service
Traffic is load balanced across all clusters automatically
Failover: Automatically excludes unhealthy clusters

```
apiVersion: v1
kind: Service
metadata:
  name: global-service
  namespace: default
  annotations:
    service.cilium.io/global: "true"  # Makes service global (one service for all clusters)
    service.cilium.io/affinity: "local" # Prioritize local clusters (or "remote")
    service.cilium.io/shared: "false" # Make service local (one service per cluster)
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

## Observability Architecture

### Hubble
To enable:
```
hubble:
  enabled: true

  relay:
    enabled: true

  ui:
    enabled: true
```

To use hubble CLI:
```
cilium hubble port-forward # start communication

hubble observe --pod pod_name # show in/out traffic for pod
hubble observe --from-pod pod_name # show out traffic from pod
hubble observe --to-pod pod_name # show in traffic to pod

hubble observe --to-pod pod_name --port 3000 --protocol http --VERDICT [FORWARDED|DROPPED...] # filter by port, protocol and result
```

To use hubble UI:
```
cilium hubble ui # start server
```

## BGP & External Networking

### Egress gateway
Provides centralized egress traffic control, routing pod traffic through specific gateway nodes with predictable source IPs.

To enable:
```
bpf:
  masquerade: true

kubeProxyReplacement: true

egressGateway:
  enable: true
```

Configuration:
```
apiVersion: cilium.io/v2
kind: CiliumEgressGatewayPolicy
metadata:
  name: egress-policy
spec:
  selectors:
  - podSelector:
      matchLabels:
        app: frontend
  destinationCIDRs:
  - "10.10.10.0/24"  # External service subnet
  gatewayConfig:
    nodeSelector:
      matchLabels:
        egress-gateway: "true"  # Designated gateway nodes
```

Verification:
```
cilium-dbg bpf egress list
```

### LoadBalancer IPAM

Automatically assigns IP addresses to Kubernetes LoadBalancer services from predefined pools, eliminating dependency on cloud provider load balancers.
Cilium allocates IPs from configured pools to LoadBalancer services.

```
apiVersion: "cilium.io/v2alpha1"
kind: CiliumLoadBalancerIPPool
metadata:
  name: "main-pool"
spec:
  blocks:
  - cidr: "10.0.10.0/24"
  serviceSelector:
    matchLabels:
      color: red
```

### BGP
Enables dynamic route advertisement to network infrastructure, making pod/service IPs reachable from outside the cluster.
- Used with routing mode native.
- Cilium nodes peer with BGP routers
- Advertise routes for pod CIDRs, service IPs, and LoadBalancer IPs
- Network infrastructure learns routes and forwards traffic to correct nodes

To enable:
```
bgpControlPlane:
  enabled: true
```

Configuration:
```
apiVersion: "cilium.io/v2alpha1"
kind: CiliumBGPClusterConfig
metadata:
  name: cilium-bgp
spec:
  nodeSelector:
    matchLabels:
      bgp: "true" # every node must have this label to use bgp
  bgpInstances:
  - name: "instance-64000"
    localASN: 64000
    peers:
    - name: "peer-1"
      peerASN: 65000
      peerAddress: "10.0.0.1"  # Router switch IP
      peerConfigRef:
        name: "cilium-peer" # same as CiliumBGPPeerConfig name
---
apiVersion: "cilium.io/v2alpha1"
kind: CiliumBGPPeerConfig
metadata:
  name: cilium-peer
spec:
  timers:
    holdTimeSeconds: 9
    keepAliveTimeSeconds: 3
ebgpMultiHop: 5
  families:
  - afi: ipv4
    safi: unicast
    advertisments:
      matchLabels:
        advertise: bgp # must be the same of CiliumBGPAdvertisement labels
---
apiVersion: "cilium.io/v2alpha1"
kind: CiliumBGPAdvertisement
metadata:
  name: bgp-advertisement
  labels:
    advertise: bgp
spec:
  advertisements:
  - advertisementType: PodCIDR
    attributes:
      communities:
        standard: [ "65000:99" ]
      localPreference: 99
  - advertisementType: Service
    service:
      addresses:
      - ClusterIP
      - ExternalIP
      - LoadBalancerIP
    selector:
      matchExpressions:
      - key: type
        operator: In
        values: ["LoadBalancer"]
```

Troubleshoot:
```
cilium bgp peers # nodes where bgp is running and all other info
cilium bgp routes available # routes that the Cilium agent has learned
cilium bgp routes advertised # routes that the Cilium instances has advertised
```

## L2Announcement
Makes LoadBalancer service IPs reachable by responding to ARP requests on the local network segment, without requiring BGP infrastructure.

To enable:
```
kubeProxyReplacement: true

l2announcements:
  enable: true
```

Configuration:
```
apiVersion: "cilium.io/v2alpha1"
kind: CiliumL2AnnouncementPolicy
metadata:
  name: "l2-policy"
spec:
  serviceSelector:
    matchLabels:
      app: myapp
  nodeSelector:
    matchLabels:
    - key: node-role.kubernetes.io/control-plane
      operator: DoesNotExist
  interfaces:
  - "^eth[0-9]+"  # Network interfaces to announce on
  externalIPs: true
  loadBalancerIPs: true
```