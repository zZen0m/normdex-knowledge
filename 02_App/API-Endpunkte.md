# API-Endpunkte

Die REST-API läuft auf Port 8000. Alle gesicherten Endpunkte erfordern ein gültiges JWT-Cookie.

## Authentifizierung

```
POST /auth/register          → Registrierung
POST /auth/login             → Login
POST /auth/logout            → Logout (invalidiert Session)
POST /auth/verify/send       → Verifikations-E-Mail senden
GET  /auth/verify?token=...  → E-Mail verifizieren
POST /auth/password/forgot   → Reset-E-Mail anfordern
POST /auth/password/reset    → Passwort zurücksetzen
POST /auth/password/change   → Passwort ändern (auth)
POST /auth/email-change/start → E-Mail-Änderung initiieren
```

## Nutzer & Profil

```
GET    /me                        → Eingeloggter Nutzer
GET    /users/me/profile          → Vollständiges Profil
PUT    /users/me/profile          → Profil aktualisieren
PATCH  /users/me/settings         → Einstellungen (JSON Patch)
POST   /users/me/avatar           → Avatar hochladen
POST   /users/me/export/request   → Daten-Export anfordern
POST   /users/me/delete/start     → Account-Löschung initiieren
POST   /users/me/delete/confirm   → Account-Löschung bestätigen
```

## Projekte

```
GET    /api/projects/      → Projektliste
POST   /api/projects/      → Neues Projekt
GET    /api/projects/{id}  → Projektdetail
PUT    /api/projects/{id}  → Projekt bearbeiten
DELETE /api/projects/{id}  → Projekt löschen
```

## Economics Berechnungen

```
POST /economics/calculate                       → Standalone-Berechnung
POST /economics/projects/{projectId}/calculate  → Projektgebundene Berechnung
GET  /economics/{id}                            → Berechnungsergebnis abrufen
```

## Organisation & Team

```
GET    /organizations/me                              → Eigene Organisation
PATCH  /organizations/{org_id}                        → Organisation bearbeiten
GET    /organizations/{org_id}/members                → Mitgliederliste
DELETE /organizations/{org_id}/members/{user_id}      → Mitglied entfernen
PATCH  /organizations/{org_id}/members/{user_id}      → Rolle ändern
GET    /organizations/{org_id}/invites                → Einladungen
POST   /organizations/{org_id}/invites                → Einladung senden
DELETE /organizations/{org_id}/invites/{invite_id}    → Einladung widerrufen
POST   /organizations/{org_id}/logo                   → Logo hochladen
DELETE /organizations/{org_id}/logo                   → Logo löschen
```

## Lizenzen

```
GET  /licenses/                         → Aktive Lizenzen der Organisation (Legacy-/Usage-kompatibel)
GET  /licenses/pools                    → Pool-Zusammenfassung monatlich/jährlich
GET  /licenses/pools/{pool}/items       → Einzellizenzen eines Pools
GET  /licenses/{id}/history             → Lizenzhistorie
POST /licenses/checkout/preview         → Kaufvorschau mit Staffelpreisen
POST /licenses/checkout/create          → Kauf starten oder bestehende Subscription erweitern
POST /licenses/checkout/confirm         → Stripe Checkout nach Redirect bestätigen
POST /licenses/checkout/cancel          → Abgebrochenen Checkout bereinigen
POST /licenses/{id}/cancel              → Lizenz zum Laufzeitende kündigen
POST /licenses/{id}/reactivate-cancel   → Kündigung innerhalb der Restlaufzeit zurückziehen
POST /licenses/{id}/undo-purchase       → Direkten Zusatzkauf innerhalb von 10 Minuten rückgängig machen
POST /licenses/{id}/start-usage         → Session starten (Lizenz belegen)
POST /licenses/{id}/heartbeat           → Keep-Alive
POST /licenses/{id}/end-usage           → Session beenden (Lizenz freigeben)
POST /licenses/{id}/force-release       → Session forciert freigeben (Admin)
POST /licenses/{id}/assign-user         → Benutzer zuweisen (Backend vorhanden, UI nicht exponiert)
POST /licenses/{id}/unassign-user       → Benutzerzuweisung entfernen (Backend vorhanden, UI nicht exponiert)
POST /licenses/promotions/validate      → Stripe Promotion Code prüfen
POST /admin/licenses/grant-complimentary → Kostenlose/interne Lizenz vergeben (System-Admin)
```

Newsletter-Gutscheincodes werden lokal gegen `newsletter_coupon_claims.expires_at` geprueft. Gueltige Newsletter-Codes werden als Stripe `promotion_code` an Checkout Sessions uebergeben.

## Newsletter

```
POST /newsletter/subscribe                -> Brevo Double-Opt-in starten und Pending-Claim speichern
POST /newsletter/brevo/webhook?secret=... -> Brevo list_addition verarbeiten und Gutschein versenden
```

Der Webhook verarbeitet nur `event = list_addition` und nur dann, wenn `list_id` die konfigurierte `BREVO_LIST_ID` enthaelt. Vor bestaetigtem Double-Opt-in wird kein Gutschein erzeugt.

## Abonnements (Stripe)

```
GET  /subscriptions/config                → Stripe Public Key
POST /subscriptions/create-portal-session → Stripe Customer Portal öffnen
```

Käufe und Kündigungen laufen über die neuen `/licenses/...`-Endpunkte. Rabattcodes werden aktuell nur im Stripe Checkout eingegeben, wenn eine neue Subscription entsteht.

## Support (öffentlich)

```
POST /support/tickets   → Ticket erstellen (eingeloggter Nutzer)
POST /support/upload    → Anhang hochladen
POST /support/public-tickets → Ticket via Landingpage-Kontaktformular
```

## Admin

```
GET    /admin/users                        → Nutzerliste
GET    /admin/users/{id}                   → Nutzerdetail
PATCH  /admin/users/{id}                   → Nutzer bearbeiten
POST   /admin/users/{id}/password          → Passwort setzen
DELETE /admin/users/{id}                   → Nutzer löschen
GET    /admin/audit                        → Audit-Log
GET    /admin/organizations                → Organisationsliste
PATCH  /admin/organizations/{id}           → Organisation bearbeiten
GET    /admin/projects                     → Alle Projekte (plattformweit)
GET    /admin/support/tickets              → Ticket-Liste
GET    /admin/support/tickets/{id}         → Ticket-Detail
POST   /admin/support/tickets/{id}/reply   → Antwort / Status-Update
PATCH  /admin/support/tickets/{id}         → Status/Priorität/Kategorie/Assignee
GET    /admin/support/stats                → Support-Metriken
```

## Stats & Health (API-Key geschützt)

```
GET /stats/subscriptions/count   → Lizenz-Zählungen
GET /stats/health/summary        → System-Gesundheitscheck
GET /stats/support/summary       → Support-Metriken
GET /health                      → Server-Healthcheck
```

---

## Verwandte Dokumente

- [[Authentifizierung & Sicherheit]]
- [[Wirtschaftlichkeitsrechner]]
- [[Datenmodell]]
