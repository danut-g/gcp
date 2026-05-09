# Section 4 -- Operations Cheat Sheet

## Compute Engine

### Connect
```bash
gcloud compute ssh VM --zone=ZONE                    # Standard SSH
gcloud compute ssh VM --zone=ZONE --tunnel-through-iap  # No external IP
gcloud compute connect-to-serial-port VM --zone=ZONE    # Boot issues
gcloud compute reset-windows-password VM --zone=ZONE --user=admin  # RDP
gcloud compute scp file VM:~/file --zone=ZONE        # Copy files
```

### VM Lifecycle
```bash
gcloud compute instances stop|start|reset|suspend|resume|delete VM --zone=ZONE
gcloud compute instances delete VM --zone=ZONE --keep-disks=boot  # Keep disk
gcloud compute instances set-machine-type VM --zone=ZONE --machine-type=TYPE  # Must be STOPPED
gcloud compute instances move VM --zone=ZONE --destination-zone=ZONE2
gcloud compute instances list --filter="status=RUNNING"
gcloud compute instances describe VM --zone=ZONE
```

### Snapshots
```bash
gcloud compute snapshots create SNAP --source-disk=DISK --source-disk-zone=ZONE
gcloud compute snapshots list
gcloud compute snapshots delete SNAP
gcloud compute disks create DISK --source-snapshot=SNAP --zone=ZONE  # Restore

# Scheduled snapshots
gcloud compute resource-policies create snapshot-schedule SCHED \
  --region=REGION --max-retention-days=14 --daily-schedule --start-time=02:00
gcloud compute disks add-resource-policies DISK --resource-policies=SCHED --zone=ZONE
```

### Images
```bash
gcloud compute images create IMG --source-disk=DISK --source-disk-zone=ZONE --family=FAM
gcloud compute images list --no-standard-images           # Custom only
gcloud compute instances create VM --image-family=FAM --image-project=PROJ  # Latest in family
gcloud compute images deprecate OLD --state=DEPRECATED --replacement=NEW
```

| | Snapshot | Image |
|---|---|---|
| **For** | Backup any disk | Boot disk template |
| **Incremental** | Yes | No |
| **Families** | No | Yes |
| **VM creation speed** | Slower | Faster |

> :warning: Changing machine type requires VM to be **stopped**.
> :warning: Deleting a snapshot in an incremental chain is safe -- Google migrates data automatically.
> :warning: Serial console = last resort when SSH/RDP broken.

---

## GKE

### Cluster & Inventory
```bash
gcloud container clusters list
gcloud container clusters get-credentials CLUSTER --zone=ZONE  # Connect kubectl
kubectl get nodes -o wide
kubectl get pods -A                         # All namespaces
kubectl describe pod POD
kubectl logs POD [-c CONTAINER] [--previous] [-f]
kubectl top nodes|pods
```

### Node Pools
```bash
gcloud container node-pools create POOL --cluster=C --zone=Z \
  --machine-type=TYPE --num-nodes=3 --enable-autoscaling --min-nodes=1 --max-nodes=10
gcloud container clusters resize C --node-pool=POOL --num-nodes=5 --zone=Z  # Manual scale
gcloud container node-pools update POOL --cluster=C --zone=Z \
  --enable-autoscaling --min-nodes=2 --max-nodes=10
gcloud container node-pools delete POOL --cluster=C --zone=Z
```

### Artifact Registry Access
```
Image URL: LOCATION-docker.pkg.dev/PROJECT/REPO/IMAGE:TAG
```
- **Default**: Grant `roles/artifactregistry.reader` to node SA
- **Workload Identity** (recommended): Bind K8s SA to GCP SA with reader role
- **ImagePullSecrets**: For cross-project / external registries

### Kubernetes Resources

| Resource | Use |
|---|---|
| **Deployment** | Stateless apps, random pod names, shared PVC |
| **StatefulSet** | Stateful apps (DBs), ordered names (pod-0,1,2), individual PVCs |

```bash
kubectl apply -f FILE.yaml
kubectl run POD --image=IMG --restart=Never
kubectl delete pod POD --grace-period=0 --force   # Force delete stuck pod
kubectl scale statefulset STS --replicas=N
```

**Service types**: ClusterIP (internal) | NodePort | LoadBalancer (external)

### Autoscaling

| Autoscaler | Scales | Trigger |
|---|---|---|
| **HPA** | Pod replicas | CPU/memory/custom metrics |
| **VPA** | Pod CPU/memory requests | Historical usage |
| **Cluster Autoscaler** | Nodes | Unschedulable (pending) pods |

```bash
kubectl autoscale deployment DEP --min=2 --max=10 --cpu-percent=70
gcloud container clusters update C --zone=Z --enable-autoscaling \
  --min-nodes=2 --max-nodes=10 --node-pool=POOL
```

VPA modes: **Off** (recommend only) | **Initial** (creation only) | **Recreate** | **Auto**

### GKE Autopilot Pod Resource Requests

In Autopilot, billing is **per pod resource** — always set explicit requests.

| Resource | Autopilot Default (if omitted) | Min | Max |
|---|---|---|---|
| CPU | 0.5 vCPU | 0.25 vCPU | 30 vCPU |
| Memory | 2 GiB | 0.5 GiB | 110 GiB |

```yaml
# Correct: explicit requests in Autopilot
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"
```

```bash
# Check what requests are set
kubectl describe pod POD | grep -A5 Requests
kubectl get pod POD -o jsonpath='{.spec.containers[*].resources}'
```

- VPA in Autopilot: works in **recommendation mode only** (`updateMode: Off`)
- **Overprovisioning** = wasted cost; **underprovisioning** = OOMKilled pods

> :warning: HPA + VPA on same deployment = conflict. Don't combine on same metric.
> :warning: Pending pods trigger Cluster Autoscaler, not high CPU.
> :warning: `kubectl get pods -A` = all namespaces (exam shortcut).
> :warning: Autopilot minimum billable: 0.25 vCPU + 0.5 GiB per pod — set requests to match actual usage.

---

## Cloud Run

### Deploy & Revisions
```bash
gcloud run deploy SVC --image=IMG --region=R              # 100% to new revision
gcloud run deploy SVC --image=IMG --region=R --no-traffic  # Deploy without traffic
gcloud run deploy SVC --image=IMG --region=R --no-traffic --tag=canary  # Tagged URL
gcloud run revisions list --service=SVC --region=R
gcloud run services describe SVC --region=R --format="value(status.url)"
```

### Traffic Splitting & Rollback
```bash
gcloud run services update-traffic SVC --region=R \
  --to-revisions=REV_OLD=90,REV_NEW=10              # Canary
gcloud run services update-traffic SVC --region=R --to-latest  # 100% latest
gcloud run services update-traffic SVC --region=R \
  --to-revisions=PREV_REV=100                        # Instant rollback
gcloud run services describe SVC --region=R --format="yaml(status.traffic)"
```

### Scaling

| Param | Default | Purpose |
|---|---|---|
| `--min-instances` | 0 | Warm instances (0 = scale to zero) |
| `--max-instances` | 100 | Upper bound |
| `--concurrency` | 80 | Requests per instance |
| `--no-cpu-throttling` | off | CPU always on (WebSockets, background) |

```bash
gcloud run services update SVC --region=R --min-instances=2 --max-instances=50
gcloud run services update SVC --region=R --concurrency=50
gcloud run services update SVC --region=R --no-cpu-throttling
```

### IAM
```bash
gcloud run services add-iam-policy-binding SVC --region=R \
  --member="allUsers" --role="roles/run.invoker"          # Public
```

> :warning: `--no-traffic` + `--tag=canary` = test via tag URL without affecting prod.
> :warning: Rollback = just reroute traffic to old revision, no redeployment needed.
> :warning: `min-instances=0` = cold starts. Set >= 1 for latency-sensitive services.
> :warning: `--no-cpu-throttling` required for WebSockets / background work.

---

## Storage & Databases

### Cloud Storage Operations
```bash
gcloud storage cp|mv|rm|ls gs://BUCKET/PATH
gcloud storage buckets update gs://B --uniform-bucket-level-access  # IAM only
gcloud storage buckets update gs://B --versioning
gcloud storage buckets update gs://B --lifecycle-file=lifecycle.json
gcloud storage buckets update gs://B --retention-period=90d
gcloud storage buckets update gs://B --lock-retention-period        # IRREVERSIBLE
gcloud storage sign-url gs://B/file --duration=1h --private-key-file=key.json
```

### Lifecycle Rules (JSON)
- `SetStorageClass` + `age` condition = auto-transition (Standard -> Nearline -> Coldline -> Archive)
- `Delete` + `age` = auto-delete
- `Delete` + `isLive: false` + `numNewerVersions: N` = prune old versions

### Key Storage Roles
| Role | Access |
|---|---|
| `objectViewer` | Read |
| `objectCreator` | Write (no read/delete) |
| `objectAdmin` | Full object control |
| `storage.admin` | Full bucket + object control |

### Database Queries
```bash
gcloud sql connect INST --user=root --database=DB
bq query --use_legacy_sql=false 'SQL'
bq query --dry_run 'SQL'                               # Cost estimate, no charge
gcloud spanner databases execute-sql DB --instance=I --sql="SQL"
```

### Cloud SQL Backups
```bash
gcloud sql backups create --instance=INST
gcloud sql backups restore ID --restore-instance=INST
gcloud sql instances patch INST --backup-start-time=02:00 --enable-bin-log  # PITR (MySQL)
gcloud sql instances clone INST CLONE --point-in-time="2024-06-15T10:30:00Z"  # PITR restore
```

### Firestore Backup
```bash
gcloud firestore export gs://BUCKET/path [--collection-ids=col1,col2]
gcloud firestore import gs://BUCKET/path
```

### AlloyDB Backup
```bash
# On-demand backup
gcloud alloydb backups create my-backup --cluster=CLUSTER --region=REGION

# List backups
gcloud alloydb backups list --region=REGION

# Restore from backup
gcloud alloydb clusters restore my-restored-cluster --region=REGION \
  --backup=projects/P/locations/R/backups/my-backup

# Point-in-time restore (continuous backup enabled by default)
gcloud alloydb clusters restore my-restored-cluster --region=REGION \
  --source-cluster=CLUSTER --point-in-time=2025-06-01T10:00:00Z
```

### Cloud Spanner Backup
```bash
gcloud spanner backups create my-backup --instance=INST \
  --database=DB --retention-period=7d \
  --expiration-date=2025-07-01T00:00:00Z

gcloud spanner backups list --instance=INST
gcloud spanner databases restore --destination-instance=INST \
  --destination-database=DB_RESTORED --source-backup=my-backup
```

### Bigtable Backup
```bash
gcloud bigtable backups create my-backup --instance=INST --cluster=CLUSTER \
  --table=TABLE --retention-period=7d

gcloud bigtable backups list --instance=INST --cluster=CLUSTER
gcloud bigtable instances tables restore --source=my-backup \
  --destination=TABLE_RESTORED --destination-instance=INST
```

### Backup Comparison Matrix

| Service | Backup Type | PITR | gcloud command prefix |
|---|---|---|---|
| Cloud SQL | Automated + on-demand | Yes (bin log) | `gcloud sql backups` |
| AlloyDB | Continuous (default on) | Yes | `gcloud alloydb backups` |
| Spanner | On-demand | Yes (TrueTime) | `gcloud spanner backups` |
| Firestore | Manual export | No | `gcloud firestore export` |
| Bigtable | On-demand | No | `gcloud bigtable backups` |
| BigQuery | Auto snapshots (table-level) | No | `bq cp` |

### Database Center

- Unified console for ALL GCP database services (Cloud SQL, AlloyDB, Spanner, Bigtable, Firestore, Memorystore)
- Find: Console → Database Center
- Shows: fleet health, query performance, backup status, recommendations

> :warning: Database Center is **read-only management plane** — you still use service-specific commands to make changes.

### Job Status
```bash
gcloud dataflow jobs list --region=R [--status=active]
gcloud dataflow jobs drain JOB_ID --region=R   # Graceful stop (finish in-flight)
gcloud dataflow jobs cancel JOB_ID --region=R  # Immediate stop
bq ls -j --max_results=20                     # BQ jobs
bq show -j JOB_ID
```

> :warning: `--lock-retention-period` is **irreversible** -- cannot shorten or remove.
> :warning: Firestore has **no built-in scheduled backups** -- use Cloud Scheduler + Cloud Function.
> :warning: `bq query --dry_run` = free cost estimate.
> :warning: Dataflow `drain` vs `cancel`: drain finishes in-flight data, cancel stops immediately.
> :warning: Storage early deletion fees: Nearline 30d, Coldline 90d, Archive 365d.

---

## Networking

### Subnets
```bash
gcloud compute networks subnets create SUB --network=VPC --region=R --range=10.1.0.0/24
gcloud compute networks subnets create SUB --network=VPC --region=R --range=CIDR \
  --enable-private-ip-google-access --enable-flow-logs \
  --secondary-range=pods=10.8.0.0/14,svcs=10.12.0.0/20   # GKE
gcloud compute networks subnets expand-ip-range SUB --region=R --prefix-length=20
```

### Static IPs
```bash
gcloud compute addresses create IP --region=R              # Regional external
gcloud compute addresses create IP --global                # Global (for LB)
gcloud compute addresses create IP --region=R --subnet=SUB --addresses=10.0.1.50  # Internal
gcloud compute addresses list
gcloud compute addresses delete IP --region=R
```

### Cloud DNS
```bash
gcloud dns managed-zones create ZONE --dns-name=example.com. --visibility=public
gcloud dns managed-zones create ZONE --dns-name=internal.co. --visibility=private --networks=VPC
# Records via transactions:
gcloud dns record-sets transaction start --zone=Z
gcloud dns record-sets transaction add "1.2.3.4" --name=www.ex.com. --ttl=300 --type=A --zone=Z
gcloud dns record-sets transaction execute --zone=Z
```

### Cloud NAT
```bash
gcloud compute routers create RTR --network=VPC --region=R
gcloud compute routers nats create NAT --router=RTR --region=R \
  --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges
```

> :warning: Subnets can only **expand**, never shrink. One-way operation.
> :warning: Unused reserved static IPs **incur charges**.
> :warning: Cloud NAT = **outbound only**. No inbound from internet.
> :warning: DNS records require transaction workflow: start -> add/remove -> execute.
> :warning: DNS names must end with a dot (e.g., `example.com.`).

---

## Monitoring & Logging

### Alerts
```bash
gcloud alpha monitoring channels create --type=email --channel-labels=email_address=X
gcloud alpha monitoring policies create --policy-from-file=alert.json
gcloud monitoring uptime create CHK --resource-type=uptime-url --hostname=HOST --path=/health
```

### Key Metrics Quick Reference

| Service | Metric prefix |
|---|---|
| Compute Engine | `compute.googleapis.com/instance/cpu/utilization` |
| Cloud SQL | `cloudsql.googleapis.com/database/cpu/utilization` |
| GKE | `kubernetes.io/container/cpu/request_utilization` |
| Cloud Run | `run.googleapis.com/request_count` |
| Load Balancer | `loadbalancing.googleapis.com/https/request_count` |

### Log-Based Metrics
```bash
gcloud logging metrics create NAME --log-filter='severity>=ERROR' --description="DESC"
# Use in alerts: metric.type = "logging.googleapis.com/user/NAME"
```

### Log Sinks (Export)

| Destination | Use |
|---|---|
| Cloud Storage | Archival, compliance |
| BigQuery | SQL analysis |
| Pub/Sub | Real-time / SIEM |

```bash
gcloud logging sinks create SINK DEST --log-filter='FILTER'
# Grant writer identity access to destination after creation!
gcloud logging sinks describe SINK --format="get(writerIdentity)"
```

### Log Filters
```bash
gcloud logging read 'severity>=ERROR' --limit=20
gcloud logging read 'resource.type="gce_instance"' --limit=20
gcloud logging read 'logName="projects/P/logs/cloudaudit.googleapis.com%2Factivity"'
gcloud logging read 'protoPayload.methodName="v1.compute.instances.delete"'
```

### Log Buckets

| Bucket | Retention | Configurable |
|---|---|---|
| `_Required` | 400 days | No |
| `_Default` | 30 days | Yes |
| Custom | 1-3650 days | Yes |

```bash
gcloud logging buckets create BUCKET --location=LOC --retention-days=90
gcloud logging sinks update _Default --add-exclusion=name=X,filter='severity=DEBUG'
```

### Audit Logs

| Type | Default | Cost |
|---|---|---|
| Admin Activity | Always on, **cannot disable** | Free |
| Data Access | Off (except BigQuery) | Charged |
| System Event | Always on | Free |
| Policy Denied | Always on | Free |

### Ops Agent
```bash
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install
# Config: /etc/google-cloud-ops-agent/config.yaml
sudo systemctl restart google-cloud-ops-agent
```

### Managed Prometheus
```bash
gcloud container clusters update C --zone=Z --enable-managed-prometheus
# Then create PodMonitoring CRDs to scrape app metrics
```

### Query Insights

Diagnose slow queries across database services:

| Service | Query Insights location |
|---|---|
| Cloud SQL | Console → Cloud SQL → Instance → Query Insights |
| AlloyDB | Console → AlloyDB → Cluster → Query Insights |
| Spanner | `gcloud spanner databases execute-sql` + Query Optimizer stats |

Key features: Top queries by latency, Index Advisor (suggests missing indexes), execution plans.

```bash
# Spanner query stats
gcloud spanner databases execute-sql DB --instance=I \
  --sql="SELECT * FROM SPANNER_SYS.QUERY_STATS_TOP_HOUR LIMIT 20"
```

### Personalized Service Health

- Shows Google Cloud incidents affecting **services your projects actually use**
- vs Public status page (status.cloud.google.com) — public page shows org-wide incidents

Find: Console → Active Issues | Console → Cloud Monitoring → Service Health

### Gemini Cloud Assist for Monitoring

- Summarizes complex log entries in plain language
- Helps write alert policies and PromQL expressions
- Accessible from Cloud Logging and Cloud Monitoring sidebars
- Key exam point: Gemini **assists** but cannot take action

### Active Assist (AI Recommendations)

| Recommender | What it finds |
|---|---|
| IAM Recommender | Over-provisioned roles — suggests least privilege |
| VM Rightsizing | Under/over-utilized VMs |
| Committed Use Recommender | When to buy CUDs for savings |
| Firewall Insights | Unused/shadowed firewall rules |
| Cost Recommendations | Idle resources, unattached disks |
| Unattended Project Recommender | Projects with no activity |

```bash
# List recommendations
gcloud recommender recommendations list \
  --recommender=google.iam.policy.Recommender \
  --project=PROJECT_ID --location=global

# Mark recommendation as claimed (in progress)
gcloud recommender recommendations mark-claimed REC_ID \
  --recommender=google.iam.policy.Recommender \
  --project=PROJECT_ID --location=global --etag=ETAG

# Apply recommendation (follow the described action then mark succeeded)
gcloud recommender recommendations mark-succeeded REC_ID \
  --recommender=google.iam.policy.Recommender \
  --project=PROJECT_ID --location=global --etag=ETAG
```

> :warning: Admin Activity audit logs = **always on, free, cannot be disabled**.
> :warning: Data Access audit logs = **off by default** (except BigQuery), **cost money**.
> :warning: Log sink writer identity needs **explicit permissions** on destination.
> :warning: `_Required` bucket retention (400 days) **cannot be changed**.
> :warning: Ops Agent replaces **both** legacy Monitoring and Logging agents.
> :warning: Exclusion filters reduce log storage costs but excluded logs are **gone forever**.
> :warning: Active Assist recommendations require **manual application** — they do not auto-apply.
> :warning: Query Insights is **per-instance/cluster** — you navigate to each database separately.
