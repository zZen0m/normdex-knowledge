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
- Git-Status: Arbeitsbaum sauber; lokaler Branch liegt 4 Commits vor `origin/dev-server`

## 2. Audit-Abdeckung

- Normdex-Vault-Kontext gelesen: Architektur, Authentifizierung und Sicherheit, Datenschutzprozess, API-Endpunkte, Integrationen sowie Deployment-Workflow
- Statische Codeanalyse von FastAPI-Routen, Authentifizierung, Rollen/Rechten, Mandantentrennung, Support, Admin- und Stripe-Flows
- Frontend-Prüfung von Registrierung, Admin-Support, geschützten Routen, API-Nutzung und sicherheitsrelevanten DOM-Ausgaben
- Datenbank- und Migrationsprüfung: 40 Alembic-Migrationen, genau ein Head (`ffd3bbde6b6a`), keine fehlenden Vorgänger
- Git-Historie und aktuelle Remote-Refs auf versehentlich eingecheckte Secrets geprüft, ohne Secret-Werte auszugeben
- Backend-Tests isoliert in Python 3.11 ausgeführt: Gesamtsuite durch 2 Collection-Fehler blockiert; Restlauf mit den beiden betroffenen Modulen ausgeschlossen: **276 Tests grün, 162 Warnungen**
- Frontend-Tests isoliert ausgeführt: **32 Tests grün**
- Frontend-Build ausgeführt: **erfolgreich**
- Dependency-Scans ausgeführt: `pip-audit` und `npm audit`
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

Die wesentlichen Findings sind direkt im aktuellen Commit belegt und wurden durch reale Test-, Build-, Dependency- und History-Scans ergänzt. Die Einladungslücke, der öffentliche Attachment-Zugriff, der Refund-Flow und die Test-Collection-Fehler sind ohne Annahmen aus dem Code ableitbar. Einschränkungen bestehen bei der tatsächlichen Erreichbarkeit alter Git-Objekte auf GitHub, der Gültigkeit externer Zugangsdaten und dem Verhalten realer Stripe-/Produktivsysteme.

## 5. Kurzfazit

Der Branch enthält mehrere solide Verbesserungen und 276 der ausführbaren Backend-Tests sowie alle 32 Frontend-Tests laufen grün. Für einen Release ist der Stand trotzdem nicht freigabefähig: Zwei Testmodule mit 33 Lizenztests werden gar nicht gesammelt, und die neue Sofortkündigung kann eine Erstattung dem falschen Stripe-Charge zuordnen oder einen Refund-Fehler trotz erfolgter Kündigung verschweigen. Sicherheitsseitig sind die fehlende E-Mail-Bindung von Einladungen und öffentlich ausgelieferte Support-Anhänge die wichtigsten Anwendungsrisiken. Zusätzlich enthalten Backend und Frontend bekannte verwundbare Abhängigkeiten; bei Starlette und `python-multipart` ist der konkrete Normdex-Einsatz direkt betroffen. Alte lokale Git-Refs enthalten historische SMTP-Zugangsdaten, die mit der aktuellen Dev-Konfiguration übereinstimmen, auch wenn diese Legacy-Branches derzeit nicht mehr als Remote-Refs auf GitHub sichtbar sind. Der Trend gegenüber dem 2026-06-07-Audit ist deshalb gemischt: frühere Qualitätsprobleme wurden behoben, gleichzeitig sind neue Release- und Sicherheitsrisiken hinzugekommen.

## 6. Wichtigste Findings

1. Historische SMTP-Zugangsdaten liegen in lokalen Git-Refs und entsprechen weiterhin der Dev-Konfiguration. - Risk, **hoch**
2. Ein Einladungstoken ist backendseitig nicht an die eingeladene E-Mail-Adresse gebunden. - Bug, **hoch**
3. Support-Anhänge werden ohne Authentifizierung über `/static/attachments/...` ausgeliefert. - Bug, **hoch**
4. Die Sofortkündigung kann den falschen Stripe-Charge erstatten und Refund-Fehler als Erfolg zurückgeben. - Bug, **hoch**
5. Die Backend-Gesamtsuite scheitert bei der Collection; 33 Lizenztests laufen nicht. - Bug, **hoch**
6. `pip-audit` und `npm audit` melden verwundbare Runtime- und Tooling-Abhängigkeiten. - Risk, **hoch**
7. Verifizierungs- und Passwort-Reset-Endpunkte erlauben Account-Enumeration über unterschiedliche Antworten. - Risk, **mittel**
8. Das Brevo-Webhook-Secret bleibt in der Query-String. - Risk, **mittel**
9. Die versionierte Dev-Fixture enthält weiterhin nicht reservierte, real wirkende E-Mail-Domains und fehlende Asset-Referenzen. - Risk, **mittel**
10. Der Notifications-TODO in der Sidebar ist veraltet. - Improvement, **niedrig**

## 7. Detaillierte Findings je Punkt

### Finding 1 - Historische SMTP-Zugangsdaten entsprechen der aktuellen Dev-Konfiguration

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: lokale Git-Branches `06.02.2026`, `21.02.2026`, `v0.0.1`, `v0.1.0`; historische Datei `apps/api/.env`; aktuelle ignorierte Datei `deploy/env/.env.api.dev`
- Evidenz: `apps/api/.env` war in den Commits `488d69d`, `6065d52`, `902f95b` und `b2c558e` versioniert. Ein wertfreier Hashvergleich bestätigt, dass `SMTP_USERNAME` und `SMTP_PASSWORD` aus diesen Commits mit der aktuellen Dev-Konfiguration übereinstimmen. Die Legacy-Branches sind lokal erreichbar; `git ls-remote` zeigt sie aktuell nicht mehr als GitHub-Remote-Refs.
- Beschreibung des Problems: Ein Secret, das einmal in Git gespeichert war, muss als potenziell offengelegt behandelt werden. Das Löschen der Datei oder des Remote-Branches entfernt den Wert nicht zuverlässig aus allen vorhandenen Klonen, Backups oder GitHub-Caches.
- Warum das relevant ist: Bei weiterhin gültigen SMTP-Zugangsdaten könnten Angreifer E-Mails im Namen von Normdex versenden oder die Absenderreputation beschädigen. Ob die Credentials extern noch akzeptiert werden und ob die alten Branches früher gepusht waren, wurde nicht aktiv getestet.
- Business Impact: Phishing-, Zustellbarkeits- und Reputationsrisiko; möglicher Datenschutzvorfall bei missbräuchlichem Zugriff auf das Mailkonto.
- Konkrete Handlungsempfehlung: SMTP-Benutzer beziehungsweise Passwort sofort rotieren, alle Deployments aktualisieren, Provider-Logs auf Missbrauch prüfen und die Legacy-Refs nach gesicherter Archivierung bereinigen. Zusätzlich GitHub Secret Scanning/Push Protection aktivieren und die alte Historie gezielt mit `git filter-repo` bereinigen, falls sie jemals remote veröffentlicht war.

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

- SMTP-Zugangsdaten rotieren und Provider-Logs prüfen.
- Invite-Annahme um den normalisierten E-Mail-Vergleich ergänzen und einen Negativtest hinzufügen.
- `python-multipart`, Pillow, Vitest und `react-router-dom` auf gepatchte Versionen aktualisieren.
- Die zwei veralteten `undo_license_purchase`-Imports entfernen beziehungsweise die 33 Tests auf den aktuellen Flow migrieren.
- Brevo-Secret aus der Query entfernen oder den Query-String am Proxy gezielt vom Logging ausschließen.
- Den veralteten Sidebar-TODO löschen.

## 9. Strategische Empfehlungen

- Sicherheitsrelevante Endpunkte mit einem kleinen, verpflichtenden Testpaket absichern: Invite-E-Mail-Bindung, Mandantentrennung, Attachment-Download, Account-Enumeration und Stripe-Teilfehler.
- Externe Zahlungsoperationen als nachvollziehbare Zustandsmaschine mit Idempotency Keys, Pending-/Failed-Zuständen und Reconciliation-Job modellieren. Ein Datenbank-Rollback kann Stripe-Änderungen nicht zurückrollen.
- Vertrauliche Uploads nicht im selben öffentlichen Static-Storage wie Avatare und Logos halten. Ein separates Storage- und Berechtigungsmodell reduziert den künftigen Prüfaufwand.
- Dependency- und Secret-Scanning als CI-Gates etablieren: `pip-audit`, `npm audit`, GitHub Secret Scanning und ein grünes `pytest` müssen vor Merge erfüllt sein.
- Lokale Legacy-Branches und alte Tags inventarisieren. Nicht mehr benötigte Refs nach Secret-Rotation und Archivierung entfernen, damit versehentlich enthaltene Daten nicht dauerhaft in Entwicklerklonen bleiben.

## 10. Empfohlene nächste Aktion

**Vor dem Push beziehungsweise Release der vier lokalen Commits einen kurzen Security-/Billing-Fix-Sprint durchführen: SMTP rotieren, Invite-E-Mail serverseitig binden, Support-Anhänge schützen, den Sofortkündigungs-Refund korrigieren und die vollständige Backend-Suite wieder grün machen.**

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Waren die lokalen Legacy-Branches mit der alten `.env` früher auf GitHub oder in anderen Remotes veröffentlicht? Aktuell zeigt `git ls-remote` nur `dev-server`, `develop` und `main`.
- Sind die historischen SMTP-Credentials beim Provider noch gültig, und gibt es verdächtige Login-/Versandereignisse?
- Welche Dependency-Versionen laufen tatsächlich in Dev und Produktion? Der Audit bewertet Lockfile/Requirements, nicht bereits gebaute Images.
- Sind Support-Attachment-URLs in Traefik-, Browser-, Monitoring- oder E-Mail-Logs enthalten?
- Wie verhält sich die Bulk-Sofortkündigung bei realen Organisationen mit mehreren Stripe-Subscriptions, Schedules und Teilrefunds?
- Sind die Produktivschalter für BasicAuth, Stripe-Live-Modus, reCAPTCHA, `COOKIE_SECURE` und `APP_ENV=prod` korrekt gesetzt?
- Enthält `apps/api/dev.db` ausschließlich synthetische beziehungsweise ausdrücklich freigegebene Daten?
- Kein Live-Penetrationstest, keine PostgreSQL-Prüfung und kein vollständiger E2E-Test wurden durchgeführt.
