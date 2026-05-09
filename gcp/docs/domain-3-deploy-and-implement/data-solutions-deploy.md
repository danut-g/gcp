# Data Solutions Deployment: Cloud SQL, Bigtable, Spanner, Firestore, BigQuery

## Overview

Deploying GCP data services involves understanding their creation parameters, high availability configurations, connection mechanisms, and data loading patterns. Each service has specific deployment considerations that differ significantly from one another.

---

## Key Concepts

### Cloud SQL Deployment

#### Instance Creation Key Parameters

| Parameter | Description | Notes |
|-----------|-------------|-------|
| **Database engine** | MySQL, PostgreSQL, SQL Server | Cannot change after creation |
| **Region** | Where instance runs | Cannot change after creation |
| **Zone** | Primary zone | Can specify preferred zone |
| **Machine type** | vCPU + memory | Tier (db-n1-standard-X, etc.) |
| **Storage type** | SSD or HDD | SSD recommended |
| **Storage capacity** | Starting size | Can auto-increase |
| **High availability** | Single zone vs Regional (HA) | HA adds standby in different zone |
| **Backup** | Automated backup + PITR | PITR requires binary logging (MySQL) / WAL (PG) |
| **Network** | Public IP, Private IP, or both | Private IP requires VPC |

#### High Availability Configuration

- **HA configuration**: Creates a primary instance + standby in a different zone of the same region
- Replication: Synchronous — standby receives writes in sync with primary
- Failover: Automatic in ~60 seconds when primary is unavailable
- After failover, the old standby becomes the new primary
- **Cost**: HA instance costs 2x (primary + standby)

#### Read Replicas

- Asynchronous replication; lag can vary
- Cross-region read replicas: Available for MySQL and PostgreSQL
- Read replicas can be promoted to standalone instances (for disaster recovery or regional failover)
- Up to 10 read replicas per primary

#### Connecting to Cloud SQL

| Method | Description | Use Case |
|--------|-------------|---------|
| **Cloud SQL Auth Proxy** | Local proxy that handles IAM auth + encryption | Recommended; works from any environment |
| **Private IP** | Direct connection within VPC | Lowest latency; requires VPC |
| **Public IP + SSL** | Public endpoint; restrict with authorized networks | When private IP not available |
| **Public IP + no auth restrictions** | Insecure | Dev/test only — never production |

- **Cloud SQL Auth Proxy**: Runs as a sidecar or local process; handles authentication via service account credentials; encrypts all traffic; you connect to `127.0.0.1:3306` (or similar) locally

#### Cloud SQL Editions

- **Enterprise**: Standard performance
- **Enterprise Plus**: Higher performance, faster failover, longer backup retention

---

### Cloud Bigtable Deployment

#### Instance Configuration

| Parameter | Description |
|-----------|-------------|
| **Instance type** | Production (3+ nodes, SLA) or Development (1 node, no SLA) |
| **Storage type** | SSD (default, recommended) or HDD (for less latency-sensitive, large data, lower cost) |
| **Cluster** | At least 1 cluster per instance; 2+ for replication |
| **Number of nodes** | Minimum 3 for production SLA |
| **Region/Zone** | Each cluster is in a zone |

#### Replication

- Add a second cluster in another zone or region for high availability / read scalability
- **Replication is asynchronous** for multi-cluster; eventual consistency
- Replication policies: `Any-cluster routing`, `Multi-cluster routing` (read from nearest), `Single-cluster routing` (read from one cluster)

#### Scaling

- Add/remove nodes without downtime
- Performance scales linearly with nodes: ~10,000 QPS per node (read or write)
- After adding nodes, performance improves within ~20 minutes as data rebalances

#### Table Design (Key Points for Exam)

- **Row key design** is the most critical design decision
- Bad row keys: Sequential IDs or timestamps as prefix → create write hotspots
- Good row keys: Reverse timestamps (for time-series: most recent data accessible), hash prefixes, or composite keys
- Column families: Group related columns; each family can have its own GC policy (garbage collection)
- Cell versions: Each cell stores multiple timestamped versions; GC policy controls retention

---

### Cloud Spanner Deployment

#### Instance Configuration

| Parameter | Description |
|-----------|-------------|
| **Configuration** | Regional or Multi-regional |
| **Processing units** | Unit of compute (1 node = 1000 PUs); minimum 100 PUs |
| **Databases** | Create one or more databases within an instance |
| **Schema** | DDL-based; supports interleaved tables |

#### Regional vs Multi-Regional Configurations

| Type | SLA | RPO/RTO | Latency | Cost |
|------|-----|---------|---------|------|
| Regional | 99.99% (4 nines) | Near-zero | Low single-region | Lower |
| Multi-regional | 99.999% (5 nines) | Zero (synchronous replication) | Higher for writes | Higher |

- Multi-regional Spanner replicates synchronously across regions — this causes slightly higher write latency but guarantees zero data loss on regional failure

#### Interleaved Tables

- Child rows are physically stored with parent rows
- Declared via `INTERLEAVE IN PARENT` in DDL
- Critical for performance when child data is always accessed with parent (avoids cross-shard joins)
- Example: `Orders` interleaved in `Customers` — retrieving all orders for a customer is a local read

---

### Firestore Deployment

#### Mode Selection (Permanent)

- **Native mode**: Modern API, real-time updates, offline support
- **Datastore mode**: Legacy Datastore API compatibility
- Mode cannot be changed after database creation within a project
- A project can have only one Firestore database in "default" mode; multiple named databases are supported in newer configurations

#### Data Model

- **Collections** → **Documents** → **Fields** + **Sub-collections**
- Documents are limited to **1 MB** each
- Collection groups: Query across all collections with the same name (using Collection Group queries)
- Composite indexes: Required for queries with multiple filter fields; defined in `firestore.indexes.json`

#### Security Rules

- Firestore has its own **Security Rules** system (separate from IAM)
- Used for client-side SDK access (mobile/web)
- Server-side access (backend services) uses IAM roles: `roles/datastore.user`, `roles/datastore.viewer`

---

### BigQuery Deployment and Data Loading

#### Dataset Creation

- Datasets are project-level containers for tables
- Regional or multi-regional location (set at creation; cannot change)
- IAM permissions set at dataset or table level: `roles/bigquery.dataEditor`, `roles/bigquery.dataViewer`

#### Table Types

| Type | Description |
|------|-------------|
| **Native table** | Standard BigQuery table |
| **External table** | Query data in GCS/Drive/Bigtable/Sheets without loading it |
| **Materialized view** | Precomputed results of a query; automatically refreshed |
| **View** | Virtual table; query runs on demand |
| **Partitioned table** | Partitioned by column (date/timestamp) or ingestion time |
| **Clustered table** | Data sorted by one or more columns |

#### Data Loading Methods

| Method | Description | Use Case |
|--------|-------------|---------|
| **Batch load (bq load)** | Load from GCS, local files | Initial loads, scheduled ETL |
| **Streaming insert** | Insert rows in real-time via API | Low-latency data ingestion |
| **Storage Write API** | High-throughput, exactly-once streaming | Modern streaming alternative |
| **Transfer Service** | Scheduled transfers from SaaS (Google Ads, YouTube) | Marketing data pipelines |
| **Dataflow** | Apache Beam pipelines | Complex ETL with transformations |
| **Pub/Sub → BigQuery** | Pub/Sub subscription directly writes to BigQuery | Event-driven streaming |

#### Partitioning and Clustering

- **Partition by ingestion time**: Automatic, no column needed; `_PARTITIONTIME` pseudo-column
- **Partition by column**: Specify a DATE/TIMESTAMP/INT64 column
- **Clustering**: Sort data within partitions by up to 4 columns
- Always filter queries by partition column to avoid full table scans (reduces cost)

---

## When to Use Each Service

See [storage-planning.md](../domain-2-plan-and-configure/storage-planning.md) for the decision matrix. Deployment-specific considerations:

- **Cloud SQL HA** for any production relational workload with RPO/RTO requirements
- **Cloud SQL Auth Proxy** always — safest connection method
- **Bigtable SSD** for production; HDD only when large volumes of less-accessed data
- **Spanner multi-regional** only when 5-nines SLA and global ACID transactions justify the cost
- **Firestore Native** for all new applications; Datastore mode only for Datastore migration
- **BigQuery partitioned tables** for any time-series data; mandatory for cost control at scale

---

## Related Services / Concepts

- **Storage Planning**: Service selection — see [storage-planning.md](../domain-2-plan-and-configure/storage-planning.md)
- **Managing Storage**: Lifecycle, backups, retention — see [managing-storage.md](../domain-4-ensure-success/managing-storage.md)
- **Data Security**: CMEK, encryption at rest — see [data-security.md](../domain-5-configure-access-and-security/data-security.md)
- **Networking Deploy**: Private networking for databases — see [networking-deploy.md](networking-deploy.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Cloud SQL region is permanent**: Like App Engine, you can't change the region of a Cloud SQL instance. Plan carefully.

2. **HA Cloud SQL costs 2x**: The standby instance is always running and billed. Budget accordingly.

3. **Cloud SQL failover is ~60 seconds**: HA doesn't mean zero downtime. There's a ~60-second failover window. If your app can't tolerate that, you need connection pooling and retry logic.

4. **Bigtable HDD vs SSD**: HDD is for archival/analytical workloads with infrequent access. SSD for all latency-sensitive workloads. The exam may present cost vs latency tradeoffs.

5. **Bigtable production minimum 3 nodes**: Fewer than 3 nodes = no SLA. Development instances have 1 node but no SLA.

6. **Firestore mode is permanent within a project**: Native vs Datastore mode — choose carefully. No switching. For new apps, always Native.

7. **BigQuery streaming inserts have no free tier**: Batch loads are free; streaming inserts are charged per GB. For cost-sensitive scenarios, batch loading into BigQuery is preferred.

8. **Spanner minimum cost**: 100 processing units (0.1 node) still costs ~$72/month (US regional). Not a cheap option for small workloads.

9. **BigQuery external tables don't store data in BQ**: Querying external data (e.g., GCS files) incurs GCS read costs + BQ query costs. Data isn't replicated into BQ storage.

### Keywords
- Cloud SQL Auth Proxy, HA configuration, regional standby, read replica, Bigtable row key hotspot, interleaved tables, Spanner processing units, multi-regional Spanner, Firestore Native vs Datastore mode, BigQuery partitioning, BigQuery clustering, streaming inserts, batch load

---

## Source

- https://cloud.google.com/sql/docs/mysql/create-instance
- https://cloud.google.com/bigtable/docs/creating-instance
- https://cloud.google.com/spanner/docs/create-manage-instances
- https://cloud.google.com/firestore/docs/manage-databases
- https://cloud.google.com/bigquery/docs/datasets-intro
