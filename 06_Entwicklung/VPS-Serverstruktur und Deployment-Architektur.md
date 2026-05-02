# VPS-Serverstruktur und Deployment-Architektur

Der Normdex-VPS ist die zentrale Serverumgebung fuer Repositories, Docker-Stacks, Reverse Proxy, Datenbanken, Backups und Deployments.

Grundsaetzlich trennen wir auf dem Server zwischen:

- Code / Git-Repositories
- Docker-Stacks / Deployment-Konfiguration
- Datenbanken / persistente Volumes
- Backups
- Reverse Proxy / Routing
- Dokumentation / Vault

Diese Trennung ist wichtig, damit Code, Infrastruktur und Daten nicht vermischt werden.

## Was der VPS-Server kann

Der VPS uebernimmt mehrere Aufgaben:

1. Hosten der produktiven Normdex-App.
2. Hosten der Dev-Version der Normdex-App.
3. Hosten der Landingpage.
4. Bereitstellen der API-Endpunkte.
5. Bereitstellen der PostgreSQL-Datenbanken.
6. Routing ueber Traefik.
7. TLS/HTTPS ueber Let's Encrypt.
8. Ausfuehren von Docker-Containern.
9. Speichern der Git-Repositories.
10. Speichern von Backups.
11. Bereitstellen interner Tools wie n8n.
12. Bereitstellen des Normdex Vaults / der Dokumentation.

Der Server ist also nicht nur ein Webspace, sondern eine vollstaendige Docker-basierte Betriebsumgebung.

## Grundidee der Ordnerstruktur

Die Struktur folgt dieser Logik:

```text
/opt/repos      -> Quellcode / Git-Repositories
/opt/stacks     -> Docker-Compose-Stacks / Deployment-Konfigurationen
/opt/...        -> zusaetzliche persistente Daten, Assets oder interne Dienste
/home/deploy    -> Benutzerumgebung des deploy-Users, SSH, Workspaces usw.
```

Wichtig:

- `/opt/repos` enthaelt Code.
- `/opt/stacks` enthaelt Deployment-Konfiguration.
- `/home/deploy` enthaelt benutzerspezifische Arbeitsdateien.

Man sollte nicht beliebig neue Ordner irgendwo anlegen, sondern diese Struktur beibehalten.

## Repositories unter /opt/repos

Die Git-Repositories liegen auf dem VPS unter:

```text
/opt/repos
```

Aktuell relevante Repositories:

```text
/opt/repos/normdex-landingpage
/opt/repos/normdex-webapp-dev
/opt/repos/normdex-webapp
/opt/repos/normdex-vault
```

### /opt/repos/normdex-landingpage

Dieses Repo enthaelt die oeffentliche Normdex-Landingpage.

Typische Domains:

```text
https://normdex.at
https://www.normdex.at
```

Die Landingpage ist vom eigentlichen App-System getrennt.

### /opt/repos/normdex-webapp-dev

Dieses Repo enthaelt die Dev-Version der Normdex Web-App.

Dieses Repo sollte normalerweise auf dem Branch `dev-server` stehen.

Pruefen:

```bash
cd /opt/repos/normdex-webapp-dev
git branch --show-current
```

Erwartet:

```text
dev-server
```

Dieses Repo ist mit dem Dev-System verbunden:

```text
https://dev.normdex.at
https://dev-api.normdex.at
```

### /opt/repos/normdex-webapp

Dieses Repo enthaelt die produktive Normdex Web-App.

Dieses Repo sollte normalerweise auf dem Branch `main` stehen.

Pruefen:

```bash
cd /opt/repos/normdex-webapp
git branch --show-current
```

Erwartet:

```text
main
```

Dieses Repo ist mit dem Produktivsystem verbunden:

```text
https://app.normdex.at
https://api.normdex.at
```

### /opt/repos/normdex-vault

Dieses Repo enthaelt Dokumentation, interne Notizen, technische Konzepte, Betriebsanleitungen und Wissensbasis.

Der Vault ist kein produktives App-System, sondern dient als interne Dokumentationsquelle.

Typische Inhalte:

- Architekturentscheidungen
- Deployment-Anleitungen
- Branch-Workflow
- Normdex-Dokumentation
- Lizenz-/Stripe-Konzepte
- Support-Prozesse
- Technische Betriebsdokumentation

## Docker-Stacks unter /opt/stacks

Die Docker-Compose-Konfigurationen liegen unter:

```text
/opt/stacks
```

Typische Struktur:

```text
/opt/stacks/proxy
/opt/stacks/normdex
/opt/stacks/normdex-dev
/opt/stacks/n8n
```

Je nach tatsaechlichem Setup koennen einzelne Dienste auch leicht anders strukturiert sein. Die Grundlogik bleibt:

```text
/opt/repos  = Code
/opt/stacks = Docker-Compose-Konfiguration und Deployment
```

## Wichtige Docker-Stacks

### /opt/stacks/proxy

Hier liegt der Reverse-Proxy-Stack, typischerweise Traefik.

Traefik uebernimmt:

- Routing von Domains/Subdomains zu Containern.
- HTTPS/TLS-Zertifikate.
- Let's Encrypt.
- Weiterleitung von `app.normdex.at` zur App.
- Weiterleitung von `api.normdex.at` zur API.
- Weiterleitung von `dev.normdex.at` zum Dev-Frontend.
- Weiterleitung von `dev-api.normdex.at` zur Dev-API.

Typische Domains, die ueber Traefik geroutet werden:

```text
normdex.at
www.normdex.at
app.normdex.at
api.normdex.at
dev.normdex.at
dev-api.normdex.at
n8n.normdex.at
```

Der Proxy ist zentral. Wenn Traefik nicht laeuft, sind die Dienste von aussen meist nicht erreichbar.

Pruefen:

```bash
cd /opt/stacks/proxy
docker compose ps
```

### /opt/stacks/normdex

Das ist der Produktiv-Stack der Normdex-App.

Er verwendet typischerweise das produktive Repo:

```text
/opt/repos/normdex-webapp
```

Dieser Stack ist zustaendig fuer:

- Produktiv-Frontend
- Produktiv-API
- Produktiv-Datenbank
- Produktiv-Umgebungsvariablen
- Produktiv-Container

Typische externe URLs:

```text
https://app.normdex.at
https://api.normdex.at
```

Deployment:

```bash
cd /opt/stacks/normdex
docker compose up -d --build
```

Status pruefen:

```bash
docker compose ps
docker compose logs -f --tail=100
```

### /opt/stacks/normdex-dev

Das ist der Dev-/Staging-Stack der Normdex-App.

Er verwendet typischerweise das Dev-Repo:

```text
/opt/repos/normdex-webapp-dev
```

Dieser Stack ist zustaendig fuer:

- Dev-Frontend
- Dev-API
- Dev-Datenbank
- Dev-Umgebungsvariablen
- Dev-Container

Typische externe URLs:

```text
https://dev.normdex.at
https://dev-api.normdex.at
```

Deployment:

```bash
cd /opt/stacks/normdex-dev
docker compose up -d --build
```

Status pruefen:

```bash
docker compose ps
docker compose logs -f --tail=100
```

### /opt/stacks/n8n

Falls n8n auf dem Server betrieben wird, liegt der Stack typischerweise hier.

n8n wird fuer Automatisierungen verwendet, z. B.:

- Ticket-Benachrichtigungen
- Webhook-Verarbeitung
- E-Mail-/Support-Automatisierung
- Telegram-Benachrichtigungen
- Interne Workflows

Typische URL:

```text
https://n8n.normdex.at
```

## Docker-Netzwerke

Docker verwendet Netzwerke, um Container miteinander zu verbinden.

Typische Netzwerke:

```text
proxy
normdex_internal
```

### proxy

Das Netzwerk `proxy` verbindet oeffentlich erreichbare Dienste mit Traefik.

Container, die von aussen ueber eine Domain erreichbar sein sollen, muessen in der Regel mit dem `proxy`-Netzwerk verbunden sein.

Beispiele:

- Frontend
- API
- n8n
- Traefik

### normdex_internal

Das Netzwerk `normdex_internal` ist fuer interne Kommunikation gedacht.

Beispiele:

- API <-> Datenbank
- API <-> interne Dienste
- Backend <-> PostgreSQL

Die Datenbank sollte nicht direkt oeffentlich erreichbar sein.

## Docker-Compose-Grundprinzip

Docker Compose beschreibt, welche Container gestartet werden und wie sie miteinander verbunden sind.

Ein Stack kann z. B. enthalten:

- `frontend`
- `api`
- `db`
- `redis`
- `worker`

Nicht jeder Stack muss alle Dienste enthalten. Wichtig ist: Die konkrete Wahrheit steht immer in der jeweiligen `docker-compose.yml`.

Compose-Dateien finden:

```bash
find /opt/stacks /opt/repos -name "docker-compose*.yml" -o -name "compose*.yml"
```

Container anzeigen:

```bash
docker ps
```

Container eines bestimmten Stacks anzeigen:

```bash
cd /opt/stacks/normdex
docker compose ps
```

Logs anzeigen:

```bash
docker compose logs -f --tail=100
```

Einzelnen Dienst neu starten:

```bash
docker compose restart api
```

Gesamten Stack neu bauen und starten:

```bash
docker compose up -d --build
```

## Verbindung zwischen Repository und Docker-Stack

Wichtig fuer neue IT-Mitarbeiter:

```text
Das Git-Repository allein deployed noch nichts.
```

Der Ablauf ist immer:

```text
Git-Branch aktualisieren
-> Repository auf dem Server pullen
-> Docker-Stack neu bauen/starten
-> ggf. Migration ausfuehren
-> Logs und Healthchecks pruefen
```

Beispiel Dev:

```bash
cd /opt/repos/normdex-webapp-dev
git checkout dev-server
git pull --ff-only origin dev-server

cd /opt/stacks/normdex-dev
docker compose up -d --build
```

Beispiel Produktion:

```bash
cd /opt/repos/normdex-webapp
git checkout main
git pull --ff-only origin main

cd /opt/stacks/normdex
docker compose up -d --build
```

## Umgebungsvariablen und Secrets

Produktiv- und Dev-Systeme muessen getrennte Umgebungsvariablen verwenden.

Typische Inhalte:

- Datenbank-URL
- Stripe Keys
- Brevo API Keys
- Microsoft Graph Credentials
- JWT Secrets
- Admin-Konfiguration
- API Keys
- Webhook Secrets

Grundregel:

```text
Keine Secrets direkt in Git committen.
```

Umgebungsdateien liegen typischerweise im Deployment-/Stack-Bereich, nicht im normalen Quellcode.

Beispiele:

```text
/opt/stacks/normdex/...
/opt/stacks/normdex-dev/...
/opt/repos/normdex-webapp/deploy/env/...
```

Die tatsaechliche Lage kann je nach Stack variieren. Wichtig ist: `.env`-Dateien mit echten Secrets duerfen nicht oeffentlich oder versehentlich in Git landen.

## Backups auf dem VPS

Backups sollten moeglichst stacknah abgelegt werden.

Beispiele:

```text
/opt/stacks/normdex/backups
/opt/stacks/normdex-dev/backups
```

Produktiv-Backups sollten klar erkennbar benannt werden:

```text
normdex_prod_2026-05-02_18-30-00.sql
```

Dev-Backups entsprechend:

```text
normdex_dev_2026-05-02_18-30-00.sql
```

Wichtig:

- Produktivbackup vor Produktivmigration.
- Dev-Backup vor Dev-Migration.
- Backup-Datei nach Erstellung pruefen.
- Keine Migration blind ausfuehren.

Backup pruefen:

```bash
ls -lh backups
```

## VS-Code-Workspace auf dem Server

Der VS-Code-Workspace ist eine Komfortfunktion, damit mehrere Repositories gleichzeitig sichtbar sind.

Die Workspace-Datei liegt typischerweise hier:

```text
/home/deploy/workspaces/normdex.code-workspace
```

Sie enthaelt z. B.:

```json
{
  "folders": [
    {
      "name": "01 Landingpage",
      "path": "/opt/repos/normdex-landingpage"
    },
    {
      "name": "02 Normdex App",
      "path": "/opt/repos/normdex-webapp"
    },
    {
      "name": "03 Normdex App Dev",
      "path": "/opt/repos/normdex-webapp-dev"
    },
    {
      "name": "04 Normdex Vault",
      "path": "/opt/repos/normdex-vault"
    }
  ],
  "settings": {
    "git.openRepositoryInParentFolders": "never",
    "terminal.integrated.defaultProfile.linux": "bash"
  }
}
```

Wichtig:

- Der Workspace deployed nichts.
- Der Workspace ist nur die Arbeitsoberflaeche.
- Deployment erfolgt weiterhin ueber Git und Docker Compose.

## Was darf man wo bearbeiten?

### In /opt/repos/normdex-webapp-dev

Hier duerfen Aenderungen getestet werden.

Trotzdem gilt:

- Aenderungen sauber committen.
- Keine unklaren lokalen Aenderungen liegen lassen.
- Branch `dev-server` beachten.

### In /opt/repos/normdex-webapp

Hier liegt Produktion.

Grundregel:

```text
Keine direkte Entwicklung im Produktiv-Repo.
```

Dieses Repo wird normalerweise nur aktualisiert durch:

```bash
git pull --ff-only origin main
```

### In /opt/repos/normdex-landingpage

Hier liegt die Landingpage.

Auch hier gilt:

- Aenderungen ueber Git nachvollziehbar machen.
- Vor Produktivdeployment testen.

### In /opt/repos/normdex-vault

Hier liegt Dokumentation.

Hier koennen Anleitungen, interne Konzepte und Betriebsdokumentation gepflegt werden.

## Haeufige Pruefkommandos fuer neue IT-Mitarbeiter

Aktuellen Benutzer pruefen:

```bash
whoami
```

Serverpfad pruefen:

```bash
pwd
```

Repos anzeigen:

```bash
ls -la /opt/repos
```

Docker-Stacks anzeigen:

```bash
ls -la /opt/stacks
```

Git-Branch pruefen:

```bash
git branch --show-current
```

Git-Status pruefen:

```bash
git status
```

Alle laufenden Container anzeigen:

```bash
docker ps
```

Alle Container anzeigen, auch gestoppte:

```bash
docker ps -a
```

Docker-Netzwerke anzeigen:

```bash
docker network ls
```

Docker-Volumes anzeigen:

```bash
docker volume ls
```

Speicherplatz pruefen:

```bash
df -h
```

Docker-Speicherverbrauch pruefen:

```bash
docker system df
```

## Saubere Betriebsregel fuer den VPS

Fuer neue IT-Mitarbeiter ist die wichtigste Regel:

```text
Nicht einfach irgendwo am Server Aenderungen machen.
```

Immer zuerst verstehen:

1. In welchem Repo bin ich?
2. Auf welchem Branch bin ich?
3. Welcher Docker-Stack nutzt dieses Repo?
4. Ist es Dev oder Produktion?
5. Brauche ich vor der Aenderung ein Datenbank-Backup?
6. Muss nach der Aenderung eine Migration laufen?

Der VPS ist produktionsnah. Deshalb gilt:

- Dev darf getestet werden.
- Produktion wird nur bewusst aktualisiert.
- Datenbankmigrationen nur mit Backup.
- Secrets niemals in Git.
- Docker-Stacks nicht blind loeschen.
- Volumes nicht loeschen, ohne sicher zu sein.

## Gesamtbild der Normdex-Architektur

Vereinfacht sieht der Ablauf so aus:

```text
Lokaler Rechner
    -> git push
GitHub
    -> git pull
VPS /opt/repos
    -> docker compose build
VPS /opt/stacks
    -> Container
Docker
    -> Routing
Traefik
    -> HTTPS
normdex.at / app.normdex.at / api.normdex.at
```

Fuer Dev:

```text
develop
    -> merge
dev-server
    -> pull auf /opt/repos/normdex-webapp-dev
    -> docker compose up -d --build in /opt/stacks/normdex-dev
    ->
https://dev.normdex.at
https://dev-api.normdex.at
```

Fuer Produktion:

```text
dev-server
    -> merge nach main
main
    -> pull auf /opt/repos/normdex-webapp
    -> Backup der Produktivdatenbank
    -> docker compose up -d --build in /opt/stacks/normdex
    -> Migration
    ->
https://app.normdex.at
https://api.normdex.at
```

## Kurzfassung fuer neue IT-Mitarbeiter

- Code liegt unter `/opt/repos`.
- Deployment-Konfiguration liegt unter `/opt/stacks`.
- Traefik routet Domains zu Docker-Containern.
- Dev-App und Produktiv-App sind getrennt.
- Dev nutzt `normdex-webapp-dev` und den Branch `dev-server`.
- Produktion nutzt `normdex-webapp` und den Branch `main`.
- Neue Entwicklung laeuft zuerst ueber `develop`.
- Der saubere Git-Weg ist `develop -> dev-server -> main`.
- Vor Datenbankmigrationen wird immer ein Backup erstellt.
- Auf Produktion wird nicht direkt entwickelt.
- Docker-Volumes und Datenbanken werden niemals unueberlegt geloescht.

Damit ist erklaert, wie der VPS grundsaetzlich organisiert ist und wie Code, Docker, Datenbank und Domains zusammenspielen.

## Verwandte Dokumente

- [[IT-Workflow Branches Deployments Backups Migrationen]]
