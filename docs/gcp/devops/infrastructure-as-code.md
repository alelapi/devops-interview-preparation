# 1.2 – Infrastructure as Code (IaC) & Infrastructure Management

## GCP-Native IaC Tools

### Infrastructure Manager (Infra Manager)
- GCP-managed service to deploy **Terraform configs** at scale
- Handles state storage, locking, and execution in a managed environment
- Tracks deployments as **resources** in GCP (auditable via Cloud Audit Logs)
- Use when you want **Terraform without managing state backends**

```bash
# Deploy infrastructure via Infrastructure Manager
gcloud infra-manager deployments apply \
  --project=PROJECT_ID \
  --location=us-central1 \
  --service-account=SA@PROJECT.iam.gserviceaccount.com \
  projects/PROJECT/locations/us-central1/deployments/my-deployment \
  --git-source-repo=https://github.com/org/repo \
  --git-source-directory=terraform/ \
  --git-source-ref=main
```

### Cloud Foundation Toolkit (CFT)
- Google-maintained **Terraform module library** implementing GCP best practices
- Covers: project factory, networking, GKE, IAM, logging, etc.
- Available at `github.com/GoogleCloudPlatform/cloud-foundation-toolkit`
- Use for **landing zone bootstrapping** and compliance-ready configs

### Config Connector
- Kubernetes operator that manages **GCP resources as Kubernetes CRDs**
- Define GCP infrastructure in YAML manifests → GitOps-native
- Works with GKE; reconciles desired state against GCP APIs

```yaml
# Example: GCS bucket via Config Connector
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-bucket
  namespace: config-connector
spec:
  location: US
  uniformBucketLevelAccess: true
```

### Helm
- Kubernetes package manager — deploy apps and infrastructure components to GKE
- Used in CI/CD to template and version K8s manifests
- GCP-specific Helm charts available for services like Config Connector, Cert Manager

---

## Terraform on GCP

### GCP Provider Setup

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  backend "gcs" {
    bucket = "my-tf-state-bucket"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = var.project_id
  region  = "us-central1"
}
```

### State Management Best Practices
- Store state in **GCS bucket** with versioning enabled
- Enable **state locking** (GCS backend uses Cloud Storage object locking)
- Use **separate state files** per environment (dev/staging/prod)
- Restrict GCS bucket access via IAM

### Key Terraform Patterns for GCP DevOps

| Pattern | Description |
|---|---|
| **Project Factory** | Automate project creation with consistent IAM, billing, APIs |
| **Workspace per env** | `terraform workspace select prod` — separate state per env |
| **Module composition** | Network module → GKE module → App module (layered) |
| **Data sources** | `data.google_project`, `data.google_container_cluster` for referencing existing resources |

```hcl
# GKE cluster example
resource "google_container_cluster" "primary" {
  name     = "prod-cluster"
  location = "us-central1"

  remove_default_node_pool = true
  initial_node_count       = 1

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }
}
```

---

## GitOps for Infrastructure

### GitOps Principles
1. **Declarative** — desired state stored in Git
2. **Versioned and immutable** — Git is the single source of truth
3. **Pulled automatically** — agents sync Git → actual state
4. **Continuously reconciled** — drift detection and auto-correction

### GCP GitOps Stack
```
Developer PR → GitHub/GitLab → Cloud Build trigger →
  Terraform plan (reviewed) → merge → Cloud Build apply →
  Infrastructure Manager / GKE cluster updated
```

### ArgoCD on GKE (your stack)
- Watches Git repo for K8s manifests
- Reconciles cluster state — auto-sync or manual approval
- **ApplicationSet**: manage many apps/clusters from one template
- Integrates with **Config Connector** for GCP resource management

---

## Making Infrastructure Changes: GCP Best Practices

1. **Always plan before apply** — review `terraform plan` output in PR
2. **Use CMEK** (Customer-Managed Encryption Keys) for sensitive infra
3. **Immutable infrastructure** — replace, don't patch (bake images with Packer)
4. **Separate pipelines for infra vs app** — different blast radius, approval flows
5. **Audit trail** — Infrastructure Manager + Cloud Audit Logs track every change
6. **Use service accounts with minimal IAM** for Terraform/Cloud Build — not owner role
7. **Lock provider versions** — avoid surprise breaking changes

---

## Automation with Scripting

### Python (google-cloud-* libraries)

```python
from google.cloud import compute_v1

def list_instances(project_id: str, zone: str):
    client = compute_v1.InstancesClient()
    return list(client.list(project=project_id, zone=zone))
```

### Go (cloud.google.com/go/*)

```go
import "google.golang.org/api/compute/v1"

svc, _ := compute.NewService(ctx)
instances, _ := svc.Instances.List(projectID, zone).Do()
```

### gcloud Scripting Tips

```bash
# Output as JSON for scripting
gcloud projects list --format=json | jq '.[].projectId'

# Use --filter for server-side filtering (faster)
gcloud compute instances list --filter="status=RUNNING AND zone:us-central1"

# Impersonate SA in scripts (no key file needed)
gcloud config set auth/impersonate_service_account SA@PROJECT.iam.gserviceaccount.com
```

---

## GCP Blueprints

- Pre-built, opinionated Terraform configurations for common architectures
- Available in: **Google Cloud Architecture Center** and **Cloud Foundation Fabric**
- Cover: GKE enterprise, landing zones, data platform, security foundation
- Use as starting point — customize rather than build from scratch

---

## Exam Tips

- **Infrastructure Manager** = GCP-managed Terraform execution (no self-hosted runners)
- **Config Connector** = GitOps-native GCP resource management via K8s CRDs
- **CFT** = opinionated Terraform modules following Google best practices
- Always separate **infra pipeline** (slow, needs approval) from **app pipeline** (fast, automated)
- Terraform state in **GCS** — use versioning + IAM; never store locally in CI
