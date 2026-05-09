# Resource Hierarchy: Organizations, Folders, Projects

## Overview

Google Cloud resources are organized in a hierarchical structure: **Organization → Folders → Projects → Resources**. This hierarchy is not just organizational — it is the backbone of **policy inheritance**, **billing**, **ownership**, and **access control** across all GCP services. Understanding the hierarchy is foundational to every other domain.

---

## Key Concepts

### The Four Levels

| Level | Description | Who Creates It |
|-------|-------------|----------------|
| **Organization** | Root node, tied to a Google Workspace or Cloud Identity domain | Automatically created when domain is linked |
| **Folder** | Grouping mechanism within org; can nest folders up to 10 levels | Org admins |
| **Project** | Fundamental unit for enabling APIs, billing, and resource ownership | Any user with `resourcemanager.projects.create` |
| **Resource** | VMs, buckets, databases, etc. | Project owners/editors |

### Organization Node

- Automatically created when a Google Workspace or Cloud Identity account is linked to GCP
- All projects and folders in the org are children of the org node
- The organization node has a **numeric Organization ID** distinct from the domain name
- **Organization Admin** role (`roles/resourcemanager.organizationAdmin`) controls org-level settings
- Without an org, projects are created under a personal "no organization" context — these cannot use folders and have limited policy control

### Folders

- Used to represent departments, environments (prod/dev/staging), or teams
- Policies applied to a folder are **inherited by all child folders and projects**
- Maximum folder nesting depth: **10 levels**
- A project can have **only one parent** (folder or org — not both directly)
- Folders require an Organization to exist

### Projects

- Every GCP resource belongs to exactly one project
- Projects have:
  - A **Project Name** (human-readable, mutable)
  - A **Project ID** (globally unique, immutable after creation, chosen at creation time)
  - A **Project Number** (auto-assigned, immutable, numeric)
- APIs must be enabled per-project
- A project is linked to exactly **one billing account** (but a billing account can pay for many projects)
- Deleting a project begins a **30-day recovery window** before permanent deletion
- Default quota: **Up to 30 projects per billing account** (can be increased)

### Resource Hierarchy and Policy Inheritance

- IAM policies attach at any level: org, folder, project, or resource
- Policies **cascade down** — a policy granted at the org level applies to all folders, projects, and resources below
- **More permissive policy wins**: if a user has `viewer` at org level but `editor` at project level, the effective permission is `editor`
- **You cannot use an IAM policy to restrict permissions granted at a higher level** — only grant, never deny (except with Deny policies, a newer feature)
- Organization Policy constraints are **different from IAM** — they restrict what *can* be done regardless of IAM permissions

### Organization Policies

- `constraints/` namespace — restrict GCP service behaviors org-wide, folder-wide, or project-wide
- Common constraints:
  - `constraints/compute.disableSerialPortAccess` — blocks serial port on VMs
  - `constraints/compute.requireOsLogin` — enforces OS Login on all VMs
  - `constraints/iam.allowedPolicyMemberDomains` — restrict IAM to specific domains
  - `constraints/compute.restrictCloudNATUsage` — limit NAT usage
  - `constraints/gcp.resourceLocations` — restrict resource creation to specific regions
- Org policies are evaluated **hierarchically** and can be **overridden at lower levels** if allowed
- **Inheritance behavior**: `ALLOW` lists, `DENY` lists, or `restoreDefault` — lower nodes can narrow or replicate parent constraints

---

## When to Use

- **Folders** when you need to separate environments (prod vs. dev), teams, or business units with distinct policies or billing visibility
- **Multiple projects** to isolate network blast radius, separate billing, enforce API enablement boundaries, and maintain distinct IAM scopes
- **Organization Policies** when you need to enforce compliance constraints that IAM cannot achieve (e.g., preventing external sharing of data)
- **Project-level structure** when each application needs its own VPC, firewall rules, and resource lifecycle

---

## When NOT to Use

- Do **not** use a single project for all environments — it creates IAM complexity, blast radius risks, and billing opacity
- Do **not** create folders unnecessarily in small organizations — it adds management overhead without benefit
- Do **not** rely solely on IAM at the resource level without folder/project-level policies — hard to audit and enforce consistently
- Do **not** assume project deletion is instant — the 30-day pending deletion period means billing continues until permanent deletion

---

## Related Services / Concepts

- **IAM**: Policies attach at any hierarchy level — see [iam-overview.md](iam-overview.md) and [iam-advanced.md](../domain-5-configure-access-and-security/iam-advanced.md)
- **Billing Accounts**: Projects link to billing accounts — see [billing.md](billing.md)
- **Shared VPC**: Requires host and service projects in the same org — see [network-planning.md](../domain-2-plan-and-configure/network-planning.md)
- **Cloud Asset Inventory**: Queries resources across the entire hierarchy
- **Resource Manager API**: Used to programmatically manage projects, folders, orgs

---

## Exam-Relevant Notes

### Common Traps

1. **Project ID vs Project Number vs Project Name**: The exam distinguishes these. Project ID is the one you set, globally unique, immutable. Project Number is auto-assigned numeric. Project Name is display-only and mutable.

2. **Policy inheritance direction**: Policies only flow **downward**, never upward. A policy on a project does NOT affect sibling projects or the parent folder.

3. **"Cannot restrict inherited permissions"**: A critical rule — if `Owner` is granted at org level, you cannot use a project-level IAM policy to reduce that to `Viewer`. You would need Deny policies (newer feature) or restructure the hierarchy.

4. **Org policy vs IAM policy**: Org policies restrict *capabilities* of resources (e.g., "no VMs in us-east1"). IAM policies restrict *who can do what*. They are orthogonal.

5. **No org = no folders**: If you haven't linked a Google Workspace/Cloud Identity domain, you cannot create folders. Personal Gmail accounts create "no org" projects.

6. **30-day deletion window**: Deleted projects go into a pending deletion state. Resources inside stop incurring charges after deletion is initiated, but the project persists for 30 days for recovery. The exam may ask about this.

7. **Folder depth limit**: Maximum 10 levels of folder nesting — rarely tested but mentioned as a constraint.

8. **Up to 30 projects default**: This is a soft quota, not a hard limit, but the exam may reference it.

### Decision Tree: Where to Apply IAM

```
Need to apply to ALL resources in the company?
  → Apply at Organization level

Need to apply to a department or environment?
  → Create a Folder, apply at Folder level

Need to apply to a specific application/team?
  → Apply at Project level

Need fine-grained control on a specific resource?
  → Apply at Resource level (e.g., specific GCS bucket)
```

### Keywords
- Resource hierarchy, policy inheritance, organization node, Cloud Identity, Google Workspace, folder nesting, project ID, project number, org constraints, resource manager

---

## Source

- https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy
- https://cloud.google.com/resource-manager/docs/organization-policy/overview
- https://cloud.google.com/iam/docs/overview
