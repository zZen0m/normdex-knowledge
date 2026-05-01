# T020-06 · Stripe-Customer-Adress-Sync (kein Backfill)

**Phase:** 2 (Funktionale Lücke)
**Priorität:** P1 · risikobehaftet (Stripe-Datenfluss)
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Aktuell wird die `billing_address` der Organisation **nicht** an Stripe übertragen. Rechnungen werden mit Stripe-eigenen Daten erstellt, was zu inkorrekten Adressen auf Rechnungen führen kann. Es gibt zwei Felder: `address` (Unternehmen) und `billing_address` (Rechnung) — auf der Rechnung soll `billing_address` erscheinen. Bestandskunden werden bewusst **nicht** per Backfill aktualisiert; Sync greift erst bei nächster Adressänderung oder neuer Subscription.

## Betroffene Dateien
- `apps/api/app/routers/teams.py:73, 102-103` — PATCH `/teams/{org_id}`.
- `apps/api/app/routers/licenses_v2.py:428-431` — `stripe.Customer.create()`.
- Neuer Helper: `apps/api/app/services/stripe_helpers.py` (oder bestehend) — `org_billing_address_to_stripe_address(org)`.

## Umsetzung
1. Helper, der `org.billing_address` (JSON) → Stripe-`address`-Dict mappt (line1, line2, postal_code, city, state, country).
2. Bei Customer-Create `address`, `name` und `email` aus Org-Daten setzen.
3. Bei Org-PATCH (wenn `billing_address` oder `name` geändert): `stripe.Customer.modify(...)` aufrufen, sofern `stripe_customer_id` existiert.
4. Fehler weich behandeln (Stripe-Fehler nicht den DB-Update blockieren — log + Audit-Event).

## Akzeptanzkriterien
- [ ] Neuer Customer in Stripe hat `address` aus `billing_address` gesetzt.
- [ ] Adressänderung in App propagiert zu Stripe Dashboard.
- [ ] Nächste Stripe-Rechnung enthält die aktuelle `billing_address`.
- [ ] Stripe-Fehler verursachen keinen 500 im Org-PATCH.

## Verifikation
1. In Test-Org `billing_address` ändern → Stripe Dashboard zeigt aktualisierte Adresse.
2. Test-Subscription anlegen, Rechnung generieren lassen → Adresse korrekt.
3. Neue Org anlegen → Customer in Stripe hat sofort Adresse.
