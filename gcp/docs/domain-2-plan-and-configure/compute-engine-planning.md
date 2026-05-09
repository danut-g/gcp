# Compute Engine Planning: Machine Types, Preemptible, Committed Use

## Overview

**Compute Engine** provides virtual machines (VMs) running on Google's infrastructure. Planning involves selecting the right machine type, understanding pricing models, and choosing between different VM lifecycle options. Decisions made here have significant cost and performance implications.

---

## Key Concepts

### Machine Type Families

#### General Purpose

| Series | Use Case | Architecture |
|--------|----------|-------------|
| **N2** | Balanced compute/memory, most workloads | Intel Ice Lake |
| **N2D** | AMD EPYC, better price/performance than N2 | AMD EPYC |
| **N4** | Latest gen general purpose | Intel Emerald Rapids |
| **E2** | Cost-optimized, moderate workloads, dev/test | Various (Google selected) |
| **T2D** | Scale-out workloads, web servers, cost-effective | AMD EPYC Milan |
| **T2A** | ARM-based workloads | Ampere Altra ARM |

- **E2** machines are ~31% cheaper than N1 but have shared vCPUs — good for dev/test and bursty workloads
- **N2** is the recommended default for production general-purpose workloads

#### Compute Optimized

| Series | Use Case |
|--------|----------|
| **C2** | High-performance computing, game servers, ad serving |
| **C2D** | AMD EPYC, high-memory bandwidth, HPC |
| **C3** | Latest gen compute optimized |
| **H3** | HPC with high-bandwidth networking |

#### Memory Optimized

| Series | vCPU Range | Memory per vCPU | Use Case |
|--------|-----------|-----------------|---------|
| **M1** | 40-160 vCPU | Up to 24 GB/vCPU | SAP HANA, in-memory databases |
| **M2** | 208-416 vCPU | Up to 14.9 TB total | Largest in-memory workloads |
| **M3** | Up to 128 vCPU | ~31 GB/vCPU | SAP HANA |

#### Accelerator Optimized

| Series | GPU | Use Case |
|--------|-----|---------|
| **A2** | NVIDIA A100 | AI/ML training, HPC |
| **A3** | NVIDIA H100 | Large-scale AI/ML |
| **G2** | NVIDIA L4 | AI inference, video transcoding |

---

### Custom Machine Types

- Create VMs with **custom vCPU and memory** combinations not available in predefined types
- Available on N1, N2, N2D, E2 series
- Memory constraints:
  - Must be between **0.9 GB and 6.5 GB per vCPU** (N2)
  - Memory must be a multiple of **256 MB**
- **Extended memory**: Add memory beyond the 6.5 GB/vCPU limit for an additional cost (up to ~8 GB/vCPU for N1, varies by series)
- Custom types cost slightly more than equivalent predefined types but allow precise resource matching

---

### vCPU and Memory Pricing Model

- Priced per **vCPU/hour** + **GB memory/hour** separately
- Predefined types are simply bundles of vCPUs + memory at list prices
- **Fractional vCPU**: E2 shared-core types (e2-micro, e2-small, e2-medium) use fractional vCPUs billed as partial core hours

---

### Preemptible VMs

- Short-lived VMs at up to **~90% discount** versus standard pricing
- Can be preempted (terminated) by Google with a **30-second warning** when Google needs the capacity
- Maximum runtime: **24 hours** — will be terminated after 24 hours even if not preempted
- Not suitable for:
  - Long-running, stateful workloads
  - Latency-sensitive applications
  - Databases or anything requiring high availability
- Suitable for:
  - Batch processing, HPC, rendering, ML training (with checkpointing)
  - Dev/test environments
  - Any fault-tolerant, restartable workload
- **Preemption rate** varies by machine type and region; typically 5–15% daily preemption probability but can be higher

### Spot VMs (successor to Preemptible)

- **Spot VMs** replace Preemptible VMs as the current recommended option
- Key differences from Preemptible:
  - **No maximum 24-hour runtime limit**
  - Same pricing model (~90% discount)
  - Same preemption behavior (30-second warning, can be reclaimed anytime)
- For new workloads, prefer Spot VMs over Preemptible

---

### Committed Use Discounts (CUDs)

Two types of commitments:

| Type | Discount | How It Works |
|------|----------|-------------|
| **Resource-based** | Up to 57% (1-year), up to 70% (3-year) | Commit to specific vCPU/memory amounts in a region |
| **Spend-based** | Up to 25–28% | Commit to a minimum hourly spend amount |

#### Resource-Based CUDs

- Commit to a **specific amount of vCPU and memory** in a specific **region** for 1 or 3 years
- Applied automatically to matching usage — no need to specify which VMs get the discount
- Commitment is on the resource type (e.g., N2 vCPUs in us-central1), not individual VMs
- Unused commitment is still billed (you pay for what you committed to, whether used or not)
- Can be combined: use CUDs for baseline, Spot/Preemptible for burst

#### Spend-Based CUDs

- Available for: Cloud SQL, Cloud Spanner, Anthos, VMware Engine
- Commit to a minimum hourly spend
- Applied as discounts against actual usage

---

### Sustained Use Discounts (SUDs)

- **Automatic** discounts applied when a VM runs for a significant fraction of the month
- No action required — GCP calculates automatically
- Applies to: N1, N2, N2D, C2, M1, M2 (general purpose and compute/memory optimized)
- **Does NOT apply to**: E2, T2D, preemptible/spot VMs, Sole-tenant nodes
- Discount structure (for N1/N2):
  - 0–25% of month: full price
  - 25–50% of month: ~20% off
  - 50–75% of month: ~40% off
  - 75–100% of month: ~60% off
  - Effective discount for a VM running all month: ~30%
- SUDs and CUDs are mutually exclusive for the same usage — GCP applies the better discount

---

### Sole-Tenant Nodes

- Physical servers dedicated exclusively to your VMs
- Use cases:
  - Compliance requirements (hardware isolation)
  - Bring Your Own License (BYOL) for Windows Server/SQL Server
  - Workloads requiring physical isolation from other customers
- Higher cost than standard VMs
- VMs on sole-tenant nodes **do not receive SUDs**

---

### VM Metadata and Startup Scripts

- Every VM has access to the **metadata server** at `169.254.169.254` (link-local)
- Provides: project metadata, instance metadata, service account tokens, startup scripts
- Startup scripts set in instance metadata execute at VM startup
- See [compute-engine-deploy.md](../domain-3-deploy-and-implement/compute-engine-deploy.md) for deployment details

---

## When to Use

| Scenario | Recommended |
|----------|-------------|
| Standard web/app servers | N2 or E2 |
| Memory-heavy databases | M1/M2 |
| ML training with GPU | A2/A3 |
| Batch jobs, fault-tolerant | Spot/Preemptible + checkpointing |
| Predictable, always-on baseline | Committed Use Discounts |
| Compliance/BYOL isolation | Sole-tenant nodes |
| Precise resource fit | Custom machine types |

---

## When NOT to Use

- **Preemptible/Spot**: Not for stateful databases, critical services, or workloads without checkpointing/retry logic
- **Custom types**: Avoid unless predefined types genuinely don't fit — they're slightly more expensive and harder to manage at scale
- **Committed use**: Don't commit to resources you're uncertain about — unused commitments are billed
- **Sole-tenant**: Not for cost-sensitive workloads without compliance requirements

---

## Related Services / Concepts

- **Compute Engine Deployment**: Instance creation, templates, MIGs — see [compute-engine-deploy.md](../domain-3-deploy-and-implement/compute-engine-deploy.md)
- **Managing Compute**: Autoscaling, rolling updates — see [managing-compute.md](../domain-4-ensure-success/managing-compute.md)
- **Pricing Optimization**: Full cost comparison framework — see [pricing-optimization.md](pricing-optimization.md)
- **GKE Planning**: If containerized workloads are more appropriate — see [gke-planning.md](gke-planning.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Preemptible vs Spot**: Both offer ~90% discount. Key difference: Preemptible has 24-hour max runtime; Spot does not. For new workloads, Spot is preferred.

2. **SUDs don't apply to E2 or Preemptible/Spot**: A common distractor. E2 machines are cheap but don't get SUDs. Preemptible/Spot don't get SUDs either — they already have the discount.

3. **CUDs apply to a region's resource pool, not specific VMs**: You commit to N2 vCPUs in us-central1, and any N2 VMs in that region get the discount — even if you delete and recreate them.

4. **CUD + SUD interaction**: They don't stack. GCP applies the better of the two for any given usage.

5. **E2 shared-core for dev/test**: e2-micro, e2-small, e2-medium use fractional CPUs. Great for low-traffic or dev use. Not for production workloads needing consistent performance.

6. **Memory per vCPU limits for custom types**: 0.9 GB minimum, 6.5 GB standard maximum, ~8 GB with extended memory. These limits are tested.

7. **Sole-tenant no SUD**: Cannot combine sole-tenant with sustained use discounts.

### Decision Tree: VM Type Selection

```
Is the workload fault-tolerant/restartable?
├── Yes → Is it cost-critical batch/HPC?
│          └── Yes → Spot VM (best price)
│
Is the workload containerized?
├── Yes → Consider GKE or Cloud Run instead
│
What's the memory requirement per vCPU?
├── <6.5 GB/vCPU → General purpose (N2, E2)
├── >6.5 GB/vCPU → Memory optimized (M1, M2)
│
Is there a GPU/ML requirement?
├── Yes → A2 (A100) or G2 (L4)
│
Is it running >50% of the month (always-on)?
├── Yes → Consider Committed Use Discount
```

### Keywords
- Machine type, N2, E2, M1, M2, A2, custom machine type, preemptible VM, Spot VM, sustained use discount, committed use discount, sole-tenant node, resource-based CUD, spend-based CUD, vCPU, extended memory

---

## Source

- https://cloud.google.com/compute/docs/machine-resource
- https://cloud.google.com/compute/docs/instances/spot
- https://cloud.google.com/compute/docs/sustained-use-discounts
- https://cloud.google.com/compute/docs/instances/signing-thirdparty-software
