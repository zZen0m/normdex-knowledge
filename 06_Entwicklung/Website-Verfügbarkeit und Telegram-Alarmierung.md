# Website-Verfügbarkeit und Telegram-Alarmierung

Stand: 28.10.2026. Umsetzung in n8n auf dem Normdex-VPS, ohne externen Anbieter.

Workflow: **Normdex – Website-Verfügbarkeit**, ID `normdexWebsiteMonitor`.
Quellstand: `normdex-backup/monitoring/n8n/website-monitoring.json`.

## Prüfungen

Jede Minute normale HTTPS-GET-Anfragen an:

| Ziel | URL | Erfolg |
|---|---|---|
| Landingpage | https://normdex.at/ | HTTP 200 und Normdex im HTML |
| App | https://app.normdex.at/ | HTTP 200 und Normdex im HTML |
| API | https://api.normdex.at/health | HTTP 200 und JSON status=ok |

TLS-Zertifikate werden geprüft, Timeout pro Anfrage 12 Sekunden.
Drei aufeinanderfolgende Fehler eines Ziels lösen einen Telegram-Alarm aus.
Während desselben Ausfalls kein Alarm pro Minute. Nach erfolgreicher Prüfung
folgt eine Entwarnung. Fehlgeschlagene Telegram-Zustellungen werden erneut versucht.
Der Workflow nutzt das vorhandene Credential „Telegram Normdex Tickets“ und den
bestehenden Zielchat. Status und Fehlerzähler werden im Workflow gespeichert.

## Keine künstlichen Analytics-Besuche

Die Checks nutzen keinen Browser, führen kein JavaScript aus und laden keine
Tracking-Skripte oder eingebetteten Ressourcen. Die vorhandenen Einbindungen von
Google Analytics und Plausible erzeugen daher keine Pageviews, Sitzungen oder
Conversions durch diese Prüfungen. GA4 wird in der Landingpage erst nach
Browser-Einwilligung geladen; Plausible läuft als Browser-Skript im HTML.

User-Agent: `Normdex-UptimeMonitor/1.0`. Zusätzlich `DNT: 1` und
`Cache-Control: no-cache`; die fehlende JavaScript-Ausführung ist ausschlaggebend.
Technische Zugriffslogs und HTTP-Zähler enthalten diese Anfragen weiterhin.
Bei Auswertungen aus Serverlogs anhand des User-Agent ausschließen.
Bei späterer Einführung serverseitigen Trackings ist diese Annahme neu zu prüfen.

## Grenzen

Der Nutzer hat die Überwachung auf demselben VPS ausdrücklich bevorzugt.
Bei Ausfall von VPS, n8n oder dessen Internetzugang kann sie nicht alarmieren.
Lokale HTTPS-Prüfungen beweisen keine weltweite Erreichbarkeit. Die HTML-Prüfung
führt keinen Login oder Berechnungsablauf aus. Der derzeitige API-Healthcheck
prüft nicht die Datenbankverbindung.

## Betrieb und Tests

Der zusätzliche Workflow **TEST – Normdex Website-Alarm und Entwarnung (manuell)**
(`normdexWebsiteAlertTest`) bleibt inaktiv und dient nur dem manuellen Alarmtest.
Er simuliert einen Fehler und anschließende Erholung und sendet klar als TEST
markierte Nachrichten. Produktive Websites werden für Tests nicht abgeschaltet.

Bei einem Update bestehenden Workflow-Status erhalten; kein routinemäßiger Reset
von staticData. n8n-CLI-Publishing braucht einen kontrollierten Neustart, damit der
Zeitplan in der laufenden Instanz registriert wird. Vorher laufende Ausführungen
prüfen und bestehende Workflows unverändert erhalten.

Erfolgreiche normale Ausführungen speichern keine vollständigen Antwortkörper;
Fehlerausführungen bleiben zur Diagnose erhalten.
