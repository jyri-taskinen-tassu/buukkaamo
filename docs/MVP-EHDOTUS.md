# Buukkaamo Agent OS — MVP-ehdotus

**Versio:** 0.1  
**Päiväys:** 2026-09-19  
**Tavoite:** Yrityksen hallinnointijärjestelmä, jossa AI-agentit hoitavat myyntiä, markkinointia, legal/tutkimusta ja analyysiä — sinä hyväksyt kriittiset asiat.

---

## 1. Tiivistelmä

Rakennetaan **Agent OS**: yksi hallintapaneeli + agenttiroolit, jotka työskentelevät taustalla (myös yön yli). MVP ei yritä automatisoida koko yritystä. Se automatisoi **valmistelun ja rutiinin**, ja pitää ihmisen päätöksissä, joissa on raha-, brändi- tai juridinen riski.

**MVP:n lupaus**

> Aamulla avaat paneelin. Yön aikana agentit ovat valmistelleet follow-upit, markkinointiluonnokset, riskiliput ja yhden markkinayhteenvedon. Hyväksyt / hylkäät / muokkaat — ja työ jatkuu.

Tämä rakentuu nykyisen Buukkaamo-datamallin päälle (`Campaign`, `CallLog`, `Meeting`, `KpiSnapshot`, käyttäjäroolit).

---

## 2. Visio vs. MVP

| Alue | Visio (myöhemmin) | MVP (ensin) |
|------|-------------------|-------------|
| Myynti | Autonominen outreach, neuvottelu, sulkeminen | Leadien priorisointi, follow-up-luonnokset, CRM-päivitykset, päivän myyntilista |
| Markkinointi | Automaattinen julkaisu kaikilla kanavilla | Sisältöluonnokset + **pakollinen hyväksyntä** ennen julkaisua |
| Legal | Jatkuva markkina-/sääntelyseuranta + sopimusneuvonta | Riskiliput, kilpailija-/sääntelyyhteenvedot, checklistit (ei juridista neuvoa) |
| Konsultointi | Täysi analyysitiimi | Viikoittainen markkinaraportti + kampanja-KPI-insights |
| Hallinta | Koko yrityksen automaatio | Hyväksyntäjono, audit-loki, agenttien status |

**Periaate:** agentti saa *ehdottaa ja valmistella*, ihminen *hyväksyy* kaiken ulospäin menevän.

---

## 3. Agenttiroolit (MVP)

### 3.1 Sales Agent — “yön myyjä”

**Tekee**
- Priorisoi leadit / kontaktit kampanjoittain (vanhat follow-upit, kuumat liidit)
- Luo seuraavan kontaktin luonnoksen (sähköposti / LinkedIn / soittoscript)
- Ehdottaa tapaamisaikoja ja päivittää CRM-kenttiä (status, next action)
- Tuottaa aamuksi “Top 10 toimenpiteet”

**Ei tee MVP:ssä**
- Automaattisia massasoittoja ilman ihmistä
- Hintaneuvottelua tai sopimuksen allekirjoitusta ilman hyväksyntää
- Lupausten tekemistä asiakkaalle ilman hyväksyttyä scriptiä

**Hyväksyntä:** ulospäin lähtevät viestit (paitsi jos erikseen sallittu “safe template” -moodi)

### 3.2 Marketing Agent — “sisältökone”

**Tekee**
- Luo some-postaukset, uutiskirjeet, landing-copyt, kampanjateemat
- Ehdottaa julkaisuaikataulun
- Muuntaa hyväksytyn kontsan kanavakohtaisiin versioihin

**Ei tee MVP:ssä**
- Julkaise mitään ilman Accept-nappia
- Käytä budjettia mainontaan ilman erillistä budjettisääntöä

**Hyväksyntä:** aina ennen julkaisua (ydinominaisuus)

### 3.3 Legal / Research Agent — “vahti”

**Tekee**
- Seuraa määriteltyjä lähteitä (kilpailijat, toimialauutiset, julkinen sääntely)
- Liputtaa mahdollisia riskejä (väittämät markkinoinnissa, GDPR-huomiot, lupaukset myyntiscripteissä)
- Tuottaa lyhyen “mitä muuttui” -yhteenvedon

**Ei tee MVP:ssä**
- Anna sitovaa juridista neuvontaa
- Muuta sopimuksia ilman ihmistä

**Hyväksyntä:** riskiliput menevät jonoon; kriittiset estävät julkaisun kunnes katsottu

### 3.4 Insights / Consulting Agent — “analyytikko”

**Tekee**
- Kampanja-KPI-yhteenvedot olemassa olevasta datasta (`KpiSnapshot`, `CallLog`, `Meeting`)
- Viikoittaisen markkina-/kilpailijayhteenvedon
- Ehdottaa 2–3 konkreettista toimenpidettä (hinnoittelu, kanava, script)

**Ei tee MVP:ssä**
- Strategiapäätöksiä itsenäisesti

---

## 4. Käyttäjäkokemus (MVP)

### Aamunäkymä (Command Center)

1. **Hyväksyntäjono** — markkinointiluonnokset, myyntiviestit, riskiliput  
2. **Yön yhteenveto** — mitä agentit tekivät, mitä odottaa sinua  
3. **Myynnin päivälista** — priorisoidut kontaktit + luonnokset  
4. **KPI-strip** — soitetut, bookatut, conversion (olemassa olevasta mallista)

### Hyväksyntävirta

```
Agentti tuottaa draftin
        ↓
Status: pending_approval
        ↓
Sinä: Approve / Edit / Reject
        ↓
Approve → queue julkaisuun / lähetykseen
Reject  → palaute agentille (oppii prompt-/policy-tasolla)
```

### Audit

Jokaisesta agenttitoiminnasta tallennetaan: aika, agentti, input-yhteenveto, output, päätös (approve/reject), käyttäjä.

---

## 5. Tekninen arkkitehtuuri (ehdotus)

### Stack (linjassa nykyiseen)

- **App:** Next.js (hallintapaneeli)
- **DB:** PostgreSQL + Prisma (laajennetaan nykyistä `schema.prisma`)
- **Auth:** roolipohjainen (`admin` omistaa hyväksynnät)
- **Jobs:** taustajonot (cron / queue) yön ajoille
- **LLM:** yksi päämalli + edullisempi malli rutiiniin
- **Integraatiot (MVP-minimi):** sähköposti, kalenteri, (myöhemmin) LinkedIn / Meta / Google Ads

### Uudet ydintaulut (luonnos)

- `Agent` — rooli, status, config
- `AgentRun` — ajo, trigger, token-käyttö, tulos
- `ApprovalItem` — tyyppi, payload, status, reviewer
- `ContentDraft` — markkinointi-/myyntiluonnos
- `ResearchBrief` — legal/insights-yhteenveto
- `PolicyRule` — mitä agentti saa tehdä ilman hyväksyntää

Nykyiset `Campaign`, `CallLog`, `Meeting`, `KpiSnapshot` jäävät myynnin totuuden lähteeksi.

### Turvallisuusperiaatteet

- Ulospäin menevä = default deny (hyväksyntä pakollinen)
- Budjettikatto per agentti / päivä
- PII-minimointi promptteihin
- Kaikki ajot lokitetaan

---

## 6. MVP-laajuus (mitä rakennetaan ensin)

### Mukana

1. Command Center + hyväksyntäjono  
2. Sales Agent: priorisointi + viestiluonnokset kampanjoille  
3. Marketing Agent: sisältöluonnokset + approve/publish-jono  
4. Research Agent: viikoittainen brief + riskiliput draftiin  
5. Insights Agent: KPI-yhteenveto Buukkaamo-datasta  
6. Audit-loki + perusasetukset (tone of voice, kielto-listat, brand rules)

### Pois MVP:stä (tietoinen rajaus)

- Täysin autonominen cold calling
- Automaattinen mainosbudjetin käyttö
- Sopimusten automaattinen generointi + allekirjoitus
- Moniyrittäjä / white-label tuote
- Syvä integraatio kaikkiin somekanaviin kerralla

---

## 7. Kustannusarvio

Kustannukset jakautuvat kolmeen koriin: **infra**, **AI-käyttö**, **työkalut**. Alle on realistinen haarukka pienelle suomalaiselle B2B-operaatiolle.

### 7.1 Kiinteä / kuukausittainen infra

| Kuluerä | Arvio / kk | Huomio |
|---------|------------|--------|
| Hosting (esim. Vercel / vastaava) | 0–40 € | Alkuun usein ilmainen/hobby riittää |
| Tietokanta (Supabase / Postgres) | 0–25 € | Nykyinen suunta sopii |
| Taustajonot / cron | 0–20 € | Voi olla osana hostia |
| Domain + sähköposti (transaktio) | 10–40 € | Lähetysvolyymista riippuen |
| **Infra yhteensä** | **~20–120 €/kk** | |

### 7.2 AI / LLM-käyttö (muuttuva)

Käyttö riippuu siitä, kuinka paljon agentit generoivat.

| Käyttötaso | Arvio / kk | Mitä se tarkoittaa |
|------------|------------|--------------------|
| Kevyt | 30–80 € | Muutama kymmenen draftia/pv + viikkoraportti |
| Normaali MVP | 80–250 € | Päivittäiset myyntiluonnokset + markkinointi + research |
| Raskas | 250–700+ € | Paljon pitkiä raportteja, monikanavasisältöä, jatkuvaa researchiä |

**Kustannusvipu:** edullisempi malli rutiiniin, kalliimpi vain hyväksyntäkriittisiin teksteihin; tiukat promptit; caching; max tokens per run.

### 7.3 Työkalut & integraatiot (valinnainen MVP:ssä)

| Kuluerä | Arvio / kk | Pakko MVP:ssä? |
|---------|------------|----------------|
| Sähköpostityökalu / sequenser | 0–50 € | Ei (voi aloittaa manuaalisella lähetyksellä hyväksynnän jälkeen) |
| Some-julkaisutyökalu | 0–60 € | Ei (MVP = draft + copy clipboard / manuaalinen publish) |
| Data / news API | 0–100 € | Ei (voi aloittaa julkisilla lähteillä + manuaalisilla URL-listoilla) |
| CRM (jos erillinen) | 0–80 € | Ei, jos Buukkaamo-DB on CRM |

### 7.4 Kokonaisarvio operatiivisista kuluista

| Vaihe | Arvio / kk |
|-------|------------|
| MVP kevyt | **~50–200 €** |
| MVP normaali käyttö | **~150–400 €** |
| Kasvu (enemmän agentteja + kanavia) | **~400–1000+ €** |

Nämä ovat **järjestelmän juoksevia kuluja**, eivät myynnin mediaostoksia (Google/Meta-budjetti on erillinen).

### 7.5 Rakentamisen investointi (työ)

Koska toteutus voidaan tehdä agenttivetoisesti nykyisen scheman päälle, isoin “kustannus” on priorisointi ja hyväksyntäsääntöjen määrittely — ei lisenssimaksu.

Suuntaa-antava vaihtoehtoinen hinta, jos ostettaisiin ulkoa perinteisenä projektina:

| Kokonaisuus | Haarukka |
|-------------|----------|
| MVP Command Center + 2 agenttia (Sales + Marketing) | tyypillisesti nelinumeroinen–matala viisinumeroinen € |
| + Research/Insights + audit + jonot | lisäkerros samaa suuruusluokkaa |
| Täysi visio (autonominen myynti + kanavat + legal) | selvästi suurempi, vaiheistettava |

Suositus: **rakenna agenttivetoisesti vaiheittain** nykyiseen Buukkaamo-repoon — säästää merkittävästi verrattuna perinteiseen kokonaistoimitukseen.

---

## 8. Vaiheistus

### Vaihe A — Perusta
- ApprovalItem + ContentDraft -malli
- Command Center (jono + yön yhteenveto)
- Yksi agentti: Marketing draftit

### Vaihe B — Myynti
- Sales Agent + kampanjadata (`Campaign` / `CallLog` / `Meeting`)
- Päivälista + viestiluonnokset
- KPI-yhteenveto Insights-agentilla

### Vaihe C — Vahti & analyysi
- Research Agent + riskiliput
- Viikkobrief
- Policy rules (mitä saa tehdä ilman Acceptia)

### Vaihe D — Ulospäin automaatio
- Sähköpostilähetys hyväksynnän jälkeen
- Some-publish-integraatio
- Budjettikatot ja safe templates

Jokaisen vaiheen jälkeen mitataan: säästetty aika, hyväksyntäprosentti, bookatut tapaamiset, virheet/riskiliput.

---

## 9. Menestyksen mittarit (MVP)

- **Aika hyväksyntään:** &lt; 15 min / aamu
- **Hyväksyttyjen draftien osuus:** &gt; 60 % ilman isoa editointia
- **Myynti:** follow-up coverage (jokaisella hot leadillä next action)
- **Markkinointi:** julkaisutiheys ilman laadun romahdusta
- **Riski:** 0 julkaisua ilman Acceptia; kriittiset liput katsottu ennen publishia
- **Kustannus:** LLM &lt; sovittu kk-katto

---

## 10. Riskit ja hallintakeinot

| Riski | Hallinta |
|-------|----------|
| Agentti “hallusinoi” faktoja | Lähteet pakollisiksi researchissä; brand facts -tiedosto |
| Brändivahinko somessa | Julkaisu vain Acceptin kautta |
| Hallitsematon LLM-lasku | Päivä-/kk-katto, max runit, edullinen malli rutiiniin |
| Liikaa hyväksyttävää | Erähyväksyntä, templatet, auto-approve vain matalan riskin tyypeille myöhemmin |
| Juridinen yliymmärrys | Legal Agent = research + checklist, ei “lakimies” |

---

## 11. Suositus

**Tehdään MVP näin:**

1. **Command Center + hyväksyntäjono** (pakollinen runko)  
2. **Marketing Agent** ensin (nopein näkyvä arvo, matala riski Accept-mallilla)  
3. **Sales Agent** heti perään (hyödyntää olemassa olevaa Buukkaamo-dataa)  
4. **Insights + Research** kolmantena (raportit ja riskiliput)  

Operatiivinen kk-kustannus pysyy alussa tyypillisesti **alle muutaman satasen**, jos käyttöä ohjataan katoilla.

---

## 12. Seuraava päätös

Valitse yksi lähtö:

**A.** Hyväksy tämä MVP-rajaus → aloitetaan Vaihe A (schema + Command Center + Marketing drafts)  
**B.** Kavenna: vain Sales + hyväksyntäjono  
**C.** Laajenna jo MVP:hen yksi kanavaintegraatio (esim. sähköposti hyväksynnän jälkeen)

Kun valinta on selvä, seuraava konkreettinen deliverable on: Prisma-laajennus agentti-/approval-tauluille + Clickable Command Center -luonnos.
