# Section 2.2 — Planning and Configuring Data Storage Options

## Exam Relevance
This topic is part of **Section 2: Planning and implementing a cloud solution (~30 % of the exam)**. You must know how to choose the right database or storage product for a given scenario, understand storage class differences, and plan for multi-region redundancy.

---

## 1. Database and Storage Decision Overview

> 📖 **Docs:** [Storage options overview](https://cloud.google.com/docs/choosing-a-storage-option) | [Database products](https://cloud.google.com/products/databases) | 🖥️ **Console:** n/a (planning reference)

Google Cloud offers multiple storage and database services, each optimized for specific use cases:

```
                    Relational                    Non-Relational
                    ──────────                    ──────────────
Transactional       Cloud SQL                     Firestore
                    AlloyDB                       Bigtable
                    Spanner                       Memorystore (Redis/Memcached)

Analytical          BigQuery                      BigQuery
                    AlloyDB (OLAP)

Streaming/Messaging Pub/Sub                       Managed Apache Kafka

Object Storage      Cloud Storage (Standard, Nearline, Coldline, Archive)
Block Storage       Persistent Disk (Zonal, Regional) | Hyperdisk
File/NFS Storage    Filestore | Google Cloud NetApp Volumes
```

---

## 2. Cloud SQL

> 📖 **Docs:** [Cloud SQL overview](https://cloud.google.com/sql/docs/overview) | [Choosing a Cloud SQL edition](https://cloud.google.com/sql/docs/mysql/editions-intro) | 🖥️ **Console:** SQL → Create instance

### What Is Cloud SQL?
- **Fully managed relational database** service
- Supports **MySQL**, **PostgreSQL**, and **SQL Server**
- Handles backups, replication, patches, and failover automatically

### Key Features
- Up to **128 vCPUs** and **864 GB RAM**
- Up to **64 TB storage**
- Automatic storage increase
- High availability with **regional failover** (synchronous replication)
- Read replicas (within region, cross-region, and external)
- Point-in-time recovery (PITR)
- Private IP and public IP connectivity
- Cloud SQL Auth Proxy for secure connections

### When to Choose Cloud SQL
- Traditional relational workloads (web apps, CMS, ERP)
- Existing MySQL/PostgreSQL/SQL Server applications
- Need for ACID transactions
- Data size up to ~64 TB
- Moderate throughput requirements

### When NOT to Choose Cloud SQL
- Need horizontal scaling beyond a single region
- Data exceeds 64 TB
- Need global distribution with strong consistency
- Need millisecond-latency at massive scale
- Analytical workloads requiring petabyte-scale processing

### Pricing
- Per vCPU, memory, and storage (per second, 1-minute minimum)
- Network egress charges
- HA instances cost ~2x (two instances running)

---

## 3. AlloyDB

> 📖 **Docs:** [AlloyDB overview](https://cloud.google.com/alloydb/docs/overview) | [AlloyDB vs Cloud SQL](https://cloud.google.com/alloydb/docs/overview#alloydb_vs_cloud_sql_for_postgresql) | 🖥️ **Console:** AlloyDB → Create cluster

### What Is AlloyDB?
- **Fully managed, PostgreSQL-compatible** database
- Combines the best of open-source PostgreSQL with Google's infrastructure
- Up to **4x faster for transactional** and **100x faster for analytical** queries than standard PostgreSQL

### Key Features
- PostgreSQL 14+ compatible
- Columnar engine for analytical queries
- ML-driven adaptive caching and indexing
- 99.99% SLA with regional availability
- Automated backups and PITR

### When to Choose AlloyDB
- High-performance PostgreSQL workloads
- Mixed transactional and analytical (HTAP) workloads
- Migrating from Oracle or other enterprise databases
- Need PostgreSQL compatibility with better performance than Cloud SQL

---

## 4. Cloud Spanner

> 📖 **Docs:** [Cloud Spanner overview](https://cloud.google.com/spanner/docs/overview) | [When to use Spanner](https://cloud.google.com/spanner/docs/whitepapers/life-of-cloud-spanner-reads-and-writes) | 🖥️ **Console:** Spanner → Create instance

### What Is Cloud Spanner?
- **Globally distributed, strongly consistent** relational database
- Combines relational structure with horizontal scalability
- **Unlimited scale** with 99.999% SLA (five nines)

### Key Features
- SQL interface with relational schemas
- ACID transactions across regions and continents
- Horizontal scaling (add nodes for more throughput)
- Automatic sharding
- Multi-region and single-region configurations
- Change streams for real-time data processing

### When to Choose Cloud Spanner
- **Global applications** needing strong consistency (financial systems, inventory management)
- Relational data that **exceeds single-node capacity** (>64 TB or high throughput)
- Applications requiring **99.999% availability**
- Gaming leaderboards, ad-tech, retail supply chain

### When NOT to Choose Cloud Spanner
- Small-scale applications (Cloud SQL is more cost-effective)
- Non-relational data
- Budget-constrained projects (Spanner starts at ~$0.90/node/hour)
- Workloads that can tolerate eventual consistency

### Pricing
- Per node-hour (minimum 1 node)
- Per GB of storage
- Multi-region configurations cost more than single-region

---

## 5. BigQuery

> 📖 **Docs:** [BigQuery overview](https://cloud.google.com/bigquery/docs/introduction) | [BigQuery pricing](https://cloud.google.com/bigquery/pricing) | 🖥️ **Console:** BigQuery → Explorer

### What Is BigQuery?
- **Serverless, highly scalable data warehouse**
- Designed for **analytical workloads** (OLAP)
- Processes petabytes of data using SQL
- Separate storage and compute (pay independently)

### Key Features
- Serverless — no infrastructure management
- Standard SQL support
- Real-time analytics with streaming inserts
- ML built-in (BigQuery ML)
- Geospatial analysis (BigQuery GIS)
- BI Engine for fast dashboards
- Automatic data encryption
- Column-oriented storage (Capacitor format)
- Slots-based pricing for predictable costs (optional)

### When to Choose BigQuery
- **Data warehousing** and business intelligence
- **Ad-hoc SQL queries** on large datasets
- **Log analytics** and clickstream analysis
- **Machine learning** with SQL (BigQuery ML)
- **Real-time dashboards** and reporting
- Processing data from Cloud Storage, Cloud SQL, Sheets, etc.

### When NOT to Choose BigQuery
- Transactional (OLTP) workloads (use Cloud SQL or Spanner)
- Low-latency point lookups (use Bigtable or Firestore)
- Streaming with sub-second requirements (use Bigtable)
- Small datasets that don't benefit from distributed processing

### Pricing Models
- **On-demand**: $6.25 per TB processed (first 1 TB/month free)
- **Capacity (Editions)**: Purchase slots for predictable costs
- **Storage**: $0.02/GB/month (active), $0.01/GB/month (long-term, 90+ days untouched)
- **Streaming inserts**: $0.05 per GB

---

## 6. Firestore (Datastore Mode and Native Mode)

> 📖 **Docs:** [Firestore overview](https://cloud.google.com/firestore/docs/overview) | [Native vs Datastore mode](https://cloud.google.com/datastore/docs/firestore-or-datastore) | 🖥️ **Console:** Firestore → Create database

### What Is Firestore?
- **Serverless, NoSQL document database**
- Designed for mobile, web, and IoT applications
- Real-time sync and offline support (Native mode)
- Scalable to millions of concurrent connections

### Two Modes

| Feature | Native Mode | Datastore Mode |
|---------|-------------|----------------|
| Data model | Documents in collections | Entities in kinds |
| Real-time listeners | Yes | No |
| Offline support | Yes (mobile SDKs) | No |
| Transactions | Multi-document ACID | Entity group transactions |
| Best for | Mobile/web apps | Server-side applications |
| Querying | Collection queries, collection group queries | GQL, structured queries |

### When to Choose Firestore
- **Mobile and web applications** needing real-time sync
- **User profiles**, session data, game state
- **Product catalogs** with hierarchical data
- **IoT device data** collection
- Semi-structured or hierarchical data

### When NOT to Choose Firestore
- Analytical queries across millions of records (use BigQuery)
- Time-series data at massive scale (use Bigtable)
- Need for complex joins and relational queries (use Cloud SQL)
- Data exceeding 1 TB per database (consider Bigtable)

### Pricing
- Per document read, write, and delete
- Per GB stored
- Free tier: 50K reads, 20K writes, 20K deletes per day

---

## 7. Cloud Bigtable

> 📖 **Docs:** [Cloud Bigtable overview](https://cloud.google.com/bigtable/docs/overview) | [Bigtable vs HBase](https://cloud.google.com/bigtable/docs/hbase-bigtable) | 🖥️ **Console:** Bigtable → Create instance

### What Is Cloud Bigtable?
- **Fully managed, wide-column NoSQL database**
- Designed for **massive analytical and operational workloads**
- Same technology that powers Google Search, Maps, and Gmail

### Key Features
- Millisecond latency at any scale
- Handles billions of rows and thousands of columns
- Scales linearly by adding nodes
- Native HBase API compatibility
- Integration with Hadoop, Dataflow, and Dataproc
- Automatic replication across zones and regions
- SSD or HDD storage options

### When to Choose Cloud Bigtable
- **Time-series data** (IoT, financial ticks, monitoring metrics)
- **Marketing analytics** (user behavior, ad impressions)
- **Graph data** and relationships
- Workloads requiring **>1 TB of data** with low latency
- **MapReduce** or stream processing pipelines
- High-throughput **read/write** applications (>10,000 QPS)

### When NOT to Choose Cloud Bigtable
- Data less than 1 TB (not cost-effective at small scale)
- Need for complex SQL queries (use BigQuery or Cloud SQL)
- ACID transactions across multiple rows (use Spanner)
- Document storage (use Firestore)

### Pricing
- Per node-hour (minimum 1 node, recommended 3+ for production)
- Per GB of storage (SSD or HDD)
- Network egress

---

## 8. Database Comparison Matrix

> 📖 **Docs:** [Choose a database](https://cloud.google.com/docs/choosing-a-storage-option#choosing_a_storage_option_by_usage) | 🖥️ **Console:** n/a (planning reference)

| Feature | Cloud SQL | AlloyDB | Spanner | BigQuery | Firestore | Bigtable |
|---------|-----------|---------|---------|----------|-----------|----------|
| **Type** | Relational | Relational | Relational | Analytical | Document (NoSQL) | Wide-column (NoSQL) |
| **Workload** | OLTP | OLTP + OLAP | OLTP | OLAP | Mobile/Web | Time-series, IoT |
| **Scale** | Vertical | Vertical | Horizontal | Serverless | Serverless | Horizontal |
| **Max data** | 64 TB | 128+ TB | Unlimited | Unlimited | 1 TB/database | Unlimited |
| **Consistency** | Strong | Strong | Strong (global) | Eventual | Strong | Eventual |
| **SQL support** | Full SQL | Full SQL (PG) | Google SQL | Standard SQL | No (queries) | No |
| **SLA** | 99.95-99.99% | 99.99% | 99.999% | 99.99% | 99.999% | 99.99% |
| **Global** | Read replicas | Read replicas | Multi-region | Multi-region | Multi-region | Multi-region |
| **Min cost** | Low | Medium | High | Free tier | Free tier | Medium |

---

## 9. Cloud Storage Classes

> 📖 **Docs:** [Storage classes](https://cloud.google.com/storage/docs/storage-classes) | [Choosing storage class](https://cloud.google.com/storage/docs/access-frequency) | 🖥️ **Console:** Cloud Storage → Buckets → Create bucket → Choose storage class

Cloud Storage is Google's object storage service. Data is organized in **buckets** and stored as **objects** (files).

### Storage Classes

| Class | Min Storage Duration | Retrieval Cost | Use Case |
|-------|---------------------|----------------|----------|
| **Standard** | None | Free | Frequently accessed data ("hot data") |
| **Nearline** | 30 days | Per GB | Data accessed less than once per month |
| **Coldline** | 90 days | Per GB (higher) | Data accessed less than once per quarter |
| **Archive** | 365 days | Per GB (highest) | Data accessed less than once per year |

### Key Pricing Concepts
- **Storage cost decreases** as you move from Standard → Archive
- **Retrieval cost increases** as you move from Standard → Archive
- **Early deletion charges**: If you delete data before the minimum storage duration, you're charged for the remaining time
- All classes have the **same millisecond access latency** (unlike AWS Glacier)

### Storage Class Comparison

```
                 Storage Cost    Retrieval Cost    Min Duration
Standard    ████████████████    ░░░░░░░░░░░░░░    None
Nearline    ██████████████░░    ██░░░░░░░░░░░░    30 days
Coldline    ████████████░░░░    ████░░░░░░░░░░    90 days
Archive     ██████████░░░░░░    ██████░░░░░░░░    365 days
```

### Bucket-Level vs. Object-Level Storage Class
- **Default storage class** is set at the bucket level
- Individual objects can have **different storage classes** within the same bucket
- Use **Object Lifecycle Management** to automatically transition objects between classes

### Common Use Cases

| Storage Class | Example Use Cases |
|--------------|-------------------|
| Standard | Website assets, streaming media, mobile app data, gaming content |
| Nearline | Monthly backups, long-tail multimedia, data for monthly analytics |
| Coldline | Disaster recovery, quarterly accessed data |
| Archive | Regulatory compliance archives, long-term backups, digital preservation |

### Cloud Storage Location Types

| Location Type | Description | Example |
|--------------|-------------|---------|
| **Region** | Data stored in a single region | `us-central1` |
| **Dual-region** | Data replicated across two specific regions | `nam4` (Iowa + S. Carolina) |
| **Multi-region** | Data replicated across a large geographic area | `US`, `EU`, `ASIA` |

- Dual-region and multi-region provide higher availability (99.95% SLA) at higher cost than region (99.9% SLA).
- Multi-region is ideal for serving users globally; region is cheapest and offers lowest cross-resource latency.

### Object Versioning
- When enabled on a bucket, previous (noncurrent) versions of objects are retained when objects are overwritten or deleted.
- Protects against accidental deletion or overwrite.
- Enabled via `gcloud storage buckets update gs://BUCKET --versioning`.
- Versioned objects count toward storage costs; use lifecycle rules to delete old versions.

### Object Lifecycle Management
Rules automate storage class transitions or deletions based on object conditions (age, versioning, storage class, etc.).

```bash
# Example lifecycle policy (lifecycle.json): delete objects older than 365 days
# and move objects to Coldline after 90 days
gcloud storage buckets update gs://my-bucket --lifecycle-file=lifecycle.json
```

Common lifecycle actions:
- **SetStorageClass** — Transition to a colder class (e.g., Standard → Nearline after 30 days)
- **Delete** — Remove objects after a certain age
- **AbortIncompleteMultipartUpload** — Clean up failed uploads

### Retention Policies and Object Holds
- **Retention Policy**: Enforces a minimum retention period at the bucket level; objects cannot be deleted or overwritten until they reach the configured age.
- **Bucket Lock**: Makes a retention policy permanent (cannot be reduced or removed). Required for regulatory compliance (e.g., WORM).
- **Object Hold** (temporary or event-based): Prevents a specific object from being deleted or overwritten regardless of retention policy.

### Autoclass
- A bucket-level feature that automatically transitions objects to the most cost-effective storage class based on access patterns.
- Eliminates the need to write custom lifecycle rules.
- Adds a small per-object management fee but removes early-deletion charges when objects move to colder classes.

### Signed URLs and Signed Cookies
- **Signed URL**: A time-limited URL that grants temporary, authenticated access to a Cloud Storage object without requiring the recipient to have a Google account.
- **Signed Cookie**: Similar concept, used to grant access to a set of URLs under a common prefix.
- Use case: serving private content to end users (e.g., premium downloads, short-lived video URLs).

### Uniform Bucket-Level Access vs. Fine-Grained ACLs
- **Uniform Bucket-Level Access** (recommended): Disables per-object ACLs; all access is controlled via IAM at the bucket level. Simpler, consistent, and enforceable via org policy.
- **Fine-Grained ACLs**: Legacy per-object ACL system; allows mixing IAM and ACLs. Harder to audit.

### Data Transfer Options

| Service | Use Case | Data Size |
|---------|----------|-----------|
| **gcloud storage cp** / **gsutil cp** | Ad-hoc uploads and downloads | Small to medium |
| **Storage Transfer Service** | Scheduled, managed transfers from S3, Azure, on-premises, or other GCS buckets | Medium to large |
| **Transfer Appliance** | Physical device shipped to you for offline data transfer | Multi-TB to PB (slow networks) |
| **BigQuery Data Transfer Service** | Automated transfers of data from SaaS apps (Google Ads, YouTube) into BigQuery | Varies |

### Common gcloud/gsutil Commands

```bash
# Create a bucket
gcloud storage buckets create gs://my-bucket --location=us-central1 --default-storage-class=STANDARD --uniform-bucket-level-access

# Upload/download objects
gcloud storage cp ./local-file.txt gs://my-bucket/
gcloud storage cp gs://my-bucket/file.txt ./

# List buckets / objects
gcloud storage ls
gcloud storage ls gs://my-bucket/

# Change storage class of an object
gcloud storage objects update gs://my-bucket/file.txt --storage-class=NEARLINE

# Set IAM on a bucket
gcloud storage buckets add-iam-policy-binding gs://my-bucket \
  --member=user:alice@example.com --role=roles/storage.objectViewer

# Enable versioning
gcloud storage buckets update gs://my-bucket --versioning

# Apply a retention policy (10 years)
gcloud storage buckets update gs://my-bucket --retention-period=10y
```

---

## 10. Persistent Disk Options

> 📖 **Docs:** [Persistent Disk overview](https://cloud.google.com/compute/docs/disks) | [Hyperdisk overview](https://cloud.google.com/compute/docs/disks/hyperdisks) | 🖥️ **Console:** Compute Engine → Disks → Create disk

Persistent Disks are durable block storage for Compute Engine and GKE.

### Disk Types

| Type | Description | IOPS (Read/Write) | Throughput | Use Case |
|------|-------------|-------|------------|----------|
| **pd-standard** | Standard HDD | Low | Low | Bulk storage, sequential I/O |
| **pd-balanced** | Balanced SSD | Medium | Medium | General-purpose (default) |
| **pd-ssd** | Performance SSD | High | High | Databases, random I/O |
| **pd-extreme** | Highest performance SSD | Highest | Highest | Mission-critical databases |

### Zonal vs. Regional Persistent Disks

| Feature | Zonal PD | Regional PD |
|---------|----------|-------------|
| **Availability** | Single zone | Two zones in same region |
| **Replication** | Within zone | Synchronous across 2 zones |
| **Use case** | Standard workloads | HA workloads needing failover |
| **Cost** | Standard | ~2x zonal cost |
| **Failover** | Manual | Faster (data already in 2 zones) |

### Google Cloud Hyperdisk

**Hyperdisk** is Google Cloud's next-generation block storage that decouples IOPS and throughput from capacity, enabling independent scaling of performance and size.

| Type | Best For | Max IOPS | Max Throughput |
|------|----------|----------|---------------|
| **Hyperdisk Balanced** | General-purpose workloads, databases | 160,000 | 2,400 MB/s |
| **Hyperdisk Extreme** | Highest-performance OLTP databases | 350,000 | 5,000 MB/s |
| **Hyperdisk Throughput** | Streaming, MapReduce, Kafka | 1,200,000 | 2,400 MB/s |
| **Hyperdisk ML** | ML model serving, large data loading | N/A | Very high |

#### Hyperdisk vs. Persistent Disk

| Feature | Persistent Disk | Hyperdisk |
|---------|----------------|-----------|
| IOPS tied to capacity | Yes | No (provision independently) |
| Dynamic IOPS resize | No | Yes (online, no VM restart) |
| Throughput tied to capacity | Yes | No |
| Multi-writer support | Limited | Hyperdisk Balanced supports |
| Generation | Older | Current (recommended for new deployments) |

```bash
# Create a Hyperdisk Balanced disk
gcloud compute disks create my-hyperdisk \
  --type=hyperdisk-balanced \
  --size=500GB \
  --provisioned-iops=10000 \
  --provisioned-throughput=250 \
  --zone=us-central1-a

# Create a Hyperdisk Extreme disk
gcloud compute disks create my-extreme-disk \
  --type=hyperdisk-extreme \
  --size=1TB \
  --provisioned-iops=100000 \
  --zone=us-central1-a

# Resize IOPS without restarting VM
gcloud compute disks update my-hyperdisk \
  --provisioned-iops=20000 \
  --zone=us-central1-a
```

#### Key Exam Points
- Choose Hyperdisk when you need **high IOPS/throughput independent of disk size**
- Hyperdisk Balanced is the default recommendation for most new workloads
- Hyperdisk Extreme is for maximum IOPS (e.g., SAP HANA, high-throughput OLTP)
- Hyperdisk Throughput is cost-effective for sequential read/write (streaming, analytics)

### Local SSDs
- Physically attached to the host machine
- **Extremely high IOPS** (up to 2.4M read IOPS)
- **Ephemeral** — data lost when VM stops or is deleted
- Use cases: Caches, temp storage, scratch processing
- Fixed sizes: 375 GB per device (up to 24 devices)

---

## 11. Filestore

> 📖 **Docs:** [Filestore overview](https://cloud.google.com/filestore/docs/overview) | [NetApp Volumes overview](https://cloud.google.com/netapp/volumes/docs/discover/overview) | 🖥️ **Console:** Filestore → Instances → Create instance

### What Is Filestore?
- **Managed NFS file storage** service
- Provides shared file system accessible by multiple VMs
- Use cases: Media rendering, EDA, genome processing, web content

### Tiers
- **Basic HDD / Basic SSD** — Development and testing
- **High Scale SSD** — High-performance workloads
- **Enterprise** — Mission-critical, regional availability

---

## 11b. Google Cloud NetApp Volumes

### What Is Google Cloud NetApp Volumes?
- **Fully managed enterprise NFS/SMB file storage** built on NetApp ONTAP technology
- A joint offering between Google Cloud and NetApp
- Provides advanced storage capabilities beyond Filestore

### Key Features
- Supports **NFS v3/v4.1** and **SMB/CIFS** (dual-protocol access)
- **Snapshots and clones** — Instant, space-efficient backups
- **SnapVault and SnapMirror** — Disaster recovery and replication
- **Data tiering** — Automatic tiering between performance and capacity tiers
- **Encryption** at rest and in transit
- Compliance: SOC 2, HIPAA, PCI DSS

### Service Levels

| Level | Performance | Use Case |
|-------|-------------|----------|
| **Standard** | 16 MiB/s per TiB | General file shares |
| **Premium** | 64 MiB/s per TiB | File-intensive workloads |
| **Extreme** | 128 MiB/s per TiB | Latency-sensitive enterprise apps |

### When to Choose NetApp Volumes vs. Filestore

| Criteria | Google Cloud NetApp Volumes | Filestore |
|----------|-----------------------------|-----------|
| Protocol | NFS + SMB | NFS only |
| Enterprise features | Full ONTAP stack | Basic NFS |
| Snapshots | Yes (ONTAP native) | Yes (basic) |
| Replication | Cross-region SnapMirror | Limited |
| Cost | Higher | Lower |
| Use case | Enterprise SAP, Oracle, legacy ERP | General NFS, GKE persistent volumes |

```bash
# Enable the API
gcloud services enable netapp.googleapis.com

# Create a storage pool
gcloud netapp storage-pools create my-pool \
  --service-level=PREMIUM \
  --capacity-gib=4096 \
  --network=projects/PROJECT/global/networks/my-vpc \
  --location=us-central1

# Create a volume
gcloud netapp volumes create my-volume \
  --storage-pool=my-pool \
  --capacity-gib=1024 \
  --protocols=NFSV3 \
  --location=us-central1
```

---

## 12. Caching — Memorystore

> 📖 **Docs:** [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/redis-overview) | [Memorystore for Memcached](https://cloud.google.com/memorystore/docs/memcached/memcached-overview) | 🖥️ **Console:** Memorystore → Create instance

- **Memorystore** is fully managed Redis or Memcached
- Use when: repeated reads of the same data, session storage, rate limiting, leaderboards
- **Redis**: supports persistence, replication (STANDARD_HA tier), Lua scripting, Pub/Sub, Streams
  - STANDARD_HA: read replica in a different zone for HA
  - BASIC: single node, no replication
- **Memcached**: multi-threaded, simpler, pure cache (no persistence)
- **Memorystore for Redis Cluster**: horizontally scalable Redis cluster for very large datasets
- Decision: default to Redis; use Memcached only when multi-threading is required and no persistence needed; use Redis Cluster for >300 GB or very high throughput
- Planning tip: Size the cache to fit your working set; typical starting point is 20-30% of dataset size

```bash
# Create a Memorystore Redis instance (STANDARD_HA)
gcloud redis instances create my-redis \
  --tier=STANDARD_HA \
  --size=5 \
  --region=us-central1 \
  --redis-version=redis_7_0

# Create a Memorystore Memcached instance
gcloud memcache instances create my-memcached \
  --node-count=3 \
  --node-cpu=1 \
  --node-memory=1 \
  --region=us-central1
```

---

## 12b. Google Cloud Managed Service for Apache Kafka

### What Is Managed Apache Kafka?
- **Fully managed Apache Kafka** service on Google Cloud
- Provides a fully compatible Kafka API without cluster management overhead
- Integrates natively with Google Cloud services (Dataflow, BigQuery, Pub/Sub, GKE)

### Key Features
- **Standard Apache Kafka API** — No code changes for existing Kafka applications
- **Serverless** — No brokers to manage, auto-scaling
- **Built-in encryption** and VPC-native networking
- **IAM-based authentication** — Use GCP service accounts for Kafka auth
- Multi-zone high availability by default
- Integrates with **Kafka Connect** for source/sink connectors

### When to Choose Managed Apache Kafka vs. Pub/Sub

| Criteria | Managed Apache Kafka | Cloud Pub/Sub |
|----------|---------------------|---------------|
| Kafka compatibility | Native | No |
| Consumer groups | Yes (Kafka semantics) | Subscriptions |
| Message retention | Configurable (days) | 7 days (default) |
| Ordering | Per-partition guaranteed | Per-message key |
| Replay | Yes (offset-based) | Seek to timestamp |
| Migration from Kafka | Drop-in replacement | Requires code rewrite |
| Latency | Very low | Low |
| Best for | Migrating Kafka workloads, event streaming | New GCP-native apps |

```bash
# Enable the API
gcloud services enable managedkafka.googleapis.com

# Create a Kafka cluster
gcloud managed-kafka clusters create my-kafka \
  --location=us-central1 \
  --cpu=3 \
  --memory-bytes=3221225472 \
  --subnets=projects/PROJECT/regions/us-central1/subnetworks/my-subnet

# Create a Kafka topic
gcloud managed-kafka topics create my-topic \
  --cluster=my-kafka \
  --location=us-central1 \
  --partition-count=10 \
  --replication-factor=3
```

---

## 13. BigQuery Partitioning and Clustering

> 📖 **Docs:** [Partitioned tables](https://cloud.google.com/bigquery/docs/partitioned-tables) | [Clustered tables](https://cloud.google.com/bigquery/docs/clustered-tables) | 🖥️ **Console:** BigQuery → Create table → Partition and cluster settings

- **Partitioning**: divides a table into segments by a column value; queries that filter on the partition column scan only relevant partitions (reduces cost and latency)
  - Partition types: ingestion time (auto), DATE/TIMESTAMP/DATETIME column, INTEGER range
  - Max 4,000 partitions per table
- **Clustering**: sorts data within each partition by up to 4 columns; further reduces bytes scanned for queries that filter on clustered columns
  - Clustering columns: high-cardinality columns used in WHERE or GROUP BY
  - Clusters are automatically maintained as data is inserted

### Planning Guidance

| If you filter by... | Use... |
|--------------------|-------|
| Date/time | Date partitioning |
| Integer ranges | Integer range partitioning |
| High-cardinality label | Clustering |
| Both | Partition + cluster |

- Exam tip: Partitioning reduces cost (fewer bytes billed); clustering reduces latency (less data scanned within partition). Both are free features.

---

## 14. Encryption Options

> 📖 **Docs:** [Encryption at rest](https://cloud.google.com/docs/security/encryption/default-encryption) | [Cloud KMS overview](https://cloud.google.com/kms/docs/overview) | 🖥️ **Console:** Security → Key Management

- **Default**: all GCP data encrypted at rest with Google-managed keys (AES-256) — automatic, no configuration needed
- **CMEK (Customer-Managed Encryption Keys)**: you manage keys in Cloud KMS; GCP uses them to encrypt your data
  - Use case: compliance requirements, key lifecycle control, ability to revoke access by disabling the key
  - Supported by: Cloud Storage, BigQuery, Compute Engine disks, Cloud SQL, Pub/Sub, Cloud Run, GKE
  ```bash
  # Create a KMS key ring and key
  gcloud kms keyrings create my-keyring --location=us-central1
  gcloud kms keys create my-key --keyring=my-keyring --location=us-central1 --purpose=encryption
  # Apply to a GCS bucket
  gcloud storage buckets create gs://my-bucket --default-encryption-key=projects/PROJECT/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key
  ```
- **CSEK (Customer-Supplied Encryption Keys)**: you supply raw key material to GCP; GCP never stores it; you must supply the key for every operation; highest control, highest responsibility
  - Only supported by Cloud Storage and Compute Engine disks
- Exam tip: CMEK = keys in Cloud KMS (GCP stores metadata, you own the key); CSEK = raw key you supply per-request (GCP stores nothing); Google-managed = simplest, no action required

---

## 15. Maintaining Multi-Region Redundancy Across Data Solutions

> 📖 **Docs:** [Disaster recovery planning](https://cloud.google.com/architecture/dr-scenarios-planning-guide) | [Regional and multi-regional resources](https://cloud.google.com/about/locations) | 🖥️ **Console:** per-service region selection at creation

A key exam topic is planning data solutions that remain available and durable across multiple regions. Here is the strategy per service:

### Multi-Region Redundancy Reference

| Service | Multi-Region Option | How to Configure |
|---------|--------------------|--------------------|
| **Cloud Storage** | Multi-region or dual-region bucket | `--location=us` or `--location=NAM4` at creation |
| **Cloud SQL** | Cross-region read replicas + external replicas | `gcloud sql instances create ... --region=us-east1 --master-instance-name=...` |
| **AlloyDB** | Cross-region replication (preview) | Configure secondary cluster in a different region |
| **Cloud Spanner** | Multi-region instance config | Choose `nam-eur-asia1` or similar multi-region config |
| **BigQuery** | Multi-region datasets (`US`, `EU`) | `bq mk --location=US dataset` |
| **Firestore** | Multi-region location | Set at database creation (`--location=nam5`) |
| **Bigtable** | Multi-cluster replication | Add cluster in second region: `gcloud bigtable clusters create` |
| **Memorystore** | Manual — create instances per region | Deploy one instance per region; app handles routing |
| **Pub/Sub** | Global by default | Topics are global; no configuration needed |
| **Managed Kafka** | Use multiple clusters per region | Deploy per-region clusters; use Kafka MirrorMaker 2 |

### Key Design Principles
1. **Active-active**: Both regions serve traffic (Spanner, Bigtable multi-cluster, Cloud Storage multi-region)
2. **Active-passive**: Primary serves traffic; secondary takes over on failure (Cloud SQL with cross-region replica)
3. **Data residency**: Multi-region keeps data within a geographic area; dual-region pins to two specific regions
4. **SLA impact**: Multi-region configurations have higher SLAs (99.999% for Spanner, 99.95% for Cloud Storage dual-region)
5. **Cost**: Multi-region configurations typically cost 2x+ single-region due to replication traffic

### Exam Scenarios

| Scenario | Recommended Approach |
|----------|---------------------|
| 99.999% SLA relational database | Cloud Spanner multi-region |
| Compliance: data must stay in EU | Firestore `europe` or BigQuery `EU` location |
| Global user base with low latency reads | Bigtable multi-cluster OR Cloud SQL read replicas per region |
| Files served globally with high durability | Cloud Storage multi-region (`us`, `eu`, `asia`) |
| Kafka workload resilient to region failure | Managed Kafka with MirrorMaker 2 across regions |

---

## Exam Practice Questions

1. **A global financial services company needs a relational database with 99.999% availability and strong consistency across continents. Which service should they use?**
   - Answer: **Cloud Spanner** — Only GCP database offering global strong consistency with 99.999% SLA.

2. **You need to store and analyze 5 PB of historical log data using SQL. Which service should you use?**
   - Answer: **BigQuery** — Serverless data warehouse designed for petabyte-scale SQL analytics.

3. **A mobile app needs real-time sync of user data across devices with offline support. Which database should you use?**
   - Answer: **Firestore (Native mode)** — Built for mobile/web with real-time listeners and offline support.

4. **You need to store backup data that is accessed about once per quarter. Which storage class minimizes cost?**
   - Answer: **Coldline** — Designed for data accessed less than once per quarter, with lower storage costs than Standard/Nearline.

5. **An IoT application generates millions of time-series data points per second with low-latency read requirements. Which database is best?**
   - Answer: **Cloud Bigtable** — Designed for high-throughput time-series data with millisecond latency.

6. **A web application uses PostgreSQL and needs to handle both transactional queries and complex analytical reports. Which service would you recommend?**
   - Answer: **AlloyDB** — PostgreSQL-compatible with excellent performance for both OLTP and OLAP (HTAP) workloads.

7. **What is the key difference between Nearline and Coldline storage?**
   - Answer: Minimum storage duration (30 days vs 90 days) and retrieval costs (Coldline is more expensive to retrieve). Both have the same access latency (milliseconds).

8. **You need a block storage device that provides high availability across zones for a production database. What should you use?**
   - Answer: **Regional Persistent Disk (pd-ssd or pd-extreme)** — Synchronously replicates data across two zones for HA.

---

## Glossary

**ACID (Atomicity, Consistency, Isolation, Durability)** — A set of properties that guarantee database transactions are processed reliably; Cloud SQL, AlloyDB, and Spanner provide full ACID compliance.

**AlloyDB** — GCP's fully managed, PostgreSQL-compatible database service combining open-source PostgreSQL with Google's infrastructure; up to 4x faster for OLTP and 100x faster for analytical queries than standard PostgreSQL.

**Archive** — The coldest Cloud Storage class, with a minimum storage duration of 365 days; lowest storage cost but highest retrieval cost; designed for data accessed less than once per year.

**Automatic Sharding** — A feature of Cloud Spanner that automatically distributes data across nodes based on key ranges without manual partitioning, enabling horizontal scalability.

**Autoclass** — A Cloud Storage bucket-level feature that automatically transitions objects to the most cost-effective storage class based on access patterns, removing the need for manual lifecycle rules.

**AES-256 (Advanced Encryption Standard, 256-bit)** — The encryption algorithm used by GCP to protect all data at rest by default using Google-managed keys.

**Backup** — A copy of data used to restore the original in case of loss or corruption; Cloud SQL, AlloyDB, and Spanner offer automated managed backups, while Compute Engine disks rely on snapshots.

**BigQuery Data Transfer Service** — A BigQuery feature that automates scheduled, managed data ingestion from SaaS applications (e.g., Google Ads, YouTube, Campaign Manager) and other cloud storage sources.

**BI Engine** — BigQuery's in-memory analysis service that accelerates SQL queries for business intelligence dashboards and reporting tools.

**BigQuery** — GCP's serverless, highly scalable data warehouse designed for OLAP analytical workloads; processes petabytes of data using Standard SQL with separate storage and compute billing.

**BigQuery GIS** — BigQuery feature for geospatial analysis, enabling SQL queries over geographic data types such as points, lines, and polygons.

**BigQuery ML** — Feature within BigQuery that enables creating, training, and executing machine learning models using SQL statements without moving data.

**Bigtable (Cloud Bigtable)** — GCP's fully managed wide-column NoSQL database built for massive analytical and operational workloads; powers Google Search, Maps, and Gmail internally; offers millisecond latency at any scale.

**Block Storage** — Storage type that presents raw storage volumes to VMs as block devices; Persistent Disk and Local SSD are GCP's block storage options.

**Bucket** — The fundamental container in Cloud Storage where objects (files) are stored; every object belongs to exactly one bucket.

**Bucket Lock** — A Cloud Storage feature that permanently enforces a retention policy on a bucket; once locked, the retention duration cannot be reduced or removed, enabling WORM-style compliance.

**Capacitor** — The columnar storage format used internally by BigQuery to optimize compression and scan performance for analytical queries.

**Change Streams** — A Cloud Spanner feature that captures data changes in real time, enabling event-driven architectures and downstream data synchronization.

**CIDR (Classless Inter-Domain Routing)** — A notation for specifying IP address ranges (e.g., `10.0.0.0/24`); used in networking and referenced when Cloud Storage buckets have IP-based access restrictions.

**Clustering (BigQuery)** — A BigQuery optimization that sorts data within each partition by up to four columns, reducing the bytes scanned for queries filtering on those columns; clusters are automatically maintained as data is inserted.

**Cloud KMS (Key Management Service)** — GCP's managed service for creating, rotating, and controlling cryptographic keys; used with CMEK to encrypt data across multiple GCP services.

**Cloud SQL** — GCP's fully managed relational database service supporting MySQL, PostgreSQL, and SQL Server; handles backups, replication, patches, and failover automatically.

**Cloud SQL Auth Proxy** — A binary that provides secure, authenticated access to Cloud SQL instances without requiring SSL certificates or authorized networks.

**Cloud Storage** — GCP's object storage service for storing and retrieving unstructured data in buckets; offers four storage classes (Standard, Nearline, Coldline, Archive) with millisecond access latency for all classes.

**CMEK (Customer-Managed Encryption Keys)** — An encryption option where the customer manages keys in Cloud KMS and GCP uses those keys to encrypt data; allows key revocation and lifecycle control.

**Coldline** — A Cloud Storage class designed for data accessed less than once per quarter; has a 90-day minimum storage duration with higher retrieval costs than Standard or Nearline.

**Collection** — In Firestore Native mode, a grouping of documents analogous to a table in relational databases; collections can contain sub-collections for hierarchical data.

**Columnar Engine** — AlloyDB's built-in analytical acceleration layer that stores data in column-oriented format for fast analytical query execution alongside transactional workloads.

**CSEK (Customer-Supplied Encryption Keys)** — An encryption option where the customer supplies raw key material directly to GCP per request; GCP never stores the key; supported only by Cloud Storage and Compute Engine disks.

**Dataflow** — GCP's fully managed stream and batch data processing service based on the Apache Beam programming model; integrates natively with Bigtable, BigQuery, and Cloud Storage.

**Dataproc** — GCP's managed Apache Hadoop and Spark service; integrates with Bigtable for MapReduce workloads.

**Dataset (BigQuery)** — A top-level container in BigQuery that holds tables, views, and models; datasets are created within a project and have a single location (region or multi-region).

**Dual-Region** — A Cloud Storage location type that replicates data across two specific, user-chosen regions for higher availability and lower cross-regional latency (e.g., `nam4`).

**Datastore Mode** — A Firestore operational mode backward-compatible with the original Cloud Datastore API; uses entities and kinds instead of documents and collections; no real-time listeners or offline support.

**Document** — The fundamental data unit in Firestore; a set of key-value pairs (fields) organized within a collection; analogous to a row in a relational database.

**Early Deletion Charge** — A Cloud Storage fee applied when an object is deleted before its storage class's minimum storage duration expires; the charge covers the remaining minimum duration period.

**EDA (Electronic Design Automation)** — Engineering workloads for designing integrated circuits; a use case for Filestore's shared file storage.

**Encryption at Rest** — Encryption of data stored on physical media; GCP encrypts all data at rest by default using AES-256 with Google-managed keys.

**Entity** — The data record type in Firestore Datastore mode; equivalent to a document in Native mode; belongs to a "kind" (analogous to a collection).

**Eventual Consistency** — A consistency model in which, given enough time without updates, all replicas will converge to the same value; used by Bigtable and BigQuery; contrasts with strong consistency.

**File Storage** — Storage type providing a shared file system accessible by multiple clients via standard file protocols; Filestore is GCP's managed file storage service.

**Fine-Grained ACL** — The legacy Cloud Storage access control system that applies ACL entries to individual objects; superseded by Uniform Bucket-Level Access for simpler, consistent management.

**Free Tier** — The monthly quota of free usage offered for many GCP services (e.g., BigQuery's first 1 TB processed and 10 GB of active storage per month).

**Filestore** — GCP's managed NFS file storage service providing shared file systems accessible by multiple Compute Engine VMs and GKE pods; available in Basic HDD, Basic SSD, High Scale SSD, and Enterprise tiers.

**Firestore** — GCP's serverless NoSQL document database designed for mobile, web, and IoT applications; available in Native mode (real-time sync, offline support) and Datastore mode (server-side compatibility).

**Five Nines (99.999% SLA)** — The highest availability tier offered by GCP, provided by Cloud Spanner and Firestore; allows less than 5.26 minutes of downtime per year.

**gcloud** — The primary command-line tool for interacting with GCP services, including Cloud Storage (via `gcloud storage`), BigQuery, Cloud SQL, and other data services.

**Google-Managed Encryption Keys** — The default encryption model in GCP where Google automatically generates, manages, and rotates encryption keys on behalf of the customer with no user action required.

**GQL (Google Query Language)** — SQL-like query language used in Firestore Datastore mode for querying entities.

**gsutil** — The legacy command-line tool for Cloud Storage operations; being replaced by `gcloud storage`, which offers equivalent and newer functionality.

**HA (High Availability)** — Design approach ensuring a system remains operational with minimal downtime; achieved in Cloud SQL via synchronous regional failover, and in Persistent Disk via Regional PD.

**HBase API** — The Apache HBase client API; Cloud Bigtable is compatible with it, allowing Hadoop ecosystem tools to read from and write to Bigtable without code changes.

**HDD (Hard Disk Drive)** — Magnetic spinning disk storage; used in pd-standard Persistent Disks and Filestore Basic HDD tier; lower cost but lower IOPS than SSD.

**Hot Data** — Frequently accessed data; best stored in Cloud Storage Standard class or in databases with low-latency access patterns.

**HTAP (Hybrid Transactional/Analytical Processing)** — A database workload pattern combining OLTP and OLAP queries; AlloyDB is optimized for HTAP workloads.

**IAM (Identity and Access Management)** — GCP's system for controlling access to resources; applies to Cloud Storage buckets, BigQuery datasets, and all other storage services.

**Instance (Database)** — A deployed database server managed by a GCP service (e.g., a Cloud SQL instance or Spanner instance) that hosts databases and serves queries.

**Integer Range Partitioning** — A BigQuery partitioning strategy that divides a table into segments based on integer column value ranges.

**IOPS (Input/Output Operations Per Second)** — A measure of storage performance; pd-ssd and pd-extreme Persistent Disks offer high IOPS for database workloads.

**IoT (Internet of Things)** — Network of physical devices generating sensor data; a primary use case for Firestore (device state) and Bigtable (time-series metrics).

**Key Ring (Cloud KMS)** — A logical grouping of cryptographic keys within a Cloud KMS location; keys are created within a key ring, which exists in a specific region or multi-region.

**Kind** — In Firestore Datastore mode, the category or type of an entity; analogous to a table in relational databases or a collection in Native mode.

**Lifecycle Rule** — A Cloud Storage configuration that automatically transitions objects between storage classes or deletes them based on conditions such as object age, storage class, or number of noncurrent versions.

**Local SSD** — Physically attached NVMe SSD on a Compute Engine host; offers up to 2.4 million read IOPS but is ephemeral — data is lost when the VM stops or is deleted; sold in 375 GB increments.

**Long-Term Storage Pricing** — BigQuery's reduced storage pricing ($0.01/GB/month) automatically applied to tables or partitions not modified for 90 or more consecutive days.

**MapReduce** — Distributed data processing programming model; Bigtable integrates natively with MapReduce via its HBase API compatibility.

**Memcached** — Open-source, multi-threaded in-memory key-value caching system; available via GCP Memorystore; suitable when no persistence or complex data structures are needed.

**Memorystore** — GCP's fully managed in-memory caching service supporting Redis and Memcached; used for session storage, rate limiting, leaderboards, and frequently read data.

**Multi-Region** — A Cloud Storage or database configuration spanning multiple geographic regions for higher availability and lower latency for geographically distributed users.

**MySQL** — Open-source relational database management system; one of the three database engines supported by Cloud SQL.

**Native Mode** — The default and recommended Firestore operational mode; supports real-time listeners, offline sync for mobile SDKs, multi-document ACID transactions, and collection group queries.

**Nearline** — A Cloud Storage class designed for data accessed less than once per month; has a 30-day minimum storage duration with lower retrieval costs than Coldline or Archive.

**NFS (Network File System)** — Standard file-sharing protocol; used by Filestore to provide shared file access to multiple Compute Engine VMs and GKE pods.

**NoSQL** — A category of database management systems that do not use traditional relational (tabular) data models; Firestore and Bigtable are GCP's primary NoSQL databases.

**Object** — The fundamental unit stored in Cloud Storage; consists of data (the file content) plus metadata; objects are stored within buckets.

**Object Hold** — A Cloud Storage feature that prevents a specific object from being deleted or overwritten, regardless of the bucket's retention policy; available as temporary or event-based holds.

**Object Lifecycle Management** — A Cloud Storage feature that automatically transitions objects between storage classes or deletes them based on configurable rules (age, storage class, number of versions, etc.).

**Object Versioning** — A Cloud Storage bucket-level feature that preserves noncurrent (previous) versions of objects when they are overwritten or deleted, protecting against accidental data loss.

**OLAP (Online Analytical Processing)** — Workload category focused on complex analytical queries over large datasets; BigQuery and AlloyDB's columnar engine serve OLAP workloads.

**OLTP (Online Transactional Processing)** — Workload category focused on fast, high-volume read/write transactions; Cloud SQL, AlloyDB, and Spanner serve OLTP workloads.

**On-Demand Pricing (BigQuery)** — BigQuery pricing model charging $6.25 per TB of data processed by queries; the first 1 TB per month is free.

**Oracle** — Third-party enterprise relational database; AlloyDB is commonly chosen as a migration target for Oracle workloads due to its PostgreSQL compatibility and high performance.

**Partitioning (BigQuery)** — A BigQuery optimization that divides a table into segments by a column value (date/time, integer range, or ingestion time); queries filtering on the partition column scan only relevant partitions, reducing cost.

**pd-balanced** — Balanced SSD Persistent Disk type offering medium IOPS and throughput; the default general-purpose disk type for Compute Engine.

**pd-extreme** — Highest-performance SSD Persistent Disk type providing the maximum IOPS and throughput; designed for mission-critical database workloads.

**pd-ssd** — Performance SSD Persistent Disk type offering high IOPS and throughput; suitable for database workloads requiring fast random I/O.

**pd-standard** — Standard HDD Persistent Disk type with low IOPS; suited for bulk storage and sequential I/O workloads where cost is prioritized over performance.

**Persistent Disk** — GCP's durable network-attached block storage for Compute Engine VMs and GKE nodes; available as Zonal or Regional PD in HDD and SSD variants.

**PITR (Point-in-Time Recovery)** — Database feature that allows restoring data to any specific moment within a retention window; supported by Cloud SQL and AlloyDB.

**PostgreSQL** — Open-source object-relational database management system; one of the three engines supported by Cloud SQL and the compatibility layer for AlloyDB and Spanner.

**QPS (Queries Per Second)** — A measure of read/write throughput; Bigtable is recommended for workloads exceeding 10,000 QPS with low-latency requirements.

**Read Replica** — A copy of a database that serves read traffic, offloading it from the primary instance; supported by Cloud SQL within a region, cross-region, and externally.

**Region** — A specific geographic location (e.g., `us-central1`) where GCP resources are hosted; Cloud Storage buckets, persistent disks, and database instances can be placed in a region for data locality and compliance.

**Regional (Storage)** — A Cloud Storage location type that stores data within a single region, offering the lowest cost and lowest cross-resource latency for applications in that region.

**Redis** — Open-source in-memory data structure store supporting persistence, replication, Pub/Sub, and Lua scripting; available via GCP Memorystore; the default recommendation for caching.

**Regional Persistent Disk** — A Persistent Disk that synchronously replicates data across two zones within a single region; provides faster failover and higher availability than Zonal PD at approximately 2x the cost.

**Relational Database** — A database that organizes data into structured tables with defined schemas and supports SQL queries; Cloud SQL, AlloyDB, and Spanner are GCP's relational database offerings.

**Retention Policy** — A Cloud Storage bucket-level setting that enforces a minimum retention period; objects cannot be deleted or overwritten until they reach the configured age.

**Resource** — Any addressable entity within GCP (e.g., a Cloud Storage bucket, BigQuery dataset, Spanner instance); resources exist in projects and are the target of IAM bindings and quotas.

**Signed Cookie** — A time-limited, cryptographically signed cookie granting access to a set of Cloud Storage URLs under a common prefix; useful for protecting premium or private content.

**Signed URL** — A time-limited URL granting temporary, authenticated access to a specific Cloud Storage object without requiring the recipient to have a Google account.

**Slots (BigQuery)** — Units of BigQuery query processing capacity; purchased in the Capacity Editions pricing model for predictable, flat-rate query costs independent of data scanned.

**Spanner (Cloud Spanner)** — GCP's globally distributed, strongly consistent, horizontally scalable relational database; offers unlimited scale with a 99.999% SLA; supports ACID transactions across regions.

**SQL (Structured Query Language)** — Standard language for querying and manipulating relational data; supported by Cloud SQL, AlloyDB, Spanner (Google SQL dialect), and BigQuery (Standard SQL).

**SQL Server** — Microsoft's relational database management system; one of the three database engines supported by Cloud SQL.

**SLA (Service Level Agreement)** — A contractual commitment from GCP for a service's minimum level of availability (e.g., Spanner 99.999%, Cloud SQL HA 99.95%); used for planning and compliance.

**SSD (Solid-State Drive)** — Flash-based storage offering high IOPS and low latency; used in pd-balanced, pd-ssd, pd-extreme Persistent Disks, Local SSDs, and Bigtable SSD storage.

**Storage Class** — A property assigned to Cloud Storage buckets or objects (Standard, Nearline, Coldline, Archive) that determines pricing and minimum storage duration.

**Storage Transfer Service** — A GCP managed service that automates scheduled data transfers into Cloud Storage from on-premises file systems, AWS S3, Azure Blob Storage, or other Cloud Storage buckets.

**Standard (Storage Class)** — The default Cloud Storage class with no minimum storage duration and no retrieval cost; designed for frequently accessed ("hot") data.

**Standard SQL** — ANSI-compliant SQL dialect used by BigQuery for querying data warehouse tables.

**Streaming Inserts** — A BigQuery feature for inserting individual rows in near-real time (rather than batch loads); enables real-time analytics at a per-GB cost.

**Strong Consistency** — A consistency model guaranteeing that all reads reflect the most recent write; provided by Cloud SQL, AlloyDB, Spanner (globally), and Firestore Native mode.

**Sub-Collection** — A collection nested within a Firestore document, enabling hierarchical data modeling without denormalization.

**Table (BigQuery)** — A BigQuery structured data store that holds rows with a defined schema; belongs to a dataset and may be partitioned and/or clustered.

**Throughput** — The rate of data transfer measured in MB/s; a key performance characteristic of Persistent Disk types alongside IOPS.

**Time-Series Data** — Measurements recorded at successive points in time (e.g., IoT sensor readings, financial ticks, monitoring metrics); the primary use case for Cloud Bigtable.

**Transfer Appliance** — A rack-mounted physical storage device shipped by Google to customers for securely transferring large volumes of data (tens of terabytes to petabytes) offline into Cloud Storage.

**Uniform Bucket-Level Access** — A Cloud Storage setting (enforceable via org policy) that disables per-object ACLs and requires all access to be managed through IAM; simplifies access control.

**Vertex AI** — GCP's unified machine learning platform; relevant to data storage planning as it integrates with BigQuery for ML model training on structured data.

**Wide-Column Database** — A NoSQL database type that organizes data into rows with a flexible set of columns; Bigtable is GCP's wide-column database, similar in design to Apache HBase and Cassandra.

**WORM (Write Once, Read Many)** — A data storage paradigm ensuring data cannot be modified after it is written; implemented in Cloud Storage via retention policies and Bucket Lock for regulatory compliance.

**Zone** — An isolated deployment area within a GCP region (e.g., `us-central1-a`); zonal resources such as Zonal Persistent Disks and Compute Engine VMs reside in a single zone.

**Zonal Persistent Disk** — A Persistent Disk that resides within a single zone; the standard, lower-cost block storage option for Compute Engine when cross-zone HA is not required.
