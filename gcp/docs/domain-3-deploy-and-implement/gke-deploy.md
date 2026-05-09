# GKE Deployment: Clusters, Deployments, Services, Ingress, ConfigMaps, Secrets

## Overview

Deploying on Google Kubernetes Engine involves creating clusters and configuring Kubernetes workloads: Deployments, StatefulSets, Services, Ingress, ConfigMaps, and Secrets. Understanding how GKE-specific features (Workload Identity, Container-native load balancing, etc.) integrate with standard Kubernetes concepts is critical.

---

## Key Concepts

### Cluster Creation Decisions

Key parameters set at cluster creation time (many cannot be changed later):

| Parameter | Description | Notes |
|-----------|-------------|-------|
| **Mode** | Autopilot vs Standard | Cannot change after creation |
| **Location** | Zonal (single zone) vs Regional | Cannot change after creation |
| **Network/Subnet** | VPC and subnet | Cannot change after creation |
| **Networking mode** | VPC-native (Alias IP) vs Routes-based | VPC-native is default and recommended |
| **Release channel** | Rapid/Regular/Stable/None | Can change after creation |
| **Private cluster** | Nodes without external IPs | Cannot easily change after creation |
| **Node version** | Kubernetes version | Can upgrade later |

#### Post-Creation Changes

- Can add node pools, modify node pool sizes, change autoscaling settings
- Cannot change region/zone, network, VPC-native setting
- Cluster deletion is permanent

---

### Node Pools

- Collections of nodes with identical configuration
- Commands operate at cluster level but target specific node pools
- Use `--node-labels` and `--node-taints` at node pool creation time for workload scheduling

---

### Kubernetes Workload Objects

#### Deployments

- Manages a desired state for stateless Pods
- Provides: rolling updates, rollback, scaling
- Controller ensures the specified number of Pod replicas are running
- `spec.strategy`:
  - `RollingUpdate` (default): Gradually replaces old Pods; use `maxSurge` and `maxUnavailable`
  - `Recreate`: Kill all old Pods first, then create new ones; causes downtime but avoids two versions running simultaneously

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

#### StatefulSets

- For stateful applications (databases, message brokers)
- Pods have stable network identities (predictable hostnames: `pod-0`, `pod-1`)
- Pods have persistent volume claims tied to individual Pods (not shared)
- Ordered startup and shutdown
- Each Pod in a StatefulSet gets its own PVC (not shared)

#### DaemonSets

- Ensures exactly one Pod runs on each (or selected) node
- Use cases: Log collectors, monitoring agents, network plugins
- In Autopilot mode: Custom DaemonSets are restricted (only Google-managed ones run)

#### Jobs and CronJobs

- **Job**: Run one or more Pods to completion
- **CronJob**: Run Jobs on a schedule (cron syntax)
- On GKE, prefer **Cloud Scheduler + Cloud Run Jobs** for simpler use cases

---

### Kubernetes Services

Services expose Pods to network traffic. Four types:

| Type | Description | Connectivity |
|------|-------------|-------------|
| **ClusterIP** | Internal cluster IP only | Pod-to-Pod within cluster |
| **NodePort** | Exposes on each node's IP at a static port | External via node IPs (not production) |
| **LoadBalancer** | Provisions a GCP external load balancer | External access; creates Network LB by default |
| **ExternalName** | DNS CNAME alias to an external name | DNS-level forwarding |

#### GKE-Specific: Container-Native Load Balancing

- GKE integrates with GCP **HTTP(S) Load Balancer** via the `container.googleapis.com/load-balancer-type: "External"` annotation
- **Container-native load balancing**: Load balancer sends traffic directly to Pod IPs (using NEGs — Network Endpoint Groups) rather than routing through node IP + kube-proxy
- Benefits: More efficient routing, better health checking at Pod level
- Required for: Advanced traffic management, HTTPS with custom certs via Ingress

---

### Ingress

- Kubernetes Ingress manages external HTTP/HTTPS access to Services
- In GKE, creating an Ingress resource provisions a **GCP HTTP(S) Load Balancer**
- Annotations control GCP-specific behavior (e.g., SSL certificate, health check path)

#### Ingress Classes in GKE

| Class | Backend | Use Case |
|-------|---------|---------|
| `gce` (default) | GCP HTTP(S) External LB | External traffic from internet |
| `gce-internal` | GCP HTTP(S) Internal LB | Internal traffic (within VPC) |
| `gce-xlb` | Cross-region LB | Global multi-region |

#### GKE Gateway API (newer)

- Replacement for Ingress, more expressive
- Supports traffic splitting, header-based routing, multiple backends
- `Gateway` and `HTTPRoute` resources
- Backed by GCP Load Balancers

---

### ConfigMaps

- Store non-sensitive configuration data as key-value pairs
- Injected into Pods as:
  - **Environment variables**
  - **Mounted files** (ConfigMap → Volume → file in container)
- Updates to ConfigMaps take ~1 minute to propagate to mounted volumes; env var injection requires Pod restart
- **Not encrypted at rest** by default
- Max size: 1 MB

---

### Secrets

- Store sensitive data (passwords, tokens, keys)
- Base64 encoded (not encrypted) by default
- Kubernetes Secrets are stored in etcd; by default not encrypted at rest in etcd unless configured
- In GKE, enable **Application-layer Secrets Encryption** to encrypt Secrets in etcd using a Cloud KMS key
- Injection methods: Environment variables, mounted volumes
- Best practice: Use **Secret Manager** (GCP native) via Workload Identity for production secrets — avoid Kubernetes Secrets for sensitive long-lived credentials

#### Secret Manager Integration

- Pods with Workload Identity can access Secret Manager secrets directly
- No secrets stored in etcd at all — fetched at runtime
- See [data-security.md](../domain-5-configure-access-and-security/data-security.md) and [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)

---

### Workload Identity in GKE

- Maps a Kubernetes Service Account (KSA) to a GCP Service Account (GSA)
- Steps:
  1. Enable Workload Identity on the cluster
  2. Create a GCP Service Account (GSA)
  3. Annotate the Kubernetes Service Account with GSA email
  4. Grant `roles/iam.workloadIdentityUser` to `serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]` on the GSA
- Pods using the KSA automatically receive GSA credentials via the metadata server
- No key files needed

---

### Persistent Volumes in GKE

- **PersistentVolume (PV)**: Cluster-level storage resource
- **PersistentVolumeClaim (PVC)**: Pod's request for storage
- **StorageClass**: Defines the type of storage to provision
- GKE default StorageClasses:
  - `standard`: `pd-standard` (HDD)
  - `standard-rwo`: `pd-balanced` (SSD) — ReadWriteOnce
  - `premium-rwo`: `pd-ssd` — ReadWriteOnce
- ReadWriteMany: Not supported by GCE persistent disks; use Filestore (NFS) or GCS FUSE for shared access

---

### Horizontal Pod Autoscaler (HPA)

- Automatically scales the number of Pods based on CPU/memory or custom metrics
- Works with Deployments and StatefulSets
- Requires resource requests defined on containers
- In Autopilot, HPA is always available

### Vertical Pod Autoscaler (VPA)

- Automatically adjusts resource requests/limits for containers
- **Cannot be used simultaneously with HPA on CPU/memory** (conflict)
- VPA in "auto" mode can restart Pods to apply new resource values

---

## When to Use

- **Deployments**: Always for stateless workloads
- **StatefulSets**: Databases, message brokers, any workload needing stable identity or per-Pod storage
- **Ingress with GCE class**: Production HTTPS endpoints with custom routing
- **ConfigMaps**: Application configuration that changes between environments (dev vs prod)
- **Workload Identity**: Always — instead of SA key files in Pods
- **Container-native load balancing**: When you need Pod-level health checking and direct traffic routing

---

## When NOT to Use

- **LoadBalancer Service for every microservice**: Creates too many external IPs; use Ingress to consolidate
- **Kubernetes Secrets for sensitive credentials**: Store in Secret Manager instead
- **DaemonSets in Autopilot**: Not supported for custom workloads
- **StatefulSets without persistent volume planning**: Ensure storage class and access modes are compatible

---

## Related Services / Concepts

- **GKE Planning**: Autopilot vs Standard, cluster topology — see [gke-planning.md](../domain-2-plan-and-configure/gke-planning.md)
- **Managing GKE**: Upgrades, workload identity management — see [managing-gke.md](../domain-4-ensure-success/managing-gke.md)
- **Networking Deploy**: Load balancer types — see [networking-deploy.md](networking-deploy.md)
- **Data Security**: KMS encryption of Secrets, Secret Manager — see [data-security.md](../domain-5-configure-access-and-security/data-security.md)

---

## Exam-Relevant Notes

### Common Traps

1. **LoadBalancer Service = Network Load Balancer**: Creating a Service of type `LoadBalancer` in GKE provisions a **TCP/UDP (Layer 4) Network Load Balancer** by default, NOT an HTTP(S) Load Balancer. Use Ingress for HTTP(S) LB.

2. **Ingress = HTTP(S) Load Balancer**: Creating a GKE Ingress provisions a Google Cloud HTTP(S) Load Balancer with URL maps, backend services, and SSL.

3. **ConfigMap updates don't restart Pods**: When you update a ConfigMap, Pods using it via environment variables are NOT updated until they restart. Volume-mounted ConfigMaps update within ~1 minute.

4. **Kubernetes Secrets are base64, not encrypted**: Base64 is encoding, not encryption. Without Application-layer Secrets Encryption, Secrets in etcd are readable.

5. **StatefulSet PVCs are not shared**: Each StatefulSet Pod gets its own PVC. Unlike Deployments where all Pods might share a ConfigMap, StatefulSet Pods each have individual persistent storage.

6. **Workload Identity requires 4 steps**: Missing any step (especially the IAM binding with the Kubernetes namespace/SA projection) is a common source of auth failures.

7. **Regional cluster node count math**: A regional cluster in 3 zones with `num-nodes=3` per zone = 9 total nodes. Know this for capacity planning questions.

8. **VPA and HPA conflict**: Cannot use VPA in "auto" mode and HPA both scaling on CPU/memory simultaneously. Use VPA in "recommendation" mode to get sizing advice without auto-apply.

### Keywords
- Deployment, StatefulSet, DaemonSet, Service types, LoadBalancer vs Ingress, ClusterIP, container-native load balancing, NEG, ConfigMap, Kubernetes Secret, Workload Identity, HPA, VPA, StorageClass, PersistentVolumeClaim

---

## Source

- https://cloud.google.com/kubernetes-engine/docs/concepts/deployment
- https://cloud.google.com/kubernetes-engine/docs/concepts/ingress
- https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity
- https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/gce-pd-csi-driver
