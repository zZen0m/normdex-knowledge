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

| ID      | Todo                                            | Bereich                          | Status |
| ------- | ----------------------------------------------- | -------------------------------- | ------ |
| T050    | [[T050-ga4-tracking-nach-consent-fix-fast-keine-daten]] | App / Marketing / Tracking | offen |
| T049    | [[T049-google-marketing-kpis-verwaltungsportal]] | App / Admin / Verwaltungsportal / Marketing | offen |
| T045    | [[T045-systemlogs-in-adminoberflaeche]]          | App / Admin / Verwaltungsportal / Diagnose | offen |
| T043    | [[T043-oenorm-seite-inhaltlicher-ausbau]]        | Landingpage / Marketing / SEO / Content | offen |
| T042    | [[T042-seo-analyse-landingpage]]                 | Landingpage / Marketing / SEO    | offen |
| T041    | [[T041-referenzwerte-kostenkategorien-systeme]]  | App / Wirtschaftlichkeitsberechnung / Eingabe | offen |
| T040    | [[T040-projekt-duplizieren]]                     | App / Projektverwaltung / Effizienz | offen |
| T039    | [[T039-kommentar-aktivitaets-feed-pro-projekt]]  | App / Projektverwaltung / Zusammenarbeit | offen |
| T034    | [[T034-excel-export-wirtschaftlichkeitsbericht]] | App / Wirtschaftlichkeitsberechnung / Export | offen |
| T026    | [[T026-secret-rotation-und-history-cleanup]]     | App / Security / Infrastruktur   | in Arbeit |

## Zusammengeführte Todo-IDs

Die früheren Einzel-Todos `T003` bis `T011` wurden in [[T013-lizenzsystem-rollout-abschliessen]] zusammengeführt. Diese IDs werden nicht wiederverwendet.

## Abgeschlossene Todos

| ID      | Todo                                                | Bereich                          | Abgeschlossen |
| ------- | --------------------------------------------------- | -------------------------------- | ------------- |
| T037    | [[T037-fachbeitrag-content-struktur-landingpage]] | Landingpage / Marketing / Content / SEO | 2026-07-24 |
| T051    | [[T051-beispielbericht-oeffentlich-downloadbar]] | Landingpage / Marketing / Conversion | 2026-07-24 |
| T048    | [[T048-linkedin-marketing-kpis-verwaltungsportal]] | App / Admin / Verwaltungsportal / Marketing | 2026-07-19 |
| T047    | [[T047-linkedin-api-freigabe-statistiken]]       | Marketing / LinkedIn-Markenkanal / Reporting | 2026-07-19 |
| T046    | [[T046-ssg-vite-react-ssg-landingpage]]          | Landingpage / Technisches SEO / Rendering | 2026-07-19 |
| T044    | [[T044-letzter-login-mitgliederliste-verwaltungsportal]] | App / Admin / Kundenakte / Mitglieder | 2026-07-09 |
| T038    | [[T038-support-ticket-direkt-schliessen-ohne-bounce-loop]] | App / Support / Ticketsystem / E-Mail | 2026-07-04 |
| T035    | [[T035-checkout-session-expired-trial-reset]]    | App / Lizenzen / Stripe / Checkout | 2026-07-04 |
| T036    | [[T036-dashboard-erste-schritte-popup-zustand-speichern]] | App / Frontend / Backend / UX | 2026-07-01 |
| T028    | [[T028-newsletter-nurture-brevo-umsetzung]]      | Marketing / Newsletter / Brevo   | 2026-06-28 |
| T020-16 | [[T020-16-Lizenz-und-Billing-Support-Aktionen]]     | App / Verwaltungsportal / Stripe | 2026-06-23 |
| T032    | [[T032-einheitlicher-versand-und-abrechnungsdokumente]] | App / Billing / Stripe / E-Mail | 2026-06-24 |
| T031    | [[T031-webapp-produktivrelease-und-prod-stack-konsolidierung]] | App / Release / Infrastruktur | 2026-06-24 |
| T029    | [[T029-webapp-audit-rollout-externe-konfiguration]] | App / Security / CI / Deployment | 2026-06-23 |
| T033    | [[T033-brevo-dev-prod-listentrennung]]           | App / Newsletter / Brevo / Infrastruktur | 2026-06-24 |
| T030    | [[T030-automatisierte-stripe-gutschriften-bei-refunds]] | App / Verwaltungsportal / Stripe / Rechnungslegung | 2026-06-22 |
| T027    | [[T027-landingpage-marketing-schaerfung]]           | Landingpage / Marketing          | 2026-06-13    |
| T024    | [[T024-account-einrichtung-onboarding-hinweise]]    | App / UX / Onboarding / Notifications | 2026-06-07 |
| T025    | [[T025-upload-retention-und-avatar-loeschung]]      | App / Support / Dateien / Datenschutz | 2026-06-04 |
| T023    | [[T023-app-ux-workflow-und-aktivitaetsverlauf]]     | App / UX / Workflows / Notifications | 2026-06-01  |
| T022    | [[T022-notifications-system]]                       | App / Backend / Frontend / Sidebar | 2026-05-30  |
| T020-12 | [[T020-12-Konzept Verwaltungsportal]]               | App / Verwaltungsportal          | 2026-05-30    |
| T020    | [[T020-allgemeine Todos]]                           | App                              | 2026-05-30    |
| T017    | [[T017-testzeitraum-fuer-lizenzen]]                 | App / Lizenzen / Stripe / Legal  | 2026-05-30    |
| T019    | [[T019-newsletter-gutschein-brevo-webhook-rollout]] | App / Newsletter / Brevo / Stripe | 2026-05-30    |
| T016    | [[T016-bericht-tab-pdf-export-ueberarbeiten]]       | App / WBR / Bericht              | 2026-05-30    |
| T020-11 | [[T020-11-Resuemee-Tab]]                            | App / WBR / PDF                  | 2026-05-01    |
| T020-10 | [[T020-10-Export-Tab Umbau]]                        | App / WBR / Export               | 2026-05-01    |
| T020-09 | [[T020-09-Berechnungen-Karte Hero]]                 | App / Projektdetail              | 2026-05-01    |
| T020-08 | [[T020-08-Lizenz hinzufuegen Redesign]]             | App / Lizenzen / Stripe          | 2026-05-01    |
| T020-07 | [[T020-07-Testlizenz kuendigen]]                    | App / Lizenzen / Stripe          | 2026-05-01    |
| T020-06 | [[T020-06-Stripe Adress-Sync]]                      | App / Stripe / Billing           | 2026-05-01    |
| T020-05 | [[T020-05-3 Karten Polish]]                         | App / Projektdetail              | 2026-05-01    |
| T020-04 | [[T020-04-Char-Counter Projekt bearbeiten]]         | App / Projektdetail              | 2026-05-01    |
| T020-03 | [[T020-03-Projekttabelle Layout]]                   | App / Projekte                   | 2026-05-01    |
| T020-02 | [[T020-02-Spilling-Portal neuer Tab]]               | App / Lizenzen / Billing-Portal  | 2026-05-01    |
| T020-01 | [[T020-01-Economics-Label entfernen]]               | App / WBR / Projektdetail        | 2026-05-01    |
| T018    | [[T018-landingpage-kaufintent-in-app-checkout]]     | Landingpage / App / Lizenzen     | 2026-04-30    |
| T015    | [[T015-toast-guideline-und-notify-helper]]          | App / Designsystem / Frontend    | 2026-04-27    |
| T014    | [[T014-kuendigungs-email-fuer-lizenzen]]            | App / Lizenzen / E-Mail          | 2026-05-25    |
| T013    | [[T013-lizenzsystem-rollout-abschliessen]]          | App / Infrastruktur / Marketing  | 2026-05-25    |
| T012    | [[T012-projektstatus-einfuehren]]                   | App                              | 2026-04-27    |
| T002    | [[T002-repo-docs-vault-pruefung]]                   | Vault                            | 2026-04-27    |
| T001    | [[T001-normdex-complete-guide-loeschen]]            | Vault                            | 2026-04-27    |


## Nächste freie ID

`T052`
