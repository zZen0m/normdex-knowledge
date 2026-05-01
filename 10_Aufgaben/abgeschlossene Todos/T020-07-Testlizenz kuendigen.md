# T020-07 · Testlizenz kündigen (Endpoint + UI)

**Phase:** 2 (Funktionale Lücke)
**Priorität:** P1
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Eine Testlizenz kann aktuell nicht vom Nutzer gekündigt werden. Es soll möglich sein, die Trial direkt zu beenden — sie läuft jedoch bis zum regulären Trial-Ende weiter und wird **nicht** in eine aktive Hauptlizenz überführt. Nach Ablauf wird sie entfernt.

## Betroffene Dateien
- Backend: `apps/api/app/routers/licenses_v2.py:1649-1705` — bestehender Cancel-Endpoint, prüfen ob Status `trial` zugelassen werden kann; sonst neuer Endpoint `POST /licenses/trial/cancel`.
- Frontend: `apps/frontend/src/pages/Licenses.tsx:846-991` — `LicenseCard`-Aktionsmenü.

## Umsetzung
### Backend
- Subscription mit `cancel_at_period_end=True` auf Stripe markieren (period_end = trial_ends_at).
- Sicherstellen, dass beim Webhook-Event `customer.subscription.deleted` keine Konvertierung zu paid passiert.
- Audit-Log-Eintrag schreiben.

### Frontend
- Im Aktionsmenü der Trial-`LicenseCard` Eintrag "Testlizenz kündigen".
- Bestätigungs-Modal mit Erläuterung:
  - "Deine Testlizenz läuft bis zum [trial_ends_at]."
  - "Nach Ablauf wird sie entfernt — kein automatischer Wechsel zu einer Hauptlizenz."
- Nach Bestätigung Status-Badge auf "läuft aus" / "endet am ..." setzen.

## Akzeptanzkriterien
- [ ] Trial-Lizenz hat Aktion "Testlizenz kündigen".
- [ ] Bestätigungsdialog erläutert Konsequenzen.
- [ ] Trial bleibt bis `trial_ends_at` aktiv.
- [ ] Kein automatischer Übergang zu paid Subscription.
- [ ] Stripe Subscription hat `cancel_at_period_end=true`.

## Verifikation
Test-Account mit Trial → kündigen → Status zeigt "läuft aus" → nach Ablauf Lizenz entfernt, keine Rechnung.
