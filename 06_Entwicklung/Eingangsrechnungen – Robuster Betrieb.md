# Eingangsrechnungen: Betrieb

Stand 28.10.2026. Produktiver Einstieg `bEpmh46HpQ4vOlAGKoVC8`, Outlook office@normdex.at.
Groq-Credential bleibt im vom Betreiber bestätigten Free-Tarif; kein bezahlter Fallback.
Modell `openai/gpt-oss-120b`, striktes JSON-Schema, begrenzte Antwortlänge.

## Verarbeitung

Outlook-Abfrage wie bisher jede Minute, maximal zehn Anhänge je Nachricht.
Nicht-PDFs/Inline-Dateien und sicher erkannte Nicht-Rechnungen werden ignoriert;
alle Schleifenpfade kehren zur nächsten Datei zurück. PDF maximal 10 MiB,
extrahierter Text maximal 18.000 Zeichen. Bei fehlendem Text wird lokales OCR als begrenzter Fallback ausgeführt (maximal acht Seiten,
120 Sekunden, 512 MiB und 0,75 CPU). Unlesbare/zu lange oder unsicher klassifizierte PDFs
benötigen weiterhin manuelle Prüfung und lösen Telegram aus. Das Original-PDF bleibt unverändert.
Erkennungsqualität bleibt modellabhängig; keine Garantie für jede Rechnung.

SharePoint: `02_Finanzen Buchhaltung/02_Rechnungen Eingang/<Jahr>`.
Jahresordner werden bei Bedarf angelegt. SHA256-Register vermeidet wiederholte Ablage.
Vorhandene Dateien werden byteweise verglichen; abweichende Kollisionen werden
nicht überschrieben. Neue Uploads werden heruntergeladen und byteweise geprüft.

Native n8n-Datentabellen: `normdex_incoming_mail_queue` (Status/Versuche/Frist/Zusammenfassung),
`normdex_incoming_documents` (SHA256/Pfad/Größe/Zeit). Dauerhaft in n8n-Datenbank;
Bestandteil bestehender Datenbanksicherung. Keine vollständigen Mailtexte im Queue-Status.
Erledigte Status-/Hash-Datensätze dienen weiterhin der Duplikaterkennung.

HTTP-Aufrufe maximal zwei Versuche. Groq 429: einmal 65 Sekunden warten und erneut
versuchen. Technisch fehlgeschlagene Mails werden alle drei Stunden um Minute17
auf Fälligkeit geprüft; Backoff 3/6/12/24 Stunden, maximal fünf Gesamtversuche.
Danach Status `manual`: Ursache beheben und gezielt erneut verarbeiten.
Erfolgreich abgelegte Anhänge werden beim nächsten Versuch anhand des Hashs übersprungen.
Ein laufender Versuch erhält eine 30-Minuten-Lease; abgebrochene Versuche sind danach
wieder aufnehmbar. Unklare Klassifikation wird nicht automatisch endlos wiederholt.

Telegram nur bei technischen Fehlern oder notwendiger manueller Prüfung.
Keine regelmäßigen Testnachrichten oder Testmails. Error-Workflow meldet Workflow,
Schritt und Ausführungs-ID, keine rohen KI-Antworten oder Zugangsdaten.

## Überwachung und Neustart

`normdexIncomingHealth` prüft täglich 06:00 Europe/Vienna Outlook-Inbox,
Groq-Modellliste, SharePoint-Ordner und einen echten lokalen OCR-Selbsttest. Das ist eine Zugangsprüfung, kein täglicher
KI-/Upload-Ende-zu-Ende-Test. Vorhandener stündlicher Host-Wächter prüft zusätzlich
Aktivierung aller sieben Workflows und einen erfolgreichen Befund innerhalb30h.
Ein einmaliger, zeitlich ablaufender Erstnachweis überbrückt den ersten Tageslauf.
Cron/Trigger und Datentabellen liegen persistent in n8n; Neustart wurde geprüft.
Der gemeinsame VPS bleibt ein gemeinsamer Ausfallpunkt für Telegram-Alarmierung.

## Veröffentlichung und Nachweise

`build.py` erzeugt Quellen aus credentialreferenziertem Template und JS-Dateien.
Niemals unbesehen importieren: beim Einstieg den aktuellen Outlook-Poll-Lesestand
`staticData` bewahren. Erst Unterworkflows, dann Einstieg veröffentlichen; geänderte
Servertrigger durch kontrollierten n8n-Neustart laden. Setup nur einmal ausführen;
keine Testwebhooks im Produktivbetrieb aktiv lassen.
Private Rollback-Snapshots liegen auf dem VPS unter
`/home/deploy/workspaces/normdex-incoming-invoices/release-rollback` (0700/0600),
inklusive vorher veröffentlichtem Verlauf und Outlook-Lesestand. Nicht ins Git aufnehmen.

Tests: `node incoming-invoices/test_logic.cjs`; Host-Wächter-Unittests.
Serverintegration Ausführung477/479: vier synthetische Anhänge (Bild ignoriert,
Rechnung mit echter Groq-Inferenz erkannt, Nicht-Rechnung ignoriert, Scan manuell).
Ausführung484: zwei Fehlversuche einer künstlichen Mail-ID dauerhaft gespeichert.
Ausführung487: künstliches PDF in isoliertem SharePoint-Testordner hochgeladen,
byteweise rückgelesen, zweiter Versuch dedupliziert; eigene Testdatei/Statuszeilen gelöscht.
Outlook/Groq/SharePoint-Zugangsprüfung separat erfolgreich. Keine echten Mails verändert.
Alle temporären Test-/Setup-Workflows anschließend deaktiviert.

## OCR-Ergänzung 28.10.2026

Lokaler Tesseract-/Poppler-Dienst ohne öffentliche Ports oder zusätzliche OCR-API-Kosten
aktiviert; täglicher Selbsttest und stündliche Containerüberwachung ergänzt. Details und
Tests: `normdex-backup/incoming-invoices/ocr/README.md`. Synthetischer Scan mit Groq
erfolgreich, Text-PDF umgeht OCR, leeres PDF wird zur Prüfung gemeldet. Acht Testseiten
3,62 Sekunden, gemessener Containerhöchststand 80,9 MiB. Neustart von OCR und n8n geprüft.

## Rechnungs-Mails nach Ablage verschieben (29.10.2026)

Nach mindestens einer bestätigten Rechnungsablage und ohne fehlerhafte/ungeklärte
Anhänge wird die gesamte Nachricht nach **Posteingang/Rechnungen** im freigegebenen
Postfach `office@normdex.at` verschoben. Sicher ignorierte Anhänge verhindern das nicht.
Reine Nicht-Rechnungen bleiben im Posteingang; ungeklärte/fehlgeschlagene Mails ebenso.
Der vorhandene Rechnungsordner wird über seine feste ID verwendet.

Neuer interner Worker `normdexIncomingMove`: vor dem Verschieben wird der Auftrag
mit ursprünglicher Outlook-ID und Internet-Message-ID in der nativen Queue gespeichert.
Der Worker sucht zuerst im Zielordner nach der exakten Internet-Message-ID. Ein Treffer
bestätigt einen bereits ausgeführten Auftrag, mehr als ein Treffer wird als unklar
abgebrochen. Ein POST wird nicht blind wiederholt. Bei verlorener Antwort/Neustart
wird der gespeicherte Auftrag fortgesetzt, ohne Groq oder SharePoint erneut aufzurufen.
Die reguläre Outlook-ID ändert sich beim Verschieben; deshalb ist allein diese ID
kein ausreichender Wiederaufnahmenachweis. Microsofts alternative ID-Übersetzung
war im delegierten Shared-Mailbox-Kontext mit vorhandenem Credential nicht erlaubt;
der veröffentlichte Ablauf benötigt diese zusätzliche Berechtigung nicht.

Erst nach Zielbestätigung wird Mailstatus `complete`; Verschiebefehler nutzen die
bestehende begrenzte Wiederholungswarteschlange und Telegram-Fehleralarmierung.
Täglicher Healthcheck prüft zusätzlich den Zielordner. Der stündliche Hostwächter
prüft auch den neuen Worker auf Aktivierung.

Wichtig: Das Outlook-Credential verwendet `useShared=true` für `office@normdex.at`.
Native Outlook-Knoten berücksichtigen dies; direkte HTTP-Aufrufe tun das nicht
von selbst. Alle neuen Mail-HTTP-Schritte und der tägliche Inbox-Test adressieren
daher ausdrücklich `/users/office@normdex.at/` statt `/me/`.

Nachweise: `node incoming-invoices/test_move.cjs` und bestehende Logiktests.
Serverintegration mit künstlichem, ungesendetem Office-Entwurf, simuliert bestätigter
Anhangsablage: Move-Auftrag persistent gespeichert, Nachricht verschoben, verlorene
Antwort simuliert, Wiederaufnahme ohne erneute Anhangsverarbeitung erfolgreich.
Eigener Testentwurf/Queue-Datensatz entfernt; keine E-Mails versendet oder echten
Bestandsmails verschoben. Frühere künstliche Testdaten im angemeldeten anderen
Postfach und der dort irrtümlich angelegte leere Testordner ebenfalls entfernt.
Temporäre Testwebhooks deaktiviert. Neue Regeln wirken auf künftige Verarbeitung;
keine automatische Nachbearbeitung bereits abgeschlossener historischer Mails.
Rollback-Snapshots: `/home/deploy/workspaces/normdex-mail-move/rollback` (0700/0600).

## Kategorie „abgelegt“ nach Verschieben (29.10.2026)

Der Verschiebe-Worker setzt anschließend `abgelegt` in der Kategorienliste der
Ziel-Mail. Bestehende Kategorien werden übernommen; fehlende Kategorien werden
ergänzt, nicht ersetzt. Die Liste wird zurückgelesen und auf neue sowie zuvor
vorhandene Einträge geprüft. Erst dann liefert der Worker `moved:true,categorized:true`
und die Queue wird vollständig abgeschlossen. Eine fehlgeschlagene Kennzeichnung
bleibt als Verschiebeauftrag wiederholbar; bereits verschobene Mails werden im
Zielordner über Internet-Message-ID gefunden und anschließend nur noch markiert.
Keine nachträgliche automatische Änderung bereits abgeschlossener historischer Mails.

Die Kategorie wird erfolgreich als Name an Nachrichten gesetzt. Ihre gewünschte
Farbe Grün ist noch **nicht** per API eingerichtet: Die Kategorienliste im Office-
Postfach ist lesbar, `POST /users/office@normdex.at/outlook/masterCategories` für
`{displayName:"abgelegt",color:"preset4"}` liefert jedoch403. Bestehende Definitionen
wurden nicht verändert. Microsoft verlangt für das Erstellen/Ändern von Kategorien
`MailboxSettings.ReadWrite`; die derzeitige Verbindung erlaubt diesen Schritt nicht.
Einmalig in Outlook für das freigegebene Office-Postfach eine Kategorie namens
**abgelegt** mit grüner Farbe anlegen. Die automatische Namenszuweisung läuft bereits;
eine erneute Microsoft-Autorisierung ist für die manuelle Farbzuordnung nicht nötig.
Eine bestehende Kategorie „in SharePoint abgelegt“ mit anderer Farbe wurde bewusst
nicht verändert. Outlook-Clients bestimmen die konkrete Darstellung des Farbcodes.

Nachweise: `node incoming-invoices/test_category.cjs`; künstlicher ungesendeter
Office-Entwurf wurde verschoben und markiert, vorhandene „Rote Kategorie“ blieb
bestehen, zweiter identischer Aufruf erfolgreich, Testentwurf anschließend gelöscht.
Tests versendeten keine Nachrichten und lösten keine regulären Telegram-Meldungen aus.
Neustart, aktive Veröffentlichungsstände und gesunder Hostwächter nach Deployment bestätigt.
Rollback für diesen Nachtrag: `/home/deploy/workspaces/normdex-mail-category/rollback`.

Microsoft-Referenzen:
- https://learn.microsoft.com/en-us/graph/api/resources/outlookcategory?view=graph-rest-1.0
- https://learn.microsoft.com/en-us/graph/api/outlookcategory-update?view=graph-rest-1.0

## Historischer Nachlauf ab 01.08.2026 – abgeschlossen am 29.10.2026

Auf ausdrücklichen Nutzerauftrag empfangene Office-Mails ab 01.08.2026, 00:00
Europe/Vienna (31.07.2026 22:00 UTC), bis zum festen Startzeitpunkt paginiert geprüft.
13 Mails: elf mit 14 PDFs, zwei reine Preisänderungsmitteilungen. Neun unterschiedliche
Rechnungen bestätigt: vier neu ergänzt, fünf bereits vorhanden. Zwei Microsoft-
Rechnungen lagen jeweils in zwei Mails vor. Drei Anthropic-Zahlungsbestätigungen
wurden anhand Rechnungsnummer, Datum und Betrag der jeweiligen Rechnung zugeordnet;
sie bleiben in der Mail erhalten und erzeugen keine zusätzliche Rechnungsablage.

Vorher vorhandene 36 PDFs byteweise gelesen und per SHA256 im Dokumentregister
registriert. Abweichende PDF-Metadaten und Aussteller-Schreibweisen erforderten
zusätzlich vollständige normalisierte Textvergleiche: World4You-Variante inhaltlich
identisch; drei zunächst neu erzeugte Varianten nach bestätigter Textgleichheit
wieder entfernt, einschließlich genau ihrer neuen Registerzeilen. Ausschließlich
eigene neue Kopien bereinigt; keine vorher vorhandene Archivdatei verändert.
Endbestand: 40 PDFs, davon neun Rechnungen aus dem geprüften Zeitraum.

Die vier gesondert geprüften Mailstatus wurden mit Nachweis im Queue-Zusammenfassungsfeld
abgeschlossen, anschließend durch den normalen Verschiebe-/Kategorie-Worker geführt.
Alle elf Rechnungs-Mails tatsächlich im bestehenden Ordner Posteingang/Rechnungen
und mit abgelegt markiert; keine technischen Fehler oder ungeklärten Fälle verbleiben.
Regulärer Outlook-Poll-Lesestand nicht geändert. Einmaligen Nachlauf
`normdexIncomingBackfillAugust` nach Abschluss deaktiviert und Testwebhook entfernt.
Kein wiederkehrender historischer Nachlauf eingerichtet; Produktionslogik unverändert.

Geschützter Prüfbericht und Rohbelege auf dem VPS:
`/home/deploy/workspaces/normdex-incoming-backfill/Pruefbericht.md` (Verzeichnis0700).
Keine Mail-IDs, Nachrichtentexte oder echten PDF-Inhalte ins Repository übernommen.
Im geprüften Postfach wurde keine im Oktober eingegangene Microsoft-Rechnung gefunden;
der Nachlauf bestätigt vorhandene Nachrichten und ersetzt keinen Lieferantenportal-Abgleich.
Die drei Receipt-Sonderfälle und normalisierten PDF-Textgleichheiten waren bewusste
Einzelfallprüfungen dieses Nachlaufs; die Produktionslogik bleibt bei abweichenden
Datei-Bytes weiterhin konservativ und meldet Konflikte zur Prüfung.
