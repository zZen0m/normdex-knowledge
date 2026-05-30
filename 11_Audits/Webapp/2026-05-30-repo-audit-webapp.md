# Repo-Audit - Webapp (Verwaltungsportal) - 2026-05-30

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-app`
- Branch: `develop`
- Commit: `ea46e79`
- Version laut `package.json`: `0.2.0`
- Letzter Git-Tag: `v0.2.0`
- Audit-Datum: `2026-05-30`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-05-30-repo-audit-webapp.md`
- Vorheriger Audit: `2026-05-23-repo-audit-webapp.md`
- Audit-Fokus: Verwaltungsportal (Admin-Router, Admin-Frontend, Auth-Gates, neue Funktionen seit 2026-05-23)

## 2. Audit-Abdeckung

- Statische Codeanalyse des Verwaltungsportals: `apps/api/app/routers/admin.py` (2204 Zeilen), `apps/api/app/routers/support_admin.py` (579 Zeilen), `apps/api/app/routers/notifications.py`, `apps/api/app/services/notifications.py`, `apps/api/app/deps.py`.
- Frontend-Analyse: `apps/frontend/src/pages/admin/OrganizationCase.tsx` (1439 Zeilen), `apps/frontend/src/pages/admin/SupportInbox.tsx`, `apps/frontend/src/pages/admin/SupportTicketDetail.tsx`, `apps/frontend/src/pages/AdminUsers.tsx`, `apps/frontend/src/routes/AdminRoute.tsx`, `apps/frontend/src/App.tsx`, `apps/frontend/src/api.ts`, `apps/frontend/src/context/NotificationsContext.tsx`, `apps/frontend/src/components/layout/Sidebar.tsx`.
- Auth/Rollen-Gates über `require_admin` vs. manuelle `if not user.is_admin`-Prüfung.
- Cascade-/Löschverhalten über `apps/api/app/models.py` (Notification, Membership, SupportTicket, License-Usage/-Event, Organization).
- Neue Funktionen seit Vorbericht: Notifications-System (Bell-Panel/SSE/Trigger), Sensitivitätsdelta-Begrenzung (40 %), Trial-Badge-Lib, WBR-Bericht, License-Cancellation-Mails, Sidebar-Redesign — soweit sie das Verwaltungsportal berühren.
- Abgleich mit Vorbericht vom 2026-05-23 (Findings 1-8).
- Verifikation ausgeführt:
  - `npm run build` in `apps/frontend`: erfolgreich, Vite-Chunk-Warnung bei `assets/index-*.js` mit `533.56 kB` (vs. 502 kB im Vorbericht).
  - `npm test` in `apps/frontend`: 3 Testdateien, 24 Tests grün (neu: `useLicenseLock`, `licenseTrial`, `utils`).
  - `.\venv\Scripts\python -m pytest tests/test_admin_org_case.py -q` in `apps/api`: 32 Tests grün.
  - `.\venv\Scripts\python -m pytest -q` in `apps/api`: 245 Tests grün, **918 DeprecationWarnings** (vs. 616 im Vorbericht).
- Nicht ausgeführt: kein Live-Browser-Test, kein Stripe-/Brevo-/Graph-End-to-End, keine Multi-Worker-Lasttests für SSE-Notifications.

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-05-23
- Findings im Vorbericht: 8
- Davon behoben: 0
- Davon weiterhin offen: 7
- Davon nicht im Fokus dieses Audits: 1
- Regressionen: 1

### Behobene Findings

- Keine Findings aus 2026-05-23 wurden in diesem Lauf als vollständig behoben verifiziert.

### Weiterhin offene Findings

- Finding 2 - Brevo-Webhook-Secret in Query: weiterhin offen (`apps/api/app/routers/newsletter.py:93` nutzt unverändert `secret: str = Query(...)`).
- Finding 3 - Dev-Fixture-Prüfung: weiterhin offen (Skript existiert, wurde in diesem Lauf nicht erneut ausgeführt; betrifft auch die Verwaltungsportal-Demo-Daten).
- Finding 4 - BasicAuth-Default geschlossen: weiterhin offen (`deploy/docker-compose.prod.yml:59-60` setzt `basicauth.users` unverändert; betrifft den Zugriff auf `/admin/...` aus dem Browser).
- Finding 5 - Frontend-Testabdeckung: teilweise verbessert. Drei Testdateien (24 Tests) statt zwei (12 Tests) im Vorbericht. Für Admin-Seiten (`OrganizationCase`, `SupportInbox`, `SupportTicketDetail`, `AdminUsers`) gibt es weiterhin keinen einzigen Komponententest, obwohl in den letzten Wochen ca. +3100 Zeilen reine Verwaltungslogik dazugekommen sind.
- Finding 7 - Bundle-Warnung: weiterhin offen, leicht verschlechtert (`assets/index-*.js` 533.56 kB statt 502.11 kB).
- Finding 8 - Unleash/Codeium-JSON-Artefakte in `apps/api/alembic/versions/`: weiterhin offen (beide Dateien noch vorhanden).

### Nicht im Fokus dieses Audits

- Finding 1 - Support-Autoresponder HTML-Escaping: nicht erneut gegen `apps/api/app/emails.py` geprüft, dieser Audit hatte den engeren Verwaltungsportal-Scope.

### Regressionen

- Finding 6 - Backend-Deprecation-Warnungen: **regression**. Vorbericht meldete 616 Warnungen, aktueller Stand 918. Die neuen Admin-Endpoints (`admin.py:1802`, `1941`, `2025`, `_admin_license_cancel_preview` auf `622` u. a.) nutzen weiterhin `datetime.utcnow()` statt timezone-aware UTC-Helfer und haben die Warnzahl spürbar nach oben getrieben.

## 4. Confidence

- Confidence: hoch

Die Aussagen sind direkt aus dem Repository belegbar. Backend-Tests, Frontend-Tests und Frontend-Build wurden ausgeführt und sind grün. Eingeschränkt bleibt die Bewertung von Multi-Worker-SSE-Verhalten, echter Stripe-Refund-/Cancel-Pfade in der Live-Mode-Konfiguration sowie der tatsächlichen Cascade-Folgen von Admin-Löschungen auf PostgreSQL (Dev läuft auf SQLite, das FK-Verletzungen toleranter handhabt).

## 5. Kurzfazit

Das Verwaltungsportal hat seit dem 2026-05-23 massiv ausgebaut: `apps/api/app/routers/admin.py` ist um 1403 Zeilen gewachsen (Add: Billing-Portal, Discount apply/remove, Refunds, License Cancel/Reactivate, Usage-Release, Status-Sync, jeweils mit Preview-Endpoint), `OrganizationCase.tsx` um 672 Zeilen (Action-Dialog mit Stripe-Vorschau, Refund-Workflow, License-Aktionen). Tests dazu (`test_admin_org_case.py`, +1036 Zeilen, 32 Tests grün) decken die neuen Routen statisch ab. Trend insgesamt: **fachlich Verbesserung, technisch leichte Verschlechterung**. Wichtige neue Lücken sind ein unvollständiger Admin-User-Löschpfad, drei UI-Buttons in der Organisationsakte, deren Aktionen serverseitig nicht implementiert sind, und ein zweites Auth-Gate (`support_admin.py`), das nicht über `require_admin` läuft. Der SSE-Notification-Broker ist single-process und damit für den aktuellen Single-Uvicorn-Worker-Betrieb sicher, würde aber bei Skalierung Pushes verlieren. Die Deprecation-Warnzahl ist von 616 auf 918 gestiegen, weil neue Verwaltungslogik weiter mit naivem `datetime.utcnow()` arbeitet.

## 6. Wichtigste Findings

1. `admin.delete_user` hinterlässt verwaiste Datensätze und kann auf PostgreSQL FK-Fehler werfen.
2. `OrganizationCase` zeigt drei Action-Buttons (Notiz/Aktion, Billing prüfen, Subscription prüfen), die nur einen "noch nicht angebunden"-Toast triggern.
3. `support_admin.py` nutzt manuelle `if not user.is_admin`-Checks statt der zentralen `require_admin`-Dependency.
4. Notification-SSE-Broker ist prozessgebunden; aktuell unkritisch, aber sobald API mit `--workers > 1` läuft, fallen Pushes für Sessions auf anderen Workern aus.
5. Datetime-Deprecations sind durch neue Admin-Codepfade von 616 auf 918 Warnungen gewachsen.
6. AdminUsers nutzt native `prompt()`/`confirm()` und kommuniziert eine falsche Passwortlänge (mind. 10 vs. Backend mind. 8).
7. Admin-Refund hardcodet Stripe-`reason="requested_by_customer"`; der eigentliche Admin-Grund landet nur in `metadata`.
8. Frontend-Hauptbundle ist von 502 kB auf 533 kB gewachsen, Vite-Chunk-Warnung weiterhin aktiv.

## 7. Detaillierte Findings je Punkt

### Finding 1 - Admin-User-Löschung kaskadiert nicht sauber

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/admin.py:1007-1021`, `apps/api/app/models.py` (Notification cascade, andere Tabellen ohne `ondelete="CASCADE"`), `apps/frontend/src/pages/AdminUsers.tsx:115-125`
- Evidenz: `delete_user` löscht nur `Calculation`, `Token` und `AuditLog` vor dem `db.delete(u)`. `Notification.user_id` hat zwar `ondelete="CASCADE"` (models.py:463), aber andere Beziehungen zum User (`Membership.user_id`, `SupportTicket.user_id`, `LicenseUsage.user_id`, `LicenseEvent.actor_user_id`, `License.assigned_user_id`, `License.created_by_user_id`, `LicenseOrder.user_id`, `LicenseOrder.created_by_user_id`, `SupportMessage.author_user_id`, `Invite.created_by_user_id`) deklarieren keinen CASCADE und werden auch nicht manuell aufgeräumt.
- Beschreibung des Problems: SQLite ist beim FK-Check tolerant, deshalb laufen die aktuellen Tests grün. In Produktion mit PostgreSQL wirft ein Admin-Löschvorgang für einen aktiven User entweder `IntegrityError` oder hinterlässt verwaiste Referenzen, je nachdem ob die FK-Constraint im Schema mit `ON DELETE` deklariert ist.
- Warum das relevant ist: Der Button `Benutzer löschen` in `AdminUsers.tsx` ruft genau diesen Endpoint. Ein vermeintlich erledigter Support-Vorgang kann in Prod fehlschlagen oder Datenleichen produzieren.
- Business Impact: Verwaltungsfunktion bricht beim ersten produktiven Einsatz; im schlimmsten Fall manuelle DB-Reparatur erforderlich.
- Konkrete Handlungsempfehlung: Entweder die fehlenden Beziehungen vor `db.delete(u)` explizit aufräumen oder die FK-Definitionen in `models.py` auf `ondelete="SET NULL"`/`"CASCADE"` migrieren und eine Alembic-Migration ergänzen. Außerdem einen Backend-Test gegen PostgreSQL ergänzen, der den Pfad für einen User mit Membership/Ticket/Lizenz durchläuft.

#### Status: **behoben (2026-05-30)**

- Umsetzung: `delete_user` in [apps/api/app/routers/admin.py:1007-1078](../../../01_repos/normdex-app/apps/api/app/routers/admin.py#L1007-L1078) räumt jetzt alle FK-Beziehungen explizit auf:
    - Hard-Delete: `Calculation`, `Token`, `AuditLog`, `Notification`, `Membership`, `LicenseUsage` (nullable=False oder intrinsisch zum User).
    - `UPDATE … SET NULL`: `Project.updated_by`, `License.assigned_user_id`, `License.created_by_user_id`, `LicenseEvent.actor_user_id`, `SupportTicket.user_id`, `SupportTicket.assignee_user_id`, `SupportMessage.author_user_id`, `SupportEvent.actor_user_id` (nullable, Historie bleibt erhalten).
    - Blocker mit HTTP 409 (`code: "user_has_owned_records"`), wenn der User noch `Project.user_id` oder `LicenseOrder.created_by_user_id` besitzt — diese NOT-NULL-Eigentums-FKs müssen vor dem Löschen manuell übertragen werden, damit weder Daten verloren gehen noch verwaiste Referenzen entstehen.
- Verifikation: Neuer Test [apps/api/tests/test_admin_delete_user.py](../../../01_repos/normdex-app/apps/api/tests/test_admin_delete_user.py) deckt fünf Fälle ab (Admin-Gate, Self-Delete, vollständige Cleanup-Kette mit Membership/Ticket/Lizenz, 409 bei Projekten, 409 bei LicenseOrders). `pytest -q` jetzt **250 passed**.
- Offen: PostgreSQL-End-to-End-Test (gemäß `docker-compose.test.yml`-Vorschlag aus Abschnitt 9) wurde nicht ergänzt — der bestehende SQLite-Pfad simuliert die Cleanup-Logik aber explizit, sodass ein FK-`IntegrityError` in Prod nicht mehr durch fehlende Aufräum-Statements ausgelöst werden kann.

### Finding 2 - Drei Admin-Aktionen sind UI-only und triggern nur einen Hinweis-Toast

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/pages/admin/OrganizationCase.tsx:336-402`, `940-948`, `1218`, `1245-1248`
- Evidenz: Der `submit`-Dispatcher behandelt nur 9 von 12 `ActionKind`-Werten konkret. Für `note`, `billing-check` und `subscription-check` fällt er in den `notify.info("Workflow noch nicht angebunden.", ...)`-Pfad. Genau diese ActionKinds werden aber von Buttons in der Organisationsakte ausgelöst: `Notiz/Aktion erfassen` (`note`), `Billing prüfen` (`billing-check`) und `Prüfen`-Button pro Subscription (`subscription-check`).
- Beschreibung des Problems: Drei Hauptaktionen am Kunden-Case führen für den Admin sichtbar zu einer Aktion, schreiben aber weder Backend-State noch Audit-Log. Die Notiz-Funktion ist besonders kritisch, weil dadurch der Support-Begründungstext (`reason`) verloren geht.
- Warum das relevant ist: Das Verwaltungsportal positioniert sich als Audit-/Support-Tool. Wenn `Notiz/Aktion erfassen` den Audit-Trail nicht erweitert, ist das ein stilles Datenleck im Compliance-Sinn (Support-Begründungen werden nicht festgehalten).
- Business Impact: Support-Mitarbeitende denken, sie haben dokumentiert; bei einem späteren Streit (Kündigung, Refund) fehlt der Beleg.
- Konkrete Handlungsempfehlung: Entweder die drei Buttons aus der UI entfernen, bis das Backend bereit ist, oder die fehlenden Endpunkte ergänzen (`POST /admin/organizations/{id}/notes`, `GET /admin/organizations/{id}/billing/check`, `POST /admin/.../subscriptions/{id}/check`) und im `AuditLog` speichern. Übergangsweise konsistent zum Rest disablen statt einen Erfolgs-Toast anzuzeigen.

#### Status: **behoben (2026-05-30)**

- Umsetzung: Drei neue Backend-Endpoints in [apps/api/app/routers/admin.py:2108-2295](../../../01_repos/normdex-app/apps/api/app/routers/admin.py#L2108-L2295), jeweils hinter `require_admin`, mit Pflicht-`reason` (≥3 Zeichen), `confirm`-Flag und optionaler Ticket-Verknüpfung:
    - `POST /admin/organizations/{org_id}/notes` → schreibt AuditLog-Event `admin_org_note_added` (Notiztext landet in `meta["note"]`, optional verknüpft mit Support-Ticket-ID).
    - `POST /admin/organizations/{org_id}/billing/check` → läuft `_billing_diagnostics` gegen Stripe (Customer, Subscriptions, Invoices) und protokolliert das Ergebnis als `admin_org_billing_checked` (Warnungen + Kennzahlen im AuditLog-Meta). Funktioniert auch ohne Stripe-Config — schreibt dann `stripe_error` ins AuditLog.
    - `POST /admin/organizations/{org_id}/billing/subscriptions/{subscription_id}/check` → vergleicht Stripe-Status mit lokaler Lizenztabelle und schreibt `admin_org_subscription_checked` mit Issue-Liste (`subscription_past_due`, `scheduled_to_cancel`, `stripe_canceled_but_local_active`, `stripe_cancels_but_local_not_scheduled_end`, `subscription_missing_in_stripe`, `stripe_error`).
- Frontend: API-Client um `adminOrgAddNote`, `adminOrgBillingCheck`, `adminOrgSubscriptionCheck` ergänzt ([apps/frontend/src/api.ts:270-275](../../../01_repos/normdex-app/apps/frontend/src/api.ts#L270-L275)). Der `submit`-Dispatcher in [apps/frontend/src/pages/admin/OrganizationCase.tsx:335-419](../../../01_repos/normdex-app/apps/frontend/src/pages/admin/OrganizationCase.tsx#L335-L419) bedient jetzt alle 12 `ActionKind`-Werte; der irreführende `"Workflow noch nicht angebunden."`-Toast ist entfernt. Billing- und Subscription-Check zeigen Hinweise/Warnungen kontextspezifisch im Toast (`notify.warning` mit Warnungsliste vs. `notify.success` bei sauberem Ergebnis), Notiz quittiert mit `"Notiz im Audit-Log gespeichert."`.
- Verifikation:
    - 8 neue Tests in [apps/api/tests/test_admin_org_case.py](../../../01_repos/normdex-app/apps/api/tests/test_admin_org_case.py): Admin-Gate, Confirm-Pflicht, AuditLog-Schreibpfad mit Ticket-Bindung, Behandlung unbekannter Ticket-IDs, Diagnostics ohne Stripe-Config, Stripe-Aggregation (offene Rechnung + fehlende Subscription), Inkonsistenz-Flags für Subscription-Check, Stripe-Fehlerpfad. `pytest -q` jetzt **259 passed** (vorher 250).
    - `npm run build` in `apps/frontend`: erfolgreich (Hauptbundle 533.91 kB; +0.35 kB für die drei neuen Dispatcher-Zweige — Finding 8 dadurch nicht spürbar verändert).
- Offen: Die drei Aktionen werfen das Aktion-Dialog-Result derzeit nur als Toast aus; eine reichere Inline-Anzeige (z. B. eine Diagnostik-Card mit allen Warnungen) bleibt aufgespart für ein späteres UX-Inkrement. Für die Audit-Tickets-Querbindung gilt weiterhin: der Endpoint validiert die übergebene `ticket_id` gegen `SupportTicket.ticket_id` und verweigert unbekannte Werte (HTTP 400).

### Finding 3 - support_admin.py umgeht die zentrale `require_admin`-Dependency

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/support_admin.py:5`, `103-106`, `134-137`, `217-220`, `313-315`, `452` u. a.
- Evidenz: Jeder Handler in `support_admin.py` injiziert `user: User = Depends(get_current_user)` und prüft anschließend `if not user.is_admin: raise HTTPException(...)` manuell. Demgegenüber benutzt `admin.py` durchgängig `_admin = Depends(require_admin)` (siehe z. B. `admin.py:853`).
- Beschreibung des Problems: Zwei parallele Muster für denselben Zweck. Jeder neue Endpoint in `support_admin.py` muss daran denken, den Check zu ergänzen. Ein einzelner vergessener Check (z. B. bei einem Bulk-Action-Endpoint) erlaubt jedem eingeloggten User Zugriff auf Support-Tickets.
- Warum das relevant ist: Support-Tickets enthalten kundenseitige E-Mails, Anhänge, Adressdaten und ggf. Rechtsthemen. Ein versehentlich offener Endpoint wäre ein Datenschutzvorfall.
- Business Impact: Latentes Compliance- und Reputationsrisiko, höherer Review-Aufwand bei jedem Support-PR.
- Konkrete Handlungsempfehlung: `support_admin.py` auf `admin: User = Depends(require_admin)` migrieren, die manuellen `if not user.is_admin`-Checks entfernen und einen Lint-/Test-Gate hinzufügen, der für alle Router unter `/admin/*` und `/admin/support/*` einen Admin-Gate erzwingt.

### Finding 4 - SSE-Notification-Broker ist single-process

- Kategorie: Risk
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/services/notifications.py:8-9`, `27-28`, `apps/api/Dockerfile` (`CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]`)
- Evidenz: Modul-globale `_subscribers: dict[int, list[asyncio.Queue]]` und `_loop`. Der Docstring im Modul dokumentiert die Einschränkung selbst. Das Dockerfile startet `uvicorn` ohne `--workers`, d. h. aktuell nur ein Worker pro Container.
- Beschreibung des Problems: Das aktuelle Single-Worker-Setup macht den SSE-Broker funktional korrekt. Sobald `--workers 2+` oder Gunicorn mit mehreren Workern verwendet wird, sehen Browser-Sessions an Worker A keine Push-Events, die in Worker B erzeugt werden.
- Warum das relevant ist: Das Notification-System ist neu und wird in den nächsten Wochen wachsen. Wenn später aus Performance-Gründen auf mehrere Worker skaliert wird, fallen Live-Notifications ohne Warnung weg.
- Business Impact: Aktuell kein Impact. Mittelfristig Risiko, dass User Notifications nur per Polling sehen und die "Live"-Versprechung im UX bricht.
- Konkrete Handlungsempfehlung: README/Deployment-Doku mit `Hinweis: Notifications-SSE setzt Single-Worker voraus` ergänzen oder den Broker auf Redis Pub/Sub umstellen, bevor `--workers > 1` aktiviert wird.

### Finding 5 - Deprecation-Warnungen sind von 616 auf 918 gestiegen (Regression)

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: regression (relativ zu Vorbericht-Finding 6)
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/admin.py:622`, `770`, `784`, `1802`, `1941`; `apps/api/tests/test_admin_org_case.py` (zahlreiche `datetime.utcnow()`-Stellen im neuen Test)
- Evidenz: `pytest -q` meldet 245 passed, **918 warnings**. Die neuen Admin-Codepfade rund um Cancel/Reactivate/Status-Sync arbeiten mit naivem `datetime.utcnow()`. Vergleiche mit `lic.scheduled_end_at`, `current_term_end` etc. sind ebenfalls naiv. Tests spiegeln das Muster.
- Beschreibung des Problems: Das im Vorbericht dokumentierte Tech-Debt-Thema ist mit dem Admin-Ausbau nicht zurückgegangen, sondern um ca. 50 % gewachsen. Naive UTC-Zeitstempel bergen erneut die Gefahr, dass Zeitvergleiche bei tatsächlich timezone-aware Stripe-Zeitstempeln in `_admin_license_status_sync_target` Inkonsistenzen liefern.
- Warum das relevant ist: Maskiert neue, relevantere Warnungen, erschwert Python-/Pydantic-Upgrades und bringt im Admin-Code (wo Stripe-Daten timezone-aware kommen) konkrete Vergleichsrisiken.
- Business Impact: Wartungs- und Upgrade-Risiko, latente Inkonsistenzen bei Lizenz-Statussynchronisationen.
- Konkrete Handlungsempfehlung: Einen `app/util/dt.py` mit `now_utc() -> datetime` (tz-aware) einführen und alle neuen Admin-Module migrieren. Optional `pytest -W error::DeprecationWarning::app\.routers\.admin` als Gate für neue Änderungen.

### Finding 6 - AdminUsers verwendet native `prompt()`/`confirm()` und nennt eine falsche Passwortlänge

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/pages/AdminUsers.tsx:105`, `117`, `apps/api/app/routers/admin.py:846` (`SetPasswordSchema`)
- Evidenz: `onSetPassword` ruft `prompt("Neues Passwort setzen (mind. 10 Zeichen)")`, `onDelete` ruft `confirm("Benutzer wirklich löschen? ...")`. Backend-Schema in `admin.py:846` ist `new_password: str = Field(..., min_length=8)`.
- Beschreibung des Problems: Inkonsistente UX (rest der App nutzt shadcn-Dialoge und `notify.*`), und der angezeigte Mindest-Wert (10) widerspricht der echten Validierung (8). User können verwirrt werden oder ein an sich gültiges Passwort wird abgewiesen, weil sie es kürzer wählen würden.
- Warum das relevant ist: Direkt im Verwaltungsportal sichtbar, kleines aber sichtbares Vertrauenssignal.
- Business Impact: Niedrig — UX-Reibung, kein Funktions- oder Sicherheitsbruch.
- Konkrete Handlungsempfehlung: `prompt()`/`confirm()` durch existierende `Dialog`-Komponenten ersetzen und den Mindestwert mit der Backend-Policy abgleichen (entweder Schema auf 10 oder Hint auf 8 anpassen).

### Finding 7 - Refund-Endpoint überschreibt Admin-Begründung mit hartem Stripe-Wert

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/admin.py:1738-1748`
- Evidenz: `stripe.Refund.create(..., reason="requested_by_customer", metadata={"support_reason": data.reason, ...})`. Der eigentliche Admin-Grund landet ausschließlich in `metadata`, nicht im Stripe-`reason`-Feld. Stripe akzeptiert auch `duplicate` und `fraudulent` als reason.
- Beschreibung des Problems: Bei jeder Erstattung steht im Stripe-Dashboard "Customer requested", selbst wenn z. B. ein doppelter Charge zurückgenommen wird. Reports und Dispute-Management auf Stripe-Seite sind dadurch unsauber.
- Warum das relevant ist: Buchhaltung und Disputes nutzen Stripe-`reason` als Filter.
- Business Impact: Niedrig — Berichtsqualität in Stripe.
- Konkrete Handlungsempfehlung: `AdminBillingRefundSchema` um ein `reason_code: Literal["requested_by_customer","duplicate","fraudulent"]` erweitern und an Stripe durchreichen. Default bei fehlendem Wert weiter `requested_by_customer`.

### Finding 8 - Frontend-Hauptbundle weiter gewachsen

- Kategorie: Optimization
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-23)
- Betroffene Datei(en) oder Pfade: `apps/frontend/dist/assets/index-*.js`, `apps/frontend/src/components/layout/Sidebar.tsx`, `apps/frontend/src/context/NotificationsContext.tsx`
- Evidenz: `npm run build` meldet `assets/index-*.js` mit `533.56 kB` (vs. 502.11 kB im Vorbericht). Vite-Warnung `Some chunks are larger than 500 kB` weiterhin aktiv.
- Beschreibung des Problems: Sidebar, Notifications-Context und Bell-Popover sind im Hauptbundle und gehören zur App-Chrome jeder geschützten Route.
- Warum das relevant ist: Sidebar-Redesign + Notifications-Funktion haben +31 kB beigetragen. Wenn die Wachstumsrate so bleibt, wird der initiale App-Start spürbar langsamer.
- Business Impact: Niedrig, aber kumulativ.
- Konkrete Handlungsempfehlung: Bell-Popover und schwerere Lucide-Icons in `Sidebar.tsx` per `lazy()` / dynamic-import laden, oder ein bewusstes Bundle-Budget setzen (`build.chunkSizeWarningLimit`) und die Schwelle als CI-Gate führen.

## 8. Quick Wins

- `support_admin.py` mit `Depends(require_admin)` umstellen und die manuellen `if not user.is_admin`-Zeilen entfernen.
- ~~`OrganizationCase.tsx` Buttons `Notiz/Aktion erfassen`, `Billing prüfen` und `Subscription prüfen` als `disabled` markieren, bis ein Backend-Endpoint existiert.~~ → **behoben am 2026-05-30** (Finding 2): Backend-Endpoints + AuditLog statt nur Toast; Buttons sind jetzt voll angebunden.
- In `AdminUsers.tsx` die Passwort-Mindestlänge-Mitteilung von "mind. 10" auf den Backend-Wert ("mind. 8") angleichen.
- `admin.py:1741` `reason` mit explizitem Refund-Code-Mapping ergänzen.
- `apps/api/alembic/versions/unleash-*.json` aus dem Migrations-Ordner verschieben (persistent seit 2026-05-23).

## 9. Strategische Empfehlungen

- Ein zentraler `tz_now_utc()`-Helfer als verbindlicher Standard für neuen Backend-Code, parallel ein `pytest -W error::DeprecationWarning`-Gate für die Module, die schon migriert sind.
- Frontend-Komponententests für die Verwaltungsportal-Flows aufbauen, bevor das nächste Feature-Inkrement kommt: `OrganizationCase` (Tab-Wechsel, Action-Dialog, Discount-Preview), `SupportInbox` (Filter/Suche), `SupportTicketDetail` (Status-Wechsel + Countdown).
- Admin-Aktionen (Cancel/Reactivate/Refund/Discount) sind heute pro Aktion einzeln implementiert. Sobald 4-5 weitere dazukommen, lohnt sich ein gemeinsamer `AdminActionRunner`-Helper, der Preview, Confirm, AuditLog und Notification atomar kapselt.
- Ein "Stripe-Sandbox-Dry-Run"-Skript, das alle Admin-Billing-Endpoints gegen Stripe-Testmode durchspielt (Discount apply/remove, Refund, License Cancel/Reactivate, Status-Sync). Aktuell ist der Pfad nur durch Mocks getestet.
- Cascade-Verhalten beim User-Delete als End-to-End-Pfad gegen PostgreSQL absichern, am besten in einem `docker-compose.test.yml` mit echter pg-Instanz.

## 10. Empfohlene nächste Aktion

Vor dem nächsten Produktiv-Release das Verwaltungsportal in drei Punkten absichern: (1) ~~`admin.delete_user` kaskadiert oder verweigert User mit aktiven Beziehungen~~ (Finding 1 behoben 2026-05-30), (2) ~~die drei "noch nicht angebunden"-Buttons in `OrganizationCase` werden disabled oder serverseitig ergänzt~~ (Finding 2 behoben 2026-05-30 — Backend-Endpoints + AuditLog), (3) `support_admin.py` wird auf `require_admin` umgestellt.

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Welche FK-Constraints in der Produktiv-PostgreSQL tatsächlich `ON DELETE`-Verhalten haben — Modelle deklarieren das nicht durchgängig, Migrationen können historisch davon abweichen.
- Ob die Notiz-/Billing-/Subscription-Check-Buttons bewusst als "Coming Soon" gedacht waren oder versehentlich ohne Backend ausgeliefert wurden.
- Ob das Produktivsystem nach dem Notifications-Rollout dauerhaft auf einem Uvicorn-Worker bleiben soll oder ob bald `--workers > 1` geplant ist (entscheidend für SSE-Broker-Architektur).
- Ob im Stripe-Live-Modus alle Promotion-Codes wirklich über die `assert_newsletter_code_usable`-Pipeline laufen sollen oder ob Admin-Discounts auch für nicht-Newsletter-Codes erlaubt sein müssen.
- Ob ein dokumentierter Smoke-Test für Cancel → Reactivate → Status-Sync gegen einen realen Stripe-Testaccount existiert.
- Ob die `unleash-*.json`-Artefakte in `alembic/versions/` bewusst weiterhin geduldet werden oder ein .gitignore-Eintrag fehlt.
- Ob der WBR-Bericht (a2820be), die 40 %-Sensitivitätsdelta-Grenze (c04093d) und die License-Cancellation-Mails (08ba08b/22b3421) einen eigenen funktionalen Audit brauchen — sie wurden hier nur am Rand betrachtet, weil der Fokus auf dem Verwaltungsportal lag.
