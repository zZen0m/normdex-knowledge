# T048 · LinkedIn-Marketing-KPIs im Verwaltungsportal anzeigen

**Phase:** App / Admin / Verwaltungsportal / Marketing
**Priorität:** P3 · Wichtig für Wachstumsanalyse, kein Blocker für laufenden Betrieb
**Status:** offen
**Datum:** 2026-07-19

## Ziel

Die LinkedIn-Kennzahlen der Unternehmensseite (Follower-Verlauf, Post-Performance) nicht nur per Skript auf dem VPS abrufbar machen, sondern direkt im Normdex-Verwaltungsportal als eigener „Marketing"-Bereich anzeigen — mit Verlaufs-Chart und Post-Übersicht, damit sich auswerten lässt, welche Posts/Zeitpunkte zu Follower-Wachstum geführt haben.

## Kontext / Ist-Stand

- [[T047-linkedin-api-freigabe-statistiken]] ist abgeschlossen: LinkedIn-App mit Community-Management-API-Zugriff ist eingerichtet, OAuth-Token liegen vor.
- Aktuell laufen Abruf und Speicherung **außerhalb der App**, direkt auf dem VPS: `/home/deploy/tools/linkedin-orgpage/` enthält `fetch_stats.py` (Live-Abruf) und `snapshot.py` (Abruf + Anhängen an `history.jsonl`). Ein Cronjob (täglich 05:00 Uhr, `deploy`-User) ruft `snapshot.py` auf.
- Das ist bewusst als schneller, isolierter erster Schritt entstanden (Zugangsdaten liegen dort mit 600-Rechten, nicht im Git-Repo) — für eine Anzeige im Verwaltungsportal ist das nicht die richtige Grundlage, weil die Datei außerhalb der App-Infrastruktur liegt und in Prod (separates Repo/Deployment, Postgres statt lokaler Datei) nicht sauber mitläuft.
- Für dieses Todo soll die Abruf-Logik stattdessen **in die App wandern** (Backend-Service + Scheduler-Job, analog bestehendem `app/services/scheduler.py`-Pattern mit APScheduler), Ergebnisse landen in der App-eigenen Datenbank statt in einer Datei auf dem VPS.

## Recherchierte Konventionen (für die Umsetzung relevant)

- **Modelle:** `apps/api/app/models.py`, `Base` aus `app.database`, `created_at = Column(DateTime(timezone=True), server_default=func.now())` als Standardmuster.
- **Migrationen:** `apps/api/alembic/versions/`, Naming `{revision_hash}_{beschreibung}.py`. **Vor jeder Migration den `.claude/skills/db_migration/SKILL.md`-Skill lesen** (Projektvorgabe, gilt auch hier).
- **Admin-Routen:** `apps/api/app/routers/admin.py`, Absicherung über `Depends(require_admin)` aus `app.deps`.
- **Scheduler:** `apps/api/app/services/scheduler.py`, APScheduler (`AsyncIOScheduler`), Jobs via `scheduler.add_job(func, CronTrigger(...), id=..., replace_existing=True, misfire_grace_time=...)`. Beispiel-Job: `check_expiring_licenses`.
- **Frontend-Navigation:** `apps/frontend/src/components/layout/Sidebar.tsx` (Admin-Einträge ab Zeile ~73, bedingt gerendert mit `user?.is_admin`), Routing in `apps/frontend/src/App.tsx` unter `<AdminRoute />`.
- **Charts:** Recharts ist bereits im Einsatz (z. B. `EconomicsReport.tsx`).
- **KPI-Cards:** `apps/frontend/src/components/dashboard/KPICard.tsx` (Wert, Trend/Delta, Farbton) als Stilvorlage.
- **Externe Zugangsdaten:** statische Secrets normalerweise über `.env` + `app/config.py` (Pydantic Settings), z. B. `STRIPE_SECRET_KEY`, `BREVO_API_KEY`. Für LinkedIn passt das für Client-ID/Secret, **nicht** für Access-/Refresh-Token (siehe offene Frage unten).

## Vorgeschlagene Design-Punkte (nicht final entschieden — siehe offene Fragen)

- **Zwei neue Tabellen** statt einer: Follower-Snapshots und Post-Snapshots getrennt, weil Post-Kennzahlen (Impressions etc.) sich über die Zeit weiterentwickeln und man so auch die Post-Performance-Entwicklung nachvollziehen kann, nicht nur den letzten Stand:
  - `linkedin_follower_snapshots`: `id`, `measured_at`, `followers_organic`, `followers_paid`, `created_at`
  - `linkedin_post_snapshots`: `id`, `post_urn`, `measured_at`, `lifecycle_state`, `impression_count`, `unique_impression_count`, `click_count`, `like_count`, `comment_count`, `engagement`, `created_at`
- **Keine automatische Löschung/Retention** — anders als bei Serverlogs ([[T045-systemlogs-in-adminoberflaeche]]) handelt es sich um aggregierte Geschäftskennzahlen ohne personenbezogene Einzeldaten, historische Werte sind für die Trendanalyse gerade der Zweck.
- **Post-Publish-Marker im Follower-Chart:** Veröffentlichungszeitpunkte der Posts als Referenzlinien/-punkte im Follower-Verlaufs-Chart einblenden, um einen visuellen Zusammenhang zwischen Postings und Follower-Wachstum herzustellen — das ist der eigentliche Mehrwert gegenüber den reinen Rohdaten.
- **Bestehendes Standalone-Skript** (`fetch_stats.py`/`snapshot.py` auf dem VPS) liefert die Blaupause für die Backend-Service-Logik (Endpunkte, Query-Eigenheiten wie `viewContext=AUTHOR`, `shares=List(...)`-Syntax, Token-Refresh) — kann beim Portieren direkt als Referenz dienen.

## Geplante Umsetzung (grob)

**Backend:**
- Neue Modelle `LinkedInFollowerSnapshot`, `LinkedInPostSnapshot` + Alembic-Migration (Skill zuerst lesen)
- Neuer Service `app/services/linkedin_service.py`: OAuth-Token-Handling (Refresh), Follower-Statistik, Post-Liste (inkl. geplanter Posts via `viewContext=AUTHOR`), Einzelpost-Statistik — Logik analog zum bestehenden VPS-Skript, aber gegen App-DB statt JSONL-Datei
- Scheduler-Job (täglich, analog bestehendem Muster) speichert einen Snapshot in beide Tabellen
- Admin-Endpoint(e), z. B. `GET /admin/marketing/linkedin/overview` — liefert Follower-Zeitreihe + aktuelle KPIs + Post-Liste mit letztem Stand für das Frontend
- Absicherung über `require_admin`, analog anderen Admin-Routen

**Frontend:**
- Neue Seite `apps/frontend/src/pages/admin/Marketing.tsx`, Route `/admin/marketing`
- Neuer Sidebar-Eintrag „Marketing" (Icon TBD, siehe offene Fragen), nur sichtbar für Admins
- KPI-Cards (Follower gesamt, Impressions gesamt, Engagement-Rate, ggf. Follower-Delta letzte 7/30 Tage) mit `KPICard`
- Recharts-Liniendiagramm: Follower-Verlauf über Zeit, mit Referenzmarkern an Post-Veröffentlichungsdaten
- Tabelle: Posts mit Datum, Textanfang, Impressions, Klicks, Likes, Kommentare, Engagement — sortierbar nach Datum

## Offene Fragen

- **Token-Speicherort:** Access-/Refresh-Token rotieren bei jedem Refresh — passen nicht gut in eine statische `.env`-Datei. Client-ID/Secret können in `.env`, aber wohin mit dem sich ändernden Token? Vorschlag: eigene kleine Tabelle/Settings-Zeile in der DB, vom Scheduler-Job lesend/schreibend genutzt. Sicherheitsaspekt dabei klären: DB-Zugriff ist potenziell breiter als der aktuelle 600-Datei-Zugriff nur für den `deploy`-User.
- **Läuft der Scheduler-Job nur in Prod oder auch in Dev?** Es ist derselbe LinkedIn-Account/Token, keine Kundendaten-Trennung nötig wie sonst zwischen Dev/Prod — evtl. reicht ein Lauf ausschließlich in Prod, Dev nutzt zum Testen einen manuellen Trigger oder Fixture-Daten.
- **Wird das bestehende VPS-Standalone-Tool nach der App-Integration stillgelegt** oder als einfacher manueller Fallback/Debug-Werkzeug behalten?
- **Wird die bisherige `history.jsonl` weitergeführt** (z. B. als Backup außerhalb der DB) oder komplett durch die DB-Tabellen abgelöst?
- Icon und genaue Bezeichnung für den neuen Sidebar-Eintrag, Positionierung relativ zu „Verwaltung"/„Ticket Portal"

## Akzeptanzkriterien

- [ ] Follower-Zahl und Post-Statistiken werden täglich automatisch erfasst und in der App-DB gespeichert (nicht mehr nur im VPS-Tool-Ordner)
- [ ] Neuer „Marketing"-Bereich im Verwaltungsportal, nur für Admins sichtbar
- [ ] Follower-Verlauf als Zeitreihen-Chart, mit erkennbaren Post-Veröffentlichungszeitpunkten
- [ ] KPI-Cards mit aktuellem Stand (Follower, Impressions, Engagement-Rate)
- [ ] Post-Tabelle mit Einzelstatistik pro Post
- [ ] Migration folgt dem DB-Migration-Skill-Prozess (Dev getestet, Prod-Vorgehen dokumentiert)

## Aufwandsschätzung

Ca. 1,5–2,5 Tage. Backend (Modelle, Migration, Service-Portierung inkl. Token-Handling, Scheduler-Job, Endpoint) macht den größeren Teil aus, da die OAuth-/Token-Refresh-Logik sauber in die App-Architektur eingepasst werden muss (siehe offene Frage Token-Speicherort). Frontend (Seite, Chart, Tabelle, Nav-Eintrag) folgt bestehenden Mustern (KPICard, Recharts) und ist der leichtere Teil.
