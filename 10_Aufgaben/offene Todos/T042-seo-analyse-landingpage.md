# T042 · SEO-Analyse Landingpage + Verbesserungsvorschläge

**Phase:** Landingpage / Marketing / SEO
**Priorität:** P2 · Reichweite – kurzfristig umsetzbar
**Status:** in Arbeit
**Datum:** 2026-07-08

## Ziel

Vollständige SEO-Analyse der bestehenden Landingpage (normdex.at) mit konkreten, priorisierten Verbesserungsvorschlägen, um organische Sichtbarkeit für die relevanten Suchanfragen der Zielgruppe (Energieberater, TGA-Ingenieure, Planer in Österreich) zu erhöhen.

## Kontext

Die Landingpage hat aktuell kaum organischen Traffic. Google Analytics (G-WGPWWKYJRW) ist eingebunden, aber die Daten wurden noch nicht systematisch ausgewertet. Die wichtigsten SEO-Hebel sind:
- On-Page-Optimierung der bestehenden Seiten
- Keyword-Ausrichtung auf echte Suchanfragen der Zielgruppe
- Technisches SEO (Meta-Tags, Structured Data, Canonical URLs, Sitemaps)
- Content-Lücken identifizieren (Themen, die gesucht werden, aber noch fehlen)

Bestehende Infrastruktur im Repo: `src/lib/seo-routes.json` + Build-Script `scripts/generate-route-html.mjs` erzeugen statisches HTML pro Route. `<Seo>`-Komponente setzt Title/Description/OG-Tags per `react-helmet-async`. Die Seite `/oenorm-m-7140` ist inhaltlich stark, aber noch nicht für Long-Tail-Keywords optimiert.

## Umsetzung

### 1. Keyword-Recherche (Grundlage für alles weitere)
- Ziel-Keywords der Zielgruppe ermitteln: „ÖNORM M 7140 Berechnung", „Wirtschaftlichkeitsberechnung Energiesystem Österreich", „Wärmepumpe Vergleich Kapitalwertmethode" etc.
- Suchvolumen + Wettbewerb prüfen (Google Search Console, Ubersuggest o. ä.)
- Keywords priorisieren: High-Intent (kaufbereit) vs. Informational (Fachbeitrag-Kandidaten)

### 2. On-Page-Analyse aller bestehenden Seiten
Seiten analysieren: `/`, `/features`, `/preise`, `/oenorm-m-7140`, `/ueber-uns`, `/kontakt`
Pro Seite prüfen:
- Title-Tag (Länge 50–60 Zeichen, Keyword vorne)
- Meta-Description (130–160 Zeichen, Call-to-Action)
- H1 eindeutig + keyword-relevant
- Interne Verlinkungsstruktur
- Duplicate Content / Thin Content

### 3. Technisches SEO
- Sitemap vorhanden und korrekt? (aktuell keine `sitemap.xml` im Build)
- `robots.txt` prüfen
- Structured Data (Schema.org): SoftwareApplication, Organization
- Core Web Vitals (LCP, CLS, FID) via PageSpeed Insights prüfen
- Canonical-URLs vollständig in `seo-routes.json`?
- Hreflang (nur de-AT relevant)

### 4. Verbesserungsvorschläge dokumentieren
Priorisierte Liste mit:
- Konkrete Änderung (was genau)
- Betroffene Datei / Route
- Aufwand (Klein / Mittel / Groß)
- Erwarteter Impact

## Akzeptanzkriterien

- [x] Keyword-Liste mit mind. 10 priorisierten Ziel-Keywords (Suchvolumen nicht verfügbar, Priorisierung nach Intent/Konkurrenzsignalen)
- [x] On-Page-Bewertung aller Hauptseiten (auf 9 Seiten erweitert, inkl. `/wissen`-Bereich aus T037)
- [x] Technische SEO-Checkliste abgehakt (PageSpeed-Score offen, siehe Notizen)
- [x] Verbesserungsvorschläge als priorisierte Liste (8 konkrete Maßnahmen)
- [x] Quick-Win identifiziert und direkt umgesetzt: fehlende `/wissen`-Seiten in `public/sitemap.xml` ergänzt (Commit ausstehend)

## Verifikation

- Google PageSpeed Insights Score für normdex.at > 80 (Mobile)
- Google Search Console zeigt keine Critical Issues
- Alle Title-Tags und Meta-Descriptions innerhalb empfohlener Zeichenlänge

## Notizen / Fortschritt

- 2026-07-08: Todo angelegt. Hintergrund: Landingpage hat kaum organischen Traffic, LinkedIn-Account steht bereits, Fokus jetzt auf SEO als zweitem Reichweiten-Kanal. Analyse soll Grundlage für [[T037-fachbeitrag-content-struktur-landingpage]] und [[T043-oenorm-seite-inhaltlicher-ausbau]] liefern.
- 2026-07-26: Todo geschärft und Analyse durchgeführt, siehe vollständigen Report: [[SEO-Analyse-2026-07]]. Kernfunde: GSC liefert nach 90 Tagen nur 12 Marken-Tippfehler-Queries, 0 Klicks — keine Keyword-Opportunities aus GSC ableitbar, daher Keyword-Liste manuell aus Fachwissen + Konkurrenzrecherche (Pokorny Technologies, Urban-Energy) erstellt. Kritischer Fund: `/wissen`-Seiten (aus T037) fehlten in `public/sitemap.xml` und waren dadurch nicht/kaum indexiert — als Sofort-Fix behoben (Änderung noch nicht committet). Offen: PageSpeed-Score manuell erheben, Plausible-Traffic-Auswertung manuell nachholen (kein MCP-Zugriff), Content-Lücke „Annuität vs. durchschnittliche Kosten" als möglicher Folge-Fachbeitrag.
