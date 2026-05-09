# Section 1 Flashcards -- Setting Up a Cloud Solution Environment

Covers: Resource Hierarchy, Org Policies, IAM, Cloud Identity, APIs, Operations Suite, Quotas, Billing

---
### Q1: What are the four levels of the Google Cloud resource hierarchy, from top to bottom?
**A:** Organization -> Folders (optional, up to 10 levels deep) -> Projects -> Resources. IAM policies and org policies inherit downward through this chain.

---
### Q2: What three identifiers does every Google Cloud project have?
**A:** (1) Project Name -- human-readable, not unique. (2) Project ID -- globally unique, immutable, user-chosen at creation. (3) Project Number -- globally unique, auto-assigned by Google.

---
### Q3: True or False: If a parent folder grants Editor and a child project grants Viewer, the effective role is Viewer.
**A:** False. The more permissive policy wins. The effective role is Editor because IAM policy inheritance always takes the union of permissions.

---
### Q4: What gcloud command creates a new project inside a folder?
**A:** `gcloud projects create PROJECT_ID --name="Project Name" --folder=FOLDER_ID`

---
### Q5: What is the difference between an Organization Policy and an IAM policy?
**A:** IAM controls *who* can do things (identity-based access). Organization Policies control *what* can be done (configuration constraints on resources), such as restricting regions or blocking external IPs.

---
### Q6: Which organization policy constraint restricts which VMs can have external IP addresses?
**A:** `constraints/compute.vmExternalIpAccess`. Setting it to DENY ALL at the organization level prevents any VM from having an external IP.

---
### Q7: What gcloud command lists all available organization policy constraints?
**A:** `gcloud org-policies list-available-constraints --organization=ORG_ID`

---
### Q8: What is the IAM model expressed as a sentence?
**A:** *Who* (member/principal) has *what access* (role) to *which resource*. A policy binding ties these three elements together at a specific hierarchy level.

---
### Q9: What are the three IAM role types and when should you use each?
**A:** (1) Basic/Primitive -- broad permissions (Owner, Editor, Viewer), use sparingly. (2) Predefined -- fine-grained, service-specific (e.g., `roles/storage.objectViewer`), recommended. (3) Custom -- user-defined subset of permissions for least-privilege needs.

---
### Q10: What is the difference between the Editor and Owner basic roles?
**A:** Editor grants read/write access to all resources but cannot manage IAM or billing. Owner includes everything Editor has plus the ability to manage IAM policies and billing settings.

---
### Q11: A company has 50 developers who need Compute Admin access. What is the IAM best practice?
**A:** Create a Google Group (e.g., `developers@company.com`), add all developers, then grant `roles/compute.admin` to the group. Never assign roles to individual users at scale.

---
### Q12: What gcloud command grants a user the Viewer role on a project?
**A:** `gcloud projects add-iam-policy-binding PROJECT_ID --member="user:alice@example.com" --role="roles/viewer"`

---
### Q13: What is Cloud Identity?
**A:** Google's Identity as a Service (IDaaS) solution. It provides user/group management, device management, SSO, and MFA. It is the foundation for managing identities that interact with Google Cloud.

---
### Q14: What tool synchronizes users and groups from on-premises Active Directory to Cloud Identity?
**A:** Google Cloud Directory Sync (GCDS). It syncs users, groups, and org units from AD or LDAP to Cloud Identity.

---
### Q15: True or False: Most Google Cloud APIs are enabled by default in a new project.
**A:** False. Most APIs are disabled by default. You must explicitly enable an API before using its service. This reduces attack surface and prevents unexpected costs.

---
### Q16: What gcloud command enables the Compute Engine API?
**A:** `gcloud services enable compute.googleapis.com`

---
### Q17: True or False: Enabling an API immediately incurs charges.
**A:** False. Enabling an API does not incur charges. You are only charged when you actually use the service.

---
### Q18: What are the five main components of the Google Cloud Operations Suite?
**A:** Cloud Monitoring (metrics, dashboards, alerts), Cloud Logging (centralized logs), Cloud Trace (distributed latency tracing), Cloud Profiler (CPU/memory profiling), and Error Reporting (error aggregation).

---
### Q19: What is the difference between a rate quota and an allocation quota?
**A:** A rate quota resets after a time interval (e.g., API calls per 100 seconds). An allocation quota limits the total number of resources you can have (e.g., max VMs per project).

---
### Q20: At what level do quotas apply by default?
**A:** Quotas apply at the project level. They are not tied to billing accounts or organizations by default.

---
### Q21: What HTTP error code indicates a quota has been exceeded?
**A:** HTTP 429 (Too Many Requests). You may also see specific quota-exceeded error messages in the API response.

---
### Q22: What are the two types of billing accounts in Google Cloud?
**A:** (1) Self-serve (online) -- linked to a credit/debit card, charges processed automatically. (2) Invoiced (offline) -- Google sends monthly invoices, requires application and approval.

---
### Q23: True or False: A project can be linked to multiple billing accounts at the same time.
**A:** False. A project can be linked to only one billing account at a time. However, a single billing account can be linked to multiple projects.

---
### Q24: What happens to running VMs when you unlink a project from its billing account?
**A:** VMs are stopped (not deleted). Cloud Storage buckets become read-only. Most services stop functioning. After a grace period, resources may be permanently deleted.

---
### Q25: What two IAM roles are needed to link a project to a billing account?
**A:** `roles/billing.user` on the billing account AND either Project Owner or Project Billing Manager on the project.

---
### Q26: True or False: A billing budget automatically stops spending when the threshold is reached.
**A:** False. Budgets only send alerts and notifications. To cap spending programmatically, you must set up a Pub/Sub topic that triggers a Cloud Function to disable billing or scale down resources.

---
### Q27: What are the three default alert thresholds for a billing budget?
**A:** 50%, 90%, and 100% of the budget amount. These can be customized, and alerts can be based on actual or forecasted spend.

---
### Q28: A team wants to automatically shut down resources when spending exceeds $5,000. How?
**A:** Create a billing budget with a Pub/Sub notification at the $5,000 threshold. Deploy a Cloud Function triggered by that Pub/Sub topic to disable billing on the project or scale down resources.

---
### Q29: Which billing export method provides the most granular cost data?
**A:** BigQuery export, specifically the "Detailed usage cost" export type. It provides resource-level line-item detail. Cloud Storage export is less detailed and considered legacy.

---
### Q30: True or False: BigQuery billing export backfills historical data from before you enabled the export.
**A:** False. Export starts from the enablement date only. No historical data is backfilled, so enable it as early as possible.

---
### Q31: What is the difference between Committed Use Discounts (CUDs) and Sustained Use Discounts (SUDs)?
**A:** CUDs require a 1-year or 3-year commitment for up to 57%/70% savings. SUDs are automatic discounts (up to 30%) applied when you run resources more than 25% of the month -- no commitment needed.

---
### Q32: What is the `roles/billing.costsManager` role used for?
**A:** It allows managing budgets and viewing/exporting costs without granting payment management rights. It is ideal for finance teams that need cost visibility but should not modify payment methods.

---
### Q33: What gcloud command lists all billing accounts you have access to?
**A:** `gcloud billing accounts list`

---
### Q34: Where do billing accounts sit in the resource hierarchy?
**A:** Billing accounts exist at the organization level, alongside (not inside) the folder/project hierarchy. A billing account can be linked to projects across different folders.

---
### Q35: What organization policy constraint restricts where resources can be created geographically?
**A:** `constraints/gcp.resourceLocations`. You can use it to enforce that resources are only created in specific regions, such as EU-only for data residency compliance.

---
### Q36: What is the purpose of labels in cost management?
**A:** Labels are key-value pairs (e.g., `environment:production`, `team:backend`) attached to resources. They allow filtering and grouping costs in billing reports and can be used as budget scope criteria.

---
### Q37: What gcloud command links a project to a billing account?
**A:** `gcloud billing projects link PROJECT_ID --billing-account=BILLING_ACCOUNT_ID`

---
### Q38: What is the recovery window after deleting a Google Cloud project?
**A:** 30 days. During this period the project can be restored. After 30 days, the project and all its resources are permanently deleted.

---
### Q39: What is the difference between `allAuthenticatedUsers` and `allUsers` in IAM?
**A:** `allAuthenticatedUsers` includes anyone with a Google account who is signed in. `allUsers` includes everyone on the internet, including unauthenticated users. Use `allUsers` with extreme caution.

---
### Q40: A developer gets "API not enabled" when trying to create a Cloud Function. What should they do?
**A:** Enable the Cloud Functions API: `gcloud services enable cloudfunctions.googleapis.com`. Most APIs are disabled by default and must be explicitly enabled before use.

---
### Q41: What is Cloud Asset Inventory?
**A:** A Google Cloud service that provides an org-wide inventory of all resources and IAM policies. It supports real-time and point-in-time queries, exports to BigQuery or GCS, and can trigger Pub/Sub feeds on resource changes.

---
### Q42: What gcloud command searches all Compute Engine instances in an organization that have names starting with "prod-"?
**A:** `gcloud asset search-all-resources --scope=organizations/ORG_ID --query="name:prod-*" --asset-types=compute.googleapis.com/Instance`

---
### Q43: What is Gemini Cloud Assist and what can it NOT do?
**A:** An AI assistant embedded in the Cloud Console that summarizes logs, explains alerts, suggests fixes, and helps write alert policies. It cannot take actions on your behalf — it assists but does not act.

---
### Q44: You need to find out who currently has read access to a specific Cloud Storage bucket across your organization. What's the most efficient approach?
**A:** Use Cloud Asset Inventory's IAM policy analysis: `gcloud asset analyze-iam-policy --organization=ORG_ID --full-resource-name=//storage.googleapis.com/projects/_/buckets/BUCKET --permissions=storage.objects.list`

---
### Q45: What is the difference between a Cloud Asset Inventory feed and a point-in-time export?
**A:** A **feed** delivers real-time change notifications via Pub/Sub when assets are created, updated, or deleted. A **point-in-time export** is a one-time snapshot of all asset states saved to GCS or BigQuery.
