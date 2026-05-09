# Infrastructure as Code: Deployment Manager and Terraform on GCP

## Overview

Infrastructure as Code (IaC) allows GCP resources to be defined declaratively and managed through version control. GCP offers **Cloud Deployment Manager** (native GCP IaC) while **Terraform** (by HashiCorp) is the industry standard third-party tool widely used with GCP. Understanding both, and when to choose each, is tested in the ACE exam.

---

## Key Concepts

### Cloud Deployment Manager

Google's native IaC tool for GCP resources.

#### Core Concepts

| Concept | Description |
|---------|-------------|
| **Configuration** | YAML file(s) describing resources to create |
| **Template** | Jinja2 or Python template for reusable configurations |
| **Deployment** | Instance of a configuration; represents a set of resources |
| **Manifest** | Immutable record of what was deployed (version snapshot) |
| **Preview** | Shows what changes will be made without applying them |

#### Configuration Structure (YAML)

```yaml
resources:
- name: my-vm
  type: compute.v1.instance
  properties:
    zone: us-central1-a
    machineType: zones/us-central1-a/machineTypes/n1-standard-1
    disks:
    - deviceName: boot
      type: PERSISTENT
      boot: true
      initializeParams:
        sourceImage: projects/debian-cloud/global/images/family/debian-11
    networkInterfaces:
    - network: global/networks/default
```

- `type` uses the GCP API resource type (e.g., `compute.v1.instance`)
- `properties` maps to the REST API request body for that resource type

#### Templates (Jinja2 or Python)

- **Jinja2 templates**: YAML with Jinja2 templating syntax; simpler
- **Python templates**: Full Python scripts that generate YAML configurations; more powerful for complex logic
- Templates receive parameters and can be reused across configurations
- Template schemas: Define input parameters with types and defaults

#### Deployment Manager Operations

- `gcloud deployment-manager deployments create NAME --config config.yaml`
- `gcloud deployment-manager deployments update NAME --config config.yaml`
- `gcloud deployment-manager deployments delete NAME`
- `--preview` flag: Show planned changes before applying
- On update: Deployment Manager performs a diff and only modifies changed resources

#### Supported Resource Types

- All major GCP services: Compute Engine, GKE, Cloud SQL, Cloud Storage, IAM, Networking, etc.
- Uses GCP REST API types — anything the API supports can be managed
- Third-party resource types via **Type Providers**

#### Limitations of Deployment Manager

- GCP-only (not multi-cloud)
- Smaller community and ecosystem than Terraform
- State is managed internally by GCP (no external state file)
- Python/Jinja2 templates are less intuitive than Terraform HCL for many users
- Error messages can be less clear than Terraform

---

### Terraform on GCP

HashiCorp Terraform is the most widely used IaC tool for GCP in practice and is heavily referenced in GCP documentation and exams.

#### Key Terraform Concepts

| Concept | Description |
|---------|-------------|
| **Provider** | Plugin for interacting with a cloud/API (`google` provider) |
| **Resource** | A GCP resource to manage (`google_compute_instance`) |
| **Data source** | Read-only reference to existing resources |
| **Module** | Reusable group of resources |
| **State** | Terraform's record of managed infrastructure (`.tfstate`) |
| **Plan** | Show what changes will be made (`terraform plan`) |
| **Apply** | Execute the planned changes (`terraform apply`) |

#### The Google Provider

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = "my-project-id"
  region  = "us-central1"
}
```

- The `google` provider maps to most GCP services
- `google-beta` provider: Access to beta/preview features
- Authentication: Follows ADC — uses `GOOGLE_APPLICATION_CREDENTIALS`, `gcloud auth application-default login`, or Workload Identity on GCE/GKE

#### Terraform State Management on GCP

- Default: State stored locally in `terraform.tfstate`
- **Remote state in Cloud Storage**: Use GCS bucket as Terraform backend

```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state-bucket"
    prefix = "terraform/state"
  }
}
```

- **State locking**: GCS backend uses Cloud Storage object metadata for state locking — prevents concurrent modifications
- State file contains sensitive data (passwords, service account keys) — enable encryption and restrict access to the bucket

#### Terraform Workflow

```
terraform init     → Initialize providers and modules
terraform plan     → Show planned changes (diff)
terraform apply    → Apply changes
terraform destroy  → Destroy all managed resources
terraform fmt      → Format code
terraform validate → Syntax check
```

#### Remote Backend Benefits

- Enables team collaboration (shared state)
- State locking prevents concurrent applies
- Versioning on GCS bucket preserves history
- State not stored on developer laptops

---

### Config Connector

- Kubernetes-native way to manage GCP resources using Kubernetes manifests (YAML CRDs)
- Install as a GKE add-on
- Define GCP resources as Kubernetes resources; `kubectl apply` provisions/manages GCP resources
- Uses Workload Identity for authentication
- Good for teams that prefer Kubernetes-native tooling

---

### Comparison: Deployment Manager vs Terraform

| Dimension | Deployment Manager | Terraform |
|-----------|------------------|-----------|
| Native GCP tool | Yes | No (third-party) |
| Multi-cloud | No | Yes |
| Language | YAML + Jinja2/Python | HCL (HashiCorp Config Language) |
| State management | Managed by GCP | External (local or remote backend) |
| Community/ecosystem | Small | Very large |
| Modules/registry | Limited | Extensive (Terraform Registry) |
| Exam relevance | Mentioned, less common | Primary IaC tool in GCP context |
| GCP documentation preference | Legacy, less featured | Actively recommended |

---

### IaC Best Practices for GCP

- Store state remotely in a GCS bucket with versioning and access controls
- Use separate state files per environment (dev/staging/prod)
- Use Terraform modules for reusable patterns (VPC module, GKE module)
- Protect the state bucket: Enable versioning, restrict access to CI/CD service account + admins
- Encrypt state file at rest using CMEK on the GCS bucket
- Use `terraform plan` in CI/CD before `terraform apply` — review before executing
- Service account for Terraform: Create a dedicated SA with only the permissions needed for the infrastructure being managed
- Use `terraform import` to bring existing resources under Terraform management

---

## When to Use

- **Terraform**: For new GCP projects, teams with existing Terraform expertise, multi-cloud environments, or when using the GCP documentation as a guide
- **Deployment Manager**: For existing deployments that already use it; GCP-only environments where a native tool is preferred; integration with GCP's native tooling
- **Config Connector**: Teams already managing everything through Kubernetes who want to manage GCP resources the same way

---

## When NOT to Use

- **Deployment Manager**: Not for new projects unless there's a specific requirement for a native GCP tool
- **Terraform without remote state**: Don't use local state in team environments — concurrent apply conflicts will corrupt state
- **IaC for manually created resources without `import`**: Managing resources with IaC while other processes create/delete them out-of-band causes drift and state conflicts

---

## Related Services / Concepts

- **Cloud SDK**: `gcloud` is used for day-to-day operations; IaC for systematic provisioning — see [cloud-sdk-cli.md](../domain-1-setup-and-configure/cloud-sdk-cli.md)
- **Service Accounts**: Terraform/Deployment Manager authentication — see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)
- **IAM**: Permissions needed for IaC service accounts — see [iam-advanced.md](../domain-5-configure-access-and-security/iam-advanced.md)
- **Cloud Storage**: Terraform remote backend — see [storage-planning.md](../domain-2-plan-and-configure/storage-planning.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Deployment Manager state is managed by GCP**: Unlike Terraform, you don't manage a state file. GCP tracks the deployment internally. This is both simpler (no state file management) and less transparent.

2. **Terraform GCS backend provides locking**: GCS state locking prevents concurrent applies that could corrupt state. This is a key benefit of remote backends.

3. **Config Connector requires GKE**: It's a GKE add-on — only applicable when you're already using GKE.

4. **Deployment Manager preview ≠ destructive**: The `--preview` flag shows what would happen without actually making changes. Useful for change review.

5. **Terraform init downloads providers**: You must run `terraform init` after changing provider versions or adding new providers. This downloads the required provider binaries.

6. **Terraform destroy removes everything**: Running `terraform destroy` deletes all resources managed by that state file. In production, this should be protected by CI/CD gates.

7. **GCP Terraform provider vs GCP API types**: In Terraform, you use `google_compute_instance`; in Deployment Manager, you use `compute.v1.instance`. Both manage the same resource via different syntax.

### Keywords
- Deployment Manager configuration, YAML template, Jinja2, Python template, deployment preview, Terraform provider, HCL, GCS backend, state locking, terraform plan, terraform apply, Config Connector, remote state, IaC service account

---

## Source

- https://cloud.google.com/deployment-manager/docs/overview
- https://registry.terraform.io/providers/hashicorp/google/latest/docs
- https://developer.hashicorp.com/terraform/language/settings/backends/gcs
- https://cloud.google.com/config-connector/docs/overview
