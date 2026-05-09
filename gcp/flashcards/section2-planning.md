# Section 2 Flashcards -- Planning and Configuring a Cloud Solution

Covers: Compute Resources, Data Storage Options, Network Resources

---
### Q1: What are the four main Google Cloud compute options from most control to most managed?
**A:** Compute Engine (IaaS) -> GKE (CaaS) -> Cloud Run (serverless containers) -> Cloud Functions (FaaS). Moving right means less operational overhead but less control.

---
### Q2: What is a Spot VM and how much can you save?
**A:** A Spot VM is a heavily discounted VM (60-91% off) that Google can reclaim at any time with a 30-second warning. Unlike legacy Preemptible VMs, Spot VMs have no maximum lifetime, but they are not covered by an SLA.

---
### Q3: When should you NOT use Spot VMs?
**A:** Never use Spot VMs for production web servers, databases requiring HA, applications that cannot tolerate interruptions, or workloads with strict SLA requirements. They are best for fault-tolerant batch jobs, CI/CD, and dev/test.

---
### Q4: What gcloud command creates a Spot VM?
**A:** `gcloud compute instances create my-spot-vm --provisioning-model=SPOT --instance-termination-action=STOP --zone=us-central1-a --machine-type=e2-standard-4`

---
### Q5: What does the machine type name `e2-standard-4` mean?
**A:** E2 family, standard memory ratio (4 GB per vCPU), 4 vCPUs. So it has 4 vCPUs and 16 GB RAM. Other ratios: `highmem` = 8 GB/vCPU, `highcpu` = 1 GB/vCPU.

---
### Q6: Your app needs exactly 6 vCPUs and 20 GB RAM but no predefined type matches. What should you do?
**A:** Use a custom machine type: `gcloud compute instances create my-vm --custom-cpu=6 --custom-memory=20GB --zone=us-central1-a`. Custom types are available for N1, N2, N2D, and E2 families.

---
### Q7: What is the difference between GKE Standard and GKE Autopilot?
**A:** In Standard mode, you manage nodes and pay per VM. In Autopilot, Google manages nodes, security hardening, and scaling; you pay per pod (CPU, memory, storage). Autopilot is recommended for most workloads to reduce ops overhead.

---
### Q8: What is the difference between Cloud Run and Cloud Functions?
**A:** Cloud Run deploys container images (any language/framework) with up to 60 min timeout and multiple concurrent requests per instance. Cloud Functions deploys source code functions in supported runtimes. Gen 2 Cloud Functions are actually built on Cloud Run.

---
### Q9: A startup has unpredictable traffic that sometimes drops to zero. Which compute option minimizes cost?
**A:** Cloud Run. It scales to zero when idle (zero cost), automatically scales up for traffic spikes, and charges per request plus CPU/memory time. No infrastructure to manage.

---
### Q10: What is a Sole-Tenant Node and when would you use one?
**A:** A physical server dedicated exclusively to your VMs. Use it for compliance requirements, bring-your-own-license (BYOL) scenarios, or when you need hardware-level performance isolation.

---
### Q11: What is the difference between Cloud SQL and Cloud Spanner?
**A:** Cloud SQL is a managed relational DB (MySQL, PostgreSQL, SQL Server) that scales vertically up to 64 TB. Cloud Spanner is a globally distributed relational DB with horizontal scaling, unlimited capacity, and a 99.999% SLA -- but at a higher cost.

---
### Q12: A global financial company needs a relational database with strong consistency across continents and 99.999% availability. Which service?
**A:** Cloud Spanner. It is the only GCP database offering global strong consistency with a 99.999% SLA and horizontal scaling.

---
### Q13: What is the difference between BigQuery and Cloud SQL?
**A:** BigQuery is a serverless data warehouse for analytical (OLAP) workloads -- it processes petabytes using SQL. Cloud SQL is for transactional (OLTP) workloads with traditional relational databases. Never use BigQuery for OLTP or Cloud SQL for large-scale analytics.

---
### Q14: What are the two modes of Firestore and when do you use each?
**A:** Native mode supports real-time listeners and offline sync for mobile/web apps. Datastore mode is optimized for server-side applications with entity-based storage. You choose the mode at database creation and cannot change it.

---
### Q15: An IoT platform generates millions of time-series data points per second with low-latency reads. Which database?
**A:** Cloud Bigtable. It is designed for massive time-series, IoT, and analytical workloads with millisecond latency at any scale. It supports billions of rows and scales linearly by adding nodes.

---
### Q16: True or False: Cloud Bigtable is cost-effective for datasets under 1 TB.
**A:** False. Bigtable requires a minimum of 1 node and is not cost-effective for small datasets. For data under 1 TB, consider Firestore or Cloud SQL instead.

---
### Q17: What are the four Cloud Storage classes and their minimum storage durations?
**A:** Standard (none), Nearline (30 days), Coldline (90 days), Archive (365 days). Storage cost decreases from Standard to Archive, but retrieval cost increases.

---
### Q18: True or False: Archive class in Cloud Storage has higher access latency than Standard, similar to AWS Glacier.
**A:** False. All Cloud Storage classes have the same millisecond access latency. The difference is only in storage cost, retrieval cost, and minimum storage duration.

---
### Q19: What happens if you delete a Coldline object after 30 days (before the 90-day minimum)?
**A:** You are charged an early deletion fee for the remaining 60 days of storage. The minimum storage duration charge applies regardless of when the object is actually deleted.

---
### Q20: You need to store backup data accessed about once per quarter. Which storage class minimizes cost?
**A:** Coldline. It is designed for data accessed less than once per quarter (90-day minimum storage), with lower storage costs than Standard or Nearline.

---
### Q21: What is the difference between a Zonal and Regional Persistent Disk?
**A:** Zonal PD exists in a single zone. Regional PD synchronously replicates across two zones in the same region, providing HA with faster failover -- at roughly 2x the cost.

---
### Q22: What are Local SSDs and what is their key limitation?
**A:** Local SSDs are physically attached to the host machine, offering extremely high IOPS (up to 2.4M). However, they are ephemeral -- data is lost when the VM stops, is preempted, or is deleted. Use for caches and scratch data only.

---
### Q23: What is AlloyDB and when should you choose it over Cloud SQL?
**A:** AlloyDB is a fully managed, PostgreSQL-compatible database that is up to 4x faster for transactions and 100x faster for analytics than standard PostgreSQL. Choose it for high-performance PostgreSQL workloads or mixed OLTP/OLAP (HTAP) scenarios.

---
### Q24: What is a VPC in Google Cloud and what is its scope?
**A:** A VPC (Virtual Private Cloud) is a software-defined network. VPCs are global resources that span all GCP regions. Subnets within a VPC are regional and span all zones within that region.

---
### Q25: What is the difference between auto mode and custom mode VPCs?
**A:** Auto mode VPCs automatically create one subnet per region with predefined IP ranges. Custom mode VPCs require you to manually create subnets. Custom mode is recommended for production because it gives full control over IP ranges.

---
### Q26: A company needs to serve a global web application with SSL termination, URL-based routing, and DDoS protection. Which load balancer?
**A:** External Application Load Balancer (HTTP/S). It is global, Layer 7, supports URL maps for path-based routing, integrates with Cloud Armor (DDoS/WAF) and Cloud CDN, and handles SSL termination.

---
### Q27: What load balancer should you use for internal microservices communicating over HTTP?
**A:** Internal Application Load Balancer. It is regional, Layer 7, and designed for internal HTTP/HTTPS traffic within a VPC.

---
### Q28: A gaming application uses UDP and needs an external-facing load balancer. Which one?
**A:** External Passthrough Network Load Balancer. It is regional, Layer 4, supports UDP traffic, and is non-proxied (preserves source IP).

---
### Q29: What is the difference between Premium Tier and Standard Tier networking?
**A:** Premium Tier routes traffic through Google's private global backbone from the nearest edge PoP (lower latency, supports global LB). Standard Tier routes through the public internet, entering Google's network near the destination region (higher latency, lower cost, regional LB only).

---
### Q30: True or False: Standard Tier networking supports global load balancing.
**A:** False. Standard Tier only supports regional load balancing. Global load balancing requires Premium Tier, which is the default.

---
### Q31: What is Cloud NAT used for?
**A:** Cloud NAT allows VMs without external IP addresses to access the internet for outbound connections (e.g., downloading updates, calling external APIs). It provides outbound-only connectivity -- external traffic cannot initiate inbound connections.

---
### Q32: What is Cloud CDN and how is it enabled?
**A:** Cloud CDN caches content at Google's edge locations worldwide to reduce latency. It is integrated with the External Application Load Balancer -- you enable it on a backend service of that load balancer.

---
### Q33: What five components make up an External Application Load Balancer configuration?
**A:** (1) Frontend (IP, port, SSL cert), (2) URL Map (host/path routing rules), (3) Backend Service (config for backends, health checks, session affinity), (4) Backends (instance groups, NEGs, or Cloud Storage buckets), (5) Health Checks.

---
### Q34: What are the four key factors for choosing a Google Cloud region?
**A:** (1) Latency -- proximity to users. (2) Compliance -- data residency laws. (3) Service availability -- not all services in all regions. (4) Pricing -- costs vary by region.

---
### Q35: What is the scope of VMs, subnets, and VPCs respectively?
**A:** VMs are zonal resources. Subnets are regional resources (span all zones in a region). VPCs are global resources (span all regions).

---
### Q36: What connectivity option provides the lowest latency and highest bandwidth between on-premises and GCP?
**A:** Dedicated Interconnect. It provides 10-200 Gbps over a direct physical connection to Google's network, with 99.9-99.99% SLA depending on configuration.

---
### Q37: What is the difference between Dedicated Interconnect and HA VPN?
**A:** Dedicated Interconnect provides a direct physical connection (10-200 Gbps, lowest latency). HA VPN sends encrypted traffic over the public internet (up to 3 Gbps per tunnel, variable latency) but is cheaper and easier to set up.

---
### Q38: A web application uses PostgreSQL and needs both fast transactions and complex analytical reports. Which service?
**A:** AlloyDB. It is PostgreSQL-compatible with a columnar engine for analytics, making it ideal for HTAP (hybrid transactional/analytical processing) workloads.

---
### Q39: Which compute option is best for a team running 12 microservices that need auto-healing, rolling updates, and service discovery?
**A:** GKE Autopilot. It provides full Kubernetes orchestration (auto-healing, rolling updates, service discovery) while Google manages the nodes, reducing operational burden.

---
### Q40: True or False: Cloud Functions Gen 2 is built on Cloud Run.
**A:** True. Gen 2 Cloud Functions run on Cloud Run infrastructure, which gives them features like up to 60-minute timeout, up to 1,000 concurrent requests per instance, and traffic splitting.

---
### Q41: What is Google Cloud Hyperdisk and how does it differ from Persistent Disk?
**A:** Hyperdisk is next-gen block storage that decouples IOPS and throughput from capacity — you set them independently. Persistent Disk ties IOPS/throughput to disk size (e.g., pd-ssd = 30 IOPS/GB). Hyperdisk supports much higher max performance.

---
### Q42: When would you choose Hyperdisk Throughput over Hyperdisk Balanced?
**A:** Hyperdisk Throughput is optimized for sequential, high-throughput workloads like Kafka, Hadoop, and data pipelines. Hyperdisk Balanced is for general mixed workloads. Throughput has lower IOPS but very high MB/s at lower cost.

---
### Q43: What is Hyperdisk ML and what makes it unique?
**A:** Hyperdisk ML is a multi-reader block disk that can be mounted by many VMs simultaneously in read-only mode. It is designed for ML model serving — you load a model once and serve it across many VMs without copying.

---
### Q44: Your team is migrating an existing Apache Kafka workload to GCP. Which messaging service should you use?
**A:** Google Cloud Managed Service for Apache Kafka. It is API-compatible with Apache Kafka, so no code changes are needed. Pub/Sub would require rewriting the client code since it uses a proprietary API.

---
### Q45: What is Google Cloud NetApp Volumes and when would you choose it over Filestore?
**A:** NetApp Volumes is enterprise NFS/SMB storage built on NetApp ONTAP, offering advanced features like SnapMirror, storage tiering, ONTAP snapshots, and multi-protocol access. Choose it for enterprise NAS workloads (SAP, EPIC, Windows file servers). Filestore is simpler, with less overhead.

---
### Q46: Your company runs a global e-commerce app and needs the database to be available in US, EU, and Asia simultaneously with strong consistency. Which service?
**A:** Cloud Spanner with a multi-region configuration (e.g., `nam-eur-asia1`). It is the only GCP service offering global relational SQL with external strong consistency at 99.999% SLA. Multi-region config is set at instance creation.
