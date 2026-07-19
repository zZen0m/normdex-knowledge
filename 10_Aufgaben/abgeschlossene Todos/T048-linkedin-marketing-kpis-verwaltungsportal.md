# T048 · LinkedIn-Marketing-KPIs im Verwaltungsportal anzeigen

**Phase:** App / Admin / Verwaltungsportal / Marketing  
**Priorität:** P3 · Wichtig für Wachstumsanalyse, kein Blocker für laufenden Betrieb  
**Status:** erledigt  
**Abgeschlossen:** 2026-07-19  
**Zuletzt aktualisiert:** 2026-07-19 · Nutzerabnahme bestätigt

## Ziel

Die Kennzahlen der LinkedIn-Unternehmensseite werden direkt im Normdex-Verwaltungsportal in einem nur für Admins sichtbaren Bereich „Marketing“ dargestellt. Die Seite verbindet den Follower-Verlauf mit den Veröffentlichungszeitpunkten der Posts und zeigt die jeweils neuesten Post-Kennzahlen, damit zeitliche Zusammenhänge zwischen Veröffentlichungen und Follower-Wachstum untersucht werden können.

Die Anzeige ist eine deskriptive Auswertung. Sie behauptet nicht, dass ein Post ein bestimmtes Follower-Wachstum verursacht hat.

## Kontext / Ist-Stand

- [[T047-linkedin-api-freigabe-statistiken]] hat den API-Zugriff und die Eigenheiten der LinkedIn-Endpunkte verifiziert.
- Auf dem VPS liegen unter `/home/deploy/tools/linkedin-orgpage/` derzeit `fetch_stats.py`, `snapshot.py`, `credentials.json`, `history.jsonl` und `cron.log`.
- `snapshot.py` wird seit 2026-07-19 täglich um 05:00 Uhr durch einen Cronjob des `deploy`-Users ausgeführt.
- Da Cronjob und Historisierung erst am 2026-07-19 eingerichtet wurden, gibt es keine relevante Althistorie. **`history.jsonl` wird nicht in die App-Datenbank importiert.**
- Das Standalone-Tool dient als verifizierte technische Referenz für Token-Refresh, API-Endpunkte, Paging und Query-Kodierung. Die produktive Datenhaltung soll vollständig in die App wechseln.
- Bekannte LinkedIn-Besonderheiten:
  - `/rest/posts?q=author&viewContext=AUTHOR` liefert veröffentlichte und geplante Posts.
  - Auf das unzuverlässige Paging-Feld `total` darf nicht vertraut werden; stattdessen bis zu einer Seite ohne neue Post-URNs paginieren.
  - Einzelpost-Statistiken verwenden `shares=List(...)`; URNs werden kodiert, die Klammern selbst nicht.
  - Geplante Posts haben `lifecycleState: PUBLISH_REQUESTED`, aber der tatsächliche geplante Veröffentlichungstermin wird über den verifizierten Endpunkt nicht geliefert.
  - Die REST-API benötigt einen gültigen `LinkedIn-Version`-Header; dessen Wert muss konfigurierbar sein.

## Verbindliche Produktentscheidungen

- Nur Produktion kommuniziert mit LinkedIn.
- Lokal und auf `dev-server` werden realistische Fixture-Daten verwendet; es gibt dort keine LinkedIn-Tokens und keine Live-Abfragen.
- Der automatische Produktionslauf erfolgt täglich um 05:00 Uhr in `Europe/Vienna`.
- Alle Datenbank-Zeitstempel werden in UTC gespeichert und im Frontend in `Europe/Vienna` angezeigt.
- Admins sehen Sync-Status, letzten erfolgreichen Lauf und verständliche Fehlerzustände und können in Produktion einen manuellen Hintergrund-Sync starten.
- Der manuelle Sync ist außerhalb der Produktion sichtbar, aber deaktiviert und mit „Nur in Produktion verfügbar“ erklärt.
- Client-ID, Client-Secret, Organisations-URN, REST-Version und Verschlüsselungsschlüssel liegen als Umgebungsvariablen vor.
- Access- und Refresh-Token werden verschlüsselt in der App-Datenbank gespeichert und bei einem Refresh atomar aktualisiert.
- Ersteinrichtung und erneute Autorisierung erfolgen über einen kontrollierten Backend-/VPS-Befehl, nicht über einen OAuth-Dialog im Frontend.
- Beim ersten Live-Lauf werden alle über die API verfügbaren Posts der letzten zwölf Monate übernommen.
- Metriken veröffentlichter Posts werden bis 90 Tage nach Veröffentlichung täglich aktualisiert. Ältere Posts bleiben unbegrenzt erhalten, werden aber nicht mehr täglich abgefragt.
- Follower- und Post-Snapshots haben keine automatische Retention.
- Geplante Posts erscheinen in einem eigenen Tab, fließen aber nicht in KPIs oder den Follower-Chart ein.
- Der bisherige VPS-Cronjob wird erst nach einem erfolgreich geprüften App-Lauf deaktiviert. Skripte und `history.jsonl` bleiben anschließend zunächst als inaktiver Diagnose-/Notfall-Fallback archiviert.

## Fachliche Definitionen

### Zeitraum

- Standard: letzte 30 Wiener Kalendertage.
- Auswahl: 7 Tage, 30 Tage, 90 Tage oder benutzerdefinierter Von-/Bis-Zeitraum.
- Der Zeitraum steuert KPI-Karten, Follower-Chart und die Tabelle veröffentlichter Posts gemeinsam.
- Bei Post-Kennzahlen bedeutet der Zeitraum: **Posts, die innerhalb des Zeitraums veröffentlicht wurden, jeweils mit ihrem neuesten verfügbaren Gesamtstand.** Snapshot-Werte mehrerer Tage dürfen nicht aufsummiert werden.

### KPI-Karten

1. Follower gesamt am Ende des Zeitraums
2. Follower-Zuwachs zwischen Beginn und Ende des Zeitraums
3. Impressionen der im Zeitraum veröffentlichten Posts
4. Klicks der im Zeitraum veröffentlichten Posts
5. Gewichtete Engagement-Rate
6. Anzahl der im Zeitraum veröffentlichten Posts

Die gewichtete Engagement-Rate lautet:

`(Summe Klicks + Summe Likes + Summe Kommentare + Summe Shares) / Summe Impressionen × 100`

Bei insgesamt null Impressionen wird keine künstliche Rate von `0 %`, sondern „–“ bzw. `null` ausgegeben.

### Follower-Chart

- Hervorgehobene Standardlinie: Follower gesamt.
- Optional einblendbar: organische und bezahlte Follower.
- Tooltip: lokales Datum, Gesamtwert, organischer/bezahlter Anteil und Veränderung zum vorherigen Snapshot.
- Veröffentlichte Posts werden als Marker am Veröffentlichungstag angezeigt.
- Mehrere Posts desselben Tages werden zu einem Marker zusammengefasst.
- Marker-Tooltip: Uhrzeit, Textanfang und neueste Impressionen.
- Klick auf einen Post im Marker scrollt zur veröffentlichten Post-Tabelle und hebt die Zeile hervor.
- Geplante Posts erscheinen nicht im historischen Chart.
- Nicht mehr abrufbare veröffentlichte Posts behalten ihren historischen Marker und werden gekennzeichnet.
- Es wird kein „Post Impact“, keine Kausalitätsbewertung und kein automatisch zugerechneter Follower-Gewinn berechnet.

### Post-Sektion

**Tab „Veröffentlicht“**

- Spalten: Veröffentlichungsdatum, Status, Textanfang, Impressionen, eindeutige Impressionen, Klicks, Likes, Kommentare, Shares und Engagement-Rate.
- Standardsortierung: neueste Veröffentlichung zuerst.
- Sortierung nach Datum und allen Leistungskennzahlen.
- Suche im Beitragstext.
- 25 Einträge pro Seite mit Pagination.
- Klick auf eine Zeile öffnet den Beitrag auf LinkedIn in einem neuen Tab.

**Tab „Geplant“**

- Zeigt Textanfang, Status, Anlagezeitpunkt und LinkedIn-Link.
- Zeigt ausdrücklich „Termin wird von LinkedIn nicht bereitgestellt“ statt eines erfundenen Veröffentlichungstermins.
- Hat keine leeren Performance-Spalten.
- Nach Veröffentlichung wird derselbe Beitrag anhand seiner URN im Tab „Veröffentlicht“ weitergeführt.

## Zielarchitektur und Datenmodell

Es werden fünf getrennte Tabellen angelegt. Die endgültigen SQLAlchemy-Typen, Fremdschlüssel, Indizes und Constraint-Namen sind in der Migration explizit festzuhalten.

### 1. `linkedin_connections`

- eine Zeile für die Normdex-Unternehmensseite
- Organisations-URN
- verschlüsselter Access-Token
- verschlüsselter Refresh-Token
- Ablaufzeitpunkte beider Tokens
- Verbindungsstatus, z. B. `active`, `reauthorization_required`, `disabled`
- Zeitpunkt des letzten erfolgreichen Token-Refreshs
- `created_at`, `updated_at`
- niemals Client-Secret oder Verschlüsselungsschlüssel in dieser Tabelle speichern

### 2. `linkedin_posts`

- interne ID und eindeutige LinkedIn-Post-URN
- Lifecycle-Status
- Beitragstext bzw. benötigter Textinhalt
- LinkedIn-URL
- LinkedIn-Anlagezeitpunkt
- tatsächlicher Veröffentlichungszeitpunkt, soweit von der API geliefert
- erster und letzter erfolgreicher Sichtkontakt
- Kennzeichen für einen inzwischen nicht mehr abrufbaren Post
- `created_at`, `updated_at`

### 3. `linkedin_post_snapshots`

- Fremdschlüssel auf `linkedin_posts`
- Fremdschlüssel auf `linkedin_sync_runs`
- Messzeitpunkt
- Impressionen
- eindeutige Impressionen
- Klicks
- Likes
- Kommentare
- Shares
- von LinkedIn gelieferte Engagement-Rate optional als Roh-/Vergleichswert
- Eindeutigkeits-Constraint pro Post und Sync-Lauf
- Index auf Post-ID und Messzeitpunkt

### 4. `linkedin_follower_snapshots`

- Fremdschlüssel auf `linkedin_sync_runs`
- Messzeitpunkt
- organische Follower
- bezahlte Follower
- Gesamtzahl
- höchstens ein Follower-Snapshot pro Sync-Lauf
- Index auf Messzeitpunkt

### 5. `linkedin_sync_runs`

- Trigger: `scheduled` oder `manual`
- Status: `queued`, `running`, `success`, `partial`, `failed`
- Start- und Endzeitpunkt
- auslösender Admin bei manuellen Läufen, sonst `null`
- Anzahl gefundener, aktualisierter und fehlgeschlagener Posts
- sanitierte Fehlerkategorie und benutzergeeignete Kurzmeldung
- technische Details ausschließlich in den Serverlogs
- `created_at`, `updated_at`

## Umsetzungsschritte

### Phase 1 · Technische Grundlage und Konfiguration

- [ ] Vor jeder Alembic-Arbeit `.claude/skills/db-migration/SKILL.md` vollständig lesen und dessen Backup-, Upgrade-, Downgrade- und Produktionsprozess befolgen.
- [ ] Das verifizierte VPS-Skript lesen und die verwendeten Scopes, Endpunkte, Header, Paging-Abbruchbedingungen, Query-Kodierung und Token-Felder dokumentiert in den App-Service übertragen.
- [ ] Vor der Implementierung die aktuell akzeptierte LinkedIn-REST-Version prüfen; keinen dauerhaft angenommenen Versionswert ausschließlich im Code festschreiben.
- [ ] Konfiguration in `apps/api/app/config.py` ergänzen:
  - `LINKEDIN_CLIENT_ID`
  - `LINKEDIN_CLIENT_SECRET`
  - `LINKEDIN_ORGANIZATION_URN`
  - `LINKEDIN_REST_VERSION`
  - `LINKEDIN_TOKEN_ENCRYPTION_KEY`
  - `LINKEDIN_LIVE_SYNC_ENABLED`
  - `LINKEDIN_SCHEDULER_ENABLED`
- [ ] Platzhalter und Erläuterungen in den passenden Dateien unter `deploy/env/` ergänzen; keine realen Secrets versionieren.
- [ ] Startup-Validierung definieren: Scheduler darf nur bei `APP_ENV=prod`, aktiviertem Live-Sync und vollständig vorhandener Konfiguration registriert werden.
- [ ] Authentifizierte Verschlüsselung für Tokens implementieren; benötigte Abhängigkeit in `apps/api/requirements.txt` explizit ergänzen, falls sie nicht bereits direkt verfügbar ist.
- [ ] Sicherstellen, dass Tokens, Auth-Codes und Client-Secret weder in Logs noch in API-Antworten oder Exceptions erscheinen.

### Phase 2 · Modelle und Migration

- [ ] Die fünf Modelle in `apps/api/app/models.py` ergänzen.
- [ ] Beziehungen, UTC-Zeitstempel, Eindeutigkeits-Constraints und Indizes für Zeitbereichsabfragen anlegen.
- [ ] Alembic-Migration mit Upgrade und belastbarem Downgrade erzeugen und manuell prüfen.
- [ ] Migration auf einer Dev-Datenbanksicherung vorwärts und rückwärts testen.
- [ ] Falls `apps/api/dev.db` durch Fixtures verändert und committed wird, vom Repo-Root `.\apps\api\venv\Scripts\python apps\api\scripts\verify_dev_fixture.py` ausführen und gemeldete Risiken prüfen.

### Phase 3 · Token-Bootstrap und erneute Autorisierung

- [ ] Kontrolliertes Skript unter `apps/api/scripts/` anlegen, das die bestehende `/home/deploy/tools/linkedin-orgpage/credentials.json` über einen Dateipfad einliest, validiert, verschlüsselt und in `linkedin_connections` upsertet.
- [ ] Token-Werte nicht als Kommandozeilenparameter verlangen, damit sie nicht in Shell-History oder Prozesslisten landen.
- [ ] Das Skript darf nur Status und Ablaufzeiten ausgeben, niemals Token-Inhalte.
- [ ] Derselbe Ablauf muss für eine spätere erneute Autorisierung verwendbar sein.
- [ ] Eine reine Statusprüfung bereitstellen: Verbindung vorhanden, Access-Token gültig bis, Refresh-Token gültig bis, erneute Autorisierung erforderlich ja/nein.

### Phase 4 · LinkedIn-Client und Synchronisierungsservice

- [ ] `apps/api/app/services/linkedin_service.py` anlegen und HTTP-Kommunikation, Token-Handling und DB-Persistenz voneinander trennen.
- [ ] Access-Token rechtzeitig vor Ablauf erneuern und Access-/Refresh-Token samt Ablaufzeiten in derselben DB-Transaktion aktualisieren.
- [ ] Bei nicht erneuerbarem Token Verbindung auf `reauthorization_required` setzen und eine verständliche UI-Fehlerkategorie erzeugen.
- [ ] Follower-Statistik abrufen und organischen, bezahlten und gesamten Stand normalisieren.
- [ ] Posts mit `viewContext=AUTHOR` vollständig paginieren und anhand der URN idempotent upserten.
- [ ] Paging beenden, sobald eine Seite keine neuen URNs liefert; nicht auf `paging.total` verlassen.
- [ ] Einzelpost-Statistiken nur für veröffentlichte Posts abrufen und `shares=List(...)` korrekt kodieren.
- [ ] Beim ersten Sync veröffentlichte Posts der letzten zwölf Monate übernehmen.
- [ ] Bei Folgesyncs Kennzahlen nur für Posts bis 90 Tage nach Veröffentlichung neu abrufen.
- [ ] Geplante Posts ohne Performance-Abfrage speichern; unbekannten Termin nicht selbst ableiten.
- [ ] Nicht mehr gelieferte veröffentlichte Posts nicht löschen, sondern als nicht mehr abrufbar markieren.
- [ ] Pro Sync genau einen `linkedin_sync_runs`-Datensatz führen.
- [ ] Erfolgreich abgerufene Geschäftsdaten atomar persistieren; ein fataler Lauf darf den letzten gültigen Datenstand nicht beschädigen.
- [ ] Technische Fehler mit bestehendem Systemlog-/Audit-Muster protokollieren und API-/Token-Inhalte sanitisieren.
- [ ] Administrativen Backend-Befehl für einen vollständigen Neuabruf älterer Posts vorsehen; nicht als normalen UI-Button anbieten.

### Phase 5 · Scheduler und manueller Hintergrund-Sync

- [ ] Produktionsjob in `apps/api/app/services/scheduler.py` nach bestehendem APScheduler-Muster registrieren.
- [ ] `CronTrigger(hour=5, minute=0, timezone="Europe/Vienna")`, feste Job-ID, `replace_existing=True`, sinnvolle `misfire_grace_time` und `max_instances=1` verwenden.
- [ ] Scheduler-Registrierung zusätzlich mit `LINKEDIN_SCHEDULER_ENABLED` absichern.
- [ ] Gemeinsame Sperre für Scheduler und manuellen Trigger implementieren, sodass maximal ein LinkedIn-Sync gleichzeitig läuft.
- [ ] Verwaiste `running`-Zustände nach Prozessabbruch sicher erkennen und als fehlgeschlagen abschließen.
- [ ] Fünf Minuten Cooldown zwischen manuellen Starts erzwingen.
- [ ] Manuellen Sync als Hintergrundjob starten; ein Seitenwechsel darf ihn nicht abbrechen.
- [ ] Bestehende Werte bei Fehlern weiter ausliefern und den fehlgeschlagenen Lauf separat anzeigen.

### Phase 6 · Admin-API

- [ ] Einen fokussierten Router, vorzugsweise `apps/api/app/routers/admin_marketing.py`, unter `/admin/marketing/linkedin` anlegen und in `apps/api/app/main.py` registrieren.
- [ ] Alle Endpunkte mit `Depends(require_admin)` schützen; normale Benutzer dürfen weder Daten noch Status erhalten.
- [ ] `GET /overview` umsetzen: Verbindungs-/Sync-Status, KPI-Werte, Follower-Zeitreihe und gruppierbare Post-Marker für den gewählten Zeitraum.
- [ ] `GET /posts` umsetzen: Tab/Status, Suche, serverseitige Sortierung, Seite, Seitengröße 25 und Gesamtzahl.
- [ ] `POST /sync` umsetzen: nur Produktion mit aktiviertem Live-Sync; Rückgabe `202` und Sync-Run-ID.
- [ ] `GET /sync-runs/{id}` für das Frontend-Polling umsetzen.
- [ ] Aussagekräftige Antworten definieren: laufender Sync bzw. Cooldown `409`, fehlende/ungültige Konfiguration `503`, Nicht-Produktion als fachlich verständlicher deaktivierter Zustand.
- [ ] Nur sanitierte Fehlermeldungen ausgeben; keine Tokens, Secrets, kompletten LinkedIn-Responses oder internen Tracebacks.
- [ ] Alle Zeitstempel als ISO-8601 in UTC liefern.
- [ ] KPI-Aggregation zentral im Backend implementieren:
  - neuester Snapshot je gefiltertem Post
  - keine Summierung historischer Snapshots
  - geplante Posts ausgeschlossen
  - gewichtete Engagement-Rate nach der festgelegten Formel
  - Follower-Delta anhand der letzten verfügbaren Snapshots an den Zeitraumgrenzen

### Phase 7 · Fixture-Daten für lokal und `dev-server`

- [ ] Idempotentes Seed-Skript `apps/api/scripts/seed_linkedin_marketing_fixtures.py` anlegen.
- [ ] Skript hart gegen `APP_ENV=prod` absichern; ein versehentlicher Produktionslauf muss ohne Schreibzugriff abbrechen.
- [ ] Mindestens 120 Tage realistische Follower-Snapshots erzeugen, damit 7-/30-/90-Tage-Ansichten prüfbar sind.
- [ ] Fixtures für organische und bezahlte Follower, mehrere Posts an einem Tag, Posts mit null Impressionen, Posts älter als 90 Tage, zwei geplante Posts und einen nicht mehr abrufbaren veröffentlichten Post anlegen.
- [ ] Einen aktuellen erfolgreichen sowie einen später fehlgeschlagenen Sync-Lauf mit weiterhin sichtbaren Bestandsdaten abbilden.
- [ ] Seed-Prozess für lokale SQLite-DB und `dev-server`-Postgres dokumentieren.
- [ ] Außerhalb der Produktion beide Sync-Schalter deaktiviert lassen; Fixture-Daten werden ausschließlich aus der lokalen Datenbank gelesen.

### Phase 8 · Frontend-Seite

- [ ] API-Typen und Client-Funktionen in der vorhandenen Frontend-API-Struktur ergänzen.
- [ ] `apps/frontend/src/pages/admin/Marketing.tsx` anlegen und nach Bedarf in fokussierte LinkedIn-Unterkomponenten aufteilen.
- [ ] Route `/admin/marketing` lazy-loaded unter `<AdminRoute />` in `apps/frontend/src/App.tsx` registrieren.
- [ ] In `Sidebar.tsx` nach „Ticket Portal“ den Admin-Eintrag „Marketing“ mit Megafon-Icon ergänzen.
- [ ] Seitentitel „Marketing“ verwenden und LinkedIn als ersten Kanalabschnitt gestalten, damit weitere Kanäle später ergänzt werden können.
- [ ] Statusbereich umsetzen: Verbindung, letzter erfolgreicher Lauf, letzter Versuch, laufender Sync und Warnung bei mehr als 36 Stunden ohne erfolgreichen Lauf.
- [ ] „Jetzt synchronisieren“ nur in Produktion aktivieren; in Dev und auf `dev-server` deaktiviert mit erklärendem Tooltip darstellen.
- [ ] Manuellen Sync starten, Status pollen, Button während des Laufs sperren und Daten nach Erfolg automatisch neu laden.
- [ ] Bei nicht erneuerbarem Token ausdrücklich „LinkedIn muss neu autorisiert werden“ anzeigen.
- [ ] Gemeinsamen Zeitraumfilter für 7, 30, 90 Tage und benutzerdefinierten Zeitraum umsetzen; Standard 30 Tage.
- [ ] Sechs KPI-Karten mit `KPICard` oder einer kompatiblen, zugänglichen Variante umsetzen.
- [ ] Recharts-Follower-Chart mit Gesamtlinie, umschaltbaren organischen/bezahlten Linien, Tooltips und gruppierten Post-Markern umsetzen.
- [ ] Marker-Klick mit Scrollen und temporärer Hervorhebung der zugehörigen Tabellenzeile verbinden.
- [ ] Post-Sektion mit Tabs „Veröffentlicht“ und „Geplant“ umsetzen.
- [ ] Veröffentlichte Posts: Suche, serverseitige Sortierung, Pagination und LinkedIn-Link umsetzen.
- [ ] Geplante Posts: fehlenden Veröffentlichungstermin transparent erklären und keine Performance-Spalten zeigen.
- [ ] Lade-, Leer-, Fehler-, veralteter-Daten- und Teilfehlerzustände gestalten.
- [ ] Responsive Darstellung und Tastaturbedienbarkeit prüfen; Chart-Inhalte zusätzlich textlich bzw. tabellarisch zugänglich halten.

### Phase 9 · Automatisierte Tests

**Backend**

- [ ] Token-Ver- und Entschlüsselung testen und sicherstellen, dass DB-Werte kein Klartext sind.
- [ ] Token-Refresh kurz vor Ablauf, rotierende Token-Werte und den Zustand `reauthorization_required` testen.
- [ ] LinkedIn-Paging mit unzuverlässigem `total`, doppelten URNs und leerer Folgeseite testen.
- [ ] Exakte `shares=List(...)`-Kodierung als Regressionstest festhalten.
- [ ] Trennung veröffentlichter und geplanter Posts testen.
- [ ] Initialfenster zwölf Monate und tägliches Aktualisierungsfenster 90 Tage testen.
- [ ] Idempotentes Upsert und Eindeutigkeits-Constraints testen.
- [ ] KPI-Formeln, null Impressionen, neuesten Snapshot je Post und Zeitraumgrenzen testen.
- [ ] Fehlerfall testen: letzter gültiger Datenstand bleibt erhalten, Sync-Lauf wird fehlgeschlagen.
- [ ] Gleichzeitigen manuellen/Scheduler-Start und Fünf-Minuten-Cooldown testen.
- [ ] Scheduler-Gates prüfen: kein LinkedIn-Job außerhalb Produktion oder bei deaktivierter Einstellung.
- [ ] Admin-Endpunkte auf `401`/`403`, Validierungsfehler, Pagination, Sortierung und sanitierte Fehler prüfen.
- [ ] Fixture-Seed auf Idempotenz und Produktionssperre testen.

**Frontend**

- [ ] Admin-Navigation und geschützte Route testen.
- [ ] KPI-, Zeitraum- und Statusdarstellung mit gemockten API-Antworten testen.
- [ ] Deaktivierten Sync-Button außerhalb Produktion sowie laufenden, erfolgreichen und fehlgeschlagenen Sync testen.
- [ ] Chart-Linien, gruppierte Marker und Tabellenzeilen-Verknüpfung testen.
- [ ] Beide Post-Tabs, Suche, Sortierung und Pagination testen.
- [ ] Geplanten Post ohne verfügbaren Veröffentlichungstermin testen.
- [ ] Lade-, Leer-, Fehler- und veralteten Datenzustand testen.

### Phase 10 · Manuelle Prüfung und Rollout

- [ ] Backend-Tests mit `\.\venv\Scripts\python -m pytest` aus `apps/api/` ausführen.
- [ ] Frontend-Tests und Build mit `npm test` und `npm run build` aus `apps/frontend/` ausführen.
- [ ] Migration lokal auf leerer DB und vorhandener Dev-DB prüfen.
- [ ] Auf `dev-server` migrieren, Fixtures seeden und alle Zeiträume, KPI-Karten, Chart-Marker, Tabs, Suche, Sortierung, Pagination und deaktivierten Sync-Button abnehmen.
- [ ] Produktionsdatenbank nach DB-Migration-Skill sichern.
- [ ] Produktion zunächst mit `LINKEDIN_LIVE_SYNC_ENABLED=true` und `LINKEDIN_SCHEDULER_ENABLED=false` deployen.
- [ ] Migration anwenden und vorhandene Credentials über das sichere Bootstrap-Skript importieren.
- [ ] Manuellen Produktions-Sync auslösen und Follower, veröffentlichte/geplante Posts, Post-Metriken, Token-Ablaufzeiten und UI-Anzeige gegen das Standalone-Skript plausibilisieren.
- [ ] Parallelstart, Cooldown und einen kontrollierten Fehlerzustand prüfen, ohne echte Secrets zu protokollieren.
- [ ] Erst nach erfolgreicher manueller Prüfung `LINKEDIN_SCHEDULER_ENABLED=true` setzen und Anwendung neu starten.
- [ ] Den ersten automatischen 05:00-Uhr-Lauf prüfen.
- [ ] Danach den alten `deploy`-Cronjob gezielt deaktivieren; andere Cronjobs unverändert lassen.
- [ ] `fetch_stats.py`, `snapshot.py`, `history.jsonl` und alte Credentials zunächst mit bisherigen restriktiven Dateirechten als inaktiven Fallback archivieren; nicht weiter in `history.jsonl` schreiben.
- [ ] Rollback dokumentieren: beide LinkedIn-Schalter deaktivieren, alten gültigen Datenstand weiter anzeigen, Anwendungsversion zurückrollen; Datenbank-Downgrade nur gemäß Migration-Skill und mit Sicherung.
- [ ] `docs/features/FEATURES.md` und Aufgabenstatus erst nach Nutzerabnahme auf „Erledigt“ setzen.

## Akzeptanzkriterien

- [ ] Nur Admins sehen Navigation, Seite und LinkedIn-Admin-Endpunkte.
- [ ] Produktion erfasst Follower und Post-Statistiken täglich um 05:00 Uhr in `Europe/Vienna` in der App-Datenbank.
- [ ] Ein manueller Produktions-Sync läuft im Hintergrund, zeigt seinen Status und verhindert parallele bzw. zu häufige Starts.
- [ ] Lokal und `dev-server` zeigen realistische Fixture-Daten und führen keine LinkedIn-Live-Abfragen aus.
- [ ] Tokens liegen verschlüsselt in der DB; Client-Secret und Verschlüsselungsschlüssel bleiben ausschließlich in Produktions-Secrets.
- [ ] Der Zeitraumfilter 7/30/90/benutzerdefiniert steuert KPIs, Chart und veröffentlichte Posts konsistent.
- [ ] KPIs verwenden den neuesten Snapshot pro Post und die definierte gewichtete Engagement-Formel.
- [ ] Das Follower-Diagramm zeigt Gesamt-, organische und bezahlte Werte sowie gruppierte Veröffentlichungsmarker.
- [ ] Veröffentlichte Posts sind suchbar, sortierbar, paginiert und mit LinkedIn verlinkt.
- [ ] Geplante Posts erscheinen getrennt, ohne Performance-KPIs und ohne erfundenen Veröffentlichungstermin.
- [ ] Fehler beschädigen den letzten gültigen Datenstand nicht; UI zeigt letzten Erfolg, verständlichen Status und ab 36 Stunden eine Warnung.
- [ ] Posts werden initial zwölf Monate übernommen, bis 90 Tage täglich aktualisiert und danach ohne automatische Löschung aufbewahrt.
- [ ] Migration, automatisierte Tests, Frontend-Build, Dev-Server-Abnahme und Produktionsvorgehen sind erfolgreich dokumentiert und durchgeführt.
- [ ] Der alte VPS-Cronjob wird erst nach erfolgreichem App-Scheduler-Lauf deaktiviert.

## Nicht Bestandteil von T048

- Posts erstellen, bearbeiten, planen, veröffentlichen oder löschen
- OAuth-Verbindungsdialog im Verwaltungsportal
- E-Mail- oder externe Benachrichtigungen bei Sync-Fehlern
- CSV-/Excel-Export
- mehrere LinkedIn-Unternehmensseiten
- Live-LinkedIn-Daten auf lokalem System oder `dev-server`
- Import der am 2026-07-19 begonnenen `history.jsonl`
- automatische Kausalitäts- oder „Post Impact“-Bewertung des Follower-Wachstums

## Notizen / Fortschritt

### 2026-07-19 · Technische Umsetzung abgeschlossen

- Fünf SQLAlchemy-Modelle und Alembic-Revision `a5c6d7e8f9b0` mit expliziten Fremdschlüsseln, Eindeutigkeits-Constraints, Indizes und belastbarem Downgrade umgesetzt.
- Migration gemäß Database-Migration-Skill vorab auf einer Dev-Sicherung mit `upgrade → downgrade → upgrade` geprüft; danach `dev.db` migriert und Fixture-Risikoprüfung ohne Warnung ausgeführt.
- Konfiguration und Env-Beispiele ergänzt. Aktuelle offizielle LinkedIn-REST-Version am 19.07.2026 geprüft: `202607`; Wert bleibt konfigurierbar.
- Fernet-Verschlüsselung, sicherer Credentials-Datei-Import, reine Statusprüfung und administrativer Vollabruf umgesetzt.
- LinkedIn-Client mit Token-Rotation, Follower-/Post-Abruf, Paging ohne `total`, korrekter `shares=List(...)`-Kodierung, 12-Monats-Initialfenster und 90-Tage-Metrikfenster umgesetzt.
- Persistenter Sync-Workflow mit Scheduler-/Manuell-Sperre, fünf Minuten Cooldown, verwaisten Laufzuständen, Hintergrundausführung und sanitierter Fehlerpersistenz umgesetzt.
- Produktions-Scheduler um 05:00 Uhr `Europe/Vienna` mit vollständigen Env-Gates umgesetzt.
- Admin-API für Overview, Posts, Sync-Start und Polling umgesetzt; alle Endpunkte über `require_admin` geschützt.
- Idempotenter, gegen Produktion hart gesperrter Fixture-Seed mit 121 Tagen, organischen/bezahlten Followern sowie allen definierten Post-/Fehlerfällen umgesetzt und in `dev.db` angewendet.
- Lazy-loaded Admin-Route `/admin/marketing`, Sidebar-Eintrag, Statusbereich, sechs KPI-Karten, gemeinsamer Zeitraum, Follower-Chart, umschaltbare Linien, gruppierte Post-Marker, Tabellenverknüpfung sowie veröffentlichte/geplante Tabs umgesetzt.
- Dokumentation erstellt: [[LinkedIn-Marketing-KPIs]]; API-, Datenmodell-, Routen- und Integrationsdokumentation aktualisiert.

### Verifikation

- Backend: `438 passed`, davon `38` fokussierte LinkedIn-Tests.
- Frontend: `53 passed`, davon `19` fokussierte Marketing-/Admin-Routen-Tests.
- Frontend-Produktionsbuild erfolgreich.
- Alembic-Graph: 45 Revisionen, einzelner Head `a5c6d7e8f9b0`, `dev.db` aligned.
- `verify_dev_fixture.py`: keine Warnungen; nur `example.com` als E-Mail-Domain.
- In-App-Browser-Abnahme in dieser Sitzung nicht möglich, weil kein Browser registriert war.
- Abschluss-Audit hat eine fehlerhaft platzierte Zeitstempel-Konvertierung entdeckt und behoben; ISO-Offsets und LinkedIn-Epoch-Millisekunden werden nun nach UTC normalisiert und direkt getestet.
- Produktive Referenz ohne Secret-Ausgabe erneut geprüft: REST-Version `202607`, Posts-Finder, Follower- und Share-Statistik erfolgreich; `token_expires_at` und `followerCountsBySeniority` aus dem tatsächlichen VPS-Tool übernommen.
- Dependency-Audits: Frontend ohne Befund; behebbarer Python-Befund durch `cryptography 48.0.1` und `Pillow 12.3.0` geschlossen. Der transitive ECDSA-Befund ist im Quality-Gate dokumentiert ausgenommen, da Normdex ausschließlich `HS256` verwendet.

### 2026-07-19 · Dev-Server-Rollout

- Branch-Workflow eingehalten: `develop` (`00698fc`) → `dev-server` (`37aa223`); GitHub Quality Gates und Secret Scan auf beiden Ständen erfolgreich.
- Vor der Migration PostgreSQL-Custom-Backup `/opt/stacks/normdex-dev/backups/normdex_dev_20260719_115702.dump` erstellt, mit `pg_restore -l` validiert und auf Dateimodus `600` gesetzt.
- Dev-Repo auf `37aa223` aktualisiert, API-/Frontend-Images gebaut, Migration im isolierten Compose-Container auf `a5c6d7e8f9b0` angewendet und Fixtures geseedet.
- `APP_ENV=dev`, `LINKEDIN_LIVE_SYNC_ENABLED=false` und `LINKEDIN_SCHEDULER_ENABLED=false` im tatsächlichen Dev-Environment verifiziert.
- Container gesund; externe `https://dev-api.normdex.at/health` liefert HTTP 200. Das Dev-Frontend bleibt erwartungsgemäß durch BasicAuth geschützt.
- Deployte API-Smokes erfolgreich: Adminzugriff, 401/403, 7/30/90/Custom, KPIs, 30 Followerpunkte, 8 Marker, 25 veröffentlichte Posts in 90 Tagen, 2 geplante Posts, Suche/Sortierung, direkte Marker-ID, Dev-Sync 503 sowie 34 Posts mit zwei Pagination-Seiten im erweiterten Zeitraum.
- Frontend-Container enthält den lazy geladenen Marketing-Chunk samt Sync- und Termin-Hinweisen; visuelle Nutzerabnahme bleibt offen, weil in dieser Sitzung kein In-App-Browser registriert ist.

### 2026-07-19 · Nachschärfung aus der Nutzerabnahme

- Erste visuelle Rückmeldung ist positiv; die Nutzerabnahme läuft weiter und ist noch nicht abgeschlossen.
- Die geplante Mehrkanal-Architektur wird in der Oberfläche sichtbar gemacht: zugänglicher Umschalter für LinkedIn, Google Ads und Google Analytics; Google-Kanäle zeigen bis zur echten Anbindung einen transparenten Planungszustand ohne erfundene Daten.
- Die vollständigen Google-Ads- und Google-Analytics-Integrationen werden als separates Todo [[T049-google-marketing-kpis-verwaltungsportal]] geführt.
- Fokussierte Marketing-Tests nach der Nachschärfung: `15 passed`; vollständige Frontend-Suite: `53 passed`; Produktionsbuild erfolgreich.
- Branch-Workflow für die Nachschärfung: `develop` (`e88494e`) → `dev-server` (`d1a980f`); Quality Gates und Secret Scan auf beiden Branches erfolgreich.
- Staging-Frontend auf `d1a980f` neu gebaut und ausgerollt. Container läuft, Dev-API bleibt gesund, externer BasicAuth-Schutz liefert erwartungsgemäß HTTP 401 und der ausgelieferte Marketing-Chunk enthält alle drei Kanalnamen sowie den transparenten Planungszustand.
- Die redundante blaue Chipliste aller Posts unter dem Follower-Diagramm wurde auf Nutzerfeedback entfernt. Diagramm-Marker, Marker-Tooltip, Tabellenverknüpfung, zugängliche Follower-Datentabelle und die vollständige Beitragstabelle bleiben erhalten.
- Nach dieser UI-Nachschärfung erneut verifiziert: `15` fokussierte Marketing-Tests, vollständige Frontend-Suite mit `53 passed` und Produktionsbuild erfolgreich.
- Diagramm-Bereinigung über `develop` (`b5ce497`) nach `dev-server` (`cea1c4d`) ausgerollt; Quality Gates und Secret Scan grün. Der deployte Chunk enthält weiterhin Markerhinweis und zugängliche Datentabelle, aber nicht mehr die entfernte Chipliste; Frontend läuft und Dev-API bleibt gesund.
- Follower-Datentabelle auf Nutzerfeedback unabhängig vom chronologisch aufsteigenden Diagramm nach Messzeitpunkt absteigend sortiert und die Datumsspalte mit `aria-sort="descending"` gekennzeichnet; Regressionstest, vollständige Frontend-Suite (`53 passed`) und Build erfolgreich.
- Sortierkorrektur über `develop` (`7c1dbfa`) nach `dev-server` (`bb2d358`) ausgerollt; beide CI-Gates grün. Deploytes Bundle enthält die absteigende Sortierkennzeichnung, die entfernte Chipliste bleibt abwesend, Frontend läuft und Dev-API ist gesund.
- Auf ausdrückliche Nutzeranweisung wurde der alte LinkedIn-Cronjob vorgezogen deaktiviert. Der übrige Crontab blieb unverändert; Sicherung: `/home/deploy/tools/linkedin-orgpage/crontab-before-linkedin-disable-20260719T125709+0000.txt` mit Dateimodus `600`.
- Abweichung von der ursprünglichen Rolloutreihenfolge: Bis der App-Scheduler produktiv aktiviert und geprüft ist, findet kein automatischer LinkedIn-Snapshot statt.

### 2026-07-19 · Nutzerabnahme und Produktionsrollout

- Nutzerabnahme abgeschlossen. Die abschließenden Rückmeldungen zur entfernten Post-Chipliste und zur absteigenden Sortierung der Follower-Datentabelle wurden umgesetzt und auf `dev-server` geprüft.
- Release-Version auf `0.2.0` angehoben und über `develop` (`9f86fa0`) → `dev-server` (`836fb67`) → `main` (`9364eeb`) ausgerollt; Quality Gates und Secret Scan auf allen drei Branches erfolgreich. Release-Tag: `v0.2.0`.
- Staging nach dem Versionssprung erneut gebaut und als `0.2.0` gesund verifiziert.
- Vor der Produktionsmigration vollständiges Backup `20260719_160644` erstellt. Datenbankdump und Upload-Archiv sind lokal vorhanden; der SharePoint-Upload beider Dateien wurde über Log und Remote-Auflistung bestätigt.
- Produktive Konfiguration mit Live-Sync und zunächst deaktiviertem Scheduler eingerichtet; Env-Datei und Sicherungen auf Dateimodus `600` gesetzt. Keine Secrets wurden ausgegeben oder versioniert.
- Alembic-Revision `a5c6d7e8f9b0` erfolgreich auf Produktion angewendet; API und Frontend laufen gesund auf `0.2.0`.
- Credentials kontrolliert importiert. Verbindung ist `active`, Access-Token bis 2026-09-17 und Refresh-Token bis 2027-07-19 gültig; erneute Autorisierung ist nicht erforderlich.
- Manueller Vollabruf erfolgreich: 1 Follower, 3 Posts (1 veröffentlicht, 2 geplant), 1 Post-Metrikdatensatz und 0 Fehler. Die Werte stimmen exakt mit einem unabhängigen Abruf über das archivierte Standalone-Tool und REST-Version `202607` überein.
- Produktive Admin-API geprüft: unauthentifiziert `401`, authentifiziert Overview sowie beide Post-Tabs `200`; Cooldown und Parallelstartsperre greifen. Access-/Refresh-Token und Client-Secret kommen nicht in den Containerlogs vor.
- Scheduler anschließend aktiviert. Konfigurations-Gate ohne Fehler; Job `linkedin_marketing_daily_sync` mit `cron[hour='5', minute='0']`, `max_instances=1`, 3600 Sekunden Misfire-Frist und nächstem Lauf am 20.07.2026 um 05:00 Uhr `Europe/Vienna` registriert.
- Standalone-Dateien nach erfolgreichem App-Lauf nach `/home/deploy/tools/linkedin-orgpage/archive/20260719-pre-app-scheduler/` verschoben und einschließlich Credentials auf Dateimodus `600` gesetzt. Im Crontab verbleibt ausschließlich der unveränderte Landingpage-Job.
- Produktives Marketing-Bundle enthält LinkedIn, Google Ads, Google Analytics und die absteigende Sortierkennzeichnung der Follower-Datentabelle.
- Produktionsfeedback zum 7-Tage-Zeitraum behoben: Bei aktuell nur einem Follower-Snapshot und ohne Post-Marker zeigt das Diagramm nun einen sichtbaren Punkt und erklärt, dass die Verlaufslinie nach dem nächsten täglichen Sync entsteht. Patch `0.2.1` über `develop` (`dfb1982`) → `dev-server` (`6a3e20f`) → `main` (`17e080b`) veröffentlicht und als `v0.2.1` getaggt; alle CI-Gates grün, Staging und Produktion gesund, ausgeliefertes Bundle verifiziert.
- Nach produktivem Stacktrace zusätzlich die eigentliche Absturzursache geschlossen: Recharts kann die benutzerdefinierte Marker-Form auch ohne Payload aufrufen. `ChartMarker` validiert deshalb Payload und `posts` defensiv; bei null Markern wird keine Scatter-Ebene mehr gerendert. Direkter Regressionstest mit `payload={undefined}` ergänzt. Patch `0.2.2` über `develop` (`0fca0fd`) → `dev-server` (`0ec9957`) → `main` (`fe16bcc`) veröffentlicht und als `v0.2.2` getaggt; alle CI-Gates grün, produktives Bundle enthält den Guard und der Scheduler bleibt aktiv.

### Noch offen bis „Erledigt“

- Ersten tatsächlich automatisch ausgeführten Scheduler-Lauf am 20.07.2026 nach 05:00 Uhr prüfen. Danach Todo abschließen, in `abgeschlossene Todos/` verschieben und `docs/features/FEATURES.md` auf „Erledigt“ setzen.

## Aufwandsschätzung

Ca. **4–6 Entwicklungstage** inklusive Datenmodell und Migration, verschlüsseltem Token-Lifecycle, robuster Synchronisierung, Admin-API, Fixture-Seed, vollständiger Frontend-Seite, automatisierten Tests und gestuftem Produktionsrollout. Die ursprüngliche Schätzung von 1,5–2,5 Tagen ist nach der fachlichen Präzisierung nicht mehr realistisch.
