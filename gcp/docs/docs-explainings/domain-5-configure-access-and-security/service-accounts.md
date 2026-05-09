# Service Accounts: Creation, Keys, Workload Identity, Impersonation — Dual-Layer Explanation

---

# What Is a Service Account

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A robot employee badge. While human employees authenticate with their own identity (username + password), automated processes (applications, VMs, Cloud Functions) need their own badge to access GCP resources. The service account is that badge — it's an identity for a non-human, automated actor.

### B. TECHNICAL EXPLANATION
A service account is a special type of Google account associated with an application or VM instance rather than a human user. Service accounts serve a dual role: they are both a **principal** (can be granted IAM roles) and a **resource** (can have IAM policies set ON them, controlling who can use the service account). They authenticate using private keys or GCP metadata server tokens.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
The robot badge is registered in the company directory (IAM). When the robot (VM, Cloud Function) scans its badge at a door (calls a GCP API), the door checker (IAM) verifies the badge identity and checks whether that badge is authorized to enter.

### B. TECHNICAL EXPLANATION
Service account authentication mechanisms:
1. **Metadata server** (recommended): VMs and managed services automatically receive access tokens from the GCP metadata server (169.254.169.254). No key management required — tokens are auto-rotated by GCP.
2. **Service account key** (JSON file): A long-lived private key downloaded and stored outside GCP. Used when running outside GCP (on-premises, other clouds). Security risk: key files can be leaked.
3. **Workload Identity Federation**: Federated identity from external IdP (AWS, Azure, GitHub Actions) can impersonate a service account without a key file.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of service accounts as "identities for code." Every piece of code that needs to call a GCP API needs an identity. That identity should have only the permissions needed — not full admin access.

### B. TECHNICAL EXPLANATION
Default service account: GCE VMs get a default service account with project editor-level access. This is dangerous — a compromised VM can modify any resource in the project. Best practice: always create dedicated service accounts per workload with only the minimum required permissions. Never use the default service account for production workloads.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
A Cloud Function that reads from Cloud Storage needs only a "storage reader" badge. Give it a dedicated badge (service account) with exactly that permission — not the master key to the whole building.

### B. TECHNICAL EXPLANATION
Create: `gcloud iam service-accounts create my-sa --display-name="App Service Account"`. Grant permission: `gcloud projects add-iam-policy-binding PROJECT --member=serviceAccount:my-sa@PROJECT.iam.gserviceaccount.com --role=roles/storage.objectViewer`. Attach to VM: `gcloud compute instances create VM --service-account=my-sa@PROJECT.iam.gserviceaccount.com --scopes=cloud-platform`. Attach to Cloud Run: `gcloud run deploy SERVICE --service-account=my-sa@PROJECT.iam.gserviceaccount.com`.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The metadata server is like a badge dispenser at the entrance — every robot picks up a fresh, time-limited badge (access token) when it starts work and can refresh it automatically without ever touching a physical key.

### B. TECHNICAL EXPLANATION
When a VM calls the metadata server endpoint (`http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token`), GCP returns a short-lived OAuth 2.0 access token (1-hour expiry) signed for the attached service account. Google client libraries handle this automatically — no manual token management required. This is the most secure authentication method for GCP-hosted workloads.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
Leaving a master key (JSON key file) in a public restroom (GitHub repository, container image, public S3 bucket) means anyone who finds it has full access indefinitely.

### B. TECHNICAL EXPLANATION
Service account key files are the highest-risk credential in GCP. Common exposure vectors: committed to Git repositories, embedded in container images, stored in environment variables that appear in logs, passed via unencrypted channels. Key files don't expire by default (up to 10 years). Best practice: avoid key files entirely; use metadata server for GCP-hosted workloads, Workload Identity Federation for external workloads.

---

## 7. TRADE-OFFS

### A. ANALOGY
Metal keys (JSON files) work everywhere but are hard to replace when lost. Electronic tokens (metadata server) only work inside the building but are automatically cycled daily.

### B. TECHNICAL EXPLANATION
Key files: work outside GCP, maximum compatibility, but require secure storage, rotation policies, and revocation management. Metadata server tokens: most secure for GCP workloads, auto-rotated, no management overhead. Workload Identity Federation: eliminates key files for external workloads (CI/CD, other clouds) — exchange external IdP tokens for short-lived GCP tokens via federation. Strongly preferred over key files for all external workloads.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"Using the default service account is fine — it's the easy option." The default service account has project editor access — it's the riskiest choice, not a safe default.

### B. TECHNICAL EXPLANATION
The Compute Engine default service account (`PROJECT_NUMBER-compute@developer.gserviceaccount.com`) has `roles/editor` by default — broad write access across the project. A compromised VM using the default service account can modify or delete most project resources. Always create dedicated service accounts with minimal required permissions. This is consistently tested on the ACE exam.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Security professionals issue one badge per job function, track every badge, revoke them immediately on role change, and never issue skeleton keys.

### B. TECHNICAL EXPLANATION
Expert service account practices:
- One service account per workload (not one per project)
- Never download or distribute service account keys
- Enable service account key usage audit logs (`cloudaudit.googleapis.com/activity`)
- Use org policy constraint `iam.disableServiceAccountKeyCreation` to prevent key creation org-wide
- Implement Workload Identity for all GKE workloads
- Use IAM conditions for time-bounded access grants
- Regularly audit unused service accounts with `gcloud iam service-accounts list`

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
A robot employee badge — each automated workload gets its own badge with only the access it needs. Never use the master key (JSON files); use the badge dispenser (metadata server) instead.

### B. TECHNICAL SUMMARY
Service accounts are non-human identities for GCP workloads. Authenticate via metadata server (recommended for GCP workloads), JSON key files (avoid — use only when necessary for external workloads), or Workload Identity Federation. The default service account has project editor access — always create dedicated minimal-privilege service accounts instead.

---

---

# Workload Identity for GKE

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
Instead of giving each robot in the factory a physical key card (JSON file) to access company systems, you set up a trusted handshake: "Any robot that wears the 'data-processor' uniform and works in Building A can automatically be recognized as the data-processor employee." The uniform and building are the Kubernetes service account and namespace; the employee identity is the GCP service account.

### B. TECHNICAL EXPLANATION
Workload Identity binds a Kubernetes service account (KSA) to a GCP service account (GSA). Pods using the KSA can request GCP access tokens by calling the GKE metadata server, which returns tokens for the bound GSA — without any JSON key file. The binding is established via an IAM policy that grants the KSA the `roles/iam.workloadIdentityUser` role on the GSA.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
The factory tells the security system: "Any robot in uniform 'app/my-app-sa' (namespace/KSA) is authorized to use employee badge 'gsa@project.iam.gserviceaccount.com' (GCP SA)." When the robot requests a badge from the dispenser (metadata server), the dispenser checks the uniform and issues the right badge.

### B. TECHNICAL EXPLANATION
Setup steps:
1. Enable Workload Identity on GKE cluster: `--workload-pool=PROJECT_ID.svc.id.goog`
2. Create GCP service account: `gcloud iam service-accounts create gsa-name`
3. Grant binding: `gcloud iam service-accounts add-iam-policy-binding GSA_EMAIL --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]" --role=roles/iam.workloadIdentityUser`
4. Annotate Kubernetes service account: `kubectl annotate serviceaccount KSA_NAME iam.gke.io/gcp-service-account=GSA_EMAIL`

---

## 3. MENTAL MODEL

### A. ANALOGY
Workload Identity = "your uniform IS your badge." No physical key needed. The uniform (Kubernetes service account) is validated against a registry (IAM binding) and the badge (GCP token) is issued automatically.

### B. TECHNICAL EXPLANATION
The critical mental model: Workload Identity eliminates the need for service account JSON keys in GKE. Pods automatically receive GCP access tokens via the projected service account token volume, which is exchanged for a GCP token via the GKE metadata server proxy. Applications using Google client libraries receive tokens transparently — no code changes required.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
A Cloud Run service that processes files from Cloud Storage: attach a dedicated service account with `roles/storage.objectViewer` — no key files, auto-rotated tokens, principle of least privilege.

### B. TECHNICAL EXPLANATION
Workload Identity is required for: GKE Pods accessing GCP APIs (recommended, replaces key files), Cloud Functions (use `--service-account` flag — function assumes that identity), Cloud Run (use `--service-account` at deployment), App Engine (automatically uses the default service account or a configured one). For all serverless platforms: assign a dedicated service account at deployment; the platform handles token lifecycle.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The badge dispenser (metadata server) in a Workload Identity setup checks not just "who is this robot" but also "does this robot's uniform and building match an approved pairing in the registry?"

### B. TECHNICAL EXPLANATION
GKE runs a metadata proxy on each node that intercepts Pod requests to the metadata server endpoint. It validates: the requesting Pod's service account annotation matches an existing Workload Identity binding. Valid bindings return short-lived GCP access tokens (1-hour expiry, auto-refreshed). This proxy eliminates the need for JSON credentials in the Pod filesystem.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If the pairing registry (IAM binding) is misconfigured — wrong namespace, wrong KSA name, or wrong GSA — the robot can't get a badge and all GCP API calls fail with permission denied.

### B. TECHNICAL EXPLANATION
Common Workload Identity failures:
- Wrong namespace in IAM binding: `NAMESPACE/KSA_NAME` must exactly match
- Missing annotation on Kubernetes service account: `iam.gke.io/gcp-service-account` annotation is required
- KSA annotation references the wrong GSA email
- GSA lacks the required permissions for the intended GCP APIs
Debugging: check Pod logs for 403 errors, verify binding with `gcloud iam service-accounts get-iam-policy GSA_EMAIL`, verify KSA annotation with `kubectl describe sa KSA_NAME -n NAMESPACE`.

---

## 7. TRADE-OFFS

### A. ANALOGY
Electronic tokens (Workload Identity) are more convenient and secure but require initial setup. Physical keys (JSON files) work everywhere but require secure distribution and rotation.

### B. TECHNICAL EXPLANATION
Workload Identity: no key management, short-lived tokens (auto-rotated), minimal blast radius on compromise (token expires in 1 hour). Setup complexity: 4-step configuration, namespace-scoped. JSON keys: work anywhere (on-prem, any cluster), simple setup. Risk: long-lived credential, must be securely stored and rotated, easy to leak. For any production GKE workload: Workload Identity is the mandatory best practice.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"I only need to set up Workload Identity once and it works for all pods." No — each KSA needs its own binding and annotation.

### B. TECHNICAL EXPLANATION
Workload Identity bindings are per-(KSA, namespace, GSA) triple. Each unique combination of Kubernetes namespace + service account needs its own IAM binding and annotation. Multiple KSAs can share a single GSA (many-to-one) if they need the same GCP permissions.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Security architects enforce "no physical keys in any factory robot." All factory robots must use the electronic uniform system — zero exceptions.

### B. TECHNICAL EXPLANATION
Expert practice: enforce Workload Identity at the org level via org policy constraints. Disable service account key creation (`iam.disableServiceAccountKeyCreation`). For legacy apps that require key files (e.g., migration period): store in Secret Manager, inject via Kubernetes Secret (with Secret Manager CSI Driver), rotate every 90 days maximum. Monitor key usage via Cloud Audit Logs.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Badge-less factory access: your Kubernetes "uniform" (service account) is verified against a registry, and GCP tokens are issued automatically — no physical keys needed.

### B. TECHNICAL SUMMARY
Workload Identity binds Kubernetes service accounts to GCP service accounts via an IAM policy, enabling Pods to receive GCP access tokens via the metadata server without JSON key files. Requires 4-step setup: enable Workload Identity on cluster, create GSA, create IAM binding, annotate KSA. Eliminates key management overhead and is the production standard for GKE workloads.

---

---

# Service Account Impersonation

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A manager temporarily wearing an employee's badge to test whether that employee can access the filing cabinet. They're not permanently changing their own badge — they're temporarily acting as someone else for a specific task.

### B. TECHNICAL EXPLANATION
Service account impersonation allows a principal (user or service account) to generate access tokens for a target service account — thereby acting with the target SA's permissions for the duration of the impersonation. The impersonating principal requires `roles/iam.serviceAccountTokenCreator` on the target service account.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
The manager requests a temporary copy of the employee's badge from the badge office. The badge office checks: "Is this manager authorized to borrow this employee's badge?" If yes, a temporary badge copy is issued.

### B. TECHNICAL EXPLANATION
Impersonation uses the IAM Credentials API to generate short-lived tokens for a target SA. `gcloud` flag: `--impersonate-service-account=SA_EMAIL`. API: `generateAccessToken`, `generateIdToken`, `signBlob` on the service account. Requires `iam.serviceAccounts.getAccessToken` permission (included in `roles/iam.serviceAccountTokenCreator`). Tokens are short-lived (1 hour default, configurable up to 1 hour max for impersonation).

---

## 3. MENTAL MODEL

### A. ANALOGY
Impersonation is for testing and authorized delegation — not for permanently bypassing your own access level. The action is audited; you can't hide that you borrowed the badge.

### B. TECHNICAL EXPLANATION
Impersonation use cases: testing permissions of a service account without running actual workloads; granting CI/CD pipelines the ability to assume workload identities without distributing key files; Terraform service account with elevated permissions impersonated by developers only when applying infrastructure changes. All impersonation events are logged in Cloud Audit Logs.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
A developer tests "can the data processing service access this BigQuery dataset?" by temporarily impersonating the data-processing service account in their local CLI.

### B. TECHNICAL EXPLANATION
gcloud impersonation: `gcloud --impersonate-service-account=proc-sa@project.iam.gserviceaccount.com bigquery datasets describe DATASET`. The gcloud command generates an access token for `proc-sa` and uses it for the BigQuery API call. For Terraform: `provider "google" { impersonate_service_account = "tf-admin@project.iam.gserviceaccount.com" }` — Terraform uses the impersonated SA's permissions for all API calls.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
Impersonation chains: a manager can borrow an employee's badge, and if that employee is also a manager, the borrowed badge could be used to borrow another employee's badge — creating a chain. GCP limits this to prevent excessive chaining.

### B. TECHNICAL EXPLANATION
Impersonation chaining: SA-A can impersonate SA-B which can impersonate SA-C. Each step in the chain requires the `serviceAccountTokenCreator` role. GCP allows up to 10 hops in an impersonation chain. Impersonation for delegated access is an alternative to key files for cross-project workflows — one central SA with `serviceAccountTokenCreator` on multiple project SAs.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If you give everyone manager status (everyone can impersonate everyone), the badge system is meaningless — anyone can access anything.

### B. TECHNICAL EXPLANATION
Misconfiguring `roles/iam.serviceAccountTokenCreator` broadly (e.g., granting to all users in a project) effectively grants all those users the combined permissions of all service accounts in the project. This is a critical security misconfiguration. Scope `serviceAccountTokenCreator` binding precisely to the specific principal + SA combination needed.

---

## 7. TRADE-OFFS

### A. ANALOGY
Borrowing badges is audited and time-limited — safer than handing out copies. But tracking who borrowed what and when adds audit complexity.

### B. TECHNICAL EXPLANATION
Impersonation advantages over key files: short-lived tokens, audited, no credential file to leak. Disadvantages: requires IAM binding setup, slightly more complex tooling. Overall: impersonation is the preferred pattern for giving humans or external systems access to service account permissions without distributing long-lived credentials.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"Anyone can impersonate any service account in a project by default." No — requires an explicit IAM binding for each impersonation pair.

### B. TECHNICAL EXPLANATION
Impersonation requires explicit `roles/iam.serviceAccountTokenCreator` grant on the TARGET service account for the impersonating principal. This is not automatic or default. Each principal-SA pair requires a separate binding.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Expert security teams build impersonation into their CI/CD pipelines: the pipeline runner (e.g., GitHub Actions) impersonates a limited Terraform SA for infrastructure changes — never using a long-lived key.

### B. TECHNICAL EXPLANATION
Expert pattern: GitHub Actions → Workload Identity Federation → impersonate Terraform service account → apply infrastructure changes. This completely eliminates long-lived key files in CI/CD while maintaining precise access control. Audit trail in Cloud Logging shows exactly which pipeline runs applied which infrastructure changes with which identity.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Temporarily borrowing another service account's badge for specific tasks — short-lived, audited, requires explicit permission to borrow each badge.

### B. TECHNICAL SUMMARY
Service account impersonation allows a principal with `roles/iam.serviceAccountTokenCreator` to generate short-lived access tokens for a target SA. Used for testing permissions, delegating access in CI/CD, and Terraform workflows. All impersonation events are logged. Prefer over JSON key files for giving external systems access to SA permissions.
