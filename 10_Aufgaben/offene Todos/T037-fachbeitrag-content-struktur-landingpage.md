# T037 · Fachbeitrag-/Content-Struktur auf der Landingpage

**Phase:** Landingpage / Marketing / Content / SEO
**Priorität:** P3 · Vorbereitung für Marketingplan-Phase 3 (Kumulieren)
**Status:** offen
**Datum:** 2026-07-01

## Ziel

Für den monatlichen Fachbeitrag aus dem [[Marketingplan 2026 - Erste Kunden]] (Phase 3, "Kumulieren") braucht es eine Content-Struktur auf der Landingpage, auf der diese Beiträge veröffentlicht werden können. Aktuell gibt es dafür keine Seite und keinen Mechanismus.

## Kontext

Der Fachbeitrag ist im Marketingplan als eigener Kanal "SEO und Content" definiert, getrennt vom Newsletter-Kanal: Ziel ist Long-Tail-SEO-Sichtbarkeit auf Suchanfragen, die der Käufer wirklich tippt (z. B. die drei Verfahren erklärt, eine durchgerechnete Beispielrechnung, typische Fragen zur Norm). Jeder Beitrag soll am Ende einen Aufruf zum Newsletter enthalten, ist aber primär als Content **auf der Landingpage** gedacht, nicht als reiner Newsletter-Versand.

Reihenfolge laut Plan: Dieses Todo ist Phase 3, kommt also nach Phase 2 (LinkedIn-Unternehmensseite, Google Ads) dran und ist nicht dringend.

Technischer Befund im Repo `normdex-landingpage` (Stand 2026-07-01): reines React-SPA (Vite + `react-router-dom`), jede Seite eine eigene Komponente unter `src/pages/`, Routen manuell in `src/App.tsx` registriert. SEO-Metadaten kommen aus `src/lib/seo-routes.json`, das ein Build-Script (`scripts/generate-route-html.mjs`) beim Build in statisches HTML pro Route einbrennt. Es existiert kein CMS, kein Markdown-/MDX-Rendering, kein Blog-System. Die bestehende Seite `src/pages/OenormM7140.tsx` folgt bereits einem fachartikel-artigen Muster (Überschriften, Card-Grid mit Icons, Prosa, `Seo`-Komponente) und eignet sich als Vorlage.

Bewusste Entscheidung gegen ein CMS oder Markdown-/MDX-Pipeline: Bei rund 12 Beiträgen im Jahr und Solo-Betrieb lohnt sich der Zusatzaufwand einer generischen Content-Pipeline nicht. Einfacher, wiederholbarer manueller Prozess ist hier angemessener als Vorab-Investition in Infrastruktur.

## Umsetzung

- Neue Übersichtsseite `/wissen` anlegen: Liste aller veröffentlichten Fachbeiträge (Titel, Kurzbeschreibung, Link), Komponente unter `src/pages/Wissen.tsx` (oder passender Name), Route in `App.tsx` registrieren, Eintrag in `seo-routes.json` ergänzen.
- Wiederverwendbares Muster für einzelne Beiträge festlegen, angelehnt an `OenormM7140.tsx`: eigene Seite pro Beitrag unter Route `/wissen/<slug>`, mit `Seo`-Komponente, Prosa-/Card-Aufbau passend zum Thema, Newsletter-CTA am Ende (z. B. Wiederverwendung von `NewsletterStrip`/`CTA`-Komponenten).
- Pro neuem Beitrag (wiederkehrender monatlicher Ablauf, kein Einmalaufwand):
  1. Neue `.tsx`-Datei unter `src/pages/wissen/<slug>.tsx` (oder passende Struktur) anlegen.
  2. Route in `App.tsx` ergänzen (lazy import, analog zu bestehenden Seiten).
  3. Eintrag in `seo-routes.json` ergänzen (Title, Description, Canonical, Typ).
  4. Beitrag in der `/wissen`-Übersichtsliste verlinken.
  5. Bei Bedarf Verweis aus LinkedIn-Post und/oder Newsletter auf den Beitrag setzen.
- Kein CMS, kein Markdown-Parser, keine Datenbank-Anbindung einführen — bewusst beim bestehenden manuellen Seiten-Muster bleiben.

## Akzeptanzkriterien

- [ ] `/wissen` zeigt eine Übersicht aller veröffentlichten Fachbeiträge mit Titel, Kurzbeschreibung und Link.
- [ ] Für mindestens einen ersten Fachbeitrag existiert eine vollständige Seite unter `/wissen/<slug>` mit korrektem SEO-Eintrag (Title, Description, Canonical) in `seo-routes.json`.
- [ ] Jeder Beitrag endet mit einem Newsletter-CTA.
- [ ] Der Ablauf zum Anlegen eines neuen Beitrags ist so einfach und wiederholbar, dass er ohne Rückfragen jeden Monat solo durchführbar ist.
- [ ] Bestehende Routen und SEO-Generierung (`scripts/generate-route-html.mjs`) bleiben unverändert funktionsfähig.

## Verifikation

- Frontend-Build (`npm run build`) erfolgreich, inklusive SEO-Route-Generierung.
- Manuelle Prüfung: `/wissen`-Übersicht und mindestens ein Beitrag im Dev-Server aufrufen, Newsletter-CTA und Links prüfen.
- Stichprobe der generierten `dist/`-HTML-Datei für den neuen Beitrag auf korrekte Meta-Tags.

## Notizen / Fortschritt

- 2026-07-01: Todo aus einer Marketingplan-Besprechung angelegt. Reihenfolge bewusst nach Phase 2 (LinkedIn, Google Ads) eingeordnet, keine Umsetzung vor Abschluss dieser Schritte nötig.
