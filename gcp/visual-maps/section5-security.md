# Section 5 — Security Visual Maps
## IAM & Service Accounts

---

## 1. IAM Model Diagram: WHO + WHAT + WHERE = Policy

```
 ┌─────────────────────────────────────────────────────────────────────────────────────┐
 │                           IAM POLICY BINDING MODEL                                  │
 │                                                                                     │
 │    ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐               │
 │    │       WHO         │   │       WHAT        │   │      WHERE       │               │
 │    │  (Principal)      │   │  (Role)           │   │  (Resource)      │               │
 │    │                  │   │                  │   │                  │               │
 │    │  user:            │   │  Role contains    │   │  Organization    │               │
 │    │  group:           │ + │  Permissions:     │ + │  Folder          │  = IAM Policy │
 │    │  serviceAccount:  │   │   ├─ service.     │   │  Project         │               │
 │    │  domain:          │   │   │   resource.   │   │  Resource        │               │
 │    │  allUsers         │   │   │   verb        │   │  (bucket, VM...) │               │
 │    │  allAuthenticated │   │   └─ ...          │   │                  │               │
 │    └──────────────────┘   └──────────────────┘   └──────────────────┘               │
 │                                                                                     │
 │    Example:                                                                         │
 │    user:alice@example.com + roles/storage.objectViewer + gs://my-bucket             │
 │    ────────────────────────────────────────────────────────────────────              │
 │    "Alice can view objects in my-bucket"                                            │
 │                                                                                     │
 │    ┌─────────────────────────────────────────────────────────────────┐               │
 │    │ Policy JSON Structure                                          │               │
 │    │                                                                │               │
 │    │  { "bindings": [                                               │               │
 │    │      { "role": "roles/storage.objectViewer",                   │               │
 │    │        "members": ["user:alice@example.com",                   │               │
 │    │                    "group:readers@example.com"] }               │               │
 │    │    ],                                                          │               │
 │    │    "etag": "BwXXX=",  ◄── concurrency control                 │               │
 │    │    "version": 3       ◄── needed for conditions                │               │
 │    │  }                                                             │               │
 │    └─────────────────────────────────────────────────────────────────┘               │
 └─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Policy Inheritance Flow: Additive Nature

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                       POLICY INHERITANCE (ADDITIVE)                                  │
 │                                                                                     │
 │  ┌──────────────────────────────────────────────────────┐                            │
 │  │              ORGANIZATION                            │                            │
 │  │  Policy: user:admin@co.com ► roles/viewer            │                            │
 │  └──────────────────────┬───────────────────────────────┘                            │
 │                         │ inherits ▼                                                 │
 │       ┌─────────────────┴────────────────────────────┐                               │
 │       │              FOLDER                          │                               │
 │       │  Policy: group:devs@co.com ► roles/editor    │                               │
 │       │  Inherited: admin@co.com ► roles/viewer      │                               │
 │       └─────────────────┬────────────────────────────┘                               │
 │                         │ inherits ▼                                                 │
 │     ┌───────────────────┴──────────────────────────────┐                             │
 │     │              PROJECT                             │                             │
 │     │  Policy: sa:my-sa@... ► roles/storage.admin      │                             │
 │     │  Inherited: admin@co.com ► roles/viewer          │                             │
 │     │  Inherited: devs@co.com  ► roles/editor          │                             │
 │     └───────────────────┬──────────────────────────────┘                             │
 │                         │ inherits ▼                                                 │
 │    ┌────────────────────┴───────────────────────────────┐                            │
 │    │              RESOURCE (e.g., GCS Bucket)           │                            │
 │    │  Policy: user:bob@co.com ► roles/storage.viewer    │                            │
 │    │  Inherited: ALL of the above                       │                            │
 │    └────────────────────────────────────────────────────┘                            │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │ KEY RULES                                                                  │      │
 │  │                                                                            │      │
 │  │  1. Policies are ADDITIVE ► permissions are unioned, never subtracted      │      │
 │  │  2. Child inherits ALL parent policies automatically                       │      │
 │  │  3. A restrictive child CANNOT override a permissive parent                │      │
 │  │  4. Only DENY POLICIES can block inherited permissions                     │      │
 │  │                                                                            │      │
 │  │  Effective Permissions = Org ∪ Folder ∪ Project ∪ Resource                 │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Role Types Hierarchy: Basic, Predefined, Custom

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                          ROLE TYPES HIERARCHY                                        │
 │                                                                                     │
 │  Broadest Scope                                                  Narrowest Scope    │
 │  (AVOID)                                                         (PREFERRED)        │
 │  ◄──────────────────────────────────────────────────────────────────────────────►    │
 │                                                                                     │
 │  ┌──────────────────┐    ┌──────────────────────┐    ┌────────────────────────┐      │
 │  │   BASIC ROLES     │    │  PREDEFINED ROLES     │    │    CUSTOM ROLES        │      │
 │  │  (Primitive)      │    │  (Google-managed)      │    │  (You create)          │      │
 │  │                  │    │                      │    │                        │      │
 │  │  roles/viewer     │    │  roles/compute.admin   │    │  projects/P/roles/     │      │
 │  │   └─ read-only    │    │  roles/compute.        │    │    myCustomRole        │      │
 │  │      everything   │    │    instanceAdmin.v1    │    │                        │      │
 │  │                  │    │  roles/storage.        │    │  Only the exact perms  │      │
 │  │  roles/editor     │    │    objectViewer        │    │  you specify:          │      │
 │  │   └─ read-write   │    │  roles/storage.        │    │   ├─ compute.          │      │
 │  │      everything   │    │    objectCreator       │    │   │   instances.get    │      │
 │  │   └─ NO IAM mgmt  │    │  roles/run.invoker     │    │   ├─ compute.          │      │
 │  │                  │    │  roles/container.       │    │   │   instances.list   │      │
 │  │  roles/owner      │    │    developer           │    │   └─ storage.          │      │
 │  │   └─ editor +     │    │  roles/bigquery.       │    │       objects.get      │      │
 │  │      IAM admin    │    │    dataEditor          │    │                        │      │
 │  │   └─ billing mgmt │    │  roles/pubsub.         │    │  Stages:               │      │
 │  │                  │    │    subscriber          │    │   ALPHA ► BETA ► GA    │      │
 │  │  roles/browser    │    │                      │    │   DISABLED              │      │
 │  │   └─ browse       │    │  Fine-grained,         │    │                        │      │
 │  │      hierarchy    │    │  service-specific      │    │  Max 3000 permissions  │      │
 │  │                  │    │                      │    │  Project or Org level   │      │
 │  │  1000s of perms   │    │  10s-100s of perms     │    │  Only what you need    │      │
 │  └──────────────────┘    └──────────────────────┘    └────────────────────────┘      │
 │          │                        │                            │                     │
 │          │                        │                            │                     │
 │          ▼                        ▼                            ▼                     │
 │     NEVER use in            RECOMMENDED                  USE when predefined        │
 │     production              for most cases               is too broad               │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │  EXAM TIP: Editor does NOT include IAM management or billing.             │      │
 │  │  Owner = Editor + IAM + Billing.  Custom roles: NOT at folder level.      │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Service Account Identity Diagram: Principal vs Resource

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │             SERVICE ACCOUNT: DUAL IDENTITY                                           │
 │                                                                                     │
 │         my-app-sa@PROJECT_ID.iam.gserviceaccount.com                                │
 │                                                                                     │
 │  ┌───────────────────────────────────┐  ┌────────────────────────────────────────┐   │
 │  │  SA as PRINCIPAL (Identity)       │  │  SA as RESOURCE (Managed Object)       │   │
 │  │                                   │  │                                        │   │
 │  │  "What can this SA do?"           │  │  "Who can act ON this SA?"             │   │
 │  │                                   │  │                                        │   │
 │  │  SA ──► has roles ──► on resources│  │  Users/SAs ──► have roles ──► on SA    │   │
 │  │                                   │  │                                        │   │
 │  │  Granted via:                     │  │  Granted via:                          │   │
 │  │  gcloud projects                  │  │  gcloud iam service-accounts           │   │
 │  │    add-iam-policy-binding         │  │    add-iam-policy-binding              │   │
 │  │    PROJECT_ID                     │  │    SA_EMAIL                            │   │
 │  │    --member="serviceAccount:SA"   │  │    --member="user:alice@..."           │   │
 │  │    --role="roles/storage.admin"   │  │    --role="roles/iam.                  │   │
 │  │                                   │  │           serviceAccountUser"          │   │
 │  │  Example roles granted TO SA:     │  │                                        │   │
 │  │  ├─ roles/storage.objectViewer    │  │  Roles granted ON SA:                  │   │
 │  │  ├─ roles/pubsub.publisher        │  │  ├─ roles/iam.serviceAccountUser       │   │
 │  │  └─ roles/bigquery.dataEditor     │  │  │   (attach SA to resources)          │   │
 │  │                                   │  │  ├─ roles/iam.serviceAccountToken-     │   │
 │  │                                   │  │  │   Creator (impersonate SA)          │   │
 │  │                                   │  │  ├─ roles/iam.serviceAccountAdmin      │   │
 │  │                                   │  │  │   (full SA management)              │   │
 │  │                                   │  │  └─ roles/iam.serviceAccountKeyAdmin   │   │
 │  │                                   │  │      (create/manage keys)              │   │
 │  └───────────────────────────────────┘  └────────────────────────────────────────┘   │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │ SA TYPES                                                                   │      │
 │  │                                                                            │      │
 │  │  User-managed ─── You create  ── my-app@PROJECT.iam.gserviceaccount.com    │      │
 │  │  Default ──────── Auto-created ── NUM-compute@developer.gserviceaccount... │      │
 │  │  Google-managed ─ Internal ───── service-NUM@compute-system.iam.g...       │      │
 │  │                                                                            │      │
 │  │  WARNING: Default SAs get roles/editor -- TOO BROAD for production!        │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Authentication Methods Flowchart

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                  WHICH AUTHENTICATION METHOD SHOULD I USE?                           │
 │                                                                                     │
 │                     ┌──────────────────────┐                                        │
 │                     │  Where does the       │                                        │
 │                     │  workload run?        │                                        │
 │                     └──────────┬───────────┘                                        │
 │                                │                                                    │
 │          ┌─────────────────────┼───────────────────────┐                            │
 │          ▼                     ▼                       ▼                             │
 │  ┌───────────────┐   ┌─────────────────┐   ┌───────────────────┐                    │
 │  │  ON GCP        │   │  ON GKE          │   │  OUTSIDE GCP      │                    │
 │  │  (VM, Run,     │   │  (Kubernetes     │   │  (AWS, Azure,     │                    │
 │  │   Functions)   │   │   pods)          │   │   GitHub, on-prem)│                    │
 │  └───────┬───────┘   └────────┬────────┘   └─────────┬─────────┘                    │
 │          ▼                    ▼                       ▼                              │
 │  ┌───────────────┐   ┌─────────────────┐   ┌───────────────────┐                    │
 │  │ ATTACHED SA    │   │ WORKLOAD        │   │  Can use OIDC /   │                    │
 │  │               │   │ IDENTITY        │   │  SAML provider?   │                    │
 │  │ Assign SA to   │   │                 │   └────────┬──────────┘                    │
 │  │ the resource   │   │ K8s SA bound    │            │                               │
 │  │ at creation.   │   │ to GCP SA via   │     ┌──────┴──────┐                        │
 │  │ No keys, auto  │   │ WI annotation.  │     ▼             ▼                        │
 │  │ credential via │   │ No keys needed. │  ┌────────┐  ┌──────────┐                  │
 │  │ metadata.      │   │                 │  │  YES   │  │  NO      │                  │
 │  │               │   │ BEST for GKE    │  └───┬────┘  └────┬─────┘                  │
 │  │ BEST for GCP  │   │                 │      ▼            ▼                         │
 │  └───────────────┘   └─────────────────┘  ┌─────────┐ ┌──────────┐                  │
 │                                           │WORKLOAD │ │SA KEYS   │                  │
 │  ┌───────────────────────────────┐        │IDENTITY │ │(last     │                  │
 │  │  HUMAN USER needs SA perms?   │        │FEDER-   │ │ resort!) │                  │
 │  │                               │        │ATION    │ │          │                  │
 │  │         ▼                     │        │         │ │ Must     │                  │
 │  │  ┌──────────────────────┐     │        │ No keys │ │ rotate,  │                  │
 │  │  │  SA IMPERSONATION     │     │        │ needed  │ │ secure,  │                  │
 │  │  │                      │     │        │         │ │ audit    │                  │
 │  │  │ User gets Token-      │     │        └─────────┘ └──────────┘                  │
 │  │  │ Creator role on SA.   │     │                                                  │
 │  │  │ Uses --impersonate-   │     │                                                  │
 │  │  │ service-account flag. │     │        ┌──────────────────────────────────────┐  │
 │  │  │ Short-lived, audited. │     │        │ SECURITY RANKING (best to worst):    │  │
 │  │  └──────────────────────┘     │        │  1. Attached SA / Workload Identity  │  │
 │  └───────────────────────────────┘        │  2. Workload Identity Federation     │  │
 │                                           │  3. Impersonation (short-lived)      │  │
 │                                           │  4. SA Keys (long-lived -- AVOID)    │  │
 │                                           └──────────────────────────────────────┘  │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Workload Identity Flow (GKE)

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                        GKE WORKLOAD IDENTITY FLOW                                    │
 │                                                                                     │
 │  ┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────────┐ │
 │  │             │     │              │     │              │     │                  │ │
 │  │    POD      │────►│  Kubernetes  │────►│  Workload    │────►│   GCP Service    │ │
 │  │             │     │  Service     │     │  Identity    │     │   Account        │ │
 │  │  (your app) │     │  Account     │     │  (bridge)    │     │                  │ │
 │  │             │     │  (KSA)       │     │              │     │  my-app-sa@      │ │
 │  └─────────────┘     └──────────────┘     └──────────────┘     │  PROJECT.iam.    │ │
 │                                                                │  gserviceaccount │ │
 │                                                                │  .com            │ │
 │                                                                └────────┬─────────┘ │
 │                                                                         │           │
 │                                                                         ▼           │
 │                                                                ┌──────────────────┐ │
 │                                                                │   Google Cloud   │ │
 │                                                                │   APIs           │ │
 │                                                                │                  │ │
 │                                                                │  Cloud Storage   │ │
 │                                                                │  Pub/Sub         │ │
 │                                                                │  BigQuery        │ │
 │                                                                │  ...             │ │
 │                                                                └──────────────────┘ │
 │                                                                                     │
 │  SETUP STEPS:                                                                       │
 │  ─────────────────────────────────────────────────────────────────────────────       │
 │                                                                                     │
 │  Step 1: Annotate K8s Service Account                                               │
 │  ┌────────────────────────────────────────────────────────────────────────┐          │
 │  │ kubectl annotate serviceaccount my-ksa \                               │          │
 │  │   --namespace=default \                                                │          │
 │  │   iam.gke.io/gcp-service-account=my-app-sa@PROJECT.iam.g...           │          │
 │  └────────────────────────────────────────────────────────────────────────┘          │
 │                                                                                     │
 │  Step 2: Allow KSA to impersonate GCP SA                                            │
 │  ┌────────────────────────────────────────────────────────────────────────┐          │
 │  │ gcloud iam service-accounts add-iam-policy-binding \                   │          │
 │  │   my-app-sa@PROJECT.iam.gserviceaccount.com \                          │          │
 │  │   --role="roles/iam.workloadIdentityUser" \                            │          │
 │  │   --member="serviceAccount:PROJECT.svc.id.goog[NAMESPACE/KSA_NAME]"   │          │
 │  └────────────────────────────────────────────────────────────────────────┘          │
 │                                                                                     │
 │  Step 3: GCP SA has the needed IAM roles on resources                               │
 │  ┌────────────────────────────────────────────────────────────────────────┐          │
 │  │ gcloud projects add-iam-policy-binding PROJECT_ID \                    │          │
 │  │   --member="serviceAccount:my-app-sa@PROJECT.iam.g..." \               │          │
 │  │   --role="roles/storage.objectViewer"                                  │          │
 │  └────────────────────────────────────────────────────────────────────────┘          │
 │                                                                                     │
 │  RESULT: Pod ► KSA ► Workload Identity ► GCP SA ► APIs   (NO KEYS NEEDED)          │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Service Account Impersonation Chain

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                    SERVICE ACCOUNT IMPERSONATION                                     │
 │                                                                                     │
 │  SIMPLE IMPERSONATION:                                                              │
 │  ─────────────────────                                                              │
 │                                                                                     │
 │  ┌─────────────┐  TokenCreator  ┌───────────────┐  roles/storage  ┌──────────────┐  │
 │  │             │   role ON SA   │               │  .objectAdmin   │              │  │
 │  │  User       │───────────────►│  SA-deploy     │───────────────►│  Cloud       │  │
 │  │  Alice      │  "can create   │               │  "SA has perms  │  Storage     │  │
 │  │             │   tokens as    │  target-sa@    │   on resources" │  Bucket      │  │
 │  │             │   this SA"     │  PROJECT.iam.. │               │              │  │
 │  └─────────────┘                └───────────────┘                └──────────────┘  │
 │                                                                                     │
 │  gcloud storage ls gs://my-bucket \                                                 │
 │    --impersonate-service-account=target-sa@PROJECT.iam.gserviceaccount.com           │
 │                                                                                     │
 │                                                                                     │
 │  DELEGATED CHAIN (A ► B ► C):                                                       │
 │  ────────────────────────────                                                       │
 │                                                                                     │
 │  ┌─────────┐ TokenCreator ┌─────────┐ TokenCreator ┌─────────┐ IAM roles ┌───────┐ │
 │  │         │  on SA-B     │         │  on SA-C     │         │ on bucket │       │ │
 │  │  User   │─────────────►│  SA-B   │─────────────►│  SA-C   │──────────►│ Cloud │ │
 │  │  Alice  │              │         │              │         │           │ Store │ │
 │  └─────────┘              └─────────┘              └─────────┘           └───────┘ │
 │                                                                                     │
 │  gcloud storage ls gs://my-bucket \                                                 │
 │    --impersonate-service-account=sa-c@PROJECT.iam.gserviceaccount.com \              │
 │    --impersonation-delegation=sa-b@PROJECT.iam.gserviceaccount.com                   │
 │                                                                                     │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │  WHY IMPERSONATION OVER KEYS?                                              │      │
 │  │                                                                            │      │
 │  │  ┌────────────────────┐        ┌──────────────────────────┐                │      │
 │  │  │  SA Keys (BAD)     │        │  Impersonation (GOOD)    │                │      │
 │  │  │                    │        │                          │                │      │
 │  │  │  Long-lived         │        │  Short-lived tokens      │                │      │
 │  │  │  Must rotate        │        │  Auto-expire (1 hr)      │                │      │
 │  │  │  File on disk       │        │  No files to manage      │                │      │
 │  │  │  Can be leaked      │        │  Full audit trail        │                │      │
 │  │  │  Hard to audit      │        │  Centralized IAM control │                │      │
 │  │  └────────────────────┘        └──────────────────────────┘                │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Scopes vs IAM: Effective Permissions (Venn Diagram)

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                   SCOPES vs IAM ROLES (Compute Engine VMs)                           │
 │                                                                                     │
 │  Effective Permission = Scope  INTERSECT  IAM Role                                  │
 │                                                                                     │
 │  ┌──────────────────────────────────────────────────────────────────────────┐        │
 │  │                                                                          │        │
 │  │           SCOPES                          IAM ROLES                      │        │
 │  │       (legacy mechanism)              (modern mechanism)                 │        │
 │  │                                                                          │        │
 │  │     ┌─────────────────────────────────────────────────────┐              │        │
 │  │     │              ┌──────────────────────┐               │              │        │
 │  │     │   Allowed    │                      │   Allowed     │              │        │
 │  │     │   by scope   │   EFFECTIVE           │   by IAM     │              │        │
 │  │     │   but NOT    │   PERMISSIONS         │   but NOT    │              │        │
 │  │     │   by IAM     │                      │   by scope   │              │        │
 │  │     │              │   (what actually      │               │              │        │
 │  │     │   (blocked)  │    works)             │   (blocked)  │              │        │
 │  │     │              │                      │               │              │        │
 │  │     │              └──────────────────────┘               │              │        │
 │  │     └─────────────────────────────────────────────────────┘              │        │
 │  │                                                                          │        │
 │  └──────────────────────────────────────────────────────────────────────────┘        │
 │                                                                                     │
 │  EXAMPLE:                                                                           │
 │  ─────────────────────────────────────────────────────────────────────               │
 │                                                                                     │
 │  SA has: roles/editor (IAM)     +  --scopes=storage-read-only                       │
 │                                                                                     │
 │  ┌─────────────────────┐   ┌─────────────────┐   ┌─────────────────────┐            │
 │  │ IAM allows:          │   │ Scope allows:    │   │ Effective:          │            │
 │  │  storage.read   YES  │   │ storage.read YES │   │ storage.read   YES  │            │
 │  │  storage.write  YES  │ ∩ │ storage.write NO │ = │ storage.write  NO   │            │
 │  │  compute.read   YES  │   │ compute.read  NO │   │ compute.read   NO   │            │
 │  │  compute.write  YES  │   │ compute.write NO │   │ compute.write  NO   │            │
 │  └─────────────────────┘   └─────────────────┘   └─────────────────────┘            │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │  BEST PRACTICE:  Use --scopes=cloud-platform  (allow everything)           │      │
 │  │                  then control access ONLY through IAM roles.               │      │
 │  │                  This makes scopes effectively transparent.                │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. IAM Deny Policy Evaluation Order

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                     IAM POLICY EVALUATION ORDER                                      │
 │                                                                                     │
 │  When a principal makes an API request, Google evaluates in this order:              │
 │                                                                                     │
 │         ┌──────────────────────────────────────────┐                                │
 │         │  INCOMING REQUEST                         │                                │
 │         │  User: alice@co.com                       │                                │
 │         │  Action: storage.objects.delete            │                                │
 │         │  Resource: gs://prod-bucket                │                                │
 │         └─────────────────┬────────────────────────┘                                │
 │                           │                                                         │
 │                           ▼                                                         │
 │  ┌─ STEP 1 ──────────────────────────────────────────────────┐                      │
 │  │                                                            │                      │
 │  │  CHECK DENY POLICIES          (evaluated first!)           │                      │
 │  │                                                            │                      │
 │  │  Is this permission explicitly DENIED for this principal?  │                      │
 │  │                                                            │                      │
 │  └────────────────────┬───────────────────┬──────────────────┘                      │
 │                       │                   │                                         │
 │                  YES  │              NO   │                                         │
 │                       ▼                   ▼                                         │
 │              ┌──────────────┐  ┌─ STEP 2 ──────────────────────────────┐            │
 │              │              │  │                                        │            │
 │              │   ACCESS     │  │  CHECK ALLOW POLICIES                  │            │
 │              │   DENIED     │  │                                        │            │
 │              │              │  │  Is this permission granted to this    │            │
 │              │   (stop)     │  │  principal via any allow policy?       │            │
 │              │              │  │  (org + folder + project + resource)   │            │
 │              └──────────────┘  │                                        │            │
 │                                └───────────┬───────────────┬───────────┘            │
 │                                            │               │                        │
 │                                       YES  │          NO   │                        │
 │                                            ▼               ▼                        │
 │                                   ┌──────────────┐ ┌─ STEP 3 ─────────────────┐    │
 │                                   │              │ │                            │    │
 │                                   │   ACCESS     │ │  DEFAULT DENY              │    │
 │                                   │   GRANTED    │ │                            │    │
 │                                   │              │ │  No explicit allow found   │    │
 │                                   │              │ │  ► ACCESS DENIED           │    │
 │                                   └──────────────┘ │                            │    │
 │                                                    └────────────────────────────┘    │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │  SUMMARY:   DENY wins ► then ALLOW checked ► else DEFAULT DENY            │      │
 │  │                                                                            │      │
 │  │  Deny policies:  Explicitly block permissions even if allow policy grants  │      │
 │  │  Allow policies: Additive union of all inherited + direct bindings         │      │
 │  │  Default deny:   If nothing explicitly allows it, access is denied         │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Best Practices Mind Map: Least Privilege & Security

```
 ┌──────────────────────────────────────────────────────────────────────────────────────┐
 │                        IAM & SERVICE ACCOUNT BEST PRACTICES                          │
 │                                                                                     │
 │                           ┌──────────────────┐                                      │
 │                           │  LEAST PRIVILEGE  │                                      │
 │                           │    PRINCIPLE      │                                      │
 │                           └────────┬─────────┘                                      │
 │                                    │                                                │
 │         ┌──────────────────────────┼──────────────────────────┐                     │
 │         │                          │                          │                     │
 │         ▼                          ▼                          ▼                     │
 │  ┌──────────────┐        ┌──────────────────┐       ┌─────────────────┐             │
 │  │  ROLE         │        │  SCOPE            │       │  SERVICE        │             │
 │  │  SELECTION    │        │  NARROWING        │       │  ACCOUNTS      │             │
 │  └──────┬───────┘        └────────┬─────────┘       └────────┬────────┘             │
 │         │                         │                          │                      │
 │    ┌────┴────────────┐    ┌───────┴──────────┐    ┌─────────┴──────────┐            │
 │    │                 │    │                  │    │                    │            │
 │    ▼                 ▼    ▼                  ▼    ▼                    ▼            │
 │ ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌────────┐ ┌────────────┐ ┌──────────┐     │
 │ │Predefined│ │Custom    │ │Resource-  │ │Project │ │One SA per  │ │Never use │     │
 │ │over      │ │roles     │ │level      │ │over    │ │application │ │default   │     │
 │ │basic     │ │when pre- │ │bindings   │ │org-    │ │            │ │SA in     │     │
 │ │roles     │ │defined   │ │over       │ │level   │ │            │ │production│     │
 │ │          │ │too broad │ │project    │ │        │ │            │ │          │     │
 │ └──────────┘ └──────────┘ └───────────┘ └────────┘ └────────────┘ └──────────┘     │
 │                                                                                     │
 │         ┌──────────────────────────┼──────────────────────────┐                     │
 │         │                          │                          │                     │
 │         ▼                          ▼                          ▼                     │
 │  ┌──────────────┐        ┌──────────────────┐       ┌─────────────────┐             │
 │  │  IDENTITY     │        │  CREDENTIAL       │       │  AUDIT &        │             │
 │  │  MANAGEMENT   │        │  MANAGEMENT       │       │  MONITORING     │             │
 │  └──────┬───────┘        └────────┬─────────┘       └────────┬────────┘             │
 │         │                         │                          │                      │
 │    ┌────┴────────────┐    ┌───────┴──────────┐    ┌─────────┴──────────┐            │
 │    │                 │    │                  │    │                    │            │
 │    ▼                 ▼    ▼                  ▼    ▼                    ▼            │
 │ ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌────────┐ ┌────────────┐ ┌──────────┐     │
 │ │Use       │ │Separation│ │Avoid SA   │ │Prefer  │ │IAM         │ │Enable    │     │
 │ │Google    │ │of duties │ │keys --    │ │imperso-│ │Recommender │ │Data      │     │
 │ │Groups    │ │(network/ │ │use        │ │nation  │ │for over-   │ │Access    │     │
 │ │not       │ │security/ │ │attached   │ │or WI   │ │provisioned │ │audit     │     │
 │ │individual│ │billing   │ │SAs or WI  │ │Feder-  │ │access      │ │logs      │     │
 │ │users     │ │admins)   │ │instead    │ │ation   │ │            │ │          │     │
 │ └──────────┘ └──────────┘ └───────────┘ └────────┘ └────────────┘ └──────────┘     │
 │                                                                                     │
 │  ┌────────────────────────────────────────────────────────────────────────────┐      │
 │  │  QUICK REFERENCE COMMANDS                                                  │      │
 │  │                                                                            │      │
 │  │  Audit:   gcloud projects get-iam-policy PROJECT --flatten="bindings[]"    │      │
 │  │  Analyze: gcloud asset analyze-iam-policy --organization=ORG              │      │
 │  │  Suggest: gcloud recommender recommendations list --recommender=...iam..  │      │
 │  │  Disable: gcloud iam service-accounts disable SA_EMAIL                    │      │
 │  │  Org pol: constraints/iam.disableServiceAccountKeyCreation                │      │
 │  └────────────────────────────────────────────────────────────────────────────┘      │
 └──────────────────────────────────────────────────────────────────────────────────────┘
```
