# Compute Engine Deployment: Instances, Templates, MIGs, Startup Scripts

## Overview

Deploying Compute Engine resources involves creating VM instances, standardizing them via instance templates, managing groups of VMs with Managed Instance Groups (MIGs), and using startup scripts for configuration. These concepts underpin scalable, resilient compute deployments on GCP.

---

## Key Concepts

### VM Instance Creation

Key configuration decisions when creating a VM:

| Parameter | Options / Notes |
|-----------|----------------|
| Machine type | Standard predefined or custom |
| Boot disk | Image (OS), size, type (pd-standard, pd-balanced, pd-ssd, pd-extreme) |
| Additional disks | Multiple persistent disks, local SSD |
| Region/Zone | Where the VM runs |
| Network/Subnet | VPC and subnet |
| External IP | Ephemeral, static, or none |
| Service account | Which SA identity the VM uses |
| Access scopes | What APIs the SA can call (legacy; prefer full cloud-platform scope + IAM) |
| Startup script | Script run on first boot (or every boot) |
| Metadata | Key-value pairs accessible from VM |
| Tags | Network tags for firewall rule targeting |
| Labels | Cost attribution, organization |
| Shielded VM | Secure Boot, vTPM, Integrity Monitoring |
| Confidential VM | Memory encryption (AMD SEV) |

#### Disk Types

| Type | IOPS | Throughput | Use Case |
|------|------|-----------|---------|
| `pd-standard` | Low | Low | Dev/test, batch, low-priority |
| `pd-balanced` | Medium | Medium | General purpose (recommended default) |
| `pd-ssd` | High | High | Production databases, high-frequency I/O |
| `pd-extreme` | Highest | Highest | Mission-critical databases, highest I/O |
| `local-ssd` | Very high | Very high | Ephemeral scratch disk; data lost on VM stop |

- **Persistent disks**: Survive VM deletion (if not auto-deleted); can be attached to multiple VMs read-only
- **Local SSD**: Physically attached to the host; much lower latency than persistent disk; **data is lost when VM stops** (not just restarts)
- Boot disk default auto-delete: ON. Best practice: Keep auto-delete OFF for production to preserve data during accidental VM deletion

#### Static vs Ephemeral External IPs

- **Ephemeral**: Assigned at VM start; released when VM stops; may change
- **Static (reserved)**: Fixed IP; billed when not attached to a running VM (~$0.01/hr)
- VMs don't need external IPs if they communicate only with internal resources or via Cloud NAT

---

### Instance Templates

- **Immutable** blueprint for VM configuration: machine type, disk image, network, metadata, service account, startup scripts, labels, tags
- Templates are **global** resources — can be used across any zone/region
- Once created, a template cannot be modified — create a new version
- Used by: Managed Instance Groups (MIGs), individual VM creation
- Best practice: Use instance templates for any VM that might be replicated or managed at scale

#### Instance Templates and Updates

- When updating a MIG to use a new template, existing VMs are not automatically updated
- Update requires a **rolling update** or **canary update** action on the MIG
- This is intentional — prevents accidental replacement of running instances

---

### Managed Instance Groups (MIGs)

A MIG is a collection of identical VMs managed as a unit using an instance template.

#### MIG Types

| Type | Description | Use Case |
|------|-------------|---------|
| **Stateless MIG** | Instances are interchangeable; can be replaced freely | Web servers, API endpoints, batch workers |
| **Stateful MIG** | Preserves per-instance state (disk, metadata, IP) across restarts/updates | Databases, stateful apps |

#### Key MIG Features

- **Autoscaling**: Scale in/out based on metrics (CPU utilization, HTTP load balancing utilization, Pub/Sub queue depth, custom metrics)
- **Autohealing**: Replaces unhealthy instances automatically using a **health check**
- **Rolling updates**: Update instances gradually (configurable max-surge, max-unavailable)
- **Multi-zone distribution**: Regional MIGs distribute VMs across multiple zones for HA
- **Zonal MIG**: All VMs in a single zone (simpler, less resilient)

#### Autoscaling Policies

| Signal | Description |
|--------|-------------|
| CPU utilization | Scale based on average CPU across the group |
| HTTP load balancing utilization | Scale based on backend load balancer metrics |
| Pub/Sub subscription backlog | Scale workers based on queue depth |
| Cloud Monitoring custom metric | Scale based on any custom metric |
| Schedule-based | Scale up/down at specific times (predictable patterns) |

- **Cooldown period**: Time after a new VM starts before autoscaler considers it for scaling decisions (default 60s; set to match your app startup time)
- **Stabilization window**: How long the autoscaler waits before removing VMs after scale-in recommendation (prevents oscillation)
- **Min/Max instances**: Define the scaling bounds; autoscaler will not go below min or above max

#### Autohealing

- Requires configuring a **health check** on the MIG
- Health check: HTTP/HTTPS/TCP probe to verify instance is serving traffic
- If an instance fails the health check for a configurable number of consecutive failures, MIG replaces it
- Autohealing is independent of the load balancer health check — configure it separately on the MIG

#### Rolling Updates

- `gcloud compute instance-groups managed rolling-action start-update` (or Console)
- **Maximum surge**: Additional VMs created above the target size during the update
- **Maximum unavailable**: VMs that can be unavailable at any time during update
- Both set as absolute numbers or percentages
- **Canary rollout**: Update a subset of instances to new template, verify, then continue

---

### Startup Scripts

- Shell scripts executed on VM start (every boot) or first boot (depends on configuration)
- Set via instance metadata key `startup-script` (inline) or `startup-script-url` (GCS URL)
- Run as root
- Shutdown scripts: Run before VM shuts down (key `shutdown-script`); limited to 90 seconds to complete
- **Startup script on MIG**: Set in instance template metadata — all new instances execute the script
- Use cases: Install software, configure the OS, join a domain, pull secrets, register with a service registry

#### Cloud-Init

- Alternative to startup scripts for more portable instance initialization
- Uses YAML configuration format (`cloud-config`)
- Supported on GCP through the `cloud-init` package in most Linux images

---

### VM Snapshots

- Point-in-time backup of a persistent disk
- Incremental: First snapshot is full; subsequent snapshots only store changed blocks
- Cross-region: Snapshots can be stored in any region or multi-region
- Use to: Back up data, create base images for new instances, move disks between zones/regions
- **Custom images**: Created from a running or stopped VM's boot disk; used as the source for new VMs
- Snapshot vs Image: Snapshots are disk backups; images are bootable VM templates

---

### Live Migration

- GCP automatically migrates VMs to other hardware during host maintenance
- No VM downtime during live migration (transparent to the VM)
- Some VM types don't support live migration: GPU VMs, preemptible VMs (they get preempted instead)
- `terminateOnHostMaintenance` option: VM terminates instead of migrating (required for some specialized VMs)

---

## When to Use

- **Instance templates**: Always, for any VM that might be replicated, scaled, or needs consistent configuration
- **Regional MIGs**: For production stateless workloads needing HA across zones
- **Autohealing**: For production MIGs — ensures instances are replaced if they become unhealthy
- **Startup scripts**: For bootstrapping VMs with software installation or configuration
- **Snapshots**: For regular backups of persistent disks; before major changes
- **Stateful MIGs**: When VMs need per-instance identity (e.g., stateful clustered databases)

---

## When NOT to Use

- **Zonal MIGs for production**: Single zone = single point of failure
- **Local SSD for persistent data**: Data is lost on VM stop. Only use for ephemeral scratch storage (caching, temp files)
- **Auto-delete boot disk enabled in production**: Risk of accidental data loss on VM deletion
- **Default service account on VMs**: Use dedicated service accounts with minimal permissions

---

## Related Services / Concepts

- **Compute Engine Planning**: Machine type selection — see [compute-engine-planning.md](../domain-2-plan-and-configure/compute-engine-planning.md)
- **Managing Compute**: Autoscaling configuration, rolling updates — see [managing-compute.md](../domain-4-ensure-success/managing-compute.md)
- **Load Balancing**: MIGs as backends for load balancers — see [networking-deploy.md](networking-deploy.md)
- **IAM**: Service accounts for VMs — see [iam-overview.md](../domain-1-setup-and-configure/iam-overview.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Instance template is immutable**: Cannot edit a template after creation. Must create a new template and update the MIG to reference it.

2. **Local SSD data loss on stop**: A classic trap. Local SSD data survives a **restart** but is permanently lost when the VM is **stopped** (deleted from host). Important distinction.

3. **Autohealing ≠ Load balancer health check**: They're separate configurations. A load balancer may route traffic away from an unhealthy instance, but only MIG autohealing will **replace** it.

4. **Regional MIG node distribution**: If you create a regional MIG with 3 zones and specify 6 instances, GCP puts 2 in each zone. If one zone is unavailable, the remaining 4 instances serve traffic.

5. **Rolling update max-surge and max-unavailable**: Setting max-surge=0 and max-unavailable=1 does an in-place rolling update (one at a time, no extra capacity). Setting max-surge=1 and max-unavailable=0 ensures full capacity during update (adds one new, then removes one old).

6. **Startup script runs every boot**: By default, startup scripts run on every boot, not just the first. Use instance metadata or a flag file to skip re-initialization.

7. **Snapshots are incremental after first**: Don't assume each snapshot is full — they're incremental. But deletion of a snapshot doesn't reclaim the space if subsequent snapshots reference the same blocks.

8. **GPU VMs don't live migrate**: They terminate and restart on host maintenance (or preempt if preemptible). Plan for this in GPU workloads.

### Keywords
- Instance template, MIG, managed instance group, autoscaling, autohealing, rolling update, startup script, startup-script-url, local SSD, persistent disk, snapshot, custom image, health check, max-surge, max-unavailable, stateful MIG

---

## Source

- https://cloud.google.com/compute/docs/instances/create-start-instance
- https://cloud.google.com/compute/docs/instance-templates
- https://cloud.google.com/compute/docs/instance-groups/creating-groups-of-managed-instances
- https://cloud.google.com/compute/docs/disks/snapshots
