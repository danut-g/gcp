# Network Planning: VPC, Subnets, CIDR, Peering, Shared VPC

## Overview

**Virtual Private Cloud (VPC)** is the fundamental networking construct in GCP. Every GCP resource that needs network connectivity lives inside a VPC. Planning involves designing IP address spaces, understanding subnet scoping, choosing connectivity models, and planning for multi-project or hybrid architectures.

---

## Key Concepts

### VPC Architecture in GCP

GCP VPCs are **global** resources — a single VPC spans all regions. This is a fundamental difference from AWS, where VPCs are regional.

| Dimension | GCP VPC | AWS VPC |
|-----------|---------|---------|
| Scope | **Global** | Regional |
| Subnets | Regional (within global VPC) | AZ-specific (within regional VPC) |
| Routing | Global by default | Requires explicit route tables per subnet |

#### Default VPC

- Every new project comes with a **default VPC** pre-created with:
  - Subnets in every region with non-overlapping RFC 1918 CIDR ranges
  - A default firewall rule allowing ingress from other instances in the same VPC
  - A default firewall rule allowing SSH (port 22) and RDP (port 3389) from `0.0.0.0/0`
- **Exam important**: The default VPC's permissive firewall rules make it unsuitable for production without modification
- Best practice: Create custom VPCs instead of modifying the default VPC

---

### Subnets

- Subnets are **regional** resources within a global VPC
- A subnet is defined by a **primary CIDR range** (IPv4)
- Subnets can have **secondary IP ranges** for GKE (Pods and Services)
- Subnet CIDR ranges must be within:
  - RFC 1918 private address space (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
  - Publicly routable addresses are supported but not recommended for private resources
- No CIDR overlap between subnets within the same VPC (GCP enforces this)
- Subnet mode:
  - **Auto mode**: Automatically creates one subnet per region with pre-defined ranges (/20 for each region); default VPC uses auto mode
  - **Custom mode**: You define all subnets explicitly; recommended for production

#### Subnet Expansion

- You can expand a subnet's primary CIDR range (make it larger) but **cannot shrink it**
- Cannot overlap with other subnets in the VPC
- Expansion is a non-disruptive operation

#### Reserved IPs in Each Subnet

Each subnet reserves 4 IP addresses:
- x.x.x.0 — Network address
- x.x.x.1 — Default gateway
- x.x.x.2 — Reserved by Google (DNS)
- x.x.x.3 — Reserved by Google
- x.x.x.255 — Broadcast (reserved, not used)

A /29 subnet has 8 IPs, only 4 usable for VMs.

---

### CIDR Planning

Key planning considerations:
- Choose non-overlapping CIDR ranges across VPCs if you plan to use VPC Peering (peered VPCs cannot have overlapping ranges)
- Reserve address space for GKE secondary ranges (Pods need large ranges — a /14 provides ~64k Pod IPs)
- RFC 1918 ranges to use:
  - `10.0.0.0/8` — largest space, commonly split by environment/region
  - `172.16.0.0/12` — medium, useful for smaller environments
  - `192.168.0.0/16` — small, good for on-premises overlap avoidance

---

### Firewall Rules

- Stateful packet filtering at the VPC level (not subnet or VM level)
- Applied to **VM instances** based on network tags or service accounts
- Direction: Ingress (inbound) or Egress (outbound)
- Priority: 0–65535 (lower = higher priority); default deny rules are at 65534 (deny all ingress) and 65535 (allow all egress)
- Action: Allow or Deny
- Targets: All instances in VPC, or specific instances by **network tag** or **service account**

#### Default Firewall Rules (Default VPC Only)

- `default-allow-internal`: Allow all traffic between instances in the VPC
- `default-allow-ssh`: Allow TCP:22 from 0.0.0.0/0 (any internet IP)
- `default-allow-rdp`: Allow TCP:3389 from 0.0.0.0/0
- `default-allow-icmp`: Allow ICMP from 0.0.0.0/0

#### Implied Firewall Rules (All VPCs)

- Implied deny all ingress (lowest priority)
- Implied allow all egress (lowest priority)
- These exist even if no rules are defined; cannot be deleted but can be overridden

See [vpc-security.md](../domain-5-configure-access-and-security/vpc-security.md) for advanced firewall configurations.

---

### VPC Peering

- Connects two VPC networks so resources can communicate using internal IPs
- Works across projects and even across organizations
- Peering is **non-transitive**: VPC A peered with VPC B, and VPC B peered with VPC C does NOT mean A and C can communicate
- **CIDR ranges cannot overlap** between peered VPCs
- No bandwidth limits beyond what the VM/link can handle
- No charge for peering itself; traffic charges apply for cross-region traffic
- Each side must create a peering connection (both peering connections must be established for traffic to flow)

---

### Shared VPC

- Allows multiple projects (service projects) to use subnets from a single host project's VPC
- Host project: owns the VPC and subnets
- Service projects: deploy resources (VMs, GKE clusters) into the host project's subnets
- Shared VPC requires an **Organization** (cannot use with personal accounts)
- Permissions: `roles/compute.networkUser` on the shared subnet (granted to service project's service accounts and users)

#### Shared VPC vs VPC Peering

| Aspect | Shared VPC | VPC Peering |
|--------|-----------|------------|
| Connectivity model | One VPC, multiple projects use it | Two separate VPCs with peering link |
| Management | Centralized (host project owns network) | Distributed (each project manages its VPC) |
| Transitive routing | N/A (same VPC) | Non-transitive |
| IP range overlap | Not possible (same VPC) | Not allowed |
| Organization required | Yes | No |
| Use case | Centralized network governance | Connect separate teams/orgs |

---

### Regions and Zones

- **Region**: Geographic area (e.g., `us-central1`, `europe-west1`)
  - ~40+ regions globally
  - Within a region, data and resources stay in that geographic boundary
- **Zone**: Isolated location within a region (e.g., `us-central1-a`, `us-central1-b`)
  - At least 3 zones per region
  - Zones are independent failure domains (separate power, cooling, network)
  - Resources in different zones are insulated from each other's failures

#### Availability Patterns

- **Single zone**: Simple, low cost; no HA
- **Multi-zone**: Replicate across zones in a region; protects against zone failure
- **Multi-region**: Replicate across regions; protects against regional disasters; highest cost

---

### Cloud NAT

- Allows VMs **without external IPs** to initiate outbound connections to the internet
- NAT is not bidirectional — external sources cannot initiate connections to NAT-ed VMs
- Regional service: created per region
- Configurable: NAT all primary/secondary ranges, or specific subnets
- See [managing-networking.md](../domain-4-ensure-success/managing-networking.md) for operational details

---

### Private Google Access

- Allows VMs without external IPs to reach Google APIs and services (storage.googleapis.com, etc.)
- Enabled per subnet
- When enabled: VMs in that subnet can reach Google services via internal routing (not through the internet)
- Required for GKE private cluster nodes to pull container images from Artifact Registry

---

### Alias IP Ranges

- Allows assigning multiple IP addresses (additional CIDRs) to a single VM interface
- Used for: containers running on a VM, services needing multiple IPs
- GKE VPC-native clusters use alias IPs for Pod IP addresses

---

## When to Use

- **Custom mode VPC** for any production environment — control your IP space
- **Shared VPC** when you need centralized network governance across multiple projects in an organization
- **VPC Peering** to connect networks across teams, projects, or organizations without centralization
- **Cloud NAT** for any private VMs (no external IP) that need outbound internet access
- **Private Google Access** enabled on all private subnets — required for private VMs to access GCP APIs

---

## When NOT to Use

- **Default VPC** in production — permissive default rules are a security risk
- **Auto-mode VPC** if you plan to VPC peer (pre-defined ranges may conflict)
- **VPC Peering** when you need transitive routing (use Cloud VPN or Interconnect instead)
- **Shared VPC** without an organization (requires org)

---

## Related Services / Concepts

- **Networking Deploy**: VPC creation, firewall rules, load balancing — see [networking-deploy.md](../domain-3-deploy-and-implement/networking-deploy.md)
- **Managing Networking**: Cloud NAT, VPN, Interconnect, routing — see [managing-networking.md](../domain-4-ensure-success/managing-networking.md)
- **VPC Security**: Firewall policies, VPC Service Controls — see [vpc-security.md](../domain-5-configure-access-and-security/vpc-security.md)
- **GKE Planning**: VPC-native cluster networking — see [gke-planning.md](gke-planning.md)

---

## Exam-Relevant Notes

### Common Traps

1. **GCP VPCs are global, subnets are regional**: This is opposite to AWS. A single GCP VPC spans all regions. Subnets within it are regional.

2. **VPC Peering is non-transitive**: A–B peered and B–C peered does NOT allow A–C communication. This is tested repeatedly.

3. **Auto-mode VPC overlap risk**: Auto-mode VPCs use predefined CIDR ranges that might conflict with on-premises networks or other VPCs you want to peer with. Use custom mode for production.

4. **Shared VPC requires org**: Cannot create Shared VPC without an organization. If a question describes isolated personal projects or no-org environments, Shared VPC isn't applicable.

5. **Firewall rules are at VPC level, not subnet level**: Unlike AWS Security Groups (per ENI) or NACLs (per subnet), GCP firewall rules apply to VMs based on tags/service accounts — not to subnets.

6. **Reserved IPs per subnet**: Remember the 4 reserved IPs (network, gateway, 2 Google reserved). A /29 = 8 total − 4 reserved = 4 usable.

7. **Private Google Access vs Cloud NAT**: Private Google Access allows reaching *Google APIs* specifically. Cloud NAT allows reaching *any internet destination*. They are separate features and may both be needed.

8. **Subnet expansion is permanent (one direction)**: You can expand a subnet but not shrink it. Plan CIDR ranges with growth in mind.

### Keywords
- Global VPC, regional subnet, auto-mode vs custom-mode, VPC peering non-transitive, Shared VPC host project, service project, network tag, firewall rule priority, Cloud NAT, Private Google Access, alias IP, RFC 1918, CIDR overlap

---

## Source

- https://cloud.google.com/vpc/docs/overview
- https://cloud.google.com/vpc/docs/shared-vpc
- https://cloud.google.com/vpc/docs/vpc-peering
- https://cloud.google.com/nat/docs/overview
