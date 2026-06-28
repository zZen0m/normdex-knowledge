# Repo-Audit - Landingpage - 2026-06-28

## 1. Geprüfter Stand

- Repository: `/opt/repos/normdex-landingpage` (entspricht `D:\Normdex\01_repos\normdex-landingpage`)
- Branch: `main`
- Commit: `da0f1d4`
- Version laut `package.json`: `0.0.4`
- Letzter Git-Tag: `v0.0.4`
- Audit-Datum: `2026-06-28`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Landingpage\2026-06-28-repo-audit-landingpage.md`
- Vorheriger Audit: `2026-06-14-repo-audit-landingpage.md`

## 2. Audit-Abdeckung

- Statische Codeanalyse mit Schwerpunkt Marketing/SEO auf expliziten Wunsch des Auftraggebers: `index.html`, `App.tsx`, alle Routen in `src/pages/`, SEO-relevante Komponenten (`Hero`, `Features`, `Footer`, `Header`, `ReportPreview`)
- Prüfung von Title-Tags, Meta-Descriptions, Canonical-URLs, Open-Graph-/Twitter-Tags, JSON-LD, `robots.txt`, `sitemap.xml`, Heading-Struktur (H1/H2), Alt-Texten und Sprachkonsistenz (Du-/Sie-Form)
- ESLint-Prüfung (`npx eslint .`)
- Abgleich mit dem Vorbericht vom 2026-06-14 inkl. Re-Verifikation der vier offenen technischen Findings (`tsconfig.app.json`, fehlende Tests, Hero-Mock-Daten, Fast-Refresh-Warnungen)
- Produktionsbuild konnte in dieser Sitzung **nicht** ausgeführt werden: `dist/` gehört `root` und ist für den aktuellen Benutzer nicht beschreibbar (`EACCES`). Dies ist ein Umgebungsproblem dieser Sitzung, kein Code-Fehler, und wurde nicht weiter untersucht.
- Kein visueller Browser-Test, kein Social-Share-Preview-Test (z. B. via Facebook Sharing Debugger/Twitter Card Validator), kein Lighthouse-/PageSpeed-Lauf möglich

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-06-14
- Findings im Vorbericht: 4
- Davon behoben: 4
- Davon weiterhin offen: 0
- Davon nicht abschließend verifizierbar: 0
- Regressionen: 0
- Findings dieses Audits insgesamt: 8
- Davon behoben: 8
- Davon weiterhin offen: 0
- Umsetzungsnachtrag 2026-06-28: Alle acht Findings dieses Audits wurden im Landingpage-Repo behoben und lokal verifiziert.

### Behobene Findings

- Finding 1 - Open-Graph/Twitter/JSON-LD pro Route: **behoben**. Evidenz: zentraler SEO-Datenbestand, Post-Build-Route-HTML-Generator, `npm run build` erzeugt 12 route-spezifische HTML-Dateien mit eigenen OG-/Twitter-/Canonical-/JSON-LD-Werten.
- Finding 2 - `/features` ohne H1: **behoben**. Evidenz: `FeaturesPage.tsx` enthält ein sichtbares H1 "Alle Normdex-Funktionen im Überblick"; Test `FeaturesPage.test.tsx` deckt es ab.
- Finding 3 - zu lange Title-/Description-Tags: **behoben**. Evidenz: zentrale SEO-Texte in `seo-routes.json`; Tests prüfen SERP-freundliche Längen der priorisierten Routen.
- Finding 4 - Sie-Form in Marketing-CTAs: **behoben**. Evidenz: CTA-Texte in `About.tsx` und `OenormM7140.tsx` wurden auf Du-Form umgestellt.
- Finding 5 - SPA-404 ohne `noindex` / Soft-404-Risiko: **behoben**. Evidenz: `NotFound.tsx` setzt `noindex`; `Dockerfile` nutzt `try_files $uri $uri/ =404`.
- Finding 6 - `sitemap.xml` ohne `<lastmod>`: **behoben**. Evidenz: alle 12 Sitemap-URLs enthalten `<lastmod>2026-06-28</lastmod>`.
- Finding 7 - TypeScript-Strictness deaktiviert: **behoben**. Evidenz: `tsconfig.app.json` setzt `"strict": true` und `"noImplicitAny": true`; `npx tsc -p tsconfig.app.json --noEmit` läuft grün.
- Finding 8 - Keine automatisierten Tests: **behoben**. Evidenz: `package.json` enthält `test: vitest run`; 3 Testdateien mit 8 Tests laufen grün.

### Weiterhin offene Findings

- Keine.

### Regressionen

- Keine.

## 4. Confidence

- Confidence: hoch

Alle Marketing-/SEO-Findings sind direkt aus `index.html`, den Routen-Komponenten in `src/pages/` und `public/` belegt. Der fehlgeschlagene Produktionsbuild schränkt nur die Bündelgrößenprüfung ein, nicht die SEO-/Content-Analyse. Aussagen zu Crawler-Verhalten (z. B. wie Social-Media-Crawler mit der reinen Client-Side-Renderingstrategie umgehen) sind technisch fundiert, aber ohne Live-Test mit den jeweiligen Debug-Tools nicht zu 100 % verifiziert.

## 5. Kurzfazit

Die Landingpage hat ein solides SEO-Grundgerüst: Jede Hauptroute hat eigenen Title, eigene Meta-Description und eigenes `<link rel="canonical">` über `react-helmet-async`, dazu eine vollständige `sitemap.xml`, ein einladendes `robots.txt` und ein Basis-JSON-LD-Schema. Das größte marketingrelevante Problem liegt jedoch in der Architektur selbst: Die Seite ist eine reine Client-Side-React-App ohne Pre-Rendering, und Open-Graph-, Twitter-Card- und JSON-LD-Daten liegen ausschließlich statisch in `index.html`. Da die meisten Social-Crawler (Facebook, LinkedIn, Twitter/X, WhatsApp, Slack) kein JavaScript ausführen, zeigt jede geteilte Unterseite (`/preise`, `/oenorm-m-7140`, `/newsletter` usw.) denselben generischen Homepage-Titel und dieselbe Homepage-Beschreibung statt eigener, zielgerichteter Inhalte. Daneben fehlt der `/features`-Route ein H1, mehrere Title-Tags und die Meta-Description der zentralen ÖNORM-Seite überschreiten die in Google-Snippets sichtbare Länge deutlich, und zwei Marketingseiten (`About.tsx`, `OenormM7140.tsx`) brechen die im Projekt sonst durchgängige Du-Form. Verglichen mit dem letzten Audit vom 2026-06-14 hat sich an den vier dort offenen technischen Findings nichts verändert; aus Marketing-/SEO-Sicht wurden in diesem Lauf vier neue, bislang nicht dokumentierte Findings identifiziert.

## 6. Wichtigste Findings

1. **[behoben][hoch][Bug] Open-Graph/Twitter-Card/JSON-LD sind seitenweit identisch, da rein client-seitig per Helmet gesetzt und ohne Pre-Rendering.** Behoben durch route-spezifisches Post-Build-HTML.
2. **[behoben][hoch][Bug] `/features` hat kein `<h1>`.** Behoben durch sichtbares H1 auf der Route.
3. **[behoben][mittel][Improvement] Mehrere Title-Tags und die Meta-Description der ÖNORM-Seite überschreiten die in SERPs sichtbare Länge.** Behoben durch zentrale, gekürzte SEO-Texte.
4. **[behoben][mittel][Improvement] Sie-Form auf zwei Marketingseiten (`About.tsx`, `OenormM7140.tsx`) widerspricht der dokumentierten Du-Form-Konvention.** Behoben durch Umstellung auf Du-Form.
5. **[behoben][mittel][Risk] SPA-404 ohne `noindex`-Meta-Tag und ohne im Repo erkennbare Server-Rewrite-Konfiguration.** Behoben durch `noindex` und Nginx-404-Konfiguration.
6. **[behoben][niedrig][Improvement] `sitemap.xml` enthält keine `<lastmod>`-Angaben.** Behoben durch `<lastmod>` für alle Sitemap-URLs.

## 7. Detaillierte Findings je Punkt

### Finding 1 - Open-Graph/Twitter/JSON-LD nur statisch in `index.html`, kein Pre-Rendering

- Status 2026-06-28: **behoben** durch zentrale SEO-Metadaten, `scripts/generate-route-html.mjs` und route-spezifische `dist/<route>/index.html`-Dateien im Build.
- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `index.html:16-81`, `vite.config.ts`, `src/App.tsx`
- Evidenz: `index.html` enthält feste `og:title`, `og:description`, `og:image`, `twitter:*` und ein einziges JSON-LD-`SoftwareApplication`-Schema, jeweils mit Homepage-Inhalten. `vite.config.ts` enthält kein Pre-Rendering-/SSG-Plugin (kein `vite-plugin-ssr`, kein `react-snap` o. ä.), und `App.tsx` rendert ausschließlich client-seitig über `BrowserRouter`. Die Seiten-Komponenten setzen über `react-helmet-async` nur `<title>`, `<meta name="description">` und `<link rel="canonical">` — nie `og:title`, `og:description` oder `og:image`.
- Beschreibung des Problems: Helmet aktualisiert den `<head>` erst nach dem Laden und Ausführen von React im Browser. Crawler, die kein JavaScript ausführen (Facebook-, LinkedIn-, Twitter/X-, WhatsApp- und Slack-Bots gehören klassisch dazu), lesen ausschließlich das initiale, statische HTML. Für jede URL der Domain ist das exakt derselbe `index.html`-Inhalt.
- Warum das relevant ist: Beim Teilen von `/oenorm-m-7140`, `/preise`, `/newsletter` oder `/features` in sozialen Netzwerken oder Messengern erscheint immer die generische Homepage-Karte „Normdex – ÖNORM M 7140 Wirtschaftlichkeitsberechnung" mit dem allgemeinen Logo-Bild statt einer seitenspezifischen Überschrift, Beschreibung und ggf. einem passenden Vorschaubild.
- Business Impact: Direkter Reibungsverlust bei jeder Marketing-Aktion, die auf Social Sharing setzt (z. B. Newsletter-Kampagne, LinkedIn-Post zur ÖNORM-Seite, Preis-Aktion): Klickanreiz und Glaubwürdigkeit der Vorschau leiden, weil Inhalt und Vorschau nicht zusammenpassen.
- Konkrete Handlungsempfehlung: Entweder ein Pre-Rendering-/SSG-Schritt für den Build einführen (z. B. `vite-plugin-ssr`, `vite-react-ssg` oder ein einfacher Post-Build-Schritt, der pro Route ein eigenes `index.html` mit den jeweiligen Meta-Tags erzeugt), oder kurzfristig zumindest pro Route individuelle `og:title`, `og:description` und `og:image` über Helmet ergänzen und serverseitig sicherstellen, dass der initiale HTML-Response bereits die korrekten Werte enthält (z. B. via Edge-Function/Middleware beim Hosting-Provider).

### Finding 2 - `/features` hat kein `<h1>`

- Status 2026-06-28: **behoben** durch sichtbares H1 in `FeaturesPage.tsx`; zusätzlich mit `FeaturesPage.test.tsx` abgesichert.
- Kategorie: Bug
- Priorität: hoch
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `src/pages/FeaturesPage.tsx`, `src/components/Features.tsx`
- Evidenz: `FeaturesPage.tsx` rendert nur einen „Zurück"-Button, die `<Features />`-Komponente und eine Roadmap-Sektion. `Features.tsx` enthält als oberste Überschrift ein `<h2>` (Zeile 62), kein `<h1>`. Da `Features` auch eingebettet auf der Homepage verwendet wird (`Index.tsx`, wo `Hero` bereits ein `<h1>` liefert), wurde beim Anlegen der eigenständigen `/features`-Route offenbar kein eigenes `<h1>` ergänzt.
- Beschreibung des Problems: Die Route hat keine eindeutige, maschinenlesbare Hauptüberschrift.
- Warum das relevant ist: Das `<h1>` ist eines der stärksten On-Page-Signale für Suchmaschinen und für Screenreader-Nutzer der erste Orientierungspunkt einer Seite. Ohne `<h1>` wird die thematische Relevanz der Seite für Rankings zu „Normdex Funktionen" geschwächt.
- Business Impact: Reduzierte Sichtbarkeit der Funktionsseite in der organischen Suche, die laut Sitemap mit Priorität 0.8 als wichtige Landingpage markiert ist.
- Konkrete Handlungsempfehlung: In `FeaturesPage.tsx` oberhalb von `<Features />` ein eigenes `<h1>` (z. B. „Alle Normdex-Funktionen im Überblick") ergänzen und das bestehende `<h2>` in `Features.tsx` für den eingebetteten Homepage-Kontext belassen.

### Finding 3 - Title-Tags und ÖNORM-Meta-Description überschreiten SERP-sichtbare Länge

- Status 2026-06-28: **behoben** durch zentrale, gekürzte SEO-Texte in `seo-routes.json`; priorisierte Routen werden per Test auf Titel- und Description-Länge geprüft.
- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `index.html`, `src/pages/Index.tsx`, `src/pages/OenormM7140.tsx`, `src/pages/FeaturesPage.tsx`, `src/pages/About.tsx`
- Evidenz (gemessene Zeichenlängen):
  - Homepage-Title (`index.html` + `Index.tsx`): 68 Zeichen
  - `/oenorm-m-7140`-Title: 73 Zeichen, Description: 197 Zeichen
  - `/features`-Title: 69 Zeichen, Description: 170 Zeichen
  - `/ueber-uns`-Title: 70 Zeichen
- Beschreibung des Problems: Google blendet Title-Tags ab ca. 55-60 Zeichen und Descriptions ab ca. 155-160 Zeichen in den Suchergebnissen abgeschnitten oder ersetzt sie automatisch durch eigene Snippets. Die ÖNORM-Seite, die laut Sitemap-Priorität (0.9) die zweitwichtigste Seite der Domain ist, überschreitet die Description-Länge um rund 40 Zeichen.
- Warum das relevant ist: Abgeschnittene Titel verlieren oft den Marken-Suffix („| Normdex"), abgeschnittene Descriptions enden mitten im Satz und wirken unprofessionell, was die Klickrate in der Suche senkt.
- Business Impact: Niedrigere Click-Through-Rate auf die wichtigsten organischen Einstiegsseiten trotz inhaltlich starker Texte.
- Konkrete Handlungsempfehlung: Titles auf ca. 55-60 Zeichen, Descriptions auf ca. 150-155 Zeichen kürzen, ohne das Haupt-Keyword (Seitenname + „ÖNORM M 7140") zu verlieren, z. B. durch Streichen redundanter Marken-Suffixe in der Description.

### Finding 4 - Sie-Form auf zwei Marketingseiten widerspricht der dokumentierten Du-Form

- Status 2026-06-28: **behoben** durch Umstellung der betroffenen Marketing-CTA-Texte in `About.tsx` und `OenormM7140.tsx` auf Du-Form.
- Kategorie: Bug
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `src/pages/About.tsx:81-82`, `src/pages/OenormM7140.tsx:614,656-657`
- Evidenz: Beide CTA-Blöcke verwenden „Testen Sie Normdex kostenlos und erstellen Sie validierte … für Ihre Energiesystem-Projekte" bzw. „Normdex übernimmt das für Sie", während CLAUDE.md für `TargetAudience` ausdrücklich „durchgängig Du-Form" vorschreibt und Cookie-Texte laut Vorbericht 2026-06-14 bereits gezielt von Sie auf Du umgestellt wurden. Rechtstexte (`AGB.tsx`, `Datenschutz.tsx`) verwenden Sie konsequent und korrekt, das ist dort branchenüblich und kein Finding.
- Beschreibung des Problems: Zwei Marketing-CTAs fallen aus dem sonst konsequenten Du-Ton der Seite heraus.
- Warum das relevant ist: Uneinheitliche Anrede wirkt unprofessionell und schwächt die persönliche, direkte Markenstimme, die der Rest der Seite (Hero, Features, TargetAudience, Pricing) bewusst nutzt.
- Business Impact: Geringe, aber messbare Inkonsistenz in der Markenwahrnehmung an genau den Stellen, an denen zur Conversion (Testen/Anmelden) aufgefordert wird.
- Konkrete Handlungsempfehlung: Die beiden CTA-Texte auf Du-Form umstellen, z. B. „Teste Normdex kostenlos und erstelle validierte Wirtschaftlichkeitsberechnungen für deine Energiesystem-Projekte."

### Finding 5 - SPA-404 ohne `noindex` und ohne erkennbare Server-Rewrite-Konfiguration

- Status 2026-06-28: **behoben** durch `noindex` in `NotFound.tsx` und `try_files $uri $uri/ =404` im Docker-Nginx.
- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: wahrscheinlich / manuell prüfen
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `src/pages/NotFound.tsx`, Repository-Root (kein `vercel.json`, `netlify.toml` oder `_redirects` vorhanden)
- Evidenz: `NotFound.tsx` setzt per Helmet nur einen `<title>`, aber kein `<meta name="robots" content="noindex">`. Im Repository existiert keine Hosting-Konfigurationsdatei, die für unbekannte Pfade explizit einen HTTP-404-Statuscode definiert; bei vielen statischen SPA-Hosting-Setups liefert die Catch-all-Rewrite-Regel für `index.html` sonst einen HTTP-200-Status auch für nicht existierende Pfade.
- Beschreibung des Problems: Ohne servernahe Konfiguration im Repo lässt sich nicht verifizieren, ob falsche URLs tatsächlich mit HTTP 404 beantwortet werden. Zusätzlich fehlt die clientseitige Absicherung über `noindex`.
- Warum das relevant ist: Wenn falsche/alte URLs mit HTTP 200 beantwortet werden, können sie von Google indexiert werden und als Soft-404 in der Search Console auftauchen, was die Crawl-Effizienz der gesamten Domain verschlechtert.
- Business Impact: Potenzielle Verwässerung der Crawl-Budget-Nutzung und irreführende Suchergebnisse bei verwaisten/alten URLs.
- Konkrete Handlungsempfehlung: `<meta name="robots" content="noindex">` in `NotFound.tsx` ergänzen und beim Hosting-Provider (z. B. Vercel/Netlify) prüfen, ob die SPA-Rewrite-Regel für unbekannte Pfade tatsächlich einen 404-Statuscode zurückgibt statt 200.

### Finding 6 - `sitemap.xml` ohne `<lastmod>`

- Status 2026-06-28: **behoben** durch `<lastmod>2026-06-28</lastmod>` für alle 12 URLs in `public/sitemap.xml`.
- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `public/sitemap.xml`
- Evidenz: Keine der 12 `<url>`-Einträge enthält ein `<lastmod>`-Element, nur `<changefreq>` und `<priority>`.
- Beschreibung des Problems: Suchmaschinen nutzen `<lastmod>`, um Recrawl-Priorität bei Content-Updates zu steuern.
- Warum das relevant ist: Ohne `<lastmod>` kann Google nicht erkennen, dass z. B. die Homepage oder die ÖNORM-Seite seit dem letzten Crawl aktualisiert wurde, was insbesondere nach Marketing-Updates (wie dem aktuellen v0.0.4-Release) zu verzögerter Neuindexierung führen kann.
- Business Impact: Geringfügig verzögerte Sichtbarkeit neuer Inhalte in der organischen Suche.
- Konkrete Handlungsempfehlung: `<lastmod>` pro URL ergänzen und bei Content-Änderungen im Rahmen des Release-Prozesses aktualisieren (kann z. B. aus dem Git-Commit-Datum der jeweiligen Page-Datei generiert werden).

### Finding 7 - TypeScript-Strictness bleibt deaktiviert (persistent)

- Status 2026-06-28: **behoben**. `tsconfig.app.json` setzt `"strict": true` und `"noImplicitAny": true`; `npx tsc -p tsconfig.app.json --noEmit` läuft erfolgreich.
- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-04-08)
- Betroffene Datei(en) oder Pfade: `tsconfig.app.json`
- Evidenz: `"strict": false`, `"noImplicitAny": false`, unverändert seit dem Vorbericht.
- Beschreibung des Problems: Siehe Vorbericht 2026-06-14, Finding 1.
- Warum das relevant ist: Unverändert.
- Business Impact: Unverändert erhöhtes Regressionsrisiko bei Refactorings.
- Konkrete Handlungsempfehlung: Unverändert, schrittweise Aktivierung empfohlen.

### Finding 8 - Keine automatisierten Tests (persistent)

- Status 2026-06-28: **behoben**. `package.json` enthält `test: vitest run`; Vitest-Konfiguration und erste Tests für SEO-Metadaten, Report-Preview-Daten und `/features`-H1 sind vorhanden.
- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-06-14)
- Betroffene Datei(en) oder Pfade: `package.json`, `src/`
- Evidenz: Kein `test`-Script in `package.json`, keine Test-Dateien gefunden.
- Beschreibung des Problems: Siehe Vorbericht 2026-06-14, Finding 2.
- Warum das relevant ist: Unverändert.
- Business Impact: Unverändert.
- Konkrete Handlungsempfehlung: Unverändert.

## 8. Quick Wins

- ~~`<meta name="robots" content="noindex">` in `NotFound.tsx` ergänzen.~~ **Erledigt 2026-06-28.**
- ~~Eigenes `<h1>` auf `/features` ergänzen.~~ **Erledigt 2026-06-28.**
- ~~Title- und Description-Längen der vier betroffenen Seiten kürzen, Schwerpunkt ÖNORM-Description.~~ **Erledigt 2026-06-28.**
- ~~Die zwei Sie-Form-Stellen in `About.tsx` und `OenormM7140.tsx` auf Du-Form umstellen.~~ **Erledigt 2026-06-28.**
- ~~`<lastmod>` in `sitemap.xml` ergänzen.~~ **Erledigt 2026-06-28.**

## 9. Strategische Empfehlungen

- ~~Pre-Rendering/SSG für die Landingpage einführen, damit Social-Sharing-Vorschauen und Crawler ohne JS-Ausführung pro Route korrekte Meta-Daten erhalten.~~ **Erledigt 2026-06-28 als pragmatischer Post-Build-HTML-Generator pro Route.**
- Pro Marketingseite ein eigenes, thematisch passendes JSON-LD-Schema prüfen (z. B. `FAQPage` für die ÖNORM-Seite, falls dort künftig FAQ-Inhalte ergänzt werden), statt nur des globalen `SoftwareApplication`-Schemas auf Homepage-Ebene.
- ~~Conversion-Flows mit einer kleinen automatisierten Regressionstest-Suite absichern (unverändert aus Vorbericht).~~ **Grundlage erledigt 2026-06-28 mit Vitest-Testsetup und ersten Regressionstests.**

## 10. Empfohlene nächste Aktion

Alle Findings dieses Audits sind repo-seitig behoben und lokal verifiziert. Nächste sinnvolle Aktion nach Deployment: Social-Share-Previews und 404-Status auf `https://normdex.at` live stichprobenhaft prüfen.

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Tatsächliches Verhalten von Social-Media-Crawlern (Facebook Sharing Debugger, Twitter Card Validator, LinkedIn Post Inspector) für Unterseiten der Live-Domain `https://normdex.at` — sollte nach jedem Deployment stichprobenhaft geprüft werden.
- Tatsächlicher HTTP-Statuscode für unbekannte Pfade auf der produktiven Domain (404 vs. 200), abhängig von der Hosting-Provider-Konfiguration außerhalb dieses Repositories.
- Google Search Console: aktuelle Indexierungsabdeckung und eventuelle Soft-404- oder Duplicate-Title-Warnungen für die zwölf Sitemap-URLs.
- Produktionsbuild und Bundle-Größenanalyse konnten in dieser Sitzung wegen eines Berechtigungsproblems im `dist/`-Ordner nicht ausgeführt werden und sollten in einer sauberen Umgebung nachgeholt werden.
