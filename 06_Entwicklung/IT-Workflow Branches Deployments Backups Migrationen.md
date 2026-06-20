# IT-Workflow: Branches, Deployments, Backups und Migrationen

Dieses Dokument beschreibt, wie Normdex mit Git-Branches, Dev-/Produktivsystemen, Datenbank-Backups und Migrationen arbeitet.

Ziel ist ein sauberer, nachvollziehbarer Ablauf:

```text
Entwicklung -> Dev-Server -> Produktion
```

Es wird nicht direkt in Produktion entwickelt.

## Systemuebersicht

Auf dem Server gibt es mehrere Repositories bzw. Arbeitsbereiche:

```text
/opt/repos/normdex-landingpage   -> Landingpage
/opt/repos/normdex-webapp-dev    -> Dev-Version der Normdex Web-App
/opt/repos/normdex-webapp        -> Produktive Normdex Web-App
/opt/repos/normdex-vault         -> Dokumentation / Wissensbasis
```

Die wichtigsten Domains sind:

```text
https://dev.normdex.at      -> Dev-Frontend
https://dev-api.normdex.at  -> Dev-API
https://app.normdex.at      -> Produktives Frontend
https://api.normdex.at      -> Produktive API
```

Der VS-Code-Workspace auf dem Server dient nur als Arbeitsoberflaeche. Der eigentliche Ablauf laeuft ueber:

```text
Git -> Branches -> Pull auf Server -> Docker Compose -> Deployment
```

## Branch-Konzept der Normdex Web-App

Fuer die Web-App verwenden wir drei zentrale Branches.

### develop

`develop` ist der Branch fuer aktive Entwicklung.

Hier werden neue Features, kleinere Anpassungen, Bugfixes und experimentelle Aenderungen entwickelt.

```text
develop = laufende Entwicklung
```

Dieser Branch kann instabil sein. Er muss nicht jederzeit produktionsreif sein.

### dev-server

`dev-server` ist der Branch fuer das Dev-/Staging-System.

Dieser Branch entspricht dem Stand, der auf dem Dev-Server laufen soll.

```text
dev-server = Stand auf dev.normdex.at
```

Nur Aenderungen, die auf dem Dev-System getestet werden sollen, werden von `develop` in `dev-server` uebernommen.

### main

`main` ist der Branch fuer den produktiven Betrieb.

```text
main = Stand auf app.normdex.at
```

Alles, was auf `main` liegt, gilt als freigegeben fuer die Produktivumgebung. In `main` duerfen keine unfertigen oder ungetesteten Aenderungen landen.

## Sauberer Workflow

Der Standardablauf lautet:

```text
develop -> dev-server -> main
```

Also:

1. Lokal oder im Entwicklungsrepo auf `develop` entwickeln.
2. Aenderungen committen und nach GitHub pushen.
3. `develop` in `dev-server` mergen.
4. Dev-Server aktualisieren und deployen.
5. Auf `dev.normdex.at` testen.
6. Vor Produktivdeployment Datenbank-Backup erstellen.
7. `dev-server` in `main` mergen.
8. Produktivserver aktualisieren.
9. Datenbankmigration ausfuehren.
10. Produktivsystem deployen und pruefen.

## Nicht sauberer Workflow

Folgende Arbeitsweisen sollen vermieden werden:

```text
develop -> main
```

Das ist nicht sauber, weil Aenderungen dabei am Dev-Server vorbeigehen.

Ebenfalls nicht sauber:

- Direkt auf dem Produktivserver Code aendern.
- Direkt im `main`-Branch experimentieren.
- Migrationen ohne Backup ausfuehren.
- Produktivdeployment ohne vorherigen Test auf `dev.normdex.at`.
- Uncommitted Changes auf dem Server liegen lassen.
- Server als eigentliche Entwicklungsumgebung missbrauchen.

Der Server soll im Normalfall nur den gewuenschten Git-Stand ziehen und deployen.

## Lokale Entwicklung auf develop

Auf dem lokalen Rechner:

```bash
cd "D:\Normdex\01_repos\normdex-app"
git checkout develop
git pull origin develop
```

Dann Aenderungen durchfuehren.

Danach pruefen:

```bash
git status
```

Aenderungen committen:

```bash
git add .
git commit -m "Kurze Beschreibung der Aenderung"
git push origin develop
```

Damit ist der neue Entwicklungsstand auf GitHub im Branch `develop`.

## Uebergabe von develop auf dev-server

Wenn eine Aenderung auf dem Dev-Server getestet werden soll:

```bash
git checkout dev-server
git pull origin dev-server
git merge develop
git push origin dev-server
```

Damit enthaelt `dev-server` den aktuellen Stand aus `develop`.

## Dev-Server aktualisieren

Auf den Server verbinden:

```bash
ssh normdex-vps
```

Dann ins Dev-Repo wechseln:

```bash
cd /opt/repos/normdex-webapp-dev
git status
git checkout dev-server
git pull --ff-only origin dev-server
```

Danach Dev-System neu bauen und starten:

```bash
cd /opt/stacks/normdex-dev
docker compose up -d --build
```

Status pruefen:

```bash
docker compose ps
docker compose logs -f --tail=100
```

Danach testen:

```text
https://dev.normdex.at
https://dev-api.normdex.at
```

## Datenbank-Backups vor Migrationen

Grundregel:

```text
Keine Datenbankmigration ohne vorheriges Backup.
```

Das gilt besonders fuer:

- Produktivdeployments.
- Updates mit Alembic-/SQL-Migrationen.
- Neue Branches, die auf dem Dev- oder Produktivsystem mit Datenbankaenderungen getestet werden.
- Groessere Strukturaenderungen an Tabellen, Spalten, Constraints oder Datenmodellen.

Ein rein lokaler Git-Branch ohne Deployment braucht normalerweise noch kein Server-Datenbank-Backup. Sobald der Branch aber gegen eine echte Dev- oder Produktivdatenbank deployed wird und Migrationen laufen, wird vorher ein Backup erstellt.

## Datenbank-Backup erstellen

Zuerst in den passenden Stack wechseln.

Fuer Dev:

```bash
cd /opt/stacks/normdex-dev
```

Fuer Produktion:

```bash
cd /opt/stacks/normdex
```

Dann pruefen, wie der Datenbank-Service heisst:

```bash
docker compose ps
```

Typische Namen sind zum Beispiel:

```text
db
postgres
database
```

Backup-Ordner erstellen:

```bash
mkdir -p backups
```

Backup erstellen, Beispiel mit Service `db`:

```bash
docker compose exec -T db pg_dump -U postgres -d normdex > backups/normdex_$(date +%Y-%m-%d_%H-%M-%S).sql
```

Falls der Datenbank-Service anders heisst, muss `db` entsprechend ersetzt werden.

Beispiel mit Service `postgres`:

```bash
docker compose exec -T postgres pg_dump -U postgres -d normdex > backups/normdex_$(date +%Y-%m-%d_%H-%M-%S).sql
```

Danach pruefen, ob die Datei erstellt wurde:

```bash
ls -lh backups
```

Das Backup muss sichtbar sein und eine plausible Dateigroesse haben. Eine Datei mit `0 B` ist kein gueltiges Backup.

## Migrationen ausfuehren

Migrationen werden erst nach erfolgreichem Backup ausgefuehrt.

Der konkrete Befehl haengt vom API-Service ab. Typisch ist bei Alembic:

```bash
docker compose exec api alembic upgrade head
```

Falls der API-Service anders heisst, vorher pruefen:

```bash
docker compose ps
```

Beispiele:

```bash
docker compose exec backend alembic upgrade head
docker compose exec normdex-api alembic upgrade head
docker compose exec api alembic upgrade head
```

Nach der Migration pruefen:

```bash
docker compose logs --tail=100 api
```

API-Healthcheck testen:

```bash
curl https://dev-api.normdex.at/health
curl https://api.normdex.at/health
```

## Ablauf bei Dev-Deployment mit Migration

Wenn ein neuer Stand auf dem Dev-Server getestet werden soll und Datenbankaenderungen enthaelt:

1. `dev-server`-Branch aktualisieren.
2. Dev-Repo am Server pullen.
3. Backup der Dev-Datenbank erstellen.
4. Container bauen/starten.
5. Migration ausfuehren.
6. Logs pruefen.
7. Dev-App testen.

Beispiel:

```bash
cd /opt/repos/normdex-webapp-dev
git checkout dev-server
git pull --ff-only origin dev-server

cd /opt/stacks/normdex-dev
mkdir -p backups
docker compose exec -T db pg_dump -U postgres -d normdex > backups/normdex_dev_$(date +%Y-%m-%d_%H-%M-%S).sql

docker compose up -d --build
docker compose exec api alembic upgrade head

docker compose ps
docker compose logs -f --tail=100
```

## Freigabe von Dev nach Produktion

Wenn auf dem Dev-System alles funktioniert, wird der getestete Stand in `main` uebernommen.

Lokal:

```bash
cd "D:\Normdex\01_repos\normdex-app"
git checkout main
git pull origin main
git merge dev-server
git push origin main
```

Damit enthaelt `main` den getesteten Stand aus `dev-server`.

## Produktivdeployment

Auf dem Server:

```bash
cd /opt/repos/normdex-webapp
git status
git checkout main
git pull --ff-only origin main
```

Dann zuerst Backup der Produktivdatenbank erstellen:

```bash
cd /opt/stacks/normdex
mkdir -p backups
docker compose exec -T db pg_dump -U postgres -d normdex > backups/normdex_prod_$(date +%Y-%m-%d_%H-%M-%S).sql
```

Backup pruefen:

```bash
ls -lh backups
```

Erst danach deployen:

```bash
./deploy/prod-compose.sh up -d --build
```

Falls Migrationen vorhanden sind:

```bash
./deploy/prod-compose.sh --profile tools run --rm api-migrate
```

Danach pruefen:

```bash
docker compose ps
docker compose logs -f --tail=100
```

Produktivsystem testen:

```text
https://app.normdex.at
https://api.normdex.at
```

### Produktiv-Gate für geschlossene Live-Testphase

`app.normdex.at` kann bewusst hinter Traefik BasicAuth bleiben, solange Stripe, E-Mail, Webhooks, Lizenzkauf und weitere Anbindungen noch privat getestet werden. Dieser Schutz ist kein Fehler, sondern die aktuelle geschlossene Live-Testphase.

Der Schalter liegt im App-Repo unter:

```text
deploy/env/.env.deploy.prod
```

Werte:

```env
NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=true
NORMDEX_FRONTEND_BASIC_AUTH_USERS='admin:$2y$...'
```

`true` bedeutet: `app.normdex.at` bleibt per BasicAuth geschützt. Das ist der gewollte Zustand, solange noch keine Kundenregistrierung möglich sein soll. `NORMDEX_FRONTEND_BASIC_AUTH_USERS` enthält einen frisch erzeugten htpasswd-Eintrag und liegt ausschließlich in der nicht versionierten Serverdatei. Der Wrapper bricht ab, wenn BasicAuth aktiv ist und kein Credential gesetzt wurde.

```env
NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false
```

`false` bedeutet: `app.normdex.at` ist öffentlich erreichbar. Registrierung, Login und Checkout sind dann ohne vorgelagertes BasicAuth-Passwort erreichbar. Dieser Wert wird erst für den echten Kundenstart gesetzt.

Wichtig:

- Produktivdeployments der App laufen über `./deploy/prod-compose.sh`, damit der Schalter angewendet wird.
- Nicht direkt `docker compose -f deploy/docker-compose.prod.yml ...` verwenden, wenn der Gate-Schalter gelten soll.
- BasicAuth-Passwort mit einem starken Zufallswert erzeugen, den htpasswd-Eintrag in einfache Anführungszeichen setzen und bei Verdacht oder nach Repository-Leaks rotieren.
- Vor Kundenstart bewusst prüfen: Ist `NORMDEX_FRONTEND_BASIC_AUTH_ENABLED=false` gesetzt?
- Wenn die App öffentlich sichtbar sein soll, Registrierung aber weiterhin blockiert bleiben muss, braucht es zusätzlich einen App-/Backend-Schalter für Registrierungen; BasicAuth sperrt immer das gesamte Frontend.

## Wichtige Regel fuer Migrationen

Migrationen sollen immer versioniert sein.

Das bedeutet:

- Datenbankaenderungen gehoeren als Alembic-Migration ins Repository.
- Keine manuellen Aenderungen direkt in der Produktivdatenbank.
- Keine Tabellen direkt per SQL aendern, ohne dass die Aenderung als Migration nachvollziehbar ist.

Sauber:

```text
Modellaenderung im Code
-> Alembic-Migration erstellen
-> Migration committen
-> Dev testen
-> Produktion nach Backup migrieren
```

Nicht sauber:

```text
Direkt in psql eine Spalte aendern
-> Code spaeter irgendwie anpassen
-> keine Dokumentation
-> nicht reproduzierbarer Zustand
```

## Rollback-Grundsatz

Falls nach einem Deployment ein Fehler auftritt:

1. Nicht hektisch weiterdeployen.
2. Logs pruefen.
3. Letzten funktionierenden Git-Stand identifizieren.
4. Container ggf. mit altem Stand neu bauen.
5. Datenbank nur aus Backup wiederherstellen, wenn die Migration oder Datenaenderung problematisch war.

Ein Code-Rollback ist einfacher als ein Datenbank-Rollback.

Darum gilt:

```text
Vor jeder Migration ein Backup.
```

## Merksatz fuer neue IT-Mitarbeiter

Der zentrale Ablauf bei Normdex lautet:

```text
develop -> dev-server -> main
```

Und fuer Deployments mit Datenbankaenderungen:

```text
Backup -> Deploy -> Migration -> Test
```

Produktiv gilt immer:

- Kein Produktivdeployment ohne vorherigen Dev-Test.
- Keine Produktivmigration ohne vorheriges Datenbank-Backup.
- Keine direkten Codeaenderungen auf dem Produktivserver.

Das ist unser sauberer Standard-Workflow.

## Verwandte Dokumente

- [[VPS-Serverstruktur und Deployment-Architektur]]
- [[Automatisierter Backup-Service]] – täglicher automatischer Backup mit SharePoint-Offsite
