# Section 4.5 — Managing Networking Resources

## Exam Relevance
This topic is part of **Section 4: Ensuring successful operation of a cloud solution (~20 % of the exam)**. You must know how to add and expand subnets, reserve static IPs, and work with Cloud DNS and Cloud NAT.

---

## 1. Adding a Subnet to an Existing VPC

> 📖 **Docs:** [Create and manage VPC networks](https://cloud.google.com/vpc/docs/create-modify-vpc-networks) | [Add subnets](https://cloud.google.com/vpc/docs/subnets#add-subnet) | 🖥️ **Console:** VPC network → VPC networks → select VPC → Subnets tab → Add Subnet

### Creating a New Subnet

```bash
# Add a subnet to an existing custom VPC
gcloud compute networks subnets create new-subnet \
  --network=my-vpc \
  --region=europe-west1 \
  --range=10.1.0.0/24

# Add a subnet with Private Google Access
gcloud compute networks subnets create private-subnet \
  --network=my-vpc \
  --region=asia-east1 \
  --range=10.2.0.0/24 \
  --enable-private-ip-google-access

# Add a subnet with VPC Flow Logs
gcloud compute networks subnets create logged-subnet \
  --network=my-vpc \
  --region=us-west1 \
  --range=10.3.0.0/24 \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-5-sec \
  --logging-flow-sampling=0.5

# Add a subnet with secondary ranges for GKE
gcloud compute networks subnets create gke-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.4.0.0/24 \
  --secondary-range=pod-range=10.8.0.0/14,service-range=10.12.0.0/20
```

### Key Considerations When Adding Subnets
- Subnet IP ranges **cannot overlap** with other subnets in the same VPC
- Subnets are **regional** — they span all zones in a region
- VPC must be in **custom mode** (auto mode creates subnets automatically)
- You can add subnets without downtime or affecting existing resources

---

## 2. Expanding a Subnet to Have More IP Addresses

> 📖 **Docs:** [Expand a subnet](https://cloud.google.com/vpc/docs/subnets#expand-subnet) | [Valid subnet ranges](https://cloud.google.com/vpc/docs/subnets#valid-ranges) | 🖥️ **Console:** VPC network → VPC networks → select VPC → Subnets tab → select subnet → Edit

### Expanding Subnet CIDR Range

You can **expand** a subnet's primary IP range without downtime. This is useful when you're running out of IP addresses.

```bash
# Expand a subnet (increase the range)
gcloud compute networks subnets expand-ip-range my-subnet \
  --region=us-central1 \
  --prefix-length=20

# Example: Expand from /24 (256 IPs) to /20 (4,096 IPs)
# Original: 10.0.1.0/24 → Expanded: 10.0.0.0/20
```

### Rules for Subnet Expansion
- You can only **expand** (make larger), never shrink
- The new range must **contain the original range**
- The operation is **non-disruptive** (no downtime)
- Cannot overlap with other subnets
- The expanded range must be a valid CIDR block

### CIDR Reference

| CIDR | Subnet Mask | Usable IPs | Example |
|------|-------------|------------|---------|
| /28 | 255.255.255.240 | 12 | Small test environments |
| /24 | 255.255.255.0 | 252 | Standard subnet |
| /20 | 255.255.240.0 | 4,092 | Medium workloads |
| /16 | 255.255.0.0 | 65,532 | Large workloads |
| /12 | 255.240.0.0 | 1,048,572 | Very large workloads |

**Note**: Google reserves 4 IPs per subnet (network, gateway, reserved, broadcast), so usable IPs = total - 4.

---

## 3. Reserving Static IP Addresses

> 📖 **Docs:** [Reserve a static external IP](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address) | [Reserve a static internal IP](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-internal-ip-address) | 🖥️ **Console:** VPC network → IP addresses → Reserve External/Internal Static Address

### Types of IP Addresses

| Type | Scope | Use Case |
|------|-------|----------|
| **Ephemeral external** | Releases when VM stops | Dev/test VMs |
| **Static external** | Persists independently | Production VMs, DNS, load balancers |
| **Ephemeral internal** | Auto-assigned from subnet range | Standard VMs |
| **Static internal** | Specific internal IP reserved | Databases, DNS servers |

### Reserving External Static IPs

```bash
# Reserve a regional external IP
gcloud compute addresses create my-external-ip \
  --region=us-central1

# Reserve a global external IP (for global load balancers)
gcloud compute addresses create my-global-ip \
  --global

# Reserve with a specific network tier
gcloud compute addresses create standard-ip \
  --region=us-central1 \
  --network-tier=STANDARD

# List reserved addresses
gcloud compute addresses list

# Describe a reserved address
gcloud compute addresses describe my-external-ip --region=us-central1

# Get just the IP address
gcloud compute addresses describe my-external-ip \
  --region=us-central1 \
  --format="get(address)"
```

### Assigning a Static External IP to a VM

```bash
# At creation time
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --address=my-external-ip

# To an existing VM (must delete and recreate the access config)
gcloud compute instances delete-access-config my-vm \
  --zone=us-central1-a \
  --access-config-name="External NAT"

gcloud compute instances add-access-config my-vm \
  --zone=us-central1-a \
  --address=MY_STATIC_IP
```

### Reserving Internal Static IPs

```bash
# Reserve a specific internal IP from a subnet
gcloud compute addresses create my-internal-ip \
  --region=us-central1 \
  --subnet=my-subnet \
  --addresses=10.0.1.50

# Reserve any available internal IP from a subnet
gcloud compute addresses create auto-internal-ip \
  --region=us-central1 \
  --subnet=my-subnet

# Use the reserved internal IP when creating a VM
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --network-interface=network=my-vpc,subnet=my-subnet,private-network-ip=10.0.1.50,no-address
```

### Releasing Static IPs

```bash
# Release (delete) a reserved IP
gcloud compute addresses delete my-external-ip --region=us-central1

# Release a global IP
gcloud compute addresses delete my-global-ip --global
```

**Important**: Unused reserved static IPs incur charges. Release them when no longer needed.

---

## 4. Working with Cloud DNS

> 📖 **Docs:** [Cloud DNS overview](https://cloud.google.com/dns/docs/overview) | [Create zones and records](https://cloud.google.com/dns/docs/zones) | [DNS best practices](https://cloud.google.com/dns/docs/best-practices) | 🖥️ **Console:** Cloud DNS → Create Zone

### What Is Cloud DNS?
- **Managed authoritative DNS** service
- 100% uptime SLA for authoritative name serving
- Supports public and private zones
- Supports DNSSEC

### DNS Zone Types

| Type | Visibility | Use Case |
|------|-----------|----------|
| **Public** | Internet | External-facing services |
| **Private** | VPC networks only | Internal services |
| **Forwarding** | VPC → on-premises DNS | Hybrid cloud |
| **Peering** | VPC → another VPC's DNS | Cross-project DNS |

### Managing Public DNS Zones

```bash
# Create a public DNS zone
gcloud dns managed-zones create my-zone \
  --dns-name=example.com. \
  --description="My public DNS zone" \
  --visibility=public

# List zones
gcloud dns managed-zones list

# Describe a zone
gcloud dns managed-zones describe my-zone
```

### Managing DNS Records

```bash
# Start a DNS transaction
gcloud dns record-sets transaction start --zone=my-zone

# Add an A record
gcloud dns record-sets transaction add "203.0.113.1" \
  --name=www.example.com. \
  --ttl=300 \
  --type=A \
  --zone=my-zone

# Add a CNAME record
gcloud dns record-sets transaction add "example.com." \
  --name=blog.example.com. \
  --ttl=300 \
  --type=CNAME \
  --zone=my-zone

# Add an MX record
gcloud dns record-sets transaction add "10 mail.example.com." \
  --name=example.com. \
  --ttl=300 \
  --type=MX \
  --zone=my-zone

# Execute the transaction
gcloud dns record-sets transaction execute --zone=my-zone

# Abort a transaction
gcloud dns record-sets transaction abort --zone=my-zone

# List records in a zone
gcloud dns record-sets list --zone=my-zone

# Delete a record (via transaction)
gcloud dns record-sets transaction start --zone=my-zone
gcloud dns record-sets transaction remove "203.0.113.1" \
  --name=www.example.com. \
  --ttl=300 \
  --type=A \
  --zone=my-zone
gcloud dns record-sets transaction execute --zone=my-zone
```

### Managing Private DNS Zones

```bash
# Create a private DNS zone (visible only within specified VPCs)
gcloud dns managed-zones create my-private-zone \
  --dns-name=internal.example.com. \
  --description="Internal DNS" \
  --visibility=private \
  --networks=my-vpc

# Add the zone to additional VPCs
gcloud dns managed-zones update my-private-zone \
  --networks=my-vpc,my-other-vpc
```

### DNS Forwarding (Hybrid Cloud)

```bash
# Create a forwarding zone (forward queries to on-premises DNS)
gcloud dns managed-zones create on-prem-zone \
  --dns-name=corp.example.com. \
  --description="Forward to on-prem DNS" \
  --visibility=private \
  --networks=my-vpc \
  --forwarding-targets=192.168.1.10,192.168.1.11
```

### Common DNS Record Types

| Type | Description | Example |
|------|-------------|---------|
| **A** | IPv4 address | `www.example.com → 203.0.113.1` |
| **AAAA** | IPv6 address | `www.example.com → 2001:db8::1` |
| **CNAME** | Canonical name (alias) | `blog.example.com → example.com` |
| **MX** | Mail exchange | `example.com → 10 mail.example.com` |
| **TXT** | Text record | SPF, DKIM, domain verification |
| **NS** | Name server | Zone delegation |
| **SOA** | Start of authority | Zone metadata |
| **SRV** | Service location | Service discovery |
| **PTR** | Reverse DNS | IP → hostname |

---

## 5. Working with Cloud NAT

> 📖 **Docs:** [Cloud NAT overview](https://cloud.google.com/nat/docs/overview) | [Create Cloud NAT](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation) | 🖥️ **Console:** Cloud NAT → Create Cloud NAT Gateway

### What Is Cloud NAT?
- Provides **outbound internet access** for VMs without external IPs
- Software-defined, fully managed (no NAT VMs or appliances)
- Works with Compute Engine and GKE
- **Regional resource** associated with a Cloud Router

### Setting Up Cloud NAT

```bash
# Step 1: Create a Cloud Router (if not already created)
gcloud compute routers create my-router \
  --network=my-vpc \
  --region=us-central1

# Step 2: Create Cloud NAT configuration
gcloud compute routers nats create my-nat \
  --router=my-router \
  --region=us-central1 \
  --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges

# Create NAT for specific subnets only
gcloud compute routers nats create my-nat \
  --router=my-router \
  --region=us-central1 \
  --auto-allocate-nat-external-ips \
  --nat-custom-subnet-ip-ranges=my-subnet

# Create NAT with manual IP allocation
gcloud compute routers nats create my-nat \
  --router=my-router \
  --region=us-central1 \
  --nat-external-ip-pool=my-nat-ip-1,my-nat-ip-2 \
  --nat-all-subnet-ip-ranges
```

### Managing Cloud NAT

```bash
# List NAT configurations
gcloud compute routers nats list --router=my-router --region=us-central1

# Describe a NAT configuration
gcloud compute routers nats describe my-nat \
  --router=my-router \
  --region=us-central1

# Update NAT configuration
gcloud compute routers nats update my-nat \
  --router=my-router \
  --region=us-central1 \
  --min-ports-per-vm=128 \
  --max-ports-per-vm=4096

# Enable logging
gcloud compute routers nats update my-nat \
  --router=my-router \
  --region=us-central1 \
  --enable-logging \
  --log-filter=ALL

# Delete NAT configuration
gcloud compute routers nats delete my-nat \
  --router=my-router \
  --region=us-central1
```

### Cloud NAT Key Properties
- **Outbound only** — External traffic cannot initiate connections to NAT'd VMs
- **No proxy** — Traffic is not proxied; source IP is rewritten at the network layer
- **Per-region** — Each region needs its own Cloud NAT configuration
- **Port allocation** — Controls how many outbound connections each VM can make
- **Logging** — Can log NAT translations for monitoring and troubleshooting

### Cloud NAT vs. External IP

| Feature | Cloud NAT | External IP on VM |
|---------|-----------|-------------------|
| Outbound internet | Yes | Yes |
| Inbound from internet | No | Yes |
| IP management | Centralized | Per-VM |
| Security | Higher (no public exposure) | Lower (publicly addressable) |
| Cost | NAT gateway charges | IP address charges |
| Use case | VMs that need internet but not exposure | Public-facing servers |

---

## 8. Firewall Rules Management

> 📖 **Docs:** [VPC firewall rules overview](https://cloud.google.com/vpc/docs/firewalls) | [Create firewall rules](https://cloud.google.com/vpc/docs/using-firewalls) | [Firewall policies](https://cloud.google.com/vpc/docs/network-firewall-policies) | 🖥️ **Console:** VPC network → Firewall → Create Firewall Rule

```bash
# Create ingress rule
gcloud compute firewall-rules create allow-http \
  --network=MY_VPC \
  --direction=INGRESS \
  --priority=1000 \
  --action=ALLOW \
  --rules=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server

# Create egress deny rule
gcloud compute firewall-rules create deny-all-egress \
  --network=MY_VPC \
  --direction=EGRESS \
  --priority=65534 \
  --action=DENY \
  --rules=all

# Update existing rule
gcloud compute firewall-rules update allow-http --priority=900

# List rules sorted by priority
gcloud compute firewall-rules list --sort-by=priority --filter="network=MY_VPC"

# Delete rule
gcloud compute firewall-rules delete allow-http
```

Key concepts:
- Implied rules cannot be deleted: default-deny-all-ingress (priority 65535) and default-allow-all-egress (priority 65535)
- Lower priority number = evaluated first
- DENY at same priority beats ALLOW
- Target service account (instead of tag): more secure, prevents tag spoofing
  ```bash
  --target-service-accounts=SA@PROJECT.iam.gserviceaccount.com
  --source-service-accounts=SA@PROJECT.iam.gserviceaccount.com
  ```

---

## 8b. Routes Management

Every VPC has **system-generated routes** (default internet gateway route, subnet routes for each subnet). You can add **custom static routes** or dynamic routes (via Cloud Router/BGP).

```bash
# Create a static route (send a destination range to a next-hop VM)
gcloud compute routes create route-to-appliance \
  --network=MY_VPC \
  --destination-range=10.100.0.0/16 \
  --next-hop-instance=nat-gateway-vm \
  --next-hop-instance-zone=us-central1-a \
  --priority=1000

# Route through an internal load balancer
gcloud compute routes create route-via-ilb \
  --network=MY_VPC \
  --destination-range=0.0.0.0/0 \
  --next-hop-ilb=ilb-forwarding-rule \
  --next-hop-ilb-region=us-central1

# List routes
gcloud compute routes list --filter="network=MY_VPC"

# Delete a route
gcloud compute routes delete route-to-appliance
```

Key concepts:
- **Lowest priority number wins** (same as firewall rules)
- Default route (`0.0.0.0/0`) points to the default internet gateway; can be replaced for forced egress through an appliance
- Subnet routes are auto-created and cannot be deleted while the subnet exists
- Routes are applied based on **network tags** or globally to all VMs in the VPC

---

## 9. VPC Peering Management

> 📖 **Docs:** [VPC Network Peering overview](https://cloud.google.com/vpc/docs/vpc-peering) | [Create VPC peering](https://cloud.google.com/vpc/docs/using-vpc-peering) | 🖥️ **Console:** VPC network → VPC network peering → Create Connection

```bash
# Both sides must create peering connections
gcloud compute networks peerings create peer-a-to-b \
  --network=VPC_A \
  --peer-project=PROJECT_B \
  --peer-network=VPC_B \
  --export-custom-routes \
  --import-custom-routes

# On project B side:
gcloud compute networks peerings create peer-b-to-a \
  --network=VPC_B \
  --peer-project=PROJECT_A \
  --peer-network=VPC_A

# List, describe, delete
gcloud compute networks peerings list --network=VPC_A
gcloud compute networks peerings delete peer-a-to-b --network=VPC_A
```
- Peering is ACTIVE only when both sides are created
- No transitive routing
- Overlapping CIDR ranges cannot be peered
- Exam tip: each VPC can have up to 25 peering connections

---

## 9b. Shared VPC

Shared VPC lets an organization connect resources from multiple projects to a common VPC network. The project that hosts the VPC is the **host project**; the projects whose resources use it are **service projects**.

```bash
# Enable the project as a Shared VPC host (organization admin)
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Attach a service project to the host
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# List service projects attached to a host
gcloud compute shared-vpc list-associated-resources HOST_PROJECT_ID

# Remove a service project
gcloud compute shared-vpc associated-projects remove SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID
```

Key concepts:
- Requires the **Shared VPC Admin** role (`roles/compute.xpnAdmin`) at the organization or folder level
- Service Project Admins are granted `roles/compute.networkUser` on specific subnets in the host project
- Centralizes network administration: networking, firewall rules, and routes live in the host project
- A project can be **either** a host or a service project, not both
- VMs in service projects consume IPs from subnets defined in the host project

---

## 10. Hybrid Connectivity

> 📖 **Docs:** [Cloud VPN overview](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview) | [Cloud Interconnect overview](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/overview) | [Choose a hybrid connectivity option](https://cloud.google.com/network-connectivity/docs/how-to/choose-product) | 🖥️ **Console:** Network Connectivity → Cloud VPN / Cloud Interconnect

### Cloud VPN (HA VPN)

```bash
# Create HA VPN gateway (2 interfaces for 99.99% SLA)
gcloud compute vpn-gateways create my-vpn-gw --network=MY_VPC --region=us-central1
# Create external VPN gateway (peer side)
gcloud compute external-vpn-gateways create peer-vpn-gw \
  --interfaces 0=PEER_IP_0,1=PEER_IP_1
# Create VPN tunnels (one per interface pair)
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-external-gateway=peer-vpn-gw \
  --peer-external-gateway-interface=0 \
  --vpn-gateway=my-vpn-gw --vpn-gateway-region=us-central1 \
  --interface=0 --shared-secret=SECRET \
  --router=MY_ROUTER --region=us-central1
# Configure BGP session on Cloud Router
gcloud compute routers add-bgp-peer MY_ROUTER \
  --peer-name=bgp-peer-1 --peer-ip=169.254.0.2 \
  --peer-asn=65001 --interface=vpn-tunnel-1 --region=us-central1
```

### Cloud Interconnect Comparison

| | Dedicated Interconnect | Partner Interconnect |
|--|--|--|
| Bandwidth | 10 Gbps or 100 Gbps per link | 50 Mbps – 50 Gbps |
| Setup | Direct peering with Google PoP | Via service provider |
| SLA | 99.99% (4 connections) | 99.99% (redundant) |
| Use when | >10 Gbps or strict latency needs | Can't reach Google PoP directly |

Exam tip: Cloud VPN max bandwidth ~3 Gbps per tunnel; for >3 Gbps use Interconnect

---

## Exam Practice Questions

1. **You are running out of IP addresses in a /24 subnet. How can you increase the range without downtime?**
   - Answer: Use `gcloud compute networks subnets expand-ip-range` to expand the prefix length (e.g., from /24 to /20). This is non-disruptive.

2. **You need a stable IP address for a production VM that persists even if the VM is stopped. What should you do?**
   - Answer: **Reserve a static external IP** address and assign it to the VM. Static IPs persist independently of the VM lifecycle.

3. **Internal microservices need to resolve custom DNS names like `api.internal.example.com`. How should you configure this?**
   - Answer: Create a **private DNS zone** in Cloud DNS with `--dns-name=internal.example.com.` and associate it with the VPC. Add A records for each service.

4. **VMs in a private subnet (no external IPs) need to download packages from the internet. What's the best solution?**
   - Answer: Set up **Cloud NAT** on the VPC with a Cloud Router. This provides outbound internet access without exposing VMs to inbound traffic.

5. **You have unused reserved static IP addresses. Are you being charged for them?**
   - Answer: **Yes**. Google Cloud charges for reserved static IP addresses that are not assigned to a resource. Release unused IPs to avoid charges.

6. **You need to forward DNS queries for `corp.company.com` to on-premises DNS servers at 10.0.0.53 and 10.0.0.54. How?**
   - Answer: Create a **Cloud DNS forwarding zone** with `--dns-name=corp.company.com.` and `--forwarding-targets=10.0.0.53,10.0.0.54`, associated with the appropriate VPC.

7. **Can you shrink a subnet's IP range after expanding it?**
   - Answer: **No**. Subnet expansion is a one-way operation. You can only increase the range, never decrease it.

---

## Glossary

**A record**: A DNS resource record type that maps a hostname to an IPv4 address (e.g., `www.example.com → 203.0.113.1`).

**AAAA record**: A DNS resource record type that maps a hostname to an IPv6 address.

**Access config**: The Compute Engine configuration object attached to a VM's network interface that defines the external (public) IP assignment; to change a VM's external IP without recreation you must delete and re-add the access config.

**ASN (Autonomous System Number)**: A globally unique identifier assigned to a network (or group of networks) that share a routing policy, used in BGP peering configurations for Cloud VPN and Cloud Interconnect.

**Auto mode VPC**: A VPC network mode in which Google automatically creates one subnet per region with predefined IP ranges; contrasted with custom mode VPCs.

**BGP (Border Gateway Protocol)**: A dynamic routing protocol used by Cloud Router and Cloud VPN tunnels to exchange routes between the Google Cloud network and on-premises or peer networks.

**BGP peer**: A neighbor router that exchanges routes over a BGP session; configured on Cloud Router with `gcloud compute routers add-bgp-peer` and identified by a peer IP and peer ASN.

**CIDR (Classless Inter-Domain Routing)**: A compact notation for specifying IP address ranges (e.g., `10.0.0.0/24`), where the suffix indicates the number of fixed bits in the network prefix.

**Cloud DNS**: Google Cloud's fully managed, authoritative DNS service that provides 100% uptime SLA for name serving and supports public zones, private zones, forwarding zones, and DNSSEC.

**Cloud Interconnect**: A Google Cloud service that provides direct, high-bandwidth, low-latency physical connectivity between an on-premises network and Google's network, available as Dedicated or Partner Interconnect.

**Cloud NAT (Network Address Translation)**: A fully managed, software-defined Google Cloud service that provides outbound internet access for VMs and GKE nodes that have no external IP addresses, without exposing them to inbound connections.

**Cloud Router**: A fully managed Google Cloud service that dynamically exchanges routes between a VPC network and on-premises networks using BGP; required by Cloud VPN (HA VPN) and Cloud NAT.

**Cloud VPN**: A Google Cloud service that provides encrypted connectivity between a VPC network and an on-premises or another cloud network over the public internet using IPsec tunnels; HA VPN offers 99.99% SLA.

**CNAME record**: A DNS resource record type that maps an alias hostname to the canonical (true) hostname of another domain.

**Custom mode VPC**: A VPC network mode where subnets must be created manually with user-defined IP ranges, giving full control over subnet regions and address spaces.

**Dedicated Interconnect**: A Cloud Interconnect option that provides direct physical connections of 10 Gbps or 100 Gbps per link between an on-premises data center and a Google Point of Presence (PoP).

**DKIM (DomainKeys Identified Mail)**: An email authentication method used to sign outgoing email; the public key is published as a TXT DNS record.

**DNS (Domain Name System)**: The hierarchical naming system that translates human-readable domain names (e.g., `example.com`) into IP addresses.

**DNSSEC (DNS Security Extensions)**: A set of extensions that add cryptographic signatures to DNS records, allowing resolvers to verify that DNS responses have not been tampered with.

**DNS forwarding zone**: A Cloud DNS zone type that forwards DNS queries for a given domain to specified external DNS server IP addresses, used for hybrid cloud DNS resolution.

**DNS peering zone**: A Cloud DNS zone type that allows one VPC to use DNS zones from another VPC, enabling cross-project DNS resolution.

**DNS transaction**: A Cloud DNS mechanism for atomically staging and applying multiple record changes (add, remove) to a managed zone using `transaction start`, `add/remove`, and `execute` commands.

**Default internet gateway**: The implicit next-hop for the `0.0.0.0/0` system route in every VPC; can be replaced by a custom static route to force egress traffic through a NAT appliance or VPN.

**Egress**: Network traffic that leaves a VPC network or Google's network; firewall egress rules control outbound traffic from VMs.

**Ephemeral external IP**: A public IP address automatically assigned to a VM that is released when the VM is stopped; not suitable for production use cases requiring a stable address.

**Ephemeral internal IP**: A private IP address automatically assigned from the subnet range each time a VM starts; not suitable when a stable internal address is required.

**External VPN gateway**: A Compute Engine resource that represents the peer side of a Cloud VPN connection (e.g., an on-premises VPN device), storing the peer's public IP addresses used by HA VPN tunnels.

**Firewall rule**: A VPC-level rule that allows or denies traffic to or from VMs based on protocol, port, source, and destination, applied by priority number.

**Flow sampling (VPC Flow Logs)**: The fraction of network flows captured by VPC Flow Logs (between 0.0 and 1.0), controlling the overhead and volume of logged traffic.

**Forwarding rule**: A Google Cloud networking object that directs traffic matching a given IP address and protocol to a target (such as a load balancer backend or another VPN gateway).

**Global external IP**: A static external IP address that is not tied to a single region; used by global load balancers and forwarding rules that serve traffic worldwide.

**GKE (Google Kubernetes Engine)**: Google Cloud's managed Kubernetes service; subnets used for GKE require secondary IP ranges for pod and service addresses.

**HA VPN (High Availability VPN)**: A Cloud VPN configuration with two gateway interfaces that provides a 99.99% uptime SLA when paired with a redundant peer gateway and two VPN tunnels.

**Host project (Shared VPC)**: The project in a Shared VPC configuration that owns the VPC network, subnets, routes, and firewall rules, and allows resources in attached service projects to consume them.

**IAM (Identity and Access Management)**: Google Cloud's system for granting permissions to principals at the project, folder, or organization level; referenced in firewall rules through target service accounts.

**Implied firewall rule**: One of two built-in firewall rules (implied allow all egress and implied deny all ingress) that exist in every VPC at priority 65535 and cannot be deleted, only overridden by higher-priority explicit rules.

**Ingress**: Network traffic that enters a VPC network from an external source; firewall ingress rules control inbound traffic to VMs.

**Internal load balancer (ILB)**: A regional Google Cloud load balancer that distributes traffic within a VPC using private IP addresses; can also be used as a next-hop target for custom static routes.

**IPsec (Internet Protocol Security)**: The tunneling and encryption protocol used by Cloud VPN to secure traffic traveling over the public internet between two endpoints.

**Logging aggregation interval**: A VPC Flow Logs setting (e.g., `interval-5-sec`) that controls how frequently captured flows are aggregated and emitted to Cloud Logging.

**Managed zone**: A Cloud DNS resource representing a DNS zone with its records, visibility (public or private), and associated VPCs for private zones.

**MX record**: A DNS resource record type that specifies the mail server(s) responsible for accepting email for a domain, along with a priority value.

**NAT (Network Address Translation)**: The process of rewriting the source IP address of outbound packets from a private IP to a public IP; Cloud NAT performs this at the network layer without a proxy.

**Network tag**: A character-string label attached to a VM used as the source or target in firewall rules and routes; less secure than service-account-based targeting because tags can be self-applied (tag spoofing).

**Network tier**: A Google Cloud feature that controls whether a VM's traffic uses the premium global backbone (`PREMIUM`) or the standard regional network (`STANDARD`); affects performance and egress cost.

**NS record**: A DNS resource record type that delegates authority for a zone to specific name servers.

**Partner Interconnect**: A Cloud Interconnect option that provides connectivity to Google's network through a supported service provider, with bandwidths from 50 Mbps to 50 Gbps.

**Point of Presence (PoP)**: A physical facility where Google's network meets the broader internet and where Dedicated Interconnect connections are terminated.

**Port allocation**: A Cloud NAT setting that controls the minimum and maximum number of source ports available per VM for outbound connections, affecting the total number of simultaneous connections.

**Prefix length**: The number of bits in a CIDR block that represent the network portion (e.g., `/24`); smaller prefix length means a larger address range.

**Priority (firewall rule)**: An integer (0–65535) that determines the evaluation order of firewall rules and routes; lower numbers are evaluated first, and at equal priority a DENY rule takes precedence over ALLOW.

**Private DNS zone**: A Cloud DNS managed zone visible only to specified VPC networks, used for resolving internal hostnames that should not be exposed on the public internet.

**Private Google Access**: A subnet-level setting that allows VMs without external IP addresses to reach Google APIs and services (e.g., Cloud Storage, BigQuery) over Google's internal network.

**PTR record**: A DNS resource record type used for reverse DNS lookups, mapping an IP address back to a hostname.

**Public DNS zone**: A Cloud DNS managed zone that is accessible from the public internet, used for external-facing services.

**Record set**: A Cloud DNS object containing one or more records of the same type for the same name (e.g., two A records for `www.example.com`).

**Regional external IP**: A static external IP address reserved in a specific Google Cloud region; used for regional resources such as regional load balancers and VMs.

**Regional resource**: A GCP resource that exists in a specific region and spans all zones of that region; Cloud NAT and Cloud Router are examples, whereas subnets are also regional.

**RFC 1918**: The IETF standard defining the private IP address ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) used inside VPCs and subnets.

**Route**: A VPC-level object that defines the next hop for traffic destined to a given IP range; routes can be system-generated (default internet gateway, subnet routes) or user-created static or dynamic routes.

**Secondary IP range**: An additional IP address range configured on a GKE subnet to provide addresses for Kubernetes pods and services, separate from the primary subnet range used by VMs.

**Service project (Shared VPC)**: A project attached to a Shared VPC host project whose resources (VMs, GKE clusters, etc.) can use the host's VPC networks and subnets.

**Shared secret (VPN)**: A pre-shared key (PSK) used by both ends of an IPsec VPN tunnel to authenticate each other during tunnel establishment.

**Shared VPC**: A Google Cloud feature that allows multiple projects (service projects) to share a common VPC network hosted in a central project (host project), centralizing network administration across the organization.

**SLA (Service Level Agreement)**: A contractual commitment to a minimum level of service availability; Cloud DNS offers 100% SLA for authoritative serving, and HA VPN offers 99.99%.

**SOA record**: The Start of Authority DNS record that contains metadata about a DNS zone, including the primary name server, contact email, and timing parameters.

**SPF (Sender Policy Framework)**: An email authentication mechanism published as a TXT DNS record that specifies which mail servers are authorized to send email for a domain.

**SRV record**: A DNS resource record type that specifies the location (host and port) of a service, used for service discovery.

**Static external IP**: A public IP address reserved independently of any VM that persists until explicitly released; billed even when not attached to a resource.

**Static internal IP**: A private IP address from a subnet range explicitly reserved for a specific resource such as a database server or internal DNS server, ensuring it does not change.

**Static route**: A user-defined route in a VPC that forwards traffic for a specific destination IP range to a next hop such as a VM, internal load balancer, or VPN tunnel.

**Subnet**: A regional IP address range within a VPC network to which VM instances are attached; subnets can only be expanded (not shrunk) after creation.

**Tag spoofing**: The act of a malicious VM applying a network tag it should not have in order to match permissive firewall rules; mitigated by using target service accounts in firewall rules instead of tags.

**Target service account**: A firewall rule targeting mechanism that applies a rule to VMs running under a specific service account; more secure than network tags because service accounts cannot be self-assigned.

**Target tag**: A firewall rule targeting mechanism that applies a rule to VMs with a matching network tag; less secure than using target service accounts.

**Transitive routing**: The ability of network A to reach network C through network B; VPC peering does not support transitive routing — each VPC pair must create its own peering connection.

**TTL (Time to Live)**: The number of seconds a DNS resolver should cache a record before re-querying the authoritative server; lower values allow faster propagation of changes.

**TXT record**: A DNS resource record type that holds arbitrary text, commonly used for domain verification, SPF, and DKIM records.

**VPC (Virtual Private Cloud)**: Google Cloud's isolated virtual network that provides private IP space, routing, firewall rules, and connectivity services for cloud resources.

**VPC Flow Logs**: A feature that captures a sample of network traffic flowing through a subnet's VM interfaces and sends it to Cloud Logging, used for network analysis and security monitoring.

**VPC peering**: A networking connection between two VPC networks (in the same or different projects or organizations) that enables private IP communication; requires both sides to create a peering connection and does not support transitive routing.

**VPN tunnel**: The encrypted IPsec connection between a Cloud VPN gateway and a peer VPN gateway, carrying traffic between a VPC and an external network.

**xpnAdmin (`roles/compute.xpnAdmin`)**: The Shared VPC Admin role that allows designating a project as a Shared VPC host and attaching service projects; granted at the organization or folder level.
