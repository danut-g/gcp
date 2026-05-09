# Section 4 — Ensuring Successful Operation of a Cloud Solution — Flashcards

---
### Q1: What gcloud command connects to a VM's serial console for troubleshooting when SSH is not working?
**A:** `gcloud compute connect-to-serial-port my-vm --zone=us-central1-a`. This is useful for diagnosing boot issues or when SSH/RDP is unreachable.

---
### Q2: What gcloud command resets the Windows password on a Compute Engine VM?
**A:** `gcloud compute reset-windows-password my-windows-vm --zone=us-central1-a --user=admin`. It returns the IP, username, and password for RDP access.

---
### Q3: True or False: You can change a VM's machine type while it is running.
**A:** False. You must stop the VM first, change the machine type with `gcloud compute instances set-machine-type`, then start it again.

---
### Q4: What is the difference between stopping and suspending a VM?
**A:** Stopping releases CPU and memory but preserves the disk. Suspending saves the memory state to disk, allowing the VM to resume exactly where it left off.

---
### Q5: What are snapshots in Compute Engine?
**A:** Point-in-time, incremental copies of persistent disk data. They are global resources used for backups, disaster recovery, and disk cloning. Only changes since the last snapshot are stored.

---
### Q6: What happens when you delete a snapshot that is part of an incremental chain?
**A:** Nothing is lost. Google automatically transfers any data needed by later snapshots. Deleting a middle snapshot does not affect other snapshots or disks created from them.

---
### Q7: What gcloud command creates an automated daily snapshot schedule with 14-day retention?
**A:** `gcloud compute resource-policies create snapshot-schedule my-schedule --region=us-central1 --max-retention-days=14 --daily-schedule --start-time=02:00`

---
### Q8: What is the difference between a snapshot and a custom image?
**A:** Snapshots are incremental backups of any persistent disk (boot or data), used for disaster recovery. Images are full copies of boot disks used to create new VMs. Images support families; snapshots do not. Creating a VM from an image is faster than from a snapshot.

---
### Q9: What is an image family in Compute Engine?
**A:** A group of related images (e.g., different versions of an application). When you reference a family, you always get the latest non-deprecated image. Useful in instance templates for automatic updates.

---
### Q10: What gcloud command creates a custom image from a VM's boot disk?
**A:** `gcloud compute images create my-custom-image --source-disk=my-vm --source-disk-zone=us-central1-a`. The VM should be stopped first for consistency.

---
### Q11: What is Artifact Registry?
**A:** Google's managed container image and package repository, the successor to Container Registry (gcr.io). It supports Docker, Maven, npm, Python, Go, and other formats.

---
### Q12: What is the recommended way for GKE to access images in Artifact Registry across projects?
**A:** Use Workload Identity to map the pod's Kubernetes service account to a Google Cloud service account that has `roles/artifactregistry.reader` in the target project.

---
### Q13: What is a GKE node pool?
**A:** A group of nodes within a cluster that share the same configuration (machine type, disk size, labels, taints). A cluster can have multiple node pools with different configurations.

---
### Q14: What gcloud command resizes a GKE node pool manually?
**A:** `gcloud container clusters resize my-cluster --node-pool=my-pool --num-nodes=5 --zone=us-central1-a`

---
### Q15: What is the difference between a Kubernetes Deployment and a StatefulSet?
**A:** Deployments manage stateless pods with random names and shared storage. StatefulSets manage stateful applications with stable network identities (pod-0, pod-1), individual persistent volumes per pod, and ordered deployment/scaling.

---
### Q16: A pod is stuck in "Pending" state and kubectl describe shows "Insufficient cpu". What should you do?
**A:** The cluster lacks CPU capacity. Either resize the node pool manually or enable the cluster autoscaler to automatically add nodes when pods are unschedulable.

---
### Q17: What is the difference between HPA, VPA, and the Cluster Autoscaler?
**A:** HPA scales the number of pod replicas based on metrics. VPA adjusts CPU/memory requests per pod based on historical usage. Cluster Autoscaler scales the number of nodes when pods cannot be scheduled.

---
### Q18: What are the VPA update modes?
**A:** Off (recommendations only), Initial (sets resources at pod creation only), Recreate (evicts and recreates pods with new resources), and Auto (similar to Recreate, applies in-place when available).

---
### Q19: What kubectl command creates an HPA that scales a deployment between 2 and 10 replicas based on 70% CPU?
**A:** `kubectl autoscale deployment my-app --min=2 --max=10 --cpu-percent=70`

---
### Q20: How do HPA and Cluster Autoscaler work together?
**A:** When traffic increases, HPA creates more pods. If there is not enough node capacity, pods become unschedulable. The Cluster Autoscaler then adds nodes so the pods can be scheduled.

---
### Q21: What is a Cloud Run revision?
**A:** An immutable snapshot of a service's configuration (code + settings). Each deployment creates a new revision. Traffic can be split across multiple revisions for canary deployments and rollbacks.

---
### Q22: How do you deploy a new Cloud Run version without sending it any traffic?
**A:** Use the `--no-traffic` flag: `gcloud run deploy my-service --image=IMAGE --region=us-central1 --no-traffic`. Optionally add `--tag=canary` to create a named URL for testing.

---
### Q23: What gcloud command instantly rolls back a Cloud Run service to a previous revision?
**A:** `gcloud run services update-traffic my-service --region=us-central1 --to-revisions=PREVIOUS_REVISION=100`

---
### Q24: What does the --min-instances parameter control in Cloud Run?
**A:** The minimum number of instances always running. Setting it to 0 enables scale-to-zero (lowest cost). Setting it to 1+ keeps warm instances to avoid cold starts.

---
### Q25: What is the difference between CPU throttled and CPU always allocated in Cloud Run?
**A:** CPU throttled (default) allocates CPU only during request processing and charges per request. CPU always allocated keeps CPU available between requests and is needed for WebSockets, background processing, or long-lived connections.

---
### Q26: What is the default concurrency setting for a Cloud Run instance?
**A:** 80 concurrent requests per instance. Higher concurrency means fewer instances needed (lower cost); lower concurrency means more isolation.

---
### Q27: What is Uniform Bucket-Level Access in Cloud Storage?
**A:** A setting that disables object-level ACLs and enforces all access control through IAM only. This simplifies access management and is the recommended approach.

---
### Q28: What is a signed URL in Cloud Storage?
**A:** A URL that provides time-limited access to a specific object without requiring authentication. It can be configured for GET (download) or PUT (upload) and is useful for sharing with external users.

---
### Q29: What is the purpose of a Cloud Storage lifecycle policy?
**A:** To automate storage management by transitioning objects to cheaper storage classes (e.g., Standard to Nearline after 30 days) or deleting objects after a specified time, reducing cost automatically.

---
### Q30: A regulation requires files to be retained for 7 years and never deleted early. How do you enforce this in Cloud Storage?
**A:** Set a retention policy of 7 years on the bucket, then lock it with `--lock-retention-period`. A locked retention policy is irreversible -- it cannot be shortened or removed.

---
### Q31: What bq command estimates the cost of a BigQuery query without running it?
**A:** `bq query --use_legacy_sql=false --dry_run 'SELECT ...'`. It returns the number of bytes that would be scanned without executing the query or incurring charges.

---
### Q32: What is required for Cloud SQL point-in-time recovery (PITR)?
**A:** Automated backups must be enabled (`--backup-start-time`) and binary logging enabled for MySQL (`--enable-bin-log`) or WAL archiving for PostgreSQL.

---
### Q33: What is the difference between cancelling and draining a Dataflow streaming job?
**A:** Cancel stops the job immediately. Drain processes all remaining in-flight data in the pipeline before stopping, ensuring no data is lost.

---
### Q34: True or False: Firestore has built-in automated backup scheduling.
**A:** False. You must implement automated backups using Cloud Scheduler to trigger a Cloud Function that runs `gcloud firestore export` on a schedule.

---
### Q35: What gcloud command expands a subnet's IP range without downtime?
**A:** `gcloud compute networks subnets expand-ip-range my-subnet --region=us-central1 --prefix-length=20`. You can only expand (make larger), never shrink a subnet.

---
### Q36: What is the difference between a static external IP and an ephemeral external IP?
**A:** A static external IP persists independently of the VM lifecycle (even when stopped/deleted). An ephemeral IP is released when the VM is stopped and a new one is assigned on restart.

---
### Q37: True or False: Unused reserved static IP addresses incur charges.
**A:** True. Google Cloud charges for reserved static IPs that are not assigned to a running resource. Release them when no longer needed.

---
### Q38: What is Cloud DNS?
**A:** A managed authoritative DNS service with 100% uptime SLA. It supports public zones, private zones (VPC-only), forwarding zones (to on-premises DNS), and peering zones.

---
### Q39: What gcloud command creates a DNS forwarding zone to on-premises DNS servers?
**A:** `gcloud dns managed-zones create on-prem-zone --dns-name=corp.example.com. --visibility=private --networks=my-vpc --forwarding-targets=10.0.0.53,10.0.0.54`

---
### Q40: What is Cloud NAT?
**A:** A fully managed, software-defined NAT service that provides outbound internet access for VMs without external IPs. It is a regional resource associated with a Cloud Router and does not allow inbound connections.

---
### Q41: What are the four types of Google Cloud audit logs?
**A:** Admin Activity (always on, free), Data Access (must be enabled, charged), System Event (always on, free), and Policy Denied (always on, free).

---
### Q42: True or False: Admin Activity audit logs can be disabled.
**A:** False. Admin Activity audit logs are always enabled and cannot be disabled. They log all API calls that modify resources and are free of charge.

---
### Q43: What is a log sink in Cloud Logging?
**A:** A resource that routes log entries matching a filter to a destination. Supported destinations are Cloud Storage (archival), BigQuery (analysis), Pub/Sub (real-time processing), and Cloud Logging buckets.

---
### Q44: What must you do after creating a log sink for it to work?
**A:** Grant the sink's writer identity (service account) write access to the destination. Get the identity with `gcloud logging sinks describe SINK_NAME --format="get(writerIdentity)"`.

---
### Q45: What are the default Cloud Logging log buckets and their retention?
**A:** _Required bucket: stores admin activity and system event audit logs for 400 days (not configurable). _Default bucket: stores all other logs for 30 days (configurable).

---
### Q46: What is the Ops Agent?
**A:** A unified agent that collects both logs and metrics from VMs, replacing the legacy separate Monitoring and Logging agents. It is based on Fluent Bit (logs) and OpenTelemetry Collector (metrics).

---
### Q47: What is a log-based metric?
**A:** A custom Cloud Monitoring metric created from log entries. For example, a counter metric that counts log entries matching `severity>=ERROR`. It can be used in alert policies and dashboards.

---
### Q48: How do you export error logs to an on-premises SIEM system in real-time?
**A:** Create a log sink to a Pub/Sub topic with `--log-filter='severity>=ERROR'`, then configure the SIEM to subscribe to that Pub/Sub topic.

---
### Q49: What is Managed Service for Prometheus?
**A:** A Google-managed Prometheus-compatible monitoring solution that stores metrics in Google Cloud's Monarch backend. It is a drop-in replacement for self-managed Prometheus, queryable with PromQL or Cloud Monitoring.

---
### Q50: You need to investigate who deleted a VM in your project. Which logs do you check and what filter do you use?
**A:** Check Admin Activity audit logs: `gcloud logging read 'logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Factivity" AND protoPayload.methodName="v1.compute.instances.delete"'`. These logs are always enabled and free.

---
### Q51: What is Database Center and which databases does it cover?
**A:** Database Center is a unified GCP console for managing all database services in one place — Cloud SQL, AlloyDB, Spanner, Bigtable, Firestore, and Memorystore. It shows fleet health, query performance, and recommendations. It is read-only; changes are made via each service's own console/CLI.

---
### Q52: How do you perform a point-in-time restore on AlloyDB?
**A:** `gcloud alloydb clusters restore RESTORED_CLUSTER --region=REGION --source-cluster=CLUSTER --point-in-time=2025-06-01T10:00:00Z`. AlloyDB has continuous backup enabled by default, so PITR is always available.

---
### Q53: What gcloud command creates a Spanner backup with 7-day retention?
**A:** `gcloud spanner backups create my-backup --instance=INST --database=DB --retention-period=7d`. Unlike Cloud SQL, Spanner backups are on-demand only (no automatic scheduled backups built-in, though you can schedule via Cloud Scheduler).

---
### Q54: What is Query Insights and what problem does it solve?
**A:** Query Insights is a diagnostic tool (available for Cloud SQL and AlloyDB) that identifies slow queries, shows execution plans, suggests missing indexes via Index Advisor, and visualizes query performance over time. It solves the "my database is slow, but I don't know which query" problem.

---
### Q55: What is the difference between Personalized Service Health and the public GCP status page?
**A:** The public status page (status.cloud.google.com) shows incidents affecting all GCP users. Personalized Service Health (in Cloud Console) shows only incidents affecting services **your projects are actually using**, making it much more actionable for troubleshooting.

---
### Q56: What is Active Assist and give three examples of its recommenders.
**A:** Active Assist is GCP's AI-powered recommendation engine. Examples: (1) IAM Recommender — finds over-provisioned roles, (2) VM Rightsizing Recommender — finds over/under-utilized VMs, (3) Firewall Insights — finds unused or shadowed firewall rules.

---
### Q57: In GKE Autopilot, what are the default resource requests applied to a pod if you don't specify any?
**A:** 0.5 vCPU and 2 GiB memory. In Autopilot you are billed per-pod based on resource requests, so unset requests lead to higher costs than necessary. Always set explicit `requests` to match actual application needs.

---
### Q58: You have a GKE Autopilot pod that keeps getting OOMKilled. What should you do?
**A:** Increase the memory `request` and `limit` in the pod spec. In Autopilot, you cannot modify node size — the solution is always to adjust pod resource requests. You can also use VPA in `Off` (recommendation) mode to get sizing suggestions without automatic changes.
