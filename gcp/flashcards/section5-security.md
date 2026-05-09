# Section 5 — IAM & Service Accounts Flashcards

---
### Q1: What three questions does IAM answer?
**A:** (1) **Who** is making the request? (principal/member), (2) **What** are they allowed to do? (role containing permissions), and (3) **On which resource** does the permission apply? (project, folder, org, or specific resource).

---
### Q2: What is the IAM policy formula?
**A:** WHO (Principal) + WHAT (Role/Permissions) + WHERE (Resource) = IAM Policy. Policies are collections of bindings that map roles to members on a resource.

---
### Q3: What are the four basic (primitive) IAM roles?
**A:** **Viewer** (read-only), **Editor** (read-write, no IAM management), **Owner** (full access including IAM and billing), and **Browser** (browse project hierarchy). Basic roles are very broad and not recommended for production.

---
### Q4: What is the difference between the Editor and Owner basic roles?
**A:** Editor grants read-write access to most resources but cannot manage IAM policies or billing. Owner includes everything Editor has plus IAM policy management and billing management.

---
### Q5: True or False: A more restrictive child IAM policy can override a permissive parent policy.
**A:** **False.** IAM allow policies are additive. Permissions are unioned across the hierarchy. A child cannot reduce permissions granted at a parent level. You would need a **deny policy** to explicitly block permissions.

---
### Q6: What is a predefined role?
**A:** A Google-managed role with fine-grained, service-specific permissions (e.g., `roles/compute.instanceAdmin.v1`, `roles/storage.objectViewer`). Google recommends predefined roles over basic roles for following the principle of least privilege.

---
### Q7: What gcloud command lists all the permissions included in a predefined role?
**A:** `gcloud iam roles describe roles/ROLE_NAME` — for example, `gcloud iam roles describe roles/storage.objectViewer --format="value(includedPermissions)"`.

---
### Q8: When should you create a custom IAM role?
**A:** When predefined roles are too broad and grant more permissions than needed. Custom roles let you select exactly the permissions required, following the principle of least privilege.

---
### Q9: What gcloud command creates a custom role at the project level?
**A:** `gcloud iam roles create ROLE_ID --project=PROJECT_ID --title="Title" --permissions=perm1,perm2 --stage=GA`. You can also use `--file=role-definition.yaml` instead of inline permissions.

---
### Q10: At which levels can custom roles be created?
**A:** Custom roles can only be created at the **project** level or the **organization** level. They **cannot** be created at the folder level.

---
### Q11: What are the four launch stages for a custom role?
**A:** **ALPHA** (early testing), **BETA** (tested but may change), **GA** (generally available and stable), and **DISABLED** (role cannot be granted). Set with the `--stage` flag.

---
### Q12: True or False: Deleting a custom role permanently removes it immediately.
**A:** **False.** Deleting a custom role is a soft delete. It can be **undeleted within 7 days** using `gcloud iam roles undelete ROLE_ID --project=PROJECT_ID`.

---
### Q13: What is a conditional IAM binding?
**A:** A role binding that includes a CEL (Common Expression Language) condition. The role is only effective when the condition evaluates to true, for example based on time (`request.time`), resource name, or resource tags.

---
### Q14: A contractor needs temporary Compute Engine access that expires on December 31. How do you configure this?
**A:** Use a conditional IAM binding: `gcloud projects add-iam-policy-binding PROJECT_ID --member="user:contractor@example.com" --role="roles/compute.instanceAdmin.v1" --condition='expression=request.time < timestamp("2025-12-31T00:00:00Z"),title=temp-access'`.

---
### Q15: What is an IAM deny policy and how does it differ from a regular allow policy?
**A:** A deny policy explicitly **prevents** principals from using specific permissions, even if those permissions are granted through allow policies. Allow policies are additive and can only grant access; deny policies can block access regardless of other bindings.

---
### Q16: What gcloud command creates a deny policy?
**A:** `gcloud iam policies create POLICY_NAME --attachment-point="cloudresourcemanager.googleapis.com/projects/PROJECT_ID" --kind=denypolicies --policy-file=deny-policy.json`.

---
### Q17: What is the purpose of the etag field in an IAM policy?
**A:** The etag provides **concurrency control**. It prevents race conditions when multiple users update the same policy simultaneously. If the etag has changed since you fetched the policy, the update will fail, forcing you to re-fetch and retry.

---
### Q18: What gcloud command checks which IAM roles a specific user has on a project?
**A:** `gcloud projects get-iam-policy PROJECT_ID --flatten="bindings[].members" --filter="bindings.members:user@example.com" --format="table(bindings.role)"`.

---
### Q19: What is a service account?
**A:** A special type of Google account used by applications, VMs, and services (not humans) to authenticate and authorize API calls. It authenticates using keys or metadata, not passwords, and has the format `SA_NAME@PROJECT_ID.iam.gserviceaccount.com`.

---
### Q20: What are the three types of service accounts in GCP?
**A:** (1) **User-managed** — created by you, (2) **Default** — created automatically by Google when you enable certain APIs (e.g., Compute Engine default SA), and (3) **Google-managed** — internal service agents created and managed by Google.

---
### Q21: True or False: The Compute Engine default service account follows the principle of least privilege.
**A:** **False.** The default Compute Engine SA is granted `roles/editor`, which is far too broad. Best practice is to create a **dedicated service account** with only the minimal permissions required.

---
### Q22: What gcloud command creates a new service account?
**A:** `gcloud iam service-accounts create SA_NAME --display-name="Display Name" --description="Description"`. The resulting email will be `SA_NAME@PROJECT_ID.iam.gserviceaccount.com`.

---
### Q23: What is the maximum number of user-managed service accounts per project?
**A:** **100** user-managed service accounts per project. Service account names must be 6-30 characters, lowercase, with hyphens allowed.

---
### Q24: What is the difference between granting roles TO a service account vs. ON a service account?
**A:** Granting roles **TO** a SA (via `gcloud projects add-iam-policy-binding`) defines what the SA can do. Granting roles **ON** a SA (via `gcloud iam service-accounts add-iam-policy-binding`) defines who can use, manage, or impersonate that SA.

---
### Q25: What is the difference between roles/iam.serviceAccountUser and roles/iam.serviceAccountTokenCreator?
**A:** `serviceAccountUser` allows **attaching** the SA to resources (VMs, Cloud Run, etc.). `serviceAccountTokenCreator` allows **creating tokens as the SA** (impersonation). You need User for deploying resources with an SA, and TokenCreator for impersonation.

---
### Q26: What gcloud command assigns a service account to a new Compute Engine VM?
**A:** `gcloud compute instances create VM_NAME --zone=ZONE --service-account=SA_EMAIL --scopes=cloud-platform`. The `--scopes=cloud-platform` flag is recommended so that access is controlled entirely through IAM roles.

---
### Q27: True or False: You can change the service account on a running Compute Engine VM without stopping it.
**A:** **False.** You must **stop** the VM first, then use `gcloud compute instances set-service-account` to change the SA, and then start the VM again.

---
### Q28: What is the relationship between scopes and IAM roles on a VM?
**A:** Effective permissions are the **intersection** of the scope and the IAM role. A scope can limit what the IAM role allows but cannot grant more than the IAM role provides. Best practice is to use `--scopes=cloud-platform` and control everything via IAM roles.

---
### Q29: A service account has roles/editor but the VM scope is storage-read-only. Can the VM write to Cloud Storage?
**A:** **No.** Effective permissions = scope intersection IAM role. The storage-read-only scope restricts access even though the IAM role would allow writes. This is why `--scopes=cloud-platform` is recommended.

---
### Q30: What is service account impersonation?
**A:** A user or service account temporarily **acts as** another service account without needing its keys. It requires `roles/iam.serviceAccountTokenCreator` on the target SA and produces short-lived, auditable tokens.

---
### Q31: What gcloud command enables impersonation for a single command?
**A:** Add `--impersonate-service-account=SA_EMAIL` to any gcloud command. For global impersonation across all commands: `gcloud config set auth/impersonate_service_account SA_EMAIL`. Remove with `gcloud config unset auth/impersonate_service_account`.

---
### Q32: Name three advantages of service account impersonation over service account keys.
**A:** (1) **No key management** — no files to rotate or risk exposing, (2) **Audit trail** — all impersonation is logged, and (3) **Time-limited** — tokens expire automatically, unlike long-lived keys.

---
### Q33: What are the types of short-lived service account credentials?
**A:** **Access token** (API calls, 1h max), **ID token** (identity verification, 1h max), **Self-signed JWT** (Google API access, 1h max), **Signed blob** (sign data), and **OIDC token** (authenticate to Cloud Run, etc., 1h max).

---
### Q34: What gcloud command generates a short-lived access token via impersonation?
**A:** `gcloud auth print-access-token --impersonate-service-account=SA_EMAIL`. The resulting token can be used in API calls with `Authorization: Bearer TOKEN`.

---
### Q35: What is Workload Identity Federation?
**A:** A mechanism that allows external identities (from AWS, Azure, GitHub Actions, etc.) to access Google Cloud resources **without service account keys**. It uses a workload identity pool and an OIDC/SAML provider to exchange external tokens for short-lived GCP credentials.

---
### Q36: A GitHub Actions workflow needs to deploy to GCP without storing service account keys. What should you use?
**A:** **Workload Identity Federation.** Create a workload identity pool with a GitHub OIDC provider, then grant `roles/iam.workloadIdentityUser` on a Google Cloud service account to the pool's principal set representing the GitHub repo.

---
### Q37: What is the order of preference for authenticating a service account (most secure to least)?
**A:** (1) **Attached SA / metadata** (VMs, Cloud Run, Functions), (2) **Workload Identity** (GKE), (3) **Workload Identity Federation** (external workloads), (4) **Impersonation** (human users), (5) **SA keys** (last resort, long-lived, must be rotated).

---
### Q38: What organization policy constraint prevents service account key creation?
**A:** `constraints/iam.disableServiceAccountKeyCreation` — when enforced at the org level, it prevents anyone in the organization from creating new service account keys.

---
### Q39: What is the IAM Recommender and what does it do?
**A:** IAM Recommender analyzes actual permission usage and identifies **over-provisioned access**. It suggests removing unused roles or replacing broad roles with narrower ones. Query with: `gcloud recommender recommendations list --project=PROJECT_ID --recommender=google.iam.policy.Recommender --location=global`.

---
### Q40: A company has 15 developers who all need identical GKE access. What is the most efficient way to manage this?
**A:** Create a **Google Group** (e.g., `gke-devs@example.com`), add all developers to it, and grant `roles/container.developer` to the group at the project level. This is easier to manage, audit, and revoke than individual bindings.

---
### Q41: What is the `roles/iam.workloadIdentityUser` role and when is it used?
**A:** It allows a Kubernetes Service Account (K8s SA) to impersonate a GCP Service Account (GCP SA). It is the key binding in Workload Identity: you grant this role on the GCP SA to the member `serviceAccount:PROJECT.svc.id.goog[NAMESPACE/K8S_SA]`.

---
### Q42: What is the difference between Workload Identity (GKE) and Workload Identity Federation?
**A:** **Workload Identity for GKE** maps K8s SAs to GCP SAs for pods running inside GKE. **Workload Identity Federation** maps external identities (GitHub Actions, AWS, Azure, on-prem) to GCP SAs — no GCP service account keys needed for CI/CD or cross-cloud access.

---
### Q43: What does Cloud NGFW's `goto_next` action do in a hierarchical firewall policy rule?
**A:** It means "do not make a final allow/deny decision at this level — pass the packet evaluation down to the next policy level." This lets org-level policies define baseline rules while still allowing projects/VPCs to add their own rules.

---
### Q44: A VM is being created with a Secure Tag assigned. A malicious VM admin tries to self-assign a different Secure Tag to escalate privileges. Why can't they?
**A:** Secure Tags require `roles/resourcemanager.tagUser` on the tag key to create bindings. VM admins do not have this IAM permission by default. Unlike network tags (stored as VM metadata anyone with compute.instances.setMetadata can change), Secure Tag bindings are IAM-governed resource operations.

---
