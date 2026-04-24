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
product_key (z.B. economics_basic)
billing_period (monthly / yearly)
status (active / canceled / expired / trial)
valid_from, valid_until
max_concurrent_users
stripe_customer_id, stripe_subscription_id
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

---

## Verwandte Dokumente

- [[Authentifizierung & Sicherheit]]
- [[Funktionen im Detail]]
- [[API-Endpunkte]]
