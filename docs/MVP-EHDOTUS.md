# New Company Agent OS — Plan review (v0.3)

**Versio:** 0.3  
**Päiväys:** 2026-09-19  
**Konteksti:** Uusi yritys. Halpa polku. Legal ensin (myöhemmin skipattavissa).  
**Tärkeä lähtötieto:** Myyntiin on **jo** AI + email -järjestelmä: Slack-komento (esim. “send ten audit messages”) → lähettää CRM-sheetin kontakteille. **Emme rakenna sitä uudelleen.**

---

## 1. Mitä rakennetaan — ja mitä ei

| Kerros | Rakennetaanko? | Miksi |
|--------|----------------|------|
| Slack → email outreach → CRM sheet | **Ei** | Sinulla on jo. Uudelleenrakennus = rahan hukkaa |
| Legal / Research + Accept-jono | **Kyllä (ensin)** | Puuttuu; halpa; voi sammuttaa myöhemmin |
| Marketing Studio (LinkedIn / email / IG…) | **Kyllä (ydin uuteen firmään)** | Puuttuu; tämä on “content + approve + schedule” |
| Sales Agent (täysi outbound) | **Ei MVP:ssä** | Päällekkäinen nykyisen kanssa |
| Sales “thin layer” (valinnainen myöhemmin) | Ehkä | Vain jos tuo jotain mitä Slack-flow ei tee |

**Rahallinen nyrkkisääntö:** älä maksa kahdesti samasta työstä (lähetä N viestiä sheetistä).

---

## 2. Sales — mitä se tekisi, mitä se osaisi, onko se rahan arvoista?

### 2.1 Mitä sinulla on jo (oletus)

```
Slack: "send 10 audit messages"
        ↓
AI + email system
        ↓
CRM sheet (kontaktit / status)
        ↓
Viestit lähtevät
```

Tämä kattaa jo: **volume outreach**, sheet-pohjaisen CRM:n, komennettavan lähetyksen.

### 2.2 Mitä “Sales Agent” *voisi* tehdä teoriassa

| Kyky | Kuvaus | Tarvitaanko sinulle? |
|------|--------|----------------------|
| Batch send N emails | Slack → sheet → send | **Ei — jo olemassa** |
| Kirjoita cold email -template | AI draftaa audit/myyntiviestin | Todennäköisesti jo olemassa |
| Priorisoi ketä lähestyä | Ranking sheetistä | Ehkä, jos sheetissä ei ole logiikkaa |
| Follow-up timing | “Kenelle ei vastattu 5 pv” | Ehkä hyödyllinen |
| Reply handling | Ehdottaa vastausta saapuneeseen | Hyödyllinen, jos ei ole |
| Meeting prep | Brief ennen callia | Hyödyllinen later |
| Pipeline coaching | “Nämä 5 dealia liikkuvat” | Later |
| Autonominen neuvottelu / close | Agentti sulkee diilin | Ei MVP, ei cheap path |
| Soittaminen | Voice/outbound calls | Ei tässä suunnitelmassa |

### 2.3 Suositus Salesille (uusi yritys + olemassa oleva AI email)

**MVP: älä rakenna Sales Agentia outboundiin.**

Se olisi **waste of money**, jos se vain toistaa: “lähetä 10 viestiä sheetille.”

**Myöhemmin (vain jos tarve):** ohut kerros, joka **ei korvaa** Slack-flow’ta vaan täydentää:

1. **Reply coach** — saapunut vastaus → ehdotettu vastaus Acceptilla  
2. **Stale follow-up list** — “nämä 12 eivät vastanneet” → sinä päätät, Slack-komento hoitaa sendin  
3. **Nightly sales digest** — lyhyt yhteenveto sheetistä (lähetetty / vastattu / bookattu)

Integraatiomalli:

```
[Olemassa oleva] Slack + email + CRM sheet   ← pysyy totuuden lähteenä lähetykselle
        ↑
[Uusi, valinnainen] Sales digest / reply drafts  ← lukee sheettiä, ei lähetä itse
```

### 2.4 Onko Sales-agentti rahan arvoinen?

| Vaihtoehto | Arvo | Kustannuslogiikka |
|------------|------|-------------------|
| Uudelleenrakenna email outreach | **Huono** | Tupla työ + tupla ylläpito |
| Älä rakenna Salesia MVP:ssä | **Hyvä** | 0 € ylimääräistä |
| Thin layer (digest + replies) myöhemmin | **OK** | Pieni LLM-kulu, iso hyöty vain jos replyt ovat pullonkaula |

**Verdict:** Sales = **out of MVP scope** paitsi jos erikseen pyydät thin layeriä. Rahasi menevät Marketingiin + Legaliin.

---

## 3. Marketing — mitä järjestelmä tarkalleen on?

### 3.1 Yksi lause

**Marketing Studio** = agentti tuottaa kanavakohtaiset draftit → sinä Accept/Edit → julkaisu (aluksi puolimanuaalinen, myöhemmin kytketty API).

Ei ole “mainostoimisto joka polttaa budjettia itsestään.”  
Ei ole Meta Ads Manager.  
Se on **sisällön tuotanto + hyväksyntä + kanavajono** uudelle brändille.

### 3.2 Kanavat (mitä tuetaan ja miten)

| Kanava | Mitä agentti tuottaa | Julkaisu MVP:ssä | Myöhemmin |
|--------|----------------------|------------------|-----------|
| **LinkedIn** (org / personal) | Post copy, carousel-tekstit, kommentointiehdotukset | Accept → copy tai schedule-jonoon | LinkedIn API / Buffer-tyyppinen |
| **Email** (newsletter / nurture) | Subject + body + CTA | Accept → lähetys olemassa olevalla email-työkalulla **tai** manuaalisesti | Automaattinen send Acceptin jälkeen |
| **Instagram** | Caption, hashtag-ehdotus, reel/script outline | Accept → copy paste / Creator Studio | Meta Graph API |
| **(Valinnainen later)** X / TikTok / blog | Sama malli: draft → Accept | Skip MVP | Lisää kanava kytkimellä |
| **Ads copy** (LinkedIn/Meta) | Mainostekstit + kulmat | Vain draft; **budjettia ei käytetä automaattisesti** | Ihminen laittaa Ads Manageriin |

**Email-markkinointi ≠ sales Slack-send.**  
Sales sheet-outreach = 1:1 / cold.  
Marketing email = lista, newsletter, nurture, brand.

### 3.3 Päivittäinen / viikoittainen flow

```
Brand kit (ääni, kieltomaiset väittämät, CTA:t, logo-linkit)
        ↓
Marketing Agent ajaa (esim. 3×/viikko tai yön ajo)
        ↓
Tuottaa paketin, esim.:
  - 3× LinkedIn post
  - 2× Instagram caption (+ reel outline)
  - 1× email newsletter draft
        ↓
Kaikki → Approval queue
        ↓
Sinä: Approve / Edit / Reject  (< 15 min)
        ↓
Approved → "Ready to publish" per channel
        ↓
MVP: sinä julkaiset (tai yksi klikki myöhemmin)
Legal (jos on): voi liputtaa riskiväittämiä ennen Acceptia
```

### 3.4 Mitä UI:ssa näkyy (Marketing)

- **Calendar** — miltä viikko näyttää per kanava  
- **Queue** — pending drafts  
- **Brand rules** — tone, banned claims, links  
- **Per-channel variants** — sama idea → LinkedIn vs IG vs email -versiot  
- **Status:** draft → pending → approved → published (manual check)  

### 3.5 Mitä Marketing **ei** tee cheap pathissa

- Ei osta mainoksia eikä liikuta ad-budjettia  
- Ei skrapaa Instagramia massana  
- Ei tarvitse maksullista social suitea day-1  
- Ei korvaa designeria (kuvat: aluksi template / oma asset / myöhemmin erillinen image-step)

### 3.6 Onko Marketing rahan arvoinen?

**Kyllä — uudelle firmalle tämä on todennäköisesti arvokkain uusi osa**, koska:
- Sales outreach on jo hoidossa  
- Brändi tarvitsee tasaisen läsnäolon LinkedIn + IG + email  
- Accept-malli pitää riskin pienenä  
- LLM-kulu pysyy pienenä jos ajetaan harvoin ja lyhyillä teksteillä  

Arvio LLM:lle Marketing-only kevyesti: usein **~5–25 €/kk** (muutama paketti / viikko).

---

## 4. Legal — lyhyt muistutus (ensin, skip later)

- URL-lista → brief + riskiliput (esim. väittämät markkinointidrafteissa)  
- `enabled: false` milloin tahansa  
- Ei lakimieskorvike  

Uudelle firmalle hyödyllinen alussa (väittämät, kilpailija, compliance-huomiot), ei pakollinen ikuisesti.

---

## 5. Mitä tarvitaan oikeasti (prioriteetti uudelle firmalle)

### Worth money

1. **Marketing Studio** (LinkedIn + email + Instagram drafts + Accept)  
2. **Approval / Command Center** (yhteinen jono)  
3. **Legal/Research** (ensin, kytkettävä pois)  
4. **LLM-katot** (jotta pysyy halvana)  

### Not worth money (nyt)

1. Uusi Sales outbound -moottori (Slack+email+sheet jo hoitaa)  
2. Maksullinen news-API  
3. Täysi social publishing SaaS day-1  
4. Autonominen ads spend  
5. Toinen CRM Buukkaamo-scheman / sheetin rinnalle “varmuuden vuoksi”

### Maybe later (thin, cheap)

- Sales digest / reply coach lukemaan CRM-sheettiä  
- API-julkaisu Acceptin jälkeen (LinkedIn/IG)  
- Insights viikkoraportti  

---

## 6. Päivitetty vaiheistus

### Vaihe A — Runko + Legal
Command Center, ApprovalItem, Legal brief, agent on/off, €-katto

### Vaihe B — Marketing Studio (uuden firman ydin)
Kanavat: **LinkedIn, Email, Instagram**  
Draft packets → Accept → ready-to-publish  
Brand kit

### Vaihe C — (valinnainen) Sales thin layer
Vain digest + reply drafts; **lähetys jää nykyiseen Slack-järjestelmään**

### Vaihe D — Julkaisuautomaatio
API/schedule hyväksytyille posteille; ads copy edelleen manuaalinen budjetti

---

## 7. Cheap path kulut (tämä rajaus)

| | Arvio / kk |
|--|------------|
| Hosting/DB | 0–15 € |
| LLM Legal + Marketing | 10–40 € |
| Sales rebuild | **0 €** (ei tehdä) |
| Social SaaS | 0 € MVP |
| **Yhteensä** | **~10–50 €/kk** |

---

## 8. Seuraava päätös

**A.** Vaihe A (Legal + jono), sitten B (Marketing Studio LI/Email/IG) — Sales skip  
**B.** Skip Legal → suoraan Marketing Studio (jos Legal ei nyt tärkeä)  
**C.** Silti haluat Sales thin layerin (digest/replies) jo B:n rinnalle  

Kerro myös Marketingiin: julkaistaanko MVP:ssä **vain draft+Accept** (halvin) vai heti **schedule/publish-kytkentä** LinkedIniin/IG:hen (enemmän työtä, vähän enemmän kulua/riskiä).
