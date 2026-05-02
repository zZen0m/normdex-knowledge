# Verwaltungsportal-Ausbau

## Zielbild

Das Verwaltungsportal wird zu einer organisationszentrierten Kundenakte ausgebaut. Betreiber starten Support- und Verwaltungsarbeit bei einer Organisation und sehen dort Stammdaten, Mitglieder, Nutzerkontext, Projekte, Lizenzen, Billing-/Stripe-Kontext, Support-Tickets, Timeline und Audit-Historie an einer Stelle.

Das bestehende Ticket-Portal bleibt der operative Support-Posteingang. In der Verwaltung werden Tickets nur als Kontext angezeigt und in das Ticket-Portal verlinkt. Antworten, Inbox-Queue, Status-Workflows, Priorisierung und interne Notizen bleiben im Ticket-Portal.

## Rollen und Berechtigungen

- V1 nutzt weiterhin das bestehende `is_admin`-Flag.
- RBAC wird vorbereitet, aber nicht in T020-13 umgesetzt.
- Spätere Rollen können `Owner`, `Manager` und `Support-Agent` unterscheiden.
- Zahlungs-, Datenschutz- und Projekt-Support-Aktionen werden nicht als freie Datenbankbearbeitung geplant, sondern als geführte Admin-Aktionen.

## Organisationsakte

### Use Cases

- Betreiber findet eine Organisation über Name, Kundennummer, E-Mail, Ticket-ID, Projektname, Stripe Customer ID oder Subscription ID.
- Betreiber öffnet eine Kundenakte und sieht alle relevanten Support-Informationen in einer Arbeitsfläche.
- Betreiber erkennt, welche Benutzer:innen zur Organisation gehören und welche Rolle sie haben.
- Betreiber sieht Projekte und kann bei Supportfällen in bestehende Projektdetails wechseln.
- Betreiber sieht Lizenz- und Billing-Kontext, ohne in Stripe oder die Datenbank wechseln zu müssen.

### Betroffene Endpunkte

- Bestehend: `GET /admin/organizations`, `GET /admin/organizations/{org_id}`, `PATCH /admin/organizations/{org_id}`.
- Neu für T020-13: `GET /admin/organizations/{org_id}/case`.
- Später: globale Suche als eigener Endpunkt, z. B. `GET /admin/search`.

### Datenmodell-Auswirkungen

V1 benötigt keine Migration. Die Organisationsakte aggregiert bestehende Modelle:

- `Organization`
- `Membership`
- `User`
- `Project`
- `License`
- `LicenseOrder`
- `LicenseEvent`
- `SupportTicket`
- `SupportEvent`
- `AuditLog`

### UI-Anforderungen

- Einstieg über Organisationen.
- Detailansicht als kompakte Betreiber-Arbeitsfläche.
- Sektionen oder Tabs: Übersicht, Mitglieder, Projekte, Lizenzen, Support-Tickets, Timeline.
- Stripe-Referenzen als kopierbare IDs oder direkte externe Referenzen anzeigen, aber keine mutierenden Stripe-Aktionen in V1.

## Ticket-Kontext

### Use Cases

- Betreiber sieht offene Tickets der Organisation priorisiert oben.
- Betreiber sieht gelöste und geschlossene Tickets über Filter oder separate Liste.
- Betreiber öffnet ein Ticket im bestehenden Ticket-Portal.
- Betreiber nutzt Tickets als Kontext für spätere Admin-Aktionen.

### Zuordnung

Tickets werden über vorhandene Felder zugeordnet:

- `SupportTicket.user_id` zu Organisationsmitgliedern.
- `SupportTicket.requester_email` zu E-Mails der Organisationsmitglieder.
- `SupportTicket.customer_no` zu `Organization.customer_number`.
- Optionaler Fallback über `company_name` bleibt nur unterstützend, nicht führend.

### Abgrenzung

Keine Ticket-Antworten, keine Inbox-Queue und keine Statusbearbeitung in der Organisationsakte. Die Akte zeigt Ticket-ID, Status, Priorität, Betreff, letztes Update und Link zum Ticket-Portal.

## Kunden- und Organisations-Timeline

### Use Cases

- Betreiber erkennt chronologisch, was beim Kunden passiert ist.
- Betreiber sieht Admin-Aktionen, Lizenzereignisse, Ticketereignisse, Projektänderungen und Systemkontext gebündelt.
- Betreiber kann spätere kritische Aktionen mit bestehender Historie nachvollziehen.

### Quellen

- `AuditLog` für Admin- und Systemaktionen.
- `LicenseEvent` für Lizenzlebenszyklus.
- `SupportTicket` und `SupportEvent` für Ticketkontext.
- `Project.updated_at` für Projektänderungen.
- Stripe-/Billing-Ereignisse nur soweit lokal vorhanden oder über Stripe lesend abrufbar.

## Admin-Statistikbereich

Der Statistikbereich wird als eigener Admin-Bereich geplant, getrennt vom Kunden-Dashboard und vom externen `/stats` Monitoring.

V1-KPIs:

- Organisationen gesamt.
- Neukunden der letzten 7 und 30 Tage.
- Lizenzen nach Status: aktiv, testend, gekündigt bzw. auslaufend, beendet, fehlgeschlagen.
- Monatlicher Lizenzumsatz auf Basis von Stripe.
- Offene und neue Tickets.
- Projekte gesamt und aktive Projekte.

Für Umsatzmetriken ist Stripe die führende Quelle. Lokale Lizenzdaten bleiben ergänzend für Counts, Statusanzeigen und Verknüpfung zur Organisation.

## Billing und Stripe

### Grundsatz

Stripe bleibt führende Quelle für reale Subscription-, Umsatz-, Refund- und Coupon-Zustände. Lokale Daten bilden die Verbindung zur Organisation, die Lizenzanzeige und den letzten bekannten Status.

### V1 Organisationsakte

- Stripe Customer ID anzeigen.
- Subscription IDs aus lokalen Lizenzen anzeigen.
- Stripe Price IDs und Subscription Item IDs anzeigen, sofern vorhanden.
- Lizenzstatus, Laufzeit, Testzeitraum, Kündigungsstatus und lokale Order-Daten anzeigen.
- Keine Stripe-Mutationen.

### Spätere Aktionen

- Lizenz kündigen, pausieren oder reaktivieren.
- Refund ausführen.
- Coupon oder Promotion Code an Subscription binden.
- Billing-Adresse mit Stripe Customer synchronisieren.

Diese Aktionen werden separat geplant und müssen jeweils Vorschau, Pflichtgrund, Bestätigung und Audit-Log haben.

## Kontrollierte Admin-Aktionen

Kritische Aktionen werden als geführte Workflows umgesetzt:

- Vorschau der Auswirkung.
- Pflichtfeld für Grund oder Kundenwunsch.
- Optionale Verknüpfung mit bestehendem Support-Ticket.
- Explizite Bestätigung.
- Audit-Log mit Admin, Zielobjekt, Aktion, Zeitpunkt, Grund und relevanten Vorher-/Nachher-Metadaten.

Betroffene spätere Aktionen:

- Stammdaten auf Kundenwunsch ändern.
- Lizenzen kündigen, pausieren, reaktivieren oder neu starten.
- Refunds.
- Coupons und Rabatte.
- DSGVO-Datenlöschung.
- Projekt-Supportzugriff und Projektänderungen.

## DSGVO und Datenexport

V1 bleibt lesend. Spätere Umsetzung:

- Export aller Kundendaten pro Organisation.
- DSGVO-Wipe mit Vorschau, Bestätigung, Pflichtgrund und Audit.
- Klare Unterscheidung zwischen Single-User-Delete und vollständiger Organisationslöschung.
- Schutz vor versehentlichem Löschen der letzten Admin- oder Owner-Kontexte.

## Projekt-Supportzugriff

V1 zeigt Projekte in der Organisationsakte an und verlinkt in bestehende Projektkontexte. Schreibender Supportzugriff auf fremde Projekte kommt später.

Spätere Anforderungen:

- Admin öffnet Kundenprojekt im Supportmodus.
- Schreibzugriff nur mit Grund und optionalem Ticket.
- Jede Änderung wird mit Admin-ID, Projekt-ID und Ticket-Kontext protokolliert.

## Modul-Priorisierung

1. T020-13 Organisationsakte mit globaler Einstiegsmöglichkeit, aggregierter Detailansicht und Timeline.
2. T020-14 Ticket-Kontext-Verknüpfung mit Filterung und Ticket-Portal-Links.
3. T020-15 Admin-Statistikbereich.
4. T020-16 Lizenz- und Billing-Support-Aktionen.
5. T020-17 Refunds.
6. T020-18 Coupons und Rabatte.
7. T020-19 DSGVO-Wipe und Datenexport.
8. T020-20 Projekt-Supportzugriff mit Protokollierung.
9. T020-21 RBAC-Konzept und spätere Rollenmigration.

## Akzeptanzkriterien-Abdeckung

- Pro Modul sind Use Cases, Endpunkte, Datenmodell-Auswirkungen und UI-Anforderungen beschrieben.
- Die Organisationsakte ist als zentrale Admin-Ansicht definiert.
- Das bestehende Ticket-Portal bleibt operativ führend.
- Ticket-Übersicht pro Organisation ist geplant.
- Timeline mit Admin-, Lizenz-, Ticket-, Projekt- und Billing-Kontext ist geplant.
- Billing-Support-Kontext nutzt Stripe als führende Quelle.
- Admin-KPIs sind definiert.
- Kritische Admin-Aktionen sind mit Grund, Bestätigung, Ticket-Kontext und Audit geplant.
- RBAC ist für später skizziert; V1 nutzt weiter `is_admin`.
