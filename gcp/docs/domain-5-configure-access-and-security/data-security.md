# Data Security: CMEK, Cloud KMS, Secret Manager, Cloud DLP

## Overview

Data security in GCP encompasses encryption key management (Cloud KMS, CMEK), secure storage of sensitive configuration and credentials (Secret Manager), and data classification and de-identification (Cloud DLP). These services protect data at rest and in transit and are tested in the ACE exam.

---

## Key Concepts

### Encryption in GCP

#### Default Encryption

- GCP encrypts **all data at rest by default** using Google-managed encryption keys
- This is automatic, transparent, and requires no configuration
- Keys are managed by Google; Google can access them (government requests, Google employees — controlled by internal policies)

#### Encryption Key Management Options

| Option | Who Manages Keys | Key Storage | Use Case |
|--------|-----------------|-------------|---------|
| **Google-managed keys** | Google | Managed internally | Default; minimal admin overhead |
| **CMEK (Customer-managed)** | Customer, via Cloud KMS | Cloud KMS | Compliance, audit requirements |
| **CSEK (Customer-supplied)** | Customer | Customer's own systems | Maximum customer control; keys never stored by Google |
| **Cloud EKM** | Customer, via external KMS | External (Thales, Fortanix, etc.) | Regulated industries, data sovereignty |

---

### Cloud Key Management Service (Cloud KMS)

#### Overview

- Fully managed service for generating, storing, and using cryptographic keys
- Keys never leave Cloud KMS in plaintext — operations (encrypt/decrypt) happen within KMS
- Supports: Symmetric (AES-256), Asymmetric (RSA, EC), MAC operations
- FIPS 140-2 Level 1 compliance for standard; Level 3 for Cloud HSM

#### Key Hierarchy

```
Key Ring
└── CryptoKey
    └── CryptoKey Version (contains the actual key material)
```

- **Key Ring**: Organizational container; tied to a specific location (regional or global)
- **CryptoKey**: A logical key with a rotation policy; may have multiple versions
- **CryptoKey Version**: The actual key material; can be ENABLED, DISABLED, SCHEDULED_FOR_DESTRUCTION, DESTROYED

#### Key Rotation

- **Automatic rotation**: Set a rotation period on a CryptoKey (e.g., every 90 days)
- A new primary version is created; old versions remain to decrypt existing data
- Old versions are NOT automatically destroyed — you must explicitly schedule them for destruction
- **Manual rotation**: Create a new version and set it as primary

#### Key Destruction

- Schedule a key version for destruction: 24-hour pending period (default; configurable up to 120 days)
- During pending period: Can cancel destruction
- After destruction: Data encrypted with that version is permanently inaccessible
- **Critical**: Destroying the key version destroys all data encrypted with it (if you have no other copies)

#### Cloud HSM

- Hardware Security Module-backed key storage
- Keys never exist in software; cryptographic operations occur in hardware
- FIPS 140-2 Level 3 certification
- Higher cost than standard Cloud KMS; used for highest-sensitivity workloads

#### IAM for Cloud KMS

| Role | Permissions |
|------|-------------|
| `roles/cloudkms.admin` | Manage key rings and keys; cannot use keys for encryption/decryption |
| `roles/cloudkms.cryptoKeyEncrypterDecrypter` | Encrypt and decrypt data |
| `roles/cloudkms.cryptoKeyEncrypter` | Encrypt only (write) |
| `roles/cloudkms.cryptoKeyDecrypter` | Decrypt only (read) |
| `roles/cloudkms.viewer` | View key metadata |

- **Separation**: Admin role deliberately separated from encrypt/decrypt role — admins cannot use keys, key users cannot manage them

---

### Customer-Managed Encryption Keys (CMEK)

#### What CMEK Does

- Instead of Google-managed keys, you specify a Cloud KMS key to encrypt a specific resource
- Google uses your KMS key (via the service's service account) when encrypting/decrypting data
- You retain control: **If you delete or disable the KMS key, the data becomes inaccessible**

#### Services Supporting CMEK

- Cloud Storage (bucket-level or object-level)
- BigQuery (dataset-level)
- Compute Engine (persistent disk encryption)
- Cloud SQL (instance-level)
- GKE (etcd encryption, persistent volumes)
- Cloud Logging (log buckets)
- Pub/Sub
- Artifact Registry
- Many others

#### CMEK Configuration Pattern

1. Create a Cloud KMS key in the same region as the resource (usually required)
2. Grant the service's service account `roles/cloudkms.cryptoKeyEncrypterDecrypter` on the key
3. Specify the key when creating the resource

#### Cloud Storage CMEK

- **Bucket-level CMEK**: Set a default KMS key for all new objects in the bucket
- **Object-level CMEK**: Override with a different key per object
- Existing objects are NOT automatically re-encrypted when you set a new bucket CMEK key
- To re-encrypt existing objects with CMEK: Use `gsutil rewrite -k`

#### CMEK vs CSEK (Customer-Supplied)

| Aspect | CMEK | CSEK |
|--------|------|------|
| Key storage | Cloud KMS | Customer's systems |
| Key management | Cloud KMS features (rotation, audit) | Customer manages entirely |
| Risk | KMS must be available | If you lose the key, data is gone |
| Auditability | Cloud Audit Logs (KMS API calls) | No GCP audit trail for key usage |
| Best for | Most regulated workloads | Maximum sovereignty requirements |

---

### Secret Manager

Securely stores and manages API keys, passwords, certificates, tokens, and other sensitive data.

#### Core Concepts

| Concept | Description |
|---------|-------------|
| **Secret** | Named container for secret data |
| **Secret Version** | Immutable payload associated with a secret (each update creates a new version) |
| **State** | ENABLED, DISABLED, or DESTROYED |

#### Secret Access Patterns

- **Application code**: Uses Secret Manager API (client libraries or REST) to fetch secret value at runtime
- **Cloud Run**: Inject as environment variable or volume mount (value fetched from Secret Manager at startup)
- **Cloud Functions**: Same as Cloud Run
- **GKE**: Fetch via code using Workload Identity, or use External Secrets Operator / Config Connector

#### Accessing Secrets (IAM)

| Role | Access |
|------|--------|
| `roles/secretmanager.secretAccessor` | Read secret version values (most workloads need this) |
| `roles/secretmanager.secretVersionManager` | Add/enable/disable/destroy versions |
| `roles/secretmanager.admin` | Full control |

- Grant `secretAccessor` to service accounts that need to read secrets
- Grant at the secret level (not project level) for least privilege

#### Secret Versioning

- Every time you update a secret's value, a new version is created
- Old versions are retained unless explicitly disabled/destroyed
- Applications reference secrets by:
  - **Latest version**: `projects/PROJECT/secrets/MY_SECRET/versions/latest` — always current
  - **Specific version**: `projects/PROJECT/secrets/MY_SECRET/versions/3` — pinned
- Best practice: Use `latest` for most cases; pin specific version for compliance/rollback scenarios

#### Secret Rotation

- Cloud Secret Manager can send Pub/Sub notifications when secrets near expiration
- Configure a Cloud Function to auto-rotate credentials and update the secret
- Automatic rotation isn't built-in — requires custom implementation

#### CMEK for Secret Manager

- Secret payloads can be encrypted with a customer-managed KMS key
- Additional protection if GCP's default encryption isn't sufficient for compliance

---

### Cloud Data Loss Prevention (Cloud DLP)

#### What It Does

- Discovers, classifies, and de-identifies sensitive data across GCP and external systems
- Supports: 150+ built-in InfoTypes (patterns for PII, credentials, sensitive data)
- Works on: Cloud Storage objects, BigQuery tables, Datastore entities, strings, binary data

#### Key Capabilities

| Capability | Description |
|------------|-------------|
| **Inspection** | Scan data for sensitive content (credit cards, SSNs, API keys) |
| **De-identification** | Remove or obfuscate sensitive data (masking, pseudonymization, tokenization, encryption) |
| **Re-identification** | Reverse de-identification if using cryptographic methods |
| **Risk analysis** | Analyze data for re-identification risk (k-anonymity, l-diversity) |

#### De-identification Methods

| Method | Description | Reversible |
|--------|-------------|-----------|
| **Masking** | Replace characters with `*` or fixed character | No |
| **Bucketing** | Replace numeric values with ranges | No |
| **Tokenization (pseudonymization)** | Replace with a token that can be reversed with the key | Yes (with key) |
| **Format-preserving encryption** | Encrypt maintaining format (e.g., CC number → same format) | Yes (with key) |
| **Date shifting** | Shift dates by random amount (preserves relative time) | No |
| **Character replacement** | Replace characters based on rules | Configurable |

#### Common Use Cases

- **Scan GCS/BigQuery for accidentally stored secrets or PII**: Run DLP inspection job
- **De-identify data before sharing with analysts**: Remove SSNs, credit cards from a dataset
- **Protect data in motion**: DLP can inspect text in real-time via API

---

## When to Use

- **Default encryption**: Always (automatic) for all data at rest
- **CMEK**: When you need to prove key custody, must comply with regulations requiring customer-managed keys, or need to be able to "destroy" data by destroying the key
- **Cloud HSM**: For FIPS 140-2 Level 3 requirements (PCI, HIPAA specific requirements)
- **Secret Manager**: For all passwords, API keys, certificates, and other secrets — never environment variables or code
- **Cloud DLP**: For compliance scanning (GDPR, HIPAA), data catalog quality, and before sharing sensitive datasets

---

## When NOT to Use

- **CSEK**: Unless you have very specific data sovereignty requirements; CMEK with Cloud KMS/Cloud HSM covers most needs
- **Secret Manager for very high-throughput secrets**: Each API call to retrieve a secret is a separate HTTP call with latency; cache secrets for high-throughput scenarios (cache for reasonable TTL)
- **Cloud DLP on all data continuously**: Can be expensive; run targeted scans on specific buckets/tables

---

## Related Services / Concepts

- **VPC Security**: VPC Service Controls protect access paths to encrypted resources — see [vpc-security.md](vpc-security.md)
- **IAM Advanced**: KMS roles — see [iam-advanced.md](iam-advanced.md)
- **Service Accounts**: Service accounts access secrets via Secret Manager — see [service-accounts.md](service-accounts.md)
- **Managing Storage**: CMEK for Cloud Storage buckets — see [managing-storage.md](../domain-4-ensure-success/managing-storage.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Destroying a KMS key = destroying the data**: If you delete/destroy the only KMS key version used to encrypt a Cloud Storage bucket's data, that data is permanently unrecoverable. This is both the feature (right to erasure) and the risk.

2. **CMEK doesn't encrypt everything by default**: When you set a bucket-level CMEK key, only NEW objects get encrypted with it. Existing objects remain encrypted with the old key. Re-encrypt existing objects explicitly.

3. **KMS admin can't encrypt/decrypt**: Separation of duties is built-in. `cloudkms.admin` manages keys but cannot use them for operations. The service using the key needs `cryptoKeyEncrypterDecrypter`.

4. **Secret Manager vs environment variables**: The exam will test whether you know to use Secret Manager instead of environment variables for secrets. Environment variables are visible in console, logs, and by anyone with deployment access.

5. **Secret versions are immutable**: You cannot edit a secret's value — you create a new version. Old versions remain (and are still billed) unless explicitly disabled/destroyed.

6. **Cloud DLP is not a firewall**: DLP scans and de-identifies data; it doesn't block access. It's a discovery and transformation tool.

7. **Key rotation doesn't destroy old versions**: When a key auto-rotates, the old version remains enabled (for decrypting existing data). You must explicitly schedule old versions for destruction when you're confident all data has been re-encrypted.

8. **Secret Manager CMEK requires same region**: The KMS key and the Secret Manager must be in compatible regions. Global secrets can use global KMS key rings.

### Keywords
- CMEK, CSEK, Cloud KMS, CryptoKey, key rotation, pending destruction, Cloud HSM, FIPS 140-2, Secret Manager, secret version, `secretAccessor`, de-identification, tokenization, format-preserving encryption, Cloud DLP, InfoType, Google-managed key

---

## Source

- https://cloud.google.com/kms/docs/overview
- https://cloud.google.com/storage/docs/encryption/customer-managed-keys
- https://cloud.google.com/secret-manager/docs/overview
- https://cloud.google.com/dlp/docs/overview
