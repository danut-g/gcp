# IAM Overview: Roles, Policies, and Service Accounts

## Overview

**Cloud Identity and Access Management (IAM)** controls who can do what on which GCP resources. IAM answers the question: "Is this principal allowed to perform this action on this resource?" It is the central authorization mechanism across all GCP services, and understanding it is critical for both administration and security.

---

## Key Concepts

### The IAM Model

IAM policies express: **Who** (principal) can do **what** (role/permissions) on **which** resource (resource).

#### Principals (Identity Types)

| Principal Type | Format | Description |
|---------------|--------|-------------|
| Google Account | `user:email@gmail.com` | Individual Google or Workspace user |
| Service Account | `serviceAccount:sa@project.iam.gserviceaccount.com` | Non-human identity for apps/services |
| Google Group | `group:devs@company.com` | Collection of users/service accounts |
| Google Workspace domain | `domain:company.com` | All users in a Workspace domain |
| Cloud Identity domain | `domain:company.com` | All Cloud Identity users |
| `allUsers` | Special | Anyone, including unauthenticated |
| `allAuthenticatedUsers` | Special | Any authenticated Google account |

- **`allUsers` and `allAuthenticatedUsers`** are disabled by default for most resources if `constraints/iam.allowedPolicyMemberDomains` org policy is set

#### Permissions

- Fine-grained operations expressed as `service.resource.verb` (e.g., `compute.instances.start`)
- Permissions are **never directly assigned to principals** — they are bundled into roles
- Permissions can only be granted, not explicitly denied (standard IAM — Deny policies are a separate newer feature)

#### Roles

Three categories of roles:

**1. Primitive (Basic) Roles** — Legacy, coarse-grained, project-level only

| Role | Permissions |
|------|-------------|
| `roles/viewer` | Read-only access to all services |
| `roles/editor` | Viewer + create/modify most resources |
| `roles/owner` | Editor + manage IAM + billing |

- Avoid in production — too broad. `Editor` can create service accounts, modify most resources.

**2. Predefined Roles** — Service-specific, least-privilege, Google-managed

- Examples: `roles/compute.instanceAdmin.v1`, `roles/storage.objectViewer`, `roles/bigquery.dataEditor`
- Each service has a set of predefined roles with specific permission sets
- Google maintains and updates these as new features are added
- Always prefer predefined over primitive roles

**3. Custom Roles** — User-defined, permission-by-permission

- Created at org or project level
- Can combine any permissions from any service
- Must be explicitly maintained — no automatic updates from Google
- Cannot include permissions marked as `NOT_SUPPORTED_IN_CUSTOM_ROLES`
- See [iam-advanced.md](../domain-5-configure-access-and-security/iam-advanced.md) for details

---

### IAM Policies

- An IAM **policy** is a collection of **bindings**
- Each **binding** maps a **role** to a list of **principals** (members)
- Policies are attached to resources (org, folder, project, or individual resource)
- Policies are evaluated at request time — effective permissions = union of all bindings from all levels in the hierarchy
- Policies are **additive** — no deny semantics (except Deny policies)

#### Policy Structure (JSON representation)

```json
{
  "bindings": [
    {
      "role": "roles/compute.instanceAdmin.v1",
      "members": [
        "user:alice@company.com",
        "serviceAccount:deploy-sa@my-project.iam.gserviceaccount.com"
      ]
    },
    {
      "role": "roles/storage.objectViewer",
      "members": ["group:data-readers@company.com"]
    }
  ],
  "etag": "BwXXXXXX",
  "version": 3
}
```

- **etag**: Used for optimistic concurrency control — prevents race conditions when updating policies
- **version**: Policy schema version. Version 3 is required for conditional bindings

---

### Service Accounts — Overview

A **service account** is a special identity used by applications, VMs, and services — not humans.

#### Key Properties

- Email format: `NAME@PROJECT_ID.iam.gserviceaccount.com`
- Can be both a **principal** (in IAM policies: it's being granted access) AND a **resource** (IAM policies can control who can use or impersonate it)
- Service accounts authenticate using:
  - **Short-lived tokens** (from metadata server — preferred)
  - **Service account key files** (long-lived JSON keys — security risk, avoid when possible)

#### Types of Service Accounts

| Type | Description |
|------|-------------|
| **User-managed** | Created explicitly by users for workloads |
| **Default service accounts** | Auto-created for App Engine, Compute Engine; include editor role by default |
| **Google-managed** | Used internally by GCP services; usually not directly managed |

#### Default Service Accounts — Important Warning

- **Compute Engine default SA**: `PROJECT_NUMBER-compute@developer.gserviceaccount.com` — auto-created with `roles/editor` (very broad!)
- **App Engine default SA**: `PROJECT_ID@appspot.gserviceaccount.com` — auto-created with `roles/editor`
- Best practice: Create dedicated service accounts with minimal permissions instead of using defaults

#### Service Account Scopes (Legacy)

- Access scopes on Compute Engine VMs act as a **secondary authorization layer** on top of IAM
- Even if a SA has broad IAM permissions, scopes can restrict what the VM can actually do
- Recommended: Use `cloud-platform` scope + restrict via IAM policies rather than managing individual scopes
- Scopes are a Compute Engine-specific legacy concept; not used in GKE/Cloud Run

For advanced service account usage (Workload Identity, key rotation, impersonation), see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md).

---

### IAM Conditions

- Allow conditional bindings based on attributes: resource type, resource name, request time, IP address
- Require policy version 3
- Example: Grant `storage.objectViewer` only for objects with `resource.name.startsWith("projects/_/buckets/public-")`
- See [iam-advanced.md](../domain-5-configure-access-and-security/iam-advanced.md) for details

---

### Deny Policies (newer feature)

- A separate IAM feature (distinct from allow policies)
- Allow explicit DENY of specific permissions, even if an allow policy would grant them
- Evaluated **before** allow policies — deny always wins
- Useful for enforcing boundaries (e.g., "no one can delete production databases, even owners")
- Applied at org, folder, or project level

---

## When to Use

- **Predefined roles**: Always prefer over primitive roles for least-privilege
- **Service accounts**: For any non-human workload (VMs, Cloud Functions, Cloud Run, pipelines)
- **Groups**: Grant IAM roles to groups rather than individual users — simplifies management
- **IAM Conditions**: When access needs to be time-bound or resource-scoped
- **Custom roles**: When predefined roles are too broad and no predefined role matches exactly

---

## When NOT to Use

- Do **not** use `roles/editor` or `roles/owner` for service accounts in production
- Do **not** use `allUsers` or `allAuthenticatedUsers` unless intentionally making resources public
- Do **not** grant IAM at the resource level if the same can be achieved at project/folder level — harder to audit
- Do **not** use service account key files when running on GCP infrastructure (use metadata server/Workload Identity instead)

---

## Related Services / Concepts

- **Resource Hierarchy**: Policies are inherited through hierarchy — see [projects-and-org.md](projects-and-org.md)
- **Advanced IAM**: Custom roles, conditions, deny policies — see [iam-advanced.md](../domain-5-configure-access-and-security/iam-advanced.md)
- **Service Accounts (advanced)**: Keys, Workload Identity, impersonation — see [service-accounts.md](../domain-5-configure-access-and-security/service-accounts.md)
- **VPC Service Controls**: Network-level access restrictions layered on top of IAM — see [vpc-security.md](../domain-5-configure-access-and-security/vpc-security.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Primitive roles are project-scoped only**: `roles/owner`, `roles/editor`, `roles/viewer` can only be applied at project level (and above). They don't exist for individual resources.

2. **IAM is additive only (standard)**: Permissions accumulate. A user with `viewer` at org AND `owner` at project level has `owner` permissions on that project. You cannot "subtract" permissions with standard IAM.

3. **Service accounts as both principal and resource**: A service account `sa@project.iam.gserviceaccount.com` can be in an IAM binding as the identity being granted a role (principal) OR have an IAM policy on itself controlling who can impersonate it (resource).

4. **Default compute SA has Editor role**: This is a major security anti-pattern that the exam tests. VMs using the default SA inherit `editor` permissions on the entire project.

5. **Access scopes ≠ IAM permissions**: Both must allow the operation. Access scopes can restrict even if IAM allows. The recommended pattern is `cloud-platform` scope + tight IAM.

6. **Policy version 3 for conditions**: If you add an IAM condition to a policy, you MUST use version 3. Attempting to update a version 1 policy with conditions will fail.

7. **etag for concurrent policy updates**: When updating IAM policies programmatically, always read-then-write using the etag. Without the etag, concurrent updates can overwrite each other.

8. **Groups vs individual users**: Exam often asks about best practices — always grant roles to groups rather than individuals for maintainability.

### Role Selection Decision Tree

```
What access is needed?
├── Read-only access to everything in a project?
│     → roles/viewer (primitive — acceptable for view-only)
├── Specific service operation?
│     → Find the predefined role: roles/<service>.<function>
├── Cross-service permissions needed, no predefined fits?
│     → Create a custom role
└── App/automation needs GCP access?
      → Create a dedicated service account
      → Grant minimum necessary predefined role
      → Avoid default SAs
```

### Keywords
- IAM binding, principal, role, permission, policy inheritance, predefined role, primitive role, service account, access scopes, etag, policy version 3, IAM conditions, deny policy, allUsers

---

## Source

- https://cloud.google.com/iam/docs/overview
- https://cloud.google.com/iam/docs/understanding-roles
- https://cloud.google.com/iam/docs/service-accounts
- https://cloud.google.com/iam/docs/deny-overview
