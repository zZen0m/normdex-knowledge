# T020-16 · Lizenz- und Billing-Support-Aktionen

**Phase:** 4 (Konzept und Umsetzung abgeschlossen)  
**Priorität:** P2 · Verwaltungsportal / Stripe / Support  
**Status:** abgeschlossen (2026-06-23)  
**Parent:** [[T020-allgemeine Todos]]  
**Referenz:** [[Verwaltungsportal-Billing-und-Abo-Support]]

## Ziel

Admins sollen aus der Organisationsakte heraus Abo-, Zahlungs- und Billing-Support in vollem Umfang für Standardfälle leisten können. Normdex übernimmt kontrollierte Support-Aktionen direkt; komplexe Sonderfälle bleiben über Stripe-Dashboard-Links erreichbar.

## Umfang

### Diagnose

- Stripe Customer, Subscriptions, Subscription Items, Rechnungen, Zahlungen und Payment-Status anzeigen.
- Lokale Lizenzdaten und Stripe-Zustand gemeinsam darstellen.
- Abweichungen zwischen Stripe und Normdex sichtbar machen.
- Offene Rechnungen, fehlgeschlagene Zahlungen und Past-Due-Kontext hervorheben.

### Abo-Aktionen

- Lizenz/Subscription zum Periodenende kündigen.
- Kündigung zurücknehmen bzw. Subscription reaktivieren.
- Lizenzstatus aus Stripe resynchronisieren.
- Aktive Lizenznutzung freigeben.
- Keine Lizenzzuweisung im Support-Workflow, da Normdex ein Concurrent-User-Modell nutzt und Zuweisung fachlich nicht relevant ist.
- Kein Pause/Resume-Workflow: Eine Subscription läuft oder läuft nicht. Pausen werden bewusst nicht als eigener Supportfall abgebildet.

### Zahlungs- und Rechnungs-Support

- Offene Rechnungen anzeigen.
- Hosted Invoice URL und Rechnungs-PDF öffnen.
- Payment Method Update über Billing-Portal-Session starten.
- Zahlungsprobleme sichtbar erklären, ohne Rechnungen manuell zu überschreiben.

### Refunds und Rabatte

- Refunds mit Vorschau, Pflichtgrund, optionalem Ticket-Kontext und Audit-Log vorbereiten.
- Promotion Code prüfen.
- Discount auf bestehende Subscription anwenden oder entfernen.
- T020-17 und T020-18 können als separate Unterpakete ausgekoppelt werden, wenn der Umfang für einen Sprint zu groß ist.

### Kundendaten-Sync

- Rechnungsadresse, Name/Firma, E-Mail und USt-ID lokal mit Stripe vergleichen.
- Sync-Vorschau anzeigen.
- Daten kontrolliert nach Stripe synchronisieren.

## Vorgesehene Backend-Endpunkte

- `GET /admin/organizations/{org_id}/billing/summary`
- `POST /admin/organizations/{org_id}/billing/sync-customer-preview`
- `POST /admin/organizations/{org_id}/billing/sync-customer`
- `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/cancel-preview`
- `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/cancel`
- `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/reactivate-preview`
- `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/reactivate`
- `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/discount-preview`
- `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/discount`
- `DELETE /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/discount`
- `POST /admin/organizations/{org_id}/billing/payments/{payment_intent_or_charge_id}/refund-preview`
- `POST /admin/organizations/{org_id}/billing/payments/{payment_intent_or_charge_id}/refund`
- `POST /admin/organizations/{org_id}/billing/portal-session`

## Kontrollanforderungen

Jede mutierende Aktion braucht:

- Vorschau der Auswirkung.
- Pflichtfeld für Grund / Kundenwunsch.
- Optionale Verknüpfung mit bestehendem Support-Ticket.
- Explizite Bestätigung.
- Audit-Log mit Admin, Organisation, Zielobjekt, Stripe-ID, Aktion, Zeitpunkt, Grund, Vorher-/Nachher-Metadaten und Ergebnisstatus.
- Fehlerprotokollierung bei Stripe-Fehlern.

## Akzeptanzkriterien

- [x] Billing-Summary zeigt Stripe- und lokale Lizenzdaten in der Organisationsakte.
- [x] Offene Rechnungen, Zahlungsprobleme und Past-Due-Kontext sind sichtbar.
- [x] Admin-Aktionen sind als geführte Workflows mit Vorschau, Grund, Bestätigung, Ticket-Kontext und Audit umgesetzt. *(Standardumfang: Kundendaten-Sync, Billing-Portal, Kündigung/Reaktivierung, Nutzungsfreigabe, Status-Resync, Rabatte und Refunds umgesetzt; Pause/Resume bewusst nicht im Umfang)*
- [x] Kündigung und Reaktivierung funktionieren aus Admin-Sicht ohne Bruch der bestehenden Kunden-Self-Service-Logik.
- [x] Aktive Lizenznutzung kann durch Admin-Support freigegeben werden.
- [x] Lizenzstatus kann aus Stripe resynchronisiert werden.
- [x] Kundendaten-Sync zeigt Vorher/Nachher und schreibt nach erfolgreicher Stripe-Aktion Audit-Log.
- [x] Refund- und Rabattumfang ist entweder umgesetzt oder als T020-17/T020-18 ausgekoppelt. *(Standardfälle umgesetzt: Promotion Codes/Discounts und eindeutige Stripe-Charge-Refunds)*
- [x] Komplexe Sonderfälle führen zu Stripe-Dashboard-Links statt zu freier Objektbearbeitung in Normdex.
- [x] Backend-Tests decken Erfolg, fehlende Adminrechte, falsche Organisation, Stripe-Fehler und Audit-Logging ab. *(teilweise: Summary, Kundendaten-Sync, Kündigung/Reaktivierung, Nutzungsfreigabe, Status-Resync, Rabatte und Refunds abgedeckt)*
- [x] Frontend zeigt Lade-, Fehler-, Preview- und Erfolgzustände. *(teilweise: Billing-Summary, Kundendaten-Sync, Billing-Portal, Kündigung/Reaktivierung, Nutzungsfreigabe, Status-Resync, Rabatte und Refunds angebunden)*

## Notizen / Fortschritt

- 2026-05-30: Konzept unter [[Verwaltungsportal-Billing-und-Abo-Support]] angelegt. Umsetzung noch offen.
- 2026-05-30: Umsetzung gestartet mit read-only Billing-Summary für die Organisationsakte (`GET /admin/organizations/{org_id}/billing/summary`) und UI-Diagnose im Tab "Billing & Stripe".
- 2026-05-30: Kundendaten-Sync ergänzt: Preview, bestätigter Admin-Sync, Pflichtgrund, optionales Ticket und Audit-Log.
- 2026-05-30: Admin-Kündigung und Admin-Reaktivierung für Lizenzen ergänzt: Preview-Endpunkte, bestätigte Aktionen mit Pflichtgrund/Ticket-Kontext, Audit-Log, UI-Anbindung und Backend-Tests.
- 2026-05-30: Billing-Portal-Session für Admin-Support ergänzt: Kunden können Zahlungsart und Rechnungsdaten über Stripe-hosted Self-Service aktualisieren; Admin-Aktion schreibt Audit-Log und führt zurück in den Billing-Tab.
- 2026-05-30: Aktive Lizenznutzung freigeben ergänzt: Admin-Support kann blockierte Concurrent-User-Slots mit Vorschau, Pflichtgrund, optionalem Ticket-Kontext, LicenseEvent und Audit-Log freigeben. Lizenzzuweisung wird nicht umgesetzt, da sie nicht zum Concurrent-User-Modell passt.
- 2026-05-30: Stripe-Status-Resync ergänzt: Admin-Support kann eine Stripe Subscription erneut lesen, Status-/Periodenabweichungen vorab prüfen und nach Bestätigung auf die lokale Lizenz übernehmen.
- 2026-05-30: Rabatt-/Coupon-Support ergänzt: Admin-Support kann Promotion Codes prüfen, auf bestehende Stripe Subscriptions anwenden und bestehende Discounts entfernen. Neue generische Coupons bleiben bewusst außerhalb von Normdex.
- 2026-05-30: Refund-Support ergänzt: Admin-Support kann bezahlte Rechnungen mit eindeutiger Stripe Charge vollständig oder partiell erstatten. Umsetzung enthält Preview, Pflichtgrund, optionalen Ticket-Kontext, Stripe Refund, Audit-Log und UI-Anbindung.
- 2026-05-30: Stripe-Dashboard-Deep-Links ergänzt: Billing-Summary liefert test/live-korrekte Dashboard-URLs für Customer, Subscriptions, Invoices und Payments/Charges; UI zeigt direkte Stripe-Links für Sonderfälle.
- 2026-05-30: Pause/Resume fachlich ausgeschlossen. Zielmodell bleibt: Subscription läuft oder läuft nicht.
- 2026-05-30: Manuelle Abnahme-Checkliste angelegt: [[T020-16-Abnahme-Checkliste-Billing-Support]].
- 2026-06-23: Rabatt-/Coupon-Bugs behoben: Coupon-Details wurden nicht angezeigt, weil die neue Stripe-Discount-Struktur (`source.coupon`) nicht gelesen wurde; bestehende Discounts wurden in Vorschau und Entfernen-Dialog nicht angezeigt; Discount-Entfernung wirkte nur lokal. Fix: erweiterte Stripe-Expands, neue Coupon-Resolver-Helper und `stripe.Subscription.delete_discount(...)` statt `modify(discounts=[])`. Backend-Tests ergänzt (5 Discount-Tests grün).
- 2026-06-23: Diagnose verbessert: Stripe-Subscriptions zeigen jetzt zugeordnete Normdex-Lizenz(en) inkl. Lizenzart, Abrechnungsrhythmus und Status; "Lokale Lizenzen" in "Normdex-Lizenzen" umbenannt.
- 2026-06-23: Manuelle Re-Abnahme aller zuvor offenen Checklisten-Punkte erfolgreich. T020-16 abgeschlossen.
