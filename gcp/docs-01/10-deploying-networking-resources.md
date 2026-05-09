# Section 3.5 — Deploying and Implementing Networking Resources

## Exam Relevance
This topic is part of **Section 2: Planning and implementing a cloud solution (~30 % of the exam)**. You must know how to create VPCs with subnets, configure Cloud NGFW policies with secure Tags and service accounts, set up VPN, VPC peering, and Cloud Interconnect, and differentiate Network Service Tiers.

---

## 1. Creating a VPC with Subnets

> 📖 **Docs:** [Create and modify VPC networks](https://cloud.google.com/vpc/docs/create-modify-vpc-networks) | [VPC network overview](https://cloud.google.com/vpc/docs/vpc) | 🖥️ **Console:** VPC Network → VPC networks → Create VPC network

### VPC Fundamentals
- A VPC is a **global resource** that spans all GCP regions
- Subnets are **regional resources** (span all zones within a region)
- Every GCP project has a **default VPC** with auto-mode subnets

### Auto Mode vs. Custom Mode VPC

| Feature | Auto Mode | Custom Mode |
|---------|-----------|-------------|
| Subnet creation | Automatic (one per region) | Manual |
| IP ranges | Predefined (10.128.0.0/9) | You choose |
| New region subnets | Added automatically | Must add manually |
| Production use | Not recommended | Recommended |
| Control | Limited | Full |

### Creating VPCs

```bash
# Create a custom mode VPC
gcloud compute networks create my-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional

# Create an auto mode VPC
gcloud compute networks create my-auto-vpc \
  --subnet-mode=auto

# Convert auto mode to custom mode (one-way, irreversible)
gcloud compute networks update my-auto-vpc \
  --switch-to-custom-subnet-mode

# Delete a VPC
gcloud compute networks delete my-vpc
```

### Creating Subnets

```bash
# Create a subnet
gcloud compute networks subnets create my-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24

# Create a subnet with secondary ranges (for GKE pods/services)
gcloud compute networks subnets create gke-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.0.0/24 \
  --secondary-range=pods=10.4.0.0/14,services=10.8.0.0/20

# Create a subnet with Private Google Access
gcloud compute networks subnets create private-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24 \
  --enable-private-ip-google-access

# Create a subnet with flow logs
gcloud compute networks subnets create logged-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.3.0/24 \
  --enable-flow-logs

# List subnets
gcloud compute networks subnets list --network=my-vpc

# Describe a subnet
gcloud compute networks subnets describe my-subnet --region=us-central1
```

### Private Google Access
- Allows VMs with **only internal IPs** to access Google APIs and services
- Required for VMs without external IPs to reach Cloud Storage, BigQuery, etc.
- Enabled at the **subnet level**

```bash
# Enable Private Google Access on a subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access
```

### VPC Flow Logs
- Capture network flow data for monitoring and forensics
- Logs IP, port, protocol, bytes, packets for each flow
- Can export to Cloud Logging, BigQuery, or Cloud Storage

---

## 2. Shared VPC

> 📖 **Docs:** [Shared VPC overview](https://cloud.google.com/vpc/docs/shared-vpc) | [Set up Shared VPC](https://cloud.google.com/vpc/docs/provisioning-shared-vpc) | 🖥️ **Console:** VPC Network → Shared VPC

### What Is Shared VPC?
Shared VPC allows an **organization** to share a VPC network across multiple projects.

```
Organization
└── Host Project (owns the Shared VPC)
    ├── Shared VPC Network
    │   ├── Subnet A (us-central1)
    │   └── Subnet B (europe-west1)
    │
    ├── Service Project 1 (uses subnets from Shared VPC)
    ├── Service Project 2 (uses subnets from Shared VPC)
    └── Service Project 3 (uses subnets from Shared VPC)
```

### Key Concepts
- **Host Project** — Contains the Shared VPC network and subnets
- **Service Projects** — Projects that use subnets from the host project
- Network administration is centralized in the host project
- Service projects can create resources (VMs, GKE, etc.) in shared subnets

### Setting Up Shared VPC

```bash
# Enable Shared VPC on the host project
gcloud compute shared-vpc enable HOST_PROJECT_ID

# Associate a service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# Grant a user permission to use shared subnets
gcloud projects add-iam-policy-binding HOST_PROJECT_ID \
  --member="user:developer@example.com" \
  --role="roles/compute.networkUser"

# List service projects
gcloud compute shared-vpc list-associated-resources HOST_PROJECT_ID
```

### When to Use Shared VPC
- Centralized network management
- Consistent firewall rules across projects
- Different teams/projects needing the same network
- Billing separation with shared networking

---

## 3. Firewall Rules and Policies

> 📖 **Docs:** [VPC firewall rules](https://cloud.google.com/vpc/docs/firewalls) | [Create firewall rules](https://cloud.google.com/vpc/docs/using-firewalls) | [Firewall policies](https://cloud.google.com/vpc/docs/firewall-policies) | 🖥️ **Console:** VPC Network → Firewall / Firewall Policies

### Firewall Rule Basics
- Applied at the **VPC level**
- **Stateful** — If ingress is allowed, return traffic is automatically allowed
- Default rules: Allow egress to all, deny ingress from all (except from within the VPC)
- Rules are evaluated by **priority** (0 = highest priority, 65535 = lowest)

### Creating Firewall Rules

```bash
# Allow HTTP traffic from any source
gcloud compute firewall-rules create allow-http \
  --network=my-vpc \
  --allow=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server \
  --priority=1000 \
  --direction=INGRESS

# Allow HTTPS traffic
gcloud compute firewall-rules create allow-https \
  --network=my-vpc \
  --allow=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=https-server \
  --priority=1000 \
  --direction=INGRESS

# Allow SSH from a specific IP range
gcloud compute firewall-rules create allow-ssh-office \
  --network=my-vpc \
  --allow=tcp:22 \
  --source-ranges=203.0.113.0/24 \
  --priority=1000 \
  --direction=INGRESS

# Allow internal traffic between subnets
gcloud compute firewall-rules create allow-internal \
  --network=my-vpc \
  --allow=tcp,udp,icmp \
  --source-ranges=10.0.0.0/8 \
  --priority=1000 \
  --direction=INGRESS

# Allow IAP tunneling for SSH
gcloud compute firewall-rules create allow-iap-ssh \
  --network=my-vpc \
  --allow=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --priority=1000 \
  --direction=INGRESS

# Deny all egress to a specific destination
gcloud compute firewall-rules create deny-external \
  --network=my-vpc \
  --action=DENY \
  --rules=all \
  --destination-ranges=0.0.0.0/0 \
  --priority=1000 \
  --direction=EGRESS

# List firewall rules
gcloud compute firewall-rules list --filter="network=my-vpc"

# Describe a firewall rule
gcloud compute firewall-rules describe allow-http

# Delete a firewall rule
gcloud compute firewall-rules delete allow-http
```

### Firewall Rule Components

| Component | Description | Example |
|-----------|-------------|---------|
| **Direction** | INGRESS (incoming) or EGRESS (outgoing) | `--direction=INGRESS` |
| **Priority** | 0-65535 (lower = higher priority) | `--priority=1000` |
| **Action** | ALLOW or DENY | `--allow=tcp:80` or `--action=DENY` |
| **Targets** | Which instances the rule applies to | Tags, service accounts, or all |
| **Source (ingress)** | Where traffic comes from | IP ranges, tags, SAs |
| **Destination (egress)** | Where traffic goes to | IP ranges |
| **Protocol/Port** | Which traffic | `tcp:80`, `udp:53`, `icmp` |

### Target Specification Methods

#### 1. Network Tags

```bash
# Rule targets instances with the tag "web-server"
gcloud compute firewall-rules create allow-web \
  --network=my-vpc \
  --allow=tcp:80,tcp:443 \
  --target-tags=web-server

# Apply tag to an instance
gcloud compute instances create my-vm \
  --tags=web-server \
  --zone=us-central1-a
```

#### 2. Service Accounts

```bash
# Rule targets instances running with a specific service account
gcloud compute firewall-rules create allow-db-access \
  --network=my-vpc \
  --allow=tcp:3306 \
  --target-service-accounts=db-server@PROJECT_ID.iam.gserviceaccount.com \
  --source-service-accounts=web-server@PROJECT_ID.iam.gserviceaccount.com
```

#### 3. All Instances (No Target Specified)
If no target is specified, the rule applies to **all instances** in the VPC.

### Network Tags vs. Service Accounts for Firewall Rules

| Feature | Network Tags | Service Accounts |
|---------|-------------|-----------------|
| Management | Applied per instance | IAM-managed |
| Security | Any instance admin can change tags | Requires IAM permission to assign |
| Granularity | Instance-level | Service identity level |
| Recommended | Simple setups | Production (more secure) |

### Hierarchical Firewall Policies
- Applied at the **organization or folder level**
- Inherited downward through the hierarchy
- Evaluated **before** VPC firewall rules
- Useful for organization-wide security policies

---

## 3b. Cloud Next Generation Firewall (Cloud NGFW)

Cloud NGFW is GCP's advanced stateful firewall that supersedes the legacy VPC firewall rules model. It uses **firewall policies** (not individual rules) and supports **secure Tags** and service accounts for more granular, scalable control.

### Cloud NGFW vs. Legacy Firewall Rules

| Feature | Legacy VPC Firewall Rules | Cloud NGFW Policies |
|---------|--------------------------|---------------------|
| Scope | Per-VPC | Hierarchical (org/folder/project/VPC) |
| Tags | Network tags (any VM admin can set) | Secure Tags (IAM-controlled) |
| Policy type | Flat rules | Ordered policies with rules |
| L7 inspection | No | Yes (NGFW Enterprise) |
| Threat prevention | No | Yes (NGFW Enterprise) |
| Address groups | No | Yes |
| Association | Auto-applied to VPC | Explicitly associated with VPC/folder/org |

### Firewall Policy Types

| Policy Type | Scope | Inheritance |
|-------------|-------|-------------|
| **Hierarchical policy** | Organization or folder | Inherited by all child resources |
| **VPC network policy** | Single VPC | Applied only to that VPC |
| **Regional network policy** | VPC in a specific region | Applied to regional resources |

### Creating and Applying Cloud NGFW Policies

```bash
# Create a hierarchical firewall policy at the organization level
gcloud compute firewall-policies create my-org-policy \
  --short-name=my-org-policy \
  --organization=ORG_ID

# Add an ingress rule to the policy (allow HTTPS from anywhere)
gcloud compute firewall-policies rules create 1000 \
  --firewall-policy=my-org-policy \
  --direction=INGRESS \
  --action=allow \
  --layer4-configs=tcp:443 \
  --src-ip-ranges=0.0.0.0/0 \
  --organization=ORG_ID

# Add an egress rule (deny all egress to a destination)
gcloud compute firewall-policies rules create 2000 \
  --firewall-policy=my-org-policy \
  --direction=EGRESS \
  --action=deny \
  --layer4-configs=all \
  --dest-ip-ranges=10.0.0.0/8 \
  --organization=ORG_ID

# Associate the policy with the organization (applies to all projects)
gcloud compute firewall-policies associations create \
  --firewall-policy=my-org-policy \
  --organization=ORG_ID

# Associate the policy with a specific folder
gcloud compute firewall-policies associations create \
  --firewall-policy=my-org-policy \
  --folder=FOLDER_ID

# Create a VPC-level network firewall policy
gcloud compute network-firewall-policies create my-vpc-policy \
  --global

# Add a rule
gcloud compute network-firewall-policies rules create 500 \
  --firewall-policy=my-vpc-policy \
  --direction=INGRESS \
  --action=allow \
  --layer4-configs=tcp:80,tcp:443 \
  --src-ip-ranges=0.0.0.0/0 \
  --global

# Associate the VPC policy with a network
gcloud compute network-firewall-policies associations create \
  --firewall-policy=my-vpc-policy \
  --network=my-vpc \
  --global
```

### Secure Tags in Cloud NGFW

**Secure Tags** are IAM-governed tags that can only be assigned to resources by principals with the `tagUser` role. They provide stronger security than network tags (which any VM admin can change).

```bash
# Create a tag key and value
gcloud resource-manager tags keys create web-tier \
  --parent=organizations/ORG_ID \
  --short-name=web-tier

gcloud resource-manager tags values create production \
  --parent=tagKeys/TAG_KEY_ID \
  --short-name=production

# Bind a secure tag to a VM
gcloud resource-manager tags bindings create \
  --tag-value=tagValues/TAG_VALUE_ID \
  --parent=//compute.googleapis.com/projects/PROJECT/zones/us-central1-a/instances/my-vm

# Use secure tags in a NGFW policy rule (allow traffic to VMs tagged as web-tier/production)
gcloud compute network-firewall-policies rules create 300 \
  --firewall-policy=my-vpc-policy \
  --direction=INGRESS \
  --action=allow \
  --layer4-configs=tcp:80,tcp:443 \
  --src-ip-ranges=0.0.0.0/0 \
  --target-secure-tags=tagValues/TAG_VALUE_ID \
  --global
```

### Service Accounts in Cloud NGFW Policy Rules

Service accounts can be used as both sources and targets in firewall policy rules, providing identity-based filtering that is more secure than IP ranges:

```bash
# Allow traffic from VMs running as "frontend" SA to VMs running as "backend" SA
gcloud compute network-firewall-policies rules create 400 \
  --firewall-policy=my-vpc-policy \
  --direction=INGRESS \
  --action=allow \
  --layer4-configs=tcp:8080 \
  --src-service-accounts=frontend@PROJECT.iam.gserviceaccount.com \
  --target-service-accounts=backend@PROJECT.iam.gserviceaccount.com \
  --global
```

### Rule Evaluation Order

```
Hierarchical Policy (org-level)
  ↓
Hierarchical Policy (folder-level)
  ↓
VPC Network Policy (global)
  ↓
VPC Network Policy (regional)
  ↓
Legacy VPC Firewall Rules
```

Higher-level policies are evaluated first. The first matching rule wins.

### Key Exam Points
- Cloud NGFW is the **preferred** approach for new deployments; legacy VPC firewall rules still work
- **Secure Tags** require IAM (`tagUser` role) to assign; cannot be spoofed by VM admins
- Hierarchical policies enable org-wide security baselines with per-project customization
- **GOTO_NEXT** action allows lower-priority policies to continue evaluation (useful for org defaults)
- NGFW Enterprise adds Layer 7 inspection, threat prevention, and TLS decryption (not in ACE scope)

---

## 4. VPC Network Peering

> 📖 **Docs:** [VPC Network Peering](https://cloud.google.com/vpc/docs/vpc-peering) | [Using VPC Peering](https://cloud.google.com/vpc/docs/using-vpc-peering) | 🖥️ **Console:** VPC Network → VPC Network Peering → Create

### What Is VPC Peering?
- Connects two VPC networks so resources can communicate using **internal IPs**
- Works across projects and organizations
- Traffic stays on Google's network (never traverses the public internet)

### Key Properties
- **Non-transitive** — If VPC A peers with VPC B, and VPC B peers with VPC C, A and C cannot communicate through B
- **Symmetric** — Both sides must configure peering
- **No overlapping IP ranges** — Peered VPCs cannot have overlapping CIDR ranges
- No network bandwidth restrictions (uses Google's backbone)
- Each VPC can have up to **25 peering connections** (configurable)

### Setting Up VPC Peering

```bash
# From VPC-A, peer with VPC-B
gcloud compute networks peerings create peer-a-to-b \
  --network=vpc-a \
  --peer-network=vpc-b \
  --peer-project=PROJECT_B_ID

# From VPC-B, peer with VPC-A (both sides required)
gcloud compute networks peerings create peer-b-to-a \
  --network=vpc-b \
  --peer-network=vpc-a \
  --peer-project=PROJECT_A_ID

# List peering connections
gcloud compute networks peerings list --network=vpc-a

# Delete a peering connection
gcloud compute networks peerings delete peer-a-to-b --network=vpc-a
```

### Export/Import Custom Routes

```bash
# Export custom routes to peered network
gcloud compute networks peerings update peer-a-to-b \
  --network=vpc-a \
  --export-custom-routes

# Import custom routes from peered network
gcloud compute networks peerings update peer-a-to-b \
  --network=vpc-a \
  --import-custom-routes
```

---

## 5. Cloud VPN

> 📖 **Docs:** [Cloud VPN overview](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview) | [Create HA VPN](https://cloud.google.com/network-connectivity/docs/vpn/how-to/creating-ha-vpn) | 🖥️ **Console:** Hybrid Connectivity → VPN → Create VPN connection

### What Is Cloud VPN?
- Connects your on-premises network to GCP VPC over an **encrypted IPsec tunnel** through the public internet

### VPN Types

| Type | SLA | Tunnels | Routing | Use Case |
|------|-----|---------|---------|----------|
| **HA VPN** | 99.99% | 2+ (required for SLA) | Dynamic (BGP) | Production |
| **Classic VPN** | 99.9% | 1 | Static or dynamic | Legacy/simple |

### Setting Up HA VPN

```bash
# Create an HA VPN gateway
gcloud compute vpn-gateways create my-ha-vpn \
  --network=my-vpc \
  --region=us-central1

# Create a Cloud Router (required for dynamic routing)
gcloud compute routers create my-router \
  --network=my-vpc \
  --region=us-central1 \
  --asn=65001

# Create an external VPN gateway (peer)
gcloud compute external-vpn-gateways create my-peer-gateway \
  --interfaces=0=PEER_IP_1,1=PEER_IP_2

# Create VPN tunnels
gcloud compute vpn-tunnels create tunnel-0 \
  --peer-external-gateway=my-peer-gateway \
  --peer-external-gateway-interface=0 \
  --vpn-gateway=my-ha-vpn \
  --vpn-gateway-region=us-central1 \
  --ike-version=2 \
  --shared-secret=SHARED_SECRET \
  --router=my-router \
  --interface=0

gcloud compute vpn-tunnels create tunnel-1 \
  --peer-external-gateway=my-peer-gateway \
  --peer-external-gateway-interface=1 \
  --vpn-gateway=my-ha-vpn \
  --vpn-gateway-region=us-central1 \
  --ike-version=2 \
  --shared-secret=SHARED_SECRET \
  --router=my-router \
  --interface=1

# Configure BGP sessions on the Cloud Router
gcloud compute routers add-interface my-router \
  --interface-name=if-tunnel-0 \
  --vpn-tunnel=tunnel-0 \
  --ip-address=169.254.0.1 \
  --mask-length=30 \
  --region=us-central1

gcloud compute routers add-bgp-peer my-router \
  --peer-name=bgp-peer-0 \
  --interface=if-tunnel-0 \
  --peer-ip-address=169.254.0.2 \
  --peer-asn=65002 \
  --region=us-central1
```

### VPN vs. Interconnect

| Feature | Cloud VPN | Dedicated Interconnect | Partner Interconnect |
|---------|-----------|----------------------|---------------------|
| Connection | Over public internet | Direct physical connection | Via service provider |
| Encryption | IPsec (encrypted) | Not encrypted by default | Not encrypted by default |
| Bandwidth | Up to 3 Gbps per tunnel | 10-200 Gbps | 50 Mbps - 50 Gbps |
| Latency | Variable | Lowest | Low |
| SLA | 99.99% (HA VPN) | 99.9-99.99% | 99.9-99.99% |
| Cost | Lowest | Highest | Medium |
| Setup time | Hours | Weeks | Days |
| Use case | Small/medium traffic, encrypted | Large, predictable workloads | Medium bandwidth needs |

---

## 6. Cloud NAT (Network Address Translation)

> 📖 **Docs:** [Cloud NAT overview](https://cloud.google.com/nat/docs/overview) | [Set up Cloud NAT](https://cloud.google.com/nat/docs/set-up-manage-network-address-translation) | 🖥️ **Console:** Network Services → Cloud NAT → Create NAT gateway

- Allows instances with **no external IP** to initiate **outbound** connections to the internet (e.g., downloading packages, calling external APIs)
- Does **NOT** allow inbound connections — external hosts cannot initiate a connection to a NATted VM
- Statelessly translates outbound traffic; response packets are allowed back automatically
- **Cloud Router is required** — Cloud NAT is configured on top of a Cloud Router

```bash
# Create Cloud Router (required by Cloud NAT)
gcloud compute routers create MY_ROUTER --network=MY_VPC --region=REGION
# Create Cloud NAT
gcloud compute routers nats create MY_NAT --router=MY_ROUTER --region=REGION --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges
# View NAT config
gcloud compute routers nats describe MY_NAT --router=MY_ROUTER --region=REGION
```

- `--auto-allocate-nat-external-ips` — GCP automatically allocates and manages external IPs for NAT
- `--nat-all-subnet-ip-ranges` — applies NAT to all subnets in the VPC; use `--nat-custom-subnet-ip-ranges` to restrict scope

---

## 7. Cloud Armor (DDoS and WAF)

> 📖 **Docs:** [Cloud Armor overview](https://cloud.google.com/armor/docs/cloud-armor-overview) | [Configure security policies](https://cloud.google.com/armor/docs/configure-security-policies) | 🖥️ **Console:** Network Security → Cloud Armor → Create policy

- **Layer 7 security policy** attached to **external HTTP(S) load balancers** (External Application LB)
- Protects against OWASP Top 10 vulnerabilities, volumetric DDoS attacks, and allows IP allowlist/blocklist enforcement

```bash
gcloud compute security-policies create my-policy --description="WAF policy"
gcloud compute security-policies rules create 1000 --security-policy=my-policy --expression="inIpRange(origin.ip, '192.0.2.0/24')" --action=deny-403
gcloud compute security-policies rules create 2000 --security-policy=my-policy --action=allow --expression="true"
gcloud compute backend-services update MY_BACKEND --security-policy=my-policy --global
```

### Key Concepts
- Rules are evaluated in **priority order** (lower number = evaluated first); first matching rule wins
- **Preconfigured WAF rules** use Google-maintained signatures, e.g.:
  ```
  evaluatePreconfiguredWaf('xss-v33-stable')
  evaluatePreconfiguredWaf('sqli-v33-stable')
  ```
- Security policies can have a **default rule** (lowest priority) that acts as the catch-all

> **Exam tip**: Cloud Armor attaches to **backend services**, not to individual VMs, subnets, or firewall rules directly. It only works with external HTTP(S) load balancers, not Network LBs.

---

## 8. Custom Static Routes

> 📖 **Docs:** [Routes overview](https://cloud.google.com/vpc/docs/routes) | [Create static routes](https://cloud.google.com/vpc/docs/using-routes) | 🖥️ **Console:** VPC Network → Routes → Create route

- Route traffic to specific destinations via a chosen **next-hop**
- Evaluated **after** dynamic routes; within the same priority, **most specific prefix wins** (longest prefix match first); ties broken by **priority number** (lower = higher priority)

### Next-Hop Types

| Next-Hop | Use Case |
|----------|----------|
| Instance | Route through a VM (e.g., firewall appliance) |
| IP address | Specific internal IP as gateway |
| VPN tunnel | Route over a Cloud VPN tunnel |
| Interconnect attachment | Route over Dedicated/Partner Interconnect |
| Default internet gateway | Route to the public internet |

```bash
gcloud compute routes create my-route --network=MY_VPC --destination-range=10.100.0.0/16 --next-hop-instance=MY_VM --next-hop-instance-zone=ZONE --priority=1000
gcloud compute routes list --filter="network=MY_VPC"
```

- The **default route** (`0.0.0.0/0 → default-internet-gateway`) is auto-created in every VPC. Delete it to prevent internet access from a VPC (useful for private/restricted environments).

> **Exam tip**: Routes are evaluated by **most specific prefix first**, then by **priority** (lower number = higher priority). A `/28` route always wins over a `/16` route for matching destinations, regardless of priority.

---

## 9. Cloud Load Balancing Overview

> 📖 **Docs:** [Choosing a load balancer](https://cloud.google.com/load-balancing/docs/choosing-load-balancer) | [Load balancing overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview) | 🖥️ **Console:** Network Services → Load balancing → Create load balancer

GCP provides several load balancer types. For ACE, know when to pick which:

| Load Balancer | Scope | Layer | Typical Use |
|---------------|-------|-------|-------------|
| **Global external Application LB** | Global | L7 (HTTP/S) | Public web apps, Cloud CDN, Cloud Armor |
| **Regional external Application LB** | Regional | L7 (HTTP/S) | Region-scoped web apps |
| **Internal Application LB** | Regional | L7 (HTTP/S) | Internal microservices |
| **External proxy Network LB** | Global/Regional | L4 (TCP/SSL proxy) | Non-HTTP TCP traffic needing proxy |
| **External passthrough Network LB** | Regional | L4 (TCP/UDP) | High-performance TCP/UDP, preserves client IP |
| **Internal passthrough Network LB** | Regional | L4 (TCP/UDP) | Internal services, preserves client IP |

### Common LB Components

- **Forwarding rule** — IP + port + protocol; the front door to the LB
- **Target proxy** — For HTTP(S) and SSL/TCP proxy LBs
- **URL map** — Routes HTTP(S) requests to backend services (L7 only)
- **Backend service** — Defines backends (instance groups, NEGs, Cloud Run services) and health checks
- **Health check** — Probes backends to determine which are able to serve traffic
- **Backend** — Instance group (MIG/UIG), zonal/serverless/internet NEG, bucket (for CDN)

```bash
# Example: basic external Application LB for a managed instance group
gcloud compute health-checks create http my-hc --port=80
gcloud compute backend-services create my-backend --global --protocol=HTTP --health-checks=my-hc
gcloud compute backend-services add-backend my-backend --global --instance-group=my-mig --instance-group-region=us-central1
gcloud compute url-maps create my-url-map --default-service=my-backend
gcloud compute target-http-proxies create my-proxy --url-map=my-url-map
gcloud compute forwarding-rules create my-fwd-rule --global --target-http-proxy=my-proxy --ports=80
```

---

## 10. External IP Addresses

> 📖 **Docs:** [Reserve static external IP](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address) | [IP address types](https://cloud.google.com/vpc/docs/ip-addresses) | 🖥️ **Console:** VPC Network → IP addresses → Reserve external static address

```bash
# Reserve a regional static external IP
gcloud compute addresses create my-regional-ip --region=us-central1

# Reserve a global static external IP (for global load balancers)
gcloud compute addresses create my-global-ip --global

# List addresses
gcloud compute addresses list

# Assign a reserved IP to a VM
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --address=my-regional-ip
```

- **Ephemeral IP** — Default for new VMs; released when the VM is deleted or stopped
- **Static IP** — Reserved; persists independently of VMs; billed when unattached

---

## 11. Default VPC Firewall Rules

> 📖 **Docs:** [Default firewall rules](https://cloud.google.com/vpc/docs/firewalls#default_firewall_rules) | [Implied firewall rules](https://cloud.google.com/vpc/docs/firewalls#default-and-implied-rules) | 🖥️ **Console:** VPC Network → Firewall → filter by network

Every new VPC comes with **implied rules** that cannot be deleted:

- **Implied allow egress** — Allows all outbound traffic to any destination (priority 65535)
- **Implied deny ingress** — Denies all inbound traffic (priority 65535)

The **default VPC** additionally has pre-created rules: `default-allow-internal`, `default-allow-ssh`, `default-allow-rdp`, `default-allow-icmp`. These can be deleted. A **custom VPC** has no pre-created rules beyond the implied ones.

---

## 12. Network Service Tiers

> 📖 **Docs:** [Network Service Tiers](https://cloud.google.com/network-tiers/docs/overview) | [Configure tiers](https://cloud.google.com/network-tiers/docs/using-network-service-tiers) | 🖥️ **Console:** VPC Network → Network Service Tiers

Google Cloud offers two **Network Service Tiers** that affect how traffic is routed to and from your resources:

### Premium Tier (Default)
- Traffic travels on **Google's private global backbone** as much as possible
- Traffic enters Google's network at the **nearest PoP** to the user (hot potato routing)
- **Lower latency** and higher performance
- Global load balancers (External Application LB) require Premium tier
- **Higher cost**

### Standard Tier
- Traffic uses the **public internet** for most routing (cold potato routing)
- Traffic enters Google's network **close to the destination region**
- **Higher latency**, variable performance
- Only supports **regional** resources and load balancers
- **Lower cost** — ~25-30% cheaper than Premium for egress

### Comparison

| Feature | Premium Tier | Standard Tier |
|---------|-------------|---------------|
| Routing | Google backbone (hot potato) | Public internet (cold potato) |
| Latency | Lower (Google backbone) | Higher (internet routing) |
| Global LB | Yes | No (regional only) |
| Static IP scope | Global | Regional only |
| Cost | Higher | Lower |
| Best for | Public-facing production apps | Internal/batch traffic, cost-sensitive |

```bash
# Reserve a Premium tier static IP (default)
gcloud compute addresses create premium-ip \
  --region=us-central1 \
  --network-tier=PREMIUM

# Reserve a Standard tier static IP
gcloud compute addresses create standard-ip \
  --region=us-central1 \
  --network-tier=STANDARD

# Create a VM with Standard tier networking
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --network-tier=STANDARD
```

### Key Exam Points
- Default tier is **Premium** — you must explicitly choose Standard to reduce costs
- Standard tier only supports **regional** forwarding rules and static IPs
- Use Standard tier for cost-sensitive non-critical workloads (e.g., batch data processing VMs that need internet access)
- Mixing tiers is possible: some VMs on Premium, others on Standard

---

## Exam Practice Questions

1. **You need to create a VPC where you control all subnet IP ranges and don't want subnets automatically created in new regions. What should you create?**
   - Answer: A **custom mode VPC**. Auto mode creates subnets automatically; custom mode gives you full control.

2. **Two teams in different projects need their VMs to communicate over internal IPs. What's the simplest approach?**
   - Answer: Set up **VPC Network Peering** between the two project VPCs. Traffic uses internal IPs and stays on Google's network.

3. **You need to allow HTTP traffic only to instances tagged as "web-server". How should you configure the firewall rule?**
   - Answer: Create an ingress firewall rule with `--allow=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=web-server`.

4. **Your organization wants to enforce a common set of firewall rules across all projects. Where should these rules be applied?**
   - Answer: Use **hierarchical firewall policies** at the organization or folder level. These are evaluated before VPC-level rules and are inherited by all child projects.

5. **VPC A peers with VPC B, and VPC B peers with VPC C. Can VPC A communicate with VPC C?**
   - Answer: **No**. VPC peering is non-transitive. VPC A cannot reach VPC C through VPC B. You would need a direct peering between A and C.

6. **You need to connect on-premises to GCP with 99.99% SLA and encrypted traffic. Which option should you choose?**
   - Answer: **HA VPN** — Provides 99.99% SLA with IPsec encryption. (Dedicated Interconnect has higher bandwidth but doesn't encrypt by default.)

7. **VMs in a subnet with no external IPs need to access Cloud Storage. What do you need to enable?**
   - Answer: Enable **Private Google Access** on the subnet. This allows VMs with internal-only IPs to access Google APIs and services.

8. **Your security team wants firewall rules that cannot be changed by project-level administrators. What should you use?**
   - Answer: Use **Cloud NGFW hierarchical firewall policies** at the organization or folder level. Project admins cannot override higher-level policies; they are evaluated before project-level rules.

9. **You need to ensure that only VMs with a specific secure Tag can receive traffic on port 8080. How do you configure this with Cloud NGFW?**
   - Answer: Create a **network firewall policy** with an ingress rule on port 8080 that uses `--target-secure-tags` pointing to the appropriate secure Tag value. Secure Tags require IAM (`tagUser` role) to assign — they cannot be spoofed by VM admins.

10. **A batch processing workload requires outbound internet access but doesn't need low latency. How can you reduce networking costs?**
    - Answer: Use **Standard Network Service Tier** for those VMs — it routes traffic via the public internet at lower egress cost than Premium tier, which uses Google's global backbone.

---

## Glossary

**Application Load Balancer** — A GCP Layer 7 (HTTP/HTTPS) load balancer available in global external, regional external, and internal variants. Supports URL-based routing, Cloud CDN, and Cloud Armor.

**ASN (Autonomous System Number)** — A unique number assigned to a network or group of networks under a common routing policy, used in BGP routing to identify peer networks.

**Auto Mode VPC** — A VPC network mode in which Google Cloud automatically creates one subnet per region with predefined IP ranges; not recommended for production use.

**Backend** — An individual endpoint (or group) that a load balancer sends traffic to. Backends can be managed/unmanaged instance groups, zonal/serverless/internet NEGs, or Cloud Storage buckets.

**Backend Service** — A GCP load balancer component that defines how traffic is distributed to backends, including protocol, session affinity, health checks, and the list of backends. Cloud Armor security policies attach to backend services.

**BGP (Border Gateway Protocol)** — A dynamic routing protocol used to exchange routing information between networks; required for HA VPN and Cloud Interconnect dynamic routing.

**CIDR (Classless Inter-Domain Routing)** — A method for allocating IP addresses and routing, expressed as an IP address followed by a slash and a prefix length (e.g., 10.0.0.0/24).

**Classic VPN** — An older GCP VPN gateway type offering 99.9% SLA that supports static or dynamic routing via a single tunnel; considered legacy compared to HA VPN.

**Cloud Armor** — A GCP Layer 7 security service that provides DDoS protection and Web Application Firewall (WAF) capabilities, attached to external HTTP(S) load balancers.

**Cloud CDN** — GCP's content delivery network that caches HTTP(S) responses at Google's edge locations. Enabled on a backend service behind an external Application Load Balancer.

**Cloud Interconnect** — A GCP service providing high-bandwidth, low-latency direct or provider-mediated physical connections between on-premises networks and Google's network.

**Cloud Load Balancing** — GCP's family of fully managed global and regional load balancers, available in Layer 4 (Network LB) and Layer 7 (Application LB) variants, external and internal.

**Cloud Logging** — GCP's managed log storage and analysis service that can receive VPC Flow Log data, firewall log data, and other infrastructure logs.

**Cloud NGFW (Cloud Next Generation Firewall)** — GCP's advanced stateful firewall service that uses hierarchical firewall policies, secure Tags, and service accounts for granular ingress/egress control; supersedes the flat per-VPC firewall rules model.

**Hierarchical Firewall Policy** — A Cloud NGFW policy applied at the organization or folder level that is inherited by all child resources; cannot be overridden by project-level policies.

**Network Service Tiers** — GCP's two-tier networking pricing model: Premium (routes via Google's global backbone, hot-potato routing, lower latency) and Standard (routes via public internet, cold-potato routing, lower cost).

**Secure Tag** — A resource tag managed by IAM (`tagUser` role required to assign); used in Cloud NGFW policy rules for targeting; more secure than network tags because VM admins cannot self-assign them.

**VPC Network Policy** — A Cloud NGFW policy scoped to a single VPC network; explicitly associated with the network and evaluated after hierarchical policies.

**Cloud NAT (Network Address Translation)** — A GCP managed service that allows VM instances without external IP addresses to initiate outbound connections to the internet, while blocking unsolicited inbound connections.

**Cloud Router** — A GCP resource that provides dynamic routing using BGP; required for Cloud NAT and HA VPN dynamic routing.

**Cloud Storage** — GCP's scalable object storage service; used for storing VPC Flow Logs and as a destination for exported logs.

**Cloud VPN** — A GCP service that connects on-premises or other cloud networks to a GCP VPC network over an encrypted IPsec tunnel through the public internet.

**Custom Mode VPC** — A VPC network mode where subnets are created manually with user-defined IP ranges, providing full control over the network topology; recommended for production.

**DDoS (Distributed Denial of Service)** — A type of cyberattack that floods a service with traffic from multiple sources to make it unavailable; Cloud Armor helps mitigate these attacks.

**Dedicated Interconnect** — A GCP service providing a direct physical connection between an on-premises network and Google's network, offering 10–200 Gbps bandwidth without traversing the public internet.

**Default VPC** — A pre-created auto-mode VPC network present in every new GCP project, with one subnet per region and default firewall rules.

**Destination Range** — In firewall rules and routes, the IP address range that identifies the target destination of network traffic.

**Egress** — Network traffic flowing outbound from a resource (e.g., from a VM to the internet or another network).

**Ephemeral IP** — A temporary external IP address automatically assigned to a VM when created; released when the VM is stopped or deleted.

**External IP** — A publicly routable IP address assigned to a GCP resource, allowing it to communicate directly with the internet.

**Firewall Rule** — A VPC-level policy that allows or denies specific network traffic to or from GCP instances based on protocol, port, source/destination, and target.

**Forwarding Rule** — A GCP load balancer component that specifies the IP address, protocol, and port(s) on which the LB accepts traffic. Each forwarding rule routes matching packets to a target proxy or backend service.

**GCP (Google Cloud Platform)** — Google's suite of cloud computing services, including compute, networking, storage, and data services.

**GKE (Google Kubernetes Engine)** — GCP's managed Kubernetes service; GKE pods and services require secondary IP ranges on subnets.

**HA VPN (High Availability VPN)** — A GCP VPN gateway type that provides a 99.99% SLA by requiring two or more tunnels and dynamic BGP routing.

**Health Check** — A GCP probe that periodically tests backend instances to determine whether they are able to serve traffic. Load balancers route traffic only to healthy backends.

**Hierarchical Firewall Policies** — GCP firewall policies applied at the organization or folder level that are inherited by all child resources and evaluated before VPC-level firewall rules.

**Host Project** — In a Shared VPC configuration, the GCP project that owns the shared VPC network and subnets.

**HTTPS (Hypertext Transfer Protocol Secure)** — The encrypted version of HTTP using TLS/SSL; Cloud Armor attaches to external HTTPS load balancers.

**IAM (Identity and Access Management)** — GCP's system for controlling who (identity) has what access (roles/permissions) to which GCP resources.

**Implied Rule** — A non-deletable firewall rule built into every VPC: implied allow-egress to all destinations and implied deny-ingress from all sources, both at priority 65535.

**Instance Group** — A collection of Compute Engine VMs managed as a single unit. **Managed instance groups (MIGs)** use an instance template for autoscaling, autohealing, and rolling updates; **unmanaged instance groups** are heterogeneous collections of existing VMs.

**IAP (Identity-Aware Proxy)** — A GCP service that provides context-aware access control for applications and VMs; allows SSH/RDP tunneling to VMs without external IPs.

**ICMP (Internet Control Message Protocol)** — A network protocol used for diagnostic and error-reporting purposes (e.g., ping); referenced in firewall rule configurations.

**IKE (Internet Key Exchange)** — A protocol used to set up security associations in IPsec; Cloud VPN tunnels support IKE version 2.

**Ingress** — Network traffic flowing inbound to a resource (e.g., from the internet to a VM).

**Internal IP** — A private IP address used within a VPC network that is not directly reachable from the public internet.

**IPsec (Internet Protocol Security)** — A suite of protocols for authenticating and encrypting IP traffic; used by Cloud VPN to secure tunnel traffic.

**IPv4** — The fourth version of the Internet Protocol, using 32-bit addresses; GCP VPC subnets use IPv4 CIDR ranges.

**Layer 4 (L4)** — The transport layer of the OSI model (TCP/UDP). GCP Network Load Balancers operate at L4 and forward packets without terminating connections (passthrough) or as TCP/SSL proxies.

**Layer 7 (L7)** — The application layer of the OSI model (HTTP/HTTPS). GCP Application Load Balancers terminate HTTP(S) connections and route based on host, path, and headers.

**Load Balancer** — A GCP resource that distributes incoming requests across multiple backends for scalability, high availability, and global/regional reach.

**Longest Prefix Match** — A routing algorithm principle where the most specific IP prefix (longest subnet mask) in the routing table is selected when multiple routes match a destination.

**MIG (Managed Instance Group)** — A Compute Engine resource that maintains a set of identical VMs from an instance template, providing autoscaling, autohealing, rolling updates, and multi-zone redundancy.

**NAT (Network Address Translation)** — The process of remapping source IP addresses, allowing VMs with private IPs to communicate with external networks through a shared public IP.

**NEG (Network Endpoint Group)** — A GCP load balancer backend type that references endpoints other than VM instance groups. Variants include zonal NEG (IP:port pairs), serverless NEG (Cloud Run/Functions/App Engine), and internet NEG (external backends).

**Network Load Balancer** — A GCP Layer 4 load balancer that operates on TCP/UDP traffic. Available as passthrough (preserves client IP) or proxy variants.

**Network Tag** — A label applied to a Compute Engine VM instance used to apply firewall rules or routes selectively to tagged instances.

**Next-Hop** — In routing, the next device or gateway that a packet is forwarded to on its way to the destination; options include instances, IP addresses, VPN tunnels, and internet gateways.

**OWASP Top 10** — A standard list of the ten most critical web application security risks; Cloud Armor provides preconfigured WAF rules to defend against these.

**Partner Interconnect** — A GCP service that provides connectivity between an on-premises network and GCP through a supported third-party service provider, offering 50 Mbps to 50 Gbps bandwidth.

**Peering Connection** — A network link between two VPC networks that enables resources in each VPC to communicate using internal IP addresses.

**Port** — A 16-bit identifier for an endpoint on a host, used by TCP and UDP to distinguish multiple services. Firewall rules and forwarding rules specify ports (e.g., 80 for HTTP, 443 for HTTPS).

**Priority (Firewall/Route)** — A 0–65535 integer determining the evaluation order of firewall rules or routes. Lower numbers are evaluated first; the first matching rule wins.

**Private Google Access** — A subnet-level feature that allows VM instances with only internal IP addresses to reach Google APIs and services (e.g., Cloud Storage, BigQuery).

**Protocol** — In networking, a set of rules governing data exchange; firewall rules specify protocols such as TCP, UDP, and ICMP.

**RDP (Remote Desktop Protocol)** — A Microsoft protocol for remote graphical access to Windows systems; used to connect to Windows VMs in GCP.

**RFC 1918** — An Internet standard defining private IP address ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) used for internal networking.

**Role (IAM)** — A named collection of permissions granted to a principal on a resource. For Shared VPC, `roles/compute.networkUser` allows using subnets from a host project.

**Route** — A rule that specifies how to forward packets destined for a particular IP range to a specific next-hop.

**Service Account** — A GCP identity associated with a VM or application (rather than a human user), used to authenticate API calls and control resource access; can also be used to target firewall rules.

**Service Project** — In a Shared VPC configuration, a GCP project that uses subnets from the host project's shared VPC network to create its resources.

**Shared VPC** — A GCP networking feature that allows a single VPC network (owned by a host project) to be shared across multiple service projects within an organization.

**SLA (Service Level Agreement)** — A contractual commitment from a cloud provider guaranteeing a minimum level of service availability; HA VPN offers 99.99% SLA.

**Source Range** — In firewall rules, the IP address range(s) from which ingress traffic originates.

**SSH (Secure Shell)** — A cryptographic network protocol for secure remote login and command execution on Linux/Unix systems.

**Static IP** — A reserved external IP address in GCP that persists independently of any VM. Available as regional (for VMs and regional LBs) or global (for global LBs).

**Stateful Firewall** — A firewall that tracks the state of network connections; GCP VPC firewall rules are stateful, so allowed ingress automatically permits the corresponding egress reply traffic.

**Subnet (Subnetwork)** — A regional IP address range within a VPC network; VMs and other resources are assigned IP addresses from the subnet's CIDR range.

**Target Proxy** — A GCP load balancer component (HTTP, HTTPS, SSL, or TCP) that terminates client connections and forwards requests to a URL map (for L7) or backend service (for L4 proxy).

**TCP (Transmission Control Protocol)** — A connection-oriented transport layer protocol; firewall rules commonly specify TCP ports such as 80 (HTTP) and 443 (HTTPS).

**UDP (User Datagram Protocol)** — A connectionless transport layer protocol; used in firewall rules for protocols like DNS (port 53).

**URL Map** — A GCP load balancer component used by Application Load Balancers to route HTTP(S) requests to different backend services based on host and path rules.

**VPC (Virtual Private Cloud)** — A logically isolated, global network within GCP that provides networking for Compute Engine VMs, GKE clusters, and other resources.

**VPC Flow Logs** — A VPC feature that captures metadata about network flows (source/destination IP, port, protocol, bytes, packets) for monitoring, forensics, and security analysis.

**VPC Network Peering** — A GCP feature that connects two VPC networks so their resources can communicate using internal IP addresses; peering is non-transitive and requires configuration on both sides.

**VPN (Virtual Private Network)** — An encrypted network connection over a public network; Cloud VPN creates IPsec-encrypted tunnels between GCP VPCs and external networks.

**VPN Gateway** — The GCP resource that serves as the endpoint for a Cloud VPN tunnel, either HA VPN or Classic VPN.

**VPN Tunnel** — An encrypted logical connection between two VPN gateways over the public internet, used to securely connect networks.

**WAF (Web Application Firewall)** — A security layer that filters and monitors HTTP traffic to protect web applications from attacks; Cloud Armor provides WAF capabilities.
