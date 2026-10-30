# Eingangsrechnungen: Diagnose und Reparaturvorschlag

Historischer Diagnosebefund vom 28.10.2026, vor der Reparatur.
Die Reparatur wurde inzwischen veröffentlicht; siehe [[Eingangsrechnungen – Robuster Betrieb]].
Zum Diagnosezeitpunkt: Produktiver Ablauf nicht verändert; nur inaktive Diagnoseworkflows
und lesende Verbindungstests. Keine echten E-Mails/Anhänge an neue KI-Anbieter gesendet.

## Gesicherte Befunde

- Workflow: `Eingangs-Rechnungen Ablegen`, ID `bEpmh46HpQ4vOlAGKoVC8`, aktiviert.
- Gespeicherter Entwurf: Version `ef2aea8c-401a-494b-8880-af06abf5285b`, Groq,
  Modell `openai/gpt-oss-120b`.
- Veröffentlicht: Version `d27e43f7-975f-4c28-8e13-dabbd944d537`, Google Gemini.
  Der Gemini-Knoten v1 setzt kein Modell explizit und verwendet in der installierten
  n8n-Version den Default `models/gemini-2.5-flash`.
- Beide Provider-Schlüssel funktionieren für lesende Modelllisten-Abfragen (HTTP 200).
  Die genannten Modelle sind verfügbar. Das bestätigt Authentifizierung/Angebot,
  nicht automatisch Inferenzquota oder erfolgreichen Dokumentdurchlauf bei Gemini.
- Nutzer bestätigt Groq-Free-Tarif. n8n-Test mit erfundenem Kurztext, Groq-Modell,
  maximal 512 Ausgabetokens und vorhandenem JSON-Parser erfolgreich. Keine echten
  Dokumente verwendet; keine Ablagen oder Ticket-/Mail-Benachrichtigungen ausgelöst.
- Outlook-Metadatenabfrage mit dem vorhandenen n8n-Credential erfolgreich.
- Historische Ausführungen/Fehler dieses Workflows sind aktuell nicht mehr vorhanden.
  Die ursprüngliche KI-Fehlermeldung kann daher nicht sicher rekonstruiert werden.

## Weitere Ablaufprobleme

Die Anhangschleife kehrt nur nach erfolgreichem Upload über `Replace Me` zurück.
Nicht-PDFs, Inline-Dateien, Nicht-Rechnungen und der Pfad nach geringer Confidence
führen nicht zur nächsten Iteration. Ein ignorierter erster Anhang kann daher die
Prüfung weiterer Anhänge beenden. Der Abruf ist zusätzlich auf zehn Anhänge begrenzt.
Der PDF-Schritt extrahiert Text; eine OCR-Stufe für gescannte PDFs ist nicht vorhanden.
Der Prompt enthält den Text mehrfach und einen Verweis auf ein möglicherweise
fehlendes Feld. Die reine Prompt-Auswertung wurde getestet und verursachte dabei
keinen Fehler; dies wird nicht als nachgewiesene Verbindungsursache dargestellt.

## Empfehlung

Groq Free beibehalten: der echte n8n-KI-/JSON-Test funktioniert bereits. Nach Reparatur
und synthetischen Positiv-/Negativtests ausdrücklich die korrekte Version veröffentlichen.
Kein kostenpflichtiger Fallback und kein Tarifwechsel. Free bleibt kontingentiert;
zu lange PDFs und Rate-Limits müssen sichtbar behandelt werden.

Für reine Dokumentklassifikation eine einfache LLM Chain mit festem JSON-Schema
statt eines unnötig komplexen Agenten erwägen. Der vorhandene Agent hat im Test
funktioniert; die Vereinfachung ist eine Designempfehlung, kein bewiesener Fehlerfix.
Alle ignorierten/abgearbeiteten Anhänge zur Schleife zurückführen; vollständige
Seitennavigation verwenden. Leeren Text/OCR-Bedarf und unklare Ergebnisse als
prüfbedürftig behandeln, statt sie stillschweigend zu ignorieren. Pflichtfelder
und PDF-Upload validieren; Duplikate vermeiden. Im normalen Betrieb still,
Telegram nur bei Fehlern oder notwendigen manuellen Entscheidungen.

Eine lokale KI mit Ollama wäre ohne externe Tokenabrechnung möglich. Der VPS hatte
beim Check 7,7 GiB RAM, davon 3,8 GiB verfügbar, bei vielen produktiven Diensten.
Kleines Modell nur mit RAM-/CPU-Grenzen und separatem Last-/Qualitätstest erwägen;
keine Leistung oder zuverlässige Erkennungsquote ohne Benchmark zusagen.

## Quellen

- https://console.groq.com/docs/rate-limits
- https://console.groq.com/docs/billing-faqs
- https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.chainllm/
- https://docs.ollama.com/faq

## Nachtrag 29.10.2026: Shared-Mailbox-Kontext

Das Credential spricht nativ das freigegebene Postfach office@normdex.at an.
Der ursprüngliche direkte HTTP-Metadatentest verwendete /me und bestätigte daher
nur den Zugriff auf das angemeldete Postfach. Der Tagescheck ist inzwischen auf
/users/office@normdex.at umgestellt; tatsächliche Office-Verarbeitung und
Rechnungsordner wurden im Zuge der Verschiebe-Erweiterung separat geprüft.
