# T053 · Wirtschaftlichkeitsberechnung: Bedienzustände und Standardwerte

**Phase:** App / Wirtschaftlichkeitsberechnung / Eingabe  
**Priorität:** P2 · UX-Verbesserung  
**Status:** offen  
**Datum:** 2026-08-08

## Ziel

Das Berechnungsformular soll beim Öffnen nicht mehr wie ein defektes, ausgegrautes Formular wirken, der Bearbeiten-Modus soll keine Sackgasse mehr sein, und Nutzer sollen erkennen können, welche Werte ÖNORM-Standard und welche individuell gesetzt sind.

## Kontext

Beim Betreten der Rahmenbedingungen ist **jedes** Feld `disabled` — Währung, Zinssatz, alle vier Preisentwicklungsraten, Textfeld, Sensitivitätsanalyse. Alle tragen `disabled:opacity-50 disabled:cursor-not-allowed`. Die Seite wirkt dadurch nicht gespeichert, sondern kaputt. Der Einstieg über „Bearbeiten" oben rechts muss erraten werden.

Sobald der Bearbeiten-Modus aktiv ist, wird zusätzlich die **gesamte linke Navigation deaktiviert** (`EconomicsForm.tsx:1478`, `disabled={isEditingGlobal}`). Wer eine Zahl ändert, sitzt fest, bis er speichert oder abbricht — ohne dass die graue Sidebar erklärt, warum.

Die Semantik „leeres Feld = ÖNORM-Standard" ist unsichtbar. `PriceRateInput` kennt keinen Default-Begriff: Beim Leeren wird `onChange(undefined)` gefeuert (`PriceRateInput.tsx:41`). Es gibt einen `placeholder`-Prop, aber `PriceRateField` übergibt ihn nie — ein leeres Feld sieht schlicht vergessen aus. Dazu existiert eine dreistufige Hierarchie (ÖNORM → Projekt → Position), die die Oberfläche an keiner Stelle abbildet.

Der ÖNORM-Infokasten kostet rund 200 px oberhalb der Falz, auf jedem Besuch, inklusive dreizeiligem Kleingedruckten.

Der Ablauf hat **keine Weiter-/Zurück-Navigation**. Das einzige „Zurück" im gesamten Formular sitzt auf einem Fehlerbildschirm (`EconomicsForm.tsx:1326`). Sechs Schritte — Projektdaten, Rahmenbedingungen, Systeme, Ergebnisse, Resümee, Export — ohne jede Vorwärtsbewegung.

## Abgleich mit der Norm — zwei Fehler im Backend

Quelle: `03_external/sharepoint-normdex/03_Product-Development/03_Normen/OENORM_M_7140_2021_01_15_de.pdf`, Abschnitt 6.3, Tabelle 4 („Zinssätze und Preisentwicklungsraten – Vorschlagswerte (informativ)").

| Wert | ÖNORM Tabelle 4 | Infokasten (`FrameworkSection.tsx:102-112`) | Backend (`economics.py`) |
|---|---|---|---|
| Kalkulatorischer Zinssatz q | **2,5 %** | 2,5 % ✓ | **5** (`:80`) ✗ |
| Preisentwicklungsrate Energiekosten | 2,7 % | 2,7 % ✓ | 2.7 (`:92`) ✓ |
| Preisentwicklungsrate Personal HKLS | 3,2 % | 3,2 % ✓ | 3.2 (`:93`) ✓ |
| Preisentwicklungsrate haustechnische Anlagenteile | 3,5 % | 3,5 % ✓ | 3.5 (`:94`) ✓ |
| Preisentwicklungsrate Entsorgungskosten | **existiert nicht** | nicht genannt | **3.0** (`:95`) ✗ |

**Fehler 1:** Der Backend-Fallback für den kalkulatorischen Zinssatz ist mit 5 doppelt so hoch wie der Normwert von 2,5. Der Infokasten im Frontend ist korrekt.

**Fehler 2:** Tabelle 4 enthält **keinen** Vorschlagswert für Entsorgungskosten — sie hat genau vier Zeilen. Der Backend-Default `pm_disposal = 3.0` ist frei gesetzt, und die Feldbeschriftung „Standardpreisentwicklungsrate für Entsorgungskosten" suggeriert einen normativen Standard, den es nicht gibt.

**Wortwahl:** Die Norm hält eine normative Festlegung ausdrücklich für „nicht zweckmäßig" und legt daher nur *informative Richtwerte* fest, die verwendet werden dürfen, „wenn keine anderen fundierten Quellen zur Verfügung stehen". Zudem verlangt sie, dass die eingesetzten Werte samt Quelle im Bericht ausgewiesen werden. Der Marker darf deshalb nicht „ÖNORM-Standard" heißen, sondern muss den informativen Charakter transportieren — etwa „Vorschlagswert (informativ)".

**Zusatzfund:** Die Feldbeschreibung „Anzahl der Jahre der Analyse (Standard: 20 Jahre)" (`FrameworkSection.tsx:155`) ist ebenfalls nicht gedeckt. Abschnitt 6.2 legt keinen Betrachtungszeitraum fest, sondern verlangt einen zur Nutzungsdauer passenden und **begründeten** Zeitraum, der im Bericht anzuführen ist.

**Lizenzhinweis am Rande:** Die vorliegende PDF-Kopie trägt im Fußbereich eine Einzelplatz-Lizenz, ausgestellt auf ein anderes Unternehmen. In `00_Allgemein/` liegen separate Normdex-Lizenznachweise (`normdex_oenorm_m7140_basic_hauptlizenz.png`, `..._zusatzlizenz.png`). Ablage im geteilten SharePoint bei Gelegenheit prüfen — nicht Teil dieser Aufgabe.

## Umsetzung

- **Lesemodus statt disabled-Optik**
  - Nicht-bearbeitete Felder in voller Textfarbe darstellen: kein `opacity-50`, kein `cursor-not-allowed`, ruhiger Hintergrund statt Eingaberahmen.
  - Wichtig: **kein globales Suchen-und-Ersetzen.** Manche Elemente sind echt deaktiviert und müssen es bleiben — etwa „+ Parameter", solange die Sensitivitätsanalyse ausgeschaltet ist (`FrameworkSection.tsx:322`). Lesemodus und deaktiviert brauchen zwei klar unterscheidbare Darstellungen.
- **Navigation während des Bearbeitens freigeben**
  - `disabled={isEditingGlobal}` in `EconomicsForm.tsx:1478` entfernen.
  - Beim Sectionwechsel mit ungespeicherten Änderungen Dialog: Speichern / Verwerfen / Bleiben. Der Dirty-Zustand ist über `formState.isDirty` von react-hook-form verfügbar.
  - Gleiche Rückfrage bei den neuen Weiter-/Zurück-Schaltflächen.
- **Normwerte korrigieren** (zuerst, unabhängig von der Darstellung)
  - `discount_rate_pct` in `economics.py:80` von 5 auf 2,5 korrigieren.
  - Für `pm_disposal` (`:95`) entscheiden: entweder als bewusst gesetzten Hauswert kennzeichnen und im Bericht entsprechend ausweisen, oder auf `None` setzen und die Eingabe verpflichtend machen. Ein normativer Vorschlagswert existiert dafür nicht.
  - Prüfen, ob bestehende Projekte den fehlerhaften Zinssatz-Fallback tatsächlich angewendet haben. Da das Frontend den Wert immer mitsendet, greift der Fallback vermutlich nie — vor der Änderung verifizieren.
- **Vorschlagswerte sichtbar machen**
  - Eine einzige Quelle für die Werte aus Tabelle 4, aus der Frontend-Anzeige und Backend-Fallback gemeinsam gespeist werden.
  - Leeres Feld zeigt den geltenden Vorschlagswert als Platzhalter.
  - Marker am Feld: **„Vorschlagswert (informativ)"** bzw. „Individuell" — nicht „ÖNORM-Standard", weil die Norm ausdrücklich keine normative Festlegung trifft.
  - Bei abweichendem Wert ein Zurücksetzen-Symbol, das das Feld leert (= zurück auf den Vorschlagswert).
  - Das Entsorgungs-Feld darf keinen Vorschlagswert-Marker bekommen, solange keine Quelle dafür benannt ist.
- **Feldbeschriftungen an die Norm angleichen**
  - „Standard: 20 Jahre" beim Betrachtungszeitraum (`:155`) entfernen oder durch den Normhinweis ersetzen: Zeitraum passend zur Nutzungsdauer wählen und im Bericht begründen (Abschnitt 6.2).
  - „Standardpreisentwicklungsrate für Entsorgungskosten" umbenennen, da kein Standard existiert.
- **ÖNORM-Infokasten einklappbar**
  - Überschrift „Standardwerte gemäß ÖNORM M 7140" bleibt als schmale Zeile sichtbar, standardmäßig geschlossen. Titel auf „Vorschlagswerte" ändern, passend zur Tabellenüberschrift der Norm.
  - Tabelle und der Hinweis zu Großgewerbe/Industrie öffnen sich auf Klick.
  - Quellenangabe ergänzen (ÖNORM M 7140:2021-01, Abschnitt 6.3, Tabelle 4), da die Norm die Nennung der Quelle im Bericht verlangt.
- **Weiter-/Zurück-Navigation**
  - Fußbereich am Ende jeder Section mit „Zurück" und „Weiter zu: <nächster Schritt>".
  - Sidebar bleibt für freie Sprünge, die Schaltflächen bedienen den normalen Durchlauf.
  - Gesperrte Schritte (`status === "locked"`) müssen im Weiter-Ziel berücksichtigt werden.

## Offene Fragen (vor Umsetzung klären)

- [x] ~~Welcher Zinssatz gilt — 2,5 % oder 5 %?~~ Geklärt am 2026-08-08: ÖNORM M 7140:2021-01, Tabelle 4 nennt 2,5 %. Das Backend ist falsch.
- [ ] Woher stammt der Backend-Wert `pm_disposal = 3.0`? Die Norm kennt keinen Vorschlagswert für Entsorgungskosten. Eigener Erfahrungswert mit belegbarer Quelle, oder ersatzlos streichen?
- [ ] Wo liegt künftig die einzige Quelle der Vorschlagswerte — Backend liefert sie an das Frontend aus, oder geteilte Konstantendatei?
- [ ] Sollen bestehende Projekte, in denen ein Wert zufällig exakt dem Vorschlagswert entspricht, als „Vorschlagswert" oder als „Individuell" markiert werden? (Gespeichert sind sie als expliziter Wert, semantisch ist das ein Unterschied.)
- [ ] Die Norm verlangt, eingesetzte Werte samt Quelle im Bericht auszuweisen. Soll der neue Standard-/Individuell-Marker diese Angabe automatisch in den PDF-Bericht übernehmen, oder ist das ein eigener Folge-Task?

## Akzeptanzkriterien

- [ ] Beim Öffnen der Rahmenbedingungen ist kein Feld ausgegraut; alle Werte sind in voller Textfarbe lesbar.
- [ ] Echt deaktivierte Bedienelemente sind optisch klar von schreibgeschützten Feldern unterscheidbar.
- [ ] Die linke Navigation ist im Bearbeiten-Modus bedienbar.
- [ ] Ein Wechsel mit ungespeicherten Änderungen führt zu einer Rückfrage; nichts geht ungefragt verloren, nichts wird ungefragt gespeichert.
- [ ] Der Backend-Fallback für den kalkulatorischen Zinssatz beträgt 2,5 und stimmt damit mit ÖNORM Tabelle 4 überein.
- [ ] Ein leeres Preisentwicklungs-Feld zeigt den Vorschlagswert als Platzhalter, und dieser stimmt mit dem tatsächlich angewendeten Backend-Fallback überein.
- [ ] Vorschlagswert und individuell gesetzt sind am Feld unterscheidbar; die Beschriftung transportiert den informativen Charakter.
- [ ] Keine Feldbeschriftung behauptet einen normativen Standard, den die ÖNORM nicht festlegt (Betrachtungszeitraum, Entsorgungsrate).
- [ ] Der Infokasten ist beim Öffnen geschlossen und belegt höchstens eine Zeile.
- [ ] Jede Section außer der letzten hat eine Weiter-Schaltfläche, jede außer der ersten eine Zurück-Schaltfläche.
- [ ] Keine Änderung an Berechnungsergebnissen bestehender Projekte.

Siehe auch: [[T052-wirtschaftlichkeit-layout-skalierung]], [[T054-wirtschaftlichkeit-designsystem]]
