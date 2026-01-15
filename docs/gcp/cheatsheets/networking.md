# Networking Essentials

## 🎯 Heavy Hitters (High Frequency)

### 1. **Shared VPC vs VPC Peering**
- **Shared VPC**: One host project shares subnets with service projects
  - **When**: Centralized network admin, IAM control, shared resources
  - **Benefit**: Single firewall rules, centralized logging
  - **Exam Clue**: "Central IT team manages network" → Shared VPC
  
- **VPC Peering**: Connect two VPCs (can be across orgs)
  - **When**: Connect different organizations, no transitive routing
  - **Limitation**: NOT transitive (A→B→C doesn't work)
  - **Exam Clue**: "Connect VPCs across organizations" → Peering

### 2. **Cloud VPN vs Interconnect**
- **Cloud VPN (HA VPN)**: Encrypted IPsec tunnel over internet
  - **SLA**: 99.99% with HA VPN (two tunnels)
  - **Bandwidth**: Up to 3 Gbps per tunnel
  - **Cost**: Lower, good for < 10 Gbps
  - **Exam Clue**: "Quick setup, encrypted, budget-friendly" → HA VPN

- **Dedicated Interconnect**: Direct physical connection (10/100 Gbps)
  - **SLA**: 99.99% or 99.9% depending on topology
  - **When**: Need > 10 Gbps, low latency, predictable performance
  - **Exam Clue**: "High bandwidth, colocation facility" → Dedicated

- **Partner Interconnect**: Through service provider (50 Mbps - 50 Gbps)
  - **When**: Can't reach Google colocation, need < 10 Gbps
  - **Exam Clue**: "Not near Google facility" → Partner Interconnect

### 3. **Private Google Access & Private Service Connect**
- **Private Google Access (PGA)**: VMs without public IP access Google APIs
  - **Enable**: Subnet-level setting
  - **Use Case**: Access GCS, BigQuery from private VMs
  - **Exam Clue**: "No public IP, access Cloud Storage" → PGA

- **Private Service Connect (PSC)**: Private endpoint for Google/third-party services
  - **Use Cases**: Access Cloud SQL, AlloyDB, partner SaaS privately
  - **Benefit**: Service appears as internal IP in your VPC
  - **Exam Clue**: "Private connection to managed service" → PSC

---

## 🌐 Core VPC Concepts

### **VPC Basics**
- **Global Resource**: Subnets are regional
- **Subnet IP Ranges**: Can expand (can't shrink), auto-mode or custom
- **RFC 1918 Private IPs**: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- **Default VPC**: Auto-created, one subnet per region (auto-mode)

### **Firewall Rules**
- **Default**: Implied deny ingress, implied allow egress
- **Priority**: 0-65535 (lower number = higher priority)
- **Direction**: Ingress or egress
- **Target**: Tags, service accounts, or all instances
- **Exam Tip**: Use service accounts for dynamic workloads (not tags)

### **Routes**
- **System Routes**: Automatically created (default 0.0.0.0/0 to internet)
- **Custom Routes**: Static or dynamic (BGP via Cloud Router)
- **Priority**: 0-65535
- **Exam Clue**: "Route traffic through appliance" → Custom static route

---

## 🔗 Connectivity Options

### **Hybrid Connectivity Decision Tree**
```
Need < 3 Gbps, encrypted           → HA VPN
Need > 10 Gbps, in colocation      → Dedicated Interconnect  
Need 50 Mbps - 50 Gbps, no colo    → Partner Interconnect
Need redundancy across regions     → Multiple Interconnects + Cloud Router
Need backup for Interconnect       → HA VPN as backup path
```

### **Cloud VPN Types**
- **Classic VPN**: Legacy, single tunnel, 99.9% SLA (avoid on exam)
- **HA VPN**: Two tunnels, 99.99% SLA, recommended
  - **On-prem setup**: Need two on-prem gateways for 99.99% SLA
  - **Regional resource**: Create one per region

### **Cloud Router**
- **What**: Enables dynamic routing (BGP) between GCP and on-prem
- **Required For**: HA VPN, Interconnect, dynamic route updates
- **ASN**: Each side needs unique Autonomous System Number
- **Exam Clue**: "Automatically learn new routes" → Cloud Router

---

## 🔐 Network Security

### **Cloud Armor**
- **What**: DDoS protection + WAF (Web Application Firewall)
- **Works With**: Global external HTTP(S) load balancers
- **Features**: 
  - Rate limiting
  - Geo-based blocking
  - OWASP Top 10 protection
  - Custom rules with CEL expressions
- **Exam Clue**: "DDoS protection, WAF, rate limiting" → Cloud Armor

### **Hierarchical Firewall Policies**
- **What**: Organization/folder-level firewall rules
- **Priority**: Org policies override VPC rules
- **Use Case**: Enforce security standards across all projects
- **Exam Clue**: "Centrally enforce firewall rules" → Hierarchical Firewall

### **VPC Service Controls**
- **What**: Security perimeter around Google Cloud services
- **Prevents**: Data exfiltration, unauthorized access
- **Use With**: GCS, BigQuery, Vertex AI, etc.
- **Exam Clue**: "Prevent data exfiltration from services" → VPC-SC

---

## 📡 Load Balancers (Critical!)

### **External vs Internal**
- **External**: Internet-facing (global or regional)
- **Internal**: Within VPC only (regional or cross-region)

### **Global vs Regional**
- **Global**: Anycast IP, route to nearest healthy backend
- **Regional**: Single region, lower latency for regional traffic

### **Decision Matrix**
| Traffic Type | Protocol | Scope | Load Balancer |
|-------------|----------|-------|---------------|
| External HTTP(S) | L7 | Global | **External HTTP(S) LB** (recommended) |
| External HTTP(S) | L7 | Regional | External regional HTTP(S) LB |
| External TCP/UDP | L4 | Global | External TCP/UDP Proxy LB |
| External TCP/UDP | L4 | Regional | External passthrough network LB |
| Internal HTTP(S) | L7 | Regional/Cross-region | **Internal HTTP(S) LB** |
| Internal TCP/UDP | L4 | Regional | **Internal TCP/UDP LB** |

### **Key Features**
- **Cloud CDN**: Works with external HTTP(S) LB (global)
- **SSL Offloading**: Terminate SSL at load balancer
- **Health Checks**: TCP, HTTP, HTTPS (configure properly!)
- **Backend Services**: Group of backends (instance groups, NEGs)

---

## 🚀 Advanced Networking

### **Cloud NAT**
- **What**: Managed NAT gateway for outbound internet access
- **Use Case**: VMs without public IPs access internet
- **Regional Resource**: Configure per region
- **Logging**: Can log NAT translations
- **Exam Clue**: "Private VMs need internet access" → Cloud NAT

### **Network Endpoint Groups (NEGs)**
- **What**: Collection of endpoints (IPs, serverless, internet)
- **Types**:
  - **Zonal NEG**: GCE instances
  - **Internet NEG**: External endpoints
  - **Serverless NEG**: Cloud Run, Cloud Functions, App Engine
  - **Hybrid NEG**: On-prem endpoints
- **Exam Clue**: "Load balance to Cloud Run" → Serverless NEG

### **Cloud CDN**
- **What**: Content Delivery Network (cache static content)
- **Works With**: External HTTP(S) load balancer (global)
- **Cache**: Based on HTTP headers, query parameters
- **Benefit**: Reduce latency, lower egress costs
- **Exam Clue**: "Serve static content globally" → Cloud CDN

### **Private Service Connect (PSC)**
- **Producer**: Service provider (Google or third-party)
- **Consumer**: Your VPC
- **Use Cases**:
  - Access Cloud SQL privately
  - Consume partner SaaS
  - Publish your own services
- **Benefit**: No VPC peering, no public IPs

---

## 🎓 Exam Decision Trees

### **VPC Connectivity**
```
Connect VPCs in same org           → Shared VPC (if centralized) or Peering
Connect VPCs across orgs           → VPC Peering
Connect on-prem to GCP             → VPN or Interconnect
Need transitive routing            → Use VPN/Interconnect (NOT Peering)
```

### **Hybrid Connectivity**
```
Budget-friendly, < 3 Gbps          → HA VPN
High bandwidth (> 10 Gbps)         → Dedicated Interconnect
No access to colocation            → Partner Interconnect
Need encryption                    → HA VPN (Interconnect is NOT encrypted)
Need lowest latency                → Dedicated Interconnect
```

### **Internet Access**
```
VM needs public IP                 → Ephemeral or static external IP
VM needs outbound only             → Cloud NAT
VM needs Google API access         → Private Google Access
VM needs no internet               → No external IP, no Cloud NAT
```

### **Load Balancing**
```
Global HTTP(S) traffic             → External HTTP(S) LB (global)
Regional HTTP(S) traffic           → External HTTP(S) LB (regional)
Internal HTTP(S) traffic           → Internal HTTP(S) LB
TCP/UDP, need global anycast       → TCP/UDP Proxy LB
TCP/UDP, regional only             → Network passthrough LB
Load balance to Cloud Run          → External HTTP(S) LB + Serverless NEG
```

---

## ⚡ Quick Reminders

### **Bandwidth Limits**
- **VPC Peering**: No bandwidth limit (same as internal)
- **HA VPN**: ~3 Gbps per tunnel (can use multiple)
- **Dedicated Interconnect**: 10 Gbps or 100 Gbps per connection
- **Partner Interconnect**: 50 Mbps - 50 Gbps

### **SLAs**
- **HA VPN**: 99.99% (two tunnels, two gateways)
- **Classic VPN**: 99.9% (deprecated)
- **Dedicated Interconnect**: 99.99% or 99.9% (depends on topology)
- **Shared VPC**: Same as regular VPC

### **Pricing Tips**
- **Egress**: Expensive! Use Cloud CDN, regional architecture
- **Ingress**: Usually free
- **VPC Peering**: No bandwidth charges between VPCs
- **Interconnect**: Lower egress costs than internet

### **Common Gotchas**
- **VPC Peering**: NOT transitive
- **Shared VPC**: Can't share with VPCs outside org
- **Interconnect**: NOT encrypted (use VPN on top if needed)
- **Cloud Armor**: Only works with HTTP(S) load balancer
- **Cloud CDN**: Only works with external HTTP(S) load balancer

---

## 🔍 Troubleshooting Quick Checks

**Can't reach Google APIs from private VM:**
- ✅ Check Private Google Access enabled on subnet
- ✅ Check route to 199.36.153.8/30 exists
- ✅ Check firewall allows egress to APIs

**VPN tunnel down:**
- ✅ Check IKE/IPsec settings match on both sides
- ✅ Check BGP session status (if using Cloud Router)
- ✅ Verify pre-shared key matches
- ✅ Check firewall allows UDP 500, 4500 and ESP

**Load balancer not routing:**
- ✅ Check health checks passing
- ✅ Check backend service has healthy instances
- ✅ Check firewall allows health check ranges (35.191.0.0/16, 130.211.0.0/22)
- ✅ Verify forwarding rule and target proxy configured

**Can't SSH to VM:**
- ✅ Check firewall allows port 22 from your IP (or IAP range: 35.235.240.0/20)
- ✅ Check VM has external IP (or use IAP tunnel)
- ✅ Check OS login enabled if using that
- ✅ Verify routes allow traffic

---

## 📚 Pro Tips for Exam

1. **Read carefully**: "Global" vs "Regional", "Internal" vs "External"
2. **Cost optimization**: Always consider egress costs
3. **High availability**: Always prefer HA VPN over Classic VPN
4. **Security**: Use service accounts for firewall rules, not network tags
5. **Transitive routing**: VPC Peering doesn't support it, but VPN/Interconnect + Cloud Router does
6. **Load balancer**: Match the scope (global vs regional) to your needs
7. **Interconnect**: Remember it's NOT encrypted by default
8. **Cloud Armor**: Only for HTTP(S) load balancers, not for L4

---

## 🎯 Memorization Shortcuts

**HTTPS LB with CDN and DDoS protection:**
External HTTP(S) LB (global) + Cloud CDN + Cloud Armor

**Private VM needs internet:**
Cloud NAT (no public IP on VM)

**Private VM needs Google APIs:**
Private Google Access (subnet setting)

**Connect on-prem < 3 Gbps:**
HA VPN + Cloud Router

**Connect on-prem > 10 Gbps:**
Dedicated Interconnect + Cloud Router

**Central IT manages network:**
Shared VPC (host + service projects)

**Load balance to Cloud Run:**
External HTTP(S) LB + Serverless NEG

**Prevent data exfiltration:**
VPC Service Controls

**DDoS + WAF protection:**
Cloud Armor (requires HTTP(S) LB)