# Associate Cloud Engineer — Study Guide

A self-contained study guide aligned to the official **Associate Cloud Engineer Certification Exam Guide**. Everything you need to learn the material and pass the exam should be inside this folder — no external references required.

---

## Table of contents

1. [How this guide is structured](#how-this-guide-is-structured)
2. [About the exam](#about-the-exam)
3. [Recommended study plan](#recommended-study-plan)
4. [Lesson index](#lesson-index)
5. [Test-taking tips](#test-taking-tips)
6. [Master glossary](#master-glossary)
7. [Self-assessment checklist](#self-assessment-checklist)

---

## How this guide is structured

Each lesson file follows the same shape:

1. **Why this matters for the exam** — what is actually tested.
2. **Core concepts** — pure theory, the *what* and the *why*.
3. **Decision criteria** — when to choose what (most exam questions are scenarios).
4. **Console walkthrough** — where to click in the GUI.
5. **gcloud reference** — a small CLI cheat sheet at the very end.
6. **Exam gotchas** — the subtle traps Google likes to test.
7. **Practice scenarios** — 5 exam-style questions with detailed answers.

The order is deliberate: **concept → judgment → console → CLI → self-test**. The CLI is a thin layer on top of the concepts; if you understand the concept, the command becomes obvious.

---

## About the exam

### Format
- **Duration:** 2 hours.
- **Number of questions:** ~50 (multiple choice and multiple select).
- **Languages:** English, Japanese, Spanish (Latin American), Portuguese (Brazilian), French, Indonesian.
- **Delivery:** In-person at a test centre, or **online proctored** from your home/office (webcam + screen monitoring).
- **Cost:** $125 USD (subject to change; check the official exam page when you register).
- **Recertification:** The certification is valid for **3 years**.

### Scoring
- Google does **not** publish the exact passing score, but the consensus is **70–75%**.
- You receive a pass/fail result immediately at the end of the exam, and a detailed breakdown by domain a few days later.
- A failed attempt has a **14-day waiting period** before you can retake; a second fail is **60 days**; further attempts are 365 days apart.

### Question style
- **Scenario-based.** A short paragraph describes a situation; you pick the best action.
- Some questions have *multiple correct answers*; the question will explicitly say "Choose 2" or "Choose 3".
- Distractors are usually plausible — the wrong answers are typically *technically possible* but not the **best practice**, **least privileged**, or **most cost-effective** choice.
- Watch for keywords: *cheapest*, *least privileged*, *fastest*, *minimum downtime*, *Google-recommended*.

### Domain weights (memorise these)
| Domain | Weight |
|---|---|
| 1. Setting up a cloud solution environment | ~23% |
| 2. Planning and implementing a cloud solution | ~30% |
| 3. Ensuring successful operation of a cloud solution | ~27% |
| 4. Configuring access and security | ~20% |

### Registration steps
1. Go to the official certification page.
2. Create or sign in to a **Webassessor** account.
3. Choose in-person or online-proctored.
4. Pay the exam fee.
5. For online proctored: install the **Sentinel** secure browser, run a system check, and ensure you have a clean desk + valid ID.

---

## Recommended study plan

A focused 4-week plan, ~1–2 hours per day. Adjust to your schedule.

### Week 1 — Foundations
- Days 1–2: [1.1 Projects and accounts](section-1-environment/1.1-projects-and-accounts.md), [1.2 Billing](section-1-environment/1.2-billing.md).
- Days 3–4: [2.1 Compute](section-2-planning/2.1-compute.md) — first pass.
- Days 5–6: [2.2 Storage and data](section-2-planning/2.2-storage-and-data.md) — first pass.
- Day 7: Re-read both 2.x lessons; do their *Practice scenarios*.

### Week 2 — Building
- Days 1–2: [2.3 Networking](section-2-planning/2.3-networking.md).
- Day 3: [2.4 Infrastructure as code](section-2-planning/2.4-iac.md).
- Days 4–5: [3.1 Managing compute](section-3-operations/3.1-managing-compute.md), [3.2 Managing storage and data](section-3-operations/3.2-managing-storage-and-data.md).
- Days 6–7: Re-read all of section 2; do *Practice scenarios* for section 2.

### Week 3 — Operations and security
- Day 1: [3.3 Managing networking](section-3-operations/3.3-managing-networking.md).
- Days 2–3: [3.4 Monitoring and logging](section-3-operations/3.4-monitoring-and-logging.md).
- Days 4–5: [4.1 IAM](section-4-security/4.1-iam.md), [4.2 Service accounts](section-4-security/4.2-service-accounts.md).
- Days 6–7: Re-read sections 3 and 4; do their *Practice scenarios*.

### Week 4 — Consolidation
- Days 1–2: Re-read every lesson focusing on **Decision criteria** and **Exam gotchas**.
- Day 3: [Mock exam](mock-exam.md) — full 50-question pass under timed conditions.
- Day 4: Review every wrong answer; identify which domains are weakest.
- Day 5: Targeted re-reading of the weak domains.
- Day 6: Drill the *gcloud reference* sections to solidify CLI patterns.
- Day 7: Light review only. Sleep well. Take the exam.

---

## Lesson index

### Section 1 — Setting up a cloud solution environment (~23%)
- [1.1 — Setting up cloud projects and accounts](section-1-environment/1.1-projects-and-accounts.md)
- [1.2 — Managing billing configuration](section-1-environment/1.2-billing.md)

### Section 2 — Planning and implementing a cloud solution (~30%)
- [2.1 — Planning and implementing compute resources](section-2-planning/2.1-compute.md)
- [2.2 — Planning and implementing storage and data solutions](section-2-planning/2.2-storage-and-data.md)
- [2.3 — Planning and implementing networking resources](section-2-planning/2.3-networking.md)
- [2.4 — Planning and implementing resources through infrastructure as code](section-2-planning/2.4-iac.md)

### Section 3 — Ensuring successful operation of a cloud solution (~27%)
- [3.1 — Managing compute resources](section-3-operations/3.1-managing-compute.md)
- [3.2 — Managing storage and data solutions](section-3-operations/3.2-managing-storage-and-data.md)
- [3.3 — Managing networking resources](section-3-operations/3.3-managing-networking.md)
- [3.4 — Monitoring and logging](section-3-operations/3.4-monitoring-and-logging.md)

### Section 4 — Configuring access and security (~20%)
- [4.1 — Managing Identity and Access Management (IAM)](section-4-security/4.1-iam.md)
- [4.2 — Managing service accounts](section-4-security/4.2-service-accounts.md)

### Cross-cutting
- [Mock exam — 50 questions](mock-exam.md)

---

## Test-taking tips

### Before the exam
- The night before: light review only — re-read *Decision criteria* and *Exam gotchas* sections, no new material.
- Sleep 8 hours. Eat properly. Be hydrated, but don't over-caffeinate.
- For online proctored: clear your desk completely — no phones, papers, or extra monitors. Have a single bottle of water and a valid government-issued ID. Confirm webcam and microphone work.

### During the exam
- **Read the entire question and all answers** before deciding. Many distractors look right at first glance.
- **Eliminate first.** Even if you're unsure, you can usually rule out 2 obviously wrong answers, leaving a 50/50 between two plausible ones.
- **Watch the qualifiers**: *most cost-effective*, *least privileged*, *minimum downtime*, *Google-recommended*. They change the right answer.
- **Mark for review** any question you're not certain about. Don't dwell — finish the exam first, then come back. You have ~2.4 minutes per question on average.
- **Don't change answers without a clear reason.** Your first instinct is usually right unless you spot a specific mistake.
- For "Choose 2" / "Choose 3" — the system will not let you submit until you select the exact number.

### Common decision heuristics
- "Cheapest" → usually Spot/preemptible VMs, Cloud Storage Coldline/Archive, smaller machine types, on-demand BigQuery.
- "Least privileged" → predefined or custom IAM role, **never** Editor or Owner.
- "Minimum downtime" → managed migration tools (Database Migration Service, Storage Transfer Service), or HA deployments.
- "Google-recommended" → managed services over self-managed; Workload Identity over keys; OS Login over metadata SSH keys; Uniform Bucket-Level Access over ACLs.
- "Quickly" / "Without changing code" → console + managed services usually beat scripts.

### Red flags in answers
- Anything involving **service account keys** when impersonation or Workload Identity is an option → wrong.
- **Owner** or **Editor** as the role for a workload → wrong.
- **`allUsers` public access** to fix a sharing problem → wrong (use Signed URLs).
- "Just enable billing/quota *immediately*" → wrong; quota increases take time.
- Manual scripts when a managed service exists → usually wrong.

---

## Master glossary

A consolidated glossary of every term that appears across the lessons. Use this for quick lookups during study.

### Identity & access
- **IAM (Identity and Access Management)** — the GCP system for granting permissions; policies = bindings of (member, role, [condition]).
- **Member / Principal** — an identity that can be granted a role: user, group, service account, domain, or special (`allUsers`, `allAuthenticatedUsers`).
- **Role** — a collection of permissions. Three categories: **basic** (Owner/Editor/Viewer), **predefined** (per-service Google-maintained), **custom** (your own).
- **Permission** — a single action (`compute.instances.start`); roles are bundles of these.
- **Cloud Identity** — Google's identity provider (separate from Workspace); manages users and groups.
- **Service account (SA)** — a non-human identity that workloads authenticate as. Both a member (in policies) and a resource (others can impersonate it).
- **Default service account** — auto-created (Compute Engine default SA, App Engine default SA); over-privileged; replace in production.
- **Service agent** — Google-owned SA (`gcp-sa-*`) that managed services use to act in your project.
- **Impersonation** — letting one identity act as a service account via short-lived tokens; uses `roles/iam.serviceAccountTokenCreator`.
- **Workload Identity (GKE)** — bind a Kubernetes service account to a Google service account so pods authenticate without keys.
- **Workload Identity Federation** — same idea for *external* workloads (AWS, on-prem, GitHub Actions).
- **OS Login** — IAM-controlled SSH access to VMs; replaces metadata SSH keys.
- **IAP (Identity-Aware Proxy)** — Google's identity-aware reverse proxy; provides SSH/TCP tunneling to VMs without external IPs.
- **CMEK / CSEK** — Customer-Managed / Customer-Supplied Encryption Keys.
- **Org policy** — constraint at the org/folder/project that restricts what's possible (separate from IAM).

### Resource hierarchy
- **Organization** — top-level node tied to a Cloud Identity / Workspace domain.
- **Folder** — optional grouping; up to 10 levels of nesting.
- **Project** — the unit that owns resources, billing, APIs.
- **Project ID / Project number / Project name** — three identifiers per project; ID is immutable and globally unique.
- **Resource hierarchy** — Org → Folders → Projects → Resources; IAM and org policies inherit downward.

### Compute
- **Compute Engine (GCE)** — VM-as-a-service.
- **Machine type** — predefined (e2, n2, c3, m3, a3, …) or custom CPU/memory.
- **Spot VM** — heavily discounted preemptible VM with no time limit but can be reclaimed any time.
- **Persistent Disk (PD)** — network block storage for VMs; zonal or regional.
- **Hyperdisk** — newer PD variant with independently tunable IOPS/throughput.
- **Local SSD** — physically attached SSD; ephemeral (data lost on stop).
- **Instance template** — immutable VM spec used by MIGs.
- **Managed Instance Group (MIG)** — autoscaled, auto-healed fleet from a template; zonal or regional.
- **Snapshot** — point-in-time backup of a PD; incremental.
- **Custom image** — disk image used as a VM boot template.
- **GKE (Google Kubernetes Engine)** — managed Kubernetes.
- **GKE Autopilot** — Google manages nodes; you pay per pod requests.
- **GKE Standard** — you manage node pools.
- **Node pool** — group of identical worker nodes in a GKE cluster.
- **HPA / VPA / CA** — Horizontal / Vertical Pod Autoscaler / Cluster Autoscaler.
- **Cloud Run** — serverless containers; scale to zero.
- **Cloud Run jobs** — run-to-completion containers.
- **Cloud Run functions** — successor to Cloud Functions; built on Cloud Run.
- **Knative** — open-source spec Cloud Run implements.
- **Eventarc** — unified CloudEvents-based event router (Pub/Sub, Audit Logs, Cloud Storage, … → Cloud Run/GKE/Workflows).
- **Artifact Registry** — successor to Container Registry; stores container images, Helm charts, language packages.

### Storage and data
- **Cloud Storage** — object storage (buckets + objects); classes: Standard / Nearline / Coldline / Archive.
- **Bucket** — top-level container in Cloud Storage; globally unique name.
- **UBLA (Uniform Bucket-Level Access)** — IAM-only access on a bucket; preferred over ACLs.
- **Signed URL** — temporary URL with embedded credentials for one object.
- **Object lifecycle management** — automatic transitions (class change, deletion) based on rules.
- **Versioning / Retention policy / Object hold** — three protection mechanisms in Cloud Storage.
- **Cloud SQL** — managed MySQL / PostgreSQL / SQL Server.
- **AlloyDB** — high-performance PostgreSQL-compatible database.
- **Spanner** — globally distributed, strongly consistent relational DB.
- **Firestore** — document database (Native or Datastore mode).
- **Bigtable** — wide-column NoSQL for IoT / time-series / ad-tech.
- **BigQuery** — serverless analytical data warehouse.
- **Time travel (BigQuery)** — query a table at a past timestamp; default 7-day retention.
- **PITR (Point-in-Time Recovery)** — Cloud SQL/AlloyDB/Spanner feature for restoring to a specific moment.
- **Filestore** — managed NFS file storage.
- **NetApp Volumes** — managed ONTAP (NFS + SMB).
- **Memorystore** — managed Redis / Memcached.
- **Pub/Sub** — global publish/subscribe messaging.
- **Pub/Sub Lite** — cheaper, zonal, lower-throughput Pub/Sub variant.
- **Managed Kafka** — Google's managed Apache Kafka service.
- **Dataflow** — managed Apache Beam (streaming + batch).
- **Storage Transfer Service** — online transfers from S3/Azure/on-prem to GCS.
- **Transfer Appliance** — physical box for offline petabyte transfers.
- **Database Migration Service (DMS)** — online migration of MySQL/Postgres/Oracle/SQL Server to Cloud SQL/AlloyDB.
- **BigQuery Data Transfer Service** — scheduled imports into BigQuery from SaaS sources.
- **Database Center** — fleet console across all Google databases.

### Networking
- **VPC (Virtual Private Cloud)** — global private network on Google Cloud.
- **Subnet** — regional CIDR block within a VPC.
- **Auto / Custom mode VPC** — auto-mode auto-creates subnets, custom-mode is explicit.
- **Shared VPC** — host project owns VPC, service projects use it.
- **VPC Network Peering** — non-transitive private connectivity between two VPCs.
- **Cloud VPN** — IPsec VPN over the public internet (Classic 99.9% / HA 99.99% SLA).
- **Cloud Interconnect** — Dedicated (your own fiber) / Partner (via telco) / Cross-Cloud.
- **Cloud Router** — runs BGP for dynamic route exchange across hybrid links.
- **Cloud NAT** — managed outbound NAT for VMs without external IPs.
- **Private Google Access** — subnet toggle so internal-only VMs can reach Google APIs.
- **Cloud DNS** — managed authoritative DNS (public, private, forwarding, peering zones).
- **Cloud NGFW (Next Generation Firewall)** — Google's firewall product; rules + hierarchical policies.
- **Network tag / Secure tag / Service account (firewall target)** — three ways to identify which VMs a rule applies to. Secure tags + service accounts are the modern preference.
- **Network Service Tier** — Premium (Google backbone end-to-end) vs. Standard (cheaper, public internet).
- **Application Load Balancer** — L7 HTTP(S) LB; global or regional, internal or external.
- **Network Load Balancer** — L4 TCP/UDP; passthrough (preserves client IP) or proxy.
- **Alias IP / Secondary range** — extra IPs assigned to a VM/subnet; used by GKE for pods.

### Observability & operations
- **Cloud Observability** — umbrella for Monitoring, Logging, Trace, Profiler, Error Reporting.
- **Metrics scope** — a project that aggregates metrics from one or more monitored projects.
- **Alerting policy** — condition + duration + notification channel.
- **Logs-based metric** — counter or distribution metric derived from log filter.
- **Log bucket** — storage for logs; `_Required` (audit logs, immutable, 400-day retention) and `_Default` (everything else).
- **Log Router / sink** — routes logs to destinations: BigQuery, Cloud Storage, Pub/Sub, another bucket/project.
- **Log Analytics** — SQL queries over upgraded log buckets without exporting.
- **Audit logs** — Admin Activity, System Event, Data Access (mostly off by default), Policy Denied.
- **Ops Agent** — unified VM agent for system metrics + logs.
- **Managed Service for Prometheus** — Google-managed Prometheus stack.
- **Cloud Trace / Profiler / Error Reporting** — distributed tracing / continuous profiler / aggregated errors.
- **Active Assist** — recommendations across IAM, sizing, idle resources, firewall insights, etc.
- **Database Center / Service Health / Gemini Cloud Assist** — newer cross-cutting consoles.

### Billing
- **Billing account** — payment instrument; pays for one or more projects.
- **Self-serve vs. Invoiced** — billing account types (credit card vs. monthly invoice).
- **Project Billing Manager / Billing Account User** — minimum role pair to link a project to a billing account.
- **Budget** — monitoring construct (not a hard cap); fires alerts at thresholds.
- **Billing export** — to BigQuery (analysis), Cloud Storage (archive); standard or detailed schemas.
- **Pricing calculator** — pre-purchase forecast tool.

### IaC
- **Terraform** — declarative HCL IaC; primary GCP IaC tool.
- **Fabric FAST** — Google's opinionated Terraform reference for org bootstrapping.
- **Config Connector** — manage GCP resources from Kubernetes manifests.
- **Helm** — Kubernetes package manager (charts); for K8s workloads, not GCP resources.
- **State (Terraform)** — source of truth for what Terraform manages; remote state in GCS recommended.

---

## Self-assessment checklist

Print this and tick boxes as you study. By exam day every box should be checked.

### Section 1 — Environment
- [ ] I can draw the resource hierarchy and explain inheritance for IAM and org policies.
- [ ] I can explain the difference between an organization, folder, project, and resource.
- [ ] I know all three project identifiers (name, ID, number) and which is mutable.
- [ ] I can grant IAM at the project/folder/org level via console and gcloud.
- [ ] I understand Cloud Identity vs. Workspace; users vs. groups; standalone organization.
- [ ] I can enable APIs in a project and recognize "API not enabled" errors.
- [ ] I know how to request a quota increase and that they are not instantaneous.
- [ ] I know what Cloud Asset Inventory does and that it keeps 35 days of history.
- [ ] I can create and link a billing account; I know the dual-role requirement.
- [ ] I can set up a budget with thresholds and Pub/Sub notifications.
- [ ] I can export billing to BigQuery and Cloud Storage.
- [ ] I understand labels for cost allocation.

### Section 2 — Planning
- [ ] I can pick the right compute service (GCE / GKE / Cloud Run / Cloud Run functions) for a workload.
- [ ] I can launch a VM with appropriate machine type, image, network, SA, OS Login, availability policy.
- [ ] I can choose the right disk (zonal/regional PD, Hyperdisk, Local SSD).
- [ ] I can create a MIG with autoscaling and an instance template.
- [ ] I understand Spot VMs, custom machine types, sole-tenant nodes.
- [ ] I can install kubectl and configure it for a GKE cluster.
- [ ] I know GKE Autopilot vs. Standard, regional vs. zonal, public vs. private clusters.
- [ ] I can deploy a containerized app to GKE and to Cloud Run.
- [ ] I can configure event triggers (Pub/Sub, Cloud Storage, Eventarc).
- [ ] I can pick the right data product for any workload type.
- [ ] I can pick the right Cloud Storage class and explain min-storage-duration charges.
- [ ] I can load data via gcloud, Storage Transfer Service, Transfer Appliance, DMS.
- [ ] I understand multi-region redundancy options for each data service.
- [ ] I can create a custom-mode VPC with subnets across regions.
- [ ] I can write firewall rules with secure tags and service accounts as targets.
- [ ] I know when to use VPC peering vs. VPN vs. Interconnect.
- [ ] I can pick the right load balancer for a scenario.
- [ ] I understand Premium vs. Standard network tiers.
- [ ] I know what Terraform, Fabric FAST, Config Connector, Helm each do.
- [ ] I understand Terraform remote state in GCS with locking and versioning.

### Section 3 — Operations
- [ ] I can SSH a VM (console, gcloud, IAP tunnel).
- [ ] I can manage snapshots, custom images, and snapshot schedules.
- [ ] I can view GKE inventory and configure GKE → Artifact Registry.
- [ ] I can add/remove node pools and configure autoscaling.
- [ ] I know HPA, VPA, Cluster Autoscaler — what they scale and when.
- [ ] I can deploy a new Cloud Run revision with traffic splitting and rollback.
- [ ] I can set Cloud Run autoscaling parameters (min/max instances, concurrency, CPU).
- [ ] I can manage and secure Cloud Storage objects; set lifecycle policies.
- [ ] I can run queries against each data product.
- [ ] I can back up and restore each data product (Cloud SQL PITR, Spanner backups, etc.).
- [ ] I can review job status (BigQuery, Dataflow drain vs. cancel).
- [ ] I can add a subnet, expand a subnet, reserve static IPs.
- [ ] I can add custom static routes; use Cloud DNS (private/forwarding); deploy Cloud NAT.
- [ ] I can create a Cloud Monitoring alert from a metric.
- [ ] I can create a logs-based metric.
- [ ] I can configure Log Router sinks (BigQuery, GCS, Pub/Sub, log bucket).
- [ ] I can use Log Analytics on a log bucket.
- [ ] I know what Audit Log types exist and which are off by default.
- [ ] I know Cloud Trace, Profiler, Error Reporting, Query Insights, index advisor purposes.
- [ ] I can install Ops Agent and enable Managed Service for Prometheus.

### Section 4 — Security
- [ ] I can read an IAM policy and determine effective access for a user.
- [ ] I can pick the right role type (basic / predefined / custom) for a scenario.
- [ ] I can create a custom role with specific permissions.
- [ ] I understand IAM Conditions and time-bounded access.
- [ ] I can use IAM Policy Troubleshooter to investigate access issues.
- [ ] I understand the dual nature of service accounts (member + resource).
- [ ] I can create a user-managed SA and replace a default one.
- [ ] I know `serviceAccountUser` vs. `serviceAccountTokenCreator` exactly.
- [ ] I can configure GKE Workload Identity end-to-end.
- [ ] I understand short-lived credentials and impersonation as the alternative to keys.
- [ ] I know when SA keys are appropriate (rarely) and the org policy that disables them.

### Final readiness
- [ ] I scored ≥80% on the [mock exam](mock-exam.md).
- [ ] I have re-read all *Exam gotchas* sections.
- [ ] I can complete each lesson's *Practice scenarios* without checking the answer.
- [ ] I have my Webassessor account, payment confirmed, ID ready.
- [ ] (Online proctored) Sentinel installed, system check passed, desk ready.
