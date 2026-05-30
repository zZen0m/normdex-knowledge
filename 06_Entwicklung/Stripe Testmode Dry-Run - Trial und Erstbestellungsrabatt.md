# Stripe-Testmode Dry-Run: 14-Tage-Trial und Erstbestellungsrabatt

Dokumentierter End-to-End-Prüfablauf für den 14-Tage-Testvorteil und den Erstbestellungsrabatt im Stripe-Testmode. Wird vor jedem Produktiv-Deployment im Lizenz-/Trial-Bereich einmalig durchlaufen und das Ergebnis hier bzw. in der zugehoerigen Todo-/Audit-Datei vermerkt.

## Voraussetzungen

- Backend laeuft lokal (`apps/api/`) mit Stripe-Testmode-Keys.
- Frontend laeuft lokal (`apps/frontend/`) gegen das lokale Backend.
- Stripe CLI ist installiert und mit dem Testaccount eingeloggt: `stripe login`.
- `.env` enthaelt:
  - `STRIPE_COUPON_ID_NEW_CUSTOMER_DISCOUNT=QHQESezY`
  - `LICENSE_TRIAL_BENEFIT_DAYS=14`
- Stripe-CLI-Webhook-Forwarder laeuft: `stripe listen --forward-to http://localhost:8000/subscriptions/webhook`.
- Stripe-Testkarte: `4242 4242 4242 4242`, beliebiges zukuenftiges Ablaufdatum, beliebiger CVC.

## Szenario 1 - Qualifizierter Einzel-Erstkauf (echter 14-Tage-Trial)

1. Neue Test-Organisation anlegen oder bestehende mit leerem `trial_used_at` verwenden.
2. In der App `/licenses` oeffnen, eine monatliche Lizenz im Kaufdialog auswaehlen.
3. Preview pruefen:
   - `free_trial_applies = true`
   - `first_order_discount_applies = false`
   - `trial_conversion_applies = false`
   - Banner zeigt "14 Tage Testphase fuer die erste Lizenz".
4. Checkout absenden, in Stripe-Checkout die Testkarte hinterlegen, Bestaetigung abwarten.
5. Backend pruefen:
   - Lizenz hat Status `trial`, `trial_started_at` und `trial_ends_at` gesetzt (14 Tage in der Zukunft).
   - `organizations.trial_used_at` ist gesetzt.
   - Stripe-Subscription hat Status `trialing`, `trial_end` ist gesetzt, keine Erst-Rechnung mit Betrag > 0.
6. In der Lizenzverwaltung erscheint das Badge `Test Lizenz · 14 Tage uebrig`.

## Szenario 2 - Qualifizierter Mehrfach-Erstkauf (Erstbestellungsrabatt)

1. Neue Test-Organisation ohne Lizenzhistorie verwenden.
2. Im Kaufdialog drei monatliche Lizenzen auswaehlen.
3. Preview pruefen:
   - `free_trial_applies = false`
   - `first_order_discount_applies = true`
   - `first_order_discount_amount_gross = 24.5`
   - Rabatt-Zeile "Erstbestellungsrabatt -24,50 EUR" ist sichtbar.
   - `total_gross` enthaelt 49 + 29 + 29 - 24,50 = 82,50 EUR.
4. Checkout absenden, Stripe-Testkarte hinterlegen, Zahlung bestaetigen.
5. Backend pruefen:
   - Alle drei Lizenzen haben Status `active` (keine `trial`).
   - Erste Stripe-Rechnung enthaelt einmalig den Coupon `QHQESezY` mit `amount_off = 2450` Cent.
   - `organizations.trial_used_at` ist gesetzt (Sperre bleibt fuer spaetere Bestellungen).

## Szenario 3 - Gemischte Erstbestellung (monatlich + jaehrlich)

1. Neue Test-Organisation ohne Lizenzhistorie.
2. Im Kaufdialog je eine monatliche und eine jaehrliche Lizenz waehlen.
3. Preview pruefen:
   - `first_order_discount_applies = true`
   - `first_order_discount_billing_pool = "monthly"` (deterministische Wahl: monatlich vor jaehrlich).
   - Genau eine Rabatt-Zeile von 24,50 EUR sichtbar.
4. Checkout absenden, Zahlung bestaetigen.
5. Backend pruefen:
   - Beide Lizenzen `active`, keine `trial`.
   - Coupon wird nur einmal auf die monatliche Rechnung angewendet.

## Szenario 4 - Zusatzkauf waehrend Trial (Trial-Konvertierung mit aliquoter Gutschrift)

1. Mit der Test-Organisation aus Szenario 1 weiterarbeiten (Trial laeuft).
2. Optional Stripe-Testclock auf `trial_started_at + 7 Tage` vorspulen, sodass 7 Trial-Tage uebrig sind.
3. Im Kaufdialog eine weitere monatliche Lizenz hinzufuegen.
4. Preview pruefen:
   - `trial_conversion_applies = true`
   - `trial_conversion_remaining_days = 7`
   - `trial_conversion_credit_amount_gross = 12.25` (24,50 / 14 * 7).
   - Banner "Die Testphase endet bei erfolgreicher Zahlung. 7 verbleibende Testtage werden als Gutschrift abgezogen."
   - Zeile "Umwandlung der Testlizenz in eine Hauptlizenz" mit Bruttobetrag der bisherigen Trial-Lizenz.
   - Rabatt-Zeile "Gutschrift fuer verbleibende Testtage -12,25 EUR".
5. Checkout absenden, Stripe-Testkarte bestaetigen.
6. Backend pruefen:
   - Bisherige Trial-Lizenz: Status `active`, `trial_converted_at` gesetzt.
   - Event `license.trial_converted` ist im Audit-Log.
   - Stripe-Subscription: Status `active`, `trial_end` in Vergangenheit oder cleared.
   - Erste neue Rechnung enthaelt Gutschrift von 12,25 EUR auf die Hauptlizenz.

## Szenario 5 - Abbruch waehrend Trial-Checkout

1. Neue Test-Organisation ohne Lizenzhistorie.
2. Einzel-Erstkauf starten, Stripe-Checkout aber nicht abschliessen (Tab schliessen oder "Cancel" in Stripe).
3. App ruft `checkout/cancel` auf.
4. Backend pruefen:
   - Pending Lizenz hat `checkout_discarded_at`.
   - `organizations.trial_used_at` ist wieder `NULL`.
   - `trial_benefit_release_reason` ist gesetzt (z. B. `checkout_cancelled_without_stripe_subscription`).
5. Neuen Einzel-Erstkauf mit derselben Organisation starten - Trial wird wieder gewaehrt.

## Szenario 6 - Bestehender Kunde, kein neuer Testvorteil

1. Mit der Organisation aus Szenario 1 (Trial-Lizenz oder bereits konvertierte Lizenz).
2. Im Kaufdialog eine zusaetzliche Lizenz waehlen.
3. Preview pruefen:
   - `free_trial_applies = false`
   - `first_order_discount_applies = false`
   - `trial_benefit_reason` z. B. `organization_has_license_history` oder `trial_benefit_already_used`.
   - Keine Rabatt-Zeile, kein Trial-Banner.

## Szenario 7 - Trial-Erinnerungs-E-Mail

1. Testorganisation mit aktiver Trial-Lizenz aus Szenario 1.
2. Stripe-Testclock so vorspulen, dass `trial_will_end`-Webhook gefeuert wird (3 Tage vor Trial-Ende).
3. Backend-Log und Inbox pruefen:
   - Webhook-Handler `_handle_subscription_trial_will_end` greift.
   - E-Mail an den Org-Owner mit Template `tpl_trial_will_end` wird verschickt (Brevo-Testaccount).

## Ergebnisprotokoll

Pro Dry-Run-Durchlauf das Ergebnis kurz festhalten:

- Datum/Tester: ...
- Stripe-Account: Test
- Backend-Version / Git-SHA: ...
- Ergebnis je Szenario: OK / abgewichen
- Auffaelligkeiten: ...

Bei Abweichungen Issue/Todo anlegen und referenzieren.
