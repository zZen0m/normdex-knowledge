# T024 · Account-Einrichtung – Nutzer zur vollständigen Einrichtung führen

**Priorität:** P2 · App / UX / Onboarding / Notifications  
**Status:** offen  
**Datum:** 2026-06-01  
**Referenz:** [[T022-notifications-system]]

## Ziel

Neue und unvollständig eingerichtete Nutzer sollen aktiv und freundlich dazu geführt werden, ihren Account vollständig einzurichten. Drei Schritte stehen im Fokus:

1. **E-Mail-Adresse verifizieren** – Bestätigung der Registrierungs-E-Mail.
2. **Profilbild hochladen** – persönliches Avatar für Profil, Team-Ansicht und Aktivitätsverlauf.
3. **Firmenlogo hochladen (nur Administratoren)** – damit das Logo im PDF-Bericht (Wirtschaftlichkeitsbericht) erscheint.

Ziel ist eine erkennbare, nicht aufdringliche Führung, die den Einrichtungsfortschritt sichtbar macht und nach Abschluss verschwindet.

## Kontext

- Die Schritte sind unterschiedlich relevant je nach Rolle: Schritt 3 betrifft ausschließlich Administratoren der Organisation.
- Das Firmenlogo hat einen konkreten geschäftlichen Effekt: Ohne Logo erscheint der PDF-Bericht generisch/unmarkiert. Dieser Nutzen soll im Hinweis klar kommuniziert werden („Ihr Logo erscheint dann im PDF-Bericht").
- Ein Notifications-System (Bell-Icon, Badge, Panel, Backend-API) existiert bereits aus [[T022-notifications-system]] und kann für die Sidebar-Variante genutzt werden.
- Vor Umsetzung ist der Ist-Zustand im Code zu prüfen: Gibt es bereits E-Mail-Verifizierung, Avatar-Upload und Logo-Upload als Funktionen, oder müssen Teile davon erst geschaffen werden? Das Todo beschreibt die Nutzerführung; fehlende Basis-Funktionen sind ggf. als Voraussetzung mitzudenken.

## Umfang

### Einrichtungsstatus ermitteln

- Pro Nutzer bestimmen, welche der drei Schritte noch offen sind:
  - E-Mail verifiziert? (Flag am User)
  - Profilbild vorhanden?
  - Bei Admins: Firmenlogo der Organisation vorhanden?
- Schritt 3 nur für Administratoren auswerten; Nicht-Admins sehen nur Schritt 1 und 2.
- Einen zusammengefassten Einrichtungsfortschritt bereitstellen (z. B. „2 von 3 erledigt"), den Frontend-Komponenten abfragen können.

### Nutzerführung (mindestens eine, idealerweise kombinierte Variante)

**Variante A – Hinweiskasten im Dashboard (empfohlen als Primärlösung)**

- Gut sichtbarer, aber dezenter Einrichtungs-/Onboarding-Block oben im Dashboard.
- Checklisten-Darstellung der offenen Schritte mit Fortschritt (z. B. Progress-Bar oder „2/3").
- Jeder offene Schritt hat einen direkten Call-to-Action, der zur passenden Stelle führt:
  - „E-Mail verifizieren" → Verifizierungs-Mail erneut senden / Hinweis öffnen.
  - „Profilbild hochladen" → Profil-/Einstellungsseite mit Avatar-Upload.
  - „Firmenlogo hochladen" → Organisations-/Einstellungsseite mit Logo-Upload.
- Erledigte Schritte werden abgehakt dargestellt.
- Ist alles erledigt, verschwindet der Block vollständig (kein Dauer-Rauschen).
- Optional: pro Nutzer ausblendbar/„später erinnern", ohne dass offene Schritte dauerhaft verloren gehen.

**Variante B – Sidebar-Notifications (ergänzend)**

- Über das bestehende Notifications-System ([[T022-notifications-system]]) je offenem Schritt eine Benachrichtigung erzeugen, die zur jeweiligen Aktion verlinkt.
- Beim Abschließen eines Schritts wird die zugehörige Notification automatisch gelesen/entfernt.
- Keine Duplikate erzeugen (idempotent pro Nutzer und Schritt).

**Weitere gängige Methoden (zur Abwägung, optional)**

- Persistenter, schließbarer Banner unterhalb der Topbar, bis alles erledigt ist.
- Geführter Setup-Wizard direkt nach der ersten Anmeldung (Empty-State-Onboarding).
- Badge/Indikator am Profil-/Einstellungs-Menüpunkt, solange Schritte offen sind.

### Inhalt und Ton

- Freundlich, kurz, deutsch mit echten Umlauten.
- Jeweils klar machen, warum der Schritt nützlich ist (besonders Firmenlogo → PDF-Bericht).
- Keine blockierende Pflicht: Die App bleibt nutzbar, die Hinweise sind Führung, kein Zwang.

## Akzeptanzkriterien

- [ ] Der Einrichtungsstatus (E-Mail verifiziert, Profilbild, Firmenlogo) ist pro Nutzer/Organisation rollenkorrekt ermittelbar.
- [ ] Nicht-Admins sehen ausschließlich die Schritte „E-Mail verifizieren" und „Profilbild hochladen".
- [ ] Administratoren sehen zusätzlich „Firmenlogo hochladen" mit dem Hinweis, dass das Logo im PDF-Bericht erscheint.
- [ ] Im Dashboard erscheint ein Einrichtungs-Hinweiskasten mit offenen Schritten, Fortschritt und direkten Call-to-Actions.
- [ ] Jeder Call-to-Action führt zielgenau zur passenden Aktion (E-Mail-Verifizierung, Avatar-Upload, Logo-Upload).
- [ ] Erledigte Schritte werden als erledigt dargestellt; sind alle Schritte erledigt, verschwindet der Hinweis vollständig.
- [ ] Optional umgesetzte Sidebar-Notifications sind idempotent und werden beim Abschluss des jeweiligen Schritts automatisch aufgelöst.
- [ ] Texte sind deutsch mit echten Umlauten und kommunizieren den Nutzen jedes Schritts.
- [ ] Nach Hochladen eines Firmenlogos erscheint dieses tatsächlich im generierten PDF-Bericht.

## Offene Fragen

- Existieren E-Mail-Verifizierung, Avatar-Upload und Firmenlogo-Upload bereits vollständig, oder sind Teile davon noch zu bauen? (Vor Umsetzung im Code prüfen.)
- Soll der Hinweiskasten manuell dauerhaft ausblendbar sein, oder nur bis zum nächsten offenen Schritt?
- Reicht Variante A (Dashboard-Hinweiskasten), oder werden A und B (Sidebar-Notifications) kombiniert gewünscht?

## Notizen / Fortschritt

- 2026-06-01: Todo angelegt aus dem Wunsch, Nutzer aktiv zur vollständigen Account-Einrichtung (E-Mail-Verifizierung, Profilbild, Firmenlogo für Admins) zu führen. Bevorzugte Primärlösung: Dashboard-Hinweiskasten mit Checkliste; Sidebar-Notifications als Ergänzung über das bestehende [[T022-notifications-system]].
