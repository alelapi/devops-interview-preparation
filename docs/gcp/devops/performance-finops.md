# Section 5 – Optimizing Performance and Cost

## 5.1 – Application Performance Monitoring

### Cloud Profiler
- Continuous **CPU and memory profiling** in production (low overhead ~0.5%)
- Languages: Go, Python, Java, Node.js, Ruby, PHP
- Identifies: hot functions, memory allocation hotspots
- No sampling configuration needed — automatic continuous profiling

```bash
# Python — add profiler to app
pip install google-cloud-profiler
```

```python
import googlecloudprofiler
googlecloudprofiler.start(
    service='my-service',
    service_version='1.0.0',
    verbose=3
)
```

### Active Assist
- AI-powered **recommendations** across GCP services
- Available for: cost, security, performance, reliability, manageability
- Access via: Cloud Console → Active Assist, or `gcloud recommender`

```bash
# View cost recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --location=us-central1 \
  --project=PROJECT

# View idle resource recommendations
gcloud recommender recommendations list \
  --recommender=google.compute.instance.IdleResourceRecommender \
  --location=global \
  --project=PROJECT
```

### Cloud Run Performance Monitoring
```bash
# Monitor cold starts
gcloud logging read 'resource.type="cloud_run_revision"' \
  --filter='jsonPayload.message:"Cold start"' --limit=100

# Key metrics
run.googleapis.com/request_latencies  # Request latency histogram
run.googleapis.com/container/startup_latency  # Cold start duration
run.googleapis.com/request_count  # Request count by response code
```

---

## 5.2 – FinOps Practices

## Observability Cost Management

Observability costs often grow unexpectedly. Key cost drivers:

| Component | Cost driver | Optimization |
|---|---|---|
| **Cloud Logging** | Log ingestion volume | Exclusion filters, reduce sampling |
| **Cloud Monitoring** | Custom metric samples | Reduce cardinality, drop unused metrics |
| **Cloud Trace** | Trace samples | Adjust sampling rate (0.1 = 10%) |
| **BigQuery** | Log export + query | Partition tables, use log-based metrics instead |

```bash
# Check metric ingestion volume (Metrics Management page)
gcloud monitoring metrics list --project=PROJECT | wc -l

# Exclude high-volume health check logs
gcloud logging sinks update _Default \
  --add-exclusion='name=healthchecks,filter=httpRequest.requestUrl="/health" AND httpRequest.status=200'

# Reduce trace sampling
# In OTel SDK:
from opentelemetry.sdk.trace.sampling import TraceIdRatioBased
sampler = TraceIdRatioBased(0.1)  # Sample 10% of traces
```

---

## Compute Pricing Models

### Sustained Use Discounts (SUD)
- **Automatic** — no purchase required
- Applied to Compute Engine VMs running > 25% of a billing month
- Incremental discounts: 25%→50%→75%→100% usage = 0%→10%→20%→30% discount
- **Does NOT apply to**: App Engine flexible, Dataflow, GPU accelerators (some), E2 machine series

### Committed Use Discounts (CUD)
- Commit to **1 or 3 years** of resource usage
- Two types:

| Type | Description | Discount |
|---|---|---|
| **Resource-based** | Commit to specific vCPU/RAM in a region/machine family | Up to 57% (1yr) / 70% (3yr) |
| **Spend-based (Flex CUD)** | Commit to minimum hourly spend | 28% (1yr) / 46% (3yr) |

- **Flex CUD** covers: Compute Engine VMs, GKE Standard, GKE Autopilot, Cloud Run
- CUDs **cannot be cancelled** — commit only to your stable baseline

```bash
# Purchase a CUD
gcloud compute commitments create my-commitment \
  --plan=12-month \
  --region=us-central1 \
  --resources=vcpu=100,memory=400GB
```

### Spot VMs
- Up to **91% cheaper** than on-demand
- Can be **preempted** with 30-second notice when GCP needs capacity
- **No maximum runtime** (unlike old preemptible VMs which had 24h max)
- Good for: batch jobs, CI/CD workers, ML training, data processing

```bash
# Create Spot VM
gcloud compute instances create my-spot \
  --machine-type=n2-standard-4 \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# Use Spot in GKE node pool
gcloud container node-pools create spot-pool \
  --cluster=my-cluster \
  --spot \
  --num-nodes=3
```

### When to use which

| Workload | Recommendation |
|---|---|
| Stable, long-running (prod) | Resource CUD (1yr) |
| Variable, mixed workloads | Flex CUD |
| Batch, CI/CD, ML training | Spot VMs |
| Short-lived variable (< monthly) | SUD (auto) |
| Dev/test | On-demand (no commitment) |

---

## Network Cost Optimization

### Network Tiers

| Tier | Description | Egress cost |
|---|---|---|
| **Premium** | Google's global backbone — low latency | Higher |
| **Standard** | ISP routing — acceptable latency | ~30% cheaper |

```bash
# Set network tier for instance
gcloud compute instances create my-vm \
  --network-tier=STANDARD  # or PREMIUM (default)
```

### Minimize Egress Costs
- Keep compute and storage in **same region** → free intra-region traffic
- Use **CDN** (Cloud CDN) for static content — reduce origin egress
- **Cloud Interconnect** for high-volume on-prem ↔ GCP: cheaper than internet egress
- Use **VPC Flow Logs sampling** < 100% — significant logging cost reduction

---

## GKE Cost Optimization

### Node Efficiency
```bash
# Right-size nodes with VPA
kubectl apply -f vertical-pod-autoscaler.yaml

# Use Autopilot for automatic right-sizing
gcloud container clusters create-auto my-cluster \
  --region=us-central1

# Enable cluster autoscaler with scale-down
gcloud container node-pools update pool \
  --cluster=my-cluster \
  --enable-autoscaling \
  --min-nodes=1 --max-nodes=20

# Spot node pool for non-critical workloads
gcloud container node-pools create spot-pool \
  --cluster=CLUSTER --spot
```

### Namespace-Level Cost Visibility
- Use **GKE Cost Allocation** (built-in): allocates costs to namespaces/labels
- Enable in cluster settings → available in BigQuery export
- Use **Kubecost** or custom Prometheus + BigQuery pipeline for detailed breakdowns

```bash
# Enable GKE cost allocation
gcloud container clusters update CLUSTER \
  --enable-cost-allocation
```

---

## Cloud Run Cost Optimization

```bash
# Set minimum instances (avoid cold start penalty at cost of always-on)
gcloud run services update myapp --min-instances=1

# Set concurrency high to maximize instance utilization
gcloud run services update myapp --concurrency=1000

# Use CPU always-on only if needed (default: CPU throttled when not serving)
gcloud run services update myapp --no-cpu-throttling  # Always-on CPU (more expensive)
```

---

## GCP Recommenders (Active Assist)

| Recommender | What it finds |
|---|---|
| `MachineTypeRecommender` | Over-provisioned VMs → suggest smaller |
| `IdleResourceRecommender` | VMs with <1% CPU usage — idle |
| `DiskIdleRecommender` | Unused persistent disks |
| `AddressIdleRecommender` | Unused static IPs ($7.20/month each) |
| `SnapshotRecommender` | Old snapshots costing money |
| `GKENodeRecommender` | Over/under-provisioned GKE node pools |
| `IAMRecommender` | Overly broad IAM bindings (security + cost) |
| `FirewallInsightRecommender` | Unused firewall rules |

```bash
# Apply a recommendation
gcloud recommender recommendations mark-claimed RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --location=us-central1-a \
  --project=PROJECT

# Mark applied
gcloud recommender recommendations mark-succeeded RECOMMENDATION_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --location=us-central1-a \
  --project=PROJECT
```

---

## Cost Monitoring and Budgets

```bash
# Create budget alert
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Prod Monthly Budget" \
  --budget-amount=5000USD \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=80 \
  --threshold-rule=percent=100,basis=FORECASTED_SPEND \
  --filter-projects=projects/prod-project-id
```

### Cost Allocation Labels
```bash
# Label resources for cost attribution
gcloud compute instances add-labels my-instance \
  --labels=team=payments,env=prod,cost-center=engineering

# Apply default labels via Org Policy
# constraints/compute.requireShieldedVm + labels via ConstraintCustom
```

---

## Summary: Key Cost Levers

| Priority | Action | Typical Savings |
|---|---|---|
| 🔴 High | Apply CUDs to stable compute baseline | 30-70% |
| 🔴 High | Use Spot VMs for batch/CI workloads | 60-91% |
| 🟡 Medium | Right-size instances (use Recommender) | 20-40% |
| 🟡 Medium | Delete idle resources (IPs, disks, snapshots) | Variable |
| 🟡 Medium | Reduce log ingestion (exclusions) | 20-50% of logging bill |
| 🟢 Low | Enable Cloud CDN for static assets | 30-50% of egress |
| 🟢 Low | Use Standard network tier where latency allows | ~30% of egress |

---

## Exam Tips

- **SUD** = automatic, no commitment; **CUD** = manual purchase, 1/3 year commitment
- Spot VMs = no max runtime (unlike old preemptible 24h limit) + 30s termination notice
- **Flex CUD** = spend-based commitment; covers GKE Autopilot + Cloud Run (not just VMs)
- Recommenders are **per-resource-type** and **per-location** — check correct recommender ID
- **GKE Cost Allocation** = built-in namespace/label cost breakdown → exports to BigQuery
- Cloud Profiler = production profiling with <0.5% overhead (safe in prod)
- Active Assist = umbrella term for all GCP recommenders + insights
- Idle static IPs cost ~$7.20/month each — small but adds up; use `AddressIdleRecommender`
