# Google Associate Cloud Engineer — Exam Overview

## About the Certification

The **Google Associate Cloud Engineer** certification demonstrates your ability to deploy and secure applications and infrastructure, monitor operations of multiple projects, and maintain enterprise solutions to ensure they meet target performance metrics. This is a foundational-level certification ideal for cloud professionals who want to validate their GCP skills.

---

## Exam Details

| Item | Detail |
|------|--------|
| **Exam length** | 2 hours (120 minutes) |
| **Question count** | 50–60 questions |
| **Question format** | Multiple choice and multiple select |
| **Passing score** | ~70 % (Google does not publish exact cut-off) |
| **Registration fee** | $200 USD |
| **Languages** | English, Japanese, Spanish, Portuguese, French, German, Indonesian, Korean, Chinese |
| **Recommended experience** | 6+ months hands-on with GCP |
| **Certification validity** | 2 years (recertification required) |
| **Delivery** | Online proctored or test center (Kryterion) |

---

## Exam Sections and Weights

| Section | Weight |
|---------|--------|
| 1. Setting up a cloud solution environment | ~23 % |
| 2. Planning and implementing a cloud solution | ~30 % |
| 3. Ensuring successful operation of a cloud solution | ~27 % |
| 4. Configuring access and security | ~20 % |

---

## Section 1 — Setting Up a Cloud Solution Environment (~23 %)

### 1.1 Setting up cloud projects and accounts
- Creating a resource hierarchy
- Applying organizational policies to the resource hierarchy
- Granting members IAM roles within a project
- Managing users and groups in Cloud Identity (manually and automated)
- Enabling APIs within projects
- Provisioning and setting up products in Google Cloud Observability
- Assessing quotas and requesting increases
- Setting up standalone organizations
- Setting up cloud networking
- Confirming availability of products in geographical locations (regional, global)
- Configuring Cloud Asset Inventory and using Gemini Cloud Assist to analyze resources

### 1.2 Managing billing configuration
- Creating one or more billing accounts
- Linking projects to a billing account
- Establishing billing budgets and alerts
- Setting up billing exports

---

## Section 2 — Planning and Implementing a Cloud Solution (~30 %)

### 2.1 Planning and implementing compute resources
- Selecting appropriate compute choices for a given workload (Compute Engine, GKE, Cloud Run, Cloud Run functions, Knative serving)
- Launching a compute instance (availability policy, SSH keys)
- Choosing appropriate storage for Compute Engine (zonal Persistent Disk, regional Persistent Disk, Google Cloud Hyperdisk)
- Creating an autoscaled managed instance group by using an instance template
- Configuring OS Login
- Configuring VM Manager
- Using Spot VM instances and custom machine types
- Installing and configuring kubectl for Kubernetes
- Deploying a GKE cluster with different configurations (GKE Autopilot, regional clusters, private clusters)
- Deploying a containerized application to GKE
- Deploying an application to serverless compute platforms including for the processing of Google Cloud events (Pub/Sub events, Cloud Storage object change notification events, Eventarc)

### 2.2 Planning and implementing storage and data solutions
- Choosing and deploying data products (Cloud SQL, BigQuery, Firestore, Spanner, Bigtable, AlloyDB, Dataflow, Pub/Sub, Google Cloud Managed Service for Apache Kafka, Memorystore)
- Choosing and deploying storage products (Cloud Storage, Filestore, Google Cloud NetApp Volumes) and Cloud Storage options (Standard, Nearline, Coldline, Archive)
- Loading data (command-line upload, load data from Cloud Storage, Storage Transfer Service)
- Maintaining multi-region redundancy across data solutions

### 2.3 Planning and implementing networking resources
- Creating a VPC with subnets (custom mode VPC, Shared VPC)
- Creating and applying Cloud Next Generation Firewall (Cloud NGFW) policies with ingress and egress rules (action, source, destination, targets, protocols, ports)
- Using Tags (secure Tags) and service accounts in Cloud NGFW policy rules
- Establishing network connectivity (Cloud VPN, VPC Network Peering, Cloud Interconnect)
- Choosing and deploying load balancers
- Differentiating Network Service Tiers

### 2.4 Planning and implementing resources through infrastructure as code
- Infrastructure as code tooling (Fabric FAST, Config Connector, Terraform, Helm)
- Planning and executing infrastructure as code deployments, including versioning, state management, and updates

---

## Section 3 — Ensuring Successful Operation of a Cloud Solution (~27 %)

### 3.1 Managing compute resources
- Remotely connecting to a Compute Engine instance
- Viewing current running Compute Engine instances
- Working with snapshots and images (create, view, delete; schedule a snapshot)
- Viewing current running GKE cluster inventory (nodes, Pods, Services)
- Configuring GKE to access Artifact Registry
- Working with GKE node pools (add, edit, remove; autoscaling node pool)
- Working with Kubernetes resources (Pods, Services, StatefulSets)
- Managing horizontal and vertical Pod autoscaling configurations
- Managing GKE Autopilot Pod resource requests
- Deploying new versions of a Cloud Run application
- Adjusting application traffic splitting parameters (Cloud Run, Cloud Run functions, GKE)
- Configuring autoscaling for a Cloud Run application

### 3.2 Managing storage and data solutions
- Managing and securing objects in Cloud Storage buckets
- Setting object lifecycle management policies for Cloud Storage buckets
- Executing queries to retrieve data from data instances (Cloud SQL, BigQuery, Bigtable, Spanner, Firestore, AlloyDB)
- Estimating costs of data storage resources
- Backing up and restoring database instances (Cloud SQL, Firestore, Spanner, AlloyDB, Bigtable)
- Reviewing job status (Dataflow, BigQuery)
- Using Database Center to manage the Google Cloud database fleet

### 3.3 Managing networking resources
- Adding a subnet to an existing VPC
- Expanding a subnet to have more IP addresses
- Reserving static external or internal IP addresses
- Adding custom static routes in a VPC
- Working with Cloud DNS and Cloud NAT

### 3.4 Monitoring and logging
- Creating Cloud Monitoring alerts based on resource metrics
- Creating and ingesting Cloud Monitoring custom metrics (from applications or logs)
- Exporting logs to external systems (on-premises, BigQuery)
- Configuring log buckets, log analytics, and log routers
- Viewing and filtering logs in Cloud Logging
- Viewing specific log message details in Cloud Logging
- Using cloud diagnostics to research an application issue (Cloud Trace, Cloud Profiler, Query Insights, index advisor)
- Viewing the Personalized Service Health dashboard
- Configuring and deploying Ops Agent
- Deploying Google Cloud Managed Service for Prometheus
- Configuring audit logs
- Using Gemini Cloud Assist for Cloud Monitoring
- Using Active Assist to optimize resource utilization

---

## Section 4 — Configuring Access and Security (~20 %)

### 4.1 Managing Identity and Access Management (IAM)
- Viewing and creating IAM policies
- Managing the various role types and defining custom IAM roles (basic, predefined, custom)

### 4.2 Managing service accounts
- Creating service accounts
- Using service accounts in IAM policies with minimum permissions
- Assigning service accounts to resources
- Managing IAM permissions of a service account
- Managing service account impersonation
- Creating and managing short-lived service account credentials
- Using a Google Cloud service account with a GKE application

---

## Study Strategy

### Recommended Resources
1. **Google Cloud Skills Boost** — Free labs and quests (cloud.google.com/training)
2. **Official Documentation** — cloud.google.com/docs
3. **Coursera: Google Cloud Fundamentals** — Core Infrastructure course
4. **A Cloud Guru / Pluralsight** — Dedicated ACE courses
5. **Qwiklabs** — Hands-on labs for each service
6. **Practice Exams** — Google offers a free practice exam

### Study Tips
- **Hands-on practice is critical** — The exam is practical, not theoretical
- **Know the gcloud CLI** — Many questions test command-line knowledge
- **Understand pricing** — Know when to use Spot VMs, committed use discounts, storage classes
- **Focus on Section 2** — It has the highest weight (30 %) and covers planning + deploying
- **Learn IAM deeply** — Roles, service accounts, Workload Identity, and policies appear throughout
- **Know when to use each service** — Compute Engine vs GKE vs Cloud Run vs Cloud Run functions vs Knative
- **Understand Cloud NGFW** — The exam now tests Next Generation Firewall policies, secure Tags, not just basic firewall rules
- **Know new services** — Gemini Cloud Assist, Cloud Asset Inventory, Database Center, Active Assist, Hyperdisk, Fabric FAST
- **Understand networking** — VPCs, subnets, Cloud NGFW, load balancing, Network Service Tiers, Cloud NAT
- **Time management** — ~2 minutes per question; flag and revisit difficult ones

### Key gcloud Commands to Memorize
```bash
gcloud config set project PROJECT_ID
gcloud compute instances list
gcloud compute instances create INSTANCE_NAME --zone=ZONE
gcloud container clusters create CLUSTER_NAME
gcloud run deploy SERVICE_NAME --image IMAGE
gcloud functions deploy FUNCTION_NAME
gcloud iam roles create ROLE_ID
gcloud projects add-iam-policy-binding PROJECT_ID --member=MEMBER --role=ROLE
```

---

## File Index — Study Guide Contents

| File | Topic |
|------|-------|
| `00-exam-overview.md` | This file — exam overview and study guide |
| `01-setting-up-cloud-projects-and-accounts.md` | Section 1.1 — Resource hierarchy, IAM, APIs, quotas, Cloud Asset Inventory, Gemini Cloud Assist |
| `02-managing-billing-configuration.md` | Section 1.2 — Billing accounts, budgets, alerts, exports |
| `03-planning-compute-resources.md` | Section 2.1 — Compute Engine, GKE, Cloud Run, Cloud Run functions, Knative |
| `04-planning-data-storage-options.md` | Section 2.2 — Cloud SQL, BigQuery, Firestore, Spanner, Bigtable, AlloyDB, Kafka, Memorystore, Hyperdisk, NetApp Volumes, Storage classes |
| `05-planning-network-resources.md` | Section 2.3 — Load balancing, Network Service Tiers |
| `06-deploying-compute-engine.md` | Section 2.1 — Launching VMs, MIGs, OS Login, VM Manager, Hyperdisk |
| `07-deploying-gke-resources.md` | Section 2.1 — kubectl, GKE clusters, Autopilot, private clusters |
| `08-deploying-cloud-run-and-functions.md` | Section 2.1 — Cloud Run, Cloud Run functions, Eventarc, Knative |
| `09-deploying-data-solutions.md` | Section 2.2 — Data products, loading data, multi-region redundancy |
| `10-deploying-networking-resources.md` | Section 2.3 — VPCs, Cloud NGFW policies, secure Tags, VPN, peering |
| `11-infrastructure-as-code.md` | Section 2.4 — Terraform, Fabric FAST, Config Connector, Helm, state management |
| `12-managing-compute-engine.md` | Section 3.1 — VM management, snapshots, images |
| `13-managing-gke-resources.md` | Section 3.1 — Cluster management, node pools, Autopilot, autoscaling |
| `14-managing-cloud-run.md` | Section 3.1 — Versioning, traffic splitting, autoscaling |
| `15-managing-storage-and-databases.md` | Section 3.2 — Object management, lifecycle, queries, backups, Database Center |
| `16-managing-networking-resources.md` | Section 3.3 — Subnets, IPs, static routes, Cloud DNS, Cloud NAT |
| `17-monitoring-and-logging.md` | Section 3.4 — Monitoring, logging, alerts, Ops Agent, Gemini Cloud Assist, Active Assist |
| `18-managing-iam.md` | Section 4.1 — IAM policies, role types, custom roles |
| `19-managing-service-accounts.md` | Section 4.2 — Service accounts, Workload Identity, impersonation, credentials |

---

## Glossary

**ACE (Associate Cloud Engineer)** — Google Cloud certification that validates the ability to deploy, monitor, and manage cloud solutions on GCP.

**Active Assist** — Google Cloud's suite of AI-powered recommendations and insights that automatically analyze resources to identify cost savings, security improvements, and performance optimizations (e.g., Idle VM Recommender, IAM Recommender, Unattended Project Recommender).

**AlloyDB** — Fully managed, PostgreSQL-compatible database service offering high performance for both transactional and analytical workloads.

**AlloyDB** — Fully managed, PostgreSQL-compatible database service offering high performance for both transactional and analytical workloads.

**Anthos** — Google's hybrid and multi-cloud application platform, enabling GKE workloads to run on-premises or across multiple clouds.

**API (Application Programming Interface)** — A programmatic interface that exposes a service's functionality; GCP services are consumed via APIs that must be explicitly enabled per project.

**App Engine** — Google Cloud's fully managed platform-as-a-service (PaaS) for deploying web applications without managing infrastructure.

**Artifact Registry** — Google Cloud's fully managed service for storing and managing container images, language packages, and other build artifacts.

**Audit Logs** — Records of administrative activity and data access events within GCP, used for security and compliance monitoring via Cloud Logging.

**Autoscaler** — GCP component that automatically adjusts the number of VM instances in a managed instance group based on defined metrics and policies.

**Autoscaling** — Automatic adjustment of compute capacity (VMs, pods, Cloud Run instances) in response to load; can be horizontal (more instances) or vertical (bigger instances).

**Autopilot** — A GKE mode where Google fully manages node provisioning and scaling; billing is per pod rather than per node.

**Basic Role** — One of the original IAM roles (Owner, Editor, Viewer, Browser) that grant broad permissions across all services in a project; also called primitive roles.

**Bigtable (Cloud Bigtable)** — Fully managed, wide-column NoSQL database designed for massive analytical and operational workloads with millisecond latency.

**BigQuery** — Serverless, highly scalable data warehouse for analytical (OLAP) workloads, capable of processing petabytes of data using SQL.

**BigQuery ML** — Feature within BigQuery that allows creation and execution of machine learning models using SQL.

**Cloud DNS** — Google Cloud's scalable, reliable, and managed authoritative Domain Name System (DNS) service.

**Cloud Asset Inventory** — A GCP service that provides inventory discovery and monitoring for all resources in a project, folder, or organization; supports real-time change feeds, export, and policy analysis.

**Cloud Foundation Toolkit (CFT)** — A set of reference templates and best practices for deploying GCP infrastructure using Terraform and other IaC tools.

**Cloud NGFW (Cloud Next Generation Firewall)** — GCP's advanced, stateful firewall service that uses hierarchical firewall policies with secure Tags and service accounts for granular ingress and egress control; supersedes the older per-VPC firewall rules model.

**Cloud Functions** — Serverless, event-driven Function as a Service (FaaS) platform where code runs in response to triggers without managing infrastructure.

**Cloud Identity** — Google's Identity as a Service (IDaaS) solution for managing users, groups, devices, and SSO across Google Cloud.

**Cloud Logging** — Centralized log management service that ingests, stores, and routes logs from GCP services and applications.

**Cloud Console (Google Cloud Console)** — Web-based graphical user interface at console.cloud.google.com for managing GCP resources, projects, IAM, billing, and services.

**Cloud Monitoring** — GCP service for collecting metrics, creating dashboards, setting alerts, and performing uptime checks on resources.

**Cloud NAT** — Managed network address translation service that allows VMs without external IPs to access the internet.

**Cloud Profiler** — Continuous CPU and memory profiling service for production applications running on GCP.

**Cloud Run** — Fully managed serverless platform for running containers, scaling automatically including to zero.

**Cloud Run for Anthos** — Cloud Run deployed on GKE clusters, enabling serverless container execution in hybrid environments.

**Cloud Scheduler** — Fully managed cron job service for scheduling tasks such as invoking Cloud Functions or HTTP endpoints.

**Cloud SQL** — Fully managed relational database service supporting MySQL, PostgreSQL, and SQL Server.

**Cloud Storage** — Google Cloud's object storage service for storing and retrieving unstructured data in buckets.

**Cloud Trace** — Distributed tracing service for analyzing request latency across applications on GCP.

**Cloud VPN** — Managed service for securely connecting on-premises networks or other cloud networks to GCP VPCs over IPsec tunnels.

**CaaS (Container as a Service)** — Cloud service model where the provider manages the container orchestration infrastructure; GKE is GCP's CaaS offering.

**Committed Use Discounts (CUDs)** — Discounts of up to 57% (1-year) or 70% (3-year) on Compute Engine and Cloud SQL in exchange for a resource usage commitment.

**Compute Engine** — GCP's Infrastructure as a Service (IaaS) offering providing virtual machines running on Google's infrastructure.

**Config Connector** — Kubernetes add-on that allows managing GCP resources using Kubernetes manifests and the Kubernetes API.

**Custom Machine Type** — Compute Engine configuration that lets users specify an arbitrary number of vCPUs and amount of memory rather than using a predefined machine type.

**Custom Metric** — A user-defined metric published to Cloud Monitoring from an application, VM agent, or log-based extraction, used for alerting and dashboards.

**Custom Mode VPC** — A VPC where users manually create subnets and specify IP ranges, as opposed to auto mode where Google creates default subnets in every region.

**Custom Role** — User-defined IAM role containing a specific subset of permissions, used to enforce least-privilege access beyond what predefined roles offer.

**Database Center** — A Google Cloud unified control plane for managing, monitoring, and governing all GCP database instances (Cloud SQL, AlloyDB, Spanner, Bigtable, Firestore, Memorystore) from a single place.

**Dataflow** — Fully managed stream and batch data processing service based on the Apache Beam programming model.

**Eventarc** — GCP service that enables event-driven architectures by routing events from GCP services and custom sources to Cloud Run, Cloud Functions, and other targets.

**Fabric FAST** — Google Cloud's opinionated Terraform blueprint framework (Fast Automation for Setups and Tenants) for deploying production-ready GCP landing zones and infrastructure; successor approach to Cloud Foundation Toolkit for organization-scale deployments.

**External IP Address** — A publicly routable IPv4 or IPv6 address that allows a GCP resource (such as a VM) to be reached from the internet; can be ephemeral or static.

**FaaS (Function as a Service)** — Serverless compute model where individual functions are deployed and executed in response to events; Cloud Functions is GCP's FaaS offering.

**Firestore** — Serverless, NoSQL document database designed for mobile, web, and IoT applications with real-time sync and offline support.

**Firewall Rule** — A VPC-level rule that allows or denies network traffic to or from VMs based on protocol, port, source, destination, network tags, or service accounts.

**Folder** — Optional grouping node in the GCP resource hierarchy between the organization and projects; can be nested up to 10 levels deep and used to apply IAM and org policies.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services running on the same infrastructure that Google uses internally.

**Gemini Cloud Assist** — Google Cloud's AI-powered assistant integrated into the Cloud Console that provides natural-language answers, resource analysis, log summaries, configuration recommendations, and troubleshooting guidance for GCP environments.

**gcloud CLI** — The primary command-line interface for interacting with GCP services and resources.

**GKE (Google Kubernetes Engine)** — Google Cloud's managed Kubernetes service for deploying and operating containerized applications.

**GKE Enterprise** — Enhanced tier of GKE that provides additional features for multi-cluster and hybrid deployments, formerly part of Anthos.

**Google Workspace** — Google's suite of cloud-based productivity tools (Gmail, Drive, Docs, etc.) that includes Cloud Identity capabilities and is tied to the Organization node.

**GPU (Graphics Processing Unit)** — Specialized hardware accelerator used for machine learning training/inference, video transcoding, and HPC; attachable to Compute Engine VMs.

**Helm** — Package manager for Kubernetes that uses charts (pre-configured application definitions) to deploy and manage applications.

**Google Cloud Hyperdisk** — Google Cloud's next-generation block storage for Compute Engine that decouples IOPS and throughput provisioning from capacity, offering higher performance than Persistent Disks; types include Hyperdisk Balanced, Hyperdisk Extreme, and Hyperdisk Throughput.

**Horizontal Autoscaling** — Scaling by adding or removing instances (VMs, pods, Cloud Run replicas) in response to load metrics.

**HPC (High-Performance Computing)** — Workloads requiring significant computational resources, often using clusters of compute-optimized VMs or GPUs.

**HTTP(S) Load Balancer** — GCP's global Layer 7 load balancer that distributes HTTP and HTTPS traffic across backends.

**IaaS (Infrastructure as a Service)** — Cloud service model providing virtualized computing resources such as VMs, storage, and networking.

**IAM (Identity and Access Management)** — GCP's system for controlling who (members/principals) has what access (roles) to which resources.

**IAM Policy** — A document that binds one or more members to a role at a specific level of the resource hierarchy; stored and enforced by GCP.

**IDaaS (Identity as a Service)** — Cloud-delivered identity management; Cloud Identity is Google's IDaaS offering.

**Impersonation** — IAM feature allowing a principal to temporarily act as a service account to obtain its permissions, without using long-lived service account keys.

**Internal IP Address** — An RFC 1918 private IP address assigned to a VM within a VPC subnet; reachable only from inside the VPC or connected networks.

**Knative** — An open-source Kubernetes-based platform for building, deploying, and managing serverless workloads; Cloud Run is built on Knative and GKE supports Knative serving directly for in-cluster serverless.

**Kryterion** — Testing platform used to deliver Google Cloud certification exams at physical test centers.

**kubectl** — Command-line tool for interacting with Kubernetes clusters, used to deploy and manage applications on GKE.

**Label** — Key-value pair (e.g., `env:prod`) attached to GCP resources used for organizing, filtering, and grouping costs in billing reports.

**Log Analytics** — Cloud Logging feature that allows running SQL queries directly against log entries stored in log buckets.

**Log Bucket** — Cloud Logging storage container for log entries; each GCP project has default `_Required` and `_Default` log buckets.

**Log Router** — Cloud Logging component that evaluates sinks and routes log entries to destinations such as Cloud Storage, BigQuery, or Pub/Sub.

**Managed Apache Kafka (Google Cloud Managed Service for Apache Kafka)** — A fully managed Apache Kafka service on Google Cloud providing a compatible Kafka API without managing clusters; integrates with Dataflow and other GCP services.

**Managed Instance Group (MIG)** — A group of identical VM instances managed as a single entity, supporting autoscaling, auto-healing, and rolling updates.

**Managed Service for Prometheus** — Google Cloud's managed monitoring solution compatible with the Prometheus open-source monitoring system.

**Member (Principal)** — An identity that can be granted IAM roles, including Google accounts, service accounts, groups, or domains.

**MFA (Multi-Factor Authentication)** — Authentication mechanism requiring more than one verification factor; available in Cloud Identity for user accounts.

**Network Service Tiers** — GCP networking pricing and routing tiers: Premium (global Google backbone) and Standard (regional routing through public internet).

**Network Tag** — A string label applied to VMs used to target firewall rules and routes without referencing individual instance names.

**NetApp Volumes (Google Cloud NetApp Volumes)** — A fully managed, enterprise-grade network-attached storage (NFS/SMB) service on Google Cloud built on NetApp ONTAP, providing high performance and protocol compatibility.

**Node Pool** — A group of nodes within a GKE cluster that share the same configuration (machine type, OS image, disk type); clusters can have multiple node pools.

**Ops Agent** — Unified agent for Compute Engine VMs that collects logs and metrics and sends them to Cloud Logging and Cloud Monitoring.

**Operations Suite** — Formerly Stackdriver; Google Cloud's integrated suite of monitoring, logging, tracing, and diagnostics tools.

**Organization** — Top-level node in the GCP resource hierarchy tied to a Google Workspace or Cloud Identity domain; provides centralized control over all resources.

**OS Login** — Feature that allows managing SSH access to Compute Engine instances using Google account IAM roles instead of SSH keys.

**PagerDuty** — Third-party incident management platform that can receive Cloud Monitoring alert notifications.

**Permission** — The right to perform a specific action on a resource (e.g., `compute.instances.create`); permissions are grouped into roles and cannot be granted directly.

**Persistent Disk (PD)** — Durable block storage for Compute Engine VMs; available as Zonal or Regional, in Standard (HDD), Balanced, SSD, and Extreme variants.

**Pod** — The smallest deployable unit in Kubernetes, consisting of one or more containers that share network and storage resources.

**Predefined Role** — Google-managed, fine-grained IAM role scoped to a specific GCP service (e.g., `roles/compute.instanceAdmin.v1`).

**Preemptible VM** — Legacy low-cost VM type that Google can reclaim at any time with a 30-second warning and a maximum lifetime of 24 hours; superseded by Spot VMs.

**Project** — Core organizational unit in GCP; every resource belongs to exactly one project identified by a unique Project ID and Project Number.

**Pub/Sub (Cloud Pub/Sub)** — Fully managed asynchronous messaging service for decoupling applications using a publish-subscribe model.

**Quota** — A per-project limit on API usage (rate quota) or resource count (allocation quota) enforced by GCP; some can be increased via request.

**Qwiklabs** — Hands-on cloud learning platform providing lab environments for practicing GCP skills.

**Region** — A specific geographic location (e.g., `us-central1`) containing multiple zones where GCP resources can be deployed; regional resources span multiple zones in the same region.

**Resource** — An individual GCP service instance (VM, bucket, database, etc.) that belongs to exactly one project and inherits IAM policies from its parents.

**Role** — A named collection of IAM permissions granted to a principal on a resource; roles come in three types: Basic (Primitive), Predefined, and Custom.

**Service Account** — Special Google account representing an application or workload rather than a human user, used for authenticating service-to-service calls.

**Service Account Impersonation** — Mechanism that allows an authorized principal to obtain short-lived credentials for a service account without downloading its keys.

**Shared VPC** — GCP feature allowing a host project to share its VPC network with service projects, centralizing network management.

**Short-Lived Credentials** — Temporary access tokens or signed JWTs (typically valid for up to 1 hour) generated for service accounts to minimize the risk of leaked long-lived keys.

**SLA (Service Level Agreement)** — Google's commitment to a specific level of availability or performance for a service; breaches may qualify for service credits.

**Snapshot** — Point-in-time copy of a Persistent Disk used for backup and disaster recovery.

**Spanner (Cloud Spanner)** — Globally distributed, strongly consistent, horizontally scalable relational database with 99.999% SLA.

**Spot VM** — Low-cost Compute Engine VM type offering up to 91% discount; Google can reclaim it at any time with 30-second notice and no maximum runtime limit.

**SSH Key** — Public/private key pair used to authenticate to a Compute Engine VM via the SSH protocol; superseded by OS Login for IAM-managed access.

**SSO (Single Sign-On)** — Authentication scheme allowing users to log in once and access multiple systems; configurable through Cloud Identity.

**Stackdriver** — Former name of the Google Cloud Operations Suite (now Cloud Monitoring, Cloud Logging, Cloud Trace, etc.).

**Statefulset** — Kubernetes workload object for managing stateful applications, providing stable network identities and persistent storage.

**Storage Class** — Cloud Storage tier (Standard, Nearline, Coldline, Archive) that determines storage cost, retrieval cost, and minimum storage duration.

**Storage Transfer Service** — GCP service for transferring large amounts of data from on-premises, other clouds, or URLs into Cloud Storage.

**Subnet (Subnetwork)** — A regional IP address range within a VPC where VMs are provisioned; each subnet belongs to exactly one region.

**Sustained Use Discounts (SUDs)** — Automatic Compute Engine discounts of up to 30% applied when a VM runs for more than 25% of a billing month.

**Terraform** — Open-source Infrastructure as Code tool by HashiCorp used to define and provision GCP resources declaratively.

**TPU (Tensor Processing Unit)** — Google-designed application-specific integrated circuit (ASIC) optimized for machine learning workloads, particularly TensorFlow and JAX.

**Traffic Splitting** — Cloud Run / App Engine feature that directs a configurable percentage of traffic to multiple revisions, enabling canary or blue/green deployments.

**Uptime Check** — Cloud Monitoring feature that periodically probes a URL or IP from multiple locations to verify service availability.

**Vertical Autoscaling** — Scaling by increasing or decreasing the CPU and memory allocated to existing instances (as opposed to adding more instances).

**VPC (Virtual Private Cloud)** — Logically isolated network within GCP where you can launch and manage resources with custom IP ranges, subnets, and routing.

**VPC Network Peering** — Feature that connects two VPC networks so resources in each network can communicate using internal IP addresses.

**VM (Virtual Machine)** — Emulation of a physical computer running an operating system and applications; the core unit of Compute Engine.

**VM Manager** — Suite of tools for managing operating systems on Compute Engine VM fleets, including patch management and inventory.

**WebSocket** — Protocol providing full-duplex communication over a single TCP connection, supported by Cloud Run for real-time applications.

**Zone** — An isolated deployment area within a region (e.g., `us-central1-a`); zonal resources exist in exactly one zone, and zones within a region have low-latency connectivity.
