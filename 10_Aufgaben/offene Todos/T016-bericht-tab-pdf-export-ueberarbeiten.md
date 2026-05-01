# T016 – Bericht-Tab PDF-Export überarbeiten

## Status

in Arbeit

## Bereich

App / Wirtschaftlichkeitsberechnung / Bericht

## Ziel

Der Tab „Bericht“ in der Wirtschaftlichkeitsberechnung soll nur noch die aktiven Berichtsfunktionen enthalten:

- PDF-Export über die aktuelle serverseitige Python-/ReportLab-Erzeugung
- direkte Konfiguration der Berichtsinhalte

Die veraltete Berichtsvorschau wird nicht mehr benötigt.

## Kontext

Die bisherige Berichtsvorschau im Frontend ist veraltet. Der aktuelle Bericht wird über den Backend-Endpunkt `/reports/economics/pdf` als PDF erzeugt und heruntergeladen.

Vor der Überarbeitung musste zuerst im Ergebnistab eine Berechnung angestoßen werden, damit der PDF-Export Daten hatte. Künftig soll der Export auch ohne vorherigen Wechsel in den Ergebnistab funktionieren, indem bei Bedarf automatisch eine Berechnung im Hintergrund gestartet wird.

Zusätzlich war der Header der Wirtschaftlichkeitsberechnung nicht bündig mit dem Seiteninhalt.

## Akzeptanzkriterien

- [ ] Header der Wirtschaftlichkeitsberechnung ist bündig mit Tabs und Seiteninhalt.
- [ ] Bericht-Tab zeigt keine veraltete Berichtsvorschau mehr.
- [ ] Bericht-Tab enthält einen direkt sichtbaren PDF-Export.
- [ ] Bericht-Tab enthält die Konfiguration der Berichtsinhalte direkt auf der Seite.
- [ ] PDF-Export funktioniert auch ohne vorherige Berechnung im Ergebnistab.
- [ ] Wenn keine Ergebnisse im Frontend-State vorhanden sind, wird für den Export automatisch eine Berechnung angestoßen.
- [ ] Während der PDF-Erstellung erscheint eine Toast-Benachrichtigung.
- [ ] Erfolgreicher Export und Fehlerfall zeigen passende Toasts.
- [ ] Bestehender Backend-Endpunkt `/reports/economics/pdf` bleibt unverändert.

## Notizen / Fortschritt

- 2026-04-27: Umsetzung gestartet.
- Header-Ausrichtung in `apps/frontend/src/components/layout/Header.tsx` korrigiert.
- PDF-Export aus der alten Berichtsvorschau gelöst und direkt in `apps/frontend/src/pages/EconomicsForm.tsx` integriert.
- Bericht-Tab auf Export und Berichtsinhalte-Konfiguration reduziert.
- Berichtsinhalte um Inhaltsverzeichnis, Kostenaufschlüsselung, Sensitivitätsanalyse, Abbildungs-/Tabellenverzeichnis und Glossar ergänzt.
- Neue Berichtsschalter in Frontend, API-Schema und PDF-Generator verdrahtet.
- Berichtskonfiguration als einklappbare Hauptpunkte mit Unterpunkten strukturiert.
- Sensitivitätsanalyse im Export berücksichtigt nur auswählbare/durchgeführte Szenarien; Projektdaten zeigen bei abgewählter Sensitivität „Nicht durchgeführt“.
