# 04 — Processing Pub/Sub Events on Serverless

## Exam Relevance

Pub/Sub is the canonical messaging backbone in Google Cloud. The exam tests whether you can:
- Distinguish **push vs. pull** subscriptions
- Wire Pub/Sub to **Cloud Run / Cloud Functions / App Engine**
- Configure **acknowledgement, retries, dead-letter topics, ordering**

---

## 1. Pub/Sub in 60 Seconds

```
Publisher → Topic → Subscription → Subscriber
```

- **Topic** — named channel
- **Subscription** — durable cursor on the topic; one subscription per consumer group
- **Message** — bytes + attributes, retained up to **7 days** (default) or 31 days (max)
- Delivery: **at-least-once** (duplicates possible — make handlers **idempotent**)

---

## 2. Push vs. Pull Subscriptions

| | **Push** | **Pull** |
|---|---|---|
| Consumer | Pub/Sub HTTP-POSTs to your endpoint | Your client calls `Subscriber.pull()` |
| Best for | Cloud Run / Cloud Functions / App Engine | GCE / GKE workers, batch consumers |
| Auth | OIDC token (Pub/Sub service account) | App credentials |
| Backpressure | Pub/Sub adapts to your endpoint's response time | Client controls rate |
| Ordering | Per-subscription `enable_message_ordering` | Same |

> **Exam shortcut:** Serverless target → **push**. Long-running custom worker → **pull**.

---

## 3. Push Subscription to Cloud Run / Functions / App Engine

### Create a topic
```bash
gcloud pubsub topics create orders
```

### Create the push subscription
```bash
gcloud pubsub subscriptions create orders-sub \
  --topic=orders \
  --push-endpoint=https://my-service-xxxxx-uc.a.run.app/ \
  --push-auth-service-account=pubsub-invoker@PROJECT_ID.iam.gserviceaccount.com \
  --ack-deadline=30 \
  --message-retention-duration=7d
```

### Required IAM
The `--push-auth-service-account` SA must be allowed to invoke the target:
```bash
# Cloud Run
gcloud run services add-iam-policy-binding my-service \
  --region=us-central1 \
  --member=serviceAccount:pubsub-invoker@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/run.invoker

# Cloud Functions Gen 2 (also Cloud Run under the hood)
# Same as above, on the underlying Cloud Run service

# Pub/Sub service-managed account also needs:
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:service-PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
  --role=roles/iam.serviceAccountTokenCreator
```

---

## 4. Push Message Format

Pub/Sub sends a JSON HTTP POST to your endpoint:

```json
{
  "message": {
    "attributes": { "key1": "v1" },
    "data": "SGVsbG8gV29ybGQh",            // base64-encoded payload
    "messageId": "1234567",
    "publishTime": "2026-04-25T10:00:00Z"
  },
  "subscription": "projects/PROJECT_ID/subscriptions/orders-sub"
}
```

### Acknowledgement = HTTP 2xx
- `200/204` → message **ack**'d (removed from subscription)
- Any non-2xx (or timeout) → **nack**, will be **redelivered** (with exponential backoff)

### Cloud Run handler example (Python / Flask)
```python
import base64, json
from flask import Flask, request

app = Flask(__name__)

@app.post("/")
def consume():
    envelope = request.get_json(force=True)
    msg = envelope["message"]
    payload = base64.b64decode(msg["data"]).decode("utf-8")
    print(f"Got: {payload}")
    return ("", 204)   # ack
```

---

## 5. Cloud Functions Direct Pub/Sub Trigger

For Cloud Functions you can skip the manual subscription:

```bash
gcloud functions deploy process-orders \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=process \
  --trigger-topic=orders
```

The function receives a CloudEvent (Gen 2):
```python
import base64, functions_framework

@functions_framework.cloud_event
def process(event):
    data = base64.b64decode(event.data["message"]["data"]).decode()
    print(f"Order: {data}")
```

Behind the scenes, Cloud Functions Gen 2 uses **Eventarc + a managed Pub/Sub subscription**.

---

## 6. Reliability Patterns

### Acknowledgement deadline
- Default 10 s (raise with `--ack-deadline=60` up to 600 s)
- If your handler is slow, Pub/Sub assumes the message failed and redelivers
- Push subscriptions also auto-extend deadline based on response patterns

### Retry policy
```bash
gcloud pubsub subscriptions update orders-sub \
  --min-retry-delay=10s \
  --max-retry-delay=600s
```

### Dead-letter topic (DLT)
After N failed deliveries, Pub/Sub routes the message to a **dead-letter topic** for inspection.

```bash
gcloud pubsub topics create orders-dlq

gcloud pubsub subscriptions update orders-sub \
  --dead-letter-topic=orders-dlq \
  --max-delivery-attempts=5
```

The Pub/Sub SA needs `roles/pubsub.publisher` on the DLT and `roles/pubsub.subscriber` on the source sub.

### Ordering
```bash
gcloud pubsub subscriptions create orders-ordered \
  --topic=orders \
  --enable-message-ordering
```
Publishers must set an **ordering key**. Order is preserved per key, per region.

### Exactly-once delivery (Pull subs only)
```bash
gcloud pubsub subscriptions create orders-eod \
  --topic=orders \
  --enable-exactly-once-delivery
```
> Push subscriptions still rely on idempotent handlers + at-least-once.

---

## 7. App Engine Push Subscription

```bash
gcloud pubsub subscriptions create app-engine-sub \
  --topic=orders \
  --push-endpoint=https://PROJECT_ID.uc.r.appspot.com/_ah/push-handlers/orders
```
App Engine **automatically authenticates** push requests if the URL starts with `/_ah/push-handlers/` (Standard env).

---

## 8. Schema Validation (Optional)

Pub/Sub topics can enforce a **schema** (Avro or Protobuf):
```bash
gcloud pubsub schemas create order-schema \
  --type=avro \
  --definition-file=order.avsc

gcloud pubsub topics create orders \
  --message-encoding=json \
  --schema=order-schema
```
Publishes that don't match the schema are rejected.

---

## 9. Common gcloud / Console Operations

```bash
# Publish a test message
gcloud pubsub topics publish orders --message='{"id":"123","total":99.99}'

# Pull (debug only — pulls into your terminal)
gcloud pubsub subscriptions pull orders-sub --limit=5 --auto-ack

# Modify push endpoint
gcloud pubsub subscriptions modify-push-config orders-sub \
  --push-endpoint=https://new-target.example.com/

# Snapshot (for replay)
gcloud pubsub snapshots create orders-snap --subscription=orders-sub

# Seek back to a snapshot
gcloud pubsub subscriptions seek orders-sub --snapshot=orders-snap
```

Console: **Pub/Sub → Topics / Subscriptions → Create / Edit**.

---

## 10. Choosing the Subscription Type

| Scenario | Subscription |
|---|---|
| Cloud Run / Functions / App Engine consumer | **Push** |
| GCE/GKE worker pool, custom client | **Pull** |
| Need exactly-once delivery semantics | **Pull + EoD** |
| Streaming-style consumption from Dataflow | Pull (StreamingPull API) |
| Need ordering | Push or Pull, with ordering key |

---

## 11. Exam Traps & Keywords

| If you see... | Answer |
|---|---|
| "Cloud Run consumes Pub/Sub" | Push subscription + OIDC SA + `run.invoker` |
| "Messages keep being redelivered" | Handler returns non-2xx, or exceeds ack deadline → fix or raise deadline |
| "Bad messages should be quarantined" | Dead-letter topic + max delivery attempts |
| "Preserve order per customer" | Ordering key on publish, `--enable-message-ordering` on sub |
| "Avoid duplicate processing" | Pull + Exactly-once delivery, or idempotent handler |
| "Reset processing back in time" | Snapshot + `seek` |

---

## 12. Sources

- [Pub/Sub overview](https://cloud.google.com/pubsub/docs/overview)
- [Push subscriptions](https://cloud.google.com/pubsub/docs/push)
- [Authenticate push](https://cloud.google.com/pubsub/docs/authenticate-push-subscriptions)
- [Dead-letter topics](https://cloud.google.com/pubsub/docs/handling-failures)
- [Ordering](https://cloud.google.com/pubsub/docs/ordering)
- [Exactly-once delivery](https://cloud.google.com/pubsub/docs/exactly-once-delivery)
