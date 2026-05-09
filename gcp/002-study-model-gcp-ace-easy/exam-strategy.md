# Exam strategy and gcloud cheat-sheet

Same content as the original `001-study-model-gcp-ace-study-guide/exam-strategy.md`, rewritten for clarity. Use this as your final-week revision page.

---

## The honest truth about gcloud commands

Everyone worries about memorizing gcloud. **You don't have to write commands from scratch.** The exam shows you four full commands and asks which one is correct. So your job is **recognition**, not recall.

That said — when you read a question stem like "Which command creates a Cloud Storage bucket with uniform access?", you should be able to mentally complete the command stub before looking at the choices. That's how you spot wrong distractors fast.

**What you actually need:**

1. The verb-noun structure: `gcloud <product> <thing> <action>`. E.g., `gcloud compute instances create`, `gcloud iam service-accounts create`.
2. The 5–10 most common flags per product.
3. The shape of the output (so you spot fabricated/fake commands).

---

## CLI-to-job mapping

The first word after `gcloud` is the product. Memorize this list:

| Goal | Command stem |
|---|---|
| VM operations | `gcloud compute instances …` |
| MIG operations | `gcloud compute instance-groups managed …` |
| Disks and snapshots | `gcloud compute disks …` / `gcloud compute snapshots …` |
| VPC, subnets, firewall | `gcloud compute networks …`, `gcloud compute firewall-rules …` |
| GKE clusters | `gcloud container clusters …` |
| Kubectl on GKE | `kubectl …` (after `gcloud container clusters get-credentials`) |
| Cloud Run | `gcloud run services …` / `gcloud run jobs …` |
| Cloud Functions (legacy) / Run functions | `gcloud functions …` (gen2 is now `gcloud run`) |
| Cloud Storage | `gcloud storage …` (modern; replaces `gsutil`) |
| BigQuery | `bq …` (separate CLI) |
| Cloud SQL | `gcloud sql instances …`, `gcloud sql databases …`, `gcloud sql users …` |
| Pub/Sub | `gcloud pubsub topics …`, `gcloud pubsub subscriptions …` |
| IAM bindings | `gcloud projects add-iam-policy-binding …`, `gcloud iam service-accounts …` |
| Org policies | `gcloud org-policies set-policy …` (or `gcloud resource-manager org-policies …` legacy) |
| Logging | `gcloud logging logs list`, `gcloud logging read …`, `gcloud logging sinks …` |
| Monitoring | Console-first; `gcloud alpha monitoring …` for some |
| Project mgmt | `gcloud projects create/list/delete/undelete …` |
| Billing | `gcloud billing accounts …`, `gcloud billing projects …` |
| Services / APIs | `gcloud services enable/list/disable …` |

---

## Key flags you'll see again and again

These flags repeat across many commands. If you see them in a stem, they're hints:

- `--project=PROJECT_ID` — explicit project (often needed in script-style questions).
- `--region=REGION` / `--zone=ZONE` — most resources are zonal or regional.
- `--machine-type=…` — VM machine type.
- `--image-family=…` / `--image-project=…` — boot image.
- `--service-account=…` — attach a user-managed SA.
- `--scopes=…` — VM access scopes (mostly legacy; prefer IAM roles on the SA).
- `--tags=…` / `--network-tags=…` — for firewall targeting.
- `--no-address` — VM with no external IP.
- `--metadata=enable-oslogin=TRUE` — turn on OS Login.
- `--tunnel-through-iap` — SSH via IAP.
- `--impersonate-service-account=…` — run as another SA (short-lived token).
- `--maintenance-policy=MIGRATE|TERMINATE` — live migration vs. force shutdown.
- `--preemptible` / `--provisioning-model=SPOT` — discount VMs.
- `--member=…` and `--role=…` — IAM bindings.
- `--condition=…` — IAM Condition (CEL).

---

## What to skip (you have limited time)

The exam is broad. **Do not** spend hours on:

- Deep Spanner internals (interleaved tables, change streams beyond awareness).
- Apigee internals.
- DataFlex / specific Dataflow window semantics.
- Anthos service mesh details (general awareness only).
- Filestore performance tiers.
- All Cloud SDK component flags.

**Do** spend time on:

- IAM (every domain bleeds into it).
- Service accounts (the #1 trap area).
- Picking the right compute service for a workload.
- Picking the right database for a workload.
- Networking primitives (VPC, firewall, peering, Shared VPC, Cloud NAT, Private Google Access).
- Logging routes and sinks.
- Storage classes, lifecycle, signed URLs, UBLA.

---

## Question-format patterns to watch for

### "What's the *cheapest* way to…"
Pick the **most-managed** option that meets the requirement. Then look for storage class downgrade, Spot VMs, sustained-use discounts, regional vs multi-region, Standard tier networking.

### "What's the *least-privileged* role…"
Predefined > basic. Custom > predefined when no predefined fits exactly. Never `roles/owner` or `roles/editor`.

### "Without an external IP…"
- Egress to internet → **Cloud NAT**.
- Reach Google APIs → **Private Google Access**.
- SSH from laptop → **IAP TCP forwarding**.
- Internal LB target → already fine.

### "No data loss…"
- Streaming Dataflow upgrade → **drain**, not cancel.
- Cloud SQL recovery → **PITR with binary logs/WAL**.
- Disk that survives zone failure → **Regional PD**.

### "Globally distributed…"
- SQL with strong consistency → **Spanner**.
- Object store with global reads → **multi-region GCS**.
- Public-facing HTTP frontend → **Global external Application LB + Premium tier**.

### "Compliance / archive / 7+ years…"
- **Archive** storage class.
- **Bucket lock** + retention policy.
- **Data Access audit logs**.

### "Time-limited access…"
- IAM Condition with `request.time < timestamp("…")`.

### "Without keys…"
- GKE → **Workload Identity**.
- External (AWS, GitHub Actions, on-prem) → **Workload Identity Federation**.
- Humans → **impersonation** with `serviceAccountTokenCreator`.

---

## Time management on exam day

You have **2 hours for ~50–60 questions**. That's roughly 2 minutes each.

**Recommended pacing:**

1. **First pass (60–70 min):** Answer easy questions; flag hard ones; keep moving.
2. **Second pass (30–40 min):** Tackle flagged questions. By now, easier ones earlier may have jogged your memory.
3. **Last 10–15 min:** Re-read flagged answers carefully. Don't change answers based on second-guessing — only if you spot a clear mistake.

If a question takes >3 minutes, **flag and skip**. Diminishing returns set in fast.

---

## On exam day

- Bring two forms of ID (online proctoring requires this).
- Test the webcam and mic well in advance.
- Clear your desk completely. Online proctors flag *anything* in view.
- Keep water in a clear container (no labels).
- Restroom breaks: depend on the proctor — usually no during the exam.
- Read every option before picking. Google often plants a "looks right but…" first option to trip fast readers.

---

## Recap

- **Recognition over recall** for gcloud — recognize the right command in a list.
- **Map workloads to products** — that's what most questions test.
- **IAM and SAs** are the heaviest topics — invest accordingly.
- **Common phrases** map to common right answers (cheapest, least privileged, no data loss, etc.).
- **Pace yourself** — flag and skip rather than getting stuck.

Good luck — and remember: the exam tests breadth, not depth. Trust your prep.
