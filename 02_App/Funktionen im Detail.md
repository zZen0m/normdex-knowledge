# Funktionen im Detail

## Projektverwaltung

Projekte sind die organisatorische Einheit, unter der Berechnungen und Metadaten zusammengefasst werden.

| Feld | Beschreibung |
|---|---|
| Projektnummer | Automatisch generiert, eindeutig innerhalb der Organisation |
| Name | Freitext-Bezeichnung des Projekts |
| Beschreibung | Optionale längere Beschreibung |
| Berechnungstyp | Derzeit: `economics` (Wirtschaftlichkeit) |
| **Standort-Adresse** | Straße, PLZ, Ort, Land |
| **Auftraggeber** | Name, Straße, PLZ, Ort, Land |
| **Verfasser** | Unternehmen, Kontakt, Straße, PLZ, Ort, Land |
| Formular-Daten | JSON-Feld für modulare Eingabedaten |

**Projektliste:** Zeigt alle Projekte der Organisation, filterbar und sortierbar.

**Dashboard-Statistiken:** Gesamt-Projekte, aktiv (letzte 14 Tage), neu (letzte 30 Tage).

---

## Team & Organisation

Normdex ist **mandantenfähig**: Jede Organisation ist ein isolierter Bereich.

**Rollen:**

| Rolle | Berechtigungen |
|---|---|
| **Owner** | Voller Zugriff: Mitglieder verwalten, Abonnement, Einstellungen |
| **Member** | Zugriff auf Projekte & Berechnungen der Organisation |

**Einladungssystem:**
1. Owner gibt E-Mail-Adresse eines neuen Mitglieds ein
2. System sendet Einladungs-E-Mail mit Token-Link
3. Empfänger öffnet Link → wird der Organisation als Member hinzugefügt
4. Bestehende Einladungen können widerrufen werden

---

## Lizenz- & Abonnementverwaltung

Normdex verwendet ein **Concurrent-User-Seat-Modell**.

| Produkt-Key | Beschreibung |
|---|---|
| `economics_basic` | Economics Basic – Wirtschaftlichkeitsberechnung nach ÖNORM M 7140 |

**Lizenz-Lebenszyklus:** `Trial (14 Tage) → Active → Canceled / Expired`

**Concurrent-User-Enforcement:**
- Heartbeat-Mechanismus: Frontend sendet regelmäßig Keep-Alive-Requests
- Nach **30 Minuten Inaktivität** wird der Seat automatisch freigegeben

**Subscription-Flow (Stripe):**
1. Nutzer klickt „Abonnieren" → Checkout-Session wird erstellt
2. Stripe Checkout-Seite → Zahlung
3. Lizenzstatus wird auf `active` gesetzt
4. Stripe Customer Portal für Änderungen / Kündigung

---

## Support-Ticket-System

### Ticket-ID-Format

```
NDX-YYYY-######   Beispiel: NDX-2026-000042
```

### Status-Workflow

```
new → triaged → in_progress → waiting_on_customer → resolved → closed
                                                         ↑
                             Sonderstatus: spam, duplicate
```

| Status | Bedeutung |
|---|---|
| `new` | Eingegangen, noch nicht bearbeitet |
| `triaged` | Eingestuft, Priorität gesetzt |
| `in_progress` | Agent arbeitet aktiv daran |
| `waiting_on_customer` | Warte auf Rückmeldung vom Kunden |
| `resolved` | Problem behoben, Kunden informiert |
| `closed` | Endgültig abgeschlossen |
| `spam` | Spam, ignoriert |
| `duplicate` | Duplikat eines anderen Tickets |

**Auto-Close:** Tickets im Status `resolved` mit 7+ Tagen Inaktivität → automatisch `closed`.

**Prioritäten:** P1 (Kritisch) / P2 (Hoch) / P3 (Mittel) / P4 (Niedrig)

**Ticket-Quellen:** Webapp (`/support`) / Landingpage-Kontaktformular / E-Mail (`support@normdex.at`)

### Webhook-Integration (n8n)

Bei jedem neuen Ticket wird ein Webhook ausgelöst:

```json
{
  "event_type": "ticket.created",
  "ticket": {
    "ticket_id": "NDX-2026-000042",
    "subject": "...",
    "priority": "medium",
    "source": "webapp",
    "requester_email": "kunde@beispiel.at"
  }
}
```

**Sicherheits-Header:** HMAC-SHA256 Signatur (`X-Normdex-Signature`), SHA-256 Payload-Hash, Zeitstempel, Event-Typ.

**Retry-Logik:** Exponentielles Backoff (2, 4, 8, 16 Minuten), max. 5 Versuche.

---

## Benutzerkonto & Profil

**Passwort-Richtlinie:** Min. 8 Zeichen, 1 Großbuchstabe, 1 Kleinbuchstabe, 1 Ziffer, 1 Sonderzeichen (`!§$%&=?-_+`)

**Profil-Felder:** Anzeigename, Vorname/Nachname, Anrede, Unternehmen, Geburtsdatum, Zeitzone, Sprache/Locale, Avatar

**Sicherheits-Funktionen:**
- Passwort ändern, E-Mail-Adresse ändern (mit Bestätigungs-Token)
- Account löschen (Self-Service mit E-Mail-Bestätigung)
- Daten-Export (DSGVO-konform)

---

## Dashboard & Übersicht

Das Dashboard (`/app`) ist die Startseite nach dem Login:
- Organisations-Statistiken (Lizenzauslastung, Mitgliederanzahl, Projektzahlen)
- Aktivitäts-Feed (Audit-Log-Auszug)
- Team-Snippet (aktive Mitglieder mit Rollen)
- Onboarding-Checkliste für neue Nutzer

---

## Admin-Panel

Nur für Nutzer mit dem Flag `is_admin = true`.

**Benutzerverwaltung (`/admin/users`):** Profil einsehen/bearbeiten, E-Mail-Verifizierung setzen, Account aktivieren/deaktivieren, Admin-Flag vergeben.

**Support-Admin (`/admin/support`):** Tickets filtern, Status ändern, Priorität setzen, Antwort senden, interne Notizen, Statistiken.

---

## Verwandte Dokumente

- [[Wirtschaftlichkeitsrechner]]
- [[App-Routen]]
- [[Datenmodell]]
- [[E-Mail-System]]
