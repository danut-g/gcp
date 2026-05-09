# Cloud Run Deployment: Containers, Revisions, Traffic Splitting, Scaling

## Overview

**Cloud Run** is a fully managed serverless platform for running containerized applications. Deployment involves pushing container images and configuring the service's scaling, concurrency, traffic routing, and security settings. Understanding Cloud Run's revision model and its scaling behaviors is essential for the exam.

---

## Key Concepts

### Cloud Run Service vs Cloud Run Job

| Type | Description | Use Case |
|------|-------------|---------|
| **Service** | Long-running HTTP server; handles requests | APIs, web apps, microservices |
| **Job** | Runs containers to completion (not HTTP) | Batch processing, migrations, scripts |

---

### Container Requirements

- Container must listen on the port specified by the `PORT` environment variable (default: 8080)
- Container must start and begin accepting requests within the **startup probe** timeout
- Stateless: No persistent local storage — use external storage (Cloud Storage, Cloud SQL, Firestore)
- Container images stored in **Artifact Registry** (or Container Registry, legacy)

---

### Revisions

- Every deployment creates an immutable **revision**
- A revision captures: container image, environment variables, resource limits, concurrency, scaling settings
- Traffic is routed to revisions (not directly to the service)
- Old revisions persist (and can receive traffic) until explicitly deleted
- Revision names are auto-generated or can be specified: `--revision-suffix`

---

### Traffic Splitting

- Traffic can be distributed across multiple revisions by percentage
- Use cases: **Canary deployments** (e.g., 90% to stable, 10% to new) and **Blue/Green** (100% switch on validation)
- Traffic split is configured on the service, referencing revision names or the `LATEST` tag
- **Tags**: Named pointers to specific revisions; can be used to send traffic to a tagged revision without a percentage split; useful for testing a revision before routing production traffic

```
LATEST → current newest revision
TAG (e.g., "canary") → specific revision
```

---

### Scaling Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| **Min instances** | Minimum instances to keep warm | 0 (scale-to-zero) |
| **Max instances** | Maximum simultaneous instances | 1000 |
| **Concurrency** | Max concurrent requests per instance | 80 |
| **Request timeout** | Max time for a single request | 300s (default), up to 3600s |
| **CPU allocation** | Request-only (default) or Always-on | Request-only |

#### Scale-to-Zero

- By default, Cloud Run scales to zero when there are no requests
- **Cold start**: When traffic arrives at a zero-scaled service, a new instance must start (latency penalty)
- Cold start time depends on container image size, language runtime, and initialization code
- **Minimum instances**: Setting `--min-instances=1` keeps at least one instance warm; eliminates cold starts; costs money even with no traffic

#### Concurrency

- Cloud Run can handle **multiple simultaneous requests per instance** (unlike Cloud Functions gen1)
- Default concurrency: 80 requests/instance
- Increase concurrency when your app is I/O-bound (e.g., async Node.js, Python async, Go)
- Decrease when your app is CPU-bound (set to 1 for CPU-intensive tasks to prevent contention)
- Autoscaler adds new instances when: `(incoming_requests / concurrency_per_instance) > current_instances`

#### CPU Allocation Modes

- **CPU only during request processing** (default): CPU is throttled when not handling a request; background tasks stop; lower cost
- **CPU always allocated**: CPU available continuously; enables background tasks (e.g., background goroutines, timers); costs more (like a regular server)

---

### Cloud Run Networking

#### Ingress Settings

| Setting | Description |
|---------|-------------|
| `all` | Accept traffic from anywhere (internet + internal) |
| `internal` | Only VPC-internal traffic and Pub/Sub/Cloud Tasks |
| `internal-and-cloud-load-balancing` | Internal + requests through Google Cloud LB |

- For private services (internal microservices), set ingress to `internal`
- To put Cloud Run behind an Application Load Balancer with a custom domain and SSL, use `internal-and-cloud-load-balancing`

#### Egress (VPC Connectivity)

- By default, Cloud Run instances cannot access VPC resources (Cloud SQL via private IP, Memorystore, etc.)
- Options for VPC access:
  - **VPC Connector (Serverless VPC Access)**: Create a connector in a subnet; Cloud Run traffic routes through it; limited bandwidth
  - **Direct VPC egress**: Native VPC egress for Cloud Run (newer, higher throughput, no connector needed); specified as `--network` and `--subnet`

#### Accessing Cloud SQL

- Via **public IP + Cloud SQL Auth Proxy** (sidecar container or built-in connection string with IAM auth)
- Via **private IP + VPC connector/Direct VPC egress**
- Best practice: Private IP + Direct VPC egress for production

---

### Authentication and Security

#### Invoker Authentication

- Services can require authentication (`--no-allow-unauthenticated`) — requires callers to present a valid token
- Callers need `roles/run.invoker` on the Cloud Run service
- Public services: `--allow-unauthenticated` grants `allUsers` the invoker role

#### Service Identity

- Each Cloud Run service runs as a GCP service account
- Default: Compute Engine default SA (avoid — has broad permissions)
- Best practice: Create a dedicated SA per service with least-privilege permissions

#### Secret Injection

- Secrets from Secret Manager can be injected as:
  - **Environment variables**: Mounted at startup
  - **Volume mounts**: Files mounted into the container filesystem
- Using Secret Manager requires the service account to have `roles/secretmanager.secretAccessor`

---

### Cloud Run Jobs

- For tasks that run to completion (not HTTP servers)
- Configuration: container image, parallelism, task count, timeout per task
- Execution: Manual trigger, Cloud Scheduler, or Workflows
- Retries: Configurable per-task retry count on failure
- Use cases: Database migrations, batch data processing, report generation

---

### Artifact Registry Integration

- Cloud Run deploys container images from Artifact Registry
- Reference format: `REGION-docker.pkg.dev/PROJECT/REPOSITORY/IMAGE:TAG`
- Cloud Run automatically uses the service account's permissions to pull images
- Grant `roles/artifactregistry.reader` on the repository to the service account

---

## When to Use

- **Scale-to-zero** for low-traffic or sporadic services where cost is prioritized
- **Min instances > 0** for latency-sensitive production services
- **Traffic splitting** for incremental rollouts and A/B testing
- **CPU always-on** for services with background tasks or time-based processing
- **Cloud Run Jobs** for batch workloads that don't need HTTP endpoints
- **Internal ingress** for private microservices

---

## When NOT to Use

- Not for workloads that maintain in-memory state between requests (stateless only)
- Not for workloads needing direct GPU access (use GKE or Compute Engine)
- Not when containers must run continuously beyond HTTP triggers without the "CPU always-on" option
- VPC connector has bandwidth limits (~1 Gbps); use Direct VPC egress for high-throughput VPC connectivity

---

## Related Services / Concepts

- **Cloud Run / Functions Planning**: Service selection framework — see [cloud-run-functions-planning.md](../domain-2-plan-and-configure/cloud-run-functions-planning.md)
- **Cloud Functions Deploy**: Event-driven alternative — see [cloud-functions-deploy.md](cloud-functions-deploy.md)
- **Networking Deploy**: Load balancers in front of Cloud Run — see [networking-deploy.md](networking-deploy.md)
- **Data Security**: Secret Manager integration — see [data-security.md](../domain-5-configure-access-and-security/data-security.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Scale to zero = cold starts**: If `min-instances=0` (default), any request to a zero-instance service incurs a cold start delay. The exam may ask about eliminating cold starts — answer is `min-instances=1`.

2. **CPU throttled when not handling requests**: By default, CPU is only available during request handling. If a service needs to process data in the background (e.g., flush buffers, run timers), enable "CPU always allocated."

3. **Concurrency matters for scaling**: Cloud Run doesn't auto-scale based on CPU alone. It scales based on concurrent requests. If concurrency=1 and you get 100 simultaneous requests, you'll have 100 instances.

4. **Traffic tags ≠ traffic splitting**: Tags are named pointers to revisions for testing purposes. Traffic percentages control actual user traffic routing. Both can be set simultaneously.

5. **VPC connector bandwidth limits**: The Serverless VPC Access connector supports up to ~1 Gbps. For high-throughput scenarios, use Direct VPC egress instead.

6. **Default SA = Compute Engine default SA**: Cloud Run services default to the Compute Engine default SA, which has `Editor` role. Always create a dedicated SA.

7. **`--no-allow-unauthenticated` doesn't encrypt traffic**: It requires authentication tokens but the channel is always HTTPS anyway. The flag controls authorization, not encryption.

8. **Request timeout vs execution timeout**: For Cloud Run Services, it's "request timeout" (max time for one HTTP request). For Cloud Run Jobs, it's "task timeout" (max time for one task execution).

### Keywords
- Revision, traffic splitting, canary, min instances, max instances, concurrency, scale to zero, cold start, CPU allocation, ingress setting, VPC connector, Direct VPC egress, Cloud Run Job, Artifact Registry, invoker role

---

## Source

- https://cloud.google.com/run/docs/deploying
- https://cloud.google.com/run/docs/configuring/request-timeout
- https://cloud.google.com/run/docs/configuring/vpc-direct-vpc
- https://cloud.google.com/run/docs/triggering/https-request
