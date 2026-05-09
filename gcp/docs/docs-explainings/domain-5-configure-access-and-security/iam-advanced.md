# Advanced IAM: Custom Roles, Conditions, Policy Troubleshooting — Dual-Layer Explanation

---

# Custom IAM Roles

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A tailor-made job description that combines specific duties from multiple departments. Instead of giving an employee an "IT Manager" role (too broad — they could access finance systems they don't need) or separate "Server Admin" and "Database Viewer" roles (too narrow — requires managing two assignments), you create a custom "App Team Lead" role with exactly the permissions required.

### B. TECHNICAL EXPLANATION
Custom IAM roles allow you to define a precise set of GCP permissions that don't map to any existing predefined role. You choose individual permissions (e.g., `compute.instances.get`, `storage.buckets.list`) to include, creating a role with exactly the access required — neither more nor less.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
You review the full job duties catalog (all available GCP permissions), select only the duties this employee actually needs, and file the custom job description either at the company level (org-level role) or department level (project-level role).

### B. TECHNICAL EXPLANATION
Custom roles are created at either **organization level** (usable in any project within the org) or **project level** (usable only within that project). Note: custom roles CANNOT be created at the **folder level** — this is a common exam trap. Constraints: cannot include permissions marked `NOT_SUPPORTED_IN_CUSTOM_ROLES`; cannot include permissions you don't already have (prevents privilege escalation). No automatic updates when GCP adds sub-permissions to a service.

---

## 3. MENTAL MODEL

### A. ANALOGY
Think of custom roles as "permission bundles you define." The tradeoff vs predefined roles: predefined roles update automatically as GCP adds features; custom roles require manual maintenance.

### B. TECHNICAL EXPLANATION
Custom role lifecycle stages: ALPHA → BETA → GA → DISABLED → DELETED. Disabled roles: existing bindings remain but cannot be used in new bindings. Deleted roles: soft-deleted and retained briefly before permanent removal. Unlike predefined roles, custom roles require manual updates when new GCP features add new permissions to a service — you must explicitly add those new permissions to your custom role.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
A read-only deployment audit role: "Can list VMs, view deployment configs, read GCS bucket contents — but cannot create, modify, or delete anything."

### B. TECHNICAL EXPLANATION
Create via: `gcloud iam roles create my-role --project=my-project --title="App Auditor" --permissions=compute.instances.list,compute.instances.get,storage.objects.list --stage=GA`. Copy from predefined and prune: `gcloud iam roles copy --source=roles/compute.admin --destination=roles/custom.computeViewer --dest-project=my-project`, then remove unneeded permissions. Apply to a principal: `gcloud projects add-iam-policy-binding PROJECT --member=user:user@domain.com --role=projects/PROJECT/roles/my-role`.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The job description is stored centrally (org or project level). When someone with that job title tries to access a resource, the system checks their title's permission list against the required permission for that action.

### B. TECHNICAL EXPLANATION
When a principal attempts an action, GCP IAM evaluates: (1) collect all IAM bindings for that principal across the resource hierarchy, (2) collect all permissions from all bound roles, (3) check if the required permission is in the collected permission set. Custom roles are resolved the same way as predefined roles — no performance difference. Policy inheritance: bindings on parent resources (organization, folder) are inherited by child resources (projects, resources).

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If the department-level job description includes duties that require company-level clearance, those duties are rejected — you can only assign duties you're authorized to grant.

### B. TECHNICAL EXPLANATION
You cannot create a custom role that includes permissions your account doesn't currently hold. This prevents privilege escalation via custom role creation. Also: `roles/owner`, `roles/editor`, `roles/viewer` (primitive roles) include broad permissions including `resourcemanager.projects.setIamPolicy` — avoid granting these; use specific predefined or custom roles instead.

---

## 7. TRADE-OFFS

### A. ANALOGY
Tailored job descriptions are perfect but require ongoing maintenance. Off-the-shelf job titles (predefined roles) are broader but self-updating when job duties expand.

### B. TECHNICAL EXPLANATION
Custom roles: precise least-privilege, no unwanted permissions. Maintenance burden: when GCP adds new sub-services, your custom role doesn't automatically include them — users may lose access to new features. Predefined roles: maintained by Google, automatically updated. Broader than necessary for some use cases. Best practice: use predefined roles where they fit the use case; create custom roles only when predefined roles are too broad.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"I can create a custom role at the folder level." No — only at organization or project level.

### B. TECHNICAL EXPLANATION
The most common exam trap: custom roles cannot be created at the **folder** level — only at organization or project. This distinction is consistently tested. Another misconception: custom roles update automatically like predefined roles. They do not — manual updates required when new permissions are needed.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Expert HR managers periodically review all custom job descriptions to ensure they still align with current duties — and retire ones that are no longer needed.

### B. TECHNICAL EXPLANATION
Expert practice: audit custom roles quarterly using `gcloud iam roles list --project=PROJECT`. Remove unused custom roles. Review whether predefined roles have been updated to cover the same permissions (allowing custom role retirement). Use IAM Recommender to identify over-granted permissions and rightsize bindings. Document the business justification for every custom role in a naming convention.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
A tailor-made permission bundle: precise least-privilege access, but requires manual updates as GCP adds features. Only available at org or project level — not folder level.

### B. TECHNICAL SUMMARY
Custom IAM roles define precise permission sets for use cases where predefined roles are too broad. They can be created at organization or project level only (not folder — common exam trap). They don't auto-update like predefined roles. Lifecycle: ALPHA → BETA → GA → DISABLED → DELETED.

---

---

# IAM Conditions

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A conditional access badge: "This employee can enter the server room ONLY during business hours (8am-6pm, Monday-Friday) AND only if they're in the office building (IP range)." The same badge, with time and location restrictions added.

### B. TECHNICAL EXPLANATION
IAM Conditions allow you to add attribute-based access constraints to IAM bindings. A condition is a CEL (Common Expression Language) expression evaluated at request time. If the condition evaluates to true, the binding applies; otherwise, it's ignored. Conditions can check: request time, resource tags, resource type, resource name prefix, and more.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
When the employee swipes their badge, the system doesn't just check "does this badge allow server room access?" — it also evaluates "is it currently 9am on a Tuesday?" AND "is the badge reader at the main building?". Only if all conditions pass does access succeed.

### B. TECHNICAL EXPLANATION
IAM Conditions are specified in the IAM policy binding alongside the `role` and `member`. At evaluation time, GCP's IAM service evaluates the CEL expression against request attributes (request time, resource tags). Condition components: resource type conditions (`resource.type == "compute.googleapis.com/Instance"`), request time conditions (`request.time < timestamp('2025-12-31T00:00:00Z')`), resource name conditions (`resource.name.startsWith('projects/my-project/regions/us-central1')`).

---

## 3. MENTAL MODEL

### A. ANALOGY
IAM Conditions turn static ("this person always has this access") into dynamic ("this person has this access IF these conditions are met").

### B. TECHNICAL EXPLANATION
Conditions enable: temporary access grants (expiry timestamp in condition), environment-based access (only resources tagged with `env=prod`), resource-specific scoping within a role that's broader than needed, and time-windowed access for incident response. Conditions apply at the binding level — each binding can have a different condition.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
Grant a contractor temporary access for 30 days, automatically expiring without manual revocation.

### B. TECHNICAL EXPLANATION
Time-bounded access: `request.time < timestamp("2025-06-30T00:00:00Z")`. Resource-tag based: `resource.matchTag('environment', 'prod')`. Resource name prefix: `resource.name.startsWith("projects/my-project/zones/us-central1-a/instances/db-")`. Grant via: `gcloud projects add-iam-policy-binding PROJECT --member=user:contractor@domain.com --role=roles/compute.viewer --condition='expression=request.time < timestamp("2025-06-30T00:00:00Z"),title=temp-access'`.

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The condition evaluation happens inside the badge reader (IAM engine), not on the employee's badge. The employee can't forge the timestamp or location.

### B. TECHNICAL EXPLANATION
CEL evaluation happens server-side in GCP's IAM service. Principals cannot influence condition evaluation — it uses request attributes from the API call context (authenticated request metadata). Not all GCP services support all condition attributes — check the IAM conditions reference for supported attributes per service. Some older GCP APIs may not support conditions.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If the badge reader can't check the time (unsupported condition), it denies the employee by default rather than ignoring the condition check.

### B. TECHNICAL EXPLANATION
If a service doesn't support IAM Conditions, requests using conditioned bindings are denied by default (fail-closed). Not all resources support resource tag conditions — verify service support before relying on tag-based conditions. Malformed CEL expressions cause binding creation to fail with validation errors.

---

## 7. TRADE-OFFS

### A. ANALOGY
Conditional badges add security granularity but increase policy complexity — the more conditions you add, the harder it is to reason about who has access to what.

### B. TECHNICAL EXPLANATION
Benefits: tighter least-privilege, time-bound access, environment isolation within a single role binding. Costs: increased IAM policy complexity, harder to audit effective access, CEL expression bugs can accidentally deny access. Use conditions judiciously — don't add conditions where simpler IAM structure (separate roles or projects per environment) would achieve the same result more clearly.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"Conditions make my role more powerful." No — conditions restrict when a role applies; they only narrow access, never expand it beyond the base role.

### B. TECHNICAL EXPLANATION
IAM Conditions only restrict when an existing binding applies — they cannot grant access that the base role doesn't include. A condition on `roles/compute.viewer` can only restrict which viewer access applies when; it cannot elevate the viewer to admin-level access.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Security experts use conditions for "just-in-time" access: a principal gets access only when needed, for exactly as long as needed, automatically expiring without manual cleanup.

### B. TECHNICAL EXPLANATION
Expert use case: just-in-time privileged access. When an incident requires elevated access, grant a role with a short-duration expiry condition (e.g., 4 hours). The condition automatically expires, requiring a new explicit grant for any subsequent need. This eliminates standing privileged access while maintaining operational flexibility. Automate with Cloud Functions that create time-bounded IAM bindings triggered by approved incidents.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Conditional access restrictions on IAM bindings — "this role applies only WHEN these conditions (time, resource tag, name prefix) are met."

### B. TECHNICAL SUMMARY
IAM Conditions add CEL-based constraints to IAM bindings, evaluated at request time. They enable time-bounded access, resource-tag-based scoping, and resource-name filtering. Conditions restrict (never expand) when a binding applies. Not all GCP services support all condition types; unsupported conditions fail closed.

---

---

# IAM Policy Troubleshooting

## 1. WHAT IS IT (INTUITIVE OVERVIEW)

### A. ANALOGY
A detective's toolkit for figuring out why an employee can or cannot get through a door. The tools tell you: which rules apply to this person, which rule blocked or allowed the entry, and which keys they have.

### B. TECHNICAL EXPLANATION
IAM policy troubleshooting involves: Policy Troubleshooter (explains why a principal has/lacks a specific permission on a specific resource), IAM Recommender (identifies over-granted permissions), `gcloud iam` commands for policy inspection, and understanding the resource hierarchy and policy inheritance model.

---

## 2. HOW IT WORKS (MECHANISM LEVEL)

### A. ANALOGY
The detective looks at the door's access log and traces which badge reader let the person in (or out). If blocked, it traces which rule blocked them and from where the rule originated.

### B. TECHNICAL EXPLANATION
IAM Policy Troubleshooter: Console → IAM → Policy Troubleshooter. Input: principal + resource + permission. Output: ALLOWED or DENIED, and the specific binding that granted or denied access, including inherited bindings from parent resources. Effective policy: union of all bindings at and above the resource in the hierarchy. Any ALLOW at any level grants access (no explicit DENY in GCP IAM, except for `deny policies` which is a separate newer feature).

---

## 3. MENTAL MODEL

### A. ANALOGY
IAM uses a "guilty until proven innocent" model for denials and "innocent until proven guilty" for allowances: if NO binding grants the permission anywhere in the hierarchy, access is denied. Any binding granting the permission at any level grants access.

### B. TECHNICAL EXPLANATION
GCP IAM (traditional): no explicit DENY concept. If no binding grants a permission, it's denied. Permissions are additive — you cannot remove a permission granted by a parent resource. The effective permission set = union of all permissions from all bindings at all levels (org, folder, project, resource). IAM Deny Policies (newer feature): explicit deny rules that override allow policies; useful for organization-wide restrictions.

---

## 4. PRACTICAL USAGE

### A. ANALOGY
"Why can the user list storage buckets but can't delete objects?" Run the troubleshooter: it shows that they have `storage.buckets.list` from a folder-level binding but lack `storage.objects.delete` — it was never granted.

### B. TECHNICAL EXPLANATION
Common troubleshooting steps:
1. Policy Troubleshooter (Console) → check for the specific resource + permission
2. `gcloud projects get-iam-policy PROJECT` → list all project-level bindings
3. Check parent resources: `gcloud organizations get-iam-policy ORG_ID`
4. Verify service account is active: `gcloud iam service-accounts describe SA_EMAIL`
5. Check group membership if the principal is a group member
6. Verify IAM conditions aren't blocking the request (conditional bindings)

---

## 5. UNDER THE HOOD (DEEP DIVE)

### A. ANALOGY
The hierarchy is like nested boxes. Rules on the outer box (org) apply to everything inside. Rules on the middle box (project) override nothing on the outer box — they add to it.

### B. TECHNICAL EXPLANATION
Policy inheritance: org policy → folder policy → project policy → resource policy. All are additive. `resourcemanager.folders.setIamPolicy` is required to set folder-level policies. Service accounts are both principals (can be granted permissions) and resources (can have permissions granted ON them). IAM audit logs (Cloud Logging) record every permission check evaluation — invaluable for security analysis.

---

## 6. EDGE CASES & FAILURE MODES

### A. ANALOGY
If the company merges with another and employees from the other company are given company-wide access at the building level, they automatically get access to every room in every building — including rooms that weren't intended.

### B. TECHNICAL EXPLANATION
Overly broad bindings at the organization or folder level can inadvertently grant unintended access to all descendant projects and resources. Use IAM Recommender regularly to identify principals with permissions they're not using (likely over-granted). The principle of least privilege requires regular auditing — access creep over time is the most common real-world IAM failure mode.

---

## 7. TRADE-OFFS

### A. ANALOGY
Granting access at the top level (org) is convenient and centralized but maximally permissive. Granting at the bottom level (individual resource) is maximally restrictive but extremely verbose and hard to manage at scale.

### B. TECHNICAL EXPLANATION
Coarse-grained access (org/folder level): easy to manage, hard to scope. Fine-grained access (project/resource level): precise, scales poorly. Best practice: grant at the lowest level that makes operational sense. For cross-project access: prefer separate service accounts with project-level grants over organization-level grants.

---

## 8. COMMON MISCONCEPTIONS

### A. ANALOGY
"I can revoke access at a lower level that was granted at a higher level." No — you can't remove an inherited permission without changing the parent binding.

### B. TECHNICAL EXPLANATION
If `roles/storage.admin` is granted at the org level, removing it at the project level does nothing — the org-level binding still applies. To revoke: modify the binding at the level where it was granted. This is why careful, minimal org/folder-level bindings are critical — they cannot be overridden by child resources.

---

## 9. EXPERT INSIGHT

### A. ANALOGY
Security architects draw an access map: every principal, every binding, every inherited permission. They treat IAM like code — in version control, reviewed, and audited regularly.

### B. TECHNICAL EXPLANATION
Expert IAM practices: manage all IAM bindings as Terraform code in Git (not manual console changes). Run IAM Recommender monthly and act on "over-granted" findings. Use Cloud Asset Inventory to export org-wide IAM policies for analysis. Set org-level IAM constraints to prevent binding primitive roles (owner/editor) at project level. Implement IAM audit logging and alert on sensitive permission grants.

---

## 10. TL;DR

### A. ANALOGY (1–2 lines)
Detective tools for access: the troubleshooter shows exactly which IAM binding grants or denies a specific permission; permissions are additive across the hierarchy with no overrides downward.

### B. TECHNICAL SUMMARY
IAM Policy Troubleshooter identifies why a principal has/lacks a permission by tracing bindings through the resource hierarchy. GCP IAM permissions are additive — child resources cannot override grants from parent resources. Regular auditing via IAM Recommender and Cloud Asset Inventory prevents permission accumulation over time.
