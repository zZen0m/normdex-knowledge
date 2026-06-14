# T025 · Upload-Retention und Avatar-Löschung

**Phase:** App-Betrieb / Datenschutz / Storage-Hygiene  
**Priorität:** P2 · Support / Benutzerprofil / Dateien  
**Status:** erledigt  
**Datum:** 2026-06-03  
**Abgeschlossen:** 2026-06-04  

## Ziel

Normdex soll hochgeladene Dateien kontrolliert aufbewahren und löschen, damit der Server langfristig keinen unnötigen Datenbestand ansammelt.

Konkret:

- Support-Ticket-Anhänge sollen bei geschlossenen Tickets nach 365 Tagen gelöscht werden.
- Benutzer sollen ihr Avatarbild explizit löschen können; dabei muss auch die Datei auf dem Server entfernt werden.

## Kontext

Aktueller Stand laut Codeprüfung am 2026-06-03:

- Support-Anhänge werden unter `uploads/attachments/` gespeichert und über `support_ticket_attachments.storage_key` referenziert.
- Beim Schließen eines Tickets bleiben Anhänge aktuell erhalten.
- Es gibt Auto-Close-Logik für gelöste Tickets, aber keinen Cleanup-Job für Anhänge geschlossener Tickets.
- Avatare werden beim Hochladen eines neuen Avatars und bei Konto-Löschung vom Server gelöscht.
- Ein expliziter Endpoint zum Entfernen des eigenen Avatarbilds fehlt aktuell.
- Firmenlogos werden beim Ersetzen und beim expliziten Löschen bereits vom Server entfernt.

## Umfang

### Support-Anhänge

- Cleanup-Policy definieren: Anhänge von Tickets mit `status = closed` werden gelöscht, wenn `closed_at` mindestens 365 Tage zurückliegt.
- Geplanten Backend-Job ergänzen, der regelmäßig alte Support-Anhänge entfernt.
- Physische Dateien unter `uploads/attachments/` löschen.
- Zugehörige DB-Einträge in `support_ticket_attachments` kontrolliert behandeln:
  - entweder löschen,
  - oder als gelöschten Anhang markieren, falls UI-Historie erhalten bleiben soll.
- UI-Verhalten für gelöschte Anhänge festlegen:
  - z.B. „Anhang aus Aufbewahrungsgründen gelöscht“ statt Download-Link.
- Fehler robust protokollieren, wenn eine Datei fehlt oder nicht gelöscht werden kann.
- Cleanup idempotent gestalten, damit wiederholte Läufe ungefährlich sind.

### Avatar-Löschung

- Backend-Endpoint ergänzen, z.B. `DELETE /users/me/avatar`.
- Der Endpoint muss:
  - die aktuelle Avatar-Datei aus `uploads/avatars/` löschen,
  - `user.avatar_url` auf `null` setzen,
  - ein Audit-/Profil-Event schreiben.
- Frontend-Funktion „Profilbild entfernen“ im Profilbereich ergänzen.
- Sicherstellen, dass kein bloßes Setzen von `avatar_url = null` möglich ist, ohne die Datei zu löschen.

## Akzeptanzkriterien

- [x] Ein geschlossener Supportfall mit `closed_at` älter als 365 Tage verliert seine physischen Anhänge beim Cleanup-Lauf.
- [x] Tickets, die weniger als 365 Tage geschlossen sind, behalten ihre Anhänge.
- [x] Offene, neue, in Bearbeitung befindliche, wartende oder gelöste Tickets werden nicht von der Retention gelöscht.
- [x] Der Cleanup-Lauf ist idempotent und bricht nicht ab, wenn eine Datei bereits fehlt.
- [x] Die Support-UI zeigt gelöschte Anhänge verständlich an oder blendet sie bewusst aus.
- [x] Backend-Tests decken Grenzfälle ab: genau 365 Tage, jünger als 365 Tage, fehlende Datei, mehrere Anhänge pro Ticket.
- [x] Nutzer können ihr Avatarbild explizit löschen.
- [x] Beim Avatar-Löschen wird die Datei aus `uploads/avatars/` entfernt und `avatar_url` geleert.
- [x] Avatar-Löschung ist im Frontend sichtbar und nach Reload konsistent.
- [x] Backend-Tests prüfen, dass Avatar-Datei und DB-Feld gemeinsam bereinigt werden.

## Hinweise

- Backups enthalten gelöschte Dateien noch bis zum Ablauf der Backup-Retention (`LOCAL_KEEP`, `REMOTE_KEEP`).
- Die Support-Anhang-Retention betrifft nur lokale App-Dateien; falls E-Mail-Anhänge zusätzlich in Microsoft 365 verbleiben, ist deren Aufbewahrung separat über Microsoft-/Mailbox-Policies zu regeln.
- Für Firmenlogos ist aktuell kein neues Todo nötig, da Upload-Ersetzung und explizite Löschung bereits Datei-Cleanup enthalten.

## Notizen / Fortschritt

- 2026-06-03: Todo aus Betriebsfrage zu Backup, Support-Anhängen, Avataren und Logos angelegt.
- 2026-06-04: Umgesetzt und mit 15 neuen Backend-Tests abgesichert (gesamte Suite grün: 292 Tests, Frontend-Typecheck sauber).
  - **Support-Anhang-Retention:**
    - Neue Spalte `support_ticket_attachments.deleted_at` (Alembic-Migration `ffd3bbde6b6a`). Anhänge werden *als gelöscht markiert*, die DB-Zeile + `storage_key` bleiben für die Ticket-Historie erhalten.
    - Cleanup-Job `cleanup_support_attachments()` in `apps/api/app/services/scheduler.py`, registriert täglich um 03:30 (`CronTrigger`). Kernlogik in der testbaren Helper-Funktion `_cleanup_due_attachments(db)` (Muster wie `_expire_due_licenses`). Frist: `SUPPORT_ATTACHMENT_RETENTION_DAYS = 365` ab `closed_at`, nur Status `closed`. Idempotent, robustes Logging je Datei, fehlende Datei ist kein Fehler.
    - Serializer `SupportAttachmentOut` um `deleted_at` erweitert; `SupportTicketDetail.tsx` zeigt statt Download-Link den Hinweis „Anhang aus Aufbewahrungsgründen gelöscht".
  - **Avatar-Löschung:**
    - Neuer Endpoint `DELETE /users/me/avatar` (`apps/api/app/routers/users.py`): nutzt den bestehenden Helper `_delete_avatar_file()`, setzt `avatar_url = None`, schreibt Audit-Event `avatar_removed`. Datei-Löschung und Feld-Reset sind gekoppelt.
    - `PUT /users/me/profile` setzt `avatar_url` bewusst nicht mehr — Avatar-Änderungen laufen ausschließlich über die dedizierten Upload-/Delete-Endpoints (keine verwaisten Dateien, kein Setzen ohne Datei).
    - Admin-Delete (`DELETE /admin/users/{id}`) entfernt jetzt ebenfalls die Avatar-Datei (war zuvor ein Leak — nur DB-Zeile gelöscht).
    - Frontend: `api.deleteAvatar()` + „Profilbild entfernen"-Button im Avatar-Dialog (`SettingsProfile.tsx`), sichtbar nur bei vorhandenem Avatar, mit Bestätigung und `refetch()`.
  - **Prod-Hinweis:** Migration `ffd3bbde6b6a` muss in Prod via Docker-Profil angewendet werden (`docker compose -f deploy/docker-compose.prod.yml --profile tools run --rm api-migrate`).
