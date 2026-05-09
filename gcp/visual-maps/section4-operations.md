# Section 4 -- Ensuring Successful Operation of a Cloud Solution -- Visual Maps

---

## 1. Snapshot vs Image Comparison Visual

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    SNAPSHOT vs IMAGE -- SIDE BY SIDE                          │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────┐    ┌─────────────────────────────────┐
  │           SNAPSHOT              │    │            IMAGE                │
  ├─────────────────────────────────┤    ├─────────────────────────────────┤
  │                                 │    │                                 │
  │  Purpose:                       │    │  Purpose:                       │
  │  BACKUP & DISASTER RECOVERY     │    │  CREATE NEW VMs / BOOT DISKS    │
  │                                 │    │                                 │
  │  ┌───────────┐  ┌───────────┐  │    │  ┌───────────────────────────┐  │
  │  │ Boot Disk │  │ Data Disk │  │    │  │      Boot Disk ONLY       │  │
  │  │  (YES)    │  │  (YES)    │  │    │  │                           │  │
  │  └───────────┘  └───────────┘  │    │  └───────────────────────────┘  │
  │  Works on ANY persistent disk   │    │  Only from boot disks           │
  │                                 │    │                                 │
  │  Storage: INCREMENTAL           │    │  Storage: FULL COPY             │
  │  ┌─────┐                        │    │  ┌─────────────────────────┐   │
  │  │Snap1│ = Full copy             │    │  │     Complete image      │   │
  │  └─────┘                        │    │  │     every time          │   │
  │  ┌─────┐                        │    │  └─────────────────────────┘   │
  │  │Snap2│ = Only changes          │    │                                │
  │  └─────┘    since Snap1         │    │  Image Families: YES           │
  │  ┌─────┐                        │    │  ┌─────────────────────────┐   │
  │  │Snap3│ = Only changes          │    │  │ my-app-family           │   │
  │  └─────┘    since Snap2         │    │  │  ├── my-app-v1          │   │
  │                                 │    │  │  ├── my-app-v2          │   │
  │  Families: NO                   │    │  │  └── my-app-v3 (latest) │   │
  │                                 │    │  └─────────────────────────┘   │
  │  Schedule: YES                  │    │                                │
  │  (automated daily/weekly/hourly)│    │  Schedule: NO                  │
  │                                 │    │  (manual creation)             │
  │  Create VM from: SLOWER         │    │  Create VM from: FASTER        │
  │  (restore to disk, then boot)   │    │  (directly boot from image)   │
  │                                 │    │                                │
  │  Cross-region: YES              │    │  Cross-region: YES             │
  │  (restore disk in any zone)     │    │  (create VM in any zone)      │
  └─────────────────────────────────┘    └─────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    WHEN TO USE WHICH?                                         │
 └──────────────────────────────────────────────────────────────────────────────┘

  "I need to back up my database disk before an upgrade"
       ──► SNAPSHOT

  "I need daily automated backups with 14-day retention"
       ──► SNAPSHOT SCHEDULE

  "I need a golden image with my app pre-installed for MIG templates"
       ──► CUSTOM IMAGE (with image family)

  "I need to clone a disk to a different region for DR"
       ──► SNAPSHOT (create disk from snapshot in target region)

  "I need to create 100 identical VMs quickly"
       ──► IMAGE (in instance template)


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    SNAPSHOT LIFECYCLE                                         │
 └──────────────────────────────────────────────────────────────────────────────┘

                     Create                   Use                    Delete
  ┌──────────┐      ┌──────────┐      ┌──────────────────┐      ┌──────────┐
  │  Disk    │─────►│ Snapshot │─────►│ New Disk (any    │      │ Snapshot │
  │ (source) │      │ (stored  │      │ zone/region)     │      │ deleted  │
  │          │      │  in GCS) │      └──────────────────┘      │          │
  └──────────┘      └──────────┘                                │ Data auto│
                         │            ┌──────────────────┐      │ merged   │
                         └───────────►│ New Image        │      │ into next│
                                      │ (for templates)  │      │ snapshot │
                                      └──────────────────┘      └──────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    SNAPSHOT SCHEDULE SETUP                                    │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌────────────────────┐     attach      ┌──────────────────┐
  │  Snapshot Schedule │────────────────►│  Persistent Disk │
  │                    │                 │                  │
  │  - daily @ 02:00   │                 │  Snapshots auto- │
  │  - retain 14 days  │                 │  created per     │
  │  - location: US    │                 │  schedule        │
  └────────────────────┘                 └──────────────────┘
                                                  │
                                                  ▼
                              Day 1: snap-1 (full)
                              Day 2: snap-2 (incremental)
                              Day 3: snap-3 (incremental)
                              ...
                              Day 15: snap-1 auto-deleted (>14 days)
```

---

## 2. GKE Autoscaling Hierarchy

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    GKE AUTOSCALING HIERARCHY                                 │
 │             HPA ──► VPA ──► Cluster Autoscaler                               │
 └──────────────────────────────────────────────────────────────────────────────┘


  LEVEL 1: HPA (Horizontal Pod Autoscaler)
  ════════════════════════════════════════════════
  Scales: NUMBER OF POD REPLICAS
  Based on: CPU, memory, custom metrics

  Before HPA:                        After HPA (high load):
  ┌──────────────────────┐           ┌──────────────────────────────────────┐
  │  Deployment: my-app  │           │  Deployment: my-app                  │
  │  ┌─────┐ ┌─────┐    │           │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
  │  │Pod 1│ │Pod 2│    │    ──►    │  │Pod 1│ │Pod 2│ │Pod 3│ │Pod 4│  │
  │  └─────┘ └─────┘    │           │  └─────┘ └─────┘ └─────┘ └─────┘  │
  │  replicas: 2         │           │  replicas: 4 (scaled up)           │
  └──────────────────────┘           └──────────────────────────────────────┘

  Config: min=2, max=10, target CPU=70%


  LEVEL 2: VPA (Vertical Pod Autoscaler)
  ════════════════════════════════════════════════
  Scales: CPU/MEMORY REQUESTS PER POD
  Based on: Historical resource usage

  Before VPA:                        After VPA:
  ┌────────────────────┐             ┌────────────────────┐
  │  Pod                │             │  Pod                │
  │  ┌──────────────┐  │             │  ┌──────────────┐  │
  │  │ CPU: 250m    │  │             │  │ CPU: 500m    │  │  Right-sized!
  │  │ Memory: 256Mi│  │    ──►      │  │ Memory: 512Mi│  │
  │  │ (too small!) │  │             │  │ (optimal)    │  │
  │  └──────────────┘  │             │  └──────────────┘  │
  └────────────────────┘             └────────────────────┘

  VPA Modes:
  ┌──────────┬──────────────────────────────────────────────┐
  │ Off      │ Recommendations only (no changes)            │
  │ Initial  │ Set resources only at pod creation           │
  │ Recreate │ Evict + recreate pods with new resources     │
  │ Auto     │ Like Recreate (in-place when available)      │
  └──────────┴──────────────────────────────────────────────┘


  LEVEL 3: CLUSTER AUTOSCALER
  ════════════════════════════════════════════════
  Scales: NUMBER OF NODES in a node pool
  Based on: Pending (unschedulable) pods

  Before:                            After:
  ┌──────────┐ ┌──────────┐         ┌──────────┐ ┌──────────┐ ┌──────────┐
  │  Node 1  │ │  Node 2  │         │  Node 1  │ │  Node 2  │ │  Node 3  │
  │ ┌──────┐ │ │ ┌──────┐ │         │ ┌──────┐ │ │ ┌──────┐ │ │ ┌──────┐ │
  │ │ Pod  │ │ │ │ Pod  │ │         │ │ Pod  │ │ │ │ Pod  │ │ │ │ Pod  │ │
  │ │ Pod  │ │ │ │ Pod  │ │  ──►   │ │ Pod  │ │ │ │ Pod  │ │ │ │ Pod  │ │
  │ │ Pod  │ │ │ │ Pod  │ │         │ │ Pod  │ │ │ │ Pod  │ │ │ (new)  │ │
  │ │ FULL │ │ │ │ FULL │ │         │ │      │ │ │ │      │ │ │        │ │
  │ └──────┘ │ │ └──────┘ │         │ └──────┘ │ │ └──────┘ │ │ └──────┘ │
  └──────────┘ └──────────┘         └──────────┘ └──────────┘ └──────────┘
  Pending pods! (no room)           New node added, pods scheduled


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    HOW ALL THREE WORK TOGETHER                               │
 └──────────────────────────────────────────────────────────────────────────────┘

  Traffic INCREASES:
  ─────────────────────────────────────────────────────────────────────────────

  Step 1:  Traffic spike detected
              │
              ▼
  Step 2:  HPA sees CPU > 70%
              │
              ▼
  Step 3:  HPA creates more pod replicas (2 ──► 6)
              │
              ▼
  Step 4:  New pods are PENDING (not enough node capacity)
              │
              ▼
  Step 5:  Cluster Autoscaler detects pending pods
              │
              ▼
  Step 6:  Cluster Autoscaler ADDS new nodes (2 ──► 4)
              │
              ▼
  Step 7:  Pending pods get scheduled on new nodes


  Traffic DECREASES:
  ─────────────────────────────────────────────────────────────────────────────

  Step 1:  Traffic drops
              │
              ▼
  Step 2:  HPA sees CPU < 70%
              │
              ▼
  Step 3:  HPA reduces pod replicas (6 ──► 2)
              │
              ▼
  Step 4:  Nodes become underutilized
              │
              ▼
  Step 5:  Cluster Autoscaler waits (cool-down period)
              │
              ▼
  Step 6:  Cluster Autoscaler REMOVES underutilized nodes (4 ──► 2)


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    AUTOSCALER COMPARISON                                      │
 ├──────────────┬──────────────────────┬────────────────────────────────────────┤
 │  Autoscaler  │  What It Scales      │  Trigger Signal                       │
 ├──────────────┼──────────────────────┼────────────────────────────────────────┤
 │  HPA         │  Pod replica count   │  CPU, memory, custom metrics          │
 │  VPA         │  Pod CPU/mem request │  Historical resource usage            │
 │  Cluster AS  │  Node count          │  Pending (unschedulable) pods         │
 └──────────────┴──────────────────────┴────────────────────────────────────────┘

  NOTE: Do NOT use HPA and VPA on the same metric (e.g., both on CPU).
        They will conflict. Use HPA for scaling out, VPA for right-sizing.
```

---

## 3. Cloud Run Traffic Splitting Visual

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD RUN TRAFFIC SPLITTING                               │
 └──────────────────────────────────────────────────────────────────────────────┘


  SCENARIO: Canary Deployment (test new version safely)
  ══════════════════════════════════════════════════════════

    Users send requests to: https://my-service-xxx.a.run.app
                                    │
                                    ▼
                          ┌──────────────────┐
                          │  Traffic Router   │
                          │  (Google-managed) │
                          └────────┬─────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │ 90%                    10%  │
                    ▼                             ▼
          ┌──────────────────┐          ┌──────────────────┐
          │  REVISION v2     │          │  REVISION v3     │
          │  (stable)        │          │  (canary)        │
          │                  │          │                  │
          │  ┌────────────┐  │          │  ┌────────────┐  │
          │  │ Instance 1 │  │          │  │ Instance 1 │  │
          │  │ Instance 2 │  │          │  └────────────┘  │
          │  │ Instance 3 │  │          │                  │
          │  └────────────┘  │          │  Tag URL:        │
          └──────────────────┘          │  canary---my-    │
                                        │  service-xxx     │
                                        │  .a.run.app      │
                                        └──────────────────┘


  GRADUAL ROLLOUT TIMELINE:
  ══════════════════════════════════════════════════════════

  Phase 1:  v2 ████████████████████████████████████████████████  95%
            v3 ███                                                5%

  Phase 2:  v2 ████████████████████████████████████████          80%
            v3 ██████████                                        20%

  Phase 3:  v2 █████████████████████████                         50%
            v3 █████████████████████████                         50%

  Phase 4:  v2                                                    0%
            v3 ██████████████████████████████████████████████████ 100%

  Commands:
  ┌─────────────────────────────────────────────────────────────────────┐
  │ # Step 1: Deploy without traffic                                    │
  │ gcloud run deploy my-service --image=v3 --no-traffic                │
  │                                                                     │
  │ # Step 2: Send 5% to canary                                         │
  │ gcloud run services update-traffic --to-revisions=v2=95,v3=5        │
  │                                                                     │
  │ # Step 3: Monitor metrics, increase gradually                       │
  │ gcloud run services update-traffic --to-revisions=v2=50,v3=50       │
  │                                                                     │
  │ # Step 4: Complete rollout                                           │
  │ gcloud run services update-traffic --to-revisions=v3=100            │
  │                                                                     │
  │ # ROLLBACK (if issues detected at any step):                        │
  │ gcloud run services update-traffic --to-revisions=v2=100            │
  └─────────────────────────────────────────────────────────────────────┘


  TAGGED REVISIONS (testing without affecting production traffic):
  ══════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────────────────┐
  │                                                                     │
  │  Main URL: my-service-xxx.a.run.app ──────► v2 (100% production)   │
  │                                                                     │
  │  Tag URL:  canary---my-service-xxx.a.run.app ──► v3 (testing only) │
  │                                                                     │
  │  The tag URL lets you test v3 WITHOUT any production traffic        │
  │  going to it. Only explicit requests to the tag URL reach v3.      │
  └─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Log Routing Architecture

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD LOGGING -- LOG ROUTING ARCHITECTURE                  │
 └──────────────────────────────────────────────────────────────────────────────┘


  LOG SOURCES
  ┌──────────────────────────────────────────────────────────────────────┐
  │ Compute Engine │ GKE │ Cloud Run │ Cloud SQL │ Cloud Functions │ ... │
  └────────────────────────────────┬─────────────────────────────────────┘
                                   │
                                   │ All logs flow into
                                   ▼
                     ┌──────────────────────────┐
                     │                          │
                     │       LOG ROUTER         │
                     │  (processes every entry)  │
                     │                          │
                     └────────────┬─────────────┘
                                  │
          ┌───────────────────────┼───────────────────────────┐
          │                       │                           │
          │    Inclusion          │    Exclusion              │
          │    Filters            │    Filters                │
          │    (route matching    │    (drop matching         │
          │     logs to sinks)    │     logs entirely)        │
          │                       │                           │
          ▼                       ▼                           ▼
  ┌───────────────┐   ┌───────────────┐            ┌──────────────┐
  │ _Required     │   │ _Default      │            │  EXCLUDED    │
  │ Bucket        │   │ Bucket        │            │  (dropped)   │
  │               │   │               │            │              │
  │ Admin Activity│   │ All other     │            │  e.g. DEBUG  │
  │ System Events │   │ logs          │            │  severity    │
  │               │   │               │            │  logs        │
  │ 400 days      │   │ 30 days       │            └──────────────┘
  │ (fixed)       │   │ (configurable)│
  │ CANNOT        │   │               │
  │ disable       │   │               │
  └───────────────┘   └───────────────┘
                              │
                              │  Additional routing via SINKS:
                              │
          ┌───────────────────┼────────────────────────┐
          │                   │                        │
          ▼                   ▼                        ▼
  ┌───────────────┐   ┌───────────────┐    ┌───────────────────┐
  │ Cloud Storage │   │   BigQuery    │    │     Pub/Sub       │
  │               │   │               │    │                   │
  │ Long-term     │   │ SQL analysis  │    │ Real-time         │
  │ archival      │   │ & dashboards  │    │ processing        │
  │ Compliance    │   │ Reporting     │    │ Forward to        │
  │               │   │               │    │ SIEM / on-prem    │
  │ Cheapest      │   │ Queryable     │    │ Third-party tools │
  └───────────────┘   └───────────────┘    └───────────────────┘
                              │
                              ▼
                      ┌───────────────┐
                      │ Custom Log    │
                      │ Bucket        │
                      │               │
                      │ Configurable  │
                      │ retention     │
                      │ (1-3650 days) │
                      │ Log Analytics │
                      │ (SQL queries) │
                      └───────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    SINK SETUP PROCESS                                         │
 └──────────────────────────────────────────────────────────────────────────────┘

  Step 1: Create the sink
  ┌──────────────────────────────────────────────────────────────────────┐
  │ gcloud logging sinks create my-bq-sink \                            │
  │   bigquery.googleapis.com/projects/PROJ/datasets/logs_dataset \     │
  │   --log-filter='resource.type="gce_instance"'                       │
  └──────────────────────────────────────────────────────────────────────┘
         │
         ▼
  Step 2: Get the writer identity (service account)
  ┌──────────────────────────────────────────────────────────────────────┐
  │ gcloud logging sinks describe my-bq-sink \                          │
  │   --format="get(writerIdentity)"                                    │
  │                                                                     │
  │ Output: serviceAccount:p123-456@gcp-sa-logging.iam.gserviceaccount  │
  └──────────────────────────────────────────────────────────────────────┘
         │
         ▼
  Step 3: Grant the writer identity access to the destination
  ┌──────────────────────────────────────────────────────────────────────┐
  │ gcloud projects add-iam-policy-binding PROJECT_ID \                 │
  │   --member="WRITER_IDENTITY" \                                      │
  │   --role="roles/bigquery.dataEditor"                                │
  └──────────────────────────────────────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    LOG BUCKET COMPARISON                                      │
 ├──────────────────┬────────────────┬──────────────────────────────────────────┤
 │     Bucket       │   Retention    │   Notes                                 │
 ├──────────────────┼────────────────┼──────────────────────────────────────────┤
 │ _Required        │ 400 days       │ Always on, cannot disable, free         │
 │                  │ (fixed)        │ Admin Activity + System Events           │
 ├──────────────────┼────────────────┼──────────────────────────────────────────┤
 │ _Default         │ 30 days        │ All other logs, configurable retention  │
 │                  │ (configurable) │ Can add exclusion filters               │
 ├──────────────────┼────────────────┼──────────────────────────────────────────┤
 │ Custom           │ 1-3650 days    │ User-created, Log Analytics support     │
 │                  │ (configurable) │ Can be locked (prevent deletion)        │
 └──────────────────┴────────────────┴──────────────────────────────────────────┘
```

---

## 5. Monitoring Alert Pipeline Flow

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD MONITORING -- ALERT PIPELINE                         │
 └──────────────────────────────────────────────────────────────────────────────┘


  METRIC SOURCES                     ALERT EVALUATION               NOTIFICATION
  ┌──────────────────┐
  │ Built-in Metrics │
  │ ┌──────────────┐ │     ┌───────────────────────┐     ┌───────────────────┐
  │ │ CPU util     │ │────►│                       │     │ Notification      │
  │ │ Memory       │ │     │   ALERT POLICY        │     │ Channels:         │
  │ │ Disk I/O     │ │     │                       │     │                   │
  │ │ Network      │ │     │ Condition:             │     │ ┌───────────────┐ │
  │ │ Request count│ │     │  metric > threshold    │────►│ │ Email         │ │
  │ │ Latency      │ │     │  for duration          │     │ │ SMS           │ │
  │ └──────────────┘ │     │                       │     │ │ Slack         │ │
  └──────────────────┘     │ Example:               │     │ │ PagerDuty    │ │
                           │  CPU > 80%             │     │ │ Webhook      │ │
  ┌──────────────────┐     │  for 5 minutes         │     │ │ Pub/Sub      │ │
  │ Custom Metrics   │     │                       │     │ └───────────────┘ │
  │ ┌──────────────┐ │     │ Combiner:             │     │                   │
  │ │ App-specific │ │────►│  AND / OR             │     │ Documentation:    │
  │ │ queue depth  │ │     │                       │     │  Runbook link     │
  │ │ business KPI │ │     └───────────────────────┘     │  Troubleshooting  │
  │ └──────────────┘ │              │                     │  steps            │
  └──────────────────┘              │                     └───────────────────┘
                                    │
  ┌──────────────────┐              │
  │ Log-Based Metrics│              ▼
  │ ┌──────────────┐ │     ┌───────────────────────┐
  │ │ Error count  │ │────►│   ALERT STATES        │
  │ │ 500 errors   │ │     │                       │
  │ │ Custom filter│ │     │  ┌─── OK ◄──────────┐ │
  │ └──────────────┘ │     │  │                   │ │
  └──────────────────┘     │  │   Condition       │ │
                           │  │   not met         │ │
  ┌──────────────────┐     │  │                   │ │
  │ Uptime Checks    │     │  ▼                   │ │
  │ ┌──────────────┐ │     │ FIRING ──────────────┘ │
  │ │ HTTP check   │ │────►│  (sends notification)  │
  │ │ TCP check    │ │     │                       │
  │ │ every 60s    │ │     │ Optional:             │
  │ └──────────────┘ │     │  Snooze / Silence     │
  └──────────────────┘     └───────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    COMPLETE ALERT EXAMPLE                                     │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────────────┐
  │ Alert: "High CPU on Production VMs"                                 │
  │                                                                     │
  │ Metric:    compute.googleapis.com/instance/cpu/utilization           │
  │ Filter:    resource.type = "gce_instance"                           │
  │ Threshold: > 0.8 (80%)                                              │
  │ Duration:  300 seconds (5 minutes)                                  │
  │ Alignment: 60s, ALIGN_MEAN                                          │
  │                                                                     │
  │ Notification: Email to admin@example.com + Slack #ops-alerts        │
  │ Documentation: "CPU above 80%. Check for runaway processes."        │
  └─────────────────────────────────────────────────────────────────────┘

  Timeline:
                                    threshold = 80%
  CPU %                          ┌─────────────────────
  100 ┤                         ╱│
   90 ┤                  ┌─────╱ │
   80 ┤─ ─ ─ ─ ─ ─ ─ ─┌┘─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ (threshold)
   70 ┤             ┌──┘        │
   60 ┤         ┌──┘            │
   50 ┤─────────┘               │
      └────────────────────────────────────────────────► Time
                          │         │
                          │         │
                     Above 80%   5 min elapsed
                     detected    ──► ALERT FIRES
                                     notification sent


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    OPS AGENT ARCHITECTURE                                     │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌─────────── VM Instance ────────────────────────────────┐
  │                                                         │
  │  ┌─────────────────────────────────────────────────┐   │
  │  │              OPS AGENT                           │   │
  │  │                                                  │   │
  │  │  ┌───────────────────┐  ┌─────────────────────┐ │   │
  │  │  │    Fluent Bit     │  │   OpenTelemetry     │ │   │
  │  │  │    (LOGS)         │  │   Collector          │ │   │
  │  │  │                   │  │   (METRICS)          │ │   │
  │  │  │  System logs      │  │  CPU, memory, disk   │ │   │
  │  │  │  App logs (nginx) │  │  Network             │ │   │
  │  │  │  Custom logs      │  │  3rd-party (MySQL,   │ │   │
  │  │  │                   │  │   nginx, Redis...)   │ │   │
  │  │  └─────────┬─────────┘  └──────────┬──────────┘ │   │
  │  └────────────┼───────────────────────┼────────────┘   │
  │               │                       │                 │
  └───────────────┼───────────────────────┼─────────────────┘
                  │                       │
                  ▼                       ▼
          ┌──────────────┐       ┌──────────────┐
          │Cloud Logging │       │  Cloud       │
          │              │       │  Monitoring  │
          └──────────────┘       └──────────────┘
```

---

## 6. Backup and Restore Decision Tree

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    BACKUP & RESTORE DECISION TREE                             │
 └──────────────────────────────────────────────────────────────────────────────┘


                    What service needs backup?
                              │
          ┌───────────────────┼───────────────────────┐
          │                   │                       │
          ▼                   ▼                       ▼
  ┌──────────────┐   ┌──────────────┐       ┌──────────────────┐
  │ Compute      │   │  Cloud SQL   │       │  Firestore       │
  │ Engine       │   │              │       │                  │
  │ (Disks)      │   │              │       │                  │
  └──────┬───────┘   └──────┬───────┘       └──────┬───────────┘
         │                  │                       │
         ▼                  ▼                       ▼
  ┌──────────────┐   Need point-in-time      ┌──────────────────┐
  │ Use SNAPSHOT │   recovery?                │ Use EXPORT to    │
  │              │        │                   │ Cloud Storage    │
  │ Automated?   │   ┌────┴────┐              │                  │
  │     │        │   │         │              │ gcloud firestore │
  │ ┌───┴───┐   │  YES       NO              │ export gs://...  │
  │ │       │   │   │         │              │                  │
  │YES     NO   │   ▼         ▼              │ Automate with:   │
  │ │       │   │ ┌────────┐ ┌────────┐      │ Cloud Scheduler  │
  │ ▼       ▼   │ │Enable  │ │On-demand│     │ + Cloud Function │
  │Snap   Manual│ │backups │ │ backup  │     │                  │
  │Schedule snap│ │+ bin   │ │only     │     │ Restore:         │
  │       shot  │ │logging │ └────────┘      │ gcloud firestore │
  │             │ │(PITR)  │                  │ import gs://...  │
  └──────────────┘ └────────┘                 └──────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD SQL BACKUP TYPES                                     │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────────────────┐
  │                                                                         │
  │  TYPE 1: AUTOMATED BACKUPS                                              │
  │  ═══════════════════════                                                │
  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐            │
  │  │ Mon │ │ Tue │ │ Wed │ │ Thu │ │ Fri │ │ Sat │ │ Sun │            │
  │  │02:00│ │02:00│ │02:00│ │02:00│ │02:00│ │02:00│ │02:00│            │
  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘            │
  │  --backup-start-time=02:00                                              │
  │  --retained-backups-count=7                                             │
  │                                                                         │
  │  TYPE 2: ON-DEMAND BACKUP                                               │
  │  ═══════════════════════                                                │
  │  gcloud sql backups create --instance=my-instance                       │
  │  (manual, any time, for pre-upgrade safety)                             │
  │                                                                         │
  │  TYPE 3: POINT-IN-TIME RECOVERY (PITR)                                  │
  │  ═══════════════════════════════════                                    │
  │  ┌──────────────────────────────────────────────┐                       │
  │  │ ██████████████████████████████████████████████│                      │
  │  │ Binary Log / WAL (continuous recording)      │                       │
  │  └──────────────────────────────────────────────┘                       │
  │          ▲                                                              │
  │          │                                                              │
  │     Can restore to ANY SECOND within retention window                   │
  │     gcloud sql instances clone my-inst restored \                       │
  │       --point-in-time="2024-06-15T10:30:00.000Z"                       │
  │                                                                         │
  │     Requires: --enable-bin-log (MySQL) or WAL archiving (PostgreSQL)    │
  └─────────────────────────────────────────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    RESTORE PATHS                                              │
 └──────────────────────────────────────────────────────────────────────────────┘

  COMPUTE ENGINE DISK RESTORE:
  ┌───────────┐     ┌────────────────┐     ┌────────────────┐     ┌──────────┐
  │ Snapshot  │────►│ Create new     │────►│ Attach to VM   │────►│ Mount &  │
  │           │     │ disk from      │     │ (same or       │     │ verify   │
  │           │     │ snapshot       │     │  different zone)│     │          │
  └───────────┘     └────────────────┘     └────────────────┘     └──────────┘
                    Can be any zone!
                    (cross-region DR)

  CLOUD SQL RESTORE:
  ┌───────────┐     ┌────────────────────────────────┐
  │ Backup    │────►│ Restore to SAME instance       │ (overwrites current data)
  │           │     │ gcloud sql backups restore ID  │
  │           │     │   --restore-instance=same-inst │
  │           │     └────────────────────────────────┘
  │           │
  │           │     ┌────────────────────────────────┐
  │           │────►│ Restore to DIFFERENT instance  │ (safer, can verify first)
  │           │     │ gcloud sql backups restore ID  │
  │           │     │   --restore-instance=new-inst  │
  │           │     │   --backup-instance=orig-inst  │
  │           │     └────────────────────────────────┘
  └───────────┘

  FIRESTORE RESTORE:
  ┌──────────────────┐     ┌─────────────────────────────────────┐
  │ Cloud Storage    │────►│ gcloud firestore import gs://...    │
  │ Export           │     │                                     │
  │ (gs://bucket/    │     │ Can import specific collections:    │
  │  firestore-      │     │ --collection-ids=users,orders       │
  │  backup)         │     │                                     │
  └──────────────────┘     └─────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    BACKUP STRATEGY COMPARISON                                 │
 ├──────────────────┬────────────────┬──────────────┬──────────────────────────┤
 │    Service       │ Backup Method  │ Automated?   │ Point-in-Time Recovery?  │
 ├──────────────────┼────────────────┼──────────────┼──────────────────────────┤
 │ Compute Engine   │ Snapshots      │ Yes          │ No (use snapshot         │
 │ (disks)          │ (incremental)  │ (schedules)  │  closest to time)       │
 ├──────────────────┼────────────────┼──────────────┼──────────────────────────┤
 │ Cloud SQL        │ Built-in       │ Yes          │ Yes (binary log / WAL)  │
 │                  │ backups        │ (daily)      │                          │
 ├──────────────────┼────────────────┼──────────────┼──────────────────────────┤
 │ Firestore        │ Export to GCS  │ No (DIY      │ No (restore from        │
 │                  │                │  Scheduler)  │  export only)           │
 ├──────────────────┼────────────────┼──────────────┼──────────────────────────┤
 │ Spanner          │ Built-in       │ Yes          │ Yes                      │
 │                  │ backups        │              │                          │
 ├──────────────────┼────────────────┼──────────────┼──────────────────────────┤
 │ AlloyDB          │ Built-in       │ Yes          │ Yes (continuous)         │
 │                  │ backups        │              │                          │
 └──────────────────┴────────────────┴──────────────┴──────────────────────────┘
```

---

## 7. Audit Log Types Visual

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    AUDIT LOG TYPES                                            │
 └──────────────────────────────────────────────────────────────────────────────┘


  ┌───────────────────────────────────────────────────────────────────────────┐
  │  ADMIN ACTIVITY LOGS                                                      │
  │  ══════════════════                                                       │
  │  What: API calls that MODIFY resources (create, delete, update, patch)    │
  │  Enabled: ALWAYS ON (cannot be disabled)                                  │
  │  Cost: FREE                                                               │
  │  Retention: 400 days (_Required bucket)                                   │
  │  Example: Someone deletes a VM, changes IAM policy, creates a bucket     │
  │                                                                           │
  │  ┌───────┐  delete VM   ┌────────────┐  logged in  ┌────────────────┐    │
  │  │ User  │─────────────►│ GCP API    │────────────►│ Admin Activity │    │
  │  └───────┘              └────────────┘              │ Audit Log      │    │
  │                                                     └────────────────┘    │
  └───────────────────────────────────────────────────────────────────────────┘

  ┌───────────────────────────────────────────────────────────────────────────┐
  │  DATA ACCESS LOGS                                                         │
  │  ═════════════════                                                        │
  │  What: API calls that READ data or metadata                               │
  │  Enabled: MUST enable manually (except BigQuery: always on)               │
  │  Cost: CHARGED (can generate high volume)                                 │
  │  Retention: 30 days (_Default bucket) unless routed elsewhere             │
  │  Example: Someone reads a file in Cloud Storage, queries BigQuery         │
  │                                                                           │
  │  Types: ADMIN_READ, DATA_READ, DATA_WRITE                                │
  └───────────────────────────────────────────────────────────────────────────┘

  ┌───────────────────────────────────────────────────────────────────────────┐
  │  SYSTEM EVENT LOGS                                                        │
  │  ══════════════════                                                       │
  │  What: Google-initiated system actions                                    │
  │  Enabled: ALWAYS ON (cannot be disabled)                                  │
  │  Cost: FREE                                                               │
  │  Example: Live migration, auto-scaling, auto-repair events               │
  └───────────────────────────────────────────────────────────────────────────┘

  ┌───────────────────────────────────────────────────────────────────────────┐
  │  POLICY DENIED LOGS                                                       │
  │  ═══════════════════                                                      │
  │  What: Security policy violations (denied API calls)                      │
  │  Enabled: ALWAYS ON                                                       │
  │  Cost: FREE                                                               │
  │  Example: User tries to access a resource without permission              │
  └───────────────────────────────────────────────────────────────────────────┘


  SUMMARY TABLE:
  ┌────────────────────┬────────────────┬──────────┬─────────┐
  │ Log Type           │ Default State  │ Cost     │ Disable?│
  ├────────────────────┼────────────────┼──────────┼─────────┤
  │ Admin Activity     │ Always ON      │ Free     │ No      │
  │ Data Access        │ OFF (enable)   │ Charged  │ Yes     │
  │ System Event       │ Always ON      │ Free     │ No      │
  │ Policy Denied      │ Always ON      │ Free     │ No      │
  └────────────────────┴────────────────┴──────────┴─────────┘
```

---

## 8. Cloud Storage Lifecycle Management

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD STORAGE LIFECYCLE FLOW                               │
 └──────────────────────────────────────────────────────────────────────────────┘

  Object created
       │
       ▼
  ┌──────────────┐   30 days   ┌──────────────┐   90 days   ┌──────────────┐
  │   STANDARD   │────────────►│   NEARLINE   │────────────►│   COLDLINE   │
  │              │             │              │             │              │
  │  Frequent    │             │  Monthly     │             │  Quarterly   │
  │  access      │             │  access      │             │  access      │
  │  $/GB: $$$   │             │  $/GB: $$    │             │  $/GB: $     │
  │  Retrieval:  │             │  Retrieval:  │             │  Retrieval:  │
  │  Free        │             │  $/GB        │             │  $$/GB       │
  │  Min storage:│             │  Min storage:│             │  Min storage:│
  │  None        │             │  30 days     │             │  90 days     │
  └──────────────┘             └──────────────┘             └──────┬───────┘
                                                                   │
                                                              365 days
                                                                   │
                                                                   ▼
                                                            ┌──────────────┐
                                                            │   ARCHIVE    │
                                                            │              │
                                                            │  Yearly      │
                                                            │  access      │
                                                            │  $/GB: ~0    │
                                                            │  Retrieval:  │
                                                            │  $$$/GB      │
                                                            │  Min storage:│
                                                            │  365 days    │
                                                            └──────┬───────┘
                                                                   │
                                                              730 days
                                                                   │
                                                                   ▼
                                                            ┌──────────────┐
                                                            │   DELETED    │
                                                            └──────────────┘


  Lifecycle Rule JSON:
  ┌─────────────────────────────────────────────────────────────────────┐
  │ age: 30  ──► SetStorageClass: NEARLINE   (if STANDARD)             │
  │ age: 90  ──► SetStorageClass: COLDLINE   (if NEARLINE)             │
  │ age: 365 ──► SetStorageClass: ARCHIVE    (if COLDLINE)             │
  │ age: 730 ──► Delete                                                 │
  │ isLive: false, numNewerVersions: 3 ──► Delete old versions         │
  └─────────────────────────────────────────────────────────────────────┘
```

---

## 9. Managed Prometheus on GKE

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    MANAGED PROMETHEUS ON GKE                                  │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌─── GKE Cluster ─────────────────────────────────────────────────────────┐
  │                                                                         │
  │  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐               │
  │  │  App Pod 1   │   │  App Pod 2   │   │  App Pod 3   │               │
  │  │  /metrics    │   │  /metrics    │   │  /metrics    │               │
  │  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘               │
  │         │                  │                   │                        │
  │         └──────────────────┼───────────────────┘                        │
  │                            │                                            │
  │                            ▼                                            │
  │              ┌──────────────────────────┐                               │
  │              │ PodMonitoring Resource   │   Defines scraping config:    │
  │              │                          │   - which pods (labels)       │
  │              │ selector:                │   - which port                │
  │              │   app: my-app            │   - interval: 30s            │
  │              │ endpoints:               │   - path: /metrics           │
  │              │   port: metrics          │                               │
  │              └────────────┬─────────────┘                               │
  │                           │                                             │
  │                           ▼                                             │
  │              ┌──────────────────────────┐                               │
  │              │ Managed Collection Agent │  (auto-deployed by GKE)       │
  │              │ (runs on each node)      │                               │
  │              └────────────┬─────────────┘                               │
  │                           │                                             │
  └───────────────────────────┼─────────────────────────────────────────────┘
                              │
                              ▼
                 ┌──────────────────────────┐
                 │  Google Cloud Monarch    │
                 │  (long-term storage)     │
                 │                          │
                 │  Query with:             │
                 │  - PromQL               │
                 │  - Cloud Monitoring     │
                 │  - Grafana              │
                 └──────────────────────────┘
```

---

## 10. Networking Operations Summary

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    NETWORKING OPERATIONS -- KEY TASKS                         │
 └──────────────────────────────────────────────────────────────────────────────┘


  EXPANDING A SUBNET:
  ┌─────────────────────────────────────────────────────────────────────┐
  │                                                                     │
  │  Before: 10.0.1.0/24  (252 usable IPs)                             │
  │  ├────────────────────┤                                             │
  │                                                                     │
  │  After:  10.0.0.0/20  (4,092 usable IPs)                           │
  │  ├────────────────────────────────────────────────────────────────┤  │
  │                                                                     │
  │  Rules:                                                             │
  │  - Can only EXPAND (never shrink)                                   │
  │  - New range must CONTAIN original range                            │
  │  - NON-DISRUPTIVE (no downtime)                                     │
  │  - Cannot overlap with other subnets                                │
  └─────────────────────────────────────────────────────────────────────┘


  STATIC IP TYPES:
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │  ┌──────────────────┐     ┌──────────────────┐                          │
  │  │ EPHEMERAL        │     │ STATIC           │                          │
  │  │                  │     │                  │                          │
  │  │ External:        │     │ External:        │                          │
  │  │ Released when    │     │ Persists after   │                          │
  │  │ VM stops         │     │ VM stops/deletes │                          │
  │  │                  │     │                  │                          │
  │  │ Internal:        │     │ Internal:        │                          │
  │  │ Auto-assigned    │     │ Specific IP      │                          │
  │  │ from subnet      │     │ reserved         │                          │
  │  │                  │     │                  │                          │
  │  │ Cost: Free       │     │ Cost: Charged    │                          │
  │  │                  │     │ if UNUSED        │                          │
  │  │ Use: Dev/test    │     │ Use: Production  │                          │
  │  └──────────────────┘     │ DNS, LB          │                          │
  │                           └──────────────────┘                          │
  └──────────────────────────────────────────────────────────────────────────┘


  CLOUD NAT MODEL:
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │  ┌─── Private VMs (no external IPs) ───┐                                │
  │  │                                      │                                │
  │  │  ┌──────┐  ┌──────┐  ┌──────┐      │                                │
  │  │  │ VM 1 │  │ VM 2 │  │ VM 3 │      │                                │
  │  │  │ .10  │  │ .11  │  │ .12  │      │                                │
  │  │  └──┬───┘  └──┬───┘  └──┬───┘      │                                │
  │  │     │         │         │           │                                │
  │  └─────┼─────────┼─────────┼───────────┘                                │
  │        │         │         │                                             │
  │        └─────────┼─────────┘                                             │
  │                  │                                                       │
  │                  ▼                                                       │
  │        ┌─────────────────┐                                              │
  │        │   Cloud Router  │                                              │
  │        └────────┬────────┘                                              │
  │                 │                                                        │
  │                 ▼                                                        │
  │        ┌─────────────────┐      ┌──────────────────────────────┐        │
  │        │   Cloud NAT     │─────►│  INTERNET (outbound only)    │        │
  │        │                 │      │                              │        │
  │        │ NAT external IP │      │  Inbound: BLOCKED            │        │
  │        │ (auto or manual)│      │  (VMs not publicly exposed)  │        │
  │        └─────────────────┘      └──────────────────────────────┘        │
  │                                                                          │
  │  Cloud NAT is REGIONAL -- one per region needed                          │
  │  OUTBOUND ONLY -- external traffic cannot initiate connections           │
  └──────────────────────────────────────────────────────────────────────────┘


  CLOUD DNS ZONE TYPES:
  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │
  │  │   PUBLIC     │  │   PRIVATE    │  │  FORWARDING  │  │  PEERING   │  │
  │  │              │  │              │  │              │  │            │  │
  │  │ Visible to   │  │ Visible to   │  │ Forward to   │  │ Query     │  │
  │  │ the entire   │  │ specified    │  │ on-premises  │  │ another   │  │
  │  │ internet     │  │ VPCs only    │  │ DNS servers  │  │ VPC's DNS │  │
  │  │              │  │              │  │              │  │            │  │
  │  │ External     │  │ Internal     │  │ Hybrid       │  │ Cross-    │  │
  │  │ services     │  │ services     │  │ cloud        │  │ project   │  │
  │  └──────────────┘  └──────────────┘  └──────────────┘  └────────────┘  │
  └──────────────────────────────────────────────────────────────────────────┘
```

---

## 11. VM Connection Methods

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    VM CONNECTION METHODS                                      │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌───────────────────────────────────────────────────────────────────────────┐
  │                                                                           │
  │  Does the VM have an external IP?                                         │
  │         │                                                                 │
  │    ┌────┴────┐                                                            │
  │    │         │                                                            │
  │   YES       NO                                                            │
  │    │         │                                                            │
  │    │         ├──► Is it a boot issue? (SSH/RDP not working)               │
  │    │         │         │                                                  │
  │    │         │    ┌────┴────┐                                             │
  │    │         │    │         │                                             │
  │    │         │   YES       NO                                             │
  │    │         │    │         │                                             │
  │    │         │    ▼         ▼                                             │
  │    │         │ ┌─────────┐ ┌──────────────────┐                          │
  │    │         │ │ SERIAL  │ │ IAP TUNNEL       │                          │
  │    │         │ │ CONSOLE │ │                  │                          │
  │    │         │ │         │ │ --tunnel-through- │                          │
  │    │         │ │ gcloud  │ │    iap            │                          │
  │    │         │ │ compute │ │                  │                          │
  │    │         │ │ connect-│ │ Needs:           │                          │
  │    │         │ │ to-     │ │ - IAP role       │                          │
  │    │         │ │ serial- │ │ - FW: allow      │                          │
  │    │         │ │ port    │ │   35.235.240.0/20│                          │
  │    │         │ └─────────┘ │   port 22        │                          │
  │    │         │             └──────────────────┘                          │
  │    │         │                                                            │
  │    ▼         │                                                            │
  │ ┌───────────────────┐                                                     │
  │ │ Is it Linux       │                                                     │
  │ │ or Windows?       │                                                     │
  │ └────────┬──────────┘                                                     │
  │     ┌────┴────┐                                                           │
  │     │         │                                                           │
  │  LINUX    WINDOWS                                                         │
  │     │         │                                                           │
  │     ▼         ▼                                                           │
  │ ┌─────────┐ ┌──────────────────┐                                         │
  │ │ SSH     │ │ RDP              │                                         │
  │ │         │ │                  │                                         │
  │ │ gcloud  │ │ gcloud compute   │                                         │
  │ │ compute │ │ reset-windows-   │                                         │
  │ │ ssh     │ │ password         │                                         │
  │ │ my-vm   │ │                  │                                         │
  │ └─────────┘ │ Then use RDP     │                                         │
  │             │ client with      │                                         │
  │             │ returned creds   │                                         │
  │             └──────────────────┘                                         │
  └───────────────────────────────────────────────────────────────────────────┘
```

---

## 12. Dataflow and BigQuery Job Status

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    JOB STATUS MONITORING                                      │
 └──────────────────────────────────────────────────────────────────────────────┘


  DATAFLOW JOB LIFECYCLE:
  ══════════════════════════

  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ RUNNING  │───►│  DONE    │    │ FAILED   │
  │          │    │ (batch)  │    │          │
  └────┬─────┘    └──────────┘    └──────────┘
       │                               ▲
       │              ┌────────────────┘
       │              │
       ├───► Cancel ──┼──► CANCELLED
       │              │
       └───► Drain ───┼──► DRAINING ──► DRAINED
                      │
                      └──► UPDATED (replaced by new version)

  Cancel vs Drain:
  ┌──────────────────────────────────────────────────────────────┐
  │ CANCEL: Stops immediately. In-flight data may be lost.      │
  │ DRAIN:  Processes remaining data, then stops gracefully.    │
  │         Use drain for streaming jobs to avoid data loss.    │
  └──────────────────────────────────────────────────────────────┘


  BIGQUERY JOB TYPES:
  ══════════════════════

  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │  QUERY   │  │   LOAD   │  │ EXTRACT  │  │   COPY   │
  │          │  │          │  │          │  │          │
  │ SQL      │  │ Import   │  │ Export   │  │ Table    │
  │ execution│  │ data     │  │ data     │  │ to table │
  └──────────┘  └──────────┘  └──────────┘  └──────────┘

  Cost estimation (before running):
  ┌──────────────────────────────────────────────────────────────┐
  │ bq query --dry_run 'SELECT * FROM ...'                       │
  │                                                              │
  │ Returns: Bytes to be scanned (no cost, no execution)         │
  │ Use this to estimate cost = bytes scanned * $5/TB            │
  └──────────────────────────────────────────────────────────────┘
```
