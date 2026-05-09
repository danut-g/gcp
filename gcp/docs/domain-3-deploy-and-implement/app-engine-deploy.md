# App Engine Deployment: Standard vs Flexible, Versions, Traffic Splitting

## Overview

**App Engine** is GCP's original PaaS offering, allowing application deployment without managing servers. Understanding the differences between Standard and Flexible environments, version management, and traffic routing is important for the ACE exam — especially for questions involving legacy applications or the 60-second request timeout distinction.

---

## Key Concepts

### Standard vs Flexible Environment

| Feature | Standard | Flexible |
|---------|----------|---------|
| Runtimes | Specific language versions only | Any (Docker) |
| Scale to zero | Yes | **No** (min 1 instance) |
| Scale speed | Very fast (seconds) | Slower (minutes) |
| Request timeout | **60 seconds (hard limit)** | 60 minutes |
| Background tasks | Limited | Full OS access |
| Pricing | Per instance-hour (F-class) | Per vCPU/hour + memory |
| Sustained Use Discount | No | No |
| SSH to instances | No | Yes |
| Network egress | Via App Engine | Via underlying GCE VMs |
| Custom domain + SSL | Yes | Yes |
| Local disk | Limited (/tmp, 32 MB) | Persistent disk |
| System packages | No (sandbox) | Yes (Docker) |

#### Standard Environment Details

- **Instances classes**: F1 (128 MB, ~600 MHz), F2 (256 MB), F4 (1 GB), B1-B4 (background tasks)
- F-class: Frontend (web serving); B-class: Backend (background tasks)
- Sandboxed: Cannot use arbitrary system calls, write to disk outside /tmp, or make network connections via raw sockets
- **Languages**: Python, Java, Node.js, Go, Ruby, PHP (specific versions only)
- **Free tier**: F1 instances include 28 free instance-hours/day

#### Flexible Environment Details

- Runs Docker containers on Compute Engine VMs (App Engine manages them)
- Does NOT scale to zero — always maintains at least one instance
- Instance replacement during deployment takes minutes
- Uses standard GCE machine types
- No SUD (Sustained Use Discounts) for Flexible instances

---

### App Engine Application Structure

- **Application**: One per project, per region (cannot change region after creation — permanent)
- **Service**: (formerly "modules") — logical components of the app; each service runs independently
- **Version**: A specific deployment of a service; multiple versions can coexist
- **Instance**: Running copy of a version

```
Application (project-level, single region)
└── Service (default service + optional named services)
    └── Version (multiple versions per service)
        └── Instance (running copies)
```

---

### Deploying Versions

- Deploy with: `gcloud app deploy`
- Each `gcloud app deploy` creates a new **version**
- By default, all traffic routes to the newly deployed version (`--promote` flag, on by default)
- `--no-promote`: Deploy without routing traffic to new version (safe deployment for testing)
- `--no-stop-previous-version`: Keep old version running (otherwise old version stops after traffic moves)

---

### Traffic Splitting

- Distribute traffic across multiple versions of the same service
- Methods of traffic splitting:
  - **IP address**: Split based on client IP (sticky for same client)
  - **Cookie**: Split based on HTTP cookie (stickier than IP; persists across IPs)
  - **Random**: Random assignment per request (no stickiness; true percentage split)
- Use case: Canary deployments, A/B testing, gradual rollouts
- Each version must be running to receive traffic

---

### Autoscaling in App Engine Standard

Three scaling types:

| Type | Description | Use Case |
|------|-------------|---------|
| **Automatic scaling** | Scales based on request rate, CPU, latency | Most web apps |
| **Basic scaling** | Scale to 0 when no traffic; slower scale-up | Intermittent workloads |
| **Manual scaling** | Fixed number of instances | Predictable load, background workers |

- Automatic scaling can scale to zero (instances stop when idle)
- Minimum/maximum idle instances, pending latency, and concurrent requests are tunable
- **Warmup requests**: GCP can send warmup requests to new instances to pre-initialize before they receive traffic

---

### App Engine Services

- Applications can have multiple services (default + named)
- Each service is deployed independently with its own versions
- Services can be scaled independently
- Dispatch rules (`dispatch.yaml`): Route URL patterns to specific services
- Cron jobs (`cron.yaml`): Scheduled tasks sent as HTTP requests to a URL
- Task queues (`queue.yaml`): Configuration for Cloud Tasks queues (App Engine integration)

---

### app.yaml Configuration

The `app.yaml` file defines the App Engine application configuration:

```yaml
runtime: python312
instance_class: F2

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.65

env_variables:
  DB_HOST: "10.0.0.1"

handlers:
- url: /static
  static_dir: static/

- url: /.*
  script: auto
```

- `runtime`: Language and version
- `instance_class`: F1–F4 for Standard (affects CPU/memory)
- `env_variables`: Environment variables (not for secrets)
- `handlers`: URL routing rules

---

### Deploying to App Engine: Key Files

| File | Purpose |
|------|---------|
| `app.yaml` | Main application config (required) |
| `dispatch.yaml` | Route URLs across services |
| `cron.yaml` | Scheduled tasks |
| `queue.yaml` | Task queue config |
| `index.yaml` | Datastore/Firestore composite index config |

---

### App Engine Firewall

- App Engine has its own firewall (independent of VPC firewall rules)
- Rules apply to incoming HTTP requests to App Engine services
- Configured via the App Engine firewall page in Console or `gcloud app firewall-rules`
- Rules: allow or deny based on IP range; evaluated in priority order
- Default: allow all

---

## When to Use

- Migrating legacy App Engine applications (Python 2.7, Java 8 → newer runtimes with minimal code change)
- Simple web applications where the team doesn't want to manage containers
- Applications needing task queues and cron jobs tightly integrated with App Engine
- When scale-to-zero on Standard is needed for cost efficiency on low-traffic apps

---

## When NOT to Use

- **New applications**: Prefer Cloud Run (more flexible, more modern, better container support)
- **60+ second request handling**: App Engine Standard's hard 60s limit disqualifies it
- **Custom system packages or binary dependencies**: App Engine Standard is sandboxed; use Flexible or Cloud Run
- **App Engine Flexible for new apps**: The "Flexible" option is largely superseded by Cloud Run; avoid for new workloads

---

## Related Services / Concepts

- **Cloud Run/Functions Planning**: Selection framework — see [cloud-run-functions-planning.md](../domain-2-plan-and-configure/cloud-run-functions-planning.md)
- **Cloud Run Deploy**: The modern serverless alternative — see [cloud-run-deploy.md](cloud-run-deploy.md)
- **Cloud Tasks**: Replaces App Engine task queues for new projects
- **Monitoring**: App Engine metrics — see [monitoring-cloud-ops.md](../domain-4-ensure-success/monitoring-cloud-ops.md)

---

## Exam-Relevant Notes

### Common Traps

1. **App Engine region is permanent**: You choose a region when you create the App Engine application for a project. It cannot be changed. This is a common scenario question: "The team chose the wrong region. How do they fix it?" — Answer: Must create a new project.

2. **Standard: 60-second hard limit**: No exceptions. A question mentioning requests taking > 60s + App Engine Standard = wrong combination. Use Flexible or Cloud Run.

3. **Flexible doesn't scale to zero**: If a question says "minimize cost when there's no traffic" + App Engine, the answer is Standard (scales to zero). Flexible always runs at least one instance.

4. **Traffic splitting methods differ in stickiness**: IP splitting is less sticky than cookie. If you need a specific user to consistently see the same version (important for A/B tests), cookie-based splitting is better.

5. **`--no-promote` for safe deployment**: Deploy a new version without sending it any traffic, test it directly at its URL, then migrate traffic manually. This is the safe deployment pattern.

6. **App Engine default SA has Editor role**: Just like Cloud Functions, be aware that the default SA has broad permissions.

7. **One App Engine app per project**: A project can have only one App Engine application. The application can have multiple services, but there's only one App Engine "app" per project.

8. **Warmup requests**: App Engine Standard can send warmup requests to new instances (configured via `/_ah/warmup` handler) before they receive user traffic, reducing perceived cold start latency.

### Keywords
- App Engine Standard, App Engine Flexible, app.yaml, version, service, traffic splitting, IP-based split, cookie-based split, 60-second timeout, scale to zero, `--no-promote`, dispatch.yaml, cron.yaml, instance class, automatic scaling, manual scaling

---

## Source

- https://cloud.google.com/appengine/docs/standard
- https://cloud.google.com/appengine/docs/flexible
- https://cloud.google.com/appengine/docs/standard/traffic-splitting
- https://cloud.google.com/appengine/docs/standard/deploying-an-app
