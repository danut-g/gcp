# Managing Compute Engine: Autoscaling, Rolling Updates, Snapshots

## Overview

Managing Compute Engine resources involves configuring autoscaling policies, performing rolling updates to MIGs, managing disk snapshots and backups, and maintaining instance health. These are ongoing operational responsibilities distinct from initial deployment.

---

## Key Concepts

### Autoscaling Deep Dive

Autoscaling is configured on a Managed Instance Group (MIG) and adjusts the number of instances based on demand.

#### Autoscaling Policies

| Policy | Signal | Best For |
|--------|--------|---------|
| **CPU utilization** | Average CPU % across the MIG | CPU-bound workloads |
| **HTTP load balancing utilization** | Backend utilization from LB | Web applications behind LB |
| **Pub/Sub subscription backlog** | Number of undelivered messages | Queue-based workers |
| **Cloud Monitoring custom metric** | Any metric you can query | Application-specific scaling |
| **Schedule-based** | Time-based scale up/down | Predictable load patterns |

#### Autoscaling Configuration Parameters

- **Target utilization**: The desired steady-state utilization; autoscaler tries to maintain this
  - Example: CPU target 60% → autoscaler adds VMs if average CPU exceeds 60%, removes if below 60%
- **Cool-down period**: Time after a VM starts before autoscaler uses its metrics (default: 60 seconds; match to your app startup time)
  - If too short: new VMs not yet ready will appear to have 0% CPU → triggers scale-down before stabilization
- **Stabilization period** (scale-in window): Prevents hasty scale-down; autoscaler waits this long after the last scale-up recommendation before scaling in
- **Min replicas**: Minimum number of instances regardless of load; set to > 0 to avoid complete scale-to-zero
- **Max replicas**: Hard cap on scaling

#### Multi-Metric Autoscaling

- Can configure multiple autoscaling signals simultaneously
- Autoscaler takes the policy requiring the **most instances** at any given time
- Example: CPU target 60% + Pub/Sub backlog target 100 messages → autoscaler scales to satisfy whichever requires more VMs

#### Schedule-Based Scaling

- Pre-warm instances before a known traffic spike (sales events, batch jobs)
- Define minimum instances for specific time windows
- Layered with reactive autoscaling: schedule sets a floor; reactive can still scale above it

---

### Rolling Updates and Canary Deployments

Managed Instance Groups support two update types.

#### Rolling Update Types

| Type | Behavior |
|------|---------|
| **Proactive** | Immediately replaces instances according to the update policy |
| **Opportunistic** | Only replaces instances when they're stopped/restarted/recreated; no forced replacements |

- Use **proactive** for production updates where you want consistent rollout
- Use **opportunistic** for lower-priority updates that can happen gradually as instances cycle

#### Rolling Update Parameters

- **maxSurge**: How many extra instances to create above target size during update (0 = in-place, N = N extra first)
- **maxUnavailable**: How many instances can be unavailable at once (0 = zero-downtime rolling, N = N instances replaced simultaneously)
- Common production pattern: `maxSurge=1, maxUnavailable=0` → Full capacity maintained throughout, adds one new VM at a time

#### Canary Updates

- Update a **subset** of instances to a new template version first
- Monitor for errors/performance issues
- If healthy, complete the rollout; if not, stop and roll back
- In GCP Console: Configure "Canary rollout" section in the MIG rolling update

#### Rollback

- Rollback = start another rolling update with the **previous instance template**
- No built-in "undo" button — just specify the old template and run a new rolling update
- This is why maintaining multiple versions of instance templates is important

---

### Instance Repair and Autohealing

#### Autohealing

- Configured with a **health check** on the MIG
- GCP polls instances via the health check
- If health check fails for `initialDelay` + several consecutive failures → instance is recreated
- `initialDelay`: Grace period after instance starts before health checks begin (prevents premature replacement during boot)
- Setting `initialDelay` too short → app still starting up → health check fails → instance replaced in a loop

#### Recreating vs Restarting

- If you modify an instance template and then use `recreate-instances`, instances are **deleted and new ones are created** — any local disk data is lost
- `restart-instances`: Performs a stop + start (preserves persistent disk data, but local SSD data is lost)

---

### Disk Snapshots

#### Snapshot Behavior

- Incremental after the first (full) snapshot
- Stored in Cloud Storage (regional or multi-regional)
- Can be used to create new persistent disks (in any zone)
- Cross-region: Specify `--storage-location` when creating a snapshot to store in a different region/multi-region

#### Snapshot Scheduling

- Create a **snapshot schedule** resource and attach it to disks
- Configure: frequency (hourly, daily, weekly), retention count, start time
- Attached disks will be automatically snapshotted according to the schedule
- Requires the default Google service account to have `compute.snapshots.create` permission (automatic in most cases)

#### Snapshot Deletion and Chaining

- Deleting a snapshot doesn't delete data referenced by subsequent snapshots (incremental chain)
- GCP handles chain dependencies automatically; space isn't freed until all dependents are deleted
- Avoid deleting snapshots mid-chain unnecessarily — understand the implications

#### Custom Images vs Snapshots

| Aspect | Snapshot | Custom Image |
|--------|----------|-------------|
| Purpose | Disk backup, cross-zone disk move | Base image for new VM boot disks |
| Boot from it? | Must create a disk first | Yes, directly |
| Storage | Cloud Storage | Image storage (separate from GCS) |
| Encryption | Inherits disk encryption | Can set CMEK |

---

### Sole-Tenant Node Management

- **Node templates**: Define the server type, GPU, and affinity labels
- **Node groups**: Autoscaling groups of sole-tenant nodes
- VMs affined to a node group: Use `node affinity` labels in the instance config

---

### VM Lifecycle Management

#### Instance Schedules

- Define schedules to start/stop VMs automatically (e.g., dev VMs only during business hours)
- Cost savings: Stop VMs when not in use; billing stops when VM is stopped (except persistent disk storage)
- Create as a `Resource Policy` and attach to instances

#### OS Login

- Manages SSH access via IAM instead of per-VM SSH keys
- `constraints/compute.requireOsLogin` org policy enforces OS Login across all VMs
- Users with `roles/compute.osLogin` can SSH to VMs without needing SSH keys in metadata
- `roles/compute.osAdminLogin` for sudo access

---

## When to Use

- **Autoscaling with CPU target**: Standard for web/application servers
- **Pub/Sub autoscaling**: For worker pools processing messages from a queue
- **Schedule-based scaling + reactive**: For predictable traffic with occasional spikes
- **Proactive rolling updates**: Standard for deploying application updates to production
- **Opportunistic updates**: For non-urgent infrastructure updates (OS patches, etc.)
- **Snapshot schedules**: For all production persistent disks with data
- **Instance schedules**: For dev/test VMs that don't need to run 24/7

---

## When NOT to Use

- **Very short cool-down periods**: Apps with slow startup need a cool-down period matching their startup time; too-short periods cause oscillation
- **`maxUnavailable=0` and `maxSurge=0` simultaneously**: This is an invalid combination that results in no updates being applied
- **Recreate-instances without data backup**: Lost local SSD data; consider snapshotting persistent disks first

---

## Related Services / Concepts

- **Compute Engine Deploy**: MIG creation, templates — see [compute-engine-deploy.md](../domain-3-deploy-and-implement/compute-engine-deploy.md)
- **Compute Engine Planning**: Machine type selection — see [compute-engine-planning.md](../domain-2-plan-and-configure/compute-engine-planning.md)
- **Cloud Monitoring**: Autoscaling signals, alerting on VM metrics — see [monitoring-cloud-ops.md](monitoring-cloud-ops.md)
- **Networking Deploy**: Load balancers backing MIGs — see [networking-deploy.md](../domain-3-deploy-and-implement/networking-deploy.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Cool-down period must match app startup time**: If your app takes 2 minutes to start, set cool-down to at least 120 seconds. Too-short cool-down causes premature scale-in or oscillation.

2. **Rolling update rollback**: No "undo" button — rollback requires specifying the previous template and running a new rolling update.

3. **maxSurge=0 and maxUnavailable=0**: Invalid combination. You must allow either some extra capacity (maxSurge > 0) or some unavailability (maxUnavailable > 0). Otherwise no instances can be updated.

4. **Snapshot deletion and chains**: Deleting an intermediate snapshot doesn't free storage until all dependent snapshots are deleted. GCP manages this automatically.

5. **autohealing initialDelay**: Set initialDelay long enough for your application to start up. If too short, instances get replaced in a boot loop.

6. **Instance schedule vs autoscaling**: Instance schedules are for start/stop times (dev VMs). Autoscaling adjusts count based on load (production). They're different features for different purposes.

7. **Custom images for golden images**: If you've configured an OS and installed software, create a custom image (not a snapshot) for use as a base image in instance templates.

8. **Stateful MIGs and rolling updates**: Stateful MIGs (preserving per-instance state) have different update behavior — rolling updates may need special handling for state preservation.

### Keywords
- Autoscaling policy, cool-down period, stabilization period, min/max replicas, maxSurge, maxUnavailable, rolling update, opportunistic update, proactive update, canary rollout, snapshot schedule, custom image, autohealing, initialDelay, instance schedule, OS Login

---

## Source

- https://cloud.google.com/compute/docs/autoscaler
- https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups
- https://cloud.google.com/compute/docs/disks/scheduled-snapshots
- https://cloud.google.com/compute/docs/os-login
