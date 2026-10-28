# Automatisierter Backup-Service

Stand: 28.10.2026. Repo: `zZen0m/normdex-backup`, VPS: `/opt/repos/normdex-backup`.

## Sicherungen und Erfolgsnachweis

Täglich 02:00 Europe/Vienna sichert `normdex-backup`:

- PostgreSQL `normdex_prod_database` als `normdex_prod_db.dump` (Custom-Format).
- Volume `normdex_prod_normdex_uploads` als `uploads.tar.gz`.
- `manifest.json` mit Dateigrößen, SHA-256 und Erstellungszeit.

Ein Datenbankdump gilt nur bei erfolgreichem `pg_dump` und `pg_restore --list` als
lokal erfolgreich. Uploads werden mit `tar -tzf` geprüft. Unvollständige Sicherungen
bleiben `.partial` und gelten nicht als Erfolg.

Cloud-Ziel: SharePoint, Normdex – Dokumente,
`03_Product-Development/01_Backups Normdex Produktion VPS`.
Nach jedem Upload vergleicht `rclone check --download` den tatsächlichen Inhalt.
Erst danach wird Cloud-Erfolg gespeichert. Maßgeblich für Frische ist das Alter
der Sicherung, nicht der Zeitpunkt eines späteren erneuten Uploads.

## Persistente Anmeldung

`config/rclone/rclone.conf` enthält die Microsoft-Verbindung und den aktuellen
Refresh-Token. Das Verzeichnis ist dauerhaft gemountet; die Datei ist Modus 600.
Die bisherigen `RCLONE_CONFIG_SHAREPOINT_*`-Variablen dienen nur dem einmaligen
Import, falls die Datei noch nicht existiert. Vor jedem rclone-Lauf werden diese
Overrides entfernt, damit ein alter Umgebungs-Token keinen erneuerten Token ersetzt.
Nie Token oder Client Secret in Git, Chat, Telegram oder Logs schreiben.

Incident: Vollständige Cloud-Backups waren zuletzt am 31.08.2026 erfolgreich.
Ab 01.09. scheiterte die Anmeldung. Am 28.10. wurde der neuere Refresh-Token aus
der Konfiguration im alten Container vor dessen Austausch geschützt gerettet.
Damit war die Verbindung ohne erneute Benutzeranmeldung wiederherstellbar.
Ein kompletter Upload mit Inhaltsprüfung wurde erfolgreich getestet.

## Alarmierung und unabhängige Kontrolle

Telegram verwendet den vorhandenen Bot „Telegram Normdex Tickets“ und den
bestehenden Zielchat aus n8n. Zugangsdaten liegen ausschließlich in
`env/.env.alerts` (600); der Versand erfolgt direkt, unabhängig von n8n-Workflows.

Der Benutzer-Cron des VPS prüft alle 15 Minuten, unabhängig vom Backup-Container:

- letzte lokale und Cloud-Sicherung höchstens 26 Stunden alt;
- letzter Wiederherstellungstest höchstens 8 Tage alt;
- letzter Backup-Lauf und Wiederherstellungstest erfolgreich;
- mindestens 15 % freier Speicherplatz.

Fehler werden sofort beim Lauf bzw. spätestens bei der nächsten Prüfung gemeldet.
Gleiche Alarme werden höchstens täglich wiederholt, Änderungen und Entwarnung
werden gemeldet. Fehlgeschlagene Zustellungen werden erneut versucht.
Ein Testversand ist mit `python3 host_watchdog.py notify-test` möglich.

Für den Totalausfall des VPS ist zusätzlich ein externer Dead-Man-Monitor nötig.
Die Anbindung über `BACKUP_HEARTBEAT_URL` ist vorbereitet, aber ohne eingetragenen
externen Dienst **nicht aktiv**. Kein vollständiger VPS-Ausfallschutz behaupten.

## Aufbewahrung und Platzbegrenzung

- Lokal: letzte 14 Sicherungen, zusätzlich Schutz der letzten lokal erfolgreichen,
  Cloud-geprüften und wiederhergestellten Sicherung (ggf. mehr als 14 Ordner).
- Extern: letzte 30 tägliche Sicherungen und jeweils die letzte Sicherung der
  letzten 12 Monate. Manuelle Zusatzläufe zählen bei den letzten 30 mit.
- Lokale Bereinigung läuft auch nach Cloud-Fehlern. Bei der Umstellung wird erst
  nach dem ersten neuen bestätigten Cloud-Upload bereinigt.
- Verwaiste `.partial`-Ordner werden nach zwei Tagen entfernt.
- Extern werden automatisch nur vom neuen Ablauf hochgeladene und geprüfte
  Sicherungen gelöscht. Historische externe Sicherungen bleiben bis zur gesonderten
  Prüfung erhalten.
- Containerlogs: maximal 3 × 5 MB. `data/backup.log` bleibt historische Altdatei.

## Wiederherstellungstest

Jeden Sonntag um 04:00 UTC (05:00/06:00 Wien) startet ein separater Container ohne
Netzwerkverbindung, ohne Produktionszugangsdaten und ohne produktive Datenbank-
Volumes. Der aktuelle geprüfte Dump wird in eine temporäre PostgreSQL-Instanz
zurückgespielt; eine nichtleere Tabellenstruktur ist erforderlich. Backup und
Wiederherstellung teilen eine Sperre, damit die Bereinigung den Dump nicht entfernt.
Beim Integrationstest am 28.10. wurden 31 Tabellen erfolgreich wiederhergestellt.

## Betrieb

```sh
cd /opt/repos/normdex-backup
docker logs --tail 100 normdex-backup
cat state/watchdog.log state/restore.log
crontab -l
docker exec normdex-backup /app/backup.sh
docker compose --profile maintenance run --rm --no-deps restore-check
python3 host_watchdog.py
```

`state/local.json`, `cloud.json`, `restore.json`, `run.json` und `restore-run.json`
enthalten die Erfolgszeitpunkte/-ergebnisse. Verzeichnis 2770 und Statusdateien
660 erlauben dem Benutzer `deploy` die unabhängige Kontrolle; Nutzdaten und
Zugangsdaten sind davon getrennt und bleiben restriktiv geschützt.
Die mitgelieferten systemd-Units sind eine Alternative für Administratoren;
aktuell ist Benutzer-Cron aktiv. Nicht beide Zeitpläne gleichzeitig aktivieren.

## Wiederherstellung im Ernstfall

Neue Dumps sind Custom-Format: `pg_restore --exit-on-error --no-owner
--no-privileges --dbname LEERE_TESTDATENBANK normdex_prod_db.dump`.
Zuerst Prüfsummen prüfen und in einer isolierten Datenbank wiederherstellen,
danach App-Daten validieren. Upload-Archiv separat prüfen und erst bei geplanter
Wiederherstellung das Produktionsvolume ersetzen.
Historische `.sql.gz`-Dumps werden weiterhin mit `gunzip` und `psql` wiederhergestellt.
Keine Produktionswiederherstellung ohne eigene Wartungsfreigabe.

Vor Migrationen bleibt ein frisches Pre-Deployment-Backup verpflichtend.
Siehe [[IT-Workflow Branches Deployments Backups Migrationen]].
