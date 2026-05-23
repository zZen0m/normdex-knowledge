# Repo-Audit - Webapp - 2026-05-23

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-app`
- Branch: `develop`
- Commit: `6098b53`
- Version laut `package.json`: `0.2.0`
- Letzter Git-Tag: `v0.2.0`
- Audit-Datum: `2026-05-23`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-05-23-repo-audit-webapp.md`
- Vorheriger Audit: `2026-05-22-repo-audit-webapp.md`

## 2. Audit-Abdeckung

- Statische Codeanalyse von `apps/frontend/`, `apps/api/`, `deploy/`, Alembic-Migrationen, zentralen Konfigurationsdateien und öffentlich erreichbaren API-Flows.
- Backend/API-Prüfung mit Fokus auf Auth, Rollen/Rechte, Organisations- und Lizenzflows, Newsletter/Brevo, Support, Datenschutzfunktionen, statische Dateien und Deployment-Sicherheit.
- Frontend/Routen-/Komponentenprüfung mit Fokus auf Routing, Lazy Loading, API-Client, Registrierung, Datenschutzseiten, Support-UI, Lizenzverwaltung und Build-Konfiguration.
- Datenbank-/Migrationsprüfung statisch über `apps/api/app/models.py`, `apps/api/alembic/versions/`, `apps/api/alembic/env.py` und die versionierte Dev-Fixture-Prüfung. Keine Alembic-Migration wurde ausgeführt.
- Metadaten-/SEO-/Routing-Prüfung über `index.html`, `PageMeta`, `robots.txt`, `sitemap.xml` und `App.tsx`.
- Abgleich mit Vorbericht vom 2026-05-22.
- Vault-Abgleich mit `00_Start/AI Kontext - Einstieg.md`, `10_Aufgaben/Aufgaben.md`, `T013-lizenzsystem-rollout-abschliessen.md`, `T017-testzeitraum-fuer-lizenzen.md`, `T019-newsletter-gutschein-brevo-webhook-rollout.md` und `T020-allgemeine Todos.md`.
- Verifikation ausgeführt:
  - `npm run build` in `apps/frontend`: erfolgreich, mit Vite-Chunk-Warnung bei `assets/index-*.js` mit `502.11 kB`.
  - `npm test` in `apps/frontend`: 2 Testdateien, 12 Tests grün.
  - `npm audit --omit=dev` in `apps/frontend`: 0 Vulnerabilities.
  - `.\venv\Scripts\python -m pytest` in `apps/api`: 190 Tests grün.
  - `.\venv\Scripts\python -m pip check` in `apps/api`: keine kaputten Python-Abhängigkeiten.
  - `.\venv\Scripts\python scripts\verify_dev_fixture.py` in `apps/api`: Exit 0, aber Fixture-Warnungen zu nicht reservierten E-Mail-Domains sowie fehlenden/nicht getrackten Static-Assets.
- Keine verlässliche Browser-Laufzeitprüfung, kein Staging-/Produktiv-Smoke-Test, kein echter Stripe-/Brevo-/Graph-End-to-End-Test und keine Prüfung gegen Produktivdatenbank durchgeführt.

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-05-22
- Findings im Vorbericht: 10
- Davon behoben: 8
- Davon weiterhin offen: 2
- Davon nicht abschließend verifizierbar: 0
- Regressionen: 0

### Behobene Findings

- Finding 1 - Kritische npm-Vulnerabilities in Production-Abhängigkeiten: behoben (Evidenz: `npm audit --omit=dev` meldet `found 0 vulnerabilities`; `tremor`/`@tremor` sind nicht mehr in `package.json`, `package-lock.json` oder `src/` referenziert).
- Finding 2 - Backend-Testlauf ist aktuell rot: behoben (Evidenz: `.\venv\Scripts\python -m pytest` meldet `190 passed`).
- Finding 4 - Deployment-Env passt nicht zum neuen Lizenz-Pool-Modell: behoben (Evidenz: `deploy/env/.env.api.prod.example`, `deploy/env/.env.api.dev.example` und `deploy/README.md` enthalten die vier `STRIPE_PRICE_ID_BASIC_*`-Variablen, Product-IDs und `APP_ENV`).
- Finding 6 - Owner-Rolle kann sich selbst oder den letzten Owner degradieren: behoben (Evidenz: `teams.py` blockiert die letzte Owner-Degradierung; `tests/test_team_roles.py` deckt Last-Owner- und Self-Demotion ab).
- Finding 7 - Datenexport und Kontolöschung sind nicht vollständig: behoben (Evidenz: Self-Service-Export-Endpunkte geben HTTP 410 zurück, die Konto-Löschung anonymisiert Support-/Lizenz-/Newsletter-Referenzen; `tests/test_user_privacy.py` deckt Export-Deaktivierung und Anonymisierung ab).
- Finding 8 - Support-Upload hat keine robuste serverseitige Größen-/Typgrenze: behoben (Evidenz: `SUPPORT_UPLOAD_MAX_BYTES`, positive Content-Type-/Extension-Allowlist, Magic-Byte-Prüfung und Attachment-Limit in `support.py`; `tests/test_support_upload.py` ist grün).
- Finding 9 - Frontend-Routing und Public-Chrome sind inkonsistent: behoben (Evidenz: `APP_PATH_PREFIXES`, explizite Public/Auth/App-Routen, `NotFound`-Route und Lazy-Route-Struktur in `App.tsx`).
- Finding 10 - Frontend-Hauptbundle ist groß und nicht gesplittet: behoben im Kern (Evidenz: Route-Level `React.lazy`/`Suspense` in `App.tsx`; Build erzeugt getrennte Chunks für AppHome, Licenses, Economics, Admin und Support).

### Weiterhin offene Findings

- Finding 3 - Versionierte Dev-Datenbank braucht Fixture-Governance: weiterhin offen. Das Prüfskript existiert und läuft, meldet aber aktuell nicht reservierte E-Mail-Domains, ein fehlendes Static-Asset und ein vorhandenes, aber nicht getracktes Static-Asset.
- Finding 5 - Produktions-Frontend ist per statischem BasicAuth geschützt: weiterhin offen, aber inzwischen dokumentiert und steuerbar. `deploy/prod-compose.sh` und `docker-compose.prod.public.yml` können BasicAuth entfernen; der Default in `docker-compose.prod.yml` und `.env.deploy.prod.example` bleibt jedoch geschlossen.

### Regressionen

- Keine Regressionen gegenüber dem Vorbericht festgestellt.

## 4. Confidence

- Confidence: hoch

Die wichtigsten Aussagen sind direkt durch Repository-Code und aktuelle Toolausgaben belegbar. Backend-, Frontend- und Dependency-Gates wurden ausgeführt und sind überwiegend grün. Eingeschränkt bleibt die Bewertung bei Produktivzustand, Stripe-Live-Konfiguration, Brevo-Webhook-Verhalten, Microsoft-Graph-Mailversand, Browser-UX und tatsächlichem Server-Deployment, weil diese Punkte nicht live geprüft wurden.

## 5. Kurzfazit

Der aktuelle Stand ist gegenüber dem Audit vom 2026-05-22 klar verbessert: npm-Audit, Frontend-Tests, Backend-Tests und Python-Dependency-Check sind grün. Mehrere frühere Release-Blocker wurden gezielt abgearbeitet, darunter kritische Frontend-Abhängigkeiten, rote Backend-Tests, Datenschutzlöschung, Support-Upload-Limits und Routing-/Bundle-Struktur. Die verbleibenden Risiken sind weniger akute Testfehler als Release- und Betriebsränder: öffentliche Support-/Newsletter-Flows, Dev-Fixture-Hygiene, BasicAuth-Launch-Schalter und fehlende Staging-/Live-Verifikation für Lizenz-/Stripe-/Brevo-Flows. Besonders relevant ist, dass der öffentliche Support-Ticket-Flow nutzergelieferte Inhalte ungeescaped in branded HTML-E-Mails einbettet. Vor einem öffentlichen Launch sollte dieses E-Mail-/Support-Thema priorisiert werden, danach die Deployment-Schalter und Staging-Dry-Runs.

## 6. Wichtigste Findings

1. Öffentliche Support-Autoresponder betten nutzergeliefertes HTML ungeescaped in Normdex-E-Mails ein.
2. Brevo-Webhook-Secret wird als Query-Parameter übertragen und ist damit log-/proxy-sensibel.
3. Versionierte Dev-Fixture enthält weiterhin Prüf-Warnungen zu real wirkenden E-Mail-Domains und Static-Asset-Referenzen.
4. Der Produktionszugang bleibt im Default per BasicAuth geschlossen und hängt für den Kundenlaunch an einem korrekt genutzten Deploy-Wrapper.
5. Kritische Frontend-Flows sind kaum durch Komponententests abgedeckt.
6. Backend erzeugt 616 Test-Warnungen durch Pydantic-/Datetime-Deprecations und mehrere Pydantic-V3-Migrationsstellen.
7. Frontend-Build bleibt knapp über der Vite-Chunk-Warnschwelle.
8. Alembic-Versionenordner enthält große Codeium-/Unleash-JSON-Artefakte, die keine Migrationen sind.

## 7. Detaillierte Findings je Punkt

### Finding 1 - Support-Autoresponder übernimmt ungeescapedes Nutzer-HTML

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/support.py`, `apps/api/app/emails.py`, `apps/frontend/src/pages/admin/SupportTicketDetail.tsx`
- Evidenz: `create_public_ticket()` ist öffentlich erreichbar und speichert `payload.message` als `body_html=f"<p>{payload.message}</p>"`. `send_auto_reply()` ruft `tpl_support_ticket_created_html(..., content)` auf. `render_support_email_html()` setzt `s_subject` und `s_content` ohne Escaping direkt in die HTML-Mail-Zusammenfassung ein. Die Admin-UI sanitizt beim Rendern mit `DOMPurify.sanitize(...)`, der Mail-Renderer tut das nicht.
- Beschreibung des Problems: Ein externer Nutzer kann über den öffentlichen Support-Endpunkt HTML in eine Normdex-branded Autoresponder-Mail bringen. Da `payload.email` frei angegeben wird, kann dieser Flow als begrenzter Mail-Relay-/Phishing-Vektor missbraucht werden, auch wenn der Endpoint rate-limited ist.
- Warum das relevant ist: Support- und Kontaktformulare sind öffentlich erreichbar und senden E-Mails im Namen von Normdex. Unescaped HTML in E-Mail-Templates ist ein Reputation-, Security- und Trust-Risiko.
- Business Impact: Risiko für Mail-Sender-Reputation, Phishing-Missbrauch mit Normdex-Branding, Supportaufwand und potenziell schlechtere Zustellbarkeit transaktionaler E-Mails.
- Konkrete Handlungsempfehlung: User-Content im Support-Mail-Renderer standardmäßig mit `html.escape()` behandeln. Nur bewusst erlaubtes Agent-HTML über eine serverseitige Allowlist/Sanitizer-Pipeline zulassen. Zusätzlich Tests ergänzen, die `<a>`, `<img>`, `<script>` und einfache HTML-Tags in Supportnachrichten als escaped in Autoresponder-Mails erwarten.

### Finding 2 - Brevo-Webhook-Secret steckt in der URL

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/newsletter.py`, `apps/api/tests/test_newsletter.py`, `D:\Normdex\02_knowledge\normdex-vault\10_Aufgaben\offene Todos\T019-newsletter-gutschein-brevo-webhook-rollout.md`
- Evidenz: `brevo_webhook()` verlangt `secret: str = Query(...)` und vergleicht gegen `settings.BREVO_WEBHOOK_SECRET`. Tests rufen `/newsletter/brevo/webhook?secret=secret` auf. T019 dokumentiert den Endpoint ebenfalls als `POST /newsletter/brevo/webhook?secret=...`.
- Beschreibung des Problems: Secrets in Query-Parametern landen typischerweise in Reverse-Proxy-Logs, Access-Logs, Monitoring-Tools und Fehlermeldungen leichter als Header. Bei Leak kann ein Angreifer Brevo-Events simulieren und Coupon-/Mail-Logik anstoßen, solange weitere Payload-Bedingungen passen.
- Warum das relevant ist: Der Newsletter-Gutschein-Flow erzeugt individuelle Stripe Promotion Codes und sendet E-Mails. Die Absicherung sollte deshalb nicht auf einem URL-Secret beruhen, das in Logs sichtbar werden kann.
- Business Impact: Missbrauch von Gutscheincodes, unnötige Coupon-Erzeugung, Support- und Monitoring-Aufwand, potenzieller Vertrauensverlust beim Newsletter-Rollout.
- Konkrete Handlungsempfehlung: Wenn Brevo Custom Headers unterstützt, Secret in einen Header verschieben. Falls nicht, einen schwer erratbaren Webhook-Pfad plus zusätzliche Payload-Prüfungen verwenden und Proxy-/App-Logs so konfigurieren, dass Query-Strings für diesen Pfad nicht gespeichert werden. T019 und Tests entsprechend anpassen.

### Finding 3 - Dev-Fixture-Prüfung meldet reale Daten- und Asset-Risiken

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-22)
- Betroffene Datei(en) oder Pfade: `apps/api/dev.db`, `apps/api/scripts/verify_dev_fixture.py`, `apps/api/uploads/`
- Evidenz: `.\venv\Scripts\python scripts\verify_dev_fixture.py` meldet nicht reservierte E-Mail-Domains wie `gmail.com`, `gmx.at`, `normdex.at` und mehrere private Domains in `users.email`, `outbox_emails.to`, `support_tickets.requester_email`, `support_ticket_messages.*`, `invites.email` und `audit_logs.meta`. Zusätzlich meldet das Skript `/static/logos/...` als fehlend und `/static/avatars/10_1777303968.png` als vorhanden, aber nicht getrackt.
- Beschreibung des Problems: Die Dev-Datenbank ist bewusst versioniert, aber der aktuelle Snapshot ist nicht vollständig fixture-sauber. Die Warnungen beweisen nicht automatisch echte personenbezogene Daten, sie zeigen aber, dass die Anonymisierung/Fixture-Regeln aktuell manuelle Prüfung brauchen.
- Warum das relevant ist: Eine versionierte Dev-Fixture ist nur dann sicher und reproduzierbar, wenn Daten und referenzierte Assets eindeutig Demo-Charakter haben und alle referenzierten Assets auf frischen Klonen verfügbar sind.
- Business Impact: Datenschutz- und Reputationsrisiko bei versehentlichen Echtdaten, inkonsistente lokale Demo-Setups, zusätzliche Onboarding- und Debugzeit.
- Konkrete Handlungsempfehlung: Vor dem nächsten Commit von `apps/api/dev.db` die Warnungen bereinigen oder bewusst dokumentiert akzeptieren. Demo-E-Mails auf reservierte Domains (`example.com`, `normdex.local`) migrieren, fehlende Static-Referenzen entfernen oder Assets bewusst versionieren, und bei Snapshot-Commits `verify_dev_fixture.py --strict` als Gate verwenden.

### Finding 4 - BasicAuth-Gate bleibt ein Release-Schalter mit Default geschlossen

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-22)
- Betroffene Datei(en) oder Pfade: `deploy/docker-compose.prod.yml`, `deploy/docker-compose.prod.public.yml`, `deploy/prod-compose.sh`, `deploy/env/.env.deploy.prod.example`, `deploy/README.md`
- Evidenz: `docker-compose.prod.yml` setzt weiterhin `traefik.http.middlewares.normdex-auth.basicauth.users` und `traefik.http.routers.normdex-frontend.middlewares=normdex-auth`. `deploy/prod-compose.sh` entfernt dieses Gate nur bei `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false`; `.env.deploy.prod.example` steht auf `true`.
- Beschreibung des Problems: Die Absicherung für die geschlossene Live-Testphase ist inzwischen dokumentiert und steuerbar. Der Default bleibt aber geschlossen, und ein direkter `docker compose -f deploy/docker-compose.prod.yml ...` ignoriert den Public-Override.
- Warum das relevant ist: Für einen öffentlichen Kundenlaunch blockiert ein versehentlich aktiviertes BasicAuth-Gate Registrierung, Login und Checkout für echte Nutzer.
- Business Impact: Conversion-Blocker, verlorene Registrierungen/Käufe und Supportanfragen direkt nach Launch.
- Konkrete Handlungsempfehlung: Für den Launch eine explizite Deploy-Checkliste mit `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false`, `./deploy/prod-compose.sh config` und Smoke-Test gegen `https://app.normdex.at/auth/register` pflegen. Optional: im Produktiv-Runbook raw `docker compose -f deploy/docker-compose.prod.yml` als Anti-Pattern markieren.

### Finding 5 - Frontend-Testabdeckung schützt zentrale Flows kaum

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/pages/Licenses.tsx`, `apps/frontend/src/pages/Register.tsx`, `apps/frontend/src/pages/Support.tsx`, `apps/frontend/src/pages/admin/SupportTicketDetail.tsx`, `apps/frontend/src/App.tsx`, `D:\Normdex\02_knowledge\normdex-vault\10_Aufgaben\offene Todos\T013-lizenzsystem-rollout-abschliessen.md`
- Evidenz: `npm test` führt nur `src/hooks/useLicenseLock.test.ts` und `src/lib/utils.test.ts` aus, insgesamt 12 Tests. T013 listet UI-Komponententests für `Licenses.tsx` und zentrale Lizenzstatus ausdrücklich als offen.
- Beschreibung des Problems: Backend-Tests sind breit, aber die kundenkritischen Frontend-Flows für Registrierung, Lizenzkauf/-kündigung, Gutscheinfeld, Trial-Zustände, Support und Admin-Support sind nicht automatisiert gegen Rendering-, Copy- und Zustandsregressionen geschützt.
- Warum das relevant ist: Die aktuellen offenen Roadmap-Punkte liegen stark in Lizenz-, Trial-, Newsletter- und Support-Flows. Dort entstehen viele Fehler erst durch UI-Zustände, nicht durch isolierte Backend-Logik.
- Business Impact: Höheres Regressionrisiko bei Checkout und Lizenzverwaltung, mehr manuelle Abnahme vor Releases, potenziell verlorene Käufe durch UI-Fehler.
- Konkrete Handlungsempfehlung: Als nächstes kleine, zielgerichtete React-Testing-Library-Tests für `Licenses.tsx` ergänzen: `active`, `trial`, `scheduled_end`, `ended`, `payment_failed`, Checkout-Cancel, Rücknahme-Button und Promotion-Code. Danach Smoke-Tests für Register und Support.

### Finding 6 - Backend-Warnungen zeigen Pydantic-/Datetime-Migrationsschuld

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/*.py`, `apps/api/app/security.py`, `apps/api/app/models.py`, `apps/api/app/services/*.py`, `apps/api/tests/*.py`
- Evidenz: Der Backend-Testlauf ist grün, meldet aber 616 Warnungen. Darunter sind `PydanticDeprecatedSince20` für `class Config`, zahlreiche `datetime.datetime.utcnow()`-/`utcfromtimestamp()`-Deprecations und `UnsupportedFieldAttributeWarning` für mehrere `Field(alias=...)`-Verwendungen in `projects.py`.
- Beschreibung des Problems: Die Anwendung läuft aktuell auf Python 3.13 und Pydantic v2, nutzt aber weiterhin Patterns, die in künftigen Versionen weiter brechen oder schwerer wartbar werden können.
- Warum das relevant ist: Warnungen in dieser Menge maskieren neue relevante Warnungen und erhöhen das Risiko bei Pydantic-v3-/Python-Upgrades. Datetime-naive UTC-Werte sind außerdem eine wiederkehrende Quelle für Anzeige- und Vergleichsfehler.
- Business Impact: Erhöhte Wartungskosten, Upgrade-Risiko und potenziell falsche Zeitdarstellung in Support-, Lizenz- und Audit-Flows.
- Konkrete Handlungsempfehlung: Ein eigenes Tech-Debt-Ticket anlegen: Pydantic-Modelle auf `ConfigDict(from_attributes=True)` migrieren, `datetime.utcnow()` schrittweise durch timezone-aware UTC-Helfer ersetzen, Projekt-Schema-Warnungen gezielt beheben und danach `pytest -W error::DeprecationWarning` für ausgewählte Module prüfen.

### Finding 7 - Frontend-Build bleibt knapp über Chunk-Warnschwelle

- Kategorie: Optimization
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/App.tsx`, `apps/frontend/vite.config.ts`, `apps/frontend/dist/`
- Evidenz: `npm run build` ist erfolgreich, Vite meldet aber weiterhin `Some chunks are larger than 500 kB after minification`. Der größte initiale Chunk ist `assets/index-*.js` mit `502.11 kB` und `154.14 kB` gzip; `ReportTypes-*.js` liegt bei `332.94 kB`.
- Beschreibung des Problems: Route-Level Lazy Loading hat die Hauptlast deutlich reduziert, aber die Warnschwelle wird knapp überschritten. Der Hauptchunk enthält weiterhin genug gemeinsame App-/Layout-/UI-Basis, um den Vite-Hinweis auszulösen.
- Warum das relevant ist: Kein akuter Release-Blocker, aber ein Signal, dass Common-Code und Vendor-Chunks noch nicht sauber budgetiert sind.
- Business Impact: Potenziell langsamere First Load Experience auf schwächeren Geräten oder mobilen Verbindungen, aber aktuell deutlich niedrigeres Risiko als im Vorbericht.
- Konkrete Handlungsempfehlung: Bundle-Budget bewusst festlegen. Entweder `manualChunks` für große Vendor-Gruppen prüfen oder die Warnschwelle dokumentiert auf den aktuellen Zielwert anheben, wenn 154 kB gzip für den initialen App-Start akzeptiert wird.

### Finding 8 - Alembic-Versionenordner enthält Nicht-Migrations-Artefakte

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/alembic/versions/unleash-backup-codeium-extension.json`, `apps/api/alembic/versions/unleash-repo-schema-v1-codeium-language-server.json`
- Evidenz: Im Alembic-Versionenordner liegen zwei JSON-Dateien mit 325.167 bzw. 414.066 Bytes. Der Ordner enthält ansonsten Python-Migrationsdateien mit `revision`/`down_revision`.
- Beschreibung des Problems: Die Dateien sehen nach Codeium-/Unleash-Artefakten aus und gehören nicht in den Alembic-Migrationspfad. Alembic selbst lädt Python-Migrationen, aber der Ordner ist ein kritischer Betriebsbereich und sollte nicht mit fremden Tool-Artefakten vermischt werden.
- Warum das relevant ist: Migrationen sind Release-kritisch. Nicht-Migrationsdateien in diesem Ordner erschweren Reviews, Audits und potenziell eigene Migrations-Checks.
- Business Impact: Niedriges unmittelbares Risiko, aber vermeidbare Wartungs- und Review-Reibung vor produktiven Datenbankänderungen.
- Konkrete Handlungsempfehlung: Dateien prüfen und bei fehlender fachlicher Relevanz aus `apps/api/alembic/versions/` entfernen. Falls sie lokal gebraucht werden, in einen Tool-/Cache-Pfad verschieben und per `.gitignore` absichern.

## 8. Quick Wins

- In `apps/api/app/emails.py` Support-Mail-Zusammenfassungen mit `html.escape()` für `subject`, `category` und Nutzer-`content` absichern.
- Test ergänzen, der HTML in `payload.message` über `/support/public-tickets` als escaped im Autoresponder erwartet.
- `verify_dev_fixture.py --strict` ausführen und die aktuell gemeldeten E-Mail-Domains/Static-Assets bereinigen oder als bewusst akzeptierte Demo-Daten dokumentieren.
- Vor Public Launch `deploy/env/.env.deploy.prod` auf `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false` setzen und `./deploy/prod-compose.sh config` prüfen.
- Brevo-Webhook-Secret aus der Query entfernen oder Query-Logging für den Webhook-Pfad explizit unterbinden.
- Die zwei `unleash-*.json`-Artefakte im Alembic-Versionenordner prüfen und entfernen/verschieben.

## 9. Strategische Empfehlungen

- Release-Gate fest etablieren: `npm audit --omit=dev`, `npm run build`, `npm test`, Backend-`pytest`, `pip check`, `verify_dev_fixture.py --strict`, danach Staging-Smoke-Test.
- Lizenz-/Stripe-/Brevo-Rollout erst nach dokumentiertem Staging-Dry-Run schließen: echter Stripe-Testmode-Checkout, gültige Webhook-Signatur, Brevo-Double-Opt-in, individueller Promotion-Code und Checkout-Einlösung.
- Frontend-Tests entlang der Business-Flows priorisieren, nicht entlang technischer Utilities: Lizenzstatus, Kaufdialog, Trial-Konvertierung, Gutschein, Registrierung und Support.
- Öffentliche Kommunikations- und Support-Flows als Abuse-Surface behandeln: Captcha/Rate-Limits, E-Mail-Escaping, sender reputation, Monitoring und klare Logs.
- Backend-Deprecation-Warnungen schrittweise als Upgrade-Readiness-Projekt abbauen, bevor Pydantic-/Python-Upgrades erzwungen werden.

## 10. Empfohlene nächste Aktion

Sofort beheben: den öffentlichen Support-Autoresponder serverseitig escapen/sanitisieren und mit Regressionstest absichern; anschließend Dev-Fixture-Warnungen und BasicAuth-Launch-Schalter vor dem nächsten Staging-/Public-Release prüfen.

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Ob `app.normdex.at` aktuell bewusst in der geschlossenen Live-Testphase bleiben soll oder schon öffentlich erreichbar sein muss.
- Ob der Produktivserver konsequent `deploy/prod-compose.sh` nutzt oder teilweise raw `docker compose -f deploy/docker-compose.prod.yml`.
- Ob Brevo Custom Headers oder signierte Webhook-Payloads für den aktuellen Tarif/Setup verfügbar sind.
- Ob die im Dev-Fixture-Snapshot gefundenen Domains vollständig Demo-Daten sind oder aus echten Test-/Kundenkontakten stammen.
- Ob die fehlende Logo-Datei in der Dev-Fixture bewusst toleriert wird oder lokale Demo-Berichte tatsächlich kaputt rendert.
- Ob Stripe-Testmode und Live-Mode Products/Prices bereits außerhalb des Repos vollständig angelegt und dokumentiert sind.
- Ob ein aktueller Staging-Smoke-Test für Registrierung, Login, Lizenzkauf, Trial, Gutschein, Supportticket und Admin-Support existiert.
- Ob die Backend-Deprecation-Warnungen im nächsten geplanten Python-/Pydantic-Upgrade-Fenster berücksichtigt sind.
