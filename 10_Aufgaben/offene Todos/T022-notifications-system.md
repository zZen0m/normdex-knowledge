# T022 - Notifications-System (Bell-Icon, Backend-API, Frontend-Panel)

**Status:** offen
**Bereich:** App / Backend / Frontend / Sidebar
**Erstellt:** 2026-05-24
**Abgeschlossen:** -

## Ziel

Die Sidebar enthält bereits ein Bell-Icon mit vorbereitetem Badge-Anzeige-Mechanismus, aber ohne tatsächliche Funktionalität. Es soll ein vollständiges Benachrichtigungs-System entstehen, das relevante App-Events (Lizenzkauf, Support-Antworten, Team-Einladungen, etc.) als persistente, lesbare/ungelesene Notifications für den User bereitstellt.

## Kontext

Aktueller Zustand in [apps/frontend/src/components/layout/Sidebar.tsx](apps/frontend/src/components/layout/Sidebar.tsx):

- `BellItem`-Komponente ist gerendert, zeigt Bell-Icon und (bei `notifCount > 0`) einen pinken Badge mit Anzahl (`9+` bei >9).
- `notifCount` ist hartcodiert auf `0` (siehe Kommentar `// TODO: notifCount an die Notifications-API anschließen`).
- `handleBellClick` ist ein leerer Placeholder (siehe Kommentar `// Placeholder: hier später das Notification-Panel öffnen`).

Es gibt aktuell **keine** Notifications-Infrastruktur im Backend ([apps/api/app/routers/](apps/api/app/routers/) — geprüft, kein `notifications.py`-Router). Auch im Frontend existiert kein `useNotifications`-Hook und kein Panel-Component.

## Umfang

### Backend (FastAPI)

**Neuer Router:** `apps/api/app/routers/notifications.py`

Endpoints:

- `GET /notifications` — Liste der Notifications des eingeloggten Users, paginiert
  - Query-Params: `limit` (default 50), `offset`, `unread_only` (bool)
  - Response: `{ items: Notification[], unread_count: number, total: number }`
- `GET /notifications/unread-count` — schneller Endpoint nur für das Badge
  - Response: `{ unread_count: number }`
- `POST /notifications/{id}/read` — einzelne Notification als gelesen markieren
- `POST /notifications/read-all` — alle ungelesenen Notifications des Users als gelesen markieren
- `DELETE /notifications/{id}` — einzelne Notification löschen (optional)

**Neues Datenmodell:** `apps/api/app/models.py` — neue Tabelle `notifications`

Vorgeschlagene Felder:

- `id` (PK)
- `user_id` (FK auf users, indexiert)
- `type` (Enum: `license_purchased`, `license_expired`, `license_expiring_soon`, `support_reply`, `team_invite`, `team_role_changed`, `system_announcement`, `welcome`, ...)
- `title` (string, max 200) — kurze Headline
- `body` (text) — längerer Erklärungstext
- `link` (string, nullable) — interne Route auf die der User springen soll, z.B. `/licenses/42`
- `read_at` (datetime, nullable) — wenn null → ungelesen
- `meta` (JSON, nullable) — typ-spezifische Daten (z.B. `{ "ticket_id": 12 }`)
- `created_at` (datetime, default now)

**Indizes:**

- `(user_id, read_at)` — für Badge-Query "WHERE user_id=X AND read_at IS NULL"
- `(user_id, created_at DESC)` — für Listing

**Alembic-Migration:** gemäß `db_migration`-Skill anlegen.

### Event-Trigger (wer erzeugt Notifications?)

Notifications werden vom Backend erzeugt, wenn bestimmte Events passieren. Konkret zu klärende Trigger-Quellen:

- **Lizenz-Events** (Stripe-Webhook in `apps/api/app/routers/stripe.py`):
  - Lizenz erfolgreich gekauft → `license_purchased`
  - Lizenz läuft in 14/7/1 Tagen ab → `license_expiring_soon` (per Scheduler)
  - Lizenz abgelaufen → `license_expired`
- **Support-Events** (`apps/api/app/routers/support.py` / `support_admin.py`):
  - Admin antwortet auf eigenes Ticket → `support_reply` an Ticket-Author
  - Neues Ticket eingegangen → `support_new_ticket` an alle Admins
- **Team-Events** (`apps/api/app/routers/users.py`):
  - User wurde in Organisation eingeladen → `team_invite`
  - User-Rolle wurde geändert → `team_role_changed`
- **System-Events**:
  - Erstanmeldung → `welcome`
  - Wartungsfenster / Major Release → `system_announcement` (Admin-erzeugt, an alle User)

**Hilfs-Service:** `apps/api/app/services/notifications.py` mit Funktion `create_notification(user_id, type, title, body, link=None, meta=None)` als zentraler Einstiegspunkt — Trigger-Code ruft diese Funktion auf, nicht direkt das ORM.

### Frontend

**Neuer Hook:** `apps/frontend/src/hooks/useNotifications.ts`

- Lädt `GET /notifications/unread-count` periodisch (z.B. alle 60s via `setInterval` oder `react-query` mit `refetchInterval`)
- Stellt `unreadCount`, `notifications`, `markAsRead(id)`, `markAllAsRead()`, `refetch()` bereit
- Optional: Server-Sent Events (`/notifications/stream`) für Live-Updates ohne Polling — Phase 2

**Sidebar-Integration:** [apps/frontend/src/components/layout/Sidebar.tsx](apps/frontend/src/components/layout/Sidebar.tsx)

- `const notifCount = 0;` ersetzen durch `const { unreadCount } = useNotifications();`
- `handleBellClick` öffnet das Panel (State im Sidebar oder via globalem Store)

**Neue Komponente:** `apps/frontend/src/components/layout/NotificationPanel.tsx`

UI-Form-Optionen (zu entscheiden):

- **Variante A:** Popover, ankert am Bell-Icon, ca. 380px breit, zeigt 5-10 jüngste Notifications + "Alle ansehen"-Link
- **Variante B:** Sheet (slide-in von rechts), volle Höhe, zeigt alle Notifications mit Pagination
- **Empfehlung:** A als Quick-View + dedizierte Seite `/notifications` für Vollansicht

Inhalt pro Eintrag:

- Icon je nach `type` (Money für License, MessageSquare für Support, ...)
- Titel + Body
- Relative Zeit ("vor 3 Stunden")
- Bei Klick: zur `link`-Route springen + automatisch als gelesen markieren
- Optional: Action-Button "X" zum Löschen

Footer:

- "Alle als gelesen markieren" (nur wenn ungelesene da)
- "Alle ansehen" (Link auf `/notifications` falls dedizierte Seite)

**Optional:** Toast bei Live-Empfang neuer Notifications (über `sonner` / existierender `notify`-Helper in `apps/frontend/src/lib/notify.tsx`).

## Designentscheidungen (offen)

1. **Polling vs. SSE vs. WebSocket** für Live-Badge-Updates
   - Polling 60s ist simpel und reicht für MVP
   - SSE wäre nice-to-have für sofortige Reaktion auf Admin-Antworten

2. **Popover vs. Sheet vs. dedizierte Seite** (siehe oben)

3. **Default-Lebensdauer:** Sollen Notifications nach X Tagen automatisch gelöscht werden? (z.B. Cron: gelesene > 30 Tage löschen)

4. **Push-Browser-Notifications** (Service Worker / Web Push API) — out-of-scope für MVP, aber im Hinterkopf behalten

5. **E-Mail-Spiegelung:** Sollen bestimmte Notification-Typen auch per E-Mail an den User gehen? Wenn ja, in `apps/api/app/emails.py` integrieren und Notification-Type-Konfiguration ("send_email: bool") in User-Settings ermöglichen

6. **Admin-Tool:** Soll Admin manuell System-Announcements an alle User schicken können? (Neuer Admin-Bereich in `/admin/notifications`)

## Akzeptanzkriterien

- Backend hat `notifications`-Tabelle mit korrektem Schema und Alembic-Migration
- Endpoints liefern korrekte Daten und sind über Auth abgesichert (User sieht nur eigene Notifications)
- `create_notification()`-Service-Funktion existiert und wird mindestens vom Stripe-Webhook (Lizenzkauf), Support-Reply-Endpoint und Team-Invite-Endpoint aufgerufen
- Sidebar zeigt korrekten Badge-Count (live aktualisiert nach Bell-Click und periodisch)
- Klick auf Bell öffnet Panel, listet ungelesene Notifications zuerst
- Klick auf Notification springt zur Ziel-Route und markiert sie als gelesen
- "Alle als gelesen"-Button funktioniert
- Badge verschwindet bei 0 ungelesenen Notifications
- Tests: Backend-Unit-Tests für Service-Funktion + Endpoints; Frontend mindestens visuelle Verifikation

## Tests / Verifikation

Backend:

- Test: `create_notification` legt Datensatz korrekt an
- Test: `GET /notifications` liefert nur Notifications des authentifizierten Users
- Test: `POST /notifications/{id}/read` setzt `read_at`, schlägt fehl bei fremder Notification
- Test: `GET /notifications/unread-count` liefert korrekte Zahl
- Test: Stripe-Webhook für erfolgreichen Checkout erzeugt `license_purchased`-Notification
- Test: Support-Admin-Reply erzeugt `support_reply`-Notification für Ticket-Author
- Test: Team-Invite erzeugt `team_invite`-Notification für eingeladenen User

Frontend:

- Test: Badge zeigt korrekte Zahl, `9+` bei mehr als 9
- Test: Badge verschwindet nach "Alle als gelesen"
- Manuelle Verifikation: Notification erzeugen → Sidebar zeigt Badge nach max. 60s
- Manuelle Verifikation: Klick auf Notification springt zur korrekten Route

## Offene Prüfpunkte

- Welche Trigger-Events haben tatsächlich Priorität für den MVP? (Lizenzkauf + Support-Reply reichen für Phase 1?)
- DSGVO: Wie lange dürfen Notifications gespeichert bleiben?
- Internationalisierung: Notification-Texte direkt persistiert (DE) vs. i18n-Key + Variables im `meta`-Feld?
- Mobile: Wie verhält sich das Panel auf schmalen Viewports?

## Notizen / Fortschritt

- 2026-05-24: Todo angelegt nach Sidebar-Redesign. Bell-Icon und Badge-Mechanik sind bereits in [Sidebar.tsx](apps/frontend/src/components/layout/Sidebar.tsx) vorbereitet, `notifCount` und `handleBellClick` warten auf Anbindung.
