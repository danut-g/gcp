# Section 3.6 — Implementing Resources Through Infrastructure as Code

## Exam Relevance
This topic is part of **Section 2: Planning and implementing a cloud solution (~30 % of the exam)**. You must understand infrastructure as code (IaC) tooling including Fabric FAST, Config Connector, Terraform, and Helm, as well as versioning, state management, and deployment updates.

---

## 1. Infrastructure as Code (IaC) Overview

> 📖 **Docs:** [IaC on Google Cloud](https://cloud.google.com/docs/terraform) | [DevOps tech: Infrastructure as code](https://cloud.google.com/architecture/devops/devops-tech-infrastructure-as-code) | 🖥️ **Console:** n/a (planning reference)

### What Is IaC?
Infrastructure as Code means defining and managing cloud resources using **declarative configuration files** rather than manual Console operations. Benefits:

- **Repeatability** — Deploy identical environments (dev, staging, prod)
- **Version control** — Track changes in Git
- **Automation** — CI/CD pipelines for infrastructure
- **Documentation** — Configuration files serve as documentation
- **Consistency** — Eliminate configuration drift
- **Collaboration** — Code review for infrastructure changes

---

## 2. Terraform

> 📖 **Docs:** [Get started with Terraform on GCP](https://cloud.google.com/docs/terraform/get-started-with-terraform) | [Google Cloud provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs) | 🖥️ **Console:** Cloud Shell (recommended for Terraform)

### What Is Terraform?
- **Open-source IaC tool** by HashiCorp
- Most widely used IaC tool for Google Cloud
- Declarative language (HCL — HashiCorp Configuration Language)
- Supports multiple cloud providers (multi-cloud)
- Manages resource lifecycle (create, update, delete)

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Provider** | Plugin for a cloud platform (e.g., `google`, `aws`) |
| **Resource** | A piece of infrastructure to manage (e.g., VM, VPC) |
| **Data Source** | Read-only information from existing resources |
| **Variable** | Input parameters for configurations |
| **Output** | Values returned after applying |
| **State** | Record of managed resources (stored in `terraform.tfstate`) |
| **Module** | Reusable collection of resources |
| **Plan** | Preview of changes before applying |

### Basic Terraform Workflow

```
terraform init → terraform plan → terraform apply → terraform destroy
    │                 │                  │                  │
    │                 │                  │                  └── Delete all resources
    │                 │                  └── Create/update resources
    │                 └── Preview changes (dry run)
    └── Download providers and initialize backend
```

### Terraform Configuration Example

```hcl
# main.tf

# Configure the Google provider
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

# Create a VPC
resource "google_compute_network" "vpc" {
  name                    = "my-vpc"
  auto_create_subnetworks = false
}

# Create a subnet
resource "google_compute_subnetwork" "subnet" {
  name          = "my-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = var.region
  network       = google_compute_network.vpc.id

  private_ip_google_access = true
}

# Create a VM instance
resource "google_compute_instance" "vm" {
  name         = "my-vm"
  machine_type = "e2-standard-4"
  zone         = "${var.region}-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
      size  = 50
      type  = "pd-ssd"
    }
  }

  network_interface {
    subnetwork = google_compute_subnetwork.subnet.id
    # No external IP (access_config block omitted)
  }

  tags = ["web-server"]

  metadata = {
    enable-oslogin = "TRUE"
  }

  service_account {
    email  = google_service_account.vm_sa.email
    scopes = ["cloud-platform"]
  }
}

# Create a service account
resource "google_service_account" "vm_sa" {
  account_id   = "vm-service-account"
  display_name = "VM Service Account"
}

# Create a firewall rule
resource "google_compute_firewall" "allow_http" {
  name    = "allow-http"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]
}
```

```hcl
# variables.tf
variable "project_id" {
  description = "GCP project ID"
  type        = string
}

variable "region" {
  description = "GCP region"
  type        = string
  default     = "us-central1"
}
```

```hcl
# outputs.tf
output "vm_internal_ip" {
  value = google_compute_instance.vm.network_interface[0].network_ip
}

output "vpc_id" {
  value = google_compute_network.vpc.id
}
```

### Terraform Commands

```bash
# Initialize (download providers, configure backend)
terraform init

# Format configuration files
terraform fmt

# Validate configuration
terraform validate

# Plan changes (dry run)
terraform plan

# Apply changes
terraform apply

# Apply without confirmation prompt
terraform apply -auto-approve

# Destroy all resources
terraform destroy

# Show current state
terraform show

# List resources in state
terraform state list

# Import an existing resource into state
terraform import google_compute_instance.vm projects/PROJECT/zones/ZONE/instances/NAME

# Remove a resource from state (without deleting it)
terraform state rm google_compute_instance.vm
```

### Terraform State Management

| Backend | Description | Use Case |
|---------|-------------|----------|
| **Local** | `terraform.tfstate` on local disk | Development only |
| **GCS** | State stored in Cloud Storage bucket | Team collaboration, production |
| **Remote (Terraform Cloud)** | HashiCorp's managed backend | Enterprise management |

**Best practices for state**:
- **Never store state locally in production** — Use a remote backend (GCS)
- **Enable state locking** — Prevents concurrent modifications
- **Enable versioning** on the GCS bucket — Recover from bad applies
- **Never edit state manually** — Use `terraform state` commands

### Terraform Provider Authentication

The Google provider uses **Application Default Credentials (ADC)** to authenticate to GCP. In order of precedence:

1. `GOOGLE_APPLICATION_CREDENTIALS` environment variable pointing to a service account key file
2. `gcloud auth application-default login` credentials in `~/.config/gcloud/application_default_credentials.json`
3. The attached service account of the VM / Cloud Build / Cloud Shell Terraform runs in

```bash
# User-mode ADC for local development
gcloud auth application-default login

# Service-account key for CI/CD (use sparingly; prefer Workload Identity Federation)
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/sa-key.json
```

- **Exam tip**: On Cloud Build or a Compute Engine VM, Terraform automatically uses the attached service account — no explicit credentials needed.

### Environment-Specific Variable Files (`.tfvars`)

```hcl
# dev.tfvars
project_id = "my-project-dev"
region     = "us-central1"
```

```hcl
# prod.tfvars
project_id = "my-project-prod"
region     = "us-east1"
```

```bash
terraform plan -var-file=dev.tfvars
terraform apply -var-file=prod.tfvars
```

- Commit `.tfvars` files (non-sensitive values) to version control; keep secrets out of Terraform and use Secret Manager instead

---

## 3. Fabric FAST (and Cloud Foundation Toolkit)

> 📖 **Docs:** [Cloud Foundation Toolkit](https://cloud.google.com/foundation-toolkit) | [Fabric FAST on GitHub](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast) | 🖥️ **Console:** n/a (CLI/Terraform based)

### What Is Fabric FAST?
**Fabric FAST** (Fast Automation for Setups and Tenants) is Google Cloud's current recommended opinionated Terraform framework for deploying production-ready GCP **landing zones** and organizational infrastructure. It is built on top of the google-cloud-foundation-fabric library of modules.

- **Purpose**: Set up organization-level governance, networking, security, and project structure from day one
- **Approach**: Composable stages that build on each other (bootstrap → resource management → networking → security → projects)
- **Audience**: Platform teams setting up GCP environments for multiple teams

### Fabric FAST Stages

```
Stage 0: Bootstrap        — org policies, root IAM, billing, state bucket
Stage 1: Resource Mgmt    — folder structure, project factory
Stage 2: Networking       — VPCs, VPNs, interconnects, Shared VPCs
Stage 2b: Security        — KMS, Secret Manager, org-level security
Stage 3: Project Factory  — Tenant projects from YAML descriptors
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Stages** | Ordered Terraform deployments, each outputting variables consumed by later stages |
| **FAST modules** | Reusable Terraform modules in the `google-cloud-foundation-fabric` library |
| **Variable files** | Each stage reads outputs from prior stages via `.tfvars` files |
| **Project factory** | YAML-driven pattern for creating projects with consistent structure |

### Using Fabric FAST

```bash
# Clone the FAST repository
git clone https://github.com/GoogleCloudPlatform/cloud-foundation-fabric

# Navigate to a stage
cd cloud-foundation-fabric/fast/stages/0-bootstrap

# Initialize and apply
terraform init
terraform apply -var-file=terraform.tfvars

# Pass outputs to the next stage
terraform output -json > ../1-resman/terraform.tfvars.json
```

### Fabric FAST vs. Cloud Foundation Toolkit (CFT)

| Aspect | Fabric FAST | Cloud Foundation Toolkit (CFT) |
|--------|------------|-------------------------------|
| Scope | Landing zone (org-level) | Individual service modules |
| Approach | Staged, opinionated end-to-end | Module library (mix-and-match) |
| Abstraction | Higher (YAML project factory) | Lower (direct HCL) |
| Best for | New GCP org setup | Adding specific services |
| Relationship | Uses fabric modules internally | Independent Terraform modules |

### Cloud Foundation Toolkit (CFT) — Still Relevant

CFT is the collection of individual Terraform modules for specific GCP services. It remains useful for adding specific resources to existing infrastructure:

| Module | Description |
|--------|-------------|
| `terraform-google-project-factory` | Create and configure projects |
| `terraform-google-network` | Create VPCs, subnets, firewall rules |
| `terraform-google-kubernetes-engine` | Create GKE clusters |
| `terraform-google-sql-db` | Create Cloud SQL instances |
| `terraform-google-iam` | Manage IAM bindings |
| `terraform-google-log-export` | Configure log exports |

### When to Use Each

| Scenario | Recommended Tool |
|----------|-----------------|
| Setting up a new GCP organization | Fabric FAST (stages 0-3) |
| Adding a GKE cluster to existing infra | CFT `terraform-google-kubernetes-engine` |
| GitOps K8s-native GCP resource management | Config Connector |
| Packaging app for K8s deployment | Helm |
| Simple infrastructure automation | Terraform directly |

### Key Modules

| Module | Description |
|--------|-------------|
| `terraform-google-project-factory` | Create and configure projects |
| `terraform-google-network` | Create VPCs, subnets, firewall rules |
| `terraform-google-kubernetes-engine` | Create GKE clusters |
| `terraform-google-sql-db` | Create Cloud SQL instances |
| `terraform-google-iam` | Manage IAM bindings |
| `terraform-google-log-export` | Configure log exports |

### Example: Using CFT Network Module

```hcl
module "vpc" {
  source  = "terraform-google-modules/network/google"
  version = "~> 9.0"

  project_id   = var.project_id
  network_name = "my-vpc"
  routing_mode = "REGIONAL"

  subnets = [
    {
      subnet_name           = "web-subnet"
      subnet_ip             = "10.0.1.0/24"
      subnet_region         = "us-central1"
      subnet_private_access = true
    },
    {
      subnet_name           = "db-subnet"
      subnet_ip             = "10.0.2.0/24"
      subnet_region         = "us-central1"
      subnet_private_access = true
    }
  ]
}
```

---

## 4. Config Connector

> 📖 **Docs:** [Config Connector overview](https://cloud.google.com/config-connector/docs/overview) | [Install Config Connector](https://cloud.google.com/config-connector/docs/how-to/install-upgrade-uninstall) | 🖥️ **Console:** Kubernetes Engine → Clusters → Add-ons → Config Connector

### What Is Config Connector?
- A **Kubernetes add-on** that manages GCP resources using Kubernetes resource definitions (YAML/kubectl)
- Treats GCP resources as **Kubernetes custom resources**
- Enables GitOps workflows for infrastructure

### How It Works

```
Developer → kubectl apply -f gcs-bucket.yaml → Config Connector → GCP API → Cloud Storage Bucket
```

### Example: Creating a Cloud Storage Bucket

```yaml
# bucket.yaml
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-config-connector-bucket
  namespace: config-connector
  annotations:
    cnrm.cloud.google.com/project-id: "my-project-id"
spec:
  location: US
  storageClass: STANDARD
  uniformBucketLevelAccess: true
  versioning:
    enabled: true
```

```bash
# Apply the resource
kubectl apply -f bucket.yaml

# Check status
kubectl get storagebucket my-config-connector-bucket -n config-connector

# Delete the resource (also deletes from GCP)
kubectl delete -f bucket.yaml
```

### Example: Creating a Compute Instance

```yaml
apiVersion: compute.cnrm.cloud.google.com/v1beta1
kind: ComputeInstance
metadata:
  name: my-vm
  namespace: config-connector
spec:
  machineType: e2-standard-4
  zone: us-central1-a
  bootDisk:
    initializeParams:
      sourceImageRef:
        external: debian-cloud/debian-12
  networkInterface:
    - networkRef:
        name: my-vpc
      subnetworkRef:
        name: my-subnet
```

### When to Use Config Connector
- Teams already using Kubernetes and kubectl
- Want to manage GCP resources alongside K8s resources
- GitOps workflows (ArgoCD, Flux)
- Prefer YAML over HCL (Terraform)

### Config Connector vs. Terraform

| Feature | Config Connector | Terraform |
|---------|-----------------|-----------|
| Language | YAML (K8s manifests) | HCL |
| Requires | GKE cluster | Terraform CLI |
| Reconciliation | Continuous (controller loop) | On-demand (apply/plan) |
| State | Kubernetes API (etcd) | State file (local/remote) |
| Multi-cloud | GCP only | Multi-cloud |
| Learning curve | K8s knowledge needed | Terraform knowledge needed |
| Best for | K8s-native teams | General IaC |

---

## 5. Helm

> 📖 **Docs:** [Use Helm with GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-apps-with-helm) | [Helm documentation](https://helm.sh/docs/) | 🖥️ **Console:** Cloud Shell → helm install ...

### What Is Helm?
- **Package manager for Kubernetes** (like apt/brew for K8s)
- Packages Kubernetes manifests into **charts** (reusable bundles)
- Manages releases (install, upgrade, rollback)
- Supports templating for configurable deployments

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Chart** | Package of K8s resource templates |
| **Release** | An instance of a chart deployed to a cluster |
| **Repository** | Collection of charts (like a package registry) |
| **Values** | Configuration parameters for a chart |
| **Template** | K8s manifest with Go template variables |

### Helm Commands

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add a chart repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install a chart
helm install my-nginx bitnami/nginx

# Install with custom values
helm install my-nginx bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=LoadBalancer

# Install with a values file
helm install my-nginx bitnami/nginx -f values.yaml

# List releases
helm list

# Upgrade a release
helm upgrade my-nginx bitnami/nginx --set replicaCount=5

# Rollback a release
helm rollback my-nginx 1

# Uninstall a release
helm uninstall my-nginx

# Show chart information
helm show chart bitnami/nginx
helm show values bitnami/nginx
```

### Chart Structure

```
my-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl    # Template helper functions
├── charts/             # Dependency charts
└── README.md
```

### When to Use Helm
- Deploying complex Kubernetes applications with many manifests
- Need to manage different configurations for different environments
- Using third-party applications (nginx, PostgreSQL, Redis, Prometheus)
- Need rollback capabilities for K8s deployments
- Sharing reusable application configurations across teams

---

## 6. IaC Tool Comparison

> 📖 **Docs:** [Compare IaC tools for GCP](https://cloud.google.com/docs/terraform) | 🖥️ **Console:** n/a (planning reference)

| Feature | Terraform | Fabric FAST | CFT modules | Config Connector | Helm |
|---------|-----------|------------|-------------|-----------------|------|
| **Manages** | Any cloud resource | GCP org/landing zone | GCP services (modules) | GCP resources | K8s resources |
| **Language** | HCL | HCL + YAML descriptors | HCL (Terraform modules) | YAML | YAML + Go templates |
| **Scope** | Multi-cloud | GCP org-level | GCP only | GCP only | Kubernetes only |
| **Requires** | Terraform CLI | Terraform CLI | Terraform CLI | GKE cluster | K8s cluster |
| **State** | State file | State file (per stage) | State file | K8s API | K8s API |
| **Versioning** | Git + tfvars per env | Git + stage outputs | Git + module versions | Git (GitOps) | Chart versions |
| **Best for** | General IaC, multi-cloud | New GCP org setup | Adding specific services | K8s-native GCP | K8s app packaging |

---

## 4b. IaC Versioning, State Management, and Updates

A key exam topic (Section 2.4) is understanding how to plan and execute IaC deployments with proper versioning, state management, and update strategies.

### Versioning IaC Configurations

| Practice | How |
|----------|-----|
| **Git version control** | Store all `.tf` files in a Git repo; never apply untracked changes |
| **Module versioning** | Pin module versions: `version = "~> 5.0"` to avoid breaking changes |
| **Provider versioning** | Pin provider versions in `required_providers` block |
| **State file versioning** | Enable **GCS bucket versioning** to roll back corrupt state |
| **Branching strategy** | Feature branches + PR review before merging to main |

```hcl
# Pin provider versions to avoid unexpected upgrades
terraform {
  required_version = ">= 1.5"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"   # Allow 5.x.x but not 6.x
    }
  }
}
```

### State Management Best Practices

```bash
# Configure remote GCS backend with state locking
terraform {
  backend "gcs" {
    bucket = "my-terraform-state-bucket"
    prefix = "env/production"
  }
}
```

| Practice | Command/Action |
|----------|---------------|
| Enable GCS versioning | `gcloud storage buckets update gs://state-bucket --versioning` |
| List state resources | `terraform state list` |
| Remove resource from state (without destroying) | `terraform state rm google_compute_instance.vm` |
| Import existing resource into state | `terraform import google_compute_instance.vm PROJECT/ZONE/NAME` |
| Move resource in state (rename) | `terraform state mv old_address new_address` |
| Inspect state | `terraform show` |
| Unlock stuck state | `terraform force-unlock LOCK_ID` |

### Updating IaC Deployments Safely

```bash
# 1. Plan before applying — always review what will change
terraform plan -out=my.plan

# 2. Apply from a saved plan (no interactive prompt)
terraform apply my.plan

# 3. Target a specific resource for updates (use sparingly)
terraform apply -target=google_compute_instance.vm

# 4. For destructive changes, confirm the blast radius
terraform plan | grep "will be destroyed"

# 5. Use -refresh=false to skip state refresh (faster, but risky)
terraform plan -refresh=false

# 6. Prevent accidental destruction with lifecycle rules
resource "google_storage_bucket" "critical" {
  name = "critical-data"
  lifecycle {
    prevent_destroy = true    # Terraform will error if you try to destroy this
  }
}
```

### CI/CD Pipeline for Terraform

A typical Terraform CI/CD pipeline:

```
Developer pushes code
    ↓
CI: terraform fmt --check (fail if not formatted)
    ↓
CI: terraform validate (syntax check)
    ↓
CI: terraform plan (output plan for review)
    ↓
Code Review + Approval
    ↓
Merge to main
    ↓
CD: terraform apply (auto-apply in dev) or manual approval (prod)
```

### Key Exam Points
- Always use a **remote backend** (GCS) with **versioning** for team environments
- `terraform plan` before every `terraform apply` — never skip it
- **State locking** prevents concurrent modifications — GCS backend uses native locking
- Use `terraform import` to bring existing resources under Terraform management
- `prevent_destroy = true` lifecycle rule protects critical resources from accidental deletion
- For Fabric FAST, each stage has its own state file stored in a dedicated GCS bucket

---

## 5. Cloud Deployment Manager

> 📖 **Docs:** [Deployment Manager overview](https://cloud.google.com/deployment-manager/docs) | [Create a deployment](https://cloud.google.com/deployment-manager/docs/step-by-step-guide/create-a-configuration) | 🖥️ **Console:** Deployment Manager → Deployments

- Google Cloud's native IaC tool using YAML/Python/Jinja2 templates
- Exam expects familiarity with the concept and basic commands

### Basic structure (config.yaml)

```yaml
imports:
- path: vm_template.jinja

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
      autoDelete: true
      initializeParams:
        sourceImage: projects/debian-cloud/global/images/family/debian-12
    networkInterfaces:
    - network: global/networks/default
```

### gcloud commands

```bash
gcloud deployment-manager deployments create my-deployment --config config.yaml
gcloud deployment-manager deployments update my-deployment --config config.yaml
gcloud deployment-manager deployments list
gcloud deployment-manager deployments describe my-deployment
gcloud deployment-manager deployments delete my-deployment
```

- Deployment Manager uses a Google-managed SA to create resources; this SA needs appropriate IAM roles
- **Exam tip**: For ACE, know the concept and basic gcloud commands; Terraform is preferred in practice but Deployment Manager is GCP-native

---

## 6. Terraform Workspaces

> 📖 **Docs:** [Terraform workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces) | [Managing multiple environments](https://cloud.google.com/docs/terraform/best-practices-for-terraform#environment-specific-configuration) | 🖥️ **Console:** Cloud Shell

- Manage multiple environments (dev, staging, prod) from the same Terraform code

```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace list
terraform workspace select prod
terraform workspace show     # prints current workspace name
```

- Each workspace has its own state file in the backend
- Reference current workspace in config: `terraform.workspace`
- Example: `name = "my-app-${terraform.workspace}"`
- **Exam tip**: Workspaces are simpler than separate state files/backends for environment separation; use with caution for large infra (separate root modules preferred by Terraform best practices)

---

## 7. Kustomize

> 📖 **Docs:** [Kustomize with GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/kustomize) | [Kustomize documentation](https://kustomize.io/) | 🖥️ **Console:** Cloud Shell → kubectl apply -k ...

- Kubernetes-native configuration management (built into kubectl since 1.14)
- No templating engine — uses strategic merge patches and JSON patches
- Use cases: environment-specific overlays without duplicating YAML

### Structure

```
base/
  deployment.yaml
  service.yaml
  kustomization.yaml
overlays/
  prod/
    kustomization.yaml   # patches for prod
  dev/
    kustomization.yaml   # patches for dev
```

### base/kustomization.yaml

```yaml
resources:
- deployment.yaml
- service.yaml
```

### overlays/prod/kustomization.yaml

```yaml
bases:
- ../../base
namePrefix: prod-
replicas:
- name: my-app
  count: 5
images:
- name: my-image
  newTag: "2.0.0"
```

### Commands

```bash
kubectl apply -k overlays/prod/
kubectl diff -k overlays/prod/
kustomize build overlays/prod/ | kubectl apply -f -
```

- **Exam tip**: Kustomize is the Kubernetes-native alternative to Helm for configuration management without packaging

---

## Exam Practice Questions

1. **Your team needs to deploy identical infrastructure across dev, staging, and production environments. Which approach should you use?**
   - Answer: **Terraform** with different variable files (`.tfvars`) for each environment. Use the same Terraform configuration with environment-specific values.

2. **Where should Terraform state be stored for a team of 5 engineers working on the same GCP project?**
   - Answer: In a **GCS backend** (`backend "gcs"`) with state locking enabled. This allows concurrent access while preventing conflicts.

3. **A team already uses ArgoCD and wants to manage GCP resources (Cloud SQL, Cloud Storage) using the same GitOps workflow. What should they use?**
   - Answer: **Config Connector** — Manages GCP resources as Kubernetes custom resources, integrating naturally with ArgoCD GitOps pipelines.

4. **You need to deploy a complex application (nginx + Redis + PostgreSQL) to GKE quickly using community-maintained configurations. What tool is most appropriate?**
   - Answer: **Helm** — Install community charts from Bitnami or other repositories. Each component can be deployed with `helm install`.

5. **What is Fabric FAST and when would you use it over CFT modules?**
   - Answer: **Fabric FAST** is Google Cloud's opinionated Terraform framework for deploying a complete GCP **landing zone** (org policies, networking, IAM, project factory) in staged deployments. Use it when setting up a new GCP organization. CFT modules are individual Terraform modules — use them to add specific services (GKE, Cloud SQL) to an existing environment.

5b. **What is the Cloud Foundation Toolkit (CFT)?**
   - Answer: A collection of **opinionated, production-ready Terraform modules** built by Google that implement GCP best practices for common architectures (projects, networks, GKE clusters, etc.).

6. **You accidentally deleted a resource manually that Terraform manages. What happens on the next `terraform apply`?**
   - Answer: Terraform detects the resource is missing from GCP (but still in state) and will **recreate it** to match the desired configuration.

---

## Glossary

**ACE (Associate Cloud Engineer)** — A Google Cloud certification exam that validates skills in deploying, managing, and operating cloud solutions on GCP.

**ADC (Application Default Credentials)** — Google's authentication library strategy for automatically discovering credentials from environment variables, gcloud login, or the attached service account of the host running the code. Used by the Terraform google provider.

**Apply (Terraform)** — The `terraform apply` command that executes a plan and makes real changes to infrastructure to reach the declared desired state.

**ArgoCD** — An open-source GitOps continuous delivery tool for Kubernetes that declaratively manages application deployments from Git repositories.

**Backend** — In Terraform, the configuration that determines where the state file is stored and optionally where operations are executed (e.g., local disk, GCS, Terraform Cloud).

**Base (Kustomize)** — The set of Kubernetes manifests that serve as the foundational configuration, referenced by one or more overlays in a Kustomize directory structure.

**Bitnami** — A popular publisher of maintained Helm charts for common open-source applications (nginx, PostgreSQL, Redis, etc.), referenced as `bitnami/<chart>`.

**CFT (Cloud Foundation Toolkit)** — A collection of opinionated, production-ready Terraform modules built by Google that implement GCP best practices for common architectures.

**Fabric FAST** — Google Cloud's opinionated Terraform framework (Fast Automation for Setups and Tenants) for deploying production-ready GCP landing zones in a staged approach; the recommended framework for setting up new GCP organizations with proper governance, networking, and project structures.

**Chart** — In Helm, a packaged collection of Kubernetes resource template files and metadata that defines a complete application deployment.

**Chart.yaml** — The metadata file at the root of a Helm chart containing the chart name, version, description, and dependencies.

**CI/CD (Continuous Integration / Continuous Delivery)** — A software development practice that automates building, testing, and deploying code changes; IaC configurations can be managed through CI/CD pipelines.

**Cloud Deployment Manager** — GCP's native infrastructure-as-code service that uses YAML, Python, or Jinja2 templates to define and deploy GCP resources.

**Cloud SQL** — GCP's fully managed relational database service supporting MySQL, PostgreSQL, and SQL Server; provisionable via CFT modules.

**Cloud Storage** — GCP's scalable object storage service; used as a Terraform remote backend (GCS) for storing state files.

**Config Connector** — A Kubernetes add-on that allows GCP resources to be managed using Kubernetes-style YAML manifests and kubectl, treating GCP resources as Kubernetes custom resources.

**Configuration Drift** — The divergence between the declared desired state of infrastructure and its actual running state, which IaC tools help detect and remediate.

**Custom Resource** — A Kubernetes API extension that allows users to define and manage new types of objects beyond the built-in Kubernetes resource types; Config Connector uses custom resources to represent GCP resources.

**Data Source** — In Terraform, a read-only reference to information about existing infrastructure resources that can be used in configuration without managing the resource.

**Declarative** — A configuration style that describes the desired end state of infrastructure (what should exist), in contrast to imperative scripts that describe how to achieve it step by step.

**Destroy (Terraform)** — The `terraform destroy` command that deletes all resources managed by the current Terraform state, returning the environment to an empty state.

**Dry Run** — An operation that previews what changes would be made without actually applying them; `terraform plan` performs a dry run.

**etcd** — A distributed key-value store used by Kubernetes as its primary data store; Config Connector stores GCP resource state through the Kubernetes API backed by etcd.

**Flux** — An open-source GitOps tool for Kubernetes that automatically synchronizes cluster state with Git repositories.

**GCS (Google Cloud Storage)** — GCP's object storage service; used as a Terraform remote backend for team-shared state storage and locking.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services, the primary target platform for the tools covered in this chapter.

**GKE (Google Kubernetes Engine)** — GCP's managed Kubernetes service; required to run Config Connector and can be managed via CFT Terraform modules.

**GitOps** — An operational model that uses Git as the single source of truth for declarative infrastructure and application configuration, with automated reconciliation to the desired state.

**Go Templates** — The templating language used in Helm chart templates to generate Kubernetes manifests dynamically from values files.

**HA (High Availability)** — A design principle ensuring a system remains operational with minimal downtime; referenced in GCP architecture patterns managed with CFT.

**HCL (HashiCorp Configuration Language)** — The domain-specific declarative language used to write Terraform configuration files; also used for Cloud Foundation Toolkit modules.

**Helm** — A package manager for Kubernetes that bundles Kubernetes manifests into reusable, versioned packages called charts, with support for templating and release management.

**Host Project** — In GCP, the project that owns a Shared VPC network; referenced in CFT network modules.

**Image (Container)** — A packaged, immutable snapshot of a container's filesystem and metadata. Referenced in Terraform VM resources and Kubernetes/Helm deployment manifests.

**Init (Terraform)** — The `terraform init` command that downloads provider plugins, initializes the backend, and prepares the working directory for other Terraform commands.

**Ingress (Kubernetes)** — A Kubernetes resource that manages external HTTP(S) access to services in a cluster, typically rendered from a Helm chart template.

**IaC (Infrastructure as Code)** — The practice of managing and provisioning cloud infrastructure through machine-readable configuration files rather than manual processes.

**IAM (Identity and Access Management)** — GCP's access control system; Deployment Manager uses a Google-managed service account with IAM roles to create resources.

**ImagePullSecrets** — Kubernetes objects that store credentials for authenticating to private container registries when pulling images.

**Jinja2** — A Python-based templating engine supported by Cloud Deployment Manager for generating dynamic resource configurations.

**JSON Patch** — A format for describing changes to a JSON document; used by Kustomize as an alternative to strategic merge patches for modifying Kubernetes manifests.

**K8s** — Common abbreviation for Kubernetes.

**kubectl** — The command-line tool for interacting with Kubernetes clusters; used to apply Config Connector manifests and Kustomize configurations.

**Kubernetes** — An open-source container orchestration system for automating deployment, scaling, and management of containerized applications; the foundation for GKE, Config Connector, and Helm.

**Kustomize** — A Kubernetes-native configuration management tool built into kubectl that customizes raw YAML manifests using patches and overlays without templating.

**Module** — In Terraform, a reusable container of resources grouped together; CFT provides pre-built modules for common GCP services.

**Multi-cloud** — The practice of using services from multiple cloud providers; Terraform supports multi-cloud resource management through providers.

**Namespace (Kubernetes)** — A mechanism for isolating groups of Kubernetes resources within a single cluster. Config Connector places managed GCP resources inside a namespace (e.g., `config-connector`).

**Nginx** — A popular open-source web server and reverse proxy; commonly used as an example Helm chart deployment (bitnami/nginx).

**Output** — In Terraform, a named value exposed after `terraform apply` completes, such as an IP address or resource ID, used to share information between modules or with operators.

**Overlay (Kustomize)** — Environment-specific Kustomize configuration that patches a base configuration, allowing different settings for dev, staging, and production without duplicating YAML.

**Plan (Terraform)** — The `terraform plan` command that previews the changes Terraform would make to reach the desired state without actually applying them.

**PostgreSQL** — An open-source relational database; commonly deployed on GKE using Helm charts (e.g., bitnami/postgresql).

**Provider** — In Terraform, a plugin that manages the lifecycle of resources for a specific cloud platform or service (e.g., the `google` provider for GCP).

**Redis** — An open-source in-memory data store; commonly deployed on GKE using Helm charts.

**Release** — In Helm, a specific running instance of a chart deployed to a Kubernetes cluster, identified by a user-defined name.

**Remote Backend** — A Terraform backend configuration that stores state outside of the local filesystem, enabling team collaboration and state locking (e.g., GCS bucket).

**Repository** — In Helm, a collection of charts hosted at a URL that can be searched and installed from; in Terraform, a source location for modules.

**Resource** — In Terraform, a block that declares a piece of infrastructure to be created and managed (e.g., a VM, VPC, or firewall rule).

**Role (IAM)** — A named bundle of permissions granted to a principal. Deployment Manager service accounts, Terraform-run service accounts, and Config Connector GKE node service accounts all need appropriate IAM roles to create resources.

**Rollback (Helm)** — The `helm rollback` command that reverts a Helm release to a previously deployed revision.

**Root Module** — The main Terraform configuration directory from which `terraform` commands are run; recommended over workspaces for large-scale environment separation.

**SA (Service Account)** — A GCP identity used by applications or services (rather than humans); Deployment Manager uses a Google-managed SA to provision resources.

**State** — In Terraform, a JSON file (`terraform.tfstate`) that records the current known state of all managed resources, used to calculate diffs and plan changes.

**State File** — The file (`terraform.tfstate`) or remote object that stores Terraform's record of managed resources; must be protected and never edited manually.

**State Locking** — A Terraform mechanism that prevents concurrent modifications to the state file by multiple users or pipelines; supported by GCS backends.

**Secret Manager** — GCP's service for securely storing and accessing sensitive configuration data. Recommended over embedding secrets in Terraform configurations or `.tfvars` files.

**Strategic Merge Patch** — A Kubernetes-specific patching strategy used by Kustomize to merge changes into existing manifest fields rather than replacing entire objects.

**Template (Helm)** — A Kubernetes manifest in a Helm chart's `templates/` directory containing Go template expressions that are rendered at install or upgrade time using values from `values.yaml`.

**Terraform** — An open-source infrastructure-as-code tool by HashiCorp that uses HCL to define, plan, and provision infrastructure across multiple cloud providers.

**Terraform Cloud** — HashiCorp's managed remote backend service for Terraform that provides state storage, locking, and enterprise features.

**Terraform Workspace** — A Terraform feature that maintains separate state files within the same backend for managing multiple environments (dev, staging, prod) from a single configuration.

**tfvars** — Terraform variable definition files (`.tfvars` extension) used to supply environment-specific values to input variables without modifying the main configuration.

**Values (Helm)** — Configuration parameters for a Helm chart, supplied via `values.yaml`, `-f file.yaml`, or `--set key=value` flags, used to customize rendered templates.

**Variable** — In Terraform, an input parameter that allows configuration values to be passed in at runtime, enabling reusable and environment-agnostic configurations.

**VPC (Virtual Private Cloud)** — A logically isolated network on GCP; provisionable via the `terraform-google-network` CFT module.

**YAML (YAML Ain't Markup Language)** — A human-readable data serialization format used for Kubernetes manifests, Config Connector resources, Helm values files, Kustomize configurations, and Cloud Deployment Manager templates.
