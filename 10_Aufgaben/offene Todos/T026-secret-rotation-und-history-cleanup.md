# T026 · Secret-Rotation und Remote-History-Cleanup

**Phase:** App-Betrieb / Security / Infrastruktur  
**Priorität:** P1 · Zugangsdaten / Git-Historie  
**Status:** in Arbeit  
**Datum:** 2026-06-13

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
- Neun bekannte historische Fingerprints der aktiven Historie sind vorübergehend in `.gitleaksignore` dokumentiert, damit neue Funde das CI-Gate zuverlässig stoppen.

## Akzeptanzkriterien

- [ ] SMTP-Zugangsdaten beim Provider rotieren beziehungsweise den alten SMTP-Schlüssel widerrufen.
- [ ] Betroffene Stripe Secret Keys und Webhook Secrets identifizieren, rotieren und alte Werte widerrufen.
- [ ] Betroffenes Microsoft Client Secret rotieren und den alten Wert widerrufen.
- [ ] Neue Werte ausschließlich in den vorgesehenen Dev-/Prod-Secret-Dateien beziehungsweise Deployment-Secrets hinterlegen.
- [ ] Dev- und Produktivsystem nach der Rotation neu starten und E-Mail-, Stripe-Webhook- und Microsoft-Graph-Flows testen.
- [ ] Provider-Logs für den relevanten Zeitraum auf ungewöhnliche Nutzung prüfen und das Ergebnis dokumentieren.
- [ ] Remote-Historie von `develop`, `dev-server`, `main` und betroffenen Tags koordiniert bereinigen; Force-Push nur nach Sicherung und Abstimmung durchführen.
- [ ] Alle vorhandenen Klone nach dem Remote-Rewrite neu klonen oder hart auf die bereinigten Refs ausrichten.
- [ ] Die neun historischen Einträge aus `.gitleaksignore` entfernen und einen vollständigen Gitleaks-Lauf ohne Baseline erfolgreich abschließen.
- [ ] GitHub Secret Scanning und Push Protection aktivieren, sofern im Repository verfügbar.
- [ ] Der neue GitHub-Workflow `Secret scan` läuft auf allen drei aktiven Branches grün.

## Notizen / Fortschritt

- 2026-06-13: Lokale Legacy-Historie bereinigt und präventives CI-Gate implementiert.
- 2026-06-13: Aktive Remote-Historie nicht automatisch neu geschrieben oder force-gepusht, weil dies alle bestehenden Klone und offenen Arbeiten betrifft und koordiniert erfolgen muss.
- 2026-06-13: Keine Zugangsdaten oder Secret-Werte in dieser Dokumentation gespeichert.
- 2026-06-20: Beim Audit-Fixing erneut geprüft. Provider-Rotation, Logprüfung, Remote-History-Rewrite und Entfernung der neun `.gitleaksignore`-Einträge sind weiterhin offen und wurden nicht als erledigt markiert.
