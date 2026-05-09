# Advanced IAM: Custom Roles, Conditions, Policy Troubleshooting, Best Practices

## Overview

Advanced IAM covers the nuances of role design, conditional access, policy debugging, and architectural patterns for least-privilege access. This topic builds on the fundamentals in [iam-overview.md](../domain-1-setup-and-configure/iam-overview.md) and is heavily tested in the ACE exam.

---

## Key Concepts

### Custom Roles

#### When to Create Custom Roles

- Predefined roles are too broad for your use case
- No single predefined role contains exactly the permissions you need
- You need to combine permissions from multiple services

#### Custom Role Lifecycle

| Stage | Description |
|-------|-------------|
| **ALPHA** | Testing; not generally available |
| **BETA** | Available but may change |
| **GA** | Stable; generally available |
| **DISABLED** | Temporarily disabled; cannot be used in new bindings |
| **DELETED** | Marked for deletion; cannot be used or recovered |

#### Custom Role Constraints

- Can be created at **org level** or **project level** only (not folder level — a common exam trap)
- Org-level custom roles: Usable in any project within the org
- Project-level custom roles: Only usable within that project
- Cannot include **permissions marked `NOT_SUPPORTED_IN_CUSTOM_ROLES`** — some very sensitive permissions
- Cannot include permissions your account doesn't already have (prevents privilege escalation via custom roles)
- **No automatic updates**: If Google adds new sub-permissions to a service, you must manually update custom roles to include them; predefined roles update automatically

#### Custom Role Best Practices

- Name custom roles with a clear, descriptive name indicating the scope and purpose
- Document the role's purpose and the permissions it includes
- Review custom roles quarterly — permissions may become deprecated or new ones needed
- Prefer predefined roles when they fit; use custom roles only when necessary

---

### IAM Conditions

IAM conditions add attribute-based access control to IAM bindings.

#### Condition Attributes

| Category | Attributes | Examples |
|----------|-----------|---------|
| **Resource** | `resource.type`, `resource.name`, `resource.service` | Only GCS buckets, specific project |
| **Request** | `request.time` | Business hours only |
| **IP** | `request.auth.access_levels` | Specific IP ranges (via Access Context Manager) |

#### Common Condition Use Cases

- **Time-based access**: Grant access only during business hours or a specific date range
  ```
  request.time.getHours("America/New_York") >= 9 &&
  request.time.getHours("America/New_York") < 18
  ```
- **Resource-based restriction**: Grant storage access only to objects with a specific prefix
  ```
  resource.name.startsWith("projects/_/buckets/my-bucket/objects/team-a/")
  ```
- **Temporary access**: Grant access that expires at a specific time
  ```
  request.time < timestamp("2025-12-31T23:59:59Z")
  ```

#### Condition Limitations

- Conditions require **policy version 3**
- Cannot use conditions with primitive roles (`roles/owner`, `roles/editor`, `roles/viewer`)
- Not all resource types support all condition attributes
- Access Context Manager integration requires org-level configuration

---

### Deny Policies

A newer, separate IAM feature (distinct from allow policies).

#### Key Characteristics

- **Deny always wins over allow**: Deny policies are evaluated before any allow policy
- Applied at org, folder, or project level
- Deny specific permissions to specific principals
- Even `roles/owner` cannot override a Deny policy (unlike standard IAM which is additive)
- Use cases:
  - Prevent anyone (including admins) from deleting production resources
  - Enforce that specific operations always require MFA (via conditions)
  - Override inherited permissions without changing the inheritance

#### Deny Policy vs Org Policy

| Aspect | IAM Deny Policy | Org Policy Constraint |
|--------|----------------|----------------------|
| Controls | WHO can perform actions | WHAT can be done regardless of WHO |
| Scope | Specific principals | All users/service accounts |
| Focus | Permission-level | Resource/feature-level |
| Example | "Alice cannot delete instances" | "No instances in us-east1 region" |

---

### Policy Troubleshooting

#### Common Tools

**Policy Troubleshooter:**
- Console: IAM → Policy troubleshooter
- Answers: "Can principal X perform action Y on resource Z?"
- Shows exactly which binding allowed/denied and why
- Tests hypothetical permissions without actually granting them

**Policy Analyzer:**
- More powerful than troubleshooter
- Answers: "Who has access to X resource?" or "What resources can principal Y access?"
- Supports complex queries across an entire org
- Useful for security audits

#### Common IAM Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| "Permission denied" despite having role | Role granted at wrong level, or condition not met | Check effective permissions with troubleshooter |
| Role doesn't include expected permission | Using wrong predefined role, or permission was removed | Check role's permission list in console |
| Service account can't call API | Missing IAM role on SA, or access scope restricts it | Check both IAM binding AND access scopes |
| Condition not working | Policy version < 3, or condition syntax error | Verify version 3 in policy; test condition expression |
| Inherited role can't be overridden | IAM is additive; inherited permissions cannot be removed | Use Deny policies or restructure hierarchy |

#### Policy Propagation Delay

- IAM policy changes propagate across GCP systems in **up to 60 seconds** for most cases
- Some operations (especially Org Policy changes) can take several minutes
- Don't test immediately after a policy change; allow propagation time

---

### Best Practices for Production IAM

#### Least Privilege Patterns

1. **Start with no permissions**: Don't grant access until needed
2. **Use service-specific predefined roles**: `roles/compute.instanceAdmin.v1` over `roles/editor`
3. **Grant at the narrowest scope**: Resource-level > Project > Folder > Org
4. **Audit regularly**: Use Cloud Audit Logs (Admin Activity) to see who changed what
5. **Use groups**: Grant roles to Google Groups; add/remove users from groups instead of modifying IAM

#### Separation of Duties

- No single person should have both `roles/iam.securityAdmin` and `roles/owner` in production
- Separate: those who can modify IAM from those who manage resources
- Cloud Audit Logs capture all IAM changes for forensic analysis

#### Access Review

- Quarterly review of IAM bindings using Policy Analyzer
- Remove users who no longer need access (especially contractors, departed employees)
- Alert on new `roles/owner` or `roles/editor` bindings via Log Alerts

#### Service Account Security

- Never download service account key files unless absolutely necessary
- Rotate keys every 90 days if key files are used
- Prefer Workload Identity for GCP workloads (see [service-accounts.md](service-accounts.md))
- Delete unused service accounts

---

### Organization-Level IAM Roles

| Role | Description | Sensitive Level |
|------|-------------|----------------|
| `roles/resourcemanager.organizationAdmin` | Full org administration | Very High |
| `roles/resourcemanager.folderAdmin` | Create/manage folders | High |
| `roles/billing.admin` | Full billing account admin | High |
| `roles/iam.organizationRoleAdmin` | Create/edit org-level custom roles | High |
| `roles/securitycenter.admin` | Security Command Center admin | High |
| `roles/accesscontextmanager.policyAdmin` | Manage Access Context Manager | High |

- Roles with prefix `resourcemanager.organizationAdmin` are among the most powerful in GCP — treat like root
- These should be granted to a small number of break-glass accounts, not regular users

---

## When to Use

- **Custom roles**: When predefined roles are too permissive and your security policy requires fine-grained control
- **IAM conditions**: For time-limited access (contractors), resource-scoped access (specific bucket prefixes), or IP-restricted access
- **Deny policies**: To enforce hard boundaries that even admins cannot override
- **Policy Analyzer**: Quarterly access reviews and compliance audits
- **Policy Troubleshooter**: When diagnosing "permission denied" errors

---

## When NOT to Use

- **Project-level custom roles for org-wide use**: Create org-level custom roles instead; project-level roles can't be used in other projects
- **Custom roles when a predefined role fits**: Maintenance overhead isn't worth it for a small permission tweak
- **Conditions on primitive roles**: Not supported; conditions only work with predefined and custom roles

---

## Related Services / Concepts

- **IAM Overview**: Basics of IAM — see [iam-overview.md](../domain-1-setup-and-configure/iam-overview.md)
- **Service Accounts**: Workload identity, key management — see [service-accounts.md](service-accounts.md)
- **VPC Service Controls**: Network-layer access restrictions — see [vpc-security.md](vpc-security.md)
- **Cloud Logging**: IAM audit logs — see [logging.md](../domain-4-ensure-success/logging.md)

---

## Exam-Relevant Notes

### Common Traps

1. **Custom roles cannot be at folder level**: Custom roles can only be created at org or project level. If you need a custom role applicable to multiple projects in a folder, create it at the org level.

2. **Custom roles require manual updates**: When Google adds new sub-permissions to a service, custom roles don't auto-update. This can cause breakage as new features require new permissions.

3. **Deny policies override even Owner**: This is a fundamental behavior change. If there's a Deny policy, even `roles/owner` at the same or higher level cannot override it.

4. **Conditions require version 3**: Adding a condition to a policy binding requires setting `"version": 3` in the policy. Older policies (version 1 or 2) will fail if you try to add conditions.

5. **Conditions cannot be used with primitive roles**: `roles/owner`, `roles/editor`, `roles/viewer` don't support IAM conditions.

6. **IAM propagation delay**: Policy changes take up to 60 seconds. Testing immediately after a change may give misleading results.

7. **Policy Troubleshooter vs Policy Analyzer**: Troubleshooter answers "Can Alice do X?" (specific principal + action). Analyzer answers "Who can access resource Y?" (broader audit query). Different tools for different questions.

8. **`iam.securityAdmin` is NOT the same as `owner`**: `securityAdmin` can modify IAM policies but cannot necessarily manage resources. `owner` can manage resources AND IAM. Both are very powerful.

### Keywords
- Custom role stages, ALPHA/BETA/GA/DISABLED/DELETED, org-level custom role, project-level custom role, IAM condition, Deny policy, Policy Troubleshooter, Policy Analyzer, version 3 policy, least privilege, separation of duties, propagation delay

---

## Source

- https://cloud.google.com/iam/docs/creating-custom-roles
- https://cloud.google.com/iam/docs/conditions-overview
- https://cloud.google.com/iam/docs/deny-overview
- https://cloud.google.com/policy-intelligence/docs/policy-analyzer-overview
