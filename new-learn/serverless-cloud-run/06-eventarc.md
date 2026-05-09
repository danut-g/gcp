# 06 — Eventarc: Unified Event Routing

## Exam Relevance

Eventarc is **the modern, recommended** way to route Google Cloud events to serverless targets. The exam expects you to know:
- What Eventarc does and why it exists
- The two source classes (direct sources, and Cloud Audit Log sources)
- How to create triggers for **Cloud Run / Cloud Functions Gen 2 / Workflows / GKE**
- Required IAM and the relationship to Pub/Sub

---

## 1. What Is Eventarc?

A **fully managed event router** that delivers events from **any** of 150+ Google Cloud sources to a target service, in a uniform **CloudEvents** format.

```
┌────────────────┐
│  Event Source  │  GCS, Pub/Sub, BigQuery, Cloud SQL, Audit Logs, Firestore...
└────────┬───────┘
         ▼
┌────────────────┐
│   Eventarc     │  Filters + transforms + routes
└────────┬───────┘
         ▼
┌────────────────────────────────────────────────────────┐
│ Target: Cloud Run / Functions Gen 2 / Workflows / GKE  │
└────────────────────────────────────────────────────────┘
```

Why it matters:
- **Single API** for all event routing
- **CloudEvents-compliant** payloads (portable across vendors)
- Reuses **Pub/Sub** under the hood for buffering & retries
- Replaces a patchwork of legacy triggers

---

## 2. Source Categories

### a) Direct sources (first-class)
| Source | Example event |
|---|---|
| Cloud Storage | `object.v1.finalized` |
| Pub/Sub | `pubsub.topic.v1.messagePublished` |
| Firebase Alerts | `firebase.firebasealerts.alerts.v1.published` |
| Firestore | `firestore.document.v1.created` |
| BigQuery (DML/DDL) | via Audit Logs (see below) |

### b) Cloud Audit Log sources (anything that writes to Audit Logs)
Filter by `serviceName` + `methodName`. Examples:
| Service | Method | Use case |
|---|---|---|
| `bigquery.googleapis.com` | `jobservice.jobcompleted` | React to query/load completion |
| `compute.googleapis.com` | `v1.compute.instances.insert` | New VM created |
| `iam.googleapis.com` | `CreateServiceAccount` | Audit SA creation |
| `cloudsql.googleapis.com` | `cloudsql.instances.update` | DB config changes |

---

## 3. Targets

| Target | Trigger flag |
|---|---|
| Cloud Run | `--destination-run-service` |
| Cloud Functions (Gen 2) | implicit via `gcloud functions deploy ... --trigger-event-filters=...` |
| Workflows | `--destination-workflow` |
| GKE service | `--destination-gke-cluster` |

---

## 4. Creating a Trigger (Cloud Run target)

### Direct GCS event
```bash
gcloud eventarc triggers create gcs-trigger \
  --location=us-central1 \
  --destination-run-service=image-processor \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.storage.object.v1.finalized" \
  --event-filters="bucket=my-uploads" \
  --service-account=eventarc-sa@PROJECT_ID.iam.gserviceaccount.com
```

### Pub/Sub event (creates the topic if you omit `--transport-topic`)
```bash
gcloud eventarc triggers create pubsub-trigger \
  --location=us-central1 \
  --destination-run-service=consumer \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.pubsub.topic.v1.messagePublished" \
  --transport-topic=projects/PROJECT_ID/topics/orders \
  --service-account=eventarc-sa@PROJECT_ID.iam.gserviceaccount.com
```

### Audit-log event
```bash
gcloud eventarc triggers create on-vm-create \
  --location=us-central1 \
  --destination-run-service=vm-watcher \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.audit.log.v1.written" \
  --event-filters="serviceName=compute.googleapis.com" \
  --event-filters="methodName=v1.compute.instances.insert" \
  --service-account=eventarc-sa@PROJECT_ID.iam.gserviceaccount.com
```

---

## 5. Required Setup (One Time)

### Enable APIs
```bash
gcloud services enable eventarc.googleapis.com \
  run.googleapis.com pubsub.googleapis.com \
  logging.googleapis.com cloudfunctions.googleapis.com
```

### Audit Log routes — enable Data Access logs (for audit-source triggers)
**IAM & Admin → Audit Logs** in the console — enable the `Data Read` / `Data Write` logs for the service you want to listen on.

### IAM — Eventarc trigger SA
The trigger service account needs:
- `roles/eventarc.eventReceiver` (project)
- `roles/run.invoker` on the destination Cloud Run service
- For Pub/Sub triggers: `roles/iam.serviceAccountTokenCreator`

### Pub/Sub publisher (GCS / other source SAs)
Source services publish into Eventarc-managed Pub/Sub topics — their service agents need `roles/pubsub.publisher` (for direct sources) or `roles/pubsub.publisher` on the transport topic.

---

## 6. Locations & Multi-Region

- Eventarc triggers are **regional**.
- For GCS events, the trigger region must match the bucket location (or use `global` location triggers in supported regions).
- For Pub/Sub, the trigger region matches the topic region (or any).
- For audit-log events, you can also use `global`.

---

## 7. Listing, Updating, Deleting

```bash
# List
gcloud eventarc triggers list --location=us-central1

# Describe
gcloud eventarc triggers describe gcs-trigger --location=us-central1

# Update destination
gcloud eventarc triggers update gcs-trigger \
  --location=us-central1 \
  --destination-run-service=new-service

# Delete
gcloud eventarc triggers delete gcs-trigger --location=us-central1
```

Console: **Eventarc → Triggers → Create Trigger**.

---

## 8. CloudEvents Payload

All events arrive in CloudEvents 1.0 format:
```http
POST / HTTP/1.1
ce-id: 1234567890
ce-source: //storage.googleapis.com/projects/_/buckets/my-uploads
ce-specversion: 1.0
ce-type: google.cloud.storage.object.v1.finalized
ce-time: 2026-04-25T10:00:00Z
ce-subject: objects/uploads/photo.jpg
content-type: application/json

{ "bucket": "my-uploads", "name": "uploads/photo.jpg", "size": "1024" }
```

Use the `cloudevents` SDK in your language to parse cleanly.

---

## 9. Eventarc vs. Direct Pub/Sub Push

| | **Eventarc** | **Direct Pub/Sub push** |
|---|---|---|
| Sources | 150+ Google Cloud services | Only Pub/Sub |
| Format | CloudEvents (uniform) | Raw Pub/Sub envelope |
| Setup | One trigger; transports auto-managed | You manage subscription + auth |
| Transformation | Limited (filters only) | None |
| Best for | Multi-source apps, Cloud Audit logs | Pure Pub/Sub fan-out |

> Pick **Pub/Sub direct push** for a high-volume Pub/Sub-only pipeline.
> Pick **Eventarc** for any non-Pub/Sub source, audit-log triggers, or to standardize on CloudEvents.

---

## 10. Patterns

### React to BigQuery query completion
```bash
gcloud eventarc triggers create on-bq-job \
  --location=us-central1 \
  --destination-run-service=bq-postprocess \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.audit.log.v1.written" \
  --event-filters="serviceName=bigquery.googleapis.com" \
  --event-filters="methodName=jobservice.jobcompleted" \
  --service-account=eventarc-sa@PROJECT_ID.iam.gserviceaccount.com
```

### Trigger Workflows from a Firestore document write
```bash
gcloud eventarc triggers create on-firestore-write \
  --location=nam5 \
  --destination-workflow=order-workflow \
  --destination-workflow-location=us-central1 \
  --event-filters="type=google.cloud.firestore.document.v1.created" \
  --event-filters="database=(default)" \
  --event-filters-path-pattern="document=orders/{id}" \
  --service-account=eventarc-sa@PROJECT_ID.iam.gserviceaccount.com
```

---

## 11. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Trigger created but no events | Source not emitting / Audit Logs disabled | Enable Data Access logs |
| 403 from target | Trigger SA missing `run.invoker` | Grant role on target |
| GCS events lost | GCS SA missing `pubsub.publisher` | Grant role |
| `permission denied to act as service account` | Missing `iam.serviceAccountTokenCreator` | Grant on the SA |
| Wrong region | Bucket vs trigger region mismatch | Recreate in matching region |

---

## 12. Exam Traps & Keywords

| If you see... | Answer |
|---|---|
| "Trigger Cloud Run from any GCP service" | Eventarc |
| "React to a Cloud Audit Log event" | Eventarc audit-log trigger |
| "Trigger Cloud Functions Gen 2 from BigQuery" | Eventarc with `bigquery.googleapis.com` filter |
| "Standard event format across sources" | CloudEvents (Eventarc) |
| "Replace legacy GCS → Cloud Functions trigger" | Migrate to Eventarc (Gen 2) |

---

## 13. Sources

- [Eventarc overview](https://cloud.google.com/eventarc/docs/overview)
- [Direct events list](https://cloud.google.com/eventarc/docs/reference/supported-events)
- [Cloud Audit Log triggers](https://cloud.google.com/eventarc/docs/run/create-trigger-auditlogs-gcloud)
- [Eventarc roles & permissions](https://cloud.google.com/eventarc/docs/all-roles-permissions)
- [CloudEvents spec](https://github.com/cloudevents/spec)
