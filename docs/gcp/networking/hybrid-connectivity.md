# Hybrid Connectivity

## Description

Hybrid connectivity solutions enable secure connections between on-premises networks and Google Cloud VPC networks. GCP offers two primary options: Cloud VPN (encrypted tunnels over the internet) and Cloud Interconnect (dedicated physical connections). Both work with Cloud Router to enable dynamic routing using BGP.

## Cloud VPN

### Description
IPsec VPN tunnels that encrypt traffic between your on-premises network and GCP VPC over the public internet.

### Types

**HA VPN (High Availability VPN)**
- **SLA**: 99.99% availability
- **Architecture**: Two interfaces, two external IP addresses, redundant tunnels
- **Routing**: Dynamic (BGP) only
- **Recommended**: Standard option for production workloads

**Classic VPN**
- **SLA**: 99.9% availability  
- **Architecture**: Single interface, single external IP
- **Routing**: Static or dynamic (BGP)
- **Status**: Legacy, HA VPN preferred

### Key Features

- **Encryption**: IPsec tunnel with IKEv1 or IKEv2
- **Bandwidth**: Up to 3 Gbps per tunnel, use multiple tunnels for more
- **Routing**: Dynamic routing with Cloud Router (BGP)
- **Redundancy**: Active-active configuration with HA VPN
- **Cost-Effective**: No dedicated hardware required
- **Topology**: Supports VPN to on-premises, VPN to another cloud, VPC-to-VPC

### Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Throughput per tunnel** | 3 Gbps | Use multiple tunnels for higher bandwidth |
| **Tunnels per gateway** | 8 (HA VPN) | Can create multiple gateways |
| **Tunnels per Cloud Router** | 128 | Default limit |
| **MTU** | 1460 bytes | Due to IPsec overhead |
| **Encryption** | Overhead ~10-15% | Impacts throughput |

### When to Use Cloud VPN

✅ **Use When:**
- Moderate bandwidth requirements (< 10 Gbps aggregate)
- Quick setup needed (hours, not weeks)
- Cost sensitivity (cheaper than Interconnect)
- Geographic flexibility (works anywhere with internet)
- Temporary connectivity needs
- Disaster recovery secondary path

❌ **Don't Use When:**
- Require > 10 Gbps consistent throughput
- Need lowest possible latency (Interconnect is better)
- Compliance requires dedicated connection
- Require 99.99% SLA with deterministic routing (use Dedicated Interconnect)

---

## Cloud Interconnect

### Description
Dedicated physical connections between on-premises infrastructure and Google Cloud, providing higher bandwidth and lower latency than VPN.

### Types

**Dedicated Interconnect**
- **Connection**: Direct physical connection to Google network
- **Bandwidth**: 10 Gbps or 100 Gbps per circuit
- **Location**: Must be in a supported colocation facility
- **SLA**: 99.99% with proper redundancy configuration
- **Use Case**: High bandwidth, low latency, dedicated connectivity

**Partner Interconnect**  
- **Connection**: Through supported service provider
- **Bandwidth**: 50 Mbps to 50 Gbps per VLAN attachment
- **Location**: Service provider handles physical connectivity
- **SLA**: 99.99% with proper redundancy (depends on provider)
- **Use Case**: When not near Google edge location, or need managed service

### Key Features

- **Private Connectivity**: Traffic doesn't traverse public internet
- **Low Latency**: Direct connection to Google network
- **High Bandwidth**: 10-200 Gbps (Dedicated), 50 Mbps - 50 Gbps (Partner)
- **Flexible Routing**: BGP with Cloud Router
- **Layer 2 or Layer 3**: Dedicated supports both, Partner is Layer 3
- **VLAN Attachments**: Multiple VLANs over single physical connection

### Limits

**Dedicated Interconnect**

| Limit | Value | Notes |
|-------|-------|-------|
| **Bandwidth per connection** | 10 or 100 Gbps | Fixed circuit sizes |
| **VLAN attachments per interconnect** | 16 | Per cloud router |
| **Maximum total bandwidth** | 8 x 100 Gbps | 800 Gbps per VPC |
| **MTU** | 1500 bytes (default), up to 1440 for Partner | Standard Ethernet MTU |

**Partner Interconnect**

| Limit | Value | Notes |
|-------|-------|-------|
| **Bandwidth per attachment** | 50 Mbps - 50 Gbps | Varies by provider |
| **VLAN attachments per router** | 16 | Can use multiple routers |
| **Provider availability** | Geographic dependent | Check provider coverage |

### When to Use Cloud Interconnect

✅ **Use Dedicated Interconnect When:**
- Require > 10 Gbps bandwidth
- Already have presence in Google colocation facility
- Need lowest latency to Google Cloud
- Large-scale enterprise workloads
- Compliance requires dedicated physical connection
- Predictable network performance required

✅ **Use Partner Interconnect When:**
- Need > 3 Gbps but not near Google colocation
- Want managed service from network provider
- Need flexible bandwidth (can start small, scale up)
- Faster deployment than Dedicated Interconnect
- Provider already serves your location

❌ **Don't Use Interconnect When:**
- Bandwidth requirements < 1 Gbps (VPN more cost-effective)
- Quick POC or temporary project
- Budget constraints (VPN is cheaper)
- Geographic distance makes latency gains negligible

---

## Architecture Patterns

### HA VPN with Redundancy
```
On-Premises Router 1 ═══ Tunnel 1 ═══╗
                                      ╠═══ HA VPN Gateway (Interface 0)
On-Premises Router 1 ═══ Tunnel 2 ═══╝
                                      
On-Premises Router 2 ═══ Tunnel 3 ═══╗
                                      ╠═══ HA VPN Gateway (Interface 1)
On-Premises Router 2 ═══ Tunnel 4 ═══╝
```
**Result**: 99.99% SLA, survives any single router or tunnel failure

### Dedicated Interconnect with Redundancy
```
On-Premises ══ Metro 1 Circuit 1 ══ Interconnect Location A ══╗
                                                               ╠══ VPC Network
On-Premises ══ Metro 2 Circuit 2 ══ Interconnect Location B ══╝
```
**Result**: 99.99% SLA, geographic and circuit redundancy

### Hybrid: Interconnect + VPN Backup
```
Primary:   On-Premises ══ Interconnect ══════════ VPC
Backup:    On-Premises ══ HA VPN ══════════════= VPC
```
**Result**: High availability with failover to VPN if Interconnect fails

---

## Comparison Matrix

| Feature | HA VPN | Dedicated Interconnect | Partner Interconnect |
|---------|--------|----------------------|---------------------|
| **Bandwidth** | Up to 3 Gbps/tunnel | 10 or 100 Gbps | 50 Mbps - 50 Gbps |
| **Latency** | Medium (internet) | Low (direct) | Low (direct) |
| **SLA** | 99.99% | 99.99%* | 99.99%* |
| **Setup Time** | Hours | Weeks/months | Days/weeks |
| **Cost** | Low | High | Medium |
| **Location Requirement** | None | Colocation facility | Provider coverage |
| **Encryption** | IPsec (built-in) | Optional (MACsec) | Optional |
| **Use Case** | Small-medium workloads | Large enterprise | Medium-large workloads |

*With proper redundancy configuration

---

## Configuration Requirements

### Cloud Router
- **Required**: For dynamic routing (BGP) with VPN and Interconnect
- **ASN**: Private ASN (64512-65534, 4200000000-4294967294) or public ASN
- **BGP Sessions**: One per tunnel/VLAN attachment
- **Route Advertisement**: Automatically advertises VPC subnet routes

### BGP Configuration
- **BGP Peer**: Configure on-premises router and Cloud Router
- **ASN**: Unique ASN for on-premises and Cloud Router
- **MD5 Authentication**: Optional but recommended
- **Route Priorities**: Set MED or local preference for failover
- **BFD**: Enable for faster failure detection (sub-second)

### Firewall Considerations
- Allow IKE (UDP 500, 4500) for VPN
- Allow BGP (TCP 179) if using dynamic routing
- Configure VPC firewall rules for on-premises IP ranges
- Consider hierarchical firewall policies for organization-wide rules

---

## Best Practices

1. **Redundancy**
   - Always configure redundant tunnels/circuits
   - Use separate physical paths when possible
   - Test failover scenarios regularly

2. **Route Management**
   - Use BGP for dynamic routing
   - Set appropriate route priorities (MED, AS-PATH prepend)
   - Limit route advertisements (use route filters)
   - Enable BFD for fast failure detection

3. **Monitoring**
   - Monitor tunnel/circuit status
   - Set up alerts for connection failures
   - Track bandwidth utilization
   - Use VPC Flow Logs for traffic analysis

4. **Security**
   - Use strong encryption (AES-256-GCM for VPN)
   - Enable MD5 authentication on BGP sessions
   - Consider MACsec for Interconnect encryption
   - Implement principle of least privilege for route advertisements

5. **Capacity Planning**
   - Size connections for peak + 30-50% headroom
   - Plan for growth (easier to add VPN tunnels than Interconnect circuits)
   - Consider burst traffic patterns
   - Monitor usage trends

---

## Cost Optimization

**Cloud VPN**
- Charged per tunnel hour + egress bandwidth
- Multiple tunnels increase cost (but provide redundancy)
- Egress to on-premises charged at internet egress rates

**Cloud Interconnect**
- Dedicated: Fixed port fee + egress bandwidth (discounted)
- Partner: Varies by provider + egress bandwidth
- Generally cheaper egress rates than VPN at scale
- Break-even typically around 1-3 Gbps sustained traffic

---

## Related Services

- **Cloud Router**: Dynamic routing for VPN and Interconnect
- **Cloud DNS**: Extend DNS resolution to on-premises
- **Private Google Access**: Access Google services without public IPs
- **Network Connectivity Center**: Central hub for managing hybrid connectivity
