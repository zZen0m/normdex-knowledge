# Repo-Audit - Webapp - 2026-06-20

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-app` (GitHub-Remote: `zZen0m/normdex-app`)
- Branch: `develop`
- Commit: `c92f7c8`
- Version laut `package.json`: `0.1.0`
- Letzter Git-Tag: kein lokaler Git-Tag vorhanden
- Audit-Datum: `2026-06-20`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Webapp\2026-06-20-repo-audit-webapp.md`
- Vorheriger Audit: `2026-06-13-repo-audit-webapp.md`
- Git-Status zum Audit-Zeitpunkt: `apps/api/dev.db` bereits vor Audit-Beginn modifiziert; `apps/api/preview_report_demo.py` bereits vor Audit-Beginn unversioniert. Beide Nutzeränderungen wurden nicht verändert.

## 2. Audit-Abdeckung

- Normdex-Vault-Kontext, Aufgabenübersicht, T026, T020-16, T013 und `docs/features/FEATURES.md` gelesen
- Abgleich mit dem Vorbericht vom 2026-06-13
- Statische Prüfung von FastAPI-Routen, Authentifizierung, Rollen/Rechten, Mandantentrennung, Support, Admin-, Lizenz-, Stripe- und Webhook-Flows
- Frontend-Prüfung von Routing, geschützten Routen, API-Client, Support-HTML, Build-Konfiguration und Abhängigkeiten
- Deployment-Prüfung von Dockerfiles, Compose-Dateien, BasicAuth-Umschaltung, Umgebungsbeispielen und GitHub-Workflows
- Datenbank-/Migrationsprüfung ohne Alembic-Ausführung: 40 Migrationen, genau ein Head (`ffd3bbde6b6a`), keine fehlenden Vorgänger; `dev.db` steht ebenfalls auf `ffd3bbde6b6a`
- Backend-Gesamtsuite in einer frisch erzeugten, isolierten Umgebung mit den deklarierten Requirements: **314 Tests grün, 475 Warnungen**
- Frontend nach `npm ci` mit Vite `8.0.16` und Vitest `4.1.8`: **32 Tests grün**
- Standard-Produktionsbuild `npm run build`: **fehlgeschlagen** mit `TypeError: manualChunks is not a function`
- Kontrollbuild `npx tsc -b --force && npx vite build`: erfolgreich; dadurch ist die Fehlerursache auf die veraltete generierte Vite-Konfiguration eingegrenzt
- Dependency-Scans: `pip-audit` **0 bekannte Schwachstellen**; `npm audit` **1 moderate Schwachstelle** in DOMPurify
- Bandit: keine High-Severity-Funde; zwei kontextabhängige Medium-Hinweise im VIES/XML-Code
- Dev-Fixture-Prüfung mit `apps/api/scripts/verify_dev_fixture.py`
- Secret-Dateipfad-Prüfung erfolgreich; vollständiger Gitleaks-History-Scan lokal nicht möglich, da Gitleaks nicht installiert ist
- Nicht ausgeführt: Docker-Build, PostgreSQL-Laufzeitprüfung, echter Stripe-/Brevo-/SMTP-Test, Browser-E2E-Test, aktiver Penetrationstest

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-06-13
- Findings im Vorbericht: 10
- Davon behoben: 4
- Davon weiterhin offen: 5
- Davon nicht abschließend verifizierbar: 0
- Regressionen: 1

### Behobene Findings

- Finding 2 - Einladungstoken nicht an E-Mail gebunden: weiterhin behoben; serverseitiger Vergleich, Transaktion und Sicherheitstests sind vorhanden.
- Finding 3 - Support-Anhänge öffentlich abrufbar: weiterhin behoben; Download erfolgt über den admin-geschützten Endpunkt mit Pfadprüfung.
- Finding 4 - Refund nicht eindeutig zugeordnet: weiterhin behoben; die aktuelle Admin-Logik bindet Refunds an eindeutige Stripe-Zahlungsobjekte und ist getestet.
- Finding 5 - Backend-Test-Collection fehlerhaft: weiterhin behoben; die frische Gesamtsuite läuft mit 314 Tests vollständig grün.

### Weiterhin offene Findings

- Finding 1 - Historische Zugangsdaten: weiterhin offen; T026 ist in Arbeit und `.gitleaksignore` enthält weiterhin neun historische Baseline-Einträge.
- Finding 7 - Account-Enumeration: weiterhin offen; Verifikations- und Passwort-Reset-Endpunkte liefern unterscheidbare Zustände.
- Finding 8 - Brevo-Secret im Query-Parameter: weiterhin offen.
- Finding 9 - Dev-Fixture-Governance: weiterhin offen; das Prüftool meldet reale beziehungsweise nicht reservierte Domains und Asset-Probleme.
- Finding 10 - Veralteter Notifications-TODO: weiterhin offen.

### Regressionen

- Finding 6 - Abhängigkeiten und Build: Der Vorbericht meldete scanner-saubere Abhängigkeiten und einen erfolgreichen frischen Frontend-Build. Aktuell meldet `npm audit` wieder eine Schwachstelle; zusätzlich scheitert der Standardbuild mit den deklarierten Vite-8-Abhängigkeiten.

## 4. Confidence

- Confidence: **hoch**

Die drei wichtigsten neuen Bugs sind direkt aus dem aktuellen Code und durch reproduzierbare Prüfungen belegt. Backend-Requirements, vollständige Tests, Migrationsgraph, Frontend-Installation, Tests und Build wurden frisch geprüft. Einschränkungen bestehen bei Docker/Produktion, externen Diensten, der Gültigkeit historischer Secrets und dem fehlenden vollständigen lokalen Gitleaks-Lauf.

## 5. Kurzfazit

Das Backend ist funktional gut abgesichert: 314 Tests laufen auch mit den deklarierten aktualisierten Abhängigkeiten grün, `pip-audit` ist sauber und der Migrationsgraph ist konsistent. Der aktuelle Branch ist trotzdem nicht releasefähig, weil der reguläre Frontend-Produktionsbuild mit Vite 8 fehlschlägt. Zusätzlich bestehen zwei hohe Security-Bugs: Der Block-Link einer E-Mail-Änderung verhindert die Änderung nicht, und clientgesteuerte Attachment-Pfade können den Retention-Job zu Dateilöschungen außerhalb des Upload-Verzeichnisses veranlassen. Der fehlende Test-/Build-CI-Workflow hat diese Regression nicht abgefangen. Gegenüber dem Vorbericht ist der Trend daher gemischt: mehrere frühere Findings bleiben behoben, aber Build- und Dependency-Qualität sind wieder zurückgefallen.

## 6. Wichtigste Findings

1. Der Standard-Frontend-Build scheitert mit den deklarierten Vite-8-Abhängigkeiten. - Bug, **hoch**, Regression
2. Der Block-Link für E-Mail-Änderungen deaktiviert den Bestätigungs-Token nicht. - Bug, **hoch**, neu
3. Clientgesteuerte Support-Attachment-Pfade ermöglichen verzögerte Pfadtraversal-Löschungen und ungebundene Uploads füllen dauerhaft den Datenträger. - Bug, **hoch**, neu
4. Historische SMTP-, Stripe- und Microsoft-Secrets bleiben in aktiven Historien referenziert. - Risk, **hoch**, persistent
5. Es gibt kein CI-Gate für Backend-Tests, Frontend-Tests, Build oder Dependency-Scans. - Risk, **hoch**, neu
6. DOMPurify ist wieder mit bekannten Schwachstellen im Lockfile enthalten. - Risk, **mittel**, Regression
7. Auth-Endpunkte ermöglichen weiterhin Account-Enumeration. - Risk, **mittel**, persistent
8. Das Brevo-Webhook-Secret bleibt in URL und potenziellen Logs sichtbar. - Risk, **mittel**, persistent
9. Die versionierte Dev-Fixture enthält weiterhin real wirkende Daten und inkonsistente Assets. - Risk, **mittel**, persistent
10. Der temporäre Traefik-BasicAuth-Hash ist fest im Compose-File versioniert. - Risk, **mittel**, neu
11. Versions- und Projektdokumentation sind nicht konsistent mit dem tatsächlichen Stand. - Improvement, **niedrig**, neu
12. Der Notifications-TODO in der Sidebar ist weiterhin veraltet. - Improvement, **niedrig**, persistent

## 7. Detaillierte Findings je Punkt

### Finding 1 - Standard-Frontend-Build scheitert mit Vite 8

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: regression
- Betroffene Datei(en) oder Pfade: `apps/frontend/package.json`, `apps/frontend/package-lock.json`, `apps/frontend/vite.config.ts`, `apps/frontend/vite.config.js`, `apps/frontend/vite.config.d.ts`, `apps/frontend/tsconfig.node.json`, `apps/frontend/Dockerfile`
- Evidenz: Nach `npm ci` werden Vite `8.0.16` und Vitest `4.1.8` installiert. `npm run build` scheitert reproduzierbar mit `Invalid output options ... manualChunks ... Expected Function but received Object` und `TypeError: manualChunks is not a function`. `vite.config.ts` enthält bereits die korrekte Funktion, die mitversionierte `vite.config.js` aber noch das alte Objekt. Weil `tsc -b` inkrementell arbeitet, wird die veraltete JS-Datei nicht zuverlässig neu erzeugt. `npx tsc -b --force && npx vite build` ist erfolgreich.
- Beschreibung des Problems: TypeScript-Quellkonfiguration und versioniertes Build-Artefakt sind auseinander gelaufen. Ob der Build funktioniert, hängt vom lokalen `tsconfig.node.tsbuildinfo` und damit vom Zustand des Build-Verzeichnisses ab.
- Warum das relevant ist: Der Standardbefehl aus `package.json` und Dockerfile ist nicht reproduzierbar. Ein Deployment kann je nach vorhandenem Build-Cache abbrechen.
- Business Impact: Release-Blocker, verzögerte Deployments und nicht deterministische Produktionsimages.
- Konkrete Handlungsempfehlung: Generierte `vite.config.js` und `vite.config.d.ts` nicht mehr versionieren, `tsconfig.node.json` auf `noEmit: true` umstellen oder Vite explizit mit `vite.config.ts` starten. `tsconfig.*.tsbuildinfo` zusätzlich in `.dockerignore` aufnehmen und den Standardbuild in CI aus einem frischen Checkout prüfen.

### Finding 2 - E-Mail-Änderung lässt sich nicht wirksam blockieren

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/auth.py:573-662`, `apps/api/app/models.py:112-124`, `apps/api/tests/`
- Evidenz: `/auth/email-change/start` erzeugt zwei unabhängige Tokens: `email_change_confirm_new` und `email_change_block_old`. `/auth/email-change/block` setzt nur `used_at` am Block-Token. `/auth/email-change/confirm` prüft ausschließlich den separaten Bestätigungs-Token und kennt keinen Blockzustand. Tests für den E-Mail-Änderungsflow fehlen.
- Beschreibung des Problems: Selbst nachdem der bisherige Kontoinhaber den Sicherheitslink „Änderung blockieren“ verwendet hat, kann der Inhaber des Bestätigungslinks die neue E-Mail-Adresse weiterhin setzen.
- Warum das relevant ist: Der Flow suggeriert eine Schutzmaßnahme, die technisch keine Wirkung auf die eigentliche Änderung hat.
- Business Impact: Fortgesetzte Kontoübernahme trotz rechtzeitiger Reaktion des Kontoinhabers; Zugriff auf Projekte, Team-, Lizenz- und Rechnungsdaten.
- Konkrete Handlungsempfehlung: Beide Tokens über eine gemeinsame Transaktions-ID koppeln. Der Block-Endpunkt muss alle zugehörigen Bestätigungs-Tokens atomar invalidieren; der Confirm-Endpunkt muss einen Blockzustand prüfen. Tests für Block-vor-Confirm, Confirm-vor-Block, Replay und parallele Requests ergänzen.

### Finding 3 - Attachment-Pfade sind clientgesteuert und der Retention-Job löscht ungeprüft

- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/support.py:26-36`, `apps/api/app/routers/support.py:123-256`, `apps/api/app/services/scheduler.py:199-246`, `apps/api/tests/test_support_upload.py`, `apps/api/tests/test_support_attachment_retention.py`
- Evidenz: Der Upload-Endpunkt liefert einen Pfad wie `attachments/<uuid>.pdf`. Beim Erstellen des Tickets wird `attachments[].path` jedoch ungeprüft aus dem Request als `storage_key` gespeichert. Der Retention-Job verwendet später `os.path.join("uploads", att.storage_key)` und `os.remove(...)` ohne die sichere Pfadprüfung des Download-Endpunkts. Ein Wert wie `../app/main.py` verlässt das Attachment-Verzeichnis. Außerdem ist `/support/upload` nicht rate-limitiert und speichert Dateien bereits vor der Ticketanlage; nicht verwendete Uploads besitzen keinen DB-Datensatz und werden vom Retention-Job nie erfasst.
- Beschreibung des Problems: Ein authentifizierter Nutzer kann manipulierte Storage-Keys hinterlegen. Nach Schließung und Ablauf der Aufbewahrungsfrist kann der Job Dateien außerhalb von `uploads/attachments` löschen. Ungebundene Uploads können den persistenten Upload-Datenträger zusätzlich unbegrenzt füllen.
- Warum das relevant ist: Clientdaten dürfen nie als vertrauenswürdige Dateisystempfade verwendet werden.
- Business Impact: Verfügbarkeits- und Betriebsrisiko, beschädigte Containerdateien, wachsender Storage-Verbrauch und manueller Bereinigungsaufwand.
- Konkrete Handlungsempfehlung: Uploads serverseitig als eigene temporäre DB-Objekte mit Besitzer, Größe, Status und Ablaufzeit erfassen; an das Frontend nur eine opaque Upload-ID geben. Beim Ticket atomar Eigentum und Status prüfen. Einen gemeinsamen sicheren Resolver für Download und Löschung verwenden, Upload-Quota/Rate-Limit ergänzen und verwaiste Uploads nach kurzer TTL löschen. Tests für Traversal, fremde Upload-IDs und Orphan-Cleanup ergänzen.

### Finding 4 - Historische Zugangsdaten noch nicht vollständig bereinigt

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: persistent (seit 2026-06-13)
- Betroffene Datei(en) oder Pfade: `.gitleaksignore`, `.github/workflows/secret-scan.yml`, `D:\Normdex\02_knowledge\normdex-vault\10_Aufgaben\offene Todos\T026-secret-rotation-und-history-cleanup.md`
- Evidenz: `.gitleaksignore` enthält weiterhin neun historische Fingerprints. T026 steht auf `in Arbeit`; Provider-Rotation, Logprüfung, Remote-History-Rewrite und Entfernung der Baseline sind nicht als erledigt markiert. Die zusätzliche Pfadprüfung für `.env`- und Backup-Dateien besteht und läuft erfolgreich.
- Beschreibung des Problems: Repo-seitige Prävention ist vorhanden, aber potenziell offengelegte historische SMTP-, Stripe- und Microsoft-Zugangsdaten sind noch nicht vollständig rotiert und aus aktiven Remote-Historien entfernt.
- Warum das relevant ist: Ein historisches Secret muss bis zur bestätigten Rotation als kompromittiert behandelt werden.
- Business Impact: Zahlungs-, E-Mail-, Datenschutz- und Reputationsrisiko.
- Konkrete Handlungsempfehlung: T026 vor dem nächsten Release vollständig abschließen: Provider-Secrets rotieren, Logs prüfen, Remote-Historie koordiniert bereinigen, Klone neu synchronisieren und Gitleaks ohne Baseline ausführen.

### Finding 5 - Keine CI-Gates für Tests, Build und Dependency-Scans

- Kategorie: Risk
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `.github/workflows/secret-scan.yml`, `.github/workflows/`
- Evidenz: Im Repository existiert nur der Workflow `secret-scan.yml`. Backend-Tests, Frontend-Tests, `npm run build`, `npm audit`, `pip-audit`, Migrationsgraph und Fixture-Prüfung sind keine automatischen Branch-Gates. Der aktuelle Vite-8-Buildfehler wurde dadurch auf `develop` übernommen.
- Beschreibung des Problems: Der Branch-Workflow `develop → dev-server → main` hat keine automatisierte technische Mindestfreigabe.
- Warum das relevant ist: Manuelle lokale Umgebungen können veraltet sein und erzeugen falsche grüne Signale, wie der erfolgreiche Vite-7-Build in diesem Audit zeigte.
- Business Impact: Höheres Release-Risiko, spätere Fehlererkennung und unnötige Deployment-Unterbrechungen.
- Konkrete Handlungsempfehlung: Einen verpflichtenden CI-Workflow mit frischer Installation aufsetzen: Backend `pytest` und `pip-audit`, Frontend `npm ci`, Tests, Standardbuild und `npm audit`, statische Migrationsprüfung sowie Fixture-Gate bei Änderungen an `dev.db`.

### Finding 6 - DOMPurify-Schwachstellen im aktuellen Lockfile

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: regression
- Betroffene Datei(en) oder Pfade: `apps/frontend/package.json`, `apps/frontend/package-lock.json`, `apps/frontend/src/pages/admin/SupportTicketDetail.tsx:425`, `apps/frontend/src/pages/admin/SupportTicketDetail.tsx:454`
- Evidenz: Das Lockfile installiert DOMPurify `3.4.5`. `npm audit` meldet eine direkte moderate Schwachstelle mit mehreren Sanitization-Advisories und verfügbarem Fix. DOMPurify verarbeitet HTML aus eingehenden Support-Nachrichten vor `dangerouslySetInnerHTML`.
- Beschreibung des Problems: Nicht alle gemeldeten Varianten sind bei der aktuellen String-basierten Nutzung sicher ausnutzbar; die verwundbare Bibliothek liegt aber direkt in einem sicherheitsrelevanten Renderingpfad.
- Warum das relevant ist: Support-E-Mails sind nicht vertrauenswürdige Inhalte und werden in einer Admin-Oberfläche gerendert.
- Business Impact: Potenzielles XSS-Risiko im Administrationsbereich und erneuter Verlust eines scanner-sauberen Dependency-Standes.
- Konkrete Handlungsempfehlung: DOMPurify auf eine laut Audit gefixte Version oberhalb `3.4.10` aktualisieren, Lockfile neu erzeugen und einen XSS-Regressionstest für Support-HTML ergänzen.

### Finding 7 - Auth-Endpunkte verraten weiterhin Account-Status

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-06-13)
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/auth.py:400-421`, `apps/api/app/routers/auth.py:496-528`
- Evidenz: `/auth/verify/send` liefert für verifizierte Accounts `already_verified`, für unbekannte Adressen `ok`. `/auth/password/forgot` liefert für kürzlich angeforderte Tokens `throttled`, für unbekannte Adressen `ok`.
- Beschreibung des Problems: Angreifer können registrierte beziehungsweise verifizierte E-Mail-Adressen unterscheiden.
- Warum das relevant ist: Firmenadressen lassen sich für Phishing und Credential-Stuffing priorisieren.
- Business Impact: Erhöhtes Missbrauchs- und Support-Risiko.
- Konkrete Handlungsempfehlung: Nach außen identische Antwort, Statuscode und möglichst ähnliches Timing verwenden; Details nur intern protokollieren.

### Finding 8 - Brevo-Webhook-Secret bleibt in URL und Fehlerlogs exponiert

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-05-23)
- Betroffene Datei(en) oder Pfade: `apps/api/app/routers/newsletter.py:90-97`, `apps/api/app/main.py:67-80`, `apps/api/tests/test_newsletter.py`
- Evidenz: Der Webhook erwartet weiterhin `?secret=...`. Der globale Exception-Handler speichert bei unbehandelten Fehlern `str(request.url)` einschließlich Query-String in `system_errors.meta`.
- Beschreibung des Problems: Das Secret kann in Reverse-Proxy-, Monitoring-, Browser- und Fehlerlogs erscheinen.
- Warum das relevant ist: Der Webhook steuert Newsletter-Coupon-Ereignisse.
- Business Impact: Simulierte Events, ungewollte Gutscheine, Secret-Rotation und Logbereinigung.
- Konkrete Handlungsempfehlung: Secret in einen Header oder geheimen Pfad verschieben, Query-Logging für den Endpunkt deaktivieren und im globalen Error-Logging nur Pfad plus redigierte Query-Keys speichern.

### Finding 9 - Dev-Fixture enthält weiterhin real wirkende Daten und Asset-Probleme

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: persistent (seit 2026-05-22)
- Betroffene Datei(en) oder Pfade: `apps/api/dev.db`, `apps/api/scripts/verify_dev_fixture.py`, `apps/api/uploads/`
- Evidenz: Das Prüftool meldet zahlreiche nicht reservierte Domains in Benutzern, Einladungen, Outbox, Support und Audit-Metadaten. Eine Avatar-Datei ist vorhanden, aber nicht versioniert; eine Logo-Referenz fehlt. `dev.db` ist aktuell zusätzlich uncommitted verändert.
- Beschreibung des Problems: Aus dem Repository allein lässt sich nicht belegen, dass alle Daten synthetisch oder ausdrücklich freigegeben sind. Die Fixture ist zudem nicht vollständig reproduzierbar.
- Warum das relevant ist: Eine versionierte Datenbank bleibt dauerhaft in der Git-Historie.
- Business Impact: Datenschutz-, Compliance- und Demo-Stabilitätsrisiko.
- Konkrete Handlungsempfehlung: Vor jedem Commit reservierte Domains verwenden, reale Namen/Metadaten bereinigen, fehlende Assets ersetzen und `verify_dev_fixture.py --strict` als verpflichtendes Gate einsetzen.

### Finding 10 - BasicAuth-Hash ist fest im Produktions-Compose hinterlegt

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `deploy/docker-compose.prod.yml:59`, `deploy/prod-compose.sh`, `deploy/env/.env.deploy.prod.example`
- Evidenz: Der Benutzername und ein Apache-MD5-Hash für die geschlossene Live-Testphase sind direkt im versionierten Compose-File enthalten. Die Umschaltung aktiviert oder entfernt nur die Middleware; der Credential-Hash selbst ist nicht als Deployment-Secret konfigurierbar.
- Beschreibung des Problems: Ein schwacher oder wiederverwendeter BasicAuth-Wert kann offline gegen den veröffentlichten Hash geprüft werden. Änderungen erfordern einen Code-/Compose-Änderungsprozess.
- Warum das relevant ist: Solange der geschlossene Testmodus aktiv ist, ist BasicAuth die vorgeschaltete Zugangsschranke.
- Business Impact: Unbeabsichtigte öffentliche Erreichbarkeit der Testumgebung und zusätzlicher Credential-Rotationsaufwand.
- Konkrete Handlungsempfehlung: BasicAuth-Benutzerliste über eine nicht versionierte Server-Env oder Secret-Datei einspeisen, einen modernen starken Zufallswert verwenden und den bisherigen Wert rotieren.

### Finding 11 - Versions- und Projektdokumentation sind inkonsistent

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `AGENTS.md`, `README.md`, `CHANGELOG.md`, `apps/frontend/package.json`, lokale Git-Tags
- Evidenz: `AGENTS.md` beschreibt das Frontend als Vue 3, tatsächlich ist es React. `package.json` und API-Version stehen trotz umfangreicher Releases weiterhin auf `0.1.0`; `CHANGELOG.md` endet bei `0.0.1`. Im aktuellen Klon ist kein Git-Tag vorhanden, während der Vorbericht `v0.2.0` nannte.
- Beschreibung des Problems: Technischer Stack, Releasekennung und Änderungsverlauf liefern unterschiedliche Aussagen.
- Warum das relevant ist: Support, Rollback und technische Übergaben benötigen eine eindeutige Release-Identität.
- Business Impact: Erhöhte Wartungs- und Kommunikationskosten; langsamere Fehlerzuordnung.
- Konkrete Handlungsempfehlung: Stackbeschreibung korrigieren, SemVer-Workflow konsolidieren, Changelog aktualisieren und Remote-Tags beziehungsweise den Verlust von `v0.2.0` manuell prüfen.

### Finding 12 - Veralteter Notifications-TODO

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-06-07)
- Betroffene Datei(en) oder Pfade: `apps/frontend/src/components/layout/Sidebar.tsx:10`, `apps/frontend/src/components/layout/Sidebar.tsx:214`
- Evidenz: Der Kommentar fordert weiterhin die Anbindung von `notifCount`, obwohl die Sidebar bereits `useNotifications()` verwendet.
- Beschreibung des Problems: Der Kommentar beschreibt einen veralteten Arbeitsstand.
- Warum das relevant ist: Irreführende Kommentare erschweren Reviews und Wartung.
- Business Impact: Geringe, aber unnötige Wartungskosten.
- Konkrete Handlungsempfehlung: Kommentar entfernen.

## 8. Quick Wins

- `vite.config.js` und `vite.config.d.ts` aus der Versionierung nehmen oder vor dem Build zuverlässig neu erzeugen; danach `npm run build` erneut prüfen.
- DOMPurify aktualisieren und `npm audit` auf null bringen.
- E-Mail-Block- und Attachment-Traversal-Regressionstests ergänzen.
- Im globalen Exception-Handler Query-Werte aus URLs entfernen.
- Den BasicAuth-Hash aus `docker-compose.prod.yml` in ein Server-Secret verschieben.
- Den veralteten Sidebar-TODO und die Vue-Angabe in `AGENTS.md` korrigieren.

## 9. Strategische Empfehlungen

- Für Support-Dateien ein eigenes Upload-Lifecycle-Modell einführen: temporärer Upload, Besitzerbindung, atomare Ticketzuordnung, Quota, TTL und gemeinsamer sicherer Storage-Resolver.
- Sicherheitskritische Account-Flows als Zustandsmaschine modellieren und vollständig testen: E-Mail-Änderung, Passwort-Reset, Invite-Annahme und Kontolöschung.
- Den Branch-Workflow mit verpflichtenden CI-Checks absichern. Abhängigkeiten dürfen nur zusammen mit frischer Installation, Tests, Build und Scans aktualisiert werden.
- T026 als Security-Incident-Nachbearbeitung abschließen; Prävention ohne Rotation und History-Cleanup reicht nicht.
- Warnungsbudget einführen: 475 Backend-Warnungen, veraltete Pydantic-Configs und `datetime.utcnow()` schrittweise abbauen, bevor Pydantic 3 oder weitere Python-Upgrades anstehen.

## 10. Empfohlene nächste Aktion

**Vor jedem weiteren Release zuerst den Vite-8-Standardbuild reparieren und unmittelbar danach die beiden hohen Security-Bugs in E-Mail-Änderung und Support-Attachment-Lifecycle beheben.**

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Sind die historischen SMTP-, Stripe- und Microsoft-Credentials bereits providerseitig rotiert oder weiterhin gültig?
- Existieren die im Vorbericht genannten Git-Tags, insbesondere `v0.2.0`, noch auf dem Remote?
- Enthält `apps/api/dev.db` ausschließlich synthetische beziehungsweise ausdrücklich freigegebene Daten?
- Schlägt der Frontend-Docker-Build auf dem Dev-/Produktivserver wegen vorhandener `tsconfig.node.tsbuildinfo` ebenfalls fehl?
- Welcher BasicAuth-Wert ist aktuell auf dem VPS aktiv, und wurde der versionierte Hash bereits ersetzt?
- Wie werden verwaiste Dateien unter `uploads/attachments/` aktuell operativ erkannt und bereinigt?
- Kein Live-Penetrationstest, kein PostgreSQL-Test, kein echter Stripe-/Brevo-/SMTP-Test und kein vollständiger Browser-E2E-Test wurden durchgeführt.
