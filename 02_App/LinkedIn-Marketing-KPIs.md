# LinkedIn-Marketing-KPIs

**Stand:** 2026-07-19  
**Status:** Release `0.2.0` produktiv · erster automatischer 05:00-Uhr-Lauf offen  
**Todo:** [[T048-linkedin-marketing-kpis-verwaltungsportal]]

## Zweck

Der nur für Normdex-Admins sichtbare Bereich `/admin/marketing` verbindet den Follower-Verlauf der LinkedIn-Unternehmensseite mit veröffentlichten Beiträgen und deren jeweils neuestem Leistungsstand. Die Darstellung ist deskriptiv; Normdex berechnet weder einen kausalen „Post Impact“ noch automatisch zugerechnete Follower-Gewinne.

## Architektur

- Frontend: `apps/frontend/src/pages/admin/Marketing.tsx`
- Admin-API: `apps/api/app/routers/admin_marketing.py`
- LinkedIn-Client und Sync: `apps/api/app/services/linkedin_service.py`
- Scheduler: `apps/api/app/services/scheduler.py`
- Datenmodell: fünf Tabellen mit Alembic-Revision `a5c6d7e8f9b0`
- Fixture-Seed: `apps/api/scripts/seed_linkedin_marketing_fixtures.py`
- Token-Bootstrap und Status: `apps/api/scripts/bootstrap_linkedin_connection.py`
- Administrativer Vollabruf: `apps/api/scripts/sync_linkedin_marketing.py --full-history`

Nur `APP_ENV=prod` darf LinkedIn live abfragen. Lokal und auf `dev-server` bleiben `LINKEDIN_LIVE_SYNC_ENABLED=false` und `LINKEDIN_SCHEDULER_ENABLED=false`; dort werden ausschließlich Datenbank-Fixtures angezeigt.

## Konfiguration

```env
LINKEDIN_CLIENT_ID=...
LINKEDIN_CLIENT_SECRET=...
LINKEDIN_ORGANIZATION_URN=urn:li:organization:...
LINKEDIN_REST_VERSION=202607
LINKEDIN_TOKEN_ENCRYPTION_KEY=...
LINKEDIN_LIVE_SYNC_ENABLED=false
LINKEDIN_SCHEDULER_ENABLED=false
```

`LINKEDIN_REST_VERSION` ist konfigurierbar und muss vor jedem Rollout mit der offiziellen LinkedIn-Dokumentation abgeglichen werden. Am 19.07.2026 war `202607` die aktuelle Version. Client-Secret und Verschlüsselungsschlüssel werden nicht in der App-Datenbank gespeichert.

Einen Fernet-Schlüssel erzeugen:

```powershell
python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

## Token-Bootstrap und erneute Autorisierung

Der Import liest Tokens ausschließlich aus einer Datei, ist hart auf `APP_ENV=prod` begrenzt und akzeptiert auch das produktiv verwendete Feld `token_expires_at`. Token-Werte werden nicht als Kommandozeilenargumente oder in Ausgaben verwendet.

```bash
cd /opt/repos/normdex-app/apps/api
python scripts/bootstrap_linkedin_connection.py /home/deploy/tools/linkedin-orgpage/credentials.json
python scripts/bootstrap_linkedin_connection.py --status
```

Gespeichert werden authentifiziert verschlüsselte Access-/Refresh-Tokens und ihre Ablaufzeitpunkte. Bei einer LinkedIn-Antwort, die eine erneute Autorisierung erfordert, wechselt die Verbindung auf `reauthorization_required`; das Verwaltungsportal zeigt ausdrücklich „LinkedIn muss neu autorisiert werden“.

## Synchronisierungsregeln

- Automatisch täglich um 05:00 Uhr in `Europe/Vienna`
- Erster Lauf: veröffentlichte Posts der letzten zwölf Monate
- Folgeläufe: Metriken veröffentlichter Posts bis 90 Tage nach Veröffentlichung
- Geplante Posts werden gespeichert, aber ohne Performance-Abfrage
- Nicht mehr gelieferte veröffentlichte Posts bleiben erhalten und werden als nicht abrufbar markiert
- Pro Lauf genau ein `linkedin_sync_runs`-Datensatz und höchstens ein Follower-Snapshot
- Gleichzeitige Läufe werden verhindert; manuelle Starts haben fünf Minuten Cooldown
- Verwaiste `queued`-/`running`-Läufe werden nach zwei Stunden als fehlgeschlagen abgeschlossen
- Ein fataler Remote-Fehler verändert den letzten gültigen Geschäftsdatenstand nicht
- Access-/Refresh-Token-Rotation wird separat atomar gespeichert

Der Posts-Finder verwendet `viewContext=AUTHOR` und paginiert, bis eine Seite keine neue URN liefert. `paging.total` wird nicht als Abbruchkriterium verwendet. Einzelpost-Statistiken verwenden die verifizierte Rest.li-Form `shares=List(...)`: URNs sind URL-kodiert, die Klammern bleiben unkodiert.

## Admin-API

Alle Endpunkte erfordern `require_admin`:

```text
GET  /admin/marketing/linkedin/overview
GET  /admin/marketing/linkedin/posts
POST /admin/marketing/linkedin/sync
GET  /admin/marketing/linkedin/sync-runs/{id}
```

`overview` akzeptiert `days=7|30|90` oder gemeinsam `from=YYYY-MM-DD&to=YYYY-MM-DD`. `posts` ergänzt `tab`, `q`, `sort_by`, `sort_order`, `page` und eine maximale Seitengröße von 25. Alle API-Zeitstempel sind UTC; das Frontend zeigt sie in `Europe/Vienna` an.

Die KPI-Aggregation nimmt pro im Zeitraum veröffentlichtem Post nur dessen neuesten Snapshot. Historische Post-Snapshots werden nicht summiert. Die gewichtete Engagement-Rate ist:

```text
(Klicks + Likes + Kommentare + Shares) / Impressionen × 100
```

Bei null Gesamtimpressionen liefert die API `null`.

## Fixtures

Lokal aus `apps/api/`:

```powershell
$env:APP_ENV='dev'
.\venv\Scripts\python scripts\seed_linkedin_marketing_fixtures.py
```

Auf `dev-server` im API-Container denselben Befehl mit dessen `APP_ENV=dev` und Datenbankverbindung ausführen. Der Seed ist idempotent und bricht bei `APP_ENV=prod` vor dem Öffnen einer DB-Session ab.

Enthalten sind 121 tägliche Follower-Snapshots, organische/bezahlte Follower, mehrere Beiträge an einem Tag, null Impressionen, ein Post älter als 90 Tage, zwei geplante Posts, ein nicht mehr abrufbarer Post sowie ein erfolgreicher und ein später fehlgeschlagener Lauf.

## Gestufter Produktionsrollout

1. Aktuellen offiziellen `LinkedIn-Version`-Wert prüfen.
2. Datenbank-Backup über `normdex-backup` erstellen und SharePoint-Upload im Log bestätigen.
3. Mit `LINKEDIN_LIVE_SYNC_ENABLED=true` und `LINKEDIN_SCHEDULER_ENABLED=false` deployen.
4. Migration ausführen: `docker compose -f deploy/docker-compose.prod.yml --profile tools run --rm api-migrate`.
5. Credentials mit dem Bootstrap-Skript importieren und `--status` prüfen.
6. Manuellen Sync im Verwaltungsportal starten und Werte gegen das Standalone-Tool plausibilisieren.
7. Parallelstart, Cooldown, Fehlermeldungen und Token-Ablaufzeiten prüfen; Logs auf Secret-Freiheit kontrollieren.
8. Erst danach `LINKEDIN_SCHEDULER_ENABLED=true` setzen und API neu starten.
9. Den ersten automatischen 05:00-Uhr-Lauf prüfen.
10. Erst dann gezielt den alten LinkedIn-Cronjob des `deploy`-Users deaktivieren. Andere Cronjobs bleiben unverändert.
11. `fetch_stats.py`, `snapshot.py`, `history.jsonl` und alte Credentials mit restriktiven Rechten als inaktiven Fallback archivieren.

**Abweichung am 19.07.2026:** Auf ausdrückliche Nutzeranweisung wurde Schritt 10 vorgezogen. Der LinkedIn-Cronjob wurde entfernt, der übrige Crontab unverändert gelassen und unter `/home/deploy/tools/linkedin-orgpage/crontab-before-linkedin-disable-20260719T125709+0000.txt` gesichert. Bis zur produktiven Aktivierung des App-Schedulers gibt es deshalb keinen automatischen LinkedIn-Snapshot.

## Rollback

1. `LINKEDIN_SCHEDULER_ENABLED=false` und `LINKEDIN_LIVE_SYNC_ENABLED=false` setzen.
2. API neu starten; der letzte gültige Datenstand bleibt lesbar.
3. Anwendungsversion zurückrollen.
4. Datenbank-Downgrade nur nach frischem Backup und gemäß [[IT-Workflow Branches Deployments Backups Migrationen]] ausführen:

```bash
docker compose -f deploy/docker-compose.prod.yml --profile tools run --rm api-migrate downgrade -1
```

## Verifikation vom 19.07.2026

- Migration auf Dev-Sicherung: Upgrade → Downgrade → Upgrade erfolgreich
- Dev-Datenbank: Alembic-Head `a5c6d7e8f9b0`, Fixture-Prüfung ohne Warnung
- Backend: 438 Tests erfolgreich, davon 38 fokussierte LinkedIn-Tests
- Frontend: 53 Tests erfolgreich, davon 19 fokussierte Marketing-/Admin-Routen-Tests
- Frontend-Produktionsbuild erfolgreich
- Visuelle In-App-Browser-Abnahme in dieser Sitzung nicht möglich, weil kein In-App-Browser registriert war
- Produktive Referenzprüfung ohne Datenausgabe: REST-Version `202607`, Posts-Finder sowie beide `/v2/`-Statistikendpunkte erfolgreich; produktive Follower-Payload-Variante `followerCountsBySeniority` übernommen
- `dev-server`: Commit `37aa223`; validiertes PostgreSQL-Backup `normdex_dev_20260719_115702.dump`, Migration auf Head, idempotente Fixtures und neue Container erfolgreich
- Deployte Staging-API verifiziert: Health, Adminschutz, 7/30/90/Custom, KPIs, Follower/Marker, veröffentlichte/geplante Posts, Suche, Sortierung, direkte Marker-ID, Pagination sowie fachlich deaktivierter Sync
- Externe Dev-API liefert HTTP 200; Frontend bleibt erwartungsgemäß durch BasicAuth geschützt. Der ausgelieferte Marketing-Chunk enthält die erwarteten UI-Texte.
- Visuelle Nutzerabnahme auf `https://dev.normdex.at/admin/marketing` und Produktionsrollout noch offen
- Mehrkanal-Nachschärfung lokal verifiziert: 15 fokussierte Marketing-Tests, vollständige Frontend-Suite mit 53 Tests und Produktionsbuild erfolgreich
- Mehrkanal-Nachschärfung über `develop` (`e88494e`) nach `dev-server` (`d1a980f`) ausgerollt; beide CI-Gates grün, Frontend-Container laufend und ausgelieferter Marketing-Chunk mit LinkedIn, Google Ads, Google Analytics und Planungszustand verifiziert
- Nutzerfeedback zum Follower-Chart umgesetzt: redundante Post-Chipliste unter dem Diagramm entfernt; Marker, Tooltip, Tabellenverknüpfung und Beitragstabelle unverändert beibehalten; danach 15 fokussierte und 53 vollständige Frontend-Tests sowie Build erfolgreich
- Bereinigung auf `dev-server` mit Commit `cea1c4d` ausgerollt; CI grün und deploytes Bundle auf entfernte Chipliste sowie erhaltene Marker-/Tabellenhinweise geprüft
- Follower-Datentabelle nach Nutzerfeedback auf „neueste zuerst“ umgestellt, während die Diagrammzeitachse chronologisch bleibt; absteigende Sortierung semantisch ausgezeichnet und automatisiert geprüft
- Sortierkorrektur auf `dev-server` mit Commit `bb2d358` ausgerollt; CI, deploytes Bundle und Containerzustände erfolgreich verifiziert
- Nutzerabnahme abgeschlossen; Release `0.2.0` über `develop` → `dev-server` → `main` veröffentlicht und als `v0.2.0` getaggt. Alle Quality Gates und Secret Scans sind grün.
- Produktionsbackup `20260719_160644` einschließlich SharePoint-Upload verifiziert; Migration auf `a5c6d7e8f9b0`, API-/Frontend-Deployment und Healthchecks erfolgreich.
- Credentials sicher importiert und manueller Vollabruf erfolgreich. App und unabhängiges Standalone-Referenztool liefern identisch 1 Follower, 1 veröffentlichten und 2 geplante Posts sowie einen Post-Metrikdatensatz.
- Adminschutz, Overview, beide Post-Tabs, Cooldown, Parallelstartsperre und Secret-Freiheit der Logs produktiv geprüft.
- Scheduler produktiv aktiviert; Job `linkedin_marketing_daily_sync` ist für 05:00 Uhr `Europe/Vienna` registriert, nächster Lauf 20.07.2026 um 05:00 Uhr. Die Prüfung dieses ersten automatischen Laufs steht noch aus.
- Standalone-Dateien und Credentials als restriktiver Fallback unter `/home/deploy/tools/linkedin-orgpage/archive/20260719-pre-app-scheduler/` archiviert; der alte LinkedIn-Cronjob bleibt deaktiviert.
- Patch `0.2.1`: Der produktive 7-Tage-Zeitraum enthält derzeit nur einen Follower-Snapshot und keinen Post-Marker. Statt einer scheinbar leeren Diagrammfläche werden nun ein sichtbarer Einzelpunkt und ein erklärender Hinweis dargestellt. Vollständige Frontend-Suite, Build und CI-Gates erfolgreich; produktiv ausgeliefertes Bundle und Healthcheck verifiziert.
- Patch `0.2.2`: Den anschließend per Browser-Stacktrace nachgewiesenen Recharts-Absturz bei fehlender Marker-Payload behoben. Marker-Payload und `posts` werden vor jedem Zugriff validiert; bei null Markern wird die Scatter-Ebene ausgelassen. Regressionstest mit `payload={undefined}`, vollständige Frontend-Suite, Build und sämtliche Branch-Gates erfolgreich; Produktion läuft gesund auf `0.2.2`.

## Verwandte Dokumente

- [[API-Endpunkte]]
- [[Datenmodell]]
- [[App-Routen]]
- [[Integrationen & externe Dienste]]
- [[T047-linkedin-api-freigabe-statistiken]]
