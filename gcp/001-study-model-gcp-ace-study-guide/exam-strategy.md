# Exam strategy — how to use this study guide

> Practical advice on what to actually memorize vs. what to read once and recognize. The ACE exam is **multiple-choice** (~50 questions, 2 hours, passing ~80%). It is **not a hands-on lab exam** — you will never type a command from scratch.

---

## Do I need to memorize all the gcloud commands?

**No.** The exam tests **recognition**, not recall. When `gcloud` shows up in a question, it usually shows up four times — one in each answer option — and your job is to pick which command (or which flag) does what the scenario describes.

### What you DO need

1. **The verb pattern** — almost every command is `gcloud <service> <resource> <action>`. Once your eye knows the rhythm, you can rule out 2 of 4 options as nonsense without recalling the exact spelling.

2. **Which CLI for which job** — mixing these up is a frequent trap:

   | CLI | Used for |
   |---|---|
   | `gcloud` | Most GCP services (compute, IAM, networking, run, functions, sql, …) |
   | `gcloud storage` (modern) / `gsutil` (legacy) | Cloud Storage objects and buckets |
   | `bq` | BigQuery datasets, tables, queries, loads |
   | `kubectl` | Kubernetes (Pods, Services, Deployments, …) inside any cluster |
   | `terraform` | IaC plan/apply/destroy |
   | `helm` | Kubernetes app packaging |

3. **A short list of commands that appear by name in questions:**
   - `gcloud compute instances create / start / stop / delete`
   - `gcloud projects add-iam-policy-binding`
   - `gcloud iam service-accounts add-iam-policy-binding`
   - `gcloud container clusters create / get-credentials`
   - `gcloud run deploy` and `gcloud run services update-traffic`
   - `gcloud logging sinks create` and `gcloud logging metrics create`
   - `gcloud compute firewall-rules create`
   - `gcloud sql instances create`
   - `gcloud storage buckets create / cp`
   - `terraform init / plan / apply / destroy`
   - `kubectl get / apply / expose / scale`

4. **Key flags that change behavior** — these are how the exam discriminates between answers:
   - `--impersonate-service-account=...` (run a command as another SA)
   - `--tunnel-through-iap` (SSH a VM with no external IP)
   - `--provisioning-model=SPOT` (Spot VM)
   - `--availability-type=REGIONAL` (Cloud SQL HA)
   - `--enable-private-nodes` (private GKE)
   - `--workload-pool` (GKE Workload Identity)
   - `--include-children` (org-level log sink across all projects)
   - `--service-account=...` (attach an SA to a VM / Cloud Run / function)

### What you can safely skip

- **Exact flag spellings.** The exam won't punish `--region` vs `--regions`.
- **Long compound commands** from the labs (full LB setup, complete MIG provisioning). Understand the *steps*, not the syntax.
- **Config-file schemas** — `policy.yaml`, lifecycle JSON, etc. Recognize the shape, don't memorize keys.
- **Every flag of every command** — the man pages are huge and the exam doesn't go that deep.

---

## How to use the gcloud sections in the lessons

Treat the **§5 gcloud reference** blocks as **reference + reading material**, not flashcards.

1. **First pass** — read them once so the verb pattern feels familiar. Don't try to memorize.
2. **Review pass** — skim them when revisiting a topic. Your eye should land on "ah, this is the command for HA Cloud SQL" without needing to recite it.
3. **Hands-on (optional but high-leverage)** — if you have a sandbox project, run a few labs. Twenty minutes typing one MIG, one IAM binding, one snapshot schedule will lock in the verb pattern far better than re-reading.

The points come from:
- **§2 Core concepts** (the theory)
- **§3 Decision criteria** (which product / pattern fits a scenario)
- **§6 Exam gotchas** (the traps)
- **§6.5 Named scenarios** (the canonical "Practical Scenario" stories)
- **§7 Practice scenarios** (Q&A practice)

The gcloud blocks are there so you can recognize commands when they appear as answer options, and so you can run the labs if you want.

---

## What the exam actually weights

| Domain | Approx weight |
|---|---|
| Section 1 — Setting up a cloud environment | ~20% |
| Section 2 — Planning and configuring resources | ~30% |
| Section 3 — Deploying and implementing | ~25% |
| Section 4 — Ensuring success of operations + access/security | ~20–25% |

Section 2 is the largest single section — and inside it, **compute and storage decisions** dominate. If you have limited time, prioritize:

1. The "which compute service?" decision (`2.1`) and the "which database?" decision (`2.2`).
2. IAM hierarchy + role types (`4.1`) and service accounts (`4.2`).
3. VPC + firewalls + load balancer types (`2.3`).
4. Cloud Logging routing + Cloud Monitoring alerts (`3.4`).

Everything else is secondary points.

---

## Question-format patterns to expect

The exam recycles a few shapes — recognize them:

- **"Which product?"** — given a workload, pick from {Compute Engine / GKE / Cloud Run / Cloud Run functions} or {Cloud SQL / Spanner / Firestore / Bigtable / BigQuery}. Almost always solvable from the **Decision criteria** tables.
- **"Why did this fail?"** — usually IAM (missing role), org policy (blocked action), API not enabled, quota exhausted, or firewall (deny ingress, non-transitive peering). Walk those five before guessing.
- **"Least privilege"** — the answer is almost always *the most specific predefined role*, then *resource-level binding*, then *IAM Condition* if time-bounded. Basic roles (Owner/Editor/Viewer) are almost never the right answer in production scenarios.
- **"Which command?"** — three of four options will have a wrong verb, wrong service, or impossible flag. The verb pattern alone usually eliminates two.
- **"Recommended pattern for X"** — Google has *one* blessed answer per scenario: HA VPN over Classic VPN, Workload Identity over node SA, secure tags / service accounts over network tags, signed URLs over public buckets, GCS remote backend over local Terraform state, predefined roles over basic roles. Memorize these one-liners.

---

## Time management

- 50 questions / 120 minutes ≈ **2.4 minutes per question**. Plenty of time.
- **Flag and move on.** The console lets you mark questions for review and come back. Don't burn 5 minutes on a single hard question — answer your best guess, mark it, return after the easy ones.
- **Read the last sentence first.** Many ACE questions front-load context paragraphs; the actual ask is in the last line ("which is the *most cost-effective* option?", "*least privilege*", "*minimum number of bindings*"). The qualifier changes the answer.

---

## The day before the exam

- Re-read every **§3 Decision criteria** and **§6 Exam gotchas** table — that's where the highest density of points live.
- Skim every **§6.5 Named scenarios** — they are the canonical exam shapes.
- Take the `mock-exam.md` once, time-boxed.
- Don't try to learn anything new in the last 24 hours; consolidate what you have.
