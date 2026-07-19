# Datenmodell

## User
```
id, email (unique), password_hash
is_active, is_verified, is_admin
first_name, last_name, display_name
account_type (private/company)
company, salutation, birth_date
locale, timezone, avatar_url
token_version (Session-Invalidierung)
settings (JSON)
```

## Organization
```
id (UUID), name
address (JSON: street, city, postal_code, country)
billing_address (JSON)
billing_email (zentrale Rechnungs-E-Mail; leer = kein Dokumentversand per E-Mail)
vat_id, customer_number
stripe_customer_id
trial_used_at (dauerhafter Lock für 14-Tage-Testvorteil)
company_activity (JSON-Array)
report_settings (JSON: logo_url, farben, kopfzeile)
```

## Membership
```
id (UUID), user_id, organization_id
role (owner / member)
```

## Invite
```
id (UUID), organization_id, email
token (UUID, einmalig)
role, created_by, expires_at
```

## License
```
id (UUID), organization_id
product_key (z.B. economics_v1)
billing_pool (monthly / yearly)
license_kind (base / addon)
status (pending / trial / active / scheduled_end / ended / payment_failed)
started_at
trial_started_at, trial_ends_at, trial_converted_at
current_term_start, current_term_end, committed_until
cancel_requested_at, scheduled_end_at
price_amount_gross, currency
stripe_subscription_id, stripe_subscription_item_id, stripe_price_id
assigned_user_id, created_by_user_id
meta (JSON: u.a. license_order_id, Trial-Benefit-/Checkout-Informationen)
```

## LicenseOrder
```
id (UUID), organization_id, created_by_user_id
promotion_code
subtotal_gross, discount_total_gross, total_gross, currency
status (pending / completed / failed)
stripe_checkout_session_id
meta (JSON: checkout_sessions, trial_benefit_kind, trial_benefit_applied, trial_benefit_released_at)
```

## NewsletterCouponClaim
```
id (UUID)
email (unique, normalisiert)
firstname, lastname, company, role, source
status (pending / confirmed / coupon_created / coupon_sent / failed)
stripe_coupon_id, stripe_promotion_code_id, code
brevo_contact_id, brevo_event_id
confirmed_at, coupon_sent_at, expires_at
last_error
created_at, updated_at
```

Verwendung: Speichert den Newsletter-Gutscheinprozess. `pending` entsteht bei `/newsletter/subscribe`; der Coupon wird erst nach Brevo `list_addition` erzeugt. `expires_at` ist die lokale Normdex-Gueltigkeit fuer den individuellen Code.

## LicenseOrderItem
```
id (UUID), license_order_id
billing_pool (monthly / yearly)
license_kind (base / addon)
quantity
unit_price_gross, line_total_gross
```

## LicenseUsage
```
id (UUID), license_id, user_id
last_heartbeat (Timestamp)
```

## Project
```
id (UUID), user_id, organization_id
project_number (eindeutig)
name, description
calculation_type (economics)
form_data (JSON)
Standort: street, city, postal_code, country
Auftraggeber: client_name, client_street, client_city, client_postal_code, client_country
Verfasser: author_company, author_contact, author_street, author_city, author_postal_code, author_country
```

## Calculation
```
id (UUID), user_id, project_id (optional)
type (economics)
name
input_json (Berechnungseingaben)
output_json (Berechnungsergebnisse)
```

## SupportTicket
```
id (UUID)
ticket_id (NDX-YYYY-######, unique)
source (webapp / landingpage / email)
status (new / triaged / in_progress / waiting_on_customer / resolved / closed / spam / duplicate)
priority (p1 / p2 / p3 / p4)
category
requester_email, requester_name, company_name
assignee_user_id
tags (JSON-Array)
auto_close_at
first_response_at, reopened_at
meta (JSON)
```

## SupportMessage
```
id (UUID), ticket_id
direction (inbound / outbound / internal_note / system)
channel (email / web)
author_type (customer / agent / system)
body_text, body_html
graph_message_id, internet_message_id, conversation_id
```

## AuditLog
```
id, user_id
event (z.B. login, registration, password_change, profile_updated)
ip, user_agent
meta (JSON)
```

## BillingAdjustment
```
id (UUID), request_key (eindeutig, optional für Bestandsvorgänge)
organization_id, license_ids (JSON)
admin_user_id, support_ticket_id
stripe_customer_id, stripe_subscription_id, stripe_invoice_id
stripe_payment_intent_id, stripe_charge_id
stripe_refund_id, stripe_credit_note_id
credit_note_number, credit_note_pdf_url
credit_amount_gross, refund_amount_gross, tax_amount, currency
reason, reason_code
status, attempt_count, last_error, next_attempt_at
document_delivery_started_at, document_sent_at
document_recipient, document_send_error
meta (JSON: Rechnungsnummer/-datum, Originalpositionen, Credit-Note-Lines,
      mehrere Refund-IDs, Steuerbehandlung, Request-Payload,
      persistenter Stripe-Subscription-Operationsplan)
created_at, completed_at
```

Persistenter Workflow für T030. Statusfolge: `pending → subscription_adjusted → credit_note_created → refund_created → document_sent → completed`; Fehlerpfade: `failed`, `partially_failed`, `manual_review_required`. `request_key` schützt den gesamten Admin-Request vor Duplikaten; Stripe-Schreibvorgänge verwenden zusätzlich stabile Idempotency Keys. Ein persistierter, aber nicht sicher abgeschlossener Mailversand wird nicht automatisch doppelt gesendet, sondern zur manuellen Prüfung markiert. Aktueller Alembic-Head für diesen Workflow: `e3c4d5e6f7a8`.

## BillingDocumentDelivery
```
id (UUID), organization_id
document_type (invoice / credit_note)
stripe_document_id, stripe_invoice_id
document_number, document_pdf_url
amount_gross, currency, document_created_at
delivery_started_at, sent_at, recipient
send_error, status, attempt_count, next_attempt_at
created_at, updated_at
```

Persistiert den Normdex-Versandstatus von Stripe-Abrechnungsdokumenten. Die Kombination aus `document_type` und `stripe_document_id` ist eindeutig und schützt insbesondere `invoice.paid` vor Doppelversand. 0-Euro-Rechnungen erhalten den Status `not_required`; unklare Crash-Zustände werden als `manual_review_required` markiert. Aktueller Alembic-Head: `f4d5e6f7a8b9`.

## WebhookEvent
```
id (UUID), event_type, payload (JSON)
status (pending / delivered / failed)
retry_count, last_error, next_delivery_at
```

## Notification
```
id (UUID), user_id (FK users, ondelete CASCADE)
type (z.B. welcome / license_purchased / license_expiring_soon / license_expired)
title, body, link (optional, interne Route)
meta (JSON, typ-spezifisch: license_id, organization_id, order_id, ...)
read_at (NULL = ungelesen)
created_at
```

Indizes: `(user_id)`, `(read_at)`, `(created_at)`, `(user_id, read_at)`, `(user_id, created_at)`.
Persistente In-App-Benachrichtigungen je User. Cascade-Delete beim User. Siehe [[Funktionen im Detail#Benachrichtigungen]] und [[API-Endpunkte#Benachrichtigungen]].

## LinkedIn-Marketing

```text
linkedin_connections
  Organisations-URN, verschlüsselte Access-/Refresh-Tokens,
  Ablaufzeitpunkte, Verbindungsstatus, letzter Token-Refresh

linkedin_sync_runs
  Trigger, Status, Start/Ende, auslösender Admin,
  Post-Zähler und sanitierte Fehlerkategorie/-meldung

linkedin_posts
  eindeutige LinkedIn-URN, Lifecycle, Text, URL,
  Anlage-/Veröffentlichungszeitpunkt, Sichtkontakte, unavailable-Kennzeichen

linkedin_post_snapshots
  Post-FK, Sync-Run-FK, Messzeitpunkt und Leistungskennzahlen;
  eindeutig pro Post und Sync-Lauf

linkedin_follower_snapshots
  Sync-Run-FK, Messzeitpunkt, organische/bezahlte/gesamte Follower;
  höchstens ein Snapshot pro Sync-Lauf
```

Aktueller Alembic-Head: `a5c6d7e8f9b0`. Zeitbereichs-, Status- und Post-/Messzeitpunkt-Indizes unterstützen die Admin-Auswertung. Details: [[LinkedIn-Marketing-KPIs]].

---

## Verwandte Dokumente

- [[Authentifizierung & Sicherheit]]
- [[Funktionen im Detail]]
- [[API-Endpunkte]]
