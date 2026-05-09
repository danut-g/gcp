# Section 2.3 — Planning and Configuring Network Resources

## Exam Relevance
This topic is part of **Section 2: Planning and configuring a cloud solution (~17.5 % of the exam)**. You must understand load balancing options, resource location availability, and Network Service Tiers.

---

## 1. Google Cloud Networking Overview

> 📖 **Docs:** [VPC overview](https://cloud.google.com/vpc/docs/overview) | [Networking overview](https://cloud.google.com/docs/overview/cloud-platform-services#networking) | 🖥️ **Console:** VPC Network → VPC networks

Google Cloud networking is built on Google's global private network — one of the largest in the world. Key concepts:

- **VPC (Virtual Private Cloud)** — Software-defined network that spans all GCP regions
- **Subnets** — Regional IP address ranges within a VPC
- **Firewall Rules** — Control traffic to and from instances
- **Routes** — Define paths for traffic leaving instances
- **Load Balancers** — Distribute traffic across instances

### VPC Key Properties
- VPCs are **global** (span all regions)
- Subnets are **regional** (span all zones within a region)
- VPCs can be in **auto mode** (automatic subnets) or **custom mode** (manual subnets)
- Every project gets a **default VPC** with auto-mode subnets in all regions

```
VPC (Global)
├── Subnet us-central1 (10.128.0.0/20)
│   ├── Zone us-central1-a
│   ├── Zone us-central1-b
│   └── Zone us-central1-c
├── Subnet europe-west1 (10.132.0.0/20)
│   ├── Zone europe-west1-b
│   ├── Zone europe-west1-c
│   └── Zone europe-west1-d
└── Subnet asia-east1 (10.140.0.0/20)
    ├── Zone asia-east1-a
    ├── Zone asia-east1-b
    └── Zone asia-east1-c
```

---

## 2. Load Balancing

> 📖 **Docs:** [Load balancing overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview) | [Choosing a load balancer](https://cloud.google.com/load-balancing/docs/choosing-load-balancer) | 🖥️ **Console:** Network Services → Load balancing

Google Cloud offers multiple load balancer types. Choosing the right one is a common exam topic.

### Load Balancer Decision Matrix

| Load Balancer | Scope | Layer | Traffic Type | Use Case |
|--------------|-------|-------|-------------|----------|
| **External Application LB** | Global | L7 (HTTP/S) | External HTTP/HTTPS | Web apps, APIs, content-based routing |
| **Internal Application LB** | Regional | L7 (HTTP/S) | Internal HTTP/HTTPS | Internal microservices, service mesh |
| **External Proxy Network LB** | Global | L4 (TCP/SSL) | External TCP/SSL | TCP/SSL termination, non-HTTP traffic |
| **Internal Proxy Network LB** | Regional | L4 (TCP) | Internal TCP | Internal TCP services |
| **External Passthrough Network LB** | Regional | L4 (TCP/UDP) | External TCP/UDP | UDP traffic, non-proxied TCP, gaming, IoT |
| **Internal Passthrough Network LB** | Regional | L4 (TCP/UDP) | Internal TCP/UDP | Internal TCP/UDP services, HA databases |

### External Application Load Balancer (HTTP/S LB) — Most Common on Exam

The most feature-rich load balancer, ideal for web applications.

**Key Features**:
- **Global** — Single anycast IP, serves traffic from the nearest healthy backend
- **Layer 7** — Can route based on URL path, host header, HTTP headers
- **SSL termination** — Manages SSL certificates
- **Cloud CDN integration** — Cache content at edge locations
- **Cloud Armor integration** — DDoS protection and WAF
- **URL maps** — Route `/api/*` to one backend, `/images/*` to another
- **Backend services** — Instance groups, NEGs, Cloud Storage buckets

**Architecture**:
```
User → Global Anycast IP → Google Edge → URL Map → Backend Service → Instance Group
                                              │
                                              ├── Health Checks
                                              ├── Session Affinity
                                              └── Autoscaling
```

**Configuration Components**:
1. **Frontend** — IP address, port, SSL certificate
2. **URL Map** — Routing rules (host/path-based)
3. **Backend Service** — Backend groups, health checks, session affinity
4. **Backend** — Instance groups, NEGs, Cloud Storage buckets
5. **Health Check** — Determines backend health

### Internal Passthrough Network LB

**Key Features**:
- **Regional** — Load balances within a single region
- **Layer 4** — TCP/UDP traffic
- **Internal** — Only accessible from within the VPC (or via VPN/Interconnect)
- Use cases: Database HA, internal services

### External Passthrough Network LB

**Key Features**:
- **Regional** — Serves traffic to backends in a single region
- **Layer 4** — TCP/UDP traffic
- **Non-proxied** — Traffic passes directly to backends (preserves source IP)
- Use cases: UDP gaming servers, IoT ingestion, non-HTTP protocols

### Load Balancer Selection Flowchart

```
Is the traffic internal or external?
│
├── INTERNAL
│   ├── HTTP/S? → Internal Application LB
│   └── TCP/UDP? → Internal Passthrough Network LB
│
└── EXTERNAL
    ├── HTTP/S? → External Application LB (global)
    ├── TCP with SSL termination? → External Proxy Network LB
    └── TCP/UDP (non-proxied)? → External Passthrough Network LB (regional)
```

### Health Checks

All load balancers use health checks to determine backend availability:

| Protocol | Port | Interval | Example |
|----------|------|----------|---------|
| HTTP | 80 | 10s | Check `/health` returns 200 |
| HTTPS | 443 | 10s | Check `/healthz` returns 200 |
| TCP | Any | 10s | TCP connection succeeds |
| SSL | 443 | 10s | SSL handshake succeeds |
| gRPC | Any | 10s | gRPC health check |

---

## 3. Cloud CDN (Content Delivery Network)

> 📖 **Docs:** [Cloud CDN overview](https://cloud.google.com/cdn/docs/overview) | [Cloud CDN quickstart](https://cloud.google.com/cdn/docs/setting-up-cdn-with-bucket) | 🖥️ **Console:** Network Services → Cloud CDN

### What Is Cloud CDN?
- Caches content at Google's **edge locations** worldwide
- Reduces latency for users by serving cached content from nearby locations
- Integrated with the **External Application Load Balancer**

### Key Features
- Cache static content (images, CSS, JS, videos)
- Cache keys based on URL, headers, cookies, query strings
- Signed URLs and signed cookies for access control
- Cache invalidation via API or Console
- Supports Cloud Storage buckets and instance groups as origins

### When to Use Cloud CDN
- Serving static content globally
- Reducing backend load for cacheable content
- Improving page load times for global audiences

---

## 4. Resource Location Availability

> 📖 **Docs:** [Regions and zones](https://cloud.google.com/compute/docs/regions-zones) | [GCP locations](https://cloud.google.com/about/locations) | 🖥️ **Console:** per-service region/zone dropdown at creation

### Regions and Zones

| Concept | Definition | Example |
|---------|-----------|---------|
| **Region** | Independent geographic area | `us-central1` (Iowa), `europe-west1` (Belgium) |
| **Zone** | Deployment area within a region | `us-central1-a`, `us-central1-b` |
| **Multi-region** | Large geographic area (Americas, Europe, Asia) | `US`, `EU`, `ASIA` |

### Zone and Region Counts (approximate)
- **40+ regions** worldwide
- **3-4 zones** per region (typically)
- **Regions span**: Americas, Europe, Middle East, Africa, Asia Pacific

### Resource Scope by Level

| Scope | Resources |
|-------|-----------|
| **Global** | VPCs, firewall rules, routes, external IP addresses, images, snapshots, load balancers (some), Cloud CDN |
| **Regional** | Subnets, regional persistent disks, regional instance groups, Cloud Router, Cloud NAT, IP addresses (internal) |
| **Zonal** | VM instances, zonal persistent disks, GKE node pools, local SSDs |

### High Availability Patterns

**Single zone** — No HA (single point of failure)
```
Zone A: [VM1] [VM2]
```

**Multi-zone (within region)** — Protects against zone failure
```
Zone A: [VM1] [VM3]
Zone B: [VM2] [VM4]
```

**Multi-region** — Protects against region failure
```
Region US-Central: [VM1] [VM2]
Region EU-West:    [VM3] [VM4]
Global LB → Routes to nearest healthy region
```

### Choosing a Region — Key Factors
1. **Latency** — Choose regions close to your users
2. **Compliance** — Data residency requirements (e.g., EU data must stay in EU)
3. **Service availability** — Not all services are available in all regions
4. **Pricing** — Costs vary by region (e.g., us-central1 is typically cheapest)
5. **Carbon footprint** — Some regions run on cleaner energy
6. **Disaster recovery** — Choose geographically distant regions for DR

---

## 5. Network Service Tiers

> 📖 **Docs:** [Network Service Tiers overview](https://cloud.google.com/network-tiers/docs/overview) | [Choosing a tier](https://cloud.google.com/network-tiers/docs/using-network-service-tiers) | 🖥️ **Console:** VPC Network → Network Service Tiers

Google Cloud offers two network service tiers that affect how traffic is routed between users and your GCP resources.

### Premium Tier (Default)

```
User → Nearest Google Edge PoP → Google's Global Network → GCP Region
       (enters Google network     (private backbone,        (your resources)
        close to user)            low latency)
```

**Characteristics**:
- Traffic enters Google's network at the **nearest edge Point of Presence (PoP)**
- Routed over Google's **private global fiber backbone**
- **Lower latency**, higher reliability
- Required for **global load balancing**
- **Higher cost** (but often worth it for latency-sensitive apps)
- **Default** tier for all resources

### Standard Tier

```
User → Public Internet → Google Network Entry (near GCP region) → GCP Region
       (traverses public  (enters Google network                   (your resources)
        internet longer)   near destination)
```

**Characteristics**:
- Traffic routed over the **public internet** for most of the journey
- Enters Google's network at the **region** where your resource is located
- **Higher latency**, lower reliability (depends on ISP quality)
- Only supports **regional load balancing** (no global LB)
- **Lower cost** (good for cost-sensitive, non-latency-critical workloads)

### Comparison

| Feature | Premium Tier | Standard Tier |
|---------|-------------|---------------|
| Routing | Google's global backbone | Public internet |
| Latency | Lower | Higher |
| Reliability | Higher | Variable |
| Global LB | Yes | No (regional only) |
| SLA | Google's network SLA | ISP-dependent |
| Cost | Higher | Lower |
| Default | Yes | No |
| Use case | Production, latency-sensitive | Dev/test, cost-sensitive |

### Choosing a Tier

```
Is your workload latency-sensitive or production-critical?
│
├── YES → Premium Tier
│
└── NO
    ├── Need global load balancing? → Premium Tier
    └── Cost is the primary concern? → Standard Tier
```

---

## 6. Firewall Rules

> 📖 **Docs:** [VPC firewall rules](https://cloud.google.com/vpc/docs/firewalls) | [Using firewall rules](https://cloud.google.com/vpc/docs/using-firewalls) | 🖥️ **Console:** VPC Network → Firewall

### Implied Rules
Every VPC has two implied rules that cannot be deleted:
- **Default-deny all ingress** — Blocks all inbound traffic unless a rule explicitly allows it
- **Default-allow all egress** — Permits all outbound traffic unless a rule explicitly denies it

### Firewall Rule Components

| Component | Description | Values / Example |
|-----------|-------------|-----------------|
| **Direction** | Which way traffic flows | `INGRESS` (inbound) or `EGRESS` (outbound) |
| **Priority** | Evaluation order (lower = higher priority) | 0–65535; default is 1000 |
| **Target** | Which instances the rule applies to | All instances, network tags, or service accounts |
| **Source (ingress)** | Where traffic originates | CIDR ranges, tags, service accounts |
| **Destination (egress)** | Where traffic is going | CIDR ranges |
| **Protocol/Ports** | Traffic type to match | `tcp:80`, `udp:53`, `icmp`, `all` |
| **Action** | What to do with matched traffic | `ALLOW` or `DENY` |

### gcloud Commands

```bash
gcloud compute firewall-rules create RULE_NAME --network=VPC_NAME --direction=INGRESS --priority=1000 --action=ALLOW --rules=tcp:80,tcp:443 --source-ranges=0.0.0.0/0 --target-tags=web-server
gcloud compute firewall-rules create allow-ssh-from-iap --network=default --direction=INGRESS --action=ALLOW --rules=tcp:22 --source-ranges=35.235.240.0/20 --target-tags=allow-ssh
gcloud compute firewall-rules list --filter="network=my-vpc" --sort-by=priority
gcloud compute firewall-rules describe RULE_NAME
gcloud compute firewall-rules delete RULE_NAME
```

### Priority Conflict Resolution
- Lower priority number wins (e.g., priority 500 overrides priority 1000)
- If two rules have the **same priority**, `DENY` wins over `ALLOW`

> **Exam tip**: Ingress rules specify a **source**; egress rules specify a **destination**. Mixing these up is a common exam distractor.

---

## 7. VPC Peering and Shared VPC

> 📖 **Docs:** [VPC Network Peering](https://cloud.google.com/vpc/docs/vpc-peering) | [Shared VPC overview](https://cloud.google.com/vpc/docs/shared-vpc) | 🖥️ **Console:** VPC Network → VPC Network Peering / Shared VPC

### VPC Peering

- **No transitive peering** — If A↔B and B↔C, VPC A cannot reach VPC C through B. A direct A↔C peering is required.
- **Both sides must create a peering connection** — Peering is only active when both networks configure it.
- No overlapping IP ranges allowed between peered VPCs.

```bash
gcloud compute networks peerings create PEERING_NAME --network=VPC_A --peer-project=OTHER_PROJECT --peer-network=VPC_B --export-custom-routes --import-custom-routes
gcloud compute networks peerings list --network=VPC_A
gcloud compute networks peerings delete PEERING_NAME --network=VPC_A
```

### Shared VPC

- **Host project** owns the VPC and its subnets; **service projects** use subnets from the host project.
- Requires the **Shared VPC Admin** role (`roles/compute.xpnAdmin`) granted at the **organization or folder** level.
- Network administration is centralized; billing remains separate per project.

```bash
gcloud compute shared-vpc enable HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID --host-project=HOST_PROJECT_ID
gcloud compute shared-vpc associated-projects list --host-project=HOST_PROJECT_ID
```

> **Exam tip**: Service project resources (VMs, GKE nodes, etc.) use **host project subnets**. IAM must explicitly grant `roles/compute.networkUser` on the shared subnet to service project service accounts.

---

## 8. Private Google Access

> 📖 **Docs:** [Private Google Access](https://cloud.google.com/vpc/docs/private-google-access) | [Configure Private Google Access](https://cloud.google.com/vpc/docs/configure-private-google-access) | 🖥️ **Console:** VPC Network → Subnets → Edit → Private Google Access

- Allows VM instances with **no external IP** to reach Google APIs (e.g., Cloud Storage, BigQuery) via internal IP routing — traffic stays on Google's network.
- Enabled at the **subnet level**:

```bash
gcloud compute networks subnets update SUBNET_NAME --region=REGION --enable-private-ip-google-access
```

- Does **NOT** allow general internet access — only routes traffic to `*.googleapis.com`.
- **For on-premises access to Google APIs**: use Private Google Access for on-premises, which requires Cloud VPN or Interconnect plus DNS configuration pointing `googleapis.com` to the restricted VIP range `199.36.153.8/30`.

---

## 6. Cloud Interconnect and VPN (Preview for Section 3)

> 📖 **Docs:** [Choosing a hybrid connectivity product](https://cloud.google.com/network-connectivity/docs/how-to/choose-product) | [HA VPN overview](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview) | 🖥️ **Console:** Hybrid Connectivity → VPN / Interconnect

### Connectivity Options to On-Premises

| Option | Bandwidth | Latency | SLA | Use Case |
|--------|-----------|---------|-----|----------|
| **Dedicated Interconnect** | 10-200 Gbps | Lowest | 99.9-99.99% | Large, predictable data transfer |
| **Partner Interconnect** | 50 Mbps - 50 Gbps | Low | 99.9-99.99% | Medium bandwidth needs |
| **HA VPN** | 1.4-3 Gbps per tunnel | Variable | 99.99% | Encrypted connection over internet |
| **Classic VPN** | 1.4 Gbps per tunnel | Variable | 99.9% | Legacy, simple VPN |

---

## 7. Cloud DNS

> 📖 **Docs:** [Cloud DNS overview](https://cloud.google.com/dns/docs/overview) | [Create a public/private zone](https://cloud.google.com/dns/docs/zones) | 🖥️ **Console:** Network Services → Cloud DNS

### What Is Cloud DNS?
- **Managed DNS service** — Authoritative DNS hosting
- 100% SLA for authoritative name serving
- Supports public and private DNS zones

### Key Features
- **Public zones** — Resolve domain names from the internet
- **Private zones** — Resolve names within VPC networks (not visible to internet)
- **DNS forwarding** — Forward queries to on-premises DNS servers
- **DNS peering** — Share DNS zones between VPCs
- DNSSEC support for security

---

## 9. IP Addressing — Internal, External, Static, and Ephemeral

> 📖 **Docs:** [IP address concepts](https://cloud.google.com/vpc/docs/ip-addresses) | [Reserve static IPs](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address) | 🖥️ **Console:** VPC Network → IP addresses

### Internal vs. External IP
- **Internal IP**: Private IP address from a subnet range, used for communication within the VPC and to peered VPCs.
- **External IP**: Public, internet-routable IP assigned to a resource for outside communication.

### Static vs. Ephemeral
- **Ephemeral**: Automatically assigned and released when the resource (VM, forwarding rule) is deleted or stopped.
- **Static**: Reserved IP (internal or external) that persists independently of the resource and can be reassigned.

```bash
# Reserve a static external IP in a region
gcloud compute addresses create my-static-ip --region=us-central1

# Reserve a global static external IP (for global load balancers)
gcloud compute addresses create my-global-ip --global

# Assign a static IP to a VM
gcloud compute instances create my-vm --address=my-static-ip --zone=us-central1-a
```

- **Exam tip**: Global load balancers (External Application LB, SSL Proxy) require **global static IPs**; regional load balancers use **regional static IPs**.

---

## 10. Private Service Connect and Serverless VPC Access

> 📖 **Docs:** [Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect) | [Serverless VPC Access](https://cloud.google.com/run/docs/configuring/connecting-vpc) | 🖥️ **Console:** VPC Network → Private Service Connect / Serverless VPC Access

### Private Service Connect (PSC)
- Allows consumers to privately reach services (Google APIs, managed services, other VPCs) using internal IP addresses in their own VPC.
- Use cases: access Google APIs privately, publish a service to other customers, consume third-party managed services.
- Complements Private Google Access by providing finer-grained endpoints.

### Serverless VPC Access
- Enables serverless products (Cloud Run, Cloud Functions, App Engine standard) to connect to resources inside a VPC using internal IP addresses.
- Configured as a **VPC connector** in a specific region and subnet.

```bash
gcloud compute networks vpc-access connectors create my-connector \
  --region=us-central1 --network=my-vpc --range=10.8.0.0/28
```

---

## 11. Hierarchical Firewall Policies and VPC Firewall Rules Logging

> 📖 **Docs:** [Hierarchical firewall policies](https://cloud.google.com/vpc/docs/firewall-policies) | [Cloud NGFW overview](https://cloud.google.com/firewall/docs/about-firewalls) | 🖥️ **Console:** VPC Network → Firewall Policies

- **Hierarchical firewall policies** are applied at the **organization or folder level**, enforcing rules that cannot be overridden by project-level VPC firewall rules. Useful for mandatory security baselines.
- **VPC firewall rules logging** records allowed and denied connections for auditing and troubleshooting; enabled per rule with `--enable-logging`.

```bash
gcloud compute firewall-rules update RULE_NAME --enable-logging
```

---

## 12. Cloud NAT (Network Address Translation)

> 📖 **Docs:** [Cloud NAT overview](https://cloud.google.com/nat/docs/overview) | [Set up Cloud NAT](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation) | 🖥️ **Console:** Network Services → Cloud NAT → Create NAT gateway

### What Is Cloud NAT?
- Allows VMs **without external IP addresses** to access the internet
- Provides **outbound-only** connectivity (VMs can initiate connections out, but external traffic cannot initiate connections in)
- Software-defined, **no proxy VMs** needed

### Key Features
- Automatic and manual NAT IP allocation
- Configurable min/max ports per VM
- Logging support
- Regional resource

### When to Use Cloud NAT
- VMs need to download packages or updates from the internet
- VMs need to call external APIs
- Security requirement: VMs should not have external IPs

---

## Exam Practice Questions

1. **A company needs to serve a global web application with low latency. Traffic should be distributed to the nearest healthy backend. Which load balancer should they use?**
   - Answer: **External Application Load Balancer (HTTP/S)** — Global, L7, single anycast IP, routes to nearest healthy backend.

2. **You have internal microservices that communicate over HTTP within a VPC. Which load balancer is appropriate?**
   - Answer: **Internal Application Load Balancer** — Regional, L7, for internal HTTP/HTTPS traffic.

3. **Your gaming application uses UDP protocol and needs an external-facing load balancer. Which should you choose?**
   - Answer: **External Passthrough Network Load Balancer** — Regional, L4, supports UDP traffic.

4. **What is the main difference between Premium and Standard network tiers?**
   - Answer: Premium Tier routes traffic through Google's private global backbone (lower latency), while Standard Tier routes through the public internet (higher latency, lower cost). Premium supports global LB; Standard only supports regional LB.

5. **You want VMs without external IPs to access the internet for software updates. What service should you use?**
   - Answer: **Cloud NAT** — Provides outbound internet access for VMs without external IPs.

6. **A European company must ensure all user data stays within the EU. How should they plan their resource locations?**
   - Answer: Deploy resources exclusively in EU regions (e.g., `europe-west1`, `europe-west4`). Use organization policies (`gcp.resourceLocations`) to restrict resource creation to EU locations only.

7. **Your application needs to handle 50 Gbps of data transfer between on-premises and GCP with the lowest possible latency. Which connectivity option is best?**
   - Answer: **Dedicated Interconnect** — Provides 10-200 Gbps with the lowest latency and a direct physical connection to Google's network.

---

## Glossary

**ACL (Access Control List)** — A list of permissions attached to a resource specifying which principals are granted which types of access; used in legacy Cloud Storage access control.

**Anycast IP** — A single IP address announced from multiple geographic locations; traffic is automatically routed to the nearest available endpoint. Used by the External Application Load Balancer for global routing.

**App Engine** — GCP's fully managed serverless platform for building web and mobile applications; can use Serverless VPC Access to reach VPC resources.

**Auto Mode VPC** — A VPC network in which Google automatically creates one subnet per region with predefined IP ranges. Contrast with custom mode, where subnets are created manually.

**Autoscaler** — A GCP component that automatically adjusts the number of VM instances in a MIG or GKE nodes in a cluster based on load signals.

**Backend** — A set of compute endpoints (instance group, NEG, or Cloud Storage bucket) that receive traffic from a GCP load balancer.

**Backend Service** — A load balancer component that groups one or more backends (instance groups, NEGs, or buckets), defines health checks, session affinity, and balancing mode.

**BGP (Border Gateway Protocol)** — A standard routing protocol used by Cloud Router to dynamically exchange routes between a GCP VPC and on-premises networks over Cloud Interconnect or HA VPN.

**BYOL (Bring Your Own License)** — A licensing model allowing customers to use their existing software licenses on cloud infrastructure, often requiring sole-tenant nodes for compliance.

**CDN (Content Delivery Network)** — A distributed network of edge servers that caches and serves content to users from locations geographically close to them, reducing latency.

**CIDR (Classless Inter-Domain Routing)** — A method for allocating IP addresses and routing that uses a notation like `10.0.0.0/24` to specify a network and its prefix length.

**Classic VPN** — A legacy GCP VPN option providing up to 1.4 Gbps per tunnel with a 99.9% SLA. Superseded by HA VPN for most use cases.

**Cloud Armor** — A GCP service that provides DDoS protection and Web Application Firewall (WAF) capabilities, integrated with the External Application Load Balancer.

**Cloud CDN** — Google Cloud's Content Delivery Network service; caches content at Google edge locations worldwide and is integrated with the External Application Load Balancer.

**Cloud DNS** — Google Cloud's fully managed, authoritative DNS hosting service supporting public zones, private zones, DNS forwarding, DNS peering, and DNSSEC.

**Cloud Interconnect** — A family of GCP services (Dedicated Interconnect and Partner Interconnect) providing private, high-bandwidth, low-latency connections between on-premises networks and Google Cloud.

**Cloud NAT (Network Address Translation)** — A GCP managed service that allows VM instances without external IP addresses to initiate outbound connections to the internet while preventing inbound connections from the internet.

**Cloud Functions** — GCP's fully managed serverless compute platform for running event-driven functions; can use Serverless VPC Access to reach VPC resources.

**Cloud Router** — A GCP managed service that dynamically exchanges routes between a VPC and on-premises networks using BGP (Border Gateway Protocol), used with Dedicated Interconnect, Partner Interconnect, and HA VPN.

**Cloud Run** — GCP's fully managed serverless platform for running stateless containers; can connect to a VPC via Serverless VPC Access.

**Compute Engine** — GCP's IaaS offering providing VM instances; all VM network connectivity, firewall rules, and routes are defined within a VPC.

**Custom Mode VPC** — A VPC network where subnets are created manually, giving full control over IP ranges. Recommended for production environments.

**DDoS (Distributed Denial of Service)** — An attack that floods a service with traffic from multiple sources to make it unavailable. Cloud Armor provides DDoS protection.

**Dedicated Interconnect** — A GCP connectivity option providing a direct physical connection between on-premises infrastructure and Google's network, offering 10–200 Gbps bandwidth and the lowest latency.

**Default Allow Egress** — The implied VPC firewall rule that permits all outbound traffic from VMs unless another egress rule explicitly denies it.

**Default Deny Ingress** — The implied VPC firewall rule that blocks all inbound traffic to VMs unless another ingress rule explicitly allows it.

**Default VPC** — An auto-mode VPC network automatically created in every new GCP project, with one subnet pre-configured in each region.

**DNS (Domain Name System)** — A hierarchical naming system that translates human-readable domain names (e.g., `example.com`) into IP addresses.

**DNSSEC (DNS Security Extensions)** — A set of extensions to DNS that add cryptographic signatures to DNS records, protecting against DNS spoofing and cache poisoning. Supported by Cloud DNS.

**DR (Disaster Recovery)** — Strategies and procedures to restore systems and data after a catastrophic event. Multi-region deployments are a common GCP DR pattern.

**Edge Location (PoP — Point of Presence)** — A geographically distributed network node where Google's infrastructure connects to the public internet and where Premium Tier traffic enters Google's private backbone.

**Egress** — Network traffic flowing outbound from a resource (e.g., from a VM to the internet). Cloud NAT handles egress for VMs without external IPs.

**Ephemeral IP Address** — A non-reserved IP address assigned automatically to a resource and released when the resource is deleted or stopped; contrast with static IPs that are reserved and persist.

**External Application Load Balancer** — A global, Layer 7 (HTTP/S) GCP load balancer that uses a single anycast IP, supports URL-based routing, SSL termination, Cloud CDN, and Cloud Armor. Formerly called "HTTP(S) Load Balancer."

**External IP Address** — A publicly routable IP address assigned to a GCP resource, allowing it to communicate with the internet.

**External Passthrough Network LB** — A regional, Layer 4 (TCP/UDP) load balancer that forwards traffic directly to backends without proxying, preserving the client source IP. Suitable for UDP and non-HTTP protocols.

**External Proxy Network LB** — A global, Layer 4 (TCP/SSL) load balancer that terminates TCP/SSL connections and proxies traffic to backends.

**Firewall Rule** — A GCP VPC resource that permits or denies traffic to or from VM instances based on direction, protocol, ports, and source/destination identifiers.

**Firewall Rule Logging** — A per-rule VPC firewall feature that records allowed or denied connections for auditing and troubleshooting.

**Folder** — A GCP resource hierarchy element that groups projects and sub-folders, allowing IAM and organization policies (including hierarchical firewall policies) to be applied at intermediate levels.

**Forwarding Rule** — A GCP load balancer configuration resource that associates an IP address, protocol, and port range with a target proxy or backend service.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services, including compute, storage, networking, databases, machine learning, and more.

**gcloud** — The primary command-line tool for interacting with GCP services, part of the Google Cloud SDK.

**gRPC** — A high-performance, open-source Remote Procedure Call (RPC) framework developed by Google. Supported as a health check protocol by GCP load balancers.

**HA (High Availability)** — Design approaches that minimize downtime by eliminating single points of failure, typically through redundancy across multiple zones or regions.

**HA VPN** — A GCP VPN offering with two redundant tunnels providing a 99.99% SLA. Supports up to ~3 Gbps aggregate throughput.

**Hierarchical Firewall Policy** — A firewall policy applied at the organization or folder level that enforces mandatory security rules across multiple projects and cannot be overridden by project-level VPC firewall rules.

**Health Check** — A probe configured on a load balancer to determine whether backend instances are healthy and capable of serving traffic.

**Host Project** — In Shared VPC, the project that owns the VPC network and subnets; service projects attach to the host project to use its network resources.

**HTTP (Hypertext Transfer Protocol)** — The application-layer protocol used for web communication. Layer 7 load balancers operate at this level.

**HTTPS (HTTP Secure)** — HTTP encrypted with TLS/SSL, used for secure web communication. Managed SSL certificates can be provisioned by GCP.

**IAM (Identity and Access Management)** — GCP's system for controlling who (identity) can do what (role/permission) on which resources. Used for firewall rule targets, Shared VPC access, and more.

**Implied Rule** — A VPC firewall rule that always exists and cannot be deleted: default-deny ingress and default-allow egress.

**IAP (Identity-Aware Proxy)** — A GCP service that enforces access control at the application layer, allowing secure access to GCP resources without a VPN. Used for SSH tunneling to VMs without external IPs.

**ICMP (Internet Control Message Protocol)** — A network-layer protocol used for diagnostic purposes (e.g., `ping`). Can be specified in GCP firewall rules.

**Ingress** — Network traffic flowing inbound to a resource (e.g., from the internet to a VM). Firewall rules controlling inbound traffic use `INGRESS` direction.

**Internal Application LB** — A regional, Layer 7 (HTTP/S) GCP load balancer accessible only from within a VPC or connected networks. Used for internal microservices.

**Internal Passthrough Network LB** — A regional, Layer 4 (TCP/UDP) load balancer accessible only within a VPC. Suitable for HA database configurations and internal TCP/UDP services.

**Internal Proxy Network LB** — A regional, Layer 4 (TCP) proxied load balancer for internal TCP traffic within a VPC.

**IoT (Internet of Things)** — A network of physical devices that collect and exchange data. GCP's External Passthrough Network LB is often used for IoT ingestion requiring UDP support.

**Internal IP Address** — A private, non-internet-routable IP address assigned from a subnet's range; used for VM-to-VM communication within a VPC and across peered or shared VPCs.

**IP Address** — A numerical label assigned to a device on a network. GCP resources use both internal (private) and external (public) IP addresses.

**ISP (Internet Service Provider)** — A company that provides internet access. Standard Tier routing relies on ISP networks, which can introduce variable latency.

**L4 (Layer 4)** — The transport layer of the OSI model, handling TCP and UDP traffic. L4 load balancers route based on IP address and port without inspecting application content.

**L7 (Layer 7)** — The application layer of the OSI model, handling HTTP/S and other application protocols. L7 load balancers can route based on URL paths, hostnames, and HTTP headers.

**NEG (Network Endpoint Group)** — A GCP configuration object specifying a group of backend endpoints (IP:port pairs, serverless services, or internet endpoints) that can be used as backends for load balancers.

**Network Tag** — A character string attached to Compute Engine VM instances used as a target or source for firewall rules, enabling simple identity-based access control without service accounts.

**Network Service Tiers** — GCP's two-tier system (Premium and Standard) controlling how traffic between users and GCP resources is routed over Google's private backbone vs. the public internet.

**Organization** — The top-level node in the GCP resource hierarchy, representing a company; contains folders and projects and is where organization policies and hierarchical firewall policies are applied.

**Organization Policy** — A GCP governance feature that enforces constraints on resource configurations across an entire organization, folder, or project (e.g., `gcp.resourceLocations` to restrict deployments to specific regions).

**Permission** — A specific action that can be performed on a GCP resource (e.g., `compute.instances.create`); permissions are grouped into roles and granted to principals via IAM bindings.

**Partner Interconnect** — A GCP connectivity option that provides a connection between on-premises networks and GCP through a supported service provider, supporting 50 Mbps to 50 Gbps.

**PoP (Point of Presence)** — See Edge Location. A network access point where Premium Tier traffic enters Google's private global backbone close to the user.

**Premium Tier** — The default GCP Network Service Tier that routes traffic over Google's private global fiber backbone, providing lower latency and higher reliability than Standard Tier.

**Principal** — An identity (user, group, service account, or domain) to which IAM roles can be granted; the "who" in "who can do what on which resource".

**Priority (Firewall Rule)** — An integer (0–65535) that determines the evaluation order of firewall rules; lower numbers have higher priority and override higher-numbered rules.

**Private Google Access** — A subnet-level GCP setting that allows VM instances without external IP addresses to reach Google APIs and services (e.g., Cloud Storage, BigQuery) using internal IP routing.

**Private Service Connect (PSC)** — A GCP networking feature that allows consumers to privately reach Google APIs, managed services, or services in other VPCs using internal IP addresses in their own VPC.

**Private Zone (DNS)** — A Cloud DNS zone that resolves domain names only within specified VPC networks, not visible to the public internet.

**Project** — A GCP resource container that holds resources, IAM bindings, billing configuration, and quotas; every GCP resource belongs to exactly one project.

**Protocol** — A set of rules governing data communication (e.g., TCP, UDP, ICMP, HTTP). Firewall rules and load balancers are configured to handle specific protocols.

**Public Zone (DNS)** — A Cloud DNS zone that resolves domain names for queries originating from anywhere on the internet.

**Region** — A specific geographic location where GCP resources are hosted (e.g., `us-central1` in Iowa). Contains multiple isolated zones.

**Resource (GCP)** — Any addressable entity within GCP (e.g., a VM, VPC, subnet, firewall rule, load balancer); resources belong to projects and are the target of IAM bindings.

**Restricted VIP** — A reserved Google-owned IP range (`199.36.153.4/30` private, `199.36.153.8/30` restricted) that routes Google API traffic privately through the VPC without exposing it to the public internet.

**Role** — A named bundle of IAM permissions (e.g., `roles/compute.networkUser`) that can be granted to a principal; GCP provides basic, predefined, and custom roles.

**Route** — A GCP networking resource that defines the path that outbound traffic from a VM takes to reach a destination IP range.

**Serverless VPC Access** — A GCP feature that enables serverless products (Cloud Run, Cloud Functions, App Engine standard) to reach VPC resources via internal IPs using a VPC connector.

**Service Account** — A special GCP account used by applications and services (rather than humans) to authenticate and authorize API calls. Can be used as a firewall rule target for fine-grained access control.

**Service Project** — In Shared VPC, a project that consumes subnets from a host project. Billing and resource management remain separate from the host project.

**Shared VPC** — A GCP networking feature allowing multiple projects (service projects) to share subnets from a centrally managed VPC (in the host project), enabling network administration centralization.

**SLA (Service Level Agreement)** — A commitment by GCP to a minimum level of service availability (e.g., 99.99%). Premium Tier carries Google's network SLA; Standard Tier depends on ISP quality.

**Source Range** — A CIDR range specified in an ingress firewall rule identifying permitted or denied traffic origins.

**SSL (Secure Sockets Layer)** — A cryptographic protocol for securing network communications, now largely superseded by TLS. GCP load balancers can terminate SSL/TLS connections.

**SSL Termination** — The process of decrypting SSL/TLS-encrypted traffic at a load balancer before forwarding it (often unencrypted) to backend servers.

**Standard Tier** — A GCP Network Service Tier that routes traffic over the public internet until it reaches the GCP region where the resource resides. Lower cost but higher and more variable latency than Premium Tier.

**Static IP Address** — A reserved IP address (internal or external, regional or global) that persists independently of the resource it is attached to; required for global load balancer frontends.

**Subnet** — A regional IP address range within a VPC network. VM instances in a subnet share the same IP address block and can communicate without routing.

**Target Tag** — A network tag used in a firewall rule to specify which VM instances the rule applies to; contrast with source tags that identify originators of ingress traffic.

**TCP (Transmission Control Protocol)** — A connection-oriented transport protocol providing reliable, ordered delivery. Used by HTTP, HTTPS, SSH, and many other application protocols.

**TLS (Transport Layer Security)** — The successor to SSL; a cryptographic protocol for securing communications over networks. GCP manages TLS certificates for HTTPS load balancers.

**UDP (User Datagram Protocol)** — A connectionless transport protocol providing fast but unreliable delivery. Used for DNS, gaming, IoT, and streaming applications. Supported by L4 (passthrough) load balancers.

**URL Map** — A GCP load balancer configuration resource that defines routing rules, mapping incoming request URLs (host and path) to specific backend services.

**VPC (Virtual Private Cloud)** — A global, software-defined private network in GCP that provides networking for VM instances and other resources, including subnets, firewall rules, and routes.

**VPC Connector** — A regional resource used by Serverless VPC Access that provides a bridge between serverless services and a VPC subnet, enabling internal IP access from Cloud Run, Cloud Functions, and App Engine.

**VPC Peering** — A GCP networking feature connecting two VPC networks so resources in each can communicate using internal IP addresses. Peering is non-transitive.

**VPN (Virtual Private Network)** — An encrypted tunnel over the public internet connecting an on-premises network to a GCP VPC. GCP offers Classic VPN and HA VPN.

**WAF (Web Application Firewall)** — A security system that filters and monitors HTTP traffic between a web application and the internet, protecting against common exploits. Provided by Cloud Armor.

**Zone** — An isolated deployment area within a GCP region (e.g., `us-central1-a`). Zonal failures affect only that zone, not the entire region.
