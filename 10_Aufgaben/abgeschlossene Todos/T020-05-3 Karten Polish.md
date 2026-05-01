# T020-05 · Drei Karten "Projektadresse / Auftraggeber / Sachbearbeitung" – Polish

**Phase:** 1 (Quick Win)
**Priorität:** P0
**Parent:** [[T020-allgemeine Todos]]
**Status:** erledigt
**Abgeschlossen:** 2026-05-01

## Beschreibung
Die drei Karten auf Projektdetail- und Bearbeiten-Seite wirken aktuell schlicht. Es geht um visuelles Polish (Icons, Spacing, Typografie) — Struktur bleibt unverändert.

## Betroffene Dateien
- `apps/frontend/src/pages/ProjectDetail.tsx:391-433` — View-Modus.
- `apps/frontend/src/pages/ProjectDetail.tsx:547-625` — Edit-Modus.

## Umsetzung
- Pro Karte ein passendes Lucide-Icon (z.B. `MapPin` für Projektadresse, `Briefcase` für Auftraggeber, `User` für Sachbearbeitung).
- Konsistentes Padding, klare Heading-Typografie, dezente Kartentrennung.
- Mobile-Layout verifizieren.

## Akzeptanzkriterien
- [x] Drei Karten visuell aufgewertet, mit Icons.
- [x] Inhalt und Datenfelder unverändert.
- [x] Responsive auf Mobile/Tablet/Desktop.

## Verifikation
View- und Edit-Modus durchklicken, Mobile-Breakpoint testen.
