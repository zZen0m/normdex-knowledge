# T020-09 · Berechnungen-Karte als Hero-Card

**Phase:** 3 (Mittleres Feature)
**Priorität:** P2
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Die "Berechnungen"-Karte auf der Projektdetailseite wirkt aktuell deplatziert, weil nur eine Berechnung (ÖNORM-Wirtschaftlichkeit) existiert. Die Karte bleibt erhalten — als Container für künftige Berechnungstypen — wird aber visuell zu einer aussagekräftigen Hero-Card aufgewertet.

## Betroffene Dateien
- `apps/frontend/src/pages/ProjectDetail.tsx:483-507` — aktuelle Karte.

## Umsetzung
- Status der WBR (Entwurf / Berechnet / Exportiert).
- Letzte Bearbeitung (relativer Zeitstempel).
- Schlüsselkennzahl als Vorschau (z.B. wirtschaftlichste Variante / Amortisationszeit), falls Daten vorhanden.
- Prominenter "Öffnen"-CTA.
- Layout so, dass künftig weitere Berechnungstypen daneben/darunter passen, ohne Bruch.
- Empty State falls keine WBR existiert: Aufruf "Wirtschaftlichkeitsberechnung starten".

## Akzeptanzkriterien
- [ ] Karte zeigt Status, letzte Bearbeitung, Vorschau.
- [ ] CTA prominent und klickbar.
- [ ] Empty State sinnvoll, wenn keine WBR.
- [ ] Layout vorbereitet auf künftige Berechnungstypen.

## Verifikation
1. Projekt ohne WBR → Karte zeigt Empty State.
2. Projekt mit WBR (Entwurf, fertig, exportiert) → Karte zeigt jeweils richtigen Status.
