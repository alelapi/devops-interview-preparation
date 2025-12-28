# VPC Peering

## Description

VPC Network Peering enables private RFC 1918 connectivity across two VPC networks, allowing resources to communicate using internal IP addresses regardless of whether they belong to the same project or organization. Peering is a decentralized approach where each VPC network remains under its own administrative control.

**Architecture**: Direct connection between two VPC networks creating a private connection path.

## Key Features

### Connectivity

- **Private IP Communication**: Resources communicate using internal IPs without traversing the public internet
- **Cross-Project/Organization**: Peering works across different projects and organizations
- **Bidirectional**: Traffic can flow in both directions once established
- **No Single Point of Failure**: Peering is not a physical connection but a logical configuration
- **Transitive Peering**: NOT supported - if VPC A peers with B, and B peers with C, A cannot reach C

### Performance & Security

- **Low Latency**: Private Google network backbone
- **No Bandwidth Bottleneck**: No aggregated bandwidth limit
- **Firewall Control**: Each VPC maintains its own firewall rules
- **No Encryption**: Traffic is private but not encrypted (use application-level encryption if needed)

### Administrative

- **Independent Management**: Each VPC admin controls their own network
- **Subnet Expansion**: Can expand subnet ranges after peering is established
- **Import/Export Custom Routes**: Optional custom route exchange

## Important Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Peering connections per VPC** | 25 | Default limit |
| **Subnet IP ranges** | Cannot overlap | Critical - overlapping IPs prevent peering |
| **Transitive peering** | Not supported | Must create direct peering for each connection |
| **Network tags** | Not shared | Tags don't cross peering boundaries |
| **Internal DNS** | Requires Cloud DNS peering | VPC peering doesn't automatically enable DNS resolution |
| **Peering group limit** | 25 VPCs in a peering group | All VPCs that peer to a common VPC |

## When to Use

### ✅ Use VPC Peering When:

1. **Multi-Project Architecture**
   - Different teams manage separate projects but need private connectivity
   - Hub and spoke architectures with limited spoke-to-spoke communication
   - Cost-effective private connectivity between projects

2. **Cross-Organization Collaboration**
   - Partner organizations need to connect resources privately
   - M&A scenarios where networks need quick integration
   - Service provider connecting to customer VPCs

3. **Simple Network Topologies**
   - Limited number of VPCs (under 25 per network)
   - No need for transitive routing
   - Each VPC maintains administrative independence

4. **Performance-Critical Workloads**
   - Low-latency requirements between VPCs
   - High-throughput applications without bandwidth constraints
   - Real-time data processing across projects

### ❌ Don't Use VPC Peering When:

1. **Centralized Security/Routing Required**
   - Use Shared VPC instead for centralized network administration
   - Need unified security policies across all networks

2. **Transitive Routing Needed**
   - Spoke-to-spoke communication in hub-and-spoke architecture
   - Use Cloud Router with VPN/Interconnect or redesign with Shared VPC

3. **Large-Scale Mesh Networks**
   - More than 15-20 VPCs needing full mesh connectivity
   - Quota exhaustion becomes challenging to manage
   - Consider Network Connectivity Center or Shared VPC

4. **Overlapping IP Ranges**
   - Peering is impossible with overlapping subnets
   - Would require NAT or network redesign

## Common Use Cases

**Hub and Spoke (Non-Transitive)**
```
Host Project (Hub) ←→ Spoke Project 1
                  ←→ Spoke Project 2
                  ←→ Spoke Project 3
```
Spokes cannot communicate with each other, only with the hub.

**Partner Integration**
```
Organization A VPC ←→ Organization B VPC
```
Private connectivity for data sharing, APIs, or service integration.

**Development/Production Isolation**
```
Production VPC ←→ Shared Services VPC ←→ Development VPC
```
Controlled connectivity to shared services (monitoring, logging, artifact registry).

## Configuration Considerations

1. **IP Planning**: Ensure no subnet overlap before creating peering
2. **Route Export/Import**: Configure custom route exchange if needed
3. **Firewall Rules**: Update rules to allow traffic from peered CIDR ranges
4. **DNS Setup**: Configure Cloud DNS peering for name resolution across VPCs
5. **Monitoring**: Set up VPC Flow Logs to track inter-VPC traffic
6. **IAM Permissions**: Requires `compute.networks.peer` permission on both networks

## Related Services

- **Shared VPC**: Alternative for centralized network management
- **Cloud DNS Peering**: For DNS resolution across peered VPCs
- **VPC Flow Logs**: Monitor traffic between peered networks
- **Cloud Router**: For dynamic routing in hybrid connectivity (not directly used with VPC peering)
