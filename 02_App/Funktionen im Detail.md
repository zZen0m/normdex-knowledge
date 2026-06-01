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
- Gemischte Erstbestellung (monatliche und jährliche Lizenzen in einer Bestellung): erlaubt; es wird genau einmal 24,50 EUR Rabatt auf eine Hauptlizenz angewendet. `calculate_trial_benefit()` wählt deterministisch den ersten verfügbaren Pool aus `discount_options` (Reihenfolge: monatlich vor jährlich), d. h. bei gemischten Bestellungen erhält die monatliche Hauptlizenz den Rabatt. Alle weiteren Lizenzen (auch eine evtl. jährliche Hauptlizenz) werden regulär berechnet.
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
7. Wenn die Hauptlizenz eines Pools bereits zum Laufzeitende gekündigt ist (`scheduled_end`), sind weitere Käufe in diesem Pool blockiert. Der Kunde muss die Kündigung der Hauptlizenz zuerst zurückziehen oder bis zum Laufzeitende warten.

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

**Lizenzseite `/licenses` (Oberfläche):**
- Kopfbereich mit nutzungsbasierter **KPI-Übersicht** (für alle Rollen sichtbar, ohne Kosten): Lizenzen gesamt, aktuell aktive Sitzungen, Nutzungen diesen Monat, Nutzungen letzten Monat (jeweils mit Anzahl Nutzer:innen). Datenquelle: `GET /licenses/usage-stats` (Monatsgrenzen in Europe/Vienna).
- **Aktualisieren**-Button lädt Pool-, Lizenz- und Nutzungsdaten neu (Sitzungen sind live).
- „Lizenz hinzufügen" nur für Owner/Admin.
- Optik einheitlich zur Teamseite (geteilte `StatCard`-Komponente, `rounded-2xl` + `shadow-soft`, Icon-Chips).
- Eine feste Lizenz-Zuweisung pro Nutzer:in ist bewusst **nicht** in der UI exponiert; es gilt das Floating-Seat-Modell (freie Sitzungen). Die Assign-/Unassign-Endpunkte bleiben backendseitig bestehen.

**Kaufdialog „Lizenzen hinzufügen" (Oberfläche):**
- Plan-Wahl als zwei **Auswahlkarten** (monatlich/jährlich) mit Icon-Chip, Preis, integriertem Mengen-Stepper und Aktiv-Zustand (Ring + Häkchen); beide Pools können gleichzeitig gewählt werden (getrennte Stripe-Checkouts, Hinweis-Banner).
- Jährliche Option trägt ein **„2 Monate gratis"-Badge** (rechnerisch exakt: 49 €×12 = 588 € vs. 490 €; Addon 29 €×12 = 348 € vs. 290 €).
- Gutscheincode wird als entfernbarer Chip mit Häkchen angezeigt; Bestellübersicht mit Hero-Summe (`tabular-nums`) und **„Sichere Zahlung über Stripe"**-Trust-Hinweis.
- Optik auf neue Designsprache gehoben (`rounded-2xl`/`shadow-soft`, Icon-Chip-Header, dark-taugliche Hinweisfarben). Kauf-/Preview-/Promo-Logik unverändert.

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

## Benachrichtigungen

In-App-Benachrichtigungssystem mit persistenter Historie pro User und Live-Updates über Server-Sent Events.

**UI-Einstiege:**
- **Bell-Icon** in der Sidebar mit pinkem Badge (Anzahl Ungelesene, `9+` bei mehr als 9). Klick öffnet ein Popover mit den 8 jüngsten Einträgen.
- **Vollansicht** unter `/notifications` mit Tab-Filter (Alle/Ungelesen), Hover-Delete und „Alle als gelesen“-Aktion.

**Verhalten:**
- Klick auf eine Notification → Sprung zur Ziel-Route (`link`-Feld) und automatisches Mark-as-Read.
- Live-Push: Beim Erzeugen einer Notification erscheint sofort ein `notify.info`-Toast (sonner) und der Badge zählt hoch — ohne Reload.
- Bei Verbindungsabbruch reconnected der Browser automatisch (`EventSource`-Default). Zusätzlich läuft alle 5 Minuten und beim Tab-Focus ein leichter Refetch der Unread-Anzahl als Safety-Net.

**Notification-Typen (MVP):**

| Typ | Trigger | Empfänger |
|---|---|---|
| `welcome` | Registrierung (Self oder Invite-Annahme) | Neuer User; bei Invite org-spezifisch |
| `license_purchased` | Stripe `checkout.session.completed` (idempotent über `order.meta`) | `LicenseOrder.created_by_user_id` |
| `license_expiring_soon` | Täglicher Scheduler 06:00 für Buckets 14/7/1 Tage | `created_by_user_id` → `assigned_user_id` → Org-Owner |
| `license_expired` | Täglicher Scheduler 06:00 (Bucket 0 Tage) | wie oben |

Nur Lizenzen mit `cancel_at_period_end=True` werden im Scheduler berücksichtigt — Auto-Renew-Lizenzen laufen nicht aus und brauchen keine Warnung.

**Architektur-Hinweise:**
- Persistenz in Tabelle `notifications` (siehe [[Datenmodell#Notification]]).
- Service-Layer `app/services/notifications.py` mit `create_notification(...)` als zentralem Schreibpfad — Trigger nutzen niemals direkt das ORM.
- Live-Push via In-Process-SSE-Broker (`asyncio.Queue` pro `user_id`). **Single-Worker-tauglich** — bei Multi-Worker-Deployment ist Redis Pub/Sub nötig.
- Endpoints: siehe [[API-Endpunkte#Benachrichtigungen]]. Für End-to-End-Verifikation des SSE-Pfads existiert `POST /notifications/_admin/test` (Admin only).
- Abgrenzung zu [[Toast-Benachrichtigungen]]: Toasts sind flüchtige UI-Signale (sonner), Notifications sind persistente Datensätze. Live-Notifications zeigen beim Empfang zusätzlich einen Toast als Aufmerksamkeitssignal.

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
