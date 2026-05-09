# Google Cloud Associate Cloud Engineer — Exam Documentation Index

**Depth:** Advanced | **Format:** Exam-focused | **Last Updated:** 2025

This documentation set covers all five official exam domains for the Google Cloud Associate Cloud Engineer (ACE) certification. Each file is written at advanced depth, covering behavioral nuances, architectural decision guidance, common exam traps, and cross-references between related topics.

---

## Exam Domain Weights

| Domain | Title | Weight |
|--------|-------|--------|
| 1 | Setting up a cloud solution environment | ~17.5% |
| 2 | Planning and configuring a cloud solution | ~17.5% |
| 3 | Deploying and implementing a cloud solution | ~25% |
| 4 | Ensuring successful operation of a cloud solution | ~20% |
| 5 | Configuring access and security | ~20% |

---

## Domain 1: Setting up a cloud solution environment (~17.5%)

| File | Topic | Description |
|------|-------|-------------|
| [projects-and-org.md](domain-1-setup-and-configure/projects-and-org.md) | Resource Hierarchy | Organizations, folders, projects, resource inheritance, and policy propagation |
| [billing.md](domain-1-setup-and-configure/billing.md) | Billing Accounts | Billing accounts, budgets, alerts, export, quotas, and cost control mechanisms |
| [cloud-sdk-cli.md](domain-1-setup-and-configure/cloud-sdk-cli.md) | Cloud SDK & CLI | gcloud, gsutil, bq CLI tools, configurations, and multi-project management |
| [iam-overview.md](domain-1-setup-and-configure/iam-overview.md) | IAM Overview | IAM basics, primitive/predefined/custom roles, policies, service account overview |

---

## Domain 2: Planning and configuring a cloud solution (~17.5%)

| File | Topic | Description |
|------|-------|-------------|
| [compute-engine-planning.md](domain-2-plan-and-configure/compute-engine-planning.md) | Compute Engine Planning | Machine types, custom VMs, preemptible/spot instances, committed use discounts |
| [gke-planning.md](domain-2-plan-and-configure/gke-planning.md) | GKE Planning | Autopilot vs Standard, cluster modes, node pools, regional vs zonal clusters |
| [cloud-run-functions-planning.md](domain-2-plan-and-configure/cloud-run-functions-planning.md) | Cloud Run / Functions / App Engine | Decision framework for serverless and PaaS options |
| [storage-planning.md](domain-2-plan-and-configure/storage-planning.md) | Storage Planning | Cloud Storage classes, Cloud SQL, Spanner, Bigtable, Firestore, Memorystore |
| [network-planning.md](domain-2-plan-and-configure/network-planning.md) | Network Planning | VPC design, subnets, CIDR, peering, Shared VPC, regions/zones |
| [pricing-optimization.md](domain-2-plan-and-configure/pricing-optimization.md) | Pricing & Optimization | Sustained use discounts, committed use, rightsizing, cost comparison |

---

## Domain 3: Deploying and implementing a cloud solution (~25%)

| File | Topic | Description |
|------|-------|-------------|
| [compute-engine-deploy.md](domain-3-deploy-and-implement/compute-engine-deploy.md) | Compute Engine Deployment | Instance creation, instance templates, MIGs, startup scripts, snapshots |
| [gke-deploy.md](domain-3-deploy-and-implement/gke-deploy.md) | GKE Deployment | Cluster creation, Deployments, Services, Ingress, ConfigMaps, Secrets |
| [cloud-run-deploy.md](domain-3-deploy-and-implement/cloud-run-deploy.md) | Cloud Run Deployment | Container deployment, revisions, traffic splitting, scaling, concurrency |
| [cloud-functions-deploy.md](domain-3-deploy-and-implement/cloud-functions-deploy.md) | Cloud Functions Deployment | Triggers, runtimes, environment variables, gen1 vs gen2 |
| [app-engine-deploy.md](domain-3-deploy-and-implement/app-engine-deploy.md) | App Engine Deployment | Standard vs Flexible, versions, traffic splitting, services |
| [data-solutions-deploy.md](domain-3-deploy-and-implement/data-solutions-deploy.md) | Data Solutions Deployment | Cloud SQL, Bigtable, Spanner, Firestore, BigQuery deployment patterns |
| [networking-deploy.md](domain-3-deploy-and-implement/networking-deploy.md) | Networking Deployment | VPC creation, firewall rules, load balancing, Cloud DNS |
| [iac-deployment-manager.md](domain-3-deploy-and-implement/iac-deployment-manager.md) | IaC & Deployment Manager | Deployment Manager, Terraform on GCP, Config Connector |

---

## Domain 4: Ensuring successful operation (~20%)

| File | Topic | Description |
|------|-------|-------------|
| [monitoring-cloud-ops.md](domain-4-ensure-success/monitoring-cloud-ops.md) | Cloud Monitoring | Metrics, dashboards, alerting policies, uptime checks, SLOs |
| [logging.md](domain-4-ensure-success/logging.md) | Cloud Logging | Log types, log sinks, log-based metrics, audit logs, retention |
| [managing-compute.md](domain-4-ensure-success/managing-compute.md) | Managing Compute Engine | Autoscaling, rolling updates, instance repair, snapshots, backups |
| [managing-gke.md](domain-4-ensure-success/managing-gke.md) | Managing GKE | Cluster upgrades, node pool management, workload identity, scaling |
| [managing-storage.md](domain-4-ensure-success/managing-storage.md) | Managing Storage | Lifecycle policies, versioning, retention locks, transfer services |
| [managing-networking.md](domain-4-ensure-success/managing-networking.md) | Managing Networking | Cloud NAT, Cloud VPN, Interconnect, routing, DNS management |

---

## Domain 5: Configuring access and security (~20%)

| File | Topic | Description |
|------|-------|-------------|
| [iam-advanced.md](domain-5-configure-access-and-security/iam-advanced.md) | Advanced IAM | Custom roles, IAM conditions, policy troubleshooting, least privilege patterns |
| [service-accounts.md](domain-5-configure-access-and-security/service-accounts.md) | Service Accounts | Creation, keys, Workload Identity Federation, impersonation, best practices |
| [vpc-security.md](domain-5-configure-access-and-security/vpc-security.md) | VPC Security | Firewall rules, VPC Service Controls, Private Google Access, firewall policies |
| [data-security.md](domain-5-configure-access-and-security/data-security.md) | Data Security | CMEK, Cloud KMS, Secret Manager, Cloud DLP, data classification |

---

## Cross-Reference Guide

Key relationships between topics:

- **IAM** → See both [iam-overview.md](domain-1-setup-and-configure/iam-overview.md) (basics) and [iam-advanced.md](domain-5-configure-access-and-security/iam-advanced.md) (advanced)
- **Service Accounts** → [iam-overview.md](domain-1-setup-and-configure/iam-overview.md) introduces them; [service-accounts.md](domain-5-configure-access-and-security/service-accounts.md) covers advanced usage
- **Networking** → [network-planning.md](domain-2-plan-and-configure/network-planning.md) for design; [networking-deploy.md](domain-3-deploy-and-implement/networking-deploy.md) for creation; [managing-networking.md](domain-4-ensure-success/managing-networking.md) for operations; [vpc-security.md](domain-5-configure-access-and-security/vpc-security.md) for security
- **GKE** → [gke-planning.md](domain-2-plan-and-configure/gke-planning.md) → [gke-deploy.md](domain-3-deploy-and-implement/gke-deploy.md) → [managing-gke.md](domain-4-ensure-success/managing-gke.md)
- **Billing** → [billing.md](domain-1-setup-and-configure/billing.md) for accounts/budgets; [pricing-optimization.md](domain-2-plan-and-configure/pricing-optimization.md) for cost optimization
- **Storage** → [storage-planning.md](domain-2-plan-and-configure/storage-planning.md) for selection; [data-solutions-deploy.md](domain-3-deploy-and-implement/data-solutions-deploy.md) for deployment; [managing-storage.md](domain-4-ensure-success/managing-storage.md) for lifecycle management

---

## Quick Decision Frameworks

### Compute Selection
```
Stateful/long-running workload → Compute Engine
Containerized microservices, need cluster control → GKE Standard
Containerized microservices, fully managed → GKE Autopilot or Cloud Run
Stateless HTTP containers, scale-to-zero → Cloud Run
Event-driven, short-duration functions → Cloud Functions
Monolithic web app → App Engine
```

### Storage Selection
```
Unstructured files/objects → Cloud Storage
Relational, OLTP, <few TB → Cloud SQL (PostgreSQL/MySQL/SQL Server)
Relational, global, ACID, horizontal scale → Cloud Spanner
Wide-column, petabyte-scale, low-latency reads → Bigtable
Document database, mobile/web apps → Firestore
OLAP/Analytics at scale → BigQuery
In-memory cache/session store → Memorystore (Redis/Memcached)
```

---

## Source

- Official Exam Guide: https://cloud.google.com/certification/guides/cloud-engineer
- GCP Documentation: https://cloud.google.com/docs
