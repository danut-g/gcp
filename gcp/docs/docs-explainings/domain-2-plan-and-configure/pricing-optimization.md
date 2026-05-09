# Pricing Optimization: SUDs, CUDs, Rightsizing, Cost Strategy — Dual-Layer Explanation

---

# Sustained Use Discounts (SUDs)

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Imagine a gym membership where the more hours you spend at the gym each month, the cheaper your hourly rate becomes — automatically, no contract needed. You don't sign up for anything; the gym just tracks your usage and gives you a better rate the more you show up.

### B. TECHNICAL EXPLANATION
Sustained Use Discounts are automatic, incremental discounts applied by GCP to Compute Engine VM usage when a vCPU or memory resource runs for a significant fraction of a billing month. No commitment, no sign-up, no configuration required. GCP monitors usage per resource type per region per project and applies sliding discounts as the percentage of the month the resource runs increases. A VM running 100% of the month receives an effective ~30% discount.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
GCP pools all your VMs of the same type in the same region together like gym visits. If you had four members each going 25% of the month, the gym sees that as one "full member" and applies the full-month discount across that pool.

### B. TECHNICAL EXPLANATION
SUDs are calculated per-month, per-region, per-machine-type-family — not per individual VM. GCP aggregates total vCPU-hours and memory-hours across all VMs of the same family. The discount tiers are:

| % of Month Running | Discount (N1/N2) |
|--------------------|-----------------|
| 0–25% | 0% |
| 25–50% | ~20% |
| 50–75% | ~40% |
| 75–100% | ~60% |

Four VMs each running 25% = effectively one VM running 100% = full-tier discount on that aggregate.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of SUDs as a loyalty reward that the store silently tracks for you. You don't need a loyalty card — it just happens.

### B. TECHNICAL EXPLANATION
SUDs are a passive discount mechanism. The correct mental model: they apply automatically at billing time. Never "configure" SUDs. They apply to N1, N2, N2D, C2, M1, M2 machine families. They do NOT apply to E2 (already cheaper), preemptible/Spot VMs (already discounted), or T2D/T2A.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
You're running a web server 24/7 on an N2 VM. At the end of the month, your bill automatically reflects the 30% effective discount — you did nothing to earn it.

### B. TECHNICAL EXPLANATION
A production n2-standard-4 VM running all month in us-central1 automatically receives approximately 30% effective discount vs on-demand pricing. No commitment required. SUDs are visible in billing export under the "Discount" line items. Use billing export to BigQuery to verify SUD credits are being applied.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The gym's computer runs a nightly job that tallies your check-ins and adjusts your rate tier retroactively at invoice time.

### B. TECHNICAL EXPLANATION
GCP's billing system calculates usage aggregates at the end of each billing month. The aggregation is done at the (project, region, machine family) level — not per-VM. Discounts are applied as credits on the invoice. The sliding discount curve is applied to the incremental usage in each tier bracket, not a flat discount to total usage.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If you switch gyms mid-month or use a different gym chain, your hours don't aggregate — each gym tracks independently.

### B. TECHNICAL EXPLANATION
- SUDs do NOT cross projects: Each project's usage is aggregated independently.
- Switching machine families mid-month resets aggregation for that family.
- App Engine Flexible environment VMs do NOT receive SUDs, even though they run on Compute Engine under the hood.
- Sole-tenant node VMs are not eligible.

---

## 7. TRADE-OFFS

### A. ANALOGY
The gym's automatic discount is convenient but smaller than if you signed a 1-year or 3-year contract.

### B. TECHNICAL EXPLANATION
SUDs (~30% effective discount for full-month VMs) are lower than CUDs (up to 57% for 3-year commitments). The advantage: zero commitment risk — if you stop the VM, you stop paying. CUDs give a deeper discount but you pay whether you use the resource or not. For unpredictable or variable workloads, SUDs are safer. For stable baseline capacity, CUDs outperform.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People think they need to "activate" the gym's loyalty reward. They don't — it's automatic.

### B. TECHNICAL EXPLANATION
The most common misconception: SUDs require configuration. They do not. The exam trap: "What do you need to configure to get SUDs?" — Answer: nothing. Another misconception: SUDs apply to all VM types. They do NOT apply to E2, preemptible, Spot, or T2D/T2A.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
A seasoned gym-goer knows: if you're going to be there every day, lock in the annual contract (CUD) for maximum savings. Use the automatic discount (SUD) only for machines you're unsure about.

### B. TECHNICAL EXPLANATION
Senior engineers use SUDs as the "no-brainer" floor discount for standard workloads without needing capacity planning. For predictable baseline capacity known > 1 year out, they layer CUDs on top (CUD replaces SUD for covered usage). The professional approach: identify the "committed baseline" (always-on VMs) → buy CUDs; let the remainder get SUDs automatically.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Like a gym loyalty discount you never had to sign up for — the more you use it, the more you save, automatically.

### B. TECHNICAL SUMMARY
SUDs are automatic billing discounts applied to eligible Compute Engine VMs (N1, N2, N2D, C2, M1, M2 families) based on how much of the billing month the resource runs, aggregated per region per project. No configuration needed; the discount reaches ~30% effective for a full-month VM.

---

---

# Committed Use Discounts (CUDs)

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A 1-year or 3-year gym contract: you get a significant discount in exchange for paying regardless of whether you show up. The contract guarantees the gym revenue; you get a lower rate.

### B. TECHNICAL EXPLANATION
CUDs are contractual commitments to consume a defined amount of GCP compute resources (vCPUs, memory) in a specific region for 1 or 3 years. In exchange, GCP provides discounts of up to 37% (1-year) and 57% (3-year) vs on-demand pricing. There are two types: resource-based CUDs (specific vCPU/memory amounts) and spend-based CUDs (minimum hourly spend for certain services like Cloud SQL, Spanner).

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
You sign a 1-year contract for 10 gym slots per month. Whether you use all 10 or only 3, you pay for 10. But each slot costs 37% less than the drop-in rate.

### B. TECHNICAL EXPLANATION
Resource-based CUDs: You commit to N vCPUs and M GB memory in a specific region and machine family. GCP bills the committed amount at the discounted rate whether or not you use it. Usage above the commitment is billed at the standard on-demand rate (subject to SUD). Commitments can be shared across projects within a billing account via "shared CUDs." Spend-based CUDs: Commit to a minimum hourly dollar amount for a service; receives a discount on usage up to the committed amount.

---

## 3. MENTAL MODEL

### A. ANALOGY
CUDs are "pay for capacity upfront, use it flexibly." You're buying a block of capacity at a discount, not a specific VM.

### B. TECHNICAL EXPLANATION
CUDs decouple the commitment (vCPU/memory in a region) from individual VMs. You can stop and start different VMs in the same region and family — the CUD discount applies to any matching usage. The key mental model: CUDs and SUDs do NOT stack. GCP applies whichever is greater for covered usage.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
A company running 20 N2 VMs in us-central1 for 2+ years buys a 3-year CUD for 80 vCPUs to cover their baseline, saving 57% on that capacity.

### B. TECHNICAL EXPLANATION
Purchase CUDs via Console → Billing → Committed Use Discounts, or via gcloud. Specify: region, machine family, vCPU count, memory, duration. The discount is applied automatically to matching usage in that project (or across shared projects). Additional CUDs can be purchased at any time; they're additive. For Cloud SQL/Spanner: purchase spend-based CUDs committing to minimum hourly spend.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The contract specifies the gym location (region), the type of facility (machine family), and the capacity (vCPUs). You can use any machine in that family at that location and the discount applies.

### B. TECHNICAL EXPLANATION
CUD commitments are tracked against actual resource consumption in the billing system. Matching is: same project (or shared CUD scope), same region, same machine type family, same resource type (vCPU vs memory). If a CUD is partially utilized (e.g., 60 of 80 committed vCPUs are in use), the 20 unused vCPUs still incur the committed cost. Unused commitments represent wasted spend.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
You signed a contract for 10 gym slots in the downtown location. You can't use them at the suburban location, and you can't transfer the contract if you move.

### B. TECHNICAL EXPLANATION
- CUDs are per-region: A commitment in us-central1 does not cover usage in europe-west1.
- CUDs are per-machine-family: An N1 CUD does not cover N2 VMs.
- Commitments cannot be cancelled or reduced after purchase.
- CUD + SUD don't stack: For the same usage, GCP applies only the larger discount. If CUD covers 100% of usage, SUDs provide no additional benefit.

---

## 7. TRADE-OFFS

### A. ANALOGY
Annual gym contract: best value if you're committed, worst value if your plans change.

### B. TECHNICAL EXPLANATION
| | SUDs | CUDs |
|---|---|---|
| Discount | ~30% (effective, full month) | 37–57% |
| Commitment | None | 1 or 3 years |
| Risk | None | Pay even if unused |
| Flexibility | High | Low |

CUDs deliver the maximum discount for stable, predictable workloads. Use them only for "committed baseline" capacity — the minimum you'll always run.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People think "I bought 80 vCPUs of CUD, so I can use them in any datacenter." No — they're bound to the region and family you specified.

### B. TECHNICAL EXPLANATION
- CUDs are per-region AND per-machine-family — commonly forgotten on the exam.
- CUD + SUD do not stack — a classic exam trap. They're mutually exclusive for the same resource.
- Preemptible/Spot VMs cannot have CUDs applied — they're already discounted.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
A smart business buys long-term contracts only for space they're 100% sure they'll fill. They leave room for flexibility with short-term leases.

### B. TECHNICAL EXPLANATION
The expert pattern: Analyze 90-day usage trends → identify the minimum committed baseline → buy 3-year CUDs for that baseline. Buy 1-year CUDs for the next tier of "fairly certain" capacity. Let variable/burst capacity use SUDs or preemptible VMs. This layered strategy minimizes risk while maximizing discount depth. Monitor CUD utilization in Billing to avoid paying for unused commitments.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
A 1-3 year gym contract: larger discount than drop-in, but you pay whether you show up or not. Commit only what you're certain you'll use.

### B. TECHNICAL SUMMARY
CUDs are 1- or 3-year commitments to consume specific vCPU/memory quantities in a GCP region and machine family, providing up to 37%/57% discounts vs on-demand. They are per-region and per-family; they do not stack with SUDs. Unused committed capacity is still billed.

---

---

# Preemptible and Spot VMs

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Standby airline seats: massively discounted, but the airline can bump you any time they need the seat for a full-fare passenger.

### B. TECHNICAL EXPLANATION
Preemptible VMs and Spot VMs are Compute Engine instances that use surplus/idle capacity at up to ~90% discount vs standard pricing. GCP can reclaim them at any time (preempt) when it needs capacity for standard customers. Preemptible: maximum 24-hour runtime, 30-second shutdown notice. Spot: no maximum runtime, but same preemption risk.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
You take the standby seat. If the flight isn't full, you fly at a deep discount. If a full-fare passenger books the seat, you're asked to leave before takeoff or mid-flight.

### B. TECHNICAL EXPLANATION
Preemptible/Spot VMs run on Google's excess compute capacity. When GCP needs those resources for standard workloads, it sends a preemption notice (ACPI G2 Soft Off signal) 30 seconds before terminating the VM. The workload must be designed to handle graceful shutdown. Spot VMs differ from Preemptible only in that there's no 24-hour forced termination — they can run indefinitely until preempted.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of them as "best effort" compute: you get a machine when GCP has capacity, and you lose it when GCP doesn't.

### B. TECHNICAL EXPLANATION
The key mental model: preemptible/Spot VMs are for fault-tolerant, stateless, interruptible workloads only. They cannot hold application state that can't be recreated. Workloads must checkpoint progress. They are never eligible for CUDs or SUDs (already ~90% off).

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Perfect for tasks like rendering a video in 1,000 small chunks — if some chunks are interrupted, restart them. Never for a single long-running database transaction.

### B. TECHNICAL EXPLANATION
Best use cases: batch data processing, HPC simulations, CI/CD build agents, machine learning training (with checkpointing), Dataflow workers. In a MIG, mixing preemptible with standard VMs provides a cost-effective autoscaling strategy. Configure shutdown scripts to checkpoint state and gracefully drain connections.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The airline uses a dynamic algorithm: when more full-fare bookings come in, standby passengers are automatically bumped.

### B. TECHNICAL EXPLANATION
Preemptions are driven by GCP's capacity management system. Preemption probability varies by region, zone, and time of day. Typically higher in popular regions during peak hours. GCP provides 30-second notice via the preemption signal. Within a MIG, GCP replaces preempted instances automatically if min-instances requires it.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If your task takes 30 hours and the flight is only 24 hours, standby doesn't work — you'll be forced off before landing.

### B. TECHNICAL EXPLANATION
Preemptible VMs have a hard 24-hour limit — the VM is terminated regardless of workload state after 24 hours. This is the most common exam trap. For jobs requiring > 24 hours, use Spot VMs (no time limit) or standard VMs. Spot VMs can still be preempted; they just don't have the 24-hour cap.

---

## 7. TRADE-OFFS

### A. ANALOGY
Massive savings vs. reliability. Best when the task can be restarted cheaply; worst when interruption causes work to be lost.

### B. TECHNICAL EXPLANATION
- ~90% discount is compelling for batch workloads
- Cannot be combined with CUDs or SUDs
- Cannot be used for persistent stateful services (databases, session servers)
- Preemption rate depends on zone availability — vary by region

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
People think Spot VMs are "better preemptible" in every way. The only difference is the 24-hour limit — both can be preempted at any time.

### B. TECHNICAL EXPLANATION
Preemptible: max 24h, guaranteed 30s notice. Spot: no max time, but same preemption model. The key difference is the time cap, not reliability. Neither should be used for production stateful services.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Smart travelers know to book standby for short hops only — never for once-in-a-lifetime trips where missing the flight is catastrophic.

### B. TECHNICAL EXPLANATION
Expert pattern: Use Spot VMs in Dataflow, Cloud Batch, or MIG worker pools with checkpointing enabled. Design workloads to be idempotent (re-running doesn't cause errors). Monitor preemption rates per zone and shift workloads to zones with lower rates for improved reliability.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Standby seats on a flight — up to 90% cheaper, but the airline can bump you at any time with 30 seconds notice.

### B. TECHNICAL SUMMARY
Preemptible/Spot VMs provide up to ~90% cost reduction for fault-tolerant workloads using GCP surplus capacity. Preemptible VMs have a 24-hour maximum runtime; Spot VMs do not. Both can be reclaimed by GCP with 30 seconds notice and are unsuitable for stateful or latency-sensitive production workloads.

---

---

# Rightsizing with Google Cloud Recommender

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A personal trainer who reviews your gym routine and says: "You're using the Olympic lifting platform but you're only doing light stretches. Downgrade to a yoga mat — same results, 70% cheaper."

### B. TECHNICAL EXPLANATION
Google Cloud Recommender is an AI-powered service that analyzes historical resource utilization and generates actionable cost-optimization recommendations. For Compute Engine, it identifies over-provisioned VMs (running at low CPU/memory utilization) and suggests downsizing to smaller machine types or stopping idle VMs entirely.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
The trainer monitors your gym usage for 8 days, sees you only lift 5% of the maximum weight on the rack, and recommends switching to lighter equipment.

### B. TECHNICAL EXPLANATION
Recommender analyzes Compute Engine VM metrics (CPU utilization, memory utilization via Ops Agent) over an observation window (typically 8 days). It compares actual utilization to the provisioned capacity and identifies VMs consistently underutilizing resources. Recommendations appear in the GCP Console under Compute Engine → Recommendations and via the Recommender API for programmatic consumption.

---

## 3. MENTAL MODEL

### A. ANALOGY
Recommendations are suggestions, not commands. The trainer advises; you decide whether to follow.

### B. TECHNICAL EXPLANATION
Recommendations are informational. You must act on them manually (or automate via the Recommender API). They're based on historical patterns — not predictive for future spikes. Always review peak utilization before applying a recommendation.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Review the trainer's suggestions each month, accept the ones where you're genuinely over-equipped, and reject the ones where you know you'll need that heavy equipment during quarterly events.

### B. TECHNICAL EXPLANATION
Via Console: Compute Engine → Recommendations → view suggested machine type changes. Accept/dismiss individual recommendations. Via API: integrate into cost management pipelines, Slack notifications, or automated workflows. Track savings over time in the billing export.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The trainer uses 8 days of workout logs — not your seasonal schedule or special events.

### B. TECHNICAL EXPLANATION
Recommendations use a rolling 8-day observation window. They do NOT account for seasonal peaks, periodic jobs, or planned future growth. The recommendation model considers average and p95 CPU utilization. Memory recommendations require the Ops Agent to be installed (hypervisor doesn't expose memory metrics).

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If you downgrade your equipment right before a marathon season, you'll be under-equipped when it matters.

### B. TECHNICAL EXPLANATION
Applying a rightsizing recommendation that reduces capacity below peak demand causes degraded performance or service failure. Common scenario: VM looks idle for 8 days, but there's a monthly batch job on day 9. Always validate recommendations against your full operational calendar. Observe for at least one full business cycle.

---

## 7. TRADE-OFFS

### A. ANALOGY
Rightsizing saves money but requires ongoing management attention. Overprovisioning wastes money but reduces operational risk.

### B. TECHNICAL EXPLANATION
Rightsizing benefits: immediate cost reduction, better resource utilization. Risks: performance degradation if recommendations applied without validating peak usage. Best practice: accept recommendations only after verifying with > 30 days of usage data, not just the 8-day window.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"The trainer looked at my last 8 days, so the recommendation is perfect." Not if you haven't done your seasonal event yet.

### B. TECHNICAL EXPLANATION
Recommendations do not account for: scheduled jobs, seasonal peaks, planned growth, or workloads that are intentionally low-traffic (like a warm standby). They reflect observed historical patterns only.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Experienced coaches review training logs over months, not just a week, before recommending changes to an athlete's regimen.

### B. TECHNICAL EXPLANATION
Senior engineers use Recommender outputs as a starting point, not a directive. They correlate recommendations with Cloud Monitoring metrics over 30-90 days, verify against business calendars, and test downsized VMs in staging before production. They also automate custom-metric alerting to detect post-rightsizing performance regressions.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
A trainer watching your last 8 gym sessions recommending lighter weights — valid advice most of the time, dangerous if you're training for a competition next week.

### B. TECHNICAL SUMMARY
Google Cloud Recommender analyzes 8 days of VM utilization metrics and suggests downsizing overprovisioned instances. Recommendations are informational — you must apply them manually. They do not account for periodic peaks or future workload changes; always validate before applying.
