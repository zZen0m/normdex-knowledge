# T034 · Excel-Export für Wirtschaftlichkeitsberichte

**Phase:** App / Wirtschaftlichkeitsberechnung / Export  
**Priorität:** P2 · Kundenwunsch / Berichtsnachbearbeitung  
**Status:** offen  
**Datum:** 2026-06-28

## Ziel

In der Wirtschaftlichkeitsberechnung soll zusätzlich zum bestehenden PDF-Bericht ein Excel-Export (`.xlsx`) bereitgestellt werden. Anwender sollen die berechneten Daten, Jahresreihen und Auswertungen anschließend individuell prüfen, weitergeben und bearbeiten können.

## Kontext

Laut Nutzanfragen möchten Anwender Berichte nach dem Export häufig noch individuell anpassen. Der bestehende PDF-Bericht bleibt der offizielle, layoutstabile Bericht. Die Excel-Datei ergänzt ihn als bearbeitbarer Daten- und Ergebnisexport.

Der Export soll klar als von OneText / Normdex erzeugt erkennbar sein. Das Workbook orientiert sich optisch am bestehenden PDF-Branding und enthält Deckblatt, Kopf-/Fußzeilen beziehungsweise Herkunftshinweise.

## Funktionsumfang

- Excel-Download im Export-Tab neben dem bestehenden PDF-Download.
- Serverseitige `.xlsx`-Erzeugung über einen neuen Endpunkt `POST /reports/economics/excel`.
- Wiederverwendung desselben Export-Payloads wie beim PDF: Ergebnisdaten, Formulardaten, Berichtssichtbarkeit, Projekt- und Organisationsdaten.
- Bearbeitbarer Export ohne eigene Excel-Rechenengine.
- Arbeitsblätter für:
  - Deckblatt und Metadaten
  - Eingaben
  - Ergebnisse
  - Jahresreihen
  - Systemvergleich
  - Sensitivitätsanalyse
  - Hinweise
- Branding mit OneText-/Normdex-Herkunft, Projektkontext, Exportdatum, Akzentfarbe und optionalem Organisationslogo aus den bestehenden Report-Settings.

## Umsetzung

- Backend:
  - `openpyxl` als Dependency ergänzen.
  - Neuen Excel-Generator-Service auf Basis des bestehenden PDF-Request-Schemas erstellen.
  - `POST /reports/economics/excel` als parallelen Report-Endpunkt ergänzen.
  - Excel-Dateien on-demand erzeugen und als Download zurückgeben; keine dauerhafte Speicherung.
- Frontend:
  - API-Client um `econExcelExport` ergänzen.
  - Export-Tab um Button „Excel exportieren“ mit eigenem Loading-State und Toasts erweitern.
  - Gemeinsame Payload-Erzeugung für PDF und Excel verwenden.
  - Excel-Dateiname analog zum PDF-Dateinamen erzeugen, aber mit `.xlsx`.
- Sichtbarkeit:
  - Projektbezogene Berichtssichtbarkeit auch für Excel berücksichtigen, soweit sie auf tabellarische Inhalte übertragbar ist.
  - PDF-spezifische Elemente wie Inhaltsverzeichnis werden in Excel als sinnvolle Struktur- oder Metadatenblätter interpretiert.

## Akzeptanzkriterien

- [ ] Im Export-Tab gibt es neben „PDF exportieren“ auch „Excel exportieren“.
- [ ] Excel-Export funktioniert auch ohne vorherigen Wechsel in den Ergebnistab.
- [ ] Aktuelle Projektdaten werden vor dem Export gespeichert.
- [ ] Der Download liefert eine gültige `.xlsx`-Datei mit korrektem Content-Type und `.xlsx`-Dateinamen.
- [ ] Workbook enthält Deckblatt, Eingaben, Ergebnisse, Jahresreihen, Systemvergleich, Sensitivität und Hinweise.
- [ ] Branding und Herkunft von OneText / Normdex sind im Workbook sichtbar.
- [ ] Organisationslogo, Primärfarbe und Headertext werden nach Möglichkeit aus den bestehenden Report-Settings übernommen.
- [ ] Deaktivierte Systeme oder Inhalte werden im Excel-Export sinnvoll berücksichtigt.
- [ ] Der bestehende PDF-Export bleibt unverändert funktionsfähig.

## Verifikation

- Backend-Test für Authentifizierung des Excel-Endpunkts.
- Backend-Test für erfolgreiche `.xlsx`-Antwort, ladbares Workbook und erwartete Arbeitsblätter.
- Backend-Test für Branding-Elemente und Sichtbarkeit deaktivierter Systeme.
- Frontend-Build erfolgreich ausführen.
- Manuelle QA im Export-Tab: PDF- und Excel-Download testen.

## Notizen / Fortschritt

- 2026-06-28: Todo aus Nutzanfrage angelegt. Standardentscheidung: bearbeitbarer Export ohne vollständige Excel-Formel-/Berechnungslogik.
