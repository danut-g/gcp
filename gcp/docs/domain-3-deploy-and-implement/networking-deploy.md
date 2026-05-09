# Networking Deployment: VPC, Firewall Rules, Load Balancing, Cloud DNS

## Overview

Deploying networking resources involves creating VPCs and subnets, configuring firewall rules, deploying the appropriate load balancer type, and setting up DNS. GCP's load balancing portfolio is one of the most complex areas in the exam — understanding which load balancer type to use in each scenario is critical.

---

## Key Concepts

### VPC and Subnet Creation

Key deployment decisions:
- **VPC mode**: Custom (recommended for production) vs Auto
- **Subnet region**: Which region(s) to deploy subnets in
- **CIDR planning**: Size subnets for expected VM count + growth + GKE secondary ranges
- **Private Google Access**: Enable on subnets where VMs don't have external IPs

Best practice sequence:
1. Create custom-mode VPC
2. Create subnets with appropriate CIDR ranges per region
3. Enable Private Google Access on private subnets
4. Configure firewall rules (start with deny-all, explicitly allow needed traffic)

---

### Firewall Rules

#### Firewall Rule Components

| Component | Description |
|-----------|-------------|
| **Direction** | Ingress (inbound to VM) or Egress (outbound from VM) |
| **Priority** | 0–65535; lower = higher priority |
| **Action** | Allow or Deny |
| **Target** | All instances, instances with specific tag, instances with specific SA |
| **Source/Destination** | IP ranges, source tags (ingress), destination tags (egress) |
| **Protocols/Ports** | TCP, UDP, ICMP, or all protocols |

#### Firewall Rule Targets

- **Network tags**: String tags on VMs; firewall rules match against these tags
  - More flexible but can be applied incorrectly (anyone with compute.instances.setTags can add tags)
- **Service accounts**: More secure; tie firewall rules to a SA identity
  - Only applied to VMs using that SA
  - Cannot be changed without changing the SA (more secure than tags)

#### Implied and Default Rules

- **Implied rules** (cannot be deleted, lowest priority 65535):
  - Deny all ingress from all sources
  - Allow all egress to all destinations
- **Default VPC only**: Additional default rules allowing SSH, RDP, ICMP, internal (can be deleted)

#### Hierarchical Firewall Policies

- Define firewall policies at org or folder level
- Applied before VPC-level firewall rules
- Rules can be `allow`, `deny`, or `goto_next` (pass to next policy layer)
- Enables centralized firewall management: org/folder policies enforce baseline rules; individual VPCs can add more specific rules
- See [vpc-security.md](../domain-5-configure-access-and-security/vpc-security.md) for advanced firewall details

---

### Cloud Load Balancing Portfolio

GCP has multiple load balancer types. Choosing the right one is a critical ACE exam skill.

#### Load Balancer Decision Dimensions

1. **External vs Internal**: Does traffic come from the internet or from within your VPC?
2. **Global vs Regional**: Does traffic come from a single region or globally distributed?
3. **HTTP(S) vs TCP/UDP**: Is this Layer 7 (HTTP/HTTPS) or Layer 4 (TCP/UDP)?
4. **Protocol**: HTTP, HTTPS, HTTP/2, gRPC, TCP, SSL/TLS, UDP

#### Load Balancer Types Matrix

| LB Type | Layer | Scope | Protocol | Use Case |
|---------|-------|-------|----------|---------|
| **External Application LB (Global)** | 7 | Global | HTTP/HTTPS | Internet-facing apps needing global routing, CDN |
| **External Application LB (Regional)** | 7 | Regional | HTTP/HTTPS | Internet-facing apps in one region |
| **Internal Application LB** | 7 | Regional | HTTP/HTTPS | Internal microservices (VPC) |
| **External Network LB (pass-through, regional)** | 4 | Regional | TCP/UDP | External TCP/UDP (VIPs in specific region) |
| **Internal TCP/UDP LB (pass-through)** | 4 | Regional | TCP/UDP | Internal TCP/UDP services |
| **External TCP Proxy LB** | 4 (proxy) | Global | TCP | Global TCP with proxy termination |
| **External SSL Proxy LB** | 4 (proxy) | Global | SSL/TLS | Global SSL termination (non-HTTP) |
| **Internal Regional TCP/UDP LB** | 4 | Regional | TCP/UDP | Internal regional TCP services |

#### Key Load Balancer Details

**External Application Load Balancer (Global):**
- Global Anycast IP — traffic routes to nearest GCP Point of Presence
- SSL termination, URL map routing, Cloud Armor integration, Cloud CDN
- Backend services: MIGs, NEGs, Cloud Run, Cloud Functions, GKE
- HTTP → HTTPS redirect supported

**External Network Load Balancer (Pass-through, Regional):**
- Does NOT terminate TCP — passes traffic directly to backend VMs
- Backend VMs see client IP directly
- Supports non-HTTP protocols (TCP/UDP/ESP/AH/GRE/ICMP)
- No SSL termination, no URL routing

**Internal Application Load Balancer:**
- Layer 7 load balancer for traffic within VPC
- Supports URL routing, header manipulation
- Backends: MIGs, NEGs, Cloud Run (internal)

#### Network Endpoint Groups (NEGs)

- NEGs allow load balancers to target endpoints at Pod level (for GKE) or at individual resource level
- Types:
  - **Zonal NEG**: Specific VM instances/pods in a zone
  - **Serverless NEG**: Target Cloud Run, Cloud Functions, App Engine
  - **Internet NEG**: External endpoints
  - **Private Service Connect NEG**: Access published services via PSC

#### Health Checks

- Required for all load balancers
- Types: HTTP, HTTPS, HTTP/2, TCP, SSL, gRPC
- Configured with: port, request path, healthy/unhealthy thresholds, check interval
- **LB health check ≠ Autohealing health check**: They serve different purposes but often have the same configuration

---

### Cloud DNS

- Managed authoritative DNS service
- **Zones**:
  - **Public zone**: Serves DNS responses to the internet
  - **Private zone**: Serves DNS responses only within VPC(s)
- DNS records: A, AAAA, CNAME, MX, NS, TXT, SOA, PTR, SRV, CAA

#### Split-horizon DNS

- Have the same domain resolve to different IPs inside vs outside the VPC
- Create both a public and private zone for the same domain name
- The private zone responds to queries from within the VPC; public zone for external queries

#### DNS Policies

- **Server policy**: Configure inbound/outbound forwarding
- **Forwarding zones**: Forward DNS queries for specific domains to on-premises DNS servers
- **Peering zones**: Share DNS with peered VPCs
- **DNS Server policy for outbound forwarding**: Route DNS queries from VPC to on-premises resolver

#### Cloud DNS vs Instance DNS

- By default, VMs resolve internal hostnames via GCP's internal DNS (`INSTANCE_NAME.ZONE.c.PROJECT_ID.internal`)
- Cloud DNS private zones can provide custom internal DNS names

---

### External IP Addresses

- **Ephemeral**: Assigned to VMs or load balancers; released when resource is deleted
- **Static (Reserved)**: Persistent; remain even when resource is stopped/deleted; billed when not in use
- **Premium network tier**: Uses Google's global backbone for routing (higher reliability, lower latency)
- **Standard network tier**: Uses public internet routing (lower cost)

---

## When to Use

| Scenario | Load Balancer |
|----------|--------------|
| Global HTTPS with CDN | External Application LB (Global) |
| HTTP(S) within a region | External Application LB (Regional) |
| Internal HTTP microservices in VPC | Internal Application LB |
| External TCP/UDP, non-HTTP | External Network LB (pass-through) |
| Internal TCP traffic | Internal TCP/UDP LB |
| Global SSL termination (non-HTTP) | SSL Proxy LB |

---

## When NOT to Use

- **External Application LB for internal traffic**: Creates unnecessary external exposure; use Internal Application LB
- **Pass-through Network LB for HTTP**: It doesn't understand HTTP; cannot do URL routing or SSL termination
- **Ephemeral IPs for production services**: Use static reserved IPs for production LB frontends

---

## Related Services / Concepts

- **Network Planning**: VPC design — see [network-planning.md](../domain-2-plan-and-configure/network-planning.md)
- **Managing Networking**: Cloud NAT, VPN, Interconnect — see [managing-networking.md](../domain-4-ensure-success/managing-networking.md)
- **VPC Security**: Firewall policies — see [vpc-security.md](../domain-5-configure-access-and-security/vpc-security.md)
- **GKE Deploy**: Container-native LB — see [gke-deploy.md](gke-deploy.md)

---

## Exam-Relevant Notes

### Common Traps

1. **External Network LB is pass-through**: It does not terminate TCP. The backend VM sees the client IP. This means backend VMs must handle SSL termination themselves if needed.

2. **Global vs Regional External Application LB**: Both handle HTTP/HTTPS, but Global has a single global Anycast IP routing to nearest POP. Regional targets a single region. For multi-region user bases, choose Global.

3. **Serverless NEG for Cloud Run**: To put Cloud Run behind an HTTP(S) LB (for custom domain, Cloud Armor, CDN), use a Serverless NEG — not a standard backend service.

4. **Health checks are required**: Forgetting to configure a health check means the LB doesn't know which backends are healthy. Exam scenarios will test you on this.

5. **Network tags vs SA in firewall rules**: Tags are more flexible but less secure (anyone can add tags to VMs). SA-based firewall rules are more secure. For sensitive production firewalls, prefer SA-based rules.

6. **Cloud DNS private zones must be associated with VPCs**: A private zone doesn't auto-apply to all VPCs. It must be explicitly associated with one or more VPCs.

7. **Standard vs Premium network tier**: Premium uses Google's backbone (higher quality); Standard uses public internet (cheaper). For global applications, Premium ensures lower latency and higher reliability.

8. **Hierarchical firewall policies override VPC rules**: Policies at org/folder level are evaluated first. If an org policy denies port 22 from all IPs, VPC-level rules allowing SSH won't override it.

### Keywords
- External Application LB, Internal Application LB, Network LB pass-through, SSL Proxy LB, NEG, Serverless NEG, health check, URL map, backend service, Cloud DNS, private zone, forwarding zone, static IP, Premium tier, hierarchical firewall policy

---

## Source

- https://cloud.google.com/load-balancing/docs/choosing-load-balancer
- https://cloud.google.com/vpc/docs/firewalls
- https://cloud.google.com/dns/docs/overview
- https://cloud.google.com/load-balancing/docs/negs
