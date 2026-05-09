# GCP vs AWS — Mapare Completa a Serviciilor

> Toate serviciile GCP mentionate in materialele de studiu ACE, cu echivalentul lor din AWS.

---

## Compute

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Compute Engine** | EC2 | Masini virtuale (IaaS) |
| **Spot VMs** | Spot Instances | VM-uri cu discount mare, pot fi revendicate oricand |
| **Preemptible VMs** | Spot Instances (legacy) | Predecesorul Spot VMs, max 24h |
| **Sole-Tenant Nodes** | Dedicated Hosts | Servere fizice dedicate |
| **Custom Machine Types** | — (nu exista direct) | CPU/RAM configurabil exact; AWS are doar tipuri predefinite |
| **Instance Templates** | Launch Templates | Blueprint imutabil pentru creare VM-uri |
| **Managed Instance Groups (MIG)** | Auto Scaling Groups (ASG) | Grup de VM-uri identice cu autoscaling si autohealing |
| **OS Login** | EC2 Instance Connect / SSM | Acces SSH bazat pe identitate (IAM) |
| **VM Manager** | AWS Systems Manager (SSM) | Patch management, inventar OS, configurare |
| **Google Kubernetes Engine (GKE)** | Elastic Kubernetes Service (EKS) | Kubernetes gestionat |
| **GKE Autopilot** | EKS with Fargate | Kubernetes serverless (platesti per pod) |
| **GKE Enterprise (Anthos)** | EKS Anywhere | Kubernetes hybrid/multi-cloud |
| **Cloud Run** | AWS App Runner / Fargate | Containere serverless (scale to zero) |
| **Cloud Functions** | AWS Lambda | Functions as a Service (FaaS) |
| **Cloud Build** | AWS CodeBuild | Build si CI/CD pentru containere |

---

## Storage

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Cloud Storage** | S3 | Object storage |
| **Cloud Storage — Standard** | S3 Standard | Date accesate frecvent |
| **Cloud Storage — Nearline** | S3 Standard-IA | Date accesate < 1x/luna |
| **Cloud Storage — Coldline** | S3 Glacier Instant Retrieval | Date accesate < 1x/trimestru |
| **Cloud Storage — Archive** | S3 Glacier Deep Archive | Date accesate < 1x/an |
| **Persistent Disk (pd-standard)** | EBS gp2/gp3 (HDD: st1/sc1) | Block storage — HDD |
| **Persistent Disk (pd-balanced)** | EBS gp3 | Block storage — SSD echilibrat |
| **Persistent Disk (pd-ssd)** | EBS io1/io2 | Block storage — SSD performant |
| **Persistent Disk (pd-extreme)** | EBS io2 Block Express | Block storage — IOPS maxim |
| **Regional Persistent Disk** | EBS Multi-Attach (limitat) | Block storage replicat in 2 zone |
| **Hyperdisk Balanced** | EBS gp3 (provisioned IOPS) | Block storage — IOPS/throughput independent de capacitate |
| **Hyperdisk Extreme** | EBS io2 Block Express | Block storage — IOPS maxim, mission-critical |
| **Hyperdisk Throughput** | EBS st1 (throughput) | Block storage — Kafka, Hadoop (sequential) |
| **Hyperdisk ML** | — (nu exista echivalent) | Block storage multi-reader pentru ML model serving |
| **Local SSD** | Instance Store (NVMe) | SSD efemer, atasat fizic |
| **Filestore** | EFS (Elastic File System) | NFS managed file storage |
| **Google Cloud NetApp Volumes** | Amazon FSx for NetApp ONTAP | Enterprise NFS/SMB pe NetApp ONTAP |
| **Storage Transfer Service** | AWS DataSync | Transfer date la scara mare |
| **Transfer Appliance** | AWS Snowball / Snowball Edge | Dispozitiv fizic pentru transfer offline |

---

## Databases

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Cloud SQL (MySQL)** | RDS MySQL / Aurora MySQL | MySQL gestionat |
| **Cloud SQL (PostgreSQL)** | RDS PostgreSQL / Aurora PostgreSQL | PostgreSQL gestionat |
| **Cloud SQL (SQL Server)** | RDS SQL Server | SQL Server gestionat |
| **AlloyDB** | Aurora PostgreSQL | PostgreSQL de inalta performanta (HTAP) |
| **Cloud Spanner** | Aurora Global Database (partial) / DynamoDB Global Tables | Baza de date relationala, globala, strong consistency |
| **BigQuery** | Amazon Redshift / Athena | Data warehouse serverless (OLAP) |
| **BigQuery ML** | Amazon SageMaker (partial) | ML direct in SQL |
| **Firestore (Native mode)** | DynamoDB + AppSync | Document DB cu real-time sync |
| **Firestore (Datastore mode)** | DynamoDB | Document DB pentru server-side |
| **Cloud Bigtable** | DynamoDB / Amazon Timestream | Wide-column NoSQL, time-series |
| **Memorystore (Redis)** | ElastiCache (Redis) | Redis gestionat in-memory |
| **Memorystore (Memcached)** | ElastiCache (Memcached) | Memcached gestionat |
| **Managed Service for Apache Kafka** | Amazon MSK (Managed Streaming for Kafka) | Kafka gestionat, API nativ compatibil |
| **Database Center** | — (partial: AWS Database Migration Hub) | Panou de control unificat pentru toate DB-urile GCP |

---

## Networking

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **VPC** | VPC | Retea virtuala privata |
| **Subnets** | Subnets | Segmente IP regionale |
| **Firewall Rules (Classic)** | Security Groups + NACLs | Reguli de trafic (GCP: la nivel de VPC, stateful) |
| **Cloud NGFW (Hierarchical Policies)** | AWS Network Firewall + SCP | Politici de firewall la nivel org/folder, securitate avansata |
| **Secure Tags** | — (partial: Resource Groups + SCP) | Tag-uri IAM-governed pentru targetting in firewall rules |
| **Routes** | Route Tables | Reguli de rutare trafic |
| **Cloud NAT** | NAT Gateway | Acces internet outbound pentru VM-uri fara IP extern |
| **Cloud DNS** | Route 53 | DNS gestionat (public si privat) |
| **Cloud CDN** | CloudFront | Content Delivery Network |
| **Cloud Armor** | AWS WAF + Shield | Protectie DDoS si Web Application Firewall |
| **External Application LB (HTTP/S)** | Application Load Balancer (ALB) | Load balancer L7, global |
| **Internal Application LB** | Internal ALB | Load balancer L7, intern |
| **External Passthrough Network LB** | Network Load Balancer (NLB) | Load balancer L4, TCP/UDP |
| **Internal Passthrough Network LB** | Internal NLB | Load balancer L4, intern |
| **External Proxy Network LB** | NLB with TLS termination | Load balancer L4, TCP/SSL proxy |
| **Cloud VPN (HA VPN)** | AWS Site-to-Site VPN | VPN IPsec catre on-premises |
| **Cloud Interconnect (Dedicated)** | AWS Direct Connect | Conexiune fizica dedicata |
| **Cloud Interconnect (Partner)** | AWS Direct Connect via partner | Conexiune prin provider |
| **VPC Network Peering** | VPC Peering | Conectare VPC-uri prin IP intern |
| **Shared VPC** | AWS RAM (Resource Access Manager) | VPC partajat intre proiecte/conturi |
| **Private Google Access** | VPC Endpoints (Gateway type) | Acces la API-uri Google fara IP extern |
| **Serverless VPC Access Connector** | — (Fargate uses VPC natively) | Conecteaza Cloud Run/Functions la VPC |
| **Cloud Router** | — (BGP inclus in VPN/DX) | BGP router pentru rutare dinamica |
| **Network Service Tiers (Premium)** | — (default AWS routing) | Trafic prin backbone Google |
| **Network Service Tiers (Standard)** | — (similar cu default AWS) | Trafic prin internet public |
| **Identity-Aware Proxy (IAP)** | AWS SSM Session Manager | Acces securizat la VM-uri fara IP extern |

---

## Security si Identity

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Cloud IAM** | AWS IAM | Identity and Access Management |
| **IAM Roles (Basic)** | AWS Managed Policies (broad) | Roluri largi: Viewer, Editor, Owner |
| **IAM Roles (Predefined)** | AWS Managed Policies | Roluri specifice per serviciu |
| **IAM Roles (Custom)** | IAM Custom Policies | Roluri definite de utilizator |
| **IAM Conditions** | IAM Condition Keys | Acces conditionat (timp, resursa, etc.) |
| **IAM Deny Policies** | SCPs (Service Control Policies) | Blocare explicita de permisiuni |
| **Service Accounts** | IAM Roles for Services / Instance Profiles | Identitate pentru aplicatii/VM-uri |
| **Workload Identity (GKE)** | IRSA (IAM Roles for Service Accounts) | Pod-uri K8s → identitate cloud |
| **Workload Identity Federation** | IAM OIDC Identity Providers | Identitati externe (GitHub, AWS) → GCP |
| **Service Account Keys** | Access Keys / Secret Keys | Credentiale long-lived (de evitat!) |
| **Service Account Impersonation** | AssumeRole (STS) | Impersonare temporara |
| **Cloud Identity** | AWS IAM Identity Center (SSO) | Managementul utilizatorilor si grupurilor |
| **Organization** | AWS Organizations | Nivel ierarhic de top |
| **Folders** | Organizational Units (OUs) | Grupare logica sub organizatie |
| **Organization Policies** | Service Control Policies (SCPs) | Constrangeri top-down pe resurse |
| **Secret Manager** | AWS Secrets Manager | Stocare secrete (parole, chei API) |
| **Cloud KMS** | AWS KMS | Managementul cheilor de criptare |
| **OS Login** | EC2 Instance Connect | SSH bazat pe identitate IAM |

---

## Monitoring, Logging si Operations

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Cloud Monitoring** | CloudWatch Metrics | Metrici, dashboarduri, alerte |
| **Cloud Logging** | CloudWatch Logs | Centralizare si analiza loguri |
| **Cloud Trace** | AWS X-Ray | Distributed tracing |
| **Cloud Profiler** | CodeGuru Profiler | Profilare CPU/memorie |
| **Error Reporting** | — (CloudWatch + X-Ray) | Agregare si afisare erori |
| **Ops Agent** | CloudWatch Agent | Agent unificat pentru metrici si loguri pe VM-uri |
| **Managed Service for Prometheus** | Amazon Managed Prometheus (AMP) | Prometheus gestionat |
| **Log Router / Log Sinks** | CloudWatch Subscription Filters | Rutare loguri catre destinatii |
| **Log-Based Metrics** | CloudWatch Metric Filters | Metrici derivate din loguri |
| **Uptime Checks** | Route 53 Health Checks / CloudWatch Synthetics | Verificare disponibilitate serviciu |
| **Audit Logs (Admin Activity)** | CloudTrail (Management Events) | Loguri API-uri de modificare — mereu activ, gratuit |
| **Audit Logs (Data Access)** | CloudTrail (Data Events) | Loguri API-uri de citire — optional, cu cost |
| **Cloud Scheduler** | EventBridge Scheduler | Cron jobs gestionat |
| **Google Cloud Status Dashboard** | AWS Service Health Dashboard | Starea serviciilor cloud (publica, pentru toti) |
| **Personalized Service Health** | AWS Personal Health Dashboard | Incidente care afecteaza PROIECTELE TALE |
| **Active Assist** | AWS Trusted Advisor | Recomandari AI pentru cost, securitate, performanta |
| **Gemini Cloud Assist** | Amazon Q Developer (partial) | Asistent AI in consola pentru log-uri, alerte, troubleshooting |
| **Query Insights** | Amazon DevOps Guru for RDS | Diagnosticare interogari lente in Cloud SQL/AlloyDB/Spanner |
| **Cloud Asset Inventory** | AWS Config | Inventar resurse si politici IAM la nivel de organizatie |

---

## Data & Analytics

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Pub/Sub** | Amazon SNS + SQS / Amazon Kinesis | Messaging async (topic/subscription) |
| **Dataflow** | Amazon Kinesis Data Analytics / EMR | Stream si batch processing (Apache Beam) |
| **Dataproc** | Amazon EMR | Hadoop/Spark gestionat |
| **Eventarc** | Amazon EventBridge | Rutare evenimente (100+ surse) |

---

## Infrastructure as Code si DevOps

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Terraform** | Terraform (identic) | IaC multi-cloud — HashiCorp |
| **Cloud Foundation Toolkit (CFT)** | AWS Landing Zone Accelerator | Module Terraform cu best practices GCP |
| **Fabric FAST** | AWS Control Tower | Framework opinionated pentru landing zone complet |
| **Config Connector** | AWS Controllers for K8s (ACK) | Resurse cloud ca obiecte K8s |
| **Helm** | Helm (identic) | Package manager pentru Kubernetes |
| **Artifact Registry** | Amazon ECR (Elastic Container Registry) | Registry pentru containere si pachete |
| **Cloud Source Repositories** | AWS CodeCommit | Git repository gestionat |

---

## Billing si Cost Management

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Billing Account** | AWS Account (Payer Account) | Contul care plateste |
| **Billing Budgets & Alerts** | AWS Budgets | Bugete cu notificari |
| **Billing Export (BigQuery)** | Cost and Usage Reports (CUR) → Athena | Export detaliat costuri |
| **Committed Use Discounts (CUDs)** | Reserved Instances / Savings Plans | Discounturi pe 1-3 ani |
| **Sustained Use Discounts (SUDs)** | — (nu exista echivalent exact) | Discount automat > 25% din luna |
| **Pricing Calculator** | AWS Pricing Calculator | Estimare costuri |
| **Labels** | Tags | Perechi cheie-valoare pentru organizare si cost allocation |
| **Quotas** | Service Quotas (Limits) | Limite pe resurse per proiect/cont |

---

## Proiecte si Organizare

| GCP | AWS | Descriere |
|-----|-----|-----------|
| **Organization** | AWS Organization | Nod radacina |
| **Folder** | Organizational Unit (OU) | Grupare ierarhica |
| **Project** | AWS Account | Unitatea de baza (resurse, billing, IAM) |
| **Resource Hierarchy** | AWS Organizations Hierarchy | Org → OU → Account |

---

## Diferente Cheie de Retinut

| Aspect | GCP | AWS |
|--------|-----|-----|
| **VPC** | Global (acoperă toate regiunile) | Regional (per regiune) |
| **Subnets** | Regionale (acoperă toate zonele din regiune) | Per zona de disponibilitate |
| **Firewall** | La nivel de VPC, doar allow/deny | Security Groups (allow only) + NACLs (allow/deny) |
| **Load Balancing** | Global nativ (L7) | Regional by default (Global cu CloudFront) |
| **IAM** | Roluri legate de resurse (binding) | Politici atasate utilizatorilor/grupurilor |
| **Projects vs Accounts** | Un proiect = izolare; mai multe proiecte sub un billing account | Un cont = izolare; Organizations grupeaza conturi |
| **Spot VMs** | Pot fi revendicate oricand, fara limita de timp | Spot Instances — similar, cu 2 min avertizare |
| **Sustained Use Discounts** | Automate, fara commitment | Nu exista — trebuie Reserved Instances explicit |
| **Storage classes** | Toate au aceeasi latenta (ms) | Glacier are latenta de ore |
| **Serverless containers** | Cloud Run (scale to zero, pay per request) | App Runner (similar) / Fargate (fara scale to zero) |
| **K8s managed** | GKE (cel mai matur K8s managed) | EKS (necesita mai multa configurare) |
| **Database globala** | Spanner (SQL + global strong consistency) | Nu exista echivalent direct |
