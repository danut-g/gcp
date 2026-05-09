# 00 — Serverless Compute on GCP: Overview

## Exam Relevance

**Section 3.3 — Deploying applications to serverless compute platforms** (GCP Associate Cloud Engineer).
You must know:
- Which serverless options Google Cloud offers and their differences
- When to choose each (Cloud Run vs. Cloud Functions vs. App Engine)
- How event-driven architectures fit on top of these platforms (Pub/Sub, Cloud Storage events, Eventarc)

---

## 1. The Three Pillars of GCP Serverless Compute

| Service | Unit of Deployment | Best For |
|---|---|---|
| **Cloud Run** | Container image | HTTP/gRPC microservices, web apps, APIs, event consumers (any language) |
| **Cloud Functions** | Source code (single function) | Lightweight event handlers, glue code, simple APIs |
| **App Engine** | Source code (whole app) | Traditional web applications with platform-managed runtime |

All three:
- Auto-scale from zero (App Engine Standard and Cloud Run)
- Have built-in HTTPS endpoints
- Integrate with IAM, Cloud Logging, Cloud Monitoring, Cloud Trace
- Support custom domains
- Can be triggered by HTTP requests **or** Google Cloud events (via Eventarc, Pub/Sub, or direct triggers)

---

## 2. Quick Decision Tree

```
Do you have a container?
├── YES → Cloud Run
└── NO
    ├── Single function reacting to events? → Cloud Functions
    ├── Whole web app, multiple endpoints? → App Engine
    └── Need full container control + scale-to-zero? → Build a container, use Cloud Run
```

> **Modern recommendation (2024+):** Google now positions **Cloud Run** as the default serverless platform. **Cloud Functions 2nd gen is built on Cloud Run**, so there is increasing convergence.

---

## 3. Comparison Matrix (Exam-Critical)

| Feature | Cloud Run | Cloud Functions (Gen 2) | App Engine Standard | App Engine Flex |
|---|---|---|---|---|
| **Input** | Container image | Source code | Source code | Source code or container |
| **Languages** | Any (container) | Node, Python, Go, Java, .NET, Ruby, PHP | Node, Python, Go, Java, PHP, Ruby | Any (Docker) |
| **Cold start** | Yes (mitigated by min-instances) | Yes | Very fast | Slower (VM-based) |
| **Scale to zero** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ Min 1 instance |
| **Max request time** | 60 min | 60 min (HTTP) / 9 min (event) | 10 min | 60 min |
| **Stateful** | No | No | No | No |
| **Concurrency / instance** | Up to 1000 | Up to 1000 (Gen 2) | 1 (per dynamic instance) | Configurable |
| **VPC access** | Direct VPC or Connector | Direct VPC or Connector | Connector | Native VPC |
| **Pricing** | Per request + CPU/mem (per 100ms) | Per invocation + CPU/mem | Instance-hours | Instance-hours |
| **Free tier** | 2M requests/month | 2M invocations/month | 28 instance-hours/day | None |

---

## 4. Event-Driven Architecture on GCP Serverless

Three primary ways to trigger serverless workloads from Google Cloud events:

### a) Pub/Sub Push Subscriptions
- Pub/Sub message → HTTPS push to your Cloud Run / Cloud Functions / App Engine endpoint
- Simple, direct, reliable, with retry built-in

### b) Eventarc (the modern, unified router)
- Routes events from **150+ Google Cloud sources** (Cloud Storage, BigQuery, Cloud SQL, Audit Logs, etc.) to Cloud Run, Cloud Functions, GKE, or Workflows
- Uses CloudEvents specification
- Auto-creates Pub/Sub topics and subscriptions under the hood
- **Recommended approach for new event-driven apps**

### c) Cloud Storage Direct Triggers
- Cloud Functions can be directly triggered by GCS object events (legacy 1st gen, or via Eventarc in 2nd gen)
- Gen 2 functions and Cloud Run use **Eventarc** under the hood for GCS events

```
┌──────────────┐
│ Event Source │  (GCS, Pub/Sub, Audit Log, BigQuery, Firestore, ...)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Eventarc   │  ← unified router (CloudEvents)
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Cloud Run / Cloud Functions / GKE   │
└──────────────────────────────────────┘
```

---

## 5. What This Series Covers

| File | Topic |
|---|---|
| `01-cloud-run.md` | Deploy, configure, manage Cloud Run services |
| `02-cloud-functions.md` | Cloud Functions 1st gen vs. 2nd gen, deployment, runtimes |
| `03-app-engine.md` | App Engine Standard and Flexible environments |
| `04-pubsub-event-processing.md` | Triggering serverless from Pub/Sub |
| `05-cloud-storage-events.md` | Reacting to Cloud Storage object change notifications |
| `06-eventarc.md` | Eventarc — unified event routing |
| `07-choosing-serverless-service.md` | Decision matrix and trade-offs |
| `08-exam-tips-practice-scenarios.md` | Exam traps, keywords, scenario walkthroughs |

---

## 6. Exam-Relevant Notes

- **"Container, scale to zero" → Cloud Run.**
- **"Single function, react to event" → Cloud Functions.**
- **"Existing app, just upload code" → App Engine Standard.**
- **"Route an event from any GCP service to compute" → Eventarc.**
- App Engine **Flex does not scale to zero** — minimum 1 instance.
- Cloud Functions Gen 2 is built on **Cloud Run** + **Eventarc** internally.
- Pub/Sub **push subscriptions** are the simplest way to fan out events to HTTP endpoints.
- For **GCS object events** in Gen 2 / Cloud Run → use **Eventarc** trigger.

---

## 7. Sources

- [Cloud Run docs](https://cloud.google.com/run/docs)
- [Cloud Functions docs](https://cloud.google.com/functions/docs)
- [App Engine docs](https://cloud.google.com/appengine/docs)
- [Eventarc docs](https://cloud.google.com/eventarc/docs)
- [Choosing a serverless option](https://cloud.google.com/serverless-options)
