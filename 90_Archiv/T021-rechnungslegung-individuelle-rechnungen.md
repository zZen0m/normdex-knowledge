# T021 - Rechnungslegung: eigene Rechnungsnummern und individuelles Rechnungsdesign

**Status:** zurückgestellt (geplante Erweiterung für später)
**Bereich:** App / Lizenzen / Stripe / Rechnungslegung / Rechtliches
**Erstellt:** 2026-05-02
**Zurückgestellt:** 2026-05-30
**Abgeschlossen:** -

> Hinweis: Dieses Todo ist zurückgestellt und nicht aktuell in Arbeit. Es beschreibt eine geplante Erweiterung der Rechnungslegung, die zu einem späteren Zeitpunkt umgesetzt wird. Konzept, Akzeptanzkriterien und technische Anforderungen bleiben als Grundlage für die spätere Umsetzung erhalten.

## Ziel

Normdex soll eigene, rechtlich pruefbare Rechnungen erzeugen koennen, weil Stripe bei Rechnungsnummern und Rechnungsdesign nur eingeschraenkt individualisierbar ist.

Stripe bleibt weiterhin zustaendig fuer:

- Zahlungsabwicklung
- Subscription-Verwaltung
- Zahlungsstatus
- Hosted Payment Page / Zahlungslink
- Webhooks als technischer Ausloeser

Normdex soll perspektivisch die kundenseitige Rechnung fuehren:

- eigene Normdex-Rechnungsnummer
- eigenes PDF-Layout
- eigener Rechnungsdatensatz mit Stripe-Referenzen
- rechtlich nachvollziehbarer Rechnungsnummernkreis
- unveraenderbarer Snapshot der Rechnungsdaten zum Ausstellungszeitpunkt

## Kontext

Aktuell uebernimmt Stripe die Rechnungslegung. Dadurch ist Normdex bei folgenden Punkten eingeschraenkt:

- Die Rechnungsnummer folgt Stripe-Schemata.
- Das Layout kann nur begrenzt ueber Branding, Templates, Custom Fields, Memo/Footer und Invoice Settings angepasst werden.
- Die Tabellen- und Summenlogik bleibt weitgehend Stripe-vorgegeben.
- Eine komplett eigene PDF-Rechnung mit eigener Nummernlogik ist innerhalb von Stripe nicht frei steuerbar.

Fachlich ist besonders wichtig, dass die Rechnungsnummer nicht direkt offenlegt, wie viele Rechnungen Normdex insgesamt ausgestellt hat. Eine einfache globale Sequenz wie `NDX-0001`, `NDX-0002`, `NDX-0003` ist daher nicht gewuenscht.

## Rechnungsaussteller und Kleinunternehmer-Regelung

Normdex ist nur die Marke bzw. der Produktname. Das leistende Unternehmen und damit der Rechnungsaussteller ist `Permatec e.U.`.

Auf der Rechnung muss daher bei Name und Anschrift des leistenden Unternehmens nicht `Normdex`, sondern `Permatec e.U.` mit der korrekten Unternehmensadresse stehen. Normdex kann als Marke im Logo, Produktnamen oder in der Leistungsbeschreibung sichtbar bleiben, darf aber nicht faelschlich als rechtlicher Rechnungsaussteller erscheinen.

Permatec e.U. ist aktuell Kleinunternehmer in Oesterreich, hat keine UID und ist umsatzsteuerbefreit.

Folgende Regeln muessen fuer Normdex-/Permatec-Rechnungen gelten:

- Es wird keine Umsatzsteuer ausgewiesen.
- Es wird kein USt.-Betrag angezeigt.
- Es wird keine UID von Permatec e.U. bzw. Normdex angezeigt, solange keine UID vorhanden ist.
- Jede Rechnung enthaelt sichtbar folgenden Hinweis:

> Hinweis: Umsatzsteuerbefreit – Kleinunternehmer gem. § 6 Abs. 1 Z 27 UStG

Wichtig: Die Rechnung darf nicht so formuliert sein, als wuerde Umsatzsteuer zusaetzlich berechnet. Spalten wie `Preis pro Einheit (zzgl. Steuern)` oder `Betrag (zzgl. Steuern)` sind fuer Normdex-Rechnungen ungeeignet und sollen neutral ersetzt werden.

## Rechtliche Pflichtangaben

Die finale Rechnung muss nach oesterreichischer Rechnungspraxis mindestens folgende Angaben enthalten:

- Name und Anschrift von `Permatec e.U.` als leistendem Unternehmen und rechtlichem Rechnungsaussteller.
- Name und Anschrift des Rechnungsempfaengers.
- Art und Umfang der Leistung, z. B. `OENORM M 7140 Basic - Hauptlizenz`.
- Leistungszeitraum, z. B. `26.04.2026-26.05.2026`.
- Ausstellungsdatum.
- Faelligkeitsdatum bzw. Zahlungsstatus.
- Eindeutige Rechnungsnummer.
- Entgelt ohne Umsatzsteuerausweis.
- Steuerbefreiungshinweis bei Kleinunternehmerregelung.

Quellen / Orientierung:

- WKO: [Erfordernisse einer Rechnung](https://www.wko.at/steuern/rechnung-richtig-ausstellen)
- Finanz.at: [Rechnungsnummer](https://www.finanz.at/business/rechnung/rechnungsnummer/)

Die WKO beschreibt fuer Rechnungen unter anderem eine fortlaufende Nummer mit einer oder mehreren Zahlenreihen, die zur Identifizierung einmalig vergeben wird. Fuer Kleinunternehmer ist laut WKO keine UID des Rechnungsausstellers notwendig, wenn fuer die Leistung kein Recht auf Vorsteuerabzug besteht.

## Rechnungsnummern-Konzept

Normdex braucht ein eigenes Rechnungsnummernschema, das zwei Ziele verbindet:

- rechtlich nachvollziehbar, eindeutig und fortlaufend innerhalb definierter Nummernkreise
- keine direkte Offenlegung der gesamten Rechnungsanzahl gegenueber Kunden

Beschlossenes fachliches Schema:

- Format: `NDXYY-DT-NNNNNN`
- `NDX` = Normdex
- `YY` = Rechnungsjahr, z. B. `26` fuer 2026
- `DT` = zweistellige Dokumentenart
- `NNNNNN` = 6-stellige fortlaufende Nummer innerhalb dieses Nummernkreises

Beispiel fuer 2026:

- `NDX26-RE-325001`

Die Dokumentennummern der Permatec e.U. / Normdex(TM) werden je Kalenderjahr und Dokumentenart in getrennten Nummernkreisen vergeben. Die Nummern sind innerhalb des jeweiligen Nummernkreises fortlaufend, eindeutig und werden nicht wiederverwendet.

Nummernkreise:

| Dokumentenart | Kuerzel | Nummernkreis 2026 | Startnummer |
|---|---:|---|---:|
| Rechnung | RE | `NDX26-RE-######` | `325001` |
| Gutschrift / Credit Note | CN | `NDX26-CN-######` | `125001` |
| Storno | ST | `NDX26-ST-######` | `225001` |
| Angebot | AN | `NDX26-AN-######` | `425001` |

Konkrete Beispiele:

- `NDX26-RE-325001` = erste Rechnung 2026
- `NDX26-RE-325002` = zweite Rechnung 2026
- `NDX26-CN-125001` = erste Gutschrift 2026
- `NDX26-ST-225001` = erstes Storno 2026
- `NDX26-AN-425001` = erstes Angebot 2026

Fuer 2027 wechselt der Jahrespraefix, die Dokumentenart bleibt gleich und der jeweilige Nummernkreis startet wieder bei seiner definierten Startnummer:

- `NDX27-RE-325001`
- `NDX27-CN-125001`
- `NDX27-ST-225001`
- `NDX27-AN-425001`

Wichtige Regeln:

- Keine frei gewuerfelten Zufallsnummern ohne nachvollziehbare interne Sequenz.
- Keine Wiederverwendung von Rechnungsnummern.
- Keine nachtraegliche Aenderung einer vergebenen Rechnungsnummer.
- Rechnungen verwenden `RE`, Gutschriften `CN`, Stornos `ST` und Angebote `AN`.
- Die Sequenz wird je Kalenderjahr und Dokumentenart getrennt gefuehrt.
- Abgebrochene oder nicht finalisierte Stripe-Invoices duerfen keine finale Normdex-Rechnungsnummer verbrauchen, sofern steuerlich nicht erforderlich.
- Stornos, Gutschriften und Angebote verwenden eigene Nummernkreise und duerfen den Rechnungsnummernkreis `RE` nicht beeinflussen.
- Finale Freigabe durch Steuerberater ist Pflicht, bevor das Schema produktiv verwendet wird.

## Technisches Zielbild

Es soll eine eigene Rechnungsentitaet in Normdex geben. Diese speichert alle rechnungsrelevanten Daten als Snapshot und verweist intern auf Stripe.

Vorgeschlagene Felder:

- `id`
- `organization_id`
- `normdex_invoice_number`
- `invoice_number_year`
- `invoice_number_year_prefix`, z. B. `26`
- `document_type`, z. B. `RE`, `CN`, `ST`, `AN`
- `invoice_number_scope`, z. B. `2026:RE`
- `invoice_number_sequence`
- `invoice_number_start_sequence`
- `stripe_invoice_id`
- `stripe_customer_id`
- `stripe_subscription_id`
- `stripe_checkout_session_id`
- `payment_status`
- `hosted_invoice_url`
- `stripe_invoice_pdf_url`
- `customer_name`
- `customer_email`
- `billing_address_snapshot`
- `issuer_name` mit Wert `Permatec e.U.`
- `issuer_address_snapshot`
- `issued_at`
- `due_at`
- `service_period_start`
- `service_period_end`
- `currency`
- `subtotal_gross`
- `tax_total`
- `total_gross`
- `small_business_notice`
- `pdf_path` oder `pdf_url`
- `meta`
- `created_at`
- `updated_at`

Positionen sollen separat oder strukturiert im Rechnungsdatensatz gespeichert werden:

- Beschreibung
- Menge
- Einzelpreis
- Betrag
- Leistungszeitraum je Position, falls abweichend
- Stripe-Line-Item-Referenz, falls vorhanden

## Daten- und Sync-Regeln

- Pro finalisierter Stripe-Invoice darf genau eine Normdex-Rechnung entstehen.
- Wiederholte Webhooks muessen idempotent sein und duerfen keine zweite Rechnungsnummer vergeben.
- Die Normdex-Rechnungsnummer wird erst vergeben, wenn klar ist, dass eine kundenseitige Rechnung entstehen soll.
- Rechnungsadresse, Leistung, Preise und Kleinunternehmer-Hinweis werden als Snapshot gespeichert.
- Alte Rechnungen bleiben unveraendert, auch wenn der Kunde spaeter seine Rechnungsadresse aendert.
- Die naechste Rechnung verwendet die dann aktuelle Rechnungsadresse.
- Stripe bleibt Quelle fuer Zahlungsstatus, Betrag, Subscription-Referenz und Zahlungslink.
- Normdex bleibt Quelle fuer kundenseitige Rechnungsnummer, PDF-Layout und gespeicherten Rechnungssnapshot.

## PDF-Design

Das Beispiel `C:\Users\Andreas\Downloads\Invoice-NDX-0001.pdf` dient als erste Layout-Referenz.

Beizubehalten:

- klare Ueberschrift `Rechnung`
- Normdex-Logo oben rechts
- kompakter Block mit Rechnungsnummer, Ausstellungsdatum und Faelligkeit
- Absender- und Empfaengerblock
- grosser faelliger Betrag
- tabellarische Leistungspositionen
- Summenblock
- Seitenfuss mit Seitenzahl

Anzupassen:

- Rechnungsnummer nicht als einfache globale Sequenz `NDX-0001` verwenden.
- Tabellenkopf nicht mit `zzgl. Steuern` beschriften.
- Geeignete Spalten:
  - `Beschreibung`
  - `Menge`
  - `Preis pro Einheit`
  - `Betrag`
- Summenblock ohne Umsatzsteuer-Zeile:
  - `Zwischensumme`
  - `Summe`
  - `Faelliger Betrag`
- Kleinunternehmer-Hinweis sichtbar im unteren Rechnungsbereich platzieren.
- Keine UID anzeigen, solange fuer Permatec e.U. keine UID vorhanden ist.
- Im Absenderblock `Permatec e.U.` als Rechnungsaussteller anzeigen; Normdex nur als Marke/Logo oder Leistungsbezeichnung verwenden.
- optionaler Link `Online bezahlen` entfernen
- den Wortlaut, dass die Rechnungsbetrag bereits dankend erhalten wurde z.B. durch `Rechnungsbetrag dankend erhalten` unten hinzufügen.

## UI / App-Anforderungen

Kunden sollen in der App ihre Normdex-Rechnungen sehen und herunterladen koennen.

Zu pruefen:

- Neuer Bereich `Rechnungen` in Lizenzverwaltung oder Kontoeinstellungen.
- Anzeige von Rechnungsnummer, Datum, Betrag, Status und Download-Link.
- Optional zusaetzlich Stripe-Zahlungslink anzeigen, wenn Rechnung noch offen ist.
- Stripe-PDF soll nicht als fuehrendes Rechnungsdokument erscheinen, wenn Normdex eigene Rechnungen produktiv fuehrt.

Admin-Anforderungen:

- Admin kann Rechnungen je Organisation einsehen.
- Admin sieht Stripe-Referenzen zur Nachvollziehbarkeit.
- Admin kann PDF erneut generieren, ohne Rechnungsnummer oder Snapshot zu veraendern.
- Admin kann Storno-/Gutschriftprozess anstossen, sobald fachlich definiert.

## Akzeptanzkriterien

- Es gibt eine dokumentierte Entscheidung, ob Stripe-Optimierung ausreicht oder Normdex eine eigene Rechnungsschicht umsetzt.
- Das gewaehlte Rechnungsnummernschema legt nicht direkt die Gesamtzahl aller Normdex-Rechnungen offen.
- Die Dokumentennummern folgen dem Format `NDXYY-DT-NNNNNN`.
- Die Dokumentennummern sind eindeutig und fortlaufend je Kalenderjahr und Dokumentenart.
- Die Startnummern je Dokumentenart sind umgesetzt: `RE=325001`, `CN=125001`, `ST=225001`, `AN=425001`.
- Pro finalisierter Stripe-Invoice entsteht hoechstens eine Normdex-Rechnung.
- Wiederholte Stripe-Webhooks erzeugen keine doppelte Rechnung.
- Rechnungsadresse wird als Snapshot gespeichert.
- Alte Rechnungen bleiben nach Adressaenderungen unveraendert.
- Neue Rechnungen nutzen die aktuelle Rechnungsadresse.
- Normdex-Rechnungen weisen keine Umsatzsteuer aus.
- Normdex-/Permatec-Rechnungen enthalten keine UID, solange keine UID vorhanden ist.
- Als leistendes Unternehmen steht `Permatec e.U.` mit korrekter Adresse auf der Rechnung; Normdex wird nicht als rechtlicher Rechnungsaussteller dargestellt.
- Jede Rechnung enthaelt exakt den Kleinunternehmer-Hinweis:
  - `Hinweis: Umsatzsteuerbefreit – Kleinunternehmer gem. § 6 Abs. 1 Z 27 UStG`
- Die PDF-Spalten verwenden keine Formulierung wie `zzgl. Steuern`.
- Steuerberaterpruefung ist vor Produktivsetzung dokumentiert und als Blocker markiert.

## Tests / Verifikation

Backend:

- Test: finalisierte Stripe-Invoice erzeugt genau eine Normdex-Rechnung.
- Test: wiederholter Webhook fuer dieselbe Stripe-Invoice bleibt idempotent.
- Test: Rechnungsnummer ist eindeutig und folgt `NDXYY-RE-NNNNNN`.
- Test: erste Rechnung 2026 erhaelt `NDX26-RE-325001`, zweite Rechnung 2026 `NDX26-RE-325002`.
- Test: erste Gutschrift, erstes Storno und erstes Angebot 2026 verwenden eigene Sequenzen `NDX26-CN-125001`, `NDX26-ST-225001`, `NDX26-AN-425001`.
- Test: 2027 startet je Dokumentenart wieder mit dem definierten Jahrespraefix und der definierten Startnummer, z. B. `NDX27-RE-325001`.
- Test: konkurrierende Webhook-Verarbeitung kann keine doppelte Sequenz vergeben.
- Test: Rechnungsadresse wird als Snapshot gespeichert.
- Test: spaetere Adressaenderung veraendert alte Rechnung nicht.
- Test: neue Rechnung nutzt aktualisierte Rechnungsadresse.
- Test: Kleinunternehmer-Hinweis ist im Rechnungsdatensatz vorhanden.
- Test: keine UID wird auf Rechnung ausgegeben, solange keine UID konfiguriert ist.
- Test: PDF zeigt `Permatec e.U.` als Rechnungsaussteller und verwendet Normdex nur als Marke/Logo oder Leistungsbezeichnung.

PDF:

- Test: PDF enthaelt Rechnungsnummer, Ausstellungsdatum, Faelligkeit, Empfaenger, Positionen, Leistungszeitraum, Betrag und Kleinunternehmer-Hinweis.
- Test: PDF enthaelt keine Umsatzsteuer-Zeile und keinen USt.-Betrag.
- Test: PDF enthaelt nicht `zzgl. Steuern`.
- Test: Positions- und Summenbetraege stimmen mit Stripe ueberein.
- Visuelle Pruefung gegen `Invoice-NDX-0001.pdf` als Layout-Referenz.

Stripe-Testmode:

- Subscription erzeugen.
- Stripe-Invoice finalisieren.
- Normdex-Rechnung erzeugen.
- Zahlung als `paid` synchronisieren.
- PDF herunterladen.
- Webhook erneut senden und pruefen, dass keine zweite Rechnung entsteht.

## Offene Pruefpunkte

- Steuerberater pruefen lassen, ob das Schema `NDXYY-DT-NNNNNN` mit getrennten Jahres-/Dokumentenart-Nummernkreisen und den definierten Startnummern zulaessig ist.
- Klaeren, ob Rueckerstattungen immer ueber `CN` als Gutschrift/Credit Note laufen oder zusaetzliche Dokumententypen brauchen.
- Klaeren, ob Stripe-PDFs weiterhin sichtbar bleiben oder nur intern als Stripe-Referenz dienen.
- Klaeren, ob elektronische Rechnungen per E-Mail versendet werden und wie Zustimmung, Archivierung und Nachweis der Unversehrtheit dokumentiert werden.
- Klaeren, ob bei B2B-Kunden ueber 10.000 EUR Sonderpflichten zur Empfaenger-UID relevant werden koennen.
- Klaeren, wie innergemeinschaftliche B2B-Leistungen und Reverse-Charge-Faelle behandelt werden, falls Normdex spaeter nicht mehr nur Kleinunternehmer-Inlandslogik abbildet.

## Notizen / Fortschritt

- 2026-05-02: Todo angelegt aus Anforderung zu individueller Rechnungslegung, eigener Rechnungsnummer und Kleinunternehmer-Hinweis.
- 2026-05-02: Beispielrechnung `Invoice-NDX-0001.pdf` als Layout-Referenz festgehalten.
- 2026-05-02: Kleinunternehmerstatus festgehalten: keine UID, keine Umsatzsteuer, Pflicht-Hinweis auf Steuerbefreiung.
- 2026-05-02: Ergaenzt, dass Normdex nur Marke/Produktname ist und `Permatec e.U.` als rechtlicher Rechnungsaussteller mit Adresse auf die Rechnung muss.
- 2026-05-02: Rechnungsnummernschema festgelegt: `NDXYY-DT-NNNNNN` mit getrennten Nummernkreisen je Kalenderjahr und Dokumentenart (`RE`, `CN`, `ST`, `AN`) sowie Startnummern `325001`, `125001`, `225001`, `425001`.
