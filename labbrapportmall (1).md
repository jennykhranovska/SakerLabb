# Labbrapport: praktisk laboration

*Kunskapskontroll 2, IT-säkerhet för utvecklare. Fyll i mallen och lämna in som PDF tillsammans med länken till ditt repo. Riktlängd två till tre sidor.*

**Namn:**
**Datum:**
**Repo (länk till din fork):**
**Applikation som analyserades:**

---

## 1. Kort om applikationen och analysen

Beskriv i några meningar vilken app du analyserade, vad den gör och hur du genomförde analysen. Ange vilka verktyg du använde och hur du körde dem (CodeQL default setup med språk C#, ZAP passiv och aktiv skanning mot vilken adress).

*Skriv här.*

---

## 2. Fem fynd

Fyll i tabellen. Minst ett fynd ska komma från statisk analys (CodeQL) och minst ett från dynamisk analys (ZAP). Spara bevis i form av skärmbild eller rapportutdrag och hänvisa till det per fynd.

| Nr | Källa (CodeQL/ZAP) | Regel-id eller alert | Allvarlighet (+ confidence för ZAP) | Fil och rad eller URL | Verkligt eller falskt positivt | Motivering (2–4 meningar) |
|----|--------------------|----------------------|-------------------------------------|-----------------------|--------------------------------|---------------------------|
| 1 | CodeQL|  cs/xml/insecure-dtd-handling| Critical | SakerLabb.Web/Services/ImportService.cs, rad 27 | Verkligt | Fyndet är verkligt eftersom appen tar emot XML från användaren och läser den utan tillräckligt skydd. XML-läsaren har inställningar som kan göra det möjligt att läsa osäkert innehåll.|
| 2 | CodeQL | cs/command-line-injection |Critical  | SakerLabb.Web/Services/ImportService.cs, rad 57 |  Verkligt|  |Användaren kan skriva in ett värde som sparas i host. Appen använder sedan värdet direkt i ett kommando utan att kontrollera det först.  
| 3 | ZAP | Content Security Policy (CSP) Header Not Set |Medium, Confidence: High  | http://localhost:5080/ | Verkligt | Fyndet är verkligt eftersom jag kontrollerade Response i ZAP och såg att Content-Security-Policy saknas. Det betyder att appen saknar detta säkerhetsskydd.|
| 4 | ZAP |Missing Anti-clickjacking Header  | Medium, Confidence: Medium | http://localhost:5080/ | Verkligt | Jag kontrollerade Response i ZAP och såg att X-Frame-Options saknas. Det betyder att appen saknar skydd mot clickjacking. |
| 5 |ZAP|Absence of Anti-CSRF Tokens | Medium, Confidence: Low|  | http://localhost:5080/tickets/6 | Verkligt |Jag kontrollerade formuläret i TicketDetail.razor och såg att det skickar POST-data utan någon Anti-CSRF-token.

Bevis (skärmbilder eller utdrag), numrerade efter fyndet ovan:

*Klistra in här, eller hänvisa till bilagor.*

Fynd 1: Se bilaga Fynd1-CodeQL-XML-Fore.png
Fynd 2: Se bilaga Fynd2-CodeQL-CommandInjection-Fore.png
Fynd 3: Se bilaga Fynd3-ZAP-CSP-Fore.png
Fynd 4: Se bilaga Fynd4-ZAP-AntiClickjacking-Fore.png
Fynd 5: Se bilaga Fynd5-ZAP-AntiCSRF-Fore.png
---

## 3. Prioritering

Rangordna fynden och motivera ordningen med allvarlighetsgrad, exponering och utnyttjbarhet. Vilket tar du först och varför?

*Skriv här.*

Jag tar Uncontrolled command line först. Det är Critical och användarens text kan hamna i ett kommando som körs på servern. En angripare kan därför försöka få servern att göra något annat än det som var tänkt. Därför bedömer jag att risken är hög.

Jag prioriterar Untrusted XML som nummer två eftersom det är Critical. En användare kan skicka XML-data till appen och XML-läsaren är inställd så att den kan behandla osäkra instruktioner i XML:en.

Jag prioriterar Absence of Anti-CSRF Tokens som nummer tre. Formuläret för kommentarer saknar CSRF-token. Det betyder att SakerLabb har sämre skydd mot att någon försöker skicka en falsk kommentar via en användares webbläsare.

Jag prioriterar CSP Header Not Set som nummer fyra. CSP saknas i appens Response, vilket betyder att webbläsaren saknar ett extra skydd som begränsar vilket innehåll som får köras. Jag prioriterar det efter de tidigare fynden eftersom det är Medium och främst är ett extra skydd.

Jag prioriterar Missing Anti-clickjacking Header som nummer fem. Appen saknar ett skydd mot att SakerLabb visas inuti en annan webbplats och användaren luras att klicka på något. Jag placerar fyndet sist eftersom det är Medium och risken är mer begränsad än för de andra fynden.

## 4. Åtgärder (minst tre)

Använd mönstret nedan per åtgärdat fynd. Varje åtgärd ska gå att spåra tillbaka till ett fynd i tabellen ovan, och beviset efter ska vara en **ny körning av verktyget**, inte din egen kod.

### Åtgärd 1

```
Fynd:        (nr och regel-id/alert från tabellen ovan)
Plats:       (fil och rad, eller URL)
Bevis före:  (skärmbild eller rapportutdrag som visar fyndet)
Bedömning:   (verkligt eller falskt positivt, kort motiverat)
Åtgärd:      (vad du ändrade, med commit-hash)
Bevis efter: (ny körning: CodeQL-alerten står som Fixed, eller ZAP-larmet är borta ur den nya rapporten)
```

### Åtgärd 2

```
Fynd:
Plats:
Bevis före:
Bedömning:
Åtgärd:
Bevis efter:
```

### Åtgärd 3

```
Fynd:
Plats:
Bevis före:
Bedömning:
Åtgärd:
Bevis efter:
```

---

## 5. Eventuella bortval

Om du valt att inte åtgärda ett fynd, skriv ned tre saker per bortval: risken, motivet och den kompenserande kontrollen. Sätt gärna ett datum för omprövning.

*Skriv här, eller skriv "inga bortval".*
