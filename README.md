Purple Team Protokoll – DOM-based XSS (jQuery href sink / location.search source)
Lab: DOM XSS in jQuery anchor href attribute sink using location.search source
Plattform: PortSwigger Web Security Academy
Kategorie: Client-side Injection (DOM-based XSS)
Status: erfolgreich reproduziert und gelöst

1. Zielsetzung
Ziel des Tests war der Nachweis einer DOM-basierten Cross-Site-Scripting-Schwachstelle auf der „Submit feedback“-Funktion. Dabei sollte überprüft werden, ob Benutzereingaben aus URL-Parametern ungefiltert in DOM-Attribute übernommen werden und dadurch JavaScript-Ausführung ermöglicht wird.

2. Technische Ausgangslage
Die Anwendung verwendet:
    • location.search als Eingabequelle 
    • jQuery $() zur Manipulation des DOM 
    • ein <a>-Element, dessen href-Attribut dynamisch gesetzt wird 
Problematisch ist dabei die direkte Verarbeitung von URL-Parametern ohne Validierung oder Encoding.

3. Schwachstellenanalyse
Die Angriffsstruktur lässt sich wie folgt beschreiben:
User Input (URL Query Parameter returnPath)
→ location.search (DOM Source)
→ jQuery DOM Manipulation
→ href Attribut eines Anchor-Tags (Sink)
→ Ausführung im Browser-Kontext
Die Schwachstelle entsteht durch fehlende Kontrolle über das Zielprotokoll im href-Attribut.

4. Reproduktionsschritte
    1. Aufruf der „Submit feedback“-Seite 
    2. Manipulation des URL-Parameters returnPath 
    3. Beobachtung im DOM: der Wert wird direkt in das href-Attribut übernommen 
    4. Ersetzen des Parameters durch einen JavaScript-URI 
Payload:
javascript:alert(document.cookie)
    5. Klick auf den „Back“-Link 
    6. Der Browser interpretiert den javascript:-URI und führt den Code im Kontext der Seite aus 

5. Ergebnis
Die Payload wird erfolgreich ausgeführt und zeigt den Inhalt von document.cookie an. Damit ist die DOM-based XSS Schwachstelle bestätigt.

6. Risikoanalyse
Auswirkungen:
    • Möglichkeit zur Ausführung von JavaScript im Kontext der Anwendung 
    • potenzieller Zugriff auf Session-Daten (abhängig von Cookie-Flags wie HttpOnly) 
    • Grundlage für Session Hijacking oder Phishing-Angriffe 
Bewertung:
    • Schweregrad: hoch 
    • Angriffsklasse: DOM-based XSS 
    • Voraussetzung: User Interaction (Klick auf manipulierten Link) 

7. Ursachenanalyse
Hauptursachen:
    • ungefilterte Übernahme von location.search 
    • direkte DOM-Manipulation mit jQuery 
    • keine Validierung erlaubter URL-Schemata im href-Attribut 
    • fehlende Trennung zwischen Daten und Steuerinformationen (URI-Schema) 

8. Gegenmaßnahmen
Empfohlene Fixes:
    1. Validierung und Whitelist von returnPath 
        ◦ nur interne Pfade zulassen (z. B. /home, /feedback) 
    2. Blockieren gefährlicher URI-Schemata 
        ◦ insbesondere javascript: und ähnliche Protokolle 
    3. Sichere DOM-Manipulation 
        ◦ keine direkte String-Zuweisung an href 
        ◦ stattdessen URL-Parsing oder sichere Router-Logik 
    4. Encoding 
        ◦ URL-Encoding für dynamische Parameter 
        ◦ HTML-Encoding bei DOM-Injection 
    5. Sicherheitsheader (ergänzend) 
        ◦ Content Security Policy (CSP) zur Einschränkung von inline JavaScript 

9. Zusammenfassung
Die Schwachstelle ist ein klassischer DOM-based XSS Fall, ausgelöst durch unsichere Verarbeitung von URL-Parametern in Kombination mit direkter DOM-Manipulation via jQuery. Durch die Kontrolle des href-Attributes konnte JavaScript im Kontext der Seite ausgeführt werden.
Die Ursache liegt primär in fehlender Input-Validierung und unsicherer Verarbeitung von location.search.
