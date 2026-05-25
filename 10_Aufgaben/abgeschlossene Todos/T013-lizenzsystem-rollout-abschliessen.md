# T013 - Lizenzsystem-Rollout abschließen

**Status:** erledigt
**Bereich:** App / Infrastruktur / Marketing
**Erstellt:** 2026-04-27
**Abgeschlossen:** 2026-05-25

## Zweck

Dieses Todo ist die konsolidierte Arbeitsgrundlage für die Normdex-Lizenzsystem-Umstellung auf ein Pool-Modell mit Sammelabrechnung. Es ersetzt die früheren separaten Dateien `normdex_lizenzsystem_developer_spec.md` und `lizenzsystem_implementierung_fortschritt.md`.

Die Datei enthält:

- fachliches Zielbild
- technische Soll-Architektur
- aktuellen Implementierungsstand
- bekannte Stolpersteine
- offene To-dos bis zum produktionsreifen Rollout
- Abschlusskriterien

## Zielbild

Das Lizenzsystem soll so funktionieren:

- Jede Lizenz ist ein eigener Normdex-Datensatz mit eigenem Startdatum, Preis, Status, Mindestlaufzeit, Kündigungsinformation und Stripe-Verknüpfung.
- Pro Organisation gibt es maximal zwei Abrechnungspools: `monthly` und `yearly`.
- Stripe führt je Pool eine Sammel-Subscription, nicht eine Subscription pro Einzellizenz.
- Monatliche und jährliche Lizenzen bleiben rabattlogisch getrennt.
- Staffelpreise gelten nur innerhalb desselben Pools.
- Neue Lizenzen werden sofort aktiviert und anteilig im jeweiligen Pool verrechnet.
- Kündigungen dürfen nur Admins/Owner auslösen.
- Normale Members dürfen Lizenzen sehen, aber nicht kaufen, kündigen, Rabattcodes anwenden oder Preise ändern.
- Bei Kündigung bleibt der Zugriff bis zum Laufzeitende aktiv.
- Bei Reduktion wird zuerst immer eine Add-on-Lizenz gekündigt; eine Basislizenz darf erst gekündigt werden, wenn keine Add-ons im Pool aktiv sind oder der Pool aufgelöst wird.
- Wenn die Basislizenz endet und Add-ons verbleiben, wird automatisch die älteste aktive Add-on-Lizenz zur neuen Basislizenz.
- Jede Lizenz erlaubt genau eine gleichzeitige aktive Sitzung über das bestehende Concurrent-Use-/Heartbeat-Locking.
- Die statische Benutzerzuweisung (`assigned_user_id`) ist im Datenmodell vorhanden, wird aber in der UI bewusst nicht exponiert.

## Nicht-Ziele

- Kein Umbau des grundsätzlichen Concurrent-Use-/Heartbeat-Prinzips.
- Keine vollständige Neugestaltung des Team-/Organisationsmodells.
- Keine neuen Produktlinien außerhalb von `economics_v1`.
- Keine Vereinheitlichung von monatlicher und jährlicher Abrechnung in eine Gesamtrechnung.
- Keine finale juristische Ausformulierung von AGB-Texten.

## Preise und Stripe-Objekte

### Preislogik

| Pool | Basislizenz | Add-on-Lizenz |
|---|---:|---:|
| monatlich | 49,00 EUR / Monat | 29,00 EUR / Monat |
| jährlich | 490,00 EUR / Jahr | 290,00 EUR / Jahr |

### Sandbox-Stripe-Objekte

| Produkt | Produkt-ID | Price | Price-ID |
|---|---|---:|---|
| ÖNORM M 7140 Basic - Hauptlizenz | `prod_UPCq9bTtJyg63C` | 49 EUR / Monat | `price_1TQOQMF05ipkEAzmY6FltbRW` |
| ÖNORM M 7140 Basic - Hauptlizenz | `prod_UPCq9bTtJyg63C` | 490 EUR / Jahr | `price_1TQOQMF05ipkEAzmYkug1hWf` |
| ÖNORM M 7140 Basic - Zusatzlizenz | `prod_UPCvOriaUUME9K` | 29 EUR / Monat | `price_1TQOViF05ipkEAzmsbH2I1is` |
| ÖNORM M 7140 Basic - Zusatzlizenz | `prod_UPCvOriaUUME9K` | 290 EUR / Jahr | `price_1TQOflF05ipkEAzmL2zqIb5t` |

Für Produktion müssen eigene Live-Mode Products/Prices angelegt und die produktiven Price-IDs in der App-Konfiguration hinterlegt werden.

## Datenmodell

### `licenses`

Wichtige Felder:

- `id`
- `organization_id`
- `product_key` (`economics_v1`)
- `billing_pool` (`monthly` oder `yearly`)
- `license_kind` (`base` oder `addon`)
- `status` (`pending`, `active`, `scheduled_end`, `ended`, `payment_failed`)
- `started_at`
- `current_term_start`
- `current_term_end`
- `committed_until`
- `cancel_requested_at`
- `scheduled_end_at`
- `price_amount_gross`
- `currency`
- `stripe_subscription_id`
- `stripe_subscription_item_id`
- `stripe_price_id`
- `assigned_user_id`
- `created_by_user_id`
- `cancel_at_period_end`
- `meta`
- `created_at`

Hinweise:

- `stripe_customer_id` liegt auf `Organization`, nicht mehr auf `License`.
- `max_concurrent_users` entfällt; eine Lizenz entspricht einer aktiven Nutzung.
- SQLite-Kompatibilität beachten: IDs sind als `String` gespeichert, nicht als nativer UUID-Typ.

### `license_orders`

Speichert Kaufvorgänge:

- `organization_id`
- `created_by_user_id`
- `promotion_code`
- `subtotal_gross`
- `discount_total_gross`
- `total_gross`
- `currency`
- `status` (`pending`, `completed`, `failed`)
- `stripe_checkout_session_id`
- `meta`
- `created_at`

`meta` ist wichtig für gemischte Orders mit mehreren Checkout-Sessions.

### `license_order_items`

Speichert Preiszeilen einer Bestellung:

- `license_order_id`
- `billing_pool`
- `license_kind`
- `quantity`
- `unit_price_gross`
- `line_total_gross`

### `license_events`

Audit-/Historienlog für Lizenzereignisse. Relevante Events sind unter anderem Aktivierung, Kündigung, Rebasierung, Zahlungsausfall und Statussynchronisierung.

### `license_usages`

Bleibt als Grundlage für das bestehende Concurrent-Use-/Heartbeat-Locking erhalten.

## API und Code-Orientierung

Wichtige Implementierungsorte:

- `apps/api/app/models.py`
- `apps/api/app/domain/license_pricing.py`
- `apps/api/app/routers/licenses.py`
- `apps/api/app/routers/licenses_v2.py`
- `apps/api/app/routers/subscriptions.py`
- `apps/api/app/constants.py`
- `apps/frontend/src/pages/Licenses.tsx`
- `apps/frontend/src/pages/SubscriptionPlans.tsx`
- `apps/frontend/src/hooks/useLicenseLock.ts`
- `apps/frontend/src/api.ts`

Wichtige Endpoints:

- `GET /licenses`
- `POST /licenses/checkout/preview`
- `POST /licenses/checkout/create`
- `POST /licenses/checkout/confirm`
- `POST /licenses/checkout/cancel`
- `POST /licenses/{id}/cancel`
- `POST /licenses/{id}/reactivate-cancel`
- `POST /licenses/promotions/validate`
- `POST /admin/licenses/grant-complimentary`

Veraltete Subscription-Endpunkte wurden durch die Lizenz-v2-Flows ersetzt.

## UI-Anforderungen

Die Lizenzverwaltung muss zeigen:

- getrennte Blöcke für monatliche und jährliche Pools
- jede Lizenz als eigene Zeile/Karte
- Lizenztyp: Basislizenz oder Add-on-Lizenz
- Pool: monatlich oder jährlich
- Preis
- Startdatum
- Laufzeitende bzw. nächstes Vertragsende
- Status
- aktive Sitzung mit Nutzername und Startzeit, falls belegt
- Kündigungsstatus inklusive Enddatum
- Admin-/Owner-Aktionen für Kauf, Kündigung und Rücknahme einer Kündigung

Status müssen verständlich unterscheidbar sein:

- Aktiv
- Ausstehend / Pending
- Gekündigt zum Laufzeitende
- Beendet
- Zahlung fehlgeschlagen

Gekündigte Lizenzen müssen einen klaren Hinweis mit Enddatum anzeigen, zum Beispiel: `Diese Lizenz wurde gekündigt und endet mit Ablauf des DD.MM.YYYY.`

Der Kaufdialog muss unterstützen:

- Auswahl monatlicher und jährlicher Lizenzen
- Mehrfachkauf
- Live-Preisberechnung
- transparente Darstellung von Basis-/Add-on-Preisen
- Rabattcode-Hinweis bzw. Rabattcode-Feld gemäß finaler fachlicher Entscheidung
- Hinweis bei gemischten Pools, dass getrennte Stripe-Checkouts entstehen können

## Webhooks und Hintergrundlogik

Relevante Stripe-Events:

- `checkout.session.completed`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.paid`
- `invoice.payment_failed`

Erwartetes Verhalten:

- `checkout.session.completed`: Order bestätigen, Lizenzen aktivieren, Stripe-IDs speichern, Events schreiben.
- `customer.subscription.updated`: Perioden synchronisieren, `scheduled_end` bei Ablauf auf `ended` setzen, Rebasierung auslösen, `past_due`/`unpaid` auf `payment_failed` abbilden.
- `customer.subscription.deleted`: betroffene Lizenzen beenden und Rebasierung prüfen.
- `invoice.paid`: Zahlungsstatus wieder auf aktive Zustände synchronisieren, soweit fachlich korrekt.
- `invoice.payment_failed`: betroffene Lizenzen auf `payment_failed` setzen.

Technische Regeln:

- Webhook-Verarbeitung muss idempotent bleiben.
- `checkout.session.completed` darf bereits `completed` oder `failed` Orders nicht erneut aktivieren.
- Stripe Python 14: `subscription.items` kollidiert mit der Dict-Methode `items()`, daher `subscription.get("items")` verwenden.
- Aktuelle Stripe-API-Versionen liefern Abrechnungsperioden teilweise am Subscription-Item. Code darf nicht ausschließlich `subscription.current_period_start` und `subscription.current_period_end` verwenden.
- Webhook-Handler sind synchron gehalten, um DB-Zugriff ohne Event-Loop-Konflikte einfach zu halten.
- Fehler in Stripe-Seiteneffekten wie `_cancel_stripe_item()` werden geloggt, dürfen aber nicht unkontrolliert lokale DB-Zustände zurückdrehen.

## Aktueller Implementierungsstand

### App-seitig abgeschlossen / statisch verifiziert

- Neues Pool-Datenmodell ist vorhanden: `licenses`, `license_orders`, `license_order_items`, `license_events`, `Organization.stripe_customer_id`.
- Preislogik, Trial-Benefit, Trial-Konvertierung, Add-on-/Basis-Splitting und Pool-Trennung sind in `license_pricing.py` und `licenses_v2.py` umgesetzt.
- Neue Lizenz-v2-Endpunkte sind vorhanden: Pool-Übersicht, Checkout-Preview/Create/Confirm/Cancel, Kündigung, Kündigungsrücknahme, Promotion-Validierung und Complimentary-Lizenz.
- Neue Pool-Subscriptions werden per Stripe Checkout erstellt; bestehende Pool-Subscriptions werden direkt per `Subscription.modify` erweitert.
- Gemischte monatliche/jährliche Orders werden getrennt je Pool verarbeitet und in `order.meta.checkout_sessions` abgebildet.
- Webhook-Handler für `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, `invoice.paid` und `invoice.payment_failed` sind umgesetzt.
- Idempotenz ist für abgeschlossene/fehlgeschlagene Orders abgesichert; späte Webhooks reaktivieren keine verworfenen Orders.
- Rebasierung ist umgesetzt: Wenn keine aktive Basislizenz mehr vorhanden ist, wird das älteste aktive Add-on im Pool zur Basislizenz.
- Kündigungslogik ist umgesetzt: Add-ons zuerst, Basis nur wenn zulässig; gekündigte Lizenzen bleiben bis `scheduled_end_at` nutzbar.
- Rücknahme einer Kündigung ist umgesetzt: `POST /licenses/{id}/reactivate-cancel`, solange `scheduled_end_at` in der Zukunft liegt.
- Abgebrochene Checkouts werden bereinigt: Pending-Lizenzen werden beendet, Orders werden `failed`, Trial-Locks werden nur ohne entstandene Stripe-Subscription freigegeben.
- Lizenzverwaltung im Frontend ist eingeführt: getrennte Monats-/Jahrespools, Einzellizenzen, Statusanzeige, Kaufdialog, Kündigung, Rücknahme, Checkout-Cancel/Confirm und Stripe-Portal.
- `SubscriptionPlans.tsx` leitet auf `/licenses` um; die Landingpage-/Kaufintent-Übergabe in die App ist separat mit T018 abgeschlossen.
- App-Hilfe und Vault-Doku beschreiben das Pool-Modell, Trial/Erstbestellungsrabatt, Kündigung, Stripe-Flows und Newsletter-Gutscheine.
- Rabattcodes werden sowohl bei neuen Stripe-Checkout-Subscriptions als auch bei direkter Erweiterung bestehender Pool-Subscriptions an Stripe übergeben.
- Deployment-Env-Beispiele enthalten seit 2026-05-22 die vier Pool-Price-IDs, Product-IDs, Coupon-IDs und `APP_ENV`.

### Tests / Verifikation

- 2026-05-22 ausgeführt: `.\venv\Scripts\python -m pytest tests/test_license_pricing.py tests/test_license_checkout_trial.py tests/test_license_webhooks.py tests/test_license_cancel_reactivation.py tests/test_subscription_portal.py tests/test_newsletter.py` → **117 passed**.
- 2026-05-22 ausgeführt: `npm test -- --run` in `apps/frontend/` → **12 passed**; Hinweis: npm meldet `Unknown cli config "--run"`, Vitest lief trotzdem erfolgreich.
- 2026-05-22 ausgeführt: `npm run build` in `apps/frontend/` → erfolgreich; Vite meldet nur den bekannten Chunk-Size-Hinweis.
- Noch nicht vorhanden: gezielte Komponententests für `Licenses.tsx` selbst. Die UI wurde statisch gegen den aktuellen Code abgeglichen, aber nicht per RTL/Playwright automatisiert getestet.

## Bekannte Probleme und Stolpersteine

- Lokales Stripe-Webhook-Forwarding war früher wegen Signaturprüfung blockiert; ein aktuell erfolgreicher echter Stripe-CLI-Test mit gültigem Webhook-Secret ist in diesem Todo nicht dokumentiert.
- Rabattcodes werden bei direkter Pool-Erweiterung per `discounts=[{"promotion_code": ...}]` an `Subscription.modify` übergeben. Vor Livegang sollte das einmal gegen Stripe-Testmode/Staging mit einem echten Promotion-Code verifiziert werden.
- Prod-Migration, Live-Mode Stripe Products/Prices/Keys und produktiver Testkauf/Dry-Run wurden am 2026-05-25 vom Projektverantwortlichen als erledigt bestätigt.
- Bestandskunden- und Stripe-Bestandsdatenmigration ist für den T013-Rollout nicht mehr blockierend; konkrete Folgearbeiten werden bei Bedarf separat geführt.
- Für Migrationen gilt: vor Alembic-Aktionen im App-Repo den DB-Migration-Skill lesen.
- Dev-Testdaten: `Licenses` und `LicenseUsages` können gelöscht werden, aber `Projects`, `Organizations` und `Users` müssen erhalten bleiben.
- Stripe-Subscriptions können in Dev und Prod gelöscht werden, sofern bewusst im Rahmen der Migration entschieden.

## Offene To-do-Liste

### 1. Rabattcode-Strategie final glätten

- [x] Grundentscheidung dokumentiert: Rabattcodes gelten auch bei direkter Pool-Erweiterung.
- [x] Newsletter-Codes werden validiert und als Stripe Promotion Code an Checkout Sessions übergeben.
- [x] Direkte Pool-Erweiterungen übergeben den Promotion-Code an `Subscription.modify`.
- [x] Backend-Tests für Newsletter-Code, abgelaufene Codes, Stripe-Checkout-Übergabe und Direct-Subscription-Update-Übergabe vorhanden.
- [x] Verhalten mit echtem Stripe-Testmode-/Live-Promotion-Code auf Staging/Produktion geprüft; Ergebnis am 2026-05-25 vom Projektverantwortlichen als erfolgreich bestätigt.

### 2. Stripe-Verifikation abschließen

- [x] Webhook-Handler sind per automatisierten Backend-Tests abgedeckt.
- [x] Subscription-Item-Perioden werden im Code berücksichtigt und in Webhook-Tests abgedeckt.
- [x] Stripe CLI bzw. Staging/Produktion mit gültigem Webhook-Secret geprüft; externer Test am 2026-05-25 bestätigt.
- [x] `checkout.session.completed` mit echter Stripe-Signatur verarbeitet; externer Test am 2026-05-25 bestätigt.
- [x] `customer.subscription.updated`, `invoice.paid` und `invoice.payment_failed` gegen Staging/Stripe-Testmode bzw. Produktion getestet; externer Test am 2026-05-25 bestätigt.
- [x] Ergebnis direkt unter „Notizen / Fortschritt“ dokumentiert.

### 3. Testabdeckung ergänzen

- [x] Backend-Tests für Pricing, Trial, Checkout-Cancel, Webhooks, Kündigungsrücknahme, Newsletter-Promotions und Subscription-Portal laufen.
- [x] Frontend-Basistests und Build laufen.
- [x] UI-Komponententests für `Licenses.tsx` als T013-Blocker bewusst durch manuelle Abnahme und bestehende Frontend-Basistests ersetzt.
- [x] UI-Komponententests für gekündigte Lizenzen, Rücknahme-Button, Checkout-Cancel und Gutscheinfeld je Flow als T013-Blocker bewusst durch manuelle Abnahme ersetzt.
- [x] Backend-Test für `/admin/licenses/grant-complimentary` als T013-Blocker bewusst zurückgestellt; bei Bedarf als separates Testabdeckungs-Todo führen.

### 4. Produktion und Infrastruktur

- [x] Deployment-Env-Beispiele auf Pool-Price-IDs und `APP_ENV` aktualisiert.
- [x] `deploy/README.md` dokumentiert die vier Price-IDs monatlich/jährlich × Basis/Add-on.
- [x] Prod-Migration vorbereitet, Backup-/Rollback-Vorgehen für diesen Rollout berücksichtigt.
- [x] Migration auf Dev-Server/Staging validiert.
- [x] Prod-Migration nach Freigabe angewendet.
- [x] Stripe-Live-Mode Products und Prices angelegt.
- [x] Produktive Stripe-Keys und Live-Price-IDs sicher konfiguriert.
- [x] Produktiven Testkauf oder geeigneten Dry-Run durchgeführt; am 2026-05-25 vom Projektverantwortlichen bestätigt.

### 5. Bestandskunden und Datenmigration

- [x] Betroffene Bestandskunden, Organisationen, Lizenzen und Stripe-Subscriptions für den Rollout geprüft bzw. als nicht blockierend bewertet.
- [x] Zielzustand pro Kundentyp für den Rollout ausreichend geklärt.
- [x] Entscheidung über bestehende Lizenzen/Subscriptions für den Rollout getroffen.
- [x] Migrationsreihenfolge, Verantwortlichkeiten und Prüfschritte für den Rollout ausreichend geklärt.
- [x] Bestandsdaten gemäß Rollout-Entscheidung überführt bzw. kein weiterer T013-Blocker.
- [x] Datenintegrität geprüft.
- [x] Stripe- und Normdex-Zustand abgeglichen.

### 6. Launch und Kommunikation

- [x] Grundlegende App-/Vault-Doku zum Lizenzsystem ist aktualisiert.
- [x] Launch-Kommunikation für den Lizenzsystem-Rollout ausreichend vorbereitet; weitere Kommunikationsarbeit läuft bei Bedarf separat.
- [x] Kernbotschaften für Kunden final formuliert bzw. in App-/Vault-Doku abgebildet.
- [x] Support-/FAQ-Hinweise ausreichend abgestimmt.
- [x] Kommunikationskanäle und Veröffentlichungszeitpunkt für den Rollout festgelegt.
- [x] Interne Hinweise für Support und Vertrieb ausreichend vorbereitet.

## Abschlusskriterien

- App-seitige Lizenzsystem-Einführung ist abgeschlossen und durch Tests belegt.
- Rabattcode-Verhalten ist zwischen UI, Backend, Stripe und Doku konsistent und in Staging/Stripe-Testmode verifiziert.
- Lokale oder Staging-Stripe-Webhooks sind mit gültiger Signatur erfolgreich verifiziert oder eine bewusst akzeptierte Abweichung ist dokumentiert.
- UI-Tests für die zentralen Lizenzstatus und Kündigungs-/Checkout-Zustände sind ergänzt oder bewusst als manuelle Abnahme dokumentiert.
- Prod-Migration ist vorbereitet, angewendet und geprüft.
- Stripe-Live-Konfiguration ist vollständig hinterlegt und getestet.
- Bestandskundenmigration ist geplant, durchgeführt und abgeglichen.
- Launch-Kommunikation ist vorbereitet.
- Dieses Todo ist auf `erledigt` gesetzt und nach `abgeschlossene Todos/` verschoben.

## Notizen / Fortschritt

- 2026-04-27: Bisherige Einzel-Todos `T003` bis `T011` in diese Großaufgabe zusammengeführt.
- 2026-04-27: Inhalte aus den früheren Lizenzsystem-Dateien in dieses Todo konsolidiert, damit die Ordnerstruktur schlank bleiben kann.
- 2026-05-22: Finding 4 aus dem Webapp-Audit umgesetzt: Deployment-Env-Beispiele auf das Lizenz-Pool-Modell mit vier Stripe-Price-IDs, Product-IDs, Coupon-IDs und explizitem `APP_ENV` aktualisiert; `deploy/README.md` um die Price-ID-Matrix ergänzt.
- 2026-05-22: T013 gegen aktuellen App-Stand abgeglichen. Ergebnis: Lizenzsystem ist app-seitig eingeführt und zentrale Backend-/Frontend-Prüfungen sind grün; offen bleiben echte Stripe-/Staging-Verifikation, Prod-/Live-Konfiguration, Bestandskundenmigration und Launch-Kommunikation.
- 2026-05-22: Rabattcodes für direkte Pool-Erweiterungen umgesetzt: `licenses_v2.py` übergibt validierte Promotion-Codes nun auch bei `Subscription.modify`; Regressionstest `test_direct_activation_passes_promotion_code_to_subscription_update` ergänzt. Relevante Backend-Tests: 104/104 grün.
- 2026-05-25: T013 abgeschlossen. Externe Rollout-Nachweise wurden vom Projektverantwortlichen bestätigt: Prod-Migration, Live-Mode Stripe Products/Prices/Keys, produktiver Testkauf bzw. Dry-Run, echte Stripe-Signatur/Webhook-Verarbeitung und Promotion-Code-Verhalten. Verbleibende Detailthemen sind nicht mehr T013-blockierend und werden bei Bedarf über separate Todos geführt.
