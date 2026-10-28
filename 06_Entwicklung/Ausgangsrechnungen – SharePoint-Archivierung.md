# Ausgangsrechnungen – SharePoint-Archivierung

Stand: 28.10.2026. Quellcode und Workflow-Exporte: `normdex-backup/invoice-archive/`.

## Zweck und Ziel

Finalisierte Stripe-Lizenzrechnungen (auch 0-Euro-Rechnungen), Gutschriften und
Stornobelege werden als PDF in SharePoint abgelegt. Es werden keine Rechnungen,
Zahlungen oder Lizenzen verändert und keine Rechnungs-E-Mails ausgelöst.

Ziel: **Permatec e.U. → Freigegebene Dokumente → 02_Finanzen Buchhaltung →
01_Rechnungen Ausgang → Jahr → 01_Rechnungen / 02_Gutschriften / 03_Stornos**.
Das ist eine andere Dokumentbibliothek als die Normdex-VPS-Backups.
Dateinamen bleiben `JJMMTT_RE_A_Belegnummer.pdf` bzw. `GS` und `ST`.

## Ablauf

1. Der bestehende produktive Stripe-Webhook nimmt `invoice.finalized`,
   `invoice.voided` und `credit_note.created` entgegen.
2. Der Unterworkflow lädt die aktuellen Belegdaten aus Stripe und bestimmt Pfad/Name.
3. Vorhandene Dateien werden heruntergeladen und anhand PDF-Signatur und Größe
   geprüft; keine automatische Überschreibung bereits vorhandener Dokumente.
4. Fehlende Ordner werden segmentweise angelegt und als echte Ordner bestätigt.
   Nur echte Existenzkonflikte werden akzeptiert; Anmelde- und andere Fehler brechen ab.
5. Stripe-PDF herunterladen und prüfen; Uploadantwort auf Datei-ID, Name und Größe
   prüfen. Danach SharePoint-Datei herunterladen und bytegenau mit der Quelle vergleichen.

Workflow-IDs:

| Aufgabe | ID |
|---|---|
| Bestehender produktiver Stripe-Webhook | `6naLmNJ-38bQGl2xsPn-O` |
| Einzelbeleg archivieren/verifizieren | `normdexArchiveDocument` |
| Fehler an Telegram | `normdexArchiveErrors` |
| Vollständigkeitsabgleich | `normdexArchiveReconcile` |

## Vollständigkeit und Alarmierung

Jeden Tag um **05:00 Europe/Vienna** werden sämtliche Stripe-Rechnungen und
Gutschriften mit vollständiger Seitennavigation abgefragt und einzeln abgeglichen.
Fehlende PDFs werden nachgetragen. Auch verpasste Webhooks werden damit aufgefangen.
Ein Fehler bei einem Beleg wird erfasst; andere Belege werden weiter geprüft.
Der Gesamtlauf gilt erst als erfolgreich, wenn alle erwarteten Dokumente bestätigt sind.
Nur dann wird `staticData.global.lastSuccess` mit Zeitpunkt und Anzahl aktualisiert.

Fehlgeschlagene Webhook-/Abgleichläufe melden sich über den vorhandenen Bot
„Telegram Normdex Tickets“. Keine Tokens, Rohfehler oder signierten PDF-Links in
Telegram-Nachrichten. Netzwerk-, Download- und Uploadanfragen haben begrenzte Retries.
Andere sichtbare Fehler können nach Behebung erneut gestartet werden; der tägliche
Abgleich versucht fehlende Belege wieder, ohne vorhandene Dateien zu duplizieren.

Zusätzlich prüft ein **stündlicher Host-Cron unabhängig von n8n**, ob die vier
Workflows aktiv sind und höchstens 30 Stunden seit dem letzten vollständigen
Abgleich vergangen sind. Auch eine nicht erreichbare n8n-Datenbank wird gemeldet.
Da manuelle CLI-Läufe ihren n8n-Status nicht dauerhaft speichern, überbrückt ein
einmaliger, unabhängig verifizierter Startnachweis
`state/archive-initial-verified.json` den ersten geplanten Lauf. Auch dieser
läuft nach 30 Stunden ab und wird nie automatisch aufgefrischt.
Die Meldung wird direkt per Telegram gesendet; gleiche Alarme höchstens täglich,
Entwarnung bei Erholung. Log: `normdex-backup/state/archive-watchdog.log`.
Dieser Schutz teilt weiterhin den VPS; ein kompletter Host-/Internetausfall kann
von dort nicht gemeldet werden.

## Nachweis und Altbestand

Beim Audit am 28.10. waren zehn Rechnungen ab 24.06. korrekt archiviert. Es fehlten
zwölf Ablagen aus Mai/Anfang Juni, vor Einrichtung des alten Workflows am 07.06.
Diese wurden nachgetragen. Abschließender unabhängiger Bestand: **22 passende PDFs,
keine fehlenden Belege** – 19 Rechnungen, 2 Gutschriften und 1 zusätzliche Stornoablage.

Einzelupload mit bytegenauem Rücklesen, Wiederholung ohne Duplikat und vollständiger
Abgleich wurden getestet. Ein zusätzlicher lesender Test mit zwei Stripe-Objekten
pro Seite bestätigte die Seitennavigation über mehrere Seiten. Telegram-Fehlerkanal
wurde mit einem gekennzeichneten Testalarm geprüft.

## Betrieb und Updates

Der ursprüngliche Webhook und sein Signaturschlüssel wurden erhalten. Runtime-
Exporte enthalten diesen Schlüssel in staticData und gehören niemals ins Git.
Bei Updates den aktuellen Laufzeitstatus geschützt sichern und vor dem Import
wieder in eine Runtime-Kopie einsetzen. Diagnoseworkflows bleiben inaktiv.
Nach CLI-Veröffentlichung n8n kontrolliert neu starten, damit Zeitpläne registriert
werden. Vorher andere laufende Ausführungen prüfen; bestehende Workflows erhalten.
Die n8n-Datenbank, Credentials, Workflows und Zeitpläne liegen persistent. Docker
und Benutzer-Cron starten nach einem VPS-Neustart automatisch.
