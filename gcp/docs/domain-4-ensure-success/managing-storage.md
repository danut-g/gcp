# Managing Storage: Lifecycle Policies, Versioning, Retention, Transfer

## Overview

Managing GCP storage services involves configuring data lifecycle policies, enabling versioning, setting retention locks for compliance, and transferring data into or within GCP. These operational tasks are tested in the ACE exam, particularly for Cloud Storage.

---

## Key Concepts

### Cloud Storage Lifecycle Management

#### Lifecycle Configuration

- Lifecycle rules define actions triggered by conditions on objects in a bucket
- Rules are evaluated by GCP daily (not real-time) — changes take up to 24 hours to take effect
- A rule consists of: one action + one or more conditions

#### Actions

| Action | Description |
|--------|-------------|
| **SetStorageClass** | Transition to a different storage class |
| **Delete** | Delete the object (or non-current version) |
| **AbortIncompleteMultipartUpload** | Delete incomplete uploads older than N days |

#### Conditions

| Condition | Description |
|-----------|-------------|
| `age` | Object age in days |
| `createdBefore` | Object created before a specific date |
| `isLive` | Whether the object is the current version (true) or a non-current version (false) |
| `matchesStorageClass` | Current storage class of the object |
| `numNewerVersions` | Number of newer versions of the object |
| `daysSinceNoncurrentTime` | Days since version became noncurrent |
| `matchesPrefix` / `matchesSuffix` | Object name filtering |

#### Common Lifecycle Patterns

**Cost-optimized archival:**
```
Age 30 days → SetStorageClass: NEARLINE
Age 90 days → SetStorageClass: COLDLINE
Age 365 days → SetStorageClass: ARCHIVE
```

**Clean up old versions:**
```
isLive=false AND age=90 days → Delete (removes non-current versions after 90 days)
```

**Incomplete upload cleanup:**
```
AbortIncompleteMultipartUpload if older than 7 days → prevents billing for stalled uploads
```

---

### Cloud Storage Versioning

- When enabled: Overwriting or deleting an object creates a **non-current version** instead of permanent deletion
- **Current (live)** version: The latest version; serves all requests by default
- **Non-current** version: Previous versions; retained until explicitly deleted or removed by lifecycle rule
- Accessing a specific version: Use `?generation=GENERATION_NUMBER` in the API
- **Soft delete**: Even without versioning, objects enter a soft-delete state for 7 days (configurable) before permanent deletion
- **Deleting with versioning enabled**: Creates a "delete marker" (tombstone); object appears deleted but versions remain
- **Permanently deleting a specific version**: Requires specifying the generation number

#### Versioning Cost Implications

- Each non-current version is billed at the same rate as the storage class it was in when it became non-current
- Archive old versions using lifecycle rules to reduce costs (transition to cheaper class or delete)

---

### Retention Policies and Locks

#### Retention Policy

- Prevents objects from being deleted or modified before a minimum retention duration
- Set on a bucket (applies to all objects)
- If a retention policy is set, objects cannot be deleted until `creationTime + retentionPeriod`
- Modification to object data is also blocked (not metadata-only changes)

#### Locked Retention Policy

- Permanently lock a retention policy: Cannot be shortened or removed (even by bucket owner)
- Use case: Regulatory compliance (SEC Rule 17a-4, HIPAA, FINRA) requiring WORM storage
- Locking is irreversible — cannot unlock a locked retention policy
- **Important**: Test your retention policy thoroughly before locking

#### Object Holds

- Per-object level hold (instead of bucket-level retention policy)
- **Event-based holds**: Object is held; hold is manually released when an event occurs (e.g., legal hold released after case closes)
- **Temporary holds**: Object is held; hold is released manually or when retention expires
- Objects with holds cannot be deleted or overwritten until hold is released

---

### Cloud Storage Transfer Services

#### Storage Transfer Service

- Transfers data **into** GCP from:
  - AWS S3 buckets
  - Azure Blob Storage
  - HTTP/HTTPS sources
  - Other GCS buckets (cross-region/cross-project)
  - On-premises via Transfer Service for On-Premises Data (uses agents)
- Features: Scheduled transfers, incremental sync, filtering by prefix/suffix/last modified time
- Transfer jobs are managed through the console or API
- **Pricing**: Free to use the service; you pay for GCS storage and egress from source

#### gsutil rsync

- Synchronize a local directory or GCS bucket to/from another GCS bucket
- More appropriate for one-time or ad-hoc transfers
- Use `gsutil -m rsync -r` for parallel, recursive sync
- Not suitable for large-scale or scheduled production transfers (use Transfer Service instead)

#### Transfer Appliance

- Physical hardware appliance for large-scale offline data transfer (petabyte-scale)
- Ship data to Google data center on the appliance
- For migrations where network bandwidth is the bottleneck
- Use when estimated transfer time over network would be > 1 week

#### BigQuery Data Transfer Service

- Scheduled data transfers from SaaS sources into BigQuery:
  - Google Ads, YouTube, Google Play, Campaign Manager
  - Amazon S3 to BigQuery
  - Teradata to BigQuery
- Separate from Storage Transfer Service; specifically for BigQuery destinations

---

### Cloud SQL Management

#### Backup and Recovery

- **Automated backups**: Daily snapshots; retained for 7 days by default (configurable up to 365 days)
- **On-demand backups**: Manual backups; retained until explicitly deleted
- **Point-in-Time Recovery (PITR)**:
  - MySQL: Requires binary logging enabled
  - PostgreSQL: Uses WAL archiving; enabled with PITR setting
  - Can recover to any point within the backup retention window
  - Recovery creates a new instance (not in-place)
- **Export to Cloud Storage**: Export data as SQL dump or CSV to a GCS bucket; useful for cross-project data sharing or long-term archival beyond backup retention
- Backups are stored in a Google-managed location; cannot directly access backup files

#### Maintenance Windows

- Configure preferred day/time for Cloud SQL maintenance (patches, minor updates)
- If no window is set, GCP may apply maintenance any time

---

### Bigtable Operations

#### Node Scaling

- Add or remove nodes without downtime
- After node change, performance takes ~20 minutes to stabilize as data rebalances
- Monitor utilization in Cloud Monitoring: `bigtable.googleapis.com/cluster/cpu_load`
- Target: <70% CPU load for most workloads

#### Data Export/Import

- Export to Cloud Storage (Avro or Parquet format) using a Dataflow pipeline or Cloud Bigtable Export service
- Import from Cloud Storage using Dataflow
- **Bigtable doesn't have a built-in export button in the console** — requires Dataflow or HBase jobs

---

### Firestore Operations

#### Exports and Imports

- Export Firestore data to Cloud Storage: `gcloud firestore export gs://BUCKET/EXPORT`
- Import from GCS: `gcloud firestore import gs://BUCKET/EXPORT`
- Exports are consistent at the time the export starts
- Exports to GCS can be loaded into BigQuery for analytics

#### Backup (Managed Backups)

- Firestore supports scheduled and on-demand backups
- Backups stored within Firestore's managed infrastructure
- Retention: configurable (up to 14 weeks)
- Restore creates a new database (not in-place)

---

## When to Use

- **Lifecycle rules**: Always for any bucket with objects that age over time; prevents runaway storage costs
- **Versioning**: For buckets with critical objects where accidental deletion/overwrite would be catastrophic
- **Locked retention policies**: For compliance workloads requiring WORM
- **Storage Transfer Service**: For large-scale, scheduled, or cross-cloud data migrations
- **Transfer Appliance**: When network transfer of petabyte-scale data would take too long
- **Soft delete**: As a safety net against accidental deletion without full versioning overhead

---

## When NOT to Use

- **Lock a retention policy without testing**: Permanent and irreversible — validate the policy first
- **gsutil rsync for large-scale production transfers**: Use Transfer Service for scheduled/large-scale
- **Automated backups as the only backup for critical data**: Also export to GCS periodically for an independent backup copy

---

## Related Services / Concepts

- **Storage Planning**: Storage class selection — see [storage-planning.md](../domain-2-plan-and-configure/storage-planning.md)
- **Data Solutions Deploy**: Initial database deployment — see [data-solutions-deploy.md](../domain-3-deploy-and-implement/data-solutions-deploy.md)
- **Data Security**: CMEK for storage encryption — see [data-security.md](../domain-5-configure-access-and-security/data-security.md)
- **Logging**: GCS audit logs — see [logging.md](logging.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Lifecycle rules are not real-time**: They run daily. If you add a rule expecting immediate effect (e.g., to clean up old files), there may be up to 24 hours delay.

2. **Locking a retention policy is permanent**: Once locked, the policy cannot be shortened or removed. The exam may test whether you understand this is irreversible.

3. **Deleting with versioning creates a delete marker**: The object appears deleted to most clients but the versions still exist (and accrue storage charges). You must delete the versions explicitly to free space.

4. **numNewerVersions condition**: This condition is useful for keeping only the last N versions. Setting `numNewerVersions >= 3 → Delete` keeps the 3 most recent versions and deletes older ones.

5. **PITR recovery creates a new instance**: You cannot recover in-place from a PITR backup. A new Cloud SQL instance is created from the backup; you then redirect your application.

6. **Transfer Service is free to use**: You don't pay for the Transfer Service job itself. You pay for Cloud Storage operations (reads at source if cross-cloud, writes at GCS destination) and egress from the source.

7. **Bigtable HDD vs SSD for lifecycle**: Bigtable doesn't have lifecycle rules like Cloud Storage. Data management is done through column family GC policies (max versions, max age per cell).

8. **Incomplete multipart upload cleanup**: Large file uploads that are abandoned still incur billing for the uploaded parts. Always configure `AbortIncompleteMultipartUpload` lifecycle rule.

### Keywords
- Lifecycle rule, SetStorageClass, isLive condition, versioning, retention policy, locked retention policy, WORM, event-based hold, temporary hold, Storage Transfer Service, Transfer Appliance, PITR, binary logging, non-current version, delete marker, soft delete

---

## Source

- https://cloud.google.com/storage/docs/lifecycle
- https://cloud.google.com/storage/docs/object-versioning
- https://cloud.google.com/storage/docs/retention-policy
- https://cloud.google.com/storage-transfer/docs/overview
- https://cloud.google.com/sql/docs/mysql/backup-recovery/backups
