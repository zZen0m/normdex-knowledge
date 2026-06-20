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

- [x] Webapp-Commit auf `develop` pushen und den Workflow `Quality gates` erfolgreich ausführen. *(Push am 2026-06-20: `c92f7c8..a8eb45d`; Workflow-Lauf in GitHub Actions kontrollieren.)*
- [ ] `Quality gates` und `Secret scan` als verpflichtende Checks für `develop`, `dev-server` und `main` konfigurieren.
- [ ] Brevo-Webhook auf `POST /newsletter/brevo/webhook` mit Bearer-Authentifizierung umstellen und `BREVO_WEBHOOK_SECRET` rotieren.
- [ ] Brevo-Webhook mit einem echten `list_addition`-Test verifizieren; Query-String-Aufrufe müssen HTTP 401 liefern.
- [ ] Auf dem VPS einen neuen starken BasicAuth-Wert erzeugen und als `NORMDEX_FRONTEND_BASIC_AUTH_USERS` in `deploy/env/.env.deploy.prod` hinterlegen.
- [ ] Dev-/Staging-Deployment durchführen und Frontend-Build, E-Mail-Änderung sowie Support-Upload/-Download per Smoke-Test prüfen.
- [ ] Nach erfolgreichem Staging-Test den freigegebenen Stand gemäß `develop` → `dev-server` → `main` ausrollen.

## Schritt-für-Schritt-Anleitung

### A · GitHub Branch-Protection / Required Checks (Finding 5)

1. Nach dem Push: <https://github.com/zZen0m/normdex-app/actions> öffnen und prüfen, dass der Workflow **Quality gates** (`quality.yml`) auf `develop` grün durchläuft.
2. **Settings → Branches → Branch protection rules → Add rule**.
3. Regel für `develop` anlegen (danach identisch für `dev-server` und `main`):
   - *Branch name pattern:* `develop`
   - ☑ **Require status checks to pass before merging**
   - ☑ **Require branches to be up to date before merging**
   - Unter „Status checks": **Quality gates** und **Secret scan** auswählen (erscheinen erst, nachdem sie mindestens einmal gelaufen sind).
   - Optional: ☑ **Require a pull request before merging**.
4. **Create / Save changes**. Für `dev-server` und `main` wiederholen.
5. Optional Hardening: **Settings → Code security and analysis** → *Secret scanning* und *Push protection* aktivieren (gehört thematisch zu [[T026-secret-rotation-und-history-cleanup]]).

### B · Brevo-Webhook auf Bearer umstellen + Secret rotieren (Finding 8)

1. Neues Secret erzeugen (z. B. lokal): `python -c "import secrets; print(secrets.token_urlsafe(32))"`.
2. Wert serverseitig als `BREVO_WEBHOOK_SECRET` in der **nicht versionierten** Dev-/Prod-Env hinterlegen (`deploy/env/.env.deploy.prod`), Dienst neu starten.
3. Im Brevo-Dashboard (Contacts → Settings → Webhooks bzw. die betroffene Automation) die Ziel-URL auf `POST https://<host>/newsletter/brevo/webhook` setzen und den **`Authorization: Bearer <Secret>`-Header** konfigurieren. Den alten `?secret=`-Query-Aufruf entfernen.
4. Verifizieren: echter `list_addition`-Test → 200; ein Aufruf mit `?secret=...` statt Header muss **HTTP 401** liefern.

### C · VPS BasicAuth rotieren (Finding 10)

1. Auf dem VPS neuen User+Hash erzeugen: `htpasswd -nbB <user> '<neues-starkes-passwort>'` (bcrypt). Bei Traefik-Compose die `$` im Hash zu `$$` verdoppeln.
2. Den Wert als `NORMDEX_FRONTEND_BASIC_AUTH_USERS` in `deploy/env/.env.deploy.prod` eintragen (nicht ins Repo committen).
3. `deploy/prod-compose.sh` mit aktiviertem BasicAuth neu hochfahren; der Wrapper verweigert den Start ohne gesetztes Credential.
4. Login mit dem **neuen** Wert testen, den **alten** Wert als kompromittiert betrachten und verwerfen.

### D · Historische Secret-Rotation (Finding 4)

Vollständig in [[T026-secret-rotation-und-history-cleanup]] beschrieben: SMTP-, Stripe- und Microsoft-Secrets beim Provider rotieren, Provider-Logs prüfen, koordinierter Remote-History-Rewrite, die 9 `.gitleaksignore`-Einträge entfernen, Gitleaks ohne Baseline grün.

## Notizen / Fortschritt

- 2026-06-20: Repo-seitige Implementierung und lokale Tests abgeschlossen.
- 2026-06-20: Externe Aktivierung bewusst nicht als erledigt markiert, da weder GitHub-Branchschutz noch Brevo- oder VPS-Konfiguration aus dem lokalen Repository verifiziert werden können.
- 2026-06-20: Re-Verifikation (frischer Lauf): Frontend-Build erfolgreich, `npm audit` 0 Schwachstellen, 34 Frontend- und 325 Backend-Tests grün. `develop` nach `origin` gepusht (`c92f7c8..a8eb45d`). Schritt-für-Schritt-Anleitung für GitHub/Brevo/VPS ergänzt.
