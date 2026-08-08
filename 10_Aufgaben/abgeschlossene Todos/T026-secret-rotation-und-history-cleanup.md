# T026 · Secret-Rotation und Remote-History-Cleanup

**Phase:** App-Betrieb / Security / Infrastruktur  
**Priorität:** P1 · Zugangsdaten / Git-Historie  
**Status:** erledigt  
**Datum:** 2026-06-13  
**Abgeschlossen:** 2026-08-08

## Ziel

Alle potenziell offengelegten Zugangsdaten werden beim jeweiligen Provider rotiert und aus der erreichbaren GitHub-Historie entfernt. Anschließend schützen automatisierte Secret-Scans alle aktiven Branches vor neuen Leaks.

## Kontext

Das Webapp-Audit vom 2026-06-13 meldete historische SMTP-Zugangsdaten in lokalen Legacy-Branches. Bei der Behebung wurden zusätzlich historische Stripe- und Microsoft-Secret-Fingerprints in weiterhin aktiven Branch-Historien gefunden. Aus dem Repository allein lässt sich nicht belegen, ob diese Zugangsdaten noch gültig sind oder jemals missbraucht wurden.

Repo-seitig bereits umgesetzt:

- Die lokalen Legacy-Branches `06.02.2026`, `21.02.2026`, `v0.0.1` und `v0.1.0` wurden ohne die historisch versionierten `.env`-Dateien neu geschrieben.
- Historische Datenbank-Backups wurden aus diesen Legacy-Refs entfernt.
- Alte Objekte und Reflogs wurden lokal gepruned.
- Aktive Branches und Tags blieben bei dieser lokalen Legacy-Bereinigung unverändert.
- Gitleaks und eine zusätzliche Pfadprüfung blockieren künftig neue Secret-Dateien und Datenbank-Backups.

## Akzeptanzkriterien

- [x] SMTP-Zugangsdaten beim Provider rotieren beziehungsweise den alten SMTP-Schlüssel widerrufen.
- [x] Betroffene Stripe Secret Keys und Webhook Secrets identifizieren, rotieren und alte Werte widerrufen.
- [x] Betroffenes Microsoft Client Secret rotieren und den alten Wert widerrufen.
- [x] Neue Werte ausschließlich in den vorgesehenen Dev-/Prod-Secret-Dateien beziehungsweise Deployment-Secrets hinterlegen.
- [x] Dev- und Produktivsystem nach der Rotation neu starten und E-Mail-, Stripe-Webhook- und Microsoft-Graph-Flows testen.
- [x] Provider-Logs für den relevanten Zeitraum auf ungewöhnliche Nutzung prüfen und das Ergebnis dokumentieren.
- [x] Der GitHub-Workflow `Secret scan` läuft auf allen drei aktiven Branches grün.

Bewusst nicht umgesetzt (siehe Ergebnis):

- [ ] ~~Remote-Historie von `develop`, `dev-server`, `main` und betroffenen Tags koordiniert bereinigen~~ — nicht mehr sicherheitskritisch, da Werte rotiert.
- [ ] ~~Alle vorhandenen Klone nach dem Remote-Rewrite neu klonen~~ — entfällt, da kein Rewrite durchgeführt.
- [ ] ~~Alle neun historischen `.gitleaksignore`-Einträge entfernen und vollständigen Gitleaks-Lauf ohne Baseline abschließen~~ — nur die 3 toten Einträge entfernt, 6 reale bleiben bestehen.
- [ ] ~~GitHub Secret Scanning und Push Protection aktivieren~~ — vom Nutzer als nicht notwendig eingestuft.

## Ergebnis

Alle vier betroffenen Zugangsdaten-Familien wurden beim jeweiligen Provider rotiert und **end-to-end verifiziert** (nicht nur Login-Test, sondern echte Funktionsprüfung), sowohl auf dem Dev-Server als auch in Produktion:

- **SMTP (Brevo):** Login erfolgreich, zusätzlich reale Testmail-Zustellung (17 Vorlagen via `send_test_email.py`) auf Dev (`notify-dev@normdex.at`) und Prod (`notify@normdex.at`) bestätigt — beide Umgebungen nutzen unterschiedliche Brevo-Zugangsdaten und wurden daher separat getestet.
- **Brevo Newsletter API:** Account-Abfrage auf Dev und Prod erfolgreich.
- **Stripe:** `Balance.retrieve()` erfolgreich (Dev `livemode=false`, Prod `livemode=true` — korrekt). Webhook-Signaturprüfung zusätzlich per echtem `checkout.session.expired`-Event (Session erzeugt und sofort abgelaufen, kein Zahlungsvorgang) auf beiden Systemen mit `200 OK` bestätigt. Dabei auch verifiziert, dass die bei Stripe hinterlegten Webhook-URLs korrekt sind.
- **Microsoft Graph:** Client-Credentials-Token-Erwerb auf Dev und Prod erfolgreich, keine Fehler in den Startup-Logs.

Neue Werte liegen ausschließlich in `deploy/env/.env.api.dev` (Dev-Server) und `deploy/env/.env.api.prod` (Produktiv) auf dem VPS; beide API-Container (`normdex-dev-api`, `normdex_prod-api-1`) wurden neu gestartet, um sie zu laden.

Provider-Logs (Brevo/Stripe/Azure) wurden vom Nutzer geprüft — keine Auffälligkeiten gefunden.

**Bewusste Entscheidung gegen den vollständigen Remote-History-Rewrite:** Sobald die Zugangsdaten rotiert sind, sind die historischen Werte im Git-Verlauf wertlos — ein Force-Push-Rewrite von `main`/`develop`/`dev-server` samt Neuausrichtung aller Klone wäre reine Hygiene ohne verbleibenden Sicherheitsgewinn, bei hohem Aufwand/Risiko für eine produktive Branch. Stattdessen wurden nur die zwei toten `.gitleaksignore`-Einträge entfernt, die auf inzwischen nicht mehr existierende Commits verwiesen (Überbleibsel der früheren lokalen Legacy-Bereinigung, verifiziert via `git cat-file` und GitHub API) — Commit [`b560001`](https://github.com/zZen0m/normdex-app/commit/b560001) auf `develop`, `Secret scan`-Workflow bleibt grün. Die 6 verbleibenden realen Einträge (4 Commits) bleiben bestehen; ein vollständiger History-Rewrite kann bei Bedarf später als eigene, niedrig priorisierte Aufgabe nachgeholt werden.

GitHub Secret Scanning / Push Protection sind aktuell deaktiviert (Repo ist privat, Aktivierung würde GitHub Advanced Security voraussetzen, ggf. kostenpflichtig) — vom Nutzer bewusst als nicht notwendig eingestuft.

**Nebenbefund (separat als Task gemeldet, nicht Teil von T026):** `deploy/env/.env.deploy.prod` existiert nicht auf dem VPS, wodurch `deploy/prod-compose.sh` mit einem BasicAuth-Fehler abbricht. Der reale Prod-Compose-Projektname (`normdex_prod`) ist nirgends im Repo dokumentiert, nur aus der Server-`.bash_history` rekonstruierbar.
