# Aufgaben

Zentrale Aufgabenliste für alle Bereiche des Normdex-Projekts.

**Status-Symbole:** `[ ]` offen · `[x]` erledigt · `[~]` in Arbeit · `[-]` verworfen

---

## Dokumentation & Vault

- [x] Ordnerstruktur `D:\Normdex` etabliert (01_repos, 02_knowledge, 03_external, 04_workspace)
- [x] CLAUDE.md beider Repos um Workspace-Kontext ergänzt
- [x] Branding-Guidelines in Vault überführt (Brand Identity & Voice, Designsystem & Farben)
- [x] NORMDEX_COMPLETE_GUIDE vollständig in Vault migriert (alle 22 Abschnitte)
- [x] NORMDEX_COMPLETE_GUIDE als veraltet markiert
- [ ] NORMDEX_COMPLETE_GUIDE aus Repo löschen (`docs/NORMDEX_COMPLETE_GUIDE.md`)
- [ ] Prüfen, ob weitere Docs im Repo (`docs/`) in den Vault gehören oder gelöscht werden können

---

## App (`app.normdex.at`)

### Lizenzsystem – Umbau auf Pool-Modell mit Sammelabrechnung

Vollständige Spec: [[normdex_lizenzsystem_developer_spec]]

**Ziel:** Einzellizenzen pro Datensatz, zwei getrennte Billing-Pools (monthly/yearly), Staffelpreise, Sammelabrechnung über Stripe, Admin-only-Kündigung, Rabattcodes, Rebasierungslogik.

**Aktueller Stand:** Phase 1–6 sind technisch umgesetzt und getestet. Die verbleibenden Punkte liegen vor allem in Rollout, Prod-Konfiguration, lokaler Stripe-Webhook-Verifikation und Dokumentation.

#### Phase 1 – Architektur & Datenmodell
- [x] Bestehendes Lizenzmodell analysieren
- [x] Neues Einzellizenzschema definieren
- [x] Migration für `licenses`-Tabelle erstellen (neue Felder: `billing_pool`, `license_kind`, `committed_until`, `scheduled_end_at`, etc.)
- [x] Tabellen `license_orders`, `license_order_items`, `license_events` anlegen
- [x] `LicenseOrder.meta` ergänzen (Modell + Migration `342d48e1f270`)
- [x] Rebasierungslogik fachlich finalisieren

#### Phase 2 – Stripe
- [x] Vier Prices in Stripe Sandbox anlegen (`monthly_base` 49 €, `monthly_addon` 29 €, `yearly_base` 490 €, `yearly_addon` 290 €)
- [x] Pool-Subscription-Konzept implementieren (max. 2 Subscriptions pro Org)
- [x] Mapping `Lizenz ↔ Subscription Item` implementieren
- [~] Rabattcode-/Promotion-Code-Handling definieren
  - [x] Rabattcodes bei neuen Stripe-Checkout-Subscriptions über Stripe Checkout nutzbar
  - [ ] Rabattcodes bei direkter Erweiterung bestehender Subscriptions bewusst nicht implementiert / fachlich noch offen
- [x] Complimentary/Testlizenzen-Konzept ergänzen (intern ohne Stripe)

#### Phase 3 – Backend
- [x] Preview-Endpunkt für Kauf (`POST /licenses/checkout/preview`)
- [x] Kauf-/Checkout-Endpunkte (`POST /licenses/checkout/create`, `/confirm`)
  - [x] Dev-Test 2026-04-26: Kauf einer monatlichen Basislizenz über Stripe Checkout erfolgreich; `/licenses/checkout/confirm` aktiviert die Lizenz in Normdex.
  - [x] Bugfix 2026-04-26: Stripe Python 14 kompatibel gemacht (`subscription.items`-Kollision und Periodenfelder auf Subscription-Items).
- [x] Admin-only-Checks für Kauf und Kündigung
- [x] Kündigungslogik „Add-on zuerst" implementieren
- [x] Sofortige Aktivierung + individuelle Mindestlaufzeit pro Lizenz
- [x] Rebasierungslogik nach Beendigung der Basislizenz
- [x] Lizenzhistorie / Events (`license_events`)

#### Phase 4 – Webhooks & Synchronisierung
- [x] Webhook-Handler erweitern (`checkout.session.completed`, `subscription.updated`, `invoice.paid`, `invoice.payment_failed`)
- [x] Idempotenz aller Webhook-Handler absichern
- [~] Statussynchronisierung Stripe ↔ Normdex lokal mit Stripe CLI final testen
  - [~] Dev-Test 2026-04-26: Checkout-Session wurde erstellt und Stripe-Events kamen an; Webhook schlägt lokal noch mit `400 Ungültige Stripe-Signatur` fehl. Aktivierung wurde erfolgreich über `/licenses/checkout/confirm` nachgezogen.
- [x] Fehlerfälle `invoice.payment_failed` + `past_due`/`unpaid` abbilden

#### Phase 5 – Frontend
- [x] Neue Lizenzverwaltung bauen (Monats-/Jahresblöcke getrennt, jede Lizenz als eigene Zeile/Karte)
- [x] Lizenzstatus-Badges und Hinweistexte für gekündigte Lizenzen
- [x] Kaufdialog mit Live-Kalkulation (Staffelpreise transparent, Mehrfachkauf)
- [~] Rabattcode-Hinweise im Kaufdialog
  - [x] Stripe Checkout verweist auf Rabattcode-Eingabe bei neuer Subscription
  - [ ] Kein Rabattcode-Feld für direkte bestehende Pool-Erweiterungen
- [x] Kündigungsdialog (zeigt Enddatum, Add-on-Priorisierung)
- [x] Kündigung über das Drei-Punkte-Menü wieder zurückziehen (nur bis zum endgültigen Ablaufdatum)
- [x] Rechteabhängige Buttons (Admin/Owner vs. Member)
- [x] Aktive Sitzungen je Lizenz anzeigen (Avatar, Name, Startzeit)
- [x] Statische Benutzerzuweisung in der UI ausblenden

#### Phase 6 – Tests
- [x] Unit-Tests Preislogik (Staffelung, Mehrfachkauf, gemischte Pools)
- [x] Unit-Tests Kündigungsreihenfolge (Add-on zuerst)
- [x] Integrationstests Webhook-/Aktivierungslogik
- [x] Tests für Rebasierung
- [ ] Tests für UI-Hinweise gekündigter Lizenzen
- [ ] Tests für Promotions und Complimentary-Lizenzen

#### Phase 7 – Rollout
- [ ] Prod-Migration vorbereiten und anwenden
- [ ] Prod-Stripe-Keys / Live-Mode Prices konfigurieren
- [ ] Lokalen Stripe-CLI-Webhooksignatur-Test abschließen
- [ ] Migration bestehender Kunden planen
- [ ] Bestandsdaten in neues Modell überführen
- [x] Interne Dokumentation aktualisieren
- [x] Support-/FAQ-Texte ergänzen
- [ ] Launch-Kommunikation vorbereiten

### Projekte

- [ ] Projektstatus einführen (z. B. „in Bearbeitung", „abgeschlossen") – Auswahl im Projekt-Formular, Anzeige in der Projektliste und Detailansicht

---

## Landingpage (`normdex.at`)

- [ ] 

---

## Marketing

- [ ] 

---

## Geschäft & Rechtliches

- [ ] 

---

## Entwicklung & Infrastruktur

- [ ] 
