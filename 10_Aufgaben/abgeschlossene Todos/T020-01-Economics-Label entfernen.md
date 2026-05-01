# T020-01 · "Economics"-Label überall entfernen

**Status:** erledigt  
**Phase:** 1 (Quick Win)  
**Priorität:** P0 · niedriges Risiko · hohe Sichtbarkeit  
**Parent:** [[T020-allgemeine Todos]]  
**Abgeschlossen:** 2026-05-01

## Ergebnis

Der `calculationType`-Badge (z.B. "Economics") wurde an beiden Stellen entfernt:

- `apps/frontend/src/pages/ProjectDetail.tsx` — Badge unter dem Projekttitel in der Projektdetailansicht.
- `apps/frontend/src/pages/EconomicsForm.tsx` — Badge auf der WBR-Übersichtsseite (Tab "project"), unterhalb des `<h1>` mit dem Projektnamen.

Das Layout bleibt durch das Flexbox-`gap` in beiden Fällen lückenfrei.
