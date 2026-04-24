# Wirtschaftlichkeitsrechner

Der Economics Calculator ist das **Kernprodukt** von Normdex – ein umfassendes Werkzeug zur wirtschaftlichen Bewertung von Investitionsprojekten über einen mehrjährigen Zeithorizont gemäß ÖNORM M 7140.

## Was wird berechnet?

- **Kapitalwert (Net Present Value)** und **Amortisationszeit**
- **Kapitalrendite** unter Berücksichtigung von Diskontierung
- **Sensitivitätsanalyse** (Wie verändert sich das Ergebnis bei variierenden Parametern?)
- Mehrjährige Cashflow-Prognose (0–50 Jahre konfigurierbar)

---

## Eingabe-Parameter

### Investitionen
- Bezeichnung, Investitionssumme, Lebensdauer (Jahre)
- Restwert, Ersatzinvestition, Instandhaltungskosten
- USt. (inkl./exkl.) mit konfigurierbarem Satz

### Laufende Kosten
- Energiekosten, Betriebskosten, Hilfskosten
- Preissteigerungsrate pro Kostenkategorie
- Entsorgungskosten (anteilig berechnet)

### Erlöse & Förderungen
- Energieerlöse (z. B. Einspeisevergütung)
- Einmalige Förderungen (z. B. zu Projektbeginn)
- Periodische Förderungen (z. B. jährliche Unterstützung über N Jahre)

### Berechnungsparameter
- Diskontierungssatz (%)
- Analysehorizont (Jahre)
- Startjahr der Investition

---

## Sensitivitätsanalyse

Drei Parameter können variiert werden:
- Diskontierungssatz (+/- X Prozentpunkte)
- Energiepreissteigerung (+/- X %)
- Investitionskosten (+/- X %)

---

## Ausgabe & Berichte

Vollständiger **PDF-Bericht** mit:
- Projektmetadaten (Auftraggeber, Standort, Verfasser)
- Organisations-Branding (Logo, Farben, Kopfzeile)
- Ergebniszusammenfassung
- Jahres-Cashflow-Tabelle
- Diagramme (Kumulierter Cashflow, Sensitivitätskurven)
- Normdex-Version und Erstellungsdatum

---

## Nutzungshinweise

- Berechnung kann **standalone** oder **projektgebunden** durchgeführt werden
- Berechnungsergebnisse werden persistent gespeichert
- Mehrere Berechnungen pro Projekt möglich (verschiedene Szenarien)

---

## ÖNORM M 7140 – Vollständiger Normentitel

*„Betriebswirtschaftliche Vergleichsrechnung für Energiesysteme nach der erweiterten Annuitätenmethode"*

**Drei Kostengruppen der Norm:**
1. Kapitalgebundene Kosten (Investition, Instandsetzung)
2. Betriebsgebundene Kosten (Wartung, Bedienung)
3. Verbrauchsgebundene Kosten (Energie, Brennstoffe)

**Validierung:** Abschnitt 10 enthält Validierungsbeispiele; die Berechnungslogik von Normdex wurde anhand dieser Beispiele validiert.

**8 Umsetzungspunkte von Normdex:**
1. Kapitalwert-, Annuitäten- und Amortisationsrechnung vollständig implementiert
2. Kostenstruktur gemäß Norm: kapital-, betriebs- und verbrauchsgebundene Kosten
3. Energieträgerspezifische Preissteigerungsraten über den Betrachtungszeitraum
4. Beliebig viele Varianten pro Projekt vergleichbar
5. Sensitivitätsanalyse für Energiepreise und Zinssätze
6. Validierte Berechnungslogik nach Abschnitt 10 der Norm
7. Normkonforme PDF-Reports mit allen Eingabeparametern und Ergebnissen
8. Nachvollziehbare Dokumentation jedes Berechnungsschritts

---

## Verwandte Dokumente

- [[Funktionen im Detail]]
- [[API-Endpunkte]]
- [[Datenmodell]]
