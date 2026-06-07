# Repo-Audit - Webapp - 2026-06-07

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-app`
- Branch: `develop`
- Commit: `9f4b7ed`
- Version laut `package.json`: `0.0.1`
- Letzter Git-Tag: `keiner` (stale Tags v0.1.0/v0.2.0 wurden im Vorbericht als experimentell eingestuft)
- Audit-Datum: `2026-06-07`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-06-07-repo-audit-webapp.md`
- Vorheriger Audit: `2026-06-04-repo-audit-webapp.md`
- Audit-Fokus: Go-Live-Readiness-Bewertung + vollständiger Folgeaudit; neuer Code seit 2026-06-04 (Onboarding-Widget T024, Welcome-Tutorial T025, Projekt-Redesign, Fixes); Abgleich aller Vorbericht-Findings.

---

## 2. Audit-Abdeckung

- Statische Codeanalyse: `apps/api/app/routers/auth.py`, `dashboard.py`, `licenses.py`, `economics.py`, `main.py`, `models.py`, `emails.py`; `apps/frontend/src/App.tsx`, `Sidebar.tsx`, `components/tutorial/*`, `context/TutorialContext.tsx`
- Backend-Testsuite real ausgeführt: **1 failed, 296 passed, 838 warnings** (~114 s)
- Frontend-Testsuite real ausgeführt: **32 Tests grün** (4 Dateien, identisch zu Vorbericht)
- Frontend-Build real ausgeführt: **erfolgreich**, kein Chunk-Warnung, größter Chunk `ReportTypes` = 332 kB (gzip 99 kB)
- Deployment-Konfiguration: `deploy/docker-compose.prod.yml`, `deploy/prod-compose.sh`, `deploy/env/*.example`
- Git-Log: 15 neue Commits seit 2026-06-04 analysiert
- Abgleich Vorbericht: 2026-06-04 (9 Findings, alle im selben Bericht als behoben markiert)
- Nicht ausgeführt: kein Live-Browser-/E2E-Test, keine PostgreSQL-Laufzeitprüfung, kein realer Stripe-/Brevo-Durchlauf

---

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-06-04
- Findings im Vorbericht: 9 (alle im selben Bericht als behoben markiert)
- Davon behoben: 9
- Davon weiterhin offen: 0
- Davon nicht abschließend verifizierbar: 0
- Regressionen: 0

### Behobene Findings

- Finding 1 - Self-Delete löscht org-weit geteilte Projekte hart: **behoben** (Evidenz: `app/services/account_deletion.py` mit `_reassign_or_delete_owned_data` vorhanden, `_resolve_org_recipient` transferiert Projekte an Team-Owner)
- Finding 2 - Zwei divergierende Lösch-Strategien: **behoben** (Evidenz: `account_deletion.py`-Service existiert; `admin.py` und `users.py` importieren `anonymize_user_account`)
- Finding 3 - Versionsdrift: **weiterhin geklärt** (Version 0.0.1 ist korrekt, kein Code-Handlungsbedarf)
- Finding 4 - support_admin.py umgeht require_admin: **behoben** (Evidenz: `from app.deps import require_admin` vorhanden; `get_current_user + is_admin`-Muster nicht mehr in support_admin.py)
- Finding 5 - Datetime-Deprecations: **überwiegend behoben** (838 Warnings; Reduktion von 1121 → 1068 → 838 durch schrittweise Router-Migration inkl. Commit `479a2b0 "datetime-Migration repoweit"`; `models.py` und einzelne Routen bleiben, s. Finding 3 dieses Berichts)
- Finding 6 - dev.db.backup-Binärdatei in Git getrackt: **behoben** (Evidenz: `.gitignore` enthält `apps/api/dev.db.backup*`)
- Finding 7 - Tippfehler „Ur JPG": **behoben** (Evidenz: `users.py:143` lautet nun `"Nur JPG, PNG oder WebP erlaubt."`)
- Finding 8 - Frontend-Bundle über 500 kB: **behoben** (Evidenz: Build-Output zeigt `index-*.js` = 121 kB, Chunk-Warnung entfällt; `manualChunks` in `vite.config.ts` aktiv)
- Finding 9 - unleash-*.json in alembic/versions: **behoben** (Evidenz: `.gitignore` enthält `apps/api/alembic/versions/unleash-*.json`)

### Weiterhin offene Findings

Keine.

### Regressionen

Keine formale Regression. `dashboard.py` (5×), `licenses.py` (5×), `emails.py` (2×), `economics.py` (1×) enthalten weiterhin `datetime.utcnow()`-Aufrufe — diese Module waren im Vorbericht nicht explizit in der Migrations-Scope. Wird als neues Finding 3 geführt.

---

## 4. Confidence

- Confidence: **hoch**

Backend-Testsuite (296 grün), Frontend-Testsuite (32 grün) und Frontend-Build wurden real ausgeführt. Alle Vorbericht-Findings sind anhand des aktuellen Code-Zustands verifiziert. Einschränkung: kein Laufzeitbrowser-Test, keine PostgreSQL-Umgebung, keine echten Stripe-/Brevo-Webhooks. Die Go-Live-Bewertung bezieht sich auf Deployment-Konfiguration — diese liegt außerhalb der statischen Code-Analyse.

---

## 5. Kurzfazit

Die Webapp ist seit dem letzten Audit (2026-06-04) fachlich erheblich gewachsen: Onboarding-Widget (T024) und Welcome-Tutorial (T025) wurden fertiggestellt, die Projektdetailseite wurde visuell überarbeitet, und mehrere Bugfixes (has_project, has_license, Rechnungsadresse-Schritt) wurden eingebaut. Alle 9 Vorbericht-Findings sind behoben. Das Resultat ist eine technisch solide Basis, die für einen ersten Kundenstart geeignet ist.

**Go-Live-Bewertung:** Die App ist technisch bereit. Kein Code-Blocker verhindert den Produktivbetrieb. Es gibt aber **drei Deployment-Konfigurationsschalter**, die vor dem Launch aktiv gesetzt werden müssen (BasicAuth, Stripe-Live-Keys, reCAPTCHA), und **einen Code-Fehler** im Test-Isolation-Bereich (1 failing test). Trend seit 2026-05-22: **kontinuierliche, deutliche Verbesserung**; die Qualität ist über die letzten fünf Audits konsistent gestiegen.

---

## 6. Wichtigste Findings

1. **Go-Live-Konfigurationsschalter nicht gesetzt** — HTTP BasicAuth, Stripe Test-Keys, RECAPTCHA-Bypass: alle drei müssen vor dem Launch explizit auf Prod-Werte umgestellt werden. — Risk, **kritisch für Launch**
2. **1 failing Test** (`test_register_requires_captcha_token`) — Test-Isolation-Bug; `.env` hat `RECAPTCHA_BYPASS_ENABLED=true`, Test monkeypatcht nicht → 200 statt 400. — Bug, **hoch**
3. **datetime.utcnow() in dashboard.py, licenses.py, emails.py, economics.py, models.py** — Deprecated seit Python 3.12; `models.py` ist Haupt-Quelle (Spalten-Defaults) der verbleibenden 838 Warnings. — Improvement, mittel (persistent)
4. **ReportTypes-Chunk 332 kB** — größtes App-eigenes Bundle (gzip 99 kB), bisher kein Code-Splitting für Berechnungstypen. — Optimization, niedrig
5. **Stale TODO in Sidebar.tsx** — Kommentar „notifCount an die Notifications-API anschließen" ist bereits umgesetzt (`useNotifications()` Hook). — Improvement, niedrig

---

## 7. Detaillierte Findings je Punkt

### Finding 1 - Go-Live-Deployment-Konfiguration nicht für Kundenbetrieb gesetzt

- Kategorie: Risk
- Priorität: **kritisch**
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `deploy/docker-compose.prod.yml:55-60`, `deploy/env/.env.deploy.prod.example`, `deploy/prod-compose.sh`, `apps/api/.env`

**Evidenz — drei separate Teilrisiken:**

**1a. HTTP BasicAuth aktiv (app.normdex.at nicht öffentlich erreichbar)**
`deploy/docker-compose.prod.yml:59` hat `traefik.http.middlewares.normdex-auth.basicauth.users` hartcodiert und Zeile 60 aktiviert es. `deploy/prod-compose.sh:15` liest `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED` (Default: `true`). `deploy/env/.env.deploy.prod.example:7` bestätigt `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=true` als Standardwert. Solange dieser Wert nicht auf `false` gesetzt ist, landet jeder Besucher von `app.normdex.at` vor einem Browser-Auth-Dialog.

**1b. Stripe Test-Keys in dev/.env, Prod-Env-Example zeigt `sk_live_*`-Platzhalter**
`apps/api/.env:23`: `STRIPE_SECRET_KEY=sk_test_...`; `deploy/env/.env.api.prod.example:21`: `sk_live_your_secret_key_here`. Wurden in `deploy/env/.env.api.prod` noch keine echten Live-Keys hinterlegt, laufen Zahlungen im Stripe-Test-Modus — Kunden würden keine echten Transaktionen auslösen.

**1c. RECAPTCHA_BYPASS_ENABLED=true in dev/.env**
`apps/api/.env:49`: `RECAPTCHA_BYPASS_ENABLED=true`. `deploy/env/.env.api.prod.example:51`: `RECAPTCHA_BYPASS_ENABLED=false`. Sind die Produktiv-Env-Variablen aus der Dev-.env übernommen worden (Copy-Paste-Fehler), ist die Registrierung ohne CAPTCHA-Prüfung offen für Bot-Registrierungen.

- Beschreibung des Problems: Drei kritische Deployment-Schalter haben in der Dev-Umgebung sicherheitstechnisch bewusst deaktivierte Werte, die vor dem Launch für die Produktivumgebung explizit gesetzt werden müssen. Alle drei sind dokumentiert, aber der Ablauf erfordert manuelle Eingriffe ohne automatisches Failsafe.
- Warum das relevant ist: BasicAuth verhindert jeden Kundenbesuch. Falsche Stripe-Keys führen zu wirkungslosen Zahlungen. Aktiver RECAPTCHA-Bypass öffnet die Registrierung für automatisierte Accounts.
- Business Impact: Launch-Blocker (BasicAuth), Umsatzverlust (Stripe Test-Mode), Spam/Betrugsrisiko (RECAPTCHA).
- Konkrete Handlungsempfehlung: Vor dem Launch explizit als Checkliste abarbeiten:
  1. `deploy/env/.env.deploy.prod` anlegen/prüfen: `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false`
  2. `deploy/env/.env.api.prod` prüfen: Stripe Live-Keys gesetzt, `RECAPTCHA_BYPASS_ENABLED=false`, `BREVO_API_KEY` gesetzt (in dev/.env leer!)
  3. `deploy/prod-compose.sh` für den Launch nutzen (liest `.env.deploy.prod` korrekt)
  4. Smoke-Test-Checkliste (`11_Audits/Webapp/2026-05-14-smoke-test-checkliste-webapp.md`) gegen Prod ausführen

---

### Finding 2 - Failing Test: test_register_requires_captcha_token

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/tests/test_auth_register_recaptcha.py:40-46`, `apps/api/.env:49`, `apps/api/app/routers/auth.py:72-77`

- Evidenz: `pytest -q` meldet `FAILED tests/test_auth_register_recaptcha.py::test_register_requires_captcha_token`. Der Test sendet `captcha_token=None` und erwartet HTTP 400 mit `"Bitte bestätige, dass du kein Roboter bist."`. `auth.py:72-77` prüft zuerst `if settings.RECAPTCHA_BYPASS_ENABLED: return True` — und da `apps/api/.env:49` `RECAPTCHA_BYPASS_ENABLED=true` setzt, liefert die Funktion `True` ohne 400. Der Test hat kein `monkeypatch.setattr(...RECAPTCHA_BYPASS_ENABLED, False)`, anders als die drei anderen Tests derselben Datei (Zeilen 50, 56, 76).
- Beschreibung des Problems: Test-Isolation-Bug: Der Test setzt stillschweigend voraus, dass `RECAPTCHA_BYPASS_ENABLED=False` gilt (entspricht dem Code-Default in `config.py:55`), lädt aber die `.env` des lokalen Dev-Setups, die den Wert auf `True` überschreibt. Kein Production-Bug — aber der Test ist seit der `RECAPTCHA_BYPASS_ENABLED=true`-Änderung in `.env` dauerhaft rot.
- Warum das relevant ist: Ein dauerhaft fehlschlagender Test im CI erzeugt Alert-Müdigkeit und maskiert echte Regressionen.
- Business Impact: Niedrig für Code, mittel für Teamproduktivität (CI-Rot-Signal verliert Aussagekraft).
- Konkrete Handlungsempfehlung: In `test_register_requires_captcha_token` ein `monkeypatch.setattr("app.routers.auth.settings.RECAPTCHA_BYPASS_ENABLED", False)` ergänzen (analog zu `test_register_rejects_invalid_captcha` in derselben Datei). Einzeiliger Fix.

---

### Finding 3 - datetime.utcnow() in mehreren Modulen und models.py (persistent)

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-23; teilweise behoben in Vorbericht-Scope; diese Module waren nicht in der Migrations-Scope)
- Betroffene Datei(en) oder Pfade:
  - `apps/api/app/routers/dashboard.py:78, 107, 119, 123, 226` (5 Direktaufrufe)
  - `apps/api/app/routers/licenses.py:53, 64, 123, 125, 142, 205, 229` (7 Direktaufrufe)
  - `apps/api/app/routers/economics.py:1103` (1 Direktaufruf)
  - `apps/api/app/emails.py:2140, 2154` (2 Direktaufrufe)
  - `apps/api/app/models.py:94, 108, 122, 134, 157-158, 253-254, 305, 332-333, 383, 412` (Spalten-Defaults, Haupt-Quelle der 838 Warnings)

- Evidenz: `grep -rn "datetime.utcnow" apps/api/app/` liefert diese Treffer. Das vorherige Audit hatte `users.py`, `scheduler.py`, `licenses_v2.py`, `auth.py` migriert. `dashboard.py`, `licenses.py`, `economics.py`, `emails.py` wurden nicht explizit adressiert. `models.py` hat `default=datetime.utcnow` als callables (keine Direktaufrufe per se, aber Python 3.12 warnt trotzdem).
- Beschreibung des Problems: `datetime.utcnow()` ist seit Python 3.12 deprecated. `app/util/dt.py` mit `now_utc_naive()` ist als Ersatz vorhanden. Die Migration wurde für die am häufigsten gewarnten Routen begonnen, aber nicht abgeschlossen. `models.py` ist der dominante Verursacher der verbleibenden 838 Warnings (jede DB-Operation auf einem Modell mit `default=datetime.utcnow` generiert mehrere Warnungen).
- Warum das relevant ist: Warnungen bei `models.py`-Defaults können zukünftige Python- und SQLAlchemy-Upgrades blockieren; echte neue Warnungen werden im Warning-Rauschen verschleiert.
- Business Impact: Wartungsrisiko, potenziell verzögerter Python-Upgrade-Pfad.
- Konkrete Handlungsempfehlung:
  1. Kurzfristig: `dashboard.py`, `licenses.py`, `economics.py`, `emails.py` auf `now_utc_naive()` migrieren (~15 Änderungen gesamt).
  2. Mittelfristig: `models.py`-Spalten-Defaults auf `default=now_utc_naive` umstellen (erfordert Import und ggf. Test-Anpassung); `pytest.ini`-Gate entsprechend erweitern.

---

### Finding 4 - ReportTypes-Chunk 332 kB nach Minifizierung

- Kategorie: Optimization
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/dist/assets/ReportTypes-BupkaAGu.js` (332 kB / gzip 99 kB), `apps/frontend/src/pages/ReportTypes.tsx` (via Lazy-Import in `EconomicsForm.tsx`, `EconomicsReport.tsx`)

- Evidenz: `npm run build` zeigt `ReportTypes-BupkaAGu.js` als größten App-eigenen Chunk (332 kB, gzip 99 kB); deutlich größer als `EconomicsForm` (158 kB) und `EconomicsReport` (67 kB). `ReportTypes` wird aus `EconomicsForm` und `EconomicsReport` direkt importiert, jedoch nicht dynamisch geladen. Da beide Pages selbst lazy sind, ist `ReportTypes` effektiv transitiv lazy — wird nur geladen, wenn der Nutzer zu einer Berechnungsseite navigiert.
- Beschreibung des Problems: Der `ReportTypes`-Chunk ist der größte App-Chunk und wächst mit jedem neuen Berechnungstyp. Aktuell kein Blocker, da transitiv lazy geladen.
- Warum das relevant ist: Der Chunk wird beim Öffnen der Berechnungsseite vollständig geladen — 99 kB gzip auf einem langsamen Mobilnetz sind spürbar.
- Business Impact: Kein unmittelbarer Conversion-Verlust; potenziell spürbar für den ersten Berechnung-Flow-Aufruf.
- Konkrete Handlungsempfehlung: Mittelfristig prüfen, ob `ReportTypes` dynamisch nach Bedarf (Berechnungstyp-spezifisch) geladen werden kann. Kurzfristig: als CI-Budget-Gate in `vite.config.ts` (`build.chunkSizeWarningLimit`) führen.

---

### Finding 5 - Stale TODO-Kommentar in Sidebar.tsx

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/components/layout/Sidebar.tsx:10`

- Evidenz: Zeile 10 enthält `→ TODO: notifCount an die Notifications-API anschließen`. Zeile 214 zeigt `const { unreadCount: notifCount } = useNotifications();` — die Notification-API ist bereits angebunden.
- Beschreibung des Problems: Der TODO-Kommentar beschreibt eine bereits implementierte Funktionalität. Kein fachlicher Fehler, aber irreführend bei der Code-Review.
- Konkrete Handlungsempfehlung: TODO-Kommentar entfernen.

---

## 8. Quick Wins

- **Finding 2 beheben (1 Zeile):** `monkeypatch.setattr("app.routers.auth.settings.RECAPTCHA_BYPASS_ENABLED", False)` am Anfang von `test_register_requires_captcha_token` ergänzen → Test grün, CI wieder sauber.
- **Finding 5 bereinigen (1 Zeile):** Stale TODO-Kommentar in `Sidebar.tsx:10` entfernen.
- **Finding 3 kurzfristig:** `dashboard.py`, `licenses.py`, `economics.py`, `emails.py` auf `now_utc_naive()` migrieren (~15 Stellen, ~30 Minuten Aufwand).

---

## 9. Strategische Empfehlungen

- **Vor dem Launch: Deployment-Checkliste abarbeiten (Finding 1).** Die technischen Voraussetzungen sind erfüllt — das Einzige, was den Launch verhindert, sind drei Deployment-Konfigurationsschalter. Empfehlung: `deploy/env/.env.deploy.prod` und `deploy/env/.env.api.prod` finalisieren, `deploy/prod-compose.sh` einmal dry-run auf dem Server testen, dann Smoke-Test-Checkliste ausführen.
- **Frontend-Komponententests ausbauen:** Die 32 Frontend-Tests decken nur Hooks/Libs ab. Die kritischen User Flows (Registrierung, Onboarding, Projektanlage, Lizenz-Checkout) haben keine automatisierten Tests. Für den ersten Produktivbetrieb ist das akzeptabel, mittelfristig sollten zumindest die kritischen Pfade per Playwright/Cypress gesichert werden.
- **datetime.utcnow-Migration abschließen:** Ziel: 0 utcnow-Warnings in `pytest`. `models.py` als letzten Schritt angehen, da dort Änderungen an Spalten-Defaults sorgfältig getestet werden müssen (insb. keine Migration nötig, aber Seed/Test-DB-Verhalten prüfen).
- **T024 als abgeschlossen im Vault markieren:** Die Aufgabenübersicht listet T024 noch als `offen`, obwohl die Commits zeigen, dass Onboarding-Widget, Onboarding-Abschluss-Notification und der has_license/has_project-Bugfix alle implementiert sind. Den Task formal abschließen.

---

## 10. Empfohlene nächste Aktion

**Go-Live-Deployment-Checkliste ausführen (Finding 1):** `deploy/env/.env.deploy.prod` auf `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false` setzen, `deploy/env/.env.api.prod` mit Live-Stripe-Keys und `RECAPTCHA_BYPASS_ENABLED=false` finalisieren, danach `deploy/prod-compose.sh` auf dem Server ausführen und Smoke-Test-Checkliste abarbeiten. Parallel: den 1-Zeilen-Fix für den failing Test (`test_register_requires_captcha_token`) einspielen.

---

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- **Ist `deploy/env/.env.api.prod` bereits mit Live-Stripe-Keys befüllt?** Konnte nicht aus dem Repository verifiziert werden (Datei nicht in Git getrackt, korrekt so).
- **Ist `BREVO_API_KEY` in der Produktiv-Env gesetzt?** In `apps/api/.env` ist er leer (`BREVO_API_KEY=`). Falls auch in Prod leer, funktionieren Registrierungs-E-Mails nicht (Newsletter-Subscribe-Pfad), aber SMTP-basierte Transaktions-E-Mails (Login, Passwort-Reset) sind davon unabhängig.
- **Wurden die stale Git-Tags `v0.1.0` und `v0.2.0` aus Vorbericht Finding 3 bereinigt?** `git describe --tags --abbrev=0` liefert keinen Tag mehr — möglicherweise wurden sie bereits entfernt oder waren nie gepusht.
- **Läuft der SSE-Broker auf dem Produktivserver als Single-Worker?** Der Code enthält `assert_single_process_or_warn()` als Startup-Guard. Wenn Docker mehrere Worker startet, entsteht kein Bug, aber Notifications sind nicht mehr zuverlässig.
- **Stripe Webhook-Signing-Secret in Prod korrekt gesetzt?** `STRIPE_WEBHOOK_SECRET` muss mit dem im Stripe-Dashboard eingetragenen Live-Endpoint-Secret übereinstimmen; ein Mismatch führt zu stillen Webhook-Fehlern (Zahlungen kommen an, aber werden nicht verarbeitet).
- **Hat `deploy/env/.env.api.prod` `APP_ENV=prod` gesetzt?** `main.py:34` prüft `APP_ENV == "prod"` für den `COOKIE_SECURE`-Guard. Falls `APP_ENV` fehlt oder `dev` ist, wird `COOKIE_SECURE` nicht erzwungen.
