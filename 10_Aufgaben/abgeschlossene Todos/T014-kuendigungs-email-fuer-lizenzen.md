# T014 – Kündigungs-E-Mail für Lizenzen

## Status

erledigt

## Abgeschlossen

2026-05-25

## Bereich

App / Lizenzen / E-Mail

## Ziel

Wenn ein User eine Lizenz kündigt, soll automatisch eine E-Mail versendet werden.

## Kontext

Die E-Mail soll wertschätzend formuliert sein: Normdex bedauert die Kündigung, findet das Ausscheiden schade und würde sich freuen, den User bald wieder als Kunden begrüßen zu dürfen.

Der Inhalt soll abhängig vom Kündigungsfall variieren:

- Kündigung einer Zusatzlizenz
- Kündigung der einzigen und letzten Lizenz

## Akzeptanzkriterien

- Bei Lizenzkündigung wird eine passende E-Mail ausgelöst.
- Der Text unterscheidet zwischen Zusatzlizenz-Kündigung und Kündigung der einzigen/letzten Lizenz.
- Die E-Mail verwendet die einheitliche Lizenzbezeichnung `ÖNORM M 7140 Basic`.
- Versand und Inhalt sind testbar oder mindestens lokal nachvollziehbar dokumentiert.

## Notizen / Fortschritt

- Angelegt am 2026-04-27 aus App-Feedback zur Lizenzverwaltung.
- 2026-05-25: Implementiert. Lizenzkündigungen über `POST /licenses/{license_id}/cancel` senden eine Kündigungsbestätigung an den kündigenden User. Der Text unterscheidet zwischen Zusatzlizenz und organisationsweit letzter Lizenz, verwendet `ÖNORM M 7140 Basic`, enthält Laufzeitende und Link zur Lizenzverwaltung. Mailfehler werden als `EMAIL_ERROR` protokolliert und blockieren die Kündigung nicht.
- 2026-05-25: Kündigungsbestätigung optisch an Bestellbestätigung angeglichen: HTML-Mail mit Tabelle für Produkt, Lizenzart, Abrechnung, Menge, Einzelpreis und Summe, CTA „Lizenzen verwalten“ und Abrechnungsportal-Hinweis im Footer.
- 2026-05-25: Der damalige 10-Minuten-Flow `POST /licenses/{license_id}/undo-purchase` erhielt eine Bestätigungsmail.
- 2026-06-07: Der 10-Minuten-Undo-Flow wurde produktseitig vollständig entfernt. Käufe können nicht mehr sofort rückgängig gemacht werden; reguläre Kündigungen und deren Bestätigungsmails bleiben bestehen.
- 2026-05-25: Verifiziert mit `.\venv\Scripts\python -m pytest tests/test_license_cancel_reactivation.py` in `apps/api/` → 11 passed.
