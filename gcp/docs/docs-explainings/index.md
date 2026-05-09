# GCP ACE Exam Documentation — Dual-Layer Explanation Index

This directory contains dual-layer explanations of every topic in the GCP Associate Cloud Engineer certification documentation set. Each topic is explained using the **Analogy → Technical Precision** framework, building from intuition to expert-level understanding.

---

## How to Use This Index

Each file covers all major concepts from the corresponding source file, structured as:
1. **Analogy** — real-world intuitive explanation
2. **Technical Explanation** — precise GCP-specific mechanisms
3. **10 sections** — What it is → How it works → Mental model → Usage → Under the hood → Failures → Trade-offs → Misconceptions → Expert insight → TL;DR

---

## Domain 1: Setting Up a Cloud Solution Environment (~17.5%)

| File | Topics Covered |
|------|---------------|
| [billing.md](domain-1-setup-and-configure/billing.md) | Billing accounts, budgets, alerts, quota, export to BigQuery |
| [cloud-sdk-cli.md](domain-1-setup-and-configure/cloud-sdk-cli.md) | gcloud CLI, gsutil, bq, configurations, multi-project management |
| [iam-overview.md](domain-1-setup-and-configure/iam-overview.md) | IAM fundamentals, roles (primitive/predefined/custom), policy binding, service accounts intro |
| [projects-and-org.md](domain-1-setup-and-configure/projects-and-org.md) | Resource hierarchy, organizations, folders, projects, policy inheritance |

---

## Domain 2: Planning and Configuring a Cloud Solution (~17.5%)

| File | Topics Covered |
|------|---------------|
| [cloud-run-functions-planning.md](domain-2-plan-and-configure/cloud-run-functions-planning.md) | Cloud Run vs Cloud Functions vs App Engine decision framework |
| [compute-engine-planning.md](domain-2-plan-and-configure/compute-engine-planning.md) | Machine types (N1/N2/E2/C2), custom VMs, preemptible vs Spot, committed use |
| [gke-planning.md](domain-2-plan-and-configure/gke-planning.md) | Autopilot vs Standard, zonal vs regional clusters, node pools |
| [network-planning.md](domain-2-plan-and-configure/network-planning.md) | VPC design, CIDR planning, subnets, Shared VPC, VPC peering |
| [pricing-optimization.md](domain-2-plan-and-configure/pricing-optimization.md) | **SUDs** (automatic), **CUDs** (1-3yr commitment), Preemptible/Spot VMs, Rightsizing, E2 cost-optimized |
| [storage-planning.md](domain-2-plan-and-configure/storage-planning.md) | **Cloud Storage** classes, **Cloud SQL**, **Cloud Spanner**, **Bigtable**, **Firestore**, **Memorystore**, storage selection matrix |

---

## Domain 3: Deploying and Implementing a Cloud Solution (~25%)

| File | Topics Covered |
|------|---------------|
| [app-engine-deploy.md](domain-3-deploy-and-implement/app-engine-deploy.md) | Standard vs Flexible environments, versions, traffic splitting, services |
| [cloud-functions-deploy.md](domain-3-deploy-and-implement/cloud-functions-deploy.md) | Triggers (HTTP, Pub/Sub, Storage, Eventarc), runtimes, Gen1 vs Gen2 |
| [cloud-run-deploy.md](domain-3-deploy-and-implement/cloud-run-deploy.md) | Container deployment, revisions, traffic splitting, concurrency, scaling |
| [compute-engine-deploy.md](domain-3-deploy-and-implement/compute-engine-deploy.md) | **Instance templates** (immutable), **MIGs** (autoscaling, autohealing, rolling updates), startup scripts, disk types, snapshots |
| [data-solutions-deploy.md](domain-3-deploy-and-implement/data-solutions-deploy.md) | **Cloud SQL HA** (~60s failover), **Bigtable** row key design, **Spanner** processing units, **Firestore** mode selection, **BigQuery** partitioning |
| [gke-deploy.md](domain-3-deploy-and-implement/gke-deploy.md) | Cluster creation (immutable decisions), Deployments, Services, Ingress, ConfigMaps, Secrets, Workload Identity |
| [iac-deployment-manager.md](domain-3-deploy-and-implement/iac-deployment-manager.md) | IaC principles, Deployment Manager (YAML/Python/Jinja2), Terraform on GCP (HCL, state in GCS) |
| [networking-deploy.md](domain-3-deploy-and-implement/networking-deploy.md) | **VPC** (global, custom mode), **Firewall rules** (stateful, tag-based), **Cloud Load Balancing** (global HTTP(S), health check IPs), **Cloud DNS** |

---

## Domain 4: Ensuring Successful Operation (~20%)

| File | Topics Covered |
|------|---------------|
| [logging.md](domain-4-ensure-success/logging.md) | Cloud Logging, log types, log sinks (GCS/BigQuery/Pub/Sub), audit logs, log-based metrics, retention |
| [managing-compute.md](domain-4-ensure-success/managing-compute.md) | Autoscaling configuration, rolling updates, canary deployments, snapshot schedules |
| [managing-gke.md](domain-4-ensure-success/managing-gke.md) | Cluster upgrades, release channels, node pool management, Workload Identity |
| [managing-networking.md](domain-4-ensure-success/managing-networking.md) | Cloud NAT, Cloud VPN (HA vs Classic), Dedicated/Partner Interconnect, Cloud Router |
| [managing-storage.md](domain-4-ensure-success/managing-storage.md) | **Lifecycle policies** (daily evaluation, 24h delay), **Versioning** (delete markers, non-current versions), **Retention locks** (permanent), Storage Transfer Service |
| [monitoring-cloud-ops.md](domain-4-ensure-success/monitoring-cloud-ops.md) | **Cloud Monitoring** (Ops Agent for memory), **Alerting policies** (threshold vs absence), **Uptime checks** (public endpoints only), scoping projects |

---

## Domain 5: Configuring Access and Security (~20%)

| File | Topics Covered |
|------|---------------|
| [data-security.md](domain-5-configure-access-and-security/data-security.md) | CMEK vs CSEK vs Google-managed encryption, Cloud KMS, Secret Manager, Cloud DLP basics |
| [iam-advanced.md](domain-5-configure-access-and-security/iam-advanced.md) | **Custom roles** (org/project only, not folder), **IAM Conditions** (CEL, time-bound, tag-based), policy troubleshooting, additive inheritance |
| [service-accounts.md](domain-5-configure-access-and-security/service-accounts.md) | SA creation, metadata server tokens, JSON keys (avoid), **Workload Identity** for GKE, **impersonation** (serviceAccountTokenCreator) |
| [vpc-security.md](domain-5-configure-access-and-security/vpc-security.md) | **VPC Service Controls** (perimeters, dry-run mode, access levels), **Private Google Access** (internal VMs → GCP APIs), **Hierarchical Firewall Policies** |

---

## Key Decision Frameworks (Dual-Layer)

### Compute Selection

**Analogy**: Match the vehicle to the road type.
- Long-haul truck needing full control → **Compute Engine** (stateful/long-running, full VM control)
- Container fleet with self-managed shipping yard → **GKE Standard** (containerized, need node control)
- Container fleet, managed shipping yard → **GKE Autopilot** (containerized, no node management)
- Delivery van that parks when idle → **Cloud Run** (stateless HTTP containers, scale-to-zero)
- Bicycle courier for quick errands → **Cloud Functions** (event-driven, sub-minute tasks)
- Bus on a fixed route → **App Engine** (monolithic web app, fixed patterns)

**Technical**:
```
VM needed / full OS control → Compute Engine
Containerized + cluster control → GKE Standard
Containerized + managed nodes → GKE Autopilot
Stateless HTTP container + scale-to-zero → Cloud Run
Event-driven + short-duration → Cloud Functions
Monolithic web app + PaaS → App Engine
```

---

### Storage Selection

**Analogy**: Match the storage container to the content type.
- Files and photos → filing cabinet (Cloud Storage)
- Structured customer records in one city → local bank (Cloud SQL)
- Global financial transactions → global bank (Cloud Spanner)
- Time-series sensor data at petabyte scale → specialized index system (Bigtable)
- Mobile app user profiles with real-time sync → collaborative binder (Firestore)
- Analytics on historical data → research library (BigQuery)
- Session store needing sub-millisecond response → in-memory whiteboard (Memorystore)

**Technical**:
```
Unstructured objects/files → Cloud Storage
Relational OLTP + regional + < few TB → Cloud SQL
Relational OLTP + global + horizontal scale → Cloud Spanner
Wide-column + petabyte + IoT/time-series → Bigtable
Document + mobile/web + real-time → Firestore
Analytical SQL on large datasets → BigQuery
In-memory cache/sessions → Memorystore (Redis/Memcached)
```

---

## Critical Exam Traps Summary

| Trap | Correct Answer |
|------|---------------|
| SUDs need configuration | False — automatic, no sign-up |
| E2 machines receive SUDs | False — E2 not eligible |
| CUDs + SUDs stack | False — GCP applies the greater discount only |
| Instance template can be modified | False — immutable; create new template |
| Local SSD survives VM stop | False — data lost on stop (survives restart) |
| Memory metrics come free in Cloud Monitoring | False — requires Ops Agent installation |
| Custom roles can be created at folder level | False — org or project level only |
| Firestore mode can be changed after creation | False — permanent per project |
| Cloud SQL region can be changed | False — permanent after creation |
| Bigtable development instance = production | False — minimum 3 nodes for SLA |
| Lifecycle rules run in real-time | False — evaluated daily, up to 24h delay |
| Retention policy lock is reversible | False — permanent and irreversible |
| VPC is regional in GCP | False — VPCs are global; subnets are regional |
| Uptime checks work for internal services | False — requires public endpoint |
| PITR recovery is in-place | False — creates a new Cloud SQL instance |
| Default service account is safe | False — has project editor access; use dedicated SAs |

---

## Source Reference

- Source documentation: `/Users/danutgornicioiu/learning/gcp/gcp/docs/`
- Official exam guide: https://cloud.google.com/certification/guides/cloud-engineer
- Official GCP documentation: https://cloud.google.com/docs
