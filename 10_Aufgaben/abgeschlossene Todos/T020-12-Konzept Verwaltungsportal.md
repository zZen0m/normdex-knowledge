# T020-12 · Konzept Verwaltungsportal-Ausbau

**Phase:** 4 (Konzept-Phase)  
**Priorität:** P3 · vorgelagert vor Implementierung  
**Parent:** [[T020-allgemeine Todos]]
**Status:** erledigt  
**Abgeschlossen:** 2026-05-30

## Beschreibung
Vor der Implementierung des erweiterten Verwaltungsportals wird ein Konzept-Dokument erstellt, das die fehlenden Funktionen detailliert definiert. Erst nach Konzept-Freigabe entstehen Implementierungs-To-dos (T020-13 ff.).

Das Verwaltungsportal soll als **organisationszentrierte Kundenakte mit kontrollierten Eingriffen** konzipiert werden. Zentrale Einheit ist eine Organisation bzw. ein Kunde, damit Betreiber Kundendaten, Benutzer:innen, Lizenzen, Projekte, Billing-/Stripe-Kontext, Audit-Historie und relevante Tickets an einer Stelle einsehen können.

Wichtig: Das bestehende Ticket-Portal bleibt der operative Support-Posteingang für Ticketbearbeitung, Antworten, Status, Priorität, interne Notizen und laufende Queue-Arbeit. Die neue Seite **Verwaltung** soll dieses Ticket-Portal nicht nachbauen, sondern Ticket-Kontext einbetten und per Link in das Ticket-Portal führen.

## Output
Konzept-Dokument unter `02_knowledge/normdex-vault/01_Produkt/` (oder `04_Verwaltung/`) — Anforderungsdokument inkl. Use Cases, Datenmodell-Auswirkungen, UI-Skizzen-Anforderungen, Modul-Priorisierung und Folge-To-dos.

## Leitidee
- Admins starten Verwaltungs- und Support-Kontextarbeit aus einer Organisationsakte.
- Die Verwaltung startet **Organisation zuerst** und führt von dort zu Nutzer:innen, Lizenzen, Projekten, Tickets, Billing und Audit.
- Tickets werden in der Verwaltung nur als Kontext angezeigt; Antworten, Inbox-Queue und laufende Ticketbearbeitung bleiben im bestehenden Ticket-Portal.
- Kritische Aktionen werden nicht als freie Datenbankbearbeitung umgesetzt, sondern als geführte Admin-Aktionen.
- Jede riskante Aktion braucht Vorschau, Pflicht-Begründung, explizite Bestätigung und Audit-Logging.
- Wenn eine Aktion aus einem Kundenanliegen stammt, soll optional ein Support-Ticket als Kontext verknüpft werden.

## Zu konzipierende Module
1. **Organisationsakte** — zentrale Detailansicht mit Stammdaten, Mitgliedern, Rollen, Kundennummer, Rechnungs-/Billingdaten, Stripe-Verknüpfungen, Lizenzen, Projekten, Audit-Log und Support-Ticket-Kontext.
2. **Nutzerkontext innerhalb der Organisationsakte** — Profil, Accountstatus, Verifizierung, Login-/Security-Hinweise und Zugehörigkeit zur Organisation.
3. **Ticket-Portal-Verknüpfung** — Anzeige aller offenen und bisherigen Tickets eines Users bzw. einer Organisation; direkte Navigation ins Ticket-Detail; optionale Ticket-Verknüpfung bei Admin-Aktionen; keine Antwort-/Inbox-Duplizierung.
4. **Kunden-/Org-Timeline** — chronologische Sicht auf Admin-Aktionen, Lizenzereignisse, Stripe-/Billing-Ereignisse, Ticket-Verweise, Projektänderungen, Systemfehler und relevante Audit-Einträge.
5. **Admin-Statistikbereich** — KPI-Übersicht für Betreiber: Kunden, Neukunden, Lizenzen, monatlicher Lizenzumsatz, Tickets und Projektaktivität.
6. **Daten auf Kundenwunsch ändern** — Welche Felder, wer darf was, Kontrollrahmen, Ticket-Kontext und Audit-Logging.
7. **Lizenz-Operationen aus Admin-Sicht** — kündigen, pausieren, neu starten, Lizenzdetails-Seite, Auswirkungen auf Stripe und lokale Lizenzdaten.
8. **Rabatte / Coupons vergeben** — Stripe-Coupon-Lifecycle aus Admin-UI, an konkrete Subscription binden.
9. **DSGVO-Datenlöschung** — Wipe-Endpoint mit Bestätigung; aktuell nur Single-User-Delete.
10. **Refunds** — UI + Backend-Endpoint zu `stripe.Refund.create()`.
11. **Hilfe bei WBR-Projekten** — Read/Write-Zugriff auf fremde Projekte mit Logging.
12. **RBAC-Erweiterung (optional)** — Owner / Manager / Support-Agent statt Boolean `is_admin`; für v1 kann technisch weiterhin `is_admin` reichen.

## Statistikbereich
Der Statistikbereich soll als eigener Admin-Bereich geplant werden, getrennt vom bestehenden Kunden-Dashboard und vom externen `/stats` Monitoring.

Zu konzipierende Kernmetriken für v1:
- Anzahl Kunden / Organisationen gesamt.
- Anzahl Neukunden der letzten 7 und 30 Tage.
- Anzahl aktiver, testender, gekündigter bzw. auslaufender Lizenzen.
- Monatlicher Lizenzumsatz gesamt auf Basis von Stripe.
- Offene Tickets und neue Tickets.
- Projektanzahl gesamt und aktive Projekte.

Für Umsatzmetriken soll **Stripe die führende Quelle** sein, damit Rabatte, Kündigungen, Pausen, Refunds und reale Subscription-Stände korrekt abbildbar sind. Lokale Lizenzdaten bleiben ergänzende Quelle für Counts, Statusanzeigen und die Verknüpfung zu Organisationen.

## Ticket-Verknüpfung
In der Kunden-/Organisationsakte soll ein Bereich **Support-Tickets** erscheinen.

Anforderungen:
- Offene Tickets werden priorisiert oben angezeigt.
- Gelöste und geschlossene Tickets sind darunter oder über Filter sichtbar.
- Ticket-Zeilen zeigen Status, Priorität, Betreff, letztes Update und Link zum Ticket-Portal.
- Ticket-Antworten, Ticket-Queue, Status-Workflows und operative Priorisierung bleiben im bestehenden Ticket-Portal.
- Admin-Aktionen können optional mit einem bestehenden Ticket verknüpft werden.
- Ticket-Zuordnung erfolgt über vorhandene Beziehungen/Felder wie `user_id`, `requester_email`, `customer_no` und Organisation/Mitgliedschaften.

## Organisationsakte / Verwaltung
Die Verwaltung soll nicht als Sammlung getrennter Admin-Listen wirken, sondern als Arbeitsfläche für eine konkrete Organisation.

Anforderungen:
- Einstieg über Organisationen; Nutzer:innen sind Kontext innerhalb der Organisation.
- Globale Suche über Organisationsname, Kundennummer, E-Mail, Ticket-ID, Projektname, Stripe Customer ID und Subscription ID ist als Erweiterung vorzusehen.
- Organisationsakte zeigt Stammdaten, Mitglieder/Rollen, Nutzerprofile, Accountstatus, Login-/Security-Hinweise, Lizenzen, Projekte, Billing-/Stripe-Kontext, Tickets und Audit.
- Für Billing-Support werden Stripe Customer, Subscription, letzter Zahlungsstatus, Refund-/Coupon-Kontext und direkte Stripe-Referenzen geplant.
- Projekt-Supportzugriff erfolgt kontrolliert, nachvollziehbar und mit Audit-Logging.

## Kunden-/Org-Timeline
Die Organisationsakte soll eine chronologische Timeline enthalten, damit Support und Betreiber nachvollziehen können, was beim Kunden passiert ist.

Zu berücksichtigende Ereignisse:
- Admin-Aktionen und Datenänderungen.
- Lizenzereignisse und Lizenznutzung.
- Stripe-/Billing-Ereignisse, soweit lokal verfügbar oder per Stripe abrufbar.
- Ticket-Erstellung, relevante Ticket-Statuswechsel und Links ins Ticket-Portal.
- Projektänderungen und Supportzugriffe auf fremde Projekte.
- Systemfehler, Webhook-/Graph-/Mail-Fehler und Sync-Probleme mit Kundenbezug.

## Kontrollierte Admin-Aktionen
Kritische Aktionen wie Lizenzänderung, Refund, Rabattvergabe, DSGVO-Löschung oder Datenänderung sollen nicht direkt ohne Kontext ausgeführt werden.

Konzeptanforderungen:
- Vorschau der Auswirkung vor Ausführung.
- Pflichtfeld für Grund / Kundenwunsch.
- Optionale Verknüpfung mit einem Support-Ticket.
- Explizite Bestätigung bei irreversiblen oder zahlungsrelevanten Aktionen.
- Audit-Log mit Admin, Zielobjekt, Aktion, Zeitpunkt, Grund und vorher/nachher-relevanten Metadaten.

## Bestandsaufnahme (bereits vorhanden)
- Kundenliste, Suche, Filter, Bearbeiten — `apps/api/app/routers/admin.py`.
- Org-Verwaltung, Projekt-Liste/-Edit.
- Support-Ticket-System komplett.
- Ticket-Portal für operative Ticketbearbeitung, Antworten, Status/Priorität, interne Notizen und Queue-Arbeit.
- Audit-Log.
- `grant-complimentary` für kostenlose Lizenzen.
- Externe Statistik-Endpunkte unter `/stats/*`.
- Kunden-Dashboard-Statistiken unter `/dashboard/stats`.

## Lücken (zu konzipieren)
- Kunden-/Organisationsakte als zentrale Admin-Detailseite.
- Organisation-first-Verwaltung statt separater Listenlogik.
- Ticket-Übersicht pro User und Organisation als Kontext mit Link ins Ticket-Portal.
- Kunden-/Org-Timeline mit Admin-, Lizenz-, Stripe-/Billing-, Ticket-, Projekt- und Systemereignissen.
- Billing-Support-Kontext mit Stripe Customer, Subscription, Zahlungsstatus, Refund-/Coupon-Bezug und Stripe-Referenzen.
- Admin-Statistikbereich mit Betreiber-KPIs.
- Neuer Admin-Statistik-Endpunkt, getrennt von Kunden-Dashboard und externem Monitoring.
- Lizenz aus Admin kündigen/pausieren/reaktivieren.
- Coupon-Vergabe an existierende Subscription.
- Refund-Endpoint.
- DSGVO-Bulk-Wipe.
- Datenexport (CSV/PDF) aus Admin-UI.

## Akzeptanzkriterien Konzept
- [x] Pro Modul: Use Cases, betroffene Endpoints, Datenmodell-Änderungen, UI-Anforderungen.
- [x] Modul-Priorisierung (welches Modul zuerst).
- [x] Konzept beschreibt die organisationszentrierte Kundenakte als zentrale Admin-Ansicht.
- [x] Konzept grenzt Verwaltung klar vom bestehenden Ticket-Portal ab: keine doppelte Inbox-/Antwortlogik.
- [x] Konzept enthält Ticket-Übersicht pro User und Organisation inklusive Link ins Ticket-Portal.
- [x] Konzept enthält eine Kunden-/Org-Timeline mit Admin-, Lizenz-, Stripe-/Billing-, Ticket-, Projekt- und Systemereignissen.
- [x] Konzept enthält Billing-Support-Kontext mit Stripe als führender Quelle für Subscription-, Umsatz-, Refund- und Coupon-Status.
- [x] Konzept definiert Statistik-KPI-Karten für Kunden, Neukunden, Lizenzen, Umsatz, Tickets und Projekte.
- [x] Konzept legt fest, dass monatlicher Lizenzumsatz aus Stripe abgeleitet wird.
- [x] Kritische Admin-Aktionen sind mit Begründung, Bestätigung, optionalem Ticket-Kontext und Audit-Log geplant.
- [x] Skizze Berechtigungs-Modell (RBAC ja/nein; v1 ggf. weiter `is_admin`).
- [x] Konzept als Markdown im Knowledge-Vault: [[Verwaltungsportal-Ausbau]].

## Folge-To-dos
Nach Freigabe entstehen getrennte Implementierungs-To-dos, z.B.:
- T020-13 Organisationsakte mit globaler Suche und Timeline.
- T020-14 Ticket-Kontext-Verknüpfung im Verwaltungsportal.
- T020-15 Admin-Statistikbereich inkl. Verwaltungs-KPIs.
- T020-16 Lizenz- und Billing-Support-Aktionen aus Admin-Sicht.
- T020-17 Refunds.
- T020-18 Coupons / Rabatte.
- T020-19 DSGVO-Wipe / Datenexport.
- T020-20 Projekt-Supportzugriff mit Protokollierung.
- T020-21 RBAC-Konzept für Verwaltung und Support.

## Abschlussnotiz

Abgeschlossen am 2026-05-30. Das Konzept liegt unter [[Verwaltungsportal-Ausbau]]. Der Billing- und Abo-Support wurde als vertiefendes Konzept unter [[Verwaltungsportal-Billing-und-Abo-Support]] ausgearbeitet; die Umsetzung startet mit [[T020-16-Lizenz-und-Billing-Support-Aktionen]].
