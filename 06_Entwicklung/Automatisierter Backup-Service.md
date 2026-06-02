# Automatisierter Backup-Service

Der Normdex Backup-Service sichert täglich die Produktionsdatenbank und das Uploads-Volume und speichert die Backups lokal auf dem VPS sowie offsite auf SharePoint.

## Was gesichert wird

| Quelle | Methode | Datei |
|---|---|---|
| PostgreSQL `normdex_prod_database` | `pg_dump` → gzip | `normdex_prod_db.sql.gz` |
| Docker Volume `normdex_prod_normdex_uploads` | tar | `uploads.tar.gz` |

## Architektur

- **Repo:** `git@github.com:zZen0m/normdex-backup.git`
- **Server-Pfad:** `/opt/repos/normdex-backup/`
- **Container:** `normdex-backup` (Docker, `restart: unless-stopped`)
- **Cron:** täglich um 02:00 Uhr (Europe/Vienna)
- **Lokale Retention:** letzte 14 Tage unter `./data/`
- **Remote Retention:** letzte 30 Tage auf SharePoint

## Offsite-Speicherort (SharePoint)

```
Permatec e.U > Normdex – Dokumente > 03_Product-Development > 01_Backups Normdex Produktion VPS
```

Die Verbindung läuft über rclone mit einer Azure App Registration:

| Parameter | Wert |
|---|---|
| Azure App | Normdex Backup rclone |
| App-ID (Client ID) | `0cb92941-31aa-4100-a78f-89ab613985e0` |
| Tenant | `4099f674-ae6c-47ab-a7bd-2d2e582ac426` (permateceu) |
| SharePoint Drive | Normdex – Dokumente (documentLibrary) |
| Drive ID | `b!vU7cHYgueEm3nFz3uq-D3n7DUYwwn-xBvnwt8HYELr_zbjzOJJCsR4HZAyb8DCqs` |

Credentials (Client Secret, OAuth-Token) liegen ausschliesslich in `/opt/repos/normdex-backup/env/.env.backup` auf dem Server – diese Datei ist in `.gitignore` und wird nie committet.

## Backup-Ablauf

1. `pg_dump` der Produktionsdatenbank → komprimiert als `.sql.gz`
2. `tar` des Uploads-Volumes → `.tar.gz`
3. Beide Dateien via rclone auf SharePoint hochladen
4. Lokale Rotation: Verzeichnisse älter als die letzten 14 werden gelöscht
5. Remote-Rotation: Verzeichnisse älter als die letzten 30 werden auf SharePoint gelöscht

Bei einem Fehler bricht das Skript sofort ab (`set -e`). Der Fehler ist in den Container-Logs sichtbar.

## Backup manuell ausführen

```bash
docker exec normdex-backup /app/backup.sh
```

## Logs prüfen

```bash
# Live-Log des Containers
docker logs normdex-backup -f

# Archiviertes Log aller Läufe
cat /opt/repos/normdex-backup/data/backup.log
```

## Wiederherstellung (Disaster Recovery)

### Datenbank wiederherstellen

```bash
# Neuestes lokales Backup finden
ls -lt /opt/repos/normdex-backup/data/ | head -5

# DB wiederherstellen
gunzip -c /opt/repos/normdex-backup/data/DATUM/normdex_prod_db.sql.gz \
  | docker exec -i db-postgres-1 psql -U normdex -d normdex_prod_database
```

### Uploads wiederherstellen

```bash
tar -xzf /opt/repos/normdex-backup/data/DATUM/uploads.tar.gz \
  -C /var/lib/docker/volumes/normdex_prod_normdex_uploads/_data/
```

Falls keine lokalen Backups vorhanden sind, können die letzten 30 Tage von SharePoint heruntergeladen werden.

## Azure App – Token erneuern

Der OAuth Refresh-Token erneuert sich automatisch bei jedem Backup-Lauf. Falls die Verbindung zu SharePoint dennoch bricht (z.B. nach manueller Revokation), muss der Token neu generiert werden:

1. Auf dem lokalen Windows-Rechner rclone portable herunterladen
2. Befehl ausführen:
   ```powershell
   rclone.exe authorize "onedrive" "0cb92941-31aa-4100-a78f-89ab613985e0" "CLIENT_SECRET" --onedrive-tenant "4099f674-ae6c-47ab-a7bd-2d2e582ac426"
   ```
3. Neuen Token-JSON in `/opt/repos/normdex-backup/env/.env.backup` unter `RCLONE_CONFIG_SHAREPOINT_TOKEN` eintragen
4. Container neu starten: `docker compose -f /opt/repos/normdex-backup/docker-compose.yml up -d --force-recreate`

Das Client Secret läuft am **1.6.2028** ab (Secret "Normdex VPS" in der Azure App Registration).

## Deployment-Integration

Manuelles Pre-Deployment-Backup (vor Migrationen) weiterhin empfohlen, auch wenn der automatische tägliche Backup läuft:

```bash
docker exec -t db-postgres-1 pg_dump -U normdex normdex_prod_database \
  > /opt/repos/normdex-webapp/deploy/db_backups/normdex_prod_backup_$(date +%Y%m%d_%H%M%S).sql
```

Siehe auch: [[IT-Workflow Branches Deployments Backups Migrationen]]
