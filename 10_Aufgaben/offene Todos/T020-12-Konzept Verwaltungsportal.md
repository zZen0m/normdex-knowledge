# T020-12 · Konzept Verwaltungsportal-Ausbau

**Phase:** 4 (Konzept-Phase)  
**Priorität:** P3 · vorgelagert vor Implementierung  
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Vor der Implementierung des erweiterten Verwaltungsportals wird ein Konzept-Dokument erstellt, das die fehlenden Funktionen detailliert definiert. Erst nach Konzept-Freigabe entstehen Implementierungs-To-dos (T020-13 ff.).

Das Verwaltungsportal soll als **Support-Cockpit mit kontrollierten Eingriffen** konzipiert werden. Zentrale Einheit ist eine Kunden- bzw. Organisationsakte, damit Betreiber nicht zwischen getrennten Listen, Ticket-Ansichten, Lizenzdaten und Projekten hin- und herspringen müssen.

## Output
Konzept-Dokument unter `02_knowledge/normdex-vault/01_Produkt/` (oder `04_Verwaltung/`) — Anforderungsdokument inkl. Use Cases, Datenmodell-Auswirkungen, UI-Skizzen-Anforderungen, Modul-Priorisierung und Folge-To-dos.

## Leitidee
- Admins starten Support-Arbeit aus einer Kunden-/Organisationsakte.
- Kritische Aktionen werden nicht als freie Datenbankbearbeitung umgesetzt, sondern als geführte Admin-Aktionen.
- Jede riskante Aktion braucht Vorschau, Pflicht-Begründung, explizite Bestätigung und Audit-Logging.
- Wenn eine Aktion aus einem Kundenanliegen stammt, soll optional ein Support-Ticket als Kontext verknüpft werden.

## Zu konzipierende Module
1. **Kunden-/Organisationsakte** — zentrale Detailansicht mit Benutzer:innen, Organisation, Lizenzen, Projekten, Rechnungsdaten, Stripe-Bezug, Audit-Log und Support-Tickets.
2. **Ticket-Portal-Verknüpfung** — Anzeige aller offenen und bisherigen Tickets eines Users bzw. einer Organisation; direkte Navigation ins Ticket-Detail; optionale Ticket-Verknüpfung bei Admin-Aktionen.
3. **Admin-Statistikbereich** — KPI-Übersicht für Betreiber: Kunden, Neukunden, Lizenzen, monatlicher Lizenzumsatz, Tickets und Projektaktivität.
4. **Daten auf Kundenwunsch ändern** — Welche Felder, wer darf was, Kontrollrahmen, Ticket-Kontext und Audit-Logging.
5. **Lizenz-Operationen aus Admin-Sicht** — kündigen, pausieren, neu starten, Lizenzdetails-Seite, Auswirkungen auf Stripe und lokale Lizenzdaten.
6. **Rabatte / Coupons vergeben** — Stripe-Coupon-Lifecycle aus Admin-UI, an konkrete Subscription binden.
7. **DSGVO-Datenlöschung** — Wipe-Endpoint mit Bestätigung; aktuell nur Single-User-Delete.
8. **Refunds** — UI + Backend-Endpoint zu `stripe.Refund.create()`.
9. **Hilfe bei WBR-Projekten** — Read/Write-Zugriff auf fremde Projekte mit Logging.
10. **RBAC-Erweiterung (optional)** — Owner / Manager / Support-Agent statt Boolean `is_admin`; für v1 kann technisch weiterhin `is_admin` reichen.

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
- Admin-Aktionen können optional mit einem bestehenden Ticket verknüpft werden.
- Ticket-Zuordnung erfolgt über vorhandene Beziehungen/Felder wie `user_id`, `requester_email`, `customer_no` und Organisation/Mitgliedschaften.

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
- Audit-Log.
- `grant-complimentary` für kostenlose Lizenzen.
- Externe Statistik-Endpunkte unter `/stats/*`.
- Kunden-Dashboard-Statistiken unter `/dashboard/stats`.

## Lücken (zu konzipieren)
- Kunden-/Organisationsakte als zentrale Admin-Detailseite.
- Ticket-Übersicht pro User und Organisation.
- Admin-Statistikbereich mit Betreiber-KPIs.
- Neuer Admin-Statistik-Endpunkt, getrennt von Kunden-Dashboard und externem Monitoring.
- Lizenz aus Admin kündigen/pausieren/reaktivieren.
- Coupon-Vergabe an existierende Subscription.
- Refund-Endpoint.
- DSGVO-Bulk-Wipe.
- Datenexport (CSV/PDF) aus Admin-UI.

## Akzeptanzkriterien Konzept
- [ ] Pro Modul: Use Cases, betroffene Endpoints, Datenmodell-Änderungen, UI-Anforderungen.
- [ ] Modul-Priorisierung (welches Modul zuerst).
- [ ] Konzept beschreibt die Kunden-/Organisationsakte als zentrale Admin-Ansicht.
- [ ] Konzept enthält Ticket-Übersicht pro User und Organisation inklusive Link ins Ticket-Portal.
- [ ] Konzept definiert Statistik-KPI-Karten für Kunden, Neukunden, Lizenzen, Umsatz, Tickets und Projekte.
- [ ] Konzept legt fest, dass monatlicher Lizenzumsatz aus Stripe abgeleitet wird.
- [ ] Kritische Admin-Aktionen sind mit Begründung, Bestätigung, optionalem Ticket-Kontext und Audit-Log geplant.
- [ ] Skizze Berechtigungs-Modell (RBAC ja/nein; v1 ggf. weiter `is_admin`).
- [ ] Konzept als Markdown im Knowledge-Vault.

## Folge-To-dos
Nach Freigabe entstehen getrennte Implementierungs-To-dos, z.B.:
- T020-13 Kunden-/Organisationsakte.
- T020-14 Ticket-Verknüpfung im Verwaltungsportal.
- T020-15 Admin-Statistikbereich.
- T020-16 Lizenzaktionen aus Admin-Sicht.
- T020-17 Refunds.
- T020-18 Coupons / Rabatte.
- T020-19 DSGVO-Wipe.
