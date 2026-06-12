# Plan de studiu — 28 de zile, construit pentru cineva care amână

> Acest plan nu presupune că ai disciplină. Presupune că NU ai — și e construit să funcționeze oricum.
> Bifează checkbox-urile direct în acest fișier, pe măsură ce avansezi.

---

## Regulile anti-amânare (mai importante decât planul însuși)

**1. Angajamentul zilnic e de 5 minute, nu de o oră.**
Promisiunea pe care ți-o faci NU e „învăț o oră pe zi" — e „deschid lecția de azi și citesc 5 minute". Atât. Amânarea se naște din cât de mare pare sarcina la pornire; 5 minute nu poți amâna. În 90% din cazuri, odată pornit, continui de la sine. În restul de 10%, ai voie să te oprești după 5 minute cu conștiința împăcată — ziua se bifează.

**2. Aceeași oră, același loc, lipit de un obicei existent.**
Nu decide în fiecare zi *când* înveți — decizia zilnică e exact locul unde câștigă amânarea. Alege o singură formulă și scrie-o aici:

> Eu învăț: după ____________ (ex: cafeaua de dimineață / cina), la ora ____, în ____________.

**3. Niciodată două zile ratate la rând.**
O zi ratată e normală — viața există. Două la rând omoară planul (a treia nu mai vine niciodată). Regula de salvare: în ziua de după o zi ratată, dacă n-ai timp/chef, fă DOAR fișierul `-map.md` al lecției (10 minute) — și ziua se pune.

**4. Programezi examenul în ZIUA 3 — cu bani.**
Acesta e cel mai puternic instrument din tot planul. Un examen plătit, cu dată fixă peste 4 săptămâni, transformă „ar trebui să învăț" în „am termen". Nu aștepta să te simți pregătit ca să-l programezi — îl programezi ca să devii pregătit. (Reprogramarea e posibilă dacă chiar e nevoie, deci riscul real e mic.)

**5. O zi = un checkbox. Interzis să lucrezi în avans mai mult de o zi.**
Sună ciudat, dar e anti-binge: entuziasmul din ziua 2 („fac trei lecții azi!") e exact ce produce epuizarea din ziua 5. Sarcina mică și repetată bate sprintul.

**6. Citirea pasivă nu se pune. Fiecare zi are un „livrabil".**
Ziua se bifează doar dacă ai produs ceva: scenariile rezolvate, 3 idei scrise din memorie, un lab făcut. Ochii trecuți peste pagină nu sunt învățare.

**7. Telefonul în altă cameră.** 45–60 de minute. Nu pe silențios — în altă cameră.

**8. După bifare, recompensă mică.** Serial, joc, plimbare — ceva concret, imediat, în fiecare zi. Creierul tău trebuie să asocieze studiul cu „după asta urmează ceva bun".

---

## Cum arată o sesiune (45–60 min)

1. **(5 min)** Deschide `-map.md`-ul lecției de IERI. Recitește-l. (Asta e repetiția spațiată — face mai mult decât orice altă tehnică.)
2. **(30–40 min)** Lecția de azi. Citește activ: la fiecare „Micro-summary", închide ochii și spune-l cu voce tare înainte să-l citești.
3. **(10 min)** Livrabilul zilei: rezolvă *Worked scenarios* / *Mini-Q&A* din lecție FĂRĂ să te uiți la răspunsuri, apoi verifică.
4. **(1 min)** Bifează ziua aici. Recompensa.

---

## Săptămâna 1 — Fundații (ierarhie, bani, CLI, IAM)

- [ ] **Ziua 1** — `1.1-projects-and-accounts.md`, prima jumătate (până la org policies inclusiv). Livrabil: desenează din memorie ierarhia Org → Folders → Projects → Resources și scrie regula IAM=union / OrgPolicy=intersection.
- [ ] **Ziua 2** — `1.1` a doua jumătate + `1.1-...-map.md`. Livrabil: scenariile din lecție.
- [ ] **Ziua 3** — `1.2-billing.md`. **LIVRABIL OBLIGATORIU: programează examenul pe Webassessor, peste ~4 săptămâni.** (Da, azi. Vezi regula 4.)
- [ ] **Ziua 4** — `1.3-cloud-sdk-shell.md` + deschide Cloud Shell în consolă și rulează 5 comenzi din lecție. Livrabil: fișierul `persist-test.txt` creat în `$HOME`.
- [ ] **Ziua 5** — `4.1-iam.md` (da, IAM acum — apare în toate celelalte lecții). Livrabil: Mini-Q&A fără să te uiți.
- [ ] **Ziua 6 (ușoară)** — doar map-urile: 1.1, 1.2, 1.3, 4.1. Lab mic: creează un proiect nou + un buget cu alertă la 50%.
- [ ] **Ziua 7 — LIBERĂ.** (Sau recuperare, dacă ai ratat o zi. Dacă n-ai ratat: chiar liberă — pauza e parte din plan, nu trișare.)

**Streak săptămâna 1: ___ / 6 zile**

## Săptămâna 2 — Compute & Storage (cele mai grele două lecții)

- [ ] **Ziua 8** — `2.1-compute.md`, prima jumătate (Compute Engine, tipuri de mașini, Spot, discounturi). Livrabil: explică-i cuiva (sau peretelui) diferența Spot vs. Standard vs. CUD.
- [ ] **Ziua 9** — `2.1` a doua jumătate (GKE, Cloud Run, Functions) + map. Livrabil: scenariile.
- [ ] **Ziua 10** — `2.2-storage-and-data.md`, prima jumătate (Cloud Storage, clase, lifecycle). Livrabil: scrie din memorie minimele Nearline/Coldline/Archive (30/90/365).
- [ ] **Ziua 11** — `2.2` a doua jumătate (bazele de date) + **tabelul de decizie din secțiunea 3 — învață-l ca pe poezie**. Livrabil: acoperă coloana dreaptă a tabelului și ghicește produsul pentru fiecare rând.
- [ ] **Ziua 12 (lab)** — în consolă: un VM, un bucket cu regulă de lifecycle, un deploy pe Cloud Run (imaginea hello). Livrabil: toate trei există și apoi le ștergi.
- [ ] **Ziua 13 (ușoară)** — map-urile 2.1 + 2.2. Refă tabelul de decizie cu coloana acoperită.
- [ ] **Ziua 14 — LIBERĂ / recuperare.**

**Streak săptămâna 2: ___ / 6 zile**

## Săptămâna 3 — Networking, IaC, Operations

- [ ] **Ziua 15** — `2.3-networking.md`, prima jumătate (VPC, subnets, firewall). Livrabil: scrie din memorie: VPC=global, subnet=regional, firewall implicit = deny ingress / allow egress.
- [ ] **Ziua 16** — `2.3` a doua jumătate (LB, peering, Shared VPC, VPN/Interconnect) + map. Livrabil: scenariile.
- [ ] **Ziua 17** — `2.4-iac.md` (Terraform & co). Livrabil: Mini-Q&A.
- [ ] **Ziua 18** — `3.1-managing-compute.md` (snapshots, MIG, IAP, Marketplace, rollback Cloud Run). Livrabil: scenariile.
- [ ] **Ziua 19** — `3.2-managing-storage-and-data.md` + `3.3-managing-networking.md` (sunt mai scurte, merg împreună). Livrabil: scenariile din ambele.
- [ ] **Ziua 20 (lab)** — VPC custom cu 2 subnets + regulă de firewall + un VM fără IP extern în care intri cu IAP (`--tunnel-through-iap`). Livrabil: SSH-ul funcționează.
- [ ] **Ziua 21 — LIBERĂ / recuperare.**

**Streak săptămâna 3: ___ / 6 zile**

## Săptămâna 4 — Monitoring, securitate, și cele două examene de probă

- [ ] **Ziua 22** — `3.4-monitoring-and-logging.md`. Livrabil: scrie din memorie diferența Admin Activity (mereu pornit, 400 zile) vs. Data Access (oprit implicit).
- [ ] **Ziua 23** — `4.2-service-accounts.md` (inclusiv Secret Manager) + recitește map-ul 4.1. Livrabil: scenariile.
- [ ] **Ziua 24** — **MOCK EXAM 1** (`mock-exam.md`), cronometrat, 2 ore, fără pauze, fără telefon. Livrabil: scorul scris aici: ___ / 50.
- [ ] **Ziua 25** — Analiza mock 1: pentru FIECARE greșeală, recitește secțiunea de lecție indicată în cheia de răspunsuri. Livrabil: lista greșelilor + domeniul cel mai slab: ____________.
- [ ] **Ziua 26** — Recitire țintită: lecția/lecțiile domeniului cel mai slab + toate tabelele „Exam gotchas".
- [ ] **Ziua 27** — **MOCK EXAM 2** (`mock-exam-2.md`), cronometrat. Livrabil: scorul: ___ / 50. **≥ 43/50 (85%) = ești gata.** 40–42 = încă o zi pe greșeli, apoi gata. Sub 40 = reprogramează examenul cu o săptămână și repetă zilele 25–27.
- [ ] **Ziua 28** — Analiza mock 2 + `cram-sheet.md`, prima lectură completă.

**Streak săptămâna 4: ___ / 7 zile**

## Zilele examenului

- [ ] **Seara dinainte** — DOAR `cram-sheet.md`. Nimic altceva. Nicio lecție nouă — în acest punct, materialul nou doar sapă anxietate. Somn devreme.
- [ ] **Dimineața examenului** — `cram-sheet.md` încă o dată, la cafea. La examen: întrebările scurte primele, cele lungi le marchezi și revii, nicio întrebare lăsată goală.
- [ ] **EXAMEN** — data programată: ____________

---

## Dacă planul deraiază (citește asta când se întâmplă, nu dacă)

- **Ai ratat 3+ zile?** Nu „o iei de la capăt" (asta e o formă de amânare). Continui exact de unde ai rămas, azi, cu regula de 5 minute. Examenul programat nu s-a mișcat — el e planul, fișierul ăsta e doar harta.
- **O lecție ți se pare imposibilă?** Citește doar map-ul ei + tabelul de decizie + gotchas. Treci mai departe. Mock-urile îți vor spune dacă trebuie să te întorci — nu intuiția, care la materia grea zice mereu „nu știu nimic".
- **Simți că „nu ești pregătit" în ziua 27 deși ai 85%+?** Sentimentul ăsta nu dispare niciodată, la nimeni. 85% pe un mock nevăzut E pregătirea. Du-te.
