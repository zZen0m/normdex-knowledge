# T031 · Webapp-Produktivrelease und Prod-Stack-Konsolidierung

**Phase:** App-Betrieb / Release / Infrastruktur
**Priorität:** P0 · Produktivbetrieb
**Status:** offen
**Datum:** 2026-06-22
**Voraussetzung:** [[T030-automatisierte-stripe-gutschriften-bei-refunds]]

## Ziel

Den auf `normdex-webapp-dev` vollständig geprüften Stand kontrolliert in die
Produktion übernehmen und den historisch entstandenen doppelten
Prod-API-Compose-Betrieb auf einen eindeutigen, autoritativen Stack
konsolidieren.

## Ausgangslage

- T030 ist auf `dev-server` mit Commit `a8fd748` abgeschlossen.
- Dev läuft gesund auf App-Version `0.1.1` und Alembic-Head `e3c4d5e6f7a8`.
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

- [ ] Zielversion und Release-Banner bestätigen.
- [ ] Produktiv-Repository und Vault auf sauberen Git-Stand prüfen.
- [ ] Vollständiges PostgreSQL-Backup vor der Migration erstellen und prüfen.
- [ ] `dev-server` gemäß Branch-Workflow nach `main` übernehmen.
- [ ] Produktiv-Environment einschließlich `NORMDEX_APP_VERSION` prüfen.
- [ ] Autoritatives Compose-Projekt festlegen.
- [ ] Alten bzw. nicht autoritativen Prod-API-Stack kontrolliert entfernen.
- [ ] Migration bis `e3c4d5e6f7a8` über den Migration-Service ausführen.
- [ ] API und Frontend neu bauen und starten.
- [ ] Health, App-Version, Alembic-Head, Login, Admin-Billing und Stripe-Webhooks prüfen.
- [ ] T030-Reconciliation und Systemfehler nach dem Release überwachen.
- [ ] Release und Ergebnis im Vault dokumentieren.

## Sicherheitsregeln

- Keine Produktivmigration ohne frisches Backup.
- Nicht beide API-Stacks parallel mit derselben Traefik-Route betreiben.
- Keine schreibenden Stripe-Bestandstests ohne ausdrücklich freigegebenen
  fachlichen Testfall.
- Bei fehlerhafter Migration oder Health-Prüfung Rollout stoppen und anhand
  des Backups bzw. des vorherigen Images zurückrollen.

## Akzeptanzkriterien

- [ ] `api.normdex.at` wird von genau einem autoritativen Prod-API-Container bedient.
- [ ] Produktion läuft auf der bestätigten App-Version.
- [ ] Produktivdatenbank steht auf Alembic-Head `e3c4d5e6f7a8`.
- [ ] Health- und Kernfunktionsprüfungen sind grün.
- [ ] T030-Reconciliation meldet keine offenen oder widersprüchlichen Fälle.
- [ ] Backup, Release-Commit, Deployment-Zeitpunkt und Monitoring-Ergebnis sind dokumentiert.
