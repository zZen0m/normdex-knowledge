# KI Kontext - Einstieg

Dies ist die zentrale Wissensbasis für Normdex.

## Arbeitsbereiche

- App-Code: `D:\Normdex\01_repos\normdex-app`
- Landingpage-Code: `D:\Normdex\01_repos\normdex-landingpage`
- Obsidian Vault: `D:\Normdex\02_knowledge\normdex-vault`
- SharePoint-Dokumente: `D:\Normdex\03_external\sharepoint-normdex`

## Wichtige Betriebsdokumentation

- [[IT-Workflow Branches Deployments Backups Migrationen]] beschreibt den verbindlichen Ablauf `develop -> dev-server -> main` sowie Backups, Migrationen, Dev-/Produktivdeployment und Rollback.
- [[VPS-Serverstruktur und Deployment-Architektur]] beschreibt `/opt/repos`, `/opt/stacks`, Docker-Stacks, Traefik, Secrets, Backups und grundlegende Pruefkommandos.
- Merksatz fuer Deployments mit Datenbankaenderungen: `Backup -> Deploy -> Migration -> Test`.

## Ordnerstruktur im Vault

- `00_Start` – Einstieg, Übersicht, zentrale Kontextdateien
- `01_Produkt` – Produktvision, Zielgruppen, Funktionen, Roadmap
- `02_App` – App-Architektur, Datenmodell, API, Berechnungslogik
- `03_Landingpage` – Seitenstruktur, SEO, Texte, Design
- `04_Marketing` – Newsletter, Kampagnen, Content-Ideen
- `05_Geschaeft` – Geschäftsmodell, Pricing, Lizenzen, Rechtliches
- `06_Entwicklung` – Dev-Setup, Deployment, Git, Docker, bekannte Probleme
- `07_KI_Agenten` – Claude, Codex, Paperclip, Prompts und Arbeitsweisen
- `08_Entscheidungen` – Architektur- und Produktentscheidungen
- `09_Besprechungen` – Meetingnotizen und Protokolle
- `10_Aufgaben` – zentrale Aufgabenübersicht mit Todo-Dateien
- `11_Audits` – strukturierte Audit-Berichte, unterteilt in `Landingpage/` und `Webapp/`
- `90_Archiv` – alte oder nicht mehr aktive Inhalte
- `99_Anhaenge` – Bilder, Screenshots und sonstige Anhänge

## Arbeitsregeln für KI-Tools

1. Codeänderungen erfolgen nur in den jeweiligen Repositories unter `D:\Normdex\01_repos`.
2. Dokumentationsänderungen erfolgen im Vault unter `D:\Normdex\02_knowledge\normdex-vault`.
3. Offizielle Office-Dokumente, Verträge, Rechnungen und finale Assets liegen in SharePoint.
4. Keine Git-Repositories in SharePoint verschieben.
5. Keine SharePoint-Dateien versehentlich in Git-Repositories kopieren.
6. Bei Änderungen an Code, Architektur, API, Datenmodell, Lizenzsystem, Pricing, Marketingtexten oder Supportlogik muss geprüft werden, ob diese Wissensbasis aktualisiert werden muss.
7. Aufgaben werden nach der neuen Struktur in `10_Aufgaben/Aufgaben.md` verwaltet: Die Übersicht listet alle Todos, Details stehen in einzelnen Dateien unter `10_Aufgaben/offene Todos/`, erledigte Todos werden nach `10_Aufgaben/abgeschlossene Todos/` verschoben.
