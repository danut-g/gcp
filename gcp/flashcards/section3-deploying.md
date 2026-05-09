# Section 3 — Deploying and Implementing Cloud Solutions — Flashcards

---
### Q1: What is the default boot disk size for a Linux VM on Compute Engine?
**A:** 10 GB. For Windows VMs, the default is 50 GB.

---
### Q2: What gcloud command creates a Compute Engine VM with no external IP?
**A:** `gcloud compute instances create my-vm --zone=us-central1-a --machine-type=e2-medium --image-family=debian-12 --image-project=debian-cloud --no-address`

---
### Q3: What is the difference between pd-balanced and pd-ssd persistent disks?
**A:** pd-balanced offers 6 IOPS per GB (read/write) and is the default general-purpose disk. pd-ssd offers 30 IOPS per GB and is suited for databases and random I/O workloads, at a higher cost.

---
### Q4: True or False: You can decrease the size of a Compute Engine persistent disk.
**A:** False. You can only increase disk size, never decrease it. This can be done without downtime.

---
### Q5: What happens to data on a local SSD when a VM is stopped or terminated?
**A:** The data is lost. Local SSDs are physically attached to the host and do not persist beyond the VM's lifecycle, unlike persistent disks which are network-attached.

---
### Q6: A VM has a GPU attached. What must the host maintenance policy be set to?
**A:** TERMINATE. VMs with GPUs cannot be live-migrated, so the maintenance policy must be set to TERMINATE instead of the default MIGRATE.

---
### Q7: What is a sole-tenant node?
**A:** A dedicated physical server that hosts only your project's VMs. Used for licensing compliance (BYOL), security isolation, or performance requirements.

---
### Q8: What is the difference between OS Login and metadata-managed SSH keys?
**A:** OS Login uses IAM to manage SSH access, supports 2FA, and links Linux accounts to Google identities. Metadata SSH keys require manual key management at the project or instance level and do not support 2FA.

---
### Q9: What IAM role grants SSH access with sudo privileges when using OS Login?
**A:** `roles/compute.osAdminLogin`. For standard non-root SSH access, use `roles/compute.osLogin`.

---
### Q10: What gcloud command connects via SSH to a VM that has no external IP?
**A:** `gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap`. This tunnels through Identity-Aware Proxy. A firewall rule must allow TCP from 35.235.240.0/20 on port 22.

---
### Q11: True or False: Instance templates can be edited after creation.
**A:** False. Instance templates are immutable (read-only). To change the configuration, you must create a new instance template.

---
### Q12: What is the difference between a zonal MIG and a regional MIG?
**A:** A zonal MIG runs instances in a single zone. A regional MIG distributes instances across multiple zones in a region, providing higher availability and surviving zone failures.

---
### Q13: What gcloud command performs a canary update on a MIG, sending 20% of instances to a new template?
**A:** `gcloud compute instance-groups managed rolling-action start-update my-mig --version=template=my-template --canary-version=template=my-new-template,target-size=20% --zone=us-central1-a`

---
### Q14: What are the four autoscaling signals available for a Managed Instance Group?
**A:** CPU utilization, HTTP Load Balancer utilization, Cloud Monitoring metrics (custom or built-in), and scheduled (time-based) scaling.

---
### Q15: What is VM Manager used for?
**A:** VM Manager is a suite of tools for managing OS patch deployment, OS inventory, and configuration across large VM fleets. It requires the OS Config agent on VMs.

---
### Q16: What gcloud command configures kubectl to access a GKE cluster?
**A:** `gcloud container clusters get-credentials CLUSTER_NAME --zone=us-central1-a --project=PROJECT_ID`. This updates the ~/.kube/config file.

---
### Q17: What is the difference between GKE Standard and Autopilot modes?
**A:** In Standard mode, you manage nodes and node pools and pay for node VMs. In Autopilot, Google manages nodes, security hardening, and scaling automatically; you pay only per pod resource usage.

---
### Q18: A company needs a GKE cluster that survives zone failures. What should they deploy?
**A:** A regional GKE cluster (using --region instead of --zone). This distributes 3 control plane replicas across zones and spreads nodes across multiple zones.

---
### Q19: What is the purpose of --enable-private-nodes when creating a GKE cluster?
**A:** It creates a private cluster where nodes have no external IP addresses. Outbound internet access requires Cloud NAT, and the control plane gets a private endpoint.

---
### Q20: What are the four GKE release channels?
**A:** Rapid (latest features, most frequent updates), Regular (default, balanced), Stable (most tested, least frequent), and Extended (longer support, patch-only).

---
### Q21: What is the recommended way for GKE pods to access Google Cloud APIs without service account keys?
**A:** Workload Identity. It maps a Kubernetes service account to a Google Cloud service account, eliminating the need for exported keys.

---
### Q22: What Kubernetes Service type creates an external Google Cloud load balancer?
**A:** LoadBalancer. ClusterIP is internal only, NodePort exposes on each node's IP, and ExternalName maps to a DNS name.

---
### Q23: What kubectl command rolls back a deployment to its previous version?
**A:** `kubectl rollout undo deployment/my-app`

---
### Q24: What is Cloud Run?
**A:** A fully managed serverless platform for running containers. It automatically scales from zero to thousands of instances and charges per 100ms of request handling time.

---
### Q25: What gcloud command deploys a Cloud Run service from source code without a pre-built container?
**A:** `gcloud run deploy my-service --source=. --region=us-central1 --allow-unauthenticated`. Cloud Build automatically creates the container image.

---
### Q26: How do you perform a canary deployment on Cloud Run, sending 10% of traffic to a new revision?
**A:** `gcloud run services update-traffic my-service --region=us-central1 --to-revisions=old-revision=90,new-revision=10`

---
### Q27: What is the difference between Cloud Functions Gen 1 and Gen 2?
**A:** Gen 2 is built on Cloud Run and offers higher limits: 60-minute timeout (vs. 9 min), 32 GB memory (vs. 8 GB), up to 1000 concurrent requests per instance (vs. 1), and traffic splitting support.

---
### Q28: What is Eventarc?
**A:** A unified event routing service for Google Cloud that connects 100+ event sources (Cloud Storage, Pub/Sub, Cloud Audit Logs, etc.) to targets like Cloud Run, Cloud Functions, GKE, and Workflows.

---
### Q29: A team has a containerized app with 5 HTTP endpoints and wants serverless scaling with no Kubernetes overhead. What should they use?
**A:** Cloud Run. It supports multi-endpoint container applications with serverless autoscaling and no cluster management, unlike Cloud Functions which is designed for single-purpose functions.

---
### Q30: What Cloud Run ingress setting allows traffic only from within your VPC and through an external Application LB?
**A:** `--ingress=internal-and-cloud-load-balancing`

---
### Q31: What gcloud command creates a Cloud SQL instance with high availability?
**A:** Include `--availability-type=REGIONAL` when creating the instance. This creates a failover replica in a different zone with automatic failover.

---
### Q32: True or False: You can change the Firestore database mode from Datastore to Native after creation.
**A:** False. The mode cannot be changed after the database is created. You must create a new database and migrate data.

---
### Q33: What is the difference between BigQuery partitioning and clustering?
**A:** Partitioning divides a table into segments by date, timestamp, or integer range. Clustering sorts data within partitions by specified columns. Both reduce bytes scanned and query cost.

---
### Q34: What is the relationship between Cloud Spanner processing units and nodes?
**A:** 1 node equals 1000 processing units. The minimum is 100 processing units for development workloads.

---
### Q35: What is the difference between a Pub/Sub pull and push subscription?
**A:** In a pull subscription, the subscriber explicitly requests messages. In a push subscription, Pub/Sub delivers messages to an HTTP endpoint automatically. Use push for Cloud Run/Functions; pull for custom applications.

---
### Q36: What gcloud command gracefully stops a Dataflow streaming job by processing remaining in-flight data?
**A:** `gcloud dataflow jobs drain JOB_ID --region=us-central1`. This differs from `cancel`, which stops immediately.

---
### Q37: A company needs to transfer 500 TB of on-premises data to Cloud Storage but network transfer would take months. What should they use?
**A:** Transfer Appliance -- a physical device shipped to the location, loaded with data, and shipped back to Google for upload.

---
### Q38: What is the difference between auto mode and custom mode VPC?
**A:** Auto mode automatically creates one subnet per region with predefined IP ranges. Custom mode requires manual subnet creation with user-chosen ranges. Custom mode is recommended for production. Conversion from auto to custom is one-way.

---
### Q39: True or False: VPC peering is transitive.
**A:** False. VPC peering is non-transitive. If VPC-A peers with VPC-B and VPC-B peers with VPC-C, VPC-A cannot communicate with VPC-C through VPC-B.

---
### Q40: What IP range must be allowed in a firewall rule to enable IAP-based SSH tunneling?
**A:** 35.235.240.0/20 on TCP port 22.

---
### Q41: What is the difference between targeting firewall rules with network tags vs. service accounts?
**A:** Network tags are applied per instance and any instance admin can change them. Service accounts are IAM-managed, require permissions to assign, and are more secure. Service accounts are recommended for production.

---
### Q42: What is the difference between HA VPN and Classic VPN?
**A:** HA VPN provides 99.99% SLA, requires 2+ tunnels, and uses dynamic BGP routing. Classic VPN offers 99.9% SLA, uses 1 tunnel, and supports static or dynamic routing. HA VPN is recommended for production.

---
### Q43: What is Shared VPC?
**A:** Shared VPC allows an organization to share a VPC network from a host project with multiple service projects. It centralizes network administration while allowing separate project billing.

---
### Q44: What is Private Google Access?
**A:** A subnet-level setting that allows VMs with only internal IPs to access Google APIs and services (Cloud Storage, BigQuery, etc.) without needing an external IP address.

---
### Q45: What is the basic Terraform workflow?
**A:** `terraform init` (download providers, configure backend) -> `terraform plan` (preview changes) -> `terraform apply` (create/update resources) -> `terraform destroy` (delete all resources).

---
### Q46: Where should Terraform state be stored for a team working on the same GCP project?
**A:** In a GCS backend (`backend "gcs"`) with state locking enabled and versioning on the bucket. Never store state locally in production.

---
### Q47: What is the Cloud Foundation Toolkit (CFT)?
**A:** A collection of opinionated, production-ready Terraform modules built by Google that implement best practices for common GCP architectures like projects, networks, GKE clusters, and IAM.

---
### Q48: What is Config Connector?
**A:** A Kubernetes add-on that manages GCP resources using Kubernetes resource definitions (YAML/kubectl). It treats GCP resources as custom Kubernetes resources and enables GitOps workflows.

---
### Q49: What is Helm?
**A:** A package manager for Kubernetes that bundles manifests into reusable charts. It manages releases (install, upgrade, rollback) and supports templating for configurable deployments.

---
### Q50: What happens when you run `terraform apply` after a managed resource was manually deleted in GCP?
**A:** Terraform detects the resource is missing from GCP (but still in its state file) and recreates it to match the desired configuration.

---
### Q51: What is the difference between Cloud NGFW Hierarchical Firewall Policies and VPC Firewall Rules?
**A:** Hierarchical policies are applied at Org or Folder level and evaluated before VPC rules — they enforce org-wide controls that cannot be overridden at lower levels. VPC rules apply to a single VPC. Hierarchical policies support `goto_next` action to allow lower levels to also evaluate rules.

---
### Q52: What are Secure Tags in Cloud NGFW and why are they more secure than network tags?
**A:** Secure Tags are IAM-governed resource tags that require `roles/resourcemanager.tagUser` to assign. Unlike network tags (which any VM admin can set), Secure Tags cannot be self-assigned — tag bindings are controlled via IAM, preventing privilege escalation.

---
### Q53: List the 5 stages of Fabric FAST in order.
**A:** Stage 0 (Bootstrap) → Stage 1 (Resource Management) → Stage 2 Networking → Stage 2 Security → Stage 3 (Project Factory). Each stage's outputs are inputs to the next, creating a dependency chain.

---
### Q54: What Terraform configuration prevents a critical resource from being accidentally destroyed?
**A:** Add a `lifecycle` block with `prevent_destroy = true`. This causes `terraform destroy` or any plan that would delete the resource to fail with an error.

---
### Q55: What gcloud command removes a resource from Terraform state without deleting it from GCP?
**A:** `terraform state rm RESOURCE_ADDRESS`. This "forgets" the resource from Terraform's state without destroying it in the cloud.

---
### Q56: What are the 6 steps to configure Workload Identity for a GKE pod?
**A:** (1) Enable Workload Identity on cluster (`--workload-pool=PROJECT.svc.id.goog`). (2) Create a GCP SA. (3) Grant GCP SA the needed GCP permissions. (4) Bind K8s SA to GCP SA via `roles/iam.workloadIdentityUser`. (5) Annotate K8s SA with GCP SA email. (6) Reference K8s SA in pod spec.

---
### Q57: Why is Workload Identity preferred over mounting service account key files in GKE pods?
**A:** Key files are long-lived secrets that risk leakage via logs, image layers, or git history. Workload Identity uses short-lived, automatically rotated tokens. No key file = no key rotation burden and no risk of key theft.
