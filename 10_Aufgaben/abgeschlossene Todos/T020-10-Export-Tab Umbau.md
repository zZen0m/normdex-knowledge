# T020-10 · Export-Tab Umbau (vormals "Bericht")

**Phase:** 3 (Mittleres Feature)
**Priorität:** P2 · DB-Migration nötig
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Der Tab "Bericht" wird zu "Export" umbenannt und konzeptionell überarbeitet:
- Sektionen standardmäßig eingeklappt.
- Häkchen standardmäßig alle gesetzt.
- Einstellungen pro Projekt persistiert (nicht pro Org wie bisher).
- Neuer Haken "Resümee" in Sektion "Allgemein", der die Sichtbarkeit der Resümee-Seite (siehe T020-11) im PDF steuert.

## Betroffene Dateien
- Frontend: `apps/frontend/src/pages/EconomicsForm.tsx` (Tab-Beschriftung), `apps/frontend/src/pages/EconomicsReport.tsx` (Logik).
- Types: `apps/frontend/src/pages/ReportTypes.ts:10-76` — `ReportVisibility` erweitern um `resume`.
- Backend: `apps/api/app/models.py` — neues Feld `report_visibility` (JSON) auf Project- oder Calculation-Model.
- DB-Migration via `db_migration`-Skill.
- API-Endpoint zum Lesen/Schreiben der projektgebundenen Visibility.

## Umsetzung
1. **Tab-Rename**: "Bericht" → "Export" überall im UI.
2. **DB-Migration**: Feld `report_visibility` (JSON, nullable, Default = alle Häkchen `true`) auf Project oder Calculation. Entscheidung: Project, da der Export pro Projekt sinnvoll ist.
3. **API**: GET/PUT für `report_visibility` pro Projekt. Beim ersten Öffnen ohne gespeicherte Werte → Defaults (alles `true`).
4. **UI**:
   - Alle Sektionen eingeklappt initial.
   - Häkchen-Änderung sofort speichern (debounce 500ms).
   - Neuer Haken "Resümee" in Sektion "Allgemein".
5. **PDF-Generator** (`apps/api/app/services/pdf_generator.py`): nutzt projekt-Visibility; Fallback auf Org-Settings nur wenn kein Projekt-Override.

## Akzeptanzkriterien
- [ ] Tab heißt "Export".
- [ ] Sektionen initial eingeklappt.
- [ ] Häkchen initial alle gesetzt.
- [ ] Änderungen werden pro Projekt persistiert.
- [ ] Beim erneuten Öffnen sind die Häkchen identisch zur vorherigen Session.
- [ ] Neuer Haken "Resümee" vorhanden.

## Verifikation
1. Projekt-A: Häkchen X deaktivieren → Tab schließen → wieder öffnen → Häkchen X bleibt deaktiviert.
2. Projekt-B: Häkchen X bleibt aktiviert (unabhängig von Projekt-A).
3. Migration ausführen, alte Projekte zeigen Defaults.
