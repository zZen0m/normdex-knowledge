# T032 · Einheitlicher Versand und Abrechnungsdokumente

**Phase:** App / Billing / Stripe / E-Mail  
**Priorität:** P1  
**Status:** in Arbeit  
**Datum:** 2026-06-22  
**Umgebung:** `normdex-webapp-dev`, Branch `dev-server`

## Ziel

Normdex versendet bezahlte Rechnungen und Credit Notes einheitlich über das eigene Branding an eine explizite Rechnungs-E-Mail. Organisations-Admins erhalten in der Lizenzverwaltung eine vollständige gemeinsame Stripe-Dokumenthistorie.

## Umgesetzt

- verpflichtende `Organization.billing_email`
- Backfill bestehender Organisationen mit der Adresse des ältesten Owners
- Änderung unter „Unternehmenseinstellungen“ durch `owner` und bestehende `admin`
- Änderungsinformation an alte und neue Adresse
- Synchronisation der Rechnungs-E-Mail zum Stripe Customer
- gebrandete Rechnungsmail frühestens fünf Minuten nach `invoice.paid`, PDF-Link ohne Anhang
- keine E-Mail für 0-Euro-Rechnungen
- Gutschriftversand an die zentrale Rechnungs-E-Mail
- persistenter und idempotenter Versandstatus in `billing_document_deliveries`
- Scheduler-Retry für fehlgeschlagene Rechnungsmails
- geschützter, paginierter Endpunkt `GET /subscriptions/billing-documents`
- gemeinsame Rechnung-/Gutschriftliste in „Abrechnungsdokumente & Zahlungsdaten“
- Stripe-Portal nur noch ergänzend über „Zahlungsmethode verwalten“
- Rechnungs-E-Mail wird in Frontend und Backend auf vollständige Form wie `rechnung@firma.at` geprüft
- leere Rechnungs-E-Mail deaktiviert Rechnungs- und Gutschriftmails; Dokumente bleiben online verfügbar
- scrollbare Dokumenttabelle mit Datum, Nummer, Art, Betrag, PDF und echter Seitennavigation

## Verifikation am 22.06.2026

- Dev-PostgreSQL vor Migration gesichert
- lokale SQLite-Datenbank vor Migration gesichert
- Migration Upgrade/Downgrade/Upgrade erfolgreich
- Backfill auf ältesten Owner für 7 lokale Organisationen verifiziert
- vollständige Backend-Suite nach optionaler Rechnungs-E-Mail und Tabellenumbau: 377 Tests bestanden
- TypeScript-Prüfung und Frontend-Docker-Build erfolgreich
- Dev-Migration auf `f4d5e6f7a8b9` angewendet
- Dev-API und Frontend neu gebaut und gesund gestartet
- realer lesender Stripe-Testmodus-Durchlauf: 15 Dokumente, Rechnungen und Credit Notes, PDF-Links vorhanden

## Noch offen

- manuelle fachliche/UI-Abnahme durch den Owner auf `https://dev.normdex.at` EDIT: 23.06.2026: Abnahme positiv erfolgt
- vor Produktivfreigabe Stripe-Dashboard prüfen: automatische Rechnungs- und Zahlungsbelegmails müssen deaktiviert sein EDIT: 23.06.2026: erledigt
- Produktivrollout ausschließlich über [[T031-webapp-produktivrelease-und-prod-stack-konsolidierung]]

## Referenzen

- [[E-Mail-System]]
- [[Datenmodell]]
- [[API-Endpunkte]]
- [[Integrationen & externe Dienste]]
- [[2026-06-22-einheitlicher-versand-und-abrechnungsdokumente]]
