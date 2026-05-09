# Compute Engine Planning: Dual-Layer Explanation

---

# Machine Type Families — Dual-Layer Explanation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Compute Engine machine types are like car model families: economy (E2), standard sedan (N2), sports car (C2), stretch limousine (M1/M2), and specialized racing car with a rocket engine (A2 with GPU). Each family is optimized for a different combination of speed, space, and cost. You pick the family that matches the job, not just "the most powerful one."

### B. TECHNICAL EXPLANATION
Compute Engine machine types are predefined combinations of vCPU count and memory size organized into families optimized for different workload characteristics. The family choice affects CPU architecture, memory-to-CPU ratio, networking performance, and price. General-purpose families (N2, E2, N2D) handle most workloads; compute-optimized families (C2, C3) provide maximum CPU performance; memory-optimized families (M1, M2, M3) deliver the highest memory-to-CPU ratios; accelerator-optimized families (A2, A3, G2) attach NVIDIA GPUs for ML and media workloads.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
When you order a car model (machine type), Google goes to its physical data center and allocates a specific number of CPU cores and memory chips from a physical server. The "family" determines which type of server hardware those cores come from. N2 gets Intel Ice Lake processors; N2D gets AMD EPYC; T2A gets ARM Altra chips.

### B. TECHNICAL EXPLANATION
Each machine type family maps to specific hardware:
- **N2**: Intel Ice Lake CPUs, 2 GB/vCPU memory ratio (standard), up to 128 vCPUs.
- **N2D**: AMD EPYC Milan, same memory ratios as N2, better price/performance for many workloads.
- **E2**: Google selects hardware dynamically; shared-core types (e2-micro, e2-small, e2-medium) use fractional physical CPU cores shared with other VMs.
- **C2/C2D/C3**: High sustained clock speeds for CPU-bound workloads; lower memory per vCPU.
- **M1/M2/M3**: Very high memory per vCPU (up to 24 GB/vCPU for M1, 14.9 TB total for M2); designed for in-memory databases.
- **A2/A3**: Paired with NVIDIA A100/H100 GPUs; GPU is attached directly to the host server.
- **G2**: NVIDIA L4 GPU; optimized for AI inference and video transcoding workloads.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of machine families as tools in a toolbox. Don't use a sledgehammer (M2) to hang a picture frame (simple web server). Don't use a screwdriver (E2) to build a house frame (large ML training job). Match the tool to the task.

### B. TECHNICAL EXPLANATION
The correct mental model for machine type selection:
- **vCPU:memory ratio is the primary selector**: Most apps need ~2–4 GB/vCPU (N2, E2). Memory-heavy apps (SAP HANA) need 8–24 GB/vCPU (M series). CPU-heavy apps with low memory need C series.
- **Cost vs performance trade-off by family**: E2 is cheapest but uses shared vCPUs (bursty CPU, not sustained). N2 provides dedicated vCPUs. For consistent performance: N2 or above.
- **GPU requirement**: If the workload uses CUDA (ML training, inference), A2/A3/G2 are the only applicable families.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Web server? N2 or E2. Database with lots of RAM? M1. GPU-based model training? A2. Batch job you don't care about performance? E2 or Spot. Dev laptop replacement? E2.

### B. TECHNICAL EXPLANATION
| Scenario | Machine Type |
|---|---|
| Standard web/app server | n2-standard-4 or e2-standard-4 |
| Dev/test environments | e2-small or e2-medium (cheapest) |
| SAP HANA or Oracle DB | m1-megamem-96 or m2-megamem-416 |
| Game server (CPU performance) | c2-standard-8 |
| ML training with GPU | a2-highgpu-1g (1× A100) |
| AI inference / video | g2-standard-8 (L4 GPU) |
| ARM workloads (cost-efficient) | t2a-standard-4 |
| High memory bandwidth HPC | h3-standard-88 |

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
E2 machines are like ridesharing: your workload shares a physical CPU with other VMs, getting a fraction of cycles. This is fine when traffic is bursty and unpredictable. N2 machines are like private taxis: you get dedicated CPU cycles consistently.

### B. TECHNICAL EXPLANATION
**E2 shared-core internals**: e2-micro (0.25 vCPU), e2-small (0.5 vCPU), e2-medium (1 vCPU) allocate fractional CPU cores. Physical CPU is time-shared with other VMs using hypervisor scheduling. Sustained CPU-intensive workloads may encounter CPU throttling. Not suitable for latency-sensitive or CPU-bound production workloads.

**N2/N2D dedicated vCPU**: Each vCPU maps to a dedicated hyperthread on a physical core. Consistent, predictable performance. CPU topology matters for NUMA-sensitive applications (e.g., database buffer pool pinned to a NUMA node).

**GPU attachment (A2/G2/A3)**: GPUs are physically installed in the host server and exposed to the VM via virtualization (PCI passthrough or similar). The GPU has its own memory (HBM2 for A100 = 40 GB or 80 GB). CUDA workloads communicate directly with GPU memory; bandwidth between GPU and host RAM is the primary performance bottleneck.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
Picking the wrong machine type is like packing for a trip: underpacking (E2 for a production database) leads to slowness and crashes; overpacking (M2 for a small website) wastes money. And some tools only exist in certain stores (GPU types only available in specific regions/zones).

### B. TECHNICAL EXPLANATION
- **E2 for production databases**: Shared-core CPUs cause inconsistent query latency. Never use E2 for databases requiring consistent I/O performance.
- **GPU availability by zone**: A2/A3/G2 instances are only available in specific zones. Check quota and availability before designing GPU-dependent architectures.
- **Memory-optimized minimum configuration**: M2 instances start at 208 vCPUs. You cannot use an M2 for small memory requirements — the minimum is very large and expensive.
- **C2 for non-CPU-bound work**: C2's premium is wasted on I/O-bound workloads. If the bottleneck is disk or network, C2 provides no benefit over N2.
- **T2A ARM compatibility**: Not all software compiles for ARM. Verify application and library ARM compatibility before choosing T2A.

---

## 7. TRADE-OFFS

### A. ANALOGY
Every step up in specialization adds cost and reduces availability. E2 is everywhere and cheap but inconsistent. M2 is powerful but expensive and not available in all zones. A2 is fastest for ML but costs $3+/hour and may have limited quota.

### B. TECHNICAL EXPLANATION
**E2**: Lowest cost, shared vCPU, no SUD eligible, no GPU, no extended memory. Best: dev/test, bursty workloads.
**N2**: Balanced, dedicated vCPU, SUD eligible, predictable. Best: most production workloads.
**C2/C3**: 10–20% premium over N2 for higher CPU clock speeds. Best: CPU-bound (HPC, game servers, ad tech).
**M1/M2**: 3–10× premium over N2 per vCPU. Best: in-memory databases (SAP HANA, Redis large instances).
**A2/G2**: GPU premium (A100 GPU costs ~$3.50/hr allocated). Best: ML training/inference where GPU provides 10–100× speedup over CPU.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People assume "more vCPUs = better" universally. A 64-vCPU E2 machine may actually perform worse for a single-threaded database than a 4-vCPU N2 machine because E2's shared-core CPU throttles under sustained load.

### B. TECHNICAL EXPLANATION
- **Misconception**: More vCPUs always improve performance. Reality: Single-threaded applications benefit from clock speed (C2) not vCPU count. Over-vCPU-ing wastes money.
- **Misconception**: E2 and N2 of the same size are equivalent. Reality: E2 uses shared vCPUs; N2 uses dedicated vCPUs. For latency-sensitive workloads, E2 can be significantly slower.
- **Misconception**: Machine type can be changed without downtime. Reality: Changing machine type requires stopping the VM (brief downtime), then resizing, then restarting.
- **Misconception**: GPUs work with any machine type. Reality: GPUs are only available with specific accelerator-optimized machine families (A2, A3, G2) or as add-ons to certain N-series types.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
An expert systems architect benchmarks their workload on 2–3 machine types before committing, uses custom types to right-size memory-to-CPU ratios, and lets Rightsizing Recommendations validate their choices after 2 weeks of production data.

### B. TECHNICAL EXPLANATION
Expert patterns:
- **Benchmark before committing**: Deploy to N2 standard, run load tests, examine CPU and memory utilization. If CPU is bottleneck → C2. If memory is bottleneck → M1. If both are fine at 40% utilization → downsize to N2 or E2.
- **Custom types for precise fit**: A workload needing 6 vCPU + 20 GB RAM has no exact predefined match. `n2-custom-6-20480` avoids paying for the extra 12 GB of an n2-standard-8 (32 GB).
- **GPU preemptibility**: A2/G2 support Spot VMs at ~60–70% discount. ML training jobs with checkpointing can run on Spot A2 instances at massive cost savings.
- **NUMA awareness**: For high-performance databases on large M1/M2 instances, tune memory allocation to NUMA topology to avoid cross-NUMA memory access penalties.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Machine type families are specialized tools: pick the right tool for the job — general (N2/E2), compute-heavy (C2), memory-heavy (M1/M2), or GPU-powered (A2/G2).

### B. TECHNICAL SUMMARY
Compute Engine machine type families are hardware-differentiated VM configurations optimized for different workload profiles. N2 is the default for production; E2 for cost-sensitive dev/test; M-series for high-memory workloads; C-series for CPU-intensive computing; A/G-series for GPU workloads. Matching the family to workload resource requirements prevents both over-provisioning (wasted cost) and under-provisioning (performance degradation).

---
---

# Custom Machine Types — Dual-Layer Explanation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Predefined machine types are like buying shoes in standard sizes (4, 6, 8, 10). Custom machine types are like having shoes custom-made — you specify exactly the size you need, paying only for the material used. If your foot is size 7.5, you no longer have to buy a size 8 and waste the extra space.

### B. TECHNICAL EXPLANATION
Custom machine types let you specify an exact combination of vCPUs and memory (in GB) that doesn't correspond to any predefined machine type. They solve the problem of workloads whose resource requirements fall between predefined sizes. For example, a workload needing 6 vCPUs and 20 GB RAM has no predefined match in the N2 family (n2-standard-4 = 16 GB, n2-standard-8 = 32 GB). A custom type `n2-custom-6-20480` (6 vCPU, 20 GB) avoids paying for 12 GB of unused RAM. Available on N1, N2, N2D, and E2 series.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
You tell Google "I need 6 CPU cores and 20 GB of memory." Google assembles a VM from the physical hardware pool with exactly those resources — billing you for 6 vCPU-hours and 20 GB memory-hours, not a moment more.

### B. TECHNICAL EXPLANATION
Custom types are created by specifying:
- **vCPU count**: Any even number (or 1) within the series limits.
- **Memory**: Between 0.9 GB and 6.5 GB per vCPU (standard limit); must be a multiple of 256 MB.
- **Series prefix**: `n2-custom`, `n2d-custom`, `e2-custom`, etc.

Billing is per vCPU-hour + per GB-hour at the custom type rate (slightly higher per unit than predefined types, but the total can be lower due to right-sizing).

**Extended memory**: If a workload requires more than 6.5 GB/vCPU, you can add extended memory up to ~8 GB/vCPU (varies by series) at an additional cost per GB-hour. Example: `n2-custom-4-32768-ext` (4 vCPU, 32 GB RAM with extended memory, which is 8 GB/vCPU).

---

## 3. MENTAL MODEL

### A. ANALOGY
Standard predefined types are whole pizza sizes (small, medium, large). Custom types let you order a pizza with exactly 7 slices — you pay for 7 slices, not 8.

### B. TECHNICAL EXPLANATION
The mental model: custom types are not a different category of hardware — they are the same physical hardware as predefined types but with fine-grained resource allocation. The underlying billing model (per vCPU-hour + per GB memory-hour) is exposed directly for custom types, whereas predefined types bundle this at a fixed ratio. The slight per-unit premium on custom types (vs predefined) means you break even only when the right-sizing saves enough unused resources to offset the per-unit price increase.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Use custom types when your workload consistently needs an odd ratio of CPU to memory that doesn't match any standard size — like needing a car with 4 seats but no back doors.

### B. TECHNICAL EXPLANATION
Good candidates for custom types:
- A data processing job needing 6 vCPU and 20 GB RAM (between n2-standard-4 and n2-standard-8).
- A caching layer needing 2 vCPU and 12 GB RAM (standard n2 would give 2 vCPU / 8 GB; custom allows 12 GB).
- A legacy application that historically ran on a server with a non-standard CPU:memory ratio.

Command example:
```bash
gcloud compute instances create my-vm \
  --machine-type=n2-custom-6-20480 \
  --zone=us-central1-a
```
This creates an N2 VM with 6 vCPUs and 20 GB (20480 MB) of memory.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
Google's hypervisor allocates memory and CPU slots from physical server pools. Custom types simply specify different allocation amounts from the same pool. The hardware doesn't change — only how much of it is assigned to your VM.

### B. TECHNICAL EXPLANATION
Custom machine type constraints by series:
- **N2**: 2–128 vCPU; 1–6.5 GB/vCPU (or up to ~8 GB with extended memory); memory in 256 MB increments.
- **N2D**: Same memory rules as N2; AMD EPYC hardware.
- **E2**: 2–32 vCPU; 0.5–8 GB/vCPU (E2 has a wider range but shared-core behavior applies to small sizes).
- **N1**: Legacy; 1–96 vCPU; 0.9–6.5 GB/vCPU standard, up to 624 GB with extended memory.

The slight per-unit pricing premium for custom vs predefined is approximately 5–10% more per vCPU-hour and per GB-hour. This is offset when the size reduction in memory or vCPU exceeds the premium.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
Ordering custom shoes sounds great until you realize you need them in 3 days — standard sizes ship faster. Custom types have subtle limits: memory must be in exact 256 MB increments, certain vCPU counts aren't allowed (must be even or 1), and extended memory has a hard ceiling.

### B. TECHNICAL EXPLANATION
- **Memory must be multiple of 256 MB**: Requesting 19,000 MB will fail; must be 19,200 MB or 19,456 MB (nearest valid values).
- **vCPU count restrictions**: Must be 1 or an even number. 3 vCPUs is invalid for most series.
- **Extended memory ceiling**: Each series has a maximum extended memory. Exceeding it requires moving to a memory-optimized series (M1/M2).
- **Slightly higher per-unit cost**: If workload actually fits within a predefined type at equivalent utilization, the custom type costs more per unit than the predefined equivalent.
- **Series availability**: Custom types only for N1, N2, N2D, E2. C2, M1, A2 do not support custom configurations — you must choose from their predefined types.

---

## 7. TRADE-OFFS

### A. ANALOGY
Custom shoes fit perfectly but cost more per shoe and take longer to make. The value is only realized if the perfect fit saves you enough money in other areas (like not buying two pairs of standard shoes).

### B. TECHNICAL EXPLANATION
**Pros:** Eliminate waste from overprovisioned memory or vCPU; reduce monthly cost for steady-state, consistently-sized workloads.
**Cons:** Slightly higher per-unit pricing (5–10%); management complexity at scale (tracking many custom type sizes vs a few standard ones); Sustained Use Discounts still apply (custom types in eligible families get SUDs, same as predefined).
**Break-even analysis**: If an n2-standard-8 (8 vCPU, 32 GB) is only using 6 vCPU and 20 GB, the custom n2-custom-6-20480 costs less: (6 × vCPU rate × 1.05) + (20 × mem rate × 1.05) vs (8 × vCPU rate) + (32 × mem rate). The 2 saved vCPUs and 12 saved GB typically offset the 5% per-unit premium.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People assume custom types are a completely different product line. They're not — they're the same hardware, same performance characteristics, just with different resource quantities allocated.

### B. TECHNICAL EXPLANATION
- **Misconception**: Custom types are always cheaper than predefined. Reality: They're cheaper only when the right-sizing saves enough resources to offset the per-unit premium. If a predefined type already fits well, it's slightly cheaper per unit.
- **Misconception**: Custom types are available for all machine families. Reality: Only N1, N2, N2D, E2. Not available for C2, M1, M2, A2, G2.
- **Misconception**: Memory amount can be any value. Reality: Must be a multiple of 256 MB; must be between 0.9 GB/vCPU and 6.5 GB/vCPU (or extended memory limit).
- **Misconception**: Extended memory is unlimited. Reality: Extended memory has a hard cap per series. Beyond that, you need an M-series machine.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Experts don't design systems around custom types from day one. They deploy on standard types, collect two weeks of utilization data from Cloud Recommender, then decide if a custom type provides enough savings to justify the slight management complexity.

### B. TECHNICAL EXPLANATION
Expert approach:
- Deploy on a known predefined type (e.g., n2-standard-8).
- After 2 weeks, check Cloud Recommender rightsizing suggestions.
- If consistently underutilizing memory (6 vCPU at 80% but memory at 30% of 32 GB), calculate: n2-custom-6-24576 vs n2-standard-8.
- Custom types also receive SUDs — a VM running 100% of the month on n2-custom-6-24576 gets the same ~30% SUD as an n2-standard-8.
- At scale (hundreds of VMs), even a 10% cost reduction via custom types becomes significant monthly savings.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Custom machine types are custom-fit shoes: they cost slightly more per unit but save money when your exact size falls between standard options.

### B. TECHNICAL SUMMARY
Custom machine types allow specifying exact vCPU and memory combinations outside predefined sizes, available on N1, N2, N2D, and E2 series. Memory must be in 256 MB increments between 0.9 GB and 6.5 GB per vCPU (extendable to ~8 GB). They carry a slight per-unit price premium but reduce total cost when workloads would otherwise use significantly underutilized predefined types.

---
---

# Preemptible VMs and Spot VMs — Dual-Layer Explanation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Preemptible and Spot VMs are like standby airline seats. They're dramatically cheaper than full-price tickets, but the airline can bump you if the flight fills up with paying passengers. You get 30 seconds' notice, your bags are unloaded, and you have to rebook. If your journey is flexible and you can restart, you save 90%. If you have a critical meeting at arrival, don't buy standby.

### B. TECHNICAL EXPLANATION
Preemptible VMs and Spot VMs are Compute Engine VM instances offered at up to 90% discount compared to standard on-demand pricing. In exchange, Google can terminate ("preempt") them with a 30-second shutdown notice whenever it needs the physical capacity for standard workloads. Preemptible VMs have an additional constraint: they are always terminated after 24 hours maximum. Spot VMs are the modern successor — same discounting and preemption behavior, but without the 24-hour runtime cap. Neither type is eligible for Sustained Use Discounts (they already carry the maximum discount).

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
Your standby seat is allocated from unsold capacity. When a last-minute paying customer arrives, you're bumped. The airline sends a flight attendant to your seat 30 seconds before departure to ask you to leave. You then wait for the next available flight (the VM is terminated, and you must restart or resubmit your job from a checkpoint).

### B. TECHNICAL EXPLANATION
1. You request a Spot (or Preemptible) VM through the console, CLI, or API.
2. GCP allocates it from surplus physical capacity.
3. If GCP needs the physical resources for standard VM demand, a preemption signal is sent to the VM.
4. The VM has 30 seconds to handle `ACPI G2 soft-off` signal (shutdown hook).
5. After 30 seconds (or sooner if the workload acknowledges), the VM is forcibly terminated.
6. You are responsible for detecting the shutdown and checkpointing state before termination.
7. Preemptible VMs additionally have a hard 24-hour maximum runtime — they are terminated regardless at the 24-hour mark. Spot VMs have no such limit.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of Spot/Preemptible VMs as computing resources you borrow, not own. You can use them as long as nobody else needs them. Your workload must be designed to survive being interrupted at any random point.

### B. TECHNICAL EXPLANATION
The mental model: Spot/Preemptible VMs are designed exclusively for fault-tolerant, restartable workloads. Correctness of the workload's design depends on: (1) checkpointing state to durable storage (Cloud Storage, Persistent Disk) before potential preemption; (2) detecting the preemption signal (`/shutdown` metadata server flag or ACPI signal) and saving progress; (3) a retry/restart mechanism (Managed Instance Group with auto-healing, or job queue like Cloud Tasks). The 90% discount makes these ideal for bulk compute where the savings justify the engineering investment.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Perfect use cases: rendering a 3D movie frame by frame (each frame is independent, checkpointable); training a machine learning model (save model weights every 1000 steps); running genomic analysis (process 1 chromosome at a time, save results); batch ETL (process each file independently).

### B. TECHNICAL EXPLANATION
Ideal workloads for Spot/Preemptible VMs:
- **ML training with checkpointing**: TensorFlow/PyTorch save model checkpoints every N steps; on preemption, resume from last checkpoint.
- **Batch data processing**: Process independent data chunks; on preemption, reprocess the incomplete chunk.
- **HPC simulations**: Save simulation state; resume from last saved state.
- **Rendering pipelines**: Each render job is independent; preempted frames are re-submitted.
- **CI/CD build agents**: Each build is stateless; preemption requires only a rebuild.

Deployment via Managed Instance Group:
```bash
gcloud compute instance-templates create spot-template \
  --provisioning-model=SPOT \
  --machine-type=n2-standard-4
```

Using the `--provisioning-model=SPOT` flag sets Spot VM behavior.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
Standby seats exist because airlines want to fill planes 100%. They'd rather give standby passengers a discounted seat than fly with empty rows. But operational priority always goes to paying passengers. Similarly, Google would rather monetize idle hardware at 10 cents than leave it unused — but standard customers always take priority.

### B. TECHNICAL EXPLANATION
**Preemption mechanics**: GCP's resource scheduler monitors physical server load across each zone. When a zone's demand for standard VMs exceeds capacity, the scheduler selects candidate Spot/Preemptible VMs for termination based on internal priority algorithms. The selection is not purely random — factors include time running and machine type availability.

**Preemption rate**: Typically 5–15% per day for most machine types and regions, but can spike to 50%+ during high-demand periods (e.g., holiday season for N2 in popular regions). Preemption rates are not guaranteed.

**Shutdown signal handling**: The VM receives `ACPI G2 soft-off` or can poll the metadata server at `http://metadata.google.internal/computeMetadata/v1/instance/preempted`. A startup script can check this flag on restart to detect preemption vs normal shutdown.

**24-hour preemptible limit**: Preemptible VMs (legacy) have this limit as a billing and capacity management mechanism. Spot VMs removed this constraint, making them viable for longer-running but still interruptible jobs.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
A movie render job that takes 30 hours: with Preemptible VMs (24h cap), the job is forcibly terminated before completion. With Spot VMs, it may complete if not preempted. But if it IS preempted mid-render without checkpointing, you lose 30 hours of work. The key engineering question is: how much work can I afford to lose at any preemption point?

### B. TECHNICAL EXPLANATION
- **No checkpointing = data loss on preemption**: The most common mistake. If your job has no checkpointing, a preemption may require restarting from the beginning.
- **Preemptible 24-hour limit for long jobs**: A batch job running 28 hours on Preemptible VMs will be forcibly terminated at the 24-hour mark regardless of GCP capacity. Use Spot VMs for jobs exceeding 24 hours.
- **Database on Spot/Preemptible**: Never run a primary database on Spot/Preemptible. Even 30-second preemption windows cause data corruption, connection drops, and failover cascades.
- **Preemption during critical sections**: If the 30-second shutdown window is too short to complete a checkpoint, the VM terminates with partially written state. Design checkpoints to be atomic.
- **Availability variability**: Some zones/regions have higher preemption rates. For time-sensitive batch jobs, spread across multiple zones or have fallback standard VMs.

---

## 7. TRADE-OFFS

### A. ANALOGY
Standby tickets save you 90% but require flexibility. If your travel is critical and time-sensitive, pay full price. If you're backpacking with no fixed agenda, standby is brilliant value.

### B. TECHNICAL EXPLANATION
**Pros:** Up to 90% cost reduction; same performance as standard VMs while running; no difference in network or disk performance; Spot VMs have no runtime limit.
**Cons:** Can be preempted at any time; requires fault-tolerant application design (checkpointing, retry logic); preemption rate varies unpredictably; not eligible for SUD or CUD (already discounted).
**vs Standard VMs**: Standard VMs are never preempted (barring hardware failure), making them suitable for stateful, SLA-bound workloads. Spot/Preemptible are unsuitable for those cases.
**Preemptible vs Spot**: For new workloads, always choose Spot over Preemptible. The 24-hour cap on Preemptible is the only material disadvantage, and Spot removes it with no trade-off.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People assume preemption is rare and they'll almost never be bumped. The 5–15% daily preemption rate means in a fleet of 100 Spot VMs, 5–15 will be terminated on a typical day. Design assumes preemption is normal, not exceptional.

### B. TECHNICAL EXPLANATION
- **Misconception**: Preemption is rare and only happens during unusual events. Reality: Preemption is an expected, normal event. Workloads must handle it gracefully.
- **Misconception**: Preemptible and Spot VMs are the same. Reality: Preemptible has a 24-hour max runtime cap; Spot does not. For new workloads, Spot is always preferred.
- **Misconception**: You can use Spot VMs for databases if you have HA configured. Reality: Even with HA, Spot preemption causes failover events, transaction rollbacks, and connection pool disruption. Never use Spot for database primaries or replicas.
- **Misconception**: Spot VMs receive Sustained Use Discounts. Reality: Spot VMs already carry the maximum discount; SUD cannot stack on top.
- **Misconception**: The 30-second shutdown window is enough time for any cleanup. Reality: For large checkpoints or database flushes, 30 seconds may be insufficient. Design checkpoint operations to complete in under 20 seconds.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
An expert batch-compute engineer designs every job as if preemption is guaranteed within the next 5 minutes. Checkpoints are small and fast. Job progress is tracked in Cloud Storage. The "resume from checkpoint" code path is tested more than the "happy path" code path.

### B. TECHNICAL EXPLANATION
Expert patterns:
- **Managed Instance Group (MIG) with auto-healing**: Configure a MIG with Spot VMs and an instance template. MIG automatically replaces preempted instances. Combined with a job queue (Cloud Tasks, Pub/Sub), this creates a self-healing batch compute fleet.
- **Preemption-aware shutdown script**: Add a startup script that checks the `preempted` metadata flag; on True, log preemption and restart from the last checkpoint.
- **Checkpoint frequency vs overhead trade-off**: Checkpointing every 5 minutes vs every 1 hour is a trade-off between overhead (5-min: more I/O) and re-work (1-hour: up to 1 hour of re-computation). For ML training, checkpointing every ~30 minutes is a common balance.
- **Mixed fleet**: Use CUDs for baseline standard VM capacity + Spot VMs for burst. The Spot fleet scales up during compute-heavy periods and gets terminated without cost when not needed.
- **Spot VM for cost modeling**: At 90% discount, 10 Spot VMs cost the same as 1 standard VM. A job that needs 100 standard VM-hours can be completed by 1000 Spot VM-hours at the same total cost, but 10x faster if parallelizable.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Spot/Preemptible VMs are standby airline seats — 90% cheaper, but you can be bumped at any time. Only book them for a flexible journey.

### B. TECHNICAL SUMMARY
Spot VMs (and legacy Preemptible VMs) provide up to 90% cost reduction by allowing Google to reclaim the capacity at any time with a 30-second warning. Preemptible VMs have a 24-hour maximum runtime; Spot VMs do not. Both are exclusively suitable for fault-tolerant, checkpointable workloads. Neither is eligible for CUD or SUD. For all new workloads, Spot VMs are preferred over Preemptible.

---
---

# Sustained Use Discounts (SUDs) — Dual-Layer Explanation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Sustained Use Discounts are like a loyalty discount that your gym applies automatically based on how many hours you visited that month. You don't sign up for a loyalty program — the gym just notices you've been coming every day and quietly reduces your hourly rate on your next bill. The more you attend, the bigger the discount. No paperwork, no commitment.

### B. TECHNICAL EXPLANATION
Sustained Use Discounts (SUDs) are automatic, progressive discounts that GCP applies to Compute Engine VM usage when a VM runs for a significant fraction of the billing month. No sign-up, no commitment, no configuration required. The discount scales with usage: more hours running in the month = higher discount rate. Eligibility is based on machine type family (N1, N2, N2D, C2, M1, M2 — not E2, T2D, Preemptible/Spot, or Sole-tenant nodes). For a VM running 100% of the month, the effective discount is approximately 30%.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
At the end of each month, the gym calculates your total visit hours. For the first 25% of possible visit hours: full price. For the next 25%: 20% off. For the third 25%: 40% off. For the final 25%: 60% off. The discounts apply to each "tier" of hours, not to all hours retroactively.

### B. TECHNICAL EXPLANATION
GCP calculates SUDs per month, per region, per machine type family:
1. All vCPU-hours and memory-hours for matching machine types in a region are aggregated across all VMs.
2. The first 25% of monthly hours (180 hours of a 720-hour month): no discount.
3. 25–50% of monthly hours: ~20% discount on vCPU and memory.
4. 50–75% of monthly hours: ~40% discount.
5. 75–100% of monthly hours: ~60% discount.
6. Aggregation across VMs: If you run 4 VMs each for 25% of the month, GCP combines them into an equivalent of 1 VM running 100% of the month, applying the tiered discount to the combined usage.
7. The effective blended discount for a VM running 100% of the month: approximately 30%.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of SUDs as a "volume over time" discount — the longer your VMs run within a calendar month, the higher the discount tier they reach. The discount is applied to the billing unit (vCPU-hours and memory GB-hours), not to a specific VM.

### B. TECHNICAL EXPLANATION
Critical mental model points:
- SUDs are **resource-level discounts**, not VM-level. GCP aggregates all N2 vCPU-hours in a region across all your VMs for the month.
- This means: running 4 VMs each for 7.5 days (25% of month each) = GCP bills as if 1 VM ran for 30 days (100% of month). You receive the full SUD tiering on the combined resource consumption.
- SUDs and CUDs are **mutually exclusive for the same usage** — GCP applies the higher discount.
- SUDs reset every calendar month.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
You run a production web server on an N2 machine that stays on 24/7. You do nothing. At the end of the month, Google has automatically applied a ~30% discount to your bill. No action needed — just keep the VM running.

### B. TECHNICAL EXPLANATION
SUDs apply automatically to:
- N1, N2, N2D, C2, M1, M2 machine families (predefined and custom types within these families)
- Standard and custom machine types within eligible families
- Compute Engine VMs (not GKE Autopilot Pods, not App Engine Flexible VMs)

SUDs do NOT apply to:
- E2 machine types (already cheaper by ~31%)
- T2D, T2A machine types
- Preemptible/Spot VMs (already discounted ~90%)
- Sole-tenant nodes
- App Engine Flexible (underlying VMs are managed separately)
- Cloud SQL, Cloud Spanner (not Compute Engine)

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The gym's accounting system doesn't track which specific locker you used on each visit — it tracks total visit-hours by membership tier. Similarly, GCP doesn't track SUD per individual VM — it tracks total vCPU-hours and memory-hours per machine family per region per project.

### B. TECHNICAL EXPLANATION
SUD calculation algorithm:
1. At the end of each billing month, GCP queries all vCPU-hours and memory GB-hours for each machine family in each region.
2. Usage is sorted into the four time buckets (0–25%, 25–50%, 50–75%, 75–100% of month).
3. The appropriate discount rate is applied to each bucket.
4. The discounted amounts are credited on the billing invoice.

For mixed VM sizes (e.g., some 4-vCPU, some 8-vCPU VMs): GCP calculates normalized vCPU-hours. 4× 4-vCPU VMs running half the month = 8 vCPU-hours × 360 hours = 2,880 vCPU-hours, which maps to the SUD tiers proportionally.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
The discount doesn't apply if you're using the wrong type of gym membership (E2, Preemptible). It also doesn't stack with a pre-paid annual pass (CUD). And changing your machine type mid-month might put you in a different "tier" bucket than you expected.

### B. TECHNICAL EXPLANATION
- **E2 machines never receive SUD**: Despite being eligible-looking general-purpose VMs, E2 is explicitly excluded. A common exam trap.
- **CUD + SUD don't stack**: If a CUD covers your N2 usage, SUD provides no additional benefit for that covered usage. GCP applies the higher of the two.
- **Project-level calculation**: SUDs are calculated per project, not per billing account. If you have 10 projects each with VMs running 10% of the month, they do NOT combine across projects — each project gets calculated separately (low discount). To consolidate, put VMs in fewer projects.
- **Machine family boundary**: N2 and N2D are separate families for SUD purposes. Hours from N2 VMs don't combine with N2D hours.

---

## 7. TRADE-OFFS

### A. ANALOGY
SUDs are free money — you get them automatically just by keeping VMs running. The only trade-off is that they apply only to eligible machine families. If you choose E2 for cost savings, you lose SUD; but E2 is still cheaper overall because its base price is ~31% lower.

### B. TECHNICAL EXPLANATION
**Pros:** Zero effort; no commitment risk; automatic; up to ~30% effective discount for always-on VMs.
**Cons:** Lower maximum discount than CUDs (30% effective vs 57% for 3-year CUD); don't apply to the cheapest families (E2); require running VMs for most of the month to reach peak tiers.
**SUD vs CUD trade-off**: For long-running, predictable workloads committed to 1–3 years, CUD provides higher savings (37–57% vs 30%). For workloads that are mostly-on but not 100% predictable, SUD is risk-free. For new workloads, start with SUD (automatic), then evaluate CUD after 3+ months of usage data.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People think they need to "activate" or "configure" sustained use discounts. They don't. They also think all machine types qualify. They don't — E2 and Preemptible/Spot are excluded.

### B. TECHNICAL EXPLANATION
- **Misconception**: SUDs need to be configured or opted into. Reality: They are entirely automatic. The exam tests this: "What do you need to configure to receive SUDs?" — the answer is nothing.
- **Misconception**: E2 machines receive SUDs. Reality: E2 is explicitly excluded. This is a common ACE exam trap.
- **Misconception**: SUDs apply per-VM. Reality: They apply per machine family per region per project across all matching VMs. This aggregation can work in your favor.
- **Misconception**: CUD + SUD stack for extra savings. Reality: They are mutually exclusive on the same usage. GCP applies whichever is greater.
- **Misconception**: App Engine Flexible VMs receive SUDs. Reality: App Engine Flexible's underlying GCE VMs are managed by App Engine's infrastructure layer and do not receive SUD.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
A cost-savvy cloud architect chooses N2 over E2 for always-on production VMs — not because N2 is cheaper per unit, but because N2 qualifies for SUDs (30% off) while E2 doesn't (no SUD). The total monthly cost for an N2 running 24/7 can be lower than an E2 once SUD is factored in.

### B. TECHNICAL EXPLANATION
Expert SUD optimization:
- **N2 vs E2 for always-on workloads**: E2 base price is ~31% lower than N2. N2 with SUD (running 100% of month) gets ~30% off. For 24/7 workloads, the total costs are nearly identical — but N2 provides dedicated vCPUs (consistent performance) while E2 provides shared vCPUs. For production always-on workloads, N2 with SUD is generally the better choice.
- **Project consolidation for SUD amplification**: Running 10 small VMs across 10 projects gives each project a low SUD tier. Consolidating into 1 project with 10 VMs maximizes SUD accumulation per project.
- **CUD strategy**: Use 3-month historical utilization data to determine stable baseline → purchase CUD for baseline → let SUD cover any remaining eligible usage above the commitment.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Sustained Use Discounts are automatic loyalty rewards — keep your N2 VM running all month, and Google quietly applies up to a 30% discount without you asking.

### B. TECHNICAL SUMMARY
SUDs are automatic, progressive discounts applied to eligible Compute Engine VM usage (N1, N2, N2D, C2, M1, M2) based on how much of the billing month the VM runs. A VM running 100% of the month receives approximately 30% effective discount. No configuration required. E2, Preemptible/Spot, and Sole-tenant nodes are ineligible. CUDs and SUDs do not stack — GCP applies the higher discount.

---
---

# Committed Use Discounts (CUDs) — Dual-Layer Explanation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Committed Use Discounts are like signing a 1 or 3-year lease on an apartment instead of renting month-to-month. Your monthly cost drops dramatically in exchange for the commitment. If you move out early, you still owe rent for the full lease term. The key is knowing you'll need the apartment for the foreseeable future before you sign.

### B. TECHNICAL EXPLANATION
Committed Use Discounts (CUDs) are contractual pricing commitments where you agree to use a specific amount of Compute Engine resources (vCPU and memory) in a specific region for 1 or 3 years. In exchange, GCP provides discounts of up to 37% (1-year) or 57% (3-year) for resource-based CUDs. CUDs are applied automatically to matching resource usage — you do not assign them to specific VMs. Unused commitment is still billed. Two types exist: resource-based (commit to vCPU + memory amounts) and spend-based (commit to a minimum hourly spend for specific services).

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
You sign a lease for "10 rooms in the Chicago area." Any time you occupy any rooms in Chicago, the lease price applies — you don't need to specify which exact rooms. If you use fewer than 10 rooms on a given day, you still owe for 10. If you use 12 rooms, the extra 2 are charged at full month-to-month rate.

### B. TECHNICAL EXPLANATION
**Resource-based CUD mechanics:**
1. You purchase a commitment: e.g., "8 N2 vCPUs and 32 GB memory in us-central1 for 1 year."
2. GCP records this commitment against your billing account.
3. Each billing cycle, GCP compares your actual N2 usage in us-central1 to the commitment.
4. Usage up to the commitment amount is charged at the committed (discounted) rate.
5. Usage above the commitment is charged at on-demand rates (or SUD-discounted if applicable).
6. If actual usage is below the commitment, you still pay the committed price for the unused portion.
7. Commitments can be shared across projects within a billing account (Committed Use Discount sharing must be enabled).

**Spend-based CUD mechanics:**
- You commit to a minimum hourly spend (e.g., $5/hour on Cloud SQL).
- GCP applies a 25–52% discount to your usage up to the committed spend.
- Available for Cloud SQL, Cloud Spanner, Google Cloud VMware Engine, Anthos.

---

## 3. MENTAL MODEL

### A. ANALOGY
CUDs are pre-purchases of cloud capacity at a discount. You're betting that you'll need at least X vCPUs for Y years. If you're right, you save significantly. If you're wrong (overcommit), you pay for resources you don't use. The confidence of your bet determines whether you sign a 1-year or 3-year lease.

### B. TECHNICAL EXPLANATION
The correct mental model: CUDs are a commitment to a resource floor, not a ceiling. You guarantee GCP that you'll consume at least X vCPU-hours and Y GB-hours per month in a region, in exchange for a lower rate on those resources. Key insight: CUDs apply to a region's resource pool, not to specific VMs. If you delete a VM and create a different one of the same family in the same region, the new VM still benefits from the existing CUD.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
After running 20 N2 VMs in us-central1 for 6 months and seeing consistently 80 vCPUs and 320 GB of memory in use 24/7, you're confident committing to that baseline. You purchase a 3-year CUD for 80 N2 vCPUs and 320 GB in us-central1, saving 57% on that baseline.

### B. TECHNICAL EXPLANATION
Steps to purchase a CUD:
1. Analyze 3 months of historical Compute Engine usage by machine family and region.
2. Identify stable baseline (e.g., N2 in us-central1: always at least 80 vCPU, 320 GB).
3. Purchase commitment: Google Cloud Console → Compute Engine → Committed use discounts → Purchase.
4. Select: machine series (N2), region (us-central1), vCPU amount, memory amount, term (1 or 3 year).

Example CLI:
```bash
gcloud compute commitments create my-commitment \
  --plan=12-month \
  --region=us-central1 \
  --resources=vcpu=80,memory=327680MB \
  --type=general-purpose
```

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The lease is per city (region) and per apartment type (machine family). Your Chicago lease doesn't cover apartments in New York. Your standard apartment lease doesn't cover penthouses (GPU-enabled VMs). But within Chicago, any apartment of the right type counts — it doesn't matter which specific building.

### B. TECHNICAL EXPLANATION
CUD specificity dimensions:
- **Region-scoped**: A CUD for us-central1 N2 vCPUs does not cover N2 vCPUs in europe-west1. Separate commitments required per region.
- **Machine family-scoped**: N2 and N2D are separate commitment types. A commitment for N2 vCPUs doesn't cover N2D vCPUs.
- **Shared CUDs**: Enabled at the billing account level. Allows the CUD to cover matching usage across all projects attached to the billing account. Without sharing enabled, each project's CUD covers only that project's usage.
- **CUD + SUD interaction**: For any usage covered by a CUD, SUD does not apply. For usage exceeding the CUD (on-demand rate), SUD applies if the machine type is SUD-eligible.
- **3-year CUD rates**: Resource-based CUD 3-year discount: up to 57% on N2 vCPU, up to 57% on N2 memory.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
Signing a 3-year lease and then needing to move to a different city after 6 months means paying rent on two apartments. Overcommitting resources, then scaling down, leaves you paying for unused capacity.

### B. TECHNICAL EXPLANATION
- **Overcommitment risk**: If you commit to 100 N2 vCPUs and later reduce to 60 due to workload changes, you still pay for 100 vCPUs. CUDs cannot be canceled or reduced.
- **Region lock-in**: If you move workloads from us-central1 to europe-west1, the us-central1 CUD still bills you. Always model migration risk before committing.
- **Machine family changes**: Migrating from N2 to N2D to take advantage of AMD pricing while holding an N2 CUD means the CUD continues billing, and N2D usage bills separately at on-demand.
- **Spend-based CUD overrun**: If actual spend exceeds the committed hourly amount, the excess is charged at on-demand rates without the CUD discount.
- **Project-level isolation without sharing**: A CUD on Project A does not benefit Project B unless shared CUDs are explicitly enabled on the billing account.

---

## 7. TRADE-OFFS

### A. ANALOGY
A 3-year lease gives the deepest discount but locks you in the longest. A 1-year lease is less risky but less rewarding. Month-to-month (SUD) is the least commitment and still gives about 30% off — no paperwork required.

### B. TECHNICAL EXPLANATION
**1-year CUD**: ~37% discount on resource-based vCPU/memory. Lower risk (1-year commitment). Best for reasonably stable workloads where 12-month forecast is confident.
**3-year CUD**: ~57% discount. Highest savings but highest risk (3 years of commitment). Best for stable, long-running production infrastructure with no anticipated significant scale-down.
**SUD (no commitment)**: ~30% effective discount for always-on VMs. Zero risk, zero paperwork. Slightly lower discount than 1-year CUD.
**Decision rule**: If 3-year commitment is confident → 3-year CUD (57%); if 1-year is confident → 1-year CUD (37%); if neither → SUD (automatic, ~30%).

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People assume the CUD is like a coupon applied to specific VMs. It isn't — it's a lease on resource amounts in a region, applied automatically to any matching VM usage, regardless of which VMs.

### B. TECHNICAL EXPLANATION
- **Misconception**: CUDs apply to specific VMs. Reality: CUDs apply to resource amounts (vCPU + memory) in a region/family. Any matching VM usage counts against the commitment.
- **Misconception**: Unused CUD commitment is refunded or rolled over. Reality: Unused commitment is billed at the committed rate. There is no rollover or refund.
- **Misconception**: CUD and SUD stack for extra savings. Reality: They are mutually exclusive on the same usage. GCP applies the greater discount.
- **Misconception**: CUDs cover all machine types. Reality: Spend-based CUDs cover Cloud SQL, Spanner, etc. Resource-based CUDs cover specific machine families. GPU machine types (A2, G2) and custom types have their own commitment pricing.
- **Misconception**: A single CUD covers multiple regions. Reality: CUDs are region-scoped. One commitment per region.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
An expert financial analyst of cloud costs never commits resources they haven't first verified through 6 months of historical data. They commit only the stable baseline (the floor), use Spot VMs for burst above the baseline, and review commitments annually as part of cloud cost governance.

### B. TECHNICAL EXPLANATION
Expert CUD strategy:
- **Baseline + burst model**: Purchase CUD for the stable floor (e.g., 60 N2 vCPUs always running) → use Spot VMs for burst (60–120 vCPUs during peak) → the burst capacity costs 90% less than on-demand while the baseline has the 57% CUD discount.
- **CUD sharing**: Enable at billing account level to pool commitments across projects. This prevents the scenario where Project A has excess CUD coverage while Project B pays on-demand.
- **Rightsizing before committing**: Use Cloud Recommender rightsizing suggestions for 2–3 months before purchasing CUDs. Committing to overprovisioned sizes wastes the discount.
- **Spend-based CUDs for databases**: For Cloud SQL and Spanner with consistent usage, spend-based CUDs at 25–52% discount require only a spend commitment (more flexible than resource-specific commitments).

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Committed Use Discounts are a 1 or 3-year lease on GCP resources — sign a longer lease, pay less per month, but commit to the full term regardless of usage.

### B. TECHNICAL SUMMARY
CUDs provide up to 37% (1-year) or 57% (3-year) discounts on Compute Engine vCPU and memory in exchange for a commitment to use specific resource amounts in a specific region. They apply automatically to matching usage, are region- and family-scoped, cannot be canceled, and bill for unused committed capacity. CUDs do not stack with SUDs — GCP applies the greater discount. Use for stable, predictable baseline workloads with high confidence in multi-year utilization.

---
---

# Sole-Tenant Nodes — Dual-Layer Explanation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A sole-tenant node is like renting an entire hotel floor exclusively for your company, rather than just individual rooms. Other hotel guests can't access your floor. You get the whole thing to yourself, even if some rooms are empty. The cost is higher, but the privacy and exclusivity are guaranteed.

### B. TECHNICAL EXPLANATION
Sole-tenant nodes are dedicated physical servers in GCP's data centers allocated exclusively to a single customer. Unlike standard Compute Engine where VMs from multiple customers may run on the same physical host, sole-tenant nodes guarantee no other customer's VMs share your physical hardware. Use cases: (1) compliance requirements mandating physical hardware isolation (e.g., PCI DSS, HIPAA workloads requiring physical separation); (2) Bring Your Own License (BYOL) for Windows Server or SQL Server (Microsoft licensing requires dedicated hardware for BYOL); (3) workloads with regulatory or contractual requirements for hardware isolation.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
You rent an entire hotel floor (the physical server). You can then place your guests (VMs) in any rooms on your floor. If a room is empty, you still pay for the floor. Other guests (other customers' VMs) can never enter your floor — the elevator key only works for your party.

### B. TECHNICAL EXPLANATION
1. You create a sole-tenant node template specifying machine family and node type (e.g., `n2-node-80-640`: 80 vCPU, 640 GB RAM physical server).
2. GCP allocates a physical server dedicated to your project.
3. You create VMs with the `--node-affinity` flag specifying the sole-tenant node (or node group).
4. Those VMs are scheduled only on your dedicated physical server(s).
5. You pay for the entire physical server capacity regardless of how many or how few VMs are running on it.
6. No other Google Cloud customer's workloads run on that physical server.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of sole-tenant nodes as "pay for the whole server, not just the VMs." You're buying hardware isolation, not just resource allocation. The cost model changes from "pay per VM" to "pay per physical server."

### B. TECHNICAL EXPLANATION
The mental model shift: standard Compute Engine is a shared multi-tenant infrastructure. Sole-tenant nodes are single-tenant infrastructure, mirroring on-premises data center ownership. Key implications:
- Billing is for the node (physical server) regardless of VM density.
- You can optimize density by packing multiple VMs onto the node to maximize utilization.
- BYOL benefit: Microsoft allows per-core licensing to be transferred to dedicated hardware; without physical isolation, BYOL is not permitted in shared environments.
- No Sustained Use Discounts apply — the billing model is fundamentally different (per physical server-hour vs per VM vCPU-hour).

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Scenarios where you must rent the whole floor: (1) regulatory requirement says no other company's data can touch the same hardware as yours; (2) you have an expensive Windows Server license from your old data center that you want to move to GCP without buying a new license.

### B. TECHNICAL EXPLANATION
Use sole-tenant nodes for:
- **BYOL Windows Server**: Microsoft BYOL requires dedicated hosts. Sole-tenant nodes are the only GCP option for this.
- **BYOL SQL Server**: Same Microsoft licensing requirement.
- **Compliance/regulatory**: Workloads under FedRAMP High, HIPAA, or PCI DSS may require physical isolation documentation.
- **Hardware-level performance isolation**: Workloads that need guaranteed CPU performance without any noise from neighboring VMs (though this is less common — N2's dedicated vCPUs already provide strong isolation).

Configuration:
```bash
# Create a node group
gcloud compute sole-tenancy node-groups create my-node-group \
  --node-template=my-node-template \
  --target-size=1 \
  --zone=us-central1-a

# Create VM on sole-tenant node
gcloud compute instances create my-vm \
  --machine-type=n2-standard-8 \
  --node-group=my-node-group \
  --zone=us-central1-a
```

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The hotel floor (physical server) has a fixed number of rooms (CPU cores and memory). You can arrange the rooms however you want — big suites (large VMs) or small rooms (small VMs). But the total number of rooms is fixed by the floor plan (server hardware specs).

### B. TECHNICAL EXPLANATION
Sole-tenant node types define the physical server specs:
- `n1-node-96-624`: 96 vCPU, 624 GB RAM (older N1 hardware)
- `n2-node-80-640`: 80 vCPU, 640 GB RAM (N2 hardware)
- Additional types available in specific regions

VM packing on a node: You can place multiple VMs on a single sole-tenant node as long as their total vCPU and memory don't exceed the node's capacity. This is important for cost efficiency — a sparsely populated node wastes the dedicated server cost.

Memory overcommit is not supported on sole-tenant nodes — total VM memory must not exceed physical memory.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If you only put one small guest in a 30-room hotel floor, you're paying for 30 rooms to host one person. Poor utilization = poor cost efficiency. If the floor itself has a maintenance issue, all your guests are affected — there's no automatic migration to a different floor.

### B. TECHNICAL EXPLANATION
- **Poor VM density = high cost**: A sole-tenant node with a single 4-vCPU VM wastes 76 vCPUs of an 80-vCPU node. The cost is the same as fully utilizing all 80 vCPUs. Pack VMs densely or avoid sole-tenant for small workloads.
- **No SUD**: The cost efficiency mechanism of sustained use discounts doesn't apply. For always-on workloads, this means sole-tenant has a higher baseline cost than equivalent standard VMs with SUD.
- **Limited availability**: Not all regions/zones have all sole-tenant node types. Check availability before designing around them.
- **Host maintenance**: Google must occasionally perform maintenance on physical hosts. This may require live migration of VMs (which is supported) or a restart window. Plan accordingly.

---

## 7. TRADE-OFFS

### A. ANALOGY
The whole floor is expensive but provides privacy, exclusivity, and licensing rights. Sharing a room is cheaper but you might share walls with noisy neighbors and can't bring your own furniture under some rules.

### B. TECHNICAL EXPLANATION
**Pros:** Hardware isolation for compliance; enables Microsoft BYOL; guarantees no cross-customer physical colocation; full control over VM scheduling on the node; better performance isolation for sensitive workloads.
**Cons:** Higher cost than standard VMs; no SUD; poor cost efficiency at low VM density; limited node type availability; not suitable for cost-sensitive workloads without compliance requirements.
**vs Standard VMs**: For workloads without licensing or compliance requirements, standard Compute Engine VMs are significantly cheaper (with SUD or CUD) and provide equivalent logical isolation (Google's hypervisor provides strong VM isolation even in multi-tenant environments).

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People assume that because they can't see their neighbors, the building is inherently more secure than a regular hotel. In reality, Google's hypervisor provides very strong security isolation in standard multi-tenant environments. Sole-tenant adds physical isolation, which matters for specific compliance frameworks, not for general security.

### B. TECHNICAL EXPLANATION
- **Misconception**: Sole-tenant VMs receive Sustained Use Discounts. Reality: No SUD for sole-tenant nodes. This is explicitly tested in the ACE exam.
- **Misconception**: Sole-tenant is necessary for strong security. Reality: GCP's standard multi-tenant infrastructure provides strong hypervisor-based isolation. Sole-tenant provides physical isolation, which is required only for specific compliance frameworks that explicitly require it.
- **Misconception**: You can use BYOL on standard Compute Engine VMs. Reality: Microsoft BYOL requires dedicated hardware. Sole-tenant nodes are mandatory for BYOL Windows Server and SQL Server on GCP.
- **Misconception**: Sole-tenant nodes can be resized like standard VMs. Reality: The physical server has fixed capacity. You choose the node type at creation time and cannot resize the underlying hardware.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
An expert architect uses sole-tenant nodes only when specifically required — for BYOL licensing or specific compliance frameworks. They maximize VM density on each node (pack as many VMs as the node capacity allows) to amortize the dedicated hardware cost across as much useful work as possible.

### B. TECHNICAL EXPLANATION
Expert considerations:
- **VM bin-packing**: Fill sole-tenant nodes to >80% capacity. An n2-node-80-640 running at 30% capacity (24 vCPU utilized) is paying for 80 vCPUs. Design VM sizes to pack efficiently.
- **Node affinity rules**: Use node group labels and affinity rules to control exactly which VMs land on which nodes. This enables BYOL compliance (all licensed VMs on dedicated nodes, unlicensed VMs on standard nodes).
- **Hybrid strategy**: Use sole-tenant for BYOL Windows VMs and standard Compute Engine (with SUD/CUD) for all other VMs. Don't put Linux workloads on sole-tenant unless required.
- **Cost comparison before deployment**: Calculate: (sole-tenant node monthly cost) / (number of VMs packed × average VM size). Compare to (equivalent standard VM monthly cost × number of VMs × SUD factor). Sole-tenant is justifiable only when the BYOL savings or compliance requirement outweighs the premium.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Sole-tenant nodes are renting an entire hotel floor for your VMs — complete hardware isolation and Microsoft BYOL rights, but you pay for the whole floor whether it's full or not.

### B. TECHNICAL SUMMARY
Sole-tenant nodes provide dedicated physical servers exclusively for one customer's VMs, enabling compliance requirements for physical isolation and Microsoft BYOL licensing. They do not receive Sustained Use Discounts, have a higher baseline cost than multi-tenant VMs, and require careful VM density planning to be cost-effective. Use only when compliance or BYOL licensing explicitly requires physical hardware isolation.
