# T013 - Lizenzsystem-Rollout abschließen

**Status:** in Arbeit  
**Bereich:** App / Infrastruktur / Marketing  
**Erstellt:** 2026-04-27  
**Abgeschlossen:** -

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

### Abgeschlossen

- Neues Datenmodell für `licenses`, `license_orders`, `license_order_items`, `license_events`.
- Migrationen für das neue Lizenzmodell inklusive `LicenseOrder.meta`.
- SQLite-kompatible Migrationen über `op.batch_alter_table()`.
- `Organization.stripe_customer_id`.
- Domänenlogik für Preisberechnung und Kündigungskandidaten.
- Neue Lizenz-v2-Endpunkte.
- Admin-/Owner-Checks für schreibende Lizenzaktionen.
- Kündigungslogik: Add-on zuerst, Basis erst wenn zulässig.
- Stripe-Checkout für neue Pool-Subscriptions.
- Direkte Erweiterung bestehender Pool-Subscriptions über Stripe Subscription-Änderung mit anteiliger Verrechnung.
- Gemischte Orders monthly/yearly über getrennte Checkout-Sessions, gespeichert in `order.meta.checkout_sessions`.
- Webhook-Handler neu aufgebaut.
- Idempotenz bei `checkout.session.completed`.
- Rebasierungslogik: ältestes aktives Add-on wird Basis, wenn keine aktive Basis mehr vorhanden ist.
- Stripe-Seiteneffekt beim Kündigen: Subscription-Item-Menge reduzieren, Item löschen, `cancel_at_period_end=True` bei letzter Lizenz im Pool.
- Neue Lizenzverwaltung im Frontend.
- Kaufdialog, Kündigungsdialog und Rücknahme einer Kündigung.
- Statische Benutzerzuweisung ist zwar im Code vorhanden, aber in der UI ausgeblendet.
- `SubscriptionPlans.tsx` leitet auf `/licenses` um.
- App-Hilfe und interne Doku zu Lizenzpool, Kündigung, Abrechnung und Stripe-Checkout-only-Rabattcodes wurden aktualisiert.
- Abgebrochene Stripe-Checkouts werden bereinigt: Pending-Lizenzen werden `ended`, Order wird `failed`, Pending-Lizenzen zählen nicht in die Preislogik.
- `payment_failed` wird bei Pool-Subscription-Suche berücksichtigt, damit keine doppelten Stripe-Subscriptions entstehen.
- Kündigung zurückziehen ist umgesetzt: `POST /licenses/{id}/reactivate-cancel`, solange `scheduled_end_at` noch in der Zukunft liegt.

### Tests / Verifikation bisher

- Früherer Stand: 33/33 Tests grün.
- Nach Pricing/Webhook-Tests: 91/91 Tests grün.
- Nach späteren Bugfixes: 60/60 bzw. 67/67 relevante Lizenztests grün.
- Frontend-TypeScript-Compile war ohne Fehler.
- Frontend-Build war nach Checkout-Cancel-Bugfix grün.
- `py_compile` für `licenses_v2.py` und `subscriptions.py` war erfolgreich.
- Ein Dev-Test am 2026-04-26 kaufte eine monatliche Basislizenz über Stripe Checkout erfolgreich; Aktivierung wurde wegen lokalem Webhook-Signaturproblem über `/licenses/checkout/confirm` nachgezogen.

## Bekannte Probleme und Stolpersteine

- Lokales Stripe-Webhook-Forwarding empfing Events, aber Signaturprüfung schlug mit `400 Ungültige Stripe-Signatur` fehl.
- Rabattcodes sind aktuell bewusst Stripe-Checkout-only: Sie können bei neuer Stripe-Checkout-Subscription genutzt werden, aber nicht bei direkter Erweiterung einer bestehenden Pool-Subscription.
- Die Rücknahme einer Kündigung ist nur möglich, solange `scheduled_end_at` in der Zukunft liegt. Nach Ablauf muss eine neue Lizenz gekauft werden.
- Prod-Migration ist noch offen.
- Live-Mode Stripe Products/Prices/Keys sind noch offen.
- Für Migrationen gilt: vor Alembic-Aktionen im App-Repo den DB-Migration-Skill lesen.
- Dev-Testdaten: `Licenses` und `LicenseUsages` können gelöscht werden, aber `Projects`, `Organizations` und `Users` müssen erhalten bleiben.
- Stripe-Subscriptions können in Dev und Prod gelöscht werden, sofern bewusst im Rahmen der Migration entschieden.

## Offene To-do-Liste

### 1. Fachliche Entscheidung Rabattcodes

- [ ] Entscheiden, ob Rabattcodes bei direkter Erweiterung bestehender Pool-Subscriptions unterstützt werden.
- [ ] Falls nein: Einschränkung in UI, Supporttexten und interner Dokumentation klar dokumentieren.
- [ ] Falls ja: Backend-, Frontend-, Stripe- und Testumfang definieren und umsetzen.
- [ ] Sicherstellen, dass Staffelpreise immer Grundmodell bleiben und Rabattcodes nur zusätzlich auf den berechneten Warenkorb/Kaufpreis wirken.
- [ ] Zielgruppen berücksichtigen: Launch-Rabatt, private/limitierte Aktionen, Testuser/Pilotkunden, manuelle Sonderrabatte.

### 2. Lokale Stripe-Verifikation

- [ ] Stripe CLI lokal korrekt mit Webhook-Secret konfigurieren.
- [ ] `checkout.session.completed` lokal mit gültiger Signatur verarbeiten.
- [ ] `customer.subscription.updated` lokal testen.
- [ ] `invoice.paid` lokal testen.
- [ ] `invoice.payment_failed` lokal testen.
- [ ] Prüfen, ob `current_period_start`/`current_period_end` korrekt von Subscription-Items gelesen werden.
- [ ] Ergebnis direkt in diesem Todo unter „Notizen / Fortschritt“ dokumentieren.

### 3. Tests ergänzen

- [ ] UI-Tests für gekündigte Lizenzen ergänzen.
- [ ] UI-Tests für Statusunterscheidung `active`, `scheduled_end`, `ended`, `payment_failed`, `pending` ergänzen.
- [ ] Tests für Promotions: Validierung und Discount-Berechnung.
- [ ] Tests für Complimentary-Lizenzen über `/admin/licenses/grant-complimentary`.
- [ ] Tests für abgebrochene Checkout-Flows absichern, falls noch nicht vollständig abgedeckt.
- [ ] Relevante Backend- und Frontend-Testbefehle ausführen und Ergebnis dokumentieren.

### 4. Produktion und Infrastruktur

- [ ] Prod-Migration vorbereiten.
- [ ] Backup- und Rollback-Vorgehen dokumentieren.
- [ ] Migration auf Staging oder vergleichbarer Umgebung validieren.
- [ ] Prod-Migration nach Freigabe anwenden.
- [ ] Stripe-Live-Mode Products und Prices anlegen.
- [ ] Produktive Stripe-Keys sicher konfigurieren.
- [ ] App-Konfiguration auf korrekte Live-Price-IDs setzen.
- [ ] Produktiven Testkauf oder geeigneten Dry-Run dokumentieren.

### 5. Bestandskunden und Datenmigration

- [ ] Migration bestehender Kunden fachlich planen.
- [ ] Betroffene Kunden- und Lizenzdaten identifizieren.
- [ ] Zielzustand pro Kundentyp beschreiben.
- [ ] Entscheiden, welche bestehenden Lizenzen/Subscriptions gelöscht, übernommen oder neu aufgebaut werden.
- [ ] Migrationsreihenfolge, Verantwortlichkeiten und Prüfschritte dokumentieren.
- [ ] Bestandsdaten gemäß freigegebenem Migrationsplan überführen.
- [ ] Datenintegrität prüfen.
- [ ] Stripe- und Normdex-Zustand abgleichen.

### 6. Launch und Kommunikation

- [ ] Launch-Kommunikation vorbereiten.
- [ ] Kernbotschaften für Kunden formulieren.
- [ ] Support-/FAQ-Hinweise final abstimmen.
- [ ] Kommunikationskanäle und Veröffentlichungszeitpunkt festlegen.
- [ ] Interne Hinweise für Support und Vertrieb vorbereiten.

## Abschlusskriterien

- Alle offenen fachlichen Entscheidungen sind getroffen und dokumentiert.
- Lokale Stripe-Webhooks sind erfolgreich verifiziert oder eine bewusst akzeptierte Abweichung ist dokumentiert.
- Rabattcode-Strategie für neue Subscriptions und bestehende Pool-Erweiterungen ist final.
- UI-, Promotions- und Complimentary-Lizenztests sind ergänzt.
- Prod-Migration ist vorbereitet, angewendet und geprüft.
- Stripe-Live-Konfiguration ist vollständig hinterlegt und getestet.
- Bestandskundenmigration ist geplant, durchgeführt und abgeglichen.
- Launch-Kommunikation ist vorbereitet.
- Dieses Todo ist auf `erledigt` gesetzt und nach `abgeschlossene Todos/` verschoben.

## Notizen / Fortschritt

- 2026-04-27: Bisherige Einzel-Todos `T003` bis `T011` in diese Großaufgabe zusammengeführt.
- 2026-04-27: Inhalte aus den früheren Lizenzsystem-Dateien in dieses Todo konsolidiert, damit die Ordnerstruktur schlank bleiben kann.
