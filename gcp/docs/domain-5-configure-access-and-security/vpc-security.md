# VPC Security: Firewall Rules, VPC Service Controls, Private Google Access

## Overview

VPC security encompasses the controls that restrict network access to GCP resources: firewall rules that filter traffic at the VM level, VPC Service Controls that create security perimeters around sensitive services, and Private Google Access that allows private VMs to reach Google APIs securely. These controls are complementary and often used together.

---

## Key Concepts

### Firewall Rules — Advanced

See [networking-deploy.md](../domain-3-deploy-and-implement/networking-deploy.md) for basic firewall configuration. This section covers advanced patterns.

#### Firewall Rules vs Firewall Policies

| Aspect | VPC Firewall Rules | Hierarchical Firewall Policies | Network Firewall Policies |
|--------|-------------------|-------------------------------|--------------------------|
| Location | Attached to VPC | Attached to org or folder | Attached to VPC/network |
| Priority | Within VPC | Evaluated before VPC rules | Evaluated with VPC rules |
| Targets | Tags or SAs | All instances in scope | Tags or SAs |
| Use case | VPC-specific rules | Org-wide/folder-wide baseline | Modern alternative to VPC rules |
| Rule action | Allow or Deny | Allow, Deny, or goto_next | Allow, Deny, or goto_next |

#### Hierarchical Firewall Policies (HFP)

- Created at org or folder level
- Enforced **before** VPC-level firewall rules
- Rules can have action `goto_next` — delegates decision to lower-level policy or VPC rules
- Allows org security team to enforce baseline rules (e.g., always block SSH from internet) without individual VPC admins being able to override
- VPC-level rules are then applied after the hierarchy is satisfied

#### Network Firewall Policies (NFP)

- Attached to VPC networks (not instances)
- Newer replacement for legacy VPC firewall rules
- Support `goto_next` like HFPs
- Can be global (attached to global network) or regional
- Ordered evaluation with VPC firewall rules using priority values

#### Firewall Rules Logging

- Enable per firewall rule
- Logs metadata: connection direction, allowed/denied, rule name, source/destination, protocol/port
- Available in Cloud Logging
- Useful for security auditing but can generate high volume

#### Firewall Rule Best Practices

- Use **deny-all ingress** as baseline (already implied), then explicitly allow needed traffic
- Use **service account targets** instead of network tags for production (more secure)
- Minimize use of `0.0.0.0/0` source ranges
- Use **specific port ranges** instead of opening all TCP
- Enable logging for critical deny rules and all allow rules to internet-facing services

---

### VPC Service Controls (VPC SC)

VPC Service Controls create **security perimeters** around GCP services to protect against data exfiltration.

#### Core Concept

- VPC SC defines a perimeter around a set of projects and services
- Resources inside the perimeter can communicate with each other freely
- Resources outside the perimeter (or API calls coming from outside) are denied
- Prevents: Data exfiltration via cloud services (e.g., copying data from a Cloud Storage bucket to an external project's bucket)

#### Perimeter Types

| Type | Description |
|------|-------------|
| **Regular perimeter** | Hard boundary; requests from outside are denied |
| **Bridge perimeter** | Allows two regular perimeters to communicate with each other |

#### Access Policies

- VPC SC is configured through an **Access Policy** at the org level
- One policy per org; all perimeters are defined within it
- `accesscontextmanager.policyAdmin` role required to manage

#### Access Levels

- Define conditions under which requests from outside the perimeter are allowed
- Conditions: IP ranges, device policy (Endpoint Verification), user identity
- Example access level: "Allow access from corporate IP ranges AND from compliant devices"

#### Service Restrictions

- Specify which services are protected within the perimeter
- Supported services: Cloud Storage, BigQuery, Bigtable, Spanner, Pub/Sub, Cloud Key Management, etc. (100+ services)
- When a service is in the perimeter, API calls to that service must come from inside the perimeter (or via an access level)

#### Dry-Run Mode

- Create perimeters in "dry-run" mode first
- Violations are logged but not enforced
- Review logs to understand what would be blocked before enforcing
- Transition to enforced mode after validating

#### VPC SC Use Cases

- Preventing data exfiltration: Employee cannot copy project data to personal GCS bucket
- Regulatory compliance: Data stays within defined project boundaries
- Protecting against insider threats: Even `roles/storage.admin` cannot copy data outside the perimeter

#### VPC SC vs IAM

| IAM | VPC Service Controls |
|-----|---------------------|
| Controls **who** can do what | Controls **from where** operations are permitted |
| Principal-based | Network/context-based |
| Always active | Perimeter enforcement |
| Example: Alice can read GCS | Example: GCS reads must come from within the perimeter |

Both must allow the operation for access to succeed.

---

### Private Google Access

Enables VMs **without external IP addresses** to reach Google APIs and services (Cloud Storage, BigQuery, Pub/Sub, etc.) without going through the internet.

#### Configuration

- Enabled **per subnet** (`privateIpGoogleAccess: true`)
- When enabled, traffic to Google APIs routes through Google's backbone (169.254.x.x range and Google's internal routing)
- VM uses its internal IP; Google API endpoint resolves to a special restricted address space

#### Private Google Access for On-Premises

- Allows on-premises hosts (connected via VPN or Interconnect) to access Google APIs privately
- Configure: DNS forwarding zone for `googleapis.com` to point to the restricted VIP (`199.36.153.4/30`)
- Route `199.36.153.4/30` through the VPN/Interconnect tunnel
- On-premises hosts can then reach Google APIs without using the public internet

#### Private Service Access (Cloud SQL Private IP, etc.)

- Separate from Private Google Access
- Configures a VPC peering connection between your VPC and Google's service producer VPC
- Required for: Cloud SQL private IP, Memorystore, Cloud Run Direct VPC, GKE Private Cluster
- Prerequisites: Allocate an IP range in your VPC for the service, configure the private connection

---

### Identity-Aware Proxy (IAP)

- Controls access to applications deployed on GCP based on user identity
- Works with: Compute Engine, GKE, App Engine, Cloud Run (via Load Balancer)
- Forces authentication even for web apps that don't implement their own auth
- Replaces VPN for protecting internal web apps accessible from the internet
- Access decision: User must be authenticated with Google AND have `roles/iap.httpsResourceAccessor` on the resource
- BeyondCorp Enterprise integration: Adds device posture to access decisions

---

### OS Login

- Manages SSH access to VMs using IAM instead of SSH keys in metadata
- When enabled, SSH access requires:
  - `roles/compute.osLogin` — user can SSH
  - `roles/compute.osAdminLogin` — user can SSH with sudo access
- Works with VMs across all projects in the org when enforced via org policy
- SSH keys are automatically generated; no manual key management
- All SSH sessions are tracked in Audit Logs

#### OS Login vs SSH Key Metadata

| Aspect | OS Login | SSH Key in Metadata |
|--------|----------|---------------------|
| Key management | Automatic via IAM | Manual |
| Revocation | Remove IAM role | Remove key from metadata |
| Audit trail | Full IAM + OS Login logs | No automatic log |
| Multi-project | Yes (org-wide) | Per-project or per-VM |
| Best practice | Yes | Legacy approach |

---

## When to Use

- **Hierarchical Firewall Policies**: For org-wide security baselines (e.g., block RDP from internet across all projects)
- **Network Firewall Policies**: For VPC-specific rules in a modern, centrally-managed way
- **VPC Service Controls**: For high-security environments where data exfiltration is a primary concern (financial, healthcare, government)
- **Private Google Access**: Always on private subnets — allows private VMs to reach Google APIs
- **Private Service Access**: For Cloud SQL private IP and similar managed services
- **IAP**: To protect internal web apps without requiring a VPN
- **OS Login**: Enforce for all production VMs via org policy

---

## When NOT to Use

- **VPC Service Controls in enforced mode without dry-run**: Risk of blocking legitimate traffic; always test in dry-run first
- **Hierarchical Firewall Policy `deny all` without exceptions**: Will block ALL traffic including legitimate operations; carefully plan `goto_next` rules
- **IAP as the only security control**: IAP adds authentication but doesn't replace encryption or network segmentation

---

## Related Services / Concepts

- **Network Planning**: VPC design principles — see [network-planning.md](../domain-2-plan-and-configure/network-planning.md)
- **Networking Deploy**: Basic firewall rule creation — see [networking-deploy.md](../domain-3-deploy-and-implement/networking-deploy.md)
- **IAM Advanced**: Who can configure VPC SC — see [iam-advanced.md](iam-advanced.md)
- **Data Security**: CMEK protects data; VPC SC protects access paths — see [data-security.md](data-security.md)

---

## Exam-Relevant Notes

### Common Traps

1. **VPC SC is about data exfiltration, not just access**: VPC Service Controls don't just restrict who can access data — they restrict the network context from which access is allowed. Even an authorized user cannot access data from outside the perimeter.

2. **IAM + VPC SC are additive**: Both must allow the operation. Having IAM permission to read GCS but being outside the VPC SC perimeter = access denied.

3. **Private Google Access ≠ Cloud NAT**: Private Google Access only reaches **Google APIs and services**. Cloud NAT reaches **any internet destination**. Different use cases; often both are needed.

4. **VPC SC dry-run first**: Never enforce a new perimeter without validating in dry-run mode. Mistakes in perimeter configuration can block access to critical services.

5. **OS Login with org policy**: You can enforce OS Login across ALL VMs in an organization using `constraints/compute.requireOsLogin`. This is the recommended way to ensure SSH key management is centralized.

6. **Hierarchical policies + `goto_next`**: If an HFP rule has action `goto_next`, it passes the decision to the next level (another policy or the VPC firewall rules). This is how you allow VPC-level customization while still having org-level baselines.

7. **Private Service Access prerequisites**: Before you can give Cloud SQL a private IP, you must allocate an IP range in your VPC and configure the service networking peering connection. Forgetting this step is common.

8. **IAP requires Load Balancer**: For GCE and GKE, IAP works through the Google Cloud Load Balancer. You cannot use IAP without an HTTPS LB in front.

### Keywords
- Hierarchical Firewall Policy, Network Firewall Policy, VPC Service Controls, access policy, access level, dry-run mode, security perimeter, data exfiltration, Private Google Access, Private Service Access, OS Login, Identity-Aware Proxy, `goto_next`, perimeter bridge

---

## Source

- https://cloud.google.com/vpc/docs/firewalls
- https://cloud.google.com/vpc-service-controls/docs/overview
- https://cloud.google.com/vpc/docs/configure-private-google-access
- https://cloud.google.com/iap/docs/concepts-overview
