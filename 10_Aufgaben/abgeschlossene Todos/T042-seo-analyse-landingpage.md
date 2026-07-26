# T042 · SEO-Analyse Landingpage + Verbesserungsvorschläge

**Phase:** Landingpage / Marketing / SEO
**Priorität:** P2 · Reichweite – kurzfristig umsetzbar
**Status:** erledigt
**Datum:** 2026-07-08
**Abgeschlossen:** 2026-07-26

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
- [x] Technische SEO-Checkliste abgehakt (inkl. PageSpeed-Score, siehe Notizen)
- [x] Verbesserungsvorschläge als priorisierte Liste (8 konkrete Maßnahmen)
- [x] Quick-Win identifiziert und direkt umgesetzt: fehlende `/wissen`-Seiten in `public/sitemap.xml` ergänzt (Commit ausstehend)

## Verifikation

- Google PageSpeed Insights Score für normdex.at > 80 (Mobile) — **erfüllt.** Vormittags 2026-07-26: Mobile 68, Desktop 97. Root Cause identifiziert und noch am selben Tag behoben: fehlendes Gzip + Cache-Control im nginx-Container von `normdex-landingpage` (nicht `normdex-app`, wie zunächst fälschlich angenommen). Re-Check 14:44 Uhr: **Mobile 91, Desktop 100** (LCP 6,0s → 3,5s, FCP 4,1s → 1,7s).
- Google Search Console zeigt keine Critical Issues
- Alle Title-Tags und Meta-Descriptions innerhalb empfohlener Zeichenlänge

## Notizen / Fortschritt

- 2026-07-08: Todo angelegt. Hintergrund: Landingpage hat kaum organischen Traffic, LinkedIn-Account steht bereits, Fokus jetzt auf SEO als zweitem Reichweiten-Kanal. Analyse soll Grundlage für [[T037-fachbeitrag-content-struktur-landingpage]] und [[T043-oenorm-seite-inhaltlicher-ausbau]] liefern.
- 2026-07-26: Todo geschärft und Analyse durchgeführt, siehe vollständigen Report: [[SEO-Analyse-2026-07]]. Kernfunde: GSC liefert nach 90 Tagen nur 12 Marken-Tippfehler-Queries, 0 Klicks — keine Keyword-Opportunities aus GSC ableitbar, daher Keyword-Liste manuell aus Fachwissen + Konkurrenzrecherche (Pokorny Technologies, Urban-Energy) erstellt. Kritischer Fund: `/wissen`-Seiten (aus T037) fehlten in `public/sitemap.xml` und waren dadurch nicht/kaum indexiert — als Sofort-Fix behoben (Änderung noch nicht committet). Offen: PageSpeed-Score manuell erheben, Content-Lücke „Annuität vs. durchschnittliche Kosten" als möglicher Folge-Fachbeitrag.
- 2026-07-26: Plausible-Traffic per Stats API (v2/query, API-Key vom User erzeugt) direkt abgefragt statt manuell nachzuschauen. Ergebnis: Tracking läuft erst seit 24.07., nur 2 Besucher/3 Pageviews gesamt, ausschließlich Direct-Traffic, 0 Aufrufe auf `/wissen`-Seiten — bestätigt GSC-Befund (noch kein messbarer Traffic). Erneute Prüfung in ca. 4 Wochen sinnvoll. Report aktualisiert: [[SEO-Analyse-2026-07]].
- 2026-07-26: PageSpeed Insights für `/` vom User beigesteuert (Desktop + Mobile). Desktop top (97), Mobile fällt mit 68 durch das Akzeptanzkriterium (>80). Root Cause: fehlende Kompression + ineffizientes Caching, nicht Content-bedingt. Erste Vermutung (Fix gehört zu `normdex-app`) war falsch — verifiziert per Dockerfile-Check: `normdex-landingpage` baut sein eigenes `nginx:alpine`-Image, Traefik davor komprimiert nicht.
- 2026-07-26: Fix noch am selben Tag umgesetzt und deployt: neue `nginx.conf` (Gzip + Cache-Control/expires für `/static_assets/` und weitere statische Dateien) in `normdex-landingpage`, `Dockerfile` darauf umgestellt. Merge `develop` → `main`, gepusht, auf `normdex-vps` per SSH gepullt, `docker compose build && up -d` im Stack `/opt/stacks/normdex-landingpage`. Verifiziert per `curl -I`: Gzip + `Cache-Control: public, immutable, max-age=31536000` aktiv.
- 2026-07-26, 14:44 Uhr: PageSpeed-Re-Check vom User beigesteuert — bestätigt die Wirkung eindeutig: Mobile-Score 68 → **91**, Desktop 97 → **100**, Mobile-LCP 6,0s → 3,5s, Mobile-FCP 4,1s → 1,7s. Akzeptanzkriterium erfüllt. **T042 damit vollständig abgeschlossen.** Verbleibend nur die spätere Plausible-Nachprüfung in ~4 Wochen (kein Blocker) sowie zwei neu identifizierte, nicht-kritische Folgepunkte (Security-Header, kleinere Accessibility-Fixes), die bei Bedarf als eigenes Todo aufgesetzt werden können.
