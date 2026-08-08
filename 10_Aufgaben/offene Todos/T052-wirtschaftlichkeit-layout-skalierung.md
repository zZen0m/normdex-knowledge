# T052 · Wirtschaftlichkeitsberechnung: Layout und Skalierungsverhalten

**Phase:** App / Wirtschaftlichkeitsberechnung / Darstellung  
**Priorität:** P2 · UX-Verbesserung / Responsive  
**Status:** offen  
**Datum:** 2026-08-08

## Ziel

Die Eingabekarten der Wirtschaftlichkeitsberechnung sollen sich an der Breite ihrer **eigenen Spalte** ausrichten statt am Viewport. Damit verschwindet das Zusammenschieben der Felder auf normalen Laptop-Breiten, und der Layout-Sprung bei 1280 px entfällt.

## Kontext

Auf einem 1585-px-Bildschirm ist das Feld „Anzahl Dezimalstellen" derzeit **107 px breit**. Die Rechnung:

```
1585 Viewport − 64 App-Rail − 48 px-6-Padding      = 1473
1473 − 220 Sidebar − 320 KPI-Panel − 48 (2× gap-6) =  885 Mittelspalte
885 → xl:grid-cols-3 + gap-6                       =  279 pro Karte
279 → p-6                                          =  231 Karteninhalt
231 → md:grid-cols-2 + gap-4                       =  107 pro Feld
```

Ursache ist keine Stilfrage, sondern eine falsche Bezugsgröße: Tailwind-Breakpoints messen den Viewport, die Karten sitzen aber in einer Spalte, deren Breite davon entkoppelt ist. `xl:grid-cols-3` (`FrameworkSection.tsx:129`) feuert bei 1280 px Viewport — dort hat die Mittelspalte nur rund 600 px. Die Karten erfahren nie, wie eng sie tatsächlich stehen.

Zweiter Effekt: Das KPI-Panel ist `hidden xl:block` (`EconomicsForm.tsx:2704`). Einen Pixel unter 1280 px verschwindet es ersatzlos, wodurch die Mittelspalte von 666 px auf 995 px springt — die Seite wird beim Schmalerziehen an dieser Stelle sprunghaft *besser*. Der eingeklappte 4-rem-Zustand des Panels ist bereits gebaut (`LiveKpiPanel.tsx:50-80`), wird aber nie automatisch ausgelöst.

Zur Datenlage: Für `app.normdex.at` existieren keine Nutzungszahlen. Plausible trackt ausschließlich die Landingpage `normdex.at`; `apps/frontend/index.html` enthält kein Tracking-Script, und die dortige CSP würde `analytics.normdex.at` ohnehin blockieren. Eine eigene Instrumentierung wurde bewusst verworfen. Das ist vertretbar, weil Container Queries stufenlos arbeiten und deshalb keinen konkreten px-Schwellwert benötigen.

## Umsetzung

- **Container Queries einführen**
  - `@tailwindcss/container-queries` installieren (Tailwind 3.3.5 braucht das Plugin; ab Tailwind 4 nativ) und in `tailwind.config.ts` registrieren.
  - Die Mittelspalte in `EconomicsForm.tsx:1483` als `@container` markieren.
  - In `FrameworkSection.tsx:129` `lg:/xl:`-Präfixe durch Container-Varianten ersetzen. Richtwerte: 3 Karten ab ~1100 px Spaltenbreite, 2 ab ~700 px, darunter gestapelt.
  - Ebenso die kartierten Innenraster (`:152`, `:178`, `:255`) auf Container-Varianten umstellen — sie leiden am selben Fehler.
- **Kartenraster asymmetrisch nach Inhaltsbedarf**
  - Zwölfspaltiges Raster statt gleicher Drittel. „Finanzen" hat die kürzesten Beschriftungen (Währung, USt-Satz) und darf schmal bleiben; „Allgemeines" (600-Zeichen-Textfeld) und „Preisentwicklungsraten" (sehr lange Feldnamen wie „Standardpreisentwicklungsrate für haustechnische Anlagenteile") bekommen mehr Breite.
  - Damit entfällt auch der leere Boden der Allgemeines-Karte, den `flex-1` + `h-full` derzeit erzeugt.
- **KPI-Panel progressiv verdichten statt ausblenden**
  - `hidden xl:block` entfernen. Bei knappem Platz klappt das Panel automatisch auf die vorhandene 4-rem-Leiste ein, statt zu verschwinden.
  - **Manuelle Entscheidung gewinnt:** `LIVE_PREVIEW_COLLAPSED_STORAGE_KEY` muss drei Zustände unterscheiden — Schlüssel nicht vorhanden = nie angefasst, hier greift die automatische Breitenlogik; Schlüssel gesetzt = Nutzerwunsch, bleibt unangetastet. Derzeit wird nur `"1"`/`"0"` gespeichert (`EconomicsForm.tsx:1440-1448`), der Zustand „nie angefasst" fehlt.
- **Unter 1024 px**
  - Arbeitsannahme: lesbar und bedienbar, kein Feinschliff. Kein Touch-Optimierungsaufwand.
  - Linke Navigation wird zum kompakten Schrittwähler oben statt zum vollbreiten Block über dem Inhalt.
  - KPI-Panel über Schaltfläche als Overlay erreichbar.

## Offene Fragen (vor Umsetzung klären)

- [ ] Ab welcher Spaltenbreite genau soll von 3 auf 2 Karten gewechselt werden? Richtwerte oben sind aus dem Inhalt abgeleitet, nicht gemessen — am realen Aufbau nachjustieren.
- [ ] Soll die 220-px-Navigation bei knappem Platz ebenfalls auf eine Icon-Leiste schrumpfen, oder bleibt sie bis 1024 px in voller Breite?

## Akzeptanzkriterien

- [ ] Kein Eingabefeld der Rahmenbedingungen ist bei 1366 px Viewport schmaler als 180 px.
- [ ] Beim stufenlosen Schmalerziehen von 1920 auf 1024 px gibt es keine Stelle, an der das Layout durch Wegfall eines Elements breiter wird.
- [ ] Das KPI-Panel bleibt zwischen 1024 und 1280 px erreichbar (mindestens als eingeklappte Leiste).
- [ ] Wer das Panel manuell aufklappt, behält es aufgeklappt, auch wenn die Breite darunter fällt.
- [ ] Beschriftungen der Preisentwicklungsraten brechen bei 1585 px auf höchstens zwei Zeilen um.
- [ ] Unter 1024 px ist die Seite ohne horizontales Scrollen bedienbar.
- [ ] Keine Änderung an Berechnungslogik oder gespeicherten Daten.
