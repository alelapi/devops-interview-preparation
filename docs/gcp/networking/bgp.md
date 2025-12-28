# BGP (Border Gateway Protocol)

## Description

BGP is the standard exterior gateway protocol used to exchange routing information between autonomous systems (AS) on the internet and within private networks. In Google Cloud, BGP is used with Cloud Router to enable dynamic routing between VPC networks and on-premises or other cloud networks via Cloud VPN and Cloud Interconnect.

**Purpose**: Automatically exchange route information, enabling dynamic network topology changes without manual route updates.

## BGP Fundamentals

### Autonomous System (AS)

- **Definition**: Collection of IP networks under single administrative control
- **ASN**: Unique number identifying the AS
- **Types**:
  - **Public ASN**: Registered with IANA (1-64511, 65536-4199999999)
  - **Private ASN**: For internal use only (64512-65534, 4200000000-4294967294)

### BGP Session Types

**eBGP (External BGP)**:

- Between different autonomous systems
- Default: On-premises AS to Google Cloud AS
- TTL: 1 (directly connected peers)
- Use case: Cloud VPN, Cloud Interconnect

**iBGP (Internal BGP)**:

- Within same autonomous system
- TTL: Maximum (255)
- Use case: Less common in GCP, more for traditional networks

### BGP Attributes

BGP uses various attributes to determine best path selection:

1. **AS-PATH**: List of ASNs the route has traversed
2. **Next-Hop**: IP address of next router in path
3. **MED (Multi-Exit Discriminator)**: Suggest preferred entry point
4. **Local Preference**: Prefer specific exit point (iBGP only)
5. **Weight**: Cisco proprietary (highest = best)
6. **Origin**: How route was learned (IGP, EGP, Incomplete)

## BGP Path Selection

BGP selects the best path using these criteria in order:

1. **Highest Weight** (Cisco-specific, local to router)
2. **Highest Local Preference** (default: 100)
3. **Locally originated routes** (network, aggregate, redistribute)
4. **Shortest AS-PATH**
5. **Lowest Origin type** (IGP < EGP < Incomplete)
6. **Lowest MED** (if from same AS)
7. **eBGP over iBGP**
8. **Lowest IGP metric** to BGP next hop
9. **Oldest route** (for eBGP paths)
10. **Lowest Router ID**

**In GCP Context**: You primarily control path selection via **MED** (from Cloud Router) and **AS-PATH prepending** (from on-premises).

## BGP in Google Cloud

### Cloud Router ASN Configuration

When creating a Cloud Router, you specify an ASN:

**Private ASN Ranges**:

- **16-bit**: 64512 - 65534 (for compatibility)
- **32-bit**: 4200000000 - 4294967294 (recommended)

**Best Practices**:

- Use 32-bit private ASN (4-byte ASN) for better uniqueness
- Document ASN assignments across your organization
- Ensure Cloud Router ASN differs from peer ASN
- Use consistent ASN scheme (e.g., 4200000000 + project_number)

### BGP Session Configuration

Each BGP session requires:

```yaml
Cloud Router Configuration:

  - Router ASN: 64512 (example)
  - Interface: VPN tunnel or VLAN attachment
  - Peer ASN: 65001 (on-premises ASN)
  - Peer IP: Automatically assigned (VPN) or manually configured (Interconnect)
  - Advertised Route Priority (MED): 100 (default)
  - MD5 Authentication: Optional shared secret
  - BFD: Enabled/Disabled
```

**On-Premises Router Configuration**:

- Must configure complementary BGP session
- Neighbor IP: Cloud Router's interface IP
- Neighbor ASN: Cloud Router's ASN
- Route advertisement: On-premises networks to advertise

## Route Advertisement

### From Cloud Router to On-Premises

**Default Mode**:

- Advertises all VPC subnet routes automatically
- Regional routing: Only subnets in the same region
- Global routing: All subnets in all regions

**Custom Mode**:

- Manually specify IP ranges to advertise
- Can advertise:
  - VPC subnet ranges
  - Secondary subnet ranges
  - Custom IP ranges
  - Summary routes (supernets)

**Example**:
```
VPC Subnets:

  - 10.1.0.0/24
  - 10.2.0.0/24
  - 10.3.0.0/24

Custom Advertisement:

  - 10.0.0.0/8 (summary route instead of individual /24s)
```

### From On-Premises to Cloud Router

- Configure on your on-premises router
- Advertise on-premises network ranges
- Cloud Router learns these routes via BGP
- Routes are programmed into VPC routing table

**Important**: Learned routes count against quota (default 250, max 5000 per Cloud Router).

## Route Priority and Traffic Engineering

### Using MED (Multi-Exit Discriminator)

MED influences **inbound traffic** (from on-premises to GCP):

**Configuration on Cloud Router**:
```
Cloud Router 1 (Primary Path):

  - Advertised Route Priority: 50 (lower = preferred)

Cloud Router 2 (Backup Path):

  - Advertised Route Priority: 100 (higher = less preferred)
```

**Result**: On-premises router prefers routes via Cloud Router 1.

**Important**:

- Lower MED = higher priority
- MED is compared only for routes from same neighboring AS
- Default MED: 100

### Using AS-PATH Prepending

AS-PATH prepending influences **outbound traffic** (from GCP to on-premises):

**Configuration on On-Premises Router**:
```
Primary Path:

  - Advertise: 192.168.0.0/16 (AS-PATH: 65001)

Backup Path:

  - Advertise: 192.168.0.0/16 (AS-PATH: 65001 65001 65001)
  - Prepend AS 65001 twice to make path "longer"
```

**Result**: Cloud Router prefers routes via primary path (shorter AS-PATH).

### Active-Active vs Active-Passive

**Active-Active**:

- Equal cost multi-path (ECMP)
- Same MED values on both paths
- Traffic load-balanced across both paths
- Use case: Maximize bandwidth, high availability

**Active-Passive**:

- Primary path with lower MED
- Backup path with higher MED
- Traffic uses primary path unless it fails
- Use case: Controlled failover, predictable routing

## BFD (Bidirectional Forwarding Detection)

### Purpose
Fast failure detection for BGP sessions:

- BGP keepalive: 60 seconds (slow)
- BFD: 1-3 seconds (fast)

### How It Works

- Lightweight "hello" packets exchanged between peers
- Independent of BGP protocol
- Detects link/path failures quickly
- Triggers BGP session teardown on failure

### Configuration in GCP

```yaml
BGP Session:

  - BFD: Enabled
  - Min Transmit Interval: 1000 ms (default)
  - Min Receive Interval: 1000 ms (default)
  - Multiplier: 3 (default)
```

**Calculation**: 3 missed packets × 1000ms = ~3 second failure detection

### When to Use BFD

- ✅ Production environments
- ✅ Active-active configurations requiring fast failover
- ✅ Critical workloads with HA requirements
- ❌ Non-production environments (adds complexity)

## Common Scenarios

### Scenario 1: Prefer Primary VPN Tunnel

**Requirement**: Route traffic via Tunnel 1, use Tunnel 2 as backup.

**Solution**:
```
Cloud Router BGP Sessions:

  - Tunnel 1: MED = 50
  - Tunnel 2: MED = 100

On-premises routes traffic via Tunnel 1 (lower MED preferred).
```

### Scenario 2: Regional Preference

**Requirement**: US traffic uses US region, Europe traffic uses Europe region.

**Solution**:
```
Cloud Router US (us-central1):

  - Advertises US subnets with MED = 50
  - Advertises EU subnets with MED = 100

Cloud Router EU (europe-west1):

  - Advertises EU subnets with MED = 50
  - Advertises US subnets with MED = 100

On-premises router learns both, prefers regional routes (lower MED).
```

### Scenario 3: Controlled Outbound Path

**Requirement**: Force GCP to use specific on-premises path.

**Solution**:
```
On-Premises Router:

  - Primary path: AS-PATH = 65001
  - Backup path: AS-PATH = 65001 65001 65001 (prepended)

Cloud Router prefers primary (shorter AS-PATH).
```

### Scenario 4: Load Balancing Across Tunnels

**Requirement**: Distribute traffic evenly across multiple VPN tunnels.

**Solution**:
```
Cloud Router:

  - Tunnel 1: MED = 100
  - Tunnel 2: MED = 100
  - Tunnel 3: MED = 100
  - Tunnel 4: MED = 100

Equal cost paths enable ECMP load balancing.
```

## MD5 Authentication

### Purpose
Secure BGP sessions from spoofing and unauthorized peers.

### Configuration

**In GCP**:
```bash
gcloud compute routers add-bgp-peer ROUTER_NAME \
  --peer-name=PEER_NAME \
  --peer-asn=PEER_ASN \
  --interface=INTERFACE \
  --md5-authentication-key="SHARED_SECRET"
```

**On On-Premises Router** (example):
```
router bgp 65001
  neighbor 169.254.1.1 remote-as 64512
  neighbor 169.254.1.1 password SHARED_SECRET
```

**Best Practices**:

- Use strong, random keys (16+ characters)
- Store keys securely (secrets management)
- Rotate keys periodically
- Consistent configuration on both sides

## Limits and Quotas

| Item | Limit | Notes |
|------|-------|-------|
| **BGP sessions per Cloud Router** | 128 | Default |
| **Learned routes per Cloud Router** | 250 (default), 5000 (max) | Request increase if needed |
| **Advertised routes** | Unlimited | Based on VPC subnets |
| **ASN range (private)** | 64512-65534, 4200000000-4294967294 | RFC 6996 |
| **BGP keepalive** | 20 seconds | Not configurable |
| **BGP hold timer** | 60 seconds | Not configurable |
| **BFD intervals** | Min 1000ms | Configurable per session |

## Best Practices

### 1. ASN Planning

- Document all ASN assignments
- Use 32-bit ASNs to avoid conflicts
- Reserve ASN ranges for different purposes (prod, dev, regions)
- Ensure uniqueness across entire network

### 2. Route Management

- Start with default advertisement, move to custom if needed
- Use route summarization to reduce route count
- Monitor learned routes against quotas
- Implement route filtering on both sides

### 3. Redundancy

- Configure multiple BGP sessions per Cloud Router
- Use diverse physical paths when possible
- Enable BFD for fast failover
- Test failover scenarios regularly

### 4. Traffic Engineering

- Document MED and AS-PATH configurations
- Use consistent MED values for similar paths
- Avoid excessive AS-PATH prepending (3x max)
- Monitor traffic flow and adjust as needed

### 5. Security

- Enable MD5 authentication on all BGP sessions
- Use firewall rules to restrict BGP peers
- Monitor BGP session status for anomalies
- Regular security audits of BGP configurations

### 6. Monitoring

- Set up alerts for BGP session down
- Monitor route count approaching limits
- Track route flapping (unstable routes)
- Use VPC Flow Logs to verify traffic paths

## Troubleshooting

### BGP Session Not Establishing

**Symptoms**: BGP state stuck in "Idle" or "Active"

**Common Causes**:

1. ASN mismatch (peer ASN incorrect)
2. MD5 authentication mismatch
3. Incorrect peer IP address
4. Firewall blocking TCP 179
5. Interface down (VPN tunnel down)

**Debug Steps**:
```bash
# Check BGP session status
gcloud compute routers get-status ROUTER_NAME --region=REGION

# Check interface status  
gcloud compute vpn-tunnels describe TUNNEL_NAME --region=REGION

# Verify configuration
gcloud compute routers describe ROUTER_NAME --region=REGION
```

### Routes Not Being Advertised/Learned

**Symptoms**: Expected routes missing from routing table

**Common Causes**:

1. VPC dynamic routing mode (regional vs global)
2. Custom advertisement not configured
3. Route quota exceeded
4. AS-PATH loop detection
5. Route filtering on peer side

**Debug Steps**:
```bash
# Check advertised routes
gcloud compute routers get-status ROUTER_NAME --region=REGION

# Check VPC routing table
gcloud compute routes list --filter="network:NETWORK_NAME"

# Verify learned routes count
gcloud compute routers get-status ROUTER_NAME --region=REGION \
  --format="value(result.bgpPeerStatus[].numLearnedRoutes)"
```

### Asymmetric Routing

**Symptoms**: Inbound and outbound traffic taking different paths

**Diagnosis**:

- Use VPC Flow Logs to trace traffic paths
- Check MED values on all BGP sessions
- Verify AS-PATH prepending configuration
- Review on-premises routing policy

**Solutions**:

- Adjust MED to align inbound path with outbound
- Modify AS-PATH prepending on outbound path
- Ensure symmetric route advertisements

## Related Concepts

- **Cloud Router**: GCP service that implements BGP
- **VPC Dynamic Routing**: Determines scope of route advertisement
- **Cloud VPN**: Uses BGP for dynamic routing (HA VPN)
- **Cloud Interconnect**: Uses BGP for route exchange
- **ECMP**: Equal-Cost Multi-Path routing for load balancing
- **VRF**: Virtual Routing and Forwarding (used internally by Google)

## Further Reading

- **RFC 4271**: BGP-4 Protocol Specification
- **RFC 6996**: Private Use AS Numbers
- **RFC 5880**: BFD Protocol
- **RFC 4456**: BGP Route Reflection
- **Google Cloud Documentation**: Cloud Router BGP Configuration
