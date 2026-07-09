# T045 · Systemlogs (Serverlogs) in der Adminoberfläche

**Phase:** Admin / Verwaltungsportal / Diagnose  
**Priorität:** P3 · Ops-/Supporteffizienz  
**Status:** offen  
**Datum:** 2026-07-09

## Ziel

Admins sollen Serverlogs (Anwendungslogs: INFO/WARNING/ERROR/CRITICAL, inkl. Login-Events, Requests etc. — alles, was aktuell nur per `docker logs` im Terminal sichtbar ist) direkt in der Adminoberfläche einsehen können, mit Zeitstempel, Level-Filter, Volltextsuche, Zeitraumfilter und Filter pro Organisation. Aktuell ist der einzige Weg dazu SSH + `docker logs` auf dem VPS.

## Kontext / Ist-Stand (Recherche 2026-07-09)

- Es gibt aktuell **keine Persistenz** für Anwendungslogs: uvicorn schreibt nach stdout, der Docker-Logdriver fängt das auf (`docker logs normdex_prod-api-1`), keine Rotation, keine Datenbank-Speicherung, kein Sentry/Log-Aggregator.
- Getrennt davon existiert bereits `AuditLog` (Business-Events wie Login, Ticket-Status, Lizenz-Events) — das ist NICHT dasselbe wie Serverlogs und bleibt unverändert bestehen. Serverlogs ergänzen das, ersetzen es nicht.
- Reale Volumen-Messung (Prod, `normdex_prod-api-1`, nur lesend via `docker logs`):
  - 24h: 4.073 Zeilen / ~258 KB — davon **~90% `/health`-Healthcheck-Pings** (reines Rauschen)
  - Ohne Health-Pings: ~1.055 Zeilen/Tag (~80 KB/Tag)
  - 7 Tage: nur 5 Zeilen mit „error/traceback/exception" — App läuft aktuell sehr ruhig
- Speicherplatz-Hochrechnung: bei DB-Speicherung inkl. Overhead ca. 150–300 KB/Tag → **5–10 MB für 30 Tage**, selbst bei 10-facher Skalierung nur niedrige zweistellige MB/Monat. Speicherplatz ist kein Thema; 30-Tage-Löschung ist trotzdem sinnvoll (DSGVO — IP-Adressen/User-Agents in Logs sind personenbezogene Daten), analog zu [[T025-upload-retention-und-avatar-loeschung]].

## Entschiedene Design-Punkte (aus Brainstorming)

- **Speicherort:** selbe Datenbank (Postgres in Prod / SQLite in Dev), **neue Tabelle** `server_logs` — keine eigene DB. Kein Volumen-/Performance-Grund für Trennung; nutzt bestehende Migrations-Pipeline und bestehendes Backup automatisch mit.
- **Schreibpfad:** Logging-Handler nutzt eine **eigene, schlanke DB-Connection** für Hintergrund-Writes (nicht die Request-Session), damit Log-Writes nie mit einer laufenden Request-Transaktion kollidieren. Da die API laut Dockerfile-Kommentar bewusst als **Single-Worker** läuft, muss der Handler non-blocking sein (z. B. Python `QueueHandler`/`QueueListener`, damit ein DB-Write nie die Request-Verarbeitung bremst).
- **Log-Level:** alle Level inkl. INFO sollen erfasst werden (nicht nur WARNING+), mit Level als Filter in der UI (Mehrfachauswahl).
- **Healthcheck-Rauschen:** `/health`-Requests standardmäßig aus der Ansicht ausgeblendet, aber als Filteroption einblendbar (nicht komplett verwerfen — beim Debuggen von Healthcheck-Ausfällen relevant).
- **Org-Filter:** Logs sollen, wo möglich, mit `org_id`/`user_id`/`request_id` angereichert werden (Request-Kontext via Middleware + `contextvars`), damit man pro Organisation filtern kann — analog zur Kundenakte-Timeline. Logs außerhalb eines Requests (Startup, Scheduler-Jobs) haben keine `org_id`, das ist ok.
- **Retention:** automatische Löschung nach 30 Tagen (Cron/Scheduler-Job, analog bestehendem `scheduler.py`-Pattern).

## Geplante Umsetzung (grob)

- **Backend:**
  - Neues Modell `ServerLog` (`id`, `created_at`, `level`, `logger_name`, `message`, `traceback`, `request_id`, `org_id` nullable, `user_id` nullable, `path`, `method`, `status_code`) + Alembic-Migration
  - Logging-Handler (root logger + uvicorn access/error logger) mit Queue-basiertem, non-blockendem DB-Write über eigene Connection
  - Request-Kontext-Middleware (`contextvars`) für `request_id`/`org_id`/`user_id`
  - Admin-Endpoint mit Pagination, Filtern (Level, Zeitraum, Org, `/health` ein-/ausblenden) und Volltextsuche über `message`
  - Retention-Job: täglicher Purge von Einträgen älter als 30 Tage
- **Frontend:**
  - Neuer Menüpunkt in der Admin-Navigation, z. B. „Systemlogs"
  - Tabelle: Zeitstempel, Level-Badge, Logger/Modul, Message, aufklappbarer Traceback
  - Filter: Level (Mehrfachauswahl), Zeitraum (von/bis), Organisation, Volltextsuche, Healthcheck ein-/ausblenden (default aus)
  - Pagination (Live-Tail/Auto-Refresh ist nice-to-have, nicht Teil des v1)

## Akzeptanzkriterien

- [ ] Serverlogs (alle Level inkl. INFO) werden in `server_logs` persistiert, ohne die Request-Performance spürbar zu beeinträchtigen (Single-Worker-Constraint beachten)
- [ ] Admin-Ansicht zeigt Logs mit Zeitstempel, Level, Message, Traceback (falls vorhanden)
- [ ] Filter: Level (Mehrfachauswahl), Zeitraum, Organisation, Volltextsuche funktionieren
- [ ] `/health`-Pings sind standardmäßig ausgeblendet, aber per Filter einblendbar
- [ ] Logs werden nach 30 Tagen automatisch gelöscht
- [ ] Bestehendes `AuditLog`/Kundenakte-Timeline bleibt unverändert

## Aufwandsschätzung

Ca. 1–1,5 Tage fokussierte Umsetzung. Der Queue-basierte, non-blockende Logging-Handler (wegen Single-Worker-Constraint) ist der aufwändigste Teil, der Rest folgt bestehenden Mustern (Admin-Endpoint + Tabelle analog anderen Admin-Listen, Retention-Job analog `scheduler.py`).
