# 1.1 – Shared Networking: Shared VPC, VPC Peering, Private Service Connect

## Quick Comparison

| Feature | Shared VPC | VPC Peering | Private Service Connect |
|---|---|---|---|
| **Scope** | Same org | Same or different org | Same or different org |
| **IP coordination** | Centralized (host project) | Both sides must have non-overlapping CIDRs | Not required (NAT-based) |
| **Transitive routing** | ✅ within Shared VPC | ❌ No transitive routing | ❌ |
| **Admin model** | Centralized (host project) | Peer-to-peer | Producer/consumer |
| **Use case** | Enterprise multi-project hub-and-spoke | Cross-project/org connectivity | Expose services or access Google APIs privately |

---

## Shared VPC

### Concepts

- **Host project**: owns the Shared VPC network (subnets, firewall rules, routes)
- **Service project**: attaches to host; creates resources in host subnets
- Centralized control of **network resources** — no service project can change subnets/routes/firewalls
- Supports **hub-and-spoke** architecture at scale (your PriceHubble model!)

### Key IAM Roles

| Role | Assigned to |
|---|---|
| `roles/compute.xpnAdmin` (Shared VPC Admin) | Org or folder admin; enables/disables Shared VPC |
| `roles/compute.networkAdmin` | Network admin in host project |
| `roles/compute.networkUser` | Service project SA — allows use of host subnets |
| `roles/compute.securityAdmin` | Manage firewall rules |

### Setup (gcloud)

```bash
# Enable host project
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Associate service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# Grant network user role to SA in service project
gcloud projects add-iam-policy-binding HOST_PROJECT_ID \
  --member="serviceAccount:SA@SERVICE_PROJECT.iam.gserviceaccount.com" \
  --role="roles/compute.networkUser"
```

### Exam Tips
- Linked service projects must be in the **same organization** (or same folder if folder-level admin)
- You can scope subnet access to specific SAs (subnet-level IAM)
- Host project admin controls all firewall rules — service projects cannot bypass them

---

## VPC Network Peering

### Concepts
- Connects two **separate VPC networks** — traffic uses internal IPs, same latency as intra-VPC
- Peering is **non-transitive**: A↔B and B↔C does NOT give A↔C connectivity
- Both sides must **independently accept** the peering (mutual configuration)
- CIDR ranges of peered VPCs **must not overlap**
- Can peer across **organizations** (unlike Shared VPC)

### Setup

```bash
# From VPC-A, peer to VPC-B
gcloud compute networks peerings create peer-a-to-b \
  --network=VPC_A \
  --peer-project=PROJECT_B \
  --peer-network=VPC_B \
  --export-custom-routes \
  --import-custom-routes

# From VPC-B, peer to VPC-A (both sides required)
gcloud compute networks peerings create peer-b-to-a \
  --network=VPC_B \
  --peer-project=PROJECT_A \
  --peer-network=VPC_A \
  --export-custom-routes \
  --import-custom-routes
```

### Route Exchange Options
- `--export-custom-routes` / `--import-custom-routes`: share static/dynamic routes
- `--export-subnet-routes-with-public-ip` / `--import-subnet-routes-with-public-ip`: share subnets with public IPs
- Peering does **not** exchange **firewall rules** — each VPC manages its own

### Exam Tips
- No transitive routing — common exam trap
- Firewall rules are NOT shared between peered networks
- Max **25 peering connections** per VPC network

---

## Private Service Connect (PSC)

### Concepts
- Lets **consumers** access **producer services** (Google APIs or custom services) via a private IP endpoint
- Traffic does NOT expose the entire peered network — only a single service IP (service-oriented)
- Uses **NAT** → no IP overlap constraints between consumer and producer
- Supports: Google APIs (googleapis.com), Cloud SQL, Pub/Sub, custom services, hybrid (on-prem) services

### Access Patterns

| Pattern | Use case |
|---|---|
| **PSC Endpoint** | Consumer creates endpoint in their VPC to reach a service |
| **PSC Backend** | PSC endpoint behind a Load Balancer (for custom services) |
| **PSC Interface** | Producer VPC initiates connections into consumer VPC (preview) |

### Google APIs via PSC

```bash
# Create a forwarding rule for googleapis.com
gcloud compute forwarding-rules create psc-googleapis \
  --network=VPC_NAME \
  --address=PSC_IP \
  --target-google-apis-bundle=all-apis \
  --region=REGION
```

### Custom Service via PSC
1. Producer creates **Service Attachment** (backed by an Internal Load Balancer)
2. Consumer creates **PSC Endpoint** pointing to the service attachment
3. Consumer accesses the service via a private IP — no VPN/peering needed

### Exam Tips
- PSC provides **explicit authorization** — consumers need producer approval
- Unlike VPC Peering, no need to coordinate IP ranges
- In **Shared VPC**: PSC endpoints can be deployed in service projects using host project subnets
- PSC does NOT replace Shared VPC or VPC Peering — they solve different problems

---

## Architecture: Hub-and-Spoke (Exam Favorite)

```
                    ┌─────────────────┐
                    │   Host Project  │
                    │  (Shared VPC)   │
                    │  Centralized NW │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
   ┌──────┴──────┐   ┌───────┴─────┐   ┌───────┴─────┐
   │  Service    │   │  Service    │   │  Service    │
   │  Project A  │   │  Project B  │   │  Project C  │
   │  (app team) │   │  (data team)│   │  (ML team)  │
   └─────────────┘   └─────────────┘   └─────────────┘
```

- Network admin in host project → centralized governance
- Service project admins → create VMs/GKE clusters only
- All outbound traffic routed through shared firewall/Cloud NAT in host project

---

## Multi-Project Monitoring & Logging

- Use a **centralized logging project**: route logs from all service projects via **log sinks** (Pub/Sub or BigQuery)
- Use **Metrics Scopes**: add monitored projects to a scoping project's Monitoring workspace
- **Cloud Monitoring**: create a scoping project, add all service projects as monitored projects
- Uptime checks, alerting policies defined in the scoping project apply across all monitored projects

```bash
# Add monitored project to a metrics scope
gcloud monitoring metrics-scopes create \
  --scoping-project=MONITORING_PROJECT \
  --monitored-project=SERVICE_PROJECT
```
