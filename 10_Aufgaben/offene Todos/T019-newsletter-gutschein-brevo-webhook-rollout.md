# T019 - Newsletter-Gutschein Brevo-Webhook Rollout

**Status:** in Arbeit  
**Bereich:** App / Newsletter / Brevo / Stripe / E-Mail  
**Erstellt:** 2026-05-01  
**Abgeschlossen:** -

## Zweck

Der 10%-Newsletter-Gutschein soll erst nach bestaetigtem Brevo-Double-Opt-in versendet werden. Brevo meldet die bestaetigte Listenaufnahme per Outbound Webhook an Normdex. Normdex erzeugt danach einen individuellen Stripe Promotion Code und versendet ihn per E-Mail.

## Aktueller Stand

- Backend-Implementierung wurde in `apps/api/` umgesetzt.
- Brevo Webhook Secret wurde produktiv erzeugt.
- Brevo Outbound Webhook wurde erstellt.
- Stripe Coupon ID fuer den 10%-Gutschein: `mbjs8wYE`.
- Gutscheinlaufzeit: 30 Tage.
- Codes sind pro bestaetigter E-Mail individuell und einmalig einloesbar.

## Offene To-dos

- [ ] Backend-Code auf Produktivserver deployen.
- [ ] Alembic-Migration ausfuehren: `python -m alembic upgrade head`.
- [ ] Produktiv-Env pruefen/setzen:
  - `BREVO_WEBHOOK_SECRET`
  - `STRIPE_COUPON_ID_NEWSLETTER_10_PERCENT=mbjs8wYE`
  - `NEWSLETTER_COUPON_EXPIRES_DAYS=30`
  - `BREVO_LIST_ID`
- [ ] API-Prozess nach Env-Aenderung neu starten.
- [ ] End-to-end Test mit neuer Test-E-Mail:
  - Landingpage Newsletter-Formular absenden.
  - Brevo Double-Opt-in bestaetigen.
  - Pruefen, ob Normdex einen individuellen Promotion Code erzeugt.
  - Pruefen, ob Gutschein-Mail zugestellt wird.
  - Code im Lizenz-Checkout testen.
- [ ] Fehlerfall pruefen: Webhook fuer falsche Liste darf keinen Coupon erzeugen.

## Akzeptanzkriterien

- Vor Double-Opt-in wird kein Gutschein erzeugt und keine Gutschein-Mail versendet.
- Nach `list_addition` fuer die konfigurierte Brevo-Liste wird genau ein individueller Stripe Promotion Code erzeugt.
- Wiederholte Brevo-Webhooks fuer dieselbe E-Mail erzeugen keinen zweiten Code.
- Der Code ist lokal in Normdex 30 Tage gueltig und wird in Stripe mit Ablaufdatum angelegt.
- Abgelaufene Newsletter-Codes werden vor Stripe Checkout abgelehnt.

## Technische Referenzen

- Backend-Endpoint: `POST /newsletter/brevo/webhook?secret=...`
- Subscribe-Endpoint: `POST /newsletter/subscribe`
- Tabelle: `newsletter_coupon_claims`
- Service: `apps/api/app/services/newsletter_coupon_service.py`
- Migration: `f2a3b4c5d6e7_add_newsletter_coupon_claims.py`
- Tests: `tests/test_newsletter.py`, `tests/test_license_checkout_trial.py`

## Notizen / Fortschritt

- 2026-05-01: Backend lokal implementiert und API-Test-Suite erfolgreich ausgefuehrt (`136 passed`).
- 2026-05-01: Brevo Webhook Secret wurde auf dem Produktivserver erstellt.
- 2026-05-01: Brevo Outbound Webhook wurde in Brevo erstellt. Ziel-URL muss das Secret als Query-Parameter enthalten.
