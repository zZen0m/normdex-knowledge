# Eingangsrechnungen: Betrieb

Stand 28.10.2026. Produktiver Einstieg `bEpmh46HpQ4vOlAGKoVC8`, Outlook office@normdex.at.
Groq-Credential bleibt im vom Betreiber bestätigten Free-Tarif; kein bezahlter Fallback.
Modell `openai/gpt-oss-120b`, striktes JSON-Schema, begrenzte Antwortlänge.

## Verarbeitung

Outlook-Abfrage wie bisher jede Minute, maximal zehn Anhänge je Nachricht.
Nicht-PDFs/Inline-Dateien und sicher erkannte Nicht-Rechnungen werden ignoriert;
alle Schleifenpfade kehren zur nächsten Datei zurück. PDF maximal 10 MiB,
extrahierter Text maximal 18.000 Zeichen. Kein OCR: leere/gescannte/zu lange
oder unsicher klassifizierte PDFs benötigen manuelle Prüfung und lösen Telegram aus.
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
Groq-Modellliste und SharePoint-Ordner. Das ist eine Zugangsprüfung, kein täglicher
KI-/Upload-Ende-zu-Ende-Test. Vorhandener stündlicher Host-Wächter prüft zusätzlich
Aktivierung aller sechs Workflows und einen erfolgreichen Befund innerhalb30h.
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
