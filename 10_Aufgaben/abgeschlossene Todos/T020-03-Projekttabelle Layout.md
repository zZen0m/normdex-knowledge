# T020-03 · Projekttabelle & Detail-Layout angleichen

**Status:** erledigt  
**Phase:** 1 (Quick Win)  
**Priorität:** P0  
**Parent:** [[T020-allgemeine Todos]]  
**Abgeschlossen:** 2026-05-01

## Ergebnis

Zwei Dateien angepasst, damit Projekttabelle und Projektdetailseite dieselbe Container-Breite (`max-w-[1800px]`) verwenden:

- `apps/frontend/src/pages/Projects.tsx` (Zeile 147): `container mx-auto` → `w-full max-w-[1800px] mx-auto`
- `apps/frontend/src/components/layout/Header.tsx` (Zeile 86): `/projects`-Route zur `max-w-[1800px]`-Bedingung hinzugefügt, damit die Seitenüberschrift "Projekte" bündig mit dem Tabellenrand ausgerichtet ist.

## Akzeptanzkriterien

- [x] Tabelle hat gleiche maximale Breite wie Projektdetailseite.
- [x] Überschrift "Projekte" linksbündig auf gleicher vertikaler Linie wie Tabellenrand.
- [x] Layout responsive bleibt erhalten (Mobile + Desktop).
