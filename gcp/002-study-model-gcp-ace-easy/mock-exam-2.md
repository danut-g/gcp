# Mock exam 2 — 50 fresh questions

A second full practice run with **all-new scenarios** (no overlap with [mock exam 1](mock-exam.md)). Same rules: 2 hours, no peeking, ≥80% before booking — and because you've already seen mock 1, treat **≥85% here** as your real readiness signal.

Domain mix matches the real exam: ~12 from Section 1, ~15 from Section 2, ~13 from Section 3, ~10 from Section 4.

---

## Questions

### Q1
A new developer joins the team and must be able to create and delete VMs in the `dev-sandbox` project, but must not be able to change IAM bindings or billing. Which role fits best?

- A. `roles/owner` on the project.
- B. `roles/editor` on the project.
- C. `roles/compute.instanceAdmin.v1` on the project.
- D. `roles/compute.viewer` on the project.

### Q2
Your organization has folders `prod` and `dev`. A constraint must guarantee that **no resource anywhere in the company** can ever be created outside EU regions, with one exception: the `experiments` project may use US regions. How do you set this up?

- A. A region allow-list org policy on every project individually.
- B. An EU-only list constraint at the organization node; an override allowing US values on the `experiments` project.
- C. An EU-only list constraint at the organization node; org policies cannot be overridden, so the experiments team must request a new organization.
- D. IAM deny policies for US regions at the org node.

### Q3
You run `gcloud compute instances create test-vm` in a brand-new project and get: `API [compute.googleapis.com] not enabled on project`. You have Owner. What's the fastest fix?

- A. `gcloud services enable compute.googleapis.com`
- B. Grant yourself `roles/serviceusage.serviceUsageAdmin` at the org level first.
- C. Wait — APIs enable automatically a few minutes after project creation.
- D. Link the project to a billing account; APIs enable automatically once billing is active.

### Q4
Your team of 40 engineers needs Viewer on the `analytics-prod` project. New people join and leave monthly. What's the Google-recommended way to manage this?

- A. Grant `roles/viewer` to each engineer's user account.
- B. Create a Google Group, grant `roles/viewer` to the group, manage membership in the group.
- C. Create a service account for the team and share its key.
- D. Grant `roles/viewer` to the whole domain.

### Q5
Finance asks: "We need every project's spend broken down by team, in our own SQL dashboards, updated automatically." What do you set up?

- A. Budget alerts to the finance mailing list.
- B. Billing data export to BigQuery, plus labels on resources/projects per team.
- C. Monthly PDF invoices.
- D. A Cloud Function scraping the Billing console.

### Q6
A project hit its quota of 24 vCPUs in `europe-west1` and a deployment failed. The workload genuinely needs 64 vCPUs. What do you do?

- A. Open the Quotas page (IAM & Admin → Quotas), select the vCPU quota for the region, and request an increase.
- B. Quotas are hard limits; split the workload across three projects.
- C. Switch the machine type to shared-core to bypass the quota.
- D. Buy a committed use discount, which raises the quota automatically.

### Q7
You must give an external auditor read access to **IAM policies across the whole organization** for a compliance review, with the least privilege. What's the best approach?

- A. `roles/owner` at the org node for a week.
- B. `roles/iam.securityReviewer` at the organization node.
- C. `roles/viewer` on every project, granted by a script.
- D. Export all IAM policies manually and email them.

### Q8
A startup with no Google Workspace wants to use GCP with an organization node and managed user identities, **without** paying for Workspace. What do you recommend?

- A. Personal Gmail accounts for everyone.
- B. Cloud Identity Free — managed identities and an organization, no Workspace licence.
- C. They must buy Workspace; there is no other way to get an organization.
- D. Service accounts for each employee.

### Q9
`gcloud config configurations list` shows configurations `default` and `client-b`. You need to list VMs in client B's project **once**, without switching your active configuration. Which command?

- A. `gcloud config configurations activate client-b && gcloud compute instances list`
- B. `gcloud compute instances list --configuration=client-b`
- C. `gcloud compute instances list --project=client-b-prod`
- D. Both B and C work.

### Q10
A budget is set at $1,000 with alerts at 50%, 90%, and 100%. The project hits $1,000. What happens to the running resources?

- A. They are stopped automatically.
- B. Billing is disabled and the project enters read-only mode.
- C. Nothing — budgets only send notifications; spending continues.
- D. New resource creation is blocked, existing resources keep running.

### Q11
Your company wants weekend-long batch simulations at the lowest possible compute cost. The jobs checkpoint to Cloud Storage and can be re-run from a checkpoint if interrupted. Which option?

- A. Standard VMs with sustained use discounts.
- B. Spot VMs.
- C. A 3-year committed use discount.
- D. Cloud Run with max instances.

### Q12
In Cloud Shell, a teammate stored a 2 GB dataset in `/tmp` during a session and is surprised it disappeared the next day. Why?

- A. Cloud Shell deletes files larger than 1 GB.
- B. Only `$HOME` (5 GB) persists between sessions; the rest of the VM is ephemeral.
- C. The weekly usage quota was exceeded, wiping the disk.
- D. Cloud Shell has no writable storage at all.

### Q13
A stateless Node.js API in a container gets spiky traffic: zero at night, thousands of requests at lunch. Minimal ops effort, pay only for use. Which platform?

- A. A managed instance group with autoscaling.
- B. GKE Standard with HPA.
- C. Cloud Run.
- D. A single e2-standard-4 VM sized for peak.

### Q14
A regulated workload requires that the VM's disk encryption keys be **managed by your company in Cloud KMS**, not by Google defaults. What is this called and where is it set?

- A. CSEK — keys typed in at every disk mount.
- B. CMEK — select your KMS key when creating the disk.
- C. Confidential Computing — enable it on the VM.
- D. Shielded VM — enable Secure Boot.

### Q15
You need a MySQL database for a regional e-commerce app, with automatic failover if a zone goes down. Lowest ops burden. What do you choose?

- A. Cloud SQL for MySQL with high availability (regional) configuration.
- B. MySQL on two VMs with your own replication scripts.
- C. Spanner, because it's always highly available.
- D. Memorystore.

### Q16
An IoT platform ingests 2 million sensor readings per second and serves low-latency reads by device ID and time range. Analytical SQL is **not** needed on the hot path. Which database?

- A. Cloud SQL with read replicas.
- B. BigQuery.
- C. Bigtable.
- D. Firestore.

### Q17
Log archives must be kept for 7 years for compliance. They are read perhaps once a year during audits. Cost must be minimal, and **no object may be deleted or overwritten** during the period — even by admins. Which two features? (Choose 2)

- A. Archive storage class.
- B. Nearline storage class.
- C. Bucket retention policy with **lock**.
- D. Object versioning.

### Q18
Two VPC networks in **different organizations** (your company and a partner) must communicate privately, without traversing the public internet, with non-overlapping CIDRs. Which option works?

- A. VPC Network Peering — it works across organizations.
- B. Shared VPC — host in one org, service project in the other.
- C. Cloud VPN between the two VPCs only — peering is impossible across orgs.
- D. Both networks must be merged into one organization first.

### Q19
A global HTTP(S) web app needs one anycast IP, TLS termination, URL-based routing (`/api` → backend A, `/static` → a bucket), and CDN. Which load balancer?

- A. Regional external passthrough Network Load Balancer.
- B. Global external Application Load Balancer.
- C. Internal Application Load Balancer.
- D. TCP Proxy Load Balancer.

### Q20
VMs in a private subnet (no external IPs) must call `*.googleapis.com` (Cloud Storage, BigQuery) **without** using Cloud NAT. What do you enable?

- A. Private Google Access on the subnet.
- B. An external IP on each VM, protected by firewall rules.
- C. VPC peering to a Google-owned VPC.
- D. IAP TCP forwarding.

### Q21
Your Terraform state for production lives on one engineer's laptop. What is the Google-recommended remediation?

- A. Commit the state file to the Git repository.
- B. Store the state in a Cloud Storage bucket backend with versioning enabled.
- C. Email the state file to the team weekly.
- D. Recreate state with `terraform import` whenever needed.

### Q22
The exam scenario says: *"deploy a production-grade landing zone (folders, networking, IAM) following Google's opinionated Terraform blueprint."* Which is being described?

- A. Config Connector.
- B. Cloud Deployment Manager.
- C. Fabric FAST.
- D. Helm.

### Q23
An on-prem data center connects to GCP for a latency-sensitive workload needing **10 Gbps consistent** private connectivity with an SLA. Which option?

- A. Cloud VPN (HA VPN).
- B. Dedicated Interconnect.
- C. Direct Peering.
- D. Cloud NAT.

### Q24
A development team needs DNS for `internal.corp.example` resolvable **only inside their VPC**. What do you create?

- A. A Cloud DNS public zone.
- B. A Cloud DNS private zone attached to the VPC.
- C. Entries in each VM's `/etc/hosts` via startup script.
- D. A Cloud Domains registration.

### Q25
Which statement about subnets is correct?

- A. Subnets span all regions because VPCs are global.
- B. A subnet belongs to one region but can be used by VMs in any zone of that region; its secondary ranges can serve GKE pods/services.
- C. Subnets are zonal; one per zone is required.
- D. Subnet IP ranges can overlap within the same VPC if they're in different regions.

### Q26
Your company's images must be scanned and stored in a registry that supports **regional repositories, IAM per repository, and vulnerability scanning**. Which product?

- A. Container Registry (gcr.io).
- B. Artifact Registry.
- C. Cloud Storage with a naming convention.
- D. Docker Hub with a paid plan.

### Q27
A Compute Engine VM must survive a **zone outage** with its data intact and come back quickly in another zone of the same region. Disk requirement?

- A. Zonal Persistent Disk + nightly snapshots.
- B. Regional Persistent Disk (synchronous two-zone replication) + force-attach to a standby VM.
- C. Local SSD, because it's fastest.
- D. Filestore mounted over NFS.

### Q28
The marketing team needs WordPress running by this afternoon. Nobody on the team has ever configured PHP or MySQL. What's the fastest supported path?

- A. Write Terraform for a LAMP stack.
- B. Deploy the WordPress solution from Cloud Marketplace.
- C. A GKE cluster with a community Helm chart.
- D. Cloud Run with the official WordPress image and no database.

### Q29
You SSH daily into a private VM (no external IP) using `gcloud compute ssh my-vm --tunnel-through-iap`. Today it fails with a permissions error. Which role is most likely missing?

- A. `roles/compute.osLogin` on the project.
- B. `roles/iap.tunnelResourceAccessor` on the VM/project.
- C. `roles/compute.networkUser` on the subnet.
- D. `roles/iam.serviceAccountUser` on the VM's service account.

### Q30
A managed instance group behind a load balancer must replace VMs **gradually** when you ship a new image, keeping at least 90% capacity during the change. What do you use?

- A. Delete the MIG and recreate it with the new template.
- B. A rolling update on the MIG with a new instance template, configuring maxUnavailable ≈ 10%.
- C. Stop all VMs, change the template, start them again.
- D. Canary deployments are not possible with MIGs.

### Q31
A GKE Deployment's pods are evicted with `OOMKilled` under load. The nodes have plenty of free memory. What's the most likely cause and fix?

- A. Cluster autoscaler is disabled; enable it.
- B. The pods' memory **limits** are set too low; raise requests/limits in the pod spec.
- C. The node OS needs upgrading.
- D. HPA is missing; add it.

### Q32
A Cloud SQL instance must be cloned into a **point in time** five minutes before a bad `UPDATE` ran this morning. What must have been enabled **beforehand** for this to work?

- A. Read replicas.
- B. Automated backups + point-in-time recovery (binary/WAL logging).
- C. High availability configuration.
- D. Deletion protection.

### Q33
Objects in a bucket should move to Nearline after 30 days, to Coldline after 90, and be deleted after 365. How do you implement this?

- A. A cron job on a VM running `gcloud storage objects update` nightly.
- B. Object Lifecycle Management rules on the bucket.
- C. Bucket retention policy.
- D. Storage Transfer Service with a schedule.

### Q34
A Dataflow streaming job must be stopped for an upgrade. In-flight messages must be **fully processed first** — no data loss. Which stop mode?

- A. Cancel.
- B. Drain.
- C. Force-delete the job.
- D. Pause the Pub/Sub subscription instead.

### Q35
You need a daily 02:00 batch job (a container that runs ~20 minutes and exits). Serverless, no idle cost. Which combination?

- A. A Compute Engine VM that runs 24/7 with a cron entry.
- B. Cloud Run **jobs** triggered by Cloud Scheduler.
- C. A GKE cluster with a CronJob, autoscaled to zero.
- D. Cloud Run service with min-instances = 1.

### Q36
The on-call engineer must be paged when p95 latency of a service exceeds 800 ms for 5 minutes. What do you configure in Cloud Monitoring?

- A. A log-based metric only.
- B. An alerting policy on the latency metric with a 5-minute duration condition and a notification channel (e.g., PagerDuty/SMS).
- C. An uptime check.
- D. A dashboard widget with a red threshold line.

### Q37
Security requires that **all Admin Activity audit logs across the organization** be kept for 5 years in a central, low-cost location. Best-practice answer?

- A. Do nothing — Admin Activity logs are kept forever by default.
- B. An aggregated org-level log sink exporting to a Cloud Storage bucket (Archive class) in a dedicated project.
- C. Increase the _Default bucket retention to 1,825 days in every project.
- D. Export to BigQuery in each project separately.

### Q38
A VM's application writes detailed JSON logs to a local file. The ops team wants them searchable in Cloud Logging. What's the supported way?

- A. Logs from VM files can't reach Cloud Logging.
- B. Install/configure the **Ops Agent** on the VM to tail the file and ship entries.
- C. A cron job that runs `gcloud logging write` per line.
- D. Mount the disk on a logging appliance.

### Q39
Data Loss Prevention: a bucket object was deleted by *someone* yesterday. Where do you find **who** did it?

- A. Data Access audit logs are off by default — the information may not exist unless they were enabled; otherwise check Admin Activity logs.
- B. In `gsutil ls -L` output.
- C. In the bucket's lifecycle history.
- D. In billing export.

### Q40
You manage 60 GKE microservices. The team wants request traces across service boundaries to debug latency. Which observability product?

- A. Cloud Profiler.
- B. Cloud Trace.
- C. Error Reporting.
- D. Cloud Debugger.

### Q41
A Cloud Run service suddenly serves 500s after a deploy at 14:02. The fastest safe mitigation?

- A. Delete the service and redeploy from scratch.
- B. `gcloud run services update-traffic SERVICE --to-revisions=PREVIOUS_REVISION=100`
- C. Scale min-instances to 10.
- D. Restart the underlying VMs.

### Q42
`kubectl get nodes` fails with `Unable to connect to the server`. You're sure the GKE cluster `prod-eu` in `europe-west1` exists and you have permissions. What's the usual first fix?

- A. `gcloud container clusters get-credentials prod-eu --region=europe-west1`
- B. Recreate the cluster.
- C. `gcloud components update kubectl`
- D. Open port 22 on the nodes.

### Q43
An application's service account needs to read **one specific secret** in Secret Manager — nothing else. Which grant is correct?

- A. `roles/secretmanager.admin` on the project.
- B. `roles/secretmanager.secretAccessor` on that secret.
- C. `roles/editor` on the project.
- D. `roles/secretmanager.viewer` on the project.

### Q44
Your CI pipeline on GitHub Actions must deploy to GCP. Security policy: **no long-lived JSON keys anywhere**. What's the Google-recommended setup?

- A. A JSON key stored as a GitHub encrypted secret.
- B. Workload Identity Federation: GitHub's OIDC tokens are exchanged for short-lived GCP credentials.
- C. A shared developer account with 2FA.
- D. Run the pipeline only from Cloud Shell.

### Q45
A user has `roles/iam.serviceAccountUser` on service account `deployer@…`. What does this actually let them do?

- A. Read the SA's keys.
- B. Act as / attach that SA to resources (e.g., deploy a VM that runs as it) — effectively use its permissions through resources.
- C. Create new service accounts.
- D. Change IAM policy on the project.

### Q46
The default Compute Engine service account is flagged in a security review. Why, and what's the fix?

- A. It's flagged because it can't be deleted; the fix is to ignore the finding.
- B. It historically gets the project-wide **Editor** role; the fix is to run workloads with dedicated least-privilege service accounts (and/or strip Editor).
- C. It has no permissions; the fix is to grant it Editor.
- D. It expires after 90 days and breaks workloads.

### Q47
An organization must guarantee that **service account keys can never be created** in any current or future project. How?

- A. A weekly script that deletes found keys.
- B. The org policy constraint `iam.disableServiceAccountKeyCreation` enforced at the organization node.
- C. Email a policy document to all engineers.
- D. Remove `roles/iam.serviceAccountKeyAdmin` from everyone, everywhere, forever.

### Q48
A data scientist needs to query a BigQuery dataset and **nothing else** in the project. Least-privilege grant?

- A. `roles/bigquery.admin` on the project.
- B. `roles/bigquery.dataViewer` on the dataset + `roles/bigquery.jobUser` on the project.
- C. `roles/viewer` on the project.
- D. `roles/bigquery.dataOwner` on the dataset.

### Q49
Which statement about **basic roles** (Owner/Editor/Viewer) reflects Google's guidance?

- A. They're recommended for production because they're simple.
- B. They're overly broad; production should use predefined (or custom) roles scoped to what's needed.
- C. Editor includes the ability to manage IAM policy.
- D. Viewer can read data in all services including secret payloads, so it's unsafe to ever grant.

### Q50
You suspect someone changed a firewall rule last night in `prod-net`. Where do you look first?

- A. VPC Flow Logs.
- B. Admin Activity audit logs (filter on the firewall resource and last night's window).
- C. Cloud Trace.
- D. The serial console of affected VMs.

---

## Answer key

| # | Answer | Why / lesson |
|---|---|---|
| 1 | **C** | Least privilege: instanceAdmin manages VMs without IAM/billing powers; Editor/Owner are too broad. (4.1) |
| 2 | **B** | List constraints set at the org; a child resource can be allowed to override/merge values where the policy permits. (1.1) |
| 3 | **A** | APIs are off in new projects; Owner can enable them directly. (1.1, 1.3) |
| 4 | **B** | Google Groups for role grants; membership churn is handled in the group, not in IAM. (1.1, 4.1) |
| 5 | **B** | Billing export to BigQuery + labels = SQL cost reporting. Budgets only alert. (1.2) |
| 6 | **A** | Quotas are raised via request from the Quotas page; they're protective, not absolute. (1.1) |
| 7 | **B** | `iam.securityReviewer` = read-only view of IAM across the hierarchy. (4.1) |
| 8 | **B** | Cloud Identity Free provides managed identities + an organization without Workspace. (1.1) |
| 9 | **D** | Both `--configuration` and `--project` override per-command without switching the active config. (1.3) |
| 10 | **C** | Budgets never stop spending by themselves; automation needs Pub/Sub + Cloud Function. (1.2) |
| 11 | **B** | Interruptible, checkpointed batch = Spot VMs at the deepest discount. (2.1) |
| 12 | **B** | Cloud Shell persists only the 5 GB `$HOME`; the VM (including `/tmp`) is recycled. (1.3) |
| 13 | **C** | Stateless container + spiky traffic + scale-to-zero = Cloud Run. (2.1) |
| 14 | **B** | Customer-managed keys (CMEK) = pick your KMS key on disk creation. (2.2, 4.2) |
| 15 | **A** | Cloud SQL HA gives a managed regional failover; Spanner is overkill for a regional MySQL app. (2.2) |
| 16 | **C** | Massive write throughput + key/range reads = Bigtable. (2.2) |
| 17 | **A, C** | Archive for cost; a **locked** retention policy makes deletion impossible until expiry. (2.2, 3.2) |
| 18 | **A** | VPC Peering works across projects **and organizations**; Shared VPC is same-org only. (2.3) |
| 19 | **B** | Global anycast + URL maps + CDN + bucket backends = global external Application LB. (2.3) |
| 20 | **A** | Private Google Access = Google APIs from private subnets; NAT is for general internet. (2.3, 3.3) |
| 21 | **B** | Remote state in GCS with versioning (and locking) is the standard. (2.4) |
| 22 | **C** | Fabric FAST = Google's opinionated Terraform landing-zone blueprint. (2.4) |
| 23 | **B** | Consistent 10 Gbps + SLA + private = Dedicated Interconnect; VPN rides the internet. (2.3) |
| 24 | **B** | Private zones resolve only within attached VPCs. (2.3) |
| 25 | **B** | Subnets are regional; secondary ranges serve GKE alias IPs. (2.3) |
| 26 | **B** | Artifact Registry: regional repos, per-repo IAM, scanning; Container Registry is legacy. (3.1) |
| 27 | **B** | Regional PD survives a zone outage; force-attach to a VM in the other zone. (2.1, 3.1) |
| 28 | **B** | Marketplace = fastest supported third-party deployment; you patch it afterwards. (3.1) |
| 29 | **B** | IAP tunnel use requires `iap.tunnelResourceAccessor`. (3.1, 4.1) |
| 30 | **B** | MIG rolling update with surge/unavailable controls. (3.1) |
| 31 | **B** | OOMKilled = container hit its own memory limit; node free memory is irrelevant. (3.1) |
| 32 | **B** | PITR depends on automated backups + log retention enabled **before** the incident. (3.2) |
| 33 | **B** | Lifecycle rules: SetStorageClass + Delete by age. (2.2, 3.2) |
| 34 | **B** | Drain = stop intake, finish in-flight; cancel = drop. (3.2) |
| 35 | **B** | Cloud Run jobs + Cloud Scheduler = serverless cron for containers. (3.1) |
| 36 | **B** | Alerting policy = condition (metric, threshold, duration) + notification channel. (3.4) |
| 37 | **B** | Aggregated org sink → GCS Archive = centralized, cheap, long retention. (3.4) |
| 38 | **B** | The Ops Agent tails custom log files into Cloud Logging. (3.4) |
| 39 | **A** | Object deletion is a data-access event; Data Access logs are disabled by default for Storage. (3.4, 4.1) |
| 40 | **B** | Distributed latency analysis across services = Cloud Trace. (3.4) |
| 41 | **B** | Shift 100% traffic back to the previous healthy revision. (3.1) |
| 42 | **A** | `get-credentials` writes the kubeconfig; without it kubectl has no target. (1.3, 3.1) |
| 43 | **B** | secretAccessor on the single secret = least privilege. (4.2) |
| 44 | **B** | Workload Identity Federation swaps external OIDC tokens for short-lived GCP creds — no keys. (4.2) |
| 45 | **B** | serviceAccountUser = permission to act as / attach the SA. (4.2) |
| 46 | **B** | The default CE SA historically carries project Editor; replace with least-privilege SAs. (4.2) |
| 47 | **B** | Org policy constraint enforced at the org covers all current and future projects. (1.1, 4.2) |
| 48 | **B** | dataViewer on the dataset to read + jobUser on the project to run queries. (4.1) |
| 49 | **B** | Basic roles are too broad for production; prefer predefined/custom. (4.1) |
| 50 | **B** | Config changes are Admin Activity events — always on, 400-day retention. (3.4, 4.1) |

---

## Scoring

| Score | Reading |
|---|---|
| **45–50 (90–100%)** | Ready. Book the exam. |
| **40–44 (80–88%)** | Almost. Re-read the lessons behind every miss; one more focused day. |
| **35–39 (70–78%)** | On the edge — and this is your *second* mock, so the bar is higher. Drill the weakest domain before booking. |
| **<35 (<70%)** | Not ready. Re-do the worked scenarios in the weak lessons and retake both mocks next week. |

Remember: between mock 1 and mock 2 you've now seen 100 questions. If a *third* round of practice is needed, use an external question bank — re-taking these will measure memory, not readiness.
