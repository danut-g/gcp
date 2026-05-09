# Section 4.1 — Managing Compute Engine Resources

## Exam Relevance
This topic is part of **Section 4: Ensuring successful operation of a cloud solution (~20 % of the exam)**. You must know how to remotely connect to instances, view VM inventory, work with snapshots, and work with images.

---

## 1. Remotely Connecting to Instances

> 📖 **Docs:** [Connect to Linux VMs](https://cloud.google.com/compute/docs/connect/ssh-linux) | [IAP for TCP tunneling](https://cloud.google.com/iap/docs/using-tcp-forwarding) | 🖥️ **Console:** Compute Engine → VM instances → SSH button

### SSH Connections (Linux VMs)

```bash
# Standard SSH via gcloud (recommended)
gcloud compute ssh my-vm --zone=us-central1-a

# SSH with a specific user
gcloud compute ssh alice@my-vm --zone=us-central1-a

# SSH through IAP tunnel (no external IP needed)
gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap

# SSH with port forwarding
gcloud compute ssh my-vm --zone=us-central1-a \
  --ssh-flag="-L 8080:localhost:80"

# SSH and run a command
gcloud compute ssh my-vm --zone=us-central1-a \
  --command="sudo systemctl status nginx"

# SCP files to/from a VM
gcloud compute scp local-file.txt my-vm:~/remote-file.txt --zone=us-central1-a
gcloud compute scp my-vm:~/remote-file.txt local-file.txt --zone=us-central1-a

# SCP a directory
gcloud compute scp --recurse my-vm:~/logs/ ./local-logs/ --zone=us-central1-a
```

### RDP Connections (Windows VMs)

```bash
# Set/reset Windows password
gcloud compute reset-windows-password my-windows-vm \
  --zone=us-central1-a \
  --user=admin

# This returns: IP address, username, and password
# Use these with an RDP client (Remote Desktop Connection)
```

### Serial Console Access

```bash
# Connect to serial console (troubleshooting — when SSH doesn't work)
gcloud compute connect-to-serial-port my-vm --zone=us-central1-a

# View serial port output (boot logs)
gcloud compute instances get-serial-port-output my-vm --zone=us-central1-a
```

### Connection Methods Summary

| Method | When to Use | External IP Required? |
|--------|-------------|----------------------|
| `gcloud compute ssh` | Standard SSH | Yes (or use IAP) |
| IAP tunnel | VM has no external IP | No |
| Serial console | SSH/RDP not working, boot issues | No |
| RDP | Windows VMs | Yes |
| Cloud Console SSH | Browser-based SSH | Yes (or use IAP) |

---

## 2. Viewing Current Running VM Inventory

> 📖 **Docs:** [List VM instances](https://cloud.google.com/compute/docs/instances/view-vms) | [Filter compute instances](https://cloud.google.com/sdk/gcloud/reference/compute/instances/list) | 🖥️ **Console:** Compute Engine → VM instances

### Listing Instances

```bash
# List all VMs in the project
gcloud compute instances list

# List VMs in a specific zone
gcloud compute instances list --filter="zone:us-central1-a"

# List only running VMs
gcloud compute instances list --filter="status=RUNNING"

# List VMs with specific labels
gcloud compute instances list --filter="labels.environment=production"

# Format output as table with specific columns
gcloud compute instances list \
  --format="table(name, zone, machineType.basename(), status, networkInterfaces[0].networkIP)"

# Output as JSON
gcloud compute instances list --format=json

# Output as CSV
gcloud compute instances list --format=csv

# List VMs sorted by creation time
gcloud compute instances list --sort-by=creationTimestamp
```

### Describing Instances

```bash
# Get detailed information about a VM
gcloud compute instances describe my-vm --zone=us-central1-a

# Get specific fields
gcloud compute instances describe my-vm \
  --zone=us-central1-a \
  --format="get(networkInterfaces[0].networkIP)"

# Get the external IP
gcloud compute instances describe my-vm \
  --zone=us-central1-a \
  --format="get(networkInterfaces[0].accessConfigs[0].natIP)"

# Get the machine type
gcloud compute instances describe my-vm \
  --zone=us-central1-a \
  --format="get(machineType)"
```

### Instance Details (Key Properties)

| Property | Description |
|----------|-------------|
| **Instance ID** | Unique numeric identifier |
| **Name** | User-defined name |
| **Zone** | Zone where the VM is running |
| **Machine Type** | CPU/memory configuration |
| **Status** | RUNNING, STOPPED, TERMINATED, STAGING, etc. |
| **Internal IP** | Private IP address |
| **External IP** | Public IP (if assigned) |
| **Disks** | Boot disk and additional disks |
| **Network Tags** | Tags for firewall rules |
| **Service Account** | Attached service account |
| **Labels** | Key-value metadata for organization |
| **Metadata** | Custom key-value data (startup scripts, etc.) |

### Managing VM State

```bash
# Stop a VM (preserves disk, releases CPU/memory)
gcloud compute instances stop my-vm --zone=us-central1-a

# Start a stopped VM
gcloud compute instances start my-vm --zone=us-central1-a

# Reset (hard reboot) a VM
gcloud compute instances reset my-vm --zone=us-central1-a

# Suspend a VM (save memory state to disk)
gcloud compute instances suspend my-vm --zone=us-central1-a

# Resume a suspended VM
gcloud compute instances resume my-vm --zone=us-central1-a

# Delete a VM
gcloud compute instances delete my-vm --zone=us-central1-a

# Delete a VM but keep its boot disk
gcloud compute instances delete my-vm --zone=us-central1-a --keep-disks=boot
```

### Modifying a VM

```bash
# Change machine type (VM must be stopped)
gcloud compute instances set-machine-type my-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-8

# Add/update labels
gcloud compute instances add-labels my-vm \
  --zone=us-central1-a \
  --labels=environment=staging,owner=alice

# Remove labels
gcloud compute instances remove-labels my-vm \
  --zone=us-central1-a \
  --labels=owner

# Add metadata
gcloud compute instances add-metadata my-vm \
  --zone=us-central1-a \
  --metadata=key1=value1

# Update tags
gcloud compute instances add-tags my-vm \
  --zone=us-central1-a \
  --tags=new-tag

# Move a VM to a different zone (within the same region)
gcloud compute instances move my-vm \
  --zone=us-central1-a \
  --destination-zone=us-central1-b
```

---

## 3. Working with Snapshots

> 📖 **Docs:** [Create disk snapshots](https://cloud.google.com/compute/docs/disks/create-snapshots) | [Snapshot schedules](https://cloud.google.com/compute/docs/disks/scheduled-snapshots) | 🖥️ **Console:** Compute Engine → Snapshots → Create snapshot

### What Are Snapshots?
- **Point-in-time copies** of persistent disk data
- Used for **backups**, **disaster recovery**, and **disk cloning**
- **Incremental** — Only changes since the last snapshot are stored (saves space and cost)
- **Global resource** — Can be used to create disks in any region
- Stored in Cloud Storage (managed by Google)

### Creating Snapshots

```bash
# Create a snapshot of a disk
gcloud compute snapshots create my-snapshot \
  --source-disk=my-vm \
  --source-disk-zone=us-central1-a \
  --description="Backup before upgrade"

# Create a snapshot with labels
gcloud compute snapshots create my-snapshot \
  --source-disk=my-disk \
  --source-disk-zone=us-central1-a \
  --labels=purpose=backup,created-by=admin

# Create a snapshot of a specific disk (not boot disk)
gcloud compute snapshots create data-snapshot \
  --source-disk=my-data-disk \
  --source-disk-zone=us-central1-a

# Create a snapshot and store it in a specific location
gcloud compute snapshots create my-snapshot \
  --source-disk=my-disk \
  --source-disk-zone=us-central1-a \
  --storage-location=us
```

### Viewing Snapshots

```bash
# List all snapshots
gcloud compute snapshots list

# Describe a snapshot
gcloud compute snapshots describe my-snapshot

# Filter snapshots by label
gcloud compute snapshots list --filter="labels.purpose=backup"

# List snapshots sorted by creation time
gcloud compute snapshots list --sort-by=creationTimestamp
```

### Deleting Snapshots

```bash
# Delete a snapshot
gcloud compute snapshots delete my-snapshot

# Delete multiple snapshots
gcloud compute snapshots delete snapshot-1 snapshot-2 snapshot-3
```

**Important**: Deleting a snapshot does not affect the disk or any disks created from it. Google automatically manages the incremental chain — deleting a middle snapshot transfers the data to the next snapshot.

### Scheduling Snapshots

Snapshot schedules automate regular backups:

```bash
# Create a snapshot schedule
gcloud compute resource-policies create snapshot-schedule my-schedule \
  --region=us-central1 \
  --max-retention-days=14 \
  --on-source-disk-delete=keep-auto-snapshots \
  --daily-schedule \
  --start-time=02:00 \
  --storage-location=us

# Create a weekly schedule
gcloud compute resource-policies create snapshot-schedule weekly-backup \
  --region=us-central1 \
  --max-retention-days=30 \
  --weekly-schedule-from-file=schedule.json \
  --storage-location=us

# Create an hourly schedule
gcloud compute resource-policies create snapshot-schedule hourly-backup \
  --region=us-central1 \
  --max-retention-days=7 \
  --hourly-schedule=4 \
  --start-time=00:00

# Attach schedule to a disk
gcloud compute disks add-resource-policies my-disk \
  --resource-policies=my-schedule \
  --zone=us-central1-a

# Remove schedule from a disk
gcloud compute disks remove-resource-policies my-disk \
  --resource-policies=my-schedule \
  --zone=us-central1-a

# List snapshot schedules
gcloud compute resource-policies list --filter="region:us-central1"
```

### Creating a Disk from a Snapshot

```bash
# Create a new disk from a snapshot
gcloud compute disks create new-disk \
  --source-snapshot=my-snapshot \
  --zone=us-central1-a \
  --type=pd-ssd

# Create a disk in a different zone/region (cross-region restore)
gcloud compute disks create dr-disk \
  --source-snapshot=my-snapshot \
  --zone=europe-west1-b
```

---

## 4. Working with Images

> 📖 **Docs:** [Images overview](https://cloud.google.com/compute/docs/images) | [Create custom images](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images) | 🖥️ **Console:** Compute Engine → Images

### What Are Images?
- **Boot disk templates** containing the operating system and software configuration
- Used to **create new VMs** or **new boot disks**
- Types: **Public images** (Google-provided) and **Custom images** (user-created)

### Public Images

| Image Family | Project | OS |
|-------------|---------|-----|
| `debian-12` | `debian-cloud` | Debian 12 |
| `ubuntu-2204-lts` | `ubuntu-os-cloud` | Ubuntu 22.04 LTS |
| `centos-stream-9` | `centos-cloud` | CentOS Stream 9 |
| `rhel-9` | `rhel-cloud` | Red Hat Enterprise Linux 9 |
| `windows-2022` | `windows-cloud` | Windows Server 2022 |
| `cos-stable` | `cos-cloud` | Container-Optimized OS |

```bash
# List available public images
gcloud compute images list

# List images from a specific project
gcloud compute images list --project=debian-cloud

# Describe an image
gcloud compute images describe debian-12-bookworm-v20240312 --project=debian-cloud
```

### Creating Custom Images

```bash
# Create an image from a VM's boot disk (VM should be stopped)
gcloud compute images create my-custom-image \
  --source-disk=my-vm \
  --source-disk-zone=us-central1-a \
  --description="Custom image with nginx and app"

# Create an image from a snapshot
gcloud compute images create my-image-from-snapshot \
  --source-snapshot=my-snapshot \
  --description="Image from backup snapshot"

# Create an image from a file in Cloud Storage
gcloud compute images create my-imported-image \
  --source-uri=gs://my-bucket/my-image.tar.gz

# Create an image with a specific family
gcloud compute images create my-app-v2 \
  --source-disk=my-vm \
  --source-disk-zone=us-central1-a \
  --family=my-app-family

# Deprecate an old image
gcloud compute images deprecate my-app-v1 \
  --state=DEPRECATED \
  --replacement=my-app-v2
```

### Image Families
- A **family** is a group of related images (e.g., different versions of your app)
- When you reference a family, you always get the **latest non-deprecated** image
- Use families in instance templates for automatic image updates

```bash
# Create a VM using the latest image from a family
gcloud compute instances create my-vm \
  --image-family=my-app-family \
  --image-project=PROJECT_ID \
  --zone=us-central1-a
```

### Viewing Images

```bash
# List custom images in your project
gcloud compute images list --no-standard-images

# List images in a specific family
gcloud compute images list --filter="family=my-app-family"

# Describe a custom image
gcloud compute images describe my-custom-image
```

### Deleting Images

```bash
# Delete an image
gcloud compute images delete my-custom-image

# Delete multiple images
gcloud compute images delete old-image-1 old-image-2
```

### Snapshots vs. Images

| Feature | Snapshot | Image |
|---------|----------|-------|
| **Purpose** | Backup and restore disks | Create new VMs/boot disks |
| **Scope** | Any persistent disk (boot or data) | Boot disk only |
| **Incremental** | Yes | No (full copy) |
| **Cross-region** | Yes | Yes |
| **Families** | No | Yes |
| **Use case** | Disaster recovery, backup | VM golden images, templates |
| **Speed** | Slower to create VM from | Faster to create VM from |

---

## 8. Managed Instance Groups (MIGs)

> 📖 **Docs:** [Working with MIGs](https://cloud.google.com/compute/docs/instance-groups/working-with-managed-instances) | [Rolling updates](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups) | 🖥️ **Console:** Compute Engine → Instance groups

- Group of identical VMs created from an instance template
- Stateless MIGs: auto-scaling, auto-healing, rolling updates (web frontends, batch)
- Stateful MIGs: preserve per-instance state (disks, metadata, IPs) for databases, stateful apps

```bash
# Create MIG
gcloud compute instance-groups managed create my-mig \
  --template=MY_TEMPLATE \
  --size=3 \
  --zone=us-central1-a

# Enable autoscaling
gcloud compute instance-groups managed set-autoscaling my-mig \
  --zone=us-central1-a \
  --max-num-replicas=10 \
  --min-num-replicas=2 \
  --target-cpu-utilization=0.7

# Set autohealing
gcloud compute instance-groups managed set-autohealing my-mig \
  --zone=us-central1-a \
  --health-check=MY_HEALTH_CHECK \
  --initial-delay=300

# Rolling update
gcloud compute instance-groups managed rolling-action start-update my-mig \
  --version=template=NEW_TEMPLATE \
  --zone=us-central1-a \
  --max-surge=1 \
  --max-unavailable=0

# List instances in MIG
gcloud compute instance-groups managed list-instances my-mig --zone=us-central1-a
```

- Regional MIG: distributes instances across zones in a region for HA

```bash
gcloud compute instance-groups managed create my-regional-mig --template=MY_TEMPLATE --size=3 --region=us-central1
```

- **Exam tip**: MIGs + Load Balancer is the standard pattern for scalable, highly available stateless services

---

## 9. Startup and Shutdown Scripts

> 📖 **Docs:** [Startup scripts](https://cloud.google.com/compute/docs/instances/startup-scripts/linux) | [Shutdown scripts](https://cloud.google.com/compute/docs/instances/create-start-instance#startupscript) | 🖥️ **Console:** Compute Engine → VM instances → Edit → Metadata

- Startup script: runs as root when VM boots (after OS loads)
- Shutdown script: runs when VM receives stop/preemption signal (30-second window)
- Pass via metadata:

```bash
# Inline
gcloud compute instances create my-vm \
  --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install -y nginx'
# From file
gcloud compute instances create my-vm \
  --metadata-from-file=startup-script=/local/path/startup.sh
# From Cloud Storage
gcloud compute instances create my-vm \
  --metadata=startup-script-url=gs://my-bucket/startup.sh
# Update metadata on running instance
gcloud compute instances add-metadata my-vm --metadata-from-file=startup-script=/path/script.sh
```

- View startup script logs: `sudo journalctl -u google-startup-scripts.service`
- **Exam tip**: Startup scripts are idempotent by design; use for software installation, config application

---

## 10. Instance Templates

> 📖 **Docs:** [Instance templates](https://cloud.google.com/compute/docs/instance-templates) | [Create instance templates](https://cloud.google.com/compute/docs/instance-templates/create-instance-templates) | 🖥️ **Console:** Compute Engine → Instance templates → Create instance template

- Immutable resource defining VM configuration (machine type, image, disks, network, metadata, tags, service account) used by MIGs and single VM creation
- Cannot edit an existing template — create a new one and swap it into the MIG via a rolling update

```bash
# Create an instance template
gcloud compute instance-templates create my-template \
  --machine-type=e2-standard-4 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=http-server \
  --metadata-from-file=startup-script=startup.sh \
  --service-account=my-sa@PROJECT.iam.gserviceaccount.com \
  --scopes=cloud-platform

# List templates
gcloud compute instance-templates list

# Describe a template
gcloud compute instance-templates describe my-template

# Delete a template (only if not in use)
gcloud compute instance-templates delete my-template

# Create VM from template
gcloud compute instances create my-vm \
  --source-instance-template=my-template \
  --zone=us-central1-a
```

- **Exam tip**: To change a MIG's configuration, create a new instance template and start a rolling update with `--version=template=NEW_TEMPLATE`.

---

## 11. OS Login

> 📖 **Docs:** [OS Login overview](https://cloud.google.com/compute/docs/oslogin) | [Manage OS Login in org](https://cloud.google.com/compute/docs/oslogin/manage-oslogin-in-an-org) | 🖥️ **Console:** Compute Engine → Metadata → enable-oslogin = TRUE

- OS Login ties Linux SSH access to Google identities (IAM) instead of manually managed SSH keys in metadata
- Enables centralized SSH key management, audit logging, and fine-grained access via IAM roles
- Enable per-project or per-instance via metadata: `enable-oslogin=TRUE`

```bash
# Enable OS Login at project level
gcloud compute project-info add-metadata \
  --metadata enable-oslogin=TRUE

# Enable OS Login on a single instance
gcloud compute instances add-metadata my-vm \
  --zone=us-central1-a \
  --metadata=enable-oslogin=TRUE

# Grant OS Login (non-sudo) access
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:alice@example.com" \
  --role="roles/compute.osLogin"

# Grant OS Login with sudo (admin) access
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:bob@example.com" \
  --role="roles/compute.osAdminLogin"
```

| Role | Access |
|------|--------|
| `roles/compute.osLogin` | SSH as a non-sudo user |
| `roles/compute.osAdminLogin` | SSH with sudo privileges |

- **Exam tip**: OS Login is the recommended way to manage SSH access at scale. With OS Login enabled, SSH keys stored in instance/project metadata are ignored.

---

## 12. VM Metadata Server

> 📖 **Docs:** [VM metadata](https://cloud.google.com/compute/docs/metadata/overview) | [Querying metadata](https://cloud.google.com/compute/docs/metadata/querying-metadata) | 🖥️ **Console:** n/a (accessed from within VM via curl)

- Every Compute Engine VM can query a special internal URL: `http://metadata.google.internal/computeMetadata/v1/`
- Returns VM-specific and project-specific data: instance name, zone, service account tokens, custom metadata (including startup scripts)
- Accessing requires the header `Metadata-Flavor: Google`

```bash
# Get instance name
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/name

# Get zone
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/zone

# Get default service account access token
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token

# Get custom metadata value
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/my-key
```

- **Exam tip**: Applications on GCE should obtain short-lived OAuth tokens from the metadata server rather than using downloaded service account key files.

---

## 13. Spot VMs and Preemptible VMs

> 📖 **Docs:** [Spot VMs](https://cloud.google.com/compute/docs/instances/spot) | [Handle preemption](https://cloud.google.com/compute/docs/instances/spot#handle-interruption) | 🖥️ **Console:** Compute Engine → VM instances → Create → Availability policies → Spot

- Spot VMs (current generation) and Preemptible VMs (legacy) use spare GCP capacity at a deep discount (~60-91% off)
- Can be reclaimed at any time with a 30-second shutdown signal
- No SLA, no guaranteed uptime; suitable for fault-tolerant batch workloads, CI/CD, stateless worker pools
- Preemptible VMs have a 24-hour max lifetime; Spot VMs have no max lifetime

```bash
# Create a Spot VM
gcloud compute instances create my-spot-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# Create a legacy preemptible VM
gcloud compute instances create my-preemptible-vm \
  --zone=us-central1-a \
  --preemptible
```

- **Exam tip**: Use a shutdown script to gracefully handle preemption within the 30-second window.

---

## Exam Practice Questions

1. **You need to create a backup of a production VM's boot disk before performing an upgrade. What should you do?**
   - Answer: Create a **snapshot** of the boot disk: `gcloud compute snapshots create pre-upgrade-backup --source-disk=my-vm --source-disk-zone=us-central1-a`.

2. **Your team needs daily automated backups of a disk with 14-day retention. How should you set this up?**
   - Answer: Create a **snapshot schedule** with `--daily-schedule`, `--max-retention-days=14`, then attach it to the disk with `gcloud compute disks add-resource-policies`.

3. **You need to create identical VMs across multiple zones using a standardized OS with your application pre-installed. What approach should you use?**
   - Answer: Create a **custom image** from a configured VM (with your app installed), assign it to an **image family**, then use the image family in an **instance template** for a regional MIG.

4. **How can you connect to a VM that is not booting properly (SSH is not working)?**
   - Answer: Use the **serial console**: `gcloud compute connect-to-serial-port my-vm --zone=us-central1-a`. This lets you see boot logs and interact with the console.

5. **You need to change a VM's machine type from e2-standard-4 to e2-standard-8. Can you do this while the VM is running?**
   - Answer: **No**. You must stop the VM first, change the machine type, then start it again: `gcloud compute instances stop`, `gcloud compute instances set-machine-type`, `gcloud compute instances start`.

6. **You deleted a snapshot that was part of an incremental chain. Will other snapshots be affected?**
   - Answer: **No**. Google automatically manages incremental snapshot data. When a snapshot is deleted, any data needed by later snapshots is transferred to them automatically.

---

## Glossary

**Auto-healing** — A MIG feature that automatically detects unhealthy VM instances using a health check and replaces them without manual intervention.

**Autoscaling** — The automatic adjustment of the number of VM instances in a MIG based on load metrics such as CPU utilization, to match demand while controlling cost.

**Billing** — The GCP cost accounting system; snapshot storage, disk usage, and VM runtime all incur billing against a linked billing account.

**Boot Disk** — The primary persistent disk attached to a Compute Engine VM that contains the operating system; used as the source for creating custom images.

**CentOS Stream** — A community-supported Linux distribution available as a public Compute Engine image; a continuously delivered upstream branch of RHEL.

**Cloud Console** — The web-based graphical user interface for GCP that provides a browser-based SSH option (Cloud Console SSH) as an alternative to the gcloud CLI for accessing VMs.

**Cloud Storage** — GCP's object storage service; snapshots are stored by Google in Cloud Storage, and startup scripts can be stored and referenced from Cloud Storage buckets.

**Compute Engine** — GCP's Infrastructure-as-a-Service (IaaS) offering for creating and managing virtual machine instances.

**Container-Optimized OS (cos-stable)** — A Google-maintained Linux OS image optimized for running Docker containers on Compute Engine VMs.

**CPU Utilization** — The percentage of CPU capacity being used by a VM; used as a scaling metric for MIG autoscaling (e.g., `--target-cpu-utilization=0.7`).

**Cross-Region Restore** — The ability to create a disk in a different region from a snapshot, enabling disaster recovery in an alternate geographic location.

**curl** — A command-line tool for transferring data using URL syntax; used from within a VM to query the metadata server with the `Metadata-Flavor: Google` header.

**Custom Image** — A user-created Compute Engine boot disk image containing a preconfigured operating system and software stack, used as a template for creating new VMs.

**Debian** — A popular open-source Linux distribution available as a public Compute Engine image (e.g., `debian-12` in the `debian-cloud` project).

**Deprecated (image state)** — An image marked with the `DEPRECATED` state so that it is still usable but flagged as obsolete; used with `--replacement` to point users to a newer image.

**Disaster Recovery (DR)** — A set of policies, tools, and procedures for recovering infrastructure after a failure; snapshots are a key tool for DR on Compute Engine.

**External IP** — A publicly routable IP address assigned to a VM instance, enabling direct communication with the internet.

**Firewall Rule** — A VPC configuration that allows or denies network traffic to/from VMs based on source/destination, protocol, port, and target (service accounts or network tags).

**gcloud** — Google Cloud's primary command-line tool for interacting with GCP services; the `gcloud compute` subcommand group manages Compute Engine resources.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services, including Compute Engine and related infrastructure services.

**GKE (Google Kubernetes Engine)** — GCP's managed Kubernetes service; referenced as a consumer of Compute Engine VM infrastructure.

**Golden Image** — A custom VM image that has been configured, hardened, and validated as the standard template for deploying new VM instances across an environment.

**HA (High Availability)** — A design goal achieved by distributing VM instances across multiple zones using regional MIGs, ensuring continued operation if one zone fails.

**Health Check** — A GCP resource that periodically probes a VM or backend to determine if it is healthy; used by MIGs for auto-healing.

**IAM (Identity and Access Management)** — GCP's access control system for granting principals (users, groups, service accounts) permission to perform actions on resources, including VM access via OS Login roles.

**IAP (Identity-Aware Proxy)** — A GCP service that enables SSH/RDP access to VMs over an encrypted tunnel without requiring a public external IP address (via `--tunnel-through-iap`).

**ICMP (Internet Control Message Protocol)** — A network diagnostic protocol; used in startup scripts and firewall rule context to test VM connectivity.

**Idempotent** — A property of a script or operation such that running it multiple times produces the same result as running it once; startup scripts should be idempotent because they may run on every boot.

**Image Family** — A named group of related Compute Engine images (e.g., different versions of a custom application image) where referencing the family always returns the latest non-deprecated image.

**Incremental Snapshot** — A snapshot that stores only the data changed since the previous snapshot, reducing storage consumption and cost; all GCP snapshots after the first are incremental.

**Instance ID** — A unique numeric identifier automatically assigned to each Compute Engine VM instance by GCP.

**Instance Template** — An immutable resource that defines the configuration (machine type, image, disk, network, metadata, service account, tags) for VM instances, used by Managed Instance Groups and for individual VM creation.

**Internal IP** — A private IP address assigned to a VM within its VPC subnet, used for communication within the VPC network.

**IOPS (Input/Output Operations Per Second)** — A measure of storage performance; pd-ssd persistent disks provide higher IOPS than pd-standard, relevant when choosing disk types for workloads.

**JSON (JavaScript Object Notation)** — A text-based structured data format; used with `gcloud ... --format=json` to output machine-readable details about Compute Engine resources.

**journalctl** — A Linux systemd command used to query and display logs from the systemd journal; used to inspect startup-script execution via `sudo journalctl -u google-startup-scripts.service`.

**Label** — A key-value metadata tag applied to GCP resources (including VMs and snapshots) used for organization, filtering, cost attribution, and automation.

**Machine Family (e2, n1, n2)** — A category of Compute Engine machine types optimized for a workload profile; e2 is cost-optimized, n1/n2 are general purpose balanced, and the family determines available features like GPU support.

**Machine Type** — The hardware configuration of a Compute Engine VM, specifying the number of vCPUs and amount of memory (e.g., `e2-standard-4`, `n2-standard-8`).

**Metadata** — Key-value pairs attached to a Compute Engine VM instance; used to pass startup scripts, configuration data, and custom information accessible from within the VM.

**Metadata Server** — An internal-only HTTP endpoint (`metadata.google.internal`) available from every Compute Engine VM, providing instance-specific data and short-lived service account OAuth tokens; requests require the `Metadata-Flavor: Google` header.

**MIG (Managed Instance Group)** — A group of identical Compute Engine VM instances created from an instance template, supporting autoscaling, auto-healing, and rolling updates.

**Network Tag** — A string label applied to a VM instance to make it a target for specific firewall rules or static routes.

**nginx** — A widely used open-source HTTP server and reverse proxy; used in examples as a typical workload installed by a startup script.

**OAuth Token** — A short-lived credential used by GCP clients to authenticate API calls; VMs retrieve OAuth tokens from the metadata server using their attached service account.

**OS Login** — A GCP feature that allows SSH access to VMs using Google identities (IAM accounts) instead of manually managed SSH keys; enabled via the `enable-oslogin` metadata key and controlled via `roles/compute.osLogin` and `roles/compute.osAdminLogin`.

**pd-standard** — The Compute Engine persistent disk type backed by standard hard disks, offering lower cost but lower IOPS/latency than pd-ssd.

**pd-ssd** — The Compute Engine persistent disk type backed by solid-state drives, offering lower latency and higher IOPS than standard persistent disks.

**Persistent Disk** — GCP's durable block storage for Compute Engine VMs that persists independently of the VM's lifecycle; the source for snapshots.

**Preemptible VM** — A legacy low-cost Compute Engine VM type that uses spare capacity, has a 24-hour maximum lifetime, and may be reclaimed by GCP with a 30-second shutdown signal; superseded by Spot VMs.

**Preemption** — The termination of a Spot or preemptible VM by GCP when compute capacity is needed; a 30-second shutdown signal is sent, allowing shutdown scripts to run.

**Public Image** — A Compute Engine boot disk image provided and maintained by Google or third-party vendors (e.g., Debian, Ubuntu, Windows Server), available to all GCP projects.

**RDP (Remote Desktop Protocol)** — A Microsoft protocol for remote graphical desktop access; used to connect to Windows Compute Engine VMs.

**Red Hat Enterprise Linux (RHEL)** — A commercially supported Linux distribution available as a public Compute Engine image.

**Regional MIG** — A Managed Instance Group that distributes VM instances across multiple zones within a region for higher availability.

**Resource Policy** — A GCP resource that defines scheduled operations, such as snapshot schedules, and can be attached to disks or VMs.

**RFC 1918** — An Internet standard defining private IPv4 address ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) used for internal VM IPs within a VPC.

**Role (IAM role)** — A named collection of IAM permissions; granted to principals to allow specific actions on resources (e.g., `roles/compute.osLogin`, `roles/compute.osAdminLogin`).

**Rolling Update** — A MIG feature that gradually replaces instances running an old template version with instances running a new version, with configurable `--max-surge` and `--max-unavailable` settings.

**SCP (Secure Copy Protocol)** — A protocol for securely transferring files between a local machine and a remote server; used via `gcloud compute scp` to move files to/from VMs.

**Serial Console** — A low-level text interface to a VM that is accessible even when the operating system has not fully booted; used for troubleshooting boot failures and network issues.

**Service Account** — A GCP identity assigned to a VM instance to allow it to authenticate API calls to GCP services without requiring user credentials.

**Shutdown Script** — A script executed on a Compute Engine VM when it receives a stop or preemption signal, with a 30-second execution window to perform cleanup tasks.

**Snapshot** — A point-in-time copy of a Compute Engine persistent disk, stored by Google in Cloud Storage; used for backups, disaster recovery, and disk cloning.

**Snapshot Schedule** — An automated policy (resource policy) attached to a disk that triggers periodic snapshot creation and manages retention based on configured parameters.

**Scope (access scope)** — A legacy authorization mechanism on Compute Engine VMs that restricts which GCP APIs the attached service account may call; `--scopes=cloud-platform` grants broad access deferred to IAM.

**Spot VM** — A Compute Engine VM type that uses spare GCP capacity at a lower cost but can be reclaimed by GCP with 30 seconds notice when capacity is needed; no maximum lifetime.

**SSH (Secure Shell)** — A cryptographic protocol for secure remote command-line access to Linux VMs; the primary connection method for Linux Compute Engine instances.

**Startup Script** — A script that runs automatically as root when a Compute Engine VM boots; used to install software, apply configuration, and perform initialization tasks.

**Stateful MIG** — A Managed Instance Group configured to preserve per-instance state (specific disks, metadata, and IP addresses) across instance recreation, suitable for stateful applications.

**Stateless MIG** — A Managed Instance Group where all instances are interchangeable and no per-instance state is preserved; suitable for web frontends and batch processing.

**Suspend** — A Compute Engine VM state that saves the in-memory state to disk and stops billing for CPU/memory, allowing the VM to be resumed later with its memory state intact.

**systemd** — The Linux init system and service manager used on Compute Engine Linux images; `google-startup-scripts.service` is a systemd unit that runs the VM's startup script on boot.

**TERMINATED (VM state)** — A VM lifecycle state indicating the instance has been stopped; billing for CPU/memory ceases but attached disks remain billed.

**Ubuntu** — A popular open-source Linux distribution available as a public Compute Engine image (e.g., `ubuntu-2204-lts`).

**vCPU (virtual CPU)** — A virtualized CPU core exposed to a Compute Engine VM; the number of vCPUs is part of the machine type definition (e.g., `e2-standard-4` has 4 vCPUs).

**VM (Virtual Machine)** — A software-emulated computer running on physical hardware; the primary compute resource in Compute Engine.

**VPC (Virtual Private Cloud)** — A logically isolated private network in GCP where Compute Engine VMs are provisioned; provides subnets, firewall rules, and routing.

**Windows Server** — Microsoft's server operating system available as a public Compute Engine image; requires RDP for remote access.

**Zone** — A geographically isolated deployment area within a GCP region; Compute Engine VM instances and disks are zonal resources.
