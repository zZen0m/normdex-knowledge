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

Normdex verwendet ein **Pool-Modell mit Einzellizenzen**. Jede Lizenz ist ein eigener Datensatz in Normdex; die Abrechnung wird in Stripe je Organisation in maximal zwei Pools gebündelt:
- monatlicher Pool
- jährlicher Pool

| Produkt-Key | Beschreibung |
|---|---|
| `economics_basic` | Economics Basic – Wirtschaftlichkeitsberechnung nach ÖNORM M 7140 |

**Lizenzarten und Preise:**

| Pool | Hauptlizenz | Zusatzlizenz |
|---|---:|---:|
| Monatlich | 49 € / Monat | 29 € / Monat |
| Jährlich | 490 € / Jahr | 290 € / Jahr |

Die erste aktive oder noch laufende Lizenz in einem Pool ist die Hauptlizenz. Weitere Lizenzen im selben Pool sind Zusatzlizenzen. Monats- und Jahrespool werden getrennt bewertet.

**Lizenz-Lebenszyklus:** `pending → trial → active → scheduled_end → ended`, zusätzlich `payment_failed` bei Zahlungsproblemen. `trial` ist ein eigener Status für qualifizierte Einzel-Erstkäufe mit 14-tägigem kostenlosem Testzeitraum.

**14-Tage-Testvorteil / Erstbestellungsrabatt:**
- Qualifizierter Einzel-Erstkauf: Stripe Checkout erstellt eine Subscription mit 14 Tagen Trial; Normdex führt die Lizenz lokal als `trial`.
- Qualifizierter Mehrfach-Erstkauf: kein echter Trial, sondern einmaliger Erstbestellungsrabatt von 24,50 EUR über Stripe-Coupon `QHQESezY`.
- Der Trial-Benefit wird dauerhaft pro Organisation über `organizations.trial_used_at` gesperrt. Zusätzliche Schutzschichten prüfen gleiche `stripe_customer_id`, gleiche `vat_id`, Pending Orders und Legacy-Metadaten.
- Abgebrochene Trial-Checkouts geben den Lock nur frei, wenn Stripe keine Subscription erzeugt hat.
- Zusatzkauf während Trial wird als Trial-Konvertierung behandelt: Die bestehende Trial-Subscription wird nur nach erfolgreicher Stripe-Zahlung beendet, verbleibende Trial-Tage werden mit `24,50 EUR / 14 * Resttage` als Gutschrift angerechnet und im Order-/Lizenz-Meta dokumentiert.
- Die Checkout-Vorschau zeigt bei Trial-Konvertierung Resttage, Bruttobetrag der umgewandelten Trial-Lizenz und Gutschrift separat an.

**Concurrent-User-Enforcement:**
- Heartbeat-Mechanismus: Frontend sendet regelmäßig Keep-Alive-Requests
- Jede Lizenz erlaubt genau eine aktive Sitzung
- Nach **7 Minuten Inaktivität** wird die Sitzung automatisch freigegeben
- Wenn eine Lizenz belegt ist, versucht die App automatisch die nächste aktive oder nicht abgelaufene Trial-Lizenz

**Subscription-Flow (Stripe):**
1. Owner/Admin klickt in `/licenses` auf „Lizenz hinzufügen"
2. Normdex berechnet Vorschau, Hauptlizenz/Zusatzlizenz und Pool-Zuordnung
3. Bei einem neuen Pool wird eine Stripe-Checkout-Session erstellt
4. Bei bestehendem Pool wird die bestehende Stripe-Subscription direkt erweitert und anteilig verrechnet (`payment_behavior = error_if_incomplete`)
5. Neue Lizenzen werden erst nach erfolgreicher Stripe-Annahme und Zahlung aktiv; bei Zahlungsfehlern werden lokale Pending-Lizenzen verworfen und jüngste offene Subscription-Update-Rechnungen nach Möglichkeit voided
6. Trial-Konvertierungen während Zusatzkäufen laufen im gleichen atomaren Stripe-Update mit, damit die Testphase nicht endet, wenn die Zahlung fehlschlägt

**Rabattcodes:**
- Rabattcodes können im Normdex-Kaufdialog eingegeben werden
- Bei neuer Stripe-Subscription wird der Code als Stripe Promotion Code an die Checkout Session übergeben
- Bei direkter Erweiterung einer bestehenden Pool-Subscription wird der Code als Stripe Promotion Code an `Subscription.modify` übergeben
- Der automatische Erstbestellungsrabatt nutzt den Stripe-Coupon `QHQESezY` und wird nicht als manueller Rabattcode eingegeben.

**Kündigung:**
- Nur Owner/Admin kann Lizenzen kaufen oder kündigen
- Zusatzlizenzen werden vor der Hauptlizenz gekündigt
- Gekündigte Lizenzen bleiben bis zum angezeigten Laufzeitende nutzbar (`scheduled_end`)
- Kündigungen können bis zum endgültigen Ablaufdatum zurückgezogen werden
- Nach Ablaufdatum ist keine Rücknahme mehr möglich; dann muss eine neue Lizenz gekauft werden
- Gerade aktivierte Direktkäufe können innerhalb von 10 Minuten über „Kauf rückgängig machen" zurückgenommen werden, solange die Lizenz noch nicht regulär weitergelaufen ist
- Rechnungen und Zahlungsmethoden werden über das Stripe-Portal verwaltet

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

**Webapp-Supportformular (`/support`):**
- Kategorien: Produkt & Anwendung, Technisches Problem, Störung/Ausfall, Abrechnung/Vertrag/Lizenz, Zugang & Konto, Funktionswunsch/Feedback, Sonstiges
- Die Kategorie kann per URL vorausgewählt werden, z. B. `/support?category=feature` aus What's-New-CTAs.
- Anhänge werden vor Ticketerstellung über `/support/upload` hochgeladen und anschließend als Attachment-Liste am Ticket gespeichert.
- Die neue Oberfläche nutzt ein zweispaltiges Layout: links Formular, rechts Hinweise zu Kontaktweg, erwarteter Reaktionszeit und benötigten Angaben.

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

## Newsletter-Gutschein

- Landingpage und API bieten Newsletter-Anmeldung mit 10%-Gutschein an.
- Der Gutschein wird nicht beim Formular-Submit erzeugt, sondern erst nach bestaetigtem Brevo Double-Opt-in.
- Brevo sendet dafuer einen Outbound Webhook `list_addition` an `POST /newsletter/brevo/webhook?secret=...`.
- Normdex erzeugt pro E-Mail einen individuellen Stripe Promotion Code auf Basis des Coupons `mbjs8wYE`.
- Jeder Newsletter-Code ist einmalig einloesbar und 30 Tage gueltig.
- Gueltige Codes werden im Lizenz-Checkout lokal geprueft und als Stripe Promotion Code an neue Checkout Sessions oder direkte Subscription-Updates uebergeben.

---

## Admin-Panel

Nur für Nutzer mit dem Flag `is_admin = true`.

**Benutzerverwaltung (`/admin/users`):** Profil einsehen/bearbeiten, E-Mail-Verifizierung setzen, Account aktivieren/deaktivieren, Admin-Flag vergeben.

**Support-Admin (`/admin/support`):** Tickets filtern, Status ändern, Priorität setzen, Antwort senden, interne Notizen, Statistiken.

**Organisationsakte (`/admin/organizations/:orgId`):** Admin-Detailansicht pro Organisation mit Organisation, Nutzer:innen, Projekten, Lizenzen, Bestellungen, Tickets und Timeline. Datumswerte aus verschiedenen Quellen werden zeitzonensicher sortiert, damit gemischte naive/aware Python-`datetime`-Werte keine Sortierfehler auslösen.

---

## Verwandte Dokumente

- [[Wirtschaftlichkeitsrechner]]
- [[App-Routen]]
- [[Datenmodell]]
- [[E-Mail-System]]
