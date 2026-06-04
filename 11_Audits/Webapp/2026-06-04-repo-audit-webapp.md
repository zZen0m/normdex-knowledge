# Repo-Audit - Webapp - 2026-06-04

> **Statusaktualisierung (2026-06-04, nach Audit)** — Folgende Findings dieses Berichts wurden im Anschluss behoben:
> - **Finding 1 (Self-Delete löscht org-weit geteilte Projekte hart): behoben.** Der Self-Delete-Pfad überträgt org-gebundene Projekte und deren Berechnungen jetzt an einen verbleibenden Team-Owner/Admin statt sie zu löschen; nur im echten Solo-Team-Fall (kein Empfänger) werden sie gelöscht. Umgesetzt in `apps/api/app/routers/users.py` (neue Helfer `_resolve_org_recipient`, `_reassign_or_delete_owned_data`), neue Tests in `tests/test_user_privacy.py` (16 grün). Commit `b3cc6ae`.
> - **Finding 2 (divergierende Lösch-Strategien): behoben.** Beide Pfade nutzen jetzt EINE Strategie (Anonymisierung) über den gemeinsamen Service `apps/api/app/services/account_deletion.py` (`anonymize_user_account`). Der Admin-Pfad anonymisiert statt hart zu löschen; der 409-Eigentums-Blocker entfällt, da org-gebundene Projekte an einen Team-Owner übertragen werden und keine NOT-NULL-Referenz mehr verwaisen kann. Tests in `tests/test_admin_delete_user.py` auf Anonymisierungs-Semantik umgestellt, gesamte Suite 297 grün.
> - **Finding 3 (Versionsdrift): geklärt — kein Code-Handlungsbedarf.** Version `0.0.1` in `package.json` und `version.py` ist die korrekte aktuelle Version (vom Produktinhaber bestätigt). Die Git-Tags `v0.1.0` und `v0.2.0` wurden für experimentelle Zwecke erstellt und spiegeln keinen tatsächlichen Release-Stand wider; sie sind zu bereinigen (lokal + remote `git tag -d` / `git push origin --delete`). Kein normdex-version-update-Lauf notwendig.
> - **Finding 4 (support_admin.py umgeht require_admin): behoben.** Alle 5 Handler migriert: `get_current_user` + manuelle `is_admin`-Checks entfernt, stattdessen `admin: User = Depends(require_admin)` in jedem Handler. 297 Tests grün.
> - **Finding 7 (Tippfehler „Ur JPG"): behoben.** `apps/api/app/routers/users.py:143` korrigiert auf „Nur JPG, PNG oder WebP erlaubt."
> - **Finding 8 (Frontend-Bundle >500 kB): behoben.** `manualChunks` in `vite.config.ts` eingeführt: vendor-react / vendor-radix / vendor-ui / vendor-forms als eigene Cache-Chunks. `index.js` sinkt von 533 kB auf 105 kB; kein Chunk überschreitet mehr 500 kB; Vite-Warnung entfällt.
> - **Finding 6 (dev.db-Backup-Binärdatei in Git getrackt): behoben.** `apps/api/dev.db.backup-reset-414345-20260430-201116` per `git rm --cached` aus dem Tracking entfernt; Muster `apps/api/dev.db.backup*` in `.gitignore` ergänzt. `dev.db` bleibt bewusst getrackt (Dummy-Daten). Commit `71378ef`.
> - **Finding 9 (unleash-*.json im Migrations-Ordner): behoben.** Beide Dateien per `git rm --cached` aus dem Tracking entfernt; Muster `apps/api/alembic/versions/unleash-*.json` in `.gitignore` ergänzt. Commit `71378ef`.

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-app`
- Branch: `develop`
- Commit: `ea4f6be` (plus 1 nicht committeter Folge-Commit `944c1a1` im Verlauf, Arbeitskopie nur mit `dev.db`-Änderung)
- Version laut `package.json`: `0.0.1`
- Letzter Git-Tag: `v0.2.0`
- Audit-Datum: `2026-06-04`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-06-04-repo-audit-webapp.md`
- Vorheriger Audit: `2026-05-30-repo-audit-webapp.md`
- Audit-Fokus: vollständiger Webapp-Audit mit Schwerpunkt auf den neuen Codepfaden seit 2026-05-30 (Kontolöschung/Anonymisierung, Avatar-/Upload-Retention, Timezone-Handling, Team-Ausbau) sowie Abgleich der offenen Vorbericht-Findings.

## 2. Audit-Abdeckung

- Statische Codeanalyse Backend: `apps/api/app/routers/users.py` (Konto-Löschung/Anonymisierung, Avatar-Upload), `apps/api/app/project_visibility.py`, `apps/api/app/routers/support_admin.py`, `apps/api/app/routers/auth.py` (reCAPTCHA), `apps/api/app/version.py`, `apps/api/app/config.py`.
- Datenbank-/Modellbezug: FK-Aufräumlogik beim Konto-Löschen, Projekt-Sichtbarkeitsregeln (Org-/Team-Scope), Alembic-Versions-Ordner.
- Frontend: `apps/frontend/package.json` (Version, Dependencies), Build-Artefakte (`dist/assets`).
- Versionskonsistenz: `package.json` ↔ `version.py` ↔ Git-Tags ↔ `CHANGELOG.md` ↔ `README.md`.
- Abgleich mit Vorbericht vom 2026-05-30 (Findings 1-8).
- Verifikation ausgeführt:
  - `pytest -q` in `apps/api` (`venv`): **293 passed, 1121 warnings** in ~114 s.
  - `npm run build` in `apps/frontend`: erfolgreich, `assets/index-*.js` = **533,75 kB** (gzip 162,62 kB), Vite-Chunk-Warnung weiterhin aktiv.
  - `npm test` in `apps/frontend`: 4 Testdateien, **32 Tests grün** (Vorbericht: 3 Dateien / 24 Tests).
- Nicht ausgeführt: kein Live-Browser-/E2E-Test, kein realer Stripe-/Brevo-/Graph-Durchlauf, keine PostgreSQL-Laufzeitprüfung (Dev läuft auf SQLite, das FK-Constraints toleranter behandelt), kein Multi-Worker-SSE-Test.

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-05-30
- Findings im Vorbericht: 8
- Davon behoben: 6
- Davon weiterhin offen: 2
- Davon nicht abschließend verifizierbar: 0
- Regressionen: 0 (klassisch), aber ein Tech-Debt-Trend hat sich fortgesetzt (siehe unten)

### Behobene Findings

- Finding 1 - Admin-User-Löschung kaskadiert nicht sauber: **behoben**. `admin.delete_user` räumt FK-Beziehungen explizit auf bzw. blockt mit HTTP 409; Test `test_admin_delete_user.py` vorhanden.
- Finding 2 - Drei Admin-Aktionen UI-only: **behoben**. Backend-Endpoints für Notiz/Billing-/Subscription-Check mit AuditLog vorhanden.
- Finding 4 - SSE-Broker single-process: **behoben (abgesichert)**. `assert_single_process_or_warn()` als Startup-Guard plus Deploy-Doku.
- Finding 5 - Deprecation-Warnungen (Admin-Module): **behoben für Admin-Module**. `app/util/dt.py` + `pytest.ini`-Gate eingeführt; Admin-Router tragen keine `utcnow`-Warnung mehr bei. (Gesamtzahl steigt aber weiter durch nicht-Admin-Module, siehe Finding 5 dieses Berichts.)
- Finding 6 - AdminUsers native `prompt()/confirm()` + falsche Passwortlänge: **behoben**. Dialog-Komponenten, Hinweis auf 8 Zeichen.
- Finding 7 - Refund-Reason hardcodiert: **behoben**. `reason_code`-Literal validiert und an Stripe durchgereicht.

### Weiterhin offene Findings

- Finding 3 - `support_admin.py` umgeht `require_admin`: **weiterhin offen**. Handler injizieren weiter `Depends(get_current_user)` und prüfen manuell `if not user.is_admin` (`support_admin.py:107`, `138`, `221`, `317`, `456`). Keine Migration auf `require_admin`. → siehe Finding 4 dieses Berichts.
- Finding 8 - Frontend-Hauptbundle / Chunk-Warnung: **weiterhin offen**. Build meldet 533,75 kB (vs. 533,56 kB), kein Code-Splitting eingeführt. → siehe Finding 8 dieses Berichts.
- Zusatz aus Vorbericht (Quick Win): `unleash-*.json` in `alembic/versions/` **weiterhin offen** (beide Dateien noch getrackt, kein `.gitignore`-Eintrag). → siehe Finding 9.

### Regressionen

- Keine formale Regression (kein als behoben markiertes Finding ist zurückgekehrt). Tendenz bei den Datetime-Deprecation-Warnungen ist jedoch weiter steigend (972 → 1121), weil neue Module (`users.py`, Scheduler, Support) `datetime.utcnow()` verwenden.

## 4. Confidence

- Confidence: hoch

Alle Aussagen sind direkt aus dem Repository-Code belegbar. Backend-Tests (293 grün), Frontend-Tests (32 grün) und Frontend-Build wurden in diesem Lauf real ausgeführt. Eingeschränkt bleibt die Bewertung des PostgreSQL-Laufzeitverhaltens (Dev = SQLite) und der tatsächlichen Datenfolgen einer produktiven Konto-Löschung im Team-Kontext, weil das eine Produktentscheidung berührt, nicht nur Code.

## 5. Kurzfazit

Der Webapp-Stand hat sich seit 2026-05-30 fachlich klar verbessert: sechs der acht Vorbericht-Findings sind sauber behoben, das Verwaltungsportal ist abgesichert, und mit der Konto-Löschung/Anonymisierung (`users.py`) wurde ein DSGVO-relevanter Flow ergänzt, der buchhaltungsrelevante Daten als Snapshot sichert und personenbezogene Referenzen breit anonymisiert. Die Umsetzung ist überdurchschnittlich sorgfältig (Snapshot, Eligibility-Checks, Token-Bestätigung per E-Mail, FK-Detach). Zwei strukturelle Punkte bleiben offen: `support_admin.py` umgeht weiterhin die zentrale `require_admin`-Dependency, und das Frontend-Hauptbundle stagniert über der 500-kB-Schwelle. Neu und am wichtigsten ist ein Datenverlust-Risiko: Beim Self-Delete eines normalen Team-Mitglieds werden dessen Projekte hart gelöscht, obwohl diese organisationsweit sichtbar/geteilt sind. Zusätzlich fällt eine Versionsdrift auf (App meldet 0.0.1, jüngster Git-Tag ist v0.2.0). Trend insgesamt: **deutliche Verbesserung mit zwei klar benennbaren Restrisiken**.

## 6. Wichtigste Findings

1. Self-Delete eines Team-Mitglieds löscht org-weit sichtbare Projekte hart (Datenverlust-Risiko fürs Team). — **behoben 2026-06-04** (Übertragung an Team-Owner, Commit `b3cc6ae`).
2. Zwei divergierende „User entfernen"-Strategien (Admin = Hard-Delete + 409-Blocker, Self = Anonymisierung + Projektübertragung) ohne dokumentierte Linie. — **behoben 2026-06-04** (gemeinsamer `account_deletion`-Service, beide Pfade anonymisieren).
3. Versionsdrift: Runtime-/Anzeigeversion `0.0.1`, jüngster Git-Tag `v0.2.0`, `CHANGELOG` endet bei 0.0.1.
4. ~~`support_admin.py` nutzt weiterhin manuelle `is_admin`-Checks statt `require_admin` (persistent seit 2026-05-30).~~ **behoben 2026-06-04** (5 Handler auf `Depends(require_admin)` migriert).
5. Datetime-Deprecations weiter gestiegen (972 → 1121); `util/dt.py` außerhalb der Admin-Module nicht adoptiert.
6. `dev.db` und eine `dev.db.backup-*`-Binärdatei sind in Git getrackt (Binär-Churn, PII-/Bloat-Risiko).
7. Tippfehler in nutzersichtbarer Fehlermeldung beim Avatar-Upload („Ur JPG …").
8. Frontend-Hauptbundle 533 kB, Chunk-Warnung aktiv (persistent).
9. `unleash-*.json`-Artefakte weiter in `alembic/versions/` (persistent).

## 7. Detaillierte Findings je Punkt

### Finding 1 - Self-Delete löscht organisationsweit geteilte Projekte hart

- **Status: behoben (2026-06-04, Commit `b3cc6ae`).**
- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert (Datenfolge im Team-Kontext: manuell/produktseitig zu bewerten)
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/users.py:362-363`, `apps/api/app/project_visibility.py:25-43`, `apps/api/app/routers/users.py:246-297` (Block-Logik)
- Evidenz: `_anonymize_user_account` führt `db.query(models.Project).filter(models.Project.user_id == user.id).delete(...)` aus — hart, ohne Org-Check. `build_project_visibility_filter` zeigt dagegen, dass Projekte org-weit sichtbar sind: `models.Project.organization_id.in_(org_ids)` (Zeile 32-33). Die Lösch-Blocker in `_account_delete_block_reason` / `_assert_account_can_be_deleted` greifen nur für Rollen `owner`/`admin` mit weiteren Mitgliedern bzw. den letzten Owner — ein normales Mitglied (`role="member"`) wird nicht geblockt. `tests/test_user_privacy.py:165` bestätigt, dass das Projekt beim Löschen verschwindet (dort allerdings ohne weitere Org-Mitglieder).
- Beschreibung des Problems: Löscht ein reguläres Team-Mitglied sein Konto, werden alle von ihm erstellten Projekte gelöscht — auch solche mit `organization_id`, die für das gesamte Team sichtbar und Teil der gemeinsamen Arbeit sind. Die Anonymisierung schützt zwar die Person, vernichtet aber Teaminhalte, die nach dem Ausscheiden weiterbenötigt werden.
- Warum das relevant ist: Normdex ist B2B/Team-orientiert; Projekte sind die Kernarbeitsobjekte und explizit org-weit geteilt. Ein Self-Service-Löschpfad, der geteilte Inhalte mitnimmt, ist aus Team-Sicht ein stiller Datenverlust.
- Business Impact: Verlust geteilter Berechnungs-/Projektdaten beim Ausscheiden einzelner Nutzer; potenzielle Support-Eskalationen und Vertrauensverlust bei Bestandskunden. Konflikt zwischen DSGVO-Löschpflicht (Person) und legitimem Geschäftsinteresse an Teamdaten (Organisation).
- Konkrete Handlungsempfehlung: Produktentscheidung treffen und im Code abbilden: org-gebundene Projekte (`organization_id IS NOT NULL`) beim Self-Delete nicht hart löschen, sondern an die Organisation/den Owner übertragen (`user_id` umhängen oder auf `updated_by=None`-Muster anonymisieren) und nur rein private Projekte (`organization_id IS NULL`) löschen. Falls das aktuelle Verhalten gewollt ist, mindestens im Lösch-Dialog explizit warnen („X geteilte Projekte werden gelöscht") und einen Test mit echtem Mehr-Mitglieder-Org-Szenario ergänzen.
- **Behebung (2026-06-04):** Produktentscheidung getroffen — Projekte werden übertragen, nicht gelöscht. `_anonymize_user_account` ruft jetzt `_reassign_or_delete_owned_data` auf: Projekte (und deren Berechnungen) eines ausscheidenden Mitglieds gehen an einen verbleibenden Owner/Admin der jeweiligen Organisation (`_resolve_org_recipient`, älteste Mitgliedschaft unter `TEAM_ADMIN_ROLES`, Fallback anderes Mitglied). Legacy-Projekte ohne `organization_id` folgen dem Empfänger der ersten Org-Mitgliedschaft. Nur wenn kein Empfänger existiert (Solo-Team), werden Projekte + Berechnungen gelöscht — DSGVO-konform, durch die bestehende „letzter Owner"-Block-Logik abgesichert. Keine DB-Migration nötig (User-Record bleibt anonymisiert erhalten, `ondelete="CASCADE"` feuert nicht). Neue Tests in `tests/test_user_privacy.py` (Übertragung, Solo-Team-Löschung, Legacy-Projekt, `updated_by`-Null), alle 16 grün. Vault-Doku: `06_Entwicklung/Kontoloeschung – Projekte an Team-Owner uebertragen.md`.

### Finding 2 - Zwei divergierende Strategien zum Entfernen eines Users

- **Status: behoben (2026-06-04).**
- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/users.py:356-450` (Self-Delete = Anonymisierung), `apps/api/app/routers/admin.py:1007-1078` (Admin-Delete = Hard-Delete + 409)
- Evidenz: Admin-Pfad löscht den User-Record hart (`db.delete(u)`) und blockt mit HTTP 409, wenn der User noch `Project`/`LicenseOrder` besitzt. Self-Pfad behält den User-Record (anonymisiert Felder, `is_active=False`) und sichert Billing per Snapshot in `LicenseOrder.meta`.
- Behebung (2026-06-04): Die Logik wurde in den gemeinsamen Service `apps/api/app/services/account_deletion.py` (`anonymize_user_account` + Helfer) extrahiert, den **beide** Pfade aufrufen. Der Admin-Pfad (`admin.delete_user`) anonymisiert jetzt ebenfalls statt hart zu löschen; der 409-Eigentums-Blocker (Projekte/Bestellungen) entfällt, weil org-gebundene Projekte an einen verbleibenden Team-Owner übertragen werden und der User-Record anonymisiert erhalten bleibt (keine verwaisende NOT-NULL-Referenz). Damit existiert eine einzige, dokumentierte Löschstrategie. Self-Pfad ruft `anonymize_user_account` ebenso auf; die Eligibility-/Block-Logik des Self-Pfads (letzter Owner etc.) bleibt unverändert. Tests: `tests/test_admin_delete_user.py` auf Anonymisierungs-Semantik umgestellt, `tests/test_user_privacy.py`/`test_avatar_delete.py` angepasst, gesamte Suite **297 grün**.
- Beschreibung des Problems: Für denselben fachlichen Vorgang („einen Nutzer entfernen") existieren zwei unterschiedliche Datenmodelle: einmal bleibt ein anonymisierter Geist-Record samt Projektlöschung, einmal ein echtes Hard-Delete mit Eigentums-Blocker. Das erschwert Nachvollziehbarkeit, Reporting und zukünftige Wartung.
- Warum das relevant ist: Inkonsistente Löschsemantik führt zu schwer testbaren Sonderfällen und potenziell widersprüchlichem DSGVO-Verhalten (anonymisierter Record vs. vollständige Entfernung).
- Business Impact: Mittelfristige Wartungskosten, Risiko inkonsistenter Daten bei Audits/Behördenanfragen.
- Konkrete Handlungsempfehlung: Eine gemeinsame Löschstrategie definieren (vorzugsweise Anonymisierung für beide Pfade, da FK-schonender) und in einem geteilten Service (`app/services/account_deletion.py`) kapseln, den sowohl der Self- als auch der Admin-Pfad aufrufen. Dokumentieren, welcher Pfad welche Daten entfernt/erhält.

### Finding 3 - Versionsdrift zwischen App-Version und Git-Tag

- **Status: geklärt (2026-06-04) — kein Code-Handlungsbedarf; Git-Tags zu bereinigen.**
- Kategorie: Improvement
- Priorität: mittel → entfällt nach Klärung
- Verifizierungsstatus: statisch verifiziert, produktseitig geklärt
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/frontend/package.json:4` (`"version": "0.0.1"`), `apps/api/app/version.py:6` (`DEFAULT_APP_VERSION = "0.0.1"`), Git-Tags `v0.1.0`, `v0.2.0`, `CHANGELOG.md` (endet bei `[0.0.1]`)
- Klärung (2026-06-04): Version `0.0.1` ist die korrekte und aktuelle App-Version (vom Produktinhaber bestätigt). Die Git-Tags `v0.1.0` und `v0.2.0` wurden für experimentelle Zwecke angelegt und repräsentieren keine tatsächlichen Releases. `package.json`, `version.py` und `CHANGELOG.md` sind korrekt und konsistent — kein `normdex-version-update`-Lauf notwendig. Handlungsbedarf: Stale Tags lokal und auf dem Remote bereinigen (`git tag -d v0.1.0 v0.2.0` + `git push origin --delete v0.1.0 v0.2.0`).

### Finding 4 - support_admin.py umgeht weiterhin require_admin

- **Status: behoben (2026-06-04).**
- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert, behoben
- Kontinuität: persistent (seit 2026-05-30), behoben
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/support_admin.py:107`, `138`, `221`, `317`, `456`
- Evidenz: Jeder Handler injiziert `user: User = Depends(get_current_user)` und prüft manuell `if not user.is_admin`. An `317` und `456` zusätzlich als Einzeiler `if not user.is_admin: raise HTTPException(status_code=403)` ohne `detail`. `admin.py` verwendet dagegen durchgängig `Depends(require_admin)`.
- Beschreibung des Problems: Zwei parallele Auth-Muster für denselben Zweck. Jeder neue Support-Endpoint muss den Check manuell mitführen; ein vergessener Check öffnet Support-Tickets (kundenseitige E-Mails, Anhänge, Adressdaten) für jeden eingeloggten Nutzer.
- Warum das relevant ist: Latentes Datenschutz-/Compliance-Risiko, das mit jedem neuen Support-Endpoint wächst.
- Business Impact: Risiko eines Datenschutzvorfalls bei einem einzelnen vergessenen Check; erhöhter Review-Aufwand.
- Konkrete Handlungsempfehlung: Auf `admin: User = Depends(require_admin)` migrieren, manuelle Checks entfernen, und einen Test ergänzen, der für alle Routen unter `/admin/support/*` einen 403 für Nicht-Admins erzwingt.
- **Behebung (2026-06-04):** Alle 5 Handler in `support_admin.py` migriert: `from app.deps import get_current_user` → `require_admin`; Parameter `user: User = Depends(get_current_user)` → `admin: User = Depends(require_admin)`; manuelle `if not user.is_admin`-Checks entfernt; alle `user.id`-Referenzen in Nachrichten-Konstruktoren auf `admin.id` umgestellt. 297 Tests grün.

### Finding 5 - Datetime-Deprecations weiter gestiegen (972 → 1121) ✅ behoben 2026-06-04

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (Trend seit 2026-05-23/30)
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/users.py:173`, `350`, `360`, `479`, `498`; `apps/api/app/services/scheduler.py` (8×), `apps/api/app/routers/licenses_v2.py` (17×), `apps/api/app/routers/auth.py` (15×) u. a.
- Evidenz: `pytest -q` meldet **1121 warnings** (Vorbericht-Endstand 972). Die neuen Codepfade (Konto-Löschung in `users.py`, Avatar-Timestamp) nutzen weiterhin naives `datetime.utcnow()`. `app/util/dt.py` (`now_utc`, `now_utc_naive`) ist vorhanden, aber außerhalb der Admin-Module nicht adoptiert.
- Beschreibung des Problems: Das im Vorbericht für Admin-Module gelöste Tech-Debt-Thema wächst in den übrigen Modulen weiter. Naive UTC-Zeitstempel bergen Vergleichsrisiken gegenüber timezone-aware Werten (Stripe, `Europe/Vienna`-Konvertierungen) und maskieren relevantere Warnungen.
- Warum das relevant ist: Erschwert Python-/Pydantic-Upgrades, verschleiert echte Warnungen, latente Zeitvergleichs-Bugs.
- Business Impact: Wartungs-/Upgrade-Risiko, potenzielle Zeitlogik-Fehler bei Lizenz-/Token-Ablauf.
- Konkrete Handlungsempfehlung: `now_utc_naive()`/`now_utc()` schrittweise in `users.py`, `scheduler.py`, `licenses_v2.py`, `auth.py` ausrollen und den `filterwarnings`-Gate in `pytest.ini` modulweise erweitern, sobald ein Modul migriert ist.
- **Behebung (2026-06-04):** Alle `datetime.utcnow()`-Aufrufe in `users.py` (3×), `scheduler.py` (8×), `licenses_v2.py` (17×) und `auth.py` (15×) auf `now_utc_naive()` aus `app.util.dt` migriert. `pytest.ini`-Gate auf alle 4 Module erweitert. Warnings 1121 → 1068; 297 Tests grün.

### Finding 6 - dev.db und dev.db-Backup-Binärdatei in Git getrackt ✅ behoben 2026-06-04

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: behoben
- Kontinuität: neu (explizit benannt)
- Betroffene Datei(en) oder Pfade: `apps/api/dev.db`, `apps/api/dev.db.backup-reset-414345-20260430-201116`, `.gitignore:14-16`
- Evidenz: `git ls-files` listet `dev.db` und `dev.db.backup-reset-414345-20260430-201116` als getrackt. In dieser Session hat sich `dev.db` allein durch das Test-/Devverhalten von 3.149.824 auf 3.248.128 Bytes geändert (Arbeitskopie). `.gitignore` ignoriert zwar `db_backups/*`, aber die im Repo-Root-Verzeichnis liegende `dev.db.backup-*` ist davon nicht erfasst.
- Beschreibung des Problems: Eine binäre SQLite-DB plus ein Backup-Snapshot liegen versioniert im Repo. `dev.db` als bewusste Demo-Fixture ist laut `.gitignore`-Kommentar gewollt, die `dev.db.backup-*`-Datei wirkt jedoch wie ein versehentlich eingecheckter Reset-Snapshot.
- Warum das relevant ist: Binärdateien blähen die Git-Historie auf (jede DB-Änderung erzeugt einen neuen Blob), erzeugen Merge-Konflikte und können personenbezogene Demo-/Testdaten dauerhaft in der Historie konservieren.
- Business Impact: Repo-Bloat, ständige „dirty"-Working-Trees (siehe Eingangs-Git-Status), potenzielles PII-in-Historie-Risiko.
- Konkrete Handlungsempfehlung: `dev.db.backup-*` aus dem Tracking entfernen (`git rm --cached`) und per `.gitignore`-Muster (`apps/api/dev.db.backup-*`) ausschließen. Für `dev.db` prüfen, ob ein deterministischer Seed-/Fixture-Mechanismus die getrackte Binärdatei ersetzen kann, damit Devs keine Binär-Diffs mehr committen müssen.

### Finding 7 - Tippfehler in nutzersichtbarer Avatar-Fehlermeldung

- **Status: behoben (2026-06-04).**
- Kategorie: Bug
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/users.py:143`
- Evidenz: `raise HTTPException(status_code=400, detail="Ur JPG, PNG oder WebP erlaubt.")` — „Ur" statt „Nur".
- Beschreibung des Problems: Die Fehlermeldung bei falschem Avatar-Dateityp ist orthografisch fehlerhaft und wird dem Endnutzer direkt angezeigt.
- Warum das relevant ist: Sichtbares Qualitätssignal im Profilbereich.
- Business Impact: Sehr niedrig (Vertrauens-/Politur-Aspekt).
- Konkrete Handlungsempfehlung: Auf „Nur JPG, PNG oder WebP erlaubt." korrigieren.
- **Behebung (2026-06-04):** `detail`-String in `users.py:143` von „Ur JPG …" auf „Nur JPG, PNG oder WebP erlaubt." korrigiert.

### Finding 8 - Frontend-Hauptbundle über 500 kB, Chunk-Warnung aktiv

- **Status: behoben (2026-06-04).**
- Kategorie: Optimization
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-23)
- Betroffene Datei(en) oder Pfade: `apps/frontend/dist/assets/index-*.js`, `apps/frontend/vite.config.ts`
- Evidenz: `npm run build` meldet `index-*.js` = 533,75 kB (gzip 162,62 kB) und „Some chunks are larger than 500 kB after minification". Kein `manualChunks`/`lazy()`-Splitting eingeführt.
- Beschreibung des Problems: Das App-Chrome (Sidebar, Notifications, Bell-Popover) liegt vollständig im Eintritts-Bundle und wächst kontinuierlich.
- Warum das relevant ist: Kumulativ steigende Initial-Ladezeit für jede geschützte Route.
- Business Impact: Niedrig, aber kumulativ; spürbar auf langsamen Verbindungen.
- Konkrete Handlungsempfehlung: Route-basiertes Code-Splitting (`React.lazy` für Admin-/Report-Seiten), `recharts`/`html2pdf.js` dynamisch laden, oder bewusst `build.chunkSizeWarningLimit` setzen und als CI-Budget-Gate führen.
- **Behebung (2026-06-04):** `manualChunks` in `apps/frontend/vite.config.ts` eingeführt: vendor-react (react/react-dom/react-router-dom → 164 kB), vendor-radix (alle @radix-ui/* → 115 kB), vendor-ui (lucide-react/sonner/date-fns/clsx → 105 kB), vendor-forms (react-hook-form/zod/@hookform → 84 kB) als eigenständige Cache-Chunks. Hauptbundle `index.js` sinkt von 533 kB auf **105 kB** (−80 %); kein Chunk überschreitet 500 kB; Vite-Chunk-Warnung entfällt; Build grün.

### Finding 9 - unleash-*.json-Artefakte im Migrations-Ordner

- **Status: behoben (2026-06-04).**
- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-23)
- Betroffene Datei(en) oder Pfade: `apps/api/alembic/versions/unleash-backup-codeium-extension.json`, `apps/api/alembic/versions/unleash-repo-schema-v1-codeium-language-server.json`
- Evidenz: Beide Dateien weiterhin von `git ls-files` gelistet, liegen im Alembic-Versions-Ordner, kein `.gitignore`-Eintrag.
- Beschreibung des Problems: Fremd-Artefakte (Codeium/Unleash) im Migrations-Ordner verschmutzen den Verzeichniskontext, in dem Alembic Revisionen sucht.
- Warum das relevant ist: Verwirrt bei Migrations-Reviews; potenziell irritierend für Tools, die das Verzeichnis scannen.
- Business Impact: Sehr niedrig.
- Konkrete Handlungsempfehlung: Beide Dateien aus `alembic/versions/` entfernen (`git rm`) und ein `.gitignore`-Muster für `unleash-*.json` ergänzen.
- **Behebung (2026-06-04):** Beide Dateien per `git rm --cached` aus dem Tracking entfernt. Muster `apps/api/alembic/versions/unleash-*.json` in `.gitignore` ergänzt; Dateien bleiben lokal erhalten, werden aber nicht mehr getrackt.

## 8. Quick Wins

- Tippfehler „Ur JPG …" → „Nur JPG …" in `users.py:138` korrigieren (Finding 7).
- `support_admin.py` auf `Depends(require_admin)` umstellen und manuelle `is_admin`-Checks entfernen (Finding 4).
- `unleash-*.json` aus `alembic/versions/` entfernen und ignorieren (Finding 9).
- ~~`dev.db.backup-*` per `git rm --cached` aus dem Tracking nehmen und `.gitignore` ergänzen (Finding 6).~~ **Erledigt 2026-06-04**
- `normdex-version-update`-Skill ausführen, um `package.json`/`version.py`/CHANGELOG mit Tag `v0.2.0` zu synchronisieren (Finding 3).

## 9. Strategische Empfehlungen

- ~~Konto-Löschung in einen gemeinsamen `account_deletion`-Service zusammenführen, den Self- und Admin-Pfad teilen, und dabei die org-gebundenen Projekte (Finding 1) bewusst behandeln (Übertragen statt Löschen).~~ **Erledigt 2026-06-04** (`app/services/account_deletion.py`, beide Pfade anonymisieren, Projekte werden übertragen).
- Frontend-Komponententests für die seit Wochen wachsenden kritischen Flows aufbauen — insbesondere die neue Konto-Löschung (Eligibility-Anzeige, Confirm-Flow) und das Verwaltungsportal; aktuell decken 32 Tests vor allem Hooks/Libs ab, keine Admin- oder Account-Delete-Seiten.
- Datetime-Migration auf `util/dt.py` als projektweites Ziel verfolgen und modulweise per `pytest.ini`-Gate absichern, bis `datetime.utcnow()` repoweit eliminiert ist.
- Ein deterministischer DB-Seed statt der getrackten `dev.db`-Binärdatei würde Binär-Churn und PII-in-Historie-Risiken dauerhaft beseitigen.
- Bundle-Budget als CI-Gate etablieren, bevor weitere große Features (Charts, PDF) das Eintritts-Bundle weiter aufblähen.

## 10. Empfohlene nächste Aktion

~~Vor dem nächsten Release die **Konto-Löschung im Team-Kontext klären (Finding 1)** — entweder org-gebundene Projekte beim Self-Delete übertragen statt löschen, oder den Nutzer im Lösch-Dialog explizit über den Verlust geteilter Projekte warnen.~~ **Erledigt 2026-06-04** (Übertragung an Team-Owner, Commit `b3cc6ae`). Verbleibend: den Quick-Win-Block (Versions-Sync, `support_admin.py`-Auth, Tippfehler) abarbeiten und mittelfristig die divergierenden Lösch-Strategien (Finding 2) in einem gemeinsamen Service zusammenführen.

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- ~~Ist das harte Löschen org-gebundener Projekte beim Self-Delete eine bewusste Produktentscheidung oder ein Nebeneffekt?~~ **Geklärt 2026-06-04:** Produktentscheidung getroffen — org-gebundene Projekte werden an einen Team-Owner übertragen statt gelöscht (Finding 1 behoben).
- Welche FK-Constraints die Produktiv-PostgreSQL tatsächlich mit `ON DELETE` deklariert — die Anonymisierung (Self) behält den User-Record, das umgeht das Problem; der Admin-Hard-Delete-Pfad bleibt aber von der echten Schema-Definition abhängig.
- Ob `RECAPTCHA_BYPASS_ENABLED` in der Produktivumgebung garantiert `False` ist (Default ist sicher `False`, aber die Env-Konfiguration konnte hier nicht eingesehen werden).
- Ob der deaktivierte Self-Service-Export (HTTP 410 in `users.py:229-243`) DSGVO-Art.-20-Auskunftspflichten über den Support-Prozess vollständig abdeckt.
- Ob der jüngste Tag `v0.2.0` tatsächlich den aktuellen `develop`-Stand markiert oder ob Tag und Branch auseinanderlaufen.
- Ob für die Konto-Löschung ein dokumentierter Smoke-Test (Start → E-Mail → Confirm → Anonymisierung) gegen ein reales Postfach existiert.
