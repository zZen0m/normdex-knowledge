# Session – Vault-Aufbau & Lizenzsystem-Planung

**Datum:** 24. April 2026

---

## Überblick

Erste strukturierte Session zur Einrichtung des Obsidian-Vaults als zentrale Wissensbasis und zur Vorbereitung der Lizenzsystem-Umgestaltung.

---

## Erledigte Aufgaben

### 1. Workspace-Struktur dokumentiert

Die neue `D:\Normdex`-Ordnerstruktur wurde analysiert und festgehalten:
- `01_repos/` — normdex-app + normdex-landingpage (Git-Repos)
- `02_knowledge/` — normdex-vault (Obsidian, Git-Repo)
- `03_external/` — Symlink zum SharePoint-Ordner
- `04_workspace/` — `normdex.code-workspace`

### 2. CLAUDE.md-Dateien aktualisiert

**normdex-app/CLAUDE.md:**
- Stack-Angabe von „Vue 3" auf „React 18 + TypeScript" korrigiert
- Skill-Pfad von `.agent/skills/` auf `.claude/skills/` korrigiert
- Workspace-Kontext-Tabelle ergänzt
- Verweis auf Vault als Single Source of Truth, Guide als veraltet markiert

**normdex-landingpage/CLAUDE.md:**
- Workspace-Kontext-Tabelle ergänzt
- Verweis auf Vault als Dokumentationsquelle ergänzt

### 3. Branding-Guidelines in Vault erstellt

- `01_Produkt/Brand Identity & Voice.md` — Markenpositionierung, Voice-Prinzipien, Anrederegel (**Du**-Form, außer bei direktem Support-Kontakt)
- `01_Produkt/Designsystem & Farben.md` — alle Color Tokens (App + Landingpage), Typografie, Gradienten, Schatten, E-Mail-Schema

### 4. NORMDEX_COMPLETE_GUIDE vollständig in Vault migriert

Alle 22 Abschnitte wurden auf folgende Vault-Dateien verteilt:

| Datei | Inhalt |
|---|---|
| `01_Produkt/Produkt-Übersicht.md` | Was ist Normdex, Zielgruppe |
| `02_App/App-Architektur & Tech Stack.md` | Architektur, Technologien, Deployment, Env-Vars |
| `02_App/App-Routen.md` | Alle Routen (öffentlich, Auth, App, Admin) |
| `02_App/Wirtschaftlichkeitsrechner.md` | Kernprodukt, ÖNORM M 7140-Umsetzung |
| `02_App/Funktionen im Detail.md` | Projekte, Team, Lizenzen, Support, Admin |
| `02_App/Authentifizierung & Sicherheit.md` | JWT, Argon2, Cookie-Policy |
| `02_App/Datenmodell.md` | Alle Entities |
| `02_App/E-Mail-System.md` | Brevo, Mailvorlagen, Graph API |
| `02_App/API-Endpunkte.md` | Vollständige Endpoint-Referenz |
| `03_Landingpage/Landingpage.md` | Zweck, Tech Stack, Routen, Sektionen, Pricing |
| `04_Marketing/Key Messages & CTAs.md` | Botschaften, Trust-Signals, CTAs, SEO-Keywords |
| `05_Geschaeft/Unternehmensangaben.md` | Firmendaten, Rechtliches |
| `06_Entwicklung/Integrationen & externe Dienste.md` | Stripe, Graph API, Brevo, n8n, reCAPTCHA |

Der Guide (`docs/NORMDEX_COMPLETE_GUIDE.md`) wurde mit Deprecation-Banner versehen. Löschen steht als offene Aufgabe in `10_Aufgaben/Aufgaben.md`.

### 5. To-do-System eingerichtet

- Ordner `10_Aufgaben/` angelegt
- `Aufgaben.md` als zentrale Aufgabenliste mit Status-Symbolen (`[ ]`, `[x]`, `[~]`, `[-]`)
- Bereits erledigte Schritte dieser Session eingetragen

### 6. Lizenzsystem-Spec durchgearbeitet

Die Datei `10_Aufgaben/normdex_lizenzsystem_developer_spec.md` wurde gelesen und bewertet.

**Konzept:** Umbau des Lizenzsystems auf ein Pool-Modell mit Sammelabrechnung über Stripe.

**Kernpunkte:**
- Zwei getrennte Billing-Pools pro Organisation: `monthly` und `yearly`
- Jede Lizenz = eigener Datensatz in Normdex, gebündelt in Stripe als Subscription Item
- Staffelpreise: Basislizenz (49 €/Monat, 490 €/Jahr), Add-on (29 €/Monat, 290 €/Jahr)
- Kündigung: nur Admin/Owner, Add-on-Lizenzen zuerst, Zugriff bleibt bis Laufzeitende
- Rebasierung: wenn Basislizenz endet und Add-ons verbleiben → ältestes Add-on wird Basis
- Mehrfachkauf in einem Schritt, Live-Preisvorschau im Kaufdialog
- Rabattcodes + interne Complimentary-Lizenzen

**Bewertung:** Technisch vollständig durchdacht und umsetzbar. Größter Aufwand: Datenmigration (Phase 1) und Preislogik + Rebasierung (Phase 3).

Alle 40 Implementierungsaufgaben (7 Phasen) wurden in `10_Aufgaben/Aufgaben.md` eingetragen. **Implementierung noch nicht gestartet** — vorerst nur Konzept- und Planungsphase.

---

## Offene Punkte aus dieser Session

- `docs/NORMDEX_COMPLETE_GUIDE.md` löschen, sobald sichergestellt ist, dass alles vollständig migriert ist
- Vault-Inhalte sukzessive weiter ausbauen (leere Bereiche in `01_Produkt`, `06_Entwicklung`, `07_KI_Agenten`, etc.)
- Lizenzsystem-Implementierung zu gegebener Zeit starten (Phase 1: Datenmodell)
