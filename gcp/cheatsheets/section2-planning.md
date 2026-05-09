# Section 2 Cheat Sheet -- Planning Compute, Storage & Network

---

## COMPUTE -- Decision Flowchart

```
Need full OS / GPU / Windows / lift-and-shift?      --> Compute Engine
Need container orchestration / K8s?                 --> GKE (Autopilot for less ops, Standard for control)
K8s + Knative Serving (custom autoscaling/eventing)?--> GKE + Knative
Stateless container, no K8s overhead?               --> Cloud Run
Simple event-driven function?                       --> Cloud Functions (Gen 2)
Not sure?                                           --> Cloud Run (safest default)
```

### Compute Quick-Compare

| | Compute Engine | GKE | Cloud Run | Cloud Functions |
|---|---|---|---|---|
| Model | IaaS (VMs) | CaaS (K8s) | Serverless containers | FaaS |
| State | Stateful | Both | Stateless | Stateless |
| Scale to zero | No | No | Yes | Yes |
| Startup | Minutes | Seconds | Seconds | Seconds |
| Max timeout | Unlimited | Unlimited | 60 min | 60 min (Gen2) |
| Pricing | Per-second (min 1 min) | Cluster fee + VMs | Per request + CPU/mem | Per invocation + compute |

### Compute Engine Machine Types

- Naming: `FAMILY-RATIO-vCPUs` (e.g. `e2-standard-4`)
- Ratios: `standard` = 4 GB/vCPU, `highmem` = 8 GB, `highcpu` = 1 GB
- Custom types: N1, N2, N2D, E2 only -- memory in 256 MB increments

| Family | Prefix | Use |
|---|---|---|
| General-purpose | E2, N2, N2D, C3 | Balanced workloads |
| Compute-optimized | C2, C2D, H3 | HPC, batch, gaming |
| Memory-optimized | M1, M2, M3 | SAP HANA, in-memory DB |
| Accelerator-optimized | A2, A3, G2 | ML training, GPU |

### Spot VMs

- **60-91% discount**, can be reclaimed with 30s notice
- Spot = no max lifetime; Preemptible (legacy) = 24h max
- Good for: batch, CI/CD, rendering, ML training with checkpointing
- Bad for: production serving, databases, anything needing SLA

### Compute Pricing Levers

| Mechanism | Discount | Commitment |
|---|---|---|
| On-demand | 0% | None |
| Sustained Use (auto) | Up to 30% | Run >25% of month |
| CUD 1-year | Up to 57% | 1-year commitment |
| CUD 3-year | Up to 70% | 3-year commitment |
| Spot VMs | 60-91% | None (can be reclaimed) |

- Free tier: 1x e2-micro/month (us-west1, us-central1, us-east1 only)

### GKE Modes

| | Standard | Autopilot |
|---|---|---|
| Node mgmt | You | Google |
| Pricing | $0.10/hr cluster + VMs | Per pod resources |
| Best for | Full control | Most workloads |

### Cloud Run vs Cloud Functions

| | Cloud Run | Cloud Functions |
|---|---|---|
| Deploy unit | Container image | Source code |
| Languages | Any | Limited runtimes |
| Concurrency | Multiple req/instance | 1 (Gen1) / many (Gen2) |
| Timeout | 60 min | 9 min (Gen1) / 60 min (Gen2) |

---

## STORAGE & DATABASES -- Decision Flowchart

```
Relational + single region + <64 TB?            --> Cloud SQL
Relational + PostgreSQL + need HTAP perf?       --> AlloyDB
Relational + global + strong consistency?       --> Cloud Spanner
Analytical / data warehouse / SQL on PBs?       --> BigQuery
NoSQL document / mobile+web / real-time sync?   --> Firestore (Native mode)
NoSQL wide-column / time-series / >1 TB / IoT?  --> Bigtable
Object storage (files, backups, media)?         --> Cloud Storage
Block storage for VMs?                          --> Persistent Disk
Shared filesystem (NFS)?                        --> Filestore
```

### Database Quick-Compare

| | Cloud SQL | AlloyDB | Spanner | BigQuery | Firestore | Bigtable |
|---|---|---|---|---|---|---|
| Type | Relational | Relational (PG) | Relational | Analytical | Document NoSQL | Wide-column NoSQL |
| Workload | OLTP | OLTP+OLAP | OLTP | OLAP | Mobile/Web | Time-series, IoT |
| Scale | Vertical | Vertical | Horizontal | Serverless | Serverless | Horizontal |
| Max size | 64 TB | 128+ TB | Unlimited | Unlimited | 1 TB/db | Unlimited |
| Consistency | Strong | Strong | Strong (global) | Eventual | Strong | Eventual |
| SLA | 99.95-99.99% | 99.99% | **99.999%** | 99.99% | 99.999% | 99.99% |
| Cost entry | Low | Medium | **High** | Free tier | Free tier | Medium |

### Cloud Storage Classes

| Class | Min Duration | Retrieval Cost | Use Case |
|---|---|---|---|
| Standard | None | Free | Hot data, frequently accessed |
| Nearline | 30 days | $ | < 1x/month access |
| Coldline | 90 days | $$ | < 1x/quarter access |
| Archive | 365 days | $$$ | < 1x/year, compliance |

- Storage cost: Standard > Nearline > Coldline > Archive
- Retrieval cost: Archive > Coldline > Nearline > Standard (free)
- All classes have **same millisecond latency** (not like AWS Glacier!)
- Early deletion = charged for remaining min duration
- Lifecycle rules can auto-transition objects between classes

### Persistent Disk Types

| Type | Performance | Use |
|---|---|---|
| pd-standard | Low (HDD) | Bulk, sequential I/O |
| pd-balanced | Medium (SSD) | General purpose (default) |
| pd-ssd | High (SSD) | Databases, random I/O |
| pd-extreme | Highest (SSD) | Mission-critical DBs |

- **Zonal PD**: single zone, standard cost
- **Regional PD**: 2 zones, ~2x cost, faster failover
- **Local SSD**: ephemeral, extreme IOPS, data lost on stop/delete

### Hyperdisk (Next-Gen Block Storage)

Decouples IOPS/throughput from capacity — set them independently.

| Type | Max IOPS | Max Throughput | Use |
|---|---|---|---|
| Hyperdisk Balanced | 160,000 | 2,400 MB/s | General workloads |
| Hyperdisk Extreme | 350,000 | 5,000 MB/s | Latency-critical DBs |
| Hyperdisk Throughput | 1,200 | 2,400 MB/s | Kafka, Hadoop (sequential) |
| Hyperdisk ML | — | 4,800 MB/s | Multi-reader for ML models |

```bash
gcloud compute disks create my-hd --type=hyperdisk-balanced \
  --size=1TB --zone=ZONE --provisioned-iops=6000 --provisioned-throughput=500
```

- Hyperdisk ML: **multi-reader** (many VMs read same disk simultaneously)
- Key differentiator vs PD: you provision IOPS/throughput independently of size

### NetApp Volumes (Enterprise NAS)

| Aspect | Detail |
|---|---|
| Protocol | NFS v3/v4.1, SMB 3.x |
| Built on | NetApp ONTAP |
| Service levels | Standard (16 MB/s/TiB), Premium (64), Extreme (128) |
| Use case | Enterprise NFS/SMB, SAP, EPIC, Windows workloads |
| vs Filestore | NetApp = enterprise features (snapshots, SnapMirror, tiering) |

```bash
gcloud netapp storage-pools create my-pool --location=us-central1 \
  --service-level=PREMIUM --capacity-gib=1024 --network=default
gcloud netapp volumes create my-vol --location=us-central1 \
  --storage-pool=my-pool --capacity-gib=100 --protocols=NFSV3
```

### Managed Apache Kafka

| Feature | Pub/Sub | Managed Kafka |
|---|---|---|
| API | Proprietary (HTTP/gRPC) | Native Kafka API |
| Migration from Kafka | Requires rewrite | **Drop-in replacement** |
| Ordering | Per topic | Per partition |
| Replay | No (once acked) | Yes (offset control) |
| Use when | New GCP-native apps | Migrating existing Kafka |

```bash
gcloud managed-kafka clusters create my-cluster \
  --location=us-central1 --cpu=3 --memory=3Gi \
  --subnets=projects/P/regions/R/subnetworks/default
```

---

## NETWORK -- Decision Flowchart

### Load Balancer Selection

```
EXTERNAL traffic:
  HTTP/S?           --> External Application LB (global, L7)
  TCP + SSL term?   --> External Proxy Network LB (global, L4)
  TCP/UDP raw?      --> External Passthrough Network LB (regional, L4)

INTERNAL traffic:
  HTTP/S?           --> Internal Application LB (regional, L7)
  TCP/UDP?          --> Internal Passthrough Network LB (regional, L4)
```

### Load Balancer Quick-Compare

| LB | Scope | Layer | Protocol | Key Feature |
|---|---|---|---|---|
| Ext Application | Global | L7 | HTTP/S | URL maps, CDN, Cloud Armor, anycast IP |
| Int Application | Regional | L7 | HTTP/S | Internal microservices routing |
| Ext Proxy Network | Global | L4 | TCP/SSL | SSL termination, non-HTTP |
| Ext Passthrough Network | Regional | L4 | TCP/UDP | Preserves source IP, UDP support |
| Int Passthrough Network | Regional | L4 | TCP/UDP | DB HA, internal services |

### Network Service Tiers

| | Premium (default) | Standard |
|---|---|---|
| Routing | Google private backbone | Public internet |
| Latency | Lower | Higher |
| Global LB | Yes | No (regional only) |
| Cost | Higher | Lower |
| Use | Production | Dev/test, cost-sensitive |

### VPC Essentials

- VPC = **global**, Subnet = **regional**, VM = **zonal**
- Auto-mode VPC: subnets created automatically in all regions
- Custom-mode VPC: you define subnets manually

### Resource Scope

| Global | Regional | Zonal |
|---|---|---|
| VPC, firewall rules, routes | Subnets, regional PD, Cloud NAT | VMs, zonal PD, local SSDs |
| External IPs, images, snapshots | Cloud Router, managed instance groups | GKE node pools |
| Some load balancers, CDN | Internal IPs | |

### Connectivity to On-Premises

| Option | Bandwidth | Use Case |
|---|---|---|
| Dedicated Interconnect | 10-200 Gbps | Large, predictable transfers |
| Partner Interconnect | 50 Mbps-50 Gbps | Medium bandwidth |
| HA VPN | 1.4-3 Gbps/tunnel | Encrypted over internet |

### Cloud NAT

- Outbound-only internet access for VMs **without** external IPs
- No proxy VMs needed; software-defined; regional resource

### Cloud CDN

- Works with External Application LB only
- Caches static content at Google edge PoPs
- Supports signed URLs/cookies

---

## Multi-Region Redundancy

| Service | Multi-Region Strategy |
|---|---|
| Cloud Spanner | Multi-region config at instance creation (`nam6`, `eur3`, etc.) |
| Cloud SQL | Cross-region read replicas + failover; no automatic multi-region |
| BigQuery | Multi-region dataset location (`US`, `EU`) |
| Cloud Storage | Multi-region or dual-region bucket |
| Bigtable | Replication clusters in different regions |
| Pub/Sub | Global by default |
| Cloud Run | Deploy same revision to multiple regions + Global LB |

**Design principle:** For 99.99%+ multi-region SLA, use Spanner (global strong consistency) or Bigtable with replication.

---

## EXAM TRAPS & GOTCHAS

- :warning: Cloud Storage: ALL classes have **millisecond latency** -- do NOT confuse with AWS Glacier
- :warning: Spanner is expensive (~$0.90/node/hr) -- don't pick it for small apps; Cloud SQL is cheaper
- :warning: Bigtable is NOT cost-effective for <1 TB of data
- :warning: Firestore Native mode = real-time + offline; Datastore mode = server-side only -- **cannot switch modes after creation**
- :warning: Spot VMs: no SLA, 30s warning -- never use for production serving or HA databases
- :warning: Sustained Use Discounts are **automatic** (no commitment); CUDs require commitment
- :warning: Custom machine types cost **more per unit** than predefined equivalents
- :warning: Cloud Functions Gen 1: 1 concurrent request/instance, 9 min timeout; Gen 2 fixes both
- :warning: Standard network tier does **NOT** support global load balancing
- :warning: Regional PD costs ~2x zonal PD -- only use when HA justifies the cost
- :warning: Cloud Run scales to zero = zero cost when idle; GKE does NOT scale to zero (cluster fee always running)
- :warning: Hyperdisk IOPS/throughput are **provisioned independently** of disk size (unlike Persistent Disk where IOPS scale with size)
- :warning: Hyperdisk ML is **read-only multi-attach** -- for ML model serving from one disk to many VMs
- :warning: Managed Kafka = use when migrating existing Kafka; Pub/Sub = use for new GCP-native messaging
- :warning: NetApp Volumes is for enterprise NFS/SMB (SAP, EPIC) -- Filestore is simpler but less featured
- :warning: BigQuery on-demand: first 1 TB/month scanned is free; storage auto-drops to long-term rate after 90 days
- :warning: GKE Standard charges $0.10/hr cluster management fee; Autopilot has no cluster fee (pay per pod)
- :warning: External Passthrough Network LB is the ONLY option for UDP traffic
- :warning: Cloud SQL max = 64 TB; if you need more, consider AlloyDB, Spanner, or Bigtable
- :warning: Sole-tenant nodes: for compliance (BYOL licensing, physical isolation) -- not for performance
- :warning: Region choice factors: latency, compliance/data residency, pricing (us-central1 cheapest), service availability
