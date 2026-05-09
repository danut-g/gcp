# Managing GKE: Upgrades, Node Pool Management, Workload Identity

## Overview

Managing GKE clusters over time involves handling Kubernetes version upgrades, scaling node pools, monitoring workload health, and managing security posture through Workload Identity. These operational concerns are tested in the ACE exam.

---

## Key Concepts

### GKE Version Upgrades

#### Version Terminology

- **GKE version**: Kubernetes version + GKE-specific patches (e.g., `1.28.7-gke.1026000`)
- **Minor version**: e.g., 1.28 → 1.29 (significant changes, careful testing needed)
- **Patch version**: e.g., 1.28.7 → 1.28.9 (bug fixes, security patches)

#### Release Channels

| Channel | When Versions Become Available | Notes |
|---------|-------------------------------|-------|
| **Rapid** | 1–3 months after upstream K8s | Newest features; less battle-tested |
| **Regular** | ~2–3 months after Rapid | Recommended for most production |
| **Stable** | ~3–6 months after Regular | Maximum stability |
| **None (no channel)** | Manual control | Full control; you manage all upgrades |

- Clusters on a release channel receive **automatic minor and patch upgrades** within that channel
- GKE sends notifications before automatic upgrades
- Can pause upgrades with "maintenance exclusions" during critical periods

#### Upgrade Order: Control Plane First

- GKE always upgrades the **control plane** before node pools
- A control plane can be up to one minor version ahead of node pools
- This is intentional: allows you to test new version on control plane before committing to node upgrades

#### Control Plane Upgrade Behavior

- **Zonal cluster**: Brief outage (~15 minutes) during control plane upgrade (cannot schedule new pods)
- **Regional cluster**: No downtime — control plane replicas are upgraded one at a time

#### Node Pool Upgrades

- **Automatic**: Follows the release channel; GKE will auto-upgrade nodes within the maintenance window
- **Manual**: Use `gcloud container clusters upgrade` to trigger manually
- Node upgrade strategy (Standard mode):
  - **Surge upgrade** (default): Creates extra nodes (`max-surge`) before draining old ones; configurable
  - **Blue/Green upgrade** (for production zero-downtime): Creates a complete new pool, migrates workloads, deletes old pool
- Node pool upgrade drains each node before upgrading (honors PodDisruptionBudgets)

#### Upgrade Timeline

- GKE versions have defined end-of-support dates
- Clusters running an unsupported version must be upgraded; GKE may auto-upgrade them
- Plan ahead: minor version upgrades require testing application compatibility

---

### Node Pool Management

#### Adding Node Pools

- Add specialized pools: GPU pools for ML, spot pools for batch, ARM pools for cost savings
- Node pools can have different machine types, disk types, and taints/labels within the same cluster

#### Resizing Node Pools

- Manually set node count: `gcloud container clusters resize`
- **Note**: Resizing DOWN will drain and delete nodes; running pods will be evicted (respecting PDBs)
- Autoscaling handles this automatically when configured

#### Upgrading Node Pool OS Images

- GKE offers different node images:
  - **Container-Optimized OS (COS)** with containerd: Default; Google-maintained, minimal, secure
  - **Ubuntu** with containerd: More general-purpose OS; more packages available
  - **Windows Server**: For Windows containers
- COS is recommended for most workloads; Ubuntu if you need specific OS features

#### Node Auto-Provisioning (NAP)

- Standard mode feature: Automatically creates and deletes node pools based on pending Pod requirements
- More flexible than static node pools — provisions the exact right machine type for each workload
- Configure min/max CPU/memory limits for NAP

---

### Workload Identity

Workload Identity is the recommended way to give GKE Pods access to GCP services (Cloud Storage, Pub/Sub, Cloud SQL, etc.) without service account key files.

#### How It Works

1. Kubernetes ServiceAccount (KSA) is linked to a GCP ServiceAccount (GSA)
2. Pods annotated with the KSA automatically receive GSA credentials via the GKE metadata server
3. GCP APIs see requests as coming from the GSA

#### Setup Steps

1. Enable Workload Identity on the cluster:
   ```
   --workload-pool=PROJECT_ID.svc.id.goog
   ```

2. Create a Google Service Account (GSA):
   ```
   gcloud iam service-accounts create my-gsa
   ```

3. Grant the GSA the necessary IAM permissions (e.g., `roles/storage.objectViewer`)

4. Create/annotate the Kubernetes Service Account:
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: my-ksa
     namespace: my-namespace
     annotations:
       iam.gke.io/gcp-service-account: my-gsa@PROJECT_ID.iam.gserviceaccount.com
   ```

5. Grant the KSA permission to impersonate the GSA:
   ```
   gcloud iam service-accounts add-iam-policy-binding \
     my-gsa@PROJECT_ID.iam.gserviceaccount.com \
     --role=roles/iam.workloadIdentityUser \
     --member="serviceAccount:PROJECT_ID.svc.id.goog[my-namespace/my-ksa]"
   ```

6. Use the KSA in Pod spec:
   ```yaml
   spec:
     serviceAccountName: my-ksa
   ```

#### Workload Identity Federation (External)

- Extends Workload Identity to workloads outside GCP (GitHub Actions, AWS, Azure, on-premises)
- Allows external identity providers to exchange tokens for GCP credentials
- No long-lived service account keys needed for external CI/CD systems

---

### GKE Security Hardening

#### Shielded GKE Nodes

- Uses Shielded VMs for node pools
- Provides: Secure Boot, vTPM, Integrity Monitoring
- Protects against rootkits and bootkit attacks

#### Binary Authorization

- Policy enforcement for container images
- Allows only cryptographically verified images to be deployed to GKE
- Integrate with Cloud Build to attest images after security scans

#### Workload Security Policies

- **Pod Security Standards** (PSS): Kubernetes native policy framework (replaces deprecated PodSecurityPolicy)
  - Levels: `privileged`, `baseline`, `restricted`
  - GKE Autopilot enforces `restricted` by default
- **GKE Sandbox**: gVisor-based sandboxed execution environment for untrusted workloads

---

### Cluster Operations

#### Cluster Upgrades - Maintenance Windows

- Define **maintenance windows**: Time ranges when GKE can perform automatic maintenance
- Define **maintenance exclusions**: Time ranges when GKE should NOT upgrade (critical deployments, freeze periods)
- Important for production clusters: Set maintenance windows to off-peak hours

#### Backup for GKE

- **Backup for GKE** service: Creates backups of GKE workloads (Kubernetes objects + persistent volume data)
- Scheduled backups with retention policies
- Restore individual namespaces, workloads, or entire clusters

#### Vertical Pod Autoscaler (VPA) - Operations

- VPA analyzes resource usage and recommends (or applies) new resource requests/limits
- Modes:
  - `Off`: Only generates recommendations; no action
  - `Initial`: Sets resources at Pod creation but doesn't update running Pods
  - `Auto`: Evicts and recreates Pods with new resource values (requires PDB for zero-downtime)
- Review VPA recommendations regularly to rightsize Pods

---

### GKE Networking Operations

#### Alias IP Range Management

- As a cluster grows, Pod IP exhaustion can occur
- Monitor: `kubectl describe node` shows `Allocatable` pods per node
- Expanding subnet or adding secondary ranges requires careful planning
- GKE auto-expands pod CIDR in some configurations

#### Private Cluster Considerations

- Nodes need Cloud NAT for outbound internet (e.g., pulling public container images)
- Ensure the master authorized networks are updated when admin IP addresses change
- Control plane upgrade may require updating authorized networks if you use a narrow IP range

---

## When to Use

- **Release channels**: For most clusters; removes manual version tracking burden
- **Workload Identity**: Always — instead of service account keys
- **Surge upgrades**: Standard for most node pool upgrades
- **Blue/Green upgrades**: When you cannot tolerate any pod disruption during node upgrades
- **Maintenance windows**: All production clusters — control when upgrades happen
- **VPA recommendations**: Regularly review for rightsizing cost optimization

---

## When NOT to Use

- **"No channel" (manual upgrades)**: Only for highly regulated environments where you must control exact versions; creates toil
- **Service account key files in Pods**: Always use Workload Identity instead
- **Large surge settings**: Large maxSurge creates many extra nodes that cost money; balance between upgrade speed and cost

---

## Related Services / Concepts

- **GKE Planning**: Autopilot vs Standard, cluster design — see [gke-planning.md](../domain-2-plan-and-configure/gke-planning.md)
- **GKE Deploy**: Initial cluster creation and workloads — see [gke-deploy.md](../domain-3-deploy-and-implement/gke-deploy.md)
- **Service Accounts**: Workload Identity setup — see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)
- **Monitoring**: GKE observability with Google Managed Prometheus — see [monitoring-cloud-ops.md](monitoring-cloud-ops.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Control plane upgrades first**: GKE always upgrades the control plane before node pools. You cannot upgrade nodes past the control plane version.

2. **Zonal cluster control plane outage during upgrade**: For zonal clusters, upgrading the control plane causes a brief period where the API server is unavailable. For production, use regional clusters.

3. **Workload Identity 5-step setup**: The most commonly forgotten step is adding the IAM binding on the GSA for the KSA's Workload Identity user role (`roles/iam.workloadIdentityUser`). Missing this breaks auth.

4. **Maintenance exclusions prevent all maintenance**: When you create a maintenance exclusion, GKE will not perform ANY automatic maintenance during that window. Don't set permanent exclusions.

5. **Pod autoscaler interplay**: HPA + Cluster Autoscaler work together. HPA adds more Pods; if no room, Cluster Autoscaler adds nodes. VPA changes resource requests but can conflict with HPA on CPU/memory.

6. **Binary Authorization attestation**: BA doesn't just check if images exist in Artifact Registry; it verifies cryptographic attestations from specific attestors (e.g., Cloud Build attestor).

7. **Node pool deletion drains first**: Deleting a node pool evicts Pods to other nodes (if available). If the cluster is fully packed, evicted Pods may fail. Plan capacity before reducing node pools.

### Keywords
- GKE release channel, control plane upgrade, node pool surge upgrade, blue/green upgrade, Workload Identity, KSA, GSA, workload-pool, maintenance window, maintenance exclusion, VPA, HPA, Cluster Autoscaler, Shielded GKE, Binary Authorization, Backup for GKE

---

## Source

- https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-upgrades
- https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity
- https://cloud.google.com/kubernetes-engine/docs/how-to/node-auto-provisioning
- https://cloud.google.com/kubernetes-engine/docs/concepts/maintenance-windows-and-exclusions
