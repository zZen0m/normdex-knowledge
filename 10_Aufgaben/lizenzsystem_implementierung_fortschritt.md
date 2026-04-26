# Lizenzsystem-Umbau – Implementierungsfortschritt

> **Zweck dieses Dokuments:** Vollständiges Arbeitsprotokoll für die Umstellung des Normdex-Lizenzsystems. Auch nach einem Kontextreset vollständig wiederherstellbar. Wird laufend aktualisiert.
>
> **Vollständige Spec:** [[normdex_lizenzsystem_developer_spec]]
> **Aufgabenliste:** [[Aufgaben]]

---

## Stand: 2026-04-26 (aktualisiert)

**Aktuell aktive Phase:** Phase 7 – Rollout (Phase 1–6 abgeschlossen)

---

## Eckdaten & Entscheidungen

### Datenbank
- **Dev:** SQLite (`apps/api/dev.db`)
- **Prod:** PostgreSQL in Docker (`deploy/docker-compose.prod.yml`)
- **Migration-Skill:** `.claude/skills/db_migration/SKILL.md` — muss vor jeder Migration gelesen werden
- Models verwenden `from sqlalchemy.dialects.postgresql import UUID` — Achtung: SQLite-Kompatibilität muss berücksichtigt werden (IDs werden als `String` gespeichert, nicht als nativer UUID-Typ → aktuell kein Problem da `id = Column(String, ...)`)

### Bestands-Daten
- **Licenses:** Können gelöscht werden (keine Echtkunden-Licenses vorhanden, nur Testdaten)
- **LicenseUsages:** Können gelöscht werden (hängen an Licenses)
- **Projekte/Organizations/Users:** MÜSSEN erhalten bleiben — sind Testdaten mit wichtigen Validierungsparametern
- **Stripe-Subscriptions:** Können in Dev und Prod gelöscht werden

### Berechtigungen
- Nur Admin/Owner darf Lizenzen kaufen und kündigen
- Members dürfen nur lesen

### Preise (neu)
| Stripe Price | Betrag | Intervall |
|---|---|---|
| `economics_basic_monthly_base` | 49,00 € | monatlich |
| `economics_basic_monthly_addon` | 29,00 € | monatlich |
| `economics_basic_yearly_base` | 490,00 € | jährlich |
| `economics_basic_yearly_addon` | 290,00 € | jährlich |

Stripe Prices + Products **angelegt** (Sandbox / Test-Mode) — siehe Phase 2.

---

## Ist-Zustand Codebase (Stand 2026-04-26)

### Backend

| Datei | Beschreibung |
|---|---|
| `apps/api/app/models.py` Z. 180–213 | `License` + `LicenseUsage` (altes Schema) |
| `apps/api/app/routers/licenses.py` (265 Z.) | Start/End/Heartbeat-Usage — bleibt erhalten |
| `apps/api/app/routers/subscriptions.py` (432 Z.) | Stripe Checkout + Webhooks — wird umgebaut |
| `apps/api/app/constants.py` | Product-Keys + Timeouts |

### Aktuelles `License`-Modell (altes Schema — wird ersetzt)
```python
id = Column(String, primary_key=True, ...)
organization_id = Column(ForeignKey("organizations.id"), ...)
product_key = Column(String(100), ...)       # z.B. "economics_v1"
billing_period = Column(String(50), ...)     # "monthly" | "yearly"
status = Column(String(50), ...)             # "active" | "canceled" | "expired"
valid_from = Column(DateTime, ...)
valid_until = Column(DateTime, ...)
max_concurrent_users = Column(Integer, ...)  # ENTFÄLLT
stripe_customer_id = Column(String(255), ...)
stripe_subscription_id = Column(String(255), ...)
cancel_at_period_end = Column(Boolean, ...)  # ENTFÄLLT
created_at = Column(DateTime, ...)
```

### Frontend

| Datei | Beschreibung |
|---|---|
| `apps/frontend/src/pages/Licenses.tsx` (535 Z.) | Wird komplett neu gebaut |
| `apps/frontend/src/pages/SubscriptionPlans.tsx` (271 Z.) | Wird durch Kaufdialog ersetzt |
| `apps/frontend/src/hooks/useLicenseLock.ts` (103 Z.) | Bleibt erhalten |
| `apps/frontend/src/api.ts` Z. 234–255 | Neue Endpoints ergänzen |

---

## Ziel-Schema (Soll)

### Tabelle `licenses` (neues Schema)
```python
id                         String PK (UUID)
organization_id            FK → organizations.id
product_key                String(100)          # "economics_v1"
billing_pool               String(50)           # "monthly" | "yearly"
license_kind               String(50)           # "base" | "addon"
status                     String(50)           # "pending" | "active" | "scheduled_end" | "ended" | "payment_failed"
started_at                 DateTime
current_term_start         DateTime
current_term_end           DateTime
committed_until            DateTime             # Mindestlaufzeitende
cancel_requested_at        DateTime nullable
scheduled_end_at           DateTime nullable
price_amount_gross         Float                # z.B. 49.0
currency                   String(10)           # "EUR"
stripe_subscription_id     String(255) nullable # Pool-Subscription
stripe_subscription_item_id String(255) nullable
stripe_price_id            String(255) nullable
assigned_user_id           FK → users.id nullable
created_by_user_id         FK → users.id nullable
meta                       JSON nullable
created_at                 DateTime
```

### Neue Tabelle `license_orders`
```python
id                         String PK (UUID)
organization_id            FK → organizations.id
created_by_user_id         FK → users.id
promotion_code             String nullable
subtotal_gross             Float
discount_total_gross       Float
total_gross                Float
currency                   String(10)
status                     String(50)   # "pending" | "completed" | "failed"
stripe_checkout_session_id String nullable
created_at                 DateTime
```

### Neue Tabelle `license_order_items`
```python
id                 String PK (UUID)
license_order_id   FK → license_orders.id
billing_pool       String(50)   # "monthly" | "yearly"
license_kind       String(50)   # "base" | "addon"
quantity           Integer
unit_price_gross   Float
line_total_gross   Float
```

### Neue Tabelle `license_events`
```python
id           String PK (UUID)
license_id   FK → licenses.id
event_type   String(100)  # "license.created", "license.activated", "license.cancel_requested", etc.
actor_user_id FK → users.id nullable
meta         JSON nullable
created_at   DateTime
```

### Tabelle `license_usages` — bleibt unverändert
Weiterhin für Heartbeat / aktive Nutzung.

---

## Implementierungsphasen

### Phase 1 – Architektur & Datenmodell
**Status:** ✅ Abgeschlossen (2026-04-26)

**Schritte:**
- [x] 1a. `License`-Modell in `models.py` auf neues Schema umschreiben
- [x] 1b. Neue Modelle `LicenseOrder`, `LicenseOrderItem`, `LicenseEvent` in `models.py` hinzufügen
- [x] 1c. `Organization`-Modell um `stripe_customer_id` erweitern
- [x] 1d. Alembic-Migration `a0dcbd3fd4b5_new_license_pool_model.py` erstellt
  - Besonderheit: Autogenerierte Migration verwendete `op.drop_constraint()` → SQLite-Fehler
  - Lösung: Migration manuell umgeschrieben mit `op.batch_alter_table()` (SQLite-kompatibel)
  - `server_default='monthly'`/`'base'` für NOT NULL-Spalten (SQLite-Pflicht)
  - DELETE-Statements am Anfang: bestehende `license_usages` + `licenses` geleert
- [x] 1e. Migration auf Dev angewendet (erfolgreich, 84 Routes, App startet)
- [ ] 1f. Migration auf Prod anwenden (noch offen — nach Phase 2 oder nach weiteren Tests)

**Nebenarbeiten (während Phase 1 erledigt):**
- [x] `licenses.py` Router: `LicenseSchema` auf neue Felder aktualisiert, start-usage-Logik angepasst (`max_concurrent_users` → 1 pro Lizenz, `valid_until` → `scheduled_end_at` + Status)
- [x] `dashboard.py`: `max_concurrent_users` → `len(active_licenses)`, `valid_until` → `current_term_end`
- [x] `stats.py`: `max_concurrent_users` → `license_count`
- ⚠️ `subscriptions.py` enthält noch viele alte Felder (billing_period, valid_until, max_concurrent_users, stripe_customer_id auf License) — wird in Phase 3 komplett neu gebaut, keine Breaking-Importe, aber Endpunkte funktionieren nicht korrekt

**Hinweise für spätere Kontextwiederherstellung:**
- `billing_period` → wurde zu `billing_pool` (gleiche Werte: "monthly"/"yearly")
- `valid_from`/`valid_until` → wurden zu `started_at`, `current_term_start`, `current_term_end`
- `stripe_customer_id` wechselte von `licenses` auf `organizations`
- Alte Felder `max_concurrent_users` und `cancel_at_period_end` entfallen

---

### Phase 2 – Stripe-Setup
**Status:** ✅ Abgeschlossen (2026-04-26)

**Stripe-Objekte (Sandbox / Test-Mode):**

| Produkt | Produkt-ID | Price | Price-ID |
|---|---|---|---|
| ÖNORM M 7140 Basic – Hauptlizenz | `prod_UPCq9bTtJyg63C` | 49 €/mo | `price_1TQOQMF05ipkEAzmY6FltbRW` |
| ÖNORM M 7140 Basic – Hauptlizenz | `prod_UPCq9bTtJyg63C` | 490 €/yr | `price_1TQOQMF05ipkEAzmYkug1hWf` |
| ÖNORM M 7140 Basic – Zusatzlizenz | `prod_UPCvOriaUUME9K` | 29 €/mo | `price_1TQOViF05ipkEAzmsbH2I1is` |
| ÖNORM M 7140 Basic – Zusatzlizenz | `prod_UPCvOriaUUME9K` | 290 €/yr | `price_1TQOflF05ipkEAzmL2zqIb5t` |

**Schritte:**
- [x] 2a. Stripe Produkte + Prices im Dashboard angelegt (2 Produkte × 2 Prices)
- [x] 2b. Price-IDs + Product-IDs in `.env` eingetragen
- [x] 2c. `config.py` um neue Stripe-Settings erweitert (`STRIPE_PRICE_ID_BASIC_*`, `STRIPE_PRODUCT_ID_BASIC_*`)
- [x] 2d. `constants.py` mit Preiszahlen aktualisiert (`LICENSE_PRICE_BASIC_*`)
- [ ] 2e. Prod-Keys anlegen (erst nach Go-Live — Live-Mode in Stripe)
- [ ] 2f. Hilfsfunktion `get_or_create_pool_subscription(org_id, pool)` → Phase 3

---

### Phase 3 – Backend
**Status:** ✅ Abgeschlossen (2026-04-26)

**Schritte:**
- [x] 3a. Domänenlogik in `app/domain/license_pricing.py`:
  - `calculate_preview(monthly_qty, yearly_qty, existing_monthly, existing_yearly)` → `list[PriceLineItem]`
  - `determine_cancellation_candidate(licenses)` → `License | None` (Add-on zuerst, dann Base)
  - `get_unit_price(pool, kind)` → `float`
- [x] 3b. Neue Router-Datei `app/routers/licenses_v2.py`:
  - `GET /licenses/pools` — Pool-Zusammenfassung mit Kündigungskandidat
  - `GET /licenses/pools/{pool}/items` — Einzellizenzen des Pools
  - `GET /licenses/{id}/history` — Events (neueste zuerst)
  - `POST /licenses/checkout/preview` — Live-Preiskalkulation (Admin only)
  - `POST /licenses/checkout/create` — Bestellung + Stripe Checkout Session (oder direkte Aktivierung wenn Pool-Subscription bereits existiert)
  - `POST /licenses/checkout/confirm` — Aktivierung nach Stripe-Redirect
  - `POST /licenses/{id}/cancel` — Admin-only Kündigung (Add-on-Priorisierung enforced)
  - `POST /licenses/{id}/assign-user` / `unassign-user`
  - `POST /licenses/promotions/validate` — Stripe Promotion Code prüfen
  - `POST /admin/licenses/grant-complimentary` — System-Admin: kostenlose Lizenz vergeben
- [x] 3c. `_require_org_admin()` Helper für alle schreibenden Endpoints
- [x] 3d. Kündigungslogik: Add-on zuerst; Base nur wenn keine Add-ons aktiv; Status → `scheduled_end`; `scheduled_end_at` = max(committed_until, current_term_end)
- [ ] 3e. Rebasierungslogik: wenn Base endet und Add-ons verbleiben → ältestes Add-on wird Base (**noch offen — wird als Hintergrundtask in Phase 4 implementiert**)
- [x] 3f. Bestehende Endpoints in `licenses.py` prüfen — OK (Phase 1 bereits angepasst, 33/33 Tests grün)

**Architektur-Entscheidungen:**
- `checkout/create` unterscheidet: Pool-Subscription bereits vorhanden → direkte Aktivierung via `Subscription.modify` + `proration_behavior=always_invoice`; noch keine Pool-Subscription → Stripe Checkout Session (Redirect)
- Gemischte Orders (monthly + yearly): je Pool eigene Checkout Session; Session-IDs in `order.meta.checkout_sessions`; primary Session in `stripe_checkout_session_id`
- Jede Lizenz erhält `meta.license_order_id` für spätere Zuordnung bei `checkout/confirm`
- `stripe_customer_id` wird beim ersten Checkout auf `Organization` gespeichert
- Rebasierung (3e) wurde bewusst auf Phase 4 verschoben da sie im Webhook-Kontext natürlicher ist

---

### Phase 4 – Webhooks & Synchronisierung
**Status:** ✅ Abgeschlossen (2026-04-26)

**Schritte:**
- [x] 4a. `subscriptions.py` vollständig neu gebaut:
  - `checkout.session.completed` → `license_order` bestätigen, Lizenzen auf `active`, `stripe_subscription_id` + `stripe_subscription_item_id` + `stripe_price_id` speichern, `license_events` schreiben
  - `customer.subscription.updated` → `current_term_start`/`current_term_end` syncen; `scheduled_end` → `ended` wenn neuer Zeitraum past `scheduled_end_at`; Rebasierungslogik ausführen; Stripe-Status `past_due`/`unpaid` → `payment_failed`
  - `customer.subscription.deleted` → alle Pool-Lizenzen (active/scheduled_end/pending/payment_failed) auf `ended`
  - `invoice.paid` → Event dokumentieren + `payment_failed` Lizenzen reaktivieren
  - `invoice.payment_failed` → aktive Pool-Lizenzen auf `payment_failed`
- [x] 4b. Idempotenz: `checkout.session.completed` prüft `order.status == "completed"` → Skip; `stripe_checkout_session_id` als Primary Key für Lookup
- [x] 4c. Rebasierungslogik implementiert: `_rebase_addons_if_needed(db, org_id, pool)` — wenn keine aktive Base mehr vorhanden → ältestes aktives Add-on wird Base; schreibt `license.rebased` Event; aufgerufen bei `subscription.updated` und `subscription.deleted`
- [x] 4d. Cancel-Endpoint in `licenses_v2.py` um Stripe-Seiteneffekt erweitert: `_cancel_stripe_item()` — reduziert Subscription-Item-Menge (`proration_behavior='none'`), löscht Item wenn letzte Art, setzt `cancel_at_period_end=True` wenn letzte Lizenz im Pool
- [x] 4e. Alte broken Endpoints aus `subscriptions.py` entfernt (`create-checkout-session`, `update-seats`, `cancel`, `reactivate` — ersetzen durch `licenses_v2.py`)
- [ ] 4f. Lokal mit Stripe-CLI testen (Stripe Webhook-Forwarding) — noch offen

**Architektur-Entscheidungen:**
- Webhook-Handler sind synchrone Funktionen (kein `async`) — vereinfacht DB-Zugriff ohne Event-Loop-Konflikte
- Alle Handler: try/except commit + rollback + log_system_error → Webhook gibt immer 200 zurück (Stripe retries bei 4xx/5xx)
- Rebasierung ist DB-only (kein Stripe-Aufruf) — `license_kind` wechselt, Subscription-Items bleiben unverändert
- `_cancel_stripe_item()`: Fehler werden geloggt, aber nie weitergeworfen (DB-Änderung bereits committed)

---

### Phase 5 – Frontend
**Status:** ✅ Abgeschlossen (2026-04-26)

**Schritte:**
- [x] 5a. `Licenses.tsx` komplett neu geschrieben:
  - `PoolSection`-Komponente für monatliche + jährliche Lizenzen
  - `LicenseCard` pro Lizenz: Kind-Badge, Status-Badge, Preis, Start/Enddatum, aktive Session, Action-Dropdown
  - Status-Badges: Aktiv (grün), Gekündigt zum DD.MM.YYYY (orange), Beendet, Zahlung fehlgeschlagen (rot), Ausstehend
  - Aktive Session: grüner Punkt + Avatar + Name + „Aktiv seit HH:MM"; ohne Session: „Keine aktive Sitzung"
  - Pool-Header mit "Nächste Abrechnung"-Datum
  - Leerzustand mit CTA für Owner
  - Stripe Checkout redirect: `?session_id=` beim Mount erkannt → `checkout/confirm` aufgerufen, URL bereinigt
  - Owner/Admin: Kaufen + Kündigen + Sitzungen bereinigen; Member: nur Lesen
- [x] 5b. `BuyDialog` (Modal in Licenses.tsx):
  - Mengenregler monatlich + jährlich (Chevron-Buttons)
  - Live-Preisvorschau via `checkout/preview` (debounced 400 ms)
  - Preisanzeige: aufgeschlüsselt nach Art + Pool + Rabatt
  - Rabattcode-Feld (uppercase)
  - Hinweis bei gemischten Pools (getrennte Stripe-Checkouts)
  - Direktaktivierung bei bestehendem Pool; Redirect bei neuem Pool
- [x] 5c. `CancelDialog` (Modal in Licenses.tsx):
  - Zeigt Lizenzart + Enddatum prominent
  - Hinweis: Zugriff bleibt bis Enddatum erhalten
- [x] 5d. `AssignUserDialog` (Code in Licenses.tsx vorhanden, aber nicht in der UI exponiert — statische Benutzerzuweisung bewusst ausgeblendet)
- [x] 5e. `api.ts` um alle licenses_v2 Endpoints erweitert; veraltete (subscriptionCheckout, subscriptionUpdateSeats, subscriptionCancel, subscriptionReactivate) entfernt
- [x] 5f. `SubscriptionPlans.tsx` → redirect `<Navigate to="/licenses" replace />` (Kaufen ist jetzt in Licenses integriert)
- [x] TypeScript-Compile: 0 Fehler

---

### Phase 6 – Tests
**Status:** ✅ Abgeschlossen (2026-04-26) — 91/91 Tests grün (vorher 33)

**Neue Testdateien:**

| Datei | Tests | Inhalt |
|---|---|---|
| `tests/test_license_pricing.py` | 21 | `calculate_preview` (11), `determine_cancellation_candidate` (10) |
| `tests/test_license_webhooks.py` | 37 | checkout.completed (6), subscription.updated (7), subscription.deleted (4), invoice.paid (5), invoice.payment_failed (5), `_rebase_addons_if_needed` (8), fixture-Tests (2) |

**Abgedeckte Szenarien:**
- [x] Unit-Tests `calculate_preview`: leerer Pool, buy-1, buy-N, gemischte Pools, bestehende Pools, `line_total_gross`
- [x] Unit-Tests Kündigungsreihenfolge: kein aktiver, nur Base, Base+Addon → Addon, neuestes Addon, `None`-Edge-Case
- [x] Integrationstests Webhook-Flow: Aktivierung, Idempotenz, Term-Sync, `scheduled_end → ended`, `past_due/unpaid → payment_failed`, Rebase-Trigger
- [x] Tests für Rebasierung: ältestes Addon befördert, Pool-/Org-Isolation, `ended` nicht befördert
- [ ] Tests für Promotions (validate + Discount-Berechnung) — noch offen
- [ ] Tests für Complimentary-Lizenzen (`/admin/licenses/grant-complimentary`) — noch offen

**Bugfix während Phase 6 entdeckt und behoben:**
- `_rebase_addons_if_needed` in `subscriptions.py` + `db.flush()` zu Beginn: stellt sicher, dass In-Memory-Statusänderungen (z.B. `active → ended`) in der DB sichtbar sind, bevor die Funktion nach aktiven Basislizenzen sucht (relevant wenn `autoflush=False`)
- Dev-Test 2026-04-26: Kauf einer monatlichen Basislizenz über die neue Lizenzverwaltung erfolgreich bis Stripe Checkout. Stripe-Events kamen lokal an, wurden aber vom Webhook wegen ungültiger Signatur mit `400` abgelehnt. Aktivierung wurde über `/licenses/checkout/confirm` nachgezogen.
- Bugfix 2026-04-26: Stripe Python 14 kompatibel gemacht. `subscription.items` kollidiert mit der Dict-Methode `items()` und muss über `subscription.get("items")` gelesen werden. Zusätzlich liegen `current_period_start`/`current_period_end` in der aktuellen Stripe-API am Subscription-Item; Confirm/Webhook nutzen nun die Item-Perioden als Fallback.
- Bugfix 2026-04-26: Abgebrochene Stripe-Checkouts werden explizit bereinigt. Die `cancel_url` enthält die `order_id`, das Frontend ruft `/licenses/checkout/cancel` auf, Pending-Lizenzen werden auf `ended` gesetzt und die Bestellung auf `failed`. Pending-Lizenzen zählen nicht mehr in die Preislogik hinein; dadurch kann ein abgebrochener Basiskauf keine spätere Zusatzlizenz zum falschen Preis auslösen. Falls dadurch ein Pool ohne aktive Basislizenz bleibt, wird die älteste aktive Zusatzlizenz zur Basislizenz rebased.
- Ergebnis Dev-DB 2026-04-26: `LicenseOrder 2eed95d0-6b44-4248-a890-deee7b3f45a7` steht auf `completed`; Lizenz `f56b4ea4-e925-420c-992e-8a75ffcd2c21` ist `active`, `monthly`, `base`, Laufzeit `2026-04-26 16:22:14` bis `2026-05-26 16:22:14`.
- Verifikation 2026-04-26: `py_compile` für `app/routers/licenses_v2.py` und `app/routers/subscriptions.py` erfolgreich. Pytest konnte im aktuellen `.venv` nicht ausgeführt werden, weil `pytest` nicht installiert ist.

---

### Post-Phase-6 – Bugfixes & Verbesserungen
**Status:** ✅ Abgeschlossen (2026-04-26)

**Bugfixes (gefunden bei erstem Live-Test):**

1. **`cancel_at_period_end` fehlte im ORM-Modell** (`models.py`): Das Feld war in der Alembic-Migration `a1b2c3d4e5f6` vorhanden, aber nicht in `License`. Gelöst mit direktem `ALTER TABLE` (Dev-DB dachte bereits auf Head zu sein). Spalte nachträglich ergänzt.
2. **`_get_pool_sub_id` ignorierte `payment_failed`-Lizenzen** (`licenses_v2.py`): Status-Filter ergänzt um `"payment_failed"` — verhindert doppelte Stripe-Subscriptions bei Zahlungsproblemen.
3. **`_cancel_stripe_item` persistierte `cancel_at_period_end` nicht lokal** (`licenses_v2.py`): `lic.cancel_at_period_end = True` fehlte — DB-Feld blieb auf `False`.
4. **Unsafe Fallback in `checkout_confirm`** (`licenses_v2.py`): Hardcodierter `else`-Zweig lieferte stillschweigend die falsche Price-ID. Ersetzt durch `_get_stripe_price_id()` mit explizitem `ValueError`.
5. **Webhook-Race bei `status == "failed"`** (`subscriptions.py`): `checkout.session.completed` prüfte nur auf `"completed"`, nicht auf `"failed"` → gecancelte Orders konnten erneut verarbeitet werden. Filter auf `("completed", "failed")` erweitert.
6. **Spurious `license.term_renewed` Event bei `past_due`** (`subscriptions.py`): `customer.subscription.updated` schrieb fälschlicherweise ein `term_renewed`-Event, bevor es den Status auf `payment_failed` setzte. Handler-Logik umstrukturiert: Payment-Fehler hat Vorrang.
7. **Falscher 409-String in `useLicenseLock`** (Frontend): Hook prüfte auf englische Fehlermeldung, API sendet deutsch. Korrigiert auf `e.status === 409 || e.message.includes("wird derzeit von")`.

**Tests nach Bugfixes:** 2 neue Tests in `test_license_webhooks.py` hinzugefügt → **60/60 Tests grün**.

---

**Bugfix – `useLicenseLock` wählt immer erste Lizenz:**
- `useLicenseLock.ts` verwendete `.find()` → immer die erste aktive Lizenz (Hauptlizenz) wurde gesperrt.
- Wenn die Hauptlizenz bereits belegt war, bekam ein zweiter Nutzer sofort `409 Zugriff verweigert`, obwohl weitere freie Lizenzen vorhanden waren.
- **Fix:** `.find()` durch `.filter()` + sequentielle Schleife ersetzt. Jede Lizenz wird der Reihe nach probiert; bei 409 wird zur nächsten gesprungen. Nur wenn alle Kandidaten belegt sind, erscheint die Fehlermeldung „Alle Lizenzen sind derzeit in Verwendung."

**Feature – Bestätigungsdialog vor Lizenzkauf rückgängig machen:**
- Button „Rückgängig" im Amber-Banner führte bisher die Aktion sofort aus.
- Neues Verhalten: Klick öffnet einen Bestätigungsdialog mit dem Hinweis: *„Diese Lizenz wird mit sofortiger Wirkung vollständig entfernt. Laufende Sitzungen werden beendet und die Lizenz steht danach nicht mehr zur Verfügung."*
- Dialog hat zwei Buttons: **Abbrechen** (schließt Dialog) und **Lizenz entfernen** (destructive, führt Aktion aus).
- Implementierung: neuer State `undoConfirmLic`, Handler `handleUndoPurchaseRequest` als Prop statt direktem `handleUndoPurchase`, `<Dialog>` am Ende der Page.

**Verbesserungen – Aktive Sessions in Lizenzverwaltung:**
- `GET /licenses/pools/{pool}/items` gibt jetzt `active_usages` zurück (inkl. User-Daten + Startuhrzeit). Cleanup expired sessions läuft beim Aufruf mit.
- `LicenseCard` in `Licenses.tsx` zeigt aktive Session: grüner Punkt + Avatar + Name + „Aktiv seit HH:MM". Ohne Session: „Keine aktive Sitzung".
- Statische Benutzerzuweisung (`assigned_user`) in der UI ausgeblendet: „Benutzer zuweisen"-Eintrag aus dem Aktionen-Dropdown entfernt. Backend-Endpoints bleiben vorhanden.

**Session-Timeout:**
- Geändert von 30 Minuten auf **7 Minuten** (`constants.py: LICENSE_TIMEOUT_MINUTES = 7`).
- Heartbeat-Intervall bleibt 2 Minuten → 3 Versuche vor Timeout (ausreichend Puffer für kurze Netzwerkaussetzer).
- Begründung: Kürzerer Timeout gibt abgebrochene Slots schneller frei; bei offenem Tab kein Nachteil, da Heartbeat kontinuierlich läuft.

---

### Phase 7 – Rollout
**Status:** 🔲 Offen

**Voraussetzungen (müssen davor erledigt sein, siehe „Offene Punkte" unten):**
- [x] `LicenseOrder.meta`-Feld (Modell + Migration)
- [~] Stripe-CLI-Test lokal (4f) — Checkout/Confirm erfolgreich, Webhook-Forwarding noch mit Signaturfehler `400`
- [ ] Prod-Migration (1f)
- [ ] Prod-Stripe-Keys (Live-Mode)

**Rollout-Schritte:**
- [x] Interne Dokumentation aktualisieren
- [x] Support-/FAQ-Texte ergänzen (Lizenzpool, Kündigung, Zuweisung)
- [ ] Launch-Kommunikation vorbereiten

---

## Protokoll erledigter Schritte

| Datum | Phase | Schritt | Ergebnis |
|---|---|---|---|
| 2026-04-26 | 1 | DB-Backup `dev.db.backup_20260426_074159` | OK |
| 2026-04-26 | 1 | `models.py` — License, LicenseOrder, LicenseOrderItem, LicenseEvent, Organization | OK |
| 2026-04-26 | 1 | Alembic-Migration generiert | OK (autogeneriert) |
| 2026-04-26 | 1 | Migration anwenden | FEHLER: `op.drop_constraint` nicht SQLite-kompatibel → manuell auf `op.batch_alter_table` umgestellt |
| 2026-04-26 | 1 | Migration erneut anwenden | OK — Schema korrekt, 7 Orgs + 5 Projekte erhalten, Lizenzen geleert |
| 2026-04-26 | 1 | App-Start-Test | OK — 84 Routes geladen |
| 2026-04-26 | 1 | licenses.py, dashboard.py, stats.py angepasst | OK — alte Felder entfernt |
| 2026-04-26 | 1 | test_licenses.py fixtures angepasst (max_concurrent_users entfernt) | OK |
| 2026-04-26 | 1 | Alle 33 Tests ausgeführt | ✅ 33/33 bestanden — nur Deprecation-Warnings |
| 2026-04-26 | 2 | Stripe Produkte + Prices angelegt (Sandbox) | OK |
| 2026-04-26 | 2 | config.py, constants.py, .env, .env.example aktualisiert | OK — alle 4 Price-IDs korrekt geladen |
| 2026-04-26 | 3 | `app/domain/license_pricing.py` erstellt | OK — PriceLineItem, calculate_preview, determine_cancellation_candidate |
| 2026-04-26 | 3 | `app/routers/licenses_v2.py` erstellt (10 Endpoints) | OK |
| 2026-04-26 | 3 | `main.py` aktualisiert (licenses_v2.router + admin_router registriert) | OK |
| 2026-04-26 | 3 | Alle 33 Tests ausgeführt | ✅ 33/33 bestanden |
| 2026-04-26 | 4 | `subscriptions.py` vollständig neu gebaut (5 Webhook-Handler + Rebasierung + Idempotenz) | OK |
| 2026-04-26 | 4 | `licenses_v2.py` cancel-Endpoint um `_cancel_stripe_item()` erweitert | OK |
| 2026-04-26 | 4 | Alle 33 Tests ausgeführt | ✅ 33/33 bestanden |
| 2026-04-26 | 5 | `Licenses.tsx` komplett neu gebaut (PoolSection, LicenseCard, BuyDialog, CancelDialog, AssignUserDialog) | OK |
| 2026-04-26 | 5 | `api.ts` licenses_v2 Endpoints ergänzt, veraltete entfernt | OK |
| 2026-04-26 | 5 | `SubscriptionPlans.tsx` → redirect zu /licenses | OK |
| 2026-04-26 | 5 | TypeScript-Compile 0 Fehler, Backend 33/33 Tests grün | ✅ |
| 2026-04-26 | 6 | `tests/test_license_pricing.py` erstellt (21 Unit-Tests) | OK |
| 2026-04-26 | 6 | `tests/test_license_webhooks.py` erstellt (37 Integrationstests) | OK |
| 2026-04-26 | 6 | Bugfix: `db.flush()` am Beginn von `_rebase_addons_if_needed` | OK |
| 2026-04-26 | 6 | Gesamte Test-Suite ausgeführt | ✅ 91/91 bestanden |
| 2026-04-26 | — | Bugfix `LicenseOrder.meta`: Modell + Migration `342d48e1f270` auf Dev angewendet | ✅ 91/91 Tests grün |
| 2026-04-26 | Dev-Test | Lizenzkauf in neuer Lizenzverwaltung getestet; Checkout/Confirm aktiviert monatliche Basislizenz | OK — Webhook-Signatur lokal noch offen |
| 2026-04-26 | Bugfix | Stripe Python 14: Subscription-Items und Periodenfelder robust gelesen | OK — `py_compile` grün |
| 2026-04-26 | Bugfix | Abgebrochene Stripe-Checkouts bereinigen Pending-Bestellungen/Lizenzen und beeinflussen die Preislogik nicht mehr | OK — Backend `py_compile`, Frontend `npm run build` grün |
| 2026-04-26 | Bugfix | 6 Bugs behoben: `cancel_at_period_end` im ORM, `payment_failed` in Pool-Sub-Query, Stripe-Item-Persistenz, unsafe Checkout-Fallback, Webhook-Race bei `failed` Orders, spurious `term_renewed` bei `past_due` | ✅ 60/60 Tests grün |
| 2026-04-26 | Bugfix | `useLicenseLock` 409-Fehlererkennung korrigiert (deutsche Fehlermeldung) | OK |
| 2026-04-26 | Feature | `GET /licenses/pools/{pool}/items` gibt `active_usages` mit User-Daten zurück; Cleanup expired sessions integriert | OK |
| 2026-04-26 | Feature | `LicenseCard` zeigt aktive Session (Avatar, Name, Uhrzeit) statt statischer Benutzerzuweisung | OK — TypeScript 0 Fehler |
| 2026-04-26 | Feature | Statische Benutzerzuweisung aus UI entfernt (Backend bleibt erhalten) | OK |
| 2026-04-26 | Konfig | `LICENSE_TIMEOUT_MINUTES` von 30 auf 7 Minuten gesetzt | OK |
| 2026-04-26 | Bugfix | `useLicenseLock`: `.find()` → `.filter()` + Schleife; überspringt 409 und probiert alle freien Lizenzen | OK |
| 2026-04-26 | Feature | `Licenses.tsx`: Bestätigungsdialog vor „Kauf rückgängig machen" (sofortige Entfernung + Bestätigung erforderlich) | OK |
| 2026-04-26 | Doku | Aufgabenliste synchronisiert; Vault-Doku und App-Hilfe zu Lizenzpool, Kündigung, Abrechnung und Stripe-Checkout-only-Rabattcodes aktualisiert | OK |
| 2026-04-26 | UX | Kündigungsdialog expliziter gemacht: Frage „Lizenz wirklich kündigen?", endgültiges Ablaufdatum und bestätigungspflichtiger Button | OK |
| 2026-04-26 | Feature | Kündigung zurückziehen implementiert: `POST /licenses/{id}/reactivate-cancel`, Stripe-Sync und Drei-Punkte-Menü für laufende `scheduled_end`-Lizenzen | OK — 67/67 Lizenztests grün, Frontend-Build grün |

---

## Bekannte Probleme / Stolpersteine

- `sqlalchemy.dialects.postgresql.UUID` wird in `models.py` importiert, aber IDs sind als `Column(String, ...)` definiert — kein Problem für SQLite
- ~~`LicenseOrder` fehlt `meta`-Feld~~ **✅ behoben (2026-04-26):** `meta = Column(JSON, nullable=True)` hinzugefügt, Migration `342d48e1f270_add_meta_to_license_orders` angewendet.
- `subscriptions.py` enthält sowohl Checkout- als auch Webhook-Logik und wurde durch den Umbau stark verändert — Stripe-CLI-Test teilweise erledigt: Events kommen an, lokale Signaturprüfung schlägt noch mit `Ungültige Stripe-Signatur` fehl.
- Aktuelle Stripe-API-Version liefert Abrechnungsperioden am Subscription-Item. Code darf nicht ausschließlich `subscription.current_period_start/current_period_end` verwenden.
- Rabattcodes sind aktuell bewusst Stripe-Checkout-only: Sie können eingegeben werden, wenn eine neue Stripe-Subscription über Checkout ausgelöst wird. Bei direkter Erweiterung einer bestehenden Pool-Subscription gibt es derzeit keinen separaten Normdex-Rabattcode-Flow.
- Kündigung zurückziehen ist umgesetzt. Die Rücknahme ist nur möglich, solange `scheduled_end_at` noch in der Zukunft liegt; nach Ablauf muss eine neue Lizenz gekauft werden.
