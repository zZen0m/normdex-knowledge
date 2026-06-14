# T027 · Landingpage marketingseitig schärfen

**Phase:** Landingpage / Marketing
**Priorität:** P2 · Conversion / Positionierung
**Status:** erledigt
**Datum:** 2026-06-13
**Abgeschlossen:** 2026-06-13

## Ziel

Die Landingpage wird auf die Marketingstrategie 2026 ausgerichtet. Kern ist der Schwenk der Positionierung von "normkonform" hin zu "moderne, web-basierte Lösung", weil die Normkonformität gegen den etablierten Wettbewerb kein Alleinstellungsmerkmal ist. Zusätzlich werden die Vertrauenssignale gestärkt, der Newsletter auf den Lead-Magneten umgestellt und einige sachliche Fehler bereinigt.

Grundlage ist [[Marketingplan 2026 - Erste Kunden]]. Verbindliche Regel aus dem Plan: Im Frontend wird kein Mitbewerber namentlich genannt. Durchgängig Du-Form, keine Gedankenstriche, direkte Umlaute.

## Kontext

Der Funnel ist reiner Self-Service. Die Website verkauft allein, deshalb ist sie der wichtigste Hebel. Die wichtigsten Beobachtungen aus dem aktuellen Code:

- `src/components/Hero.tsx`: Die H1 (etwa Zeile 46 bis 52) führt mit "ÖNORM-konforme" und stellt die Normkonformität in den Vordergrund. Genau dieses Feld besetzt der Wettbewerb stärker.
- `src/components/Hero.tsx`: Rechts steht ein gecodetes Mock-Dashboard. Ein öffentlicher Beispielbericht zum Ansehen fehlt.
- `src/components/NewsletterStrip.tsx` und `src/pages/Newsletter.tsx`: Beide werben fast nur mit "10 % Rabatt". Der Rabatt zieht schwach. Der Lead-Magnet (kostenloser Praxisleitfaden) soll der Anreiz werden.
- Es gibt eine sachliche Vergleichssektion (Kategorie modern gegen klassisch) noch nicht.
- Im Frontend stehen mehrere Gedankenstriche, die gegen die Schreibregeln verstoßen.

## Akzeptanzkriterien

### P1, höchste Wirkung

- [x] Hero-Headline in `Hero.tsx` umstellen: nicht mit "ÖNORM-konforme" führen, sondern mit dem Vorteil "moderne, web-basierte Wirtschaftlichkeitsberechnung, im Browser, im Team, prüffähig". Normkonformität bleibt erhalten, aber als Beleg darunter, nicht als Hauptversprechen. Subline (Zeile 54 bis 58) entsprechend anpassen.
- [x] Berichtsvorschau auf der Seite zeigen, nicht zum Download. Einige echte Berichtsseiten als anklickbare Bildergalerie oder Lightbox einbinden, die im Browser bleibt. Kein separater Download-Button, der von der Seite wegführt. Das ist das stärkste Trust-Signal ohne persönliches Auftreten und zeigt die saubere Darstellung auch großer Beträge. Der vollständige Bericht bleibt dem Newsletter-Magneten vorbehalten, damit die Vorschau überzeugt und der volle Bericht weiter die E-Mail verdient.
- [x] Hero-Trust-Zeile (Zeile 95 bis 108) um den klarstellenden Punkt "Keine Abbuchung während der Testphase" ergänzen, damit die Kreditkarten-Hürde transparent und beruhigend kommuniziert ist.

### P2

- [x] Serverstandort ist geklärt: Die Server stehen in Deutschland, liegen damit in der EU und sind DSGVO-konform. Das überall konsistent so kommunizieren. In `TrustFactors.tsx` den Datenschutz-Faktor von "DE-Server" auf "Server in der EU, DSGVO-konform" umstellen. Ads-Liste und Newsletter-Strecke sind bereits entsprechend korrigiert. Norm und Zielgruppe bleiben Österreich, das ist davon unabhängig.
- [x] In `TrustFactors.tsx` zwei der vier Faktoren durch die Differenzierungs-Vorteile "Ohne Installation" und "Im Team" ersetzen, damit der modern-Vorteil auch hier sichtbar wird. Validierung und Datenschutz bleiben.
- [x] `NewsletterStrip.tsx` und `Newsletter.tsx` auf den Lead-Magneten umstellen: Der kostenlose Praxisleitfaden mit Checkliste und Beispielbericht wird der Hauptanreiz, die 10 % nur die zweite Zutat. Eyebrow, Überschrift und Text entsprechend anpassen, Download-Angebot auf der Newsletter-Seite ergänzen.
- [x] Neue, sachliche Vergleichssektion als eigene Komponente (zum Beispiel `ComparisonSection.tsx`) anlegen und in `Index.tsx` einbinden. Inhalt: klassische Desktop-Software gegenüber moderner Web-Lösung (Installation gegen Browser, Einzelplatz gegen Team, starrer Bericht gegen prüffähiges PDF). Ohne Firmennamen.

### P3

- [x] `OenormM7140.tsx` zu einer transparenten Methodik- und Validierungsseite vertiefen, mit einem klaren Abschnitt, wie Normdex nach Abschnitt 10 validiert. Nachvollziehbarkeit ersetzt die fehlende Ausschuss-Herkunft.
- [x] Gedankenstriche im sichtbaren Frontend durch Punkt oder Komma ersetzen. Bekannte Fundstellen: `CTA.tsx` (Zeile 38), `NewsletterStrip.tsx` (Zeile 26), `Newsletter.tsx` (Zeile 34 und 35), `OenormM7140.tsx` (Zeile 244, 272, 281, 348, 489), `About.tsx` (Zeile 54 und 66), `AGB.tsx` (Zeile 443).

## Notizen / Fortschritt

- 2026-06-13: Aufgabe aus der Marketing-Strategie-Session abgeleitet. Codeanalyse des Repos `normdex-landingpage` durchgeführt. Die früher dokumentierte falsche Aussage "Keine Kreditkarte erforderlich" ist im aktuellen Hero-Code bereits entfernt.
- 2026-06-13: Offene Klärung Serverstandort (DE oder AT) als Voraussetzung für konsistente Außenkommunikation markiert.
- 2026-06-13: Serverstandort geklärt, Deutschland und damit EU, DSGVO-konform. Ads-Liste und Newsletter-Strecke entsprechend korrigiert.
- 2026-06-13: Beispielbericht als Vorschau auf der Seite entschieden, nicht als Download. Der vollständige Bericht bleibt Newsletter-Magnet.
- 2026-06-13: Umsetzung P1 bis P3 im Repo `normdex-landingpage` abgeschlossen (Build, Lint und Dev-Start grün):
  - P1 Hero: Headline auf "Im Browser. Im Team. Prüffähig." umgestellt, Subline auf den modernen Nutzen, Validierung bleibt als Beleg. Mock-Dashboard durch echte Berichtsvorschau (Teaser, öffnet Lightbox) ersetzt. Trust-Zeile um "Keine Abbuchung während der Testphase" ergänzt. Sekundärer CTA "Beispielbericht ansehen".
  - P1 Berichtsvorschau: kuratierter Beispielbericht über neues Skript `apps/api/preview_report_demo.py` (Vergleich Wärmepumpe/Gas/Pellet, Wohnanlage 60 WE, eine Position > 1 Mio. €, klar als "Beispielprojekt:" gekennzeichnet, vollständiges Resümee mit Empfehlung) generiert. Sechs Seiten als PNG unter `public/report-preview/seite-1..6.png`. Neue Komponente `ReportPreview.tsx` mit Galerie und Lightbox, in `Index.tsx` eingebunden.
  - P2 TrustFactors: "Nachvollziehbar" und "Prüfbar" durch "Ohne Installation" und "Im Team" ersetzt; Datenschutz-Faktor auf "Server in der EU, DSGVO-konform".
  - P2 Newsletter: `NewsletterStrip.tsx`, `Newsletter.tsx` und `NewsletterForm.tsx` auf den Praxisleitfaden als Hauptanreiz umgestellt, 10 % als Bonus. Direkter PDF-Download auf der Seite nach Anmeldung (`/Normdex_Praxisleitfaden_OENORM_M7140.pdf`).
  - P2 Vergleichssektion: neue `ComparisonSection.tsx` (klassische Desktop-Software gegen moderne Web-Lösung, ohne Firmennamen) in `Index.tsx`.
  - P3 OenormM7140: neuer Abschnitt "So validiert Normdex nach Abschnitt 10" (transparente Validierungsschritte, Du-Form).
  - P3 Gedankenstriche im sichtbaren Prosa-Text bereinigt (CTA, About, AGB, Datenschutz, OenormM7140). Titel-Tags, Kommentare und Adress-Bis-Strich bewusst belassen.
- 2026-06-13: Überarbeitung nach Feedback:
  - Hero-Bild gewechselt: statt Berichtsseite jetzt gecodeter Mock der App-Ergebnisseite in neuer Komponente `HeroAppMock.tsx`. Zeigt den Tab Gesamtkosten mit gestapeltem Balkendiagramm (Barwert je System nach Kostengruppen, Farben wie App: Kapital blau / Verbrauch grün / Betrieb gelb), Wärmepumpe als günstigste Variante markiert, plus Sieger-Banner. Werte konsistent zum Bericht (WP 2,97 Mio. € < Pellet 3,85 Mio. € < Gas 7,13 Mio. €). Kommuniziert „moderne Web-App im Browser" statt statischem PDF.
  - Berichtsgalerie (`ReportPreview`) im Verlauf nach unten verschoben: jetzt nach `HowItWorks`, direkt vor der Preis-Sektion (Beweis nach Nutzenerklärung). Hero-Button „Beispielbericht ansehen" scrollt weiterhin dorthin.
  - Beispieldaten korrigiert: zuvor war rechnerisch die Pelletheizung am günstigsten, das Resümee empfahl aber die Wärmepumpe (Widerspruch). Daten angepasst (energiearme WP mit PV-Eigennutzung, höhere Energiepreissteigerung bei Gas/Pellet), sodass die Wärmepumpe nun eindeutig und nachvollziehbar gewinnt (Annuität WP 199.509 € < Pellet 258.820 € < Gas 479.281 €; Amortisation WP vs. Gas 3,68 Jahre). Resümee-Text und alle 6 Vorschau-Screenshots entsprechend neu generiert.
- 2026-06-13: Lead-Magnet-PDF erstellt und unter `public/Normdex_Praxisleitfaden_OENORM_M7140.pdf` abgelegt (Kopie in `04_Marketing`). Neu mit drei Systemen (Wärmepumpe, Pellet, Gas, Wohnanlage 60 WE) passend zur Berichtsgalerie, echtem Normdex-Logo, klickbaren Links zur Website und einer Annuitäten-Vergleichsgrafik. Damit sind alle Akzeptanzkriterien erfüllt.
- 2026-06-13: Todo abgeschlossen.
