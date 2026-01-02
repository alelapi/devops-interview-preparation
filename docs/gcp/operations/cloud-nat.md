# Cloud NAT

## Core Concepts

Cloud NAT (Network Address Translation) provides outbound internet connectivity for private instances without external IP addresses. Managed service that scales automatically.

**Key Principle**: Private instances access internet without being directly accessible from internet.

## How Cloud NAT Works

```
Private VM (no external IP) → Cloud NAT → Internet
Internet responses → Cloud NAT → Private VM
```

**Direction**: Outbound only (initiated by VM)

**Inbound**: NOT possible (VMs remain private, no unsolicited inbound)

**NAT Gateway**: Managed by Google, no instances to maintain

## When to Use Cloud NAT

### ✅ Use When

- VMs need internet access (updates, APIs, downloads)
- Don't want external IPs (security)
- Centralized egress control
- Need static external IPs for egress
- Outbound-only connectivity required
- Cost-effective outbound at scale

### ❌ Don't Use When

- Need inbound access → Use external IPs or Load Balancer
- Private Google API access only → Private Google Access sufficient
- VPC-to-VPC communication → VPC peering or VPN

## Cloud NAT vs Alternatives

| Method | Direction | Cost | Management | Use Case |
|--------|-----------|------|------------|----------|
| **Cloud NAT** | Outbound | Per GB | Managed | Private VMs, internet access |
| **External IP** | Both | Per IP | Simple | Public-facing services |
| **Proxy VM** | Both | VM cost | Self-managed | Custom filtering |
| **Private Google Access** | GCP services only | Free | Simple | Access GCP APIs only |

## Architecture

### Regional Service

**Scope**: Cloud NAT is regional (serves VMs in specific region)

**Multi-region**: Deploy separate Cloud NAT per region

### Attached to VPC

**Requirement**: Cloud NAT attached to router in VPC

**Pattern**:

```
VPC → Cloud Router → Cloud NAT → Internet
  └── Subnets with private VMs
```

### IP Address Allocation

**Options**:

- **Automatic**: Google assigns IPs
- **Manual**: Specify IP addresses

**Benefit of manual**: Allowlist specific IPs with external services

### Port Allocation

**Default**: Dynamic port allocation

**Ports per VM**: 64 minimum, 64K maximum

**Consideration**: Each connection uses a port; more VMs = more ports needed

## Configuration Options

### Subnet Selection

**All subnets** (default):

- All subnets in region use Cloud NAT
- Simplest configuration

**Selected subnets**:

- Only specific subnets use Cloud NAT
- Granular control
- Mixed public/private architecture

**Primary and secondary ranges**: Choose which IP ranges use NAT

### NAT IP Addresses

**Automatic**:

- Google allocates IPs
- Scalable
- Can't predict IPs

**Manual**:

- Specify reserved external IPs
- Static IPs for allowlisting
- More control

**Number**: Start with N IPs = VMs/32, increase if port exhaustion

### Logging

**Purpose**: See which VM accessed what endpoint

**Log types**:

- Translations (connections)
- Errors (dropped packets)

**Use for**: Debugging, audit, security

**Cost**: Log storage charges apply

## Common Patterns

### Standard Private Architecture

```
VMs (private, no external IP)
  → Cloud NAT (outbound only)
  → Internet (updates, API calls)
```

**Benefit**: Security (no inbound), cost (no external IPs)

### Hybrid Cloud

```
On-premises → Cloud VPN → VPC → Cloud NAT → Internet
```

**Use case**: On-premises uses cloud for internet egress

### Multi-Environment

```
VPC-Dev → Cloud NAT (dev-nat) → Internet
VPC-Staging → Cloud NAT (staging-nat) → Internet
VPC-Prod → Cloud NAT (prod-nat) → Internet
```

**Benefit**: Separate egress IPs per environment

### Allowlist with External Services

```
VM → Cloud NAT (manual IPs: 1.2.3.4, 5.6.7.8) → External API
External API allowlist: 1.2.3.4, 5.6.7.8
```

**Use case**: Third-party services require IP allowlisting

## High Availability

**Automatic**: Cloud NAT is highly available by default

**No single point of failure**: Managed service, Google handles redundancy

**Regional**: Zone failure doesn't affect Cloud NAT

**Best practice**: No special configuration needed, works automatically

## Port Exhaustion

### Understanding Ports

**Per IP**: 64,511 usable ports (65,535 total - reserved)

**Per Connection**: One port used

**Problem**: Too many VMs or connections → run out of ports

### Symptoms

- Connection failures
- Timeouts to internet
- "Cannot assign requested address" errors

### Solutions

**Add more NAT IPs**:

- More IPs = more ports
- 2 IPs = ~130K ports

**Increase ports per VM**:

- Default 64 minimum → Increase to 128, 256, etc.
- More ports = fewer VMs per IP

**Reduce connections**:

- Connection pooling in applications
- Increase connection timeouts
- Close connections properly

**Formula**: `Max VMs per IP = 64511 / min_ports_per_vm`

**Example**: 64 ports/VM = 1008 VMs per IP

## Monitoring and Troubleshooting

### Key Metrics

**Port allocation**:

- Allocated ports per VM
- Port usage percentage
- VMs approaching port limit

**Traffic**:

- Sent/received bytes
- Dropped packets
- Connection count

**Errors**:

- NAT allocation failures
- Port exhaustion

### Alerts

**Critical**:

- Port utilization > 80%
- NAT allocation failures
- Dropped packets increasing

**Warning**:

- Port utilization > 60%
- High connection count per VM

### Common Issues

**Problem**: VMs can't reach internet

**Check**:

1. Cloud NAT configured for subnet?
2. Router exists in VPC?
3. Firewall allows egress?
4. Private Google Access interfering?

**Problem**: Connections intermittently fail

**Cause**: Port exhaustion

**Fix**: Add NAT IPs or increase min ports per VM

## Security Considerations

### Egress Control

**Benefit**: All outbound through known IPs

**Use cases**:

- Security monitoring (log egress)
- Compliance (data loss prevention)
- Cost tracking (egress costs)

### Firewall Rules

**Still apply**: VPC firewall rules control traffic

**Pattern**:

- Firewall deny all egress (default)
- Firewall allow specific destinations (allowlist)
- Cloud NAT provides translation

### No Inbound Risk

**Key security feature**: VMs unreachable from internet

**Even with Cloud NAT**: No unsolicited inbound connections

## Cost Model

**Pricing**:

- Per GB processed: ~$0.045/GB (us-central1)
- NAT gateway hours: ~$0.045/hour per gateway
- Data transfer charges apply (egress)

**Optimization**:

- Use for all private VMs (better than external IPs)
- Not charged for internal traffic
- Regional data transfer cheaper

**Comparison with external IPs**:

- External IP: ~$0.004/hour per IP (~$3/month)
- Cloud NAT: ~$33/month + $0.045/GB

**Decision**: Cloud NAT cheaper at scale, better security

## Cloud NAT vs Private Google Access

| Feature | Cloud NAT | Private Google Access |
|---------|-----------|----------------------|
| **Destination** | Internet | GCP APIs only |
| **IPs needed** | Yes (external) | No |
| **Use case** | Internet access | GCP service access |
| **Cost** | Per GB + gateway | Free |

**Use together**: Private Google Access for GCP services, Cloud NAT for internet

## Integration with Other Services

### With Load Balancers

**Inbound**: Load Balancer handles inbound

**Outbound**: Cloud NAT handles outbound from backends

**Pattern**: Private backends behind LB, outbound via NAT

### With GKE

**Node pools**: Private nodes use Cloud NAT for outbound

**Configuration**: Automatic when enabling private cluster

**Image pulls**: Container images downloaded via Cloud NAT

### With Cloud SQL

**Cloud SQL Proxy**: Works with private IPs, no Cloud NAT needed

**Direct connection**: Private IP connection (no NAT)

## Best Practices

### Use Private IPs

**Default**: VMs should NOT have external IPs

**Exception**: Public-facing services only (load balancers)

**Benefit**: Security, cost, centralized egress

### Monitor Port Usage

**Alert**: Port utilization > 60%

**Action**: Add IPs or increase min ports before exhaustion

### Manual IPs for Critical Services

**Pattern**: Manual NAT IPs for production

**Benefit**: Predictable IPs for allowlisting, stability

### Separate NAT per Environment

**Pattern**: Different NAT (and IPs) for dev/staging/prod

**Benefit**: Isolation, separate monitoring, blast radius

### Enable Logging (Selectively)

**Enable for**: Security-sensitive, troubleshooting

**Disable for**: High-traffic, cost-sensitive (logs expensive)

## Limitations

- Regional service (not global)
- Outbound only (no inbound)
- Cannot use for inter-VPC traffic (use peering/VPN)
- Max 64K ports per VM (configurable)
- Cannot change router once configured

## Migration Patterns

### From External IPs to Cloud NAT

**Process**:

1. Deploy Cloud NAT
2. Verify VM has private IP route to NAT
3. Remove external IPs from VMs
4. Test connectivity
5. Update monitoring

**Benefit**: Improved security, potentially lower cost

### From Proxy VMs to Cloud NAT

**Benefit**: Managed service, no maintenance, better availability

**Consideration**: Lose custom filtering (move to firewall rules)

## Exam Focus

### Core Concepts

- Outbound internet for private VMs
- No external IPs needed
- Managed service (highly available)
- Regional scope

### Use Cases

- Private VMs need internet access
- Security (no inbound)
- Centralized egress
- Static IP for allowlisting

### Architecture

- Attached to Cloud Router in VPC
- Subnet selection (all or selected)
- IP allocation (automatic or manual)
- Per-region deployment

### Port Exhaustion

- Understanding ports per IP (64K)
- Min ports per VM (64 default)
- Symptoms and solutions
- Monitoring and alerting

### Integration

- Works with Private Google Access
- GKE private clusters
- Load balancer backends
- Egress firewall rules still apply

### Security

- Outbound only (no inbound)
- Egress control and monitoring
- No increased attack surface
- Combine with firewall rules

### Cost

- Per GB processed + gateway hours
- Cheaper than external IPs at scale
- Egress charges apply
- Not charged for internal traffic
