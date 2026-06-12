# GCP Associate Cloud Engineer — Easier Study Guide

> Same content as the original `001-study-model-gcp-ace-study-guide/`, rewritten in plainer English with more structure, friendlier intros, and "common confusions" callouts. Every lab, command, scenario, and Q&A from the originals is preserved 1:1 — only the wrapping is gentler.

---

## How this guide is organized

Each lesson follows the same shape so you always know where to look:

1. **Why this matters for the exam** — what you'll be tested on.
2. **Plain-English intro** — the topic in everyday terms.
3. **Core concepts** — the must-know building blocks.
4. **gcloud commands you'll see** — what to memorize *for recognition* (you don't need to type them from scratch).
5. **Common confusions** — the traps the exam loves.
6. **Worked scenarios** — problem → reasoning → answer.
7. **Mini-Q&A** — flashcard-style review.
8. **Hands-on lab** — optional, do these on your own GCP project.
9. **Exam gotchas** — quick-reference list.
10. **TL;DR recap** — the elevator-pitch summary.
11. **Cross-references** — where else this concept shows up.

---

## Table of contents

### Section 1 — Setting up a cloud environment (~17%)
- [1.1 Setting up cloud projects and accounts](section-1-environment/1.1-projects-and-accounts.md)
- [1.2 Managing billing](section-1-environment/1.2-billing.md)
- [1.3 Cloud SDK and shell](section-1-environment/1.3-cloud-sdk-shell.md)

### Section 2 — Planning and configuring a cloud solution (~17%)
- [2.1 Compute resources](section-2-planning/2.1-compute.md)
- [2.2 Data storage options](section-2-planning/2.2-storage.md)
- [2.3 Network resources](section-2-planning/2.3-networking.md)
- [2.4 Infrastructure as Code (Terraform, Fabric FAST, Config Connector, Helm)](section-2-planning/2.4-iac.md)

### Section 3 — Deploying and implementing a cloud solution (~25%)
- [3.1 Managing compute (VMs, GKE, Cloud Run, Functions)](section-3-operations/3.1-managing-compute.md)
- [3.2 Managing storage and databases](section-3-operations/3.2-managing-storage-and-data.md)
- [3.3 Managing networking](section-3-operations/3.3-managing-networking.md)
- [3.4 Monitoring, logging, and observability](section-3-operations/3.4-monitoring-and-logging.md)

### Section 4 — Ensuring successful operation (~25%)
- [4.1 Identity and Access Management (IAM)](section-4-security/4.1-iam.md)
- [4.2 Service accounts](section-4-security/4.2-service-accounts.md)

### Top-level practice
- [Plan de studiu — 28 de zile, anti-amânare](plan-de-studiu.md) ← **începe de aici**
- [Exam strategy and gcloud cheat-sheet](exam-strategy.md)
- [50-question mock exam](mock-exam.md)
- [Mock exam 2 — 50 fresh questions](mock-exam-2.md)
- [Cram sheet — every number and hard fact, one page](cram-sheet.md)
- [Original Udemy notes](from-udemy.txt)

---

## The exam at a glance

| Item | Value |
|---|---|
| Format | Multiple choice / multiple select |
| Questions | ~50–60 |
| Time | 2 hours |
| Passing score | Google doesn't publish it — most people aim for **≥80%** to feel safe |
| Validity | 3 years |
| Cost (USD) | ~$125 |
| Delivery | Online proctored or test center |

**Domain weights (roughly):**

| Domain | Share |
|---|---|
| Setting up a cloud environment | ~17% |
| Planning and configuring | ~17% |
| Deploying and implementing | ~25% |
| Ensuring successful operation | ~25% |
| Configuring access and security | ~16% |

> The exam is **breadth, not depth**. You don't need to be an expert in any one product — you need to recognize when each product is the right answer.

---

## Suggested 4-week study plan

### Week 1 — Foundations
**Goal:** Resource hierarchy, IAM, billing.
- Read 1.1, 1.2, 1.3.
- Read 4.1 (IAM) early because it shows up everywhere else.
- Lab: Create a project, set up a budget, run a few `gcloud` commands.

### Week 2 — Compute and storage
**Goal:** Pick the right product for the workload.
- Read 2.1 (compute), 2.2 (storage).
- Build a memory map: "Stateless container with bursty traffic → Cloud Run." "Globally distributed strongly consistent SQL → Spanner."
- Lab: Spin up a Compute Engine VM, deploy a Cloud Run service, create a GCS bucket with lifecycle.

### Week 3 — Networking and IaC
**Goal:** VPCs, load balancers, Cloud NAT, Terraform.
- Read 2.3, 2.4.
- Read 3.3 in parallel — it's the operational side of 2.3.
- Lab: Create a custom VPC with two subnets, a firewall rule, Cloud NAT, and a global external Application LB.

### Week 4 — Operations and security
**Goal:** Day-2 tasks and guardrails.
- Read 3.1, 3.2, 3.4 (operations).
- Read 4.2 (service accounts) — second-most-tested topic after IAM.
- Take the [mock exam](mock-exam.md) at the end of the week. Score ≥80% before booking the real exam.

---

## Test-taking tips (the "decision heuristics")

The exam loves question stems with one of these phrases. Train yourself to recognize them:

| Phrase in question | What Google wants you to pick |
|---|---|
| **"Cheapest"** / "lowest cost" | Spot VMs, Coldline/Archive storage, Standard tier networking, sustained-use discounts. |
| **"Least privileged"** | Custom role > predefined role > basic role. Never `roles/owner`. |
| **"Minimum operations"** / "managed" | Cloud Run > GKE Autopilot > GKE Standard > Compute Engine. |
| **"No data loss"** | Drain (not cancel) for Dataflow. PITR with binary logs. Regional PD. |
| **"Globally distributed"** | Spanner. Global external Application LB (Premium tier). Multi-region GCS. |
| **"Compliance / audit"** | Data Access audit logs (must be enabled). Cloud Asset Inventory. Org policies. |
| **"Stateless container"** | Cloud Run. |
| **"Bursty traffic, scale to zero"** | Cloud Run or Cloud Run functions. |
| **"Without an external IP"** | Cloud NAT for egress. IAP for SSH. Private Google Access for Google APIs. |
| **"Rollback safely"** | Cloud Run traffic-shift to prior revision. |
| **"Time-limited access"** | IAM Conditions with `request.time`. |
| **"Without keys"** | Workload Identity (GKE) or Workload Identity Federation (external). Impersonation. |

### Red flags in answer choices

If you see these in an option, it's almost always wrong:

- `roles/owner` or `roles/editor` granted to humans → over-privileged.
- "Manually delete each…" / "Loop over every project…" → there's almost always a managed equivalent.
- "Disable the API" / "Restart the VM" as a fix → too blunt.
- "Public IP" / "allUsers" on private data → security violation.
- "JSON service-account key" when Workload Identity / impersonation exists → exam wants you to avoid keys.

---

## Master glossary (~110 terms)

The same glossary as the original guide — keep this page open while you read lessons.

### Resource hierarchy and IAM
- **Organization** — top of the GCP hierarchy. Tied to a Cloud Identity / Workspace domain.
- **Folder** — optional grouping under the org. Used per business unit, environment, or team.
- **Project** — the unit of resource isolation, billing, and quota. Every resource lives in exactly one project.
- **Resource** — anything you create (VM, bucket, BigQuery dataset, etc.).
- **IAM policy** — the set of bindings on a resource. Each binding = role + members.
- **Member / principal** — a user, group, service account, domain, `allAuthenticatedUsers`, or `allUsers`.
- **Role** — a bundle of permissions. Three types: basic, predefined, custom.
- **Permission** — a single allowed action like `compute.instances.start`.
- **Inheritance** — IAM bindings flow downward and **add** (never subtract).
- **IAM Condition** — CEL expression that gates a binding (e.g., time-limited).
- **Policy Troubleshooter** — tool that explains *why* a user does or doesn't have an effective permission.
- **Org policy** — guardrail on what *can* be done in an org/folder/project (e.g., disable public IPs). Distinct from IAM.
- **Service Account (SA)** — non-human identity. Has both a member side (it can act) and a resource side (you grant access *to* it).
- **Default SA** — auto-created with broad roles. Compute Engine's default has `roles/editor`. Replace in production.
- **Workload Identity** — GKE feature mapping a Kubernetes ServiceAccount to a Google SA — no JSON keys.
- **Workload Identity Federation** — same idea for external workloads (AWS, Azure, GitHub Actions, on-prem).
- **`serviceAccountUser`** — lets a member *attach/use* an SA (`actAs`). Required to deploy as that SA.
- **`serviceAccountTokenCreator`** — lets a member *impersonate* an SA (mint short-lived tokens).
- **`serviceAccountAdmin`** — lets a member *manage* SAs (create/delete/key-control).

### Compute
- **Compute Engine (GCE)** — VMs (IaaS).
- **Machine type** — `e2`, `n2`, `c3`, `m3`, etc. — CPU/memory profile.
- **Machine family** — general (e2/n2), compute-optimized (c3), memory-optimized (m3).
- **Sole-tenant node** — physical host dedicated to your project (compliance).
- **Spot VM** — discount VM that can be preempted. Use for fault-tolerant batch.
- **Preemptible VM** — older form of spot, 24-hour max. Spot is the recommended modern option.
- **Instance template** — immutable blueprint for MIG VMs.
- **Managed Instance Group (MIG)** — autoscaling group of identical VMs from a template. Zonal or regional.
- **Health check** — used by MIGs and load balancers to decide if instances/backends are healthy.
- **GKE** — managed Kubernetes.
- **GKE Standard** — you manage the node pools.
- **GKE Autopilot** — Google manages nodes; you only see pods.
- **Node pool** — set of nodes in a cluster sharing the same machine type/SA.
- **Cluster Autoscaler** — adds/removes *nodes*.
- **HPA / VPA** — Horizontal/Vertical Pod Autoscaler — scales pods (replicas / resource requests).
- **Cloud Run** — serverless containers. Stateless. HTTP, scale-to-zero.
- **Cloud Run jobs** — finite-duration container runs (no HTTP).
- **Cloud Run functions (2nd gen)** — Cloud Run with event triggers (formerly "Cloud Functions").
- **App Engine standard / flexible** — older PaaS. Standard has language-specific runtimes; flexible is container-based.

### Storage and data
- **Cloud Storage (GCS)** — object storage. Buckets and objects.
- **Storage class** — Standard / Nearline / Coldline / Archive — price vs. access frequency tradeoff.
- **Lifecycle Management** — automated transitions/deletions on objects.
- **UBLA (Uniform Bucket-Level Access)** — IAM-only; disables ACLs.
- **Signed URL** — time-bounded URL grants access without IAM.
- **Persistent Disk (PD)** — block storage for VMs. Zonal or regional.
- **Hyperdisk** — newer, higher-perf PD family.
- **Local SSD** — physically attached to host. Ephemeral (lost on stop/migrate).
- **Filestore** — managed NFS.
- **Cloud SQL** — managed MySQL/Postgres/SQL Server. Zonal HA + read replicas.
- **AlloyDB** — managed Postgres-compatible, higher perf than Cloud SQL.
- **Spanner** — globally distributed, strongly consistent, horizontally scalable SQL. Premium product.
- **BigQuery (BQ)** — serverless data warehouse. Datasets are regional/multi-regional and **immutable** in location.
- **Bigtable** — wide-column NoSQL. Single-region clusters; multi-cluster instances for replication.
- **Firestore** — document NoSQL. Native or Datastore mode.
- **Memorystore** — managed Redis or Memcached.
- **Pub/Sub** — global async messaging. Decouples producers from consumers.
- **Dataflow** — managed Apache Beam (streaming + batch).
- **Dataproc** — managed Spark/Hadoop.
- **Dataform** — SQL-based data transformations in BigQuery.
- **Storage Transfer Service** — online migration to GCS.
- **Transfer Appliance** — offline ship-a-disk for petabytes.
- **Database Migration Service** — for moving DBs to GCP.
- **PITR (Point-in-Time Recovery)** — restore a DB to any moment within a window. Cloud SQL needs binary logs/WAL.
- **Time travel (BigQuery)** — query data as of a past timestamp (7 days default).
- **Snapshot schedule** — resource policy attached to a PD for automatic snapshots.

### Networking
- **VPC** — global virtual network. Subnets are regional.
- **Auto-mode VPC** — one subnet per region, auto-assigned ranges (default project gets one).
- **Custom-mode VPC** — you create subnets explicitly. Recommended for prod.
- **Subnet** — regional IP range inside a VPC.
- **Secondary IP range** — alias range on a subnet (used for GKE pods/services).
- **Firewall rule** — VPC-level allow/deny ingress/egress. Targeted by tags, secure tags, or service accounts.
- **Network tag vs. secure tag** — network tags can be self-applied (insecure); secure tags are IAM-controlled.
- **Static IP** — reserved IP. Internal/external. Regional or global.
- **Cloud NAT** — egress-to-internet without external IPs.
- **Private Google Access** — VMs without external IPs reach Google APIs/services privately.
- **Private Service Connect (PSC)** — private endpoints for Google or third-party services.
- **VPC Network Peering** — peers two GCP VPCs (no overlapping CIDRs, non-transitive).
- **Shared VPC** — host project owns network; service projects use it.
- **Cloud VPN** — IPsec VPN to on-prem (HA VPN preferred).
- **Cloud Interconnect** — Dedicated (direct) or Partner — physical link to GCP.
- **Cloud Router** — BGP for dynamic routing (Cloud VPN/Interconnect/NAT).
- **Cloud Load Balancing** — global external Application LB, regional external/internal Application LB, external/internal Network LB (passthrough or proxy).
- **Premium vs Standard tier** — Premium = global Google backbone. Standard = regional only.
- **Cloud DNS zones** — public, private, forwarding, peering.
- **Cloud CDN** — caches HTTP(S) origin behind a global LB.
- **Cloud Armor** — WAF / DDoS at the LB.

### Observability
- **Cloud Logging** — log storage and routing.
- **Log bucket** — `_Required` (audit, 400 days, immutable) and `_Default` (everything else, 30 days, configurable).
- **Log Router** — routes logs to sinks.
- **Sink destinations** — BigQuery, Cloud Storage, Pub/Sub, another log bucket.
- **Logs-based metric** — counter or distribution metric derived from log entries.
- **Cloud Monitoring** — metrics, dashboards, alerts.
- **Metrics scope** — single dashboard pane covering many projects.
- **Alerting policy** — condition + notification channels.
- **Uptime check** — synthetic HTTP/TCP/SSL probes from multiple regions.
- **Audit logs** — Admin Activity (always on), Data Access (off by default for most), System Event, Policy Denied.
- **Cloud Trace** — distributed traces.
- **Cloud Profiler** — CPU/memory profiles.
- **Error Reporting** — groups exceptions by stack trace.
- **Ops Agent** — unified VM agent for metrics + logs (replaces older agents).
- **Managed Service for Prometheus** — GMP.
- **Personalized Service Health** — Google Cloud incident dashboard tailored to your projects.
- **Active Assist** — recommendations (rightsizing, IAM cleanup, etc.).

### Billing and ops
- **Billing account** — pays for one or more projects.
- **Budget** — *alerts*, not a hard cap. Combine with Pub/Sub + Cloud Function for hard cap.
- **Quota** — hard limit per project per service. Some increase via support.
- **Pricing Calculator** — pre-deployment estimate.
- **Cost reports / Billing dashboards** — actual spend after the fact.
- **Committed Use Discount (CUD)** — 1- or 3-year commitment for compute discount.
- **Sustained Use Discount (SUD)** — automatic discount for long-running VMs.

### IaC
- **Terraform** — HashiCorp tool for managing infrastructure as code.
- **Provider** — plugin (e.g., `google`, `google-beta`).
- **State** — Terraform's record of real resources.
- **Remote backend** — state stored in GCS for collaboration; supports locking.
- **Module** — reusable Terraform package.
- **Fabric FAST** — Google's opinionated Terraform foundation.
- **Cloud Foundation Toolkit (CFT)** — Google-published Terraform modules.
- **Config Connector** — manage GCP via Kubernetes manifests.
- **Helm** — Kubernetes package manager (not GCP-specific).
- **Cloud Deployment Manager** — older Google IaC (use Terraform now).

---

## Self-assessment checklist

Tick every box. If something is shaky, re-read the linked lesson.

### Identity, hierarchy, billing
- [ ] I can explain how IAM bindings inherit and what "additive" means.
- [ ] I know the difference between basic, predefined, and custom roles.
- [ ] I can pick the right level (org / folder / project / resource) for a binding.
- [ ] I know that a budget *alerts*, not blocks — and the Pub/Sub + Cloud Function pattern for a hard cap.
- [ ] I know `gcloud projects undelete` works for ~30 days.

### Compute
- [ ] I can match a workload to: Compute Engine, MIG, GKE Standard, GKE Autopilot, Cloud Run, Cloud Run jobs, Cloud Run functions, App Engine.
- [ ] I know instance templates are immutable; updating means new template + rolling update.
- [ ] I know HPA vs VPA vs Cluster Autoscaler.
- [ ] I know Spot VMs are for fault-tolerant batch, not steady-state.
- [ ] I can describe Workload Identity in 3 steps.

### Storage and data
- [ ] I can pick a storage class (Standard/Nearline/Coldline/Archive) by access pattern.
- [ ] I know UBLA, Signed URLs, and lifecycle rules.
- [ ] I know which database fits which workload (Cloud SQL vs Spanner vs BQ vs Firestore vs Bigtable).
- [ ] I know PITR requires binary logs/WAL enabled in advance.
- [ ] I know `drain` vs `cancel` for Dataflow.

### Networking
- [ ] I can grow a subnet but not shrink it.
- [ ] I know secondary ranges are for GKE pods/services.
- [ ] I know static IPs are global or regional, internal or external.
- [ ] I know Cloud NAT vs Private Google Access.
- [ ] I know which load balancer fits which use case.
- [ ] I know peering, Shared VPC, VPN, and Interconnect by description.

### Observability
- [ ] I know `_Required` vs `_Default` log buckets.
- [ ] I know the four sink destinations (BQ, GCS, Pub/Sub, log bucket) and when to use each.
- [ ] I know Admin Activity vs Data Access audit logs.
- [ ] I know Ops Agent is the unified VM agent.
- [ ] I can describe a metrics scope.

### Security
- [ ] I know the IAM Conditions pattern for time-limited access.
- [ ] I know the SA dual nature (member + resource).
- [ ] I know `serviceAccountUser` vs `serviceAccountTokenCreator`.
- [ ] I know the recommended pattern is impersonation, not JSON keys.
- [ ] I know the `iam.disableServiceAccountKeyCreation` org policy.

### IaC
- [ ] I know remote state in GCS + locking + versioning.
- [ ] I know Fabric FAST is the Google-published opinionated foundation.
- [ ] I know Config Connector lets you manage GCP from Kubernetes manifests.
- [ ] I know Helm is for Kubernetes apps, not GCP infra.

---

## When you're ready

1. Take the [mock exam](mock-exam.md) under timed conditions (2 hours).
2. Review every wrong answer's linked lesson — *and* the right ones you guessed.
3. Re-read your weakest section.
4. Book the real exam when you score ≥80% comfortably.

Good luck — you've got this. The exam rewards recognition over recall, so trust your reading and don't overthink obvious-feeling answers.
