# T040 · Projekt-Duplizieren

**Phase:** App / Projektverwaltung / Effizienz  
**Priorität:** P3 · UX-Verbesserung / Erfassungsaufwand  
**Status:** offen  
**Datum:** 2026-07-08

## Ziel

Anwender sollen ein bestehendes Projekt als Kopie anlegen können, um Varianten desselben Vorhabens (z. B. „mit/ohne Speicher“ oder gleicher Anlagentyp an anderem Standort) zu erstellen, ohne alle Wirtschaftlichkeits-Eingaben erneut erfassen zu müssen.

## Kontext

Im Code existiert bereits ein Duplizier-Muster auf niedrigerer Ebene: `handleDuplicateSystem` (`apps/frontend/src/pages/EconomicsForm.tsx`, Zeile 941) dupliziert ein einzelnes System per `deepClone` *innerhalb* eines Projekts. Die neue Funktion hebt dieses Prinzip auf Projekt-Ebene. Laut `Project`-Modell (`apps/api/app/models.py`, Zeile 41) enthält ein Projekt Stammdaten (Name, Adresse, Auftraggeber, Verfasser) plus `form_data` (JSON mit allen Wirtschaftlichkeits-Eingaben) — letzteres ist der eigentliche Wert eines Duplikats.

## Umsetzung

- Backend:
  - `POST /projects/{id}/duplicate` — liest bestehendes Projekt, legt neues `Project`-Objekt an mit `form_data = copy.deepcopy(original.form_data)`.
  - Neue, eindeutige `project_number` automatisch vergeben.
  - `status` wird auf `new` zurückgesetzt.
  - Name-Vorschlag `"{original.name} (Kopie)"` als Default, überschreibbar über Frontend-Dialog.
  - **Nicht** übernommen: `resume_text` / `resume_recommended_variant_id` (Berichts-Zusammenfassung ist projektspezifisch), verknüpfte `Calculation`-Historie (nur aktueller `form_data`-Stand wird kopiert, keine alten Berechnungsläufe).
  - Audit-Event `project_duplicated` mit `meta.source_project_id` ergänzen (Audit-Muster analog zu `project_created` in `apps/api/app/routers/projects.py`, Zeile 245) — landet automatisch im Aktivitäts-Feed aus [[T039-kommentar-aktivitaets-feed-pro-projekt]], falls dieses vorher oder gleichzeitig umgesetzt wird.
  - Prüfen, ob ein Projekt-Kontingent pro Lizenz existiert; falls ja, zählt ein Duplikat als neues Projekt dagegen.
- Frontend:
  - Button „Duplizieren“ in der Projekt-Detailansicht (`ProjectDetail.tsx`) und optional als Kontextmenü-Aktion in der Projektliste (`Projects.tsx`).
  - Kleiner Dialog zur Namensvergabe direkt beim Duplizieren, statt nur automatisch „(Kopie)“ anzuhängen.
  - Nach Duplizieren: Weiterleitung zur neuen Projekt-Detailseite mit Erfolgs-Toast „Projekt dupliziert als '{name}'“.

## Offene Fragen (vor Umsetzung klären)

- [ ] Soll `report_visibility` (Sichtbarkeits-Einstellungen für den PDF-Bericht) mitkopiert werden?
- [ ] Gibt es ein Projekt-Kontingent pro Lizenz, das bei der Umsetzung berücksichtigt werden muss?

## Abgrenzung

- Kein org-übergreifendes Duplizieren (z. B. als Vorlage für andere Kunden) — nur Duplizieren innerhalb derselben Organisation.

## Akzeptanzkriterien

- [ ] In der Projekt-Detailansicht gibt es eine „Duplizieren“-Aktion.
- [ ] Das Duplikat übernimmt alle Wirtschaftlichkeits-Eingaben (`form_data`) sowie Adress-/Auftraggeber-/Verfasserdaten.
- [ ] Das Duplikat erhält eine neue, eindeutige Projektnummer und den Status „new“.
- [ ] Berichts-Zusammenfassung (`resume_text`) und alte Berechnungsläufe werden nicht übernommen.
- [ ] Der Nutzer kann den Namen des Duplikats vor dem Anlegen anpassen.
- [ ] Ein `project_duplicated`-Audit-Event wird erzeugt.
- [ ] Bestehendes Projekt bleibt unverändert erhalten.
