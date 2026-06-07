# 1.3 – CI/CD Architecture Stack on GCP

## Core GCP CI/CD Services

| Service | Role |
|---|---|
| **Cloud Build** | CI — build, test, containerize |
| **Artifact Registry** | Store container images, Helm charts, Maven/npm packages |
| **Cloud Deploy** | CD — managed progressive delivery to GKE, Cloud Run, Anthos |
| **Cloud Source Repos** | ⚠️ No longer available to new customers (June 2024) — use GitHub/GitLab |

---

## Cloud Build

### Key Concepts
- Serverless CI — runs **build steps** in Docker containers
- Config file: `cloudbuild.yaml` (or `Dockerfile` for simple builds)
- Build steps run **sequentially by default**; use `waitFor` for parallelism
- Each step runs as a separate container; share workspace via `/workspace`

### cloudbuild.yaml Structure

```yaml
steps:
  # Step 1: Build Docker image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-docker.pkg.dev/$PROJECT_ID/repo/app:$SHORT_SHA', '.']
    id: 'build'

  # Step 2: Run tests (parallel with build? no — depends on build)
  - name: 'gcr.io/cloud-builders/docker'
    args: ['run', '--rm', 'us-docker.pkg.dev/$PROJECT_ID/repo/app:$SHORT_SHA', 'npm', 'test']
    id: 'test'
    waitFor: ['build']

  # Step 3: Push image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-docker.pkg.dev/$PROJECT_ID/repo/app:$SHORT_SHA']
    waitFor: ['test']

substitutions:
  _ENV: 'staging'

options:
  logging: CLOUD_LOGGING_ONLY  # Required with custom SA
  machineType: 'E2_HIGHCPU_8'  # Larger machine for faster builds

timeout: '1200s'

serviceAccount: 'projects/$PROJECT_ID/serviceAccounts/build-sa@$PROJECT_ID.iam.gserviceaccount.com'
```

### Substitution Variables

| Variable | Value |
|---|---|
| `$PROJECT_ID` | GCP project ID |
| `$BUILD_ID` | Unique build ID |
| `$COMMIT_SHA` | Full Git commit SHA |
| `$SHORT_SHA` | First 7 chars of commit SHA |
| `$BRANCH_NAME` | Git branch name |
| `$TAG_NAME` | Git tag (if triggered by tag) |
| `$_CUSTOM_VAR` | User-defined substitutions |

### Build Triggers

```bash
# Create a trigger on GitHub push to main
gcloud builds triggers create github \
  --repo-name=my-app \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml \
  --service-account=SA@PROJECT.iam.gserviceaccount.com

# Pub/Sub trigger (e.g., on Artifact Registry push)
gcloud builds triggers create pubsub \
  --topic=projects/$PROJECT_ID/topics/gcr \
  --build-config=cloudbuild.yaml
```

**Trigger types:**
- Push to branch
- Push new tag
- Pull request (GitHub/GitLab)
- Manual
- Pub/Sub message
- Webhook (any external event)

### Private Pools
- Run builds in a **dedicated, isolated VPC** (not shared Google infra)
- Required for: VPC-SC environments, private GKE/Cloud Run endpoints, compliance
- Can be connected to your VPC via VPC peering

```bash
gcloud builds worker-pools create my-pool \
  --region=us-central1 \
  --worker-machine-type=e2-medium \
  --peered-network=projects/PROJECT/global/networks/VPC_NAME
```

---

## Artifact Registry

### Key Concepts
- Replacement for **Container Registry (GCR)** — more features, regional
- Supports: Docker, Helm, Maven, npm, Python (PyPI), Apt, Yum, Go
- **Vulnerability scanning**: automatic scanning of container images on push
- IAM-controlled access per repository

### Setup

```bash
# Create a Docker repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Production container images"

# Configure Docker auth
gcloud auth configure-docker us-central1-docker.pkg.dev

# Image naming format
# LOCATION-docker.pkg.dev/PROJECT/REPO/IMAGE:TAG
us-central1-docker.pkg.dev/my-project/my-repo/app:v1.0.0
```

### Cleanup Policies (cost control)

```bash
# Delete untagged images older than 30 days
gcloud artifacts repositories set-cleanup-policies my-repo \
  --location=us-central1 \
  --policy=cleanup-policy.json
```

```json
[{
  "name": "delete-old-untagged",
  "action": {"type": "Delete"},
  "condition": {
    "tagState": "untagged",
    "olderThan": "30d"
  }
}]
```

---

## Cloud Deploy

### Key Concepts
- Managed **CD pipeline** — defines a sequence of **targets** (dev → staging → prod)
- A **release** is a specific version of an app (container image + manifests)
- A **rollout** is the deployment of a release to a target
- Supports: GKE, GKE Autopilot, Cloud Run, Anthos/GDC

### Core Resources

```yaml
# delivery-pipeline.yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: my-app-pipeline
  location: us-central1
spec:
  serialPipeline:
    stages:
    - targetId: dev
    - targetId: staging
      profiles: [staging]
    - targetId: prod
      profiles: [prod]
      strategy:
        canary:
          runtimeConfig:
            kubernetes:
              gatewayServiceMesh:
                httpRoute: my-http-route
                service: my-svc
                deployment: my-deployment
          canaryDeployment:
            percentages: [25, 50, 75]
            verify: true
---
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
  location: us-central1
spec:
  requireApproval: true  # Manual gate for prod
  gke:
    cluster: projects/PROJECT/locations/us-central1/clusters/prod-cluster
```

### Creating a Release and Rollout

```bash
# Register pipeline and targets
gcloud deploy apply --file=delivery-pipeline.yaml --region=us-central1

# Create a release (snapshot of the app version)
gcloud deploy releases create release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1 \
  --images=app=us-central1-docker.pkg.dev/PROJECT/repo/app:SHA

# Promote to next stage
gcloud deploy releases promote \
  --release=release-001 \
  --delivery-pipeline=my-app-pipeline \
  --region=us-central1

# Approve a rollout (for requireApproval targets)
gcloud deploy rollouts approve ROLLOUT_NAME \
  --delivery-pipeline=my-app-pipeline \
  --release=release-001 \
  --region=us-central1
```

### Skaffold Integration
- Cloud Deploy uses **Skaffold** under the hood for rendering and deploying
- `skaffold.yaml` defines how to build images and deploy to Kubernetes
- Supports: `kubectl`, `helm`, `kustomize` deployers

```yaml
# skaffold.yaml
apiVersion: skaffold/v4beta6
kind: Config
build:
  artifacts:
  - image: app
    docker:
      dockerfile: Dockerfile
deploy:
  kubectl:
    manifests:
    - k8s/*.yaml
```

### Kustomize Integration
- Use `kustomize` overlays per environment (base + overlays/staging, overlays/prod)
- Cloud Deploy renders Kustomize before deploying

---

## Third-Party Tooling Integration

| Tool | Role in GCP CI/CD |
|---|---|
| **Git** (GitHub/GitLab) | Source of truth; triggers Cloud Build via webhooks |
| **Jenkins** | Trigger Cloud Build or deploy to GKE from Jenkins pipelines |
| **ArgoCD** | GitOps CD for GKE — pull-based, reconciles Git → cluster |
| **Packer** | Build VM images (GCE) for immutable infrastructure |
| **kpt** | Package management for Kubernetes configs — fetch, update, apply |

---

## CI/CD Security

- **Least privilege SA** for Cloud Build — only the permissions needed
- **Never store secrets in cloudbuild.yaml** — use Secret Manager
- **Artifact scanning** — enable vulnerability scanning in Artifact Registry
- **Binary Authorization** — enforce only signed images can be deployed (see Section 2.4)
- **VPC-SC** for sensitive pipelines — restrict data exfiltration
- Prefer **Workload Identity Federation** over SA keys for GitHub Actions → GCP

```yaml
# Access Secret Manager in Cloud Build
steps:
- name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    DB_PASS=$(gcloud secrets versions access latest --secret=db-password)
    echo "$$DB_PASS" | ./deploy.sh
```

---

## Exam Tips

- Cloud Build = CI (build + test); Cloud Deploy = CD (progressive delivery to targets)
- `cloudbuild.yaml` uses `waitFor` for parallelism; default is sequential
- Cloud Deploy **requires Skaffold** — know `skaffold.yaml` basics
- `requireApproval: true` on a Cloud Deploy target = manual gate
- Canary deployment in Cloud Deploy tracks **percentages** then **stable** phase
- Private Pools = builds inside your VPC (required for VPC-SC or private endpoints)
