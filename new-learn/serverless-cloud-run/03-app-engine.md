# 03 — App Engine: Standard & Flexible Environments

## Exam Relevance

App Engine is the original GCP serverless platform. Know the **two environments** (Standard vs. Flex), the deployment workflow with `app.yaml`, services & versions, traffic splitting, and how to **dispatch routes**. You don't need to be an expert — but recognize when App Engine is the right answer (legacy apps, fast deploys, multi-version traffic splitting).

---

## 1. The Two Environments

| | **Standard** | **Flexible** |
|---|---|---|
| Runs on | Sandboxed Google-managed instances | Compute Engine VMs (Docker) |
| Languages | Node, Python, Go, Java, PHP, Ruby (specific versions) | **Any** (custom Docker) |
| Cold starts | Milliseconds | Slow (VM boot) |
| Scale to zero | ✅ Yes | ❌ Min 1 instance |
| SSH into instance | ❌ | ✅ |
| Background processes | Limited | Allowed |
| Pricing | Per instance-hour, free tier (28 hrs/day) | Per vCPU/RAM/disk hour |
| Deploy time | Seconds | Minutes |
| Best for | Stateless web apps in supported languages | Apps needing custom OS / long-running threads |

> **Exam shortcut:** "scale to zero on App Engine" → Standard. "Custom binary or OS package" → Flex (or move to Cloud Run).

---

## 2. The `app.yaml` File

Every App Engine deployment is described by `app.yaml`.

### Standard (Python) example
```yaml
runtime: python312
service: default
instance_class: F2

automatic_scaling:
  min_instances: 0
  max_instances: 10
  target_cpu_utilization: 0.65

env_variables:
  ENV: production
  DB_HOST: 10.0.0.5

handlers:
- url: /static
  static_dir: static
- url: /.*
  script: auto
```

### Flexible (custom container) example
```yaml
runtime: custom
env: flex
service: api

manual_scaling:
  instances: 2

resources:
  cpu: 2
  memory_gb: 4
  disk_size_gb: 10
```
Flexible needs a `Dockerfile` alongside the `app.yaml`.

---

## 3. Deploying

### One-time project setup
```bash
gcloud app create --region=us-central
```
> **A project can have only one App Engine application**, in one region. The region cannot be changed after creation.

### Deploy
```bash
# From source directory containing app.yaml
gcloud app deploy

# Specify yaml file or service
gcloud app deploy api/app.yaml --version=v2 --no-promote

# Open in browser
gcloud app browse
```

| Flag | Effect |
|---|---|
| `--version=NAME` | Manually name the version (default: timestamp) |
| `--promote` / `--no-promote` | Send 100% traffic to new version (default: promote) |
| `--stop-previous-version` | Stop old version after promoting |
| `--quiet` | Skip confirmation prompts |

---

## 4. Services and Versions

### Hierarchy
```
App Engine Application (1 per project)
└── Service (e.g., default, api, worker)
    └── Version (v1, v2, v3, ...)
        └── Instance(s)
```

- **Service** — independent microservice (own scaling, runtime, code)
- **Version** — deployed iteration of a service
- **Default service** is named `default`

```bash
# List
gcloud app services list
gcloud app versions list

# Stop a version (no traffic, no cost)
gcloud app versions stop v1 --service=default

# Start
gcloud app versions start v1 --service=default

# Delete a version
gcloud app versions delete v1 --service=default
```

---

## 5. Traffic Splitting

```bash
# 100% to v2 (instant cutover)
gcloud app services set-traffic default --splits=v2=1

# 80/20 split — by IP, cookie, or random
gcloud app services set-traffic default \
  --splits=v1=0.8,v2=0.2 \
  --split-by=cookie    # or random / ip
```

### Migration vs. immediate
```bash
# Gradual migration (only Standard with auto-scaling)
gcloud app services set-traffic default \
  --splits=v2=1 --migrate
```

> **Exam tip:** "Gradual canary on App Engine" → use `--migrate` (Standard only).

---

## 6. Dispatch File (Routing Across Services)

`dispatch.yaml` routes URL patterns to different services.

```yaml
dispatch:
- url: "*/api/*"
  service: api
- url: "*/admin/*"
  service: admin
- url: "*/*"
  service: default
```

Deploy:
```bash
gcloud app deploy dispatch.yaml
```

---

## 7. Scaling Modes (Standard)

| Mode | Behavior |
|---|---|
| **Automatic** | Default. Scales up on traffic, can scale to 0 if `min_instances=0` |
| **Manual** | Fixed instance count, always running |
| **Basic** | Spins up on requests, shuts down when idle (Standard only) |

```yaml
# Automatic
automatic_scaling:
  min_instances: 0
  max_instances: 20
  target_cpu_utilization: 0.6

# Manual
manual_scaling:
  instances: 3

# Basic (Standard)
basic_scaling:
  max_instances: 5
  idle_timeout: 10m
```

---

## 8. Authentication & IAP

- App Engine inherits a **default service account**: `PROJECT_ID@appspot.gserviceaccount.com`
- Use **Identity-Aware Proxy (IAP)** to gate the entire app behind Google login + IAM:
  ```bash
  gcloud iap web enable --resource-type=app-engine
  ```
- IAM role for invoking: `roles/iap.httpsResourceAccessor`

---

## 9. Triggering App Engine from Events

App Engine apps are HTTP services, so you trigger them like any web service:

- **Pub/Sub push** to `https://service-dot-PROJECT_ID.uc.r.appspot.com/handler`
- **Cloud Scheduler** for cron jobs (or `cron.yaml` for App-Engine-native cron)
- **Eventarc** to App Engine is **not** directly supported — wrap in Pub/Sub push or migrate to Cloud Run

### `cron.yaml` (Standard only)
```yaml
cron:
- description: "Nightly cleanup"
  url: /tasks/cleanup
  schedule: every day 02:00
  timezone: Europe/Bucharest
```
Deploy: `gcloud app deploy cron.yaml`

---

## 10. Common gcloud Operations

```bash
# Open the deployed app
gcloud app browse

# Logs
gcloud app logs tail -s default
gcloud app logs read --limit=50

# SSH (Flex only)
gcloud app instances ssh INSTANCE_ID --service=default --version=v1
```

---

## 11. App Engine vs. Cloud Run (When to Pick Which)

| Pick App Engine when... | Pick Cloud Run when... |
|---|---|
| Existing legacy App Engine app | Greenfield containerized service |
| You want `cron.yaml`, `dispatch.yaml`, multi-version splitting in one place | You want a smaller, simpler model |
| Standard languages are sufficient | You need custom runtime / OS |
| You like `app.yaml` config style | You prefer container/IaC workflow |

> Google's modern recommendation is **Cloud Run for new apps**. App Engine remains for backward compatibility.

---

## 12. Exam Traps & Keywords

| If you see... | Answer |
|---|---|
| "Existing App Engine app, multi-version canary" | App Engine versions + `set-traffic --migrate` |
| "Custom Docker, but staying on App Engine" | App Engine **Flex** |
| "Scale to zero on App Engine" | App Engine **Standard** (Flex does not) |
| "Route /api → service A, /admin → service B" | `dispatch.yaml` |
| "Cron job inside App Engine" | `cron.yaml` (Standard) |
| "App Engine region change" | **Not possible** — recreate the project |

---

## 13. Sources

- [App Engine docs](https://cloud.google.com/appengine/docs)
- [Standard environment](https://cloud.google.com/appengine/docs/standard)
- [Flexible environment](https://cloud.google.com/appengine/docs/flexible)
- [`app.yaml` reference](https://cloud.google.com/appengine/docs/standard/python3/config/appref)
- [Splitting traffic](https://cloud.google.com/appengine/docs/standard/splitting-traffic)
