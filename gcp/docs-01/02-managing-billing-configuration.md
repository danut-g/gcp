# Section 1.2 — Managing Billing Configuration

## Exam Relevance
This topic is part of **Section 1: Setting up a cloud solution environment (~20 % of the exam)**. You must know how to create billing accounts, link projects to them, establish budgets and alerts, and set up billing exports.

---

## 1. Google Cloud Billing Overview

> 📖 **Docs:** [Billing overview](https://cloud.google.com/billing/docs/overview) | [Billing concepts](https://cloud.google.com/billing/docs/concepts) | 🖥️ **Console:** Billing → Overview

### How Billing Works
- **Every project must be linked to a billing account** to create billable resources
- Charges are accumulated at the project level and rolled up to the billing account
- A billing account can be linked to one or more projects
- Payment is processed at the billing account level

### Billing Account Types

| Type | Description |
|------|-------------|
| **Self-serve (online)** | Linked to a credit/debit card or bank account. Costs charged automatically. |
| **Invoiced (offline)** | Available to eligible customers. Google sends monthly invoices. Requires application and approval. |

### Billing Account vs. Project Relationship

```
Billing Account
  ├── Project A (charges: $150/month)
  ├── Project B (charges: $300/month)
  └── Project C (charges: $50/month)
       Total invoice: $500/month
```

---

## 2. Creating Billing Accounts

> 📖 **Docs:** [Create a billing account](https://cloud.google.com/billing/docs/how-to/manage-billing-account) | 🖥️ **Console:** Billing → Manage Billing Accounts → Create Account

### Permissions Required
- **Billing Account Creator** (`roles/billing.creator`) — at the organization level to create new billing accounts
- **Billing Account Administrator** (`roles/billing.admin`) — manage existing billing accounts

### Steps to Create a Billing Account
1. Go to the **Cloud Console → Billing**
2. Click **Manage Billing Accounts → Create Account**
3. Enter account name, country, and currency
4. Select payment method (credit card, bank account, or invoiced)
5. Complete verification

### gcloud Commands

```bash
# List all billing accounts accessible to you
gcloud billing accounts list

# Describe a billing account
gcloud billing accounts describe BILLING_ACCOUNT_ID

# List projects linked to a billing account
gcloud billing projects list --billing-account=BILLING_ACCOUNT_ID
```

### Key Points
- A billing account can be linked to **multiple projects**
- A project can only be linked to **one billing account** at a time
- Billing accounts are separate from the resource hierarchy (they sit alongside it)
- Organization-level billing accounts can be managed centrally

---

## 3. Linking Projects to a Billing Account

> 📖 **Docs:** [Modify a project's billing account](https://cloud.google.com/billing/docs/how-to/modify-project) | 🖥️ **Console:** Billing → Account management → Projects

### Why Link Projects?
- Without a billing account, a project cannot create resources that incur charges
- Free-tier resources may still work, but billable resources will fail to create

### How to Link

```bash
# Link a project to a billing account
gcloud billing projects link PROJECT_ID \
  --billing-account=BILLING_ACCOUNT_ID

# Unlink a project from billing (disables billing — resources may be stopped)
gcloud billing projects unlink PROJECT_ID

# Check which billing account a project is linked to
gcloud billing projects describe PROJECT_ID
```

### Permissions Required
- **Billing Account User** (`roles/billing.user`) on the billing account AND
- **Project Owner** or **Project Billing Manager** on the project

### What Happens When You Unlink Billing?
- **Resources are not immediately deleted**, but:
  - Compute Engine VMs are stopped
  - Cloud Storage buckets become read-only
  - Most services stop functioning
- After a grace period, resources may be deleted
- Re-linking restores billing but stopped resources must be manually restarted

---

## 4. Establishing Billing Budgets and Alerts

> 📖 **Docs:** [Create and manage budgets](https://cloud.google.com/billing/docs/how-to/budgets) | [Budget alerts](https://cloud.google.com/billing/docs/how-to/budget-notifications) | 🖥️ **Console:** Billing → Budgets & Alerts

### Billing Budgets
A budget lets you track your actual (and forecasted) Google Cloud spending against a planned amount.

### Types of Budget Amounts
- **Specified amount** — A fixed dollar amount (e.g., $1,000/month)
- **Last month's spend** — Dynamically set based on the previous month's charges

### Budget Scope
You can scope a budget to:
- Entire billing account
- Specific projects
- Specific products/services
- Specific labels

### Alert Thresholds
- Default thresholds: **50 %**, **90 %**, **100 %** of budget
- You can customize these percentages
- Alerts can be based on **actual spend** or **forecasted spend**

### Alert Notification Options
1. **Email to billing admins and users** — Default notification method
2. **Cloud Monitoring notification channels** — Email, SMS, Slack, PagerDuty, etc.
3. **Pub/Sub topic** — For programmatic responses (e.g., auto-disable billing, send custom alerts)

### Setting Up Budgets via Console
1. Go to **Billing → Budgets & Alerts**
2. Click **Create Budget**
3. Name the budget and select scope (billing account, projects, services)
4. Set the budget amount
5. Configure alert thresholds
6. Choose notification method

### Programmatic Budget Response (Pub/Sub)

```
Budget Alert → Pub/Sub Topic → Cloud Function → Action
                                                  ├── Disable billing
                                                  ├── Scale down resources
                                                  ├── Send Slack notification
                                                  └── Cap spending
```

### Key Exam Points
- **Budgets do NOT cap spending** — They only send notifications
- To actually stop spending, you need a programmatic solution (Cloud Function + Pub/Sub)
- Forecasted alerts can warn you before you hit the threshold
- Budgets can be scoped to projects, services, or labels for granular tracking

---

## 5. Setting Up Billing Exports

> 📖 **Docs:** [Export billing to BigQuery](https://cloud.google.com/billing/docs/how-to/export-data-bigquery) | [Export to Cloud Storage](https://cloud.google.com/billing/docs/how-to/export-data-file) | 🖥️ **Console:** Billing → Billing Export

Billing exports allow you to send detailed billing data to external destinations for analysis and reporting.

### Export Destinations

| Destination | Use Case | Detail Level |
|-------------|----------|--------------|
| **BigQuery** | Detailed analysis, custom reports, dashboards | Line-item detail (most granular) |
| **Cloud Storage (CSV/JSON)** | Archival, external tool ingestion | Daily summary |

### BigQuery Export (Recommended)

#### Types of BigQuery Export
1. **Standard usage cost** — Detailed cost and usage data (most commonly used)
2. **Detailed usage cost** — Even more granular (resource-level) cost data
3. **Pricing data** — Export of Google Cloud pricing information

#### Setting Up BigQuery Export
1. Go to **Billing → Billing Export**
2. Select the **BigQuery Export** tab
3. Choose or create a BigQuery dataset
4. Select the export type(s)
5. Click **Save**

#### Key Properties
- Export is **near real-time** (data appears within hours)
- Historical data is NOT backfilled (export starts from the enablement date)
- The dataset must be in the same organization as the billing account
- BigQuery export is **free** (you only pay for queries on the exported data)

### Example BigQuery Query

```sql
-- Total cost by service for the current month
SELECT
  service.description AS service,
  SUM(cost) AS total_cost,
  SUM(IFNULL((SELECT SUM(c.amount) FROM UNNEST(credits) c), 0)) AS total_credits
FROM
  `project.dataset.gcp_billing_export_v1_XXXXXX`
WHERE
  invoice.month = FORMAT_DATE('%Y%m', CURRENT_DATE())
GROUP BY
  service
ORDER BY
  total_cost DESC;
```

```sql
-- Daily cost trend
SELECT
  DATE(usage_start_time) AS usage_date,
  SUM(cost) AS daily_cost
FROM
  `project.dataset.gcp_billing_export_v1_XXXXXX`
WHERE
  invoice.month = FORMAT_DATE('%Y%m', CURRENT_DATE())
GROUP BY
  usage_date
ORDER BY
  usage_date;
```

### Cloud Storage Export (Legacy)
- Exports daily CSV or JSON files to a Cloud Storage bucket
- Less detailed than BigQuery export
- Useful for feeding data into external tools
- **Note**: Google recommends BigQuery export as the primary method

---

## 6. Billing Roles and Permissions

> 📖 **Docs:** [Billing access control](https://cloud.google.com/billing/docs/how-to/billing-access) | [Billing IAM roles](https://cloud.google.com/billing/docs/how-to/billing-access#billing-roles) | 🖥️ **Console:** Billing → Account management → Permissions

### Key Billing IAM Roles

| Role | Description |
|------|-------------|
| `roles/billing.creator` | Create new billing accounts |
| `roles/billing.admin` | Manage billing accounts (view costs, link/unlink projects, manage payments) |
| `roles/billing.user` | Link projects to billing accounts |
| `roles/billing.viewer` | View billing account cost and transactions |
| `roles/billing.projectManager` | Link/unlink projects from a billing account |
| `roles/billing.costsManager` | Manage budgets and view/export costs (no payment management) |

### Best Practices for Billing Roles
- Use **billing.viewer** for teams that need cost visibility but no management rights
- Use **billing.user** for project creators who need to link new projects to billing
- Use **billing.admin** sparingly — it has broad permissions including payment management
- Assign billing roles at the **organization level** for centralized control

---

## 7. Cost Management Tools

> 📖 **Docs:** [View your billing reports](https://cloud.google.com/billing/docs/how-to/reports) | [GCP Pricing Calculator](https://cloud.google.com/products/calculator) | 🖥️ **Console:** Billing → Reports / Cost breakdown

### Pricing Calculator
- Estimate costs before deploying resources
- Available at cloud.google.com/products/calculator
- Model different configurations and compare costs

### Cost Table (Billing Reports)
- Visual breakdown of costs in the Cloud Console
- Filter by project, service, SKU, time period, and labels
- View trends, forecasts, and cost anomalies

### Committed Use Discounts (CUDs)
- 1-year or 3-year commitments for Compute Engine and Cloud SQL
- **Savings**: Up to 57% for 1-year, up to 70% for 3-year
- Applies to CPU and memory (not GPUs or premium OS licenses)
- Purchased at the project level

### Sustained Use Discounts (SUDs)
- **Automatic discounts** for running Compute Engine resources more than 25% of the month
- No commitment required
- Up to 30% discount for full-month usage
- Applied automatically at the project level

### Labels for Cost Allocation
- Key-value pairs attached to resources (e.g., `environment:production`, `team:backend`)
- Used to filter and group costs in billing reports
- Can be used as budget scope criteria

```bash
# Add labels to a VM instance
gcloud compute instances add-labels INSTANCE_NAME \
  --labels=environment=production,team=backend

# Add labels to a Cloud Storage bucket
gcloud storage buckets update gs://BUCKET_NAME \
  --update-labels=environment=staging
```

---

## 8. Cost Controls Beyond Budgets

> 📖 **Docs:** [Budget programmatic notifications](https://cloud.google.com/billing/docs/how-to/notify) | [Cap spending with Cloud Functions](https://cloud.google.com/billing/docs/how-to/notify#cap_disable_billing_to_stop_usage) | 🖥️ **Console:** Billing → Budgets & Alerts → Connect a Pub/Sub topic

Budgets alert but do **NOT** stop spending — they are notification-only. To actually stop spending when a budget is exceeded, combine budgets with programmatic controls:

**Option 1 — Disable billing via Pub/Sub + Cloud Function:**

```python
# Cloud Function triggered by budget Pub/Sub notification
def stop_billing(data, context):
    import base64, json
    from googleapiclient import discovery
    pubsub_data = json.loads(base64.b64decode(data['data']).decode('utf-8'))
    if pubsub_data['costAmount'] >= pubsub_data['budgetAmount']:
        # Remove billing account from project
        billing = discovery.build('cloudbilling', 'v1')
        billing.projects().updateBillingInfo(
            name='projects/MY_PROJECT',
            body={'billingAccountName': ''}
        ).execute()
```

**Option 2** — Organization policy to restrict resources (e.g., deny VM creation in certain regions) — manual but governance-based.

**Option 3** — Quotas: request a quota reduction to cap resource usage (e.g., limit number of VMs or CPUs per region).

Exam tip: A common exam scenario asks how to PREVENT overspending — the answer is NOT budgets alone; it's budgets + Pub/Sub + Cloud Function to disable billing, or quota reductions.

---

## 9. Billing Export and Analysis

> 📖 **Docs:** [Query exported data in BigQuery](https://cloud.google.com/billing/docs/how-to/bq-examples) | 🖥️ **Console:** Billing → Billing Export → BigQuery Export → Edit settings

```bash
# Enable billing export to BigQuery
# (Must be done via Console: Billing → Billing export → BigQuery export → Edit settings)
# Then query in BigQuery:
SELECT
  service.description AS service,
  SUM(cost) AS total_cost,
  SUM(IFNULL((SELECT SUM(c.amount) FROM UNNEST(credits) c), 0)) AS total_credits,
  SUM(cost) + SUM(IFNULL((SELECT SUM(c.amount) FROM UNNEST(credits) c), 0)) AS net_cost
FROM `PROJECT.DATASET.gcp_billing_export_v1_ACCOUNT_ID`
WHERE DATE(usage_start_time) BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY) AND CURRENT_DATE()
GROUP BY 1 ORDER BY net_cost DESC;

-- Cost by project
SELECT project.name, SUM(cost) AS total FROM `...` GROUP BY 1 ORDER BY 2 DESC;

-- Cost by label (e.g., team=backend)
SELECT labels.value AS team, SUM(cost)
FROM `...`, UNNEST(labels) AS labels
WHERE labels.key = 'team'
GROUP BY 1;
```

- **Standard export**: daily granularity, resource-level cost
- **Detailed export (recommended)**: hourly granularity, VM/disk-level cost, includes resource hierarchy
- Exam tip: Billing export to BigQuery is the only way to do fine-grained historical cost analysis; Cloud Console billing shows real-time but limited drill-down

---

## 10. Billing Account Hierarchy

> 📖 **Docs:** [Billing account concepts](https://cloud.google.com/billing/docs/concepts#billing_account) | [Billing in organization](https://cloud.google.com/billing/docs/onboarding-checklist) | 🖥️ **Console:** Billing → Manage Billing Accounts

```
Organization
  └── Billing Account(s)
       └── Linked Projects
            └── Resources (charges roll up to project, then billing account)
```

### Key Exam Points on Hierarchy
- A **billing account** exists at the **organization level** (not within folders/projects)
- A billing account can be linked to projects in different folders
- Multiple billing accounts can exist under one organization (e.g., per department)
- Billing account admins are different from project admins

---

## Exam Practice Questions

1. **A project creator needs to set up a new project and link it to the company's billing account. Which roles do they need?**
   - Answer: `roles/billing.user` on the billing account AND `roles/resourcemanager.projectCreator` on the organization or folder.

2. **Your team wants to automatically shut down resources when spending exceeds $5,000. How do you accomplish this?**
   - Answer: Create a budget with a Pub/Sub notification. Create a Cloud Function triggered by the Pub/Sub topic that disables billing on the project (or scales down resources).

3. **You need to analyze your Google Cloud spending by team and environment. What should you do?**
   - Answer: Apply labels (team, environment) to all resources. Enable BigQuery billing export. Query the export data filtered by labels.

4. **What happens to running VMs if a project's billing account is removed?**
   - Answer: The VMs will be stopped. They won't be immediately deleted, but they won't run. After a grace period, the project and its resources may be deleted.

5. **Which billing export method provides the most detailed cost data?**
   - Answer: BigQuery export (specifically, the "Detailed usage cost" export type) provides the most granular, resource-level cost data.

6. **Can a billing budget automatically stop spending when the threshold is reached?**
   - Answer: No. Billing budgets only send alerts/notifications. To cap spending, you need a programmatic solution using Pub/Sub and Cloud Functions.

---

## Glossary

**AES-256** — Advanced Encryption Standard with a 256-bit key; the encryption algorithm used by GCP to protect data at rest by default.

**Alert Threshold** — A percentage of the billing budget (commonly 50%, 90%, 100%) at which a billing alert notification is sent; can be applied to actual or forecasted spend.

**Billing Account** — A GCP resource that defines who is responsible for paying for a set of linked projects; exists at the organization level and processes payments.

**Billing Account Administrator** (`roles/billing.admin`) — IAM role granting full management of a billing account, including viewing costs, linking/unlinking projects, and managing payments.

**Billing Account Creator** (`roles/billing.creator`) — IAM role granted at the organization level that permits creation of new billing accounts.

**Billing Account User** (`roles/billing.user`) — IAM role that allows linking projects to a billing account; typically combined with Project Owner or Project Billing Manager.

**Billing Account Viewer** (`roles/billing.viewer`) — IAM role granting read-only access to billing account cost data and transactions.

**Billing Budget** — A GCP construct that tracks actual and forecasted spending against a defined amount and sends alert notifications when thresholds are crossed; does not cap spending.

**Billing Export** — Feature that sends detailed billing data to BigQuery or Cloud Storage for analysis and reporting purposes.

**Billing Project Manager** (`roles/billing.projectManager`) — IAM role allowing the holder to link or unlink projects from a billing account without full billing admin rights.

**BigQuery** — Google Cloud's serverless data warehouse; the recommended destination for billing exports due to its fine-grained, queryable, near-real-time cost data.

**BYOL (Bring Your Own License)** — Licensing model where a customer uses their existing software licenses on cloud infrastructure, relevant for Sole-Tenant Nodes on Compute Engine.

**Cloud Console** — The web-based graphical interface for managing GCP resources; used for creating billing accounts, setting budgets, and configuring billing exports.

**Cloud Function** — Serverless, event-driven function used in billing cost-control scenarios to programmatically disable billing when a Pub/Sub budget alert is triggered.

**Cloud Monitoring** — GCP metrics, dashboards, and alerting service; its notification channels (email, SMS, Slack, PagerDuty, etc.) can be used to deliver billing budget alerts.

**Cloud SQL** — GCP's managed relational database; eligible for Committed Use Discounts alongside Compute Engine.

**Cloud Storage** — GCP's object storage service; a destination for legacy billing exports (CSV/JSON) and a resource whose costs appear in billing reports.

**Coldline** — Cloud Storage class designed for data accessed less than once per quarter; lower storage cost but higher retrieval cost than Nearline.

**Committed Use Discounts (CUDs)** — Discounts of up to 57% (1-year) or 70% (3-year) on Compute Engine CPU and memory in exchange for a usage commitment; purchased at the project level.

**Compute Engine** — GCP's virtual machine service; subject to Sustained Use Discounts and Committed Use Discounts.

**Cost Anomaly** — An unexpected spike or change in spending visible in the Cloud Console billing reports and cost table.

**Cost Table** — Cloud Console billing report view that itemizes costs by project, service, SKU, label, and time period for drill-down analysis.

**Costs Manager** (`roles/billing.costsManager`) — IAM role allowing management of budgets and viewing/exporting costs without payment management capabilities.

**Credit** — A reduction applied to billable cost from free-tier usage, promotional offers, committed-use discounts, or service-level-agreement remediation.

**CSV (Comma-Separated Values)** — File format used by the legacy Cloud Storage billing export method.

**Dataflow** — GCP's managed stream and batch data processing service; can appear as a line item in billing reports.

**Detailed Usage Cost Export** — The most granular BigQuery billing export type, providing hourly, resource-level (VM/disk) cost data including resource hierarchy information.

**Firestore** — GCP's serverless NoSQL document database; appears as a billable service line item.

**Folder** — Optional grouping node between the organization and projects in the GCP resource hierarchy; a billing account can be linked to projects across any folder.

**Forecasted Spend** — Projected billing amount at the end of the current billing period based on current usage trends; can trigger alerts before actual thresholds are reached.

**Free Tier** — A set of GCP resources available at no cost each month, including limited Compute Engine, Cloud Storage, BigQuery, and other service usage.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services.

**gcloud CLI** — Command-line interface for GCP; used to list billing accounts, link/unlink projects, and query project billing status.

**GPU (Graphics Processing Unit)** — Specialized hardware accelerator; excluded from Committed Use Discounts on Compute Engine.

**IAM (Identity and Access Management)** — GCP's access control system; used to assign billing-specific roles to control who can manage billing accounts and budgets.

**Invoiced Billing** — A billing account type available to eligible large customers where Google sends monthly invoices instead of charging a card automatically.

**JSON (JavaScript Object Notation)** — File format used by the legacy Cloud Storage billing export method alongside CSV.

**Label** — A key-value pair (e.g., `environment:production`) attached to GCP resources; used to filter and group costs in billing reports and as budget scope criteria.

**Nearline** — Cloud Storage class for data accessed less than once per month; lower storage cost but higher retrieval cost than Standard.

**OLAP (Online Analytical Processing)** — Workload category focused on complex analytical queries over large datasets; BigQuery is designed for OLAP.

**OLTP (Online Transactional Processing)** — Workload category focused on fast read/write transactions; Cloud SQL and Spanner serve OLTP workloads.

**Organization** — Top-level node in the GCP resource hierarchy; billing accounts exist at this level and can be linked to projects across any folder.

**PagerDuty** — Third-party incident management platform that can receive Cloud Monitoring alert notifications triggered by billing budget thresholds.

**Permission** — The right to perform a specific action on a resource (e.g., `billing.accounts.create`); grouped into roles and assigned to principals via IAM policy bindings.

**Pricing Calculator** — Google Cloud tool at cloud.google.com/products/calculator for estimating GCP resource costs before deployment.

**Principal** — An identity (user, group, service account, or domain) that can be granted IAM roles such as billing roles.

**Project** — Core GCP organizational unit; charges accumulate at the project level and roll up to the linked billing account.

**Project Billing Manager** — See `roles/billing.projectManager`; a role allowing project-to-billing-account linking without full billing admin permissions.

**Project Owner** (`roles/owner`) — Basic IAM role required alongside `roles/billing.user` to link a project to a billing account.

**Pub/Sub (Cloud Pub/Sub)** — GCP's asynchronous messaging service; used as the notification channel for billing budget alerts to enable programmatic spending controls.

**Quota** — A per-project limit on API usage or resource count; reducing quotas is one way to cap spending beyond what budget alerts provide.

**Resource** — An individual GCP service instance (VM, bucket, database, etc.); each resource belongs to a project whose charges roll up to the linked billing account.

**Role** — A named collection of IAM permissions granted to a principal on a resource; billing-specific roles include `roles/billing.admin`, `roles/billing.user`, `roles/billing.viewer`, and others.

**Self-Serve Billing** — A billing account type linked to a credit/debit card or bank account where charges are processed automatically; the default for most GCP customers.

**SKU (Stock Keeping Unit)** — A unique identifier for a specific GCP resource or service configuration used to itemize costs in billing reports.

**Slack** — Third-party collaboration tool that can receive Cloud Monitoring alert notifications, including billing budget alerts.

**Spanner (Cloud Spanner)** — GCP's globally distributed relational database; appears as a billable service line item.

**Standard Usage Cost Export** — BigQuery billing export type providing daily, resource-level cost and usage data; the most commonly used export type.

**Sustained Use Discounts (SUDs)** — Automatic Compute Engine discounts of up to 30% applied when a VM runs for more than 25% of a billing month; requires no commitment.

**VM (Virtual Machine)** — A Compute Engine instance; running VMs are a common billable line item and are stopped when a project is unlinked from its billing account.

**VPC (Virtual Private Cloud)** — GCP network resource whose egress traffic can incur billing charges, visible in billing reports.
