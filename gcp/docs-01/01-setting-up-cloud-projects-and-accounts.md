# Section 1.1 — Setting Up Cloud Projects and Accounts

## Exam Relevance
This topic is part of **Section 1: Setting up a cloud solution environment (~23 % of the exam)**. You must know how to create and manage the resource hierarchy, apply organizational policies, grant IAM roles, manage Cloud Identity users/groups, enable APIs, set up Cloud Observability products, assess quotas, set up standalone organizations, confirm product availability in geographical locations, configure Cloud Asset Inventory, and use Gemini Cloud Assist to analyze resources.

---

## 1. Google Cloud Resource Hierarchy

> 📖 **Docs:** [Resource hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy) | 🖥️ **Console:** IAM & Admin → Resource Manager

The resource hierarchy is the foundation of how Google Cloud organizes resources. It provides a structure for managing access control (IAM) and organizational policies.

### Hierarchy Levels (Top to Bottom)

```
Organization (example: mycompany.com)
  └── Folders (optional, can be nested)
       └── Projects
            └── Resources (VMs, buckets, databases, etc.)
```

### Organization Node
- **Top-level node** tied to a Google Workspace or Cloud Identity domain
- Automatically created when a company signs up for Google Workspace or Cloud Identity
- Provides centralized visibility and control over all resources
- The **Organization Admin** role grants full control at this level
- Example: `organizations/123456789`

### Folders
- **Optional grouping mechanism** beneath the organization node
- Can represent departments, teams, environments (dev/staging/prod), or applications
- Can be nested up to **10 levels deep**
- IAM policies and organization policies applied to a folder are **inherited** by all child resources
- Example: A "Development" folder containing projects for dev, QA, and staging

### Projects
- **Core organizational unit** in Google Cloud
- Every resource belongs to exactly one project
- Each project has three identifiers:
  - **Project Name** — human-readable, not unique (e.g., "My Web App")
  - **Project ID** — globally unique, immutable, chosen at creation (e.g., `my-web-app-12345`)
  - **Project Number** — globally unique, auto-assigned (e.g., `123456789012`)
- Projects are the basis for:
  - Enabling APIs
  - Billing
  - Managing resources
  - Granting permissions

### Resources
- Individual Google Cloud services (VMs, Cloud Storage buckets, BigQuery datasets, etc.)
- Always belong to a single project
- Inherit IAM policies from their parent project, folder, and organization

### Key Exam Concepts
- **Policy inheritance flows downward**: Organization → Folders → Projects → Resources
- **More permissive policies win**: If a parent grants Editor and a child grants Viewer, the effective role is Editor
- **Projects are mandatory**: You cannot create resources without a project
- **Folders are optional**: Small organizations may have Organization → Projects directly
- **Project lifecycle**: Projects have states — ACTIVE, DELETE_REQUESTED, and DELETE_IN_PROGRESS. A deleted project enters DELETE_REQUESTED for ~30 days during which it can be restored with `gcloud projects undelete`; after that window the project ID is released and resources are permanently deleted.
- **Project ID is immutable**: Once chosen, a Project ID cannot be changed; deleting and recreating is the only way (and the ID may not be immediately reusable).

### gcloud Commands

```bash
# List all projects
gcloud projects list

# Create a new project
gcloud projects create PROJECT_ID --name="Project Name" --folder=FOLDER_ID

# Set active project
gcloud config set project PROJECT_ID

# Describe a project
gcloud projects describe PROJECT_ID

# Delete a project (30-day recovery window)
gcloud projects delete PROJECT_ID

# Restore (undelete) a project within the 30-day recovery window
gcloud projects undelete PROJECT_ID

# List organizations
gcloud organizations list

# List folders under an organization or parent folder
gcloud resource-manager folders list --organization=ORG_ID
gcloud resource-manager folders list --folder=PARENT_FOLDER_ID

# Create a folder
gcloud resource-manager folders create --display-name="Folder Name" --organization=ORG_ID
```

---

## 2. Organizational Policies

> 📖 **Docs:** [Organization Policy overview](https://cloud.google.com/resource-manager/docs/organization-policy/overview) | 🖥️ **Console:** IAM & Admin → Organization Policies

Organizational policies provide **centralized, top-down constraints** on how resources can be configured across the hierarchy. They complement IAM (which controls *who* can do things) by controlling *what* can be done.

### How Organization Policies Work
- Defined by the **Organization Policy Service**
- Applied at the organization, folder, or project level
- Use **constraints** — rules that restrict specific resource configurations
- Inherited downward through the hierarchy (with some override options)

### Common Constraints (Exam Favorites)

| Constraint | What It Does |
|-----------|--------------|
| `constraints/compute.vmExternalIpAccess` | Restricts which VMs can have external IPs |
| `constraints/compute.restrictSharedVpcSubnetworks` | Controls which Shared VPC subnets can be used |
| `constraints/iam.allowedPolicyMemberDomains` | Restricts which domains can be granted IAM roles |
| `constraints/compute.trustedImageProjects` | Limits which image projects can be used for boot disks |
| `constraints/storage.uniformBucketLevelAccess` | Requires uniform bucket-level access on Cloud Storage |
| `constraints/compute.restrictVpcPeering` | Controls which networks can be peered |
| `constraints/gcp.resourceLocations` | Restricts where resources can be created (regions/zones) |

### gcloud Commands

```bash
# List available constraints
gcloud org-policies list-available-constraints --organization=ORG_ID

# Describe a specific constraint
gcloud org-policies describe constraints/compute.vmExternalIpAccess --organization=ORG_ID

# Set a boolean constraint (e.g., disable external IPs)
gcloud org-policies set-policy POLICY_FILE.yaml --organization=ORG_ID

# Reset a policy to default
gcloud org-policies reset constraints/compute.vmExternalIpAccess --organization=ORG_ID
```

### Example Policy YAML

```yaml
constraint: constraints/compute.vmExternalIpAccess
listPolicy:
  allValues: DENY
```

---

## 3. Granting IAM Roles Within a Project

> 📖 **Docs:** [IAM overview](https://cloud.google.com/iam/docs/overview) | [Granting roles](https://cloud.google.com/iam/docs/granting-changing-revoking-access) | 🖥️ **Console:** IAM & Admin → IAM

### IAM Model
IAM follows the model: **Who** (Member) has **What access** (Role) to **Which resource**.

### Members (Principals)
- **Google Account** — individual user (user:alice@example.com)
- **Service Account** — application identity (serviceAccount:sa@project.iam.gserviceaccount.com)
- **Google Group** — named group of accounts (group:devteam@example.com)
- **Google Workspace Domain** — all users in a domain (domain:example.com)
- **allAuthenticatedUsers** — any authenticated Google account
- **allUsers** — anyone on the internet (use with extreme caution)

### Role Types

| Type | Description | Example |
|------|-------------|---------|
| **Basic (Primitive)** | Broad permissions; use sparingly | `roles/owner`, `roles/editor`, `roles/viewer` |
| **Predefined** | Fine-grained, service-specific | `roles/compute.instanceAdmin.v1`, `roles/storage.objectViewer` |
| **Custom** | User-defined subset of permissions | `roles/myCustomRole` |

### Basic Roles in Detail
- **Viewer** (`roles/viewer`) — Read-only access to all resources
- **Editor** (`roles/editor`) — Read/write access to all resources (cannot manage IAM)
- **Owner** (`roles/owner`) — Full control including IAM management and billing
- **Browser** (`roles/browser`) — Permission to browse the hierarchy (folders, projects)

### IAM Policy Binding
A **policy binding** associates a member with a role at a specific level of the hierarchy.

```bash
# Grant a user the Viewer role on a project
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/viewer"

# Grant a group the Compute Admin role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="group:devteam@example.com" \
  --role="roles/compute.admin"

# Remove a role binding
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/viewer"

# View the IAM policy for a project
gcloud projects get-iam-policy PROJECT_ID

# View IAM policy in a readable format
gcloud projects get-iam-policy PROJECT_ID --format=json
```

### IAM Conditions
- Allow granting roles based on attributes of the request (time, resource type, resource name, etc.)
- Example: Grant access only during business hours, or only to resources with a specific name prefix

```bash
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/compute.instanceAdmin.v1" \
  --condition='expression=request.time < timestamp("2025-12-31T00:00:00Z"),title=temporary-access'
```

---

## 4. Managing Users and Groups in Cloud Identity

> 📖 **Docs:** [Cloud Identity overview](https://cloud.google.com/identity/docs/overview) | [GCDS sync](https://support.google.com/a/answer/106368) | 🖥️ **Console:** admin.google.com → Directory → Users / Groups

### Cloud Identity
Cloud Identity is Google's Identity as a Service (IDaaS) solution. It provides:
- User account management
- Group management
- Device management
- Single sign-on (SSO) configuration
- Multi-factor authentication (MFA)

### Manual User Management
- Create users through the **Google Admin Console** (admin.google.com)
- Assign users to organizational units (OUs)
- Enable/disable accounts
- Reset passwords

### Automated User Management
- **Google Cloud Directory Sync (GCDS)** — synchronizes users and groups from Active Directory or LDAP to Cloud Identity
- **SCIM (System for Cross-domain Identity Management)** — automate provisioning from third-party identity providers
- **Admin SDK Directory API** — programmatic user and group management

### Google Groups for IAM
- Create a Google Group (e.g., `cloud-admins@example.com`)
- Add users to the group
- Grant IAM roles to the group instead of individual users
- **Best practice**: Use groups for IAM role assignments, not individual users

### Key Exam Points
- Cloud Identity Free edition provides basic user and group management
- Cloud Identity Premium adds device management, advanced security, and app management
- Google Workspace includes Cloud Identity capabilities
- Groups are the recommended way to manage access at scale

---

## 5. Enabling APIs Within Projects

> 📖 **Docs:** [APIs & Services overview](https://cloud.google.com/apis/docs/overview) | [Enable/disable APIs](https://cloud.google.com/apis/docs/getting-started) | 🖥️ **Console:** APIs & Services → Library

### Why APIs Must Be Enabled
- Google Cloud services are exposed as APIs
- By default, most APIs are **disabled** in new projects
- You must explicitly enable an API before using its service
- This provides security (reduced attack surface) and cost control

### How to Enable APIs

```bash
# Enable a single API
gcloud services enable compute.googleapis.com

# Enable multiple APIs at once
gcloud services enable \
  compute.googleapis.com \
  storage.googleapis.com \
  container.googleapis.com

# List enabled APIs
gcloud services list --enabled

# List all available APIs
gcloud services list --available

# Disable an API
gcloud services disable compute.googleapis.com
```

### Common APIs to Know

| API | Service |
|-----|---------|
| `compute.googleapis.com` | Compute Engine |
| `container.googleapis.com` | Google Kubernetes Engine |
| `storage.googleapis.com` | Cloud Storage |
| `sqladmin.googleapis.com` | Cloud SQL |
| `bigquery.googleapis.com` | BigQuery |
| `run.googleapis.com` | Cloud Run |
| `cloudfunctions.googleapis.com` | Cloud Functions |
| `iam.googleapis.com` | IAM |
| `cloudresourcemanager.googleapis.com` | Resource Manager |
| `monitoring.googleapis.com` | Cloud Monitoring |
| `logging.googleapis.com` | Cloud Logging |

### Key Exam Points
- If you get an error like "API not enabled," you need to enable the relevant API
- Some APIs are enabled by default (e.g., BigQuery in some project configurations)
- Enabling an API does not incur charges — only using the service does
- API enablement can be managed through organization policies

---

## 6. Provisioning Google Cloud Observability

> 📖 **Docs:** [Cloud Monitoring overview](https://cloud.google.com/monitoring/docs/overview) | [Cloud Logging overview](https://cloud.google.com/logging/docs/overview) | 🖥️ **Console:** Monitoring → Overview / Logging → Logs Explorer

**Google Cloud Observability** (formerly Operations Suite / Stackdriver) includes:

### Cloud Monitoring
- Collects metrics from GCP resources, AWS, and custom applications
- Dashboards, alerts, uptime checks
- **Workspace** — A Monitoring workspace can monitor one or more projects

### Cloud Logging
- Centralized log management
- Ingests logs from GCP services, VMs (via agent), and applications
- Log Router sends logs to destinations (Cloud Storage, BigQuery, Pub/Sub)

### Cloud Trace
- Distributed tracing for latency analysis
- Automatically traces requests in App Engine, Cloud Run, and Cloud Functions

### Cloud Profiler
- Continuous CPU and memory profiling for applications

### Error Reporting
- Aggregates and displays errors from cloud services and applications

### Setting Up Operations Suite

```bash
# Monitoring and Logging APIs are often enabled by default
gcloud services enable monitoring.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable cloudtrace.googleapis.com
gcloud services enable clouderrorreporting.googleapis.com
gcloud services enable cloudprofiler.googleapis.com
```

---

## 7. Organization Policy Enforcement

> 📖 **Docs:** [Organization Policy Service](https://cloud.google.com/resource-manager/docs/organization-policy/overview) | [Available constraints](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints) | 🖥️ **Console:** IAM & Admin → Organization Policies

Organization policies apply constraints at organization, folder, or project level — independent of IAM.

- A user with IAM Owner on a project **cannot** do something that an org policy prohibits at the org level
- Constraints are inherited downward; a child cannot override to be more permissive (unless the org policy allows override via `restoreDefault`)

### Key Constraints for the Exam

```bash
# List all available constraints
gcloud resource-manager org-policies list-available-constraints --organization=ORG_ID

# View effective policy on a project (includes inherited from parent)
gcloud resource-manager org-policies describe constraints/compute.vmExternalIpAccess --project=PROJECT_ID --effective

# Set a boolean constraint (enforce = deny the action)
gcloud resource-manager org-policies enable-enforce constraints/compute.requireOsLogin --project=PROJECT_ID

# Set a list constraint (deny all, or allow specific values)
cat > policy.yaml <<EOF
constraint: constraints/compute.vmExternalIpAccess
listPolicy:
  allValues: DENY
EOF
gcloud resource-manager org-policies set-policy --project=PROJECT_ID policy.yaml

# Reset to inherited/default
gcloud resource-manager org-policies delete constraints/compute.requireOsLogin --project=PROJECT_ID
```

Exam tip: IAM controls **WHO** can do something; org policies control **WHAT** can be done anywhere in the hierarchy. They are complementary — both must allow an action for it to succeed.

---

## 8. Assessing Quotas and Requesting Increases

> 📖 **Docs:** [View and manage quotas](https://cloud.google.com/docs/quotas/view-manage) | [Quota types](https://cloud.google.com/docs/quotas/quota-overview) | 🖥️ **Console:** IAM & Admin → Quotas & System Limits

### What Are Quotas?
- **Rate quotas** — reset after a specified time (e.g., API calls per 100 seconds)
- **Allocation quotas** — limit on the number of resources (e.g., max VMs per project, max VPCs per project)

### Why Quotas Exist
- Prevent accidental overspending
- Protect the shared Google infrastructure from abuse
- Ensure fair usage across all customers

### Viewing Quotas

```bash
# View quotas for a specific service
gcloud compute project-info describe --project=PROJECT_ID

# List all quota metrics
gcloud services quotas list --service=compute.googleapis.com --consumer=projects/PROJECT_ID

# View specific quota
gcloud compute regions describe REGION
```

### Console Method
- Navigate to **IAM & Admin → Quotas** in the Cloud Console
- Filter by service, metric, or location
- Select the quota and click **Edit Quotas**

### Requesting Quota Increases
1. Go to the **Quotas** page in the Cloud Console
2. Select the quota you want to increase
3. Click **Edit Quotas**
4. Enter the new limit and provide a justification
5. Submit the request — Google reviews and responds (usually within 24-48 hours)

### Key Exam Points
- Quotas apply at the **project level** (not billing account or organization level by default)
- Quota errors return HTTP 429 (Too Many Requests) or specific quota-exceeded errors
- Some quotas can be adjusted; others are hard limits
- Organization policies can also restrict quota limits
- Monitor quota usage through Cloud Monitoring to avoid hitting limits

---

## 9. Setting Up Standalone Organizations

> 📖 **Docs:** [Set up Cloud Identity](https://cloud.google.com/identity/docs/set-up-cloud-identity-admin) | [Create an organization](https://cloud.google.com/resource-manager/docs/creating-managing-organization) | 🖥️ **Console:** admin.google.com → Account → Domains

A **standalone organization** is a GCP organization node not linked to Google Workspace. It is useful when a company uses a different identity provider (Okta, Azure AD, etc.) but still wants organizational governance in GCP.

### When to Use a Standalone Organization
- Company uses a third-party IdP (Okta, Azure AD) instead of Google Workspace
- Want organizational structure (folders, org policies) without full Google Workspace
- Separate GCP governance from existing Google Workspace domain

### Creating a Standalone Organization
1. Sign up for **Cloud Identity** (free edition supports this)
2. Verify ownership of the domain
3. An Organization node is automatically created
4. Set up identity federation from your existing IdP to Cloud Identity

```bash
# After organization is created, list it
gcloud organizations list

# Grant initial org admin
gcloud organizations add-iam-policy-binding ORG_ID \
  --member="user:admin@yourdomain.com" \
  --role="roles/resourcemanager.organizationAdmin"
```

### Key Exam Points
- An Organization node requires either Google Workspace or Cloud Identity
- Cloud Identity Free provides basic user/group management without Workspace apps
- Organizations can federate with external IdPs via SAML/OIDC
- Setting up standalone organizations is done through the Google Admin Console (admin.google.com)

---

## 10. Confirming Availability of Products in Geographical Locations

> 📖 **Docs:** [GCP locations](https://cloud.google.com/about/locations) | [Products by region](https://cloud.google.com/about/locations#lightbox-regions-map) | 🖥️ **Console:** Region/zone dropdown within each service's creation page

Not all GCP services are available in every region or zone. Confirming availability before designing architecture is critical.

### How to Check Availability

```bash
# List available regions
gcloud compute regions list

# List available zones
gcloud compute zones list

# Describe a region (shows available services)
gcloud compute regions describe us-central1

# Check GKE availability in a region
gcloud container get-server-config --region=us-central1
```

### Cloud Console Method
- Navigate to a service (e.g., Cloud SQL → Create instance)
- The region/zone dropdown only shows available locations
- **GCP Products Availability** page: cloud.google.com/about/locations

### Service Availability Concepts

| Scope | Description | Examples |
|-------|-------------|---------|
| **Global** | Available everywhere | Cloud CDN, Cloud DNS, IAM |
| **Multi-region** | Available in all regions of a continent | BigQuery datasets, Cloud Storage multi-region |
| **Regional** | Available in specific regions | Cloud SQL, GKE, Cloud Run |
| **Zonal** | Available in specific zones | Compute Engine (zonal persistent disks, VMs) |

### Key Exam Points
- Cloud Spanner and BigQuery are available globally but you choose a configuration (regional/multi-region)
- Some machine types (e.g., N2, C2) are only available in certain zones
- GPUs and TPUs have limited zone availability
- Confirm availability before committing to a design — use `gcloud compute regions list` or the Console

---

## 11. Cloud Asset Inventory and Gemini Cloud Assist

> 📖 **Docs (Asset Inventory):** [Cloud Asset Inventory overview](https://cloud.google.com/asset-inventory/docs/overview) | [Search assets](https://cloud.google.com/asset-inventory/docs/searching-resources) | 🖥️ **Console:** Cloud Asset Inventory (search "Asset Inventory")
> 📖 **Docs (Gemini):** [Gemini Cloud Assist](https://cloud.google.com/gemini/docs/discover/use-gemini-with-console) | 🖥️ **Console:** ✦ Gemini icon in top bar

### Cloud Asset Inventory

**Cloud Asset Inventory** provides a unified view of all GCP resources and IAM policies in a project, folder, or organization. It supports change tracking, policy analysis, and resource export.

#### Key Features
- **Asset search** — Find any resource by name, type, or label
- **Policy analysis** — Determine who has access to what resource
- **Change history** — Track resource configuration changes over time
- **Export** — Export inventory to Cloud Storage or BigQuery
- **Real-time feed** — Stream resource changes to Pub/Sub

#### Enabling and Using Cloud Asset Inventory

```bash
# Enable the API
gcloud services enable cloudasset.googleapis.com

# Export all assets in a project to Cloud Storage
gcloud asset export \
  --project=PROJECT_ID \
  --asset-types="compute.googleapis.com/Instance,storage.googleapis.com/Bucket" \
  --content-type=resource \
  --output-path=gs://my-bucket/assets-export.json

# Export to BigQuery
gcloud asset export \
  --project=PROJECT_ID \
  --content-type=resource \
  --bigquery-table=projects/PROJECT_ID/datasets/my_dataset/tables/assets \
  --output-bigquery-force

# Search all resources by name
gcloud asset search-all-resources \
  --scope=projects/PROJECT_ID \
  --query="name:my-vm"

# Search all resources of a specific type
gcloud asset search-all-resources \
  --scope=organizations/ORG_ID \
  --asset-types="compute.googleapis.com/Instance"

# Analyze IAM policy (who has access to what)
gcloud asset analyze-iam-policy \
  --scope=projects/PROJECT_ID \
  --identity="user:alice@example.com"

# Set up a real-time feed for resource changes
gcloud asset feeds create my-feed \
  --project=PROJECT_ID \
  --content-type=resource \
  --asset-types="compute.googleapis.com/Instance" \
  --pubsub-topic=projects/PROJECT_ID/topics/asset-changes
```

#### Common Use Cases
- **Compliance auditing** — Find all unencrypted Cloud Storage buckets across an organization
- **Security review** — Identify VMs with public IPs in production projects
- **Cost attribution** — List all resources by label for chargeback
- **Change management** — Track when resources are created, modified, or deleted

### Gemini Cloud Assist

**Gemini Cloud Assist** is Google Cloud's AI-powered assistant built into the Cloud Console. It provides natural-language interaction for managing, troubleshooting, and understanding GCP resources.

#### Key Capabilities
- **Natural language queries** — Ask questions about resources in plain English
- **Resource analysis** — Analyze Cloud Asset Inventory results with AI assistance
- **Log summarization** — Summarize log errors and suggest root causes
- **Configuration recommendations** — Suggest best-practice configurations
- **Code generation** — Generate gcloud commands, Terraform HCL, and Kubernetes YAML
- **Troubleshooting** — Diagnose application issues based on logs and metrics

#### Using Gemini Cloud Assist

1. Access via the **Gemini icon** (✦) in the Cloud Console top bar
2. Type natural language queries such as:
   - "Show me all Compute Engine instances with external IPs"
   - "Which service accounts have owner role?"
   - "Summarize errors in Cloud Run logs from the last hour"
   - "How do I set up a load balancer for my GKE cluster?"

#### Analyzing Resources with Gemini + Cloud Asset Inventory
- Gemini can query Cloud Asset Inventory and explain results in plain language
- Example: "Are there any Cloud Storage buckets without uniform bucket-level access in my project?"
- Gemini provides step-by-step remediation guidance after identifying issues

#### Key Exam Points
- Gemini Cloud Assist is enabled per-project; may require `cloudaicompanion.googleapis.com` API
- Gemini does not have write access by default — it suggests commands but doesn't execute them
- For monitoring, Gemini can explain alert conditions and suggest metric queries
- Gemini + Cloud Asset Inventory is the primary pattern for org-wide resource analysis in the exam

---

## Exam Practice Questions

1. **Your organization wants to ensure that no VMs in any project can have external IP addresses. Where should you apply this policy?**
   - Answer: Apply an organization policy with the `constraints/compute.vmExternalIpAccess` constraint at the organization level, denying all values.

2. **A developer needs to deploy a Cloud Function but gets an error that the API is not enabled. What should they do?**
   - Answer: Enable the `cloudfunctions.googleapis.com` API using `gcloud services enable cloudfunctions.googleapis.com` or through the Console.

3. **You want to grant a team of 20 developers Compute Engine Admin access. What is the best practice?**
   - Answer: Create a Google Group, add all 20 developers to it, then grant `roles/compute.admin` to the group at the project level.

4. **What is the difference between a project ID and a project number?**
   - Answer: Project ID is user-chosen, globally unique, and immutable. Project number is auto-assigned and globally unique. Both can be used to reference a project.

5. **A newly created project does not have the Compute Engine API enabled. Can the project still incur charges?**
   - Answer: Yes, if other APIs are enabled and used (e.g., Cloud Storage, BigQuery), the project can still incur charges.

6. **Your security team needs to audit which service accounts have been granted `roles/owner` across the entire organization. What is the most efficient approach?**
   - Answer: Use **Cloud Asset Inventory** with `gcloud asset analyze-iam-policy --scope=organizations/ORG_ID --roles=roles/owner` to identify all principals with Owner role across the org.

7. **A developer asks how to find all Compute Engine instances with public IPs in their project. What tool provides this with minimal effort?**
   - Answer: Use **Gemini Cloud Assist** in the Cloud Console with a natural-language query ("Show all VMs with external IPs"), or use `gcloud asset search-all-resources --query="networkInterfaces.accessConfigs.natIP:*"`.

8. **Your company uses Azure Active Directory and wants GCP organizational governance. Do they need Google Workspace?**
   - Answer: No. They can create a **Cloud Identity** account (free or premium), verify their domain, and set up a **standalone organization**. Cloud Identity can federate with Azure AD via SAML/OIDC.

---

## Glossary

**Active Directory** — Microsoft's directory service for Windows domain networks; GCDS can synchronize its users and groups to Cloud Identity.

**ACTIVE (Project State)** — The default lifecycle state of a GCP project indicating that it is fully operational and billable.

**Admin Console (Google Admin Console)** — Web interface at admin.google.com for managing Google Workspace and Cloud Identity users, groups, and organizational units.

**Admin SDK Directory API** — Programmatic API for managing Cloud Identity users, groups, and organizational units in Google Workspace or Cloud Identity environments.

**Allocation Quota** — A quota type that limits the total number of a specific resource (e.g., maximum number of VMs or VPCs per project) rather than the rate of API calls.

**allAuthenticatedUsers** — IAM special identifier representing any user authenticated with a Google account; grants access to all Google-authenticated users.

**allUsers** — IAM special identifier representing anyone on the internet, including unauthenticated users; use with extreme caution as it makes resources publicly accessible.

**API (Application Programming Interface)** — A programmatic interface exposing a GCP service's functionality; APIs must be explicitly enabled per project before the service can be used.

**Basic Role** — One of the original IAM roles (Owner, Editor, Viewer, Browser) that grants broad permissions across all services in a project; also called primitive roles.

**BigQuery** — Google Cloud's serverless data warehouse; referenced in quota and API contexts (bigquery.googleapis.com).

**Billing Account** — A GCP resource that defines who is responsible for paying for a set of linked projects; must be associated with a project for billable resources to function.

**Boolean Constraint** — An organizational policy constraint that enforces or disables a specific behavior across the hierarchy (e.g., require OS Login).

**Browser Role** (`roles/browser`) — Basic IAM role granting permission to browse the resource hierarchy (view folders and projects) without accessing resource data.

**Cloud Identity** — Google's Identity as a Service (IDaaS) solution providing user management, group management, device management, SSO, and MFA for GCP organizations.

**Cloud Identity Free / Premium** — Two tiers of Cloud Identity; Free provides basic user/group management, Premium adds device management, advanced security, and app management.

**Cloud Logging** — GCP centralized log management service; its API (logging.googleapis.com) must be enabled to use it.

**Cloud Monitoring** — GCP metrics, dashboards, and alerting service; its API (monitoring.googleapis.com) must be enabled to use it.

**Cloud Profiler** — GCP continuous CPU and memory profiling service; enabled via cloudprofiler.googleapis.com.

**Cloud Run** — GCP fully managed serverless container platform; enabled via run.googleapis.com.

**Cloud Storage** — GCP object storage service; enabled via storage.googleapis.com.

**Cloud Trace** — GCP distributed tracing service for latency analysis; enabled via cloudtrace.googleapis.com.

**Cloud SQL** — GCP's fully managed relational database service (MySQL, PostgreSQL, SQL Server); enabled via sqladmin.googleapis.com.

**Compute Engine** — GCP's virtual machine service; enabled via compute.googleapis.com.

**Constraint** — A rule within the Organization Policy Service that restricts specific resource configurations across the hierarchy (e.g., `constraints/compute.vmExternalIpAccess`).

**Custom Role** — User-defined IAM role containing a specific subset of permissions, used to enforce least-privilege access beyond what predefined roles offer.

**DELETE_IN_PROGRESS** — Project lifecycle state indicating that a project and its resources are actively being purged; the project can no longer be restored.

**DELETE_REQUESTED** — Project lifecycle state indicating that deletion has been requested; the project can be restored with `gcloud projects undelete` within the ~30-day window.

**Domain** — In IAM, a Google Workspace or Cloud Identity domain (e.g., example.com) that can be granted roles, giving access to all users in that domain.

**Editor Role** (`roles/editor`) — Basic IAM role granting read/write access to all resources within the scope; cannot manage IAM policies.

**Error Reporting** — GCP operations suite service that aggregates and displays errors from cloud services and applications.

**Folder** — Optional grouping node in the GCP resource hierarchy, sitting below the organization and above projects; can be nested up to 10 levels deep.

**GCDS (Google Cloud Directory Sync)** — Tool that synchronizes users and groups from on-premises Active Directory or LDAP to Cloud Identity.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services.

**Gemini Cloud Assist** — Google Cloud's AI-powered assistant integrated into the Cloud Console; provides natural-language resource analysis, log summarization, configuration recommendations, and troubleshooting guidance, including querying Cloud Asset Inventory.

**Cloud Asset Inventory** — GCP service that discovers and inventories all resources and IAM policies across a project, folder, or organization; supports export, search, policy analysis, and real-time change feeds via Pub/Sub.

**gcloud CLI** — Primary command-line interface for interacting with GCP; used to manage projects, IAM, APIs, quotas, and more.

**Google Account** — An individual Google user identity (e.g., user:alice@example.com) that can be used as an IAM principal.

**Google Group** — A named collection of Google accounts and service accounts that can be granted IAM roles collectively; recommended for managing access at scale.

**Google Kubernetes Engine (GKE)** — GCP's managed Kubernetes service; enabled via container.googleapis.com.

**Google Workspace** — Google's suite of cloud-based productivity tools (Gmail, Drive, Docs, etc.) that includes Cloud Identity capabilities.

**HTTP 429** — HTTP status code for "Too Many Requests"; returned when a GCP API rate quota is exceeded.

**IAM (Identity and Access Management)** — GCP's system for controlling who has what access to which resource using the model: member + role + resource.

**IAM Conditions** — Attribute-based access control extensions to IAM policy bindings that restrict when a role applies (e.g., by time, resource name, or resource type).

**IAM Policy** — Document that specifies which members are granted which roles on a particular resource; stored and enforced by GCP.

**IAM Policy Binding** — Association of a member (principal) with a role at a specific level of the resource hierarchy.

**IDaaS (Identity as a Service)** — Cloud-delivered identity management; Cloud Identity is Google's IDaaS offering.

**LDAP (Lightweight Directory Access Protocol)** — Standard protocol for accessing and maintaining distributed directory information; supported as a source for GCDS synchronization.

**List Constraint** — An organizational policy constraint that allows or denies specific values (e.g., which domains can be granted IAM roles).

**Log Router** — Cloud Logging component that routes log entries to destinations such as Cloud Storage, BigQuery, or Pub/Sub.

**Member** — Synonym for principal; the identity being granted a role in an IAM policy binding.

**MFA (Multi-Factor Authentication)** — Security mechanism requiring more than one verification factor to authenticate a user; configurable in Cloud Identity.

**Monitoring Workspace** — A Cloud Monitoring configuration that can monitor resources from one or more projects; hosted by a "scoping project."

**Operations Suite / Cloud Observability** — Formerly Stackdriver; Google Cloud's integrated suite of observability tools including Cloud Monitoring, Cloud Logging, Cloud Trace, Cloud Profiler, and Error Reporting. Now branded as Google Cloud Observability.

**Org Policy (Organization Policy)** — Centralized, top-down constraints applied at the organization, folder, or project level that control what can be done with GCP resources, independent of IAM.

**Organization Admin** — IAM role granting full control at the organization node level in the GCP resource hierarchy.

**Organization Node** — Top-level node in the GCP resource hierarchy, tied to a Google Workspace or Cloud Identity domain, providing centralized control over all resources.

**Organization Policy Service** — GCP service that defines and enforces organizational policy constraints across the resource hierarchy.

**OU (Organizational Unit)** — A subdivision of users within Google Admin Console used to apply policies to groups of users.

**Owner Role** (`roles/owner`) — Basic IAM role granting full control over resources, including IAM management and billing administration.

**Permission** — The right to perform a specific action on a resource (e.g., `compute.instances.create`); permissions are grouped into roles and cannot be granted directly to principals.

**Policy Inheritance** — The mechanism by which IAM policies and organizational policies applied at a parent node (organization, folder) automatically apply to all child resources.

**Predefined Role** — Google-managed, fine-grained IAM role scoped to a specific GCP service (e.g., `roles/compute.instanceAdmin.v1`).

**Principal** — An identity that can be granted IAM roles; includes Google Accounts, service accounts, Google Groups, and domains.

**Project** — Core organizational unit in GCP; every resource belongs to exactly one project, which has a unique Project ID, Project Name, and Project Number.

**Project ID** — Globally unique, immutable, user-chosen identifier for a GCP project (e.g., `my-web-app-12345`).

**Project Number** — Globally unique, auto-assigned numeric identifier for a GCP project (e.g., `123456789012`).

**Pub/Sub (Cloud Pub/Sub)** — GCP asynchronous messaging service; its API (pubsub.googleapis.com) is used in log routing and event-driven architectures.

**Quota** — A limit on the amount of a GCP resource or API calls that a project can use; includes rate quotas (resets over time) and allocation quotas (fixed counts).

**Rate Quota** — A quota type that limits the number of API calls that can be made within a time window (e.g., 100 calls per 100 seconds); resets automatically.

**Region** — A specific geographic location (e.g., `us-central1`) containing multiple zones where GCP resources can be deployed; governed by `constraints/gcp.resourceLocations`.

**Resource** — An individual GCP service instance (VM, bucket, database, etc.) that belongs to exactly one project and inherits IAM and organization policies from its parents.

**Resource Hierarchy** — GCP's organizational structure: Organization → Folders → Projects → Resources, providing the framework for IAM policy inheritance and organizational policies.

**Resource Manager** — GCP service for managing the resource hierarchy (organizations, folders, projects); enabled via cloudresourcemanager.googleapis.com.

**restoreDefault** — Organization policy setting that resets a constraint to its default inherited value, allowing child nodes to be "un-overridden."

**Role** — A named collection of IAM permissions granted to a principal on a resource; types include Basic (Primitive), Predefined, and Custom.

**SCIM (System for Cross-domain Identity Management)** — Open standard protocol for automating user and group provisioning from third-party identity providers to Cloud Identity.

**Service Account** — Special Google account representing an application or VM rather than a human user; identified by an email like `name@project.iam.gserviceaccount.com`.

**SQL Server** — Microsoft's relational database management system; one of the three database engines supported by Cloud SQL.

**SSO (Single Sign-On)** — Authentication scheme allowing users to log in once and access multiple systems; configurable through Cloud Identity.

**Stackdriver** — Former name of Google Cloud's operations suite, now referred to as Cloud Monitoring, Cloud Logging, and related services.

**Viewer Role** (`roles/viewer`) — Basic IAM role granting read-only access to all resources within the scope.

**VM (Virtual Machine)** — Emulated computer instance running on Compute Engine; referenced in organizational policy constraints such as `constraints/compute.vmExternalIpAccess`.

**VPC (Virtual Private Cloud)** — Logically isolated network in GCP; referenced in organizational policy constraints such as `constraints/compute.restrictVpcPeering`.

**Workspace (Google Workspace)** — Google's cloud productivity suite; organizations using it automatically get an Organization node in GCP and Cloud Identity capabilities.

**Zone** — An isolated deployment area within a region (e.g., `us-central1-a`); zonal resources exist in exactly one zone and have low-latency connectivity to other zones in the same region.
