# Billing Accounts, Budgets, Alerts, Quotas, and Export

## Overview

GCP billing is managed through **Billing Accounts** that are linked to projects. Billing configuration includes setting up budgets and alerts to control spending, exporting billing data for analysis, and managing service quotas. Understanding the relationship between billing accounts, projects, and organizational hierarchy is critical for the ACE exam.

---

## Key Concepts

### Billing Accounts

- A **Billing Account** is a GCP resource that defines who is responsible for payment
- Billing accounts live **outside** projects — they are linked to projects but exist independently at the org level (or personal level for individual accounts)
- A single billing account can be linked to **multiple projects**
- Each project can be linked to **only one billing account** at a time
- Types:
  - **Self-serve (online)**: Credit card or bank account; billed monthly or when threshold is reached
  - **Invoiced (offline)**: For large enterprises; Google sends invoices; requires application
- Billing account roles:
  - `roles/billing.admin` — Full control over billing account (set payment, link/unlink projects)
  - `roles/billing.user` — Can link projects to billing accounts
  - `roles/billing.viewer` — Read-only access to billing data
  - `roles/billing.projectManager` — Can enable/disable billing on projects

### Billing Account Hierarchy

- In an organization, billing accounts are typically managed at the **organization level**
- An org can have multiple billing accounts (e.g., one per department or one per cost center)
- Billing accounts are NOT inside the project/folder hierarchy — they are separate entities
- Linking/unlinking a billing account from a project requires `billing.resourceAssociations.create` (part of `billing.user`) on the billing account AND `resourcemanager.projects.createBillingAssignment` on the project

### Sub-accounts

- Available to **Google Cloud Resellers** (partners)
- Sub-accounts roll up to a master billing account
- Not relevant for direct customers, but may appear in exam questions about resellers

---

## Budgets and Alerts

### Budgets

- Budgets are set on a **billing account** and can be scoped to:
  - The entire billing account
  - One or more specific projects
  - One or more specific services (e.g., only Compute Engine)
  - Labels (filter by resource labels)
- Budget amount can be:
  - **Fixed amount** (e.g., $1000/month)
  - **Last month's spend** (dynamic, tracks last period)
- Budget period: monthly (default), quarterly, yearly, or custom date range

### Budget Alerts

- Alert thresholds are percentages of the budget amount (e.g., 50%, 90%, 100%)
- Default thresholds: 50%, 90%, 100% of budget
- You can add up to **5 thresholds** per budget
- Alerts are sent via:
  - **Email notifications** to billing account admins and users (automatic)
  - **Pub/Sub topic** — programmatic integration for automation (e.g., auto-disable billing)
- **Critical behavior**: Budget alerts DO NOT automatically stop or throttle resources — they only send notifications
- To programmatically respond to budget alerts (e.g., shut down VMs), use Cloud Functions triggered by Pub/Sub

### Programmatic Budget Notifications via Pub/Sub

- Connect a budget to a Pub/Sub topic
- Cloud Functions subscribe to the topic and can:
  - Disable billing on a project (`billingAccounts.projects.updateBillingInfo`)
  - Send Slack/PagerDuty notifications
  - Modify quota values

---

## Billing Export

### Cloud Billing Export to BigQuery

- Exports billing data to a BigQuery dataset automatically
- Types of export:
  - **Standard usage cost**: Detailed per-resource usage and cost data; latency of a few hours
  - **Detailed usage cost**: Includes resource-level labels; more granular; latency of a few hours
  - **Pricing data**: Price book — list prices, discount rates (separate export)
- Configuration: One-time setup in Billing > Billing export; choose project and BigQuery dataset
- Historical data: Export only captures data **from the time export is enabled** — no retroactive export of prior months
- Use BigQuery to build custom dashboards, analyze cost by team/label, or detect anomalies

### Cloud Billing Export to Cloud Storage (Legacy)

- Older CSV/JSON export to a GCS bucket
- Less feature-rich than BigQuery export
- **Google recommends BigQuery export over Cloud Storage export**

---

## Quotas

### Types of Quotas

| Type | Description | Example |
|------|-------------|---------|
| **Rate quota** | Max operations per unit time | 1000 API requests/100 seconds |
| **Allocation quota** | Max number of resources that can exist | 8 vCPUs per region per project |

### Key Quota Facts

- Quotas are **per-project, per-region** (or per-zone for some resources)
- Default quotas are relatively conservative for new projects
- Quota increases are requested via the **Quotas** page in Console or via the Service Usage API
- Requests are typically auto-approved for modest increases; large increases require manual review
- **Preemptible VM quotas are separate** from regular VM quotas
- Quotas cannot be decreased below current usage

### Common Default Quotas to Know

- Default CPU quota: 8 vCPUs in most regions for new projects
- Maximum number of projects (per billing account): 30 (soft limit)
- Maximum firewall rules per VPC: 300 (default), can be increased
- Cloud Storage API: 5,000 requests/100 seconds (default)

---

## Related Services / Concepts

- **Resource Hierarchy**: Billing accounts link to projects — see [projects-and-org.md](projects-and-org.md)
- **Pricing Optimization**: Sustained use discounts and committed use interact with billing — see [pricing-optimization.md](../domain-2-plan-and-configure/pricing-optimization.md)
- **Cloud Monitoring**: Can create cost anomaly alerts — see [monitoring-cloud-ops.md](../domain-4-ensure-success/monitoring-cloud-ops.md)
- **IAM**: Billing roles are distinct from project roles — see [iam-overview.md](iam-overview.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Budgets don't stop resources**: A critical distinction. Budget alerts are *notifications only*. Resources continue running even if the budget is exceeded. Only a Pub/Sub + Cloud Function integration or manual action disables billing.

2. **Disabling billing kills resources**: If you disable billing on a project, all resources are stopped (VMs shut down, etc.) but data is retained for approximately 30 days. This is irreversible in the short term.

3. **Billing account location vs project location**: A US-based billing account can pay for resources in any region. Billing account "location" affects currency and payment terms, not where resources run.

4. **BigQuery export is not retroactive**: You cannot export billing data from before you enabled the export. Plan early.

5. **Billing roles are on the billing account, not the project**: `billing.user` is granted ON the billing account resource, not on a project. Confusing because project-level IAM doesn't cover billing account linking.

6. **Multiple billing accounts per org**: An organization can have multiple billing accounts for department chargebacks. Each project links to one.

7. **Sub-accounts are reseller-only**: Don't confuse sub-accounts with child projects.

### Decision Tree: Responding to Cost Overruns

```
Alert notification received → Budget threshold exceeded?
  ├── Just want notification?
  │     → Email alerts are sufficient
  ├── Want programmatic response?
  │     → Configure Pub/Sub on budget
  │     → Cloud Function triggered to disable billing or notify
  └── Want to hard-stop resources?
        → Disable billing on project (caution: stops all resources)
```

### Keywords
- Billing account, budget alert, Pub/Sub notification, BigQuery export, quota increase, self-serve billing, invoiced billing, billing admin, billing user, standard export, detailed export, allocation quota, rate quota

---

## Source

- https://cloud.google.com/billing/docs/concepts
- https://cloud.google.com/billing/docs/how-to/budgets
- https://cloud.google.com/billing/docs/how-to/export-data-bigquery
- https://cloud.google.com/docs/quota
