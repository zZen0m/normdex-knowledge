# Verwaltungsportal: Billing- und Abo-Support

## Zielbild

Das Verwaltungsportal erhält einen vollständigen Billing- und Abo-Supportbereich für Betreiber. Admins sollen Zahlungs- und Abo-Fälle aus der Organisationsakte heraus prüfen und häufige Support-Aktionen kontrolliert ausführen können.

Der Ansatz ist hybrid:

- Normdex übernimmt wiederkehrende, klar abgrenzbare Standardfälle direkt in der Admin-UI.
- Stripe bleibt führende Quelle für Subscription-, Invoice-, Payment-, Refund- und Discount-Zustände.
- Seltene, rechtlich oder technisch riskante Sonderfälle bleiben im Stripe-Dashboard und werden aus Normdex nur verlinkt.
- Lokale Lizenzdaten bleiben die Normdex-Abbildung für Produktzugriff, Lizenzstatus, Zuweisung und Support-Kontext.

## Support-Use-Cases

### Diagnose

Admins brauchen pro Organisation eine belastbare Sicht auf:

- Stripe Customer inklusive Name, E-Mail, Rechnungsadresse, USt-ID und direktem Stripe-Link.
- Subscriptions inklusive Status, Laufzeit, Kündigungsstatus, Trial, Pause, Items, Prices und Mengen.
- Lokale Lizenzen inklusive Pool, Lizenztyp, Status, Zuweisung, Laufzeit, Stripe Subscription ID und Subscription Item ID.
- Rechnungen inklusive Status, Betrag, Fälligkeit, Hosted Invoice URL, PDF und zugehöriger PaymentIntent/Charge.
- Zahlungen inklusive bezahlter, offener, fehlgeschlagener und erstatteter Beträge.
- Payment Method Status, soweit über Stripe lesend verfügbar.
- Dunning-/Past-Due-Lage und nächste sinnvolle Aktion.
- Abweichungen zwischen Stripe und lokalen Normdex-Lizenzdaten.

### Abo-Support

Direkt in Normdex vorgesehen:

- Lizenz oder Subscription zum Periodenende kündigen.
- Kündigung zurücknehmen bzw. Subscription reaktivieren.
- Subscription pausieren und wieder aufnehmen, wenn Stripe-Konfiguration und Produktlogik das sauber unterstützen.
- Lizenzstatus aus Stripe resynchronisieren.
- Lizenzzuweisung ändern.
- Aktive Lizenznutzung freigeben.

Weiter über Stripe-Dashboard:

- Komplexe manuelle Vertragsumbauten außerhalb der Normdex-Preislogik.
- Sonderfälle mit mehreren widersprüchlichen Stripe-Objekten.
- Manuelle Subscription-Rekonstruktion nach Dateninkonsistenz.

### Zahlungs- und Rechnungs-Support

Direkt in Normdex vorgesehen:

- Offene Rechnungen anzeigen.
- Rechnungslink und PDF öffnen.
- Zahlungsversuch über Stripe-hosted Invoice- oder Payment-Seite erneut ermöglichen.
- Billing-Portal-Session für Zahlungsart- und Rechnungsdaten-Self-Service erzeugen.
- Fehlgeschlagene Zahlungen und Past-Due-Status erklären.

Nicht direkt in Normdex vorgesehen:

- Rechnungen manuell als bezahlt oder unbezahlt überschreiben.
- Stripe-Dunning-Regeln individuell pro Kunde umbauen.
- Bank- oder Zahlungsanbieter-Sonderfälle außerhalb der Stripe-Standardoberfläche lösen.

### Gutschriften und Erstattungen

Direkt in Normdex vorgesehen:

- Letzte Zahlungen und zugehörige Rechnungen anzeigen.
- Vorschau für eindeutige Invoice-/Payment-Kontexte mit Originalposition und Steuerbehandlung.
- Vollständige oder partielle Stripe Credit Note plus Erstattung ausführen.
- Bereits vorhandene Refunds ohne erneute Auszahlung mit einer Credit Note verknüpfen.
- Gutschrift-PDF und Versandstatus anzeigen.
- Fehlgeschlagene Teilvorgänge sicher und idempotent fortsetzen.
- Grund, Ticket-Kontext, Admin-ID und Stripe-Ergebnis protokollieren.

Weiter über Stripe-Dashboard:

- Uneindeutige Mehrfachzahlungen.
- Chargebacks und Disputes.
- Manuelle Korrekturen bei uneindeutiger Rechnungszuordnung oder unklarem Belegstatus.

### Rabatte und Coupons

Direkt in Normdex vorgesehen:

- Promotion Code prüfen.
- Coupon-/Discount-Daten anzeigen: Prozent, Fixbetrag, Dauer, Gültigkeit, Stripe-ID.
- Discount auf bestehende Subscription anwenden.
- Discount von Subscription entfernen.
- Auswirkungen vor Ausführung anzeigen.

Weiter über Stripe-Dashboard oder späteren separaten Workflow:

- Generische Coupons neu erstellen.
- Massenaktionen über mehrere Kunden.
- Einmalige Sonderpreise außerhalb der bestehenden Preislogik.

### Kundendaten-Sync

Direkt in Normdex vorgesehen:

- Rechnungsadresse, Name/Firma, E-Mail und USt-ID prüfen.
- Lokale Daten und Stripe Customer vergleichen.
- Synchronisierung nach Stripe mit Vorschau ausführen.
- Vorher-/Nachher-Werte im Audit-Log erfassen.

## Kontrollrahmen

Jede mutierende Billing- oder Abo-Aktion braucht:

- Vorschau der Auswirkung.
- Pflichtfeld für Grund oder Kundenwunsch.
- Optionale Verknüpfung mit einem Support-Ticket.
- Explizite Bestätigung.
- Audit-Log mit Admin, Organisation, Zielobjekt, Aktion, Stripe-ID, Zeitpunkt, Grund, Ticket-Kontext, Vorher-/Nachher-Metadaten und Ergebnisstatus.
- Fehlerfall mit sichtbarer Meldung und persistiertem Systemfehler, wenn Stripe die Aktion ablehnt oder nicht erreichbar ist.

## Vorgesehene Admin-UI

Die bestehende Organisationsakte erhält im Tab **Billing & Stripe** folgende Bereiche:

- Diagnosekarten für Customer, Subscriptions, Lizenzstatus, Zahlungsstatus und offene Rechnungen.
- Stripe-Abgleich mit Hinweisen auf lokale/Stripe-Abweichungen.
- Tabellen für Subscriptions, Subscription Items, Rechnungen und Zahlungen.
- Aktionsleiste für Standardfälle: Billing prüfen, Kundendaten synchronisieren, Kündigung, Reaktivierung, Pause, Gutschrift und Erstattung, Rabatt, Payment-Portal öffnen.
- Aktionshistorie mit Audit-Einträgen und verknüpften Tickets.
- Direkte Links zu Stripe Customer, Subscription, Invoice und Payment.

Die UI darf keine freie Stripe-Objektbearbeitung anbieten. Admins wählen geführte Aktionen aus.

## Vorgesehene Backend-Interfaces

Neue Admin-Endpunkte unter `/admin/organizations/{org_id}/billing/*`:

- `GET /summary` für Stripe- und lokale Billing-Diagnose.
- `POST /sync-customer-preview` und `POST /sync-customer`.
- `POST /subscriptions/{subscription_id}/cancel-preview` und `POST /subscriptions/{subscription_id}/cancel`.
- `POST /subscriptions/{subscription_id}/reactivate-preview` und `POST /subscriptions/{subscription_id}/reactivate`.
- `POST /subscriptions/{subscription_id}/discount-preview` und `POST /subscriptions/{subscription_id}/discount`.
- `DELETE /subscriptions/{subscription_id}/discount`.
- `POST /payments/{payment_intent_or_charge_id}/refund-preview` und `POST /payments/{payment_intent_or_charge_id}/refund`.
- `POST /portal-session` für Zahlungsart- und Rechnungsdaten-Self-Service.

Alle mutierenden Endpunkte akzeptieren mindestens:

- `reason`
- `ticket_id` optional
- `confirm` oder explizites Bestätigungsfeld
- aktionsspezifische Parameter wie Betrag, Promotion Code oder gewünschter Kündigungszeitpunkt

## Datenmodell-Auswirkungen

Für die erste Implementierung sollen bestehende Tabellen möglichst weiterverwendet werden:

- `AuditLog` für Admin-Audit.
- `LicenseEvent` für Lizenzereignisse.
- `Organization`, `License`, `LicenseOrder` und `SupportTicket` als Kontext.
- `SystemError` bzw. bestehende Systemfehler-Mechanik für Stripe-Fehler.

Falls AuditLog-Metadaten für Billing-Aktionen nicht ausreichen, wird später eine eigene Tabelle für `AdminBillingAction` geprüft. Für V1 reicht ein klar strukturierter Audit- und LicenseEvent-Eintrag.

## Modul-Priorisierung

1. Billing-Summary als read-only Diagnose aus Stripe und lokalen Lizenzdaten.
2. Kundendaten-Sync mit Preview und Audit.
3. Abo-Aktionen: Kündigung, Reaktivierung, Resync.
4. Payment-Portal-Session und Rechnungslinks.
5. Refunds mit Preview und partieller Erstattung.
6. Discounts auf bestehende Subscriptions.
7. Pause/Resume, sofern fachlich und technisch sauber abbildbar.

## Akzeptanzkriterien Konzept

- [x] Support-Use-Cases für Abo, Zahlung, Rechnung, Refund, Rabatt, Zahlungsproblem und Kundendaten-Sync sind beschrieben.
- [x] Direkt in Normdex ausführbare Aktionen sind von Stripe-Dashboard-Sonderfällen getrennt.
- [x] Stripe ist als führende Quelle für Billing-Zustände festgelegt.
- [x] Lokale Lizenzdaten bleiben als Normdex-Produktzugriffslogik festgelegt.
- [x] Mutierende Aktionen verlangen Vorschau, Pflichtgrund, Bestätigung, optionales Ticket und Audit-Logging.
- [x] Vorgesehene UI-Bereiche in der Organisationsakte sind beschrieben.
- [x] Vorgesehene Backend-Endpoints sind skizziert.
- [x] Priorisierung für die spätere Implementierung ist festgelegt.

## Folge-Todos

- [[T020-16-Lizenz-und-Billing-Support-Aktionen]] als Haupt-Implementierungspaket.
- T020-17 Refunds als eigenes Unterpaket, wenn Refunds separat umgesetzt werden sollen.
- T020-18 Coupons / Rabatte als eigenes Unterpaket, wenn Rabattlogik separat umgesetzt werden soll.
