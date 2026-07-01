# T036 · Dashboard – Erste Schritte Popup merkt sich Zustand

**Phase:** App / Frontend / Backend / UX  
**Priorität:** P3 · UX-Verbesserung (später als Bugfix eingestuft)  
**Status:** erledigt  
**Datum:** 2026-07-01  
**Abgeschlossen:** 2026-07-01

## Ziel

Das „Erste Schritte"-Popup im Dashboard soll seinen ein- oder ausgeklappten Zustand speichern und beim nächsten Seitenaufruf wiederherstellen, statt immer standardmäßig ausgeklappt zu sein.

## Verlauf

Das Feature war bereits vollständig angelegt (serverseitige Speicherung über `user.settings.onboarding_widget_collapsed`, robuster als `localStorage` da geräteübergreifend). Todo wurde zunächst als "bereits erledigt" markiert.

Kurz darauf meldete ein Nutzer, dass sich das Popup trotzdem immer wieder von selbst ausklappt. Root-Cause-Analyse ergab zwei Bugs:

### Bug 1 — Backend: Änderung an JSON-Spalte wurde nicht zuverlässig persistiert
`apps/api/app/routers/users.py`, `PATCH /users/me/settings`: Die `settings`-Spalte ist ein SQLAlchemy-`JSON`-Feld. Ein neues Dict wurde zwar zugewiesen, aber ohne `flag_modified()` erkennt SQLAlchemy die Änderung am JSON-Blob nicht in jedem Fall zuverlässig als "dirty" — der Commit konnte dadurch wirkungslos bleiben.

### Bug 2 — Frontend: Speicherfehler wurden stillschweigend verschluckt
`apps/frontend/src/components/dashboard/OnboardingWidget.tsx`, `handleToggle()`: `api.settingsPatch(...).catch(() => {})` ignorierte jeden Fehler komplett. Schlug das Speichern fehl (z. B. Netzwerkfehler, abgelaufene Session), gab es weder einen Log-Eintrag noch eine Chance, das Problem zu bemerken — der zuletzt gewählte Zustand ging kommentarlos verloren und das Popup öffnete sich beim nächsten Laden wieder.

## Fix

- `apps/api/app/routers/users.py`: `flag_modified(user, "settings")` vor `db.commit()` ergänzt, damit Änderungen an der JSON-Spalte zuverlässig geschrieben werden.
- `apps/frontend/src/components/dashboard/OnboardingWidget.tsx`: Fehler beim Speichern werden jetzt mit `console.error(...)` geloggt statt verschluckt, um zukünftige Fehlerursachen sichtbar zu machen.

## Verifikation

- Direkter DB-Test im Dev-Container (frische SQLAlchemy-Session nach Commit) bestätigt: Mit `flag_modified()` wird der geänderte `onboarding_widget_collapsed`-Wert zuverlässig persistiert und ist nach Neuladen aus der DB vorhanden (getestet für `True` und `False`).
- End-to-End-Test über die laufende App wurde vom Nutzer selbst durchgeführt (nicht im Rahmen dieser Session protokolliert).

## Akzeptanzkriterien

- [x] Nutzer klappt das Popup ein → Zustand wird gespeichert (serverseitig, jetzt mit `flag_modified`)
- [x] Nutzer klappt das Popup aus → Zustand wird gespeichert (serverseitig)
- [x] Beim nächsten Seitenaufruf: Panel startet im zuletzt gespeicherten Zustand
- [x] Kein gespeicherter Zustand → Panel startet ausgeklappt (Standardverhalten)
- [x] Zustand ist nutzerspezifisch (serverseitig, geräteübergreifend)
- [x] Speicherfehler werden nicht mehr stillschweigend verschluckt (Logging ergänzt)
