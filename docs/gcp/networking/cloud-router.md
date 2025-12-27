# Cloud Router

## Description

Cloud Router is a fully distributed and managed Google Cloud service that uses Border Gateway Protocol (BGP) to dynamically exchange routes between your Google Cloud VPC network and on-premises or other cloud networks. It eliminates the need for static routes and enables automatic failover, scalability, and simplified network management.

**Architecture**: Software-defined router that runs in Google's network, enabling dynamic route propagation without physical router appliances.

## Key Features

### Dynamic Routing
- **BGP Protocol**: Industry-standard protocol for route exchange
- **Automatic Route Updates**: Routes automatically updated when topology changes
- **Multi-Path**: Supports ECMP (Equal-Cost Multi-Path) routing
- **Route Priorities**: Control route selection with MED and AS-PATH

### High Availability
- **Fully Managed**: No VMs to manage, patch, or scale
- **No Single Point of Failure**: Distributed service across Google infrastructure
- **Redundant BGP Sessions**: Multiple sessions for failover
- **BFD Support**: Bidirectional Forwarding Detection for fast failure detection

### Integration
- **Cloud VPN**: Required for dynamic routing with HA VPN
- **Cloud Interconnect**: Enables BGP for Dedicated and Partner Interconnect
- **Router Appliances**: Integrates with third-party network virtual appliances
- **Multiple Interfaces**: Can peer with multiple VPN tunnels or VLAN attachments

### Route Management
- **Route Advertisement**: Automatically advertises VPC subnet routes
- **Custom Route Advertisement**: Selectively advertise specific IP ranges
- **Learned Routes**: Receives routes from on-premises via BGP
- **Route Filtering**: Control which routes to accept or advertise
- **Route Priorities**: Configure local preference and MED values

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Cloud Routers per VPC** | 5 per region | Can be increased |
| **BGP sessions per router** | 128 | 8 per tunnel/VLAN for redundancy |
| **Learned routes per router** | 250 (default), 5000 (max) | Requires custom route advertisement mode |
| **Advertised routes** | Based on VPC subnets + custom ranges | |
| **BGP route priority (MED)** | 0-4294967295 | Lower is preferred |
| **ASN range (private)** | 64512-65534, 4200000000-4294967294 | RFC 6996 |
| **BGP keepalive interval** | 20 seconds (default) | Not configurable |
| **BGP hold time** | 60 seconds (default) | Not configurable |

## Route Advertisement Modes

### Default Mode
- Automatically advertises all subnet routes in the VPC
- Simple configuration, minimal management
- Routes added/removed automatically as subnets change
- Cannot selectively control which subnets to advertise

### Custom Mode
- Manually specify which IP ranges to advertise
- Greater control over route advertisement
- Required for advertising > 250 routes
- Can advertise routes outside VPC CIDR ranges
- Useful for summarizing routes

**Example Use Cases for Custom Mode**:
- Advertising summary routes instead of individual subnets
- Advertising secondary IP ranges
- Advertising IP ranges from other VPCs (via VPC peering)
- Limiting route advertisements to specific subnets

## BGP Configuration

### ASN (Autonomous System Number)
- **Cloud Router ASN**: Assigned to the Cloud Router
- **Peer ASN**: On-premises or partner router ASN
- **Private ASNs**: 64512-65534 (16-bit), 4200000000-4294967294 (32-bit)
- **Public ASNs**: Registered with IANA (if you own one)

**Important**: ASN must be unique across your BGP topology. Cloud Router and peer must have different ASNs.

### BGP Peering
Each BGP session requires:
- **Peer IP**: On-premises router IP (for VPN, automatically assigned)
- **Peer ASN**: Autonomous System Number of peer router
- **Interface**: Cloud Router interface (VPN tunnel or VLAN attachment)
- **Advertised Route Priority**: MED value for this session
- **MD5 Authentication**: Optional but recommended for security

### Route Priorities

**MED (Multi-Exit Discriminator)**:
- Lower MED = higher priority
- Default: 100
- Range: 0-4294967295
- Use case: Prefer specific paths for inbound traffic from on-premises

**AS-PATH Prepending**:
- Configured on peer side (on-premises router)
- Longer AS path = lower priority
- Use case: Influence route selection from Cloud Router perspective

## When to Use

### ✅ Use Cloud Router When:

1. **Hybrid Connectivity with Dynamic Routing**
   - Cloud VPN (HA VPN requires Cloud Router)
   - Cloud Interconnect (Dedicated or Partner)
   - Need automatic route updates between GCP and on-premises

2. **High Availability Requirements**
   - Automatic failover between redundant connections
   - Multiple VPN tunnels or VLAN attachments
   - BFD for sub-second failure detection

3. **Complex Network Topologies**
   - Multiple on-premises sites
   - Multi-region GCP deployments
   - Hub-and-spoke architectures with dynamic routing

4. **Scalable Route Management**
   - Large number of subnets that change frequently
   - Want to avoid manual route updates
   - Need route summarization

5. **Network Virtual Appliances**
   - Integrate third-party routers/firewalls
   - Custom routing requirements
   - Advanced traffic inspection/manipulation

### ❌ Don't Use Cloud Router When:

1. **Simple Static Routing Sufficient**
   - Few static routes that rarely change
   - Small-scale deployments
   - Classic VPN with static routes is adequate

2. **No Hybrid Connectivity**
   - Pure cloud-only deployments with no on-premises integration
   - VPC-to-VPC routing handled by VPC peering

3. **Layer 2 Requirements**
   - Need Layer 2 connectivity (Cloud Router is Layer 3)
   - Use Dedicated Interconnect Layer 2 connections directly

## Common Architectures

### HA VPN with Redundant Cloud Routers
```
On-Premises Router
    ├── BGP Session 1 ──▶ Cloud Router 1 (us-central1)
    │                         ├── VPN Tunnel 1
    │                         └── VPN Tunnel 2
    └── BGP Session 2 ──▶ Cloud Router 2 (us-central1)
                              ├── VPN Tunnel 3
                              └── VPN Tunnel 4

Route Advertisement: All VPC subnets
Learned Routes: On-premises CIDRs
Failover: Automatic via BGP + BFD
```

### Multi-Region with Cloud Interconnect
```
On-Premises
    └── Dedicated Interconnect
            ├── VLAN Attachment 1 ──▶ Cloud Router (us-central1)
            │                              └── Advertises: us-central1 subnets
            └── VLAN Attachment 2 ──▶ Cloud Router (europe-west1)
                                           └── Advertises: europe-west1 subnets

VPC Network Mode: Regional dynamic routing
Route Preference: Regional (routes preferred in same region)
```

### Router Appliance with Cloud Router
```
VPC Network
    ├── Cloud Router (us-central1)
    │       ├── BGP Session ──▶ Third-party Firewall/Router VM
    │       └── Advertises: Custom route 0.0.0.0/0 (default route)
    │
    └── Subnets
            └── Routes: Default route via appliance VM
```

## VPC Dynamic Routing Modes

Cloud Router behavior depends on VPC network's dynamic routing mode:

### Regional Dynamic Routing (Default)
- Cloud Router **only advertises regional subnet routes**
- Cloud Router **only programs learned routes in the same region**
- Use case: Keep traffic regional, reduce inter-region costs

**Example**:
```
VPC with Regional Dynamic Routing
├── Cloud Router (us-central1)
│       └── Advertises: Only us-central1 subnet routes
└── Cloud Router (europe-west1)
        └── Advertises: Only europe-west1 subnet routes
```

### Global Dynamic Routing
- Cloud Router **advertises all subnet routes** across all regions
- Cloud Router **programs learned routes globally** in all regions
- Use case: Global access to on-premises, multi-region failover

**Example**:
```
VPC with Global Dynamic Routing
├── Cloud Router (us-central1)
│       └── Advertises: ALL subnets (us-central1 + europe-west1 + asia-east1)
└── Cloud Router (europe-west1)
        └── Advertises: ALL subnets (us-central1 + europe-west1 + asia-east1)
```

**Consideration**: Global routing enables VMs in any region to reach on-premises via any Cloud Router, but may increase latency and inter-region egress costs.

## BFD (Bidirectional Forwarding Detection)

### Purpose
Provides fast failure detection (sub-second) for BGP sessions, enabling rapid failover.

### Configuration
- **Enabled per BGP session**
- **Min transmit interval**: 1000 ms (default)
- **Min receive interval**: 1000 ms (default)
- **Multiplier**: 3 (default) - 3 missed packets = failure

### Benefits
- Default BGP keepalive (60s) is slow; BFD detects failures in ~3 seconds
- Faster convergence on path failures
- Recommended for production deployments

### When to Enable
- ✅ Production environments requiring high availability
- ✅ Active-active VPN tunnel configurations
- ✅ Critical workloads needing fast failover
- ❌ Development/test environments (to reduce complexity)

## Configuration Best Practices

1. **Redundancy**
   - Deploy at least two Cloud Routers per region for HA
   - Use multiple BGP sessions per Cloud Router
   - Configure different route priorities for traffic steering

2. **Route Management**
   - Start with default advertisement mode, switch to custom if needed
   - Use route summarization to reduce advertised route count
   - Document custom route advertisements clearly
   - Monitor learned routes vs quota limits

3. **BGP Tuning**
   - Enable BFD for fast failure detection
   - Set appropriate MED values for traffic steering
   - Use MD5 authentication on BGP sessions
   - Coordinate AS-PATH prepending with network team

4. **Naming Conventions**
   - Use descriptive names: `cloud-router-us-central1-vpn-primary`
   - Include region and purpose in the name
   - Document ASN assignments

5. **Monitoring**
   - Monitor BGP session status
   - Track learned routes count
   - Alert on route quota approaching limits
   - Monitor route propagation delays

## Troubleshooting Common Issues

### BGP Session Not Establishing

**Check**:
- Peer ASN configured correctly
- Interface (tunnel/VLAN) is UP
- On-premises router BGP configuration
- MD5 authentication matches (if used)
- Firewall allows BGP (TCP 179)

### Routes Not Being Advertised

**Check**:
- Cloud Router advertisement mode (default vs custom)
- VPC dynamic routing mode (regional vs global)
- Custom route ranges configured correctly
- Route quota not exceeded

### Routes Not Being Learned

**Check**:
- BGP session established
- On-premises router advertising routes
- Learned route quota not exceeded
- Route priority/AS-PATH not causing rejection

### Asymmetric Routing

**Cause**: Different paths for inbound and outbound traffic

**Solutions**:
- Adjust MED values to align inbound path
- Use AS-PATH prepending on outbound path
- Review VPC dynamic routing mode
- Check for multiple Cloud Routers with different priorities

## Cost Considerations

- **No direct charge** for Cloud Router itself
- **Charges apply** for associated resources:
  - VPN tunnels (per tunnel-hour)
  - Interconnect VLAN attachments (per attachment-hour)
  - Egress traffic via VPN/Interconnect
- **Cost optimization**:
  - Use regional routing mode if global routing not needed (reduces inter-region egress)
  - Consolidate BGP sessions where possible (within limits)

## Related Services

- **Cloud VPN**: Uses Cloud Router for dynamic routing
- **Cloud Interconnect**: Requires Cloud Router for BGP
- **VPC Peering**: Does not use Cloud Router (static routing)
- **Network Connectivity Center**: Centralized management of hybrid connectivity
- **Router Appliances**: Third-party NVAs integrated with Cloud Router
- **Private Service Connect**: Access Google services privately (doesn't use Cloud Router)
