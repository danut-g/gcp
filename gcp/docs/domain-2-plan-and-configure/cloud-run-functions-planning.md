# Cloud Run vs Cloud Functions vs App Engine: Decision Framework

## Overview

GCP offers multiple serverless and PaaS compute options. Choosing correctly between **Cloud Run**, **Cloud Functions**, **App Engine Standard**, and **App Engine Flexible** is a key skill for the ACE exam. The decision depends on workload characteristics, traffic patterns, language requirements, and operational preferences.

---

## Key Concepts

### Cloud Run

- Fully managed serverless platform for **containerized applications**
- Runs any language, runtime, or binary packaged in a container image
- Scales to zero (no traffic = no cost, after a short delay)
- Scales from zero to thousands of instances in seconds
- Billing: per 100ms of request processing + CPU/memory allocation
- Concurrency: Each instance handles **multiple requests simultaneously** (default 80, configurable up to 1000)
- Request timeout: Up to 3600 seconds (1 hour) for HTTP requests
- Execution: Always triggered by **HTTP requests** (or Pub/Sub push, Cloud Tasks, etc.)
- Stateless: Each request is independent; don't rely on in-memory state across requests

#### Cloud Run Key Features

- **Revisions**: Each deployment creates a new revision; traffic can be split across revisions
- **Traffic splitting**: Route a percentage of traffic to different revisions (canary, blue/green)
- **Minimum instances**: Set a floor to avoid cold starts (costs money even with no traffic)
- **CPU allocation**: Default is CPU only allocated during requests; can enable "always-on CPU" for background tasks
- **VPC connector/Direct VPC egress**: For accessing resources in a VPC (e.g., Cloud SQL via private IP)
- **Ingress controls**: Allow all traffic, internal-only, or internal + load balancer
- **Jobs**: Cloud Run Jobs for batch/one-off tasks that don't serve HTTP traffic (run to completion)

---

### Cloud Functions

- Event-driven serverless functions — write individual functions, not full applications
- **Generation 1 (gen1)**: Original Cloud Functions; per-function scaling, 540s timeout max
- **Generation 2 (gen2)**: Built on Cloud Run; supports longer timeouts (3600s), concurrency, larger instances, better networking
- Triggers:
  - **HTTP triggers**: Direct HTTP invocations
  - **Event triggers**: Responds to events from GCP services via Eventarc (gen2) or Cloud Functions triggers (gen1)
    - Cloud Storage events (object created/deleted)
    - Pub/Sub messages
    - Firestore document changes
    - Cloud Scheduler (via Pub/Sub)
    - Firebase events
    - Audit log events
- Languages: Node.js, Python, Go, Java, .NET, Ruby, PHP
- Scales to zero; billing per invocation + duration
- Gen2 supports **multiple concurrent requests** per instance (like Cloud Run)
- Gen1: one request per instance (no concurrency)

---

### App Engine Standard

- PaaS for web applications with Google-managed runtime environments
- Supported runtimes: Python, Java, Node.js, Go, Ruby, PHP (specific versions only)
- Scales to **zero** (no traffic = zero instances = no cost beyond storage)
- Scale-up is very fast (sub-second warm-up for many runtimes)
- Request timeout: 60 seconds (hard limit for HTTP requests)
- **Constraints**: Cannot use arbitrary binaries; must use approved runtimes and libraries
- Uses **sandboxed environment** — limited system calls; no local file writes (except /tmp)
- Traffic splitting between versions: supported
- Background tasks: supports Cron jobs, Task Queues (Cloud Tasks)
- Billing: per instance-hour (F1–F4 class instances)

---

### App Engine Flexible

- PaaS with Docker containers in VMs (Compute Engine under the hood)
- Supports **any runtime** via custom Docker images
- Does **NOT scale to zero** — minimum 1 instance always running
- Request timeout: Up to 60 minutes
- Slower scaling than Standard (minutes, not seconds)
- Access to Compute Engine features: disk, higher CPU/memory
- Billing: per vCPU/hour + memory/hour (charged for underlying VMs)
- Does **not** receive Sustained Use Discounts

---

## Comparison Matrix

| Feature | Cloud Run | Cloud Functions (gen2) | App Engine Standard | App Engine Flexible |
|---------|-----------|----------------------|--------------------|--------------------|
| Container-based | Yes (required) | No (code only) | No (managed runtime) | Yes (optional) |
| Scale to zero | Yes | Yes | Yes | **No** |
| Languages | Any (containerized) | Limited (specific runtimes) | Limited (specific versions) | Any (Docker) |
| Concurrency | Multi (up to 1000) | Multi (gen2) / Single (gen1) | Single request/instance (Standard) | Multi |
| Max timeout | 3600s | 3600s (gen2) | **60s** | 60 min |
| Trigger types | HTTP + Pub/Sub push | HTTP + Events + Pub/Sub | HTTP + Cron | HTTP |
| Cold start | Seconds | Milliseconds–seconds | Very fast (Python/Go) | Slow (minutes) |
| VPC access | Direct VPC egress / Connector | VPC Connector | Limited | Full |
| State management | Stateless | Stateless | Stateless (Standard) | Can use disk |
| Minimum pricing | Scale to zero | Scale to zero | Scale to zero | Always-on minimum |
| Best for | Containerized HTTP services | Event-driven, short functions | Simple web apps, low traffic | Legacy apps, custom runtimes |

---

## When to Use

**Cloud Run:**
- Containerized HTTP services, APIs, microservices
- Workloads needing arbitrary runtimes/libraries
- Services that should scale to zero but also handle bursty traffic
- When you want to bring your own Dockerfile
- Long-running request handling (> 60 seconds)
- Multi-step workflows triggered by HTTP/Pub/Sub

**Cloud Functions:**
- Simple event-driven automation: "when a file is uploaded to GCS, process it"
- Short-lived functions responding to Pub/Sub, Firestore, or storage events
- Webhooks and lightweight HTTP endpoints
- When you want to deploy code only (no Docker knowledge needed)
- Note: Gen2 is preferred for new functions

**App Engine Standard:**
- Existing App Engine applications
- Simple web applications with predictable traffic and fast scale-to-zero requirements
- Applications using App Engine-specific APIs (Datastore, Memcache — though these are now standalone services)
- When team is familiar with the platform and migration isn't justified

**App Engine Flexible:**
- Applications that need custom runtimes but want managed infrastructure
- Long-running request handling beyond App Engine Standard's 60s limit
- When migrating legacy App Engine Standard apps that need library freedom

---

## When NOT to Use

- **Cloud Run**: Not for stateful workloads requiring persistent local disk (use Compute Engine or attach Cloud SQL/Firestore); not for workloads that need to run continuously without HTTP triggers (use Compute Engine or GKE)
- **Cloud Functions**: Not for complex applications with many dependencies; not for tasks requiring > 3600s; not for stateful processing without external storage
- **App Engine Standard**: Not when custom binaries or system packages are needed; not when request latency > 60s
- **App Engine Flexible**: Not when cost efficiency is critical (always pays for minimum VMs); not for new greenfield projects (prefer Cloud Run instead)

---

## Related Services / Concepts

- **GKE**: When containers need cluster-level orchestration, state, or daemon processes — see [gke-planning.md](gke-planning.md)
- **Cloud Run Deploy**: Deployment specifics — see [cloud-run-deploy.md](../domain-3-deploy-and-implement/cloud-run-deploy.md)
- **Cloud Functions Deploy**: Triggers, environments — see [cloud-functions-deploy.md](../domain-3-deploy-and-implement/cloud-functions-deploy.md)
- **App Engine Deploy**: Versions, traffic splitting — see [app-engine-deploy.md](../domain-3-deploy-and-implement/app-engine-deploy.md)
- **Pub/Sub**: Primary event source for Cloud Functions and Cloud Run — see [networking-deploy.md](../domain-3-deploy-and-implement/networking-deploy.md)

---

## Exam-Relevant Notes

### Common Traps

1. **App Engine Standard 60-second timeout**: Hard limit. If a question mentions a workload needing > 60s responses, App Engine Standard is wrong.

2. **App Engine Flexible doesn't scale to zero**: Always has at least one instance running. If cost optimization to zero is required, Flexible is wrong.

3. **Cloud Functions gen1 is single-concurrent**: Each gen1 instance handles one request at a time. Gen2 supports concurrency. The exam may distinguish between them.

4. **Cloud Run requires a container**: If the team only has application code (no Docker knowledge), Cloud Functions or App Engine may be the better answer.

5. **Cloud Run Jobs vs Cloud Run Services**: Jobs are for batch/one-off tasks. Services handle HTTP traffic. The exam distinguishes between them.

6. **App Engine per-project limit**: Each GCP project can have **one** App Engine application per region, and you **cannot change the region** after creation. This is permanent.

7. **Cloud Functions vs Cloud Run for HTTP**: Both handle HTTP. Cloud Run is better for multi-dependency containerized apps; Cloud Functions is better for lightweight single-purpose code.

8. **Minimum instances on Cloud Run**: Setting min-instances=1 prevents cold starts but means you pay continuously. Good for SLA-sensitive services; bad for very low-traffic endpoints.

### Decision Tree: Serverless Selection

```
Does the workload need to respond to HTTP/events?
├── Yes: Is it event-driven with a specific GCP trigger (GCS, Firestore, Pub/Sub)?
│         ├── Yes: Small function with no dependencies? → Cloud Functions
│         └── No: Need container runtime/longer execution? → Cloud Run
│
├── Is it a containerized app/microservice?
│     └── Yes → Cloud Run
│
├── Is it an existing App Engine app?
│     ├── Standard runtime, <60s → App Engine Standard
│     └── Custom runtime, longer tasks → App Engine Flexible (or migrate to Cloud Run)
│
└── Needs continuous background processing?
      └── Consider Compute Engine or GKE
```

### Keywords
- Cloud Run revisions, traffic splitting, concurrency, scale to zero, minimum instances, Cloud Functions gen1 vs gen2, App Engine Standard 60s limit, App Engine Flexible always-on, Eventarc, VPC connector

---

## Source

- https://cloud.google.com/run/docs
- https://cloud.google.com/functions/docs
- https://cloud.google.com/appengine/docs
- https://cloud.google.com/serverless-options
