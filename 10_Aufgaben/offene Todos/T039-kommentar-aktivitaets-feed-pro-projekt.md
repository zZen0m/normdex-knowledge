# T039 · Kommentar-/Aktivitäts-Feed pro Projekt

**Phase:** App / Projektverwaltung / Zusammenarbeit  
**Priorität:** P3 · UX-Verbesserung / Team-Workflow  
**Status:** offen  
**Datum:** 2026-07-08

## Ziel

In der Projekt-Detailansicht soll ein kombinierter Feed aus automatischen Aktivitäts-Events und manuellen Team-Kommentaren entstehen. Damit können Teams projektbezogene Rückfragen, Freigabe-Hinweise oder Prüfnotizen an einem Ort festhalten, bevor ein Bericht final an den Kunden geht — ohne Umweg über E-Mail oder externe Tools.

## Kontext

Der bestehende `ActivityFeed` (Dashboard, `components/dashboard/ActivityFeed.tsx`) zeigt bereits automatische `AuditLog`-Events org-weit an, jedoch nicht projektgefiltert und ohne Kommentarfunktion. `AuditLog`-Einträge wie `project_created`/`project_updated` speichern bereits `meta.project_id` (`apps/api/app/routers/projects.py`, Zeilen 245, 275) — der automatische Teil des neuen Feeds lässt sich also ohne Schemaänderung nach `project_id` filtern. Nur für manuelle Kommentare braucht es eine neue Tabelle.

## Umsetzung

- Backend:
  - Neue Tabelle `ProjectComment`: `id (UUID), project_id (FK → projects, CASCADE), user_id (FK → users), body (Text), created_at, updated_at, edited_at (nullable)`.
  - Bewusst kein Threading/Reply-Baum in v1 — flache, chronologische Liste.
  - `GET /projects/{id}/comments` — paginierte Kommentarliste.
  - `POST /projects/{id}/comments` — neuer Kommentar; Berechtigung wie bestehender Projektzugriff (Owner/Member der Org).
  - `DELETE /projects/{id}/comments/{comment_id}` — nur eigener Kommentar oder Owner.
  - `GET /projects/{id}/activity` — kombiniert `ProjectComment` mit `AuditLog`-Einträgen, deren `meta.project_id == id`, chronologisch gemischt (Kategorisierung analog zu `classify_activity_category` in `dashboard.py`).
- Frontend:
  - Neue Sektion in `ProjectDetail.tsx` (Seite arbeitet mit gestapelten `<section>`-Blöcken, kein Tab-System — passt als eigener Block am Seitenende).
  - Wiederverwendung des `ActivityFeed`-Komponentenmusters als Basis, erweitert um ein Kommentar-Eingabefeld oben.
  - Visuelle Unterscheidung: automatische Events (Icon, dezent) vs. Kommentare (Avatar, Textkarte) in gemeinsamer Zeitleiste.

## Offene Fragen (vor Umsetzung klären)

- [ ] @-Mentions mit Notification-Trigger? (Notification-System existiert bereits — ggf. als v2 auslagern)
- [ ] Löschen nur eigener Kommentare oder auch durch Owner?
- [ ] Sichtbarkeit: nur Org-Mitglieder mit Projektzugriff, oder perspektivisch auch read-only für externe Gutachter?

## Akzeptanzkriterien

- [ ] In der Projekt-Detailansicht ist ein kombinierter Aktivitäts-/Kommentar-Feed sichtbar.
- [ ] Automatische Projekt-Events (erstellt, aktualisiert, gelöscht) erscheinen im Feed, gefiltert auf das jeweilige Projekt.
- [ ] Team-Mitglieder können Kommentare verfassen und eigene Kommentare löschen.
- [ ] Kommentare sind auf Org-Mitglieder mit Projektzugriff beschränkt.
- [ ] Bestehender org-weiter Dashboard-Aktivitäts-Feed bleibt unverändert funktionsfähig.
