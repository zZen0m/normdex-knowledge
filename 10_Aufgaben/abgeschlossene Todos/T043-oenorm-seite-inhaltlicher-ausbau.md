# T043 · ÖNORM M 7140-Seite – Inhaltlicher Ausbau (Vergleichstabelle, Glossar, SEO-Meta)

**Phase:** Landingpage / Marketing / SEO / Content
**Priorität:** P2 · Reichweite – mittelfristig
**Status:** erledigt
**Datum:** 2026-07-08
**Abgeschlossen:** 2026-07-26

## Ziel

Die Seite `/oenorm-m-7140` (Repo `normdex-landingpage`, Datei `src/pages/OenormM7140.tsx`) technisch und inhaltlich weiter zur besten deutschsprachigen Ressource über die ÖNORM M 7140 ausbauen, um Long-Tail-Traffic aus der Zielgruppe anzuziehen.

## Kontext

Die Seite hat bereits:
- Erklärung der drei Berechnungsverfahren (Kapitalwert, Annuität, Amortisation)
- Normstruktur (Abschnitte der ÖNORM)
- Zielgruppenübersicht
- Normdex-Feature-Übersicht
- **FAQ-Sektion mit `FAQPage`-Schema.org-JSON-LD** (6 Fragen, implementiert und verifiziert im Code, Zeilen 245–294)
- CTA zu App und Praxisleitfaden

Von der ursprünglichen T043-Konzeption (A–D) ist damit erledigt:
- **B. FAQ-Sektion:** ✅ fertig implementiert, inkl. Schema.org `FAQPage`
- **A. Beispielrechnung:** ❌ explizit abgelehnt (User: „Ich will nicht, dass du das mit einem praktischen Beispiel machst") — bleibt draußen, kein Bestandteil mehr von T043

Verbleibender und (2026-07-26) erweiterter Scope, im Detail konzipiert und final abgenommen:

### C. Verfahrensvergleich-Tabelle

Tabelle mit den drei Berechnungsverfahren (Kapitalwert, Annuität, Amortisation) nebeneinander.

**Spalten:** Verfahren | Wann empfohlen | Vorteile | Nachteile | Anwendungsfall

**Wichtig zur Faktenbasis:** Die ÖNORM M 7140 selbst beschreibt die drei Verfahren neutral, ohne Empfehlung, wann welches Verfahren zu bevorzugen ist. Die Bewertungsspalten (Wann empfohlen / Vorteile / Nachteile) sind daher als **fachlich plausible Eigeneinschätzung** zu formulieren (branchenüblich, aber nicht direkt normzitiert) — nicht als wörtliches Normzitat darstellen.

**Referenz für die neutralen Verfahrensbeschreibungen:** `sharepoint-normdex/03_Product-Development/03_Normen/OENORM_M_7140_2021_01_15_de1.pdf`

**Platzierung:** direkt nach der bestehenden Sektion „Verfahren und Inhalte der Norm" (nach dem `normSections`-Card-Grid, `OenormM7140.tsx:430–473`).

### D. Glossar (Accordion)

**10 Begriffe**, exakt aus dem App-internen PDF-Report-Glossar übernommen (`apps/api/app/services/pdf_generator.py:3309–3392`, dort `GLOSSARY`-Liste mit insgesamt 26 Einträgen — für die Landingpage auf die Kernbegriffe reduziert, die bereits in Content/FAQ der Seite vorkommen):

1. Betrachtungszeitraum
2. Nutzungsdauer
3. Kalkulatorischer Zinssatz
4. Preisentwicklungsrate
5. Barwert
6. Annuität
7. Amortisationsdauer
8. Kapitalgebundene Kosten
9. Verbrauchsgebundene Kosten
10. Betriebsgebundene Kosten

**Wichtig:** Definitionstexte **1:1** aus `pdf_generator.py` übernehmen (exakter normkonformer Wortlaut) — keine eigene Umformulierung, um Inkonsistenz zwischen App-Report und Landingpage zu vermeiden.

**Platzierung:** als eigener Accordion-Block direkt vor der FAQ-Sektion (vor `OenormM7140.tsx:663`).

### E. SEO-Meta-Updates (aus T042 übernommen, in T043 miterledigt)

- **H1 erweitern:** aktuell nur „ÖNORM M 7140" (`OenormM7140.tsx:313–315`) → um „Wirtschaftlichkeitsberechnung" ergänzen (z. B. „ÖNORM M 7140 – Wirtschaftlichkeitsberechnung für Energiesysteme"), passend zum priorisierten Keyword „Wirtschaftlichkeitsberechnung Energiesystem" aus [[SEO-Analyse-2026-07]].
- **Meta-Description aktualisieren:** in `seo-routes.json` (Route `/oenorm-m-7140`) keyword-reicher formulieren, orientiert an den High-Priority-Keywords „ÖNORM M 7140 Berechnung", „ÖNORM M 7140 Software", „ÖNORM M 7140 online". Länge 130–160 Zeichen einhalten.

## Umsetzung

1. `OenormM7140.tsx` (Repo `normdex-landingpage`) um die Sektionen C und D erweitern (Code, Struktur wie oben festgelegt).
2. H1 (Zeile ~313–315) anpassen.
3. `seo-routes.json` Description für `/oenorm-m-7140` aktualisieren.
4. Interne Verlinkung von `/features` und `/` auf `/oenorm-m-7140` prüfen (bereits laut T042 vorhanden, hier nur gegenprüfen).

**Hinweis:** Umsetzung erfolgt im Repo `normdex-landingpage`, nicht in `normdex-app` — separate Session/Repo-Kontext nötig.

## Akzeptanzkriterien

- [x] Konzept mit allen verbleibenden Inhaltsbereichen (C, D, E) schriftlich ausgearbeitet und abgenommen (2026-07-26, per Grilling-Session)
- [x] FAQ-Sektion mit mind. 6 Fragen und `FAQPage`-Schema implementiert (bereits erledigt, verifiziert im Code)
- [x] Verfahrensvergleich als Tabelle vorhanden (Spalten: Verfahren | Wann empfohlen | Vorteile | Nachteile | Anwendungsfall)
- [x] Glossar (10 Begriffe, 1:1 aus `pdf_generator.py`) als Accordion implementiert, platziert vor der FAQ-Sektion
- [x] H1 um „Wirtschaftlichkeitsberechnung" erweitert
- [x] SEO-Meta-Description aktualisiert (Keyword-reicher, Länge 130–160 Zeichen)
- [x] Google Rich Results Test durchgeführt (Regression-Check nach den Änderungen): keine Fehler, Crawling/Indexierung erfolgreich, „Unternehmen"-Schema als gültig erkannt. `FAQPage` erscheint erwartungsgemäß nicht als Rich-Result-Kandidat mehr — Google hat FAQ-Rich-Results seit August 2023 auf autoritative Regierungs-/Gesundheitsseiten beschränkt, betrifft alle Seiten unabhängig von T043 und ist keine durch T043 verursachte Regression.

## Verifikation

- Google Rich Results Test: `https://search.google.com/test/rich-results` für `normdex.at/oenorm-m-7140`
- Manuelle Prüfung im Dev-Server (`normdex-landingpage`): neue Sektionen (Tabelle, Glossar) auf Mobile + Desktop korrekt
- Build (`npm run build`) im Repo `normdex-landingpage` erfolgreich ohne Fehler
- Fachliche Prüfung: Tabelleninhalte und Glossar-Definitionen gegen ÖNORM-PDF (`OENORM_M_7140_2021_01_15_de1.pdf`) bzw. `pdf_generator.py` gegenlesen

## Notizen / Fortschritt

- 2026-07-08: Todo angelegt. Hintergrund: Ziel ist mehr organischer Traffic über Long-Tail-Keywords rund um die ÖNORM M 7140. Die Seite ist aktuell gut strukturiert, aber zu dünn für Ranking. Baut auf [[T042-seo-analyse-landingpage]] auf. Inhaltlich verzahnt mit [[T037-fachbeitrag-content-struktur-landingpage]] (Fachbeiträge auf `/wissen` sind ein separater Kanal, diese Seite bleibt aber dauerhaft die Haupt-Ressource zur Norm).

- 2026-07-11: Inhaltliche Verbesserung der bestehenden Sektionen durchgeführt (ohne Praktisches Beispiel oder Glossar):
  - **Hero-Teaser:** Prägnanter formuliert – Problem/Lösung-Struktur, warum die Norm wichtig ist (vs. Bauchgefühl/Improvisation)
  - **Intro-Absatz ("Was ist ÖNORM M 7140?"):** Deutlich ausführlicher (3 Absätze), klärt: Definition, Problem das gelöst wird, Kostenkategorien, dynamisches Rechnen, Revision 2021, Abschnitt 10/Validierung
  - **"Dynamisch statt statisch"-Box:** Bessere Erklärungen der 3 Punkte — konkrete Beispiele, warum relevant (Zeitwert des Geldes, energieträgerspezifische Preissteigerungsraten, Impact bei 20+ Jahren)
  - **Verfahren & Inhalte (normSections):**
    - Kapitalwertmethode → "Kapitalwertmethode (Barwertmethode)" mit Erklärung Kalkulationszinssatz
    - Annuitätenmethode → "Annuitätenmethode (Jahresrente)" mit Jahresraten-Analogie
    - Amortisationsrechnung → "Amortisationsrechnung (Payback)" mit deutlicher Erklärung
    - Kostenstruktur → "Kostenstruktur nach Verursachung" – transparentere Kategorisierung
  - **FAQ:** Erste Frage erweitert und prägnanter
  - Alle Umlaute (ä, ö, ü) durchgehend, Brand Voice "Präzise, Professionell, Vertrauenswürdig" eingehalten

- 2026-07-19/26 (zwischenzeitlich, aus Code-Stand ersichtlich): FAQ-Sektion vollständig auf 6 Fragen ausgebaut inkl. `FAQPage`-JSON-LD (`faqJsonLd`, `OenormM7140.tsx:245–294`) — Bereich B damit vollständig erledigt.

- **2026-07-26: Todo per Grilling-Session vollständig neu geschärft.** Ausgangslage geprüft: Code der Seite gelesen (liegt in Repo `normdex-landingpage`, nicht `normdex-app`), T042 (jetzt abgeschlossen, siehe [[SEO-Analyse-2026-07]]) als Keyword-Grundlage herangezogen, App-internes PDF-Glossar (`apps/api/app/services/pdf_generator.py`) als Quelle für Bereich D identifiziert. Entscheidungen:
  - Scope erweitert um Bereich E (SEO-Meta: H1 + Description aus T042-Vorschlägen #3 und #7), da synergetisch zur gleichen Seite.
  - Bereich A endgültig aus dem Scope entfernt, Bereich B als erledigt markiert.
  - Vergleichstabelle (C): 5 Spalten, Bewertungen als fachlich plausible Eigeneinschätzung (Norm selbst spricht keine Empfehlung aus), Platzierung nach `normSections`.
  - Glossar (D): 10 Kernbegriffe aus dem 26-Begriffe-PDF-Glossar ausgewählt, Definitionstexte 1:1 übernommen (keine Umformulierung), Platzierung vor FAQ.
  - Nächster Schritt: Code-Umsetzung in separater Session mit `normdex-landingpage`-Repo-Kontext.

- **2026-07-26: Code-Umsetzung abgeschlossen** in `normdex-landingpage/src/pages/OenormM7140.tsx`:
  - **C. Verfahrensvergleich-Tabelle** eingefügt direkt nach der `normSections`-Sektion, 5 Spalten, Inhalte als fachlich plausible Eigeneinschätzung formuliert (kein Normzitat).
  - **D. Glossar-Accordion** eingefügt direkt vor der FAQ-Sektion, 10 Begriffe mit Definitionstexten 1:1 aus `apps/api/app/services/pdf_generator.py` übernommen.
  - **E. SEO-Meta:** H1 erweitert zu „ÖNORM M 7140 – Wirtschaftlichkeitsberechnung für Energiesysteme"; Meta-Description in `src/lib/seo-routes.json` (Route `/oenorm-m-7140`) aktualisiert (142 Zeichen, Keywords „Berechnung", „online", „Software" berücksichtigt).
  - `npm run build` (vite-react-ssg) erfolgreich, `dist/oenorm-m-7140/index.html` geprüft: neue Sektionen und aktualisierte Description im gerenderten HTML vorhanden, `FAQPage`-JSON-LD weiterhin intakt.
  - Commit `cb2afe7` in `normdex-landingpage`, gepusht nach `main`, auf `normdex-vps` per SSH gepullt (`/opt/repos/normdex-landingpage`), Stack neu gebaut und deployt (`docker compose build && up -d` in `/opt/stacks/normdex-landingpage`). Live-Check per `curl` bestätigt: neue Meta-Description und neue Sektionen auf `https://normdex.at/oenorm-m-7140/` aktiv.
- **2026-07-26: Google Rich Results Test durchgeführt** (User hat Screenshot beigesteuert): Seite erfolgreich gecrawlt, Indexierung zulässig, „Unternehmen"-Schema als 1 gültiges Element erkannt. `FAQPage`-Daten werden vom Test nicht mehr als Rich-Result-fähig aufgeführt — konsistent mit Googles Richtlinienänderung von August 2023 (FAQ-Rich-Results seitdem nur noch für autoritative Regierungs-/Gesundheitsseiten). Keine Regression durch T043, da diese Einschränkung generell und unabhängig von den heutigen Änderungen gilt. **T043 damit vollständig abgeschlossen.**
