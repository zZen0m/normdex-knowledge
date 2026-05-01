# T020-11 · Resümee-Tab + PDF-Integration

**Phase:** 3 (Mittleres Feature)
**Priorität:** P2 · DB-Migration nötig · abhängig von T020-10
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Zwischen den Tabs "Ergebnisse" und "Export" wird ein neuer Tab "Resümee" eingefügt. Dort:
1. Dropdown "Empfohlenes System" — Optionen aus den im Projekt definierten WBR-Varianten.
2. Textfeld "Resümee" mit Char-Counter (z.B. 2000 Zeichen).

Im PDF wird das Resümee auf der bestehenden Resümee-Seite angezeigt; die Sichtbarkeit wird durch den Haken aus T020-10 gesteuert.

## Betroffene Dateien
- Frontend: `apps/frontend/src/pages/EconomicsForm.tsx:1730-1787` — Tabs.
- Backend: `apps/api/app/models.py` — neue Felder `resume_recommended_variant_id` + `resume_text` auf Calculation- oder Project-Model.
- API-Endpoint: GET/PUT zum Speichern des Resümees.
- PDF: `apps/api/app/services/pdf_generator.py:3109-3144` — `_draw_resume()` auf neue Felder umstellen.
- DB-Migration via `db_migration`-Skill.

## Umsetzung
1. **Tab-Reihenfolge**: `[Übersicht, Diagramme, Sensitivität, Resümee, Export]`. Bestehende Indizes nachziehen, alle `resultsTabIndex`-Vergleiche prüfen.
2. **DB-Migration**: Felder `resume_recommended_variant_id` (nullable FK/INT) + `resume_text` (Text, nullable, Limit 2000).
3. **UI**:
   - Dropdown gespeist aus Calculation-State (alle Varianten des Projekts).
   - Textfeld mit Counter (Pattern aus T020-04 / Support.tsx).
   - Auto-Save bei Blur oder debounced.
4. **PDF**: `_draw_resume()` zeigt Empfehlungs-Hinweis prominent + Resümee-Text. Skipping wenn Visibility-Flag aus T020-10 = false.

## Akzeptanzkriterien
- [ ] Neuer Tab zwischen "Ergebnisse"/"Sensitivität" und "Export".
- [ ] Dropdown listet alle WBR-Varianten des Projekts.
- [ ] Textfeld mit Char-Counter (max 2000).
- [ ] Daten werden persistiert.
- [ ] PDF zeigt Resümee + Empfehlung korrekt.
- [ ] Häkchen "Resümee" aus T020-10 steuert Sichtbarkeit im PDF.

## Verifikation
1. Variante wählen, Text schreiben → Tab wechseln/zurück → Daten erhalten.
2. Export → PDF zeigt Resümee.
3. Häkchen deaktivieren → PDF ohne Resümee-Seite.
