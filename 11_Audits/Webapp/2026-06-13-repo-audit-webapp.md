# Repo-Audit - Webapp - 2026-06-13

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-webapp-dev` (GitHub-Remote: `zZen0m/Normdex`)
- Branch: `dev-server`
- Commit: `3651038`
- Version laut `package.json`: `0.1.0`
- Letzter Git-Tag: `v0.2.0`
- Audit-Datum: `2026-06-13`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-06-13-repo-audit-webapp.md`
- Vorheriger Audit: `2026-06-07-repo-audit-webapp.md`
- Git-Status zum Audit-Zeitpunkt: Arbeitsbaum sauber; lokaler Branch liegt 4 Commits vor `origin/dev-server`
- Nachtrag vom 2026-06-13: Finding 1 wurde repo-seitig teilweise behoben und erneut verifiziert; die Audit-Baseline aus Commit `3651038` bleibt unverändert.
- Nachtrag vom 2026-06-13: Finding 2 wurde behoben und mit neuen Backend-Tests sowie Frontend-Typecheck verifiziert.
- Nachtrag vom 2026-06-13: Finding 3 wurde behoben; Support-Anhänge sind nicht mehr statisch öffentlich erreichbar und werden ausschließlich über einen admin-geschützten Download-Endpunkt ausgeliefert.
- Nachtrag vom 2026-06-13: Finding 5 wurde behoben; die veralteten Undo-Kauf-Tests wurden an den aktuellen Lizenzvertrag angepasst und die vollständige Backend-Suite läuft ohne Ausschlüsse.
- Nachtrag vom 2026-06-13: Finding 6 wurde behoben; Backend- und Frontend-Abhängigkeiten wurden auf scanner-saubere Versionen aktualisiert und in frischen Docker-Builds verifiziert.

## 2. Audit-Abdeckung

- Normdex-Vault-Kontext gelesen: Architektur, Authentifizierung und Sicherheit, Datenschutzprozess, API-Endpunkte, Integrationen sowie Deployment-Workflow
- Statische Codeanalyse von FastAPI-Routen, Authentifizierung, Rollen/Rechten, Mandantentrennung, Support, Admin- und Stripe-Flows
- Frontend-Prüfung von Registrierung, Admin-Support, geschützten Routen, API-Nutzung und sicherheitsrelevanten DOM-Ausgaben
- Datenbank- und Migrationsprüfung: 40 Alembic-Migrationen, genau ein Head (`ffd3bbde6b6a`), keine fehlenden Vorgänger
- Git-Historie und aktuelle Remote-Refs auf versehentlich eingecheckte Secrets geprüft, ohne Secret-Werte auszugeben
- Nachbearbeitung zu Finding 1: vollständiger Gitleaks-Lauf mit Version `8.30.1` über 417 erreichbare Commits, zusätzliche Pfadprüfung für `.env*`-Dateien und Datenbank-Backups sowie Negativtests für beide Dateiklassen
- Nachbearbeitung zu Finding 1: lokale Legacy-Refs neu geschrieben, Reflogs und alte Objekte gepruned; danach keine unerreichbaren Git-Objekte mehr vorhanden
- Nachbearbeitung zu Finding 2: 4 neue Invite-Sicherheitstests, 12 fokussierte Backend-Tests, eine breitere synchrone Suite mit 249 Tests und 32 Frontend-Tests grün; TypeScript-Compile erfolgreich
- Nachbearbeitung zu Finding 3: 7 neue Attachment-Download-Sicherheitstests; zusammen mit Upload- und Retention-Tests 25 Backend-Tests grün. Zusätzlich 32 Frontend-Tests und TypeScript-Compile erfolgreich.
- Nachbearbeitung zu Finding 5: Die sechs ausschließlich auf den am 2026-06-07 entfernten Undo-Kauf-Flow bezogenen Tests und Imports wurden entfernt. Die verbleibenden 27 Tests der beiden Lizenzmodule laufen grün; der Checkout-Confirm-Test sichert explizit ab, dass keine Legacy-Metadaten `direct_activation` oder `undo_until` mehr erzeugt werden.
- Nachbearbeitung zu Finding 6: FastAPI `0.136.3`, Starlette `1.3.1`, Pillow `12.2.0` und python-multipart `0.0.32`; Vite `8.0.16`, Vitest `4.1.8`, React Router `6.30.4` und PostCSS `8.5.15`. `lovable-tagger` wurde als ungenutzte, verwundbare Build-Abhängigkeit entfernt.
- Backend-Gesamtsuite isoliert in Python 3.11 auf dem neu gebauten API-Image ausgeführt: **314 Tests grün, 132 Warnungen**, keine Ausschlüsse und keine Collection-Fehler
- Frontend-Tests isoliert ausgeführt: **32 Tests grün**
- Frontend-Produktionsimage mit Node `20.19` frisch gebaut: **erfolgreich**
- Dependency-Scans nach Aktualisierung: `pip-audit` **0 bekannte Schwachstellen**, `npm audit` **0 Schwachstellen**
- Statische Security-Analyse mit Bandit: keine High-Severity-Funde; zwei kontextabhängige Medium-Hinweise
- Dev-Fixture-Prüfung mit `apps/api/scripts/verify_dev_fixture.py`
- Abgleich mit dem Vorbericht vom 2026-06-07
- Nicht ausgeführt: Live-Browser-/E2E-Test, PostgreSQL-Laufzeitprüfung, echte Stripe-/Brevo-/SMTP-Transaktionen, aktiver Penetrationstest

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-06-07
- Findings im Vorbericht: 5
- Davon behoben: 3
- Davon weiterhin offen: 1
- Davon nicht abschließend verifizierbar: 1
- Regressionen: 0

### Behobene Findings

- Finding 2 - Fehlender reCAPTCHA-Test-Override: behoben. `test_register_requires_captcha_token` setzt den Bypass inzwischen explizit auf `False`.
- Finding 3 - `datetime.utcnow()` in den damals genannten Modulen: behoben. In `dashboard.py`, `licenses.py`, `emails.py`, `economics.py` und `models.py` bestehen die gemeldeten Treffer nicht mehr. In anderen Modulen bleiben 16 neue beziehungsweise außerhalb des damaligen Scopes liegende Treffer.
- Finding 4 - Großer `ReportTypes`-Chunk: behoben. Der aktuelle Build weist `ReportTypes` nur noch mit rund 0,95 kB aus; der größte Chunk ist der allgemeine Charts-Vendor-Chunk mit rund 343 kB.

### Weiterhin offene Findings

- Finding 5 - Stale TODO in `Sidebar.tsx`: weiterhin offen; der Kommentar behauptet eine fehlende Notifications-Anbindung, obwohl `useNotifications()` verwendet wird.

### Nicht abschließend verifizierbare Findings

- Finding 1 - Go-Live-Konfiguration: Die echten Produktiv-Secrets und Schalter sind korrekt nicht im Repository enthalten. Ob BasicAuth, Stripe-Live-Modus, reCAPTCHA und `COOKIE_SECURE` im laufenden Produktivsystem richtig gesetzt sind, lässt sich aus `dev-server` nicht abschließend belegen.

### Regressionen

Keine formale Regression gegenüber dem Vorbericht.

## 4. Confidence

- Confidence: **hoch**

Die wesentlichen Findings sind direkt im aktuellen Commit belegt und wurden durch reale Test-, Build-, Dependency- und History-Scans ergänzt. Der Refund-Flow ist ohne Annahmen aus dem Code ableitbar. Findings 2 und 3 sind durch gezielte Sicherheitstests abgesichert, Finding 5 durch die vollständige Backend-Suite. Finding 6 ist durch scanner-saubere Requirements und Lockfiles, frische Docker-Builds sowie vollständige Backend- und Frontend-Tests verifiziert. Die lokale Bereinigung von Finding 1 und das neue Secret-Scan-Gate wurden zusätzlich statisch und mit Negativtests verifiziert. Einschränkungen bestehen weiterhin bei der Gültigkeit historischer Zugangsdaten, der vollständigen Bereinigung der aktiven GitHub-Historie und dem Verhalten realer Stripe-/Produktivsysteme.

## 5. Kurzfazit

Der Branch enthält mehrere solide Sicherheitsverbesserungen. Findings 2, 3, 5 und 6 sind behoben. Die Backend-Gesamtsuite läuft mit 314 Tests ohne Ausschlüsse, beide Produktionsimages bauen frisch und die Dependency-Scanner melden keine bekannten Schwachstellen mehr. Für einen Release ist der Stand trotzdem nicht freigabefähig: Die neue Sofortkündigung kann eine Erstattung dem falschen Stripe-Charge zuordnen oder einen Refund-Fehler trotz erfolgter Kündigung verschweigen. Finding 1 benötigt weiterhin Provider-Rotation und einen koordinierten Remote-History-Rewrite.

## 6. Wichtigste Findings

1. Historische SMTP-, Stripe- und Microsoft-Secrets bleiben teilweise in aktiven Branch-Historien erreichbar; lokale Legacy-Refs sind bereinigt. - Risk, **hoch**, **teilweise umgesetzt**
2. Support-Anhänge werden ohne Authentifizierung über `/static/attachments/...` ausgeliefert. - Bug, **hoch**, **behoben**
3. Die Sofortkündigung kann den falschen Stripe-Charge erstatten und Refund-Fehler als Erfolg zurückgeben. - Bug, **hoch**
4. Die Backend-Gesamtsuite scheitert bei der Collection; 33 Lizenztests laufen nicht. - Bug, **hoch**, **behoben**
5. `pip-audit` und `npm audit` melden verwundbare Runtime- und Tooling-Abhängigkeiten. - Risk, **hoch**, **behoben**
6. Verifizierungs- und Passwort-Reset-Endpunkte erlauben Account-Enumeration über unterschiedliche Antworten. - Risk, **mittel**
7. Das Brevo-Webhook-Secret bleibt in der Query-String. - Risk, **mittel**
8. Die versionierte Dev-Fixture enthält weiterhin nicht reservierte, real wirkende E-Mail-Domains und fehlende Asset-Referenzen. - Risk, **mittel**
9. Der Notifications-TODO in der Sidebar ist veraltet. - Improvement, **niedrig**

## 7. Detaillierte Findings je Punkt

### Finding 1 - Historische Zugangsdaten in Git-Historien

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: historische Datei `apps/api/.env`; historische Versionen von `apps/api/.env.example` und `deploy/env/.env.api.prod.example`; lokale Legacy-Branches `06.02.2026`, `21.02.2026`, `v0.0.1`, `v0.1.0`; `.gitleaksignore`; `.github/workflows/secret-scan.yml`
- Evidenz: Der ursprüngliche Audit belegte historische SMTP-Zugangsdaten in vier lokalen Legacy-Branches. Diese Branch-Historien wurden anschließend ohne `.env`-Dateien und Datenbank-Backups neu geschrieben; die ursprünglichen Commitobjekte sind lokal nicht mehr vorhanden. Ein vollständiger Gitleaks-Scan identifizierte zusätzlich neun historische Fingerprints für SMTP-, Stripe- und Microsoft-Secrets in den weiterhin aktiven Branch-Historien. Die Werte werden im Bericht bewusst nicht ausgegeben. Nach Aufnahme dieser bekannten Altlasten in die dokumentierte Baseline läuft Gitleaks über 417 erreichbare Commits ohne neue Funde.
- Beschreibung des Problems: Ein Secret, das einmal in Git gespeichert war, muss unabhängig vom aktuellen Dateistand als potenziell offengelegt behandelt werden. Die lokale Legacy-Bereinigung reduziert die Erreichbarkeit in diesem Klon, entfernt Werte aber nicht aus aktiven Remote-Historien, vorhandenen Klonen, Backups oder GitHub-Caches.
- Warum das relevant ist: Bei weiterhin gültigen SMTP-, Stripe- oder Microsoft-Zugangsdaten könnten Angreifer E-Mails versenden, Zahlungs- beziehungsweise Webhook-Flows manipulieren oder auf angebundene Microsoft-Dienste zugreifen. Ob die Credentials noch gültig sind oder missbräuchlich verwendet wurden, wurde nicht aktiv getestet.
- Business Impact: Phishing-, Zahlungs-, Datenschutz-, Zustellbarkeits- und Reputationsrisiko; außerdem hoher operativer Aufwand bei verspäteter Rotation.
- Konkrete Handlungsempfehlung: Alle betroffenen Provider-Secrets rotieren beziehungsweise widerrufen, Deployments aktualisieren und Provider-Logs prüfen. Danach die aktiven Remote-Historien koordiniert neu schreiben, alle Klone aktualisieren, die neun Baseline-Einträge entfernen und Gitleaks ohne Ausnahmen ausführen. GitHub Secret Scanning und Push Protection zusätzlich aktivieren.
- Umsetzungsstand 2026-06-13: **teilweise umgesetzt**. Lokale Legacy-Historien und historische Datenbank-Backups sind bereinigt; Reflogs und alte Objekte wurden gepruned. Das neue CI-Gate kombiniert Gitleaks `8.30.1` mit einer Prüfung, die echte `.env*`-Dateien und Datenbank-Backups in aktiven Refs blockiert. Positiv- und Negativtests waren erfolgreich. Provider-Rotation, Logprüfung, GitHub Push Protection und der Remote-History-Rewrite bleiben unter [[T026-secret-rotation-und-history-cleanup]] offen.

### Finding 2 - Einladungstoken ist nicht an die eingeladene E-Mail-Adresse gebunden

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/auth.py:207-218`, `apps/api/app/routers/teams.py:421-443`, `apps/frontend/src/pages/Register.tsx:431-439`
- Evidenz: Der Registrierungs-Endpunkt sucht die Einladung ausschließlich anhand von Token, Ablauf und Status. Vor dem Erzeugen der Membership fehlt ein Vergleich zwischen `data.email` und `invite.email`. Das Frontend deaktiviert das E-Mail-Feld zwar, ein direkter API-Aufruf kann aber eine beliebige andere E-Mail-Adresse mitsenden. Es existiert kein Test für diesen Mismatch.
- Beschreibung des Problems: Wer einen gültigen Einladungslink erhält oder abfängt, kann damit einen Account unter einer anderen E-Mail-Adresse registrieren und der eingeladenen Organisation beitreten. Der öffentliche Invite-Info-Endpunkt gibt dem Tokeninhaber zusätzlich E-Mail, Organisationsname, Rechnungsadresse, UID und Tätigkeitsdaten zurück.
- Warum das relevant ist: Die UI-Sperre ist keine Sicherheitskontrolle. Die fehlende Serverprüfung betrifft unmittelbar die Mandantentrennung.
- Business Impact: Unberechtigter Zugriff auf Team-, Projekt- und Lizenzdaten einer Organisation; potenzieller Datenschutz- und Vertrauensschaden.
- Konkrete Handlungsempfehlung: Normalisierte E-Mail-Adressen serverseitig exakt vergleichen und bei Abweichung mit HTTP 400/403 abbrechen. Invite-Nutzung atomar gegen Parallelverwendung absichern, die öffentliche Invite-Antwort minimieren und Tests für falsche E-Mail, Replay und parallele Annahme ergänzen.
- Umsetzungsstand 2026-06-13: **behoben**. Der Registrierungs-Endpunkt lädt und sperrt den Invite-Datensatz vor der Benutzeranlage, prüft Tokenstatus und normalisierte E-Mail und speichert Benutzer, Mitgliedschaft sowie `accepted_at` in einer gemeinsamen Transaktion. Ungültige oder bereits verwendete Tokens erzeugen keine Ersatzorganisation mehr. Die öffentliche Invite-Antwort enthält nur noch E-Mail, Organisationsname und Rolle. Vier neue Tests decken normalisierte E-Mail, Mismatch ohne Teilregistrierung, Replay und Datenminimierung ab.

### Finding 3 - Support-Anhänge sind ohne Authentifizierung abrufbar

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/main.py:198-203`, `apps/api/app/routers/support.py:123-190`, `apps/frontend/src/pages/admin/SupportTicketDetail.tsx:470-474`
- Evidenz: Das gesamte Upload-Verzeichnis wird über `app.mount("/static", StaticFiles(...))` öffentlich bereitgestellt. Support-Dateien liegen unter `uploads/attachments/<uuid>`, und das Admin-Frontend öffnet sie direkt über `${API_URL}/static/attachments/...` ohne geschützten Download-Endpunkt.
- Beschreibung des Problems: Kennt jemand die URL, ist keine Session, Adminrolle oder Ticketberechtigung erforderlich. UUID-Dateinamen erschweren das Erraten, ersetzen aber keine Autorisierung; URLs können über Logs, Browserhistorie, Screenshots oder Weitergabe abfließen.
- Warum das relevant ist: Support-Anhänge können Verträge, Screenshots, personenbezogene Daten oder technische Kundendetails enthalten.
- Business Impact: Vertraulichkeits- und DSGVO-Risiko sowie möglicher meldepflichtiger Datenabfluss.
- Konkrete Handlungsempfehlung: Anhänge aus dem öffentlichen Static-Mount entfernen und über einen authentifizierten Download-Endpunkt mit Admin-/Ticketberechtigungsprüfung ausliefern. Alternativ nur kurzlebige signierte URLs verwenden. Avatare/Logos und vertrauliche Support-Dateien in getrennten Storage-Bereichen halten.
- Umsetzungsstand 2026-06-13: **behoben**. `/static` veröffentlicht nur noch die getrennten Bereiche `/static/avatars` und `/static/logos`; `/static/attachments/...` liefert 404. Der neue Endpunkt `GET /admin/support/tickets/{ticket_id}/attachments/{attachment_id}` verlangt `require_admin`, bindet den Anhang per Datenbank-Join an das angeforderte Ticket, lehnt gelöschte Anhänge mit HTTP 410 ab und validiert den lokalen Storage-Pfad gegen Traversal. Die Admin-Oberfläche verwendet ausschließlich diesen Endpunkt; `storage_key` wird nicht mehr an das Frontend ausgegeben. Sieben neue Tests decken öffentlichen Zugriff, 401, 403, erfolgreichen Download, Ticket-Mismatch, Retention-Löschung und ungültige Storage-Pfade ab.

### Finding 4 - Sofortkündigung ordnet Refund nicht zuverlässig den gewählten Lizenzen zu

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/admin.py:632-715`, `apps/api/app/routers/admin.py:1989-2050`
- Evidenz: `_find_latest_refundable_charge()` wählt den ersten erstattbaren Charge des Stripe-Kunden aus den letzten zehn Charges. Es gibt keine Verknüpfung zu den ausgewählten Lizenzen, deren Subscription, Invoice, Billing-Pool oder Laufzeit. Danach werden zuerst Stripe-Kündigungen ausgeführt. Ein Fehler bei `stripe.Refund.create()` wird nur geloggt; die Datenbank wird dennoch committed und der Endpunkt antwortet erfolgreich mit `refund: null`. Für `cancel-now` existieren keine gezielten Backend-Tests.
- Beschreibung des Problems: Bei Organisationen mit mehreren Abos oder Abrechnungsperioden kann eine Erstattung eine andere Zahlung treffen. Scheitert die Erstattung, sind die Lizenzen trotzdem beendet und der Admin erhält keinen Fehlerstatus. Mehrere externe Stripe-Änderungen sind außerdem nicht kompensierbar, falls eine spätere Teiloperation fehlschlägt.
- Warum das relevant ist: Der Flow verändert Zahlung und Vertragsstatus irreversibel über mehrere Systeme hinweg.
- Business Impact: Falsche Erstattungen, fehlende Rückzahlungen, manueller Buchhaltungsaufwand, Kundenbeschwerden und mögliche finanzielle Verluste.
- Konkrete Handlungsempfehlung: Refunds über Invoice/Charge eindeutig an Subscription und Lizenzpositionen binden, Mehrdeutigkeit ablehnen und die maximale Refundhöhe serverseitig prüfen. Refund-Fehler als partiellen Fehler zurückgeben, Idempotency Keys und einen Reconciliation-/Compensation-Flow ergänzen. Tests für mehrere Subscriptions, falschen Charge, zu hohen Refund und Stripe-Teilfehler sind vor Nutzung des Features erforderlich.
- Umsetzungsstand 2026-06-13: **behoben**.

### Finding 5 - Backend-Gesamtsuite scheitert vor der Testausführung

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/tests/test_license_cancel_reactivation.py:10-15`, `apps/api/tests/test_license_checkout_trial.py:10-21`, `apps/api/app/routers/licenses_v2.py`
- Evidenz: Beide Testmodule importieren `undo_license_purchase`, die seit Commit `b2cb122` nicht mehr in `licenses_v2.py` existiert. `pytest -q` bricht mit zwei Collection-Fehlern ab. Dadurch werden 15 plus 18, insgesamt 33 Lizenztests nicht ausgeführt. Mit beiden Modulen ausgeschlossen laufen die verbleibenden 276 Tests grün.
- Beschreibung des Problems: Die Tests wurden nach Entfernung des Undo-Kauffensters nicht an den neuen fachlichen Flow angepasst oder entfernt. Die Gesamtsuite kann deshalb kein grünes CI-Signal liefern.
- Warum das relevant ist: Gerade Lizenz-, Trial-, Checkout-, Kündigungs- und Reaktivierungslogik haben hohe finanzielle Auswirkung. Zusätzlich fehlen Tests für die neue Bulk-Sofortkündigung.
- Business Impact: Erhöhtes Release-Risiko und fehlende Absicherung von zahlungsrelevanten Kernflows.
- Konkrete Handlungsempfehlung: Veraltete Undo-Tests fachlich auf den aktuellen Kündigungs-/Reaktivierungsflow migrieren und die Imports bereinigen. Danach die Bulk-Sofortkündigung einschließlich Refund-Fehlern ergänzen und `pytest` als verpflichtendes CI-Gate setzen.
- Umsetzungsstand 2026-06-13: **behoben**. Die sechs Tests, die ausschließlich den am 2026-06-07 bewusst entfernten 10-Minuten-Undo-Flow und dessen E-Mail prüften, wurden einschließlich ungültiger Imports entfernt. Die verbleibenden 27 Checkout-, Trial-, Kündigungs- und Reaktivierungstests beider Module laufen vollständig. Der Checkout-Confirm-Test prüft nun den aktuellen Vertrag und stellt sicher, dass keine Legacy-Undo-Metadaten mehr geschrieben werden. Die komplette Backend-Suite sammelt und besteht ohne Ausschlüsse mit **314 Tests**.

### Finding 6 - Bekannte Schwachstellen in Backend- und Frontend-Abhängigkeiten

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/requirements.txt`, `apps/frontend/package-lock.json`, `apps/api/app/main.py`, Upload-Endpunkte
- Evidenz: `pip-audit` meldet 13 Advisory-Einträge in `Pillow 12.0.0`, `python-multipart 0.0.21` und der transitiven `Starlette 0.47.3`. Relevant sind unter anderem Multipart-CPU-DoS vor `python-multipart 0.0.27` und ein Range-Header-DoS in `StaticFiles` vor Starlette `0.49.1`. Normdex verarbeitet Multipart-Uploads und liefert Dateien über `StaticFiles` aus. `npm audit` meldet 14 Findings: 2 kritisch, 6 hoch, 6 mittel; betroffen sind unter anderem `vitest 3.2.4`, `vite 7.2.7`, `react-router-dom 6.30.3`, Rollup und esbuild.
- Beschreibung des Problems: Ein Teil der Node-Funde betrifft nur lokal oder im CI laufende Dev-Server und ist nicht im statischen Produktionsbundle aktiv. Die Python-Funde betreffen dagegen reale API-Pfade. Die Vitest-Lücke ist kritisch, falls UI/API beziehungsweise Browser Mode im Netzwerk erreichbar gemacht werden.
- Warum das relevant ist: Unauthentifizierte DoS-Angriffe auf Upload- oder Static-Endpunkte können API-Worker blockieren. Bildparser-Schwachstellen erhöhen das Risiko bei manipulierten Uploads.
- Business Impact: Verfügbarkeitsrisiko, potenzieller Ausfall der Kundenanwendung und steigende Patch-Kosten bei weiterem Aufschub.
- Konkrete Handlungsempfehlung: In einem Dependency-Update-Branch `python-multipart >=0.0.27` und `Pillow >=12.2.0` testen. FastAPI und Starlette gemeinsam auf einen kompatiblen Stand aktualisieren, der alle Scanner-Funde behebt; `pip-audit` nennt je nach Starlette-Advisory Fixstände ab `0.49.1` beziehungsweise `1.0.1`. Frontend mindestens auf `vitest >=3.2.6`, `react-router-dom >=6.30.4` und gepatchte Vite-/Rollup-/PostCSS-Versionen bringen. Danach vollständige Tests, Build und Upload-Smoke-Tests ausführen.
- Primärquellen: [python-multipart GHSA-pp6c-gr5w-3c5g](https://github.com/advisories/GHSA-pp6c-gr5w-3c5g), [Starlette GHSA-7f5h-v6xp-fcq8](https://github.com/advisories/GHSA-7f5h-v6xp-fcq8), [Vitest GHSA-5xrq-8626-4rwp](https://github.com/advisories/GHSA-5xrq-8626-4rwp)
- Umsetzungsstand 2026-06-13: **behoben**. Das Backend verwendet FastAPI `0.136.3`, Starlette `1.3.1`, Pillow `12.2.0` und python-multipart `0.0.32`; `pip-audit` meldet keine bekannten Schwachstellen. Das Frontend verwendet Vite `8.0.16`, Vitest und Coverage `4.1.8`, React Router `6.30.4`, PostCSS `8.5.15` und gepatchte transitive Pakete; `npm audit` meldet 0 Schwachstellen. `lovable-tagger` wurde entfernt, da es nicht genutzt wurde und einen verwundbaren esbuild-Zweig einbrachte. Wegen der Vite-8-/Rolldown-API wurde die bestehende Chunk-Zuordnung als funktionales `manualChunks` abgebildet. API- und Frontend-Produktionsimages bauen frisch; 314 Backend- und 32 Frontend-Tests sind grün.

### Finding 7 - Auth-Endpunkte verraten Account-Status

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/auth.py:397-418`, `apps/api/app/routers/auth.py:493-525`
- Evidenz: `/auth/verify/send` antwortet für unbekannte E-Mails mit `status: ok`, für bereits verifizierte Accounts jedoch mit `status: already_verified`. Beim Passwort-Reset antwortet eine E-Mail mit kürzlich erzeugtem Token mit `status: throttled`, während eine unbekannte Adresse weiterhin `status: ok` erhält.
- Beschreibung des Problems: Unterschiedliche Antworten ermöglichen die Prüfung, ob eine E-Mail registriert beziehungsweise bereits verifiziert ist. Beim Passwort-Reset ist dafür eine vorherige Anfrage nötig.
- Warum das relevant ist: Account-Listen erleichtern gezieltes Phishing und Credential-Stuffing.
- Business Impact: Erhöhtes Missbrauchs- und Support-Risiko, insbesondere bei öffentlich bekannten Firmenadressen.
- Konkrete Handlungsempfehlung: Nach außen immer dieselbe generische Antwort und denselben Statuscode liefern. Detaillierte Zustände nur intern loggen oder nach authentifiziertem Login anzeigen; Timing und Rate Limits ebenfalls vereinheitlichen.

### Finding 8 - Brevo-Webhook-Secret steht im Query-Parameter

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-23)
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/newsletter.py:90-97`, `apps/api/tests/test_newsletter.py`, `D:\Normdex\02_knowledge\normdex-vault\02_App\API-Endpunkte.md`
- Evidenz: Der Endpunkt erwartet weiterhin `POST /newsletter/brevo/webhook?secret=...`. Tests und Vault-Dokumentation verwenden denselben Query-String.
- Beschreibung des Problems: Query-Strings werden häufiger in Proxy-, Access-, Monitoring- und Fehlerlogs gespeichert als Header. Bei Leak können Brevo-Ereignisse simuliert werden, soweit die weiteren Payload-Bedingungen erfüllt sind.
- Warum das relevant ist: Das Secret schützt die Newsletter- und Gutscheinlogik.
- Business Impact: Missbräuchliche Coupon-/Mail-Ereignisse, Log-Bereinigung und Secret-Rotation.
- Konkrete Handlungsempfehlung: Secret nach Möglichkeit in einen Header verschieben. Falls Brevo keine Custom Header unterstützt, einen geheimen Webhook-Pfad plus strikte Payload-/Listenprüfung verwenden und Query-Logging für diesen Pfad deaktivieren.

### Finding 9 - Dev-Fixture benötigt weiterhin strengere Datenschutz-Governance

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: persistent (seit 2026-05-22)
- Betroffene Datei(en) oder Pfade: `apps/api/dev.db`, `apps/api/scripts/verify_dev_fixture.py`, `apps/api/uploads/avatars/`, `apps/api/org_data.json`
- Evidenz: Das Prüfskript meldet nicht reservierte Domains unter anderem in `users`, `invites`, `outbox_emails`, Supportdaten und Audit-Metadaten. Zusätzlich fehlen zwei referenzierte Avatar-/Logo-Dateien. Die Datenbank ist bewusst als Dev-Snapshot versioniert; aus dem Repo allein lässt sich nicht belegen, ob alle Namen, E-Mails und Unternehmensdaten vollständig synthetisch sind.
- Beschreibung des Problems: Reproduzierbare Fixtures sind sinnvoll, aber real wirkende Domains und sensible Tabellenfelder erhöhen das Risiko, dass echte Daten unbemerkt im Repository landen.
- Warum das relevant ist: Git-Historie ist langlebig und schwer nachträglich zu bereinigen.
- Business Impact: Datenschutz-, Compliance- und Reputationsrisiko sowie inkonsistente lokale Demo-Setups.
- Konkrete Handlungsempfehlung: Fixture auf reservierte Domains wie `example.com` umstellen, fehlende Asset-Referenzen bereinigen und `verify_dev_fixture.py --strict` als Pflicht-Gate vor Snapshot-Commits etablieren. Zusätzlich dokumentierte Freigabe durch eine zweite Person für jede neue `dev.db`-Version.

### Finding 10 - Veralteter Notifications-TODO

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-06-07)
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/components/layout/Sidebar.tsx:10`, `apps/frontend/src/components/layout/Sidebar.tsx:214`
- Evidenz: Der Kommentar fordert die Anbindung von `notifCount` an die Notifications-API, während der Wert bereits aus `useNotifications()` bezogen wird.
- Beschreibung des Problems: Der Kommentar beschreibt einen nicht mehr vorhandenen Arbeitsstand.
- Warum das relevant ist: Irreführender technischer Debt verlangsamt Reviews und Fehlersuche.
- Business Impact: Gering; ausschließlich Wartungskosten.
- Konkrete Handlungsempfehlung: Kommentar entfernen.

## 8. Quick Wins

- SMTP-, Stripe- und Microsoft-Zugangsdaten rotieren beziehungsweise widerrufen und Provider-Logs prüfen.
- Den neuen Secret-Scan-Workflow pushen und als verpflichtenden Branch-Check aktivieren.
- Dependency-Scans als verpflichtende Merge-Gates konfigurieren; der aktuelle Stand ist scanner-sauber.
- `pytest` als verpflichtendes CI-Gate konfigurieren; die Suite ist nach der Testmigration vollständig grün.
- Brevo-Secret aus der Query entfernen oder den Query-String am Proxy gezielt vom Logging ausschließen.
- Den veralteten Sidebar-TODO löschen.

## 9. Strategische Empfehlungen

- Sicherheitsrelevante Endpunkte mit einem kleinen, verpflichtenden Testpaket absichern: Mandantentrennung, Account-Enumeration und Stripe-Teilfehler. Invite-E-Mail-Bindung und Attachment-Download sind inzwischen abgedeckt.
- Externe Zahlungsoperationen als nachvollziehbare Zustandsmaschine mit Idempotency Keys, Pending-/Failed-Zuständen und Reconciliation-Job modellieren. Ein Datenbank-Rollback kann Stripe-Änderungen nicht zurückrollen.
- Vertrauliche Uploads nicht im selben öffentlichen Static-Storage wie Avatare und Logos halten. Ein separates Storage- und Berechtigungsmodell reduziert den künftigen Prüfaufwand.
- Das bereits ergänzte Secret-Scan-Gate nach dem Push als verpflichtenden Branch-Check konfigurieren. Ergänzend `pip-audit`, `npm audit` und ein grünes `pytest` als Merge-Gates etablieren.
- Nach der Provider-Rotation die aktiven Remote-Historien koordiniert bereinigen und alle Entwicklerklone neu synchronisieren. Erst danach die temporäre Gitleaks-Baseline entfernen.

## 10. Empfohlene nächste Aktion

**Vor dem nächsten Release zuerst T026 abschließen: betroffene SMTP-, Stripe- und Microsoft-Secrets rotieren, Provider-Logs prüfen und anschließend die aktive Remote-Historie koordiniert bereinigen.**

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Waren die inzwischen lokal bereinigten Legacy-Branches früher auf GitHub, in Backups oder in anderen Entwicklerklonen vorhanden?
- Sind die historischen SMTP-, Stripe- und Microsoft-Credentials beim jeweiligen Provider noch gültig, und gibt es verdächtige Zugriffe oder Transaktionen?
- Wann kann der koordinierte History-Rewrite der aktiven Remote-Branches und Tags durchgeführt werden, ohne laufende Arbeiten oder Deployments zu verlieren?
- Nach Deployment prüfen, ob Dev und Produktion tatsächlich die neu gebauten Images mit FastAPI `0.136.3`/Starlette `1.3.1` beziehungsweise Vite `8.0.16` verwenden.
- Sind ältere öffentliche Support-Attachment-URLs in Traefik-, Browser-, Monitoring- oder E-Mail-Logs enthalten? Die URLs funktionieren nach Deployment nicht mehr, vorhandene Logdaten können aber weiterhin sensible Pfade enthalten.
- Wie verhält sich die Bulk-Sofortkündigung bei realen Organisationen mit mehreren Stripe-Subscriptions, Schedules und Teilrefunds?
- Sind die Produktivschalter für BasicAuth, Stripe-Live-Modus, reCAPTCHA, `COOKIE_SECURE` und `APP_ENV=prod` korrekt gesetzt?
- Enthält `apps/api/dev.db` ausschließlich synthetische beziehungsweise ausdrücklich freigegebene Daten?
- Kein Live-Penetrationstest, keine PostgreSQL-Prüfung und kein vollständiger E2E-Test wurden durchgeführt.
- Der Frontend-Produktionsbuild konnte in dieser Nachbearbeitung nicht wiederholt werden, weil im vorhandenen root-eigenen `node_modules` das laut `package.json` erwartete Paket `@fontsource/jetbrains-mono` fehlt; TypeScript-Compile und alle 32 Frontend-Tests waren erfolgreich.
