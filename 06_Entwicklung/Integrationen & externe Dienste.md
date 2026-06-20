# Integrationen & externe Dienste

## Stripe (Zahlungen)

- Subscription-basiertes Billing (monatlich / jährlich)
- Checkout Sessions für neue Abos
- Customer Portal für Änderungen und Kündigung
- Webhook-Infrastruktur für Zahlungsereignisse
- Qualifizierter Einzel-Erstkauf: Checkout-Subscription mit 14 Tagen Trial; Zahlungsdaten werden im Checkout trotzdem erfasst.
- Qualifizierter Mehrfach-Erstkauf: kein Trial, sondern einmaliger Erstbestellungsrabatt von 24,50 EUR über Coupon `QHQESezY`.
- Relevante Webhooks für Trial: `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.trial_will_end`.
- Lokaler Testmodus: keine localhost-Webhooks im Stripe Dashboard anlegen; für lokale Tests Stripe CLI mit `stripe listen --forward-to localhost:8000/subscriptions/webhook` verwenden.

---

### Newsletter-Gutschein / Stripe

- Vorhandener Stripe Coupon fuer 10% Newsletter-Rabatt: `mbjs8wYE`.
- Normdex erzeugt pro bestaetigter Newsletter-E-Mail einen individuellen Stripe Promotion Code.
- Promotion Codes werden mit `max_redemptions=1` und 30 Tagen Gueltigkeit erstellt.
- Die lokale Normdex-Gueltigkeit steht in `newsletter_coupon_claims.expires_at`; Stripe erhaelt zusaetzlich `expires_at` auf dem Promotion Code.
- Gueltige Codes werden bei neuen Stripe Checkout Sessions als `discounts=[{"promotion_code": "..."}]` uebergeben.

---

## Microsoft Graph API (E-Mail-Sync, App)

- Überwacht Shared Mailbox `support@normdex.at`
- Eingehende E-Mails → automatische Ticket-Erstellung oder Ticket-Antworten
- Graph-Subscriptions werden regelmäßig erneuert (APScheduler)
- **Credentials:** Tenant-ID, Client-ID, Client-Secret (Azure AD App Registration)

---

## Brevo (E-Mail-Versand, App)

- Transaktionale E-Mails (Verifizierung, Passwort-Reset, etc.)
- SMTP-Relay: `smtp-relay.brevo.com:587`
- Absender: `notify@normdex.at`

---

### Newsletter Double-Opt-in / Brevo Webhook

- `POST /newsletter/subscribe` startet Brevo Double-Opt-in und speichert lokal nur einen Pending-Claim.
- Brevo Outbound Webhook Event: `list_addition`.
- Ziel in Normdex: `POST /newsletter/brevo/webhook`.
- Normdex verarbeitet nur Webhooks, deren `list_id` die konfigurierte `BREVO_LIST_ID` enthält.
- `BREVO_WEBHOOK_SECRET` ist ein selbst erzeugtes Normdex-Secret. Es wird in Brevo als Bearer-Authentifizierung des Webhooks und in der Backend-Env hinterlegt; es darf nicht Teil der URL sein.
- Erst nach bestätigter Listenaufnahme wird der individuelle Gutschein erzeugt und per SMTP versendet.

---

## n8n (Webhook-Automation, App)

- Empfängt `ticket.created`-Events
- HMAC-SHA256-signierte Payloads
- Retry mit exponentiellem Backoff (max. 5 Versuche, 2/4/8/16 Minuten)

---

## Google reCAPTCHA v2 (Landingpage)

- Schutz des öffentlichen Kontaktformulars
- Site Key: `6Le74XssAAAAAPGE2S4UB6dDRUXWT3XjUw8JPwIm`

---

## Normdex API (Landingpage → Backend)

- Kontaktformular-Einreichung: `POST {VITE_API_URL}/support/public-tickets`
- Newsletter-Anmeldung: `POST {VITE_API_URL}/newsletter/subscribe`
- Environment Variable: `VITE_API_URL` (Default: `http://localhost:8000`)

---

## Verwandte Dokumente

- [[App-Architektur & Tech Stack]]
- [[E-Mail-System]]
- [[Funktionen im Detail]]
