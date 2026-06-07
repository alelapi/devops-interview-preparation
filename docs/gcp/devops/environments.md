# 1.4 – Managing Multiple Environments & 1.5 – Secure Cloud Dev Environments

---

# 1.4 – Managing Multiple Environments

## Environment Strategy

```
Typical pipeline:
dev → staging → (canary/pre-prod) → production

Each environment = separate GCP project (recommended)
```

### Why Separate Projects per Environment?
- Independent **IAM policies** — devs can't touch prod
- Independent **billing** — track costs per env
- Independent **Org Policies** — stricter constraints in prod
- Blast radius containment — prod outage can't cascade from staging

---

## Ephemeral Environments

- Short-lived environments spun up per PR/branch, torn down after merge
- Use cases: PR previews, integration testing, load testing
- Implementation on GCP:

```bash
# Cloud Build step: create ephemeral namespace
- name: 'gcr.io/cloud-builders/kubectl'
  args:
  - apply
  - -f
  - k8s/
  - -n
  - preview-$SHORT_SHA
  env:
  - 'CLOUDSDK_CONTAINER_CLUSTER=dev-cluster'

# Cloud Build step: teardown after tests
- name: 'gcr.io/cloud-builders/kubectl'
  args: ['delete', 'namespace', 'preview-$SHORT_SHA']
```

- **Cloud Run** is ideal for ephemeral envs — per-revision traffic splitting, no cluster overhead
- Use **labels** (`env=preview`, `pr=123`) for cost attribution and cleanup

---

## Configuration and Policy Management

### Config per Environment

| Method | Description |
|---|---|
| **Kustomize overlays** | Base + env-specific patches; native in Cloud Deploy |
| **Helm values files** | `values-prod.yaml`, `values-staging.yaml` |
| **Secret Manager** | Per-env secrets with separate IAM bindings |
| **ConfigMap / Env vars** | Runtime config injection (non-sensitive) |

```
k8s/
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays/
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patch-replicas.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch-replicas.yaml
```

### Org Policies per Environment Folder

```yaml
# In prod folder: deny public IPs
constraint: constraints/compute.vmExternalIpAccess
listPolicy:
  deniedValues:
    - projects/prod-project/zones/*/instances/*

# In dev folder: allow all for flexibility
constraint: constraints/compute.vmExternalIpAccess
booleanPolicy:
  allowed: true
```

---

## GKE Fleet Management

### Fleet Concepts
- A **Fleet** is a logical grouping of GKE clusters for unified management
- Enables: consistent policy, multi-cluster services, Config Management, mesh

```bash
# Register a cluster to a fleet
gcloud container fleet memberships register my-cluster \
  --gke-cluster=us-central1/my-cluster \
  --enable-workload-identity

# List fleet members
gcloud container fleet memberships list
```

### Fleet Features for Multi-Env

| Feature | Use |
|---|---|
| **Config Management (ACM)** | Sync policies/configs from Git to all fleet clusters |
| **Policy Controller** | Enforce OPA/Gatekeeper policies across fleet |
| **Multi-cluster Ingress** | Route traffic across clusters/regions |
| **Multi-cluster Services (MCS)** | Service discovery across clusters |
| **Service Mesh (Cloud Service Mesh)** | Istio-based mesh across fleet |

```yaml
# ACM fleet config (apply to all clusters)
apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
spec:
  gitSync:
    syncRepo: https://github.com/org/policy-configs
    syncBranch: main
    syncDir: clusters/
```

---

## Safe Patching and Upgrading

### GKE Upgrade Strategies

| Strategy | Description |
|---|---|
| **Surge upgrades** | Add extra nodes before removing old — zero downtime |
| **Blue/green node pools** | Create new node pool, migrate workloads, delete old |
| **Release channels** | Rapid / Regular / Stable — control upgrade cadence |

```bash
# Upgrade control plane
gcloud container clusters upgrade CLUSTER_NAME \
  --master \
  --cluster-version=1.29

# Upgrade node pool with surge
gcloud container node-pools update POOL_NAME \
  --cluster=CLUSTER_NAME \
  --max-surge-upgrade=1 \
  --max-unavailable-upgrade=0
```

### GKE Release Channels

| Channel | Lag from GA | Use case |
|---|---|---|
| **Rapid** | ~0 weeks | Latest features, testing |
| **Regular** | ~2-3 months | Default, balanced |
| **Stable** | ~5-6 months | Production, conservative |

### Node Auto-Upgrade
- Enabled by default — GKE auto-upgrades nodes within release channel
- Respects **maintenance windows** (`maintenanceWindow`) and **maintenance exclusions**

```bash
# Set maintenance window
gcloud container clusters update CLUSTER_NAME \
  --maintenance-window-start="2024-01-01T02:00:00Z" \
  --maintenance-window-end="2024-01-01T06:00:00Z" \
  --maintenance-window-recurrence="FREQ=WEEKLY;BYDAY=SA,SU"
```

---

# 1.5 – Secure Cloud Development Environments

## Cloud Workstations

- Fully managed **cloud-based development environments** on GCP
- Runs as a container on a managed VM in your VPC
- Advantages over local dev: consistent config, no local credential risk, access control via IAM

```bash
# Create a workstation cluster
gcloud workstations clusters create my-cluster \
  --region=us-central1 \
  --network=projects/PROJECT/global/networks/VPC \
  --subnetwork=projects/PROJECT/regions/us-central1/subnetworks/SUBNET

# Create a workstation config (defines container image, machine type)
gcloud workstations configs create my-config \
  --cluster=my-cluster \
  --region=us-central1 \
  --machine-type=e2-standard-4 \
  --container-predefined-image=codeoss \
  --idle-timeout=7200s

# Create a workstation for a user
gcloud workstations create my-ws \
  --cluster=my-cluster \
  --config=my-config \
  --region=us-central1
```

### Custom Images
```bash
# Dockerfile for custom workstation
FROM us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest

# Install custom tooling
RUN apt-get install -y kubectl terraform
COPY custom-config /etc/workstation/
```

### Security Features
- Container runs with restricted privileges (no root by default)
- VPC isolation — no public IP required
- **Cloud Workstations config** enforces: encryption keys (CMEK), idle timeout, boot disk size
- IAM controls who can create/start workstations
- Audit logging via Cloud Audit Logs

---

## Cloud Shell

- **Temporary** browser-based shell with pre-installed GCP tooling
- Ephemeral VM — not for persistent development
- 5GB persistent home directory (`$HOME`)
- Pre-installed: gcloud, kubectl, terraform, docker, git, python, go
- Good for: quick tasks, demos, learning — NOT for production pipelines

---

## Gemini AI Assist Tools (Exam Topic)

| Tool | Purpose |
|---|---|
| **Gemini Code Assist** | AI code completion and generation in IDEs (VS Code, JetBrains, Cloud Workstations) |
| **Gemini Cloud Assist** | AI for cloud operations — explain errors, suggest fixes, analyze logs/metrics |
| **Gemini CLI** | CLI-based AI assistant for terminal tasks |

### Gemini Cloud Assist in Operations
- Available in **Cloud Logging** (explain log entries, suggest queries)
- Available in **Cloud Monitoring** (interpret metrics, suggest alerts)
- Available in **Cloud Trace** (analyze trace anomalies)
- Helps with: incident investigation, query writing, metric interpretation

---

## Exam Tips

- **Separate GCP projects per environment** — not just namespaces
- **Kustomize overlays** = the GCP-native way to manage per-env config in Cloud Deploy
- **GKE Fleet** = manage many clusters as one unit (ACM, Policy Controller, mesh)
- **Release channels** control when GKE upgrades happen — use Stable for prod
- **Cloud Workstations** = secure, consistent, cloud-native dev env — no local creds
- **Gemini Cloud Assist** = AI assistant in Cloud Console for ops tasks — appears in logging/monitoring/trace
- Maintenance windows in GKE = control when auto-upgrades run (not if they run)
