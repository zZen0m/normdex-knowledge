# T041 · Referenzwerte für Kostenkategorien je Anlagentyp

**Phase:** App / Wirtschaftlichkeitsberechnung / Eingabe  
**Priorität:** P3 · UX-Verbesserung / Norm-Konformität  
**Status:** offen  
**Datum:** 2026-07-08

## Ziel

Bei der Eingabe der Systeme im Berechnungsformular sollen für kapital-, betriebs- und verbrauchsgebundene Kosten (gemäß ÖNORM M 7140) Vorschlagswerte je Anlagentyp angeboten werden (z. B. typische Wartungskosten für Wärmepumpen, Pelletkessel, PV-Anlagen). Die Werte sind Vorschläge, keine Zwangsvorgaben — Nutzer können sie jederzeit überschreiben.

## Kontext

Aktuell gibt es in `EconomicsForm.tsx` keine strukturierte Anlagentyp-Klassifizierung pro System, nur ein freies Namensfeld ("Bezeichnung"). Für Vorschlagswerte braucht es zuerst eine leichte Kategorisierung (z. B. "Wärmepumpe", "Pelletkessel", "PV-Anlage", "Gasheizung"), an die sich Richtwerte anhängen lassen.

Ergänzend wurde die Idee diskutiert, bei Übernahme eines Vorschlagswerts automatisch im PDF-Bericht auf die Quelle zu verweisen (Norm/Tabelle bzw. Erfahrungswert). Das erhöht die Nachvollziehbarkeit im Sinne von Normdex' Kernversprechen ("nachvollziehbare Dokumentation nach Norm"), erfordert aber eine präzise, einzeln gepflegte Quellenangabe pro Wert — ein falsch oder ungenau zitierter Normverweis wäre für Planungsbüros riskanter als gar keiner.

## Umsetzung

- Datenmodell:
  - Optionales Klassifizierungsfeld "Anlagentyp" pro System ergänzen (nicht verpflichtend, um Bestandsprojekte nicht zu brechen).
  - Neue Referenzwerte-Tabelle/Konfiguration: Anlagentyp → typische Werte für kapital-, betriebs- und verbrauchsgebundene Kosten, inkl. Quellenangabe pro Wert.
- Frontend:
  - Bei Auswahl eines Anlagentyps Vorschlagswerte in den entsprechenden Kostenfeldern anbieten (z. B. als ausfüllbarer Platzhalter oder Ein-Klick-Übernahme), ohne die freie Eingabe einzuschränken.
  - Optionale Plausibilitäts-Warnung, wenn ein eingegebener Wert stark von der Richtwerte-Spanne abweicht (dezenter Hinweis, kein Blocker).
- PDF-Bericht:
  - Bei übernommenem Vorschlagswert optionalen Quellenverweis bei der jeweiligen Kostenposition einblenden.

## Offene Fragen (vor Umsetzung klären)

- [ ] Quelle der Referenzwerte: konkrete Norm/Tabelle, eigene Erfahrungswerte, oder Mix je Wert? (Stand 2026-07-08: noch offen, spätere Diskussion vorgesehen)
- [ ] Wie granular soll die Anlagentyp-Klassifizierung sein (grobe Kategorien vs. detaillierte Systemtypen)?
- [ ] Pflege der Referenzwerte: statisch im Code oder administrierbar (z. B. über Admin-Panel)?

## Akzeptanzkriterien

- [ ] Systeme können optional einem Anlagentyp zugeordnet werden.
- [ ] Bei ausgewähltem Anlagentyp werden Vorschlagswerte für die relevanten Kostenfelder angeboten.
- [ ] Vorschlagswerte sind jederzeit überschreibbar, keine Pflichtübernahme.
- [ ] Bestehende Projekte ohne Anlagentyp-Zuordnung bleiben unverändert funktionsfähig.
- [ ] Quellenverweis im PDF-Bericht ist umgesetzt, sobald die offene Frage zur Werte-Quelle geklärt ist (kann als Folge-Task abgespalten werden, falls die Klärung sich verzögert).
