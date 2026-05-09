# Section 1 Cheat Sheet — Cloud Projects, Accounts & Billing

## Resource Hierarchy

```
Organization  -->  Folders (optional, max 10 levels deep)  -->  Projects  -->  Resources
```

- Organization auto-created with Google Workspace or Cloud Identity domain
- Every resource belongs to exactly **one** project
- Policies inherit **downward**; more permissive policy **wins**
- Folders are optional; projects are mandatory

### Project Identifiers

| Identifier | Unique? | Mutable? | Chosen by |
|------------|---------|----------|-----------|
| Project Name | No | Yes | User |
| Project ID | Globally | **No** (immutable) | User at creation |
| Project Number | Globally | No | Auto-assigned |

## Organization Policies

- Control **what** can be done (IAM controls **who**)
- Applied at org/folder/project level, inherited downward

### Must-Know Constraints

| Constraint | Purpose |
|------------|---------|
| `compute.vmExternalIpAccess` | Block external IPs on VMs |
| `iam.allowedPolicyMemberDomains` | Restrict IAM to specific domains |
| `gcp.resourceLocations` | Restrict resource regions/zones |
| `compute.trustedImageProjects` | Limit allowed boot images |
| `storage.uniformBucketLevelAccess` | Force uniform bucket access |

## IAM Quick Reference

**Model**: Who (member) + What access (role) + Which resource

### Role Types

| Type | Scope | Use When |
|------|-------|----------|
| Basic (Primitive) | Broad (viewer/editor/owner) | Almost never in production |
| Predefined | Service-specific | Default choice |
| Custom | User-defined permissions | Predefined roles too broad |

### Basic Roles

| Role | Can edit resources? | Can manage IAM? | Can manage billing? |
|------|---------------------|-----------------|---------------------|
| Viewer | No | No | No |
| Editor | Yes | No | No |
| Owner | Yes | Yes | Yes |

### Member Types

`user:` / `serviceAccount:` / `group:` / `domain:` / `allAuthenticatedUsers` / `allUsers`

> **Best practice**: Grant roles to **groups**, not individuals.

## Cloud Identity

- IDaaS: user mgmt, groups, SSO, MFA
- **GCDS** syncs from Active Directory/LDAP to Cloud Identity
- Free edition = basic mgmt; Premium = device mgmt + advanced security
- Google Workspace includes Cloud Identity

## APIs

- Most APIs **disabled** by default in new projects
- Enabling an API is **free** -- usage incurs charges

## Quotas

| Type | Resets? | Example |
|------|---------|---------|
| Rate quota | Yes (time-based) | API calls per 100s |
| Allocation quota | No (hard cap) | Max VMs per project |

- Applied at **project** level
- Quota exceeded = HTTP **429**
- Request increases via Console: IAM & Admin > Quotas

## Operations Suite (formerly Stackdriver)

| Product | Purpose |
|---------|---------|
| Cloud Monitoring | Metrics, dashboards, alerts, uptime checks |
| Cloud Logging | Centralized logs, Log Router (to GCS/BQ/Pub/Sub) |
| Cloud Trace | Distributed tracing / latency |
| Cloud Profiler | CPU & memory profiling |
| Error Reporting | Error aggregation |

---

## Billing

### Core Rules

- Every project must be linked to a billing account to create billable resources
- 1 project --> 1 billing account (at a time)
- 1 billing account --> many projects
- Billing accounts live at **org level**, outside the folder/project hierarchy

### Account Types

| Type | Payment |
|------|---------|
| Self-serve (online) | Auto-charged to card/bank |
| Invoiced (offline) | Monthly invoice, requires approval |

### Billing Roles

| Role | Key Ability |
|------|-------------|
| `billing.creator` | Create new billing accounts |
| `billing.admin` | Full billing mgmt (costs, payments, link/unlink) |
| `billing.user` | Link projects to a billing account |
| `billing.viewer` | View-only costs |
| `billing.costsManager` | Manage budgets & exports (no payment mgmt) |

> To link a project: need `billing.user` on billing account **AND** Project Owner or Project Billing Manager on the project.

### Unlinking Billing from a Project

- VMs **stopped**, GCS buckets **read-only**, most services stop
- Resources **not immediately deleted** (grace period)
- Re-link restores billing; stopped resources must be restarted **manually**

### Budgets & Alerts

- Default thresholds: 50%, 90%, 100%
- Can be based on **actual** or **forecasted** spend
- Scoped to: billing account / projects / services / labels
- Notifications: email (default), Cloud Monitoring channels, **Pub/Sub**
- Auto-cap spending: Budget Alert --> Pub/Sub --> Cloud Function --> disable billing

### Billing Export

| Destination | Detail | Use Case |
|-------------|--------|----------|
| **BigQuery** (recommended) | Line-item / resource-level | Analysis, dashboards |
| Cloud Storage | Daily summary CSV/JSON | Archival, legacy |

- BigQuery export types: Standard usage cost, Detailed usage cost, Pricing data
- Export is **free** (pay only for BQ queries)
- No backfill -- starts from enablement date

### Discounts

| Type | Commitment | Savings | How |
|------|------------|---------|-----|
| Committed Use (CUD) | 1yr or 3yr | Up to 57% / 70% | Manual purchase |
| Sustained Use (SUD) | None | Up to 30% | **Automatic** if usage > 25% of month |

- Labels (`key:value`) on resources enable cost filtering in billing reports

---

## Essential gcloud Commands

```bash
# --- Projects ---
gcloud projects list
gcloud projects create PROJECT_ID --name="Name" --folder=FOLDER_ID
gcloud config set project PROJECT_ID
gcloud projects delete PROJECT_ID            # 30-day recovery window

# --- Folders / Org ---
gcloud organizations list
gcloud resource-manager folders list --organization=ORG_ID
gcloud resource-manager folders create --display-name="Name" --organization=ORG_ID

# --- Org Policies ---
gcloud org-policies list-available-constraints --organization=ORG_ID
gcloud org-policies reset constraints/CONSTRAINT --organization=ORG_ID

# --- IAM ---
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:a@b.com" --role="roles/viewer"
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="user:a@b.com" --role="roles/viewer"
gcloud projects get-iam-policy PROJECT_ID

# --- APIs ---
gcloud services enable compute.googleapis.com
gcloud services list --enabled
gcloud services disable compute.googleapis.com

# --- Billing ---
gcloud billing accounts list
gcloud billing projects link PROJECT_ID --billing-account=BILLING_ID
gcloud billing projects unlink PROJECT_ID
gcloud billing projects describe PROJECT_ID
```

---

## Cloud Asset Inventory

- Org-wide inventory of ALL resources and IAM policies across projects
- Supports: Compute, GKE, Cloud SQL, Storage, IAM, etc.
- Real-time or point-in-time queries; export to BigQuery or GCS

```bash
# Export all assets to GCS
gcloud asset export --organization=ORG_ID \
  --output-path=gs://BUCKET/assets.json --content-type=resource

# Search all resources
gcloud asset search-all-resources --scope=organizations/ORG_ID \
  --query="name:prod-*" --asset-types=compute.googleapis.com/Instance

# Analyze IAM (who can do what on resource)
gcloud asset analyze-iam-policy --organization=ORG_ID \
  --full-resource-name=//storage.googleapis.com/projects/_/buckets/BUCKET \
  --permissions=storage.objects.list

# Real-time feed for changes
gcloud asset feeds create my-feed --organization=ORG_ID \
  --asset-types=compute.googleapis.com/Instance \
  --content-type=resource --pubsub-topic=projects/P/topics/T
```

## Gemini Cloud Assist

- AI assistant built into Cloud Console
- Summarizes logs, explains alerts, helps troubleshoot resources
- No extra setup needed — available in Console sidebar
- Key exam angle: Gemini cannot act (make changes) on your behalf; it **assists** you

| Feature | Description |
|---|---|
| Log summarization | Explains complex log patterns in plain language |
| Alert assistance | Helps write alert policies and PromQL |
| Troubleshooting | Analyzes resource state and suggests fixes |

---

## Confirming Product Availability by Region

```bash
# List services available in a region
gcloud services list --available --filter="name~compute" --project=PROJECT_ID

# Check resource quotas by region
gcloud compute regions describe REGION --project=PROJECT_ID

# Services by location (from service catalog)
gcloud services list --enabled --project=PROJECT_ID
```

---

## Traps & Gotchas

- **Budgets do NOT cap spending.** They only notify. To auto-cap: Pub/Sub + Cloud Function.
- **Project deletion has a 30-day recovery window** -- it is not instant.
- **More permissive policy wins.** You cannot restrict access at a child by removing a parent grant.
- **Enabling an API does not incur charges** -- only using the service does.
- **Billing account is NOT inside the folder/project hierarchy** -- it sits alongside at the org level.
- **BigQuery billing export has no backfill.** Historical data before enablement is lost.
- **Unlinking billing stops VMs but does NOT delete them immediately.**
- **SUDs are automatic; CUDs require a manual purchase commitment.**
- **Quota errors = HTTP 429.** Quotas are per-project, not per-org.
- **`allUsers` = the entire internet.** Extremely dangerous for IAM bindings.
- **Project ID is immutable.** Choose carefully at creation time.
