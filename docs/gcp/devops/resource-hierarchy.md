# 1.1 – GCP Resource Hierarchy

## Hierarchy Levels (top → bottom)

```
Organization (root)
  └── Folder (optional, nestable up to 10 levels)
        └── Project
              └── Resources (VMs, GCS buckets, GKE clusters, etc.)
```

- Every resource has **exactly one parent** (except Organization)
- IAM policies and Org Policies **inherit downward** — set at the highest applicable level
- Projects are the **billing and API boundary**: all GCP resources live in a project

---

## Organization

- Tied to a **Google Workspace / Cloud Identity domain**
- Root node — owns all folders, projects, and resources
- Roles granted here apply to the entire hierarchy
- Key org-level roles:
  - `roles/resourcemanager.organizationAdmin` — manage org, folders, IAM
  - `roles/orgpolicy.policyAdmin` — manage Org Policies
  - `roles/billing.admin` — manage billing accounts

---

## Folders

- Group projects by **environment**, **team**, **BU**, **product**, etc.
- Max **10 levels** of nesting (Organization counts as level 0)
- Delegate folder-level admin without giving org-level access
- Useful pattern: `org → env-folder (prod/staging/dev) → team-folder → project`

```bash
# Create a folder
gcloud resource-manager folders create \
  --display-name="Production" \
  --organization=ORG_ID

# List folders under org
gcloud resource-manager folders list --organization=ORG_ID
```

---

## Projects

- Unit of **resource containment**, **billing**, **API enablement**
- Has a globally unique **Project ID** (immutable after creation)
- **Project Number** (auto-assigned, immutable)
- Deleting a project deletes all its resources (30-day soft-delete window)

```bash
gcloud projects create PROJECT_ID --folder=FOLDER_ID
gcloud projects describe PROJECT_ID
gcloud projects list --filter="parent.id=FOLDER_ID"
```

---

## IAM Policy Inheritance

| Concept | Detail |
|---|---|
| **Inheritance** | Policies granted at org/folder flow down to all children |
| **Additive** | Child cannot remove permissions granted at a higher level |
| **Deny policies** | Can override inherited allow policies (newer feature) |
| **Effective policy** | Union of all policies on the resource + all ancestors |

> **Exam tip:** IAM inheritance is additive. You cannot restrict a parent-level permission at child level using standard allow policies — use **Deny Policies** or **Org Policies** for that.

---

## Organization Policies

- Restrict what **can be configured**, not who can access
- Applied at org, folder, or project level — inherited downward
- Key constraints for DevOps:
  - `constraints/compute.requireShieldedVm` — enforce Shielded VMs
  - `constraints/iam.disableServiceAccountKeyCreation` — block SA key creation
  - `constraints/compute.restrictCloudRunRegion` — restrict Cloud Run regions
  - `constraints/gcp.resourceLocations` — enforce data residency

```bash
# View org policies on a project
gcloud resource-manager org-policies list --project=PROJECT_ID

# Set a policy
gcloud resource-manager org-policies set-policy \
  --project=PROJECT_ID policy.yaml
```

---

## Service Accounts

| Type | Description |
|---|---|
| **User-managed** | Created by you; used for workloads |
| **Default SA** | Auto-created per project/service (e.g., Compute Engine default SA) |
| **Google-managed** | Used internally by GCP services |

```bash
# Create SA
gcloud iam service-accounts create SA_NAME \
  --display-name="My SA" --project=PROJECT_ID

# Grant role to SA on a resource
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA@PROJECT.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Impersonate SA (no key file)
gcloud config set auth/impersonate_service_account SA@PROJECT.iam.gserviceaccount.com
```

> **Best practice:** Avoid SA keys. Prefer Workload Identity Federation or VM-attached SA.

---

## Naming / Structure Best Practices

- Keep folder depth **≤ 3 levels** (org → env → team)
- Separate **prod** and **non-prod** into different folders early — different Org Policies
- Use **application-centric** projects (one project per app/service/env) rather than team-centric
- Enforce **labels** (`team`, `env`, `cost-center`) for billing attribution
- Use `constraints/iam.disableServiceAccountKeyCreation` by default at org level

---

## Data Residency

- Control via **Org Policy** `constraints/gcp.resourceLocations`
- Restrict resources to specific regions/multi-regions
- Applies to resource creation — does not migrate existing resources

```yaml
# Example org policy: restrict to EU
constraint: constraints/gcp.resourceLocations
listPolicy:
  allowedValues:
    - in:eu-locations
```
