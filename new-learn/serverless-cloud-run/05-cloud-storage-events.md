# 05 — Reacting to Cloud Storage Events

## Exam Relevance

Object change notifications are one of the **most common event-driven patterns on GCP**. The exam tests:
- Which event types exist (finalize, delete, archive, metadata update)
- How to wire a GCS bucket to Cloud Functions or Cloud Run
- Differences between Pub/Sub notifications, legacy Object Change Notifications, and Eventarc

---

## 1. Three Mechanisms — Know All Three

| Mechanism | Status | Best With |
|---|---|---|
| **Pub/Sub notifications** | ✅ Recommended for messaging | Any consumer (Cloud Run, GCE, Dataflow…) |
| **Eventarc trigger** | ✅ Recommended for Cloud Run / Cloud Functions Gen 2 | Cloud Run, Functions Gen 2 |
| **Object Change Notifications (OCN)** | ⚠️ Legacy (deprecated) | Old systems only |
| **Cloud Functions Gen 1 native trigger** | ⚠️ Legacy — uses internal mechanism | Existing Gen 1 functions |

> For **new** workloads on the exam, prefer **Eventarc** (Functions Gen 2 / Cloud Run) or **Pub/Sub notifications** (other targets).

---

## 2. GCS Event Types

| Event (Eventarc / CloudEvents) | Pub/Sub `eventType` | When |
|---|---|---|
| `google.cloud.storage.object.v1.finalized` | `OBJECT_FINALIZE` | New object **created** or overwrite finalized |
| `google.cloud.storage.object.v1.deleted` | `OBJECT_DELETE` | Object permanently removed |
| `google.cloud.storage.object.v1.archived` | `OBJECT_ARCHIVE` | Versioned bucket — old version archived |
| `google.cloud.storage.object.v1.metadataUpdated` | `OBJECT_METADATA_UPDATE` | Custom metadata changed |

> **Exam tip:** "When a file is uploaded" → `finalized`. (Upload + overwrite both fire `finalized`.)

---

## 3. Path A — Eventarc Trigger (Cloud Run / Functions Gen 2)

### Cloud Functions Gen 2 example
```bash
gcloud functions deploy process-upload \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=on_finalize \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=my-uploads-bucket"
```

### Cloud Run example
```bash
gcloud eventarc triggers create gcs-upload-trigger \
  --location=us-central1 \
  --destination-run-service=image-processor \
  --destination-run-region=us-central1 \
  --event-filters="type=google.cloud.storage.object.v1.finalized" \
  --event-filters="bucket=my-uploads-bucket" \
  --service-account=eventarc-sa@PROJECT_ID.iam.gserviceaccount.com
```

### Required setup (one-time per project)
```bash
# Enable APIs
gcloud services enable eventarc.googleapis.com run.googleapis.com \
  pubsub.googleapis.com cloudfunctions.googleapis.com

# Grant the GCS service account the right to publish events
PROJECT_NUMBER=$(gcloud projects describe PROJECT_ID --format='value(projectNumber)')
GCS_SA=$(gsutil kms serviceaccount -p PROJECT_ID)

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:${GCS_SA}" \
  --role="roles/pubsub.publisher"

# Eventarc trigger SA needs to invoke the target
gcloud run services add-iam-policy-binding image-processor \
  --member=serviceAccount:eventarc-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/run.invoker \
  --region=us-central1
```

### Handler example (Python, Cloud Run / Functions Gen 2)
```python
import functions_framework

@functions_framework.cloud_event
def on_finalize(event):
    bucket = event.data["bucket"]
    name   = event.data["name"]
    size   = event.data["size"]
    print(f"New file: gs://{bucket}/{name} ({size} bytes)")
    # ... process the file
```

---

## 4. Path B — Pub/Sub Notification (Generic)

Use when the consumer is **not** Cloud Run / Functions (e.g., GKE, GCE, Dataflow).

### Configure GCS to publish to a topic
```bash
gcloud pubsub topics create gcs-events

gsutil notification create \
  -t projects/PROJECT_ID/topics/gcs-events \
  -e OBJECT_FINALIZE \
  -f json \
  gs://my-uploads-bucket
```

Flags:
- `-t TOPIC` — destination topic
- `-e EVENT` — filter by event type (omit = all)
- `-f json|none` — payload format
- `-p PREFIX` — only match objects with the given prefix

### List / delete notifications
```bash
gsutil notification list gs://my-uploads-bucket
gsutil notification delete projects/_/buckets/my-uploads-bucket/notificationConfigs/1
```

### Required IAM
The GCS service agent must be able to publish:
```bash
GCS_SA=$(gsutil kms serviceaccount -p PROJECT_ID)
gcloud pubsub topics add-iam-policy-binding gcs-events \
  --member="serviceAccount:${GCS_SA}" \
  --role=roles/pubsub.publisher
```

Then any subscriber model (push to Cloud Run, pull from a worker, etc.) works as in `04-pubsub-event-processing.md`.

---

## 5. Path C — Cloud Functions Gen 1 (Legacy)

```bash
gcloud functions deploy process-upload-v1 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=on_finalize \
  --trigger-resource=my-uploads-bucket \
  --trigger-event=google.storage.object.finalize
```

Handler signature (Gen 1):
```python
def on_finalize(data, context):
    print(f"File: gs://{data['bucket']}/{data['name']}")
```

> Prefer **Gen 2** (Eventarc) for new code.

---

## 6. Filtering Within an Event

Eventarc and Pub/Sub allow filtering only by `bucket` and `eventType`. To filter by **prefix / suffix**:
- **Pub/Sub notifications:** use the `-p` (prefix) flag on `gsutil notification create`
- **Eventarc:** filter inside the function (`if not name.startswith("uploads/"): return`) — or use multiple notifications

```bash
# Only PDFs in the 'uploads/' prefix → topic
gsutil notification create \
  -t projects/PROJECT_ID/topics/pdf-events \
  -e OBJECT_FINALIZE \
  -p uploads/ \
  gs://my-bucket
```

---

## 7. Ordering & Delivery Guarantees

- GCS events are **at-least-once** (duplicates possible)
- **No guaranteed ordering** between events (even for the same object). Build idempotent handlers.
- For ordering, push events to a **Pub/Sub topic with ordering key = object name**, then consume ordered subscription.

---

## 8. Common Patterns

### Image thumbnail generator
```
GCS upload → Eventarc → Cloud Run → write thumbnail to gs://thumbs/
```

### CSV → BigQuery loader
```
GCS upload → Pub/Sub notification → Cloud Function → load into BQ
```

### File-virus scanner
```
GCS upload → Eventarc → Cloud Run job (long-running) → quarantine bucket on hit
```

### Audit-log driven cleanup
```
Object delete → Eventarc filter → Cloud Function → notify Slack
```

---

## 9. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| No event fires after upload | Notification not created / wrong event type | `gsutil notification list`, recreate |
| Eventarc trigger fails | GCS SA missing `pubsub.publisher` | Grant on project or topic |
| Function gets duplicates | At-least-once → make idempotent | Track `event.id` (event ID) |
| Cloud Run gets 403 | Trigger SA missing `run.invoker` | Add binding |
| Function never invoked | Bucket region ≠ trigger region | Match regions or use multi-region (`global` for Eventarc) |

---

## 10. Exam Traps & Keywords

| If you see... | Answer |
|---|---|
| "Run code when a file is uploaded to GCS" | Cloud Functions Gen 2 (Eventarc, `finalized`) |
| "Existing Cloud Run service should react to GCS events" | Eventarc trigger → Cloud Run |
| "Want any consumer (worker, Dataflow…) to react" | `gsutil notification create` → Pub/Sub topic |
| "Filter by file path prefix" | `gsutil notification create -p PREFIX` |
| "How are events delivered" | At-least-once, no order — handler must be idempotent |

---

## 11. Sources

- [GCS Pub/Sub notifications](https://cloud.google.com/storage/docs/pubsub-notifications)
- [Eventarc with Cloud Storage](https://cloud.google.com/eventarc/docs/run/create-trigger-storage-gcloud)
- [`gsutil notification`](https://cloud.google.com/storage/docs/gsutil/commands/notification)
- [Cloud Functions Gen 2 GCS trigger](https://cloud.google.com/functions/docs/calling/storage)
