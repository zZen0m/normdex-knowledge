# 2026-06-22 · Einheitlicher Versand und Abrechnungsdokumente

## Entscheidung

- Normdex versendet offizielle Rechnungen und Gutschriften selbst im Normdex-Branding.
- Stripe bleibt Ersteller und Quelle der PDF-Dokumente.
- Empfänger ist ausschließlich die explizite Rechnungs-E-Mail der Organisation. Ein leeres Feld deaktiviert den E-Mail-Versand für Abrechnungsdokumente.
- Neue Organisationen verwenden zunächst die Adresse des ersten Owners; spätere Käufe und neue Admins ändern sie nicht.
- Bestellbestätigungen bleiben eine getrennte Nachricht an die kaufende Person.
- Die Lizenzverwaltung zeigt eine gemeinsame vollständige Rechnung-/Credit-Note-Historie für Organisations-Admins.
- Das Stripe Customer Portal bleibt ergänzend für Zahlungsmethoden bestehen.

## Begründung

Das Stripe Customer Portal stellt Rechnungen bereit, aber keine eigenständige vollständige Credit-Note-Liste. Ein gemeinsamer Normdex-Dokumentbereich sorgt für einen konsistenten Belegzugang, einheitliches Branding und nachvollziehbaren Versandstatus.

## Versandregeln

- bezahlte Rechnung: Vormerkung über `invoice.paid`, Versand frühestens fünf Minuten später
- Gutschrift: Versand nach erfolgreicher Credit Note
- 0-Euro-Rechnung: sichtbar, aber keine E-Mail
- historische Dokumente: sichtbar, aber kein nachträglicher Versand
- PDF-Link statt Anhang
- fehlgeschlagener Versand: Retry mit Backoff
- unklarer Crash-Zustand: manuelle Prüfung statt möglicher Doppelzustellung
- Rechnungs-E-Mail: syntaktisch vollständige Adresse mit Domain und mindestens zweistelliger Endung
- Dokumentbereich: scrollbare Tabelle mit serverseitiger Seitennavigation und den Spalten Datum, Nummer, Art, Betrag und PDF

## Produktiv-Gate

Vor der Produktivfreigabe ist im Stripe Dashboard zu bestätigen, dass keine parallelen automatischen Rechnungs- oder Zahlungsbelegmails aktiviert sind.
