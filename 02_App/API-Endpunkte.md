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
GET  /licenses/                      → Lizenzen der Organisation
POST /licenses/{id}/start-usage      → Session starten (Seat belegen)
POST /licenses/{id}/heartbeat        → Keep-Alive
POST /licenses/{id}/end-usage        → Session beenden (Seat freigeben)
POST /licenses/{id}/force-release    → Seat forciert freigeben (Admin)
```

## Abonnements (Stripe)

```
GET  /subscriptions/config                  → Stripe Public Key, Price IDs
POST /subscriptions/create-checkout-session → Stripe Checkout starten
POST /subscriptions/create-portal-session   → Stripe Customer Portal öffnen
```

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
