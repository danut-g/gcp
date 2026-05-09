# 07 — Choosing the Right Serverless Service

## Exam Relevance

Many ACE questions hand you a scenario and ask which service to use. This file is your **decision toolkit**: feature matrix, decision flow, and a list of phrasing keywords that point to each service.

---

## 1. The Big Picture Matrix

| Capability | Cloud Run | Cloud Functions Gen 2 | Cloud Functions Gen 1 | App Engine Std | App Engine Flex |
|---|---|---|---|---|---|
| **Unit of deploy** | Container | Source code (function) | Source code (function) | Source code (app) | Source or container |
| **Languages** | Any | Node, Py, Go, Java, .NET, Ruby, PHP | Same (older versions) | Node, Py, Go, Java, PHP, Ruby | Any (Docker) |
| **Scale to zero** | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Cold starts** | Yes (mitigable) | Yes | Yes | Sub-second | Slow |
| **Max timeout** | 60 min (req) / 24 h (job) | 60 min HTTP / 9 min event | 9 min | 10 min | 60 min |
| **Concurrency / instance** | Up to 1000 | Up to 1000 | 1 | 1 | Configurable |
| **WebSockets / gRPC streaming** | ✅ | Limited | ❌ | ❌ | ✅ |
| **Stateful local disk** | Tmpfs | Tmpfs | Tmpfs | Tmpfs | EFS-style |
| **VPC** | Direct VPC or Connector | Direct VPC or Connector | Connector | Connector | Native |
| **Custom domain** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Free tier** | 2M req/mo | 2M inv/mo | 2M inv/mo | 28 inst-h/day | None |
| **Pricing model** | Per-100ms CPU/mem + req | Per invocation + CPU/mem | Per invocation | Instance-hour | Instance-hour |
| **Best for** | Containerized services | Lightweight event handlers | Legacy gen 1 only | Legacy/known runtimes | Custom OS, long-running |

---

## 2. Decision Flow

```
┌────────────────────────────────────┐
│ Do I have / want a container?      │
└──────────────┬─────────────────────┘
               │
       ┌───────▼────────┐
       │ YES            │ NO
       ▼                ▼
   Cloud Run     Single function reacting to event?
                       ┌────────┴────────┐
                       │ YES             │ NO
                       ▼                 ▼
            Cloud Functions Gen 2    Whole web app?
                                          │
                                  ┌───────┴───────┐
                                  │ YES           │ NO
                                  ▼               ▼
                         App Engine Standard   Reconsider Cloud Run
                         (or Flex if custom OS)
```

---

## 3. Keyword → Service Cheat Sheet

| Phrase in the question | Likely answer |
|---|---|
| "Container", "Docker image" | **Cloud Run** |
| "Stateless service that scales to zero" | Cloud Run |
| "WebSockets", "gRPC streaming" | Cloud Run |
| "Run for up to 24 hours, batch" | **Cloud Run Job** |
| "Lightweight function", "glue code", "single endpoint" | **Cloud Functions Gen 2** |
| "Trigger on object upload" | Cloud Functions Gen 2 (Eventarc) or Cloud Run + Eventarc |
| "Trigger on Pub/Sub message" | Cloud Functions Gen 2 (`--trigger-topic`) or Cloud Run + push sub |
| "React to a Cloud Audit Log event" | Cloud Functions Gen 2 / Cloud Run + Eventarc |
| "Existing App Engine app" | App Engine (keep it) |
| "Custom OS package, but still serverless-ish" | **App Engine Flex** or Cloud Run |
| "Multi-version traffic split inside one app" | App Engine versions or Cloud Run revisions |
| "Cron job" | Cloud Scheduler → Cloud Run/Function (or `cron.yaml` on App Engine Std) |
| "Run a script once per night" | Cloud Scheduler + Cloud Run / Function |

---

## 4. Cost Considerations

- **Cloud Run** and **Cloud Functions** are **request-billed** — best for spiky / occasional traffic
- **App Engine Standard** has a **generous always-free tier**; good for very low-volume workloads
- **App Engine Flex** is **VM-billed** — most expensive, no scale-to-zero
- For **always-on heavy compute**, GCE / GKE may be cheaper

---

## 5. Common Anti-Patterns (Things to Avoid)

| Don't... | Because... |
|---|---|
| Use App Engine Flex when scale-to-zero is required | Flex has min 1 instance |
| Use Cloud Functions for a 5-language polyglot app | Use Cloud Run; Functions = single function |
| Use Cloud Run for a single 20-line event handler | Functions Gen 2 is simpler / faster to deploy |
| Use Cloud Functions Gen 1 for new code | Gen 2 supersedes it |
| Wait on long-running tasks in a Cloud Run **service** | Use a Cloud Run **job** instead |
| Connect to Cloud SQL **public IP** from Cloud Run | Use Cloud SQL **Auth Proxy** + private IP |

---

## 6. Multi-Service Pattern (Real World)

A typical event-driven serverless app on GCP:

```
                     ┌──────────────────────┐
   Public users ────▶│  Cloud Run (web/API) │
                     └─────────┬────────────┘
                               │ writes
                               ▼
                     ┌──────────────────────┐
                     │   Cloud Storage      │
                     └─────────┬────────────┘
                               │ object.finalized
                               ▼
                     ┌──────────────────────┐
                     │   Eventarc trigger   │
                     └─────────┬────────────┘
                               ▼
                     ┌──────────────────────┐
                     │ Functions Gen 2      │  ← processes upload
                     └─────────┬────────────┘
                               │ publishes
                               ▼
                     ┌──────────────────────┐
                     │  Pub/Sub topic       │
                     └─────────┬────────────┘
                               │ push subscription
                               ▼
                     ┌──────────────────────┐
                     │  Cloud Run worker    │  ← long-running consumer
                     └──────────────────────┘
```

---

## 7. Sources

- [Choosing a serverless option](https://cloud.google.com/serverless-options)
- [Cloud Run vs Cloud Functions](https://cloud.google.com/blog/topics/developers-practitioners/cloud-run-vs-cloud-functions-which-one)
- [Eventarc supported sources](https://cloud.google.com/eventarc/docs/reference/supported-events)
