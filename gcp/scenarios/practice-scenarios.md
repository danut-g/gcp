# Google Associate Cloud Engineer -- Practice Scenarios

> **28 realistic mini case studies** covering all 5 exam sections.
> Use these to test your understanding, not just memorization.
> Each scenario is designed to mirror the style and difficulty of real exam questions.

---

## Section Map

| Theme | Scenarios |
|-------|-----------|
| Choosing the right compute option | 1, 2, 3, 4, 5 |
| Choosing the right database/storage | 6, 7, 8, 9 |
| Networking decisions: VPC, LB, VPN, peering | 10, 11, 12 |
| Deployment strategies: MIGs, GKE, Cloud Run | 13, 14, 15 |
| IAM and security: roles, service accounts, least privilege | 16, 17, 18, 19 |
| Operations: monitoring, logging, backups, scaling | 20, 21, 22 |
| Billing and cost optimization | 23, 24 |
| Infrastructure as Code | 25 |
| Multi-topic scenarios that combine concepts | 26, 27, 28 |

---

## Scenario 1: Startup With Unpredictable Traffic

**Situation:** A two-person startup is launching a new REST API for their mobile app. They expect traffic to be extremely variable -- potentially zero requests at 3 AM, but spikes of thousands of requests during marketing campaigns. The team has no experience with Kubernetes and wants to minimize both cost and operational burden.

**Requirements:**
- Scale to zero when there is no traffic
- Handle sudden traffic spikes without manual intervention
- Deploy containerized Go application with 4 HTTP endpoints
- Keep costs as low as possible when idle

**Question:** Which compute service should the startup use?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Cloud Run

**Why:** Cloud Run scales to zero (no cost when idle), automatically handles traffic spikes, and supports multi-endpoint containerized applications without Kubernetes knowledge. Cloud Functions would be wrong because the application has multiple endpoints (Cloud Functions is best for single-purpose functions). GKE would add unnecessary operational overhead for a 2-person team. Compute Engine would require manual scaling configuration and would not scale to zero.

**Key services/concepts:** Cloud Run, scale-to-zero, serverless containers, per-request pricing

</details>

---

## Scenario 2: Legacy Windows Application Migration

**Situation:** A manufacturing company is migrating a legacy Windows-based ERP system to Google Cloud. The application requires Windows Server 2022, specific .NET Framework 4.8 libraries, and connects to a local SQL Server database via named pipes. The application cannot be refactored or containerized due to vendor restrictions.

**Requirements:**
- Windows Server 2022 operating system
- Full control over OS configuration and installed libraries
- Persistent local storage for temporary processing files
- Application runs 24/7 serving internal users

**Question:** Which compute service should they use, and what machine configuration is most appropriate?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Compute Engine with a general-purpose machine type (e.g., N2 or E2 family) running a Windows Server 2022 image from the `windows-cloud` project.

**Why:** Compute Engine is the only option that provides full OS control with Windows support. Cloud Run, Cloud Functions, and GKE do not support Windows Server workloads with this level of OS customization. Since the app runs 24/7, they should also consider Committed Use Discounts for cost savings. A custom machine type could be used if the app needs a non-standard CPU/memory ratio.

**Key services/concepts:** Compute Engine, Windows images, IaaS, lift-and-shift migration, Committed Use Discounts

</details>

---

## Scenario 3: Batch Video Rendering

**Situation:** A media production company runs video rendering jobs that take 2-6 hours each. They submit 50-100 jobs per day during the work week but nearly zero on weekends. Each job can be checkpointed every 15 minutes, and if interrupted, can resume from the last checkpoint. The company is highly cost-sensitive.

**Requirements:**
- Minimize compute cost for long-running batch jobs
- Jobs must tolerate interruptions and resume from checkpoints
- Cluster of VMs should scale based on the job queue
- Jobs need high CPU count (32+ vCPUs per job)

**Question:** What compute configuration would minimize costs while meeting these requirements?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Compute Engine Spot VMs in a Managed Instance Group (MIG) with compute-optimized machine types (C2 family). Use autoscaling based on a custom Cloud Monitoring metric that tracks the job queue depth.

**Why:** Spot VMs provide 60-91% discount over on-demand pricing. Since jobs support checkpointing and can resume after interruption, Spot VMs are ideal. A MIG with autoscaling automatically replaces preempted instances and scales based on demand. Cloud Functions and Cloud Run have a 60-minute timeout limit, which is insufficient for 2-6 hour jobs. C2 machines are optimized for CPU-intensive workloads like rendering.

**Key services/concepts:** Spot VMs, Managed Instance Groups, autoscaling, compute-optimized machine types, checkpointing, custom metrics

</details>

---

## Scenario 4: Microservices With Complex Orchestration

**Situation:** An e-commerce company has 15 microservices that need to communicate with each other. The team uses Kubernetes extensively in their on-premises data center and wants to replicate this architecture in the cloud. They need rolling updates, automatic pod healing, service discovery, and the ability to run GPU-based recommendation workloads alongside standard web services.

**Requirements:**
- Kubernetes-native orchestration with 15+ services
- GPU node pool for ML inference workloads
- Rolling updates with zero downtime
- Team already proficient with kubectl and Helm

**Question:** Which compute platform should they use, and should they choose Standard or Autopilot mode?

<details>
<summary>Click to reveal answer</summary>

**Answer:** GKE Standard mode.

**Why:** GKE is the natural choice for teams already using Kubernetes. Standard mode is required here because GPU node pools need custom node configurations (machine types, accelerator types, taints) that Autopilot does not fully support. Autopilot would be simpler but does not give the same level of control over node pool configuration needed for GPU workloads. Cloud Run would not provide the inter-service communication and orchestration features the team needs across 15 services.

**Key services/concepts:** GKE Standard, node pools, GPU accelerators, rolling updates, Helm, service discovery

</details>

---

## Scenario 5: Image Thumbnail Generator

**Situation:** A photo sharing platform needs to generate thumbnails whenever a user uploads an image to Cloud Storage. Each thumbnail generation takes about 5 seconds. The platform sees 10,000-50,000 uploads per day, spread unevenly throughout the day. The thumbnail function is a simple Python script that reads the uploaded image, resizes it, and writes the result back to a different bucket.

**Requirements:**
- Triggered automatically when a new image is uploaded to Cloud Storage
- Simple, single-purpose processing (resize image)
- No infrastructure management
- Cost-effective for variable workload

**Question:** Which compute service and trigger mechanism should be used?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Cloud Functions Gen 2 with an Eventarc trigger on the `google.cloud.storage.object.v1.finalized` event type.

**Why:** This is a textbook Cloud Functions use case: single-purpose function, short execution time (5 seconds), event-driven by Cloud Storage uploads. Gen 2 is recommended over Gen 1 because it supports higher concurrency (multiple requests per instance), which is beneficial at 10K-50K events per day. Cloud Run would also work but adds unnecessary complexity for a single-purpose function. Compute Engine would be wasteful because most of the time the VM would be idle.

**Key services/concepts:** Cloud Functions Gen 2, Eventarc, Cloud Storage triggers, event-driven architecture, serverless

</details>

---

## Scenario 6: Global Financial Trading Platform

**Situation:** A financial services company is building a trading platform that must serve users across North America, Europe, and Asia. The system handles millions of transactions per day, requires strong consistency for all reads (a trader must never see stale data), and must maintain 99.999% availability. The data is relational with complex join queries across orders, accounts, and positions tables.

**Requirements:**
- Global distribution across three continents
- Strong consistency for all reads and writes
- Relational data model with complex SQL joins
- 99.999% availability SLA
- Millions of transactions per day

**Question:** Which database service should they choose?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Cloud Spanner with a multi-region configuration.

**Why:** Cloud Spanner is the only Google Cloud database that provides globally distributed strong consistency with a 99.999% SLA. It supports SQL with relational schemas and ACID transactions across regions. Cloud SQL cannot scale horizontally across regions and has a lower SLA (99.95-99.99%). BigQuery is for analytics (OLAP), not transactional workloads. Firestore does not support complex relational joins. The high cost of Spanner is justified by the five-nines availability and global consistency requirements.

**Key services/concepts:** Cloud Spanner, multi-region configuration, strong consistency, ACID transactions, 99.999% SLA

</details>

---

## Scenario 7: IoT Sensor Data Pipeline

**Situation:** An agriculture technology company deploys 50,000 soil sensors across thousands of farms. Each sensor sends temperature, moisture, and pH readings every 30 seconds. The data must be ingested at low latency, stored for real-time dashboards (sub-second queries), and also available for historical batch analysis. The data volume is approximately 2 TB per day.

**Requirements:**
- Ingest millions of data points per minute
- Sub-millisecond read latency for real-time dashboards
- Time-series data optimized for range scans
- Scale linearly as sensor count grows
- Historical data available for batch analytics

**Question:** Which primary database should store the real-time sensor data, and how should historical analytics be handled?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Cloud Bigtable for real-time data storage and serving, with data also exported to BigQuery for historical batch analytics.

**Why:** Bigtable is purpose-built for high-throughput, low-latency time-series data. It handles millions of writes per second with consistent millisecond latency and scales linearly by adding nodes. The row key can be designed for efficient time-range scans (e.g., `farm_id#sensor_id#reverse_timestamp`). Firestore would not handle this volume efficiently (it is designed for mobile/web apps, not massive time-series ingestion). Cloud SQL cannot scale to this write throughput. BigQuery is excellent for analytics but has too much latency for real-time dashboards -- however it is ideal for the historical analysis component.

**Key services/concepts:** Cloud Bigtable, BigQuery, time-series data, row key design, horizontal scaling, Dataflow (for ETL)

</details>

---

## Scenario 8: E-Commerce Product Catalog

**Situation:** A mid-sized e-commerce company needs to store product information (name, description, price, categories, variants, reviews) for 500,000 products. The web and mobile applications need to display product pages with sub-50ms response times. The data is semi-structured -- different product categories have different attributes (electronics have specifications, clothing has sizes/colors). The company expects to grow to 2 million products within 2 years.

**Requirements:**
- Semi-structured data with flexible schemas per product category
- Sub-50ms read latency for product page rendering
- Real-time updates when prices or inventory change
- Mobile app needs offline capability and real-time sync
- Data volume under 500 GB currently

**Question:** Which database service is the best fit?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Firestore in Native mode.

**Why:** Firestore Native mode is ideal for semi-structured document data (products as documents with varying fields per category). It provides real-time listeners for instant price/inventory updates, offline support for the mobile app, and sub-millisecond read latency. The data volume (under 1 TB) fits within Firestore's design parameters. Cloud SQL would require complex schema design to handle varying product attributes. Bigtable is overkill for this data volume and does not support the document model well. Spanner would be unnecessarily expensive for a single-region, moderate-scale workload.

**Key services/concepts:** Firestore Native mode, document database, real-time listeners, offline support, semi-structured data

</details>

---

## Scenario 9: Long-Term Compliance Archive

**Situation:** A healthcare company must retain patient record backups for 7 years due to regulatory requirements. The data is generated monthly (approximately 500 GB per month) and is almost never accessed -- only in the rare event of a legal or compliance audit (estimated once per year or less). When accessed, a retrieval delay of a few seconds is acceptable.

**Requirements:**
- Minimize storage costs over 7 years
- Data must be retained and cannot be deleted before the retention period
- Retrieval within seconds when needed (not hours)
- Approximately 500 GB added per month

**Question:** Which storage class and configuration should be used?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Cloud Storage with Archive storage class, a locked retention policy of 7 years (2555 days), and uniform bucket-level access.

**Why:** Archive class has the lowest storage cost, which is critical for 42 TB+ of data over 7 years. Unlike AWS Glacier, Cloud Storage Archive class still provides millisecond access latency, meeting the "retrieval within seconds" requirement. The locked retention policy ensures compliance -- once locked, it cannot be shortened or removed, guaranteeing data cannot be deleted early. Coldline would also work but costs more per GB for storage. Nearline's 30-day minimum is wasteful for data accessed less than once per year. The locked retention policy is the key differentiator for compliance scenarios.

**Key services/concepts:** Cloud Storage Archive class, locked retention policy, minimum storage duration, uniform bucket-level access, compliance

</details>

---

## Scenario 10: Multi-Region Web Application Load Balancing

**Situation:** A SaaS company serves customers globally from the United States, Europe, and Asia. They have Compute Engine instances running their web application in us-central1, europe-west1, and asia-east1. They want users to be automatically routed to the nearest healthy region, with SSL termination and DDoS protection. Some URL paths (`/api/*`) should go to the API backend, while `/static/*` should be served from Cloud Storage.

**Requirements:**
- Global traffic distribution to the nearest healthy region
- URL-path-based routing to different backends
- SSL/TLS termination at the load balancer
- DDoS protection
- Single anycast IP address

**Question:** Which load balancer should they use, and what additional services should be integrated?

<details>
<summary>Click to reveal answer</summary>

**Answer:** External Application Load Balancer (Global HTTP/S LB) with Cloud CDN for static content and Cloud Armor for DDoS protection.

**Why:** The External Application LB is the only load balancer that provides all of these features: global distribution with a single anycast IP, L7 URL-path-based routing (URL maps), SSL termination, and integration with Cloud CDN and Cloud Armor. An External Passthrough Network LB is regional only and L4 (no URL routing). An Internal Application LB is for internal traffic only. The Premium network tier (default) is required for global load balancing. Cloud CDN caches the `/static/*` content at edge locations for even lower latency.

**Key services/concepts:** External Application Load Balancer, URL maps, Cloud CDN, Cloud Armor, Premium network tier, anycast IP

</details>

---

## Scenario 11: Secure Hybrid Cloud Connectivity

**Situation:** A retail company has an on-premises data center with critical inventory and POS systems. They are extending to Google Cloud for new workloads. The connection must be encrypted, provide 99.99% SLA, and support dynamic routing as they add new subnets in both environments. Expected bandwidth usage is about 500 Mbps during peak hours. Budget is moderate -- they cannot justify the cost and lead time of a dedicated physical connection.

**Requirements:**
- Encrypted connection between on-premises and GCP
- 99.99% availability SLA
- Dynamic routing (BGP) to automatically learn new routes
- Bandwidth needs up to 500 Mbps
- Cost-effective setup within days, not weeks

**Question:** Which connectivity option should they choose?

<details>
<summary>Click to reveal answer</summary>

**Answer:** HA VPN (High Availability VPN) with Cloud Router for dynamic BGP routing.

**Why:** HA VPN provides 99.99% SLA (when configured with two tunnels), IPsec encryption, and dynamic routing via BGP with Cloud Router. It can be set up in hours/days, not weeks. Dedicated Interconnect would provide higher bandwidth and lower latency but requires weeks to provision, a physical connection, is more expensive, and does not encrypt traffic by default. Partner Interconnect is also not encrypted by default. Classic VPN only offers 99.9% SLA. Since the bandwidth need (500 Mbps) is well within HA VPN's capacity of up to 3 Gbps per tunnel, HA VPN is the right choice.

**Key services/concepts:** HA VPN, Cloud Router, BGP dynamic routing, IPsec encryption, 99.99% SLA

</details>

---

## Scenario 12: Cross-Project Resource Communication

**Situation:** A company has two GCP projects: project-frontend (web application) and project-backend (API services and databases). VMs in project-frontend need to communicate with VMs in project-backend over internal IP addresses. The networking team wants to keep traffic on Google's private network, avoid traversing the public internet, and ensure IP ranges do not overlap. They do not need transitive connectivity to any third network.

**Requirements:**
- Communication between VMs in two different projects using internal IPs
- Traffic must stay on Google's private network
- Simple setup without organizational complexity
- No need for transitive connectivity

**Question:** Which networking solution should they implement?

<details>
<summary>Click to reveal answer</summary>

**Answer:** VPC Network Peering between the VPCs in project-frontend and project-backend.

**Why:** VPC peering connects two VPC networks so resources can communicate using internal IPs, with traffic staying on Google's backbone. It works across projects, is simple to set up (create peering from both sides), and has no bandwidth restrictions. Shared VPC would also work but requires an organization node and is more complex to set up (host/service project model). Since they only need connectivity between two projects and do not need transitive routing, VPC peering is the simpler solution. The key constraint to remember is that VPC peering is non-transitive and requires non-overlapping IP ranges.

**Key services/concepts:** VPC Network Peering, internal IPs, non-transitive routing, Google private backbone

</details>

---

## Scenario 13: Zero-Downtime Web Server Fleet Update

**Situation:** A company runs a fleet of 20 identical web servers using a Managed Instance Group behind an External Application Load Balancer. They have a new version of their application baked into a new custom image. They want to update all instances to the new image with zero downtime. They also want to test the new version with a small percentage of instances first.

**Requirements:**
- Update all 20 instances to the new image
- Zero downtime during the update
- Test with 20% of instances before full rollout
- Automatic rollback capability if health checks fail

**Question:** How should they perform this update?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create a new instance template with the new image, then perform a canary rolling update on the MIG using `gcloud compute instance-groups managed rolling-action start-update` with `--canary-version=template=new-template,target-size=20%` and `--max-unavailable=0`.

**Why:** Instance templates are immutable, so a new template must be created. The canary rolling update deploys the new template to 20% of instances first while keeping all instances available (`--max-unavailable=0`). The MIG's health check automatically detects if new instances are unhealthy. If the canary looks good, the update can proceed to 100%. Setting `--max-unavailable=0` with a `--max-surge` value ensures zero downtime by creating new instances before terminating old ones. Direct VM updates would require manual work and risk downtime.

**Key services/concepts:** Managed Instance Groups, instance templates (immutable), canary rolling update, max-unavailable, max-surge, health checks

</details>

---

## Scenario 14: Canary Deployment for API Service

**Situation:** A fintech company runs a payment processing API on Cloud Run. They have a new version that changes the fraud detection algorithm. They want to send only 5% of production traffic to the new version initially, monitor error rates and latency for 24 hours, and then gradually increase to 100% if metrics are healthy. They also want a dedicated URL to test the new revision without affecting production traffic.

**Requirements:**
- Deploy new version without immediately receiving production traffic
- Test the new revision via a dedicated URL
- Gradually shift traffic: 5% then 25% then 100%
- Instant rollback if issues are detected

**Question:** How should they deploy and manage traffic for the new version?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Deploy the new revision with `--no-traffic` and `--tag=canary`. Test using the tagged URL (e.g., `https://canary---service-xxx.a.run.app`). Then gradually shift traffic using `gcloud run services update-traffic --to-revisions=old=95,new=5`, then `old=75,new=25`, then `new=100`.

**Why:** The `--no-traffic` flag creates the revision without routing any production traffic to it. The `--tag=canary` creates a dedicated URL for internal testing. Cloud Run's traffic splitting is percentage-based and instantaneous, making gradual rollouts and instant rollbacks trivial. For rollback, simply route 100% back to the old revision. This approach is superior to blue/green deployment on Compute Engine because it requires no additional infrastructure and traffic shifts are instant.

**Key services/concepts:** Cloud Run revisions, traffic splitting, --no-traffic, revision tags, canary deployment, instant rollback

</details>

---

## Scenario 15: GKE Cluster With Mixed Workload Types

**Situation:** A data science company runs a GKE cluster with two types of workloads: (1) web API services that need to be always available and scale with traffic, and (2) batch ML training jobs that are cost-sensitive and can tolerate interruptions. They want to optimize costs by using cheaper nodes for the batch jobs while keeping the web services on reliable nodes.

**Requirements:**
- Web services on reliable, always-available nodes
- Batch ML jobs on cost-optimized nodes that may be reclaimed
- Prevent batch jobs from being scheduled on web service nodes
- Automatic scaling of nodes based on pending pods

**Question:** How should they configure the GKE cluster?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create two node pools: (1) a standard node pool for web services with autoscaling enabled, and (2) a Spot VM node pool for batch ML jobs. Apply node taints to the Spot pool (e.g., `dedicated=batch:NoSchedule`) and add corresponding tolerations to the batch job pod specs. Enable the cluster autoscaler on both pools.

**Why:** Spot VMs provide 60-91% discount for fault-tolerant workloads. Node taints on the Spot pool prevent web service pods from being accidentally scheduled there (only pods with matching tolerations will be scheduled). Node labels can be used with nodeSelectors or nodeAffinity for additional scheduling control. The cluster autoscaler will automatically add/remove nodes in each pool based on pending pods. This architecture cleanly separates workload types while optimizing costs.

**Key services/concepts:** GKE node pools, Spot VMs, node taints and tolerations, cluster autoscaler, workload separation

</details>

---

## Scenario 16: Developer Access Control for Multiple Teams

**Situation:** An organization has three development teams: Frontend, Backend, and Data. Each team should only manage resources relevant to their work. Frontend developers need to deploy to Cloud Run. Backend developers need to manage Compute Engine VMs and Cloud SQL instances. Data engineers need to run BigQuery queries and manage Cloud Storage buckets. No team should have access to modify IAM policies or billing.

**Requirements:**
- Frontend: deploy and manage Cloud Run services
- Backend: manage VMs and Cloud SQL
- Data: query BigQuery and manage Cloud Storage
- No team should modify IAM or billing
- Use the principle of least privilege

**Question:** Which IAM roles should be assigned to each team, and how should access be organized?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create three Google Groups (one per team) and assign predefined roles to each group at the project level:
- Frontend group: `roles/run.developer`
- Backend group: `roles/compute.instanceAdmin.v1` + `roles/cloudsql.editor`
- Data group: `roles/bigquery.user` + `roles/bigquery.jobUser` + `roles/storage.admin`

**Why:** Using Google Groups is the best practice for managing access at scale (not individual user bindings). Predefined roles provide fine-grained, service-specific permissions without the breadth of basic roles. `roles/editor` would be wrong because it grants access to all services and would violate the separation of duties requirement. None of these predefined roles include IAM or billing management permissions. If a developer joins or leaves a team, simply add/remove them from the group.

**Key services/concepts:** Google Groups, predefined IAM roles, principle of least privilege, separation of duties

</details>

---

## Scenario 17: Secure VM Without External IP

**Situation:** A security-conscious bank runs application VMs that must not have external IP addresses (enforced by organization policy `constraints/compute.vmExternalIpAccess`). Developers still need SSH access to these VMs for troubleshooting. The VMs also need to download security patches from the internet and access Cloud Storage APIs.

**Requirements:**
- VMs must have no external IP addresses
- Developers must be able to SSH into VMs
- VMs must access the internet for package updates
- VMs must access Google Cloud APIs (Cloud Storage, etc.)
- All access must be auditable

**Question:** What combination of services enables SSH access, internet access, and API access for VMs without external IPs?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Use Identity-Aware Proxy (IAP) tunneling for SSH access, Cloud NAT for outbound internet access (package updates), and Private Google Access on the subnet for accessing Google Cloud APIs. Grant developers `roles/iap.tunnelResourceAccessor` and create a firewall rule allowing TCP from `35.235.240.0/20` on port 22.

**Why:** IAP tunneling provides SSH without external IPs and is fully auditable through IAM and audit logs. Cloud NAT provides outbound-only internet access (VMs can download packages but cannot be reached from the internet). Private Google Access allows VMs with only internal IPs to reach Google APIs like Cloud Storage. This is a common exam pattern: the combination of IAP + Cloud NAT + Private Google Access covers all three connectivity needs for private VMs. Using a bastion host would be an older pattern that adds operational overhead.

**Key services/concepts:** Identity-Aware Proxy (IAP), Cloud NAT, Private Google Access, organization policy, firewall rules, `35.235.240.0/20`

</details>

---

## Scenario 18: Service Account for Automated Pipeline

**Situation:** A CI/CD pipeline running on a Compute Engine VM needs to build Docker images, push them to Artifact Registry, and deploy them to Cloud Run. The security team insists on following least privilege and prohibits the use of service account keys. The VM currently uses the default Compute Engine service account, which has `roles/editor`.

**Requirements:**
- Push container images to Artifact Registry
- Deploy services to Cloud Run
- No service account keys
- Least privilege (remove excessive permissions from default SA)
- Auditable

**Question:** How should the service account be configured?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create a new dedicated service account (e.g., `cicd-pipeline@project.iam.gserviceaccount.com`) with three specific roles: `roles/artifactregistry.writer` (push images), `roles/run.developer` (deploy to Cloud Run), and `roles/iam.serviceAccountUser` (attach service accounts to Cloud Run services). Assign this SA to the VM with `--scopes=cloud-platform`. Do not use the default SA.

**Why:** The default Compute Engine SA has `roles/editor`, which grants thousands of unnecessary permissions -- a major security risk. Creating a dedicated SA with only the needed roles follows least privilege. No keys are needed because the SA is attached directly to the VM (the VM obtains credentials from the metadata server). `roles/iam.serviceAccountUser` is needed because deploying to Cloud Run requires attaching a service account to the service. Using `--scopes=cloud-platform` is correct because IAM roles (not scopes) should control access.

**Key services/concepts:** Dedicated service accounts, least privilege, attached service accounts (no keys), `roles/iam.serviceAccountUser`, default SA risk, scopes vs. IAM roles

</details>

---

## Scenario 19: Temporary Contractor Access

**Situation:** A company hires a contractor for a 3-month project ending March 31, 2026. The contractor needs to view and create Compute Engine instances in the `dev-project` only, and must be able to SSH into the VMs they create. They should not be able to delete instances, modify networking, or access any production projects. The company wants access to automatically expire.

**Requirements:**
- Create and view VMs in dev-project only (no deletion)
- SSH access to VMs
- Access expires automatically on March 31, 2026
- No access to production projects or networking

**Question:** How should this access be configured?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create a custom IAM role with only the needed permissions (`compute.instances.create`, `compute.instances.get`, `compute.instances.list`, `compute.disks.create`, `compute.subnetworks.use`, `compute.subnetworks.useExternalIp`) and grant it with a conditional IAM binding that includes a time-based expiration: `--condition='expression=request.time < timestamp("2026-03-31T23:59:59Z"),title=contractor-expires-march'`. Also grant `roles/compute.osLogin` for SSH access with the same condition.

**Why:** A custom role ensures only the exact permissions needed (no delete, no networking changes). The IAM condition with a timestamp expression causes the binding to automatically stop being effective after March 31 -- no manual cleanup required. `roles/compute.instanceAdmin.v1` would be too broad because it includes delete and instance management permissions. Granting at the project level (dev-project only) ensures no access to production. OS Login with `roles/compute.osLogin` provides SSH access without managing SSH keys.

**Key services/concepts:** Custom IAM roles, conditional IAM bindings, time-based conditions, OS Login, principle of least privilege

</details>

---

## Scenario 20: Proactive CPU Monitoring and Alerting

**Situation:** A DevOps team manages 50 Compute Engine VMs running a distributed application. They have experienced outages when VMs silently max out CPU without anyone noticing. They want to be alerted when any VM's CPU utilization exceeds 85% for more than 5 minutes. Alerts should go to both a Slack channel and the on-call email address.

**Requirements:**
- Alert on CPU > 85% sustained for 5+ minutes on any VM
- Notify via Slack and email simultaneously
- Include runbook instructions in the alert notification
- Track the number of high-CPU incidents over time

**Question:** How should they configure this monitoring setup?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create a Cloud Monitoring alert policy using the metric `compute.googleapis.com/instance/cpu/utilization` with a threshold of 0.85, duration of 300 seconds (5 minutes), and two notification channels (one Slack webhook, one email). Add documentation to the alert policy with runbook instructions. Optionally create a log-based metric to track incident frequency over time.

**Why:** Cloud Monitoring alert policies are the standard way to detect metric threshold violations. The `duration` field ensures the alert only fires for sustained high CPU (avoiding noise from brief spikes). Multiple notification channels can be attached to a single alert policy. The documentation field in the alert policy allows embedding runbook instructions directly in the notification. A log-based metric on the alert notifications would allow tracking incident trends. Setting up alerts in Cloud Monitoring is preferred over writing custom monitoring scripts.

**Key services/concepts:** Cloud Monitoring alert policies, notification channels, metric thresholds, duration, documentation/runbook, log-based metrics

</details>

---

## Scenario 21: Automated Database Backup Strategy

**Situation:** A company runs a production Cloud SQL PostgreSQL instance that stores critical customer data. The compliance team requires: daily automated backups with 30-day retention, the ability to restore to any point in the last 7 days, and a tested disaster recovery procedure. The operations team also wants to receive alerts if backup storage exceeds a certain size.

**Requirements:**
- Daily automated backups retained for 30 days
- Point-in-time recovery within a 7-day window
- Automated backup schedule without manual intervention
- Alert if backup storage grows unexpectedly

**Question:** How should the backup strategy be configured?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Configure the Cloud SQL instance with automated backups (`--backup-start-time=02:00`), set retained backups count to 30 (`--retained-backups-count=30`), enable WAL archiving for point-in-time recovery (PostgreSQL equivalent of binary logging), and set the PITR window. For disaster recovery testing, periodically clone the instance using `gcloud sql instances clone` with `--point-in-time` to verify recovery works. Set up a Cloud Monitoring alert on the `cloudsql.googleapis.com/database/disk/utilization` metric to detect unexpected backup storage growth.

**Why:** Cloud SQL's built-in automated backup feature handles daily backups natively. Point-in-time recovery requires WAL archiving (for PostgreSQL) or binary logging (for MySQL) to be enabled. Cloning to a specific point in time is the correct way to test recovery -- restoring to the production instance would cause downtime. The disk utilization metric alert provides proactive warning about storage issues. Using Firestore-style manual exports (Cloud Scheduler + Cloud Function) would be incorrect for Cloud SQL because it has native backup support.

**Key services/concepts:** Cloud SQL automated backups, point-in-time recovery, WAL archiving, retained backups count, instance cloning, Cloud Monitoring alerts

</details>

---

## Scenario 22: Centralized Log Management With Compliance Requirements

**Situation:** A financial company has 10 GCP projects. The security team requires that all error-level logs from all projects are exported to BigQuery for analysis, all logs are retained for at least 2 years, and debug-level logs from load balancers are excluded from storage to reduce costs. They also need to identify who deleted any resource across the organization.

**Requirements:**
- Export ERROR+ logs from all projects to BigQuery
- 2-year retention for all other logs
- Exclude load balancer debug logs to save costs
- Track all resource deletions across the organization

**Question:** How should the logging infrastructure be configured?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create an aggregated log sink at the organization level with `--include-children` that routes `severity>=ERROR` logs to a BigQuery dataset. Create a custom log bucket with 730-day retention for long-term storage. Add an exclusion filter to the `_Default` sink to exclude load balancer debug logs (`resource.type="http_load_balancer" AND severity=DEBUG`). For tracking resource deletions, query the Admin Activity audit logs, which are always enabled and stored in the `_Required` bucket for 400 days -- but also route them to the 2-year custom bucket.

**Why:** An aggregated sink at the organization level with `--include-children` captures logs from all 10 projects without creating individual sinks. The sink's writer identity must be granted `roles/bigquery.dataEditor` on the destination dataset. Exclusion filters reduce costs by preventing unwanted log ingestion. Admin Activity audit logs are always on and free, making them the right source for tracking who deleted resources. The `_Required` bucket retains audit logs for only 400 days, so routing them to a custom 2-year bucket is necessary for compliance.

**Key services/concepts:** Aggregated log sinks, organization-level logging, exclusion filters, custom log buckets, retention policies, Admin Activity audit logs, BigQuery export

</details>

---

## Scenario 23: Cost Optimization for Development Environments

**Situation:** A company has 5 development teams, each with their own GCP project. Monthly cloud spending across dev projects has grown to $15,000. The CFO wants to reduce this. Upon investigation, you find: VMs are running 24/7 even though developers work only during business hours (8 AM - 6 PM weekdays), several VMs are oversized (using n2-standard-8 but only consuming 15% CPU), and many Cloud Storage buckets contain old build artifacts that have not been accessed in over 6 months.

**Requirements:**
- Reduce VM costs for off-hours and weekends
- Right-size oversized VMs
- Automatically manage stale storage data
- Set up budget alerts to prevent future overspending

**Question:** What cost optimization measures should be implemented?

<details>
<summary>Click to reveal answer</summary>

**Answer:** (1) Schedule dev VMs to stop outside business hours using Cloud Scheduler + Cloud Functions or Instance Schedules, reducing runtime by approximately 65%. (2) Right-size VMs by reviewing IAM Recommender and Cloud Monitoring CPU/memory metrics, then switching to smaller machine types or custom machine types that match actual usage. (3) Create Cloud Storage lifecycle policies to transition objects older than 30 days to Nearline and delete build artifacts older than 180 days. (4) Create billing budgets per project with alert thresholds at 50%, 90%, and 100%, notifying billing admins and a Pub/Sub topic. (5) Consider using Spot VMs for dev workloads that can tolerate interruptions.

**Why:** The combination addresses all three cost drivers: compute time (scheduling), compute sizing (right-sizing), and storage (lifecycle policies). Billing budgets with Pub/Sub enable programmatic responses (like automatically stopping resources). Remember that budgets do NOT cap spending -- they only send alerts. Committed Use Discounts would be wrong for dev workloads because they require 1-3 year commitments and dev usage is variable. Sustained Use Discounts will apply automatically once VMs run more than 25% of the month.

**Key services/concepts:** Instance Schedules, right-sizing, custom machine types, lifecycle policies, billing budgets, Pub/Sub programmatic budget actions, Spot VMs

</details>

---

## Scenario 24: Budget Alert With Automatic Spending Control

**Situation:** A startup received $10,000 in Google Cloud credits. They want to be notified at 50%, 80%, and 100% of a $8,000 monthly budget. Critically, they want to automatically disable billing on their project if spending exceeds $9,000 to protect against runaway costs from a misconfigured service.

**Requirements:**
- Budget alerts at 50%, 80%, and 100% of $8,000
- Email notifications to the finance team
- Automatic billing disablement at $9,000
- Must work without manual intervention

**Question:** How should this be configured?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create a billing budget of $8,000 with alert thresholds at 50%, 80%, and 100% (both actual and forecasted). Configure email notifications to billing admins. Additionally, set a $9,000 threshold that publishes to a Pub/Sub topic. Create a Cloud Function triggered by the Pub/Sub topic that calls the Cloud Billing API to disable billing on the project (`projects.updateBillingInfo` with an empty billing account).

**Why:** Billing budgets only send notifications -- they do NOT cap or stop spending. This is a critical exam point. To actually stop spending, you need a programmatic solution: Budget alert publishes to Pub/Sub, Cloud Function receives the message, and calls the Billing API to unlink the project from the billing account. When billing is disabled, Compute Engine VMs are stopped and most services cease functioning. The Cloud Function needs `roles/billing.projectManager` to unlink billing. Using forecasted alerts at $9,000 provides an earlier warning before actual spend reaches the threshold.

**Key services/concepts:** Billing budgets (do NOT cap spending), Pub/Sub notifications, Cloud Functions, Cloud Billing API, programmatic billing control

</details>

---

## Scenario 25: Multi-Environment Infrastructure With Terraform

**Situation:** A company needs to deploy identical infrastructure (VPC, subnets, firewall rules, Cloud SQL instance, GKE cluster) across three environments: dev, staging, and production. Each environment should be in its own project. The team wants version-controlled infrastructure, code review for changes, and the ability to preview changes before applying them. Five engineers collaborate on the infrastructure code.

**Requirements:**
- Identical infrastructure definition for all three environments
- Environment-specific configuration (project IDs, machine sizes, node counts)
- Shared state accessible by 5 engineers with locking
- Preview changes before applying
- Version control and code review

**Question:** How should the Infrastructure as Code setup be organized?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Use Terraform with the Google provider. Store the infrastructure definition in a single set of `.tf` files and use separate `.tfvars` files for each environment (e.g., `dev.tfvars`, `staging.tfvars`, `prod.tfvars`). Store Terraform state in a GCS backend (`backend "gcs"`) with a separate state file per environment (using different prefixes or buckets). Enable versioning on the GCS state bucket for state recovery. Use `terraform plan` for change preview before applying. Store all code in Git for version control and code review.

**Why:** Terraform is the most widely used IaC tool for Google Cloud and supports multi-cloud. Using `.tfvars` files per environment avoids code duplication while allowing different configurations (smaller VMs in dev, larger in prod). The GCS backend enables shared state with built-in locking to prevent concurrent modifications. Versioning on the state bucket allows recovery from bad applies. `terraform plan` provides the required change preview. Config Connector would be an alternative but requires a GKE cluster and is less suitable for managing the GKE cluster itself. Cloud Foundation Toolkit modules could accelerate development by providing best-practice templates.

**Key services/concepts:** Terraform, GCS backend, state locking, `.tfvars` files, `terraform plan`, version control, Cloud Foundation Toolkit

</details>

---

## Scenario 26: Secure Containerized Application End-to-End

**Situation:** A healthcare startup is deploying a patient portal application on GKE. The cluster must have no public node IPs. Pods need to access a Cloud SQL PostgreSQL database over a private IP and read patient documents from a Cloud Storage bucket. The application images are stored in Artifact Registry. The security team requires that no service account keys are used anywhere and that the default service account must not be used.

**Requirements:**
- Private GKE cluster (no public node IPs)
- Access Cloud SQL via private IP (no public endpoint)
- Access Cloud Storage from pods without SA keys
- Pull images from Artifact Registry
- No default service account usage

**Question:** What architecture and IAM configuration should be used?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Create a private GKE cluster with `--enable-private-nodes` and `--enable-ip-alias`. Set up Cloud NAT for outbound internet access. Configure Cloud SQL with a private IP on the same VPC. Use Workload Identity to map the Kubernetes service account to a dedicated Google Cloud service account. Grant the GCP service account `roles/cloudsql.client` (for Cloud SQL access), `roles/storage.objectViewer` (for Cloud Storage), and `roles/artifactregistry.reader` (for pulling images). Annotate the K8s service account with `iam.gke.io/gcp-service-account`. For Cloud SQL connectivity, use the Cloud SQL Auth Proxy as a sidecar container in the pod.

**Why:** This architecture follows all security best practices: private nodes eliminate public exposure, Workload Identity eliminates service account keys, dedicated service accounts follow least privilege, and Cloud SQL private IP keeps database traffic within the VPC. The Cloud SQL Auth Proxy sidecar handles authentication and encryption to the database. Cloud NAT is needed because private nodes require it for outbound internet access (pulling images from Artifact Registry if not using Private Google Access). The `_Default` compute service account should not be used because it has `roles/editor` by default.

**Key services/concepts:** Private GKE cluster, Workload Identity, Cloud SQL Auth Proxy, Cloud NAT, Private Google Access, dedicated service accounts, sidecar container

</details>

---

## Scenario 27: Migrating and Modernizing a Monolithic Application

**Situation:** An insurance company runs a monolithic Java application on-premises that handles policy management, claims processing, and customer notifications. They want to migrate to GCP in phases. Phase 1 (immediate): lift-and-shift the monolith to GCP to decommission the data center. Phase 2 (6 months later): extract the notification service into a serverless component triggered by Pub/Sub. They need to plan the architecture for both phases now.

**Requirements:**
- Phase 1: Run the existing Java monolith as-is on GCP
- Phase 1: Minimal changes to the application
- Phase 2: Notification service extracted as an event-driven component
- Phase 2: Notification service should scale independently and cost nothing when idle
- Monitoring and logging from day one

**Question:** What GCP services should be used for each phase?

<details>
<summary>Click to reveal answer</summary>

**Answer:**
- **Phase 1:** Deploy the monolith on Compute Engine with an instance template and MIG for high availability. Use Cloud SQL for the database (matching the on-premises database engine). Set up Cloud Monitoring with the Ops Agent for metrics and logs. Use an External Application Load Balancer for traffic distribution.
- **Phase 2:** Create a Pub/Sub topic for notification events. Modify the monolith to publish events to Pub/Sub instead of directly sending notifications. Deploy the notification service as a Cloud Run service (or Cloud Functions Gen 2) subscribed to the Pub/Sub topic. The notification service scales to zero when there are no events.

**Why:** Phase 1 uses Compute Engine because the monolith cannot be containerized immediately -- lift-and-shift requires minimal application changes. The MIG provides autohealing and load balancing. Phase 2 uses Pub/Sub for decoupling (the monolith does not need to know about the notification service) and Cloud Run for serverless scaling. Cloud Functions Gen 2 would also work since notifications are likely single-purpose functions. GKE would be overkill for a single monolith in Phase 1. Installing the Ops Agent from day one ensures visibility into the application's behavior on GCP.

**Key services/concepts:** Lift-and-shift, Compute Engine MIG, Cloud SQL migration, Pub/Sub decoupling, Cloud Run/Cloud Functions, Ops Agent, phased migration

</details>

---

## Scenario 28: Organization-Wide Security and Governance

**Situation:** A large enterprise is setting up Google Cloud for the first time. They have 200 developers across 15 teams, with dev, staging, and production environments. The security team has the following mandates: no VM should have an external IP in production, all data must stay in the EU, billing must be tracked per team, and all API actions must be auditable. They also need to prevent accidental project deletion by developers.

**Requirements:**
- Organization-wide restriction: no external IPs on production VMs
- All resources restricted to EU regions
- Billing tracked per team
- Full audit trail of all API actions
- Prevent developers from deleting projects

**Question:** How should the resource hierarchy, organization policies, billing, and auditing be configured?

<details>
<summary>Click to reveal answer</summary>

**Answer:**
- **Resource hierarchy:** Organization > Folders per team > Sub-folders for dev/staging/prod per team > Projects.
- **Organization policies:** Apply `constraints/compute.vmExternalIpAccess` (deny all) at the Production folder level. Apply `constraints/gcp.resourceLocations` (allow only EU regions) at the organization level.
- **Billing:** Create separate billing accounts per department (or use labels). Apply labels (`team=frontend`, `team=backend`, etc.) to all resources. Enable BigQuery billing export for detailed cost analysis with label-based grouping.
- **Auditing:** Admin Activity audit logs are enabled by default (always on, free). Enable Data Access audit logs for sensitive services (Cloud Storage, BigQuery). Route audit logs to a central BigQuery dataset using an aggregated organization-level log sink.
- **Prevent project deletion:** Create an IAM deny policy at the organization level that denies `cloudresourcemanager.googleapis.com/projects.delete` for the developers group.

**Why:** Organization policies provide centralized, top-down constraints that cannot be overridden at lower levels (unlike IAM, which is additive). The folder structure mirrors the organizational hierarchy and allows applying different policies at different levels (e.g., external IPs blocked only in production, not dev). Labels enable cost tracking without complex billing structures. IAM deny policies are the correct mechanism to prevent specific actions even when users have broad roles -- regular IAM is additive only. Admin Activity audit logs are always on and cannot be disabled, ensuring a complete audit trail.

**Key services/concepts:** Resource hierarchy (Org > Folders > Projects), organization policies, IAM deny policies, labels for cost allocation, BigQuery billing export, aggregated log sinks, Admin Activity audit logs, Data Access audit logs

</details>
