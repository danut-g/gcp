# Cloud Monitoring: Metrics, Dashboards, Alerting, Uptime Checks — Dual-Layer Explanation

---

# Cloud Monitoring and Metrics

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A control room with hundreds of gauges and dials showing the current state of every machine in your factory. Some gauges come installed on the machines (built-in metrics); others you add yourself for your own custom processes (custom metrics).

### B. TECHNICAL EXPLANATION
**Cloud Monitoring** (formerly Stackdriver) is GCP's full-stack observability service. It collects time-series metrics from GCP services automatically (built-in metrics) and from user applications (custom metrics). Metrics are identified by metric type strings, carry labels for dimensions, and are stored as time-series data with configurable retention.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
Every GCP service automatically reports readings to the central control room at regular intervals. For VMs, an optional sensor kit (Ops Agent) is installed to report OS-level readings that the factory's central sensors can't see.

### B. TECHNICAL EXPLANATION
GCP services emit metrics to Cloud Monitoring automatically (no configuration needed). Built-in metrics follow the pattern: `SERVICE.googleapis.com/RESOURCE_TYPE/METRIC_NAME`. VM OS-level metrics (memory, disk utilization, process stats) require the **Ops Agent** — the hypervisor cannot observe guest OS memory usage. Custom metrics are written via the Cloud Monitoring API or OpenTelemetry SDK with the prefix `custom.googleapis.com/`.

---

## 3. MENTAL MODEL

### A. ANALOGY
The control room doesn't know what's happening inside a machine's internal circuits unless you install a probe inside it. The built-in gauges only see external behavior.

### B. TECHNICAL EXPLANATION
The key mental model: GCP metrics reflect infrastructure and service state. They do NOT automatically include application-level data (queue depths, business KPIs, cache hit rates) — those require custom metrics. Memory utilization for VMs is the canonical example: GCP can see CPU (external to the VM), but not memory (internal to the guest OS) without the Ops Agent.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
The control room operator builds a custom dashboard view showing only the gauges relevant to their shift: CPU, memory, and disk for the three production machines they're responsible for.

### B. TECHNICAL EXPLANATION
Access metrics via Console → Monitoring → Metrics Explorer. Use the Monitoring Query Language (MQL) or the metric builder UI. Create dashboards with widgets: line charts, scorecards, heatmaps. Pre-built dashboards exist for GCE, GKE, Cloud SQL, and other services. Metrics can be filtered by label values (e.g., zone, instance_name) and aggregated (sum, mean, max, percentile). Retention: standard metrics 6 weeks; custom metrics 24 months.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The central control room aggregates all sensor readings across every factory you own into one view, but you have to tell it which factories to include.

### B. TECHNICAL EXPLANATION
Cloud Monitoring is scoped to a project. A **scoping project** (metrics scope) can include multiple GCP projects — enabling centralized monitoring across an organization. Metrics from a project are only visible within its scoping project's scope. For large organizations: create a dedicated monitoring project and add all production projects to its scope. Google Managed Prometheus (GMP) extends monitoring to Kubernetes workloads using the Prometheus query language (PromQL) with managed infrastructure.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If you only look at the dashboard reading "2% CPU," you might think the machine is idle — but if memory is at 99%, it's about to crash. Without the memory gauge (Ops Agent), you'd never know.

### B. TECHNICAL EXPLANATION
The most dangerous blind spot: not installing the Ops Agent on Compute Engine VMs. Without it: no memory, no disk utilization (beyond capacity), no per-process metrics. Alerts built only on CPU will miss memory-related incidents. Always install the Ops Agent on production VMs.

---

## 7. TRADE-OFFS

### A. ANALOGY
Every additional gauge you add to the control room adds noise. Too many metrics without alerts is just a dashboard that no one reads.

### B. TECHNICAL EXPLANATION
Metrics without alerts are only useful for retroactive debugging. Alert fatigue from too many alerting policies degrades operational response quality. Expert practice: instrument exhaustively (collect all metrics), but alert selectively (only on metrics where human action is required and achievable).

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"All my machines are reporting to the control room automatically, including their internal sensors." No — internal sensors (memory, OS metrics) require the Ops Agent to be installed.

### B. TECHNICAL EXPLANATION
The exam trap: "Why can't I see memory utilization for my VM in Cloud Monitoring?" — Answer: install the Ops Agent. This is consistently tested. Another misconception: metrics are retained indefinitely. Standard metrics are retained for only 6 weeks; custom metrics for 24 months.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Expert operators build a single control room view for the entire organization — not one per factory. They can compare all factories side-by-side and spot anomalies immediately.

### B. TECHNICAL EXPLANATION
Expert practice: create a dedicated scoping project for monitoring at the organization level. Add all projects to the scope. Build organization-wide dashboards. Use resource labels as metric dimensions to filter by team, environment, or application without creating separate dashboards. Use metric-based alerting policies rather than log-based where possible — metrics are cheaper to query at scale.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
A control room for your GCP infrastructure — GCP fills most gauges automatically, but you must install the Ops Agent to see inside VMs.

### B. TECHNICAL SUMMARY
Cloud Monitoring collects time-series metrics from GCP services automatically. VM OS-level metrics (memory, processes) require the Ops Agent. Custom metrics use the `custom.googleapis.com/` prefix via the Monitoring API. Scoping projects enable multi-project centralized monitoring.

---

---

# Alerting Policies

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A shift supervisor who watches all the control room gauges and pages the on-call engineer when any reading exceeds a threshold for a sustained period — not just momentary spikes that self-resolve.

### B. TECHNICAL EXPLANATION
An **alerting policy** in Cloud Monitoring defines: a **condition** (which metric, what threshold, for how long), a **notification channel** (who to notify and how), and documentation (context for the responder). When a condition is violated, Cloud Monitoring creates an **incident** and fires notifications to configured channels.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
The supervisor watches the gauge every minute. If the reading stays above the red line for 5 straight minutes (not just one reading), they page the engineer. One spike that resolves isn't worth waking someone up.

### B. TECHNICAL EXPLANATION
Alerting policy condition types:
- **Threshold**: Fire when metric > (or <) a value for a minimum duration (e.g., CPU > 80% for 5 minutes)
- **Absence**: Fire when expected metric data is missing entirely (e.g., no heartbeat for 5 minutes)
- **Rate of change**: Fire on sudden metric changes
- **Forecast**: Fire when a metric is predicted to cross a threshold within a future window

The **alignment period** and **aggregation** function determine how raw data points are combined before comparison. The **duration** parameter prevents alerts from firing on transient spikes.

---

## 3. MENTAL MODEL

### A. ANALOGY
The difference between "the gauge spiked once" (threshold with short duration) and "the gauge is consistently too high" (threshold with sustained duration) is controlled by the alerting policy's duration setting.

### B. TECHNICAL EXPLANATION
Critical concept: **aggregation order** matters. "Alert when the **average** CPU across all VMs in a group exceeds 80%" is fundamentally different from "alert when **any single** VM's CPU exceeds 80%." The first aggregates across the group first, then evaluates the threshold. The second evaluates the threshold per-VM first. Design conditions carefully to avoid both false positives and missed alerts.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Configure: "Page me (email + PagerDuty) if any production server's memory is above 90% for 10 minutes, and send the alert with a link to the runbook."

### B. TECHNICAL EXPLANATION
Create via Console → Monitoring → Alerting → Create Policy. Select metric → set aggregation → configure condition → add notification channels. Add documentation (markdown) that appears in the notification. A policy can have multiple conditions (OR logic — any condition triggers). Multiple notification channels can be added per policy. Channels must be verified before use (email channels require clicking a verification link).

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The supervisor's notebook tracks when a condition first triggered (incident open), whether it's still triggering (incident ongoing), and when it resolved (incident closed). Each transition sends a notification.

### B. TECHNICAL EXPLANATION
Cloud Monitoring creates an **incident** when a condition first triggers. The incident state is: `open` (condition still true), `acknowledged` (responder is working on it), `resolved` (condition is no longer true). By default, a notification is sent on open and on close. Alert policies support **snooze** to suppress notifications for a period during maintenance.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If all the machines go completely silent (network failure), a threshold alert won't fire — you need a separate "silence detector" to catch that.

### B. TECHNICAL EXPLANATION
A threshold condition can only fire if data is present. If a VM crashes and stops emitting metrics entirely, a CPU threshold alert will NOT fire — there's no data to exceed the threshold. Use **absence conditions** ("no data for 5 minutes") to detect complete service failures. This is a critical design consideration for uptime monitoring.

---

## 7. TRADE-OFFS

### A. ANALOGY
Setting the alarm threshold too low causes constant false alarms (alert fatigue). Too high means you miss real problems until they're severe.

### B. TECHNICAL EXPLANATION
Alert threshold tuning requires understanding the normal operating range of each metric. Too sensitive: too many notifications → alert fatigue → alerts ignored → missed incidents. Too insensitive: real problems go undetected. Use the metric explorer to analyze historical data before setting thresholds. Burn rate alerts for SLOs provide a better approach for reliability targets.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"One policy handles all my alerting needs." No — absence detection requires a separate policy from threshold detection.

### B. TECHNICAL EXPLANATION
Threshold conditions cannot detect metric absence. Absence conditions cannot detect threshold violations. Use both. Also: notification channels must be configured and verified separately — email channels require a verification click before they'll deliver alerts. A common exam trap: "Alerts were configured but never fired when the service went down" → likely the service stopped emitting metrics (use absence condition), or the notification channel wasn't verified.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Experts build alert policies to be actionable: "Page me only when I need to do something right now."

### B. TECHNICAL EXPLANATION
Expert principle: every alert should be actionable. If receiving an alert at 3am wouldn't prompt any action, it shouldn't page — it should create a ticket or be logged. Use multiple severity tiers: PagerDuty for critical, email for warning. Use burn rate alerts for SLOs rather than raw metric thresholds for better reliability signal. Create alert policies via Terraform to ensure consistency and version control.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
A smart supervisor who pages you only when a metric has been broken for a sustained period — but you must also set up a separate alarm for when machines go completely silent.

### B. TECHNICAL SUMMARY
Alerting policies define metric conditions (threshold, absence, rate-of-change) that trigger notifications to configured channels (email, PagerDuty, Pub/Sub, Slack). Threshold conditions cannot detect metric absence — configure separate absence conditions for complete failures. Aggregation order affects which machines trigger the alert.

---

---

# Uptime Checks

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A mystery shopper service that sends test customers to your store from multiple cities around the world every minute. If the store is closed when they arrive, they report back to you immediately.

### B. TECHNICAL EXPLANATION
Uptime checks are synthetic monitoring probes that Cloud Monitoring sends from multiple global GCP locations to your endpoints. They verify that your service is reachable and responding correctly. Failed uptime checks can trigger alerting policies, providing external-perspective availability monitoring.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
Google's test customers (probers) are in North America, Europe, and Asia-Pacific. Every minute (or every 5/10 minutes), each prober visits your store URL. If more than a configurable number of probers can't get in, you're alerted.

### B. TECHNICAL EXPLANATION
Uptime checks are configured with: type (HTTP, HTTPS, TCP), target (URL or IP), check frequency (1, 5, or 10 minutes), expected response content (optional string match), timeout, and geographic locations. GCP sends probes from multiple regions. An uptime check is considered failed when a configurable number of consecutive probes fail from a region. Can create an alerting policy directly from the uptime check configuration.

---

## 3. MENTAL MODEL

### A. ANALOGY
Uptime checks give you the customer's perspective — they tell you if your store front is open, not whether your internal kitchen is running.

### B. TECHNICAL EXPLANATION
Uptime checks test external reachability — the user-visible behavior of your service. They are NOT internal health checks (those are Load Balancer health checks or MIG autohealing health checks). Uptime checks require the endpoint to be publicly reachable. Internal services behind a VPC cannot be probed by Cloud Monitoring's global probers.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Set up a checker: "Verify that https://myapp.com/health returns HTTP 200 every minute from US, Europe, and Asia."

### B. TECHNICAL EXPLANATION
Create via Console → Monitoring → Uptime Checks → Create Uptime Check. Configure: protocol (HTTPS), resource type (URL), hostname, path, check frequency, content match (optional). Create an alerting policy that fires when the check fails from any region. Monitor check history in the Uptime Checks dashboard — shows pass/fail from each global location over time.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
Each prober location acts independently — if your store is inaccessible in Asia but fine in the US, the Asian prober reports a failure while the US prober doesn't.

### B. TECHNICAL EXPLANATION
GCP probers are distributed globally. Uptime check failures from a single location may indicate regional routing issues rather than a complete outage. Configure alert conditions to fire when failures come from multiple locations (to avoid false alerts from a single-region network issue). Uptime check data is stored as a metric: `monitoring.googleapis.com/uptime_check/check_passed`.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
You can't send mystery shoppers to a private back office — they can only test what's publicly accessible.

### B. TECHNICAL EXPLANATION
Uptime checks cannot probe: private IP addresses, services behind a VPC without public access, internal APIs without external exposure. For internal endpoint monitoring: deploy a VM within the VPC that runs a cron job to test internal endpoints and pushes pass/fail as custom metrics. Also: HTTP uptime checks succeed on any non-5xx response — even 4xx (page not found) is counted as success unless you configure content matching.

---

## 7. TRADE-OFFS

### A. ANALOGY
Mystery shoppers confirm the door is open but don't tell you whether the kitchen is functioning, the menu is correct, or the staff is competent.

### B. TECHNICAL EXPLANATION
Uptime checks confirm basic reachability but not application correctness. A 200 OK response on `/health` may not mean the database is functioning correctly. For deeper health verification: implement meaningful health check endpoints in your application that test downstream dependencies before returning 200.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"My uptime check passed, so everything is fine." Not if it's checking a static URL that returns 200 regardless of backend health.

### B. TECHNICAL EXPLANATION
Uptime checks are as useful as the endpoint they probe. A check on `/` that always returns 200 (even when the database is down) gives false confidence. Implement health check endpoints that validate actual application state. Also: uptime checks and load balancer health checks are separate — both should be configured for production services.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Experienced operators place mystery shoppers not just at the front door but at every key user journey entry point — login page, checkout page, API endpoint.

### B. TECHNICAL EXPLANATION
Expert practice: create uptime checks for each critical user-facing endpoint. Implement health check endpoints that test: database connectivity, cache availability, downstream API status. Set check frequency to 1 minute for critical services. Use content matching to verify the response body contains expected data (not just any 200). Alert with PagerDuty integration for immediate response.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Global mystery shoppers testing your endpoint every minute from multiple continents — they tell you if the door is open, not what's happening inside.

### B. TECHNICAL SUMMARY
Uptime checks send synthetic HTTP/HTTPS/TCP probes from multiple global locations at configurable intervals. They verify external reachability of public endpoints and can trigger alerting policies. They cannot probe private/internal services and only verify basic reachability unless combined with content matching.
