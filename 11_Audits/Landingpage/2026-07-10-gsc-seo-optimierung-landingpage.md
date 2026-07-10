# GSC-Analyse & SEO-Optimierung - Landingpage - 2026-07-10

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-landingpage`
- Branch: `develop`
- Umsetzungs-Commit: `bee87bb` (feat(seo): fix canonicalization and establish Normdex brand entity)
- GSC-Properties: `https://normdex.at/` (siteOwner), `https://www.normdex.at/` (siteOwner), `sc-domain:normdex.at` (**unverifiziert**)
- Analysezeitraum: 2026-04-11 bis 2026-07-10 (90 Tage)
- Datum: 2026-07-10
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Landingpage\2026-07-10-gsc-seo-optimierung-landingpage.md`

## 2. Datengrundlage (Google Search Console)

Der gesamte Traffic läuft über die **non-www-Property** `https://normdex.at/`. Die www-Property hat 0 Klicks / 1 Impression (korrekt, non-www ist kanonisch).

| Kennzahl (90 Tage) | Wert |
|---|---|
| Klicks | 18 |
| Impressionen | 114 |
| CTR | 15,8 % |
| Ø Position | 9,4 |

Einordnung: Sehr kleine Zahlen — Landingpage erst seit ~2026-06-07 live ([[project_v0_1_0_live]]). Aussagen sind richtungsweisend, nicht statistisch belastbar. Traffic wächst ab Ende Juni sichtbar.

### Top-Seiten

| Seite | Klicks | Impr. | CTR | Pos. |
|---|---|---|---|---|
| `/` (Startseite) | 14 | 56 | 25 % | 13,0 |
| `/oenorm-m-7140` | 3 | 65 | 4,6 % | 6,1 |
| `/ueber-uns` | 1 | 12 | 8,3 % | 2,5 |
| `/datenschutz` | 0 | 15 | 0 % | 6,7 |

### Suchbegriffe

Alle sichtbaren Queries sind **Tippfehler-/Fuzzy-Varianten von „normdex"** (buddex, niedex, nordex, ondex, umdex, normadex, rondex). Fast keine thematische Sichtbarkeit zu den Fachthemen. Geräte: Mobile konvertiert besser (CTR 30,8 % vs. Desktop 11,4 %). Länder: 91/114 Impressionen aus Österreich.

## 3. Identifizierte Kernprobleme

### 3.1 Trailing-Slash-Canonical-Konflikt (technisch, hoch)

Der Prod-Server (nginx `try_files $uri $uri/`) leitet jede Route **ohne** Slash per 301 auf die Slash-Variante um (`/oenorm-m-7140` → `/oenorm-m-7140/`). Aber `canonical`, `sitemap.xml` und alle internen Links verwendeten die **No-Slash-Form**.

GSC-Beleg via URL-Inspektion:
- `/oenorm-m-7140` → „Page with redirect", `google_canonical = /oenorm-m-7140/`, `user_canonical = /oenorm-m-7140` (Widerspruch)
- `/oenorm-m-7140/` → „Submitted and indexed"

Folge: Google ignoriert das deklarierte Canonical, Ranking-Signale werden auf zwei URL-Varianten gesplittet.

### 3.2 Fehlende Marken-Entität → „normadex"-Autokorrektur (strategisch, hoch)

Google kennt „Normdex" nicht als eigenständige Entität und korrigiert die Suche automatisch zu „normadex". Ursache im Markup: Als `Organization` war ausschließlich „Permatec e.U." hinterlegt; die Marke „Normdex" existierte strukturell nur als Software-Name (`SoftwareApplication.name`). Es fehlte das Entitäts-Signal.

### 3.3 Weitere Findings

- **Body nicht server-seitig gerendert:** Der Prerender (`scripts/generate-route-html.mjs`) schreibt nur den `<head>`; `<div id="root">` bleibt leer. Content-Indexierung hängt an Googles JS-Rendering.
- **Niedrige CTR der Money-Page** `/oenorm-m-7140` (4,6 % bei Position 6,1): Meta-Description zu passiv.
- **OG-/Twitter-Bild in `index.html`** zeigte auf das SVG-Logo (von Social-Scrapern nicht gerendert).
- **Domain-Property `sc-domain:normdex.at` unverifiziert.**

## 4. Umgesetzte Maßnahmen (Commit `bee87bb`)

| # | Maßnahme | Dateien |
|---|---|---|
| 1 | Canonicals, Sitemap und **alle** internen Route-Links auf Trailing-Slash-Form vereinheitlicht (die Google bereits als kanonisch gewählt hat); Header-Active-State gegen Slashes robust gemacht | `seo-routes.json`, `sitemap.xml`, `Header.tsx`, `Footer.tsx`, `CTA.tsx`, `Hero.tsx`, `Pricing.tsx`, `Contact.tsx`, `Datenschutz.tsx`, `AGB.tsx`, `CookieConsent.tsx`, `NewsletterStrip.tsx`, `ReportPreview.tsx` |
| 2 | Marken-Entität „Normdex" als `@graph` mit **`Organization` + `WebSite`** (Name, `legalName` Permatec, Logo, `sameAs` LinkedIn, Adresse, E-Mail) — konsistent in allen drei Quellen | `seo.ts`, `generate-route-html.mjs`, `index.html` |
| 3 | ÖNORM-M-7140-Meta-Description nutzenorientiert geschärft (CTR) | `seo-routes.json` |
| 4 | Sichtbare **FAQ-Sektion (6 Fragen) + `FAQPage`-Schema** auf der ÖNORM-Seite | `OenormM7140.tsx` |
| 5 | OG-/Twitter-Bild in `index.html` von SVG auf OG-PNG korrigiert (inkl. Dimensionen/Alt) | `index.html` |
| 6 | Test für Marken-Entität + Trailing-Slash-Canonicals ergänzt/angepasst | `seo.test.ts` |

**Verifikation:** 12/12 Vitest-Tests grün, `npm run build` inkl. Prerender erfolgreich, ESLint sauber. Prerendertes HTML bestätigt: Trailing-Slash-Canonicals und `@graph` (Organization/WebSite/SoftwareApplication bzw. WebPage) korrekt erzeugt. Durchgängige Schreibweise **ÖNORM** ([[feedback_oenorm_uppercase]]).

## 5. Offene Empfehlungen (nicht im Code gelöst)

1. **Server-seitiges Rendering des Body** (z. B. `vite-react-ssg` oder `react-snap`) — größter struktureller SEO-Hebel; eigene Architektur-Aufgabe mit Test.
2. **Off-Page-Marken-Signale:** Verzeichnis-/Branchen-Einträge mit exakt „Normdex" (WKO, Herold, Firmen-A1), LinkedIn-Aktivität, erste Backlinks — beschleunigt die Entitäts-Erkennung und beendet die „normadex"-Autokorrektur.
3. **`sc-domain:normdex.at` in GSC verifizieren** (konsolidiertes Reporting über www + non-www).
4. **Thematischen Content ausbauen:** eigene Seiten für weitere Suchnachfrage (Heizsystem-Vergleich, Wärmepumpe vs. Pellets/Gas, Lebenszykluskosten).
5. **Nach Deployment:** URL-Prüfung für `https://normdex.at/oenorm-m-7140/` neu anstoßen und Sitemap in GSC neu einreichen.

## 6. Nächste Messung

Wirkung frühestens 2-4 Wochen nach Deployment in GSC prüfen: Verschwinden der No-Slash-Dubletten aus dem Index, Entwicklung der Impressionen für „normdex" (statt „normadex") und CTR der ÖNORM-Seite.
