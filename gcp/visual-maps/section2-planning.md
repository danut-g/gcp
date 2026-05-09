# Section 2 -- Visual Maps: Planning and Configuring a Cloud Solution

---

## 1. Compute Decision Tree

```
                              START
                                │
                                ▼
              ┌─────────────────────────────────────┐
              │  Do you need full OS control,        │
              │  custom kernel, GPU, or Windows?      │
              └──────────────────┬──────────────────┘
                        ┌───────┴───────┐
                        ▼               ▼
                       YES              NO
                        │               │
                        ▼               ▼
              ┌──────────────┐  ┌──────────────────────────────┐
              │  COMPUTE     │  │  Is it a containerized app?  │
              │  ENGINE      │  └──────────────┬───────────────┘
              │  (IaaS)      │         ┌───────┴───────┐
              └──────────────┘         ▼               ▼
                                      YES              NO
                                       │               │
                                       ▼               ▼
                      ┌────────────────────────┐  ┌─────────────────────┐
                      │  Do you need Kubernetes │  │  Is it a simple     │
                      │  features? (orchestration,│ │  event-driven       │
                      │  service mesh, multi-cloud)│ │  function?          │
                      └──────────┬─────────────┘  └──────────┬──────────┘
                        ┌────────┴────────┐          ┌───────┴───────┐
                        ▼                 ▼          ▼               ▼
                       YES               NO         YES              NO
                        │                 │          │               │
                        ▼                 ▼          ▼               ▼
                 ┌────────────┐   ┌────────────┐ ┌──────────────┐ ┌────────────┐
                 │    GKE     │   │  CLOUD RUN │ │    CLOUD     │ │  CLOUD RUN │
                 │  (CaaS)   │   │ (Serverless│ │  FUNCTIONS   │ │  (default  │
                 │           │   │ Containers)│ │    (FaaS)    │ │   choice)  │
                 └─────┬─────┘   └────────────┘ └──────────────┘ └────────────┘
                       │
                ┌──────┴──────┐
                ▼             ▼
          ┌──────────┐  ┌──────────┐
          │ Standard │  │ Autopilot│
          │ (full    │  │ (managed │
          │ control) │  │  nodes)  │
          └──────────┘  └──────────┘
```

---

## 2. Compute Options -- Comparison Matrix

```
  ┌──────────────────┬────────────────┬────────────────┬────────────────┬────────────────┐
  │                  │ Compute Engine │      GKE       │   Cloud Run    │Cloud Functions │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Model            │  IaaS          │  CaaS          │  Serverless    │  FaaS          │
  │                  │  (VMs)         │  (Containers)  │  (Containers)  │  (Functions)   │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ You manage       │  OS, runtime,  │  Containers,   │  Container     │  Function code │
  │                  │  app, config   │  K8s config    │  image only    │  only          │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Scaling          │  Manual or     │  Pod + Node    │  Auto (0 to N) │  Auto (0 to N) │
  │                  │  autoscaler    │  autoscaler    │                │                │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Startup time     │  Minutes       │  Seconds       │  Seconds       │  Seconds       │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ State            │  Stateful      │  Both          │  Stateless     │  Stateless     │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Max timeout      │  Unlimited     │  Unlimited     │  60 min        │  60 min (gen2) │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Pricing          │  Per second    │  Cluster fee   │  Per request   │  Per invocation│
  │                  │  (min 1 min)   │  + node VMs    │  + CPU/mem     │  + compute     │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Scale to zero    │  No            │  No            │  YES           │  YES           │
  ├──────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤
  │ Best for         │  Legacy, GPU,  │  Microservices │  Stateless     │  Event-driven  │
  │                  │  Windows,      │  hybrid/multi- │  web apps,     │  webhooks,     │
  │                  │  lift-shift    │  cloud, K8s    │  APIs          │  triggers      │
  └──────────────────┴────────────────┴────────────────┴────────────────┴────────────────┘
```

### Control vs Management Spectrum

```
  MORE CONTROL                                                     MORE MANAGED
  (you manage)                                                  (Google manages)
  ◄────────────────────────────────────────────────────────────────────────────►

  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
  │ Compute Engine │  │     GKE        │  │   Cloud Run    │  │Cloud Functions │
  │                │  │                │  │                │  │                │
  │  OS            │  │  Containers    │  │  Container     │  │  Function      │
  │  Runtime       │  │  K8s config    │  │  image         │  │  code          │
  │  App           │  │  Node pools    │  │  (that's it)   │  │  (that's it)   │
  │  Network       │  │                │  │                │  │                │
  │  Storage       │  │                │  │                │  │                │
  └────────────────┘  └────────────────┘  └────────────────┘  └────────────────┘
       IaaS               CaaS            Serverless               FaaS
```

---

## 3. Machine Type Families Mind Map

```
                                          ┌── E2, N2, N2D, N1, C3, C3D
                                          │   Balanced workloads
                              ┌── General │   Web servers, dev envs
                              │  Purpose  │   MOST COMMON
                              │           └──────────────────────────
                              │
                              │           ┌── C2, C2D, H3
                              ├── Compute │   CPU-intensive
                              │  Optimized│   Gaming, HPC, batch
                              │           └──────────────────────────
  Machine Type ───────────────┤
  Families                    │           ┌── M1, M2, M3
                              ├── Memory  │   Large in-memory
                              │  Optimized│   SAP HANA, in-memory DBs
                              │           └──────────────────────────
                              │
                              │           ┌── A2, A3, G2
                              ├── Accel.  │   ML training/inference
                              │  Optimized│   GPU workloads
                              │           └──────────────────────────
                              │
                              │           ┌── Z3
                              └── Storage │   High IOPS
                                 Optimized│   Large local SSD workloads
                                          └──────────────────────────


  NAMING CONVENTION:

       e2  -  standard  -  4
       ──     ────────     ─
       │         │         └── 4 vCPUs
       │         └── CPU:Memory ratio
       │              standard = 4 GB/vCPU
       │              highmem  = 8 GB/vCPU
       │              highcpu  = 1 GB/vCPU
       └── Family prefix
```

---

## 4. Spot VMs Decision Flowchart

```
                          Is your workload
                     fault-tolerant / restartable?
                                │
                       ┌────────┴────────┐
                       ▼                 ▼
                      YES                NO
                       │                 │
                       ▼                 ▼
           Can it handle a 30-sec    ┌──────────────┐
           shutdown notice?          │  Use regular  │
                       │             │  ON-DEMAND    │
                  ┌────┴────┐        │  VMs          │
                  ▼         ▼        └──────────────┘
                 YES        NO
                  │         │
                  ▼         ▼
          ┌──────────────┐  ┌──────────────┐
          │  USE SPOT    │  │  Use regular  │
          │  VMs         │  │  ON-DEMAND    │
          │              │  │  VMs          │
          │  60-91% OFF  │  └──────────────┘
          └──────────────┘

  GOOD FOR SPOT VMs:              BAD FOR SPOT VMs:
  ┌─────────────────────────┐     ┌─────────────────────────┐
  │  Batch processing       │     │  Production web servers  │
  │  CI/CD pipelines        │     │  Databases (HA)          │
  │  Data processing        │     │  SLA-bound services      │
  │  Rendering              │     │  Cannot checkpoint       │
  │  ML training            │     │                          │
  │  Dev/test environments  │     │                          │
  └─────────────────────────┘     └─────────────────────────┘
```

---

## 5. Compute Engine Pricing Options

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                     COMPUTE ENGINE PRICING LADDER                            │
  └──────────────────────────────────────────────────────────────────────────────┘

  MOST EXPENSIVE                                              LEAST EXPENSIVE
  ◄────────────────────────────────────────────────────────────────────────────►

  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐
  │  On-Demand   │  │  SUDs        │  │  CUDs        │  │  Spot VMs            │
  │              │  │  (automatic) │  │  (commitment)│  │  (preemptible)       │
  │  Full price  │  │  Up to 30%   │  │  Up to 57%   │  │  60-91% discount     │
  │  Per second  │  │  off         │  │  (1yr) or    │  │  Can be reclaimed    │
  │  Min 1 min   │  │  25%+ usage  │  │  70% (3yr)   │  │  30-sec warning      │
  │              │  │  per month   │  │  off          │  │                      │
  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────────┘
         │                 │                 │                    │
      No commitment    No commitment    1 or 3 year         No commitment
      Full flexibility  Automatic        Manual purchase    No availability SLA
```

---

## 6. Database Decision Tree

```
                                  START
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │  Is it ANALYTICAL     │
                         │  (OLAP) workload?     │
                         └──────────┬───────────┘
                            ┌───────┴───────┐
                            ▼               ▼
                           YES              NO (Transactional / OLTP)
                            │               │
                            ▼               ▼
                    ┌──────────────┐  ┌──────────────────────┐
                    │  BIGQUERY    │  │  Is it RELATIONAL    │
                    │  (Data       │  │  (needs SQL, ACID)?   │
                    │  Warehouse)  │  └──────────┬───────────┘
                    └──────────────┘      ┌──────┴──────┐
                                          ▼             ▼
                                         YES            NO (NoSQL)
                                          │             │
                                          ▼             ▼
                          ┌────────────────────┐ ┌──────────────────────┐
                          │  Need GLOBAL scale │ │  Time-series /       │
                          │  + strong          │ │  massive throughput? │
                          │  consistency?      │ │  (>1 TB, >10K QPS)   │
                          └─────────┬──────────┘ └──────────┬───────────┘
                           ┌───────┴───────┐       ┌───────┴───────┐
                           ▼               ▼       ▼               ▼
                          YES              NO     YES              NO
                           │               │       │               │
                           ▼               ▼       ▼               ▼
                   ┌──────────────┐        │  ┌──────────┐  ┌──────────────┐
                   │   CLOUD      │        │  │ BIGTABLE │  │  FIRESTORE   │
                   │   SPANNER    │        │  │          │  │  (Document   │
                   │ (global SQL) │        │  │ (wide-   │  │   NoSQL)     │
                   └──────────────┘        │  │  column) │  │              │
                                           │  └──────────┘  │  Native:     │
                                           ▼                │   mobile/web │
                               ┌──────────────────────┐     │  Datastore:  │
                               │  Need PostgreSQL +   │     │   server     │
                               │  high perf HTAP?     │     └──────────────┘
                               └──────────┬───────────┘
                                  ┌───────┴───────┐
                                  ▼               ▼
                                 YES              NO
                                  │               │
                                  ▼               ▼
                           ┌────────────┐  ┌────────────┐
                           │  ALLOYDB   │  │  CLOUD SQL │
                           │ (PG-compat │  │ (MySQL, PG │
                           │  HTAP)     │  │  SQL Srv)  │
                           └────────────┘  └────────────┘
```

---

## 7. Database Comparison Matrix

```
  ┌─────────────┬────────────┬────────────┬────────────┬────────────┬────────────┬────────────┐
  │             │ Cloud SQL  │  AlloyDB   │  Spanner   │  BigQuery  │ Firestore  │  Bigtable  │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Type        │ Relational │ Relational │ Relational │ Analytical │ Document   │ Wide-col.  │
  │             │            │            │            │ (DW)       │ (NoSQL)    │ (NoSQL)    │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Workload    │ OLTP       │ OLTP+OLAP  │ OLTP       │ OLAP       │ Mobile/Web │ Time-series│
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Scaling     │ Vertical   │ Vertical   │ Horizontal │ Serverless │ Serverless │ Horizontal │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Max Data    │ 64 TB      │ 128+ TB    │ Unlimited  │ Unlimited  │ 1 TB/db    │ Unlimited  │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ SQL         │ Full SQL   │ Full PG    │ Google SQL │ Std SQL    │ No         │ No         │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Consistency │ Strong     │ Strong     │ Strong     │ Eventual   │ Strong     │ Eventual   │
  │             │            │            │ (GLOBAL)   │            │            │            │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ SLA         │ 99.95-     │ 99.99%     │ 99.999%   │ 99.99%     │ 99.999%   │ 99.99%     │
  │             │ 99.99%     │            │ (5 nines)  │            │ (5 nines)  │            │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Cost        │ Low        │ Medium     │ HIGH       │ Free tier  │ Free tier  │ Medium     │
  ├─────────────┼────────────┼────────────┼────────────┼────────────┼────────────┼────────────┤
  │ Engines     │ MySQL      │ PostgreSQL │ Proprietary│ Proprietary│ Proprietary│ HBase API  │
  │             │ PostgreSQL │            │            │            │            │            │
  │             │ SQL Server │            │            │            │            │            │
  └─────────────┴────────────┴────────────┴────────────┴────────────┴────────────┴────────────┘
```

---

## 8. Cloud Storage Class Spectrum

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                   CLOUD STORAGE CLASS SPECTRUM                               │
  └──────────────────────────────────────────────────────────────────────────────┘

  HOT DATA ◄─────────────────────────────────────────────────────► COLD DATA
  (frequent access)                                          (rare access)

  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
  │   STANDARD     │  │   NEARLINE     │  │   COLDLINE     │  │   ARCHIVE      │
  ├────────────────┤  ├────────────────┤  ├────────────────┤  ├────────────────┤
  │ Min duration:  │  │ Min duration:  │  │ Min duration:  │  │ Min duration:  │
  │   NONE         │  │   30 days      │  │   90 days      │  │   365 days     │
  ├────────────────┤  ├────────────────┤  ├────────────────┤  ├────────────────┤
  │ Access freq:   │  │ Access freq:   │  │ Access freq:   │  │ Access freq:   │
  │   Anytime      │  │   < 1x/month   │  │   < 1x/quarter │  │   < 1x/year    │
  ├────────────────┤  ├────────────────┤  ├────────────────┤  ├────────────────┤
  │ Retrieval:     │  │ Retrieval:     │  │ Retrieval:     │  │ Retrieval:     │
  │   FREE         │  │   Per GB       │  │   Per GB (more)│  │   Per GB (most)│
  ├────────────────┤  ├────────────────┤  ├────────────────┤  ├────────────────┤
  │ Storage cost:  │  │ Storage cost:  │  │ Storage cost:  │  │ Storage cost:  │
  │   HIGHEST      │  │   Lower        │  │   Even lower   │  │   LOWEST       │
  └────────────────┘  └────────────────┘  └────────────────┘  └────────────────┘

  ACCESS LATENCY:   ALL CLASSES = MILLISECONDS (unlike AWS Glacier!)


  COST BARS:

  Storage cost   ████████████████  ███████████░░░░░  ████████░░░░░░░░  █████░░░░░░░░░░░
                    Standard          Nearline          Coldline          Archive

  Retrieval cost ░░░░░░░░░░░░░░░░  ███░░░░░░░░░░░░░  ██████░░░░░░░░░░  █████████░░░░░░░
                    Standard          Nearline          Coldline          Archive

  INVERSE RELATIONSHIP:  Higher storage cost = Lower retrieval cost  (and vice versa)
```

### Storage Class Use Cases

```
                                     ┌── Website assets, streaming media
                         ┌── Standard │   Mobile app data, gaming content
                         │            └── CDN origin
                         │
                         │            ┌── Monthly backups
  Cloud Storage ─────────┼── Nearline │   Long-tail multimedia
  Use Cases              │            └── Monthly analytics data
                         │
                         │            ┌── Disaster recovery
                         ├── Coldline │   Quarterly accessed data
                         │            └── Compliance data (short-term)
                         │
                         │            ┌── Regulatory compliance archives
                         └── Archive  │   Long-term backups (years)
                                      └── Digital preservation
```

### Object Lifecycle Management

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  AUTOMATIC TRANSITIONS (via Lifecycle Rules)                             │
  └──────────────────────────────────────────────────────────────────────────┘

   Day 0              Day 30            Day 90             Day 365
     │                  │                 │                  │
     ▼                  ▼                 ▼                  ▼
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐
  │ Standard │───►│ Nearline │───►│ Coldline │───►│   Archive    │
  └──────────┘    └──────────┘    └──────────┘    └──────────────┘
   (hot data)    (less accessed)  (rarely accessed) (almost never)

  Set lifecycle rules on the bucket to automate these transitions.
  Can also auto-DELETE objects after a certain age.
```

---

## 9. Block and File Storage Options

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                     STORAGE TYPE OVERVIEW                                    │
  └──────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
  │  Object Storage  │  Block Storage   │  Block Storage   │  File Storage    │
  │  (Cloud Storage) │  (Persistent     │  (Local SSD)     │  (Filestore)     │
  │                  │   Disk)          │                  │                  │
  ├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
  │  Unstructured    │  Attached to VMs │  Attached to     │  NFS shared      │
  │  data (files,    │  Durable         │  host machine    │  file system     │
  │  images, backups)│  Survives VM     │  EPHEMERAL       │  Multiple VMs    │
  │                  │  delete          │  Data lost on    │  can access      │
  │                  │                  │  VM stop/delete  │                  │
  └──────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

### Persistent Disk Types

```
  ┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
  │              │ pd-standard  │ pd-balanced  │   pd-ssd     │  pd-extreme  │
  ├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
  │ Type         │ HDD          │ SSD          │ SSD          │ SSD          │
  ├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
  │ IOPS         │ Low          │ Medium       │ High         │ Highest      │
  ├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
  │ Throughput   │ Low          │ Medium       │ High         │ Highest      │
  ├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
  │ Cost         │ Cheapest     │ Moderate     │ Higher       │ Highest      │
  ├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
  │ Use case     │ Bulk storage │ General      │ Databases    │ Mission-     │
  │              │ Sequential   │ purpose      │ Random I/O   │ critical DBs │
  │              │ I/O          │ (DEFAULT)    │              │              │
  └──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘

  ZONAL vs REGIONAL PD:

  ┌──────────────────────────────────────────────────────────────────────────┐
  │                                                                          │
  │   ZONAL PD                          REGIONAL PD                          │
  │                                                                          │
  │   ┌──────────────┐                  ┌──────────────┐ ┌──────────────┐   │
  │   │   Zone A     │                  │   Zone A     │ │   Zone B     │   │
  │   │  ┌────────┐  │                  │  ┌────────┐  │ │  ┌────────┐  │   │
  │   │  │  Disk  │  │                  │  │  Copy  │◄─┼─┼─►│  Copy  │  │   │
  │   │  └────────┘  │                  │  └────────┘  │ │  └────────┘  │   │
  │   └──────────────┘                  └──────────────┘ └──────────────┘   │
  │                                        synchronous replication           │
  │   Single zone                       Two zones, same region              │
  │   Standard cost                     ~2x cost                            │
  │   No HA                             HA (faster failover)                │
  │                                                                          │
  └──────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Load Balancer Selection Flowchart

```
                              START
                                │
                                ▼
                    ┌───────────────────────┐
                    │  Is the traffic       │
                    │  INTERNAL or EXTERNAL?│
                    └───────────┬───────────┘
                       ┌────────┴────────┐
                       ▼                 ▼
                   INTERNAL          EXTERNAL
                       │                 │
                       ▼                 ▼
             ┌──────────────────┐ ┌──────────────────┐
             │ Is it HTTP/HTTPS │ │ Is it HTTP/HTTPS  │
             │ traffic?         │ │ traffic?          │
             └────────┬─────────┘ └────────┬──────────┘
              ┌───────┴───────┐    ┌───────┴───────┐
              ▼               ▼    ▼               ▼
             YES              NO  YES              NO
              │               │    │               │
              ▼               ▼    ▼               ▼
  ┌───────────────────┐  ┌───────────────┐  ┌───────────────┐  ┌──────────────────────┐
  │   INTERNAL        │  │  INTERNAL     │  │  EXTERNAL     │  │  Need SSL            │
  │   APPLICATION     │  │  PASSTHROUGH  │  │  APPLICATION  │  │  termination?         │
  │   LB              │  │  NETWORK LB   │  │  LB           │  └──────────┬───────────┘
  │                   │  │               │  │               │      ┌──────┴──────┐
  │  Regional, L7     │  │  Regional, L4 │  │  GLOBAL, L7   │      ▼             ▼
  │  HTTP/HTTPS       │  │  TCP/UDP      │  │  HTTP/HTTPS   │     YES            NO
  │  Microservices    │  │  Databases    │  │  Web apps     │      │             │
  │  Service mesh     │  │  Internal svc │  │  APIs         │      ▼             ▼
  │                   │  │               │  │  CDN, Armor   │ ┌──────────┐ ┌──────────────┐
  └───────────────────┘  └───────────────┘  └───────────────┘ │ EXTERNAL │ │ EXTERNAL     │
                                                              │ PROXY    │ │ PASSTHROUGH  │
                                                              │ NETWORK  │ │ NETWORK LB   │
                                                              │ LB       │ │              │
                                                              │          │ │ Regional, L4 │
                                                              │ Global   │ │ TCP/UDP      │
                                                              │ L4       │ │ Gaming, IoT  │
                                                              │ TCP/SSL  │ │ Source IP    │
                                                              │          │ │ preserved    │
                                                              └──────────┘ └──────────────┘
```

### Load Balancer Comparison Matrix

```
  ┌──────────────────────┬──────────┬───────┬───────────┬─────────────────────────────┐
  │  Load Balancer       │  Scope   │ Layer │ Traffic   │  Key Use Case               │
  ├──────────────────────┼──────────┼───────┼───────────┼─────────────────────────────┤
  │ Ext. Application LB  │ GLOBAL   │  L7   │ HTTP/S    │ Web apps, APIs, CDN, Armor  │
  ├──────────────────────┼──────────┼───────┼───────────┼─────────────────────────────┤
  │ Int. Application LB  │ Regional │  L7   │ HTTP/S    │ Internal microservices      │
  ├──────────────────────┼──────────┼───────┼───────────┼─────────────────────────────┤
  │ Ext. Proxy Network   │ GLOBAL   │  L4   │ TCP/SSL   │ SSL termination, non-HTTP   │
  ├──────────────────────┼──────────┼───────┼───────────┼─────────────────────────────┤
  │ Int. Proxy Network   │ Regional │  L4   │ TCP       │ Internal TCP services       │
  ├──────────────────────┼──────────┼───────┼───────────┼─────────────────────────────┤
  │ Ext. Passthrough Net │ Regional │  L4   │ TCP/UDP   │ Gaming, IoT, preserves IP   │
  ├──────────────────────┼──────────┼───────┼───────────┼─────────────────────────────┤
  │ Int. Passthrough Net │ Regional │  L4   │ TCP/UDP   │ HA databases, internal svc  │
  └──────────────────────┴──────────┴───────┴───────────┴─────────────────────────────┘

  EXAM TIP: "Global web app" = External Application LB (almost always the answer)
  EXAM TIP: "Internal microservices over HTTP" = Internal Application LB
  EXAM TIP: "UDP traffic" = Passthrough Network LB (only one that supports UDP)
```

### External Application LB Architecture

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │                EXTERNAL APPLICATION LOAD BALANCER                       │
  └─────────────────────────────────────────────────────────────────────────┘

                        ┌──────────┐
                        │  Users   │
                        │ (global) │
                        └────┬─────┘
                             │
                             ▼
                   ┌──────────────────┐
                   │  Global Anycast  │
                   │  IP Address      │
                   └────────┬─────────┘
                            │
                            ▼
                   ┌──────────────────┐
                   │  Google Edge PoP │  ◄── Cloud CDN cache check
                   │  (nearest to     │  ◄── Cloud Armor (WAF/DDoS)
                   │   user)          │  ◄── SSL Termination
                   └────────┬─────────┘
                            │
                            ▼
                   ┌──────────────────┐      ┌─────────────────────┐
                   │     URL MAP      │      │  Routing Rules:     │
                   │  (routing logic) │─────►│  /api/*  ──► API BE │
                   │                  │      │  /img/*  ──► CDN    │
                   └────────┬─────────┘      │  /*      ──► Web BE│
                            │                └─────────────────────┘
                            ▼
                   ┌──────────────────┐
                   │ Backend Service  │ ◄── Health Checks
                   │                  │ ◄── Session Affinity
                   └────────┬─────────┘ ◄── Autoscaling
                            │
                 ┌──────────┼──────────┐
                 ▼          ▼          ▼
          ┌──────────┐┌──────────┐┌──────────┐
          │ Instance ││ Instance ││ Cloud    │
          │ Group    ││ Group    ││ Storage  │
          │ (US)     ││ (EU)     ││ Bucket   │
          └──────────┘└──────────┘└──────────┘
```

---

## 11. Network Tier Comparison

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                      NETWORK SERVICE TIERS                                   │
  └──────────────────────────────────────────────────────────────────────────────┘


  PREMIUM TIER (Default)
  ──────────────────────

  ┌──────┐     ┌─────────────┐     ┌──────────────────────────────┐    ┌─────────┐
  │ User │────►│ Nearest     │────►│  Google Private Backbone     │───►│  GCP    │
  │      │     │ Google Edge │     │  (high-speed, low-latency)   │    │ Region  │
  └──────┘     │ PoP         │     │  ════════════════════════    │    └─────────┘
               └─────────────┘     └──────────────────────────────┘
                 enters Google        travels on Google's private
                 network EARLY        fiber network


  STANDARD TIER
  ─────────────

  ┌──────┐     ┌──────────────────────────────┐     ┌─────────────┐    ┌─────────┐
  │ User │────►│  Public Internet              │────►│ Google      │───►│  GCP    │
  │      │     │  (ISP networks, variable      │     │ Network     │    │ Region  │
  └──────┘     │   quality, higher latency)    │     │ Entry       │    └─────────┘
               └──────────────────────────────┘     │ (near       │
                 travels on public internet          │  region)     │
                 for MOST of the journey             └─────────────┘
                                                      enters Google
                                                      network LATE


  COMPARISON TABLE:

  ┌────────────────────┬──────────────────────┬──────────────────────┐
  │  Feature           │    PREMIUM TIER      │    STANDARD TIER     │
  ├────────────────────┼──────────────────────┼──────────────────────┤
  │  Routing           │  Google backbone     │  Public internet     │
  │  Latency           │  LOWER               │  Higher              │
  │  Reliability       │  HIGHER              │  Variable (ISP)      │
  │  Global LB         │  YES                 │  NO (regional only)  │
  │  SLA               │  Google network SLA  │  ISP-dependent       │
  │  Cost              │  Higher              │  LOWER               │
  │  Default           │  YES                 │  No                  │
  │  Best for          │  Production          │  Dev/test            │
  │                    │  Latency-sensitive    │  Cost-sensitive      │
  └────────────────────┴──────────────────────┴──────────────────────┘


  DECISION:

      Is your workload latency-sensitive or production-critical?
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                    YES                NO
                     │                 │
                     ▼                 ▼
           ┌──────────────┐   Need global load balancing?
           │ PREMIUM TIER │            │
           └──────────────┘   ┌────────┴────────┐
                              ▼                 ▼
                             YES                NO
                              │                 │
                              ▼                 ▼
                    ┌──────────────┐   ┌───────────────┐
                    │ PREMIUM TIER │   │ STANDARD TIER │
                    └──────────────┘   │ (save money)  │
                                       └───────────────┘
```

---

## 12. VPC Architecture

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                         VPC (Virtual Private Cloud)                          │
  │                              GLOBAL Resource                                │
  ├──────────────────────────────────────────────────────────────────────────────┤
  │                                                                              │
  │   ┌──────────────────────────┐    ┌──────────────────────────┐              │
  │   │  Subnet: us-central1     │    │  Subnet: europe-west1    │              │
  │   │  10.128.0.0/20           │    │  10.132.0.0/20           │              │
  │   │  (REGIONAL)              │    │  (REGIONAL)              │              │
  │   │                          │    │                          │              │
  │   │  ┌────┐ ┌────┐ ┌────┐   │    │  ┌────┐ ┌────┐          │              │
  │   │  │VM-a│ │VM-b│ │VM-c│   │    │  │VM-d│ │VM-e│          │              │
  │   │  └────┘ └────┘ └────┘   │    │  └────┘ └────┘          │              │
  │   │  zone-a  zone-b zone-c  │    │  zone-b  zone-c         │              │
  │   └──────────────────────────┘    └──────────────────────────┘              │
  │                                                                              │
  │   KEY:  VPC = Global  |  Subnet = Regional  |  VM = Zonal                   │
  │                                                                              │
  │   MODES:  Auto (subnets in all regions auto)  |  Custom (you create subnets)│
  │                                                                              │
  │   Every project gets a DEFAULT VPC (auto mode)                               │
  │                                                                              │
  └──────────────────────────────────────────────────────────────────────────────┘
```

### Resource Scope Reference

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │  RESOURCE SCOPE                                                              │
  ├──────────────┬──────────────────────────────────────────────────────────────┤
  │              │                                                              │
  │   GLOBAL     │  VPCs, Firewall rules, Routes, External IPs, Images,        │
  │              │  Snapshots, Global LBs, Cloud CDN                            │
  │              │                                                              │
  ├──────────────┼──────────────────────────────────────────────────────────────┤
  │              │                                                              │
  │   REGIONAL   │  Subnets, Regional PDs, Regional instance groups,           │
  │              │  Cloud Router, Cloud NAT, Internal IPs                       │
  │              │                                                              │
  ├──────────────┼──────────────────────────────────────────────────────────────┤
  │              │                                                              │
  │   ZONAL      │  VM instances, Zonal PDs, GKE node pools, Local SSDs        │
  │              │                                                              │
  └──────────────┴──────────────────────────────────────────────────────────────┘
```

---

## 13. High Availability Patterns

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                     HIGH AVAILABILITY PATTERNS                               │
  └──────────────────────────────────────────────────────────────────────────────┘


  SINGLE ZONE (NO HA)                MULTI-ZONE (HA within region)
  ────────────────────               ──────────────────────────────

  ┌────────────────────┐             ┌────────────────────┐  ┌────────────────────┐
  │     Zone A         │             │     Zone A         │  │     Zone B         │
  │  ┌────┐  ┌────┐    │             │  ┌────┐  ┌────┐    │  │  ┌────┐  ┌────┐    │
  │  │ VM │  │ VM │    │             │  │ VM │  │ VM │    │  │  │ VM │  │ VM │    │
  │  └────┘  └────┘    │             │  └────┘  └────┘    │  │  └────┘  └────┘    │
  └────────────────────┘             └────────────────────┘  └────────────────────┘
  Single point of failure!           Zone A fails? Zone B handles traffic.


  MULTI-REGION (HA across regions)
  ────────────────────────────────

  ┌──────────────────────────┐       ┌──────────────────────────┐
  │   Region: US-Central     │       │   Region: EU-West        │
  │  ┌────┐  ┌────┐  ┌────┐  │       │  ┌────┐  ┌────┐  ┌────┐  │
  │  │ VM │  │ VM │  │ VM │  │       │  │ VM │  │ VM │  │ VM │  │
  │  └────┘  └────┘  └────┘  │       │  └────┘  └────┘  └────┘  │
  └────────────┬─────────────┘       └────────────┬─────────────┘
               │                                  │
               └──────────┬───────────────────────┘
                          │
                 ┌────────▼────────┐
                 │  Global LB      │
                 │  (routes to     │
                 │   nearest       │
                 │   healthy       │
                 │   region)       │
                 └─────────────────┘
```

---

## 14. Connectivity to On-Premises

```
  ┌───────────────────────────────────────────────────────────────────────────┐
  │                 ON-PREMISES TO GCP CONNECTIVITY                          │
  └───────────────────────────────────────────────────────────────────────────┘

  BANDWIDTH    ◄── Low ─────────────────────────────────────────── High ──►

  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
  │  Classic VPN  │  │  HA VPN      │  │  Partner     │  │  Dedicated       │
  │               │  │              │  │  Interconnect│  │  Interconnect    │
  ├──────────────┤  ├──────────────┤  ├──────────────┤  ├──────────────────┤
  │  1.4 Gbps    │  │  1.4-3 Gbps  │  │  50 Mbps -   │  │  10-200 Gbps     │
  │  per tunnel  │  │  per tunnel  │  │  50 Gbps     │  │                  │
  ├──────────────┤  ├──────────────┤  ├──────────────┤  ├──────────────────┤
  │  99.9% SLA   │  │  99.99% SLA  │  │  99.9-99.99% │  │  99.9-99.99%     │
  ├──────────────┤  ├──────────────┤  ├──────────────┤  ├──────────────────┤
  │  Over public │  │  Over public │  │  Via service │  │  Direct physical │
  │  internet    │  │  internet    │  │  provider    │  │  connection      │
  ├──────────────┤  ├──────────────┤  ├──────────────┤  ├──────────────────┤
  │  Encrypted   │  │  Encrypted   │  │  Encrypted   │  │  NOT encrypted   │
  │  (IPsec)     │  │  (IPsec)     │  │  optional    │  │  (add VPN on top)│
  ├──────────────┤  ├──────────────┤  ├──────────────┤  ├──────────────────┤
  │  Legacy      │  │  Recommended │  │  Medium      │  │  Enterprise      │
  │  Simple      │  │  Most common │  │  bandwidth   │  │  Lowest latency  │
  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘
```

---

## 15. Cloud NAT and Cloud DNS Mind Maps

```
                                 ┌── Allows VMs without external IPs
                                 │   to access the internet
                                 │
                                 ├── OUTBOUND only
  Cloud NAT ─────────────────────┤   (cannot initiate inbound)
                                 │
                                 ├── No proxy VMs needed
                                 │   (software-defined)
                                 │
                                 ├── Regional resource
                                 │
                                 └── Use case: download packages,
                                     call external APIs securely


                                 ┌── Public zones (internet-facing)
                                 │
                                 ├── Private zones (VPC-internal)
                                 │
  Cloud DNS ─────────────────────┼── DNS forwarding (to on-prem)
                                 │
                                 ├── DNS peering (between VPCs)
                                 │
                                 ├── DNSSEC support
                                 │
                                 └── 100% SLA
```

---

## 16. Region Selection Decision Process

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │               CHOOSING A REGION -- DECISION FACTORS                      │
  └──────────────────────────────────────────────────────────────────────────┘

                        ┌───────────────────┐
                        │ Where are your    │
                        │ USERS located?    │
                        └────────┬──────────┘
                                 │
                                 ▼
                        ┌───────────────────┐
                        │ Any DATA RESIDENCY│──── YES ──► Must use compliant regions
                        │ requirements?     │              (e.g., EU data stays in EU)
                        └────────┬──────────┘
                                 │ NO
                                 ▼
                        ┌───────────────────┐
                        │ Is the SERVICE    │──── NO ──► Choose different region
                        │ available in      │
                        │ preferred region? │
                        └────────┬──────────┘
                                 │ YES
                                 ▼
                        ┌───────────────────┐
                        │ Is COST a major   │──── YES ──► Consider us-central1
                        │ factor?           │              (typically cheapest)
                        └────────┬──────────┘
                                 │ NO
                                 ▼
                        ┌───────────────────┐
                        │ Need DR in        │──── YES ──► Pick geographically
                        │ different geo?    │              distant 2nd region
                        └────────┬──────────┘
                                 │
                                 ▼
                        ┌───────────────────┐
                        │ Deploy to nearest │
                        │ region with all   │
                        │ needed services   │
                        └───────────────────┘
```

---

## 17. Complete Section 2 Exam Cheat Sheet

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │  SECTION 2: QUICK REFERENCE                                                  │
  ├──────────────────────────────────────────────────────────────────────────────┤
  │                                                                              │
  │  COMPUTE:                                                                    │
  │    Full control / GPU / Windows ─────────────────── Compute Engine           │
  │    Kubernetes / orchestration / multi-cloud ──────── GKE                     │
  │    Container, no K8s, stateless ─────────────────── Cloud Run               │
  │    Event-driven, lightweight ────────────────────── Cloud Functions          │
  │    Fault-tolerant batch + save money ────────────── Spot VMs                 │
  │    Exact CPU/RAM needs ──────────────────────────── Custom Machine Types     │
  │                                                                              │
  │  DATABASES:                                                                  │
  │    SQL, <64TB, standard ─────────────────────────── Cloud SQL                │
  │    PostgreSQL, HTAP, high perf ──────────────────── AlloyDB                  │
  │    SQL, global, unlimited scale ─────────────────── Spanner                  │
  │    Analytics, petabyte SQL ──────────────────────── BigQuery                 │
  │    Mobile/web, real-time, documents ─────────────── Firestore                │
  │    Time-series, IoT, >1TB, low latency ──────────── Bigtable                │
  │                                                                              │
  │  STORAGE:                                                                    │
  │    Hot data ─────────────────────────────────────── Standard                 │
  │    <1x/month access ─────────────────────────────── Nearline (30d min)       │
  │    <1x/quarter access ───────────────────────────── Coldline (90d min)       │
  │    <1x/year access ──────────────────────────────── Archive  (365d min)      │
  │    ALL classes have MILLISECOND access latency!                              │
  │                                                                              │
  │  NETWORKING:                                                                 │
  │    Global web app LB ────────────────────────────── Ext. Application LB      │
  │    Internal HTTP services ───────────────────────── Int. Application LB      │
  │    UDP / non-HTTP external ──────────────────────── Ext. Passthrough Net LB  │
  │    Low latency / production ─────────────────────── Premium Tier             │
  │    Cost-sensitive / dev ─────────────────────────── Standard Tier            │
  │    VPC = Global  |  Subnet = Regional  |  VM = Zonal                        │
  │                                                                              │
  └──────────────────────────────────────────────────────────────────────────────┘
```
