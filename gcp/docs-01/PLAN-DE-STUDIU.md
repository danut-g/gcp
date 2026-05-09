# Plan de Studiu — GCP Associate Cloud Engineer

> Pentru cineva cu experienta AWS/GCP de baza.
> Durata recomandata: **4-6 saptamani** (1-2 ore/zi)

---

## Principii de Invatare

### Cum sa citesti materialele

1. **Nu citi pasiv** — Dupa fiecare sectiune, inchide fisierul si incearca sa raspunzi: "Ce am invatat? Ce serviciu as folosi pentru X?"
2. **Concentreaza-te pe DIFERENTE** — Ai experienta AWS. Creierul tau va retine cel mai bine lucrurile care sunt *diferite* fata de ce stii. Foloseste `gcp-vs-aws.md` ca ancora.
3. **Regula 3-2-1** — Dupa fiecare sesiune de studiu:
   - **3** lucruri noi invatate
   - **2** lucruri pe care vrei sa le aprofundezi
   - **1** intrebare la care nu stii raspunsul
4. **Invata in spirala, nu liniar** — Treci prin tot materialul o data (surface pass), apoi revino si aprofundeaza (deep pass), apoi testeaza-te (test pass).

### Cum sa intelegi (nu sa memorezi)

- **Intreaba-te "De ce?"** — Nu memora "Cloud Run scaleaza la zero". Intelege DE CE: e serverless, platesti per request, deci daca nu sunt requesturi, nu ai nevoie de instante.
- **Compara mereu** — "De ce Spanner si nu Cloud SQL?" → Spanner e global + horizontal scaling. Cloud SQL e regional + vertical scaling. Pretul reflecta asta.
- **Gandeste in scenarii** — "Sunt CTO la un startup. Am o aplicatie web cu trafic imprevizibil. Ce aleg?" → Cloud Run (scale to zero, cost minim, no ops).
- **Deseneaza** — Deschide un caiet si deseneaza arhitecturi. VPC-uri cu subnete, LB-uri cu backend-uri, IAM inheritance. Desenul forteaza intelegerea.

---

## Saptamana 0 — Pregatire (Ziua 1)

**Obiectiv:** Intelege structura examenului si fa un self-assessment.

| Pas | Ce faci                          | Fisier                  | Timp   |
| --- | -------------------------------- | ----------------------- | ------ |
| 1   | Citeste overview-ul examenului   | `00-exam-overview.md` | 15 min |
| 2   | Parcurge rapid GCP vs AWS        | `gcp-vs-aws.md`       | 20 min |
| 3   | Noteaza ce stii deja si ce e nou | Caiet/notite proprii    | 10 min |

Dupa acest pas ar trebui sa stii: cele **4 sectiuni** (structura noua 2024/2025), procentajele, si unde ai gaps fata de ce stii din AWS.

---

## Saptamana 1 — Fundamente (Sectiunea 1: ~23%)

> **Sectiunea 1 noua (2024/2025):** Setting Up a Cloud Solution Environment (~23%) — include project setup, IAM, billing, Cloud Identity, **Cloud Asset Inventory** si **Gemini Cloud Assist**.

**De ce IAM si security in saptamana 1?** Sunt fundament pentru tot restul. Daca nu intelegi IAM si hierarchia, nimic altceva nu are sens.

### Zilele 1-2: Resource Hierarchy + IAM

| Pas | Ce faci                   | Fisier                                           | Timp   |
| --- | ------------------------- | ------------------------------------------------ | ------ |
| 1   | Citeste in profunzime     | `01-setting-up-cloud-projects-and-accounts.md` | 40 min |
| 2   | Vizualizeaza relatiile    | `visual-maps/section1-setup.md`                | 20 min |
| 3   | Testeaza-te cu flashcards | `flashcards/section1-setup.md` (Q1-Q20)        | 15 min |

**Focus:** Org → Folders → Projects → Resources. Policy inheritance (additive!). Diferenta fata de AWS: VPC e global, Project ≈ AWS Account.

### Zilele 3-4: IAM + Service Accounts

| Pas | Ce faci                   | Fisier                               | Timp   |
| --- | ------------------------- | ------------------------------------ | ------ |
| 1   | Citeste IAM in profunzime | `18-managing-iam.md`               | 40 min |
| 2   | Citeste Service Accounts  | `19-managing-service-accounts.md`  | 40 min |
| 3   | Vizualizeaza              | `visual-maps/section5-security.md` | 20 min |
| 4   | Flashcards                | `flashcards/section5-security.md`  | 20 min |

**Focus:** Basic vs Predefined vs Custom roles. Service Account = si principal SI resource. Impersonation > Keys. Workload Identity = IRSA din AWS.

### Zilele 5-6: Billing + Cheat Sheet Review

| Pas | Ce faci                            | Fisier                                                                   | Timp   |
| --- | ---------------------------------- | ------------------------------------------------------------------------ | ------ |
| 1   | Citeste billing                    | `02-managing-billing-configuration.md`                                 | 30 min |
| 2   | Review rapid cheat sheets          | `cheatsheets/section1-setup.md` + `cheatsheets/section5-security.md` | 20 min |
| 3   | Flashcards complete sectiunile 1+5 | `flashcards/section1-setup.md` + `flashcards/section5-security.md`   | 20 min |

**Focus:** Budgets NU opresc cheltuielile (trebuie Pub/Sub + Cloud Function). Billing export → BigQuery. SUDs sunt automate (nu exista in AWS!).

### Ziua 7: Recapitulare Saptamana 1

- Parcurge cheat sheets section 1 + 5 fara sa te uiti la raspunsuri
- Fa scenariile relevante din `scenarios/practice-scenarios.md` (16-19, 24, 28)
- Noteaza ce inca nu intelegi → revii saptamana urmatoare

---

## Saptamana 2 — Planning (Sectiunea 2: ~30%)

> **Sectiunea 2 noua (2024/2025):** Planning and Configuring a Cloud Solution (~30% — cea mai mare!) — include compute, storage, networking, plus **Hyperdisk**, **Knative**, **NetApp Volumes**, **Managed Kafka**, **multi-region redundancy**.

**Aceasta e saptamana deciziilor: "Ce serviciu folosesc pentru X?"**

### Zilele 1-2: Compute

| Pas | Ce faci                      | Fisier                                         | Timp   |
| --- | ---------------------------- | ---------------------------------------------- | ------ |
| 1   | Citeste compute planning     | `03-planning-compute-resources.md`           | 45 min |
| 2   | Studiaza decision flowcharts | `visual-maps/section2-planning.md` (compute) | 20 min |
| 3   | Flashcards compute           | `flashcards/section2-planning.md` (Q1-Q15)   | 15 min |

**Focus:** Flowchart-ul de decizie: CE vs GKE vs Cloud Run vs Functions. Cand Spot VMs. Custom machine types. GKE Standard vs Autopilot.

### Zilele 3-4: Databases + Storage

| Pas | Ce faci                         | Fisier                                           | Timp   |
| --- | ------------------------------- | ------------------------------------------------ | ------ |
| 1   | Citeste data storage            | `04-planning-data-storage-options.md`          | 45 min |
| 2   | Studiaza database decision tree | `visual-maps/section2-planning.md` (databases) | 20 min |
| 3   | Flashcards databases            | `flashcards/section2-planning.md` (Q16-Q30)    | 15 min |

**Focus:** OLTP vs OLAP. Cloud SQL (< 64TB, regional) vs Spanner (global, unlimited). Firestore modes (Native vs Datastore — NU se poate schimba!). Storage classes — TOATE au ms latency (spre deosebire de Glacier!).

### Zilele 5-6: Networking + Review

| Pas | Ce faci               | Fisier                                            | Timp   |
| --- | --------------------- | ------------------------------------------------- | ------ |
| 1   | Citeste networking    | `05-planning-network-resources.md`              | 35 min |
| 2   | Studiaza LB flowchart | `visual-maps/section2-planning.md` (networking) | 15 min |
| 3   | Cheat sheet complet   | `cheatsheets/section2-planning.md`              | 15 min |
| 4   | Flashcards restul     | `flashcards/section2-planning.md` (Q31-Q40)     | 10 min |

**Focus:** 6 tipuri de load balancer (memorizeaza flowchart-ul!). Premium vs Standard tier. VPC e GLOBAL in GCP (mare diferenta fata de AWS!).

### Ziua 7: Recapitulare Saptamana 2

- Scenarii din `scenarios/practice-scenarios.md` (1-12)
- Exercitiu: deseneaza pe hartie flowchart-ul de decizie compute + database din memorie

---

## Saptamana 3 — Deploying (Sectiunea 3: ~27%)

> **Sectiunea 3 noua (2024/2025):** Deploying and Implementing a Cloud Solution (~27%) — include Compute Engine, GKE, Cloud Run, Functions, Data, Networking, IaC, plus **Cloud NGFW cu Secure Tags**, **Fabric FAST**, **Workload Identity** (step-by-step), **IaC versioning/state**.

**Cea mai tehnica sectiune — trebuie sa stii COMENZI.**

### Zilele 1-2: Compute Engine + MIGs

| Pas | Ce faci               | Fisier                                          | Timp   |
| --- | --------------------- | ----------------------------------------------- | ------ |
| 1   | Citeste deploying CE  | `06-deploying-compute-engine.md`              | 45 min |
| 2   | Vizualizeaza MIG flow | `visual-maps/section3-deploying.md` (VM flow) | 15 min |
| 3   | Flashcards            | `flashcards/section3-deploying.md` (Q1-Q15)   | 15 min |

**Focus:** `gcloud compute instances create` cu parametri cheie. Instance templates sunt IMUTABILE. MIG + autoscaling + autohealing. OS Login > SSH keys. IAP tunnel pentru VM fara IP extern.

### Zilele 3-4: GKE + Cloud Run/Functions

| Pas | Ce faci                     | Fisier                                                  | Timp   |
| --- | --------------------------- | ------------------------------------------------------- | ------ |
| 1   | Citeste GKE deploying       | `07-deploying-gke-resources.md`                       | 40 min |
| 2   | Citeste Cloud Run/Functions | `08-deploying-cloud-run-and-functions.md`             | 35 min |
| 3   | Vizualizeaza                | `visual-maps/section3-deploying.md` (GKE + Cloud Run) | 15 min |
| 4   | Flashcards                  | `flashcards/section3-deploying.md` (Q16-Q30)          | 15 min |

**Focus:** `gcloud container clusters get-credentials` (conecteaza kubectl). Private clusters = nodes fara IP extern. Cloud Run revisions + traffic splitting. Cloud Functions Gen 2 > Gen 1. Eventarc.

### Zilele 5-6: Data + Networking + IaC

| Pas | Ce faci                | Fisier                                         | Timp   |
| --- | ---------------------- | ---------------------------------------------- | ------ |
| 1   | Citeste data solutions | `09-deploying-data-solutions.md`             | 30 min |
| 2   | Citeste networking     | `10-deploying-networking-resources.md`       | 30 min |
| 3   | Citeste IaC            | `11-infrastructure-as-code.md`               | 20 min |
| 4   | Flashcards             | `flashcards/section3-deploying.md` (Q31-Q50) | 15 min |

**Focus:** VPC peering e NON-TRANZITIV. Firewall rules: priority (0=max), tags vs service accounts. Terraform: init → plan → apply. State in GCS, nu local.

### Ziua 7: Recapitulare Saptamana 3

- Cheat sheet complet: `cheatsheets/section3-deploying.md`
- Scenarii: `scenarios/practice-scenarios.md` (13-15, 25-27)

---

## Saptamana 4 — Operations (Sectiunea 4: ~20%)

> **Sectiunea 4 noua (2024/2025):** Ensuring Successful Operation (~20%) — include managing compute/GKE/Cloud Run/storage/networking/monitoring, plus **Database Center**, **AlloyDB/Spanner/Bigtable backup**, **Query Insights**, **Personalized Service Health**, **Gemini for Monitoring**, **Active Assist**, **GKE Autopilot Pod resources**.

### Zilele 1-2: Compute + GKE Management

| Pas | Ce faci              | Fisier                                                          | Timp   |
| --- | -------------------- | --------------------------------------------------------------- | ------ |
| 1   | Citeste managing CE  | `12-managing-compute-engine.md`                               | 35 min |
| 2   | Citeste managing GKE | `13-managing-gke-resources.md`                                | 40 min |
| 3   | Vizualizeaza         | `visual-maps/section4-operations.md` (snapshots, autoscaling) | 15 min |
| 4   | Flashcards           | `flashcards/section4-operations.md` (Q1-Q20)                  | 15 min |

**Focus:** Snapshots = incremental, backup. Images = golden image, templates. HPA (pods) vs VPA (resources) vs Cluster Autoscaler (nodes). StatefulSets pentru databases.

### Zilele 3-4: Cloud Run + Storage + Networking

| Pas | Ce faci                     | Fisier                                          | Timp   |
| --- | --------------------------- | ----------------------------------------------- | ------ |
| 1   | Citeste managing Cloud Run  | `14-managing-cloud-run.md`                    | 25 min |
| 2   | Citeste managing storage    | `15-managing-storage-and-databases.md`        | 35 min |
| 3   | Citeste managing networking | `16-managing-networking-resources.md`         | 25 min |
| 4   | Flashcards                  | `flashcards/section4-operations.md` (Q21-Q40) | 15 min |

**Focus:** Cloud Run: `--no-traffic` + tags pentru canary. Lifecycle policies pentru cost optimization. Subnet expansion = one-way (nu poti micsora). Cloud NAT = outbound only.

### Zilele 5-6: Monitoring + Logging

| Pas | Ce faci                    | Fisier                                           | Timp   |
| --- | -------------------------- | ------------------------------------------------ | ------ |
| 1   | Citeste monitoring/logging | `17-monitoring-and-logging.md`                 | 45 min |
| 2   | Vizualizeaza log routing   | `visual-maps/section4-operations.md` (logging) | 15 min |
| 3   | Flashcards                 | `flashcards/section4-operations.md` (Q41-Q50)  | 10 min |
| 4   | Cheat sheet                | `cheatsheets/section4-operations.md`           | 15 min |

**Focus:** Admin Activity logs = mereu active, gratuite. Data Access = optional, cu cost. Log sinks + writer identity (trebuie permisiuni pe destinatie!). Ops Agent = inlocuieste agentii vechi. `_Required` bucket = 400 zile, nu se poate modifica.

### Ziua 7: Recapitulare Saptamana 4

- Scenarii: `scenarios/practice-scenarios.md` (20-22)
- Parcurge toate cheat sheets (1-5) rapid

---

## Saptamana 5 — Consolidare + Testare

### Zilele 1-2: Full Review

| Pas | Ce faci                                 | Fisier                     | Timp   |
| --- | --------------------------------------- | -------------------------- | ------ |
| 1   | Parcurge TOATE cheat sheets             | `cheatsheets/section1-5` | 30 min |
| 2   | Parcurge TOATE visual maps              | `visual-maps/section1-5` | 45 min |
| 3   | Noteaza conceptele care inca te incurca | Caiet propriu              | 15 min |

### Zilele 3-4: Flashcards Marathon

| Pas | Ce faci                                  | Fisier                            | Timp      |
| --- | ---------------------------------------- | --------------------------------- | --------- |
| 1   | Toate flashcards, sectiunile 1-5         | `flashcards/*` (~250 intrebari) | 75 min/zi |
| 2   | Marcheaza intrebarile gresite            | Caiet propriu                     | —        |
| 3   | Reciteste materialul doar pentru greseli | Fisierele relevante               | 30 min    |

### Zilele 5-6: Scenarii Complete

| Pas | Ce faci                                           | Fisier                              | Timp      |
| --- | ------------------------------------------------- | ----------------------------------- | --------- |
| 1   | Fa TOATE 28 scenariile                            | `scenarios/practice-scenarios.md` | 60 min/zi |
| 2   | Pentru fiecare raspuns gresit, citeste explicatia | —                                  | —        |
| 3   | Revizuieste sectiunile slabe                      | Fisierele relevante                 | 30 min    |

### Ziua 7: Practice Exam

- Fa examenul gratuit de practica de la Google: [cloud.google.com/certification/practice-exam/cloud-engineer](https://cloud.google.com/certification/practice-exam/cloud-engineer)
- Noteaza scorul si sectiunile slabe

---

## Saptamana 6 — Ultimele Zile Inainte de Examen

### Zilele 1-3: Focus pe Puncte Slabe

- Reciteste DOAR sectiunile unde ai gresit la practice exam
- Repeta flashcards-urile doar pentru acele sectiuni

### Zilele 4-5: Speed Review

| Pas | Ce faci                                | Timp   |
| --- | -------------------------------------- | ------ |
| 1   | Toate cheat sheets (5 fisiere)         | 25 min |
| 2   | `gcp-vs-aws.md` — diferentele cheie | 10 min |
| 3   | Scenariile gresite anterior            | 20 min |

### Ziua 6: Ziua Dinaintea Examenului

- Parcurge `cheatsheets/` o ultima data — 25 minute
- NU invata nimic nou
- Odihna, somn bun

### Ziua 7: EXAMEN

- Regula: ~2 minute per intrebare
- Flag intrebarile dificile si revino la sfarsit
- Citeste TOATE optiunile inainte sa raspunzi
- Elimina raspunsurile evident gresite mai intai

---

## Ordinea de Citire Recomandata (Rezumat)

```
SAPTAMANA 1:  01 → 18 → 19 → 02           (Sectiunea 1 ~23%: Hierarchy + IAM + Billing + Cloud Asset Inventory)
SAPTAMANA 2:  03 → 04 → 05                 (Sectiunea 2 ~30%: Compute+Hyperdisk+Knative / Storage+Kafka+NetApp / Network)
SAPTAMANA 3:  06 → 07 → 08 → 09 → 10 → 11 (Sectiunea 3 ~27%: Deploy CE/GKE/Run/Functions/Data/NGFW/Fabric FAST)
SAPTAMANA 4:  12 → 13 → 14 → 15 → 16 → 17 (Sectiunea 4 ~20%: Manage + DB Center + Backups + Query Insights + Active Assist)
SAPTAMANA 5:  Cheat sheets + Flashcards + Scenarii (Consolidare)
SAPTAMANA 6:  Puncte slabe + Speed review   (Final prep)
```

---

## Sfaturi de Examen (Exam Day Tips)

1. **Citeste intrebarea de 2 ori** — Multe intrebari au cuvinte cheie: "MOST cost-effective", "LEAST operational overhead", "MINIMUM permissions"
2. **Elimina raspunsurile gresite** — De obicei poti elimina 2 din 4 imediat
3. **"Lift and shift" = Compute Engine** — Daca intrebarea zice "migrate as-is", raspunsul e aproape mereu CE
4. **"Minimize operational overhead" = Serverless** — Cloud Run > GKE > Compute Engine
5. **"Global + strongly consistent" = Spanner** — Singurul serviciu cu aceasta combinatie
6. **"Least privilege" = Predefined role** sau **Custom role** — Niciodata basic roles
7. **"Without service account keys" = Workload Identity** sau **Impersonation**
8. **Bugete NU opresc cheltuielile** — Trebuie Pub/Sub + Cloud Function
9. **VPC Peering e non-tranzitiv** — A↔B si B↔C NU inseamna A↔C
10. **Audit logs: Admin Activity = mereu activ, gratuit**
11. **"Migrate existing Kafka" = Managed Apache Kafka** — Pub/Sub necesita rescriere cod
12. **"Secure, cannot be spoofed" = Secure Tags** — network tags pot fi setate de oricine cu compute.instances.setMetadata
13. **"All database services in one place" = Database Center** — nu face modificari, doar afiseaza
14. **"Slow queries, which index is missing" = Query Insights** — disponibil in Cloud SQL si AlloyDB
15. **Hyperdisk vs Persistent Disk: Hyperdisk = IOPS/throughput independent** de marimea discului

---

## Checklist Final — Esti Pregatit?

### Sectiunea 1 — Setup (~23%)
- [ ] Pot desena resource hierarchy din memorie (Org → Folders → Projects → Resources)
- [ ] Inteleg IAM: basic vs predefined vs custom roles, policy inheritance
- [ ] Inteleg Service Accounts: creare, atribuire, impersonation, Workload Identity
- [ ] Stiu cum functioneaza billing: budgets (nu opresc cheltuielile!), export BigQuery, CUD vs SUD
- [ ] Stiu ce face Cloud Asset Inventory si comenzile gcloud asset
- [ ] Stiu ce face Gemini Cloud Assist (si ce NU poate face: nu actioneaza, doar asista)

### Sectiunea 2 — Planning (~30%)
- [ ] Pot alege serviciul de compute corect pentru orice scenariu (CE vs GKE vs Cloud Run vs Functions vs Knative)
- [ ] Pot alege baza de date corecta pentru orice scenariu (SQL vs Spanner vs BigQuery vs Firestore vs Bigtable)
- [ ] Inteleg Hyperdisk: IOPS si throughput independente de capacitate; tipuri (Balanced/Extreme/Throughput/ML)
- [ ] Stiu cand sa folosesc Managed Kafka vs Pub/Sub (Kafka = migrare; Pub/Sub = nativ GCP)
- [ ] Inteleg NetApp Volumes vs Filestore (enterprise NAS vs simplu NFS)
- [ ] Stiu cele 6 tipuri de load balancer si cand le folosesc
- [ ] Inteleg multi-region redundancy per serviciu (Spanner global config, Bigtable replication, etc.)

### Sectiunea 3 — Deploying (~27%)
- [ ] Stiu comenzile gcloud esentiale pentru fiecare serviciu principal
- [ ] Inteleg Cloud NGFW: hierarchical policies vs VPC policies, Secure Tags vs network tags
- [ ] Stiu cei 5 pasi ai Fabric FAST si diferenta fata de CFT
- [ ] Pot explica cei 6 pasi ai Workload Identity pentru GKE
- [ ] Inteleg IaC state management: GCS backend, versioning, prevent_destroy, terraform state rm
- [ ] Inteleg VPC: subnets, firewall rules, peering (non-tranzitiv!), VPN, NAT

### Sectiunea 4 — Operations (~20%)
- [ ] Inteleg monitoring: alerts, log sinks, audit logs (Admin Activity = mereu activ, gratuit!), Ops Agent
- [ ] Stiu comenzile de backup pentru Cloud SQL, AlloyDB, Spanner, Bigtable, Firestore
- [ ] Stiu ce face Database Center (panou unificat, read-only)
- [ ] Inteleg Query Insights (diagnosticare interogari lente)
- [ ] Inteleg Active Assist si principalele tipuri de recomandari
- [ ] Stiu diferenta Personalized Service Health vs status page public
- [ ] Inteleg GKE Autopilot pod requests: billing per pod, default-uri, OOMKilled = creste memory request

### General
- [ ] Pot explica diferentele cheie GCP vs AWS (VPC global, SUD automat, Spanner fara echivalent AWS)
- [ ] Am facut practice exam-ul Google si am obtinut 80%+
