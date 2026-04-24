# Integrationen & externe Dienste

## Stripe (Zahlungen)

- Subscription-basiertes Billing (monatlich / jährlich)
- Checkout Sessions für neue Abos
- Customer Portal für Änderungen und Kündigung
- Webhook-Infrastruktur für Zahlungsereignisse

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
- Environment Variable: `VITE_API_URL` (Default: `http://localhost:8000`)

---

## Verwandte Dokumente

- [[App-Architektur & Tech Stack]]
- [[E-Mail-System]]
- [[Funktionen im Detail]]
