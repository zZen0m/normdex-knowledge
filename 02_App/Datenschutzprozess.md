# Datenschutzprozess

Stand: 2026-06-04

Dieses Dokument beschreibt den aktuellen operativen Umgang mit Auskunfts-, Datenkopie- und Kontolöschanfragen in der Normdex-App.

## Externe Dienste in der Datenschutzerklärung

Die öffentliche Datenschutzerklärung (`https://normdex.at/datenschutz`, Quelle: [normdex-landingpage/src/pages/Datenschutz.tsx](../../../01_repos/normdex-landingpage/src/pages/Datenschutz.tsx)) deklariert folgende Datenverarbeitungen:

| Dienst | Anbieter | Rolle | Standort |
|---|---|---|---|
| Hosting | IONOS (1&1 IONOS SE) | Auftragsverarbeiter Art. 28 | DE (Berlin) |
| Transaktions-E-Mail | Brevo (Sendinblue SAS) | Auftragsverarbeiter Art. 28 | EU (FR) |
| Web-Analyse | Google Analytics (Google Ireland Ltd.) | Auftragsverarbeiter, DPF-zertifiziert | EU/US |
| Bot-Schutz | Google reCAPTCHA v2 (Google Ireland Ltd.) | Auftragsverarbeiter, DPF-zertifiziert | EU/US |
| Zahlung | Stripe Payments Europe Ltd. | Eigener Verantwortlicher | EU (IE) |
| UID-Prüfung | VIES (EU-Kommission, GD TAXUD) | Kein Auftragsverarbeiter (offizieller EU-Dienst) | EU (BE) |

Bei neuen externen Diensten (z. B. CRM, Tracking-Tools, KI-APIs) ist die Datenschutzerklärung zu erweitern, **bevor** der Dienst produktiv geht.

### VIES-spezifisch

- Übermittelt werden ausschließlich `countryCode` + `vatNumber` (siehe `apps/api/app/services/vat_validation.py`).
- Kein AVV erforderlich (offizieller EU-Dienst).
- Rechtsgrundlage: Art. 6 Abs. 1 lit. c (UStG-Pflicht) und lit. f (berechtigtes Interesse).

## Grundsatz

Normdex bietet aktuell keinen Self-Service-Datenexport in der App an.

Der frühere Export-Button wurde aus der UI entfernt und die API-Endpunkte für den Self-Service-Export sind deaktiviert:

```text
POST /users/me/export/request
GET  /users/me/export/download
```

Beide Endpunkte antworten mit `410 Gone` und verweisen auf den Supportprozess. Grund: Ein unvollständiger automatischer Export wäre riskanter als ein bewusst manueller Prozess.

## Datenkopie / Auskunft

Wenn eine Nutzerin oder ein Nutzer eine Kopie personenbezogener Daten anfordert:

1. Anfrage über Support entgegennehmen, z. B. `office@normdex.at`.
2. Identität prüfen, bevor personenbezogene Daten herausgegeben werden.
3. Betroffene Datenbereiche prüfen:
   - Nutzerprofil und Kontaktdaten
   - Organisation und Mitgliedschaften
   - Projekte und Berechnungen
   - Lizenzen, Lizenzbestellungen und Lizenzereignisse
   - Supporttickets und Supportnachrichten
   - Newsletter-/Gutscheinstatus, falls vorhanden
   - relevante Audit-/Sicherheitsereignisse, soweit herausgabefähig
4. Daten in einem gängigen elektronischen Format bereitstellen, z. B. ZIP mit JSON/CSV/PDF, abhängig vom konkreten Umfang.
5. Daten Dritter, interne Notizen, Geschäftsgeheimnisse und sicherheitsrelevante Details vor Herausgabe prüfen und ggf. schwärzen.
6. Bearbeitung intern dokumentieren: Datum, anfragende Person, geprüfte Identität, bereitgestellte Datenbereiche, Bearbeiter.

## Kontolöschung

Die App-Löschung ist bewusst keine harte physische Löschung des `users`-Datensatzes.

Technischer Zielzustand:

- Login wird deaktiviert.
- Sessions und Tokens werden ungültig.
- Personenbezogene Profilfelder werden entfernt oder anonymisiert.
- Projekte und Berechnungen des Kontos werden gelöscht.
- Mitgliedschaften werden entfernt.
- Aktive Lizenznutzungen werden entfernt.
- Lizenzzuweisungen und Lizenzereignisse werden vom Nutzer entkoppelt.
- Supporttickets und Supportnachrichten werden vom Nutzer entkoppelt und personenbezogene Kontaktdaten werden anonymisiert.
- Newsletter-Gutscheinclaims werden anonymisiert.
- Bestell-, Lizenz-, Audit- und Abrechnungskontext kann erhalten bleiben, sofern er für rechtliche, abrechnungsbezogene, Sicherheits- oder Nachweiszwecke benötigt wird.

Der anonymisierte Nutzer bleibt als technischer Platzhalter bestehen, damit bestehende Fremdschlüssel, Lizenzbestellungen und Auditkontexte konsistent bleiben.

### Technische Umsetzung der Anonymisierung

Umgesetzt in `_anonymize_user_account` (`apps/api/app/routers/users.py`), ausgelöst über `POST /users/me/delete/confirm` nach E-Mail-Bestätigung. Der dreistufige Ablauf (Passwort → Dialog-Bestätigung → E-Mail-Link mit 24h-Token) ist vollständig implementiert. Der Lösch-Token kann nicht erneut verwendet werden, da im selben Commit **alle** Token des Kontos gelöscht werden.

### Abrechnungs-Snapshot bei früherer bezahlter Subscription

Buchhaltungsrelevante Daten liegen primär an der **Organisation** (Rechnungsadresse, USt-ID, Kundennummer, `stripe_customer_id`) und in **Stripe** selbst — beide werden bei der Kontolöschung nicht angetastet. `LicenseOrder` (Bestellungen, Beträge, Stripe-Checkout-ID) bleibt erhalten und über die anonymisierte Platzhalter-Zeile referenziert.

Damit der buchhalterische Bezug **Person ↔ Bestellung** auch nach dem Nullen der Profilfelder nachvollziehbar bleibt, sichert `_snapshot_billing_data` vor der Anonymisierung einen unveränderlichen Snapshot:

- Betroffen sind ausschließlich Bestellungen des Nutzers mit Status `completed` (= bezahlte, buchhaltungsrelevante Belege).
- Gesichert werden Vor-/Nachname, Anzeigename, Firma, E-Mail, Kontotyp und Zeitstempel in `license_orders.meta.billing_snapshot` (`reason: "account_deletion"`).
- Der Vorgang ist idempotent: ein bestehender Snapshot wird nicht überschrieben.
- Hat der Nutzer keine bezahlte Bestellung getätigt, wird kein Snapshot erzeugt (Org-/Stripe-Daten genügen für die Buchhaltung).

Grundsatz: Personenbezug bleibt nur dort erhalten, wo er buchhalterisch erforderlich ist; alles Übrige wird anonymisiert (Datenminimierung gem. Art. 5 Abs. 1 lit. c DSGVO).

### Audit-Trail

Beim Start der Löschung wird `account_delete_requested`, nach erfolgreichem Abschluss `account_deleted` als Audit-Event protokolliert (`apps/api/app/audit.py`).

## Owner-Schutz bei Löschung

Ein Konto kann nicht gelöscht werden, wenn es der letzte Owner einer Organisation ist und weitere Mitglieder in dieser Organisation verbleiben.

Vor der Löschung muss in diesem Fall zuerst ein anderer Owner bestimmt werden.

Erweitertes Lösch-Gate:

- Team-Administrator:innen (`owner`/`admin`) können ihr Konto nicht löschen, solange weitere Teammitglieder in der Organisation vorhanden sind. Vorher müssen die Adminrechte an ein anderes Teammitglied übergeben oder entzogen werden.
- Team-Administrator:innen können ihr Konto nicht löschen, solange in der Organisation offene Lizenzen bestehen.
- Als offene Lizenzen gelten `pending`, `trial`, `active`, `scheduled_end` und `payment_failed`. `ended` blockiert die Kontolöschung nicht.

## Upload-Aufbewahrung und Datei-Hygiene

Hochgeladene Dateien werden kontrolliert aufbewahrt und gelöscht, damit kein unnötiger personenbezogener Datenbestand wächst (umgesetzt in [[T025-upload-retention-und-avatar-loeschung]], Stand 2026-06-04).

### Support-Ticket-Anhänge

- Anhänge geschlossener Tickets (Status `closed`) werden **365 Tage nach `closed_at`** automatisch bereinigt.
- Der geplante Job `cleanup_support_attachments` läuft täglich um 03:30 (`apps/api/app/services/scheduler.py`, Kernlogik in `_cleanup_due_attachments`). Frist als Konstante `SUPPORT_ATTACHMENT_RETENTION_DAYS = 365`.
- Nur die **physische Datei** unter `uploads/attachments/` wird entfernt; der DB-Eintrag bleibt erhalten und wird über `support_ticket_attachments.deleted_at` als gelöscht markiert. So bleibt die Ticket-Historie nachvollziehbar.
- Die Support-UI zeigt für solche Anhänge den Hinweis „Anhang aus Aufbewahrungsgründen gelöscht" statt eines Download-Links.
- Der Lauf ist idempotent; fehlende Dateien führen nicht zum Abbruch.
- Nur lokale App-Dateien sind betroffen. Verbleiben E-Mail-Anhänge zusätzlich in Microsoft 365, ist deren Aufbewahrung separat über Microsoft-/Mailbox-Policies zu regeln.
- Backups enthalten gelöschte Dateien noch bis zum Ablauf der Backup-Retention (`LOCAL_KEEP`, `REMOTE_KEEP`).

### Avatare (Profilbilder)

- Nutzer können ihr Profilbild explizit entfernen: `DELETE /users/me/avatar` löscht die Datei aus `uploads/avatars/`, setzt `avatar_url = null` und schreibt das Audit-Event `avatar_removed`. Datei-Löschung und Feld-Reset sind gekoppelt — `avatar_url` lässt sich nicht mehr über das Profil-Update (`PUT /users/me/profile`) ohne Datei-Cleanup setzen.
- Beim Ersetzen eines Avatars, bei Selbst-Kontolöschung und bei Admin-Löschung (`DELETE /admin/users/{id}`) wird die alte Avatar-Datei jeweils mit entfernt.
- Firmenlogos werden bereits beim Ersetzen und beim expliziten Löschen vom Server entfernt.

## UI-Kommunikation

Die UI darf nicht behaupten, dass alle verbundenen Daten unwiderruflich gelöscht werden.

Zulässige Kernaussage:

```text
Der Zugang wird deaktiviert. Personenbezogene Kontodaten werden gelöscht oder anonymisiert. Daten, die aus rechtlichen, abrechnungsbezogenen oder sicherheitsrelevanten Gründen aufbewahrt werden müssen, bleiben gemäß Datenschutzerklärung erhalten.
```

## Offene Punkte

- Juristische Prüfung der finalen Datenschutzerklärung.
- Admin-Unterstützung für Datenkopien und DSGVO-Wipe als späterer Verwaltungsportal-Ausbau.
- Dateninventar regelmäßig nach neuen Tabellen/Funktionen aktualisieren.
