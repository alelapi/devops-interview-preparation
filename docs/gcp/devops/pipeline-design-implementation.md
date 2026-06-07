# 2.1 & 2.2 – Designing & Implementing CI/CD Pipelines

## Full GCP CI/CD Pipeline Overview

```
Code Push → Cloud Build Trigger → Build + Test → Push to Artifact Registry
    → Cloud Deploy Release → Rollout to Dev → Promote to Staging
    → Verify/Canary → Manual Approval → Rollout to Prod
    → Cloud Audit Logs / Cloud Monitoring (success metrics)
```

---

## 2.1 – Designing Pipelines

### CI Pipeline Design Principles

1. **Fail fast** — lint and unit tests first, expensive tests last
2. **Build once, deploy many** — one image, promoted through envs via tags
3. **Immutable artifacts** — never rebuild; tag by `$COMMIT_SHA`
4. **Parallel steps** — use `waitFor` in Cloud Build for parallelism
5. **Cache layers** — use Cloud Build caching to speed up builds

### CD Pipeline Design with Cloud Deploy

```yaml
# delivery-pipeline.yaml — serial progression
spec:
  serialPipeline:
    stages:
    - targetId: dev           # Auto-deploy on every release
    - targetId: staging       # Promote after dev passes
      profiles: [staging]
    - targetId: prod          # Requires approval + canary
      profiles: [prod]
      strategy:
        canary:
          canaryDeployment:
            percentages: [10, 50]
            verify: true      # Run verify job before advancing
```

### Approval Flows

```yaml
# Target with required approval
apiVersion: deploy.cloud.google.com/v1
kind: Target
metadata:
  name: prod
spec:
  requireApproval: true  # Reviewer must approve in UI or gcloud
  gke:
    cluster: projects/PROJECT/locations/us-central1/clusters/prod
```

```bash
# Approve pending rollout
gcloud deploy rollouts approve projects/P/locations/us-central1/deliveryPipelines/app/releases/r1/rollouts/r1-to-prod-0001
```

### Multi-Cloud / Hybrid Deployments

- Cloud Deploy supports **Anthos** targets (GKE Attached clusters — EKS, AKS, on-prem)
- Use **GitHub Actions + Workload Identity Federation** for cross-cloud deployments
- Artifact Registry serves images to any environment — no GCR dependency

---

## Deployment Strategies

### Rolling Update (default K8s)
- Replace pods one-by-one; controlled by `maxSurge` + `maxUnavailable`
- **Zero downtime** if `maxUnavailable: 0`
- Rollback: `kubectl rollout undo deployment/app`

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

### Blue/Green
- Two identical environments (blue = current, green = new)
- Switch traffic instantly via **Service selector** update or Load Balancer backend swap
- **Instant rollback** — switch selector back to blue
- Cost: 2x resources while green is live

```yaml
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  selector:
    matchLabels:
      app: myapp
      version: green
---
# Switch traffic: update service selector
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'
```

### Canary
- Route X% of traffic to new version; gradually increase
- Requires: service mesh (Istio/Cloud Service Mesh) OR Gateway API OR Ingress traffic splitting
- **Cloud Run**: built-in traffic splitting by revision (simplest canary implementation)
- **Cloud Deploy**: native canary with configured percentages

```yaml
# Cloud Run canary via traffic splitting
gcloud run services update-traffic myapp \
  --to-revisions=v2=10,v1=90  # 10% to new, 90% to old
```

### Feature Flags
- Decouple **deployment** from **release** — code is deployed but feature is off
- Flip flag to enable for subset of users
- GCP tool: no native feature flag service — use LaunchDarkly, Unleash, or custom config via Firestore/App Engine remote config

### Comparison Table

| Strategy | Rollback Speed | Resources | Risk | Best For |
|---|---|---|---|---|
| Rolling | Medium (pod by pod) | ~1.1x | Medium | Standard K8s workloads |
| Blue/Green | Instant (flip selector) | 2x | Low | DB migrations, high-risk releases |
| Canary | Gradual | ~1.1x | Very Low | New features, traffic-tested rollouts |
| Feature Flag | Instant | 1x | Very Low | Gradual user-facing releases |

---

## Success Metrics for Deployments

### Cloud Deploy Verification
- Run **verify jobs** after each canary phase — e.g., run test suite against deployed service
- If verify fails → rollback automatically

```yaml
# In skaffold.yaml — custom verify job
verify:
- name: verify-integration
  container:
    name: verify-container
    image: test-runner:latest
    command: ["/bin/sh"]
    args: ["-c", "curl -f http://app/healthz && ./run-smoke-tests.sh"]
```

### SLI-Based Success Metrics
- Monitor **error rate**, **latency p99**, **availability** during rollout
- Set alerting policies on these metrics during canary → alert = rollback signal
- Use **burn rate alerts** on SLOs to trigger rollback if error budget burns too fast

---

## Auditing and Tracking Deployments

### Cloud Audit Logs
- **Admin Activity logs**: who deployed what (always on, free)
- **Data Access logs**: who accessed what (must enable, billable)
- **System Event logs**: GCP system actions

```bash
# Query deployment events in Logs Explorer
resource.type="clouddeploy.googleapis.com/DeliveryPipeline"
protoPayload.methodName="CreateRollout"
```

### Artifact Registry Audit Trail
- Every push/pull logged in Cloud Audit Logs
- Image digest (`sha256:...`) provides immutable reference — link deployment to exact build

### Cloud Build History
- All builds logged with: trigger, commit SHA, steps, duration, success/fail
- Accessible in Cloud Console, gcloud, or via API

```bash
# List recent builds
gcloud builds list --limit=20 --filter="status=FAILURE"

# Describe a specific build
gcloud builds describe BUILD_ID
```

---

## Troubleshooting Deployment Issues

### Cloud Build Failures
1. Check build logs: `gcloud builds log BUILD_ID`
2. Common issues:
   - **Permission denied** → SA missing IAM role
   - **Timeout** → increase `timeout` in cloudbuild.yaml
   - **Network issues** → enable Private Google Access or use Private Pool
   - **Docker layer cache miss** → check `--cache-from` configuration

### Cloud Deploy Rollout Failures
1. Check rollout status: `gcloud deploy rollouts describe ROLLOUT_NAME`
2. Check target cluster: `kubectl get events -n default`
3. Common issues:
   - **Image not found** → check Artifact Registry path and SA permissions
   - **Verify job failed** → review verify job logs in Cloud Logging
   - **Pending approval** → not a failure — needs human action

### Rollback

```bash
# Roll back to previous release in Cloud Deploy
gcloud deploy rollouts rollback ROLLOUT_NAME \
  --delivery-pipeline=my-pipeline \
  --region=us-central1

# K8s rollback
kubectl rollout undo deployment/app
kubectl rollout undo deployment/app --to-revision=2
```

---

## Pipeline for ML Workloads

- **Vertex AI Pipelines**: managed ML pipeline execution (Kubeflow Pipelines SDK)
- CI/CD for ML = same principles + model-specific steps:
  - Data validation → training → model evaluation → registry → deployment
- **Vertex AI Model Registry**: store/version trained models
- **Cloud Deploy** can deploy ML models to: Cloud Run, GKE, Vertex AI endpoints
- Success metrics: model accuracy, inference latency, prediction error rate (telemetry-driven)

---

## Exam Tips

- `waitFor: ['-']` in Cloud Build = run step in parallel with all previous steps
- Cloud Deploy **releases** = immutable artifact + manifest snapshot; **rollouts** = deployment events
- Blue/green rollback = near-instant (switch LB/selector); canary rollback = gradual
- `requireApproval: true` blocks automatic promotion — human must approve
- Artifact Registry image digest (sha256) = immutable — always use digest in prod, not `:latest`
- Cloud Deploy canary works with: GKE Gateway API, GKE Service Mesh, Cloud Run traffic splitting
