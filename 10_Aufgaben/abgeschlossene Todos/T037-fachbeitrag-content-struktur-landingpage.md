# T037 · Fachbeitrag-/Content-Struktur auf der Landingpage

**Phase:** Landingpage / Marketing / Content / SEO
**Priorität:** P3 · Vorbereitung für Marketingplan-Phase 3 (Kumulieren)
**Status:** erledigt
**Datum:** 2026-07-01 (erledigt: 2026-07-24)

## Ziel

Für den monatlichen Fachbeitrag aus dem [[Marketingplan 2026 - Erste Kunden]] (Phase 3, "Kumulieren") braucht es eine Content-Struktur auf der Landingpage, auf der diese Beiträge veröffentlicht werden können. Aktuell gibt es dafür keine Seite und keinen Mechanismus.

## Kontext

Der Fachbeitrag ist im Marketingplan als eigener Kanal "SEO und Content" definiert, getrennt vom Newsletter-Kanal: Ziel ist Long-Tail-SEO-Sichtbarkeit auf Suchanfragen, die der Käufer wirklich tippt (z. B. die drei Verfahren erklärt, eine durchgerechnete Beispielrechnung, typische Fragen zur Norm). Jeder Beitrag soll am Ende einen Aufruf zum Newsletter enthalten, ist aber primär als Content **auf der Landingpage** gedacht, nicht als reiner Newsletter-Versand.

Reihenfolge laut Plan: Dieses Todo ist Phase 3, kommt also nach Phase 2 (LinkedIn-Unternehmensseite, Google Ads) dran und ist nicht dringend.

Technischer Befund im Repo `normdex-landingpage` (Stand 2026-07-01): reines React-SPA (Vite + `react-router-dom`), jede Seite eine eigene Komponente unter `src/pages/`, Routen manuell in `src/App.tsx` registriert. SEO-Metadaten kommen aus `src/lib/seo-routes.json`, das ein Build-Script (`scripts/generate-route-html.mjs`) beim Build in statisches HTML pro Route einbrennt. Es existiert kein CMS, kein Markdown-/MDX-Rendering, kein Blog-System. Die bestehende Seite `src/pages/OenormM7140.tsx` folgt bereits einem fachartikel-artigen Muster (Überschriften, Card-Grid mit Icons, Prosa, `Seo`-Komponente) und eignet sich als Vorlage.

Bewusste Entscheidung gegen ein CMS oder Markdown-/MDX-Pipeline: Bei rund 12 Beiträgen im Jahr und Solo-Betrieb lohnt sich der Zusatzaufwand einer generischen Content-Pipeline nicht. Einfacher, wiederholbarer manueller Prozess ist hier angemessener als Vorab-Investition in Infrastruktur.

## Architektur-Entscheidung: normübergreifend, nicht normspezifisch

`/wissen` ist bewusst eine eigene, von einzelnen Normen unabhängige Content-Sektion — kein Unterbereich der ÖNORM-M-7140-Seite. Die Hauptnavigation (`Header.tsx`) enthält aktuell nur einen flachen Link "ÖNORM M 7140" → `/oenorm-m-7140/`, keine Kategorie mit Unterseiten. Würde man Fachbeiträge stattdessen unter der jeweiligen Normseite bündeln (z. B. `/oenorm-m-7140/wissen/...`), müsste bei jeder künftigen weiteren Norm eine eigene Struktur neu gebaut werden. Stattdessen bekommt jeder Beitrag ein Norm-Tag als Metadatum (z. B. `norm: "ÖNORM M 7140"`), das als Badge angezeigt wird — die Sektion selbst wächst mit neuen Normen einfach durch weitere Einträge, ohne Strukturumbau.

## Umsetzung

- Neue Übersichtsseite `/wissen` anlegen: chronologische Liste aller veröffentlichten Fachbeiträge (Titel, Kurzbeschreibung, Norm-Badge, Link), Komponente unter `src/pages/Wissen.tsx`. Liste als **hardcoded TS-Array** direkt in der Komponente (kein separates JSON, kein CMS) — bei ~12 Beiträgen/Jahr und Solo-Betrieb reicht das. Keine Filter-/Tab-UI nach Norm; das Norm-Badge ist rein informativ.
- Route `/wissen` in `App.tsx` registrieren, Eintrag in `seo-routes.json` ergänzen (`type: "WebPage"` bzw. `"CollectionPage"`).
- Neuer Menüpunkt **"Wissen"** in `Header.tsx` (`desktopNav` und `mobileNav`), neben "ÖNORM M 7140".
- Wiederverwendbares Muster für einzelne Beiträge festlegen, angelehnt an `OenormM7140.tsx`: eigene Seite pro Beitrag unter Route `/wissen/<slug>`, mit `Seo`-Komponente, Prosa-/Card-Aufbau passend zum Thema, Newsletter-CTA am Ende via Wiederverwendung der bestehenden `NewsletterStrip`-Komponente (keine neue CTA-Variante).
- In `seo-routes.json` bekommen einzelne Beitragsseiten `type: "Article"` (statt `"WebPage"` wie der Rest der Seite) — passender schema.org-Typ für redaktionellen Content.
- Die Kurzbeschreibung auf der `/wissen`-Übersicht ist identisch mit der `description` aus `seo-routes.json` — keine separate Teaser-Textquelle pro Beitrag pflegen.
- `OenormM7140.tsx` bekommt einen Verweis/Card-Block auf passende `/wissen`-Beiträge (stärkt internen Linkaufbau, führt Norm-Interessierte zum Content).
- Pro neuem Beitrag (wiederkehrender monatlicher Ablauf, kein Einmalaufwand):
  1. Neue `.tsx`-Datei unter `src/pages/wissen/<slug>.tsx` (oder passende Struktur) anlegen.
  2. Route in `App.tsx` ergänzen (lazy import, analog zu bestehenden Seiten).
  3. Eintrag in `seo-routes.json` ergänzen (Title, Description, Canonical, `type: "Article"`).
  4. Beitrag mit Titel, Norm-Tag und Link in das Array in `Wissen.tsx` eintragen.
  5. Bei Bedarf Verweis aus LinkedIn-Post und/oder Newsletter auf den Beitrag setzen.
- Kein CMS, kein Markdown-Parser, keine Datenbank-Anbindung einführen — bewusst beim bestehenden manuellen Seiten-Muster bleiben.

## Erster Fachbeitrag

**Thema:** "Die drei Verfahren erklärt" (Kapitalwert-, Annuitäten- und Amortisationsverfahren nach ÖNORM M 7140).
**Slug:** `/wissen/die-drei-verfahren-erklaert`
Begründung: grundlegendstes, evergreen-fähigstes Thema mit der besten Long-Tail-SEO-Basis — spätere Beiträge (z. B. Beispielrechnungen) können darauf verweisen.

## Akzeptanzkriterien

- [x] `/wissen` zeigt eine chronologische Übersicht aller veröffentlichten Fachbeiträge mit Titel, Kurzbeschreibung, Norm-Badge und Link.
- [x] Menüpunkt "Wissen" ist in `Header.tsx` (Desktop- und Mobile-Navigation) vorhanden und verlinkt auf `/wissen`.
- [x] Der Beitrag "Die drei Verfahren erklärt" existiert vollständig unter `/wissen/die-drei-verfahren-erklaert`, mit korrektem SEO-Eintrag (Title, Description, Canonical, `type: "Article"`) in `seo-routes.json`.
- [x] Der Beitrag endet mit dem Newsletter-CTA (`NewsletterStrip`).
- [x] `OenormM7140.tsx` verlinkt auf den Beitrag "Die drei Verfahren erklärt" (bzw. passende Wissen-Beiträge).
- [x] Der Ablauf zum Anlegen eines neuen Beitrags ist so einfach und wiederholbar, dass er ohne Rückfragen jeden Monat solo durchführbar ist.
- [x] Bestehende Routen und SEO-Generierung bleiben unverändert funktionsfähig.

## Verifikation

- Frontend-Build (`npm run build`) erfolgreich, inklusive SEO-Route-Generierung.
- Manuelle Prüfung: `/wissen`-Übersicht und der Beitrag "Die drei Verfahren erklärt" im Dev-Server aufrufen, Navigation, Norm-Badge, Newsletter-CTA, Cross-Link von `OenormM7140.tsx` und Links prüfen.
- Stichprobe der generierten `dist/`-HTML-Datei für den neuen Beitrag auf korrekte Meta-Tags (inkl. `type: "Article"`).

## Notizen / Fortschritt

- 2026-07-01: Todo aus einer Marketingplan-Besprechung angelegt. Reihenfolge bewusst nach Phase 2 (LinkedIn, Google Ads) eingeordnet, keine Umsetzung vor Abschluss dieser Schritte nötig.
- 2026-07-24: Mit dem Nutzer im Detail durchgesprochen (Grill-Interview). Grundsatzentscheidung: `/wissen` bleibt normübergreifend statt an ÖNORM M 7140 gekoppelt, damit künftige weitere Normen ohne Strukturumbau dazukommen. Erster Beitrag, Slug, Navigation, Datenhaltung, SEO-Typ und Cross-Linking-Details festgelegt.
- 2026-07-24: Umgesetzt. Hinweis: Das Repo `normdex-landingpage` wurde zwischenzeitlich (Todo T046) auf `vite-react-ssg` migriert — Routen liegen jetzt in `src/routes.tsx` statt `App.tsx`, das manuelle Build-Script `generate-route-html.mjs` existiert nicht mehr (Prerendering läuft über `vite-react-ssg build`). Umsetzung entsprechend angepasst: Route-Registrierung in `routes.tsx`, Artikel-Komponente unter `src/pages/wissen/DieDreiVerfahrenErklaert.tsx`, Übersichtsseite unter `src/pages/Wissen.tsx` wie geplant. `seo-routes.json` um `/wissen` (`CollectionPage`) und `/wissen/die-drei-verfahren-erklaert` (`Article`) ergänzt. `scripts/verify-prerender.mjs` und `prerender.test.tsx` um die neuen Routen erweitert. `npm test`, `npm run build`, `npm run verify:prerender` und `npm run lint` laufen grün.
