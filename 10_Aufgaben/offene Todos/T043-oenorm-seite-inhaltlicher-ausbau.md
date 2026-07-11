# T043 · ÖNORM M 7140-Seite – Konzept zum inhaltlichen Ausbau

**Phase:** Landingpage / Marketing / SEO / Content
**Priorität:** P2 · Reichweite – mittelfristig
**Status:** offen
**Datum:** 2026-07-08

## Ziel

Die bestehende Seite `/oenorm-m-7140` ist technisch gut aufgebaut, aber inhaltlich noch zu oberflächlich für organische Sichtbarkeit bei spezifischen Suchanfragen. Ziel ist ein Konzept, das die Seite zur **besten deutschsprachigen Ressource über die ÖNORM M 7140** macht – und damit Long-Tail-Traffic aus der Zielgruppe anzieht.

## Kontext

Die Seite `src/pages/OenormM7140.tsx` hat bereits:
- Erklärung der drei Berechnungsverfahren (Kapitalwert, Annuität, Amortisation)
- Normstruktur (Abschnitte der ÖNORM)
- Zielgruppenübersicht
- Normdex-Feature-Übersicht
- CTA zu App und Praxisleitfaden

Was fehlt (und was Nutzer wirklich suchen):
- **Konkrete Beispielrechnung** (am häufigsten gesuchtes Thema rund um die Norm)
- **FAQ-Sektion**: typische Fragen wie „Was kostet die ÖNORM M 7140?", „Wer braucht sie?", „Unterschied zu VDI 2067?"
- **Glossar** der wichtigsten Begriffe (Annuität, Kalkulationszinssatz, Preissteigerungsrate)
- **Vergleich der drei Verfahren** – Vor-/Nachteile, wann welches Verfahren
- **Rechtlicher/normativer Kontext**: Bezug zu Förderungen, Ausschreibungspflichten in Österreich

Außerdem: Die Seite hat aktuell kein FAQ-Schema (Schema.org `FAQPage`), was Google-Rich-Results blockiert.

## Umsetzung

### 1. Konzept und Struktur definieren (dieses Todo)
Folgende Inhaltsbereiche für den Ausbau konzipieren:

**A. Beispielrechnung (Herzstück)**
- Durchgerechnetes Beispiel: Wärmepumpe vs. Gas-Kessel, 20 Jahre Betrachtungszeitraum
- Alle drei Verfahren zeigen (Kapitalwert, Annuität, Amortisation)
- Als kompaktes, verständliches Beispiel – nicht als vollständige Kalkulation

**B. FAQ-Sektion mit Schema.org `FAQPage`**
Vorgeschlagene Fragen:
1. Für welche Projekte ist die ÖNORM M 7140 verpflichtend?
2. Was ist der Unterschied zwischen Kapitalwertmethode und Annuitätenmethode?
3. Welcher Kalkulationszinssatz ist anzusetzen?
4. Wie unterscheidet sich die ÖNORM M 7140 von der VDI 2067?
5. Kann ich die Berechnung in Excel machen?
6. Wie lange dauert eine normkonforme Wirtschaftlichkeitsberechnung?

**C. Verfahrensvergleich (Tabelle)**
- Kapitalwert / Annuität / Amortisation nebeneinander
- Wann empfohlen, Vor- und Nachteile, Typischer Anwendungsfall

**D. Glossar (Accordion)**
- Kalkulationszinssatz, Annuität, Amortisationszeit, Barwert, Preissteigerungsrate, Lebenszykluskosten, Variante/Energiesystem

### 2. Umsetzung im Code
- Bestehende `OenormM7140.tsx` um die neuen Sektionen erweitern
- `FAQPage` JSON-LD via `<Seo>`-Komponente oder direkt per `<script type="application/ld+json">` einbinden
- SEO-Metadaten in `seo-routes.json` aktualisieren (Description keyword-reicher formulieren)
- Interne Verlinkung: von `/features` und `/` auf `/oenorm-m-7140` verlinken

### 3. Abhängigkeit zu T042
Vor Umsetzung die Ergebnisse aus [[T042-seo-analyse-landingpage]] abwarten – die Keyword-Recherche dort liefert, welche konkreten Formulierungen in der Seite auftauchen sollen.

## Akzeptanzkriterien

- [ ] Konzept mit allen Inhaltsbereichen (A–D) schriftlich ausgearbeitet und abgenommen
- [ ] Beispielrechnung (Bereich A) ist fachlich korrekt nach ÖNORM M 7140
- [ ] FAQ-Sektion mit mind. 6 Fragen und `FAQPage`-Schema implementiert
- [ ] Verfahrensvergleich als Tabelle vorhanden
- [ ] Glossar (mind. 5 Begriffe) als Accordion implementiert
- [ ] Google Rich Results Test zeigt FAQ-Rich-Result für die Seite
- [ ] SEO-Meta-Description aktualisiert (Keyword + Length OK)

## Verifikation

- Google Rich Results Test: `https://search.google.com/test/rich-results` für `normdex.at/oenorm-m-7140`
- Manuelle Prüfung im Dev-Server: alle neuen Sektionen sehen auf Mobile + Desktop korrekt aus
- Build (`npm run build`) erfolgreich ohne Fehler

## Notizen / Fortschritt

- 2026-07-08: Todo angelegt. Hintergrund: Ziel ist mehr organischer Traffic über Long-Tail-Keywords rund um die ÖNORM M 7140. Die Seite ist aktuell gut strukturiert, aber zu dünn für Ranking. Baut auf [[T042-seo-analyse-landingpage]] auf. Inhaltlich verzahnt mit [[T037-fachbeitrag-content-struktur-landingpage]] (Fachbeiträge auf `/wissen` sind ein separater Kanal, diese Seite bleibt aber dauerhaft die Haupt-Ressource zur Norm).

- **2026-07-11: Inhaltliche Verbesserung der bestehenden Sektionen durchgeführt** (ohne Praktisches Beispiel oder Glossar):
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

Folgende Bereichezur noch **offen** (aus T043 ursprünglich):
- **A. Beispielrechnung:** Explizit abgelehnt (User: "Ich will nicht, dass du das mit einem praktischen Beispiel machst")
- **C. Verfahrensvergleich (Tabelle):** Noch nicht implementiert
- **D. Glossar (Accordion):** Noch nicht implementiert
