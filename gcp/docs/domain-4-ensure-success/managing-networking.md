# Managing Networking: Cloud NAT, VPN, Interconnect, Routing

## Overview

Managing GCP networking involves configuring outbound internet access for private instances (Cloud NAT), connecting on-premises environments (Cloud VPN, Cloud Interconnect), and managing routing within VPCs. Understanding when to use each connectivity option is a key ACE exam skill.

---

## Key Concepts

### Cloud NAT (Network Address Translation)

#### What It Does

- Allows VM instances (and GKE nodes) **without external IPs** to initiate outbound connections to the internet
- NAT is **one-directional**: Outbound only — external hosts cannot initiate connections to NAT-ted instances
- Regional service: Create one Cloud NAT gateway per region

#### How It Works

- Cloud NAT gateway is associated with a Cloud Router in the same region
- Traffic from private VMs is SNATed (source NAT) to the NAT gateway's external IP(s)
- GCP manages the NAT gateway — no VMs to maintain
- External IPs can be auto-assigned or you can specify reserved static external IPs

#### Cloud NAT Configuration Options

| Option | Description |
|--------|-------------|
| **NAT IP allocation** | Automatic (GCP manages) or Manual (specify static IPs) |
| **Subnet scope** | All subnets in region, or specific subnets |
| **Port allocation** | Static vs dynamic port allocation per VM |
| **Min ports per VM** | Minimum ports allocated; affects how many connections each VM can make |
| **Max ports per VM** | Upper limit on port allocation (avoid SNAT port exhaustion) |

#### SNAT Port Exhaustion

- Each NAT IP supports ~64,000 ports; used for source IP:port combinations
- With many VMs or many connections per VM, ports can be exhausted
- Symptoms: Connection failures, timeouts
- Fix: Add more NAT IPs, increase max ports per VM, use dynamic port allocation

---

### Cloud VPN

Connects your on-premises network (or another VPC) to a GCP VPC over an encrypted IPsec tunnel over the public internet.

#### VPN Types

| Type | Features | Use Case |
|------|----------|---------|
| **Classic VPN** | Static routing or dynamic routing; single tunnel; 99.9% SLA | Simple connectivity, lower requirements |
| **HA VPN** | Two tunnels across two interfaces; 99.99% SLA; dynamic routing required (BGP) | Production HA connectivity |

#### HA VPN Architecture

- Two VPN gateway interfaces, each with its own external IP
- Two tunnels (one per interface) to on-premises VPN gateway
- Both tunnels active simultaneously (active/active) for load distribution and redundancy
- Requires BGP (dynamic routing) via Cloud Router

#### Cloud Router

- Manages BGP sessions for dynamic routing
- Propagates routes between GCP VPC and on-premises network
- Advertises GCP subnet routes to on-premises; learns on-premises routes from BGP
- Required for HA VPN; also used with Cloud Interconnect
- Regional resource; works with routes in that region's VPC

#### VPN Throughput

- Maximum per tunnel: ~3 Gbps for most configurations
- To increase throughput: Use multiple tunnels in parallel (ECMP routing via BGP)

---

### Cloud Interconnect

Provides a dedicated physical connection between your on-premises network and GCP. Higher bandwidth, lower latency, and more consistent performance than VPN (which goes over the public internet).

#### Interconnect Types

| Type | Connection | Bandwidth | Use Case |
|------|-----------|-----------|---------|
| **Dedicated Interconnect** | Direct physical connection to Google's network | 10 Gbps or 100 Gbps per link (up to 8 links = 200 Gbps or 800 Gbps) | Highest performance, lowest latency |
| **Partner Interconnect** | Through a service provider | 50 Mbps to 50 Gbps | When you can't reach a colocation facility |

#### Dedicated Interconnect Details

- Requires your network equipment to be physically present at a Google colocation facility (or at a partner facility)
- Establishes BGP sessions via Cloud Router
- Traffic stays on Google's network — does NOT traverse the public internet
- Better reliability, bandwidth, and lower latency than Cloud VPN
- 99.99% SLA requires 2 circuits in 2 metro areas + 4 VLAN attachments

#### VLAN Attachments (Interconnect Attachments)

- Each Dedicated/Partner Interconnect circuit can have multiple VLAN attachments
- Each VLAN attachment connects to a VPC via Cloud Router
- VLAN attachments can be configured for different VPCs or different on-premises networks

---

### Comparison: Cloud VPN vs Cloud Interconnect

| Dimension | Cloud VPN | Dedicated Interconnect | Partner Interconnect |
|-----------|-----------|----------------------|---------------------|
| Connection type | Encrypted tunnel over internet | Direct physical | Through service provider |
| Max bandwidth | ~3 Gbps/tunnel | Up to 800 Gbps | 50 Mbps–50 Gbps |
| Latency | Variable (internet-dependent) | Low, consistent | Variable (through provider) |
| Encryption | Yes (IPsec) | No (by default; use MACsec for encryption) | No (by default) |
| Setup time | Fast (hours) | Weeks–months | Weeks |
| Cost | Lower | Higher | Medium |
| SLA (best HA config) | 99.99% (HA VPN) | 99.99% | 99.99% |
| Use case | Smaller bandwidth, quick setup | Large-scale enterprise hybrid | Enterprise, can't reach colo |

---

### Routing in GCP

#### Route Types

| Type | Created By | Description |
|------|-----------|-------------|
| **Subnet route** | Automatically | Routes for each subnet's CIDR; cannot be deleted while subnet exists |
| **Static route** | Admin | Manually defined; fixed next-hop |
| **Dynamic route** | Cloud Router (BGP) | Learned via BGP from on-premises/VPN/Interconnect |
| **Default route** | Automatically | `0.0.0.0/0` route to default internet gateway |

#### Custom Static Routes

- Next-hops can be: VM instance, IP address, VPN tunnel, load balancer internal IP, VPC peering
- Use for: Routing traffic through a VM appliance (firewall/NAT), directing specific traffic to VPN tunnels

#### Policy-Based Routing (PBR)

- Route traffic based on source IP or other attributes (not just destination)
- Available on some configurations; more complex setup

#### Route Priority

- When multiple routes match the same destination, the one with the **lowest metric** (or highest priority) is used
- More specific routes (longer prefix) always win over less specific

---

### Cloud DNS Operations

#### Managing DNS Records

- Add A/AAAA records for services, CNAME for aliases, MX for mail, TXT for verification
- TTL: Set appropriately; lower TTL before planned IP changes (allows faster propagation); raise after
- **Cloud DNS changes propagate globally within 2 minutes** (much faster than traditional DNS)

#### DNS Forwarding and Peering

- **Forwarding zones**: Forward queries for specific domains to an on-premises DNS resolver
  - Requires connectivity (VPN or Interconnect) for the GCP→on-premises DNS traffic
- **DNS peering**: Allow a VPC to use another VPC's private DNS zone
  - Useful for hub-spoke networks where hub has the DNS zone
- **Server policies**: Define outbound DNS resolution servers for a VPC (override default GCP DNS)

---

### Network Connectivity Center (NCC)

- Hub-and-spoke model for connecting on-premises sites, VPCs, and cloud resources
- Hub: Central GCP VPC
- Spokes: VPN tunnels, Interconnect attachments, Router Appliance VMs
- Enables transitive routing (which VPC peering doesn't support)
- Use for complex multi-site/multi-cloud networking requiring transitive connectivity

---

## When to Use

| Scenario | Solution |
|----------|---------|
| Private VMs need internet access | Cloud NAT |
| Dev/test on-premises connectivity | Classic or HA VPN |
| Production on-premises HA connectivity, <1 Gbps | HA VPN |
| Production on-premises, high bandwidth (1–10 Gbps) | Dedicated or Partner Interconnect |
| Enterprise hybrid, >10 Gbps | Dedicated Interconnect |
| Cannot meet colo facility requirements | Partner Interconnect |
| Transitive routing between VPCs | Network Connectivity Center |

---

## When NOT to Use

- **Classic VPN for production HA**: Only 99.9% SLA vs HA VPN's 99.99%
- **Dedicated Interconnect for low bandwidth needs**: High cost and setup time; HA VPN is adequate for small-to-medium bandwidth
- **Cloud NAT for inbound traffic**: NAT only works for outbound; cannot serve inbound connections through NAT

---

## Related Services / Concepts

- **Network Planning**: VPC design for hybrid environments — see [network-planning.md](../domain-2-plan-and-configure/network-planning.md)
- **Networking Deploy**: VPC creation, firewall rules — see [networking-deploy.md](../domain-3-deploy-and-implement/networking-deploy.md)
- **VPC Security**: Firewall policies, VPC Service Controls — see [vpc-security.md](../domain-5-configure-access-and-security/vpc-security.md)
- **GKE Planning**: Private cluster needs Cloud NAT — see [gke-planning.md](../domain-2-plan-and-configure/gke-planning.md)

---

## Exam-Relevant Notes

### Common Traps

1. **HA VPN requires BGP (Cloud Router)**: Classic VPN can use static routing; HA VPN requires dynamic routing via Cloud Router and BGP. If a scenario says "static routing," it's Classic VPN.

2. **Dedicated Interconnect doesn't encrypt by default**: Traffic on Dedicated Interconnect stays on Google's network but is not IPsec-encrypted. If encryption is required over Interconnect, use MACsec or run IPsec over it.

3. **Cloud NAT is regional**: You must create a Cloud NAT gateway per region. A single Cloud NAT cannot serve instances in multiple regions.

4. **VPC Peering is non-transitive (again)**: For transitive routing between peered VPCs, use NCC. VPC peering alone doesn't allow VPC A → VPC B → VPC C routing.

5. **Cloud Router required for dynamic routing**: Cloud Router is what manages BGP sessions for VPN and Interconnect. It's not just a router — it's specifically a BGP management component.

6. **SNAT port exhaustion**: A common scenario question. Symptoms: some connections fail. Fix: add more NAT external IPs or increase port allocation per VM.

7. **HA VPN 99.99% SLA requires 2 tunnels**: A single HA VPN tunnel only achieves 99.9% SLA. Both tunnels to on-premises must be active to reach 99.99%.

8. **Partner vs Dedicated Interconnect**: If the question mentions the customer cannot physically locate equipment in a Google colo facility, the answer is Partner Interconnect.

### Keywords
- Cloud NAT, SNAT port exhaustion, HA VPN, Classic VPN, Cloud Router, BGP, Dedicated Interconnect, Partner Interconnect, VLAN attachment, dynamic routing, static routing, DNS forwarding zone, DNS peering, Network Connectivity Center

---

## Source

- https://cloud.google.com/nat/docs/overview
- https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview
- https://cloud.google.com/network-connectivity/docs/interconnect/concepts/overview
- https://cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview
