# Storage Planning: Cloud Storage, SQL, Spanner, Bigtable, Firestore, Memorystore

## Overview

GCP offers a diverse set of storage and database services, each optimized for different data models, access patterns, and scale requirements. Selecting the right storage solution is one of the most frequently tested skill areas in the ACE exam. The key is understanding the primary decision dimensions: **structure**, **scale**, **consistency**, **access pattern**, and **latency**.

---

## Key Concepts

### Cloud Storage (Object Storage)

- Stores unstructured data as **objects** within **buckets**
- Objects are immutable — you replace, not update, an object
- Globally unique bucket names across all of GCP
- Buckets have a **location type**: regional, dual-region, or multi-region

#### Storage Classes

| Class | Minimum Storage Duration | Access Frequency | Price (approx, US) |
|-------|------------------------|-----------------|-------------------|
| **Standard** | None | Frequent/continuous | Highest per-GB |
| **Nearline** | 30 days | Monthly or less | ~50% of Standard |
| **Coldline** | 90 days | Quarterly or less | ~25% of Standard |
| **Archive** | 365 days | Annual or less | Lowest per-GB |

- Access costs increase as storage costs decrease (retrieval fee per GB read)
- Early deletion fee applies if objects are deleted before minimum duration
- **No difference in latency** — all classes deliver objects with the same speed; classes only affect cost, not performance
- Objects can be transitioned between classes using **lifecycle policies**

#### Bucket Location Types

| Type | Description | Use Case |
|------|-------------|---------|
| **Regional** | Single GCP region | Lowest latency for regional workloads |
| **Dual-region** | Two specific paired regions | High availability with some geo-redundancy |
| **Multi-region** | Broad geographic area (US, EU, ASIA) | Global distribution, highest availability |

- Multi-region buckets replicate data across multiple regions within the geographic area
- Data stored in an EU multi-region bucket stays within the EU
- **Turbo replication** (dual-region/multi-region): 15-minute RPO guarantee for replication

#### Key Cloud Storage Features

- **Versioning**: Keeps old versions of objects when overwritten/deleted; must be explicitly enabled
- **Retention policies**: Lock objects for a minimum retention period (WORM — Write Once, Read Many)
- **Lifecycle rules**: Automatically transition or delete objects based on age, version count, storage class
- **Signed URLs**: Time-limited access to private objects without GCP credentials
- **Customer-managed encryption keys (CMEK)**: Use Cloud KMS keys instead of Google-managed keys
- **Requester Pays**: Requester's project is charged for access costs instead of bucket owner
- **Uniform bucket-level access**: Disables legacy ACLs; all access controlled by IAM only (recommended)

---

### Cloud SQL

- Fully managed **relational database** service
- Supported engines: **PostgreSQL** (9.6 through 16), **MySQL** (5.7, 8.0), **SQL Server** (2017, 2019, 2022)
- Use cases: OLTP workloads, web application backends, CMS, ERP
- Limitations:
  - Max instance size: 96 vCPUs, 624 GB RAM (standard tier)
  - Storage: Up to 64 TB
  - Single-region only (no multi-region active-active)
  - Vertical scaling requires instance restart
- High availability: **Regional (HA) configuration** — primary and standby in the same region, different zones; synchronous replication; automatic failover in ~60s
- Read replicas: Cross-region read replicas supported (asynchronous)
- Backups: Automated daily backups + transaction log backups for PITR (Point-In-Time Recovery)
- **Cloud SQL Auth Proxy**: Secure connection method; encrypts traffic and uses IAM for authentication; avoids opening firewall ports

#### Cloud SQL vs Self-Managed SQL

- Cloud SQL handles backups, patching, failover, monitoring
- You manage: schema design, query optimization, index tuning, data modeling

---

### Cloud Spanner

- Fully managed, **horizontally scalable, globally distributed relational database** with strong ACID consistency
- Unique capability: **global transactions with strong consistency** at any scale
- Use cases: Financial systems, global gaming leaderboards, inventory systems, any globally distributed OLTP requiring ACID
- Scale: From 1 to thousands of nodes; automatically shards data (no manual sharding)
- Latency: Single-region ~1-5ms, multi-region ~10-100ms
- 99.999% SLA for multi-region (5 nines)
- Pricing: Per **processing unit** (1 node = 1000 processing units) + storage; minimum 100 PUs per instance
- SQL support: ANSI SQL with extensions
- Interleaved tables: Parent-child table relationships stored physically together for efficient lookups

#### When to Choose Spanner vs Cloud SQL

| Dimension | Cloud SQL | Cloud Spanner |
|-----------|-----------|--------------|
| Scale | Up to ~100 vCPU, 64 TB | Unlimited horizontal |
| Global | No (regional only) | Yes (multi-region ACID) |
| Cost | Lower | Higher (10x+) |
| Migration | Standard SQL; easier migration | Requires Spanner dialect |

---

### Cloud Bigtable

- Fully managed **NoSQL wide-column store** (similar to Apache HBase / Apache Cassandra)
- Designed for: massive scale, low latency (single-digit ms), time-series, IoT, analytics
- Scale: Petabyte-scale; linear horizontal scaling via adding nodes
- Data model: rows identified by a **row key**; columns organized into **column families**; each cell has a timestamp
- Access: HBase API, application API; **not SQL-based**
- No transactions (no ACID); no joins; no indexes beyond row key
- Ideal workloads:
  - Time-series data (metrics, events, IoT sensor data)
  - Financial tick data
  - User behavioral data at scale
  - Machine learning feature store
- Key design principle: **Row key design is critical** — all queries scan by row key; inefficient row keys cause performance problems (hotspots)
- Minimum recommended nodes: 3 for production (SLA requires 3+)

---

### Firestore (Datastore)

- Fully managed **serverless NoSQL document database**
- Two modes:
  - **Firestore (Native mode)**: Modern, supports real-time updates, offline sync, collection/document model
  - **Datastore mode**: Legacy Datastore API compatibility (for existing Datastore users)
- Document data model: Documents organized into **Collections**; documents contain fields of various types
- ACID transactions across documents (limited to 500 writes per transaction)
- Real-time listeners: Client SDKs (web, mobile) receive live data updates
- Offline support: Mobile/web SDKs cache data locally for offline use
- Scale: Automatic; serverless; no capacity planning
- Pricing: Per document read/write/delete + storage
- Best for: Mobile/web backends, user profiles, catalog data, chat apps, any app needing real-time sync

#### Firestore vs Bigtable

| Dimension | Firestore | Bigtable |
|-----------|-----------|---------|
| Data model | Documents/Collections | Wide-column rows |
| SQL | No (but structured queries) | No |
| Real-time | Yes (listeners) | No |
| Transactions | Yes (limited) | No |
| Scale | Auto-scaling | Manual node scaling |
| Latency | Single-digit ms | Single-digit ms |
| Best for | App backends | Analytics, time-series, HBase |

---

### Memorystore

- Fully managed **in-memory data store** service
- Supports: **Redis** and **Memcached**
- Use cases: Caching, session storage, pub/sub messaging, leaderboards, rate limiting
- Redis features on Memorystore: persistence (RDB/AOF snapshots), replication, failover, Redis Streams, sorted sets
- Memcached on Memorystore: Simple key-value caching, no persistence

| Engine | Persistence | Replication | Use Case |
|--------|------------|-------------|---------|
| **Redis** | Yes (configurable) | Yes | Sessions, leaderboards, complex data structures |
| **Memcached** | No | No | Simple caching, shared object cache |

- High availability: Redis with replica; failover in ~1 minute
- Not accessible from outside VPC — must connect from GCP resources in the same VPC

---

### BigQuery (Analytics)

- Serverless **data warehouse** for OLAP (Online Analytical Processing)
- Not an operational database — designed for large-scale analytical queries, not OLTP
- Scale: Petabytes; serverless; automatic scaling
- Pricing: Storage per GB + queries per TB scanned (on-demand) or flat-rate reservations
- No indexes — queries scan data in columnar format; partitioning and clustering reduce scan costs
- **Partitioned tables**: Divide table by date/timestamp column — queries on a date range only scan relevant partitions
- **Clustered tables**: Physically sort data by one or more columns — reduces bytes scanned for filtered queries
- See [data-solutions-deploy.md](../domain-3-deploy-and-implement/data-solutions-deploy.md) for BigQuery deployment patterns

---

## Storage Decision Matrix

| Requirement | Service |
|-------------|---------|
| Unstructured files (images, videos, backups) | Cloud Storage |
| Relational OLTP, < few TB, single region | Cloud SQL |
| Relational OLTP, global, massive scale, ACID | Cloud Spanner |
| Wide-column, petabyte-scale, time-series, IoT | Bigtable |
| Document store, mobile/web real-time, offline | Firestore |
| Analytics, data warehouse, SQL on petabytes | BigQuery |
| In-memory cache, sessions | Memorystore (Redis/Memcached) |

---

## When to Use

- **Cloud Storage**: Any unstructured data: backups, media, data lake, static website hosting, export destination for other services
- **Cloud SQL**: Traditional relational workloads where you're moving an existing MySQL/PostgreSQL/SQL Server workload to GCP
- **Cloud Spanner**: When you've outgrown Cloud SQL or need global ACID transactions (accept higher cost)
- **Bigtable**: When you have > 1 TB of structured data needing low-latency reads with simple access patterns (no complex queries)
- **Firestore**: Mobile/web app backends needing real-time sync; user profiles; catalog data
- **Memorystore**: Any caching layer needed in front of a database or for session management

---

## When NOT to Use

- **Cloud Storage**: Not for structured data needing SQL queries (use BigQuery or Cloud SQL)
- **Cloud SQL**: Not for global multi-region writes with strong consistency (use Spanner); not for petabyte-scale analytics (use BigQuery)
- **Cloud Spanner**: Not for small-scale or single-region workloads where Cloud SQL is sufficient (cost gap is significant)
- **Bigtable**: Not for complex queries with joins/aggregations (use BigQuery); not for small datasets (minimum 3 nodes = significant cost)
- **Firestore**: Not for analytics or large-scale batch processing; not for very high write throughput with complex data
- **Memorystore**: Not for durable storage — it's a cache; Redis with persistence is still not a replacement for primary storage

---

## Related Services / Concepts

- **Data Solutions Deploy**: Deploying these services — see [data-solutions-deploy.md](../domain-3-deploy-and-implement/data-solutions-deploy.md)
- **Managing Storage**: Lifecycle policies, versioning — see [managing-storage.md](../domain-4-ensure-success/managing-storage.md)
- **Data Security**: CMEK for storage — see [data-security.md](../domain-5-configure-access-and-security/data-security.md)
- **Network Planning**: Private access to databases — see [network-planning.md](network-planning.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Cloud Storage classes have no latency difference**: All classes serve data with the same speed. The difference is only in cost and minimum storage duration. A common distractor is suggesting Coldline data is slower to access.

2. **Bigtable minimum 3 nodes for SLA**: For production SLA, Bigtable requires at least 3 nodes. Below 3 nodes, there's no SLA guarantee.

3. **Firestore vs Datastore mode**: They are different modes of the same underlying service. A project can only use one mode — you choose at database creation time. Can't switch later.

4. **Spanner minimum cost**: Even the smallest Spanner instance (100 processing units = 0.1 node) has a significant cost vs Cloud SQL. Don't recommend Spanner for small workloads.

5. **Cloud SQL is regional, not global**: Cloud SQL can have cross-region read replicas, but there's only one writable primary. Cross-region writes require Spanner.

6. **BigQuery is not a transactional database**: No row-level updates (historically); designed for batch/analytical queries. Using BigQuery for OLTP is wrong.

7. **Memorystore is VPC-only**: Cannot be accessed from outside the VPC without a VPC connector. Always requires the app to be in the same VPC.

8. **Cloud Storage uniform bucket-level access**: Once enabled, legacy ACLs are disabled permanently for that bucket. Cannot be reverted.

### Keywords
- Cloud Storage classes, lifecycle policy, signed URL, uniform bucket-level access, Cloud SQL HA, Cloud SQL Auth Proxy, Cloud Spanner processing units, Bigtable row key, Firestore native mode, Firestore Datastore mode, Memorystore Redis, BigQuery partitioning, BigQuery clustering

---

## Source

- https://cloud.google.com/storage/docs/storage-classes
- https://cloud.google.com/sql/docs/mysql/high-availability
- https://cloud.google.com/spanner/docs/overview
- https://cloud.google.com/bigtable/docs/overview
- https://cloud.google.com/firestore/docs/concepts
- https://cloud.google.com/memorystore/docs/redis
