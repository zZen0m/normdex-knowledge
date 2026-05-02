# Aufgaben

Zentrale Aufgabenliste für alle Bereiche des Normdex-Projekts.

Diese Datei ist die Übersicht. Detailinformationen stehen immer in der jeweiligen Todo-Datei unter `offene Todos/` oder `abgeschlossene Todos/`.

## Aufgabenstruktur

- `Aufgaben.md` listet alle Todos und verweist auf die Detaildateien.
- `offene Todos/` enthält alle offenen oder laufenden Todos.
- `abgeschlossene Todos/` enthält erledigte Todos, die aus `offene Todos/` verschoben wurden.
- Todo-Dateien haben stabile IDs im Format `T001-kurzer-titel.md`.
- Nummern werden nie wiederverwendet.

## Workflow

### Neues Todo anlegen

1. Nächste freie ID aus dieser Datei wählen.
2. Neue Datei unter `offene Todos/T###-kurzer-titel.md` anlegen.
3. Todo in der Liste „Offene Todos“ verlinken.
4. Ziel, Kontext, Akzeptanzkriterien und Fortschritt in der Detaildatei pflegen.

### Todo bearbeiten

- Status in der Detaildatei pflegen: `offen`, `in Arbeit`, `blockiert` oder `erledigt`.
- Neue Erkenntnisse, Entscheidungen und Zwischenstände unter „Notizen / Fortschritt“ ergänzen.
- Große Specs und Arbeitsprotokolle bleiben Referenzdokumente; daraus werden nur konkrete Arbeitspakete als Todos extrahiert.

### Todo abschließen

1. Status in der Todo-Datei auf `erledigt` setzen.
2. Abschlussdatum ergänzen.
3. Datei unverändert nummeriert nach `abgeschlossene Todos/` verschieben.
4. Link aus „Offene Todos“ nach „Abgeschlossene Todos“ verschieben.

## Referenzdokumente

## Offene Todos

| ID   | Todo                                                | Bereich                                       | Status    |
| ---- | --------------------------------------------------- | --------------------------------------------- | --------- |
| T019 | [[T019-newsletter-gutschein-brevo-webhook-rollout]] | App / Newsletter / Brevo / Stripe             | in Arbeit |
| T017 | [[T017-testzeitraum-fuer-lizenzen]]                 | App / Lizenzen / Stripe / Legal               | in Arbeit |
| T016 | [[T016-bericht-tab-pdf-export-ueberarbeiten]]       | App / Wirtschaftlichkeitsberechnung / Bericht | in Arbeit |
| T014 | [[T014-kuendigungs-email-fuer-lizenzen]]            | App / Lizenzen / E-Mail                       | offen     |
| T013 | [[T013-lizenzsystem-rollout-abschliessen]]          | App / Infrastruktur / Marketing               | in Arbeit |
| T020 | [[T020-allgemeine Todos]]                           | App                                           | offen     |

## Zusammengeführte Todo-IDs

Die früheren Einzel-Todos `T003` bis `T011` wurden in [[T013-lizenzsystem-rollout-abschliessen]] zusammengeführt. Diese IDs werden nicht wiederverwendet.

## Abgeschlossene Todos

| ID   | Todo                                     | Bereich | Abgeschlossen |
| ---- | ---------------------------------------- | ------- | ------------- |
| T012 | [[T012-projektstatus-einfuehren]]        | App     | 2026-04-27    |
| T018 | [[T018-landingpage-kaufintent-in-app-checkout]] | Landingpage / App / Lizenzen | 2026-04-30 |
| T015 | [[T015-toast-guideline-und-notify-helper]] | App / Designsystem / Frontend | 2026-04-27 |
| T001 | [[T001-normdex-complete-guide-loeschen]] | Vault   | 2026-04-27    |
| T002 | [[T002-repo-docs-vault-pruefung]]        | Vault   | 2026-04-27    |


## Nächste freie ID

`T021`
