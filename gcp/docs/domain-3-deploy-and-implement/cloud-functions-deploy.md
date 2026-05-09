# Cloud Functions Deployment: Triggers, Runtime, Environment Variables

## Overview

**Cloud Functions** is GCP's serverless function-as-a-service (FaaS) offering. Deployment involves choosing between generation 1 and generation 2, selecting the appropriate trigger type, configuring the runtime environment, and setting security and scaling parameters.

---

## Key Concepts

### Generation 1 vs Generation 2

| Feature | Gen 1 | Gen 2 |
|---------|-------|-------|
| Infrastructure | Custom | Built on Cloud Run |
| Max timeout | 540 seconds (9 min) | 3600 seconds (60 min) |
| Concurrency | 1 request/instance | Multiple (up to 1000) |
| Max instance count | 3000 | 3000 |
| Max memory | 8 GB | 32 GB |
| Max vCPUs | 2 vCPU | Up to 4 vCPU |
| Min instances | Not supported | Supported |
| VPC access | VPC Connector | VPC Connector or Direct VPC |
| Event triggers | Native GCF triggers | Eventarc |
| Traffic splitting | No | Yes (via Cloud Run revisions) |
| Preferred for new functions | No | Yes |

**Gen 2 is the current standard.** For new functions, always use gen2 unless you have a specific reason to use gen1.

---

### Trigger Types

#### HTTP Triggers

- Function is invoked via an HTTPS endpoint
- GCP provides an HTTPS URL automatically
- Authentication:
  - `--allow-unauthenticated`: Public access
  - Default: Requires authentication (caller needs `roles/cloudfunctions.invoker`)
- HTTP method: Any (GET, POST, etc.) — function receives the raw request object
- Use for: Webhooks, simple APIs, CI/CD hooks

#### Event Triggers

**Gen 1 Triggers:**
- `google.storage.object.finalize` — file created/overwritten in GCS
- `google.storage.object.delete` — file deleted
- `google.storage.object.archive` — file archived
- `google.storage.object.metadataUpdate` — metadata changed
- `google.pubsub.topic.publish` — Pub/Sub message published
- Firestore: create, update, delete, write events
- Firebase: Auth, Realtime Database events

**Gen 2 Triggers (via Eventarc):**
- Eventarc can trigger on: **Cloud Storage**, **Pub/Sub**, **Cloud Audit Logs** (for 90+ GCP services), **custom HTTP events**
- Audit log triggers: Any GCP API event can trigger a function (e.g., "when a VM is created, run this function")
- More flexible and extensible than gen1 triggers

---

### Runtime Environments

Supported runtimes (gen2):
- Node.js 16, 18, 20, 22
- Python 3.9, 3.10, 3.11, 3.12
- Go 1.20, 1.21, 1.22
- Java 11, 17, 21
- .NET 6.0, 8.0
- Ruby 3.0, 3.2
- PHP 8.1, 8.2

- You must specify the exact runtime at deployment (e.g., `--runtime=python312`)
- Runtime determines the execution environment and available system packages
- For arbitrary runtimes or system dependencies: Use Cloud Run instead

---

### Configuration Parameters

| Parameter | Description |
|-----------|-------------|
| **Region** | Where the function runs; affects latency and data residency |
| **Memory** | 128 MB to 32 GB (gen2); affects cost and available CPU |
| **CPU** | Auto-allocated based on memory (gen1); explicit in gen2 |
| **Timeout** | Up to 540s (gen1) or 3600s (gen2) |
| **Min instances** | Keep warm; eliminates cold starts (gen2 only) |
| **Max instances** | Caps concurrent invocations |
| **Concurrency** | Requests per instance (gen2 only; default 1 for gen1) |
| **Environment variables** | Key-value configuration (visible in console — not for secrets) |
| **Service account** | Identity for the function's GCP API calls |
| **VPC connector** | Access to VPC resources |

#### CPU Allocation in Gen1

- Gen1 CPU is proportional to memory:
  - 128 MB → 200 MHz
  - 256 MB → 400 MHz
  - 512 MB → 800 MHz
  - ...up to 4 GHz for 4 GB
- This is a commonly tested nuance: CPU is not directly configurable in gen1

---

### Environment Variables

- Set at deployment time; accessible as standard environment variables in the function
- **NOT suitable for secrets** — visible in Cloud Console and to anyone with access to the function
- For secrets: Use **Secret Manager** — reference secrets by ID and version; inject at startup or fetch at runtime
- Automatic environment variables set by GCP:
  - `K_SERVICE`: Function name
  - `K_REVISION`: Revision name (gen2)
  - `FUNCTION_NAME`: Function name (gen1)
  - `FUNCTION_REGION`: Region
  - `PORT`: Port to listen on (HTTP trigger)

---

### Service Account and Permissions

- Each Cloud Function runs with a GCP service account identity
- Default: App Engine default service account (`PROJECT_ID@appspot.gserviceaccount.com`) for gen1; new functions in gen2 default to the Compute Engine default SA
- Best practice: Create a dedicated, least-privilege SA for each function
- Grant the SA only the permissions needed for the function's specific work
- Caller permissions (invoking the function) are separate from the function's own SA permissions

---

### Cold Starts

- When a function instance hasn't been invoked recently, GCP recycles it
- Next invocation starts a new instance = **cold start** (seconds of latency)
- **Minimize cold starts:**
  - Move initialization code outside the function handler (runs once per instance, not per invocation)
  - Use min instances (gen2)
  - Keep dependencies lean
  - Use lightweight runtimes (Go, Node.js tend to be faster than Java/.NET)

---

### Function Versioning and Traffic Management (Gen2)

- Gen2 functions are backed by Cloud Run revisions
- Deploy a new version → new revision created
- Can use traffic splitting to gradually migrate to new version (canary)
- Rollback = redirect traffic to a previous revision

---

### Pub/Sub Integration Pattern

A very common pattern in GCP architecture:
1. Event occurs (file uploaded to GCS, user action in Firestore)
2. Event published to Pub/Sub topic
3. Cloud Function triggered by Pub/Sub subscription
4. Function processes event and writes to storage/database

- **Exactly-once delivery caveat**: Pub/Sub guarantees at-least-once delivery, so functions must be **idempotent** (safe to run multiple times for the same message)
- Failure handling: If function returns an error (non-2xx HTTP response), Pub/Sub retries with exponential backoff

---

## When to Use

- Short-duration event handlers (file processing, data transformation on upload)
- Webhooks and simple HTTP endpoints where code complexity is low
- Glue code between GCP services (e.g., GCS → Bigtable ETL trigger)
- Scheduled tasks via Cloud Scheduler + Pub/Sub + Function
- Responding to Audit Log events for automated governance
- When you want code-only deployment (no Docker required)

---

## When NOT to Use

- Long-running tasks exceeding 9 minutes (gen1) or 60 minutes (gen2) — use Compute Engine, GKE, or Cloud Run Jobs
- Complex applications with many dependencies — use Cloud Run
- Workloads needing persistent connections (e.g., long-lived WebSocket servers) — use Cloud Run
- When GPU access is needed — use Compute Engine or GKE
- When full Kubernetes orchestration is needed — use GKE

---

## Related Services / Concepts

- **Cloud Run/Functions Planning**: Selection decision — see [cloud-run-functions-planning.md](../domain-2-plan-and-configure/cloud-run-functions-planning.md)
- **Cloud Run Deploy**: For containerized event-driven functions — see [cloud-run-deploy.md](cloud-run-deploy.md)
- **Data Security**: Secret Manager integration — see [data-security.md](../domain-5-configure-access-and-security/data-security.md)
- **Monitoring**: Cloud Functions metrics and logs — see [monitoring-cloud-ops.md](../domain-4-ensure-success/monitoring-cloud-ops.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Gen1 CPU is proportional to memory**: You can't independently set CPU in gen1. More memory = more CPU. The exam may ask how to give more CPU to a gen1 function — answer is increase memory.

2. **Gen2 uses Eventarc for events**: Gen1 has built-in triggers. Gen2 uses Eventarc (more flexible, more services). Know this distinction.

3. **Environment variables are not secrets**: Setting a database password as an env var is a poor practice question. Correct answer is Secret Manager.

4. **Pub/Sub functions must be idempotent**: Pub/Sub delivers at-least-once. If your function isn't idempotent, duplicate processing can cause data issues.

5. **Default SA has broad permissions**: App Engine default SA has `Editor` role. Functions using this can access anything in the project. Create dedicated SAs.

6. **Cold start language differences**: Java and .NET have longer cold starts than Go, Node.js, or Python due to JVM/CLR startup time. This matters for latency-sensitive functions.

7. **Gen2 min instances eliminate cold starts**: Gen2 supports `--min-instances` flag to keep warm instances. Gen1 does not support this.

8. **Function timeout vs total execution**: The timeout is per-invocation. A function invoked 100 times doesn't have a combined timeout — each invocation independently has the configured timeout.

### Keywords
- Gen1 vs gen2, Eventarc, HTTP trigger, event trigger, Pub/Sub trigger, environment variables, Secret Manager, cold start, min instances, concurrency, runtime, service account, idempotent

---

## Source

- https://cloud.google.com/functions/docs/concepts/overview
- https://cloud.google.com/functions/docs/2nd-gen/overview
- https://cloud.google.com/eventarc/docs/overview
- https://cloud.google.com/functions/docs/configuring/env-var
