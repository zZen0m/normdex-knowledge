# Repo-Audit - Webapp - 2026-05-22

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-app`
- Branch: `develop`
- Commit: `1303595`
- Version laut `package.json`: `0.2.0`
- Letzter Git-Tag: `v0.2.0`
- Audit-Datum: `2026-05-22`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-05-22-repo-audit-webapp.md`
- Vorheriger Audit: kein Vorbericht vorhanden

## 2. Audit-Abdeckung

- Statische Codeanalyse von `apps/frontend/`, `apps/api/`, `deploy/`, `.gitignore`, Alembic-Migrationen und zentralen Konfigurationsdateien.
- Backend/API-Prüfung mit Fokus auf FastAPI-Routen, Auth, Rollen/Rechte, Organisations- und Lizenzflows, Support, Newsletter, Datenschutzfunktionen und Datenmodell.
- Frontend/Routen-/Komponentenprüfung mit Fokus auf Routing, API-Client, geschützte Routen, Support-/Profilflows, Lizenzverwaltung und Build-Konfiguration.
- Datenbank-/Migrationsprüfung statisch über `apps/api/app/models.py`, `apps/api/alembic/versions/` und Deployment-Env-Beispiele; keine Alembic-Migration wurde ausgeführt.
- Metadaten-/SEO-/Routing-Prüfung über `index.html`, `PageMeta`, `robots.txt`, `sitemap.xml` und `App.tsx`.
- Vault-Abgleich mit `00_Start/AI Kontext - Einstieg.md`, `10_Aufgaben/Aufgaben.md`, `T013-lizenzsystem-rollout-abschliessen.md`, `T017-testzeitraum-fuer-lizenzen.md` und `T019-newsletter-gutschein-brevo-webhook-rollout.md`.
- Verifikation ausgeführt:
  - `npm run build` in `apps/frontend`: erfolgreich, aber Vite warnt vor einem großen JS-Chunk.
  - `npm test` in `apps/frontend`: 2 Testdateien, 12 Tests grün.
  - `.\venv\Scripts\python -m pytest` in `apps/api`: 174 Tests grün, 2 Tests fehlgeschlagen.
  - `.\venv\Scripts\python -m pip check` in `apps/api`: keine kaputten Python-Abhängigkeiten.
  - `npm audit --omit=dev` in `apps/frontend`: 9 Production-Vulnerabilities, darunter kritisch.
- Keine verlässliche Laufzeitprüfung im Browser, keine Stripe-End-to-End-Prüfung und keine Prüfung gegen Produktiv-/Staging-Datenbank durchgeführt.

## 4. Confidence

- Confidence: mittel

Die statische Abdeckung ist breit und Build/Test/Audit-Kommandos wurden ausgeführt. Die wichtigsten Risiken sind direkt durch Repository-Zustand oder Toolausgaben belegbar. Eingeschränkt bleibt die Aussagekraft bei Stripe-Live-Konfiguration, realem Deployment-Verhalten, Browser-UX und rechtlicher Bewertung von Datenexport/Kontolöschung, weil diese Punkte nicht live verifiziert wurden.

## 5. Kurzfazit

Der aktuelle Branch ist funktional weit entwickelt, aber nicht release-sauber. Frontend-Build und Frontend-Tests laufen, der Backend-Testlauf ist aktuell rot und blockiert einen belastbaren Release. Das größte technische Risiko liegt in Production-Abhängigkeiten mit kritischen npm-Advisories, insbesondere durch das offenbar ungenutzte Paket `tremor@0.0.1`. Die Dev-Datenbank ist bewusst als anonymisierter Development-Snapshot versioniert; dafür braucht es klare Fixture-Governance und eine Prüfung vor DB-Snapshot-Commits. Der Lizenz-/Stripe-Rollout ist im Code weit fortgeschritten, aber die Deployment-Env-Beispiele passen noch nicht vollständig zum neuen Pool-Modell. Mehrere Admin-/Datenschutzflows brauchen vor einem Produktivrelease eine gezielte Nacharbeit.

## 6. Wichtigste Findings

1. Kritische Production-Vulnerabilities in Frontend-Abhängigkeiten.
2. Backend-Test-Suite ist rot: 2 fehlgeschlagene E-Mail-/Newsletter-Tests.
3. Versionierte Dev-Datenbank braucht Fixture-Governance.
4. Deployment-/Env-Beispiele sind nicht konsistent mit dem Lizenz-Pool-Modell.
5. Produktions-Frontend ist in `docker-compose.prod.yml` per statischem BasicAuth-Gate geschützt.
6. Organisations-Owner kann sich bzw. den letzten Owner zur Member-Rolle degradieren.
7. Datenexport und Kontolöschung decken relevante personenbezogene Daten nicht vollständig ab.
8. Support-Upload verlässt sich bei Größe/Typ teilweise auf Frontend-Validierung.
9. Frontend-Bundle wird ohne Code-Splitting als großer Hauptchunk ausgeliefert.

## 7. Detaillierte Findings je Punkt

### Finding 1 - Kritische npm-Vulnerabilities in Production-Abhängigkeiten

- Kategorie: Risk
- Priorität: kritisch
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/frontend/package.json`, `apps/frontend/package-lock.json`
- Evidenz: `npm audit --omit=dev` meldet 9 Vulnerabilities: 2 kritisch, 6 hoch, 1 moderat. Betroffen sind unter anderem `tremor@0.0.1` mit `underscore@1.5.2`, `react-router-dom@6.30.1`, `dompurify@3.3.3`, `lodash@4.17.21` über `@tremor/react`/`recharts` und `minimatch` über das alte `tremor`-Paket. `rg -n tremor` zeigt, dass `tremor` nur in `package.json` steht; im Code wird `@tremor/react` importiert, nicht `tremor`.
- Beschreibung des Problems: Das alte Paket `tremor@0.0.1` zieht kritische transitive Abhängigkeiten (`underscore`, altes `glob`/`minimatch`) in die Produktionsinstallation. Zusätzlich bestehen relevante advisories in aktiv genutzten UI-/Routing-/Sanitizing-Abhängigkeiten.
- Warum das relevant ist: Die App verarbeitet Login, Lizenzen, Supportdaten und teilweise HTML-Inhalte. XSS-/Prototype-Pollution-/DoS-Risiken in Production-Abhängigkeiten sind für eine B2B-SaaS-App nicht akzeptabel.
- Business Impact: Sicherheits- und Compliance-Risiko, potenzieller Release-Blocker, Vertrauensverlust bei Kunden, erhöhter Incident-Aufwand.
- Konkrete Handlungsempfehlung: `tremor` entfernen, wenn ungenutzt. Danach `npm audit --omit=dev` erneut ausführen. `react-router-dom`, `dompurify`, `html2pdf.js`/transitive DOMPurify-Nutzung sowie `@tremor/react`/`recharts` gezielt aktualisieren oder ersetzen, bis der Production-Audit mindestens keine kritischen/hohen Findings mehr meldet.

### Finding 2 - Backend-Testlauf ist aktuell rot

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/api/tests/test_auth_emails.py`, `apps/api/tests/test_newsletter.py`, `apps/api/app/emails.py`, `apps/api/app/routers/auth.py`, `apps/api/app/services/newsletter_coupon_service.py`
- Evidenz: `.\venv\Scripts\python -m pytest` ergibt `2 failed, 174 passed`. Fehlgeschlagen sind `test_password_forgot_sends_reset_email_with_system_html` und `test_brevo_webhook_creates_coupon_for_confirmed_list_member`, jeweils weil `"Normdex System Notification"` nicht im erzeugten Outbox-HTML/Text enthalten ist.
- Beschreibung des Problems: Die E-Mail-Template-Erwartungen und die aktuelle Ausgabe sind nicht mehr synchron. Das betrifft Passwort-Reset und Newsletter-Gutschein-Mail.
- Warum das relevant ist: Passwort-Reset und Newsletter-Gutschein sind Nutzer- und Conversion-relevante Flows. Ein roter Backend-Testlauf verhindert belastbare Releases und kann CI/CD blockieren.
- Business Impact: Release-Risiko, mögliche Inkonsistenzen in transaktionalen E-Mails, erhöhtes Risiko für Fehler im Newsletter-/Gutschein-Rollout.
- Konkrete Handlungsempfehlung: Entscheiden, ob die Tests oder die Templates die Soll-Wahrheit sind. Danach Templates oder Assertions korrigieren und den kompletten Backend-Testlauf wieder auf grün bringen. T019 sollte anschließend mit dem neuen Testergebnis aktualisiert werden.

### Finding 3 - Versionierte Dev-Datenbank braucht Fixture-Governance

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/api/dev.db`, `apps/api/org_data.json`, `apps/api/uploads/avatars/*`, `.gitignore`, `docs/development-dev-db.md`, `apps/api/scripts/verify_dev_fixture.py`
- Evidenz: `apps/api/dev.db` ist bewusst als anonymisierter Development-Snapshot versioniert, damit Laptop/PC denselben lokalen Demo-Zustand verwenden können. `git ls-files` zeigt zusätzlich `org_data.json` und mehrere Avatar-Uploads. `.gitignore` ignoriert Laufzeit-Uploads, bereits getrackte Uploads bleiben aber im Index. Das neue Prüfskript meldet außerdem nicht reservierte Demo-E-Mail-Domains und fehlende/nicht getrackte referenzierte Static-Assets.
- Beschreibung des Problems: Das Risiko ist nicht die Versionierung der Dev-Datenbank an sich, sondern fehlende Governance für den Fixture-Snapshot. Ohne klare Regeln kann die bewusst versionierte Demo-DB später versehentlich echte Kundendaten, Live-Secrets oder inkonsistente Asset-Referenzen enthalten.
- Warum das relevant ist: Eine versionierte Dev-Datenbank ist für reproduzierbare lokale Workflows nützlich, braucht aber klare Grenzen zwischen anonymisierten Fixture-Daten, lokalen Runtime-Uploads und produktiven Daten.
- Business Impact: Niedrigeres Datenschutzrisiko als ursprünglich bewertet, solange die Inhalte anonymisierte Demo-Daten bleiben. Relevante Risiken sind inkonsistente lokale Demo-Setups, versehentliche echte Daten im Snapshot und fehlende Assets auf frisch geklonten Systemen.
- Konkrete Handlungsempfehlung: `apps/api/dev.db` bewusst im Git lassen. Regeln in README/AGENTS dokumentieren, `docs/development-dev-db.md` als Leitlinie verwenden und vor DB-Snapshot-Commits `apps/api/scripts/verify_dev_fixture.py` ausführen. Benötigte Demo-Assets entweder bewusst versionieren oder DB-Referenzen auf stabile Platzhalter umstellen.

### Finding 4 - Deployment-Env passt nicht zum neuen Lizenz-Pool-Modell

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/api/app/config.py`, `apps/api/app/routers/licenses_v2.py`, `deploy/env/.env.api.prod.example`, `deploy/env/.env.api.dev.example`
- Evidenz: `config.py` erwartet `STRIPE_PRICE_ID_BASIC_MONTHLY_BASE`, `STRIPE_PRICE_ID_BASIC_YEARLY_BASE`, `STRIPE_PRICE_ID_BASIC_MONTHLY_ADDON`, `STRIPE_PRICE_ID_BASIC_YEARLY_ADDON` sowie Produkt-/Coupon-IDs. `licenses_v2.py` nutzt diese Werte in `_price_id_for_item()` und wirft sonst HTTP 500. Die Deployment-Env-Beispiele enthalten stattdessen noch `STRIPE_PRICE_ID_ECONOMICS` und `STRIPE_PRICE_ID_ECONOMICS_YEARLY`.
- Beschreibung des Problems: Wer die Deployment-Beispiele als Vorlage nutzt, konfiguriert nicht die Variablen, die der aktuelle Lizenz-v2-Code für Checkout und Pool-Erweiterung braucht.
- Warum das relevant ist: Der Vault markiert den Lizenzsystem-Rollout und Prod-Migration/Live-Stripe-Konfiguration noch als offen. Die aktuelle Beispielkonfiguration würde diesen Rollout in Staging/Prod sehr wahrscheinlich mit 500ern im Kaufprozess brechen.
- Business Impact: Checkout-Ausfall, verlorene Lizenzkäufe, Supportaufwand, Release-Risiko.
- Konkrete Handlungsempfehlung: Deployment-Env-Beispiele auf das Pool-Modell aktualisieren, alte Stripe-Variablen entfernen oder als deprecated markieren, `APP_ENV=prod` ergänzen, Live-/Dev-Price-ID-Matrix dokumentieren und vor Release einen Stripe-Dry-Run gegen Staging durchführen.

### Finding 5 - Produktions-Frontend ist per statischem BasicAuth geschützt

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `deploy/docker-compose.prod.yml`
- Evidenz: `docker-compose.prod.yml` setzt für `app.normdex.at` die Labels `traefik.http.middlewares.normdex-auth.basicauth.users=...` und `traefik.http.routers.normdex-frontend.middlewares=normdex-auth`.
- Beschreibung des Problems: Die Produktions-Compose-Datei schützt das gesamte Frontend per Traefik BasicAuth. Falls `app.normdex.at` als echte Kunden-App live gehen soll, blockiert diese Middleware Registrierung, Login und Lizenzkauf für Nutzer ohne separates BasicAuth-Passwort.
- Warum das relevant ist: BasicAuth kann für Staging sinnvoll sein, ist aber als Default in der Produktionsdatei ein hohes Risiko, wenn beim Release kein bewusster Toggle existiert.
- Business Impact: Conversion-Blocker, verlorene Registrierungen und Käufe, Supportanfragen direkt nach Release.
- Konkrete Handlungsempfehlung: BasicAuth in eine Staging-/Dev-Compose auslagern oder per dokumentierter Env/Override-Datei steuern. Für Produktion explizit festlegen, ob und wann das Gate entfernt wird.

### Finding 6 - Owner-Rolle kann sich selbst oder den letzten Owner degradieren

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/teams.py`
- Evidenz: `remove_member()` blockiert Selbstentfernung (`Cannot remove yourself`), aber `update_member_role()` setzt `target_membership.role = new_role`, ohne Self-Demotion oder Last-Owner-Demotion zu verhindern.
- Beschreibung des Problems: Ein Owner kann über den Rollen-Endpunkt sich selbst oder den letzten verbleibenden Owner auf `member` setzen. Danach hat die Organisation unter Umständen keinen Owner mehr.
- Warum das relevant ist: Owner-Rechte sind Voraussetzung für Organisationseinstellungen, Einladungen, Mitgliederverwaltung und mehrere Lizenzverwaltungsaktionen.
- Business Impact: Administrativer Lockout für Kundenorganisationen, Supportaufwand, potenziell blockierte Lizenzkäufe oder Kündigungen.
- Konkrete Handlungsempfehlung: In `update_member_role()` dieselbe Schutzlogik wie bei Admins implementieren: Self-Demotion nur erlauben, wenn mindestens ein anderer Owner bleibt; Last-Owner-Demotion immer blockieren. Regressionstest für Self-Demotion und Last-Owner ergänzen.

### Finding 7 - Datenexport und Kontolöschung sind nicht vollständig

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/users.py`, `apps/api/app/models.py`, `apps/frontend/src/pages/SettingsProfile.tsx`, `apps/frontend/src/pages/DeleteConfirm.tsx`
- Evidenz: `export_download()` exportiert nur `user` und `calculations`. `delete_confirm()` löscht `Calculation`, `Project`, `Membership`, `LicenseUsage`, `Token` und `AuditLog`, aber nicht Supporttickets/-messages, Lizenzbestellungen, Lizenz-Events, `assigned_user_id`, `created_by_user_id` oder weitere personenbezogene Referenzen. Die UI spricht von einer Kopie aller Daten und unwiderruflicher Löschung aller verbundenen Daten.
- Beschreibung des Problems: Export- und Löschversprechen sind breiter als die Backend-Implementierung. Relevante personenbezogene Daten können in Support- und Lizenzkontexten bestehen bleiben oder Fremdschlüssel-/Orphan-Probleme erzeugen.
- Warum das relevant ist: Datenschutzfunktionen sind rechtlich und vertrauensseitig sensibel. Unvollständige Löschung/Export ist ohne klare Aufbewahrungslogik schwer zu vertreten.
- Business Impact: Compliance-Risiko, Support- und Rechtsaufwand, falsche Nutzererwartung.
- Konkrete Handlungsempfehlung: Dateninventar für User-bezogene Tabellen erstellen. Für jede Tabelle definieren: löschen, anonymisieren, aufbewahren mit Rechtsgrundlage oder exportieren. UI-Texte bis dahin präzisieren und Backend-Tests für Export-/Delete-Abdeckung ergänzen.

### Finding 8 - Support-Upload hat keine robuste serverseitige Größen-/Typgrenze

- Status: **erledigt am 2026-05-22**
  - Streaming-basierter Upload mit 10-MB-Limit (Chunk-Lesen mit Abbruch + Cleanup) in `apps/api/app/routers/support.py::upload_support_file()`.
  - Positive Allow-List für Content-Types (`image/jpeg`, `image/png`, `image/bmp`, `application/pdf`) und Extensions (`.jpg`, `.jpeg`, `.png`, `.bmp`, `.pdf`).
  - Magic-Byte-Defense gegen umbenannte Executables bleibt aktiv.
  - Limit `SUPPORT_TICKET_MAX_ATTACHMENTS = 5` wird in `create_ticket()` durchgesetzt.
  - Tests in `apps/api/tests/test_support_upload.py` (7 Tests grün, kompletter Backend-Suite-Lauf: 190 passed).
- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/support.py`, `apps/frontend/src/pages/Support.tsx`
- Evidenz: Das Frontend begrenzt Support-Anhänge auf 10 MB und bestimmte MIME-Typen. Der Backend-Endpunkt `upload_support_file()` liest jedoch mit `file_bytes = await file.read()` die komplette Datei in den Speicher und validiert nur blockierte Erweiterungen/Magic Bytes, nicht eine positive Typenliste oder Maximalgröße.
- Beschreibung des Problems: Ein angemeldeter Nutzer kann die Frontend-Validierung umgehen und große oder unerwünschte Dateien direkt gegen die API hochladen.
- Warum das relevant ist: Upload-Endpunkte sind klassische Ressourcen- und Sicherheitsgrenzen. Clientseitige Checks reichen nicht aus.
- Business Impact: Speicher-/Memory-DoS, unnötige Betriebskosten, Support-Inbox mit nicht vorgesehenen Dateien.
- Konkrete Handlungsempfehlung: Serverseitig Maximalgröße, maximale Anzahl, erlaubte Content-Types/Extensions und Streaming-/Chunk-Limit durchsetzen. Upload-Tests für Oversize, verbotene Typen und erlaubte PDF/Bilddateien ergänzen.

### Finding 9 - Frontend-Routing und Public-Chrome sind inkonsistent

- Status: **erledigt am 2026-05-22**
  - Chrome-Erkennung in `apps/frontend/src/App.tsx` umgestellt: statt einer Blacklist (`hideSidebarRoutes`) wird über eine Whitelist von App-Pfad-Präfixen (`APP_PATH_PREFIXES` + `isAppPath()`) entschieden, ob Sidebar/Header gezeigt werden.
  - `/auth/verify`, `/impressum` und `/datenschutz` rendern jetzt explizit ohne App-Chrome (eigener Block in den Routes).
  - Catch-all 404-Route mit neuer Seite `apps/frontend/src/pages/NotFound.tsx` ergänzt (ohne App-Chrome).
  - Unbenutzter `Landing`-Import aus `App.tsx` entfernt; `/` bleibt bewusst Redirect auf `/auth/login` (Landingpage ist eigenes Repo).
  - Verifikation: `npm run build` und `npm test` in `apps/frontend` grün (12 Tests passed).
- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/App.tsx`, `apps/frontend/public/robots.txt`, `apps/frontend/public/sitemap.xml`
- Evidenz: `Landing` wird importiert, aber nicht geroutet. `/` redirectet auf `/auth/login`. `hideSidebarRoutes` enthält nur einige Auth-Routen; `/auth/verify`, `/impressum` und `/datenschutz` sind öffentlich, würden aber nicht vom Sidebar/Header-Chrome ausgenommen. Eine Catch-all-404-Route fehlt.
- Beschreibung des Problems: Öffentliche Seiten und App-Chrome sind nicht klar getrennt. Das kann unauthentifizierten Nutzern rechtliche Seiten oder Verifizierungsseiten im App-Layout anzeigen.
- Warum das relevant ist: Login, Registrierung, E-Mail-Verifizierung und Rechtstexte sind Eintritts- und Vertrauenstouchpoints.
- Business Impact: Unsaubere UX, potenzielle Conversion-Reibung, höheres Risiko für fehlerhafte öffentliche Navigation.
- Konkrete Handlungsempfehlung: Public-/Auth-/App-Layouts explizit trennen, `/` bewusst auf Landing oder Login routen, `/auth/verify`, `/impressum`, `/datenschutz` in das Public/Auth-Layout legen und eine 404-Route ergänzen.

### Finding 10 - Frontend-Hauptbundle ist groß und nicht gesplittet

- Status: **erledigt am 2026-05-22**
  - Route-Level Lazy Loading via `React.lazy` + `Suspense` in `apps/frontend/src/App.tsx` eingeführt.
  - Eager bleiben nur Auth-Pfad (Login, Register, ForgotPassword, ResetPassword, VerifyEmail) sowie Layout-Chrome (Header, Sidebar) und Route-Guards (ProtectedRoute, AdminRoute).
  - Lazy gesplittet: AppHome, Projects/ProjectDetail/NewProject, Team, Licenses, WhatsNew, Help, Support, EconomicsForm, EconomicsReport, Settings (Profile/Organization), SubscriptionPlans, Admin (Users/Support/OrganizationCase), DeleteConfirm, Goodbye, Impressum, Datenschutz, NotFound.
  - Initial-JS: **1.409 kB → 502 kB** (-64 %), gzip **394 kB → 154 kB** (-61 %).
  - Recharts/html2pdf landen automatisch in einem gemeinsamen 333-kB-Chunk (`ReportTypes-*.js`), der nur beim Öffnen der Economics-Routen geladen wird.
  - Frontend-Tests grün (12/12).
- Kategorie: Optimization
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/App.tsx`, `apps/frontend/vite.config.ts`, `apps/frontend/package.json`
- Evidenz: `npm run build` erzeugt `assets/index-*.js` mit `1,405.17 kB` minified und `392.95 kB` gzip. Vite warnt: `Some chunks are larger than 500 kB after minification`.
- Beschreibung des Problems: Alle großen App-Bereiche liegen im initialen Hauptbundle. Das betrifft Admin, Support, Economics, PDF/Charts und Lizenzverwaltung gemeinsam.
- Warum das relevant ist: Login- und Dashboard-Start laden Code, der für den initialen Nutzerpfad nicht sofort nötig ist.
- Business Impact: Langsamere First Load Experience, schlechtere UX bei mobilen/langsameren Verbindungen, unnötige Kosten bei häufigen App-Starts.
- Konkrete Handlungsempfehlung: Route-level Lazy Loading mit `React.lazy`/`Suspense` einführen, schwere Bereiche wie Economics, Report/PDF, Admin/Support und Lizenzverwaltung splitten und nach dem Build Chunkgrößen erneut prüfen.

## 8. Quick Wins

Alle Quick Wins **erledigt am 2026-05-22**.

- ✅ `tremor` aus `apps/frontend/package.json` entfernen — bereits weder in `package.json` noch in `apps/frontend/src/` referenziert.
- ✅ Zwei fehlgeschlagene Backend-Tests fixen — `.\venv\Scripts\python -m pytest` meldet aktuell **190 passed** (kein Failure).
- ✅ `apps/api/dev.db` als anonymisierten Dev-Snapshot dokumentieren — Regeln und Prüf-Workflow in `apps/api/docs/development-dev-db.md`; `apps/api/scripts/verify_dev_fixture.py` läuft sauber und meldet nur informelle Domain-Review-Hinweise.
- ✅ Deployment-Env-Beispiele mit den aktuellen `STRIPE_PRICE_ID_BASIC_*`-, Product- und Coupon-Variablen aktualisieren — in `deploy/env/.env.api.dev.example` und `deploy/env/.env.api.prod.example` enthalten (Test- bzw. Live-Mode).
- ✅ Last-Owner-Schutz in `teams.py` ergänzen (Finding 6, Commit `8a4da8f`).
- ✅ Server-Limits für `/support/upload` hinzufügen (Finding 8, Commit `c67dc8c`).
- ✅ Public/Auth/App-Layout-Trennung in `App.tsx` bereinigen (Finding 9, Commit `d1d17c9`).

## 9. Strategische Empfehlungen

- Vor dem nächsten Release eine kleine Release-Gate-Checkliste etablieren: `npm audit --omit=dev`, Frontend-Build, Frontend-Tests, Backend-Tests, `pip check`, Fixture-Prüfung der versionierten Dev-Datenbank.
- Lizenz-/Stripe-Rollout als eigenes Release-Paket abschließen: Env-Matrix, Staging-Dry-Run, Webhook-Test, Live-Price-ID-Dokumentation, Rollback-Plan und Vault-Status aktualisieren.
- Datenschutzfunktionen als eigenes Arbeitspaket behandeln: Dateninventar, Export-/Delete-Matrix, Tests und juristisch geprüfte UI-Texte.
- Deployment-Konfigurationen klar in `dev-server` und `prod` trennen, damit Schutzmechanismen wie BasicAuth nicht versehentlich auf dem Live-Pfad bleiben.
- Frontend-Bundle-Splitting nicht als kosmetische Optimierung behandeln, sondern entlang echter Produktbereiche einführen, weil Economics/PDF/Admin/Support natürliche Lazy-Load-Grenzen sind.

## 10. Empfohlene nächste Aktion

Sofort beheben: zuerst kritische npm-Vulnerabilities und roten Backend-Testlauf bereinigen sowie Fixture-Governance für die bewusst versionierte Dev-Datenbank dokumentieren; erst danach den Lizenz-/Stripe-Deployment-Stand für Staging/Produktion weiterziehen.

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Ob `app.normdex.at` aktuell bewusst hinter BasicAuth bleiben soll oder bereits als Kunden-App gedacht ist.
- Bestätigt: `dev.db` enthält bewusst anonymisierte Demo-/Development-Daten und darf versioniert bleiben; offen bleibt nur die dauerhafte Fixture-Prüfung vor Snapshot-Commits.
- Ob die fehlenden `"Normdex System Notification"`-Strings in den E-Mail-Tests eine bewusst geänderte Template-Entscheidung oder eine echte Regression sind.
- Ob die produktive `.env.api.prod` bereits von den veralteten Beispielvariablen abweicht.
- Ob `APP_ENV=prod` auf dem Produktivserver gesetzt ist; die Beispiel-Env enthält es nicht.
- Ob aktuelle Stripe-Live-Mode Products/Prices bereits angelegt und außerhalb des Repos dokumentiert sind.
- Ob Datenexport/Kontolöschung juristisch bereits bewusst auf bestimmte Tabellen beschränkt wurden.
- Ob Support-Anhänge produktiv zusätzlich durch Proxy/Webserver-Größenlimits begrenzt werden.
- Ob ein Browser-Smoke-Test der öffentlichen Seiten und geschützten App-Routen auf Staging bereits existiert.
