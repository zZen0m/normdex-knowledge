# SEO-Analyse Landingpage (normdex.at) — Juli 2026

**Zugehöriges Todo:** [[T042-seo-analyse-landingpage]]
**Datum:** 2026-07-26
**Datenquellen:** Google Search Console (live abgefragt), Website-Code (`normdex-landingpage` Repo), Web-Recherche zu Konkurrenz. Plausible-Traffic-Daten: kein MCP-Zugriff in dieser Session, siehe Hinweis unten.

---

## 1. Ausgangslage (GSC, 90 Tage)

Property `https://normdex.at/` liefert für den Zeitraum 2026-04-27 bis 2026-07-26 nur **12 Suchanfragen**, **0 Klicks**. Fast alle Queries sind Marken-Tippfehler/-Verwechslungen, keine der Zielgruppen-Keywords ist vertreten:

| Query | Impressions | Position |
|---|---|---|
| umdex | 8 | 23 |
| normadex | 8 | 20,8 |
| niedex | 4 | 23 |
| ondex | 4 | 64 |
| rondex | 3 | 61,7 |
| norma dex | 2 | 18 |
| önorm | 2 | 31 |
| m7140 | 1 | 6 |
| buddex | 1 | 57 |
| nordex käserei | 1 | 32 |
| nordex österreich | 1 | 32 |
| onorm | 1 | 20 |

→ **Die Seite hat faktisch noch keine organische Sichtbarkeit** für fachliche Suchbegriffe. GSC dient hier nicht als Keyword-Ideenquelle, sondern als Baseline für späteres Progress-Tracking.

**Indexierungsstatus (Stichprobe):**
| URL | Status |
|---|---|
| `/oenorm-m-7140/` | indexiert |
| `/wissen/` | gecrawlt, aber **nicht indexiert** |
| `/wissen/die-drei-verfahren-erklaert/` | **Google unbekannt** (nie gecrawlt) |

Grund: Beide `/wissen`-URLs fehlten bis heute in `sitemap.xml` → **behoben** (siehe Abschnitt 4, Quick-Win #1).

---

## 2. Konkurrenzrecherche

Direkte Konkurrenz für ÖNORM-M-7140-Software:

- **Pokorny Technologies** ([pokorny-tec.at](https://www.pokorny-tec.at/haustechnik-normen-infos/wirtschaftlichkeits-oenorm-m-7140/)) — Desktop-Software, sehr ausführliche Norm-Erklärungsseite (Title: „ÖNORM M 7140 (gültig) • Pokorny Technologies"). Deckt Themen wie Annuität vs. durchschnittliche Kosten, Lebenszykluskosten, Sensitivitätsanalysen, Validierung von Softwareprodukten, Neuerungen 2021 vs. 2013 ab.
- **Urban-Energy** (urban-energy.at) — weiteres ÖNORM-M-7140-Tool, Seite nicht abrufbar (SSL-Zertifikatsfehler bei Fetch) — indirektes Signal, dass deren technisches Setup schwach ist (Chance für Normdex).
- **Austrian Standards Shop** — verkauft "Wirtschaftlichkeit Software Standard" direkt, rankt vermutlich für Norm-nahe Suchbegriffe.

**Content-Lücke identifiziert:** Pokorny behandelt das Thema „Annuität ≠ durchschnittliche Kosten" und „Validierung von Softwareprodukten nach ÖNORM M 7140" ausführlich — bei Normdex bislang nicht als eigener Fachbeitrag vorhanden. Anknüpfungspunkt für künftige `/wissen`-Artikel.

---

## 3. Keyword-Liste (priorisiert)

| # | Keyword | Intent | Priorität | Begründung |
|---|---|---|---|---|
| 1 | ÖNORM M 7140 Berechnung | Transactional | Hoch | Kernbegriff, direkt auf Produkt |
| 2 | ÖNORM M 7140 Software | Transactional | Hoch | Direkter Vergleich zu Pokorny/Urban-Energy |
| 3 | ÖNORM M 7140 online | Transactional | Hoch | Differenzierung ggü. Desktop-Konkurrenz (Normdex = Web-App) |
| 4 | Wirtschaftlichkeitsberechnung Energiesystem | Informational/Transactional | Hoch | Breiter Kernbegriff aus H1/Title |
| 5 | Kapitalwertmethode Energiesystem | Informational | Mittel | Fachbegriff aus Featureliste, gut für `/features` |
| 6 | Annuitätenmethode ÖNORM | Informational | Mittel | Content-Lücke ggü. Pokorny, Kandidat für `/wissen`-Artikel |
| 7 | Wärmepumpe Vergleich Wirtschaftlichkeit | Informational | Mittel | Zielgruppen-nahes Thema, hohes Suchinteresse laut Web-Recherche (ISFH-Tool, Wärmepumpen-Vergleichstools) |
| 8 | Amortisationsrechnung Energiesystem | Informational | Mittel | Deckt Amortisations-Feature ab |
| 9 | ÖNORM M 7140 2021 Neuerungen | Informational | Mittel | Content-Lücke, Pokorny bedient das bereits |
| 10 | Variantenvergleich Heizsystem ÖNORM | Informational | Niedrig | Long-Tail, Planer-Zielgruppe |
| 11 | ÖNORM M 7140 PDF Report | Transactional | Niedrig | Feature-spezifisch, geringes Volumen erwartet |
| 12 | Energieberater Software Österreich | Informational | Niedrig | Breiter Zielgruppenbegriff, hoher Wettbewerb |

**Hinweis:** Keine belastbaren Suchvolumen-Zahlen verfügbar (GSC zu datenarm, kein externes Volumen-Tool angebunden) — Priorisierung basiert auf Intent-Nähe zum Produkt und Konkurrenzsignalen, nicht auf Suchvolumen. Empfehlung: nach 2–3 Monaten mit GSC-Daten neu priorisieren.

---

## 4. On-Page-Analyse (9 Seiten)

| Seite | Title | H1 | Bewertung |
|---|---|---|---|
| `/` | „Normdex – ÖNORM M 7140 Wirtschaftlichkeitsberechnung" (ok) | „Im Browser. Im Team. Prüffähig." + „Wirtschaftlichkeitsberechnungen für Energiesysteme" | H1 zweigeteilt, Kernkeyword erst in Zeile 2 — funktioniert, aber Claim-Zeile dominiert |
| `/features` | ok (~50 Z.) | „Alle Normdex-Funktionen im Überblick" | ok |
| `/preise` | „Preise für Normdex \| ÖNORM M 7140 SaaS" (~40 Z., kurz) | „Einfache, transparente Preise" | Title-Länge ausbaufähig, H1 ohne Keyword |
| `/oenorm-m-7140` | ok (~54 Z.) | „ÖNORM M 7140" (sehr kurz) | H1 verschenkt Potenzial — „Wirtschaftlichkeitsberechnung" fehlt im H1, nur im Title |
| `/ueber-uns` | ok | „Über Normdex" | ok, wenig SEO-relevant |
| `/kontakt` | „Kontakt – Normdex Support \| Permatec e.U." (~42 Z., kurz) | „Nimm Kontakt auf" | ok, wenig SEO-relevant |
| `/wissen` | ok (~50 Z.) | „Fachbeiträge zur Wirtschaftlichkeitsberechnung" | gut |
| `/wissen/die-drei-verfahren-erklaert` | ok (~55 Z.) | „Die drei Verfahren erklärt" | H1 ohne „ÖNORM M 7140" — Keyword-Chance verschenkt |
| `/newsletter` | „Praxisleitfaden ÖNORM M 7140 – Normdex" (~39 Z., kurz) | dynamisch | ok |

**Interne Verlinkung:** `/wissen` ist von Header-Navigation und `/oenorm-m-7140` verlinkt — solide. Keine Duplicate/Thin-Content-Probleme gefunden.

---

## 5. Technisches SEO

| Punkt | Status |
|---|---|
| Sitemap vorhanden & in GSC gültig | ✅ Ja (13 URLs, 0 Fehler) — **/wissen-Seiten fehlten, jetzt ergänzt** |
| robots.txt | ✅ korrekt, erlaubt alle Bots, verweist auf Sitemap |
| Structured Data: Organization, WebSite | ✅ vorhanden (`getJsonLd`, `src/lib/seo.ts`) |
| Structured Data: SoftwareApplication (mit Preis) | ✅ auf `/` |
| Structured Data: FAQPage | ✅ auf `/oenorm-m-7140` |
| Structured Data: Article-Properties (datePublished, author) | ⚠️ `/wissen/die-drei-verfahren-erklaert` ist als `Article` typisiert, aber ohne Article-spezifische Felder (datePublished, author, image) — generischer WebPage-Fallback greift |
| Canonical URLs | ✅ vollständig in `seo-routes.json` |
| Hreflang | ✅ nicht nötig (nur de-AT), `inLanguage: de-AT` gesetzt |
| Core Web Vitals / PageSpeed | ⚠️ **Mobile-Ziel verfehlt** (siehe unten) |
| Indexierung `/wissen`-Seiten | ⚠️ noch nicht indexiert (siehe Abschnitt 1) — nach Sitemap-Fix in GSC „Indexierung beantragen" für beide URLs |

---

## 5a. PageSpeed Insights (`/`, 26.07.2026)

**Vor dem Fix (vormittags):**

| | Desktop | Mobile |
|---|---|---|
| Performance | 97 | **68** ⚠️ (Ziel It. Verifikation: >80) |
| FCP | 0,8 s | 4,1 s |
| LCP | 1,1 s | **6,0 s** (Ziel <2,5 s) |
| CLS | 0 | 0 |
| TBT | 0 ms | 0 ms |

**Nach dem Fix (14:44 Uhr, Re-Check):**

| | Desktop | Mobile |
|---|---|---|
| Performance | **100** ✅ | **91** ✅ (Ziel >80 erreicht) |
| Accessibility | 89 | 89 |
| Best Practices | 96 | 96 |
| SEO | 100 | 100 |
| FCP | 0,4 s | 1,7 s |
| LCP | 0,8 s | **3,5 s** |
| CLS | 0 | 0 |
| TBT | 0 ms | 0 ms |

→ **Akzeptanzkriterium erfüllt.** Gzip + Cache-Control allein haben den Mobile-Score von 68 auf 91 gehoben (LCP von 6,0s auf 3,5s, FCP von 4,1s auf 1,7s). Verbleibendes LCP-Einsparpotenzial laut Lighthouse: „Effiziente Verweildauer im Cache" nur noch 22 KiB (vorher 798 KiB) — der große Cache-Hebel ist gezogen.

**Root Cause:** Mobile (Slow-4G-Drosselung in Lighthouse) fällt durch, Desktop ist einwandfrei — das ist kein Content-, sondern ein Infra-Thema:

- **Keine Komprimierung auf der Dokumentanfrage** (Lighthouse markiert das explizit, ~65 KiB) — gzip/brotli am Server/Reverse-Proxy fehlt offenbar
- **Ineffizientes Cache-Verhalten** — 798 KiB geschätztes Einsparpotenzial (fehlende/kurze `Cache-Control`-Header auf `/static_assets/*`)
- **Ungenutztes JS/CSS**: ~192 KiB JS (v. a. `app-*.js`, `utils-*.js`, `react-dom-*.js`) + 54–56 KiB CSS
- LCP-Element-Rendering-Verzögerung: 620 ms (Desktop) / 1.330 ms (Mobile) — der Großteil der LCP-Zeit ist Verzögerung, nicht Ladezeit

**Zusätzlich (Accessibility/Best Practices, nicht kritisch für SEO-Ranking direkt, aber leicht zu fixen):**
- Logo-Bilder ohne explizite `width`/`height` (CLS-Risiko, aktuell noch bei 0)
- Fehlende `<main>`-Landmark, Überschriften-Reihenfolge nicht durchgängig
- Kein CSP/HSTS/COOP/X-Frame-Options-Header gesetzt (Security-Hardening, siehe evtl. eigenes Todo)

→ **Behoben und verifiziert (2026-07-26):** Fix-Ort war entgegen erster Vermutung **nicht** `normdex-app`, sondern das eigene `Dockerfile`/nginx in `normdex-landingpage` — der Container baut sein eigenes nginx (`FROM nginx:alpine`), Traefik davor komprimiert nicht. Neue `nginx.conf` mit `gzip on` + `Cache-Control`/`expires` für `/static_assets/` (1 Jahr, immutable, da hashed) und weitere statische Dateien (7 Tage). Deployt auf `normdex-vps` (`/opt/stacks/normdex-landingpage`, Rebuild + Recreate). Sowohl per `curl -I` (Gzip + Cache-Control aktiv) als auch per PageSpeed-Re-Check bestätigt: Mobile-Score 68 → **91**, Desktop 97 → **100** (Details siehe Abschnitt 5a).

**Restliche, kleinere Befunde aus dem PageSpeed-Report (nicht performance-kritisch, aber leicht behebbar):**
- Ungenutztes JS weiterhin ~33 KiB (`app-DPmitkGQ.js`) — Code-Splitting-Kandidat, kein akuter Handlungsbedarf
- Logo-`<img>`-Elemente ohne explizite `width`/`height` (CLS-Risiko, aktuell bei 0 unauffällig)
- Accessibility (89): ungültige `aria-selected`-Werte am Pricing-Tab, Kontrastproblem bei „Kostenlos testen"-Button, fehlende `<main>`-Landmark, Überschriften-Reihenfolge nicht durchgängig
- Best Practices (96): Charset-Deklaration zu spät im HTML, keine CSP/HSTS/COOP/X-Frame-Options-Header — Security-Hardening, eigenes Thema außerhalb SEO

## 6. Traffic-Daten (Plausible)

Per API-Key direkt gegen die Plausible Stats API (`/api/v2/query`) abgefragt. Tracking läuft erst seit **2026-07-24** — Datenbasis dementsprechend minimal:

| Metrik | Wert (24.–26.07.2026, gesamt) |
|---|---|
| Besucher | 2 |
| Pageviews | 3 |
| Bounce Rate | 50 % |
| Ø Besuchsdauer | 44 s |
| Traffic-Quelle | ausschließlich „Direct / None" — kein Referrer |
| Top-Seiten | `/oenorm-m-7140/` (1 Besucher), `/` (1 Besucher, 2 Pageviews) |
| `/wissen`-Seiten | 0 Aufrufe |

→ Bestätigt den Kernbefund aus Abschnitt 1: kein messbarer Traffic, organisch oder sonstig, vermutlich eigene Testaufrufe. Erneut prüfen, sobald der Sitemap-Fix (Abschnitt 4) zu Indexierung geführt hat und mehr Zeit vergangen ist — z. B. in 4 Wochen.

---

## 7. Priorisierte Verbesserungsvorschläge

| # | Maßnahme | Datei/Ort | Aufwand | Impact |
|---|---|---|---|---|
| 1 | ✅ **Erledigt:** `/wissen` + Artikel-Seite zur Sitemap hinzufügen | `public/sitemap.xml` | Klein | Hoch (Voraussetzung für Indexierung) |
| 2 | In GSC „Indexierung beantragen" für beide `/wissen`-URLs | GSC UI (manuell) | Klein | Hoch |
| 3 | H1 auf `/oenorm-m-7140` um „Wirtschaftlichkeitsberechnung" ergänzen | `src/pages/OenormM7140.tsx:313` | Klein | Mittel |
| 4 | H1 auf `/wissen/die-drei-verfahren-erklaert` um „ÖNORM M 7140" ergänzen | `src/pages/wissen/DieDreiVerfahrenErklaert.tsx:87` | Klein | Mittel |
| 5 | Article-JSON-LD um `datePublished`/`author`/`image` erweitern für `/wissen`-Artikel | `src/lib/seo.ts` (`getJsonLd`) | Mittel | Mittel (Rich-Snippet-Chance) |
| 6 | Neuer Fachbeitrag zu „Annuität vs. durchschnittliche Kosten" bzw. „ÖNORM M 7140 2021 Neuerungen" — Content-Lücke ggü. Pokorny | `/wissen/...` (neue Route) | Groß | Hoch (Long-Tail + Autorität) |
| 7 | Title-Tags von `/preise`, `/kontakt`, `/newsletter` verlängern (aktuell 39–42 Zeichen, Potenzial bis 60) | `src/pages/seo-routes.json` | Klein | Niedrig-Mittel |
| 8 | ✅ **Erledigt & verifiziert:** Gzip-Kompression + Cache-Control-Header im nginx-Container aktiviert — Mobile-Score 68→91 | `normdex-landingpage/nginx.conf`, `Dockerfile` | Mittel | Hoch (bestätigt größter identifizierter Hebel) |
| 9 | Security-Header ergänzen (CSP, HSTS, COOP, X-Frame-Options) — laut PageSpeed alle mit Schweregrad „Hoch" markiert | `normdex-landingpage/nginx.conf` | Mittel | Mittel (Best-Practices-Score, nicht SEO-relevant, aber sinnvoll) |
| 10 | Accessibility-Fixes: `<main>`-Landmark ergänzen, `aria-selected` am Pricing-Tab korrigieren, Kontrast beim „Kostenlos testen"-Button prüfen | `normdex-landingpage/src` (Layout, Pricing, Button-Varianten) | Klein | Niedrig (kein direkter SEO-Effekt, aber Nutzerfreundlichkeit) |

---

## 8. Offene Punkte für T042-Abschluss

- [x] PageSpeed-Score erhoben und Root Cause behoben (Mobile 68→91)
- [ ] Plausible-Traffic erneut prüfen, sobald mehr Daten vorliegen (z. B. in 4 Wochen)
- [ ] Maßnahme #6 (neuer Fachbeitrag) als eigenes Folge-Todo anlegen, falls priorisiert
- [ ] Maßnahme #9 (Security-Header) und #10 (Accessibility-Fixes) ggf. als eigenes technisches Todo auslagern — außerhalb des SEO-Scopes von T042
