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

---

## Verwandte Dokumente

- [[Authentifizierung & Sicherheit]]
- [[Funktionen im Detail]]
- [[API-Endpunkte]]
