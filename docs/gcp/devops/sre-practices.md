# Section 3 – Site Reliability Engineering (SRE) Practices

## 3.1 – Balancing Change, Velocity, and Reliability

### SLI / SLO / SLA / Error Budget

```
SLI → measures reliability
SLO → target for that measure
Error Budget → tolerance for unreliability (100% - SLO)
SLA → contract with users about SLO (with consequences for breach)
```

### SLI (Service Level Indicator)

A **quantitative measure** of a service's behavior from the user's perspective.

| SLI Type | Example |
|---|---|
| **Availability** | % of HTTP requests returning 2xx (success) |
| **Latency** | % of requests served in < 200ms |
| **Error rate** | % of requests returning 5xx |
| **Throughput** | requests per second |
| **Freshness** | % of data updated within 1 hour |
| **Durability** | % of data successfully stored and retrieved |
| **Correctness** | % of responses with correct data |

Good SLI formula:
```
SLI = good events / valid events × 100%
```

### SLO (Service Level Objective)

A **target value** for an SLI over a **compliance period**.

```
SLO: 99.9% availability over a 30-day rolling window
    → allows 43.8 minutes of downtime per month

SLO: 95% of requests served in < 300ms
    → 5% of requests may be slow

SLO: 99.99% availability
    → allows ~4.4 minutes downtime per month
```

**Aspirational vs. achievable SLOs:**
- Don't set SLOs tighter than needed — creates toil
- Don't set SLOs looser than users need — erodes trust
- Start with current baseline + slight improvement target

### Error Budget

```
Error Budget = 100% - SLO target
Example: SLO = 99.9% → Error Budget = 0.1% of 30 days = 43.2 minutes

Burn rate = rate of consuming error budget
  - Burn rate 1 = consuming exactly at budget rate
  - Burn rate 2 = consuming budget 2x faster than allowed
  - Fast burn (> 14.4x) = alert immediately
  - Slow burn (~1x over long window) = alert before budget exhaustion
```

**Error budget policy:**
- Budget healthy → dev teams free to ship features
- Budget 50% consumed → begin reliability focus
- Budget 0% (exhausted) → freeze non-critical deployments until next period

### SLA (Service Level Agreement)
- **Legal/contractual commitment** to users about SLOs
- SLA target is always **weaker than internal SLO** (buffer for incident response)
- Example: Internal SLO = 99.9%, SLA = 99.5%
- SLA breach → financial penalty, credit, etc.

---

### GCP Cloud Monitoring SLO Setup

```bash
# Create SLO via gcloud (request-based)
gcloud monitoring slos create \
  --service=projects/PROJECT/services/my-service \
  --display-name="Availability SLO 99.9%" \
  --request-based-sli='{
    "goodTotalRatio": {
      "goodServiceFilter": "metric.type=\"loadbalancing.googleapis.com/https/request_count\" metric.labels.response_code_class=\"2xx\"",
      "totalServiceFilter": "metric.type=\"loadbalancing.googleapis.com/https/request_count\""
    }
  }' \
  --goal=0.999 \
  --rolling-period-days=30
```

### Burn Rate Alerting

```
Fast burn alert:   burn rate > 14.4×, last 1 hour  → page immediately
Slow burn alert:   burn rate > 1×, last 6 hours    → ticket/investigate
```

```bash
# Alert on burn rate
gcloud monitoring policies create \
  --display-name="Error Budget Fast Burn" \
  --condition-filter='select_slo_burn_rate("projects/PROJECT/services/svc/serviceLevelObjectives/slo")' \
  --condition-threshold-value=14.4 \
  --comparison=COMPARISON_GT \
  --condition-duration=3600s
```

---

## 3.2 – Managing Service Lifecycle

### Service Management Stages

| Stage | Key Activities |
|---|---|
| **Planning** | Define SLOs, design for reliability, capacity planning |
| **Deployment** | Progressive rollout, canary, feature flags |
| **Maintenance** | Patching, upgrades, cert rotation, dependency updates |
| **Retirement** | Drain traffic, deprecation notices, data migration |

---

### Capacity Planning

#### Quotas and Limits
- GCP enforces **quotas** (soft limits, requestable) and **hard limits**
- Check quotas: Cloud Console → IAM & Admin → Quotas
- Request quota increase: `gcloud compute project-info set-usage-export-bucket`

```bash
# Check current quota usage
gcloud compute regions describe us-central1 --format="yaml(quotas)"

# Request quota increase (via console or support ticket for large increases)
gcloud compute project-info add-metadata \
  --metadata quota-request="CPUS:500"
```

#### Reservations
- Reserve specific VM capacity in a zone — guaranteed availability
- Use when: predictable demand spikes, capacity-sensitive workloads

```bash
gcloud compute reservations create my-reservation \
  --machine-type=n2-standard-8 \
  --vm-count=10 \
  --zone=us-central1-a
```

#### Dynamic Workload Scheduler (DWS)
- Request **batch GPU/TPU capacity** for future time windows
- GCP schedules the resources when capacity is available
- Use for: ML training jobs that don't need immediate, persistent VMs

---

### Autoscaling

#### Managed Instance Groups (MIGs) — Compute Engine

```bash
gcloud compute instance-groups managed set-autoscaling my-mig \
  --min-num-replicas=2 \
  --max-num-replicas=20 \
  --target-cpu-utilization=0.7 \
  --cool-down-period=60
```

Scaling signals: CPU, HTTP load balancing, Pub/Sub queue depth, custom metrics

#### GKE Autoscaling

| Autoscaler | Scales | Based on |
|---|---|---|
| **HPA** (Horizontal Pod Autoscaler) | Pod replicas | CPU, memory, custom metrics |
| **VPA** (Vertical Pod Autoscaler) | Pod resource requests | Historical usage |
| **Cluster Autoscaler** | Nodes in node pools | Pending pods (unschedulable) |
| **KEDA** (event-driven) | Pod replicas | Pub/Sub, HTTP, custom events |

```yaml
# HPA example
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

#### Cloud Run Autoscaling
- Scales to zero by default; set `--min-instances` to avoid cold starts
- `--max-instances` to cap cost
- Scale-up based on concurrent requests per instance

```bash
gcloud run services update myapp \
  --min-instances=2 \
  --max-instances=100 \
  --concurrency=80
```

---

## 3.3 – Mitigating Incident Impact

### Traffic Management During Incidents

#### Draining / Redirecting Traffic

```bash
# GKE: Remove pod from service without downtime
kubectl drain NODE_NAME --ignore-daemonsets --delete-emptydir-data

# GKE: Cordon node (no new pods scheduled)
kubectl cordon NODE_NAME

# Cloud Run: Redirect 100% traffic to stable revision
gcloud run services update-traffic myapp --to-revisions=stable=100

# Load Balancer: Update backend service weight
gcloud compute backend-services update my-backend \
  --global \
  --update-custom-request-headers="..."
```

#### Adding Capacity

```bash
# GKE: Scale deployment immediately
kubectl scale deployment app --replicas=20

# GKE: Resize node pool
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --num-nodes=10

# MIG: Set target size
gcloud compute instance-groups managed resize my-mig --size=20
```

### Rollback Strategies

| Platform | Rollback Method |
|---|---|
| **GKE Deployment** | `kubectl rollout undo deployment/app` |
| **Cloud Run** | `gcloud run services update-traffic --to-revisions=PREV=100` |
| **Cloud Deploy** | `gcloud deploy rollouts rollback ROLLOUT` |
| **GKE node pool** | Blue/green node pool switch |
| **Terraform** | `git revert` + apply previous state |

### Incident Response Process (SRE Model)
1. **Detect** — alert fires; on-call paged
2. **Mitigate first** — restore service before root-causing (rollback, add capacity, redirect traffic)
3. **Communicate** — update status page, stakeholders
4. **Investigate** — root cause analysis while service is stable
5. **Resolve** — permanent fix
6. **Post-mortem** — blameless, action items to prevent recurrence

---

## Availability Table (Nines)

| Nines | Availability | Downtime/month | Downtime/year |
|---|---|---|---|
| **99%** (2 nines) | 99% | ~7.3 hours | ~3.65 days |
| **99.5%** | 99.5% | ~3.6 hours | ~1.83 days |
| **99.9%** (3 nines) | 99.9% | ~43.8 min | ~8.76 hours |
| **99.95%** | 99.95% | ~21.9 min | ~4.38 hours |
| **99.99%** (4 nines) | 99.99% | ~4.4 min | ~52.6 min |
| **99.999%** (5 nines) | 99.999% | ~26 sec | ~5.26 min |

---

## Toil

**Definition (SRE):** Repetitive, manual, automatable work that scales linearly with service growth and produces no lasting value.

Signs of toil: manual steps in deployments, manual alert triage, manual scaling, manual cert rotation.

**Goal:** Keep toil < 50% of SRE team time. Remainder = engineering work that reduces future toil.

---

## Exam Tips

- SLI = measurement; SLO = target; SLA = contract; Error Budget = tolerance
- Error budget = **100% - SLO%** not SLO - actual uptime
- **Burn rate** > 14.4x = alert immediately (fast burn); < 1x = no alert needed
- SLA target should always be **weaker** than internal SLO
- Canary deployment = reduces blast radius = respects error budget
- Cloud Run scales to zero; use `--min-instances` for latency-sensitive services
- GKE Cluster Autoscaler scales **nodes**; HPA scales **pods** — both may be needed
- `kubectl drain` = graceful pod eviction; `kubectl cordon` = prevent scheduling only
