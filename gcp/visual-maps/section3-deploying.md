# Section 3 -- Deploying and Implementing Cloud Solutions -- Visual Maps

---

## 1. VM Deployment Flow: Template to Load Balancer

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    COMPUTE ENGINE DEPLOYMENT PIPELINE                        │
 └──────────────────────────────────────────────────────────────────────────────┘

 STEP 1                STEP 2                STEP 3                STEP 4
 Instance Template      Managed Instance      Autoscaling           Load Balancer
 (Blueprint)            Group (MIG)           (Dynamic)             (Distribution)

 ┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
 │              │      │              │      │              │      │              │
 │  INSTANCE    │─────►│   MANAGED    │─────►│  AUTOSCALER  │─────►│     LOAD     │
 │  TEMPLATE    │      │  INSTANCE    │      │              │      │   BALANCER   │
 │              │      │   GROUP      │      │              │      │              │
 │  - Machine   │      │              │      │  Signals:    │      │  - Frontend  │
 │    type      │      │  - Zonal     │      │  - CPU util  │      │  - Backend   │
 │  - Image     │      │  - Regional  │      │  - HTTP LB   │      │  - Health    │
 │  - Disks     │      │    (HA)      │      │  - Custom    │      │    checks    │
 │  - Network   │      │              │      │    metric    │      │  - URL map   │
 │  - Tags      │      │  Features:   │      │  - Schedule  │      │              │
 │  - Metadata  │      │  - Autohealing│     │              │      │  Types:      │
 │  - SA        │      │  - Rolling   │      │  min=2       │      │  - HTTP(S)   │
 │              │      │    updates   │      │  max=10      │      │  - TCP/UDP   │
 │  IMMUTABLE   │      │  - Canary    │      │  target=70%  │      │  - Internal  │
 │  (read-only) │      │    deploy    │      │  cooldown=90s│      │  - Network   │
 └──────────────┘      └──────────────┘      └──────────────┘      └──────────────┘
        │                      │                     │                      │
        │                      │                     │                      │
        ▼                      ▼                     ▼                      ▼
   Cannot edit           Identical VMs          Scales up/down        Distributes
   Create new            from template          automatically         traffic evenly


 ┌─────────────────────────────────────────────────────────────────────────┐
 │                    ROLLING UPDATE STRATEGIES                            │
 ├─────────────────────────────────────────────────────────────────────────┤
 │                                                                         │
 │  Standard Update:           Canary Update:                              │
 │                                                                         │
 │  ┌────┐ ┌────┐ ┌────┐      ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐       │
 │  │ v2 │ │ v2 │ │ v1 │      │ v1 │ │ v1 │ │ v1 │ │ v1 │ │ v2 │       │
 │  └────┘ └────┘ └────┘      └────┘ └────┘ └────┘ └────┘ └────┘       │
 │                                                         ▲              │
 │  max-surge=3                          canary-version   20%             │
 │  max-unavailable=0                    target-size=20%                   │
 └─────────────────────────────────────────────────────────────────────────┘


 ┌─────────────────────────────────────────────────────────────────────────┐
 │                    MIG AUTOHEALING                                       │
 ├─────────────────────────────────────────────────────────────────────────┤
 │                                                                         │
 │  Health Check (HTTP :80/health)                                         │
 │       │                                                                 │
 │       ├──► VM responds 200 OK ──────► HEALTHY (keep running)            │
 │       │                                                                 │
 │       └──► VM fails 3 times ────────► UNHEALTHY                         │
 │                                            │                            │
 │                                            ▼                            │
 │                                    DELETE unhealthy VM                   │
 │                                            │                            │
 │                                            ▼                            │
 │                                    CREATE new VM from template           │
 │                                            │                            │
 │                                            ▼                            │
 │                                    WAIT initial-delay (300s)             │
 │                                            │                            │
 │                                            ▼                            │
 │                                    START health checking again           │
 └─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. GKE Architecture: Control Plane to Services

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                     GKE CLUSTER ARCHITECTURE                                 │
 └──────────────────────────────────────────────────────────────────────────────┘

                         ┌──────────────────────────────────────┐
                         │       CONTROL PLANE (Google-managed) │
                         │                                      │
                         │   ┌────────────┐  ┌──────────────┐  │
                         │   │ API Server │  │  Scheduler   │  │
                         │   └────────────┘  └──────────────┘  │
                         │   ┌────────────┐  ┌──────────────┐  │
                         │   │ Controller │  │    etcd       │  │
                         │   │  Manager   │  │ (state store) │  │
                         │   └────────────┘  └──────────────┘  │
                         └──────────────┬───────────────────────┘
                                        │
                         ┌──────────────┼──────────────┐
                         │              │              │
                         ▼              ▼              ▼
              ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
              │   NODE 1     │ │   NODE 2     │ │   NODE 3     │
              │   (VM)       │ │   (VM)       │ │   (VM)       │
              │              │ │              │ │              │
              │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │
              │ │  Pod A   │ │ │ │  Pod C   │ │ │ │  Pod E   │ │
              │ │┌────────┐│ │ │ │┌────────┐│ │ │ │┌────────┐│ │
              │ ││Container││ │ │ ││Container││ │ │ ││Container││ │
              │ │└────────┘│ │ │ │└────────┘│ │ │ │└────────┘│ │
              │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │
              │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │
              │ │  Pod B   │ │ │ │  Pod D   │ │ │ │  Pod F   │ │
              │ │┌────────┐│ │ │ │┌────────┐│ │ │ │┌────────┐│ │
              │ ││Container││ │ │ ││Container││ │ │ ││Container││ │
              │ │└────────┘│ │ │ │└────────┘│ │ │ │└────────┘│ │
              │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │
              │              │ │              │ │              │
              │  kubelet     │ │  kubelet     │ │  kubelet     │
              │  kube-proxy  │ │  kube-proxy  │ │  kube-proxy  │
              └──────────────┘ └──────────────┘ └──────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                     KUBERNETES NETWORKING MODEL                              │
 └──────────────────────────────────────────────────────────────────────────────┘

   INTERNET
      │
      ▼
 ┌──────────┐     ┌──────────────────────────────────────────────────────┐
 │  Ingress  │     │  Layer 7 routing (HTTP/S paths and hosts)           │
 │ Resource  │────►│  example.com/api  ──► Service A                     │
 │           │     │  example.com/web  ──► Service B                     │
 └──────────┘     └───────────────────────────┬──────────────────────────┘
                                              │
                                              ▼
                           ┌────────────────────────────────┐
                           │        SERVICE                  │
                           │  (Stable IP + DNS name)         │
                           │                                 │
                           │  Types:                         │
                           │  ┌──────────┐ ┌──────────────┐ │
                           │  │ClusterIP │ │ LoadBalancer  │ │
                           │  │(internal)│ │  (external)  │ │
                           │  └──────────┘ └──────────────┘ │
                           │  ┌──────────┐ ┌──────────────┐ │
                           │  │ NodePort │ │ ExternalName │ │
                           │  │ (node IP)│ │  (DNS alias) │ │
                           │  └──────────┘ └──────────────┘ │
                           └───────────────┬────────────────┘
                                           │
                            ┌──────────────┼──────────────┐
                            │              │              │
                            ▼              ▼              ▼
                       ┌────────┐    ┌────────┐    ┌────────┐
                       │ Pod    │    │ Pod    │    │ Pod    │
                       │ (app)  │    │ (app)  │    │ (app)  │
                       │ :8080  │    │ :8080  │    │ :8080  │
                       └────────┘    └────────┘    └────────┘
                       label:         label:         label:
                       app=my-app     app=my-app     app=my-app


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                  STANDARD vs AUTOPILOT CLUSTER                               │
 ├──────────────────────────────────┬───────────────────────────────────────────┤
 │         STANDARD                 │           AUTOPILOT                       │
 ├──────────────────────────────────┼───────────────────────────────────────────┤
 │                                  │                                           │
 │  ┌─────────────────────────┐     │  ┌─────────────────────────┐             │
 │  │ YOU manage:             │     │  │ GOOGLE manages:          │             │
 │  │  - Node pools           │     │  │  - Nodes                 │             │
 │  │  - Machine types        │     │  │  - Security hardening    │             │
 │  │  - Autoscaling config   │     │  │  - Scaling               │             │
 │  │  - Security hardening   │     │  │  - Bin-packing           │             │
 │  └─────────────────────────┘     │  └─────────────────────────┘             │
 │                                  │                                           │
 │  Pay for: NODE VMs               │  Pay for: POD resources (CPU/mem/disk)    │
 │  Control: Full                    │  Control: Limited                         │
 │  Overhead: Higher                 │  Overhead: Lower                         │
 │  Best for: GPUs, custom config    │  Best for: Most workloads, less ops      │
 └──────────────────────────────────┴───────────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                  WORKLOAD IDENTITY FLOW                                      │
 └──────────────────────────────────────────────────────────────────────────────┘

 ┌──────┐    ┌────────────┐    ┌────────────────┐    ┌──────────┐    ┌────────┐
 │ Pod  │───►│ K8s Service│───►│   Workload     │───►│ GCP SA   │───►│ GCP    │
 │      │    │  Account   │    │   Identity     │    │          │    │ APIs   │
 └──────┘    └────────────┘    └────────────────┘    └──────────┘    └────────┘
                                   (mapping)          roles/          Cloud
                                                      storage.       Storage,
                                                      objectViewer   BigQuery,
                                                                     etc.
   No service account keys needed -- IAM-managed identity federation
```

---

## 3. Cloud Run Revision and Traffic Model

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD RUN SERVICE MODEL                                   │
 └──────────────────────────────────────────────────────────────────────────────┘

                              CLOUD RUN SERVICE
                          (my-service.run.app)
                                    │
                         ┌──────────┼──────────┐
                         │   Traffic Router     │
                         │   (Google-managed)   │
                         └──────────┬───────────┘
                                    │
            ┌───────────────────────┼────────────────────────┐
            │                       │                        │
            ▼                       ▼                        ▼
   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
   │   REVISION 1    │    │   REVISION 2    │    │   REVISION 3    │
   │   (v1 - old)    │    │   (v2 - canary) │    │   (v3 - latest) │
   │                 │    │                 │    │                 │
   │   0% traffic    │    │   10% traffic   │    │   90% traffic   │
   │                 │    │                 │    │                 │
   │  ┌───────────┐  │    │  ┌───────────┐  │    │  ┌───────────┐  │
   │  │ Instance  │  │    │  │ Instance  │  │    │  │ Instance  │  │
   │  └───────────┘  │    │  └───────────┘  │    │  │ Instance  │  │
   │                 │    │                 │    │  │ Instance  │  │
   │  IMMUTABLE      │    │  IMMUTABLE      │    │  │ Instance  │  │
   │  code+config    │    │  code+config    │    │  └───────────┘  │
   └─────────────────┘    └─────────────────┘    └─────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    DEPLOYMENT STRATEGIES                                      │
 └──────────────────────────────────────────────────────────────────────────────┘

  Strategy 1: INSTANT (default)            Strategy 2: CANARY
  ┌──────────────────────────┐             ┌──────────────────────────┐
  │ gcloud run deploy        │             │ Deploy with --no-traffic │
  │                          │             │ Then split traffic:      │
  │ Before:                  │             │                          │
  │ ████████████ v1  100%    │             │ Before:                  │
  │                          │             │ ████████████ v1  100%    │
  │ After:                   │             │                          │
  │ ████████████ v2  100%    │             │ After:                   │
  │                          │             │ ██████████   v1   90%    │
  │ New revision gets all    │             │ ██           v2   10%    │
  │ traffic immediately      │             │                          │
  └──────────────────────────┘             └──────────────────────────┘

  Strategy 3: GRADUAL ROLLOUT              Strategy 4: ROLLBACK
  ┌──────────────────────────┐             ┌──────────────────────────┐
  │                          │             │                          │
  │ Step 1:  95% v1 / 5% v2 │             │ Problem detected!        │
  │ ████████████ ▮           │             │                          │
  │                          │             │ Instant rollback:        │
  │ Step 2:  80% v1 / 20% v2│             │                          │
  │ ██████████ ████          │             │ --to-revisions=v1=100    │
  │                          │             │                          │
  │ Step 3:  50% v1 / 50% v2│             │ Before:                  │
  │ ████████ ████████        │             │ ██████████   v1   90%    │
  │                          │             │ ██           v2   10%    │
  │ Step 4:  0% v1 / 100% v2│             │                          │
  │          ████████████████│             │ After:                   │
  └──────────────────────────┘             │ ████████████ v1  100%    │
                                           └──────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    CLOUD RUN AUTOSCALING                                      │
 └──────────────────────────────────────────────────────────────────────────────┘

  Instances
  (count)
     │
  50 ┤                          ┌────────┐
     │                     ┌────┘        └────┐
  40 ┤                ┌────┘                  └────┐
     │           ┌────┘                            └────┐
  30 ┤      ┌────┘         PEAK TRAFFIC                 └────┐
     │ ┌────┘                                                └────┐
  20 ┤─┘                                                          └──┐
     │                                                               └──┐
  10 ┤                                                                  └──
     │
   2 ┤────  min-instances=2 (warm instances, no cold starts)
   0 ┤─────────────────────────────────────────────────────────────────► Time

   max-instances=50 (ceiling)
   concurrency=80 (requests per instance)

   Key parameters:
   ┌─────────────────┬───────────────────────────────────────────────┐
   │ min-instances=0 │ Scale to zero when idle (cheapest, cold start)│
   │ min-instances=1 │ 1 warm instance always (avoid cold starts)    │
   │ min-instances=N │ N warm instances (for latency SLAs)           │
   │ concurrency     │ Higher = fewer instances = lower cost         │
   └─────────────────┴───────────────────────────────────────────────┘
```

---

## 4. VPC / Subnet / Firewall Visual Model

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    VPC NETWORKING MODEL                                       │
 └──────────────────────────────────────────────────────────────────────────────┘

 ┌─── VPC: my-vpc (GLOBAL resource) ─────────────────────────────────────────┐
 │                                                                            │
 │   us-central1 (Region)              europe-west1 (Region)                  │
 │   ┌─────────────────────────┐       ┌─────────────────────────┐           │
 │   │ Subnet: web-subnet      │       │ Subnet: api-subnet      │           │
 │   │ 10.0.1.0/24             │       │ 10.1.0.0/24             │           │
 │   │ (REGIONAL resource)     │       │ (REGIONAL resource)     │           │
 │   │                         │       │                         │           │
 │   │  ┌────┐ ┌────┐ ┌────┐  │       │  ┌────┐ ┌────┐         │           │
 │   │  │VM-1│ │VM-2│ │VM-3│  │       │  │VM-4│ │VM-5│         │           │
 │   │  │ a  │ │ b  │ │ c  │  │       │  │ b  │ │ c  │         │           │
 │   │  └────┘ └────┘ └────┘  │       │  └────┘ └────┘         │           │
 │   │  (zone) (zone) (zone)  │       │  (zone) (zone)         │           │
 │   └─────────────────────────┘       └─────────────────────────┘           │
 │                                                                            │
 │   asia-east1 (Region)                                                      │
 │   ┌─────────────────────────┐                                              │
 │   │ Subnet: db-subnet       │                                              │
 │   │ 10.2.0.0/24             │    Subnets span ALL zones in a region        │
 │   │ Private Google Access ON │    VPC is GLOBAL (spans all regions)         │
 │   │                         │                                              │
 │   │  ┌────┐ ┌────┐         │                                              │
 │   │  │VM-6│ │VM-7│         │                                              │
 │   │  └────┘ └────┘         │                                              │
 │   └─────────────────────────┘                                              │
 └────────────────────────────────────────────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    FIREWALL RULES MODEL                                      │
 └──────────────────────────────────────────────────────────────────────────────┘

  INTERNET                     VPC FIREWALL
     │                    ┌───────────────────────────────────┐
     │                    │                                   │
     │   INGRESS rules    │   Priority: 0 (highest)           │
     │   (incoming)       │        │                          │
     ├───────────────────►│        ▼                          │
     │                    │   ┌────────────────────────┐      │
     │                    │   │ Hierarchical Policies  │      │ Evaluated
     │                    │   │ (org/folder level)     │      │ first
     │                    │   └───────────┬────────────┘      │
     │                    │               ▼                   │
     │                    │   ┌────────────────────────┐      │
     │                    │   │ VPC Firewall Rules     │      │
     │                    │   │                        │      │
     │                    │   │ Rule 1: ALLOW tcp:80   │      │
     │                    │   │   source: 0.0.0.0/0    │      │
     │                    │   │   target: tag:web      │      │
     │                    │   │   priority: 1000       │      │
     │                    │   │                        │      │
     │                    │   │ Rule 2: ALLOW tcp:22   │      │
     │                    │   │   source: 35.235.240/20│      │ IAP
     │                    │   │   priority: 1000       │      │
     │                    │   │                        │      │
     │                    │   │ Rule 3: DENY all       │      │
     │                    │   │   priority: 65535      │      │ Implied
     │                    │   └────────────────────────┘      │
     │                    │               ▼                   │
     │                    │        65535 (lowest)              │
     │                    │                                   │
     │   EGRESS rules     │                                   │
     │◄───────────────────┤   Default: ALLOW all egress       │
     │   (outgoing)       │                                   │
     │                    └───────────────────────────────────┘


  TARGETING METHODS:
  ┌──────────────────────┐  ┌──────────────────────┐  ┌─────────────────────┐
  │ 1. NETWORK TAGS      │  │ 2. SERVICE ACCOUNTS  │  │ 3. ALL INSTANCES    │
  │                      │  │                      │  │                     │
  │ --target-tags=web    │  │ --target-service-    │  │ (no target = all)   │
  │                      │  │   accounts=sa@...    │  │                     │
  │ Easy to manage       │  │ More secure          │  │ Broadest scope      │
  │ Anyone can change    │  │ IAM-controlled       │  │                     │
  └──────────────────────┘  └──────────────────────┘  └─────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    SHARED VPC MODEL                                          │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌─ ORGANIZATION ──────────────────────────────────────────────────────┐
  │                                                                     │
  │  ┌─ HOST PROJECT (Shared VPC Owner) ────────────────────────┐      │
  │  │                                                           │      │
  │  │  ┌─── Shared VPC Network ──────────────────────────┐     │      │
  │  │  │                                                  │     │      │
  │  │  │  Subnet A             Subnet B                   │     │      │
  │  │  │  us-central1          europe-west1               │     │      │
  │  │  │  10.0.0.0/24          10.1.0.0/24                │     │      │
  │  │  │       │                     │                    │     │      │
  │  │  └───────┼─────────────────────┼────────────────────┘     │      │
  │  │          │                     │                          │      │
  │  └──────────┼─────────────────────┼──────────────────────────┘      │
  │             │                     │                                 │
  │      ┌──────┴──────┐       ┌──────┴──────┐       ┌──────────┐     │
  │      │ SERVICE     │       │ SERVICE     │       │ SERVICE  │     │
  │      │ PROJECT 1   │       │ PROJECT 2   │       │ PROJECT 3│     │
  │      │ (Team A)    │       │ (Team B)    │       │ (Team C) │     │
  │      │ Uses Sub A  │       │ Uses Sub B  │       │ Uses both│     │
  │      └─────────────┘       └─────────────┘       └──────────┘     │
  │                                                                     │
  │  Network admin: centralized in host project                         │
  │  Billing: separated per service project                             │
  └─────────────────────────────────────────────────────────────────────┘
```

---

## 5. IaC Tool Selection Flowchart

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    INFRASTRUCTURE AS CODE -- TOOL SELECTION                   │
 └──────────────────────────────────────────────────────────────────────────────┘


                    What do you need to manage?
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
     ┌─────────────┐  ┌────────────┐  ┌──────────────┐
     │ GCP infra   │  │ Kubernetes │  │ Multi-cloud  │
     │ resources   │  │ apps       │  │ resources    │
     └──────┬──────┘  └─────┬──────┘  └──────┬───────┘
            │               │                │
            │               ▼                │
            │     ┌──────────────────┐       │
            │     │ Does your team   │       │
            │     │ use Helm charts? │       │
            │     └────────┬─────────┘       │
            │         ┌────┴────┐            │
            │         │         │            │
            │        YES       NO            │
            │         │         │            │
            │         ▼         │            │
            │   ┌──────────┐   │            │
            │   │          │   │            │
            │   │  HELM    │   │            │
            │   │          │   │            │
            │   └──────────┘   │            │
            │                  │            │
            ▼                  ▼            ▼
   ┌──────────────────────────────────────────────┐
   │ Does your team primarily use kubectl/K8s?    │
   └────────────────────┬─────────────────────────┘
                   ┌────┴────┐
                   │         │
                  YES       NO
                   │         │
                   ▼         ▼
          ┌──────────┐  ┌───────────────────────┐
          │  CONFIG  │  │ Need best-practice    │
          │CONNECTOR │  │ GCP modules?          │
          │          │  └───────────┬────────────┘
          │ K8s YAML │         ┌───┴────┐
          │ kubectl  │         │        │
          │ GitOps   │        YES      NO
          └──────────┘         │        │
                               ▼        ▼
                       ┌──────────┐  ┌──────────┐
                       │   CFT    │  │TERRAFORM │
                       │ (Cloud   │  │          │
                       │Foundation│  │  HCL     │
                       │ Toolkit) │  │  Multi-  │
                       │          │  │  cloud   │
                       │ Google   │  │  State   │
                       │ modules  │  │  file    │
                       └──────────┘  └──────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    TOOL COMPARISON MATRIX                                     │
 ├──────────────┬───────────┬──────────┬───────────────┬────────────────────────┤
 │              │ Terraform │   CFT    │Config Connector│        Helm           │
 ├──────────────┼───────────┼──────────┼───────────────┼────────────────────────┤
 │ Manages      │ Any cloud │ GCP      │ GCP           │ K8s resources          │
 │ Language     │ HCL       │ HCL      │ YAML          │ YAML + Go templates    │
 │ Multi-cloud  │ YES       │ No       │ No            │ K8s only               │
 │ Requires     │ CLI       │ CLI      │ GKE cluster   │ K8s cluster            │
 │ State        │ File/GCS  │ File/GCS │ etcd (K8s)    │ K8s API                │
 │ Reconcile    │ On-demand │ On-demand│ Continuous    │ On-demand              │
 └──────────────┴───────────┴──────────┴───────────────┴────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    TERRAFORM WORKFLOW                                         │
 └──────────────────────────────────────────────────────────────────────────────┘

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  init    │───►│  plan    │───►│  apply   │───►│ destroy  │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
       │               │               │               │
       ▼               ▼               ▼               ▼
  Download         Preview          Create/         Delete all
  providers        changes          update          managed
  + init           (dry run)        resources       resources
  backend                           + update
                                    state file

  State Management:
  ┌──────────────────────────────────────────────────────────────┐
  │  LOCAL (dev only)  ──►  GCS BUCKET (team/production)        │
  │  terraform.tfstate      backend "gcs" { bucket = "..." }    │
  │                         + versioning + locking               │
  └──────────────────────────────────────────────────────────────┘
```

---

## 6. Data Loading Paths Diagram

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    DATA LOADING PATHS INTO GCP                               │
 └──────────────────────────────────────────────────────────────────────────────┘


  ┌──────────────────┐
  │  DATA SOURCES    │
  └────────┬─────────┘
           │
           ├──── Local files (GBs)
           │         │
           │         ▼
           │    ┌─────────────────┐    ┌──────────────────┐
           │    │ gcloud storage  │───►│  Cloud Storage   │
           │    │ cp / cp -r      │    │  (GCS bucket)    │
           │    └─────────────────┘    └────────┬─────────┘
           │                                    │
           │         ┌──────────────────────────┼───────────────────┐
           │         │                          │                   │
           │         ▼                          ▼                   ▼
           │    ┌──────────┐            ┌──────────────┐    ┌──────────────┐
           │    │ bq load  │            │ gcloud sql   │    │  Dataflow    │
           │    │          │            │ import csv   │    │  (transform  │
           │    │ CSV/JSON/│            │ import sql   │    │   + load)    │
           │    │ Avro/    │            │              │    │              │
           │    │ Parquet  │            └──────┬───────┘    └──────┬───────┘
           │    └────┬─────┘                   │                   │
           │         ▼                         ▼                   ▼
           │    ┌──────────┐            ┌──────────────┐    ┌──────────────┐
           │    │ BigQuery │            │  Cloud SQL   │    │  BigQuery /  │
           │    │          │            │  (MySQL/     │    │  Spanner /   │
           │    └──────────┘            │   Postgres)  │    │  Others      │
           │                            └──────────────┘    └──────────────┘
           │
           ├──── Large datasets (TBs)
           │         │
           │         ▼
           │    ┌──────────────────────┐
           │    │ Storage Transfer     │───► Cloud Storage ───► BigQuery
           │    │ Service              │
           │    │ (scheduled, parallel)│
           │    └──────────────────────┘
           │
           ├──── Cross-cloud (AWS S3, Azure)
           │         │
           │         ▼
           │    ┌──────────────────────┐
           │    │ Storage Transfer     │───► Cloud Storage
           │    │ Service              │
           │    │ (s3://source-bucket) │
           │    └──────────────────────┘
           │
           ├──── Offline / PBs
           │         │
           │         ▼
           │    ┌──────────────────────┐
           │    │ Transfer Appliance   │───► Cloud Storage
           │    │ (physical device)    │
           │    └──────────────────────┘
           │
           └──── Streaming data
                     │
                     ▼
                ┌──────────────────────┐
                │ Pub/Sub              │
                │ (real-time ingest)   │
                └──────────┬───────────┘
                           │
                ┌──────────┼──────────────────┐
                │          │                  │
                ▼          ▼                  ▼
           ┌─────────┐ ┌──────────┐    ┌──────────────┐
           │ Dataflow │ │ BigQuery │    │ Cloud Storage│
           │ (stream  │ │ Sub      │    │ Sub          │
           │  + ETL)  │ │ (direct) │    │ (batch files)│
           └─────────┘ └──────────┘    └──────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    TRANSFER METHOD SELECTION                                  │
 ├──────────────────┬────────────────────┬──────────────────────────────────────┤
 │     Method       │     Scale          │     Best For                         │
 ├──────────────────┼────────────────────┼──────────────────────────────────────┤
 │ gcloud storage   │ GBs                │ Ad-hoc uploads, small/medium files   │
 │ bq load          │ GBs - TBs          │ Loading directly into BigQuery       │
 │ Storage Transfer │ TBs - PBs          │ Large-scale, scheduled, cross-cloud  │
 │ Transfer Applian │ PBs                │ Offline (ship physical device)       │
 │ Dataflow         │ TBs                │ Transform-and-load ETL pipelines     │
 │ Pub/Sub          │ Streaming           │ Real-time event/message ingestion    │
 └──────────────────┴────────────────────┴──────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    PUB/SUB ARCHITECTURE                                       │
 └──────────────────────────────────────────────────────────────────────────────┘

                    ┌───────────┐
   Publishers ─────►│   TOPIC   │
                    └─────┬─────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
   │ Subscription │ │ Subscription │ │ Subscription │
   │   (Pull)     │ │   (Push)     │ │ (BigQuery)   │
   └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
          │                │                │
          ▼                ▼                ▼
   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
   │ Custom App   │ │ Cloud Run /  │ │ BigQuery     │
   │ (pulls msgs) │ │ Cloud Func.  │ │ table        │
   │              │ │ (HTTP push)  │ │ (direct)     │
   └──────────────┘ └──────────────┘ └──────────────┘
```

---

## 7. Event-Driven Architecture Patterns

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    EVENT-DRIVEN ARCHITECTURE PATTERNS                         │
 └──────────────────────────────────────────────────────────────────────────────┘


  PATTERN 1: Direct Trigger
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌───────────────┐     finalized     ┌──────────────────┐     │
  │  │ Cloud Storage │────────────────►  │ Cloud Function   │     │
  │  │ (file upload) │                   │ (process file)   │     │
  │  └───────────────┘                   └──────────────────┘     │
  └────────────────────────────────────────────────────────────────┘


  PATTERN 2: Pub/Sub Decoupling
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌───────────┐    ┌─────────┐    ┌──────────┐    ┌─────────┐ │
  │  │ Publisher │───►│ Pub/Sub │───►│   Sub    │───►│Cloud Run│ │
  │  │ (any app) │    │  Topic  │    │  (push)  │    │(process)│ │
  │  └───────────┘    └─────────┘    └──────────┘    └─────────┘ │
  └────────────────────────────────────────────────────────────────┘


  PATTERN 3: Eventarc Routing (Audit Log Events)
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌───────────────┐   ┌──────────┐   ┌──────────────────────┐  │
  │  │ Any GCP API   │──►│ Eventarc │──►│ Cloud Run / Function │  │
  │  │ (audit log)   │   │ Trigger  │   │ (react to API calls) │  │
  │  └───────────────┘   └──────────┘   └──────────────────────┘  │
  └────────────────────────────────────────────────────────────────┘


  PATTERN 4: Fan-Out
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │                        ┌─── Sub 1 ───► Cloud Function A        │
  │  ┌─────────┐          │                                        │
  │  │ Pub/Sub │──────────┼─── Sub 2 ───► Cloud Function B        │
  │  │  Topic  │          │                                        │
  │  └─────────┘          └─── Sub 3 ───► Cloud Run C              │
  │                                                                │
  └────────────────────────────────────────────────────────────────┘


 ┌──────────────────────────────────────────────────────────────────────────────┐
 │         CLOUD RUN vs CLOUD FUNCTIONS -- DECISION GUIDE                       │
 ├──────────────────────────────────┬───────────────────────────────────────────┤
 │        CLOUD RUN                 │        CLOUD FUNCTIONS                    │
 ├──────────────────────────────────┼───────────────────────────────────────────┤
 │ Multiple endpoints/routes        │ Single-purpose function                   │
 │ Custom Dockerfile                │ Simple code in supported runtime          │
 │ WebSocket / gRPC support         │ Quick event processing                    │
 │ Need > 32 GB memory              │ Short execution (< 60 min)               │
 │ Traffic splitting                │ Simple Pub/Sub / Storage triggers         │
 │ Migrating existing containers    │ Glue code between services               │
 │ Team prefers containers          │ Team prefers writing functions            │
 └──────────────────────────────────┴───────────────────────────────────────────┘
```

---

## 8. VPN and Network Connectivity

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │                    HYBRID CONNECTIVITY OPTIONS                                │
 └──────────────────────────────────────────────────────────────────────────────┘


  ON-PREMISES                                              GOOGLE CLOUD
  DATA CENTER                                              VPC
  ┌──────────────┐                                    ┌──────────────┐
  │              │    Option 1: HA VPN                 │              │
  │              │    (encrypted, over internet)       │              │
  │   Servers    │◄══════════════════════════════════► │   VMs, GKE   │
  │              │    IPsec tunnels (2+)               │   Cloud SQL  │
  │              │    BGP dynamic routing              │              │
  │              │    Up to 3 Gbps/tunnel              │              │
  │              │    99.99% SLA                       │              │
  │              │                                     │              │
  │              │    Option 2: Dedicated Interconnect  │              │
  │              │    (direct physical link)            │              │
  │              │◄──────────────────────────────────► │              │
  │              │    10-200 Gbps                       │              │
  │              │    Lowest latency                    │              │
  │              │    NOT encrypted by default          │              │
  │              │                                     │              │
  │              │    Option 3: Partner Interconnect    │              │
  │              │    (via service provider)            │              │
  │              │◄─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ►│              │
  │              │    50 Mbps - 50 Gbps                │              │
  │              │    Medium setup time                 │              │
  └──────────────┘                                    └──────────────┘


 VPC PEERING:
 ┌──────────┐         ┌──────────┐         ┌──────────┐
 │  VPC A   │◄───────►│  VPC B   │◄───────►│  VPC C   │
 │(Project1)│  peered  │(Project2)│  peered  │(Project3)│
 └──────────┘         └──────────┘         └──────────┘
      │                                         │
      └────────── NOT connected (non-transitive) ─┘

  Key: A can talk to B, B can talk to C, but A CANNOT talk to C through B
  Both sides must configure peering. No overlapping IP ranges allowed.
```
