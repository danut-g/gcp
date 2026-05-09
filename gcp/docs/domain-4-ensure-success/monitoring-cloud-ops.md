# Cloud Monitoring: Metrics, Dashboards, Alerting, Uptime Checks

## Overview

**Cloud Monitoring** (part of Google Cloud's operations suite, formerly Stackdriver) provides full-stack monitoring for GCP resources. It collects metrics, allows building dashboards, configures alerting policies, and supports SLO management. Understanding monitoring architecture and alerting configuration is important for the ACE exam.

---

## Key Concepts

### Metrics

#### Built-in GCP Metrics

- GCP services automatically emit metrics to Cloud Monitoring
- Examples:
  - `compute.googleapis.com/instance/cpu/utilization`
  - `compute.googleapis.com/instance/disk/read_ops_count`
  - `cloudsql.googleapis.com/database/cpu/utilization`
  - `kubernetes.io/container/cpu/request_utilization`
- Full metric names follow: `SERVICE/RESOURCE_TYPE/METRIC_NAME`
- Metrics have labels (dimensions) for filtering (e.g., by VM instance name, zone, project)

#### Custom Metrics

- Define your own metrics via the Cloud Monitoring API or OpenTelemetry
- Custom metric type prefix: `custom.googleapis.com/`
- Use cases: Application-level metrics (e.g., queue depth, business KPIs, cache hit rate)
- Custom metrics can be used in alerting policies and dashboards just like built-in metrics

#### Agent-based Metrics

- **Ops Agent**: Google's unified monitoring + logging agent for VMs
  - Replaces older Monitoring Agent and Logging Agent
  - Collects OS-level metrics (memory, disk, network) + application metrics
  - Memory utilization is NOT available from the hypervisor — requires the Ops Agent to be installed
- **Prometheus**: GKE integration for Kubernetes metrics; Google Managed Prometheus (GMP) ingests Prometheus metrics

#### Metric Retention

- Default: 6 weeks retention for standard resolution metrics
- High-resolution metrics: Retained with finer granularity for 24 hours
- Custom metrics: 24 months retention

---

### Workspaces / Scoping Projects

- Cloud Monitoring is scoped to a **project** (or a multi-project workspace)
- A **scoping project** (formerly Workspace) can monitor metrics from multiple GCP projects
- Create a scoping project to centralize monitoring across an organization
- Important: Metrics from a project are only visible if that project is in the scope

---

### Dashboards

- Custom dashboards with widgets: line charts, bar charts, scorecards, text, heatmaps
- **Pre-built dashboards**: Available for GCE, GKE, Cloud SQL, and other services
- Metric charts support multiple metrics, filters, and aggregation (sum, mean, max, percentile)
- Public dashboards: Can be shared via URL without authentication (read-only)

---

### Alerting Policies

An alerting policy defines: **when** to alert, **who** to notify, and **how** to notify.

#### Components

| Component | Description |
|-----------|-------------|
| **Condition** | Metric threshold, absence, or rate-of-change trigger |
| **Notification channel** | Where alerts are sent (email, Pub/Sub, PagerDuty, Slack, SMS, webhook) |
| **Documentation** | Text included in alert notifications |
| **Incident** | Created when a condition is violated; closed when condition is resolved |

#### Condition Types

- **Threshold**: Alert when metric exceeds/falls below a value for a duration
  - Example: CPU > 80% for 5 minutes
- **Absence**: Alert when expected metric data is missing
  - Example: No heartbeat from a VM for 5 minutes (detects failures that produce no data)
- **Metric rate of change**: Alert on sudden changes
- **Forecast**: Alert when a metric is predicted to cross a threshold within a lookback window

#### Alignment Period and Aggregation

- Raw metrics are aligned to a time series before comparison
- **Alignment**: How data points within a period are combined (mean, max, min, sum, count)
- **Cross-series aggregation**: How data across multiple resources is combined before evaluating
  - Example: Alert on average CPU across ALL VMs in a group (aggregate then threshold) vs alert when ANY VM exceeds CPU (threshold then aggregate)
- This ordering matters: "group by then threshold" vs "threshold then group by" produce different behaviors

#### Alert Notification Channels

- Email, mobile push, Pub/Sub (for webhook/automation), PagerDuty, Slack, OpsGenie, webhook
- Channels must be configured and verified before use
- Multiple channels can be set per alerting policy

---

### Uptime Checks

- Synthetic monitoring — GCP sends test requests to your endpoints from multiple global locations
- Types: HTTP(S), TCP, custom
- Check frequency: Every 1, 5, or 10 minutes (min is 1 minute)
- Global probing locations: Americas, Europe, Asia-Pacific
- Can create alerting policies directly from uptime checks
- **Uptime check vs Health check**: Uptime checks are for user-visible endpoint monitoring; Health checks are for load balancers and MIG autohealing
- Uptime checks count as successful if they get any non-5xx response (HTTP) or successful TCP connection
- Can configure expected content in the response body

---

### Service Monitoring and SLOs

- **Service Level Objectives (SLOs)**: Define reliability targets for services
- Cloud Monitoring supports SLO management:
  - Define SLIs (Service Level Indicators): request-based or window-based
  - Set SLO targets (e.g., 99.9% of requests succeed in 7 days)
  - Error budget tracking: Automatically calculated based on SLO and actual performance
  - Create alerts when error budget is burning too fast (burn rate alerts)

---

### Google Managed Prometheus (GMP)

- Fully managed Prometheus-compatible monitoring for GKE workloads
- Scrapes Prometheus metrics from Pods
- Data stored in Cloud Monitoring
- Compatible with Prometheus query language (PromQL)
- Reduces operational burden of running Prometheus infrastructure
- Available in both Autopilot and Standard GKE

---

## When to Use

- **Custom dashboards**: Operational visibility for a specific team or application
- **Alerting policies**: Any metric that needs automated notification when anomalous
- **Uptime checks**: External endpoint availability monitoring
- **Ops Agent**: Install on all GCE VMs for complete observability (memory, disk, application metrics)
- **SLOs**: Formal reliability commitments with error budget tracking

---

## When NOT to Use

- **Cloud Monitoring for log analysis**: Use Cloud Logging for log-based queries and log-based metrics
- **Uptime checks for internal services**: Internal endpoints can't be probed by the global check network; use synthetic monitors from within your VPC or define custom checks from a VM

---

## Related Services / Concepts

- **Cloud Logging**: Logs complement metrics — see [logging.md](logging.md)
- **Managing Compute**: Autoscaling uses metrics — see [managing-compute.md](managing-compute.md)
- **Managing GKE**: GKE-specific monitoring — see [managing-gke.md](managing-gke.md)
- **Billing**: Budget alerts use a different notification system — see [billing.md](../domain-1-setup-and-configure/billing.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Memory metrics require Ops Agent**: GCP cannot measure VM memory utilization from the hypervisor. You must install the Ops Agent on the VM to get memory metrics. A common exam trap: "Why can't I see memory metrics for my VM?" → Answer: Install Ops Agent.

2. **Absence alerting vs threshold alerting**: Use "metric absence" conditions to detect when a service has gone completely silent (no data at all), not when a metric is below a threshold.

3. **Aggregation order matters**: "Alert when the sum across all backends exceeds 1000 req/s" is different from "alert when any single backend exceeds 100 req/s." Both use the same metric but different aggregation.

4. **Uptime checks require public accessibility**: Global GCP probers need to reach your endpoint. Internal services require internal uptime monitoring.

5. **Scoping projects for multi-project monitoring**: In a large organization, create a dedicated monitoring project and add all other projects to its scope. This centralizes dashboards and alerts.

6. **Notification channels must be verified**: Email channels in particular require clicking a verification link before they'll receive alerts.

7. **Alerting policy incident vs alert**: An "incident" is the Cloud Monitoring object created when a condition is triggered. An alert/notification is what gets sent to a channel. One incident can trigger multiple notifications.

8. **GMP vs custom Prometheus**: GMP is the managed, GKE-native option. Running your own Prometheus in GKE is valid but requires more operational work. For the exam, GMP is the recommended approach.

### Keywords
- Cloud Monitoring, metrics, custom metrics, Ops Agent, alerting policy, notification channel, uptime check, absence condition, aggregation period, scoping project, SLO, error budget, Google Managed Prometheus, threshold condition

---

## Source

- https://cloud.google.com/monitoring/docs/overview
- https://cloud.google.com/monitoring/alerts/using-alerting-ui
- https://cloud.google.com/monitoring/uptime-checks
- https://cloud.google.com/stackdriver/docs/managed-prometheus
