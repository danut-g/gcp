# Section 3.3 — Deploying and Implementing Cloud Run and Cloud Functions Resources

## Exam Relevance
This topic is part of **Section 3: Deploying and implementing a cloud solution (~25 % of the exam)**. You must know how to deploy applications to Cloud Run and Cloud Functions, configure event-driven architectures using Pub/Sub, Cloud Storage, and Eventarc, and determine where to deploy applications.

---

## 1. Cloud Run — Deploying an Application

> 📖 **Docs:** [Deploy to Cloud Run](https://cloud.google.com/run/docs/deploying) | [Cloud Run quickstart](https://cloud.google.com/run/docs/quickstarts/build-and-deploy) | 🖥️ **Console:** Cloud Run → Create Service

### What Is Cloud Run?
- Fully managed serverless platform for running **containers**
- Automatically scales from zero to thousands of instances
- Pay only when handling requests (per 100ms billing granularity)
- Supports any language/framework (container-based)

### Deploying to Cloud Run

#### From a Container Image

```bash
# Deploy a container image to Cloud Run
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
  --set-env-vars="DB_HOST=10.0.0.5,ENV=production" \
  --service-account=my-sa@PROJECT_ID.iam.gserviceaccount.com
```

#### From Source Code (Cloud Build)

```bash
# Deploy directly from source (Cloud Build creates the container)
gcloud run deploy my-service \
  --source=. \
  --region=us-central1 \
  --allow-unauthenticated
```

### Key Deployment Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `--image` | Container image URL | Required |
| `--region` | Deployment region | Required |
| `--port` | Container port | 8080 |
| `--memory` | Memory allocation | 512Mi |
| `--cpu` | CPU allocation | 1 |
| `--min-instances` | Minimum instances (0 = scale to zero) | 0 |
| `--max-instances` | Maximum instances | 100 |
| `--timeout` | Request timeout (seconds) | 300 (5 min) |
| `--concurrency` | Max concurrent requests per instance | 80 |
| `--allow-unauthenticated` | Allow public access | Requires auth by default |
| `--no-allow-unauthenticated` | Require authentication | |
| `--service-account` | Service account for the service | Default compute SA |
| `--set-env-vars` | Environment variables | None |
| `--set-secrets` | Mount secrets from Secret Manager | None |
| `--vpc-connector` | Connect to a VPC | None |
| `--ingress` | Control inbound access (`all`, `internal`, `internal-and-cloud-load-balancing`) | `all` |

### Cloud Run Revisions
Each deployment creates a new **revision** (immutable snapshot of the service):

```bash
# List revisions
gcloud run revisions list --service=my-service --region=us-central1

# Describe a revision
gcloud run revisions describe REVISION_NAME --region=us-central1

# Delete a revision
gcloud run revisions delete REVISION_NAME --region=us-central1
```

### Traffic Management

```bash
# Send 100% traffic to a specific revision
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00005-abc=100

# Split traffic between revisions (canary deployment)
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00004-xyz=90,my-service-00005-abc=10

# Roll back to the previous revision
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-revisions=my-service-00004-xyz=100

# Gradual rollout with tags
gcloud run services update-traffic my-service \
  --region=us-central1 \
  --to-tags=canary=10
```

---

## 2. Cloud Functions — Deploying an Application

> 📖 **Docs:** [Cloud Functions overview](https://cloud.google.com/functions/docs/concepts/overview) | [Deploy a function](https://cloud.google.com/functions/docs/create-deploy-gcloud) | 🖥️ **Console:** Cloud Functions → Functions → Create function

### What Are Cloud Functions?
- **Function as a Service (FaaS)** — event-driven, serverless functions
- Write code in supported languages, deploy, and Google manages everything
- Two generations: Gen 1 (legacy) and Gen 2 (recommended, built on Cloud Run)

### Supported Runtimes

| Language | Gen 1 | Gen 2 |
|----------|-------|-------|
| Node.js | 16, 18, 20 | 18, 20, 22 |
| Python | 3.8-3.12 | 3.8-3.12 |
| Go | 1.19-1.22 | 1.19-1.22 |
| Java | 11, 17, 21 | 11, 17, 21 |
| .NET | 6, 8 | 6, 8 |
| Ruby | 3.0-3.3 | 3.0-3.3 |
| PHP | 8.1-8.3 | 8.1-8.3 |

### Deploying Gen 2 Cloud Functions

#### HTTP-Triggered Function

```bash
# Deploy an HTTP function
gcloud functions deploy my-http-function \
  --gen2 \
  --region=us-central1 \
  --runtime=python312 \
  --trigger-http \
  --allow-unauthenticated \
  --entry-point=hello_http \
  --memory=256Mi \
  --timeout=60s \
  --min-instances=0 \
  --max-instances=100 \
  --source=.
```

**Python example** (`main.py`):
```python
import functions_framework

@functions_framework.http
def hello_http(request):
    name = request.args.get("name", "World")
    return f"Hello, {name}!"
```

#### Event-Triggered Function

```bash
# Deploy a Pub/Sub-triggered function
gcloud functions deploy my-pubsub-function \
  --gen2 \
  --region=us-central1 \
  --runtime=python312 \
  --trigger-topic=my-topic \
  --entry-point=process_message \
  --source=.

# Deploy a Cloud Storage-triggered function
gcloud functions deploy my-storage-function \
  --gen2 \
  --region=us-central1 \
  --runtime=python312 \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=my-bucket" \
  --entry-point=process_file \
  --source=.
```

**Python Pub/Sub example** (`main.py`):
```python
import base64
import functions_framework
from cloudevents.http import CloudEvent

@functions_framework.cloud_event
def process_message(cloud_event: CloudEvent):
    data = base64.b64decode(cloud_event.data["message"]["data"]).decode()
    print(f"Received message: {data}")
```

### Gen 1 vs. Gen 2 Comparison

| Feature | Gen 1 | Gen 2 |
|---------|-------|-------|
| Max timeout | 9 minutes | 60 minutes |
| Max memory | 8 GB | 32 GB |
| Max vCPUs | 2 | 8 |
| Concurrency | 1 request per instance | Up to 1,000 per instance |
| Traffic splitting | No | Yes (built on Cloud Run) |
| Min instances | Yes | Yes |
| Event sources | HTTP, Pub/Sub, Storage, Firestore | Same + Eventarc (100+) |
| Built on | Custom infra | Cloud Run |

---

## 3. Event-Driven Architectures

> 📖 **Docs:** [Eventarc overview](https://cloud.google.com/eventarc/docs/overview) | [Pub/Sub overview](https://cloud.google.com/pubsub/docs/overview) | 🖥️ **Console:** Eventarc → Triggers

### Pub/Sub Events

**Pub/Sub** is Google Cloud's messaging service for event-driven architectures:

```
Publisher → Topic → Subscription → Subscriber
              │         │
              │         ├── Pull (subscriber pulls messages)
              │         └── Push (Pub/Sub pushes to HTTP endpoint)
              │
              └── Can have multiple subscriptions
```

```bash
# Create a Pub/Sub topic
gcloud pubsub topics create my-topic

# Create a push subscription to Cloud Run
gcloud pubsub subscriptions create my-sub \
  --topic=my-topic \
  --push-endpoint=https://my-service-xxx-uc.a.run.app

# Publish a test message
gcloud pubsub topics publish my-topic --message="Hello!"
```

### Cloud Storage Object Change Notifications

Trigger functions when objects are created, deleted, or modified in Cloud Storage:

| Event Type | Trigger |
|-----------|---------|
| `google.cloud.storage.object.v1.finalized` | Object created or overwritten |
| `google.cloud.storage.object.v1.deleted` | Object deleted |
| `google.cloud.storage.object.v1.archived` | Object archived (versioned bucket) |
| `google.cloud.storage.object.v1.metadataUpdated` | Object metadata changed |

```bash
# Deploy function triggered by new objects in a bucket
gcloud functions deploy process-uploads \
  --gen2 \
  --region=us-central1 \
  --runtime=nodejs20 \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=my-upload-bucket" \
  --entry-point=processFile
```

### Eventarc

**Eventarc** is the unified event routing service for Google Cloud:

```
Event Source → Eventarc Trigger → Target (Cloud Run, Cloud Functions, GKE, Workflows)
```

**Supported event sources** (100+):
- Cloud Storage (object changes)
- Pub/Sub (messages)
- Cloud Audit Logs (any audited API call)
- Firebase events
- Third-party sources (via Pub/Sub)
- Custom events

```bash
# Create an Eventarc trigger for Cloud Audit Logs
gcloud eventarc triggers create my-audit-trigger \
  --destination-run-service=my-service \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.audit.log.v1.written" \
  --event-filters="serviceName=storage.googleapis.com" \
  --event-filters="methodName=storage.objects.create" \
  --service-account=my-trigger-sa@PROJECT_ID.iam.gserviceaccount.com

# List triggers
gcloud eventarc triggers list --location=us-central1

# Describe a trigger
gcloud eventarc triggers describe my-trigger --location=us-central1
```

### Event Architecture Patterns

```
Pattern 1: Direct Trigger
Cloud Storage → Cloud Function (processes file)

Pattern 2: Pub/Sub Decoupling
Publisher → Pub/Sub → Cloud Run (processes messages)

Pattern 3: Eventarc Routing
Cloud Audit Log → Eventarc → Cloud Run (reacts to API calls)

Pattern 4: Fan-out
Pub/Sub Topic → Subscription 1 → Cloud Function A
              → Subscription 2 → Cloud Function B
              → Subscription 3 → Cloud Run C
```

---

## 4. Determining Where to Deploy

> 📖 **Docs:** [Choosing a serverless option](https://cloud.google.com/blog/topics/developers-practitioners/cloud-run-vs-cloud-functions-what-i-use-and-why) | [Cloud Run locations](https://cloud.google.com/run/docs/locations) | 🖥️ **Console:** Cloud Run → region dropdown

### Decision Flowchart

```
Is it a simple event-triggered function (< 60 min)?
│
├── YES → Is it a single-purpose function?
│          ├── YES → Cloud Functions (Gen 2)
│          └── NO (multiple routes/handlers) → Cloud Run
│
└── NO → Is it a containerized application?
          ├── YES → Need Kubernetes orchestration?
          │          ├── YES → GKE
          │          └── NO → Cloud Run
          └── NO → Need full OS control?
                    ├── YES → Compute Engine
                    └── NO → Cloud Run (containerize it)
```

### Cloud Run vs. Cloud Functions — When to Choose

| Choose Cloud Run When | Choose Cloud Functions When |
|----------------------|---------------------------|
| Multiple endpoints/routes | Single-purpose function |
| Need custom Dockerfile | Simple code in supported runtime |
| Need WebSocket/gRPC | Quick event processing |
| Team prefers containers | Team prefers writing functions |
| Need more than 32 GB memory | Short execution time (< 60 min) |
| Want traffic splitting | Simple Pub/Sub or Storage triggers |
| Migrating existing container | Glue code between services |

### Cloud Run for Anthos (GKE)
- Run Cloud Run workloads on your own GKE cluster
- Use case: Workloads that must run on dedicated infrastructure
- Benefits from GKE networking, GPU, and compliance capabilities
- Being deprecated in favor of standard Cloud Run + GKE

---

## 5. Cloud Run Configuration Deep Dive

> 📖 **Docs:** [Configuring Cloud Run](https://cloud.google.com/run/docs/configuring/overview) | [Concurrency and scaling](https://cloud.google.com/run/docs/configuring/concurrency) | 🖥️ **Console:** Cloud Run → select service → Edit & Deploy New Revision

### Environment Variables and Secrets

```bash
# Set environment variables
gcloud run deploy my-service \
  --set-env-vars="KEY1=value1,KEY2=value2"

# Mount Secret Manager secrets as env vars
gcloud run deploy my-service \
  --set-secrets="DB_PASSWORD=my-secret:latest"

# Mount secrets as files
gcloud run deploy my-service \
  --set-secrets="/secrets/db-password=my-secret:latest"

# Update env vars on existing service
gcloud run services update my-service \
  --update-env-vars="KEY3=value3" \
  --region=us-central1
```

### VPC Connectivity

```bash
# Create a Serverless VPC Access connector
gcloud compute networks vpc-access connectors create my-connector \
  --region=us-central1 \
  --subnet=my-subnet \
  --subnet-project=PROJECT_ID

# Deploy Cloud Run with VPC connector
gcloud run deploy my-service \
  --vpc-connector=my-connector \
  --vpc-egress=all-traffic \
  --region=us-central1
```

### Ingress Settings

| Setting | Description |
|---------|-------------|
| `all` | Accept traffic from anywhere (default) |
| `internal` | Only from VPC, Cloud Interconnect, or VPN |
| `internal-and-cloud-load-balancing` | From VPC or through external Application LB |

---

## 6. Managing Cloud Functions

```bash
# List deployed functions
gcloud functions list

# Describe a function
gcloud functions describe my-function --region=us-central1

# View function logs
gcloud functions logs read my-function --region=us-central1

# Delete a function
gcloud functions delete my-function --region=us-central1

# Call an HTTP function
gcloud functions call my-http-function \
  --data='{"name": "World"}' \
  --region=us-central1

# Update function configuration
gcloud functions deploy my-function \
  --gen2 \
  --region=us-central1 \
  --runtime=python312 \
  --memory=512Mi \
  --timeout=120s \
  --update-env-vars="NEW_VAR=value"
```

---

## 7. Cloud Scheduler Integration

> 📖 **Docs:** [Cloud Scheduler overview](https://cloud.google.com/scheduler/docs/overview) | [Schedule Cloud Run with Scheduler](https://cloud.google.com/run/docs/triggering/using-scheduler) | 🖥️ **Console:** Cloud Scheduler → Create job

- Cloud Scheduler sends HTTP requests or Pub/Sub messages on a cron schedule
- Use cases: scheduled Cloud Run jobs, periodic Cloud Functions invocations, batch pipelines

```bash
# Creating a Cloud Scheduler job to invoke Cloud Run
gcloud scheduler jobs create http my-job \
  --location=us-central1 \
  --schedule="0 9 * * 1" \
  --uri="https://my-service-xxx-uc.a.run.app/run" \
  --http-method=POST \
  --message-body='{"key":"value"}' \
  --oidc-service-account-email=MY_SA@PROJECT.iam.gserviceaccount.com \
  --oidc-token-audience="https://my-service-xxx-uc.a.run.app"
# Via Pub/Sub to Cloud Functions
gcloud scheduler jobs create pubsub my-pubsub-job \
  --location=us-central1 \
  --schedule="*/5 * * * *" \
  --topic=MY_TOPIC \
  --message-body="trigger"
gcloud scheduler jobs list --location=us-central1
gcloud scheduler jobs run my-job --location=us-central1  # manual trigger for testing
```

- OIDC token is required for Cloud Run (authenticated services); the scheduler SA must have `roles/run.invoker`

---

## 8. Service-to-Service Authentication

> 📖 **Docs:** [Service-to-service authentication](https://cloud.google.com/run/docs/authenticating/service-to-service) | [OIDC tokens](https://cloud.google.com/run/docs/securing/service-identity) | 🖥️ **Console:** Cloud Run → select service → IAM tab

- When Cloud Run service A calls Cloud Run service B (which requires auth):
  1. Service A's service account must have `roles/run.invoker` on service B
  2. Service A fetches an ID token from the metadata server with the audience set to service B's URL

```python
import google.auth.transport.requests
import google.oauth2.id_token
audience = "https://service-b-xxx-uc.a.run.app"
request = google.auth.transport.requests.Request()
token = google.oauth2.id_token.fetch_id_token(request, audience)
# Then set Authorization: Bearer <token> in the outbound request
```

- **Exam tip**: OIDC ID tokens (for identity verification) vs. OAuth2 access tokens (for GCP API calls) — Cloud Run auth uses OIDC

---

## 9. Custom Domains and Ingress

> 📖 **Docs:** [Map custom domains](https://cloud.google.com/run/docs/mapping-custom-domains) | [Ingress settings](https://cloud.google.com/run/docs/securing/ingress) | 🖥️ **Console:** Cloud Run → select service → Triggers tab → Manage Custom Domains

- Cloud Run: map custom domain via Google Cloud Console or:

```bash
gcloud beta run domain-mappings create --service=MY_SERVICE --domain=api.example.com --region=us-central1
gcloud beta run domain-mappings list --region=us-central1
```

- Requires DNS CNAME verification; Google manages the TLS certificate automatically
- Ingress options (controls who can reach the service):
  - `all`: any internet traffic
  - `internal`: only from same VPC, Shared VPC, or Cloud Run in same project
  - `internal-and-cloud-load-balancing`: internal + traffic through Google Cloud external LB

```bash
gcloud run services update MY_SERVICE --ingress=internal --region=us-central1
```

- To expose an internal-only Cloud Run service publicly: put it behind an external HTTPS LB with Cloud Run NEG backend

---

## 10. Cloud Run Jobs

> 📖 **Docs:** [Cloud Run Jobs](https://cloud.google.com/run/docs/create-jobs) | [Execute a job](https://cloud.google.com/run/docs/execute/jobs) | 🖥️ **Console:** Cloud Run → Jobs → Create job

- **Cloud Run Jobs** run a container to completion (batch workloads) rather than serving HTTP requests
- Used for batch processing, database migrations, scheduled ETL, and one-off administrative tasks
- Unlike Cloud Run Services, Jobs do not listen on a port; they run to completion and exit

```bash
# Create a Cloud Run Job
gcloud run jobs create my-job \
  --image=us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-batch:v1 \
  --region=us-central1 \
  --tasks=10 \
  --parallelism=5 \
  --max-retries=3 \
  --task-timeout=600 \
  --memory=1Gi \
  --cpu=2 \
  --set-env-vars="BATCH_SIZE=100"

# Execute the job (runs it once)
gcloud run jobs execute my-job --region=us-central1

# List executions
gcloud run jobs executions list --region=us-central1

# Schedule a job via Cloud Scheduler (through Jobs API)
gcloud scheduler jobs create http run-my-job \
  --location=us-central1 \
  --schedule="0 3 * * *" \
  --uri="https://us-central1-run.googleapis.com/apis/run.googleapis.com/v1/namespaces/PROJECT_ID/jobs/my-job:run" \
  --http-method=POST \
  --oauth-service-account-email=MY_SA@PROJECT.iam.gserviceaccount.com
```

- `--tasks` — number of independent task instances the job runs
- `--parallelism` — how many tasks run concurrently
- **Services vs. Jobs**: Services handle requests (HTTP/gRPC, scale to/from zero); Jobs run to completion (batch, no request serving)

---

## 11. Cloud Run IAM Roles

> 📖 **Docs:** [Cloud Run IAM roles](https://cloud.google.com/run/docs/reference/iam/roles) | [Access control with IAM](https://cloud.google.com/run/docs/securing/managing-access) | 🖥️ **Console:** Cloud Run → select service → Security tab → IAM

| Role | Description |
|------|-------------|
| `roles/run.admin` | Full control over Cloud Run services and jobs |
| `roles/run.developer` | Deploy and manage services (cannot set IAM policies) |
| `roles/run.invoker` | Invoke authenticated Cloud Run services (call them over HTTP) |
| `roles/run.viewer` | Read-only access to Cloud Run resources |
| `roles/iam.serviceAccountUser` | Required to deploy a service that runs as a specific SA |

```bash
# Allow a user to invoke an authenticated Cloud Run service
gcloud run services add-iam-policy-binding my-service \
  --region=us-central1 \
  --member="user:alice@example.com" \
  --role="roles/run.invoker"

# Allow public (unauthenticated) access
gcloud run services add-iam-policy-binding my-service \
  --region=us-central1 \
  --member="allUsers" \
  --role="roles/run.invoker"
```

- **Exam tip**: `--allow-unauthenticated` is equivalent to granting `roles/run.invoker` to `allUsers`
- A Cloud Function Gen 2 service is backed by a Cloud Run service — the same invoker IAM role controls access

---

## 12. Cold Starts and Min Instances

> 📖 **Docs:** [Cold starts](https://cloud.google.com/run/docs/tips/general#avoiding-cold-starts) | [Min instances](https://cloud.google.com/run/docs/configuring/min-instances) | 🖥️ **Console:** Cloud Run → Edit → Capacity → Minimum number of instances

- **Cold start** — the latency penalty incurred when Cloud Run or Cloud Functions must spin up a new container instance to handle an incoming request (no warm instances available)
- Cold start duration depends on container size, runtime, and initialization code (typically 100 ms–several seconds)
- Mitigation strategies:
  - Set `--min-instances` > 0 to keep instances warm (incurs cost even when idle)
  - Use **CPU always allocated** instead of the default request-scoped CPU, so background initialization continues between requests
  - Optimize container image size and startup code
  - Use **startup CPU boost** (`--cpu-boost`) to temporarily allocate extra CPU during instance start

```bash
# Keep 2 instances always warm and enable CPU boost
gcloud run deploy my-service \
  --min-instances=2 \
  --cpu-boost \
  --region=us-central1
```

---

## Exam Practice Questions

1. **You need to deploy a Python function that processes images uploaded to Cloud Storage. The processing takes about 30 seconds per image. Which service should you use and how?**
   - Answer: **Cloud Functions Gen 2** with a Cloud Storage trigger on the `finalized` event. Deploy with `--trigger-event-filters="type=google.cloud.storage.object.v1.finalized"`.

2. **Your team has a containerized Go application with 5 HTTP endpoints. They want serverless scaling with no Kubernetes overhead. Where should they deploy?**
   - Answer: **Cloud Run** — Supports containers with multiple endpoints, serverless scaling, and no cluster management.

3. **You want to perform a canary release, sending 10% of traffic to a new version of your Cloud Run service. How?**
   - Answer: Deploy the new version, then use `gcloud run services update-traffic --to-revisions=old=90,new=10`.

4. **A Cloud Run service needs to access a Cloud SQL instance over a private IP. What do you need to configure?**
   - Answer: Create a **Serverless VPC Access connector** and deploy the Cloud Run service with `--vpc-connector` and `--vpc-egress=private-ranges-only` (or `all-traffic`).

5. **Your function needs to react when any VM is created in your project. Which event source and service should you use?**
   - Answer: Use **Eventarc** with a Cloud Audit Log trigger: filter on `type=google.cloud.audit.log.v1.written`, `serviceName=compute.googleapis.com`, `methodName=v1.compute.instances.insert`. Route to Cloud Run or Cloud Functions.

6. **What is the maximum request timeout for Cloud Functions Gen 2?**
   - Answer: **60 minutes** (same as Cloud Run, since Gen 2 is built on Cloud Run).

7. **You need your Cloud Run service to only accept traffic from within your VPC and through your external load balancer. Which ingress setting should you use?**
   - Answer: `--ingress=internal-and-cloud-load-balancing`

---

## Glossary

**allUsers** — A special IAM identifier representing anyone on the internet, including unauthenticated users. Granting `roles/run.invoker` to `allUsers` on a Cloud Run service makes it publicly accessible (equivalent to `--allow-unauthenticated`).

**Apache Beam** — An open-source, unified programming model for defining batch and streaming data processing pipelines. Google Dataflow is a fully managed runner for Apache Beam pipelines.

**Application Load Balancer (External)** — A GCP global Layer 7 load balancer that distributes HTTP(S) traffic across backend services. Can front Cloud Run services via a Serverless NEG to expose internal-only services or add features like Cloud Armor and custom domains.

**Artifact Registry** — GCP's fully managed service for storing and managing container images and other build artifacts. Serves as the image source for Cloud Run and GKE deployments.

**Authentication** — The process of verifying the identity of a caller. Cloud Run services can require authentication (via OIDC ID tokens) or allow unauthenticated public access.

**Authorization** — The process of determining what an authenticated identity is allowed to do. On Cloud Run, governed by IAM roles such as `roles/run.invoker`.

**Backend Service** — A GCP load balancer component that defines how traffic is distributed to backends (e.g., instance groups, Serverless NEGs, Cloud Run services). Cloud Armor policies attach to backend services.

**Audit Log (Cloud Audit Log)** — A GCP log that records administrative actions (Admin Activity) and data access operations on GCP resources. Used as an Eventarc event source to trigger Cloud Run or Cloud Functions on any audited API call.

**Canary Deployment** — A deployment strategy that gradually shifts a small percentage of traffic to a new revision before fully rolling it out. Cloud Run supports canary deployments via `gcloud run services update-traffic`.

**Cloud Armor** — GCP's Layer 7 DDoS protection and Web Application Firewall service. Attaches to external Application Load Balancer backend services, including those fronting Cloud Run.

**Cloud Build** — GCP's fully managed CI/CD build service. Used with `gcloud run deploy --source=.` to automatically build a container image from source code before deploying to Cloud Run.

**Cloud Functions** — GCP's serverless Function as a Service (FaaS) offering that executes event-driven code without provisioning or managing servers. Supports Gen 1 (legacy) and Gen 2 (recommended, built on Cloud Run).

**Cloud Interconnect** — A GCP service that provides dedicated or partner-mediated physical connectivity between an on-premises network and Google's network. Referenced by Cloud Run `internal` ingress (traffic from Interconnect is considered internal).

**Cloud Run** — GCP's fully managed serverless platform for running containers. Automatically scales from zero to thousands of instances based on incoming requests; billing is per 100 ms of execution.

**Cloud Run for Anthos** — A now-deprecated feature that ran Cloud Run workloads on GKE clusters, providing serverless container execution on dedicated infrastructure.

**Cloud Run Job** — A Cloud Run resource that runs a container to completion (batch workload) rather than serving HTTP requests. Supports parallel task execution, retries, and scheduled execution via Cloud Scheduler.

**Cloud Run Service** — A Cloud Run resource that deploys a container as a request-serving application behind a stable HTTPS URL. Contrasts with Cloud Run Jobs, which run to completion.

**Cloud Scheduler** — GCP's fully managed cron job service that sends HTTP requests or Pub/Sub messages on a configurable schedule. Used to invoke Cloud Run services or Cloud Functions periodically.

**Cloud SQL** — GCP's fully managed relational database service supporting MySQL, PostgreSQL, and SQL Server. Cloud Run connects to Cloud SQL over private IP using a Serverless VPC Access connector.

**CloudEvent** — An open specification (CNCF) for describing event data in a common format. Cloud Functions Gen 2 and Eventarc use the CloudEvents format for event payloads.

**CNAME (Canonical Name)** — A DNS record type that aliases one domain name to another. Required when mapping a custom domain to a Cloud Run service.

**CNCF (Cloud Native Computing Foundation)** — A Linux Foundation project that hosts open-source cloud-native technologies, including the CloudEvents specification used by Eventarc.

**Cold Start** — The latency incurred when Cloud Run or Cloud Functions spin up a new container instance from scratch to handle an incoming request. Mitigated via `--min-instances`, CPU boost, and optimized container startup.

**Concurrency** — The number of simultaneous requests a single Cloud Run instance can handle. Cloud Run default is 80; Cloud Functions Gen 1 is 1 (one request per instance); Gen 2 supports up to 1,000.

**CPU Boost (Startup CPU Boost)** — A Cloud Run feature (`--cpu-boost`) that allocates additional CPU to a container instance during startup, reducing cold-start latency for CPU-intensive initialization code.

**Container** — A lightweight, portable unit of software packaging an application and its dependencies. Cloud Run executes containers directly; any language or framework is supported.

**CRON schedule** — A time-based expression (e.g., `0 9 * * 1`) defining when Cloud Scheduler jobs fire. Uses standard Unix cron syntax.

**Custom Domain** — A user-owned domain name (e.g., `api.example.com`) mapped to a Cloud Run service via domain mappings. Google automatically provisions and renews the TLS certificate.

**Dead Letter Topic** — A Pub/Sub topic that receives messages that could not be successfully processed after a configured maximum number of delivery attempts.

**Default Compute Service Account** — A Google-managed service account created automatically in each GCP project (`PROJECT_NUMBER-compute@developer.gserviceaccount.com`). Used by Cloud Run by default unless `--service-account` is specified.

**DNS (Domain Name System)** — The internet protocol that translates human-readable domain names into IP addresses. Required for Cloud Run custom domain mapping via CNAME records.

**Dockerfile** — A text file containing instructions for building a container image. Cloud Run supports any valid Dockerfile; Cloud Functions manages containerization automatically.

**Entry Point** — In Cloud Functions, the name of the specific function within the deployed source code file that is invoked when the trigger fires. Specified with `--entry-point`.

**Environment Variable** — A key-value pair passed to a running container or function at runtime to configure behavior without modifying code. Set with `--set-env-vars` in Cloud Run and Cloud Functions.

**Eventarc** — GCP's unified event routing service that delivers events from 100+ Google Cloud sources (Cloud Audit Logs, Pub/Sub, Cloud Storage, Firebase, etc.) to Cloud Run, Cloud Functions, GKE, and Workflows.

**Event** — A notification about an occurrence in a system (e.g., a new Cloud Storage object, a Pub/Sub message). Events drive serverless execution via triggers.

**Execution (Cloud Run Jobs)** — A single run of a Cloud Run Job. Each `gcloud run jobs execute` invocation creates a new execution, which may contain one or more tasks.

**FaaS (Function as a Service)** — A serverless compute model in which individual functions are deployed and executed in response to events without managing servers. Cloud Functions is GCP's FaaS offering.

**Firebase** — Google's mobile and web application development platform. Firebase events (Firestore changes, Authentication events, etc.) can be consumed by Cloud Run and Cloud Functions via Eventarc.

**Firestore** — GCP's fully managed, serverless NoSQL document database. Firestore events can trigger Cloud Functions Gen 2 via Eventarc.

**Function** — A single unit of deployed code in Cloud Functions that executes in response to an HTTP request or an event. Each function has its own entry point, runtime, memory, and timeout configuration.

**Functions Framework** — An open-source library (for Python, Node.js, Go, Java, etc.) that lets you write Cloud Functions as regular functions (annotated with `@functions_framework.http` or `@functions_framework.cloud_event`) and run them locally or in containers.

**Fan-out** — An event architecture pattern in which a single Pub/Sub topic has multiple subscriptions, each routing messages to a different downstream consumer (Cloud Function, Cloud Run service, etc.).

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services, including compute, storage, networking, databases, analytics, and machine learning.

**gcloud** — The primary command-line tool for interacting with GCP services, part of the Google Cloud SDK.

**GKE (Google Kubernetes Engine)** — GCP's managed Kubernetes service. Eventarc can route events to workloads running on GKE clusters.

**Gen 1 (Cloud Functions Generation 1)** — The original generation of Cloud Functions with lower limits (max 9-minute timeout, 8 GB memory, 1 request per instance). Being superseded by Gen 2.

**Gen 2 (Cloud Functions Generation 2)** — The recommended Cloud Functions generation, built on Cloud Run. Supports 60-minute timeouts, 32 GB memory, 1,000-concurrency per instance, and Eventarc as the event source.

**Go** — A statically typed, compiled programming language developed by Google. Supported as a Cloud Functions and Dataflow runtime.

**gRPC** — A high-performance Remote Procedure Call framework. Cloud Run natively supports gRPC in addition to HTTP, unlike Cloud Functions Gen 1.

**HTTP (Hypertext Transfer Protocol)** — The stateless application protocol used on the web. Cloud Run and Cloud Functions accept HTTP requests as their primary invocation mechanism.

**HTTPS (HTTP Secure)** — The TLS-encrypted version of HTTP. All Cloud Run URLs are HTTPS by default, with Google-managed certificates.

**IAM (Identity and Access Management)** — GCP's system for controlling who can perform what actions on which resources. Controls access to Cloud Run services and determines which service accounts can invoke functions.

**ID Token (OIDC Token)** — A signed token issued by Google's identity platform that asserts the identity of a service account. Used for authenticated Cloud Run service-to-service calls (set in `Authorization: Bearer` header).

**Ingress (Cloud Run)** — A Cloud Run setting that controls which network sources can send requests to a service: `all` (public internet), `internal` (VPC/Interconnect/VPN only), or `internal-and-cloud-load-balancing`.

**Java** — A widely used, object-oriented programming language. Supported as a Cloud Functions runtime in versions 11, 17, and 21.

**Min Instances** — A Cloud Run and Cloud Functions configuration that keeps a specified number of instances warm at all times, eliminating cold starts. Setting to 0 allows scale-to-zero.

**Max Instances** — The upper limit on the number of Cloud Run or Cloud Functions instances that can run concurrently, providing cost control and preventing downstream service overload.

**Memorystore** — GCP's fully managed in-memory data store service, offering Redis and Memcached. Accessible from Cloud Run only via a Serverless VPC Access connector (private IP).

**Metadata Server** — An internal GCP service (accessible at `http://metadata.google.internal`) from which Cloud Run containers and VMs retrieve instance metadata and identity tokens.

**Microservice** — An architectural pattern in which an application is built as a collection of small, independently deployable services communicating over APIs. Cloud Run is commonly used to host microservices.

**NEG (Network Endpoint Group)** — A GCP load balancer backend type that references endpoints other than VMs. A **Serverless NEG** references a Cloud Run service, Cloud Functions, or App Engine application to be fronted by an external Application Load Balancer.

**Node.js** — A JavaScript runtime for server-side applications. Supported as a Cloud Functions and Cloud Run runtime.

**OAuth2 Access Token** — A token used to authenticate calls to GCP service APIs (e.g., Cloud Storage). Distinct from OIDC ID tokens, which are used for Cloud Run service-to-service authentication.

**OIDC (OpenID Connect)** — An identity layer built on OAuth 2.0. Cloud Run and Cloud Scheduler use OIDC tokens to authenticate HTTP requests to secured Cloud Run services.

**PHP** — A server-side scripting language. Supported as a Cloud Functions runtime in versions 8.1–8.3.

**Pub/Sub** — GCP's fully managed, asynchronous messaging service. Used as an event source for Cloud Functions and Cloud Run, and as the underlying delivery mechanism for Eventarc triggers.

**Pull Subscription** — A Pub/Sub subscription model in which the subscriber explicitly calls the service to retrieve messages. Used for applications that control their own consumption rate.

**Push Subscription** — A Pub/Sub subscription model in which the service delivers messages by making HTTP POST requests to a configured endpoint (e.g., a Cloud Run URL or Cloud Functions URL).

**Python** — A general-purpose programming language. Supported as a Cloud Functions runtime in versions 3.8–3.12 and used in code examples throughout this chapter.

**Revision (Cloud Run)** — An immutable snapshot of a Cloud Run service's configuration and container image, created with each deployment. Traffic can be split across revisions for canary releases or rollbacks.

**Role (IAM)** — A named collection of permissions that can be granted to a principal (user, group, service account). Cloud Run uses roles like `roles/run.invoker`, `roles/run.admin`, and `roles/run.developer`.

**Rollback** — Shifting 100% of traffic back to a previous revision after a failed deployment. Implemented in Cloud Run by running `gcloud run services update-traffic --to-revisions=OLD_REVISION=100`.

**Ruby** — A dynamic programming language. Supported as a Cloud Functions runtime in versions 3.0–3.3.

**Scale to Zero** — A Cloud Run and Cloud Functions capability that terminates all instances when there are no incoming requests, reducing costs to zero during idle periods.

**Secret Manager** — GCP's service for securely storing, accessing, and managing sensitive configuration data (API keys, passwords, certificates). Cloud Run mounts secrets as environment variables or files via `--set-secrets`.

**Service (Cloud Run)** — A Cloud Run resource that serves HTTP/gRPC requests. Each service has one or more revisions and an auto-generated HTTPS URL. Contrasted with Cloud Run Jobs.

**Serverless** — A cloud execution model in which the provider automatically manages infrastructure provisioning, scaling, and maintenance. Cloud Run and Cloud Functions are GCP's primary serverless compute products.

**Serverless VPC Access** — A GCP feature that allows Cloud Run services and Cloud Functions to connect to resources within a VPC network (e.g., Cloud SQL on private IP, Memorystore) using a VPC connector.

**Service Account** — A GCP IAM identity used by applications and services (rather than humans) to authenticate API calls. Cloud Run services run under a specified service account for fine-grained access control.

**Storage Transfer Service** — A GCP managed service for large-scale, scheduled data transfers between Cloud Storage buckets, from AWS S3, or from HTTP/HTTPS sources.

**Tag (Cloud Run)** — A named pointer to a specific Cloud Run revision, accessible via a stable URL prefix (e.g., `canary---my-service-xxx.run.app`). Used for staged traffic rollouts.

**Task (Cloud Run Jobs)** — A single work unit within a Cloud Run Job execution. A job can be configured with multiple tasks that run in parallel (`--tasks`, `--parallelism`).

**Timeout** — The maximum duration a Cloud Run request or Cloud Functions invocation can run before being terminated. Cloud Run supports up to 3,600 seconds; Cloud Functions Gen 2 supports up to 3,600 seconds (60 minutes).

**TLS (Transport Layer Security)** — The cryptographic protocol securing HTTPS connections. Cloud Run and custom domain mappings use Google-managed TLS certificates.

**Topic (Pub/Sub)** — A named resource to which publishers send messages. One topic can have multiple subscriptions, each receiving a copy of every published message.

**Traffic Splitting** — A Cloud Run feature that distributes incoming requests across multiple revisions by percentage (e.g., 90% to stable, 10% to canary). Not available in Cloud Functions Gen 1.

**Trigger (Cloud Functions / Eventarc)** — A configuration that specifies the event type and source that causes a function or Cloud Run service to be invoked (e.g., Pub/Sub message, Cloud Storage object finalized).

**vCPU** — A virtual CPU allocated to a Cloud Run instance or Cloud Functions instance. Cloud Functions Gen 2 supports up to 8 vCPUs per instance; Gen 1 supports up to 2.

**VPC (Virtual Private Cloud)** — GCP's global, software-defined private network. Cloud Run can connect to VPC resources (e.g., Cloud SQL, Memorystore) via a Serverless VPC Access connector.

**VPC Connector** — A Serverless VPC Access resource that provides a network bridge between Cloud Run or Cloud Functions and a VPC network, enabling access to private IP resources.

**WebSocket** — A protocol providing full-duplex communication channels over a single TCP connection. Supported by Cloud Run but not by Cloud Functions, making Cloud Run the appropriate choice for real-time applications.

**Workflows** — GCP's fully managed service for orchestrating sequences of HTTP-based services and GCP APIs. Eventarc can route events to Workflows as a target.

**.NET** — Microsoft's cross-platform development framework. Supported as a Cloud Functions runtime in versions 6 and 8.
