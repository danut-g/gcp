# Cram sheet — every number and hard fact, one page

> Read this the evening before and the morning of the exam. Everything here is covered in depth in the lessons — this is pure recall fuel.

---

## Numbers to know cold

| Fact | Value |
|---|---|
| Project undelete window | **30 days** (then gone; Project ID never reusable) |
| Folder nesting depth | up to **10 levels** |
| Project identifiers | Name (mutable) / **ID (immutable, global)** / Number (auto) |
| Exam retake waits | fail #1 → **14 days**, #2 → **60 days**, #3 → **365 days** |
| Certification validity | **3 years** |
| Cloud Shell persistent disk | **5 GB** at `$HOME` only |
| Cloud Shell idle timeout | ~**20 min**; weekly cap ~**50 h**; ~**120 days** unused → home disk may be wiped |
| Audit logs: Admin Activity retention | **400 days**, always on, free |
| Audit logs: Data Access | **OFF by default** (except BigQuery), 30 days when on |
| Cloud Logging _Default bucket retention | **30 days** (configurable 1–3650) |
| Storage minimums | Nearline **30 d** / Coldline **90 d** / Archive **365 d** |
| Storage access pattern | Standard = hot, Nearline = monthly, Coldline = quarterly, Archive = yearly |
| Cloud Run default concurrency | **80** requests/instance |
| Sustained use discount | automatic on Compute Engine (no action needed) |
| Spot VMs | up to ~**60–91%** cheaper; can be preempted any time; no SLA |
| CUD (committed use) | **1 or 3 years**, biggest planned-workload discount |
| Live migration | Google patches hosts with **no VM downtime** (not on Spot/GPU) |
| Regional Persistent Disk | synchronous replication across **2 zones** |
| Default VPC firewall | implied **deny all ingress**, **allow all egress** |
| Firewall rule priority | **0–65535**, lower number = higher priority (default 1000) |
| VPC | **global**; subnets **regional**; subnets can expand, never shrink |

---

## Decision triggers — product in one line

| Trigger phrase | Answer |
|---|---|
| Stateless container, spiky traffic, scale to zero | **Cloud Run** |
| Scheduled container batch job | **Cloud Run jobs + Cloud Scheduler** |
| Full Kubernetes control | **GKE** (Autopilot = less ops, Standard = more control) |
| Event-driven small code (file lands in bucket) | **Cloud Run functions** |
| Lift-and-shift VM / custom OS / licenses | **Compute Engine** |
| Petabyte analytical SQL | **BigQuery** |
| Regional OLTP MySQL/Postgres | **Cloud SQL** (HA = regional config) |
| Global OLTP, strong consistency | **Spanner** |
| Postgres outgrowing Cloud SQL | **AlloyDB** |
| Mobile/web app docs, offline sync | **Firestore** |
| Huge write throughput, key/time-series | **Bigtable** |
| Cache | **Memorystore** |
| Messaging / decouple services | **Pub/Sub** |
| Stream/batch ETL pipelines | **Dataflow** |
| Existing Hadoop/Spark | **Dataproc** |
| NFS file share | **Filestore** |
| Objects, backups, static site | **Cloud Storage** |
| ~PB offline transfer / no bandwidth | **Transfer Appliance** |
| Recurring online transfer (S3, HTTP, buckets) | **Storage Transfer Service** |
| Third-party app fast (WordPress, GitLab) | **Cloud Marketplace** |
| DB passwords / API keys | **Secret Manager** |
| Own the encryption keys (CMEK) | **Cloud KMS** |
| SSH to VM without external IP | **IAP TCP forwarding** |
| Private subnet → Google APIs | **Private Google Access** |
| Private subnet → general internet | **Cloud NAT** |
| Connect 2 VPCs (no overlap), even cross-org | **VPC Peering** |
| Central network, many teams' projects, same org | **Shared VPC** |
| On-prem ↔ GCP, cheap/quick over internet | **Cloud VPN (HA VPN, 99.99%)** |
| On-prem ↔ GCP, dedicated bandwidth + SLA | **Interconnect** (Dedicated / Partner) |
| Global HTTP(S), one anycast IP, CDN, URL routing | **Global external Application LB** |
| Non-HTTP TCP/UDP, preserve client IP, regional | **External passthrough Network LB** |
| Hadoop-style logs searchable / who-did-what | **Cloud Logging / Admin Activity audit logs** |
| Latency across microservices | **Cloud Trace** |
| CPU/heap profiling in prod | **Cloud Profiler** |
| Is my endpoint up from around the world? | **Uptime check + alerting policy** |

---

## IAM in 10 lines

1. Hierarchy: **Org → Folder → Project → Resource**; IAM flows **down**.
2. IAM combines by **UNION** (most permissive wins); a child can't remove a parent's grant.
3. Org Policy combines by **INTERSECTION** (most restrictive wins); a child can't loosen it.
4. **Basic roles** (Owner/Editor/Viewer) = too broad for prod; prefer **predefined**; **custom** only when predefined doesn't fit.
5. Grant to **groups**, not users.
6. Least privilege: grant at the **lowest level** that satisfies the need; on the **narrowest resource** (e.g., one secret, one dataset).
7. Service account = identity for **workloads**; it's both a member **and** a resource.
8. `serviceAccountUser` = attach/act-as; `serviceAccountTokenCreator` = impersonate/mint tokens.
9. Default Compute SA carries project **Editor** → replace with dedicated least-privilege SAs.
10. No JSON keys when avoidable: attached SAs in GCP, **Workload Identity** on GKE, **Workload Identity Federation** outside GCP.

---

## gcloud one-liners you must recognize

```bash
gcloud init                                   # wizard: login + project + defaults
gcloud auth list                              # who am I
gcloud auth application-default login         # creds for local CODE (ADC)
gcloud config set project ID                  # default project
gcloud config configurations activate NAME    # switch profile
gcloud services enable api.googleapis.com     # fix "API not enabled"
gcloud projects undelete ID                   # within 30 days
gcloud compute ssh VM --tunnel-through-iap    # SSH, no external IP
gcloud container clusters get-credentials C   # make kubectl work
gcloud run services update-traffic S --to-revisions=REV=100   # rollback
gcloud compute instances list --filter=... --format=json
gcloud secrets versions access latest --secret=NAME
```

---

## Classic traps (each costs people a question)

- Budgets **only alert** — hard stop needs Pub/Sub → Cloud Function → disable billing.
- BigQuery dataset location and Firestore mode are **immutable** — copy/recreate to change.
- Instance templates are **immutable** — new template + MIG rolling update.
- `OOMKilled` = pod hit **its own limit**, not node memory.
- PITR on Cloud SQL must be enabled **before** the disaster.
- **Drain** (finish in-flight) vs **Cancel** (drop) on Dataflow.
- Data Access audit logs are **off by default** — "who read the object?" may be unanswerable.
- "Invalid choice: beta" → `gcloud components install beta` (not IAM).
- Cloud Shell: only `$HOME` survives; wrong tool for long-running jobs.
- Marketplace deploys into **your** project — **you** patch it afterwards.
- Peering works **cross-org**; Shared VPC is **same-org**. Peering is **not transitive**.
- Retention policy **lock** is irreversible until expiry — even for admins.
- The question's keywords decide: *cheapest* / *least privilege* / *no ops* / *Google-recommended* — answer for the keyword, not for what you'd personally build.

---

*Sleep well. On exam day: flag the long scenario questions, finish the short ones first, and never leave a question blank — there's no penalty for guessing.*
