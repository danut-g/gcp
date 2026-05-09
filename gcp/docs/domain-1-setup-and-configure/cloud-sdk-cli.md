# Cloud SDK & CLI Tools: gcloud, gsutil, bq

## Overview

The **Google Cloud SDK** is the primary command-line interface for managing GCP resources. It includes `gcloud` (for most GCP services), `gsutil` (Cloud Storage operations), and `bq` (BigQuery operations). Understanding SDK configuration, authentication, and multi-project management is essential for both the exam and day-to-day operations.

---

## Key Concepts

### gcloud CLI

The primary tool for managing GCP resources from the command line.

#### Configuration Hierarchy

- **gcloud configurations** allow switching between different projects, accounts, and regions without re-authenticating
- Default configuration is named `default`
- Multiple named configurations can exist (e.g., `dev`, `prod`, `staging`)
- Active configuration is used for all commands unless overridden with `--configuration` flag
- Configuration stores: `project`, `account`, `compute/region`, `compute/zone`, `core/project`

#### Authentication Types

| Method | Use Case | Credential Type |
|--------|----------|-----------------|
| `gcloud auth login` | Human user (interactive) | OAuth 2.0 user credentials |
| `gcloud auth activate-service-account` | Service accounts (automation) | Service account key file |
| Application Default Credentials (ADC) | Application code | Auto-detected credentials |
| Workload Identity | GKE/Cloud Run workloads | Federated credentials (no key file) |

#### Application Default Credentials (ADC)

- ADC is the mechanism applications use to find credentials automatically
- Search order:
  1. `GOOGLE_APPLICATION_CREDENTIALS` environment variable (points to key file)
  2. `gcloud auth application-default login` — user credentials saved locally
  3. Attached service account (Compute Engine, GKE, Cloud Run metadata server)
  4. Cloud Shell credentials
- The metadata server provides credentials to VMs/containers automatically — no key files needed on GCE

#### Key gcloud Concepts

- **Project context**: Commands use the active configuration's project by default; override with `--project=PROJECT_ID`
- **Region/zone context**: `compute/region` and `compute/zone` are set per configuration; always override with `--region` or `--zone` flags for explicit control
- **Impersonation**: `--impersonate-service-account=SA_EMAIL` runs commands as a service account (requires `roles/iam.serviceAccountTokenCreator`)
- **Verbosity flags**: `--verbosity=debug` for troubleshooting; `--format=json/yaml/csv` for output format
- **Quiet flag**: `--quiet` or `-q` suppresses interactive prompts (useful in scripts)
- **Filter and format**: `--filter` uses gcloud's filtering syntax; `--format` controls output; both are powerful for scripting

#### gcloud Components

- Core components: `gcloud`, `gsutil`, `bq`
- Optional components: `kubectl`, `app-engine-go`, `cloud-sql-proxy`, `minikube`, etc.
- Manage with `gcloud components install/update/remove`
- Cloud Shell pre-installs all common components

---

### gsutil

Tool for interacting with Cloud Storage. **Note**: Google is migrating gsutil functionality to `gcloud storage` commands, but gsutil remains valid for the exam.

#### Key gsutil Concepts

- **URI format**: `gs://bucket-name/object-name`
- **Parallelism**: Uses parallel composite uploads/downloads by default for large files
- **Parallelism control**: `-m` flag enables parallel multi-threaded operations (e.g., `gsutil -m cp`)
- **Resumable uploads**: Automatically used for files > 8 MB
- **ACL vs IAM**: gsutil can manage both legacy ACLs and IAM policies on buckets/objects
- **Signed URLs**: `gsutil signurl` generates time-limited, pre-authenticated URLs for sharing objects without requiring GCP credentials

#### Important gsutil Commands (conceptual)

- `gsutil cp` — copy objects (use `-r` for recursive, `-m` for parallel)
- `gsutil rsync` — synchronize directories (like `aws s3 sync`)
- `gsutil mv` — move/rename objects
- `gsutil rm` — delete objects
- `gsutil ls -l` — list with sizes and metadata
- `gsutil du` — disk usage
- `gsutil acl` — manage ACLs
- `gsutil iam` — manage IAM policies on buckets

---

### bq (BigQuery CLI)

Tool for BigQuery operations: running queries, managing datasets/tables, and loading data.

#### Key bq Concepts

- **Dataset**: Top-level container for BigQuery tables, within a project
- **Table**: Contains the actual data with schema
- `bq query` — runs a SQL query; use `--use_legacy_sql=false` for standard SQL
- `bq load` — loads data from GCS into a BigQuery table
- `bq mk` — creates datasets, tables, views
- `bq extract` — exports table data to GCS
- `bq show` — displays metadata about a dataset or table
- Default project is taken from gcloud configuration

---

### Cloud Shell

- Browser-based shell environment with gcloud pre-installed
- Provides 5 GB of persistent storage in `$HOME` on Cloud Shell VM
- Ephemeral VM (automatically provisioned per session, idle timeout ~20 minutes)
- Uses your Google account for authentication automatically
- Pre-authenticated — no need to run `gcloud auth login` in Cloud Shell
- Free to use; billed only for Boost mode (higher-resource VM)
- **Home directory persists** between sessions, but the VM itself is ephemeral

---

### gcloud Configurations in Multi-Environment Management

```
Scenario: Dev/Staging/Prod environments in different projects

gcloud config configurations create dev
gcloud config set project my-dev-project
gcloud config set compute/zone us-central1-a

gcloud config configurations create prod
gcloud config set project my-prod-project
gcloud config set compute/zone us-east1-b

# Switch between them
gcloud config configurations activate prod
```

This avoids accidentally running destructive commands in the wrong project.

---

## When to Use

- **gcloud configurations** when working across multiple projects or accounts regularly
- **Service account authentication** (`activate-service-account`) for CI/CD pipelines and scripts that run unattended
- **ADC** for application code — let the SDK find credentials automatically based on environment
- **Workload Identity** instead of service account keys for GKE workloads — see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)
- **gsutil -m** for bulk transfers involving many files or large files (parallel operations)
- **Cloud Shell** for ad-hoc management tasks without local SDK installation

---

## When NOT to Use

- Do **not** use service account key files in Cloud Shell — use ADC/user credentials instead
- Do **not** hardcode project IDs or zone configurations in scripts — use configuration or flags for portability
- Do **not** use `gcloud auth login` in automated scripts — use service accounts or Workload Identity
- Do **not** rely on default project/region in production scripts without explicit `--project` and `--region` flags — risky if configuration changes

---

## Related Services / Concepts

- **Service Accounts**: Authentication for automated tools — see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)
- **IAM**: Permissions required for each CLI operation — see [iam-overview.md](iam-overview.md)
- **Cloud Storage**: Primary target for gsutil — see [storage-planning.md](../domain-2-plan-and-configure/storage-planning.md)
- **Terraform**: Alternative to gcloud for IaC — see [iac-deployment-manager.md](../domain-3-deploy-and-implement/iac-deployment-manager.md)

---

## Exam-Relevant Notes

### Common Traps

1. **gcloud vs gsutil for Cloud Storage**: `gcloud storage` is the newer preferred CLI for Cloud Storage (replacing gsutil). Exam questions may use either. Know that `gs://` URI format applies to both.

2. **ADC search order matters**: If `GOOGLE_APPLICATION_CREDENTIALS` is set to a wrong/outdated key file, it takes precedence over the metadata server credentials. This is a common misconfiguration source.

3. **Cloud Shell is ephemeral, but Home is not**: The 5 GB home directory persists. The VM instance itself does not. Don't confuse the two.

4. **`gcloud auth login` vs `gcloud auth application-default login`**: `auth login` is for gcloud CLI operations. `auth application-default login` sets credentials used by application libraries (ADC). You often need both in a development environment.

5. **Configurations are local**: gcloud configurations are stored on the local machine (`~/.config/gcloud/`). They are not synced across machines or stored in GCP.

6. **`--impersonate-service-account` requires permissions**: You need `roles/iam.serviceAccountTokenCreator` on the target service account to impersonate it via CLI.

7. **`gsutil -m` parallel flag**: The exam may ask about optimizing large-scale transfers. `-m` enables parallel operations — critical for performance with many small files.

8. **bq `--use_legacy_sql=false`**: Always required for standard SQL syntax in bq CLI. Legacy SQL is the old BigQuery dialect.

### Keywords
- gcloud configuration, ADC, application default credentials, service account key, gsutil -m, Cloud Shell ephemeral, bq legacy SQL, gcloud auth login, workload identity, impersonation, --project flag

---

## Source

- https://cloud.google.com/sdk/gcloud
- https://cloud.google.com/storage/docs/gsutil
- https://cloud.google.com/bigquery/docs/bq-command-line-tool
- https://cloud.google.com/shell/docs
