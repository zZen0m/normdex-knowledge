# T020-04 · Char-Counter in "Projekt bearbeiten" einblenden

**Status:** erledigt  
**Phase:** 1 (Quick Win)  
**Priorität:** P0  
**Parent:** [[T020-allgemeine Todos]]  
**Abgeschlossen:** 2026-05-01

## Ergebnis

In `apps/frontend/src/pages/ProjectDetail.tsx` wurden unter den Feldern "Projektname" und "Beschreibung" im Edit-Modus Zeichenzähler eingefügt. Format `X / Y` wie auf der Support-Seite. Der Counter wird rot (`text-destructive`), wenn das Limit überschritten wird.

- Projektname: `X / 80` (Limit aus Validierung in `handleSave`)
- Beschreibung: `X / 300` (Limit aus Validierung in `handleSave`)

## Akzeptanzkriterien

- [x] Counter live-aktualisiert beim Tippen.
- [x] Counter wird rot/warn, wenn Limit überschritten.
- [x] Submit weiterhin durch bestehende Validierung geschützt.
- [x] Konsistentes Format (`X / Y`) wie auf Support-Seite.
