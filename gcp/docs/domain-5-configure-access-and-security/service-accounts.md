# Service Accounts: Keys, Workload Identity, Impersonation

## Overview

Service accounts are the primary non-human identity in GCP. They allow applications, VMs, and services to authenticate to Google Cloud APIs. Proper management of service accounts — including avoiding key files, using Workload Identity, and controlling impersonation — is critical for GCP security and a key exam topic.

---

## Key Concepts

### Service Account Fundamentals

- A service account is both a **principal** (can be granted IAM roles) and a **resource** (has its own IAM policy)
- Email format: `NAME@PROJECT_ID.iam.gserviceaccount.com`
- Service accounts do not have passwords — they authenticate using:
  - **OAuth 2.0 tokens** generated from cryptographic keys
  - Short-lived credentials from the metadata server (when running on GCP)
  - Workload Identity Federation (for external workloads)

#### Creating Service Accounts

- Create in Console, gcloud, or Terraform
- Name (prefix): Must be 6–30 characters, start with a letter, lowercase letters/digits/hyphens
- Description: Document what this SA is for (important for maintenance)
- Grant roles to the SA after creation (NOT during — avoid combining creation + permission in one step)

#### Default Service Accounts (Avoid in Production)

| Default SA | Auto-Created For | Default Role |
|------------|-----------------|-------------|
| `PROJECT_NUMBER-compute@developer.gserviceaccount.com` | Compute Engine | `roles/editor` |
| `PROJECT_ID@appspot.gserviceaccount.com` | App Engine | `roles/editor` |

- Both have `roles/editor` — **extremely broad permissions**
- Google recommends disabling the default SA's auto-grant of editor and using custom SAs
- New GCP projects may have a setting to disable automatic default SA creation on VM/App Engine

---

### Service Account Keys

#### Types of Keys

| Type | Description | Management |
|------|-------------|-----------|
| **Google-managed keys** | Used for metadata server tokens; no key files | Automatic, secure |
| **User-managed keys** | JSON key files downloaded by the user | Manual; security risk |

#### User-Managed Keys (JSON Key Files)

- The JSON file contains the private key — if leaked, attackers have full access to the SA's permissions
- Key rotation: Manually create new key, update configuration, delete old key
- Keys don't expire automatically — they remain valid until deleted
- Maximum 10 keys per service account
- Keys can be **disabled** (temporarily stop them from being used) or **deleted**

#### Security Risks of Key Files

- Key files can be accidentally committed to git repositories
- Key files in container images are often exposed (docker history, etc.)
- Stolen key files grant access until the key is deleted (no expiry)
- **Best practice: Avoid service account key files entirely when possible**

#### When Key Files May Be Necessary

- Accessing GCP from an environment that doesn't support Workload Identity Federation (very rare today)
- Local development (though `gcloud auth application-default login` is preferred)
- Legacy systems that cannot use any other auth method

---

### Application Default Credentials (ADC)

- The mechanism by which application libraries find credentials automatically
- Search order:
  1. `GOOGLE_APPLICATION_CREDENTIALS` environment variable (JSON key file path)
  2. gcloud user credentials (`gcloud auth application-default login`)
  3. Attached service account (from GCE/GKE metadata server or Cloud Run/Cloud Functions)
  4. GCP-provided default service account

- On Compute Engine, GKE, Cloud Run, Cloud Functions: ADC automatically uses the attached service account's credentials via the metadata server — **no key file needed**

---

### Workload Identity (GKE)

The recommended approach for GKE Pods to authenticate to Google Cloud without key files.

#### Setup Summary

1. Enable Workload Identity on the cluster (`--workload-pool=PROJECT_ID.svc.id.goog`)
2. Create a GCP Service Account (GSA)
3. Grant the GSA appropriate IAM permissions
4. Annotate a Kubernetes Service Account (KSA) with the GSA email
5. Create IAM binding: KSA → GSA with `roles/iam.workloadIdentityUser`
6. Use the KSA in Pod spec (`serviceAccountName`)

#### Workload Identity Pool Member Format

```
serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]
```

This is the `--member` format used in the IAM binding command.

#### Troubleshooting Workload Identity

- 403 errors: Missing IAM binding or missing KSA annotation
- Wrong namespace: Binding is namespace-scoped; using KSA from wrong namespace won't work
- Test with: `gcloud auth print-access-token` inside the Pod (using the metadata server)

---

### Workload Identity Federation (External Workloads)

For workloads outside GCP (GitHub Actions, GitLab CI, AWS, Azure, on-premises) that need GCP access without service account key files.

#### How It Works

1. Configure a **Workload Identity Pool** and **Provider** in GCP
2. The external system's identity token (GitHub OIDC token, AWS STS token, etc.) is presented to GCP
3. GCP validates the token against the configured identity provider
4. GCP issues short-lived credentials for a GCP service account
5. No key files needed at rest

#### Supported Identity Providers

- **GitHub Actions**: Uses GitHub's OIDC token
- **GitLab CI**: Uses GitLab's OIDC token
- **AWS**: Uses AWS STS AssumeRoleWithWebIdentity
- **Azure**: Uses Azure AD tokens
- **On-premises**: Any OIDC-compliant provider
- **Kubernetes**: Any K8s cluster with OIDC issuer

#### Benefits

- No service account key files in CI/CD systems
- Credentials are short-lived (1 hour by default)
- Attacker stealing a leaked CI token cannot reuse it after it expires
- Audit trail: All access logged in Cloud Audit Logs with the external identity details

---

### Service Account Impersonation

Allows a human user or service account to **act as another service account** without possessing its key.

#### How It Works

- Grant `roles/iam.serviceAccountTokenCreator` on the target SA to the principal that needs to impersonate
- The principal then calls `generateAccessToken` or `generateIdToken` on the SA to get a short-lived token
- Use cases:
  - DevOps engineers impersonating the production SA to test what it can access
  - Terraform running as a human user account but creating resources as a dedicated SA
  - Service-to-service calls that need a different identity

#### gcloud Impersonation

```
gcloud compute instances list --impersonate-service-account=target-sa@project.iam.gserviceaccount.com
```

- Requires `roles/iam.serviceAccountTokenCreator` on `target-sa`
- Creates an impersonation chain that appears in Audit Logs

#### Impersonation Chains

- Can chain: User → SA A → SA B (each needs `serviceAccountTokenCreator` on the next)
- Chains are limited in depth
- All steps are logged in Cloud Audit Logs

---

### Service Account Best Practices

1. **One service account per workload/function**: Don't share SAs between unrelated services
2. **Least privilege**: Grant only the specific roles the SA needs; review quarterly
3. **No key files when running on GCP**: Use metadata server credentials
4. **No key files in CI/CD**: Use Workload Identity Federation
5. **Disable unused SAs**: Disabled SAs cannot generate tokens; enable them only when needed
6. **Delete keys, not just SAs**: Deleting an SA doesn't automatically invalidate in-use tokens immediately; delete keys first
7. **Monitor SA activity**: Cloud Audit Logs record all API calls made by service accounts
8. **Never embed SAs in code**: Don't put SA emails or key contents in source code

---

## When to Use

| Scenario | Recommended Approach |
|----------|---------------------|
| VM accessing GCP APIs | Attach dedicated SA to VM; use metadata server (no key) |
| GKE Pod accessing GCP APIs | Workload Identity |
| Cloud Run / Cloud Functions | Attach dedicated SA; metadata server (automatic) |
| GitHub Actions CI/CD | Workload Identity Federation |
| Local development | `gcloud auth application-default login` |
| Admin testing target SA access | Service account impersonation |
| Legacy on-premises system | Workload Identity Federation (if OIDC supported) or key file (last resort) |

---

## When NOT to Use

- **Key files for GCP workloads**: Never — metadata server provides automatic credentials
- **Shared service accounts**: One SA serving multiple unrelated applications makes blast radius larger
- **Default Compute/App Engine SA**: Has `editor` role — too broad for production
- **More than 10 keys per SA**: Maximum is 10; having this many suggests over-reliance on key files

---

## Related Services / Concepts

- **IAM Overview**: Service account basics — see [iam-overview.md](../domain-1-setup-and-configure/iam-overview.md)
- **IAM Advanced**: Conditions and deny policies that apply to SAs — see [iam-advanced.md](iam-advanced.md)
- **Managing GKE**: Workload Identity for GKE — see [managing-gke.md](../domain-4-ensure-success/managing-gke.md)
- **Data Security**: Secret Manager (alternative to putting creds in code) — see [data-security.md](data-security.md)

---

## Exam-Relevant Notes

### Common Traps

1. **GCE SA + access scopes**: Even if the SA has broad IAM permissions, the access scope on the VM instance can restrict what APIs the VM can actually call. Both must allow the operation.

2. **Key file deletion ≠ immediate invalidation**: Existing short-lived tokens derived from the key remain valid until they expire (typically 1 hour). For immediate revocation, disable the key first, then delete it.

3. **Workload Identity requires exact namespace/KSA match**: The IAM binding must use the exact Kubernetes namespace and service account name. A typo in the namespace or SA name breaks authentication.

4. **`serviceAccountTokenCreator` vs `serviceAccountUser`**:
   - `serviceAccountTokenCreator`: Can generate tokens; used for impersonation
   - `serviceAccountUser`: Can run workloads (VMs, Cloud Functions) as the SA; does NOT allow token generation
   Both are commonly confused.

5. **Default SAs have Editor role**: New VMs and App Engine use the default SA. In production, always specify a custom SA with minimal permissions.

6. **Workload Identity Federation credentials are short-lived**: Default 1-hour expiry. This is a security feature — leaked credentials expire quickly.

7. **Max 10 keys per SA**: Exceeding this isn't common but if you're rotating keys and not deleting old ones, you'll hit this limit.

### Keywords
- Service account, key file, metadata server, ADC, Workload Identity, GKE KSA, GSA, workload-pool, Workload Identity Federation, OIDC, `serviceAccountTokenCreator`, `serviceAccountUser`, impersonation, access scope, default compute SA

---

## Source

- https://cloud.google.com/iam/docs/service-account-overview
- https://cloud.google.com/iam/docs/workload-identity-federation
- https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity
- https://cloud.google.com/iam/docs/impersonating-service-accounts
