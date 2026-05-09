# Pricing Optimization: SUDs, CUDs, Rightsizing, and Cost Strategy

## Overview

Cost optimization is a recurring theme across the ACE exam. Understanding Google Cloud's automatic and manual discount mechanisms, rightsizing strategies, and cost visibility tools allows architects to design cost-efficient solutions. This topic connects billing fundamentals with compute planning.

---

## Key Concepts

### Sustained Use Discounts (SUDs)

Automatically applied when Compute Engine VMs run for a significant portion of the billing month.

#### How SUDs Work

- Calculated automatically per month, per region, per machine type family
- GCP computes the proportion of the month each vCPU/memory unit ran
- The more a resource runs, the higher the discount
- No sign-up, no commitment, no action needed

#### SUD Discount Table (N1, N2 family example)

| % of Month Running | CPU Discount | Memory Discount |
|-------------------|-------------|----------------|
| 0–25% | 0% | 0% |
| 25–50% | ~20% | ~20% |
| 50–75% | ~40% | ~40% |
| 75–100% | ~60% | ~60% |

- A VM running 100% of the month gets approximately **30% effective discount**
- SUDs are calculated across all VMs of the same type in the same region within a project — not per VM
- If you have 4 VMs each running 25% of the month, GCP infers 1 "full month" VM and applies the sliding discount

#### SUD Eligibility

- **Eligible**: N1, N2, N2D, C2, M1, M2 machine types; standard and custom types within these families
- **NOT eligible**:
  - E2 machine types (already priced lower)
  - T2D, T2A
  - Preemptible/Spot VMs (already discounted)
  - Sole-tenant nodes
  - App Engine Flexible (underlying GCE VMs don't receive SUD)

---

### Committed Use Discounts (CUDs)

A contractual commitment to use a specific amount of resources for 1 or 3 years in exchange for a discount.

#### Resource-Based CUDs

- Commit to **vCPU and memory** amounts in a specific **region** and **machine type family**
- 1-year: up to ~37% discount; 3-year: up to ~57% discount (varies by resource)
- Applied automatically to matching usage — GCP tracks consumption against commitments
- Unused commitment is **still billed** — you pay whether you use it or not
- Can purchase additional commitments at any time
- Commitments can be shared across projects within a billing account (via shared CUDs)

#### Spend-Based CUDs

- Commit to a minimum **hourly spend** amount
- Available for: Cloud SQL, Cloud Spanner, Google Cloud VMware Engine, Anthos
- Discount applies to usage up to the committed amount; usage above commitment is at on-demand pricing
- 1-year: ~25% discount; 3-year: ~52% discount (varies by service)

#### CUD vs SUD Interaction

- CUDs and SUDs do NOT stack on the same resource usage
- GCP applies whichever discount is greater
- If your CUD covers 100% of your usage, SUD provides no additional benefit
- If you have partial CUD coverage, SUDs apply to uncovered usage

---

### Preemptible and Spot VMs (Pricing Perspective)

- Up to ~90% discount vs standard on-demand pricing
- Preemptible: Max 24h runtime; Spot: No max runtime
- Best ROI for batch, HPC, fault-tolerant workloads
- Cannot be combined with CUDs or SUDs (already discounted)
- See [compute-engine-planning.md](compute-engine-planning.md) for selection details

---

### E2 Machine Types (Cost-Optimized)

- ~31% cheaper than N1 for equivalent configurations
- Use shared-core resources (physical CPU is shared with other VMs)
- No SUDs available, but the base price is already lower
- Best for: Dev/test, low-traffic web servers, CI/CD agents, small business workloads

---

### Rightsizing Recommendations

- **Google Cloud Recommender** analyzes VM CPU and memory utilization and suggests:
  - Downsizing overprovisioned VMs (e.g., n2-standard-8 running at 5% CPU → n2-standard-2)
  - Stopping idle VMs
  - Changing to cheaper machine types
- Recommendations appear in the Compute Engine > Recommendations section in Console
- Also available via the Recommender API for programmatic integration
- **Important**: Rightsizing recommendations are based on historical utilization (typically 8 days of data); they may not account for peak usage during special events

---

### Custom Machine Types for Cost Optimization

- If a workload needs, say, 6 vCPU + 20 GB RAM but the nearest predefined type is n2-standard-8 (32 GB), a custom type n2-custom-6-20480 avoids paying for unused memory
- Custom types are slightly more expensive per vCPU/GB than predefined types, but the optimization from right-sizing can outweigh this
- Use when: workload has an unusual CPU:memory ratio that doesn't align with any predefined type

---

### Storage Cost Optimization

- **Cloud Storage lifecycle policies**: Automatically transition objects to lower-cost storage classes (Nearline → Coldline → Archive) based on age
- **BigQuery**: Use partitioned and clustered tables to reduce bytes scanned (queries charged per TB scanned on-demand pricing)
- **BigQuery flat-rate pricing**: For high query volumes, flat-rate reservations provide predictable costs
- **Cloud SQL**: Right-size instance type; enable storage auto-increase but set reasonable limits; use read replicas for read-heavy workloads instead of scaling the primary

---

### Network Egress Costs

Network egress (data leaving GCP) is charged:
- **Free**: Ingress (incoming), same-zone traffic, traffic between GCP services in same region (via internal IPs)
- **Charged**:
  - Egress to internet: varies by destination (~$0.08–$0.12/GB to most destinations)
  - Egress between regions within GCP: ~$0.01–$0.08/GB depending on regions
  - Egress to on-premises via VPN/Interconnect: varies
  - Cloud Storage egress to internet (not within GCP)

Cost reduction strategies:
- Use internal IPs for GCP-to-GCP traffic
- Place interconnected services in the same region (even better: same zone)
- Use CDN (Cloud CDN) to cache content at edge, reducing origin egress

---

### Cloud Billing Analysis Tools

| Tool | Purpose |
|------|---------|
| **Billing Export to BigQuery** | Custom SQL analysis of costs by service, project, label, region |
| **Cost Table in Console** | Built-in breakdown by project and service |
| **Cost Trends** | Visual chart of spend over time |
| **Recommender** | Actionable cost-saving recommendations |
| **Budget Alerts** | Notify when approaching spend thresholds |
| **Labels** | Tag resources for cost attribution; group by team, environment, application |

#### Resource Labels for Cost Allocation

- Labels are key-value pairs on GCP resources (e.g., `env=prod`, `team=backend`)
- Labels appear in billing export → enables cost breakdown per team/environment
- Labels are **not** enforced automatically — must be applied manually or via IaC
- **Organization Policies** can require specific labels on resources: `constraints/compute.requireShieldedVm` etc. (not directly a label policy, but label enforcement requires custom tooling or org policy with conditions)

---

## When to Use

| Scenario | Optimization |
|----------|-------------|
| Long-running, always-on VMs (predictable baseline) | Committed Use Discounts (3-year for max savings) |
| VMs running most of the month without commitment | Sustained Use Discounts (automatic) |
| Batch, HPC, fault-tolerant workloads | Preemptible/Spot VMs |
| Dev/test workloads | E2 machines + Preemptible/Spot |
| Overprovisioned VMs | Rightsizing via Recommender |
| Cost tracking across teams | Labels + BigQuery billing export |
| Rarely accessed data | Cloud Storage Nearline/Coldline/Archive lifecycle policies |

---

## When NOT to Use

- **CUDs**: Don't commit to resources you're unsure about — you pay for the commitment whether used or not
- **Preemptible/Spot**: Never for production stateful services or latency-sensitive apps
- **Rightsizing**: Don't apply blindly — check peak usage periods before downsizing

---

## Related Services / Concepts

- **Billing**: Budget alerts and export — see [billing.md](../domain-1-setup-and-configure/billing.md)
- **Compute Engine Planning**: Machine type selection — see [compute-engine-planning.md](compute-engine-planning.md)
- **Storage Planning**: Storage class optimization — see [storage-planning.md](storage-planning.md)
- **Managing Compute**: Autoscaling instead of overprovisioning — see [managing-compute.md](../domain-4-ensure-success/managing-compute.md)

---

## Exam-Relevant Notes

### Common Traps

1. **SUDs are automatic**: No sign-up needed. The exam may ask what you need to "configure" to get SUDs — answer is nothing; they're automatic.

2. **E2 machines don't get SUDs**: E2 is already priced lower; no additional SUD applies. Don't recommend combining E2 with SUD as a strategy.

3. **CUD commitment is per-region**: If you commit 10 N2 vCPUs in us-central1, that doesn't cover N2 vCPUs in europe-west1. Plan commitments per-region.

4. **CUD + SUD don't stack**: They're mutually exclusive for the same usage. GCP applies the greater discount.

5. **Preemptible 24-hour limit**: Exam may ask about a job taking 30 hours — Preemptible VMs can't run that long without interruption and restart. Spot VMs don't have the 24-hour limit.

6. **Network egress between regions is charged**: A common exam question: which traffic is free? Same-zone internal IP traffic is free. Cross-region is charged.

7. **Rightsizing doesn't account for peak events**: Recommendations are based on historical data. Always review before applying.

### Decision Tree: Discount Selection

```
Is the workload fault-tolerant/batch?
├── Yes → Preemptible/Spot VMs (largest discount)
│
Is the workload running > 75% of the month predictably?
├── Yes → Is it for 1-3 years?
│          ├── Yes → Committed Use Discount
│          └── No → Sustained Use Discount (automatic)
│
Is the machine overprovisioned?
└── Check Recommender → Rightsize the VM
```

### Keywords
- Sustained use discount, committed use discount, resource-based CUD, spend-based CUD, Preemptible, Spot VM, E2 cost-optimized, rightsizing, Cloud Recommender, billing labels, network egress, lifecycle policy

---

## Source

- https://cloud.google.com/compute/docs/sustained-use-discounts
- https://cloud.google.com/compute/docs/instances/signing-thirdparty-software#committed-use
- https://cloud.google.com/billing/docs/how-to/budgets
- https://cloud.google.com/recommender/docs/recommenders/overview
