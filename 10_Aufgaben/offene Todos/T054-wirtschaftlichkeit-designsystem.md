# T054 · Wirtschaftlichkeitsberechnung: Design-System konsolidieren

**Phase:** App / Wirtschaftlichkeitsberechnung / Frontend-Refactor  
**Priorität:** P3 · Wartbarkeit / visuelle Konsistenz  
**Status:** offen  
**Datum:** 2026-08-08

## Ziel

Die Sections der Wirtschaftlichkeitsberechnung sollen dieselben Bausteine verwenden, damit Schriftgrößen, Feldhöhen und Dunkelmodus-Farben nicht länger je Section auseinanderlaufen. Farbe soll auf diesem Bildschirm Bedeutung tragen statt zu dekorieren.

## Kontext

**`SectionShell.tsx` ist toter Code.** Die Datei wird im gesamten Repo nicht importiert. Sie definiert ein vollständiges Kopfzeilensystem mit Phasen-Label und Schrittnummer (`phase="Berechnung"`, `step={2}` → „BERECHNUNG / 2. Rahmenbedingungen") — passend zu den Phasen, die `OutlineSidebar.tsx:15-35` bereits gruppiert. Gebaut, nie angeschlossen. Stattdessen baut jede Section ihren eigenen Header: die FrameworkSection nimmt `font-heading font-bold text-lg` (`:50`), SectionShell hätte `text-2xl font-semibold` gesetzt. Daher springt die Typografie beim Durchklicken der Reiter.

**Die FrameworkSection umgeht das eigene UI-Kit.** Sie baut ein rohes `<textarea>` mit acht handgeschriebenen Utility-Klassen (`:143`), ein rohes `<input type="checkbox">` mit hartkodiertem `border-gray-300` (`:307`, im Dunkelmodus falsch) und einen eigenen `CardHeader` (`:422`) — obwohl `ui/textarea.tsx`, `ui/checkbox.tsx` und `ui/card.tsx` existieren. Selects sind mal `h-10` (Finanzen, `:187`), mal `h-9` (Sensitivitätsanalyse, `:365`), auf demselben Bildschirm. Ein `ui/select.tsx` fehlt im Kit vollständig — das erklärt, warum Selects überall neu erfunden werden.

**Tooltips sind `title`-Attribute** (`form-field.tsx:20`). Nicht tastaturerreichbar, rund eine Sekunde Verzögerung, kein Styling. `@radix-ui/react-tooltip` ist installiert und `ui/tooltip.tsx` existiert — beides ungenutzt an dieser Stelle.

**Vier dekorative Icon-Farben an den Kartenköpfen** kollidieren mit der semantischen Farbnutzung desselben Bildschirms: `primary`/Petrol für Allgemeines (`:424`), `emerald-600` für Finanzen (`:425`), `blue-600` für Preisentwicklungsraten (`:426`), `violet-600` für die Sensitivitätsanalyse (`:287`, dort direkt im Markup statt über `CardHeader`). Gleichzeitig markiert `emerald` im Live-Panel das empfohlene System (`LiveKpiPanel.tsx:174,187`) und in der Navigation den Status „abgeschlossen" (`OutlineSidebar.tsx:109`), Amber steht für „Eingabe fehlt", Rot für Fehler. Dasselbe Grün bedeutet am Kartenkopf schlicht „Finanzen", also nichts — das schwächt die Signale, die tatsächlich etwas heißen.

Zur Einordnung: Der Kontrast der Kartenüberschriften wurde geprüft und ist **kein** Barrierefreiheitsproblem — `--muted-foreground` auf `bg-muted/30` ergibt 4,6:1 im Hell- und 7,3:1 im Dunkelmodus, beides besteht WCAG AA.

## Umsetzung

- **SectionShell wiederbeleben**
  - In allen sechs Sections einsetzen (Projektdaten, Rahmenbedingungen, Systeme, Ergebnisse, Resümee, Export), jeweils mit `phase` und `step`.
  - Die selbstgebauten Header dort entfernen, u. a. `FrameworkSection.tsx:48-86`. Der bestehende Aktionsbereich (Bearbeiten / Speichern / Abbrechen) wandert in den `actions`-Slot.
  - Phasen- und Schrittzuordnung aus `SECTION_GROUPS` in `OutlineSidebar.tsx:15-35` ableiten, damit Sidebar und Kopfzeile nicht getrennt gepflegt werden müssen.
- **UI-Kit durchsetzen**
  - `ui/select.tsx` ergänzen (`@radix-ui/react-select` ist noch nicht in den Abhängigkeiten — prüfen, ob ein natives Select mit einheitlichem Styling genügt).
  - Rohes `<textarea>` → `ui/textarea.tsx`; rohes `<input type="checkbox">` → `ui/checkbox.tsx`; eigener `CardHeader` → `ui/card.tsx`.
  - Alle Selects auf einheitliche Höhe bringen (`h-10`, passend zu `ui/input.tsx`).
  - `border-gray-300` und vergleichbare feste Farben durch Tokens ersetzen, damit der Dunkelmodus stimmt.
- **Tooltips auf Radix umstellen**
  - `title`-Attribut in `form-field.tsx:20` durch `ui/tooltip.tsx` ersetzen. Tastaturfokus muss den Tooltip auslösen.
  - Bei der Gelegenheit prüfen, welche Felder wirklich einen Tooltip brauchen und welche besser eine dauerhaft sichtbare Hilfszeile bekommen — derzeit ist die Wahl zwischen `tooltip` und `description` uneinheitlich.
- **Akzentfarben vereinheitlichen**
  - Alle vier Kartenköpfe auf `primary` (Petrol) setzen; die `tone`-Zuordnung in `CardHeader` entfällt.
  - Das direkt im Markup stehende Violett der Sensitivitätsanalyse (`:287`) mit auflösen.
  - Grün, Amber und Rot bleiben ausschließlich Signalfarben.
- **Nebenbefund mitnehmen**
  - `tailwind.config.ts` definiert `safelist` zweimal (Zeile 14 und Zeile 112). Der zweite Eintrag überschreibt den ersten, `animate-shake` ist dadurch **nicht** gesafelistet. Zusammenführen und prüfen, ob die Shake-Animation aktuell irgendwo ausfällt.

## Offene Fragen (vor Umsetzung klären)

- [ ] Soll `ui/select.tsx` auf `@radix-ui/react-select` aufsetzen (neue Abhängigkeit, bessere Tastaturbedienung, freies Styling) oder ein gestyltes natives `<select>` bleiben (kein Zusatzpaket, aber begrenzte Gestaltungsmöglichkeiten)?
- [ ] Übernimmt SectionShell auch den Aktionsbereich in allen Sections, oder behalten einzelne Sections abweichende Kopfzeilen-Layouts?

## Akzeptanzkriterien

- [ ] `SectionShell` wird in allen sechs Sections verwendet; keine Section baut noch eine eigene Kopfzeile.
- [ ] Überschriftengröße und Abstände sind beim Durchklicken aller sechs Schritte identisch.
- [ ] In der FrameworkSection existiert kein rohes `<textarea>`, `<input type="checkbox">` oder ungestyltes `<select>` mehr.
- [ ] Alle Eingabefelder und Selects der Section haben dieselbe Höhe.
- [ ] Keine hartkodierten Tailwind-Farbwerte mehr in der FrameworkSection; im Dunkelmodus sind alle Rahmen sichtbar.
- [ ] Tooltips öffnen sich per Tastaturfokus.
- [ ] Alle Kartenköpfe verwenden denselben Akzentton; Grün, Amber und Rot erscheinen ausschließlich in ihrer Signalbedeutung.
- [ ] `safelist` ist in `tailwind.config.ts` nur noch einmal definiert und enthält `animate-shake`.
- [ ] Keine funktionale Änderung: gleiche Felder, gleiche Werte, gleiche Berechnungsergebnisse.

Siehe auch: [[T052-wirtschaftlichkeit-layout-skalierung]], [[T053-wirtschaftlichkeit-bedienzustaende]]
