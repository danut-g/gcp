# Section 1 -- Visual Maps: Setting Up a Cloud Solution Environment

---

## 1. Google Cloud Resource Hierarchy

```
                         ┌─────────────────────────────────────────────┐
                         │            ORGANIZATION                     │
                         │         (mycompany.com)                     │
                         │  Tied to Workspace / Cloud Identity domain  │
                         │  Role: Organization Admin                   │
                         └──────────────────┬──────────────────────────┘
                                            │
                            ┌───────────────┼───────────────┐
                            │               │               │
                            ▼               ▼               ▼
                     ┌────────────┐  ┌────────────┐  ┌────────────┐
                     │  Folder:   │  │  Folder:   │  │  Folder:   │
                     │  Finance   │  │ Engineering│  │  Marketing │
                     └─────┬──────┘  └─────┬──────┘  └────────────┘
                           │               │
                      ┌────┴────┐     ┌────┴────┐
                      ▼         ▼     ▼         ▼
                ┌──────────┐┌──────────┐┌──────────┐┌──────────┐
                │ Project: ││ Project: ││ Project: ││ Project: │
                │ fin-prod ││ fin-dev  ││ eng-prod ││ eng-dev  │
                └────┬─────┘└──────────┘└────┬─────┘└──────────┘
                     │                       │
              ┌──────┼──────┐         ┌──────┼──────┐
              ▼      ▼      ▼         ▼      ▼      ▼
            [VM]  [Bucket] [DB]     [VM]  [GKE]  [Bucket]
```

### Key Rules

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │  HIERARCHY RULES                                                     │
  ├──────────────────────────────────────────────────────────────────────┤
  │                                                                      │
  │  1. Organization ─── optional but recommended (needs Workspace       │
  │                      or Cloud Identity)                              │
  │                                                                      │
  │  2. Folders ──────── optional, up to 10 levels deep                  │
  │                                                                      │
  │  3. Projects ─────── MANDATORY (every resource needs one)            │
  │                                                                      │
  │  4. Resources ────── always belong to exactly ONE project            │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘
```

### Project Identifiers

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │                       PROJECT IDENTIFIERS                            │
  ├───────────────┬──────────────────┬───────────────────────────────────┤
  │  Identifier   │  Unique?         │  Who Sets It?                    │
  ├───────────────┼──────────────────┼───────────────────────────────────┤
  │  Project Name │  NOT unique      │  You choose, can change later    │
  │  Project ID   │  GLOBALLY unique │  You choose at creation, FIXED   │
  │  Project Num  │  GLOBALLY unique │  Auto-assigned by Google, FIXED  │
  └───────────────┴──────────────────┴───────────────────────────────────┘
```

---

## 2. IAM Inheritance Diagram

```
           ┌──────────────────────────────────────┐
           │          ORGANIZATION                 │
           │  Policy: Editor for group:ops@co.com  │
           └──────────────────┬───────────────────┘
                              │
                  INHERITS DOWNWARD
                              │
           ┌──────────────────▼───────────────────┐
           │           FOLDER: Prod                │
           │  Policy: Viewer for user:alice@co.com │
           └──────────────────┬───────────────────┘
                              │
                  INHERITS DOWNWARD
                              │
           ┌──────────────────▼───────────────────┐
           │        PROJECT: web-app-prod          │
           │  Policy: (none added here)            │
           └──────────────────┬───────────────────┘
                              │
                  INHERITS DOWNWARD
                              │
           ┌──────────────────▼───────────────────┐
           │        RESOURCE: VM instance          │
           │                                       │
           │  Effective roles for alice@co.com:     │
           │    - Viewer (from Folder)              │
           │                                       │
           │  Effective roles for ops@co.com:       │
           │    - Editor (from Organization)        │
           └──────────────────────────────────────┘


  ┌──────────────────────────────────────────────────────────────┐
  │                   GOLDEN RULES                               │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  1. Policies ONLY flow DOWNWARD (never upward)               │
  │                                                              │
  │  2. MORE PERMISSIVE wins                                     │
  │     Parent = Editor  +  Child = Viewer  >>>  Editor wins     │
  │                                                              │
  │  3. You CANNOT restrict at a lower level what was            │
  │     granted at a higher level                                │
  │                                                              │
  │  4. Best practice: grant roles at the NARROWEST scope        │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

---

## 3. IAM Model -- Who / What / Which

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                                                                   │
  │    WHO (Member/Principal)     WHAT (Role)       WHICH (Resource)  │
  │    ─────────────────────      ──────────        ────────────────  │
  │                                                                   │
  │    user:alice@co.com     +   roles/viewer   +   project:my-app   │
  │                                                                   │
  │         (Identity)           (Permissions)       (Scope)          │
  │                                                                   │
  └───────────────────────────────────────────────────────────────────┘
```

### Member Types

```
                         ┌── user:alice@co.com ............. Individual Google Account
                         │
                         ├── serviceAccount:sa@proj.iam ... Application identity
                         │
  Members / Principals ──┼── group:devs@co.com ............ Google Group (BEST PRACTICE)
                         │
                         ├── domain:company.com ........... Entire Workspace domain
                         │
                         ├── allAuthenticatedUsers ........ Any Google account
                         │
                         └── allUsers ..................... Anyone on internet (DANGER)
```

### Role Types

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                                                                              │
  │   BASIC (Primitive)                                                          │
  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
  │   │  Viewer   │  │  Editor  │  │  Owner   │  │ Browser  │                   │
  │   │ (read)    │  │ (r/w)    │  │ (full +  │  │ (browse  │                   │
  │   │           │  │ no IAM   │  │  IAM +   │  │  hierarchy│                  │
  │   │           │  │ mgmt)    │  │  billing)│  │  only)   │                   │
  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘                   │
  │                                                                              │
  │   PREDEFINED ──── Fine-grained, per-service                                  │
  │   ┌──────────────────────────────┐  ┌──────────────────────────────┐         │
  │   │ roles/compute.instanceAdmin  │  │ roles/storage.objectViewer   │         │
  │   └──────────────────────────────┘  └──────────────────────────────┘         │
  │                                                                              │
  │   CUSTOM ───────── You define exact permissions                              │
  │   ┌──────────────────────────────┐                                           │
  │   │ roles/myCustomRole           │                                           │
  │   └──────────────────────────────┘                                           │
  │                                                                              │
  │   RECOMMENDATION:  Basic = avoid  |  Predefined = preferred  |  Custom = ok │
  │                                                                              │
  └──────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Organization Policy vs IAM -- Comparison

```
  ┌──────────────────────────────────┬──────────────────────────────────┐
  │         IAM                      │      ORGANIZATION POLICIES       │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │  Controls WHO can do things      │  Controls WHAT can be done       │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │  Grants roles to members         │  Sets constraints on resources   │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │  "Alice can create VMs"          │  "No VM may have external IP"    │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │  Applied to Org/Folder/Project   │  Applied to Org/Folder/Project   │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │  Inherits downward               │  Inherits downward               │
  ├──────────────────────────────────┼──────────────────────────────────┤
  │  More permissive wins            │  Most restrictive wins           │
  └──────────────────────────────────┴──────────────────────────────────┘
```

### Common Org Policy Constraints Mind Map

```
                                    ┌── compute.vmExternalIpAccess
                                    │     (restrict external IPs)
                                    │
                                    ├── compute.trustedImageProjects
                                    │     (limit boot disk images)
                                    │
                                    ├── compute.restrictSharedVpcSubnetworks
                                    │     (control Shared VPC subnets)
                                    │
     Organization Policy ───────────┼── iam.allowedPolicyMemberDomains
        Constraints                 │     (restrict IAM domains)
                                    │
                                    ├── storage.uniformBucketLevelAccess
                                    │     (enforce uniform access)
                                    │
                                    ├── compute.restrictVpcPeering
                                    │     (control VPC peering)
                                    │
                                    └── gcp.resourceLocations
                                          (restrict regions/zones)
```

---

## 5. Policy Inheritance Flow -- Org Policies

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │  ORGANIZATION                                                        │
  │  Constraint: gcp.resourceLocations = ALLOW [us-*, eu-*]             │
  └──────────────────────┬───────────────────────────────────────────────┘
                         │ inherits
         ┌───────────────┴──────────────────┐
         ▼                                  ▼
  ┌──────────────────────────┐  ┌──────────────────────────────────────┐
  │  FOLDER: US-Team          │  │  FOLDER: EU-Team                     │
  │  (inherits parent policy) │  │  Override: gcp.resourceLocations     │
  │  ALLOW [us-*, eu-*]       │  │  = ALLOW [eu-*] only                 │
  └───────────┬───────────────┘  └─────────────┬───────────────────────┘
              │                                │
              ▼                                ▼
  ┌───────────────────────────┐  ┌──────────────────────────────────────┐
  │  PROJECT: us-app           │  │  PROJECT: eu-app                     │
  │  Can create resources in   │  │  Can ONLY create resources in        │
  │  us-* AND eu-* regions     │  │  eu-* regions                        │
  └───────────────────────────┘  └──────────────────────────────────────┘


  NOTE: Org policies can be OVERRIDDEN at lower levels (unlike IAM
        where more permissive always wins). Org policies can be set
        to DENY or ALLOW lists, and child nodes can further restrict.
```

---

## 6. Billing Flow -- How Money Moves

```
  ┌────────────────────────────────────────────────────────────────────────┐
  │                         BILLING ARCHITECTURE                           │
  └────────────────────────────────────────────────────────────────────────┘

  ┌─────────────────┐
  │  Organization   │
  │  (mycompany.com)│
  └────────┬────────┘
           │ contains (but billing accounts are SEPARATE from hierarchy)
           │
   ┌───────┴──────────────────────────────────────┐
   │                                              │
   ▼                                              ▼
  ┌────────────────────────┐        ┌────────────────────────┐
  │  Billing Account A     │        │  Billing Account B     │
  │  (Engineering)         │        │  (Marketing)           │
  │  Payment: Credit Card  │        │  Payment: Invoice      │
  └──────┬──────┬──────────┘        └──────┬─────────────────┘
         │      │                          │
         ▼      ▼                          ▼
   ┌────────┐┌────────┐             ┌────────────┐
   │Proj A  ││Proj B  │             │ Proj C     │
   │$150/mo ││$300/mo │             │ $200/mo    │
   └────────┘└────────┘             └────────────┘


  ┌──────────────────────────────────────────────────────────────────┐
  │  KEY RULES                                                       │
  ├──────────────────────────────────────────────────────────────────┤
  │  - 1 Project  ──►  1 Billing Account (at a time)                 │
  │  - 1 Billing Account  ──►  MANY Projects                        │
  │  - No billing account = no billable resources                    │
  │  - Billing accounts sit ALONGSIDE the hierarchy (not inside it)  │
  └──────────────────────────────────────────────────────────────────┘
```

### Billing Account Types

```
  ┌───────────────────────────────────┬───────────────────────────────────┐
  │     SELF-SERVE (Online)           │     INVOICED (Offline)            │
  ├───────────────────────────────────┼───────────────────────────────────┤
  │  Credit/debit card or bank        │  Google sends monthly invoices    │
  │  Charges automatically            │  Net-30 or Net-60 payment terms   │
  │  Instant setup                    │  Requires application/approval    │
  │  Good for: small-medium orgs      │  Good for: large enterprises     │
  └───────────────────────────────────┴───────────────────────────────────┘
```

---

## 7. Budget Alerts -- Process Flow

```
  Step 1                Step 2              Step 3              Step 4
  CREATE BUDGET         SET THRESHOLDS      CHOOSE ACTION       OPTIONAL: AUTO-RESPOND
  ┌────────────┐        ┌─────────────┐     ┌──────────────┐    ┌──────────────────────┐
  │ Scope:     │        │ 50% actual  │     │ Email admins │    │ Budget Alert         │
  │ - Billing  │───────►│ 90% actual  │────►│ Monitoring   │───►│       │              │
  │   account  │        │ 100% actual │     │   channels   │    │       ▼              │
  │ - Project  │        │ 100% forec. │     │ Pub/Sub      │    │ Pub/Sub Topic        │
  │ - Service  │        │ (custom %)  │     │              │    │       │              │
  │ - Labels   │        └─────────────┘     └──────────────┘    │       ▼              │
  └────────────┘                                                │ Cloud Function       │
                                                                │       │              │
                                                                │   ┌───┴───┐          │
                                                                │   ▼       ▼          │
                                                                │ Disable  Scale       │
                                                                │ billing  down        │
                                                                │          resources   │
                                                                └──────────────────────┘

  ┌───────────────────────────────────────────────────────────────────────┐
  │  WARNING: Budgets DO NOT automatically stop spending!                 │
  │  They ONLY send notifications. You need Pub/Sub + Cloud Function     │
  │  to programmatically cap costs.                                      │
  └───────────────────────────────────────────────────────────────────────┘
```

---

## 8. Billing Export -- Decision Flow

```
  Need to analyze billing data?
         │
         ▼
  ┌──────────────────────────────────────────────────────────────────┐
  │  WHERE should billing data go?                                   │
  └──────────────┬────────────────────────────┬──────────────────────┘
                 │                            │
                 ▼                            ▼
  ┌──────────────────────────┐  ┌──────────────────────────────────┐
  │  BigQuery (RECOMMENDED)  │  │  Cloud Storage (LEGACY)          │
  ├──────────────────────────┤  ├──────────────────────────────────┤
  │  - Near real-time        │  │  - Daily CSV/JSON files          │
  │  - Line-item detail      │  │  - Less detailed                 │
  │  - SQL queries           │  │  - Good for external tools       │
  │  - Free to export        │  │  - Being deprecated              │
  │  - 3 types:              │  │                                  │
  │    1. Standard usage     │  │                                  │
  │    2. Detailed usage     │  │                                  │
  │    3. Pricing data       │  │                                  │
  └──────────────────────────┘  └──────────────────────────────────┘
```

---

## 9. Billing Roles -- Comparison Matrix

```
  ┌──────────────────────┬──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
  │  Capability          │ creator  │  admin   │  user    │ viewer   │ proj-    │ costs-   │
  │                      │          │          │          │          │ Manager  │ Manager  │
  ├──────────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
  │ Create billing accts │    X     │          │          │          │          │          │
  │ Manage payments      │          │    X     │          │          │          │          │
  │ Link/unlink projects │          │    X     │    X     │          │    X     │          │
  │ View costs           │          │    X     │          │    X     │          │    X     │
  │ Manage budgets       │          │    X     │          │          │          │    X     │
  │ Export billing data  │          │    X     │          │          │          │    X     │
  └──────────────────────┴──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘

  Roles:
    creator     = roles/billing.creator
    admin       = roles/billing.admin
    user        = roles/billing.user
    viewer      = roles/billing.viewer
    projManager = roles/billing.projectManager
    costsManager= roles/billing.costsManager
```

---

## 10. Cloud Identity and User Management

```
                         ┌──────────────────────────────────────┐
                         │         CLOUD IDENTITY                │
                         │   (Identity as a Service - IDaaS)     │
                         └──────────────────┬───────────────────┘
                                            │
                    ┌───────────────┬────────┴────────┬──────────────────┐
                    ▼               ▼                 ▼                  ▼
             ┌────────────┐  ┌──────────┐    ┌──────────────┐   ┌────────────┐
             │   Users    │  │  Groups  │    │     SSO      │   │   Device   │
             │ Management │  │          │    │ Configuration│   │ Management │
             └─────┬──────┘  └────┬─────┘    └──────────────┘   └────────────┘
                   │              │
         ┌─────────┤              │
         ▼         ▼              ▼
  ┌──────────┐ ┌──────────┐ ┌───────────────────┐
  │  Manual  │ │  Auto    │ │  Google Group      │
  │  (Admin  │ │  (GCDS,  │ │  = Best practice   │
  │  Console)│ │  SCIM,   │ │  for IAM           │
  │          │ │  API)    │ │                    │
  └──────────┘ └──────────┘ └───────────────────┘


  ┌──────────────────────────────────────────────────────────────────────┐
  │  AUTOMATED SYNC OPTIONS                                              │
  ├──────────────────────────────────────────────────────────────────────┤
  │                                                                      │
  │  GCDS ─────── Sync from Active Directory / LDAP to Cloud Identity    │
  │  SCIM ─────── Sync from 3rd-party IdP (Okta, Azure AD, etc.)        │
  │  Admin SDK ── Programmatic user/group management via API             │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘
```

---

## 11. APIs -- Enable/Disable Process

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │  NEW PROJECT                                                        │
  │  Most APIs are DISABLED by default                                  │
  └──────────────────────────┬──────────────────────────────────────────┘
                             │
                             ▼
                   Want to use a service?
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
                  YES                NO
                    │                 │
                    ▼                 ▼
         ┌──────────────────┐    (do nothing)
         │ gcloud services  │
         │ enable API_NAME  │
         └────────┬─────────┘
                  │
                  ▼
         ┌──────────────────────────────────────────────┐
         │  API is now enabled                          │
         │  - No charge for enabling                    │
         │  - Charges only when you USE the service     │
         │  - Disabling does not delete resources       │
         └──────────────────────────────────────────────┘


  COMMONLY NEEDED APIs:
  ┌──────────────────────────────────────┬──────────────────────────┐
  │  API Name                            │  Service                 │
  ├──────────────────────────────────────┼──────────────────────────┤
  │  compute.googleapis.com              │  Compute Engine          │
  │  container.googleapis.com            │  GKE                     │
  │  storage.googleapis.com              │  Cloud Storage           │
  │  sqladmin.googleapis.com             │  Cloud SQL               │
  │  bigquery.googleapis.com             │  BigQuery                │
  │  run.googleapis.com                  │  Cloud Run               │
  │  cloudfunctions.googleapis.com       │  Cloud Functions         │
  │  monitoring.googleapis.com           │  Cloud Monitoring        │
  │  logging.googleapis.com              │  Cloud Logging           │
  └──────────────────────────────────────┴──────────────────────────┘
```

---

## 12. Operations Suite Mind Map

```
                                       ┌── Cloud Monitoring
                                       │     - Metrics, dashboards, alerts
                                       │     - Uptime checks
                                       │     - Workspace can monitor multiple projects
                                       │
                                       ├── Cloud Logging
                                       │     - Centralized log management
                                       │     - Log Router ──► Storage / BigQuery / Pub/Sub
   Google Cloud Operations ────────────┤     - Agents for custom VMs
   Suite (formerly Stackdriver)        │
                                       ├── Cloud Trace
                                       │     - Distributed tracing
                                       │     - Latency analysis
                                       │
                                       ├── Cloud Profiler
                                       │     - CPU and memory profiling
                                       │
                                       └── Error Reporting
                                             - Aggregate and display errors
```

---

## 13. Quotas Mind Map

```
                               ┌── Rate Quotas
                               │     - Reset after a time period
                               │     - Example: API calls per 100 seconds
                               │
        Quotas ────────────────┤
        (Project Level)        │
                               └── Allocation Quotas
                                     - Limit on number of resources
                                     - Example: max VMs per project


  ┌──────────────────────────────────────────────────────────────────────┐
  │  QUOTA INCREASE PROCESS                                              │
  ├──────────────────────────────────────────────────────────────────────┤
  │                                                                      │
  │  1. IAM & Admin ──► Quotas page                                      │
  │  2. Filter by service / metric / location                            │
  │  3. Select quota ──► Edit Quotas                                     │
  │  4. Enter new limit + justification                                  │
  │  5. Submit ──► Google reviews (24-48 hours)                          │
  │                                                                      │
  │  EXAM TIP: Quota errors = HTTP 429 (Too Many Requests)              │
  │  EXAM TIP: Some quotas are HARD LIMITS (cannot be increased)         │
  │  EXAM TIP: Monitor quota usage via Cloud Monitoring                  │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘
```

---

## 14. Cost Management Tools Overview

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │                        COST OPTIMIZATION SPECTRUM                            │
  └──────────────────────────────────────────────────────────────────────────────┘

         BEFORE DEPLOY           DURING OPERATIONS           LONG-TERM SAVINGS
         ─────────────           ─────────────────           ─────────────────
  ┌──────────────────┐   ┌───────────────────────┐   ┌────────────────────────┐
  │ Pricing          │   │ Billing Reports       │   │ Committed Use          │
  │ Calculator       │   │ (Cost Table)          │   │ Discounts (CUDs)       │
  │                  │   │                       │   │                        │
  │ Estimate costs   │   │ Filter by project,    │   │ 1-year: up to 57% off  │
  │ before deploying │   │ service, labels,      │   │ 3-year: up to 70% off  │
  │                  │   │ time period           │   │ (CPU + memory)         │
  └──────────────────┘   └───────────────────────┘   └────────────────────────┘

                         ┌───────────────────────┐   ┌────────────────────────┐
                         │ Labels                │   │ Sustained Use          │
                         │                       │   │ Discounts (SUDs)       │
                         │ key:value pairs on    │   │                        │
                         │ resources for cost    │   │ AUTOMATIC discount     │
                         │ allocation            │   │ for 25%+ monthly usage │
                         │ e.g. team:backend     │   │ Up to 30% off          │
                         └───────────────────────┘   └────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────────┐
  │  CUDs vs SUDs                                                        │
  ├──────────────────────────────┬───────────────────────────────────────┤
  │  CUDs (Committed Use)       │  SUDs (Sustained Use)                 │
  ├──────────────────────────────┼───────────────────────────────────────┤
  │  Manual commitment           │  Automatic (no action needed)         │
  │  1-year or 3-year            │  Monthly billing cycle                │
  │  Up to 70% off               │  Up to 30% off                       │
  │  Applies to CPU + memory     │  Applies to Compute Engine VMs       │
  │  Purchased per project       │  Applied per project automatically   │
  └──────────────────────────────┴───────────────────────────────────────┘
```

---

## 15. Complete Section 1 Exam Cheat Sheet

```
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │  SECTION 1: QUICK REFERENCE                                                  │
  ├──────────────────────────────────────────────────────────────────────────────┤
  │                                                                              │
  │  HIERARCHY:     Org ──► Folders (optional) ──► Projects (required) ──► Res.  │
  │  IAM:           WHO + WHAT ROLE + WHICH RESOURCE                             │
  │  INHERITANCE:   Flows DOWN, more permissive wins                             │
  │  ORG POLICIES:  Control WHAT can be done (constraints)                       │
  │  BILLING:       Separate from hierarchy, 1 project = 1 billing account       │
  │  BUDGETS:       Alert only, do NOT cap spending                              │
  │  EXPORT:        BigQuery (recommended) or Cloud Storage (legacy)             │
  │  QUOTAS:        Project-level, HTTP 429 on exceed                            │
  │  APIs:          Disabled by default, enable before use                        │
  │  GROUPS:        Best practice for IAM at scale                               │
  │  CLOUD ID:      IDaaS, sync with GCDS/SCIM                                  │
  │                                                                              │
  └──────────────────────────────────────────────────────────────────────────────┘
```
