# Load Balancing

## Description

Google Cloud Load Balancing is a fully distributed, software-defined managed service that distributes traffic across multiple backend instances. It's not instance-based or device-based, providing seamless auto-scaling and high availability. GCP offers multiple load balancer types optimized for different use cases.

## Load Balancer Types Overview

### Global vs Regional
- **Global**: Serve users globally from multiple regions, single anycast IP
- **Regional**: Serve users within a single region, useful for internal services

### External vs Internal  
- **External**: Accept traffic from the internet
- **Internal**: Accept traffic only from within VPC or connected networks

### Layer 4 vs Layer 7
- **Layer 4 (Network/Transport)**: TCP/UDP, connection-based routing
- **Layer 7 (Application)**: HTTP(S), content-based routing, URL mapping

---

## Application Load Balancer (Layer 7)

### External Application Load Balancer (Global)

**Description**: Global Layer 7 load balancer for HTTP(S) traffic with content-based routing.

**Key Features**:
- **Global load balancing**: Single anycast IP serves worldwide
- **SSL/TLS termination**: Managed certificates with automatic renewal
- **Content-based routing**: URL maps, host rules, path rules
- **Cloud CDN integration**: Caching at Google edge locations
- **Cloud Armor**: DDoS protection and WAF capabilities
- **Backend types**: Instance groups, NEGs (zonal, internet, serverless)
- **HTTP/2 and HTTP/3**: Modern protocol support

**Limits**:

| Limit | Value |
|-------|-------|
| Backend services per LB | 50 |
| URL maps | 10 per LB |
| Path matchers per URL map | 250 |
| Backends per backend service | 50 |
| Maximum request size | 32 KB (headers + URL) |

**When to Use**:
- ✅ Global web applications with users worldwide
- ✅ Need content-based routing (different paths to different backends)
- ✅ Require SSL termination and certificate management
- ✅ Want Cloud CDN for static content
- ✅ Need DDoS protection and WAF

**When Not to Use**:
- ❌ Non-HTTP traffic (use Network Load Balancer)
- ❌ Internal-only traffic (use Internal Application Load Balancer)
- ❌ UDP traffic (use Network Load Balancer)

### Internal Application Load Balancer (Regional)

**Description**: Regional Layer 7 load balancer for internal HTTP(S) traffic within VPC.

**Key Features**:
- **Private load balancing**: Internal IP addresses only
- **Regional**: Serves traffic within a region
- **Cross-region support**: Can access backends in other regions
- **Service mesh integration**: Works with Traffic Director
- **Shared VPC support**: Can be in host or service project

**When to Use**:
- ✅ Microservices architectures within VPC
- ✅ Private APIs and internal applications
- ✅ Service mesh deployments (with Traffic Director)
- ✅ Multi-tier applications (frontend to backend)

---

## Network Load Balancer (Layer 4)

### External Network Load Balancer (Regional)

**Description**: Regional Layer 4 load balancer for TCP/UDP traffic, high performance and low latency.

**Types**:
- **Premium Tier**: Anycast IP, global access
- **Standard Tier**: Regional IP, regional access

**Key Features**:
- **Pass-through**: Preserves client IP addresses
- **High throughput**: Millions of requests per second
- **UDP support**: Gaming, streaming, DNS
- **SSL/TLS**: Not terminated (passes through to backends)
- **Session affinity**: Client IP, client IP and protocol, client IP-port-proto

**Limits**:

| Limit | Value |
|-------|-------|
| Forwarding rules per project | 75 (regional) |
| Backend services per LB | 50 |
| Backends per backend service | 250 |

**When to Use**:
- ✅ Non-HTTP(S) protocols (TCP/UDP)
- ✅ Need to preserve client IP
- ✅ SSL passthrough required (SSL/TLS handled by backend)
- ✅ High-performance gaming, streaming applications
- ✅ Custom protocols on non-standard ports

### Internal Network Load Balancer (Regional)

**Description**: Regional Layer 4 load balancer for internal TCP/UDP traffic within VPC.

**Types**:
- **Standard**: Basic internal load balancing
- **Advanced**: Enhanced features, multi-subnet support

**Key Features**:
- **Private IPs**: Only accessible within VPC
- **High performance**: Low latency, high throughput
- **Failover**: Automatic with health checks
- **Multi-subnet**: Backends can be in different subnets (Advanced)

**When to Use**:
- ✅ Internal databases, message queues
- ✅ Private TCP/UDP services
- ✅ High-performance internal applications
- ✅ Load balancing for self-managed databases

---

## Proxy Network Load Balancer (Layer 4)

### External Proxy Network Load Balancer (Global/Regional)

**Description**: Global or Regional Layer 4 proxy for TCP traffic with SSL termination.

**Key Features**:
- **TCP proxy**: Terminates TCP connections
- **SSL offloading**: Terminate SSL at load balancer
- **Global**: Single anycast IP, route to nearest backend
- **Long-lived connections**: Optimized for persistent connections

**When to Use**:
- ✅ Non-HTTP TCP traffic that needs SSL termination
- ✅ Global applications on custom TCP ports
- ✅ Need SSL offloading for TCP applications
- ✅ WebSocket connections

### Internal Proxy Network Load Balancer (Regional)

**Description**: Regional Layer 4 proxy for internal TCP traffic.

**When to Use**:
- ✅ Internal TCP applications needing SSL termination
- ✅ Private WebSocket services
- ✅ Cross-region backend support with internal IPs

---

## Decision Matrix

```
                    Traffic Source
                    │
        ┌───────────┼───────────┐
        │                       │
    External                Internal
        │                       │
        │                       │
    HTTP(S)?                HTTP(S)?
    │       │               │       │
   Yes      No             Yes      No
    │       │               │       │
    │       │               │       │
    │   TCP/UDP?            │   TCP/UDP?
    │   │       │           │   │       │
    │  TCP     UDP          │  TCP     UDP
    │   │       │           │   │       │
    ▼   ▼       ▼           ▼   ▼       ▼

External    External  Ext.   Internal  Internal  Internal
Application Proxy N   Network  App LB   Proxy N   Network
LB (Global) LB       LB        (Reg)    LB (Reg)  LB (Reg)
```

---

## Common Architectures

### Global Multi-Region Web Application
```
Users (Global)
    │
    ▼
External Application Load Balancer (Anycast IP)
    │
    ├─── Backend: us-central1 (Instance Group)
    ├─── Backend: europe-west1 (Instance Group)
    └─── Backend: asia-east1 (Instance Group)
    
Cloud CDN: Enabled
Cloud Armor: DDoS Protection
SSL Certificate: Managed by Google
```

### Multi-Tier Application (Internal)
```
External Application LB ──▶ Frontend Tier (Public)
                                  │
                                  ▼
                    Internal Application LB ──▶ API Tier (Private)
                                                     │
                                                     ▼
                                       Internal Network LB ──▶ Database Tier
```

### Hybrid Cloud with Internal LB
```
On-Premises Network
    │
    ▼ (Cloud VPN/Interconnect)
    │
Internal Application Load Balancer
    │
    ├─── Backend: GKE Cluster (us-central1)
    └─── Backend: Compute VMs (us-east1)
```

---

## Key Concepts

### Backend Services
- **Health checks**: Determine backend availability
- **Session affinity**: Stick users to same backend
- **Connection draining**: Graceful shutdown, complete existing connections
- **Timeout**: How long to wait for backend response
- **Load balancing mode**: RATE, UTILIZATION, or CONNECTION based

### Backend Types
- **Instance groups**: Managed or unmanaged VM groups
- **Zonal NEGs**: Endpoints in specific zones (GCE VMs, GKE pods)
- **Internet NEGs**: External endpoints (on-prem, other clouds)
- **Serverless NEGs**: Cloud Run, App Engine, Cloud Functions
- **Hybrid NEGs**: PSC-based endpoints

### Health Checks
- **Protocol**: HTTP, HTTPS, TCP, SSL, HTTP/2
- **Interval**: Time between health checks
- **Timeout**: Time to wait for response  
- **Healthy threshold**: Consecutive successes to mark healthy
- **Unhealthy threshold**: Consecutive failures to mark unhealthy

### URL Maps (HTTP(S) Load Balancers)
- **Host rules**: Route based on hostname
- **Path matchers**: Route based on URL path
- **Path rules**: Specific path patterns
- **Default service**: Fallback for unmatched requests

---

## Features Comparison

| Feature | External App LB | Internal App LB | External Net LB | Internal Net LB |
|---------|----------------|-----------------|-----------------|-----------------|
| **Global** | ✅ | ❌ | Premium tier | ❌ |
| **SSL Termination** | ✅ | ✅ | ❌ | ❌ |
| **Cloud CDN** | ✅ | ❌ | ❌ | ❌ |
| **Cloud Armor** | ✅ | ❌ | ❌ | ❌ |
| **URL Routing** | ✅ | ✅ | ❌ | ❌ |
| **WebSocket** | ✅ | ✅ | Proxy LB | Proxy LB |
| **UDP** | ❌ | ❌ | ✅ | ✅ |
| **Preserve Client IP** | Custom header | Custom header | ✅ | ✅ |
| **IPv6** | ✅ | ❌ | ✅ | ❌ |

---

## Configuration Best Practices

1. **Health Checks**
   - Set appropriate intervals (not too aggressive)
   - Use dedicated health check endpoints
   - Return 200 OK only when truly healthy
   - Monitor health check failures

2. **Backend Configuration**
   - Enable connection draining (30-60 seconds)
   - Set reasonable session affinity when needed
   - Configure appropriate timeouts
   - Use multiple backends per region for redundancy

3. **Security**
   - Enable Cloud Armor for external HTTP(S) load balancers
   - Use managed SSL certificates (auto-renewal)
   - Implement SSL policies (minimum TLS version)
   - Restrict backend access with firewall rules

4. **Performance**
   - Enable Cloud CDN for cacheable content
   - Use HTTP/2 and HTTP/3
   - Configure appropriate backend timeout values
   - Size backend capacity for peak + 30% headroom

5. **Monitoring**
   - Set up monitoring for latency, error rates, traffic
   - Alert on health check failures
   - Monitor backend utilization
   - Use Cloud Trace for request tracing

---

## Pricing Considerations

**Application Load Balancers**:
- Forwarding rules (per hour)
- Data processing (per GB)
- Cloud CDN cache fill and egress (if enabled)

**Network Load Balancers**:
- Forwarding rules (per hour)
- No data processing charges (passthrough)

**General Tips**:
- Internal load balancers cheaper than external
- Data processing charges can be significant for HTTP(S) LBs
- Cloud CDN reduces origin traffic and data processing
- Consider regional vs global based on user distribution

---

## Related Services

- **Cloud CDN**: Content delivery network, integrates with HTTP(S) LB
- **Cloud Armor**: DDoS protection and WAF for HTTP(S) LB
- **SSL Certificates**: Managed or self-managed certificates
- **Cloud DNS**: DNS-based load balancing and failover
- **Traffic Director**: Service mesh traffic management
- **Network Endpoint Groups**: Advanced backend types
