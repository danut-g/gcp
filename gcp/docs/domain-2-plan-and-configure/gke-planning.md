# GKE Planning: Cluster Modes, Node Pools, Autopilot vs Standard

## Overview

**Google Kubernetes Engine (GKE)** is Google's managed Kubernetes service. Planning involves choosing between Autopilot and Standard mode, selecting cluster topology (regional vs zonal), designing node pools, and understanding networking and security implications before deployment.

---

## Key Concepts

### GKE Autopilot vs Standard Mode

This is the most critical planning decision for GKE:

| Dimension | **Autopilot** | **Standard** |
|-----------|--------------|-------------|
| Node management | Fully managed by Google | User manages nodes |
| Billing unit | Per Pod (vCPU + memory) | Per node (VM) |
| Node pool configuration | Not configurable | Fully configurable |
| Auto-scaling | Automatic, built-in | Manual or HPA/VPA/Cluster Autoscaler config |
| OS/kernel access | None | Full (SSH to nodes) |
| Spot Pods | Supported | Supported (via spot node pools) |
| DaemonSets | Restricted (Google-managed only) | Fully configurable |
| Custom node images | Not supported | Supported |
| Privileged containers | Not allowed | Allowed (configurable) |
| SLA | 99.9% for regional | Depends on cluster topology |
| Cluster Autoscaler | Always enabled | Optional |
| Vertical Pod Autoscaler | Always enabled | Optional |
| Best for | Most production workloads | Advanced/custom configurations |

#### Autopilot Key Behaviors

- Nodes are provisioned automatically by GCP based on Pod requests — no node pool management
- You pay for Pod resource requests (not actual usage, not node capacity)
- Pods **must specify resource requests** — Autopilot enforces minimum values if unset
- Minimum pod request: 250m CPU, 512 MB memory (per container, enforced at admission)
- Node auto-provisioning is transparent — you never see or manage the underlying VMs
- Security: Autopilot enforces strict Pod Security Standards (restricted profile)

#### Standard Mode Key Behaviors

- You create and configure node pools explicitly
- Pay for nodes (VMs) whether or not Pods are scheduled on them
- Full control over node OS, machine type, disk type/size, GPU attachment
- Cluster Autoscaler is optional but highly recommended

---

### Cluster Topology: Zonal vs Regional

| Type | Control Plane | Node Pools | Availability |
|------|--------------|-----------|-------------|
| **Zonal** | Single zone | Nodes in one or multiple zones | Lower HA, single control plane |
| **Regional** | Replicated across 3 zones | Nodes spread across 3 zones | High availability, 99.95% SLA |

#### Regional Clusters

- Control plane is replicated across 3 zones in the region — no single point of failure
- Node pools span all 3 zones by default (can be restricted)
- Each node pool zone has the same number of nodes (e.g., `num-nodes=2` → 6 total nodes: 2 per zone × 3 zones)
- **Strongly recommended for production**
- Higher cost due to 3x control plane resources

#### Zonal Clusters

- Single zone for control plane and nodes (by default)
- Can add multi-zone node pools — nodes in multiple zones but control plane in one zone
- Lower cost; acceptable for dev/test
- During control plane upgrades, there may be a brief outage

---

### Node Pools

- A **node pool** is a group of nodes within a cluster that share the same configuration
- Each cluster has at least one node pool (default pool)
- Node pool attributes: machine type, disk size/type, OS image, labels, taints, GPU count, min/max nodes (for autoscaling)

#### Node Pool Use Cases

- **Multiple pools** for different workload types: e.g., standard nodes for web tier, memory-optimized for database pods, GPU nodes for ML
- **Spot node pools**: Enable spot VMs for cost-sensitive batch workloads; mix with standard pools
- **Windows node pools**: Required for Windows containers (Linux is default)
- **ARM node pools**: T2A nodes for cost-effective ARM workloads

#### Node Taints and Labels

- **Labels**: Key-value pairs for node selection via `nodeSelector` or `nodeAffinity`
- **Taints**: Repel pods unless they have a matching `toleration` — used to reserve nodes for specific workloads
- Common pattern: GPU node pool with `nvidia.com/gpu=present:NoSchedule` taint — only pods requesting GPUs are scheduled there

---

### Cluster Autoscaler

- Automatically adds nodes when pods can't be scheduled (scale up) and removes underutilized nodes (scale down)
- Configured with `min-nodes` and `max-nodes` per node pool
- Scale-down happens when node utilization is below threshold for 10+ minutes
- **Does NOT work with:** Pods with local storage (`emptyDir` with memory medium, `hostPath`), PodDisruptionBudgets that prevent eviction
- In Autopilot mode, this is always on and transparent

---

### Networking Modes

| Mode | Description | When to Use |
|------|-------------|-------------|
| **VPC-native (Alias IP)** | Pods get IP addresses from VPC subnet | Default and recommended; enables direct Pod-to-Pod communication |
| **Routes-based** | Legacy routing mode | Legacy; not recommended for new clusters |

- VPC-native clusters: Requires a secondary IP range for Pods and another for Services
- Pod CIDR default: `/14` (can accommodate many Pods per cluster)
- Services CIDR: Must not overlap with Pod CIDR or primary subnet CIDR

#### Private Clusters

- Node VMs have no external IP addresses
- Control plane can be private or have restricted authorized networks
- Pods communicate via internal IPs only
- Requires Cloud NAT for outbound internet access from nodes
- **Private Google Access** allows nodes to reach Google APIs without external IPs

#### Workload Identity

- Best practice for GKE Pod authentication to Google Cloud services
- Maps a Kubernetes Service Account to a GCP Service Account
- No service account key files needed
- See [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md) for full details

---

### GKE Versions and Release Channels

| Channel | Update Frequency | Stability | Use Case |
|---------|-----------------|-----------|---------|
| **Rapid** | Fastest, latest features | Less stable | Testing new K8s features |
| **Regular** | Balanced | Stable | Most production workloads |
| **Stable** | Slowest | Most stable | Stability-critical production |
| **None** | Manual control | Manual | Full version control (not recommended) |

- Clusters enrolled in a channel receive automatic minor version upgrades
- Control plane upgrades first, then node pools (with configurable surge/max-unavailable)
- GKE clusters outside the support window must be manually upgraded

---

### Node Auto-Provisioning (NAP)

- Available in Standard mode; automatically creates and deletes node pools based on workload needs
- More flexible than static node pools — creates optimally-sized pools for specific Pod requests
- Useful when workloads have diverse resource requirements

---

## When to Use

| Scenario | Recommendation |
|----------|---------------|
| Most production workloads | GKE Autopilot — less ops overhead |
| Specialized hardware (GPUs, ARM) | GKE Standard with appropriate node pools |
| Need SSH access to nodes | GKE Standard only |
| Dev/test clusters | Zonal GKE Standard (cost savings) |
| Production HA requirement | Regional GKE (Autopilot or Standard) |
| Mixed workload types (CPU vs GPU vs batch) | Standard with multiple node pools |
| Cost optimization for batch | Spot node pools in Standard |

---

## When NOT to Use

- Do **not** use Autopilot if you need DaemonSets for custom monitoring agents or privileged containers
- Do **not** use zonal clusters for production workloads with uptime SLAs
- Do **not** use routes-based networking for new clusters — VPC-native is the modern standard
- Do **not** use GKE for simple, stateless HTTP services that don't need Kubernetes complexity — consider Cloud Run

---

## Related Services / Concepts

- **GKE Deployment**: Cluster creation, Deployments, Services, Ingress — see [gke-deploy.md](../domain-3-deploy-and-implement/gke-deploy.md)
- **Managing GKE**: Upgrades, workload identity — see [managing-gke.md](../domain-4-ensure-success/managing-gke.md)
- **Cloud Run vs GKE**: Decision framework — see [cloud-run-functions-planning.md](cloud-run-functions-planning.md)
- **Network Planning**: VPC, subnets for GKE — see [network-planning.md](network-planning.md)
- **Service Accounts**: Workload Identity for GKE — see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Regional cluster node count**: `num-nodes=3` on a regional cluster creates 3 nodes PER ZONE = 9 nodes total (3 zones × 3 nodes). This is a classic exam trap.

2. **Autopilot billing**: You pay for Pod resource requests, not node capacity. A pod requesting 1 vCPU costs the same regardless of whether the underlying node has 4 or 16 vCPUs.

3. **Autopilot enforces Pod resource requests**: If your Pod spec doesn't set resource requests, Autopilot sets defaults. This affects cost visibility.

4. **Workload Identity requires two bindings**: You need both the GCP IAM binding (Kubernetes SA → GCP SA with `roles/iam.workloadIdentityUser`) AND the Kubernetes annotation on the SA. Missing either breaks it.

5. **Private clusters still need Cloud NAT**: Private cluster nodes have no external IPs. For outbound internet access (pulling images from Docker Hub, etc.), Cloud NAT is required.

6. **VPC-native requires secondary IP ranges**: Before creating a VPC-native cluster, ensure the subnet has two secondary ranges: one for Pods, one for Services. Pre-plan your IP space.

7. **Cluster Autoscaler vs HPA**: HPA scales **Pods** based on metrics. Cluster Autoscaler scales **Nodes** based on Pod scheduling needs. They work together.

8. **Spot pods in Autopilot**: Available via `cloud.google.com/gke-spot: "true"` node selector. Not the same as Spot node pools in Standard mode.

### Decision Tree: Autopilot vs Standard

```
Do you need SSH access to nodes? → Standard
Do you need custom DaemonSets? → Standard
Do you need privileged containers? → Standard
Do you need specific node OS customization? → Standard
Is this dev/test with cost priority? → Standard zonal (cheap nodes)
Otherwise → Autopilot (less overhead, per-pod billing)
```

### Keywords
- Autopilot, Standard mode, regional cluster, zonal cluster, node pool, Cluster Autoscaler, VPC-native, private cluster, Workload Identity, release channel, spot node pool, node taint, node label, NAP

---

## Source

- https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview
- https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-autoscaler
- https://cloud.google.com/kubernetes-engine/docs/concepts/private-cluster-concept
- https://cloud.google.com/kubernetes-engine/docs/concepts/release-channels
