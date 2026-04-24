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

#### Phase 1 – Architektur & Datenmodell
- [ ] Bestehendes Lizenzmodell analysieren
- [ ] Neues Einzellizenzschema definieren
- [ ] Migration für `licenses`-Tabelle erstellen (neue Felder: `billing_pool`, `license_kind`, `committed_until`, `scheduled_end_at`, etc.)
- [ ] Tabellen `license_orders`, `license_order_items`, `license_events` anlegen
- [ ] Rebasierungslogik fachlich finalisieren

#### Phase 2 – Stripe
- [ ] Vier Prices in Stripe anlegen (`monthly_base` 49 €, `monthly_addon` 29 €, `yearly_base` 490 €, `yearly_addon` 290 €)
- [ ] Pool-Subscription-Konzept implementieren (max. 2 Subscriptions pro Org)
- [ ] Mapping `Lizenz ↔ Subscription Item` implementieren
- [ ] Rabattcode-/Promotion-Code-Handling definieren
- [ ] Complimentary/Testlizenzen-Konzept ergänzen (intern ohne Stripe)

#### Phase 3 – Backend
- [ ] Preview-Endpunkt für Kauf (`POST /licenses/checkout/preview`)
- [ ] Kauf-/Checkout-Endpunkte (`POST /licenses/checkout/create`, `/confirm`)
- [ ] Admin-only-Checks für Kauf und Kündigung
- [ ] Kündigungslogik „Add-on zuerst" implementieren
- [ ] Sofortige Aktivierung + individuelle Mindestlaufzeit pro Lizenz
- [ ] Rebasierungslogik nach Beendigung der Basislizenz
- [ ] Lizenzhistorie / Events (`license_events`)

#### Phase 4 – Webhooks & Synchronisierung
- [ ] Webhook-Handler erweitern (`checkout.session.completed`, `subscription.updated`, `invoice.paid`, `invoice.payment_failed`)
- [ ] Idempotenz aller Webhook-Handler absichern
- [ ] Statussynchronisierung Stripe ↔ Normdex testen
- [ ] Fehlerfälle `invoice.payment_failed` + Grace-Period-Logik

#### Phase 5 – Frontend
- [ ] Neue Lizenzverwaltung bauen (Monats-/Jahresblöcke getrennt, jede Lizenz als eigene Zeile/Karte)
- [ ] Lizenzstatus-Badges und Hinweistexte für gekündigte Lizenzen
- [ ] Kaufdialog mit Live-Kalkulation (Staffelpreise transparent, Mehrfachkauf)
- [ ] Rabattcode-Feld im Kaufdialog
- [ ] Kündigungsdialog (zeigt Enddatum, Add-on-Priorisierung)
- [ ] Rechteabhängige Buttons (Admin/Owner vs. Member)

#### Phase 6 – Tests
- [ ] Unit-Tests Preislogik (Staffelung, Mehrfachkauf, gemischte Pools)
- [ ] Unit-Tests Kündigungsreihenfolge (Add-on zuerst)
- [ ] Integrationstests Kauf mehrerer Lizenzen (monatlich + jährlich gemischt)
- [ ] Tests für Rebasierung
- [ ] Tests für UI-Hinweise gekündigter Lizenzen
- [ ] Tests für Promotions und Complimentary-Lizenzen

#### Phase 7 – Rollout
- [ ] Migration bestehender Kunden planen
- [ ] Bestandsdaten in neues Modell überführen
- [ ] Interne Dokumentation aktualisieren
- [ ] Support-/FAQ-Texte ergänzen
- [ ] Launch-Kommunikation vorbereiten

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

