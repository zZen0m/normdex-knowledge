# T001 - NORMDEX_COMPLETE_GUIDE aus Repo löschen

**Status:** erledigt  
**Bereich:** Vault  
**Erstellt:** 2026-04-27  
**Abgeschlossen:** 2026-04-27

## Ziel

Den veralteten Repo-Guide `docs/NORMDEX_COMPLETE_GUIDE.md` aus dem App-Repo entfernen, sobald alle relevanten Inhalte im Vault als Single Source of Truth vorhanden sind und keine aktuellen Arbeitsanweisungen oder Automatisierungen mehr von dieser Datei abhängen.

## Ergebnis

- `docs/NORMDEX_COMPLETE_GUIDE.md` wurde aus dem App-Repo gelöscht.
- `docs/NORMDEX_COMPLETE_GUIDE.md` wurde aus dem Landingpage-Repo gelöscht.
- Die Sync-Skripte für den Complete Guide wurden aus dem App-Repo gelöscht:
  - `scripts/sync-docs.sh`
  - `scripts/on-guide-changed.sh`
- Die Sync-Skripte für den Complete Guide wurden aus dem Landingpage-Repo gelöscht:
  - `scripts/sync-docs.sh`
  - `scripts/on-guide-changed.sh`
- Das `sync-docs`-Script wurde aus `package.json` im Landingpage-Repo entfernt.
- App-Repo `AGENTS.md`: aktiver Complete-Guide-Hinweis wurde entfernt.
- App-Repo `CLAUDE.md`: Hinweis auf den Vault als Referenz wurde aktualisiert.
- Vault-Dateien `Brand Identity & Voice.md` und `Designsystem & Farben.md`: Quellverweise wurden vom Complete Guide auf Vault-Dateien umgestellt.

## Migrationsabdeckung

| Guide-Abschnitt | Vault-Ziel | Stand |
|---|---|---|
| 1. Was ist Normdex? | `01_Produkt/Produkt-Übersicht.md` | migriert |
| 2. Zielgruppe | `01_Produkt/Produkt-Übersicht.md` | migriert |
| 3. Unternehmensangaben | `05_Geschaeft/Unternehmensangaben.md` | migriert |
| 4. Brand Identity & Voice | `01_Produkt/Brand Identity & Voice.md` | migriert |
| 5. Designsystem & Farben | `01_Produkt/Designsystem & Farben.md` | migriert |
| 6. App-Architektur | `02_App/App-Architektur & Tech Stack.md` | migriert |
| 7. App-Routen & Seiten | `02_App/App-Routen.md` | migriert |
| 8. Funktionen im Detail | `02_App/Funktionen im Detail.md` und `02_App/Wirtschaftlichkeitsrechner.md` | migriert |
| 9. Authentifizierung & Sicherheit | `02_App/Authentifizierung & Sicherheit.md` | migriert |
| 10. Datenmodell | `02_App/Datenmodell.md` | migriert |
| 11. E-Mail-System & Benachrichtigungen | `02_App/E-Mail-System.md` | migriert |
| 12. API-Endpunkte | `02_App/API-Endpunkte.md` | migriert |
| 13. Tech Stack Web-App | `02_App/App-Architektur & Tech Stack.md` | migriert |
| 14. Versionierung & Deployment | `06_Entwicklung/Integrationen & externe Dienste.md` und App-Repo-Agentenhinweise | migriert |
| 15. Zweck der Landingpage | `03_Landingpage/Landingpage.md` | migriert |
| 16. Landingpage-Routen & SEO | `03_Landingpage/Landingpage.md` | migriert |
| 17. Header & Footer | `03_Landingpage/Landingpage.md` | migriert |
| 18. Homepage-Sektionen im Detail | `03_Landingpage/Landingpage.md` | migriert |
| 19. Alle Unterseiten im Detail | `03_Landingpage/Landingpage.md` | migriert |
| 20. Tech Stack Landingpage | `03_Landingpage/Landingpage.md` | migriert |
| 21. Marketing-Inhalte & Key Messages | `04_Marketing/Key Messages & CTAs.md` | migriert |
| 22. Integrationen & externe Dienste | `06_Entwicklung/Integrationen & externe Dienste.md` | migriert |

## Verifikation

- `Test-Path` bestätigt, dass die Guide-Dateien in App- und Landingpage-Repo nicht mehr existieren.
- `Test-Path` bestätigt, dass die Sync-Skripte in App- und Landingpage-Repo nicht mehr existieren.
- `rg "NORMDEX_COMPLETE_GUIDE|Complete Guide|sync-docs|on-guide-changed"` findet keine aktiven Arbeitsanweisungen oder Skripte mehr; übrig sind nur historische Besprechungsnotizen und dieses abgeschlossene Todo.

## Notizen

- Historische Besprechungsnotizen dürfen weiterhin erwähnen, dass der Guide früher migriert oder als veraltet markiert wurde.
- Vault bleibt die einzige maßgebliche Wissensbasis für die Inhalte des früheren Guides.
