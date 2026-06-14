# Authentifizierung & Sicherheit

## Auth-Konzept

| Aspekt | Details |
|---|---|
| **Auth-Methode** | JWT-basierte Sessions im HttpOnly Cookie (`access_token`) |
| **Token-Lebensdauer** | Standard 60 Minuten (konfigurierbar via `ACCESS_TTL_MIN`) |
| **Auto-Renewal** | Bei < 50 % verbleibender Laufzeit stilles Erneuern |
| **Passwort-Hashing** | Argon2 |
| **Session-Invalidierung** | `token_version` in DB; wird bei Logout erhöht |
| **Cookie-Policy** | `SameSite=Lax`, `Secure=true` in Produktion |
| **CORS** | Nur konfigurierte Origins erlaubt |
| **Rate Limiting** | SlowAPI |

---

## Admin-Bootstrap

```sql
UPDATE users SET is_admin = true WHERE email = 'admin@normdex.at';
```

## Team-Einladungen

- Einladungstokens werden bei der Registrierung serverseitig an die normalisierte eingeladene E-Mail-Adresse gebunden.
- Abweichende E-Mail-Adressen werden mit HTTP 403 abgelehnt, bevor ein Benutzerkonto angelegt wird.
- Ungültige, abgelaufene, widerrufene oder bereits verwendete Tokens werden mit HTTP 400 abgelehnt; es gibt keinen Fallback auf eine eigene Organisation.
- Die Annahme sperrt den Invite-Datensatz per `SELECT ... FOR UPDATE`. Benutzer, Mitgliedschaft und `accepted_at` werden in einer gemeinsamen Datenbanktransaktion gespeichert.
- Der öffentliche Invite-Info-Endpunkt liefert nur eingeladene E-Mail-Adresse, Organisationsname und Rolle. Rechnungsadresse, UID und Tätigkeitsdaten werden nicht öffentlich ausgegeben.

## Support-Anhänge

- Öffentliche Static-Routen existieren nur für Avatare und Organisationslogos.
- Support-Anhänge unter `uploads/attachments/` sind nicht direkt öffentlich erreichbar.
- Downloads erfolgen über `GET /admin/support/tickets/{ticket_id}/attachments/{attachment_id}` und erfordern eine aktive Admin-Session.
- Der Endpunkt prüft die Datenbankzuordnung zum Ticket, den Retention-Löschstatus und den Storage-Pfad. Der interne `storage_key` wird nicht an das Frontend ausgegeben.

---

## Verwandte Dokumente

- [[App-Architektur & Tech Stack]]
- [[Datenmodell]]
