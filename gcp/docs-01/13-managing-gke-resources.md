# Section 4.2 — Managing Google Kubernetes Engine Resources

## Exam Relevance
This topic is part of **Section 4: Ensuring successful operation of a cloud solution (~20 % of the exam)**. You must know how to view cluster inventory, configure access to Artifact Registry, work with node pools, manage Kubernetes resources, and configure autoscaling.

---

## 1. Viewing Current Running Cluster Inventory

> 📖 **Docs:** [List clusters](https://cloud.google.com/kubernetes-engine/docs/how-to/listing-clusters) | [Cluster details](https://cloud.google.com/sdk/gcloud/reference/container/clusters/describe) | 🖥️ **Console:** Kubernetes Engine → Clusters

### Cluster-Level Commands

```bash
# List all GKE clusters in the project
gcloud container clusters list

# Describe a specific cluster
gcloud container clusters describe my-cluster --zone=us-central1-a

# Get cluster credentials (connect kubectl)
gcloud container clusters get-credentials my-cluster --zone=us-central1-a
```

### Node Inventory

```bash
# List all nodes in the cluster
kubectl get nodes

# List nodes with detailed info (IPs, status, roles, version)
kubectl get nodes -o wide

# Describe a specific node (capacity, conditions, pods running on it)
kubectl describe node NODE_NAME

# List nodes with labels
kubectl get nodes --show-labels

# List nodes with specific label
kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool

# Get node resource usage
kubectl top nodes
```

### Pod Inventory

```bash
# List all pods in the default namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# List pods with detailed info
kubectl get pods -o wide

# List pods in a specific namespace
kubectl get pods -n my-namespace

# Describe a pod (events, conditions, containers)
kubectl describe pod POD_NAME

# Get pod logs
kubectl logs POD_NAME
kubectl logs POD_NAME -c CONTAINER_NAME   # Multi-container pod
kubectl logs POD_NAME --previous           # Previous crashed container
kubectl logs POD_NAME -f                   # Stream logs

# Get pod resource usage
kubectl top pods

# List pods with labels
kubectl get pods --show-labels
kubectl get pods -l app=my-app
```

### Service Inventory

```bash
# List all services
kubectl get services
kubectl get svc

# List services in all namespaces
kubectl get svc -A

# Describe a service
kubectl describe svc my-service

# Get service endpoints
kubectl get endpoints my-service
```

---

## 2. Configuring GKE to Access Artifact Registry

> 📖 **Docs:** [Artifact Registry overview](https://cloud.google.com/artifact-registry/docs/overview) | [Authenticate to Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/authentication) | 🖥️ **Console:** Artifact Registry → Repositories → Create repository

### What Is Artifact Registry?
- Google's managed **container image and package repository**
- Successor to Container Registry (gcr.io)
- Supports Docker, Maven, npm, Python, Go, and more

### Creating an Artifact Registry Repository

```bash
# Create a Docker repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="My Docker images"

# List repositories
gcloud artifacts repositories list

# Configure Docker authentication to Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev
```

### Image URL Format
```
LOCATION-docker.pkg.dev/PROJECT_ID/REPOSITORY/IMAGE:TAG

Example:
us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1
```

### GKE Access to Artifact Registry

#### Method 1: Default (Node Service Account)
GKE nodes use their service account to pull images. The service account needs the **Artifact Registry Reader** role:

```bash
# Grant Artifact Registry Reader to the node service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:NODE_SA@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/artifactregistry.reader"
```

#### Method 2: Workload Identity (Recommended)
Map a Kubernetes service account to a Google service account with Artifact Registry access:

```bash
# Grant Artifact Registry Reader to the GCP service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:my-gcp-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/artifactregistry.reader"

# Set up Workload Identity binding
gcloud iam service-accounts add-iam-policy-binding \
  my-gcp-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"
```

#### Method 3: ImagePullSecrets
For cross-project or external registries:

```bash
# Create a secret for pulling images
kubectl create secret docker-registry my-registry-key \
  --docker-server=us-central1-docker.pkg.dev \
  --docker-username=_json_key \
  --docker-password="$(cat key.json)" \
  --docker-email=sa@project.iam.gserviceaccount.com
```

```yaml
# Reference in pod spec
spec:
  imagePullSecrets:
    - name: my-registry-key
  containers:
    - name: my-app
      image: us-central1-docker.pkg.dev/PROJECT/REPO/IMAGE:TAG
```

### Pushing Images to Artifact Registry

```bash
# Build and push with Docker
docker build -t us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-app:v1 .
docker push us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-app:v1

# Build and push with Cloud Build
gcloud builds submit --tag us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-app:v1
```

---

## 3. Working with Node Pools

> 📖 **Docs:** [Node pools](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools) | [Add/resize/delete node pools](https://cloud.google.com/kubernetes-engine/docs/how-to/node-pools) | 🖥️ **Console:** Kubernetes Engine → Clusters → select cluster → Nodes tab

### What Are Node Pools?
- A **group of nodes within a cluster** that share the same configuration
- A cluster can have **multiple node pools** with different configurations
- Each node pool specifies: machine type, disk size, number of nodes, labels, taints

### Adding a Node Pool

```bash
# Add a new node pool
gcloud container node-pools create my-new-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=n2-standard-8 \
  --num-nodes=3 \
  --disk-size=100GB \
  --disk-type=pd-ssd

# Add a node pool with autoscaling
gcloud container node-pools create autoscale-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10

# Add a node pool with Spot VMs
gcloud container node-pools create spot-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --spot \
  --num-nodes=5

# Add a node pool with GPUs
gcloud container node-pools create gpu-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --num-nodes=2

# Add a node pool with labels and taints
gcloud container node-pools create labeled-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=e2-standard-4 \
  --num-nodes=3 \
  --node-labels=workload=batch,tier=backend \
  --node-taints=dedicated=batch:NoSchedule
```

### Editing a Node Pool

```bash
# Enable autoscaling on an existing pool
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10

# Resize a node pool (manual scaling)
gcloud container clusters resize my-cluster \
  --node-pool=my-pool \
  --num-nodes=5 \
  --zone=us-central1-a

# Enable auto-repair
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --enable-autorepair

# Enable auto-upgrade
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --enable-autoupgrade

# Update node pool labels
gcloud container node-pools update my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --node-labels=environment=production
```

### Removing a Node Pool

```bash
# Delete a node pool
gcloud container node-pools delete my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a

# List node pools
gcloud container node-pools list \
  --cluster=my-cluster \
  --zone=us-central1-a

# Describe a node pool
gcloud container node-pools describe my-pool \
  --cluster=my-cluster \
  --zone=us-central1-a
```

---

## 4. Working with Kubernetes Resources

> 📖 **Docs:** [Kubernetes objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/) | [kubectl reference](https://kubernetes.io/docs/reference/kubectl/) | 🖥️ **Console:** Kubernetes Engine → Workloads / Services & Ingress

### Pods

```bash
# Create a pod from a YAML file
kubectl apply -f pod.yaml

# Run a one-off pod
kubectl run my-pod --image=nginx --restart=Never

# Run a pod and open a shell
kubectl run debug --image=busybox -it --rm -- /bin/sh

# Delete a pod
kubectl delete pod my-pod

# Force delete a stuck pod
kubectl delete pod my-pod --grace-period=0 --force

# Copy files to/from a pod
kubectl cp local-file.txt POD_NAME:/path/in/pod
kubectl cp POD_NAME:/path/in/pod/file.txt ./local-file.txt
```

### Services

```yaml
# ClusterIP Service (internal only)
apiVersion: v1
kind: Service
metadata:
  name: my-internal-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080

---
# LoadBalancer Service (external)
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080

---
# NodePort Service
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

### StatefulSets

StatefulSets manage **stateful applications** (databases, message queues) with:
- **Stable network identities** (pod-0, pod-1, pod-2)
- **Stable persistent storage** (each pod gets its own PVC)
- **Ordered deployment and scaling** (pods created in order: 0, 1, 2)
- **Ordered rolling updates**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

```bash
# List StatefulSets
kubectl get statefulsets
kubectl get sts

# Scale a StatefulSet
kubectl scale statefulset postgres --replicas=5

# Delete a StatefulSet (pods are deleted in reverse order)
kubectl delete statefulset postgres
```

### Deployments vs. StatefulSets

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod identity | Random names | Ordered (pod-0, pod-1) |
| Storage | Shared PVC (if any) | Individual PVCs per pod |
| Scaling | Parallel | Ordered |
| Updates | Rolling (parallel) | Rolling (ordered) |
| Use case | Stateless apps | Stateful apps (databases) |

---

## 5. Managing Autoscaling Configurations

> 📖 **Docs:** [Scale workloads](https://cloud.google.com/kubernetes-engine/docs/how-to/scaling-apps) | [Cluster autoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler) | 🖥️ **Console:** Kubernetes Engine → Workloads → select deployment → Actions → Autoscale

### Horizontal Pod Autoscaler (HPA)

Scales the **number of pod replicas** based on metrics:

```bash
# Create HPA based on CPU
kubectl autoscale deployment my-app \
  --min=2 \
  --max=10 \
  --cpu-percent=70

# Create HPA from YAML
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

```bash
# View HPA status
kubectl get hpa
kubectl describe hpa my-app-hpa

# Delete HPA
kubectl delete hpa my-app-hpa
```

### Vertical Pod Autoscaler (VPA)

Adjusts **CPU and memory requests** for pods:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"  # Auto, Recreate, Initial, Off
  resourcePolicy:
    containerPolicies:
    - containerName: my-app
      minAllowed:
        cpu: "100m"
        memory: "128Mi"
      maxAllowed:
        cpu: "4"
        memory: "8Gi"
```

### VPA Update Modes

| Mode | Description |
|------|-------------|
| **Off** | Only provides recommendations (no action) |
| **Initial** | Sets resources only at pod creation |
| **Recreate** | Evicts and recreates pods with new resources |
| **Auto** | Similar to Recreate (applies in-place when available) |

### Cluster Autoscaler

Scales the **number of nodes** in a node pool:

```bash
# Enable cluster autoscaler
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10 \
  --node-pool=default-pool

# Disable autoscaler
gcloud container clusters update my-cluster \
  --zone=us-central1-a \
  --no-enable-autoscaling \
  --node-pool=default-pool
```

### HPA vs. VPA vs. Cluster Autoscaler

| Autoscaler | What It Scales | Based On |
|------------|---------------|----------|
| **HPA** | Number of pod replicas | CPU, memory, custom metrics |
| **VPA** | CPU/memory requests per pod | Historical resource usage |
| **Cluster Autoscaler** | Number of nodes | Pending pods (unschedulable) |

### How They Work Together

```
Traffic increase
  → HPA creates more pods
    → Pods become unschedulable (not enough node capacity)
      → Cluster Autoscaler adds more nodes
        → Pods get scheduled on new nodes

Traffic decrease
  → HPA reduces pod count
    → Nodes become underutilized
      → Cluster Autoscaler removes nodes (after cool-down)
```

---

## 8. Cluster Upgrades

> 📖 **Docs:** [Upgrade a cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster) | [Maintenance windows](https://cloud.google.com/kubernetes-engine/docs/how-to/maintenance-windows-and-exclusions) | 🖥️ **Console:** Kubernetes Engine → Clusters → select cluster → Upgrade available

- GKE upgrades: control plane first, then node pools
- Release channels: Rapid, Regular, Stable, Extended — GKE auto-upgrades within the channel
- Manual upgrade:
  ```bash
  # Upgrade control plane to specific version
  gcloud container clusters upgrade MY_CLUSTER --master --cluster-version=1.29.0 --zone=ZONE
  # Upgrade a node pool
  gcloud container clusters upgrade MY_CLUSTER --node-pool=MY_POOL --cluster-version=1.29.0 --zone=ZONE
  ```
- Surge upgrade settings (control disruption):
  ```bash
  gcloud container node-pools update MY_POOL --cluster=MY_CLUSTER --zone=ZONE \
    --max-surge-upgrade=1 --max-unavailable-upgrade=0
  ```
- Maintenance windows:
  ```bash
  gcloud container clusters update MY_CLUSTER --zone=ZONE \
    --maintenance-window-start="2024-01-01T09:00:00Z" \
    --maintenance-window-end="2024-01-01T17:00:00Z" \
    --maintenance-window-recurrence="FREQ=WEEKLY;BYDAY=SA,SU"
  ```
- Exam tip: Control plane and node pool versions can differ by at most 2 minor versions; upgrade control plane before nodes

---

## 9. ConfigMaps and Secrets

> 📖 **Docs:** [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) | [Kubernetes Secrets](https://cloud.google.com/kubernetes-engine/docs/concepts/secret) | 🖥️ **Console:** Kubernetes Engine → Configuration → ConfigMaps / Secrets

ConfigMaps:
```bash
# Create from literal values
kubectl create configmap my-config --from-literal=DB_HOST=localhost --from-literal=DB_PORT=5432
# Create from file
kubectl create configmap my-config --from-file=config.properties
# View
kubectl get configmap my-config -o yaml
```

Use in a Pod:
```yaml
spec:
  containers:
  - name: app
    envFrom:
    - configMapRef:
        name: my-config
    # OR as a volume
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: my-config
```

Secrets:
```bash
kubectl create secret generic my-secret --from-literal=password=s3cr3t
kubectl create secret docker-registry gcr-secret \
  --docker-server=us-docker.pkg.dev \
  --docker-username=_json_key \
  --docker-password="$(cat key.json)"
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode
```

Use Secret Manager instead of K8s Secrets for sensitive data — integrate via the Secret Manager add-on or mount with Workload Identity.

---

## 10. Network Policies

> 📖 **Docs:** [Network policies](https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy) | [Kubernetes NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/) | 🖥️ **Console:** Kubernetes Engine → Clusters → Create → Networking → Enable network policy

- Controls pod-to-pod and pod-to-external traffic (requires `--enable-network-policy` on cluster)
- Default: all pods can communicate with all pods
- NetworkPolicy resources are namespaced

Example — allow only pods with label app=frontend to reach app=backend on port 8080:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```
- Exam tip: An empty `podSelector` matches ALL pods in the namespace; an empty `ingress` list means DENY ALL ingress

---

## 11. Deployments

> 📖 **Docs:** [Deployments](https://cloud.google.com/kubernetes-engine/docs/how-to/stateless-apps) | [Rolling updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) | 🖥️ **Console:** Kubernetes Engine → Workloads → Deployments

Deployments manage stateless pod replicas with rolling updates and rollbacks.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: us-central1-docker.pkg.dev/PROJECT/REPO/my-app:v1
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

```bash
# Apply a deployment
kubectl apply -f deployment.yaml

# Scale a deployment
kubectl scale deployment my-app --replicas=5

# Update the image (rolling update)
kubectl set image deployment/my-app my-app=us-central1-docker.pkg.dev/PROJECT/REPO/my-app:v2

# Check rollout status
kubectl rollout status deployment/my-app

# View rollout history
kubectl rollout history deployment/my-app

# Roll back to previous version
kubectl rollout undo deployment/my-app

# Roll back to specific revision
kubectl rollout undo deployment/my-app --to-revision=2

# Pause/resume a rollout
kubectl rollout pause deployment/my-app
kubectl rollout resume deployment/my-app
```

---

## 12. DaemonSets

> 📖 **Docs:** [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) | [Use DaemonSets on GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/daemonset) | 🖥️ **Console:** Kubernetes Engine → Workloads → filter by DaemonSet

DaemonSets ensure a copy of a pod runs on **every node** (or a subset of nodes) in the cluster.

- Typical use cases: log collectors (fluentd), node monitoring agents, CNI plugins, storage daemons
- When a new node is added, the DaemonSet controller automatically schedules a pod on it
- Can be limited to specific nodes using `nodeSelector` or node affinity

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: log-collector
  template:
    metadata:
      labels:
        name: log-collector
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - name: log-collector
        image: fluentd:v1.16
```

```bash
# List DaemonSets
kubectl get daemonsets -A
kubectl get ds

# Describe a DaemonSet
kubectl describe ds log-collector -n kube-system
```

---

## 13. Ingress

> 📖 **Docs:** [GKE Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress) | [Configure Ingress for external LB](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress) | 🖥️ **Console:** Kubernetes Engine → Services & Ingress → Ingress tab

An Ingress exposes HTTP/HTTPS routes from outside the cluster to Services inside, based on hostnames and paths. On GKE, the default Ingress controller provisions a **Google Cloud HTTP(S) Load Balancer**.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

```bash
# List ingresses
kubectl get ingress

# Describe an ingress (shows load balancer IP, backend health)
kubectl describe ingress my-ingress
```

- **Exam tip**: Use `kubernetes.io/ingress.class: "gce"` for external HTTP(S) LB; use `"gce-internal"` for internal LB. A `BackendConfig` resource can attach Cloud Armor policies and IAP to the backend service.

---

## 14. Persistent Volumes and Storage Classes

> 📖 **Docs:** [Persistent volumes](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes) | [StorageClasses on GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/gce-pd-csi-driver) | 🖥️ **Console:** Kubernetes Engine → Storage → PersistentVolumeClaims / StorageClasses

- A **PersistentVolume (PV)** is a cluster storage resource (e.g., a GCE persistent disk)
- A **PersistentVolumeClaim (PVC)** is a request for storage by a pod; binds to a matching PV
- A **StorageClass** defines the type of storage (e.g., `pd-standard`, `pd-ssd`) and enables dynamic provisioning

```yaml
# StorageClass example
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
---
# PersistentVolumeClaim example
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: fast
  resources:
    requests:
      storage: 20Gi
```

| Access Mode | Description |
|------------|-------------|
| **ReadWriteOnce (RWO)** | Volume mounted read-write by a single node |
| **ReadOnlyMany (ROX)** | Volume mounted read-only by many nodes |
| **ReadWriteMany (RWX)** | Volume mounted read-write by many nodes (requires Filestore on GKE) |

---

## 15. Node Maintenance: Cordon and Drain

> 📖 **Docs:** [Safely drain a node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/) | [GKE node maintenance](https://cloud.google.com/kubernetes-engine/docs/how-to/maintenance-windows-and-exclusions) | 🖥️ **Console:** Cloud Shell → kubectl cordon / kubectl drain

- **Cordon** — Mark a node unschedulable (prevent new pods from being placed) but leave existing pods running
- **Drain** — Cordon and also evict existing pods so the node can be safely removed or upgraded

```bash
# Cordon a node (no new pods scheduled here)
kubectl cordon NODE_NAME

# Drain a node (evict pods to other nodes)
kubectl drain NODE_NAME --ignore-daemonsets --delete-emptydir-data

# Uncordon (re-enable scheduling)
kubectl uncordon NODE_NAME
```

---

## 16. GKE Cluster Modes: Autopilot vs Standard

> 📖 **Docs:** [Autopilot overview](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview) | [Choose between Autopilot and Standard](https://cloud.google.com/kubernetes-engine/docs/concepts/choose-cluster-mode) | 🖥️ **Console:** Kubernetes Engine → Clusters → Create → Select mode

| Feature | Autopilot | Standard |
|---------|-----------|----------|
| **Node management** | Fully managed by Google | User-managed node pools |
| **Billing** | Per-pod (CPU/memory/storage) | Per-node (VM cost) |
| **Configuration** | Preconfigured, less flexible | Full control |
| **Use case** | Hands-off production workloads | Advanced/custom workloads |

```bash
# Create an Autopilot cluster
gcloud container clusters create-auto my-auto-cluster --region=us-central1

# Create a Standard regional cluster
gcloud container clusters create my-cluster \
  --region=us-central1 \
  --num-nodes=2 \
  --machine-type=e2-standard-4
```

- **Exam tip**: Autopilot automatically applies GKE best practices and restricts certain configurations (no privileged containers, specific toleration rules). Use Standard when you need GPU workloads on specific machine types, custom node configurations, or DaemonSets with special requirements.

### Managing GKE Autopilot Pod Resource Requests

In Autopilot mode, resource requests are critical because **billing is per-pod** based on CPU, memory, and ephemeral storage requests. Autopilot also enforces minimum and maximum resource limits.

#### How Autopilot Handles Resource Requests

- If a pod has **no resource requests**, Autopilot applies **default requests** (0.5 vCPU, 2 GiB memory)
- Autopilot **rounds up** resource requests to the nearest supported increment
- Autopilot enforces **minimum requests** (0.25 vCPU, 0.5 GiB memory per container)
- Autopilot enforces **maximum requests** (196 vCPU, 1392 GiB memory per pod)

#### Best Practices for Autopilot Resource Requests

```yaml
# Always set resource requests AND limits in Autopilot
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    resources:
      requests:
        cpu: "250m"        # 0.25 vCPU
        memory: "512Mi"    # 512 MiB
      limits:
        cpu: "500m"        # 0.5 vCPU (optional — Autopilot may adjust)
        memory: "1Gi"      # 1 GiB
```

```bash
# Check resource requests for all pods in a namespace
kubectl get pods -n my-namespace -o custom-columns="NAME:.metadata.name,CPU:.spec.containers[0].resources.requests.cpu,MEM:.spec.containers[0].resources.requests.memory"

# View Autopilot Pod resource recommendations
kubectl describe pod my-pod -n my-namespace
# Look for "Autopilot" annotations showing requested vs. billed resources
```

#### VPA with Autopilot
- Vertical Pod Autoscaler (VPA) works in Autopilot mode in **recommendation mode** only (`updateMode: "Off"`)
- Autopilot's built-in autoscaling handles node provisioning automatically
- Use VPA recommendations to right-size your resource requests, then update deployments manually

#### Key Exam Points
- In Autopilot, you're **billed for resource requests, not actual usage** — set accurate requests to minimize cost
- Autopilot rejects pods that request resources outside its supported ranges
- Set **both requests and limits** — if you only set limits, Autopilot uses limits as requests
- No need to manage node pools in Autopilot — focus on pod-level resource tuning

---

## Exam Practice Questions

1. **A pod is stuck in "Pending" state. kubectl describe shows "Insufficient cpu". What's happening and how do you fix it?**
   - Answer: The cluster doesn't have enough CPU capacity. Either **resize the node pool** manually or **enable the cluster autoscaler** to automatically add nodes.

2. **You need to deploy a PostgreSQL database in GKE with persistent storage that survives pod restarts. Which resource should you use?**
   - Answer: Use a **StatefulSet** with `volumeClaimTemplates`. Each pod gets a dedicated PersistentVolumeClaim that persists across pod restarts.

3. **Your GKE pods need to pull images from an Artifact Registry in a different project. What's the recommended approach?**
   - Answer: Use **Workload Identity** to map the pod's Kubernetes service account to a Google Cloud service account that has `roles/artifactregistry.reader` in the target project.

4. **You want to automatically adjust pod CPU requests based on actual usage to right-size resources. Which autoscaler should you use?**
   - Answer: **Vertical Pod Autoscaler (VPA)** — It monitors actual resource usage and adjusts CPU/memory requests accordingly.

5. **Your GKE Autopilot pods are being rejected with "resource request exceeds maximum". What's happening and how do you fix it?**
   - Answer: Autopilot enforces maximum resource limits per pod. Reduce the pod's resource requests to be within Autopilot's supported limits (max ~196 vCPU, 1392 GiB memory per pod). Split large workloads across multiple pods.

6. **In a GKE Autopilot cluster, a developer forgot to set resource requests on their pods. What are the implications?**
   - Answer: Autopilot applies **default resource requests** (0.5 vCPU, 2 GiB memory per container). The developer will be billed for these defaults even if the actual usage is lower. Best practice: always set accurate resource requests to control costs.

5. **How can you add GPU-capable nodes to an existing GKE cluster without affecting current workloads?**
   - Answer: Create a **new node pool** with GPU-capable machine types and `--accelerator` flag. Use **node taints** to ensure only GPU workloads are scheduled there.

6. **You need to view all pods running across all namespaces, including system pods. What command do you use?**
   - Answer: `kubectl get pods --all-namespaces` or `kubectl get pods -A`.

---

## Glossary

**Artifact Registry** — GCP's managed repository service for storing and managing container images and language packages (Docker, Maven, npm, Python, Go); the successor to Container Registry (gcr.io).

**Artifact Registry Reader** — A GCP IAM role (`roles/artifactregistry.reader`) that grants read-only access to pull container images from an Artifact Registry repository.

**Access Mode** — A Kubernetes PersistentVolume property specifying how a volume may be mounted: ReadWriteOnce (RWO), ReadOnlyMany (ROX), or ReadWriteMany (RWX).

**Annotation** — A Kubernetes metadata field that attaches non-identifying, arbitrary key-value information to objects; used, for example, to select the Ingress class (`kubernetes.io/ingress.class`).

**Auto-repair** — A GKE node pool feature that automatically detects and replaces unhealthy nodes, restoring them to a healthy running state without manual intervention.

**Autopilot** — A fully managed GKE cluster mode where Google manages the nodes, enforces best practices, and bills per pod resource consumption rather than per VM.

**Auto-upgrade** — A GKE node pool feature that automatically upgrades nodes to newer Kubernetes versions within the configured release channel, reducing manual maintenance.

**Autoscaling** — The automatic adjustment of the number of pods or nodes in a GKE cluster in response to workload demand; implemented via HPA, VPA, or the Cluster Autoscaler.

**BackendConfig** — A GKE CRD that attaches Google Cloud features (Cloud Armor policies, IAP, connection draining, CDN) to the backend service created by a GKE Ingress.

**Base64** — A binary-to-text encoding scheme used to encode Kubernetes Secret data; Secret values are stored as base64-encoded strings and must be decoded when retrieved.

**Busybox** — A minimal Linux container image commonly used for debugging and troubleshooting in Kubernetes environments.

**Cluster** — A set of Kubernetes worker nodes and a control plane that together run containerized workloads; the top-level GKE resource.

**Cluster Autoscaler** — A GKE component that automatically adjusts the number of nodes in a node pool based on whether pods are pending (unschedulable) due to insufficient resources.

**ClusterIP** — A Kubernetes Service type that exposes the service on an internal cluster-only IP address; the default service type, accessible only within the cluster.

**CNI (Container Network Interface)** — The Kubernetes networking plugin standard that provides pod-level networking; GKE uses CNI plugins to assign IPs to pods and connect them to the VPC.

**Cloud Armor** — A GCP network security service providing DDoS protection and WAF rules; can be attached to a GKE Ingress backend via a BackendConfig.

**Cloud Build** — GCP's managed CI/CD service that builds container images from source code and pushes them to Artifact Registry.

**Cloud Load Balancer** — A GCP load balancer (HTTP(S), TCP, UDP) provisioned by GKE when a Service of type LoadBalancer or an Ingress is created.

**Cordon** — A kubectl operation that marks a node as unschedulable so no new pods are placed on it, while leaving existing pods running.

**ConfigMap** — A Kubernetes resource that stores non-sensitive configuration data as key-value pairs, which can be injected into pods as environment variables or mounted as files.

**Container Registry (gcr.io)** — The predecessor to Artifact Registry for storing Docker container images on GCP; being replaced by Artifact Registry.

**Control Plane** — The set of components in a Kubernetes cluster (API server, etcd, scheduler, controller manager) that manage the cluster state; in GKE, the control plane is Google-managed.

**CPU Limit** — The maximum amount of CPU a Kubernetes container may consume; enforced by the kubelet through cgroups on the node.

**CPU Request** — The amount of CPU a Kubernetes container is guaranteed to receive; used by the scheduler to decide which node to place a pod on.

**CRD (Custom Resource Definition)** — A Kubernetes API extension mechanism that lets users define new resource types such as GKE's BackendConfig.

**DaemonSet** — A Kubernetes workload resource that runs a copy of a pod on every (or a selected subset of) cluster nodes, used for cluster-wide infrastructure such as log collectors, node monitoring agents, and CNI plugins.

**Deployment** — A Kubernetes resource that manages a set of identical, stateless pod replicas with support for rolling updates, rollbacks, and scaling.

**Drain** — A kubectl operation that cordons a node and then evicts its pods so the node can be safely removed or upgraded (`kubectl drain`).

**Docker** — A platform for building, shipping, and running containers; GKE nodes pull Docker-formatted container images from registries like Artifact Registry.

**Endpoint** — A Kubernetes resource that lists the IP addresses and ports of the pods matched by a Service selector; viewed with `kubectl get endpoints`.

**etcd** — A distributed key-value store used by Kubernetes as its primary data store for all cluster state and configuration; managed by Google in GKE.

**Filestore** — GCP's managed NFS file storage service, used to back Kubernetes PersistentVolumes that require ReadWriteMany (RWX) access mode.

**Fluentd** — An open-source log collector commonly deployed as a DaemonSet to collect node logs across a Kubernetes cluster.

**gcloud** — Google Cloud's command-line tool used to create, update, and delete GKE clusters, node pools, and credentials; `gcloud container` is the GKE subcommand group.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services; GKE is one of its core managed services.

**GKE (Google Kubernetes Engine)** — GCP's fully managed Kubernetes service that automates the deployment, scaling, and management of containerized applications using Kubernetes.

**GPU (Graphics Processing Unit)** — A specialized processor used for machine learning and compute-intensive workloads; GKE supports GPU node pools via the `--accelerator` flag.

**GSA (Google Service Account)** — A GCP identity that applications and services use to authenticate to GCP APIs; mapped to Kubernetes Service Accounts via Workload Identity.

**HA (High Availability)** — A design principle that ensures a system remains operational with minimal downtime; achieved in GKE through multi-zone node pools and regional clusters.

**HPA (Horizontal Pod Autoscaler)** — A Kubernetes resource that automatically scales the number of pod replicas in a Deployment or StatefulSet based on CPU, memory, or custom metrics.

**IAM (Identity and Access Management)** — GCP's access control system for managing who has what permissions on which resources; used to grant Artifact Registry and Workload Identity permissions.

**IAP (Identity-Aware Proxy)** — A GCP service that enforces identity-based access to HTTP(S) applications; attachable to a GKE Ingress backend via BackendConfig.

**ImagePullSecrets** — Kubernetes Secret objects that store credentials for authenticating to private container registries; referenced in pod specs to allow image pulls.

**Ingress** — A Kubernetes resource that exposes HTTP/HTTPS routes from outside the cluster to Services based on hostnames and paths; on GKE, the default controller provisions a Google Cloud HTTP(S) Load Balancer.

**Ingress (NetworkPolicy)** — In the context of Kubernetes NetworkPolicy, traffic flowing into a pod from other pods or external sources; controlled using `policyTypes: Ingress` rules.

**Ingress Class** — An annotation (`kubernetes.io/ingress.class`) or IngressClass resource that determines which Ingress controller handles an Ingress; GKE values include `gce` (external LB) and `gce-internal` (internal LB).

**Init Container** — A Kubernetes container that runs before the main application containers in a pod; used for setup tasks like fetching config or waiting on dependencies.

**kube-system** — The default Kubernetes namespace where system components (DNS, proxy, DaemonSets, metrics server) run.

**KSA (Kubernetes Service Account)** — A Kubernetes identity used by pods to authenticate to the Kubernetes API and, via Workload Identity, to GCP APIs.

**kubectl** — The command-line interface for managing Kubernetes clusters; used to create, view, and delete resources such as pods, services, deployments, and ConfigMaps.

**Kubernetes** — An open-source container orchestration system for automating deployment, scaling, and management of containerized applications; the foundation of GKE.

**Label** — A key-value pair attached to Kubernetes objects (pods, nodes, services) used to identify and select groups of resources via label selectors.

**Label Selector** — A Kubernetes query based on labels used by Services, Deployments, and NetworkPolicies to identify which pods they apply to (e.g., `app=backend`).

**LoadBalancer** — A Kubernetes Service type that provisions an external GCP load balancer and assigns a public IP address to expose the service to internet traffic.

**Maintenance Window** — A configurable time period during which GKE is allowed to perform auto-upgrades and maintenance operations on clusters and node pools.

**Memory Limit** — The maximum amount of memory a Kubernetes container may consume; exceeding the limit causes the kubelet to OOM-kill the container.

**Memory Request** — The amount of memory a Kubernetes container is guaranteed to receive; used by the scheduler to determine node placement for pods.

**Namespace** — A Kubernetes mechanism for isolating resources within a cluster; used to separate workloads by team, environment, or application.

**Network Policy** — A Kubernetes resource that controls how pods communicate with each other and with external endpoints; requires the `--enable-network-policy` flag on GKE clusters.

**Node** — A worker machine (Compute Engine VM) in a GKE cluster that runs containerized workloads; managed by the control plane.

**Node Affinity** — A Kubernetes pod scheduling rule that constrains pods to run on nodes matching certain labels; similar to nodeSelector but with more expressive operators.

**Node Pool** — A group of nodes within a GKE cluster that share the same configuration (machine type, disk size, labels, taints); a cluster can have multiple node pools.

**NodePort** — A Kubernetes Service type that exposes the service on a static port on each cluster node's IP address, enabling external access without a cloud load balancer.

**NodeSelector** — A pod spec field that restricts scheduling to nodes whose labels match the given key-value pairs.

**NVIDIA Tesla T4** — A GPU model from NVIDIA supported in GKE GPU node pools for machine learning and compute workloads.

**PersistentVolume (PV)** — A cluster-level Kubernetes storage resource (e.g., a GCE persistent disk) that can be dynamically or statically provisioned and consumed by pods through a PersistentVolumeClaim.

**PersistentVolumeClaim (PVC)** — A Kubernetes resource that requests storage from a StorageClass; used by StatefulSets to provide each pod with its own persistent storage.

**Pod** — The smallest deployable unit in Kubernetes; a group of one or more containers that share network and storage resources and run on the same node.

**PodSelector** — A label-based selector in Kubernetes NetworkPolicy that identifies which pods the policy applies to; an empty podSelector matches all pods in the namespace.

**PostgreSQL** — An open-source relational database; used as a StatefulSet example in GKE deployments, with each replica requiring its own PersistentVolumeClaim.

**ReclaimPolicy** — A PersistentVolume field controlling what happens to the underlying storage when its PVC is deleted: `Delete` removes the disk, `Retain` keeps it.

**Regional Cluster** — A GKE cluster whose control plane and nodes are replicated across multiple zones in a region for high availability.

**Release Channel** — A GKE setting (Rapid, Regular, Stable, Extended) that determines which Kubernetes version a cluster receives and the cadence of automatic upgrades.

**Replica** — A copy of a pod managed by a Deployment or StatefulSet; the number of replicas determines how many pod instances run simultaneously.

**Resource Limit** — See CPU Limit / Memory Limit; the maximum compute resources a container may use before being throttled or OOM-killed.

**Resource Request** — See CPU Request / Memory Request; the guaranteed amount of a resource used by the scheduler to place pods on nodes.

**Role (IAM role)** — A named collection of GCP IAM permissions (e.g., `roles/artifactregistry.reader`, `roles/iam.workloadIdentityUser`) granted to principals to control cluster and registry access.

**Rolling Update** — A Kubernetes Deployment strategy that incrementally replaces old pods with new ones, ensuring continuous availability during a version upgrade.

**Rollback** — Reverting a Deployment to a previous revision via `kubectl rollout undo` when a new rollout fails.

**Secret** — A Kubernetes resource that stores sensitive data (passwords, tokens, keys) as base64-encoded values; should be replaced with Secret Manager for production workloads.

**Secret Manager** — A GCP service for securely storing and managing sensitive configuration data (API keys, passwords, certificates); recommended over Kubernetes Secrets.

**Secret Manager Add-on** — A GKE add-on that allows Secret Manager secrets to be automatically synced and mounted into pods via the Secrets Store CSI Driver.

**Service** — A Kubernetes resource that provides a stable network endpoint (IP address and DNS name) for a set of pods selected by labels.

**Service Account (Kubernetes)** — See KSA.

**Spot VM** — A Compute Engine VM type used in GKE node pools that offers lower cost by using spare capacity but can be preempted by GCP with short notice.

**Standard (GKE mode)** — A GKE cluster mode in which the user manages node pools directly and is billed per VM (as opposed to Autopilot's per-pod billing).

**StatefulSet** — A Kubernetes workload resource for managing stateful applications (databases, message queues) that require stable network identities, persistent storage, and ordered deployment.

**StorageClass** — A Kubernetes resource that describes the type of storage offered by a cluster (e.g., pd-ssd via `pd.csi.storage.gke.io`) and enables dynamic provisioning of PersistentVolumes.

**Surge Upgrade** — A GKE node pool upgrade strategy that creates additional (surge) nodes before removing old ones, controlled by `--max-surge-upgrade` and `--max-unavailable-upgrade` settings.

**Taint** — A Kubernetes node property that repels pods from being scheduled on a node unless the pod has a matching toleration; used to dedicate nodes to specific workloads (e.g., `dedicated=batch:NoSchedule`).

**TCP (Transmission Control Protocol)** — A connection-oriented transport protocol; used in Kubernetes NetworkPolicy and Service port definitions.

**Toleration** — A pod spec field that allows a pod to be scheduled on nodes with matching taints; paired with taints to dedicate nodes to specific workloads.

**VolumeClaimTemplate** — A StatefulSet field that provides a template for automatically creating a unique PersistentVolumeClaim for each replica pod.

**VPA (Vertical Pod Autoscaler)** — A Kubernetes resource that automatically adjusts the CPU and memory requests of pods based on historical resource usage, helping right-size pod allocations.

**Workload Identity** — A GKE feature that allows Kubernetes service accounts (KSAs) to impersonate GCP service accounts (GSAs), enabling pods to authenticate to GCP APIs without storing keys.

**YAML** — A human-readable data serialization format used to define Kubernetes manifests (pods, services, deployments, StatefulSets, etc.).

**Zonal Cluster** — A GKE cluster whose control plane and default node pool live in a single zone; cheaper but has lower availability than a regional cluster.

**Zone** — A geographically isolated deployment area within a GCP region; GKE node pools can span a single zone or multiple zones for high availability.
