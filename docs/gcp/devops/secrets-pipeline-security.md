# 2.3 – Pipeline Config & Secrets  |  2.4 – Securing the Deployment Pipeline

---

# 2.3 – Managing Pipeline Configuration and Secrets

## Secret Manager

### Key Concepts
- Store **versioned, encrypted secrets** (passwords, API keys, TLS certs, SSH keys)
- Each secret has **versions** — can rotate without changing name
- IAM at **secret level** — granular access control
- Audit every access via Cloud Audit Logs

```bash
# Create a secret
echo -n "my-db-password" | gcloud secrets create db-password \
  --data-file=- \
  --replication-policy=automatic

# Add a new version
echo -n "new-password-v2" | gcloud secrets versions add db-password --data-file=-

# Access latest version
gcloud secrets versions access latest --secret=db-password

# Grant access to a SA
gcloud secrets add-iam-policy-binding db-password \
  --member="serviceAccount:SA@PROJECT.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### Secret Rotation
```bash
# Enable automatic rotation (sends Pub/Sub notification)
gcloud secrets update db-password \
  --rotation-period="7776000s" \   # 90 days
  --next-rotation-time="2024-06-01T00:00:00Z"
```

### Secret Replication
| Policy | Use case |
|---|---|
| `automatic` | GCP manages replication across regions |
| `user-managed` | You specify which regions (for data residency) |

---

## Key Management Service (Cloud KMS)

### Key Concepts
- Manage **cryptographic keys** for encryption/decryption and signing
- **CMEK (Customer-Managed Encryption Keys)**: use your own keys to encrypt GCP resources
- **Key rings** group keys by location
- **Key versions**: primary version used for encryption; older versions retained for decryption

### Key Types
| Type | Use case |
|---|---|
| **Symmetric (AES-256-GCM)** | Encrypt/decrypt data |
| **Asymmetric (RSA/EC)** | Sign/verify, encrypt (small payloads) |
| **HMAC** | Message authentication |
| **HSM-backed** | Hardware security module (higher assurance) |

```bash
# Create key ring and key
gcloud kms keyrings create my-keyring --location=us-central1
gcloud kms keys create my-key \
  --keyring=my-keyring \
  --location=us-central1 \
  --purpose=encryption

# Encrypt a file
gcloud kms encrypt \
  --key=my-key --keyring=my-keyring --location=us-central1 \
  --plaintext-file=secret.txt \
  --ciphertext-file=secret.enc

# Use CMEK for a GCS bucket
gsutil kms encryption -k \
  projects/PROJECT/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key \
  gs://my-bucket
```

### CMEK vs CSEK vs Google-Managed
| Type | Who manages key | Use case |
|---|---|---|
| Google-managed | Google | Default, minimal overhead |
| CMEK (Cloud KMS) | You, in GCP | Compliance, audit, revocation |
| CSEK (Customer-Supplied) | You, outside GCP | Maximum control; you supply key per request |

---

## Certificate Manager

- Manage **TLS/SSL certificates** for GCP load balancers and services
- Supports: Google-managed certs (auto-renew), self-managed, Certificate Authority Service
- Replaces the older `ManagedCertificate` resource for GKE Ingress

```bash
# Create a Google-managed cert
gcloud certificate-manager certificates create my-cert \
  --domains="app.example.com" \
  --project=PROJECT_ID
```

---

## Parameter Manager (New)

- **Lightweight config/parameter store** (different from Secret Manager)
- For non-sensitive configuration parameters (feature flags, env-specific config)
- Versions supported; IAM-controlled
- Use when you need **structured config** (JSON/YAML parameters) vs raw secret blobs

---

## Workload Identity Federation (WIF)

### Why WIF?
- Eliminate **SA JSON key files** — the #1 source of credential leaks
- External workloads (GitHub Actions, GitLab CI, AWS, Azure, on-prem) authenticate to GCP using their own identity tokens
- GCP STS exchanges the external token for a short-lived GCP access token

### WIF Architecture

```
GitHub Actions job → OIDC token (issued by GitHub)
    → Google STS (Secure Token Service) → validates OIDC token
    → Short-lived access token → used to call GCP APIs
```

### Setup for GitHub Actions

```bash
# Create workload identity pool
gcloud iam workload-identity-pools create github-pool \
  --location=global \
  --display-name="GitHub Actions Pool"

# Create OIDC provider
gcloud iam workload-identity-pools providers create-oidc github-provider \
  --workload-identity-pool=github-pool \
  --location=global \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --attribute-condition="assertion.repository=='org/repo'"

# Bind SA to WIF pool/provider
gcloud iam service-accounts add-iam-policy-binding deploy-sa@PROJECT.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUM/locations/global/workloadIdentityPools/github-pool/attribute.repository/org/repo"
```

```yaml
# GitHub Actions workflow
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/NUM/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
    service_account: 'deploy-sa@PROJECT.iam.gserviceaccount.com'
```

### WIF for GKE (different concept)
- Uses **GKE Workload Identity** (not the same as external WIF)
- Maps Kubernetes ServiceAccount → Google ServiceAccount via annotation
- No JSON key file ever touches the pod

```bash
# Enable workload identity on cluster
gcloud container clusters update CLUSTER \
  --workload-pool=PROJECT_ID.svc.id.goog

# Annotate K8s SA to bind to Google SA
kubectl annotate serviceaccount k8s-sa \
  iam.gke.io/gcp-service-account=gsa@PROJECT.iam.gserviceaccount.com
```

---

## Build-Time vs Runtime Secret Injection

| Timing | Method | Risk |
|---|---|---|
| **Build-time** | Inject during `docker build` — secret baked in image | ⚠️ HIGH — secret in image layer history |
| **Build-time (safe)** | Pass as env var to build step only (not COPY'd into image) | Medium — visible in build logs |
| **Runtime (recommended)** | App reads from Secret Manager / env var at startup | ✅ LOW |
| **Runtime (K8s)** | CSI Secret Store driver mounts Secret Manager secrets as files | ✅ LOW |

```yaml
# Safe: Secret Manager access at runtime in Cloud Build
steps:
- name: 'gcr.io/cloud-builders/gcloud'
  secretEnv: ['DB_PASSWORD']
  script: |
    echo "Connecting to DB with $$DB_PASSWORD"

availableSecrets:
  secretManager:
  - versionName: projects/PROJECT/secrets/db-password/versions/latest
    env: 'DB_PASSWORD'
```

---

# 2.4 – Securing the Deployment Pipeline

## Artifact Analysis & Vulnerability Scanning

- **Automatic scanning** on push to Artifact Registry (enable on repo)
- Scans for: OS vulnerabilities (CVEs), language packages (npm, pip, Maven)
- Results available in: Cloud Console, gcloud, API, Pub/Sub notifications

```bash
# Enable vulnerability scanning on a repo
gcloud artifacts repositories update my-repo \
  --location=us-central1 \
  --enable-vulnerability-scanning

# View vulnerabilities
gcloud artifacts docker images list-vulnerabilities \
  us-central1-docker.pkg.dev/PROJECT/my-repo/app:v1.0.0

# Check for critical CVEs in CI
gcloud artifacts docker images scan IMAGE_URI \
  --format="json" | jq '.response.scan.discoveryOccurrence'
```

---

## Binary Authorization

### Concepts
- **Policy enforcement**: only deploy images that satisfy configured policies
- **Attestations**: digital signatures proving an image passed a required check (e.g., vulnerability scan, QA approval)
- **Attestors**: entities that create attestations
- **Break-glass**: emergency bypass (all bypasses logged)

### Policy Modes
| Mode | Behavior |
|---|---|
| **Enforce** | Block deployments that don't meet policy |
| **Dry-run** | Log violations but allow deployment |
| **Audit (CV)** | Continuous validation — monitors running pods periodically |

### Pipeline Integration

```
Cloud Build builds image →
  Artifact Analysis scans for vulns →
  Custom attestor creates attestation (if scan passed) →
  Cloud Deploy rollout to GKE/Cloud Run →
  Binary Authorization checks attestation →
  ✅ Deploy OR ❌ Block
```

```yaml
# Binary Authorization policy
defaultAdmissionRule:
  evaluationMode: ALWAYS_DENY  # Block anything not explicitly allowed
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
admissionWhitelistPatterns:
- namePattern: gcr.io/google_containers/*
clusterAdmissionRules:
  us-central1.prod-cluster:
    evaluationMode: REQUIRE_ATTESTATION
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    requireAttestationsBy:
    - projects/PROJECT/attestors/qa-attestor
    - projects/PROJECT/attestors/vuln-scan-attestor
```

```bash
# Create attestor
gcloud container binauthz attestors create qa-attestor \
  --attestation-authority-note=NOTE_ID \
  --attestation-authority-note-project=PROJECT_ID

# Create attestation for an image
gcloud container binauthz attestations sign-and-create \
  --attestor=qa-attestor \
  --artifact-url=DIGEST_URL \
  --keyversion=KEY_VERSION
```

---

## SLSA Framework (Supply-chain Levels for Software Artifacts)

### SLSA Levels

| Level | Requirements | Cloud Build Status |
|---|---|---|
| **L0** | No guarantees | — |
| **L1** | Provenance exists | Cloud Build auto-generates |
| **L2** | Provenance signed; hosted build platform | Cloud Build (managed) |
| **L3** | Provenance unforgeable; isolated builds | Cloud Build (hardened config) |

### Build Provenance
- Cloud Build automatically generates **SLSA-compliant provenance** for container images
- Provenance = verifiable metadata: who built, when, from what source, with what config
- Stored in Artifact Analysis alongside the image

```bash
# View provenance for an image
gcloud artifacts docker images describe \
  us-central1-docker.pkg.dev/PROJECT/repo/app@sha256:DIGEST \
  --show-provenance
```

### SLSA Check with Binary Authorization CV

```bash
# Create CV policy that requires SLSA L3 provenance
gcloud container binauthz policy create-cv-policy \
  --platform-policy=platforms/gke/policies/slsa-check-policy.yaml
```

---

## IAM Policies for CI/CD Environments

### Principle: Least Privilege per Stage

| Pipeline Stage | SA Needed | Roles |
|---|---|---|
| Cloud Build (build+test) | `build-sa` | `roles/artifactregistry.writer`, `roles/secretmanager.secretAccessor` |
| Cloud Deploy (orchestrate) | `deploy-sa` | `roles/clouddeploy.releaser` |
| Cloud Deploy (deploy to GKE) | `gke-deploy-sa` | `roles/container.developer` |
| Production deployment | Separate SA | Minimal; `requireApproval: true` in pipeline |

### Separate SAs per Environment
```bash
# Dev SA - broad permissions
gcloud iam service-accounts create build-dev-sa

# Prod SA - minimal, tightly scoped
gcloud iam service-accounts create deploy-prod-sa

# Apply IAM condition: only allow prod SA in prod project
gcloud projects add-iam-policy-binding prod-project \
  --member="serviceAccount:deploy-prod-sa@PROJECT.iam.gserviceaccount.com" \
  --role="roles/container.developer" \
  --condition="resource.name.startsWith('projects/prod-project')"
```

---

## Exam Tips

- **Secret Manager** = secret storage + versioning + rotation + audit log
- **Cloud KMS** = cryptographic key management; used for CMEK
- **Workload Identity Federation** = keyless auth for external workloads (GitHub Actions, etc.)
- **Binary Authorization** = policy enforcement at deploy time using attestations
- **SLSA L2** = Cloud Build out-of-the-box; **L3** requires additional hardening
- **Artifact Analysis** = vulnerability scanning; results used by Binary Authorization
- Never inject secrets at build time into image layers — use runtime Secret Manager access
- Break-glass in Binary Authorization = emergency bypass + audit log entry
