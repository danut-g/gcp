# 01 — Cloud Run: Deploying Containerized Applications

## Exam Relevance

You must be able to deploy a container to Cloud Run, configure traffic, manage revisions, control access, and connect to other GCP services. **Cloud Run questions appear repeatedly on the ACE exam.**

---

## 1. What Is Cloud Run?

- **Fully managed serverless platform** for running stateless **containers**
- Scales from **0** to thousands of instances automatically
- Pay per **request + CPU/memory used** (rounded to nearest 100 ms)
- HTTPS endpoint provisioned automatically (`*.run.app` domain)
- Supports **HTTP/1, HTTP/2, gRPC, WebSockets**

### Two flavors
| | Cloud Run (managed) | Cloud Run on GKE (Anthos) |
|---|---|---|
| Infrastructure | Google-managed | Your own GKE cluster |
| Use case | Default — fully serverless | Hybrid / on-prem requirements |

> **Exam default:** unless told otherwise, "Cloud Run" = fully managed.

---

## 2. The Container Contract

Your container must:
1. Listen on the **port** defined by the `PORT` environment variable (default: `8080`)
2. Listen on **`0.0.0.0`** (not `127.0.0.1`)
3. Start within **4 minutes** (otherwise the deployment fails)
4. Be **stateless** — local disk is ephemeral; no file persists between requests

---

## 3. Deploying a Service

### From a container image (Artifact Registry)

```bash
gcloud run deploy my-service \
  --image=us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-app:v1 \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --port=8080 \
  --memory=512Mi \
  --cpu=1 \
  --min-instances=0 \
  --max-instances=100 \
  --timeout=300 \
  --concurrency=80 \
  --set-env-vars="ENV=prod,DB_HOST=10.0.0.5" \
  --service-account=my-sa@PROJECT_ID.iam.gserviceaccount.com
```

### From source (Cloud Build builds the image)

```bash
gcloud run deploy my-service \
  --source=. \
  --region=us-central1 \
  --allow-unauthenticated
```
Cloud Build automatically uses **Buildpacks** if there is no `Dockerfile`.

### Console path
**Cloud Run → Create Service → Deploy one revision from existing container image / source repo**

---

## 4. Key Deployment Parameters

| Parameter | Description | Default |
|---|---|---|
| `--image` | Container image URL | required |
| `--region` | Region | required |
| `--port` | Port your container listens on | 8080 |
| `--memory` | Memory per instance | 512Mi |
| `--cpu` | vCPU count (1, 2, 4, 6, 8) | 1 |
| `--min-instances` | Always-warm instances (avoids cold start) | 0 |
| `--max-instances` | Cap on concurrent instances | 100 |
| `--concurrency` | Max simultaneous requests per instance | 80 |
| `--timeout` | Max request duration in seconds | 300 (max 3600) |
| `--allow-unauthenticated` | Public access | requires auth by default |
| `--ingress` | `all` / `internal` / `internal-and-cloud-load-balancing` | `all` |
| `--service-account` | Identity used by the service | Default Compute SA |
| `--set-env-vars` | Plain env vars | none |
| `--set-secrets` | Mount secrets from Secret Manager | none |
| `--vpc-connector` | Serverless VPC Access connector | none |
| `--vpc-egress` | `all-traffic` / `private-ranges-only` | private-ranges-only |
| `--cpu-boost` | Boost CPU during cold start | off |

---

## 5. Revisions & Traffic Management

Every deploy creates a new **revision** (immutable). Revisions stay around so you can roll back or split traffic.

```bash
# List revisions
gcloud run revisions list --service=my-service --region=us-central1

# Send 100% to a specific revision
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00005-abc=100

# Canary: 90/10 split between revisions
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00004-xyz=90,my-service-00005-abc=10

# Roll back
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00004-xyz=100
```

You can also tag revisions and route to them by URL:

```bash
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --set-tags=preview=my-service-00005-abc
# Now reachable at: https://preview---my-service-xxxxx-uc.a.run.app
```

---

## 6. Authentication & Access Control

| Mode | How |
|---|---|
| Public | `--allow-unauthenticated` (grants `allUsers` the `run.invoker` role) |
| Authenticated callers only | `--no-allow-unauthenticated` (default) |
| Specific user / SA | `gcloud run services add-iam-policy-binding ... --role=roles/run.invoker` |

To call an authenticated service:
```bash
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  https://my-service-xxxxx-uc.a.run.app
```

**Key IAM roles:**
- `roles/run.invoker` — call the service
- `roles/run.developer` — deploy & manage
- `roles/run.admin` — full control

---

## 7. Scaling Behavior

- **`min-instances=0`** → scales to zero, cheapest, but **cold starts** on first request
- **`min-instances=1+`** → no cold start, you pay even when idle
- **`max-instances`** → protects downstream systems from being overwhelmed
- **`concurrency`** → high concurrency (e.g., 80) keeps fewer instances; concurrency=1 mimics traditional one-request-per-VM model

> **Exam tip:** When asked how to **eliminate cold starts**, the answer is `--min-instances >= 1`.

---

## 8. Networking

### VPC egress (calling private resources)
- **Direct VPC egress** (newer, recommended): `--network` + `--subnet`
- **Serverless VPC Access connector**: `--vpc-connector`

```bash
gcloud run deploy my-service \
  --image=... \
  --vpc-connector=my-connector \
  --vpc-egress=all-traffic
```

### Ingress
- `all` — open internet
- `internal` — only VPC + projects in same VPC SC perimeter
- `internal-and-cloud-load-balancing` — only via External HTTPS LB (used to attach Cloud Armor / IAP)

---

## 9. Secrets & Config

```bash
# Mount Secret Manager secret as env var
gcloud run deploy my-service \
  --image=... \
  --set-secrets=DB_PASSWORD=db-password:latest

# Or as a file
gcloud run deploy my-service \
  --image=... \
  --set-secrets=/etc/secrets/db=db-password:latest
```

The service's runtime SA must have `roles/secretmanager.secretAccessor`.

---

## 10. Triggering Cloud Run from Events

Cloud Run is the **target** for many event sources. Three common patterns:

### a) Pub/Sub push subscription
```bash
gcloud pubsub subscriptions create my-sub \
  --topic=my-topic \
  --push-endpoint=https://my-service-xxxxx-uc.a.run.app/ \
  --push-auth-service-account=invoker-sa@PROJECT_ID.iam.gserviceaccount.com
```
The push endpoint receives an HTTP POST with the message in JSON.

### b) Eventarc trigger (recommended)
See `06-eventarc.md`. Lets you wire any GCP event source.

### c) Cloud Scheduler (cron)
```bash
gcloud scheduler jobs create http nightly-job \
  --schedule="0 2 * * *" \
  --uri=https://my-service-xxxxx-uc.a.run.app/run \
  --oidc-service-account-email=scheduler-sa@PROJECT_ID.iam.gserviceaccount.com
```

---

## 11. Cloud Run Jobs (vs. Services)

- **Service** — long-running, request-driven (HTTP)
- **Job** — runs to completion (batch). No HTTP endpoint. Up to **24-hour** runtime per task, parallel tasks supported.

```bash
gcloud run jobs create my-job \
  --image=us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-batch:v1 \
  --region=us-central1 \
  --tasks=10 \
  --parallelism=5 \
  --task-timeout=3600

gcloud run jobs execute my-job --region=us-central1
```

> **Exam tip:** "Run a batch script that finishes in N minutes/hours" → **Cloud Run Job**, not Service.

---

## 12. Common gcloud Operations

```bash
# Update env vars without redeploy
gcloud run services update my-service \
  --region=us-central1 \
  --update-env-vars=NEW_FLAG=true

# Describe service
gcloud run services describe my-service --region=us-central1

# Get URL
gcloud run services describe my-service --region=us-central1 \
  --format="value(status.url)"

# Delete
gcloud run services delete my-service --region=us-central1
```

---

## 13. Exam Traps & Keywords

| If you see... | Answer |
|---|---|
| "fully managed, container, scale to zero" | Cloud Run |
| "eliminate cold starts" | `--min-instances >= 1` |
| "stop public access" | `--no-allow-unauthenticated` + IAM `run.invoker` |
| "canary / split traffic" | revisions + `update-traffic` |
| "private ingress only via internal LB" | `--ingress=internal-and-cloud-load-balancing` |
| "long-running batch task" | Cloud Run **Job**, not Service |
| "Cloud Run can talk to Cloud SQL on private IP" | Use VPC connector / Direct VPC + Cloud SQL Proxy |

---

## 14. Sources

- [Cloud Run overview](https://cloud.google.com/run/docs/overview/what-is-cloud-run)
- [Deploying services](https://cloud.google.com/run/docs/deploying)
- [Managing traffic](https://cloud.google.com/run/docs/managing/traffic)
- [Cloud Run Jobs](https://cloud.google.com/run/docs/create-jobs)
- [Connecting to a VPC](https://cloud.google.com/run/docs/configuring/connecting-vpc)
