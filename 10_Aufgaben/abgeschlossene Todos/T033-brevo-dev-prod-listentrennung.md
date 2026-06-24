# T033 · Brevo Dev/Prod-Listentrennung für Newsletter-Webhook

**Phase:** App / Newsletter / Brevo / Infrastruktur
**Priorität:** P2 · Vermeidet Seiteneffekte bei künftigen Dev-Tests
**Status:** erledigt
**Datum:** 2026-06-24

## Ziel

Newsletter-Tests auf dem Dev-Server lösen nicht mehr den Prod-Webhook (und damit einen echten Live-Stripe-Gutschein) aus.

## Kontext

Bei der End-to-end-Verifikation von [[T028-newsletter-nurture-brevo-umsetzung]] am 2026-06-24 festgestellt: Dev und Prod nutzen denselben Brevo-Account und dieselbe Liste **Normdex Newsletter** (`listId 3`), `BREVO_LIST_ID=3` ist in `deploy/env/.env.api.dev` **und** `deploy/env/.env.api.prod` identisch gesetzt.

Brevo-Webhooks sind nicht pro Liste gefiltert, sondern account-weite Abos auf Events (z. B. `listAddition`). Es gibt bereits zwei getrennte Webhook-URLs (`https://dev-api.normdex.at/newsletter/brevo/webhook` und `https://api.normdex.at/newsletter/brevo/webhook`), beide abonnieren `listAddition` und feuern beide bei **jeder** Aufnahme in Liste 3 — unabhängig davon, über welche Umgebung die Anmeldung kam.

Der App-Code hat bereits einen Guard dafür (`apps/api/app/routers/newsletter.py`, `brevo_webhook`): `if settings.BREVO_LIST_ID not in list_ids: return {"status": "ignored", "reason": "wrong_list"}`. Das Problem ist nicht das Fehlen eigener Webhooks, sondern dass beide Umgebungen denselben Listen-Wert prüfen.

**Konkreter Vorfall:** Ein Dev-Test mit `office+coupontest@normdex.at` hat zusätzlich zum Dev-Webhook auch den Prod-Webhook ausgelöst. Prod lief mit Live-Mode-Stripe-Key (`sk_live...`) und erzeugte einen echten Gutschein `NDX10-4TU0J9` (10 %, 1x einlösbar). Wurde nachträglich über die Stripe-API (`active: false`) deaktiviert, da unredeemed.

## Lösungsweg

1. Neue Brevo-Liste anlegen, z. B. **„Normdex Newsletter – Dev"** (separate `listId`).
2. `BREVO_LIST_ID` in `deploy/env/.env.api.dev` auf die neue Dev-Listen-ID umstellen.
3. `BREVO_DOUBLE_OPTIN_TEMPLATE_ID` ggf. prüfen, ob eine eigene (oder dieselbe) Double-Opt-in-Vorlage für die Dev-Liste gewünscht ist.
4. Bestehende Brevo-Webhooks bleiben unverändert (keine neuen Webhooks nötig) — der vorhandene `wrong_list`-Guard sorgt automatisch für die Trennung, sobald die Listen-IDs unterschiedlich sind.
5. Dev-API-Container neu starten, End-to-end-Test wiederholen und prüfen, dass dabei **kein** Eintrag mehr in der Prod-DB (`newsletter_coupon_claims`) und **kein** neuer Live-Stripe-Promotion-Code entsteht.

## Akzeptanzkriterien

- [x] Eigene Brevo-Liste für Dev-Testsignups angelegt: **„Normdex Newsletter – Dev"**, `listId 9`.
- [x] `BREVO_LIST_ID` in `.env.api.dev` auf `9` umgestellt (beide Vorkommen der Variable in der Datei), Dev-API-Container neu gestartet.
- [x] Verifiziert per synthetischem Webhook-Aufruf (`event: list_addition`, `list_id: 9`) direkt gegen beide Endpunkte: Prod-Webhook (`https://api.normdex.at/newsletter/brevo/webhook`) antwortet jetzt mit `{"status":"ignored","reason":"wrong_list"}` — kein Prod-DB-Eintrag, kein neuer Live-Stripe-Code. Dev-Webhook verarbeitet die Liste 9 weiterhin normal. Testzeile danach aus der Dev-DB entfernt.
- [x] Vorgehen in [[04 Brevo-Umsetzung und Workflow-Anleitung]] dokumentiert (Dev/Prod-Hinweis im Trigger-Abschnitt).

## Notizen / Fortschritt

- 2026-06-24: Angelegt im Rahmen der T028-Verifikation. Live-Stripe-Gutschein `NDX10-4TU0J9` aus dem betroffenen Testlauf wurde deaktiviert.
- 2026-06-24: Umgesetzt. Andreas hat Liste 9 „Normdex Newsletter – Dev" in Brevo angelegt. `BREVO_LIST_ID` in `deploy/env/.env.api.dev` (im `normdex-webapp-dev`-Repo) von `3` auf `9` geändert (Datei ist `.gitignore`d, kein Commit nötig), `normdex-dev-api`-Container neu gestartet (`docker compose -f /opt/stacks/normdex-dev/api/docker-compose.dev-build.yml up -d api`). Fix verifiziert: synthetischer Webhook-Call mit `list_id: 9` gegen Prod liefert `wrong_list`, gegen Dev wird normal verarbeitet. Künftige Dev-Tests lösen damit keinen Prod-Seiteneffekt mehr aus.

## Verwandte Dokumente

- [[T028-newsletter-nurture-brevo-umsetzung]]
- [[T019-newsletter-gutschein-brevo-webhook-rollout]]
- [[04 Brevo-Umsetzung und Workflow-Anleitung]]
