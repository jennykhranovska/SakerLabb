# Labbrapport: praktisk laboration

*Kunskapskontroll 2, IT-säkerhet för utvecklare. Fyll i mallen och lämna in som PDF tillsammans med länken till ditt repo. Riktlängd två till tre sidor.*

**Namn: Jenny Khranovska 
**Datum:2026-08-21
**Repo (länk till din fork): https://github.com/jennykhranovska/SakerLabb
**Applikation som analyserades:** SakerLabb Support

---

## 1. Kort om applikationen och analysen

Beskriv i några meningar vilken app du analyserade, vad den gör och hur du genomförde analysen. Ange vilka verktyg du använde och hur du körde dem (CodeQL default setup med språk C#, ZAP passiv och aktiv skanning mot vilken adress).

Jag analyserade SakerLabb Support, en .NET-applikation för hantering av supportärenden. Jag använde CodeQL med default setup och språket C# för statisk analys av koden. För dynamisk analys använde jag OWASP ZAP och gjorde en passiv skanning genom att köra applikationen lokalt på http://localhost:5080 och surfa i den genom ZAP proxy. Jag använde sedan samma verktyg igen för att verifiera de åtgärder jag gjort.

---

## 2. Fem fynd

Fyll i tabellen. Minst ett fynd ska komma från statisk analys (CodeQL) och minst ett från dynamisk analys (ZAP). Spara bevis i form av skärmbild eller rapportutdrag och hänvisa till det per fynd.

| Nr | Källa (CodeQL/ZAP) | Regel-id eller alert | Allvarlighet (+ confidence för ZAP) | Fil och rad eller URL | Verkligt eller falskt positivt | Motivering (2–4 meningar) |


| 1 | CodeQL | cs/command-line-injection |Critical  | SakerLabb.Web/Services/ImportService.cs, rad 57 |  Verkligt|  |Användaren kan skriva in ett värde som sparas i host. Appen använder sedan värdet direkt i ett kommando utan att kontrollera det först.  

| 2 | CodeQL|  cs/xml/insecure-dtd-handling| Critical | SakerLabb.Web/Services/ImportService.cs, rad 27 | Verkligt | Fyndet är verkligt eftersom appen tar emot XML från användaren och läser den utan tillräckligt skydd. XML-läsaren har inställningar som kan göra det möjligt att läsa osäkert innehåll.

| 3 | ZAP | Content Security Policy (CSP) Header Not Set |Medium, Confidence: High  | http://localhost:5080/ | Verkligt | Fyndet är verkligt eftersom jag kontrollerade Response i ZAP och såg att Content-Security-Policy saknas. Det betyder att appen saknar detta säkerhetsskydd.
| 4 | ZAP |Missing Anti-clickjacking Header  | Medium, Confidence: Medium | http://localhost:5080/ | Verkligt | Jag kontrollerade Response i ZAP och såg att X-Frame-Options saknas. Det betyder att appen saknar skydd mot clickjacking. 
| 5 |ZAP|Absence of Anti-CSRF Tokens | Medium, Confidence: Low|  | http://localhost:5080/tickets/6 | Verkligt |Jag kontrollerade formuläret i TicketDetail.razor och såg att det skickar POST-data utan någon Anti-CSRF-token.

Bevis (skärmbilder eller utdrag), numrerade efter fyndet ovan:

Fynd 1: Se bilaga `bilder/CodeQL_CommandInjection_Critical_Fore.png`
Fynd 1 efter: Se bilaga `bilder/CodeQL_CommandInjection_Efter.png`

Fynd 2: Se bilaga `bilder/CodeQL_UntrustedXML_Critical_Fore.png`
Fynd 2 efter: Se bilaga `bilder/CodeQL_UntrustedXML_Efter.png`

Fynd 3: Se bilaga `bilder/ZAP-CSP-Passive-Fore.png`

Fynd 4: Se bilaga `bilder/ZAP-AntiClickjacking-Passive-Fore.png`
Fynd 4 efter: Se bilaga `bilder/ZAP-AntiClickjacking-Efter.png`

Fynd 5: Se bilaga `bilder/ZAP-AntiCSRF-Passive-Fore.png`





## 3. Prioritering

Rangordna fynden och motivera ordningen med allvarlighetsgrad, exponering och utnyttjbarhet. Vilket tar du först och varför?


Jag tar Uncontrolled command line först. Det är Critical och användarens text kan hamna i ett kommando som körs på servern. En angripare kan därför försöka få servern att göra något annat än det som var tänkt. Därför bedömer jag att risken är hög.

Jag prioriterar Untrusted XML som nummer två eftersom det är Critical. En användare kan skicka XML-data till appen och XML-läsaren är inställd så att den kan behandla osäkra instruktioner i XML:en.

Jag prioriterar Absence of Anti-CSRF Tokens som nummer tre. Formuläret för kommentarer saknar CSRF-token. Det betyder att SakerLabb har sämre skydd mot att någon försöker skicka en falsk kommentar via en användares webbläsare.

Jag prioriterar CSP Header Not Set som nummer fyra. CSP saknas i appens Response, vilket betyder att webbläsaren saknar ett extra skydd som begränsar vilket innehåll som får köras. Jag prioriterar det efter de tidigare fynden eftersom det är Medium och främst är ett extra skydd.

Jag prioriterar Missing Anti-clickjacking Header som nummer fem. Appen saknar ett skydd mot att SakerLabb visas inuti en annan webbplats och användaren luras att klicka på något. Jag placerar fyndet sist eftersom det är Medium och risken är mer begränsad än för de andra fynden.

## 4. Åtgärder (minst tre)

Använd mönstret nedan per åtgärdat fynd. Varje åtgärd ska gå att spåra tillbaka till ett fynd i tabellen ovan, och beviset efter ska vara en **ny körning av verktyget**, inte din egen kod.


### Åtgärd 1

Fynd: Fynd 1 – cs/command-line-injection

Plats: SakerLabb.Web/Services/ImportService.cs, rad 57

Bevis före: Skärmbild från CodeQL på main som visar fyndet Uncontrolled command line med allvarlighetsgrad Critical.

Bedömning: Verkligt. Användaren kan ange ett värde som används som host. Värdet användes direkt för att bygga ett kommando, vilket gjorde command injection möjlig.

Åtgärd: Ändrade Ping så att användarens indata inte längre sätts ihop till ett kommando som körs via cmd.exe.

Commit: 9e68c04 – "Åtgärda command line injection"

Bevis efter: Ny CodeQL-körning genomfördes efter rättningen på branchen jenny-sakerhetsanalys. Det tidigare fyndet cs/command-line-injection rapporterades inte på den åtgärdade branchen. Den ursprungliga alerten #14 ligger fortfarande kvar som Open på main eftersom rättningen gjordes på min separata arbetsbranch.

### Åtgärd 2

Fynd: Fynd 2 – cs/xml/insecure-dtd-handling

Plats: SakerLabb.Web/Services/ImportService.cs, rad 27

Bevis före: Skärmbild från CodeQL på main som visar fyndet Untrusted XML is read insecurely med allvarlighetsgrad Critical.

Bedömning: Verkligt. Applikationen tar emot XML från användaren och XML-läsaren tillät osäker DTD-hantering. Det kunde göra att osäkert externt XML-innehåll behandlades.

Åtgärd: Ändrade XML-hanteringen så att DTD inte längre behandlas osäkert och extern XML-resolver inte används.

Commit: 88eeef3 – "Åtgärda osäker XML-hantering"

Bevis efter: Ny CodeQL-körning genomfördes efter rättningen på branchen jenny-sakerhetsanalys. Det tidigare fyndet cs/xml/insecure-dtd-handling rapporterades inte på den åtgärdade branchen. Den ursprungliga alerten finns fortfarande kvar på main eftersom rättningen gjordes på min egen branch.


### Åtgärd 3

Fynd:        Fynd 4 – Missing Anti-clickjacking Header

Plats:       http://localhost:5080/login

Bevis före:  Skärmbild från ZAP som visar fyndet.

Bedömning:   Verkligt. Appen saknade X-Frame-Options och hade därför inget skydd mot clickjacking.

Åtgärd:      Lade till X-Frame-Options med värdet DENY i Program.cs.
Commit: 8c08f1e – "Lägg till skydd mot clickjacking"

Bevis efter: Ny körning i ZAP visar att X-Frame-Options: DENY skickas i svaret.

---

## 5. Eventuella bortval

Om du valt att inte åtgärda ett fynd, skriv ned tre saker per bortval: risken, motivet och den kompenserande kontrollen. Sätt gärna ett datum för omprövning.

Fynd 3 – Content Security Policy (CSP) Header Not Set
Risk: Utan CSP finns ett sämre skydd mot exempelvis skadligt innehåll som körs i webbläsaren.
Motiv: Jag valde att prioritera andra fynd som var enklare att åtgärda och verifiera tydligt i laborationen.
Kompenserande kontroll: Anti-clickjacking-skydd har lagts till med X-Frame-Options: DENY.

Fynd 5 – Absence of Anti-CSRF Tokens
Risk: En angripare kan försöka få en inloggad användare att skicka en oönskad begäran till applikationen.
Motiv: Jag försökte åtgärda fyndet genom att lägga till CSRF-skydd. Projektet byggde utan fel, men åtgärden kunde inte verifieras ordentligt med ZAP eftersom den lokala databasen saknade tickets att testa mot. Därför valde jag att inte använda fyndet som en av de tre verifierade åtgärderna.
Kompenserande kontroll: Åtgärden med antiforgery-skydd finns i koden, men fyndet bör testas och verifieras igen när det finns testdata.
