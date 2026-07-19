# Schreib- und Designvorgaben LinkedIn

Gilt für alle Fachposts auf der Normdex-Unternehmensseite. Basis sind die [[Marketingplan 2026 - Erste Kunden|Schreibregeln aus dem Marketingplan]], hier um LinkedIn-spezifische Punkte ergänzt.

## Grundregeln (aus dem Marketingplan, unverändert)

- Durchgängig Du-Form.
- Direkte Umlaute (ä, ö, ü), keine Umschreibungen.
- Keine Gedankenstriche oder Halbgeviertstriche. Stattdessen Punkt, Komma, Doppelpunkt oder Klammer.
- Sachlich, klar, kompetent. Keine übertriebene Werbesprache.
- Kein Mitbewerber namentlich (Pokorny, BVE, Urban-Energy bleiben intern).
- Jeder Post endet mit einem Aufruf zum Newsletter oder zum Beispiel-Report.

## Oberste Priorität: nicht nach KI klingen

Ein Post darf nicht auf den ersten Blick als KI-generiert erkennbar sein. Das ist wichtiger als Vollständigkeit oder Länge.

**Verboten sind unter anderem:**

- Einstiege wie "In der heutigen schnelllebigen Welt...", "Stell dir vor...", "Hast du dich jemals gefragt..."
- Phrasen wie "Lass uns eintauchen", "Tauchen wir ein in...", "Es ist kein Geheimnis, dass...", "Fakt ist..."
- Übertreibungen wie "Gamechanger", "revolutionär", "bahnbrechend", "auf einen Blick"
- Künstliche Dreier-Aufzählungen ("schnell, einfach, effizient")
- Zusammenfassende Floskeln am Ende wie "Zusammenfassend lässt sich sagen", "Fazit:"
- Generische Kommentar-Aufforderungen ohne echten Inhalt ("Was denkst du? Schreib es in die Kommentare!") als reine Lückenfüller
- Emoji-Aufzählungen mit immer denselben Symbolen (✅ 🚀 💡) als Strukturersatz
- Wortwiederholungen und immer gleiche Satzmuster über mehrere Sätze hinweg

**Stattdessen:**

- Mit einer konkreten Beobachtung, Zahl oder einem echten Fehler aus der Praxis einsteigen, nicht mit einer allgemeinen Aussage über die Branche.
- Kurze, unterschiedlich lange Sätze. Nicht jeder Satz im selben Rhythmus.
- Ein Gedanke pro Absatz, ruhig auch mal ein einzelner kurzer Satz als eigener Absatz.
- Konkret bleiben: lieber ein Beispiel mit Zahlen als eine allgemeine Behauptung.

## Formatierung

- Hook-Zeile: die ersten ein bis zwei Zeilen müssen vor dem "Mehr anzeigen" zum Weiterlesen bewegen. Keine Frage-Floskel, sondern ein konkreter Einstieg.
- Länge: Richtwert 800 bis 1500 Zeichen.
- Kurze Absätze, ein bis zwei Sätze, feed-typisches Layout mit Leerzeilen zwischen den Absätzen.
- Emojis sind optional, kein Muss. Wenn verwendet, dann sparsam und nicht als Aufzählungssymbol-Ersatz.
- Drei bis fünf Hashtags am Ende, mehr nicht.
- **#normdex ist in jedem Post Pflicht.** Weitere Hashtags themenabhängig aus: #ÖNORM, #Wirtschaftlichkeitsberechnung, #Energieeffizienz, #Energieberatung, #Planungsbüro.

## Design-Briefing für Begleitgrafik

- Format: 1200 x 627 px (Feed) oder 1080 x 1080 px (quadratisch).
- Farben und Schrift laut [[Designsystem & Farben]].
- Ein kurzer Kernsatz als Bildtext, kein Fließtext im Bild.
- Kleine Icon-Elemente (Lucide-Linienstil, 2px Strich, pinker Kreisrahmen) sind erlaubt und bei mehrteiligen Themen (z. B. mehrere Verfahren/Begriffe) empfohlen, um das Bild aufzulockern. Kein Icon-Overkill, keine Emoji-Icons, keine Icons ohne inhaltlichen Bezug.
- **Zwei Stilvarianten verfügbar**, beide unter `04_Marketing/LinkedIn/Design-Vorlagen/` als leere Basis (nur Hintergrund, Punktraster, pinker Randbalken, ohne Text):
  - **Dunkel** (`normdex_linkedin_banner_dunkel_1200x627.png`): Gradient `#001a1b` → `#0f4547`, Text/Icons in Off-White `#fafafa`, gedämpfte Labels in `#8fb3b3`. Wirkt kräftiger, guter Kontrast für Sonderanlässe (Launch, große Ankündigungen).
  - **Hell** (`normdex_linkedin_banner_hell_1200x627.png`): Gradient `#fafafa` → `#e3f4f4` (identisch zu `--gradient-hero` aus [[Designsystem & Farben]]), Text/Icons in Dunkel-Teal `#003c3e`, gedämpfte Labels in `#4f8688`. Näher am Look von App und Landingpage, seit 2026-07-17 Standardwahl für die laufende Fachpost-Rotation.
  - Beide Varianten: pinker Randbalken `#ff2d58`, Schrift Inter (Bold für Headline, Medium für Labels), 10 px breiter Akzentbalken links.
  - Das ursprüngliche `normdex_linkedin_banner.png` (04_Marketing, 1128 x 191 px) ist das separate LinkedIn-Unternehmensseiten-Titelbild (Cover Photo), nicht die Vorlage für Post-Grafiken. Gleiche Farbwelt, andere Verwendung.
- Fertige Beispielumsetzung mit Icons (Referenz für künftige Posts): [[2026-07-14-drei-verfahren-kurz-erklaert-banner.png]] im Ordner `Entwürfe/`.

## Ablagestruktur für Post-Dateien

Jeder Fachpost besteht aus zwei Dateien mit demselben Datums-Präfix im Dateinamen: der MD-Datei mit Text und Metadaten sowie der PNG-Begleitgrafik (z. B. `2026-07-14-drei-verfahren-kurz-erklaert.md` und `2026-07-14-drei-verfahren-kurz-erklaert-banner.png`). Der gemeinsame Präfix hält beide Dateien beim Sortieren zusammen.

Während der Entwurfsphase liegen beide Dateien flach in `Entwürfe/`. Nach der Veröffentlichung wandern beide Dateien flach nach `Archiv/`, ohne eigenen Unterordner pro Post. Bei der erwarteten Menge von rund 20 bis 25 Fachposts pro Jahr reicht die Zuordnung über den Datumspräfix aus, ein Unterordner pro Post wäre unnötiger Navigationsaufwand. Unterordner sind erst sinnvoll, falls ein Post mehrere Bilder bekommt (z. B. Karussell mit mehreren Slides) oder das Archiv deutlich größer wird als aktuell absehbar.

## Praktische Hinweise zur Veröffentlichung

Gilt zusätzlich zu Text und Grafik, betrifft den Vorgang des Postens selbst. Basis: Recherche Juli 2026, überwiegend Marketing-Blogs und Tool-Anbieter, keine offizielle LinkedIn-Bestätigung, aber die Richtung deckt sich seit Jahren mit Beobachtungen aus der Praxis.

- **Externer Link im Post kostet Reichweite.** Mehrere Quellen berichten von spürbaren Reichweiteneinbußen, wenn ein Post einen ausgehenden Link enthält (Größenordnung wird unterschiedlich beziffert, tendenziell recht hoch, keine offizielle LinkedIn-Bestätigung). Ein Link im ersten Kommentar hilft laut aktuelleren Berichten nicht mehr zuverlässig.

  **Feste Regel ab jetzt:** Bei den laufenden Fachposts (Themenplan-Rotation) den CTA als reinen Text ohne https:// schreiben, z. B. "normdex.at/newsletter". Wird dadurch wahrscheinlich nicht automatisch verlinkt, kostet also keine Reichweite, ist aber nicht anklickbar (wer will, tippt es selbst oder findet den Link in der Seiteninfo). Bei seltenen großen Anlässen (Launch, große Feature-Ankündigung, echtes Angebot) darf es die volle klickbare Version mit https:// sein, weil dort der direkte Klick wichtiger ist als bei einem alle zwei Wochen wiederkehrenden Fachpost.
- **Seiteninfo vor dem ersten Post prüfen.** Logo, Titelbild, Über-uns-Text, Branche, Standort und der Website-Link in den Page-Einstellungen sollten stehen, bevor der erste Post veröffentlicht wird. Wer nach dem Post auf die Seite klickt, entscheidet danach, ob er weiter vertraut.
- **Als Unternehmensseite posten, nicht als Person.** Passt zur Grundregel "kein persönlicher Auftritt".
- **Formatierung vor dem Absenden prüfen.** Leerzeilen zwischen Absätzen manchmal beim Einfügen kontrollieren, LinkedIn übernimmt sie meist, aber nicht immer zuverlässig aus jeder Quelle.
- **Die erste Stunde nach Veröffentlichung zählt am meisten.** Frühe Kommentare und Reaktionen beeinflussen laut mehreren Quellen, wie weit ein Post danach ausgespielt wird. Bei einer neuen Seite ohne Follower hilft es, den Post vom privaten Profil aus zu teilen oder ein paar bekannte Kontakte (z. B. den Pilotkunden) auf den Post hinzuweisen.
- **Timing:** Werktags vormittags (grob 8 bis 10 Uhr) gilt für ein B2B-Fachpublikum in Österreich als üblicher guter Zeitpunkt, Wochenenden eher schwächer.
- **Bild nicht vergessen.** Ein Post mit Bild bekommt in der Regel mehr Aufmerksamkeit als reiner Text. Das Grafik-Briefing muss vor dem Posten noch tatsächlich als Bild umgesetzt werden (siehe Abschnitt oben).

## Verwandte Dokumente

- [[00 Themenplan]]
- [[Marketingplan 2026 - Erste Kunden]]
- [[Key Messages & CTAs]]
- [[Designsystem & Farben]]
