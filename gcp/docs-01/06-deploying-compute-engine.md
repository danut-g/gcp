# Section 3.1 — Deploying and Implementing Compute Engine Resources

## Exam Relevance
This topic is part of **Section 3: Deploying and implementing a cloud solution (~25 % of the exam)**. You must know how to launch compute instances, configure disks and availability policies, manage SSH keys, create autoscaled managed instance groups, configure OS Login, and configure VM Manager.

---

## 0. Machine Families and Pricing Models (Exam-Critical Background)

> 📖 **Docs:** [Machine families overview](https://cloud.google.com/compute/docs/machine-resource) | [Compute Engine pricing](https://cloud.google.com/compute/pricing) | 🖥️ **Console:** Compute Engine → VM instances → Create → Machine configuration

### Machine Families Overview

| Family | Type | Use Case |
|--------|------|----------|
| **E2** | General purpose (cost-optimized) | Dev/test, small web apps, microservices |
| **N1 / N2 / N2D** | General purpose (balanced) | Standard workloads, databases, web servers |
| **T2D / T2A** | Scale-out (Arm / x86) | Web servers, containerized microservices |
| **C2 / C3** | Compute-optimized | HPC, game servers, single-threaded apps |
| **M1 / M2 / M3** | Memory-optimized | Large in-memory DBs (SAP HANA), analytics |
| **A2 / A3** | Accelerator-optimized (GPUs) | ML training, HPC with GPUs |

- **Shared-core** types (e.g., `e2-micro`, `e2-small`, `f1-micro`): minimal vCPUs intended for low-traffic workloads.
- **Custom machine types**: allow arbitrary vCPU/memory combinations on supported families (e.g., `custom-6-20480`).

### Pricing Models

| Model | Discount | Commitment | Use Case |
|-------|----------|-----------|----------|
| **On-demand** | 0% | None | Short-lived or unpredictable workloads |
| **Sustained Use Discount (SUD)** | Up to ~30% | None (automatic) | Long-running always-on VMs |
| **Committed Use Discount (CUD)** | Up to 57% (resource) / 70% (flex) | 1 or 3 years | Predictable long-term workloads |
| **Spot VMs** | Up to ~91% | None | Fault-tolerant, interruptible workloads |

- **Sustained Use Discount**: automatic discount applied when a VM runs for a significant portion of a billing month; no action required.
- **Committed Use Discount**: purchased commitment for a specific amount of vCPUs/memory (resource-based) or spend (flex); invoiced whether or not the resources are used.

---

## 1. Launching a Compute Instance

> 📖 **Docs:** [Create a VM instance](https://cloud.google.com/compute/docs/instances/create-start-instance) | [gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) | 🖥️ **Console:** Compute Engine → VM instances → Create instance

### Basic VM Creation

```bash
# Minimal VM creation
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud

# Full-featured VM creation
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-ssd \
  --tags=http-server,https-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx' \
  --service-account=my-sa@project.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --labels=environment=production,team=backend \
  --network=my-vpc \
  --subnet=my-subnet \
  --no-address  # No external IP
```

### Key Creation Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--zone` | Zone to create the VM in | `us-central1-a` |
| `--machine-type` | CPU/memory configuration | `e2-standard-4`, `custom-6-20480` |
| `--image-family` | OS image family | `debian-12`, `ubuntu-2204-lts` |
| `--image-project` | Project containing the image | `debian-cloud`, `ubuntu-os-cloud` |
| `--boot-disk-size` | Boot disk size in GB | `50GB` |
| `--boot-disk-type` | Disk type | `pd-standard`, `pd-balanced`, `pd-ssd` |
| `--tags` | Network tags for firewall rules | `http-server,https-server` |
| `--metadata` | Key-value metadata | `startup-script=...` |
| `--service-account` | Service account for the VM | `sa@project.iam.gserviceaccount.com` |
| `--scopes` | API scopes (legacy, prefer IAM) | `cloud-platform` |
| `--labels` | Labels for organization | `env=prod,team=web` |
| `--network` | VPC network | `my-vpc` |
| `--subnet` | Subnet | `my-subnet` |
| `--no-address` | No external IP | (flag) |

---

## 2. Assigning Disks

> 📖 **Docs:** [Persistent disks](https://cloud.google.com/compute/docs/disks) | [Add a persistent disk](https://cloud.google.com/compute/docs/disks/add-persistent-disk) | [Local SSD](https://cloud.google.com/compute/docs/disks/local-ssd) | 🖥️ **Console:** Compute Engine → VM instances → Edit → Add disk

### Boot Disk
- Every VM has exactly **one boot disk**
- Contains the operating system
- Defaults: 10 GB for Linux, 50 GB for Windows
- Can specify size, type, and image at creation

### Additional Disks
You can attach additional persistent disks to a VM for data storage.

```bash
# Create a standalone persistent disk
gcloud compute disks create my-data-disk \
  --size=200GB \
  --type=pd-ssd \
  --zone=us-central1-a

# Attach the disk to an existing VM
gcloud compute instances attach-disk my-vm \
  --disk=my-data-disk \
  --zone=us-central1-a

# Create a VM with an additional disk at creation time
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --create-disk=name=data-disk,size=200GB,type=pd-ssd,auto-delete=no
```

### Disk Operations Inside the VM

```bash
# After attaching, format and mount the disk inside the VM
# List attached disks
sudo lsblk

# Format the disk (first time only)
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0 /dev/sdb

# Create mount point and mount
sudo mkdir -p /mnt/data
sudo mount /dev/sdb /mnt/data

# Add to fstab for persistence across reboots
echo '/dev/sdb /mnt/data ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

### Disk Type Selection

| Type | IOPS | Throughput | Cost | Use Case |
|------|------|-----------|------|----------|
| `pd-standard` | 0.75 R / 1.5 W per GB | Low | Lowest | Logs, bulk storage |
| `pd-balanced` | 6 R / 6 W per GB | Medium | Medium | General purpose (default) |
| `pd-ssd` | 30 R / 30 W per GB | High | Higher | Databases, random I/O |
| `pd-extreme` | Up to 120K R / 120K W | Highest | Highest | Mission-critical DBs |

### Key Disk Concepts
- **Persistent disks are network-attached** — survive VM deletion (if auto-delete is off)
- **Local SSDs are physically attached** — data lost on VM stop/termination
- **Auto-delete**: By default, boot disk is auto-deleted with VM; additional disks are not
- **Read-only mode**: A persistent disk can be attached to multiple VMs in read-only mode
- **Resize**: You can increase disk size (never decrease) without downtime

---

## 3. Availability Policy

> 📖 **Docs:** [Setting VM host options](https://cloud.google.com/compute/docs/instances/setting-vm-host-options) | [Live migration](https://cloud.google.com/compute/docs/instances/live-migration-process) | 🖥️ **Console:** Compute Engine → VM instances → Create → Availability policies

### Automatic Restart
- VMs can be configured to **automatically restart** after maintenance events or crashes
- Default: **ON** (recommended for production)
- Set `--restart-on-failure` flag

### Host Maintenance Behavior
When Google needs to perform maintenance on the physical host:

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **MIGRATE** (default) | Live-migrates the VM to another host | Production VMs, no downtime |
| **TERMINATE** | Stops the VM during maintenance | Spot VMs, GPUs (cannot live-migrate) |

```bash
# Set availability policy
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --maintenance-policy=MIGRATE \
  --restart-on-failure

# VMs with GPUs must use TERMINATE
gcloud compute instances create gpu-vm \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --maintenance-policy=TERMINATE
```

### Sole-Tenant Nodes
- Dedicated physical servers for your VMs only
- Use cases: Licensing compliance (BYOL), security isolation, performance
- VMs on sole-tenant nodes still follow maintenance policies

---

## 4. SSH Keys

> 📖 **Docs:** [SSH connections to Linux VMs](https://cloud.google.com/compute/docs/connect/ssh-linux) | [Add SSH keys](https://cloud.google.com/compute/docs/connect/add-ssh-keys) | 🖥️ **Console:** Compute Engine → VM instances → SSH button / Metadata → SSH keys

### Methods to Connect to VMs

#### 1. OS Login (Recommended)
- Uses IAM to manage SSH access
- Links Linux user accounts to Google identities
- Supports 2FA (two-factor authentication)
- No need to manage individual SSH keys

```bash
# Enable OS Login on a project
gcloud compute project-info add-metadata \
  --metadata enable-oslogin=TRUE

# Enable OS Login on a specific instance
gcloud compute instances add-metadata my-vm \
  --metadata enable-oslogin=TRUE

# Connect (IAM role required: roles/compute.osLogin or roles/compute.osAdminLogin)
gcloud compute ssh my-vm --zone=us-central1-a
```

#### 2. Metadata-Managed SSH Keys
- SSH public keys stored in project or instance metadata
- **Project-wide keys** — Apply to all VMs in the project
- **Instance-level keys** — Apply to a specific VM only

```bash
# Add a project-wide SSH key
gcloud compute project-info add-metadata \
  --metadata-from-file ssh-keys=public_keys_file

# Add an instance-level SSH key
gcloud compute instances add-metadata my-vm \
  --metadata-from-file ssh-keys=public_keys_file

# Block project-wide keys on a specific instance
gcloud compute instances add-metadata my-vm \
  --metadata block-project-ssh-keys=TRUE
```

#### 3. gcloud compute ssh
- Simplest method — handles key generation and transfer automatically
- Generates a key pair if one doesn't exist
- Stores the public key in project metadata

```bash
# Connect to a VM
gcloud compute ssh my-vm --zone=us-central1-a

# Connect with a specific user
gcloud compute ssh alice@my-vm --zone=us-central1-a

# Connect through IAP tunnel (VM has no external IP)
gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap
```

#### 4. SSH via Identity-Aware Proxy (IAP)
- Tunnel SSH connections through Google's IAP service
- VM does **not need an external IP**
- Requires IAM permission: `roles/iap.tunnelResourceAccessor`
- Firewall rule must allow TCP from `35.235.240.0/20` on port 22

---

## 5. Managed Instance Groups (MIGs)

> 📖 **Docs:** [MIG overview](https://cloud.google.com/compute/docs/instance-groups) | [Autoscaling MIGs](https://cloud.google.com/compute/docs/autoscaler) | [Autohealing](https://cloud.google.com/compute/docs/instance-groups/autohealing-instances-in-migs) | 🖥️ **Console:** Compute Engine → Instance groups → Create instance group

### What Is a Managed Instance Group?
- A group of **identical VM instances** managed as a single entity
- Based on an **instance template**
- Provides **autoscaling**, **autohealing**, **rolling updates**, and **load balancing**

### Instance Templates

An instance template is a **read-only blueprint** for creating VMs:

```bash
# Create an instance template
gcloud compute instance-templates create my-template \
  --machine-type=e2-standard-4 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-balanced \
  --tags=http-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update && apt-get install -y nginx'

# List instance templates
gcloud compute instance-templates list

# Describe a template
gcloud compute instance-templates describe my-template
```

**Key properties of instance templates**:
- **Immutable** — Cannot be edited after creation (create a new one instead)
- Define machine type, image, disks, network, metadata, labels, service account
- Can reference **custom images** or **public images**

### Creating a MIG

```bash
# Create a managed instance group
gcloud compute instance-groups managed create my-mig \
  --template=my-template \
  --size=3 \
  --zone=us-central1-a

# Create a regional MIG (spans multiple zones)
gcloud compute instance-groups managed create my-regional-mig \
  --template=my-template \
  --size=6 \
  --region=us-central1 \
  --target-distribution-shape=EVEN
```

### Autoscaling

```bash
# Configure autoscaling based on CPU utilization
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.70 \
  --cool-down-period=90

# Autoscale based on load balancing utilization
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --target-load-balancing-utilization=0.80

# Autoscale based on Cloud Monitoring metric
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --custom-metric-utilization=metric=custom.googleapis.com/my_metric,utilization-target=100,utilization-target-type=GAUGE

# Remove autoscaling
gcloud compute instance-groups managed stop-autoscaling my-mig \
  --zone=us-central1-a
```

### Autoscaling Signals

| Signal | Description | Use Case |
|--------|-------------|----------|
| **CPU utilization** | Average CPU across instances | General workloads |
| **HTTP LB utilization** | Requests per second per instance | Web applications |
| **Cloud Monitoring metric** | Custom or built-in metrics | Application-specific scaling |
| **Schedule** | Time-based scaling | Predictable traffic patterns |

### Autohealing

Health checks detect unhealthy VMs and automatically recreate them:

```bash
# Create a health check
gcloud compute health-checks create http my-health-check \
  --port=80 \
  --request-path=/health \
  --check-interval=30s \
  --timeout=10s \
  --healthy-threshold=2 \
  --unhealthy-threshold=3

# Apply health check to MIG for autohealing
gcloud compute instance-groups managed update my-mig \
  --zone=us-central1-a \
  --health-check=my-health-check \
  --initial-delay=300
```

### Rolling Updates

Update VMs without downtime:

```bash
# Update a MIG to use a new template
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=my-new-template \
  --zone=us-central1-a \
  --max-surge=3 \
  --max-unavailable=0

# Canary update (partial rollout)
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=my-template \
  --canary-version=template=my-new-template,target-size=20% \
  --zone=us-central1-a
```

### MIG Types

| Type | Scope | Use Case |
|------|-------|----------|
| **Zonal MIG** | Single zone | Simple deployments |
| **Regional MIG** | Multiple zones in a region | High availability |

---

## 6. Configuring OS Login

> 📖 **Docs:** [OS Login overview](https://cloud.google.com/compute/docs/oslogin) | [Set up OS Login](https://cloud.google.com/compute/docs/oslogin/set-up-oslogin) | 🖥️ **Console:** Compute Engine → Metadata → enable-oslogin = TRUE

### What Is OS Login?
OS Login links a user's Linux account to their Google identity, simplifying SSH access management.

### How It Works
1. Enable OS Login at the project or instance level
2. Grant appropriate IAM role to users
3. Users connect with `gcloud compute ssh` — their Google identity is used

### IAM Roles for OS Login

| Role | Permission |
|------|-----------|
| `roles/compute.osLogin` | Standard SSH access (non-root) |
| `roles/compute.osAdminLogin` | SSH access with sudo (root) privileges |

### Setup

```bash
# Enable OS Login project-wide
gcloud compute project-info add-metadata \
  --metadata enable-oslogin=TRUE

# Enable OS Login with 2FA
gcloud compute project-info add-metadata \
  --metadata enable-oslogin=TRUE,enable-oslogin-2fa=TRUE

# Grant OS Login role
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/compute.osLogin"
```

### OS Login vs. Metadata SSH Keys

| Feature | OS Login | Metadata SSH Keys |
|---------|----------|-------------------|
| Identity management | IAM-based | Manual key management |
| 2FA support | Yes | No |
| Audit trail | IAM audit logs | Limited |
| Granularity | Per-user IAM roles | Project or instance level |
| Recommended | Yes (Google best practice) | Legacy method |

---

## 7. Configuring VM Manager

> 📖 **Docs:** [VM Manager overview](https://cloud.google.com/compute/docs/manage-os) | [OS patch management](https://cloud.google.com/compute/docs/os-patch-management) | 🖥️ **Console:** Compute Engine → VM Manager → Patch

### What Is VM Manager?
A suite of tools for managing operating systems on large VM fleets:

### Components

#### OS Patch Management
- Automate OS patch deployment across VM fleets
- Schedule patch windows
- Patch compliance reporting

```bash
# Create a patch job
gcloud compute os-config patch-jobs execute \
  --instance-filter-names="zones/us-central1-a/instances/vm-1,zones/us-central1-a/instances/vm-2" \
  --duration=3600s

# Create a recurring patch deployment
gcloud compute os-config patch-deployments create my-patch-deployment \
  --instance-filter-all \
  --recurring-schedule-frequency=WEEKLY \
  --recurring-schedule-day-of-week=SUNDAY \
  --recurring-schedule-time-of-day='02:00' \
  --recurring-schedule-time-zone='America/New_York'
```

#### OS Inventory Management
- View OS information, installed packages, and system configurations
- Query across your VM fleet

```bash
# List OS inventory for instances
gcloud compute os-config inventories list \
  --location=us-central1-a

# Describe OS inventory for a specific instance
gcloud compute os-config inventories describe my-vm \
  --location=us-central1-a
```

#### OS Config Agent
- Must be installed on VMs for VM Manager to work
- Pre-installed on many Google-provided images
- Communicates with VM Manager API

### Enabling VM Manager

```bash
# Enable the OS Config API
gcloud services enable osconfig.googleapis.com

# Set metadata to enable the OS Config agent
gcloud compute project-info add-metadata \
  --metadata=enable-os-config=TRUE

# Verify agent is running on a VM
sudo systemctl status google-osconfig-agent
```

---

## 8. Preemptible and Spot VMs

> 📖 **Docs:** [Spot VMs](https://cloud.google.com/compute/docs/instances/spot) | [Preemptible VMs (legacy)](https://cloud.google.com/compute/docs/instances/preemptible) | 🖥️ **Console:** Compute Engine → VM instances → Create → Availability policies → Spot / Preemptible

### Preemptible VMs (Legacy)
- Up to **91% cheaper** than standard VMs
- Maximum runtime of **24 hours** — Google terminates them after 24 hours regardless
- May be **preempted at any time** with a 30-second shutdown notice
- **Cannot live-migrate** — must use `--maintenance-policy=TERMINATE`

### Spot VMs (Current Recommendation)
- Successor to preemptible VMs
- **No 24-hour runtime limit** — but can still be preempted at any time
- Same low pricing as preemptible
- Same restrictions: no live migration, 30-second shutdown notice

### Use Cases
- Batch processing jobs
- Fault-tolerant and stateless workloads
- CI/CD pipeline workers
- HPC and data analytics

### gcloud Flags

```bash
gcloud compute instances create my-spot-vm --machine-type=n2-standard-4 --provisioning-model=SPOT --instance-termination-action=STOP
```

- `--provisioning-model=SPOT` — creates a Spot VM (recommended)
- `--preemptible` — legacy flag; creates a preemptible VM
- `--instance-termination-action` — `STOP` (default for Spot) or `DELETE`

### Preemption Handling
- On preemption, GCP sends a **shutdown signal** → VM has ~30 seconds to run a shutdown script
- Check instance metadata to detect preemption: `instance/preempted` returns `TRUE` after a preemption event
- Use **Pub/Sub** or **Cloud Monitoring** to trigger alerts or replacement logic on preemption events

---

## 9. Snapshots and Custom Images

> 📖 **Docs:** [Create disk snapshots](https://cloud.google.com/compute/docs/disks/create-snapshots) | [Custom images](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images) | 🖥️ **Console:** Compute Engine → Snapshots / Images

### Snapshots

- **Point-in-time incremental backup** of a persistent disk
- First snapshot is a full backup; subsequent snapshots are incremental (only changed blocks)
- Stored in **Cloud Storage** (billed at Cloud Storage rates, not as a persistent disk)

```bash
gcloud compute disks snapshot DISK_NAME --zone=ZONE --snapshot-names=SNAPSHOT_NAME --storage-location=us
gcloud compute snapshots list
gcloud compute snapshots describe SNAPSHOT_NAME
gcloud compute disks create NEW_DISK --source-snapshot=SNAPSHOT_NAME --zone=ZONE --size=100GB --type=pd-ssd
gcloud compute snapshots delete SNAPSHOT_NAME
```

> **Cross-reference**: Snapshot schedules (automated recurring snapshots) are covered in file 12.

### Custom Images

- Create from a **stopped VM's disk**, an existing **snapshot**, or a **Cloud Storage file**
- Images belong to a project but can be shared across projects via IAM
- **Image families**: a logical group of images; `--image-family` always resolves to the latest non-deprecated image in that family

```bash
# Create image from stopped VM disk
gcloud compute images create my-custom-image --source-disk=MY_DISK --source-disk-zone=ZONE --family=my-app-v2
# Create from snapshot
gcloud compute images create my-image --source-snapshot=SNAPSHOT_NAME
# Deprecate old image
gcloud compute images deprecate OLD_IMAGE --state=DEPRECATED --replacement=NEW_IMAGE
# List images in a family
gcloud compute images list --filter="family=my-app-v2"
# Share image with another project
gcloud compute images add-iam-policy-binding my-image --member="serviceAccount:SA@OTHER_PROJECT.iam.gserviceaccount.com" --role=roles/compute.imageUser
```

---

## 10. Shielded VMs

> 📖 **Docs:** [Shielded VMs](https://cloud.google.com/compute/shielded-vm/docs/shielded-vm) | [Confidential VMs](https://cloud.google.com/compute/confidential-vm/docs/confidential-vm-overview) | 🖥️ **Console:** Compute Engine → VM instances → Create → Shielded VM

Shielded VMs protect against rootkits, bootkits, and firmware-level attacks by verifying the integrity of the VM's boot sequence.

### Three Security Features

| Feature | What It Does |
|---------|-------------|
| **Secure Boot** | Only signed OS kernels and drivers are allowed to load |
| **vTPM** (Virtual Trusted Platform Module) | Measures and records boot integrity; enables attestation |
| **Integrity Monitoring** | Compares each boot measurement against a known-good baseline; alerts on deviations |

### Enable at Creation

```bash
gcloud compute instances create my-vm --shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring
```

### Key Restrictions
- Not all machine types or images support Shielded VM — use Shielded-compatible images (e.g., `debian-cloud/debian-12`, `ubuntu-os-cloud/ubuntu-2204-lts`)
- Check image shielded support: `gcloud compute images list --filter="shieldedInstanceInitialState:*"`

> **Exam tip**: Shielded VM is a **prerequisite** for **Confidential Computing** (Confidential VMs). If a question asks about hardware-level memory encryption, the answer involves Confidential VMs, which require Shielded VM features.

---

## 8. Startup and Shutdown Scripts

> 📖 **Docs:** [Startup scripts](https://cloud.google.com/compute/docs/instances/startup-scripts) | [Shutdown scripts](https://cloud.google.com/compute/docs/instances/create-start-instance#startupscript) | 🖥️ **Console:** Compute Engine → VM instances → Create → Metadata → startup-script

### Startup Scripts
Run automatically when a VM boots:

```bash
# Inline startup script
gcloud compute instances create my-vm \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx'

# Startup script from Cloud Storage
gcloud compute instances create my-vm \
  --metadata=startup-script-url=gs://my-bucket/startup.sh

# Startup script from local file
gcloud compute instances create my-vm \
  --metadata-from-file=startup-script=./startup.sh
```

### Shutdown Scripts
Run when a VM is being stopped or deleted (best-effort, ~90 second window):

```bash
gcloud compute instances create my-vm \
  --metadata=shutdown-script='#!/bin/bash
    echo "Shutting down gracefully..."
    # Drain connections, save state, etc.'
```

### Metadata Server
- Every Compute Engine VM has access to a special internal metadata endpoint at `http://metadata.google.internal/computeMetadata/v1/`.
- Used to retrieve instance metadata, project metadata, and service account tokens without storing credentials on the VM.
- Required header: `Metadata-Flavor: Google`.

```bash
# From inside a VM: fetch its service account access token
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

---

## 12. Stopping vs. Deleting VM Instances

> 📖 **Docs:** [Stop or delete a VM](https://cloud.google.com/compute/docs/instances/stop-start-instance) | [Suspend a VM](https://cloud.google.com/compute/docs/instances/suspend-resume-instance) | 🖥️ **Console:** Compute Engine → VM instances → Stop / Delete

| Action | Billing | Disks | External IP (ephemeral) |
|--------|---------|-------|-------------------------|
| **Stop** | No VM CPU/RAM charges; disks still billed | Persist | Released (new one on restart) |
| **Suspend** | Reduced charge (memory saved) | Persist | Released |
| **Reset** | Still billed | Persist | Unchanged |
| **Delete** | None | Boot disk auto-deleted by default | Released |

```bash
gcloud compute instances stop my-vm --zone=ZONE
gcloud compute instances start my-vm --zone=ZONE
gcloud compute instances suspend my-vm --zone=ZONE
gcloud compute instances resume my-vm --zone=ZONE
gcloud compute instances reset my-vm --zone=ZONE
gcloud compute instances delete my-vm --zone=ZONE
```

- **Exam tip**: A stopped VM still incurs **disk** and **reserved static IP** charges. To eliminate all charges, delete the VM and release any reserved IP.

---

## Exam Practice Questions

1. **You need to deploy a fleet of identical web servers that automatically replace unhealthy instances. What should you create?**
   - Answer: Create an **instance template** defining the VM configuration, then create a **managed instance group** with a **health check** for autohealing.

2. **Your MIG needs to handle variable traffic, scaling between 2 and 20 instances based on CPU usage. How do you configure this?**
   - Answer: Use `gcloud compute instance-groups managed set-autoscaling` with `--min-num-replicas=2`, `--max-num-replicas=20`, and `--target-cpu-utilization=0.70`.

3. **A VM has a GPU attached. What must the maintenance policy be set to?**
   - Answer: `TERMINATE` — VMs with GPUs cannot be live-migrated, so the maintenance policy must be set to TERMINATE.

4. **You want to manage SSH access using corporate Google identities with sudo access and 2FA. How?**
   - Answer: Enable **OS Login** with 2FA (`enable-oslogin=TRUE`, `enable-oslogin-2fa=TRUE`), grant `roles/compute.osAdminLogin` for sudo access.

5. **You need to update a MIG's template without any downtime. How should you proceed?**
   - Answer: Create a new instance template, then use `rolling-action start-update` with `--max-unavailable=0` and `--max-surge=3` for a zero-downtime rolling update.

6. **How can you connect via SSH to a VM that has no external IP address?**
   - Answer: Use `gcloud compute ssh my-vm --tunnel-through-iap`. This tunnels the SSH connection through **Identity-Aware Proxy**. Ensure the IAP firewall rule allows TCP from `35.235.240.0/20`.

---

## Glossary

**A2 / A3 (Machine Family)** — Accelerator-optimized Compute Engine machine families with NVIDIA GPUs, used for ML training and HPC workloads.

**Arm (Architecture)** — A processor architecture used by GCP's T2A VM family (Ampere Altra), providing an alternative to x86 for scale-out workloads.

**Attestation** — The process by which a Shielded VM cryptographically proves the integrity of its boot sequence to a verifier using measurements recorded by the vTPM.

**Auto-Delete** — A Compute Engine disk setting controlling whether a persistent disk is automatically deleted when its attached VM is deleted. Boot disks default to auto-delete on; additional disks default to off.

**Autohealing** — A Managed Instance Group feature that automatically detects unhealthy VM instances via health checks and recreates them to restore the group to its desired state.

**Autoscaling** — A MIG feature that automatically increases or decreases the number of VM instances in the group based on configurable signals such as CPU utilization, HTTP LB utilization, Cloud Monitoring metrics, or a schedule.

**Availability Policy** — The configuration on a Compute Engine VM that determines behavior during host maintenance events: either live-migrate the VM (`MIGRATE`) or terminate it (`TERMINATE`).

**Backup** — A copy of data preserved for recovery purposes; for Compute Engine, backups are typically implemented as snapshots or snapshot schedules on persistent disks.

**Boot Disk** — The primary persistent disk attached to a VM that contains the operating system; every Compute Engine VM has exactly one boot disk.

**BYOL (Bring Your Own License)** — A licensing model allowing customers to use their own software licenses on cloud infrastructure, often requiring sole-tenant nodes for compliance.

**C2 / C3 (Machine Family)** — Compute-optimized Compute Engine machine families offering the highest per-thread CPU performance; ideal for HPC, gaming, and single-threaded workloads.

**Canary Update** — A rolling update strategy in which a new template version is deployed to a small percentage of instances first, allowing validation before full rollout.

**CI/CD (Continuous Integration / Continuous Delivery)** — A software development practice that automates code integration, testing, and deployment. Spot VMs are commonly used as cost-effective CI/CD workers.

**Cloud Logging** — GCP's managed service for collecting, storing, and analyzing log data from GCP services and applications; VM audit and serial console logs are available through Cloud Logging.

**Cloud Monitoring** — GCP's managed service for collecting, storing, and visualizing metrics and logs from GCP services and custom applications. Supports custom autoscaling metrics for MIGs.

**Committed Use Discount (CUD)** — A pricing model where customers commit to a specific amount of Compute Engine resources (vCPUs/memory) or spend for 1 or 3 years in exchange for significant discounts (up to 57% resource-based, 70% flex).

**Cloud Storage** — GCP's globally unified, scalable object storage service. Snapshots and custom images are stored in Cloud Storage buckets and billed at Cloud Storage rates.

**Compute Engine** — GCP's Infrastructure as a Service (IaaS) offering that provides virtual machine (VM) instances running on Google's infrastructure.

**Confidential Computing (Confidential VM)** — A Compute Engine feature that uses hardware-level memory encryption (AMD SEV) to protect data in use. Requires Shielded VM features to be enabled.

**Custom Image** — A VM boot disk image created by a user from a stopped VM's disk, a snapshot, or a Cloud Storage file. Can be shared across GCP projects via IAM.

**Custom Machine Type** — A Compute Engine machine type in which the user specifies an arbitrary number of vCPUs and amount of memory (e.g., `custom-6-20480`), rather than using a predefined configuration.

**Delete (VM)** — A Compute Engine operation that permanently removes a VM instance; boot disks with auto-delete enabled are also destroyed, and ephemeral IPs are released.

**Disk Image** — See Image / Custom Image: a read-only boot template used to create persistent disks for new VMs.

**e2 (Machine Family)** — A cost-optimized Compute Engine machine family suitable for general-purpose workloads. Includes `e2-micro`, `e2-standard-4`, etc.

**Ephemeral IP** — A non-reserved external IP address automatically assigned to a VM and released when the VM is stopped or deleted.

**ext4** — A widely used Linux filesystem type. Used as an example when formatting newly attached persistent disks inside a VM.

**fstab** — A Linux configuration file (`/etc/fstab`) that defines how disk partitions and other storage devices are automatically mounted at boot time.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services.

**gcloud** — The primary command-line tool for interacting with GCP services, part of the Google Cloud SDK.

**Google-Managed SSH Keys** — SSH public keys stored by GCP (either in project/instance metadata or via OS Login) used to grant SSH access to Compute Engine VMs.

**GPU (Graphics Processing Unit)** — A specialized processor used for parallel computation, machine learning, and graphics. VMs with GPUs must use `--maintenance-policy=TERMINATE` because they cannot be live-migrated.

**Health Check** — A probe that periodically tests the responsiveness of a VM or service endpoint. Used by MIGs for autohealing and by load balancers for traffic routing.

**HPC (High-Performance Computing)** — Computational workloads requiring large amounts of processing power, often parallelized. Spot VMs are cost-effective for HPC batch jobs.

**IAM (Identity and Access Management)** — GCP's system for controlling who can do what on which resources. OS Login uses IAM roles to control SSH access to VMs.

**IAP (Identity-Aware Proxy)** — A GCP service that enables secure access to GCP resources without exposing them to the public internet. Used to tunnel SSH connections to VMs that have no external IP.

**Image** — A boot template containing an operating system and any pre-installed software used to create the boot disk of a Compute Engine VM. Public images are provided by Google and partners; custom images are user-created.

**Image Family** — A logical grouping of Compute Engine images; the `--image-family` flag always resolves to the latest non-deprecated image within that family.

**Instance (Compute Engine)** — A single virtual machine running on Google infrastructure, created from an image and configured with a machine type, disks, network interfaces, and metadata.

**Instance Template** — A read-only, immutable resource that defines the configuration (machine type, disk, image, network, metadata, service account) used to create VM instances in a Managed Instance Group.

**Integrity Monitoring** — A Shielded VM feature that compares each boot's integrity measurements against a known-good baseline and alerts on deviations, detecting tampering.

**IOPS (Input/Output Operations Per Second)** — A measure of storage device performance. Higher IOPS persistent disk types (`pd-ssd`, `pd-extreme`) are used for databases and random I/O workloads.

**Label** — A key-value metadata pair attached to GCP resources for organization, cost attribution, and filtering (e.g., `environment=production`).

**Live Migration** — A Compute Engine feature that moves a running VM to a different physical host during maintenance events with no VM downtime. Not supported for VMs with GPUs or Local SSDs.

**M1 / M2 / M3 (Machine Family)** — Memory-optimized Compute Engine machine families offering high memory-to-vCPU ratios; ideal for SAP HANA and large in-memory database workloads.

**Local SSD** — A physically attached NVMe SSD on the Compute Engine host that provides the highest storage performance. Data is lost when the VM is stopped or terminated.

**Machine Family** — A Compute Engine classification grouping machine types by workload optimization (e.g., E2 general-purpose, C2 compute-optimized, M2 memory-optimized, A2 accelerator-optimized).

**Machine Type** — A predefined or custom configuration specifying the number of vCPUs and amount of RAM for a Compute Engine VM (e.g., `e2-standard-4`, `n2-standard-8`).

**Metadata Server** — An internal HTTP endpoint (`metadata.google.internal`) available to every Compute Engine VM that exposes instance and project metadata, service account tokens, and startup scripts; requires the `Metadata-Flavor: Google` header.

**Managed Instance Group (MIG)** — A group of identical Compute Engine VM instances created from an instance template, managed as a single entity with support for autoscaling, autohealing, and rolling updates.

**Metadata** — Key-value pairs associated with a GCP project or VM instance. Used to pass configuration data (e.g., startup scripts) to VMs at boot time.

**n1 / n2 (Machine Family)** — General-purpose Compute Engine machine families. n1 is the first generation; n2 uses Intel Cascade Lake processors with better performance. Both support GPUs.

**n2d (Machine Family)** — A general-purpose Compute Engine machine family based on AMD EPYC processors; offers an alternative to the Intel-based n2 family.

**Network Tag** — A string label attached to a VM used as a target or source identifier for firewall rules; applied via `--tags=`.

**nginx** — A high-performance open-source web server and reverse proxy. Used in examples as a package installed by startup scripts on Compute Engine VMs.

**OS Config Agent** — A software agent that must be installed on VMs to enable VM Manager features (patch management, inventory). Pre-installed on many Google-provided images.

**OS Inventory Management** — A VM Manager component that collects OS version, installed package, and system configuration data from Compute Engine VMs for fleet-wide visibility.

**OS Login** — A Compute Engine feature that links a user's Linux account to their Google identity for SSH access, managed via IAM roles. Supports 2FA and provides audit logs.

**OS Patch Management** — A VM Manager component that schedules and applies OS patches to Compute Engine VMs, including patch windows, compliance reporting, and recurring deployments.

**Patch Deployment** — A VM Manager configuration that defines a recurring schedule for applying OS patches to a fleet of VMs (e.g., weekly on Sundays at 02:00).

**Permission** — A specific action that can be performed on a GCP resource (e.g., `compute.instances.create`); permissions are grouped into roles and granted via IAM.

**pd-balanced** — A Compute Engine persistent disk type offering a balance of performance and cost; the default disk type for most workloads.

**pd-extreme** — The highest-performance Compute Engine persistent disk type, supporting up to 120,000 IOPS. Used for mission-critical databases.

**pd-ssd** — A Compute Engine persistent disk type backed by solid-state drives, offering high IOPS (30 R/W per GB) for databases and latency-sensitive workloads.

**pd-standard** — A Compute Engine persistent disk type backed by hard disk drives, offering the lowest cost for sequential workloads like logs and bulk storage.

**Persistent Disk** — A network-attached block storage device for Compute Engine VMs. Data persists independently of the VM lifecycle (unless auto-delete is enabled).

**Preemptible VM** — A legacy Compute Engine pricing model offering VMs at up to 91% discount. VMs can be preempted at any time with a 30-second notice and have a maximum runtime of 24 hours. Superseded by Spot VMs.

**Principal** — An identity (user, group, service account, or domain) to which IAM roles can be granted.

**Project** — A GCP resource container that holds resources, IAM bindings, billing, and quotas; every Compute Engine VM belongs to exactly one project.

**Pub/Sub** — GCP's managed, scalable messaging service. Used to receive notifications about Spot VM preemption events.

**Read-Only Mode (Persistent Disk)** — A mode in which a persistent disk can be attached to multiple VMs simultaneously for shared read access. Write access requires attaching to only one VM.

**Region** — A specific geographic location where GCP resources are hosted (e.g., `us-central1`). Contains multiple isolated zones.

**Regional MIG** — A Managed Instance Group that distributes VM instances across multiple zones within a single region, providing higher availability than a zonal MIG.

**Reset (VM)** — A Compute Engine operation that forcibly restarts a VM without a clean shutdown; the VM remains on the same host and billing continues.

**Resource** — Any addressable entity within GCP (e.g., a VM instance, disk, image, snapshot). Resources belong to a project and are governed by IAM.

**Role** — A named collection of IAM permissions (e.g., `roles/compute.osLogin`) granted to principals to authorize actions on GCP resources.

**Rolling Update** — A MIG update strategy that replaces instances incrementally using a new instance template, enabling zero-downtime deployments. Controlled by `--max-surge` and `--max-unavailable` parameters.

**Secure Boot** — A Shielded VM feature that ensures only digitally signed OS kernels and boot components are loaded, protecting against bootkits and rootkits.

**Service Account** — A special GCP identity used by applications (rather than humans) to authenticate API calls. Assigned to VM instances to grant them GCP API access.

**Shielded VM** — A Compute Engine security feature that uses Secure Boot, vTPM, and Integrity Monitoring to protect VMs against firmware-level attacks, rootkits, and bootkits.

**Shutdown Script** — A script specified via metadata that runs (best-effort, ~90 seconds) when a Compute Engine VM is stopped, suspended, or deleted.

**Snapshot** — A point-in-time, incremental backup of a Compute Engine persistent disk stored in Cloud Storage. The first snapshot is a full backup; subsequent ones capture only changed blocks.

**Snapshot Schedule** — A Compute Engine resource that automatically creates recurring snapshots of one or more persistent disks on a defined cadence (hourly, daily, weekly).

**Sole-Tenant Node** — A dedicated physical server in GCP that runs only your VMs, providing physical isolation for licensing compliance (BYOL), security, or performance requirements.

**Spot VM** — The current Compute Engine pricing model for fault-tolerant workloads, offering the same deep discounts as preemptible VMs but with no 24-hour runtime limit. Can be preempted at any time with 30 seconds notice.

**SSH (Secure Shell)** — A cryptographic network protocol for secure remote login to and command execution on VMs. GCP provides several methods for SSH key management.

**Startup Script** — A script (inline, from a file, or from Cloud Storage) that runs automatically when a Compute Engine VM boots. Specified via instance metadata.

**Static IP Address** — A reserved IP (internal or external) that persists independently of the VM; charged when unassigned to discourage waste.

**Stop (VM)** — A Compute Engine operation that shuts down a VM while preserving its configuration and disks; no CPU/memory charges accrue during the stopped state, but disk and reserved IP charges continue.

**Subnet** — A regional IP address range within a VPC network. VMs are created within subnets and receive internal IP addresses from the subnet's range.

**Suspend (VM)** — A Compute Engine operation that pauses a VM while preserving its in-memory state to a persistent disk; reduced billing applies and the VM can be resumed.

**Sustained Use Discount (SUD)** — An automatic Compute Engine discount applied when a VM runs for a significant portion of a billing month; requires no commitment or action.

**T2A / T2D (Machine Family)** — Scale-out optimized Compute Engine machine families (Arm-based T2A and AMD-based T2D) designed for horizontally scalable workloads such as web servers and containerized microservices.

**vCPU (virtual CPU)** — The unit of CPU capacity on a Compute Engine VM, corresponding to a single hardware hyperthread; machine type sizes are expressed in vCPUs and RAM.

**VPC (Virtual Private Cloud)** — GCP's global, software-defined private network providing networking for VM instances, including subnets, firewall rules, and routes.

**vTPM (Virtual Trusted Platform Module)** — A virtualized hardware security module used in Shielded VMs to measure and record boot integrity, enabling attestation.

**VM Manager** — A suite of GCP tools for managing operating systems on large fleets of Compute Engine VMs, including OS Patch Management and OS Inventory Management.

**Zone** — An isolated deployment area within a GCP region (e.g., `us-central1-a`). Compute Engine VMs are zonal resources.

**Zonal MIG** — A Managed Instance Group in which all VM instances reside in a single zone. Simpler than a regional MIG but susceptible to zone-level failures.
