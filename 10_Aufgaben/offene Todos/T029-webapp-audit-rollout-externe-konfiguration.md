# T029 · Webapp-Audit-Rollout und externe Konfiguration

**Phase:** App-Betrieb / Security / CI
**Priorität:** P1 · Release-Gate
**Status:** offen
**Datum:** 2026-06-20

## Ziel

Die am 2026-06-20 repo-seitig behobenen Audit-Findings werden in GitHub, Brevo und auf dem VPS aktiviert und dort verifiziert.

## Kontext

Der Code- und Dokumentationsstand behebt Build, E-Mail-Änderung, Support-Uploads, Account-Enumeration, Brevo-Secret-Transport, Dev-Fixture, BasicAuth-Konfiguration und CI-Definition. Mehrere Maßnahmen werden jedoch erst nach Push und Deployment wirksam. Die historische Secret-Rotation und History-Bereinigung bleibt separat unter [[T026-secret-rotation-und-history-cleanup]].

## Akzeptanzkriterien

- [ ] Webapp-Commit auf `develop` pushen und den Workflow `Quality gates` erfolgreich ausführen.
- [ ] `Quality gates` und `Secret scan` als verpflichtende Checks für `develop`, `dev-server` und `main` konfigurieren.
- [ ] Brevo-Webhook auf `POST /newsletter/brevo/webhook` mit Bearer-Authentifizierung umstellen und `BREVO_WEBHOOK_SECRET` rotieren.
- [ ] Brevo-Webhook mit einem echten `list_addition`-Test verifizieren; Query-String-Aufrufe müssen HTTP 401 liefern.
- [ ] Auf dem VPS einen neuen starken BasicAuth-Wert erzeugen und als `NORMDEX_FRONTEND_BASIC_AUTH_USERS` in `deploy/env/.env.deploy.prod` hinterlegen.
- [ ] Dev-/Staging-Deployment durchführen und Frontend-Build, E-Mail-Änderung sowie Support-Upload/-Download per Smoke-Test prüfen.
- [ ] Nach erfolgreichem Staging-Test den freigegebenen Stand gemäß `develop` → `dev-server` → `main` ausrollen.

## Notizen / Fortschritt

- 2026-06-20: Repo-seitige Implementierung und lokale Tests abgeschlossen.
- 2026-06-20: Externe Aktivierung bewusst nicht als erledigt markiert, da weder GitHub-Branchschutz noch Brevo- oder VPS-Konfiguration aus dem lokalen Repository verifiziert werden können.
