# T031 · Webapp-Produktivrelease und Prod-Stack-Konsolidierung

**Phase:** App-Betrieb / Release / Infrastruktur
**Priorität:** P0 · Produktivbetrieb
**Status:** erledigt
**Datum:** 2026-06-22
**Abgeschlossen:** 2026-06-24
**Voraussetzung:** [[T030-automatisierte-stripe-gutschriften-bei-refunds]]

## Ziel

Den auf `normdex-webapp-dev` vollständig geprüften Stand kontrolliert in die
Produktion übernehmen und den historisch entstandenen doppelten
Prod-API-Compose-Betrieb auf einen eindeutigen, autoritativen Stack
konsolidieren.

## Ausgangslage

- T030 ist auf `dev-server` mit Commit `a8fd748` abgeschlossen.
- T032 (einheitlicher Versand und Abrechnungsdokumente) ist fachlich abgeschlossen und durch den Owner abgenommen, Commit `9c2462f`.
- T029 (Webapp-Audit-Rollout, externe Konfiguration) ist vollständig abgeschlossen.
- Dev läuft gesund auf App-Version `0.1.2` und Alembic-Head `f4d5e6f7a8b9` (verifiziert per `alembic heads`/`alembic current` im Container, 2026-06-23). Durch T032s Migration ist dies der neue Head, vorher dokumentiertes Ziel `e3c4d5e6f7a8` ist überholt.
- Die produktive Datenbank steht vor dem Release auf `995d14683241`.
- Die vollständige Migration wurde bereits erfolgreich auf einem isolierten
  Produktionsdatenbank-Klon geprüft.
- Live-Stripe wurde read-only inventarisiert; es bestehen keine offenen
  Refund-/Credit-Note-Altfälle.
- Auf dem Host laufen derzeit zwei Prod-API-Container mit derselben
  Traefik-Route `api.normdex.at`:
  - `deploy-api-1` aus Compose-Projekt `deploy`
  - `normdex_prod-api-1` aus Compose-Projekt `normdex_prod`
- Traefik verteilt Requests dadurch auf unterschiedliche App-Stände. Der
  dokumentierte Release-Wrapper `deploy/prod-compose.sh` verwendet das
  Compose-Projekt aus `deploy/env/.env.deploy.prod`; dieses ist vor dem
  Release als autoritativer Stack zu bestätigen.

## Arbeitspakete

- [x] Zielversion und Release-Banner bestätigt: **0.1.2**, kein Release-Banner, keine neue What's-New-Ausgabe — nur die Versionsnummer auf der bestehenden Grundversions-Seite wird über `__APP_VERSION__` mitgezogen. *(2026-06-23; technisch bereits in `apps/frontend/package.json`, `apps/api/app/version.py`, `Dockerfile`/`Dockerfile.dev` und `CHANGELOG.md` auf `develop`/`dev-server` umgesetzt, Commit folgt.)*
- [x] Produktiv-Repository und Vault auf sauberen Git-Stand prüfen. *(2026-06-24: `normdex-webapp` stand vor dem Release sauber auf `origin/main`.)*
- [x] Vollständiges PostgreSQL-Backup vor der Migration erstellen und prüfen. *(2026-06-24 18:45 — siehe Durchführung unten.)*
- [x] `dev-server` gemäß Branch-Workflow nach `main` übernehmen. *(Merge-Commit `8d14ad6`, 94 Dateien, keine Konflikte.)*
- [x] Produktiv-Environment einschließlich `NORMDEX_APP_VERSION` prüfen. *(`NORMDEX_APP_VERSION` wird nicht per Env-Datei, sondern als Dockerfile-`ARG`-Default aus `apps/api/app/version.py`/`package.json` gebaut; nach Rebuild korrekt `0.1.2`.)*
- [x] Autoritatives Compose-Projekt festlegen. *(`normdex_prod`, gemäß `deploy/env/.env.deploy.prod`, bestätigt.)*
- [x] Alten bzw. nicht autoritativen Prod-API-Stack kontrolliert entfernen. *(Compose-Projekt `deploy` / Container `deploy-api-1` gestoppt und entfernt, dazugehöriges Netzwerk `deploy_default` und die leeren Volumes `deploy_normdex_uploads`/`deploy_frontend_node_modules` aufgeräumt.)*
- [x] Migration bis `f4d5e6f7a8b9` über den Migration-Service ausführen. *(Erst fehlgeschlagen: `api-migrate` hatte ein eigenes, von `./deploy/prod-compose.sh build` nicht mitgebautes Image mit veraltetem Migrationsgraph — `--profile tools build api-migrate` behoben, danach lief `995d14683241 → ffd3bbde6b6a → c1a2b3d4e5f6 → d2b3c4e5f6a7 → e3c4d5e6f7a8 → f4d5e6f7a8b9` durch.)*
- [x] API und Frontend neu bauen und starten. *(`./deploy/prod-compose.sh up -d --build`, beide Container healthy.)*
- [x] Health, App-Version, Alembic-Head, Login, Admin-Billing und Stripe-Webhooks prüfen. *(Alle grün, siehe Durchführung unten.)*
- [x] T030-Reconciliation und Systemfehler nach dem Release überwachen. *(`billing_adjustments`: 0 Zeilen, keine `manual_review_required`/`failed`/`partially_failed`-Fälle, 0 Systemfehler in der letzten Stunde.)*
- [x] Release und Ergebnis im Vault dokumentieren. *(dieser Abschnitt.)*

## Durchführung am 2026-06-24

- **Backup:** `normdex-backup`-Service manuell angestoßen (`docker exec normdex-backup /app/backup.sh`), Lauf `20260624_184532`. DB-Dump 136 KB, Uploads-Archiv 4,3 MB, beide lokal unter `/opt/repos/normdex-backup/data/20260624_184532/` und offsite via rclone nach SharePoint hochgeladen. `gzip -t` / `tar -tzf` erfolgreich geprüft.
- **Merge:** `dev-server` (Tip `a36f79b`) nach `main` (vorher `de26066`) gemergt → Merge-Commit `8d14ad6`, gepusht nach `origin/main`. Kein Konflikt; main hatte nur einen Solo-Commit (`de26066`), dessen Inhalt mit einem bereits in `dev-server` enthaltenen Commit (`69e884d`) inhaltlich identisch war.
- **Build/Migration/Deploy:** `./deploy/prod-compose.sh build`, `--profile tools run --rm api-migrate` (nach Image-Rebuild), `up -d --build`. `normdex_prod-api-1` und `normdex_prod-frontend-1` neu erstellt, beide gesund.
- **Stack-Konsolidierung:** Vor der Bereinigung antwortete `https://api.normdex.at/health` im Wechsel mit `0.1.2` (neu, `normdex_prod-api-1`) und `0.1.0` (alt, `deploy-api-1`) — der dokumentierte Traefik-Split-Brain war live reproduzierbar. Nach Entfernen von `deploy-api-1` (Compose-Projekt `deploy`) antwortete der Endpoint in 8/8 Stichproben konsistent mit `0.1.2`.
- **Health-Checks:** `GET /health` intern und extern → `{"status":"ok","version":"0.1.2"}`; `alembic current` im laufenden Container → `f4d5e6f7a8b9 (head)`; `POST /auth/login` mit Dummy-Daten → `401` (Route funktionsfähig); `POST /subscriptions/webhook` ohne Signatur → `400` (Route funktionsfähig, Signaturprüfung aktiv); `https://app.normdex.at/login` → `200`; keine Fehler/Warnungen in den API-Logs seit Neustart.
- **T030-Reconciliation:** `billing_adjustments`-Tabelle leer (0 Zeilen, keine Altfälle in Produktion — deckt sich mit der read-only-Inventarisierung aus T030), keine Systemfehler in der letzten Stunde.
- **Zeitpunkt:** Release durchgeführt 2026-06-24, ca. 18:45–18:53 (lokale Serverzeit, UTC+2 / 16:45–16:53 UTC).

## Sicherheitsregeln

- Keine Produktivmigration ohne frisches Backup.
- Nicht beide API-Stacks parallel mit derselben Traefik-Route betreiben.
- Keine schreibenden Stripe-Bestandstests ohne ausdrücklich freigegebenen
  fachlichen Testfall.
- Bei fehlerhafter Migration oder Health-Prüfung Rollout stoppen und anhand
  des Backups bzw. des vorherigen Images zurückrollen.

## Akzeptanzkriterien

- [x] `api.normdex.at` wird von genau einem autoritativen Prod-API-Container bedient.
- [x] Produktion läuft auf der bestätigten App-Version (0.1.2).
- [x] Produktivdatenbank steht auf Alembic-Head `f4d5e6f7a8b9`.
- [x] Health- und Kernfunktionsprüfungen sind grün.
- [x] T030-Reconciliation meldet keine offenen oder widersprüchlichen Fälle.
- [x] Backup, Release-Commit, Deployment-Zeitpunkt und Monitoring-Ergebnis sind dokumentiert.
