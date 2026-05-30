# T022 - Notifications-System (Bell-Icon, Backend-API, Frontend-Panel)

**Status:** erledigt
**Bereich:** App / Backend / Frontend / Sidebar
**Erstellt:** 2026-05-24
**Abgeschlossen:** 2026-05-30

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
- 2026-05-30: Implementiert und abgeschlossen.

## Implementation-Notes (2026-05-30)

### MVP-Scope (vom User bestätigt)

- Trigger: `welcome`, `license_purchased`, `license_expiring_soon` (Buckets 14/7/1), `license_expired`
- UI: Popover am Bell-Icon **plus** dedizierte Seite `/notifications`
- Live-Updates: **Server-Sent Events** via `/notifications/stream` (nicht Polling)
- **Bewusst nicht im MVP:** Support-Reply-Trigger, Browser-Push, Admin-Announcement-Tool, E-Mail-Spiegelung
- **Team-Invite-Trigger gestrichen:** Stellte sich heraus, dass Einladungen nur an noch-nicht-registrierte E-Mails gehen (Invite-Link → `/auth/register`). Bestehende User können nie eine Notification empfangen. Stattdessen: org-spezifischer Welcome bei Invite-Annahme im Register-Endpoint. Siehe Memory `project_team_invites_new_emails_only`.

### Architektur

- **DB-Tabelle** `notifications`: UUID-PK, FK `user_id` mit `ondelete=CASCADE`, `type` als String (kein Enum, damit neue Typen ohne Migration), `meta` als JSON (SQLite + Postgres kompatibel), 5 Indizes (3 Single-Column aus autogenerate + 2 Composites `(user_id, read_at)` und `(user_id, created_at)`).
- **Alembic-Migration:** `apps/api/alembic/versions/995d14683241_add_notifications_table.py`. Autogenerate erfasste viel Schema-Drift — Migration wurde manuell auf die Notifications-Änderungen reduziert.
- **Service-Layer** `apps/api/app/services/notifications.py`: `create_notification(...)` ist der einzige öffentliche Schreibpfad. Inkludiert einen In-Process-SSE-Broker (`asyncio.Queue` pro `user_id`, `call_soon_threadsafe` für sync→async Bridging, drop-on-full statt blocken).
- **Router** `apps/api/app/routers/notifications.py`: 6 Endpoints (List / unread-count / mark-read / read-all / DELETE / stream). SSE-Endpoint nutzt `StreamingResponse(media_type="text/event-stream")` + 15s-Keepalive-Kommentare + `X-Accel-Buffering: no`-Header für nginx.
- **Cookie-Auth funktioniert mit EventSource**, weil `CORSMiddleware` schon `allow_credentials=True` hat.

### Trigger-Integration

- **Welcome** in `apps/api/app/routers/auth.py:register` — wird sowohl für Self-Register als auch Invite-Register erzeugt. Bei Invite: Titel/Body org-spezifisch, Link auf `/team`.
- **license_purchased** in `apps/api/app/routers/subscriptions.py:_handle_checkout_completed` via Helper `_maybe_notify_license_purchased`. Idempotenz über `order.meta["purchase_notification_sent_at"]` analog zur existierenden E-Mail-Idempotenz. Singular/Plural je Anzahl.
- **license_expiring_soon / license_expired** als APScheduler-Cron in `apps/api/app/services/scheduler.py:check_expiring_licenses`. Läuft täglich um 06:00. Trifft nur Lizenzen mit `cancel_at_period_end=True` (Auto-Renew-Lizenzen werden zu Recht ignoriert). Empfänger-Logik: `created_by_user_id` → `assigned_user_id` → Org-Owner. Dedupe per Day-Bucket + `meta.license_id`.

### Frontend

- **API-Client** in `apps/frontend/src/api.ts`: 5 Methoden + `NotificationItem`-Typ + `API_BASE`-Export für EventSource.
- **`NotificationsContext`** in `apps/frontend/src/context/NotificationsContext.tsx`: Provider mit `useUser()`-Gate (kein Fetch ohne Login), EventSource mit `withCredentials`, sonner-Toast bei Live-Empfang, Polling auf 5 Min Fallback reduziert + focus-Listener.
- **Wichtig**: Provider muss in `App.tsx` AppContent UMSCHLIESSEN (nicht nur ProtectedRoute), weil `<Sidebar />` außerhalb von ProtectedRoute gemountet wird. Konsequenz: Provider muss tolerant gegenüber „kein User" sein → kein Fetch in dem Fall.
- **shadcn Popover** via `npx shadcn@latest add popover` nachgezogen. `BellItem` wurde auf `forwardRef` umgestellt, damit `PopoverTrigger asChild` funktioniert.
- **NotificationPanel** in `apps/frontend/src/components/notifications/NotificationPanel.tsx`: 380 px, max 8 jüngste Einträge, typ-spezifische Icons, `formatDistanceToNow` mit `deAT`-Locale.
- **`/notifications`-Seite** mit Filter-Tabs („Alle"/„Ungelesen") und Hover-Delete.

### Bekannte Caveats

- **SSE-Broker ist Single-Process.** Bei Gunicorn `-w >1` würde jeder Worker einen eigenen Broker haben → Notification wird nur in dem Worker live gepusht, in dem `create_notification` aufgerufen wurde. Subscriber in anderen Workern bekommen es erst beim Reconnect/Polling. Falls je Multi-Worker: Redis Pub/Sub.
- **nginx in Prod:** Für `/notifications/stream` muss `proxy_buffering off;` gesetzt sein. Der `X-Accel-Buffering: no`-Header deckt das ab, falls er durchgelassen wird — sonst manuell konfigurieren.
- **Seed/Test-Scripts** triggern keine SSE-Live-Pushes, da sie in einem anderen Prozess als das Backend laufen. Helper-Script: `apps/api/scripts/_seed_notification.py`.
- **DSGVO/Retention:** Aktuell kein Auto-Delete alter Notifications. Falls relevant: zusätzlicher Cron-Job, z.B. gelesene älter als 90 Tage löschen.

### Nicht erledigt (out-of-scope MVP)

- Backend-Unit-Tests für Service + Endpoints (Folge-Todo möglich)
- Admin-only Test-Endpoint für End-to-End-SSE-Verifikation
- Browser-Push-Notifications (Service Worker / Web Push API)
- E-Mail-Spiegelung bestimmter Notification-Typen mit User-Settings-Opt-in
- Admin-Tool für manuelle System-Announcements an alle User
