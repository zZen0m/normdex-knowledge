# E-Mail-System & Benachrichtigungen

## SMTP-Konfiguration

**Anbieter:** Brevo (ehemals Sendinblue)
- **Absender:** `notify@normdex.at` (Name: „Normdex")
- **Host:** `smtp-relay.brevo.com:587`

---

## E-Mail-Übersicht

| E-Mail | Auslöser |
|---|---|
| **E-Mail-Verifizierung** | Nach Registrierung (Double-Opt-In) |
| **Passwort zurücksetzen** | Bei „Passwort vergessen" |
| **E-Mail-Änderung (Bestätigung)** | Neue E-Mail-Adresse bestätigen |
| **E-Mail-Änderung (Sperrung)** | Alte Adresse informieren |
| **Team-Einladung** | Mitglied zur Organisation eingeladen |
| **Support-Ticket erstellt** | Auto-Reply an Kunden mit Ticket-ID |
| **Support-Ticket geschlossen** | Abschlussnachricht an Kunden |
| **Account-Löschung** | Bestätigung vor der Löschung |
| **Daten-Export bereit** | Wenn DSGVO-Export abholbereit ist |
| **Testzeitraum-Erinnerung** | Ca. 3 Tage vor Ablauf des 14-tägigen Testzeitraums (via Stripe-Webhook `customer.subscription.trial_will_end`); Empfänger: Org-Owner |
| **Gutschriftbeleg** | Nach erfolgreicher Stripe Credit Note; Empfänger: Organisations-Owner / Rechnungsempfänger |

---

## Lizenz- und Bestell-E-Mails

- Produktlabels in Bestell-/Lizenz-E-Mails verwenden den zentralen Economics-Produktnamen aus den App-Konstanten. Legacy-Key `economics_v1` und aktueller Produkt-Key werden auf denselben sichtbaren Produktnamen gemappt.
- Dadurch erscheinen Trial-, Zusatzkauf- und Bestellbestätigungen konsistent mit dem Namen der App-Oberfläche und nicht mehr mit dem alten Label `Normdex ÖNORM M 7140 Basic`.

---

## Newsletter-Gutschein

- E-Mail: Newsletter-Gutschein nach bestaetigter Aufnahme in die Brevo-Newsletter-Liste (`list_addition` Webhook).
- Ausloeser: Brevo Outbound Webhook `list_addition` fuer die konfigurierte Newsletter-Liste.
- Vor Double-Opt-in-Bestaetigung wird kein Gutschein versendet.
- Normdex erzeugt einen individuellen Stripe Promotion Code auf Basis des Coupons `mbjs8wYE`.
- Jeder Code ist einmalig einloesbar und 30 Tage gueltig.
- Die Gutschein-Mail enthaelt Code, Ablaufdatum und Link zu `/licenses`.
- Idempotenz: erneute Webhooks fuer dieselbe E-Mail erzeugen keinen zweiten Code und senden nach `coupon_sent_at` keine zweite Mail.

---

## Gutschriftbeleg

- Normdex versendet die gebrandete E-Mail führend über Brevo SMTP.
- Enthalten sind Gutschriftnummer, Bezugsrechnung, Bruttobetrag, Grund und Link zum Stripe-Gutschrift-PDF.
- Stripes eigener Credit-Note-Mailversand wird beim API-Aufruf mit `email_type = none` unterdrückt.
- `BillingAdjustment.document_delivery_started_at` schützt vor Doppelversand bei einem unklaren Prozessabbruch: Ein unsicherer Zustellstatus wird manuell geprüft statt automatisch erneut gesendet.
- Normale SMTP-Fehler werden unabhängig vom bereits abgeschlossenen Geldfluss mit Backoff wiederholt.
- Erfolgreiche Zustellung wird über `document_sent_at` und `document_recipient` nachvollziehbar gespeichert; Outbox und Audit bleiben erhalten.

---

## Support-Mailbox (eingehend)

- **Shared Mailbox:** `support@normdex.at`
- **Integration:** Microsoft Graph API
- Eingehende E-Mails werden automatisch in Support-Tickets umgewandelt oder bestehenden Tickets zugeordnet
- Graph-Subscriptions werden regelmäßig erneuert (APScheduler)

---

## Verwandte Dokumente

- [[Funktionen im Detail]]
- [[Integrationen & externe Dienste]]
- [[Designsystem & Farben]] (E-Mail-Farbschema)
