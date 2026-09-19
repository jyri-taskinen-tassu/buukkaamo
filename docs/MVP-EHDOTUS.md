# Buukkaamo Agent OS — MVP-ehdotus

**Versio:** 0.2  
**Päiväys:** 2026-09-19  
**Tavoite:** Yrityksen hallinnointijärjestelmä, jossa AI-agentit hoitavat legal/tutkimusta, myyntiä, markkinointia ja analyysiä — sinä hyväksyt kriittiset asiat.  
**Budjettilinja:** pidä juoksevat kulut **mahdollisimman pieninä** (tavoite alussa ~0–50 €/kk).

---

## 1. Tiivistelmä

Rakennetaan **Agent OS**: yksi hallintapaneeli + agenttiroolit, jotka työskentelevät taustalla (myös yön yli). MVP automatisoi **valmistelun ja rutiinin**; ihminen hyväksyy ulospäin menevän.

**MVP:n lupaus**

> Aamulla avaat paneelin. Yön aikana agentit ovat valmistelleet research-briefin / riskiliput (ja myöhemmin follow-upit sekä markkinointiluonnokset). Hyväksyt / hylkäät / muokkaat — ja työ jatkuu.

Rakentuu nykyisen Buukkaamo-datamallin päälle (`Campaign`, `CallLog`, `Meeting`, `KpiSnapshot`).

**Prioriteetti (päivitetty):** Legal/Research **ensin**. Se on kuitenkin **modulaarinen** — voidaan kytkeä pois myöhemmin ilman että Sales/Marketing kaatuu.

---

## 2. Visio vs. MVP

| Alue | Visio (myöhemmin) | MVP (ensin) | Pakollinen? |
|------|-------------------|-------------|-------------|
| Legal / Research | Jatkuva seuranta + syvempi analyysi | Lähdelista → brief + riskiliput | **Ensin kyllä**, myöhemmin voi skipata |
| Myynti | Autonominen outreach | Priorisointi + viestiluonnokset | Seuraava |
| Markkinointi | Automaattijulkaisu | Draftit + Accept | Seuraava |
| Insights | Täysi konsultointi | KPI-yhteenveto omasta datasta | Halpa lisä |
| Hallinta | Koko yrityksen automaatio | Jono + audit + agent on/off | Kyllä (runko) |

**Periaate:** agentti ehdottaa, ihminen hyväksyy. Jokainen agentti on **kytkin**: `enabled: true/false`.

---

## 3. Agenttiroolit

### 3.1 Legal / Research Agent — “vahti” (ensin)

**Tekee**
- Käy läpi määritellyn lähdelistan (kilpailijat, toimialauutiset, julkinen sääntely)
- Tuottaa lyhyen “mitä muuttui” -briefin
- Liputtaa riskejä (väittämät, GDPR-huomiot, liialliset lupaukset)

**Ei tee**
- Sitovaa juridista neuvontaa
- Sopimusten allekirjoitusta / muutosta

**Skip later:** agentti voidaan sammuttaa asetuksista; Sales/Marketing eivät riipu siitä.

### 3.2 Sales Agent

Priorisoi leadit, luo follow-up-luonnokset, tuottaa päivälistan. Ei autonomista sulkemista MVP:ssä.

### 3.3 Marketing Agent

Luo sisältöluonnokset + aikataulun. **Julkaisu vain Acceptin kautta.**

### 3.4 Insights Agent

KPI-yhteenvedot Buukkaamo-datasta + 2–3 toimenpide-ehdotusta. Halpa, koska data on jo omassa DB:ssä.

---

## 4. Käyttäjäkokemus (MVP)

### Aamunäkymä

1. Hyväksyntäjono (briefit, riskiliput, myöhemmin draftit)  
2. Yön yhteenveto  
3. Agenttien status (on/off) + kulukatto  
4. Myöhemmin: myyntilista + KPI

### Hyväksyntävirta

```
Agentti tuottaa draftin / briefin
        ↓
pending_approval
        ↓
Approve / Edit / Reject
        ↓
Approve → tallenna / (myöhemmin) julkaise
```

---

## 5. Tekninen arkkitehtuuri (halpa polku)

- **App:** Next.js  
- **DB:** PostgreSQL + Prisma (nykyinen)  
- **Jobs:** yksinkertainen cron (ei erillistä jono-SaaS:ia alussa)  
- **LLM:** yksi **edullinen** malli + tiukat token-katot  
- **Lähteet Legalille:** manuaalinen URL-lista / RSS — **ei maksullista news-API:a** alussa  
- **Julkaisu:** copy/clipboard tai manuaalinen — **ei some-SaaS:ia** alussa  

### Uudet taulut (luonnos)

- `Agent` (rooli, `enabled`, budget_cap)
- `AgentRun` (ajo, tokenit, kustannusarvio)
- `ApprovalItem`
- `ResearchBrief`
- `ContentDraft` (myöhemmin)
- `PolicyRule`

---

## 6. MVP-laajuus

### Mukana heti

1. Command Center + hyväksyntäjono  
2. **Legal/Research Agent** (kytkettävä pois myöhemmin)  
3. Audit-loki + kk-/päiväkatto LLM:lle  
4. Agent on/off -kytkimet  

### Seuraavaksi

5. Sales Agent  
6. Marketing Agent  
7. Insights Agent  

### Pois (säästää rahaa)

- Maksulliset news/data-API:t  
- Some-julkaisutyökalut  
- Erillinen CRM  
- Autonominen cold calling / mainosbudjetti  
- Useita kalliita malleja rinnakkain  

---

## 7. Kustannukset — miksi aiemmat luvut näyttivät “kovilta”?

### Lyhyt vastaus

Aiemmat luvut olivat **haarukoita ylöspäin**: ne summasivat infraa, LLM:ää ja **valinnaisia** työkaluja “normaali/kasvu” -skenaarioissa. Ne eivät olleet minimi, jota tarvitaan.

**Halpa polku ei tarvitse lähes mitään kuukausimaksuja.**

### Mistä kulut oikeasti syntyvät?

| Kori | Onko pakko? | Selitys |
|------|-------------|---------|
| Hosting + DB | Melkein ilmainen alussa | Free/hobby-tier riittää MVP:lle |
| LLM API | Kyllä, pieni | Ainoa oikea muuttuva kulu: maksaa per ajo/token |
| Sähköposti-SaaS, some-SaaS, news-API, erillinen CRM | **Ei** | Nämä paisuttivat aiempaa “normaali/kasvu” -haarukkaa |
| Mainosmediat (Meta/Google) | Ei järjestelmän kulu | Markkinointibudjetti erillään |
| Ulkoinen ohjelmistotalo | Ei | Rakennetaan tähän repoon agenttivetoisesti |

Eli: **järjestelmä itsessään ei ole kallis** — kalliiksi tulee vasta kun lisätään SaaS-työkaluja, raskasta researchiä jatkuvasti, ja paljon LLM-ajoja ilman kattoa.

### 7.1 Cheap path (suositus)

| Kuluerä | Arvio / kk | Huomio |
|---------|------------|--------|
| Hosting + DB | **0–15 €** | Free-tierillä usein 0 € |
| LLM (Legal brief + muutama ajo/pv) | **5–30 €** | Edullinen malli + katot |
| Muut SaaS:it | **0 €** | Skipataan |
| **Yhteensä** | **~5–45 €/kk** | Tavoite |

### 7.2 Miten LLM pidetään halpana

1. **Yksi edullinen malli** (ei premiumia joka ajoon)  
2. **Päivä- ja kk-katto** euroissa (agentti pysähtyy)  
3. **Lyhyet outputit** (brief ½–1 sivu, ei 20 sivun raportteja)  
4. **Harva ajo** Legalille: esim. 1×/yö tai 3×/viikko — ei jatkuvaa pollausta  
5. **Omat lähteet** (URL-lista), ei maksullista data-API:a  
6. Legal **voidaan sammuttaa** → kulut tippuvat lähes nollaan muilta osin kunnes Sales/Marketing otetaan käyttöön  

### 7.3 Mitä aiemmat “150–400 €” ja “400–1000 €” tarkoittivat?

Ne olivat **kasvuskenaarioita**, joissa mukana:
- enemmän agentteja päivittäin  
- pidemmät raportit  
- mahdollisesti sähköposti-/some-työkaluja  
- mahdollisesti data-API  

Ne **eivät ole lähtöhinta**. Cheap path alkaa kymmenistä euroista tai alle.

### 7.4 Rakentamiskustannus

Ei erillistä lisenssiä. Työ tehdään tähän Buukkaamo-repoon. Isoin “kustannus” on päätökset: lähdelista, tone, mitä Acceptataan.

---

## 8. Vaiheistus (päivitetty)

### Vaihe A — Runko + Legal (ensin)
- Schema: Agent, AgentRun, ApprovalItem, ResearchBrief  
- Command Center + jono  
- Legal/Research Agent (URL-lista → brief + riskiliput)  
- LLM-katto + agent on/off (**Legal skipattavissa**)

### Vaihe B — Myynti
- Sales Agent + kampanjadata  
- Päivälista + viestiluonnokset  

### Vaihe C — Markkinointi
- Marketing drafts + Accept  

### Vaihe D — Insights + (valinnainen) kevyt automaatio
- KPI-brief  
- Vasta myöhemmin: sähköpostilähetys Acceptin jälkeen  

---

## 9. Mittarit

- Aamun hyväksyntä &lt; 15 min  
- Legal brief käyttökelpoinen ilman isoa editointia  
- LLM-kulut ≤ sovittu katto (esim. **30 €/kk**)  
- Legal voidaan sammuttaa ilman regressiota muihin agentteihin  
- 0 ulospäin julkaisua ilman Acceptia  

---

## 10. Riskit

| Riski | Hallinta |
|-------|----------|
| LLM-lasku karkaa | Eurokatto + harvat ajot |
| Legal “leikkii lakimiestä” | Vain research + checklist -kieli |
| Turha agentti myöhemmin | `enabled: false` |
| Hallusinaatiot | Pakolliset lähdelinkit briefissä |

---

## 11. Suositus (v0.2)

1. **Aloita Legal/Research + Command Center** (halpa, hyödyllinen, kytkettävissä pois)  
2. Pidä juoksevat kulut **~5–45 €/kk** -linjassa  
3. Lisää Sales ja Marketing vasta kun Legal-rutiini toimii  
4. Älä osta SaaS-työkaluja ennen kuin Accept-jono on käytössä  

---

## 12. Seuraava päätös

**A.** Aloita Vaihe A (Legal + jono + katot) — cheap path  
**B.** Legal ensin, mutta vielä kapeampi: vain viikkobrief (ei yön ajoja)  
**C.** Skip Legal heti → suoraan Sales (vastoin nykyistä prioriteettia)

Seuraava deliverable valinnan jälkeen: Prisma-laajennus + Command Center -runko + Legal-agentin ensimmäinen ajo.
