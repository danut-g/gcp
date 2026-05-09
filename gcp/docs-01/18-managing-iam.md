# Section 5.1 — Managing Identity and Access Management (IAM)

## Exam Relevance
This topic is part of **Section 5: Configuring access and security (~17.5 % of the exam)**. You must know how to view and create IAM policies, manage the different role types (basic, predefined, custom), and define custom IAM roles.

---

## 1. IAM Fundamentals

> 📖 **Docs:** [IAM overview](https://cloud.google.com/iam/docs/overview) | [Understanding IAM](https://cloud.google.com/iam/docs/understanding-roles) | 🖥️ **Console:** IAM & Admin → IAM

### The IAM Model

```
WHO (Principal/Member) + WHAT (Role/Permissions) + WHERE (Resource) = IAM Policy
```

**IAM answers three questions:**
1. **Who** is making the request? (Member/Principal)
2. **What** are they allowed to do? (Role → Permissions)
3. **On which** resource? (Project, folder, organization, or specific resource)

### Principal Types

A **principal** is the identity that is granted access. GCP supports 6 principal types:

| Policy prefix | What it represents | Example |
|--------------|-------------------|---------|
| `user:` | An individual Google Account | `user:alice@example.com` |
| `serviceAccount:` | A GCP service account | `serviceAccount:my-sa@project.iam.gserviceaccount.com` |
| `group:` | A Google Group (contains multiple users) | `group:devs@example.com` |
| `domain:` | All users in a Google Workspace / Cloud Identity domain | `domain:example.com` |
| `allAuthenticatedUsers` | Any authenticated Google account in the world | *(no prefix)* |
| `allUsers` | Anyone — including unauthenticated, anonymous access | *(no prefix)* |

**Exam tips:**
- `allUsers` = fully public access (used for public GCS buckets, unauthenticated Cloud Run services)
- `allAuthenticatedUsers` ≠ "all users in my organization" — it means any Google account worldwide
- `domain:` is the most efficient way to grant access to all employees in a company
- Service accounts appear as principals when they are granted roles on resources (e.g., a SA that reads from GCS)

---

### How IAM Policies Work

```
Organization Policy
  └── Folder Policy
       └── Project Policy
            └── Resource Policy

Effective permissions = Union of all inherited + directly granted permissions
```

- Policies are **additive** — permissions are unioned, never subtracted
- **Deny policies** (introduced later) can explicitly deny permissions
- Child resources **inherit** parent policies
- A more permissive parent policy **cannot be overridden** by a more restrictive child policy

---

## 2. Viewing IAM Policies

> 📖 **Docs:** [Get and set IAM policies](https://cloud.google.com/iam/docs/managing-policies) | [Understand IAM allow policies](https://cloud.google.com/iam/docs/policies) | 🖥️ **Console:** IAM & Admin → IAM → view members and roles

### Viewing Project-Level IAM Policies

```bash
# View the IAM policy for a project
gcloud projects get-iam-policy PROJECT_ID

# View in JSON format (for editing)
gcloud projects get-iam-policy PROJECT_ID --format=json

# View in YAML format
gcloud projects get-iam-policy PROJECT_ID --format=yaml

# Filter to see specific member's roles
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:alice@example.com" \
  --format="table(bindings.role)"
```

### Viewing IAM Policies at Other Levels

```bash
# Organization level
gcloud organizations get-iam-policy ORG_ID

# Folder level
gcloud resource-manager folders get-iam-policy FOLDER_ID

# Resource level (examples)
gcloud storage buckets get-iam-policy gs://my-bucket
gcloud compute instances get-iam-policy my-vm --zone=us-central1-a
gcloud pubsub topics get-iam-policy my-topic
gcloud run services get-iam-policy my-service --region=us-central1
```

### Testing IAM Permissions

```bash
# Test if a member has specific permissions
gcloud projects test-iam-permissions PROJECT_ID \
  --permissions=compute.instances.create,compute.instances.delete
```

### IAM Policy Analyzer

```bash
# Analyze who has access to a resource
gcloud asset analyze-iam-policy \
  --organization=ORG_ID \
  --identity="user:alice@example.com"

# Find all members with a specific role
gcloud asset analyze-iam-policy \
  --organization=ORG_ID \
  --roles="roles/owner"
```

---

## 3. Creating and Modifying IAM Policies

> 📖 **Docs:** [Grant or revoke a single role](https://cloud.google.com/iam/docs/granting-changing-revoking-access) | [Set IAM policy with conditions](https://cloud.google.com/iam/docs/conditions-overview) | 🖥️ **Console:** IAM & Admin → IAM → Grant Access

### Adding Role Bindings

```bash
# Grant a role at the project level
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/compute.admin"

# Grant a role to a group
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:developers@example.com" \
  --role="roles/editor"

# Grant a role to a service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Grant a role at the organization level
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="user:admin@example.com" \
  --role="roles/resourcemanager.organizationAdmin"

# Grant a role at the folder level
gcloud resource-manager folders add-iam-policy-binding FOLDER_ID \
  --member="group:team@example.com" \
  --role="roles/viewer"
```

### Removing Role Bindings

```bash
# Remove a role binding
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/compute.admin"
```

### Conditional IAM Bindings

```bash
# Grant a role with a time-based condition
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:contractor@example.com" \
  --role="roles/compute.instanceAdmin.v1" \
  --condition='expression=request.time < timestamp("2025-12-31T00:00:00Z"),title=temporary-access,description=Expires end of year'

# Grant access only to resources with specific names
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:dev@example.com" \
  --role="roles/compute.instanceAdmin.v1" \
  --condition='expression=resource.name.startsWith("projects/PROJECT_ID/zones/us-central1-a/instances/dev-"),title=dev-vms-only'

# Grant access based on resource tags
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:team@example.com" \
  --role="roles/compute.admin" \
  --condition='expression=resource.matchTag("env","dev"),title=dev-only'
```

### IAM Deny Policies

Deny policies explicitly **prevent** principals from using specific permissions, even if they are granted through allow policies:

```bash
# Create a deny policy
gcloud iam policies create my-deny-policy \
  --attachment-point="cloudresourcemanager.googleapis.com/projects/PROJECT_ID" \
  --kind=denypolicies \
  --policy-file=deny-policy.json
```

```json
{
  "displayName": "Deny project deletion",
  "rules": [
    {
      "denyRule": {
        "deniedPrincipals": ["principalSet://goog/group/developers@example.com"],
        "deniedPermissions": ["cloudresourcemanager.googleapis.com/projects.delete"]
      }
    }
  ]
}
```

---

## 4. Role Types

> 📖 **Docs:** [Understanding roles](https://cloud.google.com/iam/docs/understanding-roles) | [Create and manage custom roles](https://cloud.google.com/iam/docs/creating-custom-roles) | [Predefined roles reference](https://cloud.google.com/iam/docs/predefined-roles) | 🖥️ **Console:** IAM & Admin → Roles

### Basic (Primitive) Roles

| Role | Name | Description |
|------|------|-------------|
| `roles/viewer` | Viewer | Read-only access to all resources |
| `roles/editor` | Editor | Read-write access (no IAM management) |
| `roles/owner` | Owner | Full access including IAM and billing |
| `roles/browser` | Browser | Browse project hierarchy |

**Important for the exam:**
- Basic roles are **very broad** — they grant thousands of permissions
- **Google recommends predefined roles** instead of basic roles
- **Owner** includes Editor + IAM admin + billing management
- **Editor** does NOT include permission to modify IAM policies
- **Never use basic roles in production** unless absolutely necessary

### Predefined Roles

Google-managed roles with fine-grained, service-specific permissions:

| Category | Example Roles |
|----------|--------------|
| **Compute Engine** | `roles/compute.admin`, `roles/compute.instanceAdmin.v1`, `roles/compute.networkAdmin`, `roles/compute.viewer` |
| **Cloud Storage** | `roles/storage.admin`, `roles/storage.objectViewer`, `roles/storage.objectCreator` |
| **BigQuery** | `roles/bigquery.admin`, `roles/bigquery.dataEditor`, `roles/bigquery.jobUser`, `roles/bigquery.user` |
| **GKE** | `roles/container.admin`, `roles/container.developer`, `roles/container.viewer` |
| **Cloud Run** | `roles/run.admin`, `roles/run.developer`, `roles/run.invoker` |
| **Cloud SQL** | `roles/cloudsql.admin`, `roles/cloudsql.editor`, `roles/cloudsql.viewer` |
| **IAM** | `roles/iam.serviceAccountAdmin`, `roles/iam.serviceAccountUser`, `roles/iam.workloadIdentityUser` |
| **Logging** | `roles/logging.admin`, `roles/logging.viewer`, `roles/logging.logWriter` |
| **Monitoring** | `roles/monitoring.admin`, `roles/monitoring.viewer`, `roles/monitoring.editor` |

```bash
# List all predefined roles
gcloud iam roles list

# List roles for a specific service
gcloud iam roles list --filter="name:roles/compute"

# Describe a role (see all permissions)
gcloud iam roles describe roles/compute.instanceAdmin.v1

# List permissions in a role
gcloud iam roles describe roles/storage.objectViewer \
  --format="value(includedPermissions)"
```

### Custom Roles

When predefined roles don't match your needs, create **custom roles** with exactly the permissions required (least privilege):

```bash
# Create a custom role at the project level
gcloud iam roles create myCustomRole \
  --project=PROJECT_ID \
  --title="My Custom Role" \
  --description="Custom role for app deployment" \
  --permissions=compute.instances.get,compute.instances.list,storage.objects.get,storage.objects.list \
  --stage=GA

# Create a custom role at the organization level
gcloud iam roles create orgCustomRole \
  --organization=ORG_ID \
  --title="Organization Custom Role" \
  --permissions=resourcemanager.projects.list,resourcemanager.folders.list

# Create from a YAML file
gcloud iam roles create myRole \
  --project=PROJECT_ID \
  --file=role-definition.yaml
```

**role-definition.yaml:**
```yaml
title: "App Deployer"
description: "Can deploy and manage applications"
stage: "GA"
includedPermissions:
  - compute.instances.get
  - compute.instances.list
  - compute.instances.create
  - compute.instances.delete
  - storage.objects.get
  - storage.objects.list
  - storage.objects.create
```

### Managing Custom Roles

```bash
# List custom roles
gcloud iam roles list --project=PROJECT_ID

# Describe a custom role
gcloud iam roles describe myCustomRole --project=PROJECT_ID

# Update a custom role (add permissions)
gcloud iam roles update myCustomRole \
  --project=PROJECT_ID \
  --add-permissions=compute.instances.start,compute.instances.stop

# Update a custom role (remove permissions)
gcloud iam roles update myCustomRole \
  --project=PROJECT_ID \
  --remove-permissions=compute.instances.delete

# Disable a custom role
gcloud iam roles update myCustomRole \
  --project=PROJECT_ID \
  --stage=DISABLED

# Delete a custom role (soft delete — can be undeleted within 7 days)
gcloud iam roles delete myCustomRole --project=PROJECT_ID

# Undelete a custom role
gcloud iam roles undelete myCustomRole --project=PROJECT_ID
```

### Custom Role Stages

| Stage | Description |
|-------|-------------|
| **ALPHA** | Role is in early testing |
| **BETA** | Role has been tested, may change |
| **GA** | Role is generally available and stable |
| **DISABLED** | Role is disabled (cannot be granted) |

### Custom Role Limitations
- Maximum **3,000 permissions** per custom role
- Maximum custom roles per organization or project (varies)
- Some permissions **cannot be used in custom roles** (check documentation)
- Custom roles can only be created at the **project or organization level** (not folder)

---

## 5. IAM Best Practices

> 📖 **Docs:** [Best practices for using IAM securely](https://cloud.google.com/iam/docs/using-iam-securely) | [IAM conditions best practices](https://cloud.google.com/iam/docs/conditions-best-practices) | 🖥️ **Console:** IAM & Admin → IAM Recommender (Policy Insights)

### Principle of Least Privilege
- Grant the **minimum permissions** needed for the task
- Use **predefined roles** instead of basic roles
- Use **custom roles** when predefined roles are too broad
- Review and audit permissions regularly

### Use Groups, Not Individual Users
- Create Google Groups for teams/functions
- Grant roles to groups, not individual users
- Easier to manage, audit, and revoke access

### Separation of Duties
- Don't give one person Owner on everything
- Separate network admin, security admin, and billing admin
- Use different service accounts for different applications

### Audit and Review
- Regularly review IAM policies
- Use **IAM Recommender** to identify over-provisioned access
- Enable **Data Access audit logs** for sensitive resources
- Use **Policy Analyzer** to understand who has access to what

```bash
# Get IAM recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --recommender=google.iam.policy.Recommender \
  --location=global
```

---

## 6. IAM Policy Structure

> 📖 **Docs:** [IAM policy structure](https://cloud.google.com/iam/docs/policies#structure) | [Resource hierarchy for access control](https://cloud.google.com/iam/docs/resource-hierarchy-access-control) | 🖥️ **Console:** IAM & Admin → IAM → select principal → View policy JSON

### Policy JSON Format

```json
{
  "bindings": [
    {
      "role": "roles/storage.objectViewer",
      "members": [
        "user:alice@example.com",
        "group:readers@example.com",
        "serviceAccount:my-sa@project.iam.gserviceaccount.com"
      ]
    },
    {
      "role": "roles/storage.objectAdmin",
      "members": [
        "user:admin@example.com"
      ],
      "condition": {
        "title": "production-only",
        "description": "Only for production bucket",
        "expression": "resource.name.startsWith('projects/_/buckets/prod-')"
      }
    }
  ],
  "etag": "BwXXXXXXXXX=",
  "version": 3
}
```

### Key Fields
- **bindings** — List of role-to-members mappings
- **role** — The IAM role being granted
- **members** — List of identities receiving the role
- **condition** — Optional CEL expression for conditional access
- **etag** — Concurrency control (prevents race conditions when updating)
- **version** — Policy version (3 for conditional bindings)

---

## 7. User Suspension

> 📖 **Docs:** [Suspend or restore a user](https://cloud.google.com/identity/docs/how-to/suspend-user) | [Disable or delete a user account](https://support.google.com/a/answer/33312) | 🖥️ **Console:** Admin console (admin.google.com) → Directory → Users → select user → Suspend

### What "Suspended" Means

A **suspended user** is a Google Workspace (formerly G Suite) account that has been temporarily disabled by a Workspace admin. Suspension is not a GCP-native IAM concept — it lives at the Google identity layer, one level below GCP IAM.

When a user is suspended:
- They **cannot sign in** to any Google service (Gmail, Console, gcloud CLI, etc.)
- Their **IAM bindings in GCP are not removed** — they still appear in every `get-iam-policy` output
- Their data, Drive files, and settings are **preserved** unchanged
- They can be **reinstated** by a Workspace admin at any time

### Why It Matters for IAM

Even though a suspended user cannot authenticate, their IAM role bindings remain active in GCP. This has practical consequences:
- If reinstated, they immediately regain all their previous GCP permissions — no re-granting needed
- IAM Recommender may flag their bindings as unused, prompting a review
- Policy Analyzer will still list them as having access to resources

Best practice: when a user is suspended (e.g., offboarding), **also remove their IAM bindings** manually. Don't rely on suspension alone as an access control measure.

### Suspension vs. Deletion

| State | Can sign in | IAM bindings | Data | Reversible |
|-------|-------------|--------------|------|-----------|
| **Active** | Yes | Active | Present | — |
| **Suspended** | No | Remain (inactive) | Preserved | Yes — admin reinstates |
| **Deleted** | No | Cleaned up by Google eventually | Deleted (20-day recovery window) | Within 20 days only |

### Service Account Disabling (Analogous Concept)

GCP's equivalent for service accounts is **disabling** (not deleting):

```bash
# Disable a service account (blocks all authentication, bindings remain)
gcloud iam service-accounts disable SA_EMAIL

# Re-enable a service account
gcloud iam service-accounts enable SA_EMAIL

# Check current status
gcloud iam service-accounts describe SA_EMAIL --format="value(disabled)"
```

A disabled service account behaves exactly like a suspended user: authentication fails, but all IAM role bindings on resources remain intact and are instantly restored when re-enabled.

### Exam Angle

The exam may describe a scenario where an employee goes on leave or is offboarded and ask the safest/most reversible action. The answer hierarchy:
1. **Suspend** the Workspace account → blocks login, preserves everything, fully reversible
2. **Remove IAM bindings** → ensures no access even if reinstated unexpectedly
3. **Delete** the account → only when certain they won't return (20-day window to undo)

For service accounts, prefer **disabling** over deleting when the SA may be needed again.

---

## 8. Organization Policies (Constraints)

> 📖 **Docs:** [Organization Policy Service overview](https://cloud.google.com/resource-manager/docs/organization-policy/overview) | [Predefined constraints](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) | 🖥️ **Console:** IAM & Admin → Organization Policies

- Organization policies enforce guardrails across an entire organization, folder, or project — regardless of IAM bindings
- Managed by the Organization Policy Administrator (`roles/orgpolicy.policyAdmin`)
- Two types of constraints: **boolean** (on/off) and **list** (allow/deny specific values)

### Common constraints

```bash
# Disable VM external IP (list constraint)
gcloud resource-manager org-policies set-policy \
  --project=PROJECT_ID \
  --policy='{
    "constraint": "constraints/compute.vmExternalIpAccess",
    "listPolicy": {"allValues": "DENY"}
  }'

# Require OS Login (boolean)
gcloud resource-manager org-policies enable-enforce \
  --project=PROJECT_ID \
  constraints/compute.requireOsLogin

# Restrict allowed cloud regions
gcloud resource-manager org-policies set-policy \
  --organization=ORG_ID \
  --policy='{
    "constraint": "constraints/gcp.resourceLocations",
    "listPolicy": {"allowedValues": ["in:us-locations"]}
  }'

# View effective policy (merging parent and current level)
gcloud resource-manager org-policies describe constraints/compute.vmExternalIpAccess --project=PROJECT_ID
gcloud resource-manager org-policies list --organization=ORG_ID
```

### Common exam constraints to know

| Constraint | Purpose |
|-----------|---------|
| `compute.vmExternalIpAccess` | Prevent VMs from getting public IPs |
| `compute.requireOsLogin` | Enforce OS Login on all VMs |
| `compute.restrictCloudSQLInstances` | Prevent Cloud SQL from having public IP |
| `iam.allowedPolicyMemberDomains` | Restrict IAM to specific domains |
| `gcp.resourceLocations` | Limit which regions resources can be created in |
| `storage.uniformBucketLevelAccess` | Enforce uniform bucket-level access on GCS |
| `iam.disableServiceAccountKeyCreation` | Prevent SA key downloads |

**Inheritance**: org-level policies are inherited by all children; a child can only restrict further (cannot override to be more permissive unless the org policy itself allows override).

---

## 9. IAM Recommender

> 📖 **Docs:** [IAM Recommender overview](https://cloud.google.com/iam/docs/recommender-overview) | [Apply and dismiss recommendations](https://cloud.google.com/iam/docs/recommender-managing) | 🖥️ **Console:** IAM & Admin → IAM → Policy Insights column

- Automatically analyzes permissions usage over **90 days** and suggests removing unused permissions
- Recommendations are per-member-per-role-per-resource
- Each recommendation has an impact score (HIGH, MEDIUM, LOW)

```bash
# List IAM recommendations for a project
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --recommender=google.iam.policy.Recommender \
  --location=global \
  --format="table(name,stateInfo.state,priority,description)"

# Apply a recommendation (replace role with smaller one)
gcloud recommender recommendations mark-claimed RECOMMENDATION_ID \
  --project=PROJECT_ID \
  --recommender=google.iam.policy.Recommender \
  --location=global \
  --etag=ETAG

# Mark as dismissed (won't apply)
gcloud recommender recommendations mark-dismissed RECOMMENDATION_ID \
  --project=PROJECT_ID --recommender=google.iam.policy.Recommender --location=global
```

**Exam tip**: Recommender suggests replacing `roles/editor` with specific predefined roles. Always review before applying — some roles may be used programmatically without being captured in 90-day logs.

---

## Exam Practice Questions

1. **A developer needs to create and manage Compute Engine VMs but should not be able to modify firewall rules or networking. Which role should you grant?**
   - Answer: `roles/compute.instanceAdmin.v1` — Grants full control over VMs but not networking resources.

2. **You need to grant a contractor temporary access to view Cloud Storage objects, expiring December 31. How?**
   - Answer: Use a **conditional IAM binding** with a time expression: `--condition='expression=request.time < timestamp("2025-12-31T00:00:00Z"),title=temp-access'` with `roles/storage.objectViewer`.

3. **A team of 15 developers needs identical access to GKE. What's the most efficient way to manage this?**
   - Answer: Create a **Google Group** (e.g., `gke-devs@example.com`), add all 15 developers, and grant `roles/container.developer` to the group at the project level.

4. **The predefined `roles/storage.admin` role grants too many permissions for your use case. You only need object read and bucket list permissions. What should you do?**
   - Answer: Create a **custom role** with only the specific permissions needed: `storage.objects.get`, `storage.objects.list`, `storage.buckets.list`.

5. **A user has `roles/viewer` at the organization level and `roles/compute.admin` at a specific project. What are their effective Compute Engine permissions in that project?**
   - Answer: They have **both** — `roles/viewer` (inherited from org) AND `roles/compute.admin` (directly on project). IAM policies are additive, so they get the union of all permissions.

6. **Can you remove `roles/viewer` permissions from a user at the project level if they were granted `roles/editor` at the folder level?**
   - Answer: **No**. IAM policies are additive and inherited. You cannot override a parent policy at a child level using allow policies. You would need to use a **deny policy** or remove the binding at the folder level.

7. **What is the difference between `roles/editor` and `roles/owner`?**
   - Answer: `roles/editor` grants read-write access to most resources but **cannot manage IAM policies or billing**. `roles/owner` grants all Editor permissions **plus** IAM management and billing management.

---

## Glossary

**allAuthenticatedUsers**: A special IAM principal identifier that represents any user who is authenticated with a Google account worldwide; it does not restrict access to users within a specific organization.

**allUsers**: A special IAM principal identifier that grants access to anyone, including unauthenticated (anonymous) users; used to make resources such as Cloud Storage buckets or Cloud Run services fully public.

**ALPHA stage**: A custom IAM role lifecycle stage indicating the role is in early testing and not yet ready for general use.

**attachment point**: The resource (organization, folder, or project) to which an IAM deny policy is attached, specified via a URL-style identifier such as `cloudresourcemanager.googleapis.com/projects/PROJECT_ID`.

**Basic role**: A coarse-grained, legacy IAM role (`roles/viewer`, `roles/editor`, `roles/owner`, `roles/browser`) that grants broad permissions across all GCP services; Google recommends using predefined or custom roles instead.

**BETA stage**: A custom IAM role lifecycle stage indicating the role has been tested but may still change before general availability.

**Binding**: See IAM policy binding — the association of an IAM role to one or more principals within an IAM policy.

**Boolean constraint**: An organization policy constraint type that is either enforced (on) or not enforced (off), such as `compute.requireOsLogin`.

**CEL (Common Expression Language)**: A lightweight expression language used in conditional IAM bindings to evaluate attributes such as request time, resource name, and resource tags at policy evaluation time.

**Cloud Identity**: Google's Identity-as-a-Service platform that provides managed user and group identities for organizations that do not use Google Workspace; supports `domain:` principal type in IAM.

**Cloud SQL**: Google Cloud's fully managed relational database service; referenced in organization policy constraints such as `compute.restrictCloudSQLInstances` to prevent public IP assignment.

**Compute Engine**: Google Cloud's Infrastructure-as-a-Service (IaaS) offering for running virtual machines; a primary resource type used in IAM policy examples and organization policy constraints.

**Condition (IAM)**: An optional CEL expression attached to an IAM role binding that restricts when and where the binding is effective, based on attributes such as request time, resource name, or resource tags.

**constraints/compute.requireOsLogin**: An organization policy boolean constraint that enforces OS Login on all Compute Engine VMs, centralizing SSH access control through IAM rather than metadata SSH keys.

**constraints/compute.restrictCloudSQLInstances**: An organization policy constraint that restricts which Cloud SQL instance configurations (such as public IP assignment) are allowed in the organization.

**constraints/compute.vmExternalIpAccess**: An organization policy list constraint that controls whether Compute Engine VMs are allowed to have external (public) IP addresses.

**constraints/gcp.resourceLocations**: An organization policy list constraint that restricts the Google Cloud regions and multi-regions in which resources may be created.

**constraints/iam.allowedPolicyMemberDomains**: An organization policy list constraint that restricts which identity domains may be granted IAM roles in the organization, preventing binding external Google accounts.

**constraints/iam.disableServiceAccountKeyCreation**: An organization policy boolean constraint that prevents creation of service account keys across the entire organization or scoped project.

**constraints/storage.uniformBucketLevelAccess**: An organization policy boolean constraint that enforces uniform bucket-level access on all new Cloud Storage buckets, disallowing object-level ACLs.

**Custom role**: A user-defined IAM role created at the project or organization level that contains exactly the permissions required for a specific use case, supporting the principle of least privilege.

**Custom role stage**: A lifecycle marker (`ALPHA`, `BETA`, `GA`, or `DISABLED`) applied to a custom IAM role to indicate its maturity and whether it is available to be granted.

**Data Access audit log**: A Cloud Audit Log type that records API calls that read resource data or metadata; disabled by default (except for BigQuery) and generates log ingestion charges when enabled.

**deniedPermissions**: A field in an IAM deny rule listing the permissions that the rule denies to the specified denied principals, using the canonical form `service.googleapis.com/resource.action`.

**deniedPrincipals**: A field in an IAM deny rule listing the principals (in `principalSet://` form) to whom the denied permissions apply.

**Deny policy**: An IAM policy type that explicitly prevents specified principals from using specified permissions, overriding any allow policies that would otherwise grant those permissions.

**DISABLED stage**: A custom IAM role lifecycle stage indicating the role is disabled and cannot be granted to any principal.

**Disabled service account**: A service account that has been deactivated via `gcloud iam service-accounts disable`, blocking authentication while preserving its IAM bindings so it can be re-enabled later.

**domain**: An IAM principal type prefix (`domain:example.com`) that grants access to all users within a specific Google Workspace or Cloud Identity domain; the most efficient way to grant access to all employees in a company.

**Editor**: A basic IAM role (`roles/editor`) that grants read-write access to most GCP resources but does not include IAM management or billing administration permissions.

**etag**: A concurrency control token included in IAM policy JSON that prevents race conditions when multiple clients read and update the same policy simultaneously; must be included when writing back a policy.

**Folder**: A GCP resource hierarchy node below the organization and above projects, used to group related projects and apply inherited IAM policies and organization policies to multiple projects at once.

**GA stage**: A custom IAM role lifecycle stage indicating the role is generally available, stable, and ready for production use.

**gcloud asset analyze-iam-policy**: A CLI command that uses Policy Analyzer to determine which principals have access to which resources, useful for security audits and access reviews.

**gcloud projects add-iam-policy-binding**: The gcloud command used to grant a role to a principal on a project-level IAM policy without needing to download, edit, and upload the full policy JSON.

**gcloud projects get-iam-policy**: The gcloud command used to retrieve the IAM policy attached to a project, available in JSON or YAML formats for inspection or editing.

**gcloud projects remove-iam-policy-binding**: The gcloud command used to remove a specific role-member binding from a project-level IAM policy.

**gcloud projects set-iam-policy**: The gcloud command used to replace a project's IAM policy with one provided in a JSON file, typically used after editing for bulk changes or when adding audit configs.

**gcloud projects test-iam-permissions**: The gcloud command used to test whether the caller has specified permissions on a given project, used to verify effective IAM access without attempting operations.

**gcloud recommender**: A CLI command group used to list and act on IAM Recommender suggestions for removing or reducing over-provisioned IAM role bindings.

**Google account**: Any Google-managed identity (gmail.com, Workspace, Cloud Identity) that can be used as a principal in an IAM binding via the `user:` prefix.

**Google Group**: A collection of Google account identities that can be used as a single IAM principal (`group:`); granting roles to groups rather than individual users simplifies access management.

**Google Workspace**: Google's cloud-based productivity and collaboration suite (formerly G Suite); provides user identities that can be suspended, deleted, or reinstated by a Workspace admin.

**group**: An IAM principal type prefix (`group:`) that grants access to all members of a specified Google Group, enabling centralized management of permissions for a team or function.

**IAM (Identity and Access Management)**: Google Cloud's system for controlling which principals can perform which actions on which resources, enforced through policies consisting of role bindings at various resource hierarchy levels.

**IAM policy**: A document attached to a GCP resource (organization, folder, project, or individual resource) that specifies a set of role bindings, each mapping a role to one or more principals.

**IAM policy binding**: A single entry in an IAM policy that associates a role with one or more principals, optionally constrained by a condition.

**IAM Recommender**: A Google Cloud service that analyzes IAM permission usage over a 90-day window and automatically generates recommendations to remove or reduce unused or over-provisioned role grants.

**IAM role**: A named collection of permissions that can be granted to a principal; roles are either basic (primitive), predefined (Google-managed), or custom (user-defined).

**Inheritance (IAM)**: The mechanism by which IAM policies applied at a parent resource level (organization or folder) are automatically granted to all descendant resources (folders, projects, and resources within them).

**List constraint**: An organization policy constraint type that enforces an allow or deny list of specific values, such as permitted resource locations or allowed project member domains.

**Member**: A synonym for principal in IAM; the identity (user, group, serviceAccount, domain, or special identifier) receiving a role binding.

**Organization**: The root node of the GCP resource hierarchy representing a company or entity, to which organization-wide IAM policies and organization policies are applied.

**Organization policy**: A governance control applied at the organization, folder, or project level that enforces guardrails (constraints) on resource configuration independently of IAM role bindings.

**Organization Policy Administrator**: The IAM role (`roles/orgpolicy.policyAdmin`) that grants the ability to view and set organization policies across the resource hierarchy.

**Owner**: A basic IAM role (`roles/owner`) that grants all Editor permissions plus the ability to manage IAM policies and billing; the most permissive basic role.

**Permission**: The lowest-level IAM unit of authorization, expressed as `service.resource.verb` (e.g., `compute.instances.create`); permissions are grouped into roles and never granted directly to principals.

**Policy Analyzer**: A GCP tool (accessible via `gcloud asset analyze-iam-policy`) that answers access questions such as which principals have a specific permission on a resource, used for auditing and compliance.

**Policy Denied audit log**: A Cloud Audit Log type that records when a request is denied by a security policy such as VPC Service Controls; always enabled and free of charge.

**Predefined role**: A Google-managed IAM role with fine-grained, service-specific permissions (e.g., `roles/compute.instanceAdmin.v1`) that is updated automatically when new features are released; preferred over basic roles.

**Principal**: An identity in Google Cloud IAM that can be granted roles; principal types include user, serviceAccount, group, domain, allAuthenticatedUsers, and allUsers.

**principalSet**: A URL-style principal identifier (e.g., `principalSet://goog/group/...`) used in IAM deny policies to reference a set of identities such as a Google Group or workload identity pool.

**Project**: A GCP resource hierarchy node that serves as the primary unit of resource ownership, billing, and IAM policy application; every resource belongs to exactly one project.

**Principle of least privilege**: The security practice of granting only the minimum permissions required for a task, reducing the blast radius of compromised credentials or misconfigured access.

**Resource hierarchy**: The organizational structure of GCP resources consisting of organization → folders → projects → resources; IAM policies are inherited down this hierarchy.

**Role**: A named collection of IAM permissions that defines what actions a principal can perform on GCP resources; roles are the unit of access grant in IAM and come in three types: basic, predefined, and custom.

**Role binding**: The association of a role to one or more principals within an IAM policy, optionally scoped by a condition; role bindings are additive — permissions are unioned across all bindings that apply to a principal.

**roles/bigquery.admin**: A predefined IAM role that grants full control over BigQuery resources including datasets, tables, and jobs.

**roles/bigquery.dataEditor**: A predefined IAM role that grants read-write access to BigQuery datasets and tables but not job management.

**roles/bigquery.jobUser**: A predefined IAM role that grants the ability to run BigQuery jobs and queries within a project.

**roles/bigquery.user**: A predefined IAM role that grants basic BigQuery access, including running jobs and reading datasets the user has been granted.

**roles/browser**: A basic IAM role that grants permission to browse the GCP project hierarchy without granting access to any resource data.

**roles/cloudsql.admin**: A predefined IAM role that grants full control over Cloud SQL instances including configuration, users, and backups.

**roles/cloudsql.editor**: A predefined IAM role that grants management of Cloud SQL instances excluding IAM and certain high-impact operations.

**roles/cloudsql.viewer**: A predefined IAM role that grants read-only access to Cloud SQL instances and their metadata.

**roles/compute.admin**: A predefined IAM role that grants full control over all Compute Engine resources including VMs, networks, and firewall rules.

**roles/compute.instanceAdmin.v1**: A predefined IAM role that grants full control over Compute Engine VM instances but not over networking resources such as firewall rules.

**roles/compute.networkAdmin**: A predefined IAM role that grants full control over Compute Engine networking resources (VPCs, subnets, firewall rules, routes) but not over VM instances.

**roles/compute.viewer**: A predefined IAM role that grants read-only access to all Compute Engine resources.

**roles/container.admin**: A predefined IAM role that grants full control over GKE clusters and their workloads.

**roles/container.developer**: A predefined IAM role that grants access to deploy and manage workloads within existing GKE clusters without cluster administration rights.

**roles/container.viewer**: A predefined IAM role that grants read-only access to GKE clusters and their workloads.

**roles/editor**: See Editor.

**roles/iam.serviceAccountAdmin**: A predefined IAM role that grants full management of service accounts within a project, including creation, deletion, and key management.

**roles/iam.serviceAccountUser**: A predefined IAM role that allows a principal to attach a service account to a resource such as a VM or Cloud Run service.

**roles/iam.workloadIdentityUser**: A predefined IAM role granted to external identities (via Workload Identity Federation) to allow them to impersonate a Google Cloud service account.

**roles/logging.admin**: A predefined IAM role that grants full control over Cloud Logging resources including log buckets, sinks, and metrics.

**roles/logging.logWriter**: A predefined IAM role that allows a principal to write log entries to Cloud Logging; commonly granted to the writer identity of a log sink.

**roles/logging.viewer**: A predefined IAM role that grants read-only access to log entries in Cloud Logging.

**roles/monitoring.admin**: A predefined IAM role that grants full control over Cloud Monitoring resources including alert policies, dashboards, and notification channels.

**roles/monitoring.editor**: A predefined IAM role that grants read-write access to Cloud Monitoring resources excluding IAM policy management.

**roles/monitoring.viewer**: A predefined IAM role that grants read-only access to Cloud Monitoring dashboards, alert policies, and metrics.

**roles/orgpolicy.policyAdmin**: A predefined IAM role that grants the ability to view and set organization policies across the GCP resource hierarchy.

**roles/owner**: See Owner.

**roles/resourcemanager.organizationAdmin**: A predefined IAM role that grants full control over the GCP organization resource, including managing IAM policies at the organization level.

**roles/run.admin**: A predefined IAM role that grants full control over Cloud Run services and their IAM policies.

**roles/run.developer**: A predefined IAM role that grants the ability to deploy and manage Cloud Run services without modifying IAM policies.

**roles/run.invoker**: A predefined IAM role that allows a principal to invoke (send HTTP requests to) a Cloud Run service.

**roles/storage.admin**: A predefined IAM role that grants full control over Cloud Storage buckets and objects, including IAM policy management.

**roles/storage.hmacKeyAdmin**: A predefined IAM role that grants the ability to create and manage HMAC keys for Cloud Storage, used for S3-compatible API access.

**roles/storage.objectAdmin**: A predefined IAM role that grants full control over Cloud Storage objects (read, write, delete) but not bucket-level configuration.

**roles/storage.objectCreator**: A predefined IAM role that allows creating Cloud Storage objects without the ability to read or delete them.

**roles/storage.objectViewer**: A predefined IAM role that grants read-only access to Cloud Storage objects and their metadata.

**roles/viewer**: A basic IAM role (`roles/viewer`) that grants read-only access to all GCP resources within the scope it is applied.

**Separation of duties**: A security principle that divides administrative responsibilities among multiple accounts or roles to prevent any single identity from having excessive control.

**serviceAccount**: An IAM principal type prefix (`serviceAccount:`) used to reference a Google Cloud service account as a member in an IAM policy binding.

**Service account (as a principal)**: A service account acting as a member in an IAM policy, receiving role bindings that define what it can do on GCP resources.

**Service account (as a resource)**: A service account treated as a GCP resource that has its own IAM policy, controlling which users or other service accounts can use or manage it.

**Soft delete**: The behavior when a custom IAM role or service account is deleted; it enters a deleted state and can be undeleted within 7 days (custom roles) or 30 days (service accounts) before being permanently removed.

**Suspended user**: A Google Workspace account that has been temporarily disabled by a Workspace administrator; the user cannot authenticate, but their GCP IAM role bindings remain in place until manually removed.

**Tag (resource tag)**: A key-value label attached to GCP resources that can be referenced in conditional IAM bindings (`resource.matchTag`) to restrict access to resources with specific tags.

**user**: An IAM principal type prefix (`user:`) that references an individual Google account by email address for use in IAM policy bindings.

**Version (IAM policy)**: The schema version of an IAM policy JSON document; version 3 is required when the policy contains conditional role bindings.

**Viewer**: A basic IAM role (`roles/viewer`) that grants read-only access to all resources within the scope it is granted; does not allow modification of any resources.

**VPC Service Controls**: A GCP security feature that creates a service perimeter around GCP resources to prevent data exfiltration; violations are recorded in Policy Denied audit logs.
