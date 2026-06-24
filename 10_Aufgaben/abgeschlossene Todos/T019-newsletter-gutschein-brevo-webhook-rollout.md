# T019 - Newsletter-Gutschein Brevo-Webhook Rollout

**Status:** erledigt
**Bereich:** App / Newsletter / Brevo / Stripe / E-Mail
**Erstellt:** 2026-05-01
**Abgeschlossen:** 2026-05-30

## Zweck

Der 10%-Newsletter-Gutschein soll erst nach bestätigtem Brevo-Double-Opt-in versendet werden. Brevo meldet die bestätigte Listenaufnahme per Outbound Webhook an Normdex. Normdex erzeugt danach einen individuellen Stripe Promotion Code und versendet ihn per E-Mail.

## Aktueller Stand

- Backend-Implementierung in `apps/api/` umgesetzt.
- Brevo Webhook Secret produktiv erzeugt.
- Brevo Outbound Webhook erstellt.
- Stripe Coupon ID für den 10%-Gutschein: `mbjs8wYE`.
- Gutscheinlaufzeit: 30 Tage.
- Codes sind pro bestätigter E-Mail individuell und einmalig einlösbar.
- Backend produktiv deployt, Migration ausgeführt, Env-Variablen gesetzt, API-Prozess neu gestartet.
- End-to-end Test erfolgreich durchlaufen.

## Erledigte To-dos

- [x] Backend-Code auf Produktivserver deployen.
- [x] Alembic-Migration ausgeführt: `python -m alembic upgrade head`.
- [x] Produktiv-Env geprüft/gesetzt:
  - `BREVO_WEBHOOK_SECRET`
  - `STRIPE_COUPON_ID_NEWSLETTER_10_PERCENT=mbjs8wYE`
  - `NEWSLETTER_COUPON_EXPIRES_DAYS=30`
  - `BREVO_LIST_ID`
- [x] API-Prozess nach Env-Änderung neu gestartet.
- [x] End-to-end Test mit neuer Test-E-Mail:
  - Landingpage Newsletter-Formular absenden.
  - Brevo Double-Opt-in bestätigen.
  - Individueller Promotion Code wird erzeugt.
  - Gutschein-Mail wird zugestellt.
  - Code im Lizenz-Checkout funktioniert.
- [x] Fehlerfall geprüft: Webhook für falsche Liste erzeugt keinen Coupon.

## Akzeptanzkriterien

- Vor Double-Opt-in wird kein Gutschein erzeugt und keine Gutschein-Mail versendet.
- Nach `list_addition` für die konfigurierte Brevo-Liste wird genau ein individueller Stripe Promotion Code erzeugt.
- Wiederholte Brevo-Webhooks für dieselbe E-Mail erzeugen keinen zweiten Code.
- Der Code ist lokal in Normdex 30 Tage gültig und wird in Stripe mit Ablaufdatum angelegt.
- Abgelaufene Newsletter-Codes werden vor Stripe Checkout abgelehnt.

## Technische Referenzen

- Backend-Endpoint: `POST /newsletter/brevo/webhook` mit `Authorization: Bearer <BREVO_WEBHOOK_SECRET>`
- Subscribe-Endpoint: `POST /newsletter/subscribe`
- Tabelle: `newsletter_coupon_claims`
- Service: `apps/api/app/services/newsletter_coupon_service.py`
- Migration: `f2a3b4c5d6e7_add_newsletter_coupon_claims.py`
- Tests: `tests/test_newsletter.py`, `tests/test_license_checkout_trial.py`

## Notizen / Fortschritt

- 2026-05-01: Backend lokal implementiert und API-Test-Suite erfolgreich ausgeführt (`136 passed`).
- 2026-05-01: Brevo Webhook Secret auf dem Produktivserver erstellt.
- 2026-05-01: Brevo Outbound Webhook in Brevo erstellt. Ziel-URL enthält das Secret als Query-Parameter.
- 2026-05-30: Rollout abgeschlossen — Deploy, Migration, Env-Setup, API-Neustart und End-to-end Test erfolgreich. Fehlerfall (falsche Liste) verifiziert.
- 2026-06-24: Nachtrag im Rahmen von [[T028-newsletter-nurture-brevo-umsetzung]]. Der Webhook erzeugte den Code und versendete ihn nur via eigener App-Mail, schrieb ihn aber nicht in das Brevo-Kontaktattribut `COUPON_CODE` zurück. Für die Nurture-Strecke (`{{ contact.COUPON_CODE }}`) wurde der Service um einen best-effort Write-back (`PUT /v3/contacts/{email}`) erweitert. Details und Tests siehe T028.
- 2026-06-24: Beim Dev-Server-Test des Write-backs festgestellt: Dev und Prod teilen sich Brevo-Liste 3 (`BREVO_LIST_ID` identisch in beiden Envs), daher hat ein Dev-Testsignup auch den Prod-Webhook ausgelöst und einen echten Live-Stripe-Code erzeugt (deaktiviert). Der hier beschriebene `wrong_list`-Guard funktioniert korrekt, greift aber nur, wenn Dev und Prod unterschiedliche Listen-IDs verwenden. Nachverfolgung in [[T033-brevo-dev-prod-listentrennung]].
