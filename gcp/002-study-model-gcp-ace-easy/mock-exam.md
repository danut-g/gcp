# Mock exam — 50 questions

Same 50 questions as the original mock exam, with the language tightened up where it was technical for technical's sake. **The questions and answer key are identical** — Google's exam phrasing is itself dense, so a too-friendly mock would be misleading.

---

## How to take it

A timed practice run. Aim for **2 hours end-to-end** (the same as the real exam) and **≥80% correct** before you book the real thing.

1. **Go straight through.** If you're unsure, mark it and keep moving — don't peek at the answer key.
2. **Cover the answer key** (everything below the horizontal rule after Q50) until you're done.
3. **Review every answer afterward** — especially the ones you got right by guessing. The lessons next to each answer tell you exactly where to re-read.

The questions roughly match the real exam's domain weights: ~12 from Section 1, ~15 from Section 2, ~13 from Section 3, ~10 from Section 4.

---

## Questions

### Q1
Your company has hundreds of GCP projects organized under folders for each business unit. The security team must have **Viewer** on every project, applied with the minimum number of bindings. Where do you add the binding?

- A. On each project individually.
- B. On each business-unit folder.
- C. On the organization node.
- D. On the resource policies of every Compute Engine VM.

### Q2
A developer with `roles/owner` cannot create a Compute Engine VM with an external IP address. Which is the most likely cause?

- A. Compute Engine API is disabled in the project.
- B. The project has reached its CPU quota.
- C. An organization policy `compute.vmExternalIpAccess` is restricting external IPs.
- D. The user lacks `roles/compute.networkAdmin`.

### Q3
You delete a project that contained a critical Cloud Storage bucket. Two days later you realize you still need the data. What can you do?

- A. Nothing — deleted projects are gone immediately.
- B. Run `gcloud projects undelete PROJECT_ID` within ~30 days.
- C. Open a Google Cloud support case to recover the bucket directly.
- D. Restore from billing history.

### Q4
The finance team wants to prevent any project from spending more than $5,000/month, automatically. The supported pattern is:

- A. A budget with a 100% threshold email alert.
- B. A budget that triggers a Pub/Sub notification consumed by a Cloud Function that disables billing on the project.
- C. A hard quota at $5,000.
- D. The Pricing Calculator with a $5,000 limit.

### Q5
You need to attach a project to an existing billing account. You have `roles/owner` on the project. What additional role do you need and where?

- A. `roles/billing.user` on the billing account.
- B. `roles/billing.projectManager` on the project.
- C. `roles/billing.admin` on the organization.
- D. None — Owner is sufficient.

### Q6
A VM running Compute Engine reports `SERVICE_DISABLED` when calling the Pub/Sub API. The fix is:

- A. Add the Pub/Sub API at the organization level.
- B. Run `gcloud services enable pubsub.googleapis.com` in the VM's project.
- C. Restart the VM.
- D. Grant the VM's service account `roles/owner`.

### Q7
You need to know every public Cloud Storage bucket in your entire organization. Which is the fastest single approach?

- A. Loop over every project listing buckets via gcloud.
- B. Cloud Asset Inventory IAM policy search for `allUsers` bindings on storage resources.
- C. Enable Data Access audit logs and review them.
- D. Open every project in the console.

### Q8
Cloud Identity vs. IAM: a question describes "we use admin.google.com to manage user accounts and groups." This is referring to:

- A. IAM.
- B. Cloud Identity (or Workspace) — the directory.
- C. Cloud Asset Inventory.
- D. Org policies.

### Q9
A multi-region BigQuery dataset is needed for a globally distributed analytics workload. After creation, can the dataset be moved to a single region?

- A. Yes, via `gcloud bq datasets update --location`.
- B. Yes, by editing the dataset.
- C. No — the dataset's location is permanent. You can copy it to a new dataset in the desired location.
- D. Only Spanner supports moving locations.

### Q10
A team needs to be alerted at 50%, 90%, and 100% of a $10,000 monthly budget, with delivery to email and Slack. What do you configure?

- A. Three separate budgets at 50%, 90%, 100%.
- B. One budget with three threshold rules; email recipients and Pub/Sub notifications consumed by a Slack function.
- C. Slack-native budget integration in the console.
- D. Quota-based alerts.

### Q11
You're choosing a compute service for a stateless, containerized REST API with bursty traffic and minimum operations. The recommended choice is:

- A. Compute Engine MIG with autoscaling.
- B. GKE Standard.
- C. Cloud Run.
- D. App Engine flexible.

### Q12
A batch job of 8-hour duration tolerates restarts. You want it cheaper. Which combination?

- A. n2-standard-4 with `--maintenance-policy=MIGRATE`.
- B. n2-standard-4 Spot VM with `--maintenance-policy=TERMINATE`.
- C. m3-megamem-64 sole-tenant.
- D. e2-medium standard.

### Q13
A regional MIG must run identical VMs created from a known template. You change a setting on the template via the console — but no VMs reflect the change. Why?

- A. The MIG must be deleted and recreated.
- B. Instance templates are immutable; you must create a new template and apply a rolling update on the MIG.
- C. The console caches changes; wait 24 hours.
- D. Templates only apply to new MIGs.

### Q14
You need a Persistent Disk that survives a zone failure with no data loss for a self-managed Postgres. Which type?

- A. Local SSD.
- B. Zonal Persistent Disk balanced.
- C. Regional Persistent Disk SSD.
- D. Hyperdisk Throughput.

### Q15
A GKE cluster must be unreachable from the public internet (control plane and nodes) and have multi-zone control-plane HA. Pick:

- A. Zonal cluster with private nodes.
- B. Regional Standard cluster with private nodes and a private endpoint.
- C. Regional Standard cluster with authorized networks.
- D. Autopilot cluster with public endpoint enabled.

### Q16
Which database fits a globally distributed, strongly-consistent, horizontally-scalable transactional workload?

- A. Cloud SQL with cross-region replicas.
- B. Spanner.
- C. AlloyDB.
- D. BigQuery.

### Q17
Choose the storage class for compliance archives accessed less than once a year:

- A. Standard.
- B. Nearline.
- C. Coldline.
- D. Archive.

### Q18
A data team wants to migrate 200 TB from on-prem to Cloud Storage in one week, with limited bandwidth. Pick:

- A. `gcloud storage cp` with parallelism.
- B. Storage Transfer Service.
- C. Transfer Appliance.
- D. Database Migration Service.

### Q19
A team needs an event-driven function that runs whenever a file lands in a Cloud Storage bucket. Pick:

- A. Cloud Run service with an HTTP trigger and a polling job.
- B. Cloud Run functions (2nd gen) with a Cloud Storage trigger.
- C. Compute Engine VM with a watcher script.
- D. Dataflow.

### Q20
A custom-mode VPC needs subnets in three regions: `europe-west1`, `us-central1`, and `asia-east1`. You can use:

- A. One VPC, three subnets (one per region).
- B. Three VPCs (one per region) peered together.
- C. Three projects, three default VPCs.
- D. Auto-mode VPC.

### Q21
A scenario requires VMs without external IPs to install OS updates from internet repositories. The simplest correct configuration is:

- A. Add external IPs.
- B. Configure Cloud NAT in the region.
- C. Enable Private Google Access.
- D. Build a NAT VM and set up routes manually.

### Q22
A scenario describes "two GCP projects in the same org need internal connectivity, no overlapping CIDRs". Pick:

- A. Cloud Interconnect.
- B. VPC Network Peering.
- C. Cloud VPN with a public tunnel.
- D. Shared VPC.

### Q23
A public e-commerce site needs HTTPS, global anycast frontend, multi-region backends, and Cloud CDN. Pick load balancer + tier:

- A. Regional external Application LB + Standard tier.
- B. Global external Application LB + Premium tier.
- C. External passthrough Network LB + Standard tier.
- D. Internal Application LB + Premium tier.

### Q24
A firewall rule is targeted by network tags, but anyone with edit on the VM can self-tag. The recommended remediation:

- A. Increase the rule's priority.
- B. Use secure tags or service accounts as targets instead.
- C. Move the VMs to another VPC.
- D. Convert ingress to egress.

### Q25
A company uses a central network team in a host project, with multiple service projects' VMs running in the host's VPC. This is:

- A. VPC Network Peering.
- B. Shared VPC.
- C. Cloud Interconnect.
- D. Workload Identity Federation.

### Q26
A team manages GCP infrastructure with declarative Terraform. Two engineers ran `apply` simultaneously and corrupted state. The recommended fix:

- A. Use a remote backend (GCS) with state locking and bucket versioning.
- B. Coordinate via Slack.
- C. Switch to Helm.
- D. Use the local state file with file-system locks.

### Q27
A team wants to bootstrap a brand-new GCP organization with org policies, hierarchy, IAM, and networking using a Google-published opinionated Terraform reference. Pick:

- A. Cloud Foundation Toolkit (CFT) modules only.
- B. Fabric FAST.
- C. Helm charts.
- D. Custom Terraform written from scratch.

### Q28
A team wants Cloud SQL instances managed declaratively, alongside their Kubernetes manifests, in the same GitOps pipeline. Pick:

- A. Helm.
- B. Config Connector.
- C. Plain Terraform locally.
- D. Cloud Deployment Manager.

### Q29
A VM has only an internal IP. You need to SSH from your laptop. The right approach:

- A. Add an external IP temporarily.
- B. `gcloud compute ssh --tunnel-through-iap`.
- C. Set up a bastion host.
- D. Use the metadata server.

### Q30
You need automated daily snapshots with 14-day retention on a Persistent Disk. Pick:

- A. A cron job in the VM that runs `gcloud compute disks snapshot`.
- B. A snapshot schedule (resource policy) attached to the disk.
- C. Custom images created weekly.
- D. Manual snapshots reminded by calendar.

### Q31
A new Cloud Run revision (`rev-12`) is broken; the previous good revision is `rev-11`. The fastest, safest rollback:

- A. Redeploy the old image as `rev-13`.
- B. `gcloud run services update-traffic --to-revisions=rev-11=100`.
- C. Delete `rev-12`.
- D. Recreate the service from scratch.

### Q32
A Cloud SQL Postgres database must be restorable to any point in the last 7 days, including transaction-level recovery. Required configuration:

- A. Daily automated backups only.
- B. Daily automated backups + point-in-time recovery (binary logs/WAL).
- C. Weekly snapshots of the underlying disk.
- D. Cross-region read replica.

### Q33
Cloud Storage objects must transition to Coldline at 90 days and be deleted at 365 days. Pick:

- A. A Cloud Function triggered by object creation that schedules deletes.
- B. Object Lifecycle Management with two rules.
- C. A scheduled BigQuery query to enumerate objects.
- D. Manually each month.

### Q34
A streaming Dataflow pipeline must pause for an upgrade with **no data loss**. Use:

- A. `gcloud dataflow jobs cancel`.
- B. `gcloud dataflow jobs drain`.
- C. Stop the source Pub/Sub subscription.
- D. Delete the BigQuery sink.

### Q35
A subnet `10.0.0.0/24` is full. You need more IPs without disrupting existing VMs. Pick:

- A. Delete and recreate the subnet with a /22.
- B. Run `gcloud compute networks subnets expand-ip-range`.
- C. Add a secondary range.
- D. Migrate to a new subnet.

### Q36
A new public-facing website needs a stable IPv4 address that you'll point DNS at. Reserve:

- A. An ephemeral external IP.
- B. A global external static IPv4 address.
- C. An internal static IP.
- D. A regional Premium IP.

### Q37
Internal services in a VPC must resolve names like `db.internal.example.com`. Pick:

- A. A Cloud DNS public zone.
- B. A Cloud DNS private zone authorized to the VPC.
- C. `/etc/hosts` files on every VM.
- D. Cloud NAT.

### Q38
You need to alert when a Cloud Run service logs more than 5 ERROR-level entries in any 1-minute window. The minimal configuration:

- A. Export logs to BigQuery + scheduled query.
- B. Logs-based counter metric + alerting policy on it.
- C. Pub/Sub log routing + custom Cloud Function.
- D. Cloud Trace alert.

### Q39
Security wants every Audit Log streamed to an on-prem SIEM. Choose the Log Router destination:

- A. BigQuery dataset.
- B. Cloud Storage bucket.
- C. Pub/Sub topic.
- D. Another log bucket.

### Q40
A finance dataset's table contents may have been viewed by an unauthorized user. The audit configuration that captures this is:

- A. Admin Activity audit logs (always on).
- B. Data Access audit logs for BigQuery (must be enabled, not on by default).
- C. Cloud Trace.
- D. Cloud Profiler.

### Q41
The SRE team wants a single dashboard showing CPU across 25 application projects. Pick:

- A. A dashboard in each project, switched manually.
- B. A metrics scope: pick one scoping project; add the 25 projects as monitored projects.
- C. Export metrics to BigQuery and use Looker.
- D. Aggregate via Pub/Sub.

### Q42
A unified VM agent that collects both system metrics and logs is:

- A. Cloud Monitoring agent.
- B. Cloud Logging agent.
- C. Ops Agent.
- D. Stackdriver agent.

### Q43
An operator must start, stop, view (but not create or delete) Compute Engine VMs. Pick:

- A. `roles/compute.admin`.
- B. `roles/editor`.
- C. `roles/compute.instanceAdmin.v1`.
- D. A custom role with `compute.instances.start/stop/get/list`.

### Q44
A contractor needs Storage Admin for 6 months only. The cleanest approach:

- A. Grant the role and remove it later.
- B. Add the contractor to a special group.
- C. Grant the role with an IAM Condition `request.time < timestamp("...")`.
- D. Set a calendar reminder.

### Q45
A user emails: "I lost access to dataset X." You need to know exactly why. The right tool:

- A. Cloud Audit Logs.
- B. IAM Policy Troubleshooter.
- C. Manually inspect IAM at every level.
- D. Cloud Asset Inventory.

### Q46
A production VM uses the default Compute Engine service account. Security flags it. The correct remediation:

- A. Restrict the default SA's permissions.
- B. Create a user-managed SA with least privilege; switch the (stopped) VM to it.
- C. Disable the default SA project-wide.
- D. Use Workload Identity Federation.

### Q47
Alice deploys Cloud Run services that should run as `deploy-sa@...`. She needs which role on the SA?

- A. `roles/iam.serviceAccountAdmin`.
- B. `roles/iam.serviceAccountUser`.
- C. `roles/iam.serviceAccountTokenCreator`.
- D. `roles/owner` on the project.

### Q48
A pod must read Cloud Storage without a JSON key. The recommended pattern:

- A. Mount a key as a Kubernetes Secret.
- B. Workload Identity binding K8s SA → Google SA.
- C. Use the node service account with `roles/owner`.
- D. SA keys are unavoidable on GKE.

### Q49
An on-call engineer occasionally needs prod-deploy permissions her user account doesn't normally have. Pick:

- A. Add her user as Owner.
- B. Grant her `roles/iam.serviceAccountTokenCreator` on a deploy SA; she runs commands with `--impersonate-service-account`.
- C. Email her a JSON key.
- D. Add `allAuthenticatedUsers` Editor.

### Q50
Compliance requires that no one can create new SA JSON keys anywhere in the organization. Pick:

- A. Remove `roles/iam.serviceAccountKeyAdmin` from every binding.
- B. Audit periodically.
- C. Set the org policy `iam.disableServiceAccountKeyCreation` at the organization node.
- D. Delete all existing keys.

---

## Answer key

Each row gives the correct option, a one-line reason, and the lesson where the topic is covered for review.

| # | Answer | Why / lesson |
|---|---|---|
| 1 | **B** | Inheritance — apply at the lowest level that universally satisfies the goal. (1.1) |
| 2 | **C** | Org policies block actions even for Owners. (1.1) |
| 3 | **B** | Project deletion has a ~30-day recovery window. (1.1) |
| 4 | **B** | Budgets only alert; the hard-cap pattern is Pub/Sub → Cloud Function disables billing. (1.2) |
| 5 | **A** | Linking requires Billing Account User on the billing account *plus* Project Billing Manager on the project. The closest single-option answer is the missing billing-side role. (1.2) |
| 6 | **B** | APIs are per-project; enable the API. (1.1) |
| 7 | **B** | Asset Inventory IAM policy search across the org. (1.1) |
| 8 | **B** | admin.google.com is the Cloud Identity / Workspace admin console. (1.1) |
| 9 | **C** | BigQuery dataset locations are immutable; copy to relocate. (2.2) |
| 10 | **B** | One budget with multiple thresholds; Pub/Sub for non-email channels. (1.2) |
| 11 | **C** | Cloud Run for stateless containers, scale-to-zero. (2.1) |
| 12 | **B** | Spot VMs with `TERMINATE` for fault-tolerant batch. (2.1) |
| 13 | **B** | Templates are immutable; new template + rolling update. (2.1) |
| 14 | **C** | Regional PD synchronously replicates across two zones. (2.1) |
| 15 | **B** | Regional Standard + private nodes + private endpoint. (2.1) |
| 16 | **B** | Spanner: globally distributed, strongly consistent, horizontally scalable. (2.2) |
| 17 | **D** | Archive class for compliance archives. (2.2) |
| 18 | **C** | Transfer Appliance for petabyte-scale offline transfer. (2.2) |
| 19 | **B** | Cloud Run functions 2nd gen with Cloud Storage trigger. (2.1) |
| 20 | **A** | One VPC is global; subnets are regional. (2.3) |
| 21 | **B** | Cloud NAT for general internet egress. Private Google Access only covers Google APIs. (3.3) |
| 22 | **B** | VPC Peering — same org, GCP↔GCP, no overlapping CIDRs. (2.3) |
| 23 | **B** | Global external Application LB requires Premium tier; supports Cloud CDN. (2.3) |
| 24 | **B** | Secure tags or service accounts as firewall targets. (2.3) |
| 25 | **B** | Shared VPC — host project + service projects. (2.3) |
| 26 | **A** | Remote state in GCS (locking + versioning). (2.4) |
| 27 | **B** | Fabric FAST — Google-published opinionated Terraform reference. (2.4) |
| 28 | **B** | Config Connector — GCP resources from Kubernetes manifests. (2.4) |
| 29 | **B** | IAP TCP forwarding for SSH to no-external-IP VMs. (3.1) |
| 30 | **B** | Snapshot schedule (resource policy). (3.1) |
| 31 | **B** | Traffic-shift rollback to the prior revision. (3.1) |
| 32 | **B** | PITR requires backups + WAL/binlogs enabled in advance. (3.2) |
| 33 | **B** | Object Lifecycle Management. (3.2) |
| 34 | **B** | Drain finishes in-flight work; cancel kills it. (3.2) |
| 35 | **B** | Subnets can be expanded (grown) without disruption. (3.3) |
| 36 | **B** | Global external static IPv4 for a global LB / public endpoint. (3.3) |
| 37 | **B** | Cloud DNS private zone authorized to the VPC. (3.3) |
| 38 | **B** | Logs-based metric + alerting policy. (3.4) |
| 39 | **C** | Pub/Sub for streaming logs externally. (3.4) |
| 40 | **B** | Data Access audit logs (off by default for BigQuery). (3.4) |
| 41 | **B** | Metrics scope. (3.4) |
| 42 | **C** | Ops Agent — unified metrics + logs. (3.4) |
| 43 | **D** | Custom role for surgical permissions when no predefined role fits. (4.1) |
| 44 | **C** | IAM Condition with `request.time` for time-bounded access. (4.1) |
| 45 | **B** | IAM Policy Troubleshooter. (4.1) |
| 46 | **B** | Replace the default Compute Engine SA with a user-managed SA. (4.2) |
| 47 | **B** | `serviceAccountUser` includes `actAs`. (4.2) |
| 48 | **B** | Workload Identity. (4.2) |
| 49 | **B** | Impersonation via `serviceAccountTokenCreator`. (4.2) |
| 50 | **C** | Org policy `iam.disableServiceAccountKeyCreation`. (4.2) |

---

## Scoring

| Score | What it means |
|---|---|
| **45–50 (90–100%)** | You're ready. Light review of *Exam gotchas* and book the exam. |
| **40–44 (80–88%)** | Solid. Re-read every wrong answer's lesson, then proceed. |
| **35–39 (70–78%)** | On the edge. Re-read the lessons in your weakest domain (where most wrong answers cluster) and retake this in 2–3 days. |
| **<35 (<70%)** | Not ready. Targeted re-reading of weak domains; redo the *Worked scenarios* in each weak lesson; come back to this in a week. |

A note on the real exam: Google's questions are a touch trickier than these — more distractors that look plausible. Aim for **≥85% here** to give yourself a safe margin.
