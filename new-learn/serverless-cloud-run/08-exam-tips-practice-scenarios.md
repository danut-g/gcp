# 08 — Exam Tips & Practice Scenarios

## How to Use This File

Each scenario below mimics the format of a real ACE exam question. Read the scenario, **answer in your head**, then check the explanation. Focus on the **why**, not just the answer.

---

## Section A — Top 20 Exam Traps

1. **Cloud Run** = container; **Cloud Functions** = source code; **App Engine** = whole app.
2. **App Engine Flex does NOT scale to zero** (min 1 instance).
3. **App Engine region cannot be changed** after `gcloud app create`.
4. **Cloud Functions Gen 1** caps at **9-minute** timeout. Need 60 min HTTP? → **Gen 2**.
5. **Cloud Run Service** for HTTP. **Cloud Run Job** for batch (up to 24 h per task).
6. **`--min-instances >= 1`** is the canonical way to **eliminate cold starts** on Cloud Run / Functions Gen 2.
7. **Cloud Run private** = drop `--allow-unauthenticated`, then grant `roles/run.invoker`.
8. To **only allow internal LB** in front of Cloud Run: `--ingress=internal-and-cloud-load-balancing`.
9. **Pub/Sub push** to Cloud Run requires `roles/run.invoker` on the **push auth SA**.
10. **GCS object.finalized** fires for both **new uploads and overwrites**.
11. **Eventarc** is the recommended trigger source for **Cloud Run** and **Cloud Functions Gen 2**.
12. **`gsutil notification create -t TOPIC -e OBJECT_FINALIZE BUCKET`** wires GCS → Pub/Sub.
13. **Pub/Sub delivery is at-least-once** — handlers must be idempotent.
14. **Pub/Sub ordering** requires **ordering key on publish + `--enable-message-ordering`** on the sub.
15. **Dead-letter topic** keeps poison messages out of your main pipeline (`--max-delivery-attempts`).
16. **Cloud Run revisions** support **traffic splitting** (canary). App Engine has versions + `--migrate`.
17. **Cloud Run secrets** come from **Secret Manager** via `--set-secrets`.
18. **VPC access**: Direct VPC egress (newer) or VPC Connector (older but still common).
19. **Cloud Functions Gen 2 is built on Cloud Run + Eventarc** — same engine, different UX.
20. **App Engine `cron.yaml`** is the only built-in cron; for Cloud Run / Functions, use **Cloud Scheduler**.

---

## Section B — 15 Practice Scenarios

### Scenario 1
> A team is building a Python API that needs to autoscale on demand and stop billing when idle. They have an existing Dockerfile. Which service?

**Answer:** Cloud Run.
**Why:** Container + scale-to-zero + custom code = Cloud Run.

---

### Scenario 2
> Files are uploaded daily into a `gs://invoices` bucket. A Python function should parse each file and load it into BigQuery. Pick the simplest deployment.

**Answer:** Cloud Functions Gen 2 with Eventarc trigger on `google.cloud.storage.object.v1.finalized`.
**Why:** Single function, GCS event source — exactly the Functions sweet spot.

---

### Scenario 3
> A legacy Cloud Functions Gen 1 has 8-minute average runtime and is now timing out (jobs sometimes take 11 min). Minimal-change fix?

**Answer:** Migrate to Cloud Functions Gen 2 (HTTP timeout up to 60 min) **or** to Cloud Run.
**Why:** Gen 1 caps at 9 min; only Gen 2 / Cloud Run support longer timeouts.

---

### Scenario 4
> Pub/Sub messages must be processed in the order they were published, per customer. Choose the configuration.

**Answer:** Publishers set an **ordering key = customer ID**; subscription has `--enable-message-ordering`.
**Why:** Order is preserved per ordering key, per region.

---

### Scenario 5
> A Cloud Run service receives sporadic Pub/Sub events — but every cold start adds 3 s latency. How do you eliminate cold starts?

**Answer:** `gcloud run services update my-service --min-instances=1`.
**Why:** Always-warm instances avoid the cold start; cost is one instance always running.

---

### Scenario 6
> An existing App Engine Standard app has 5 services routed via paths (`/api`, `/admin`, `/default`). How do you configure this?

**Answer:** Deploy a `dispatch.yaml`.
**Why:** It's the only App Engine artifact that maps URL patterns to services across the app.

---

### Scenario 7
> A worker subscribes to a Pub/Sub topic from inside a custom GCE VM cluster — which subscription type?

**Answer:** Pull subscription.
**Why:** Push is for HTTP/serverless targets; pull lets the workers control flow.

---

### Scenario 8
> A team wants events from the GCS bucket `gs://logs` to trigger a Cloud Run service. The current bucket is `EU` multi-region. Where do you create the Eventarc trigger?

**Answer:** In `eu` (or matching region/multi-region of the bucket).
**Why:** Eventarc triggers must match the source's region for GCS direct events.

---

### Scenario 9
> Some Pub/Sub messages crash the consumer over and over. You don't want them blocking the queue.

**Answer:** Configure a **dead-letter topic** with `--max-delivery-attempts=5`.
**Why:** Poison messages are routed off the main subscription after N failures.

---

### Scenario 10
> The CFO wants the lowest possible cost for an internal admin tool used 30 minutes per day by 5 employees.

**Answer:** App Engine Standard (free tier 28 inst-h/day) or Cloud Run with `min-instances=0`.
**Why:** Both scale to zero; Standard's free tier may keep it free.

---

### Scenario 11
> A Cloud Run service must be reachable **only** through an internal HTTP(S) Load Balancer in the VPC.

**Answer:** Set `--ingress=internal-and-cloud-load-balancing` and attach via Serverless NEG.
**Why:** This ingress mode rejects direct internet traffic but accepts LB-bridged requests.

---

### Scenario 12
> A new BigQuery table loads must trigger an audit/notification function. Which trigger?

**Answer:** Eventarc with **Cloud Audit Log** filter (`serviceName=bigquery.googleapis.com`, `methodName=jobservice.jobcompleted`).
**Why:** BigQuery doesn't expose direct events; audit logs are the route.

---

### Scenario 13
> A Cloud Run service must connect to a Cloud SQL PostgreSQL instance on a private IP.

**Answer:** Use **Direct VPC egress** (or Serverless VPC Access connector) + Cloud SQL Auth Proxy or private IP routing.
**Why:** Public IP not allowed → must be in VPC; Cloud Run reaches VPC through one of the two egress mechanisms.

---

### Scenario 14
> The team wants to deploy a new version of a Cloud Run service to 5% of traffic before fully promoting it.

**Answer:**
```bash
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-NEW=5,my-service-OLD=95
```
**Why:** Cloud Run revisions support percentage-based traffic splitting natively.

---

### Scenario 15
> A nightly cron job must run a 45-minute Python data export. Which service?

**Answer:** **Cloud Run Job** triggered by Cloud Scheduler (or Workflows).
**Why:** Cloud Run Job runs to completion (up to 24 h per task); Service is for HTTP requests.

---

## Section C — Required IAM Cheat Sheet

| Goal | Role on |
|---|---|
| Deploy a Cloud Run service | `roles/run.developer` (or `admin`) on project |
| Invoke a private Cloud Run service | `roles/run.invoker` on the service |
| Eventarc trigger SA | `roles/eventarc.eventReceiver` (project) + `roles/run.invoker` (target) |
| GCS → Pub/Sub notifications | GCS service agent: `roles/pubsub.publisher` on topic |
| Pub/Sub push auth | The push auth SA: `roles/run.invoker` on Cloud Run |
| Functions Gen 2 deploy | `roles/cloudfunctions.developer` + `roles/iam.serviceAccountUser` |
| App Engine deploy | `roles/appengine.deployer` + `roles/cloudbuild.builds.builder` |
| Read Secret Manager from Cloud Run | Runtime SA: `roles/secretmanager.secretAccessor` |

---

## Section D — Memorize These Commands

```bash
# Cloud Run — deploy from container
gcloud run deploy NAME --image=IMG --region=R --allow-unauthenticated

# Cloud Run — split traffic
gcloud run services update-traffic NAME \
  --region=R --to-revisions=R1=90,R2=10

# Cloud Functions Gen 2 — Pub/Sub trigger
gcloud functions deploy NAME --gen2 --runtime=python312 \
  --region=R --source=. --entry-point=fn --trigger-topic=TOPIC

# Cloud Functions Gen 2 — GCS Eventarc trigger
gcloud functions deploy NAME --gen2 --runtime=python312 \
  --region=R --source=. --entry-point=fn \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=BUCKET"

# App Engine — deploy
gcloud app deploy app.yaml --version=VX --no-promote
gcloud app services set-traffic SERVICE --splits=VX=0.1,VY=0.9

# Pub/Sub push subscription
gcloud pubsub subscriptions create SUB \
  --topic=T --push-endpoint=URL \
  --push-auth-service-account=SA@PROJECT.iam.gserviceaccount.com

# GCS → Pub/Sub notification
gsutil notification create -t projects/P/topics/T -e OBJECT_FINALIZE gs://BUCKET

# Eventarc — generic trigger
gcloud eventarc triggers create T \
  --location=R --destination-run-service=S --destination-run-region=R \
  --event-filters="type=TYPE" --event-filters="..." \
  --service-account=SA@PROJECT.iam.gserviceaccount.com
```

---

## Section E — Quick Self-Quiz

1. Which is the **simplest** way to eliminate cold starts on Cloud Run?
2. Can App Engine Flex scale to zero?
3. Which Cloud Storage event fires on a new file upload?
4. Are Pub/Sub deliveries exactly-once by default?
5. What roles does an Eventarc trigger SA need?
6. Cloud Run Service or Cloud Run Job for a 4-hour batch task?
7. How do you mount a Secret Manager secret in Cloud Run as a file?
8. What happens to Pub/Sub messages that fail N delivery attempts (with DLT configured)?
9. Which language can you run on Cloud Run that you cannot run on Cloud Functions Gen 2?
10. Cloud Functions Gen 2 is built on top of which two services?

**Answers:**
1. `--min-instances=1` (or higher).
2. No, minimum 1 instance.
3. `google.cloud.storage.object.v1.finalized` (`OBJECT_FINALIZE`).
4. No — at-least-once. Pull subs can opt-in to exactly-once delivery.
5. `roles/eventarc.eventReceiver` (project) + `roles/run.invoker` on target + (sometimes) `roles/iam.serviceAccountTokenCreator`.
6. Cloud Run Job.
7. `--set-secrets=/etc/secrets/db=db-password:latest`.
8. Routed to the dead-letter topic for inspection.
9. Anything not in the supported runtime list (e.g., Rust, Elixir, custom binary). Cloud Run runs any container.
10. Cloud Run + Eventarc.

---

## Section F — Final Reading List

- [Cloud Run docs](https://cloud.google.com/run/docs) — start with **Concepts** + **Deploying**
- [Cloud Functions docs](https://cloud.google.com/functions/docs) — read **Gen 1 vs Gen 2** carefully
- [Eventarc docs](https://cloud.google.com/eventarc/docs) — focus on **supported events**
- [Pub/Sub docs](https://cloud.google.com/pubsub/docs) — push, ordering, DLT
- [GCS notifications](https://cloud.google.com/storage/docs/pubsub-notifications)

> Walk through each scenario in this file twice — once to test, once to teach the answer back to yourself.
