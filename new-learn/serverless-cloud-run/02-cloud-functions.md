# 02 — Cloud Functions: Event-Driven Code Without Containers

## Exam Relevance

You must understand **Cloud Functions 1st gen vs. 2nd gen**, supported triggers, deployment, runtimes, and limits. The exam often asks you to pick the right serverless option for a "small piece of code that reacts to an event."

---

## 1. What Is Cloud Functions?

- **Fully managed FaaS** (Function-as-a-Service)
- You write a single function in a supported language; Google runs it on demand
- Auto-scales from 0
- Two generations co-exist:

| | **1st gen** | **2nd gen** (recommended for new code) |
|---|---|---|
| Built on | Proprietary infra | **Cloud Run + Eventarc** |
| Max timeout | 9 min (HTTP) / 9 min (background) | **60 min HTTP / 9 min event** |
| Concurrency / instance | 1 | Up to 1000 |
| Max instance memory | 8 GB | 32 GB |
| Max CPU | 2 | 8 |
| Event sources | Limited (Pub/Sub, GCS, Firestore, HTTP) | **Any source via Eventarc (150+)** |
| Traffic splitting | ❌ | ✅ (Cloud Run revisions) |
| Local disk | /tmp (in-memory) | /tmp (in-memory) |

> **Default for the exam:** when in doubt, recent docs and exam scenarios assume **Gen 2**.

---

## 2. Supported Runtimes (Gen 2)

- Node.js (18, 20, 22)
- Python (3.10, 3.11, 3.12)
- Go (1.20+)
- Java (11, 17, 21)
- .NET (6, 8)
- Ruby (3.0, 3.2)
- PHP (8.1, 8.2)

---

## 3. Trigger Types

### HTTP trigger
Function exposed at an HTTPS URL.
```bash
gcloud functions deploy my-http-fn \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=hello_world \
  --trigger-http \
  --allow-unauthenticated
```

### Pub/Sub trigger
Triggers on every message published to a topic.
```bash
gcloud functions deploy my-pubsub-fn \
  --gen2 \
  --runtime=nodejs20 \
  --region=us-central1 \
  --source=. \
  --entry-point=processMessage \
  --trigger-topic=my-topic
```

### Cloud Storage trigger (Gen 2 — uses Eventarc internally)
Triggers on object change events.
```bash
gcloud functions deploy my-gcs-fn \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=on_upload \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=my-bucket"
```

### Generic Eventarc trigger (Gen 2)
Any Google Cloud event.
```bash
gcloud functions deploy my-audit-fn \
  --gen2 \
  --runtime=go122 \
  --source=. \
  --entry-point=OnAuditEvent \
  --trigger-event-filters="type=google.cloud.audit.log.v1.written" \
  --trigger-event-filters="serviceName=storage.googleapis.com" \
  --trigger-event-filters="methodName=storage.buckets.delete"
```

---

## 4. Function Signatures

### HTTP function (Python)
```python
import functions_framework

@functions_framework.http
def hello_world(request):
    name = request.args.get("name", "World")
    return f"Hello, {name}!"
```

### Pub/Sub function (Python, Gen 2)
```python
import base64
import functions_framework
from cloudevents.http import CloudEvent

@functions_framework.cloud_event
def process_message(cloud_event: CloudEvent):
    data = cloud_event.data["message"]["data"]
    decoded = base64.b64decode(data).decode("utf-8")
    print(f"Received: {decoded}")
```

### Cloud Storage function (Python, Gen 2)
```python
@functions_framework.cloud_event
def on_upload(cloud_event):
    bucket = cloud_event.data["bucket"]
    name   = cloud_event.data["name"]
    print(f"New file: gs://{bucket}/{name}")
```

> Gen 1 used a different signature `def fn(data, context)` — Gen 2 uses **CloudEvents** uniformly.

---

## 5. Important Configuration Flags

| Flag | Purpose |
|---|---|
| `--gen2` | Deploy as 2nd-gen (omit for 1st gen — but prefer Gen 2) |
| `--runtime` | e.g. `python312`, `nodejs20`, `go122` |
| `--source` | Path or repo URL with code |
| `--entry-point` | Function name in source |
| `--trigger-http` / `--trigger-topic` / `--trigger-event-filters` | One trigger per function |
| `--allow-unauthenticated` | Public HTTP function |
| `--memory` | e.g. `512Mi`, `1Gi`, up to `32Gi` (Gen 2) |
| `--timeout` | Max execution time |
| `--min-instances` / `--max-instances` | Scaling controls |
| `--service-account` | Runtime identity |
| `--set-env-vars` | Plain env vars |
| `--set-secrets` | Secret Manager secrets |
| `--vpc-connector` | Reach private VPC resources |

---

## 6. Authentication & Permissions

- **HTTP functions** are private by default. Add `--allow-unauthenticated` for public.
- For non-HTTP triggers (Pub/Sub, GCS) the service receives events via Eventarc/Pub/Sub push.
- The **runtime service account** (default: `PROJECT_NUMBER-compute@developer.gserviceaccount.com`) needs the appropriate roles to read/write other GCP resources.
- The **trigger SA** (used by Eventarc/Pub/Sub) must have `roles/run.invoker` on the underlying Cloud Run service.

```bash
# Grant a user permission to invoke a private HTTP function
gcloud functions add-invoker-policy-binding my-http-fn \
  --region=us-central1 \
  --member=user:alice@example.com
```

---

## 7. Limits & Quotas (Gen 2)

| Resource | Limit |
|---|---|
| Function memory | 32 GiB |
| CPU | 8 vCPU |
| HTTP timeout | 60 minutes |
| Event timeout | 9 minutes |
| Concurrency / instance | 1000 |
| Max instances | 1000 (default; raisable) |
| Deployment package | 100 MB compressed / 500 MB uncompressed |
| Environment variables | 4 KB total |

---

## 8. Local Development & Testing

```bash
# Functions Framework — run locally
pip install functions-framework
functions-framework --target=hello_world --debug
# → Local URL: http://localhost:8080
```

Use **Cloud Code** (VS Code / IntelliJ extension) for local debug, deploy, log streaming.

---

## 9. Common gcloud Operations

```bash
# List functions
gcloud functions list

# Describe (Gen 2 lives in Cloud Run under the hood)
gcloud functions describe my-fn --gen2 --region=us-central1

# Logs
gcloud functions logs read my-fn --gen2 --region=us-central1 --limit=50

# Update (re-deploy with same name)
gcloud functions deploy my-fn --gen2 --source=. --region=us-central1

# Delete
gcloud functions delete my-fn --gen2 --region=us-central1
```

---

## 10. Cloud Functions vs. Cloud Run (Decision Help)

| Pick Cloud Functions when... | Pick Cloud Run when... |
|---|---|
| You have a single small function | You have a containerized app |
| You want zero packaging effort | You need full control of the runtime |
| Source-only deploy is enough | You need custom system libraries |
| Event handler glue logic | You need WebSockets / streaming gRPC |
| You want managed runtime upgrades | You want to use any language/version |

> Both Gen 2 and Cloud Run scale identically — they are the same engine under the hood.

---

## 11. Exam Traps & Keywords

| If you see... | Answer |
|---|---|
| "Tiny function reacts to message" | Cloud Functions (or Cloud Run if container is required) |
| "Trigger on object upload to GCS" | Cloud Functions (Gen 2) with `google.cloud.storage.object.v1.finalized` filter, or Cloud Run + Eventarc |
| "Need 60-minute HTTP timeout for a function" | **Gen 2** (Gen 1 caps at 9 min) |
| "1000 concurrent requests on one instance" | Gen 2 |
| "Trigger from Cloud Audit Log" | Gen 2 + Eventarc filter |
| "Custom runtime / OS package" | **Cloud Run** (not Functions) |

---

## 12. Sources

- [Cloud Functions overview](https://cloud.google.com/functions/docs/concepts/overview)
- [1st gen vs 2nd gen comparison](https://cloud.google.com/functions/docs/concepts/version-comparison)
- [Eventarc triggers](https://cloud.google.com/functions/docs/calling/eventarc)
- [Function runtimes](https://cloud.google.com/functions/docs/runtime-support)
