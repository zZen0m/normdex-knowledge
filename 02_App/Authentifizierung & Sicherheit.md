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

---

## Verwandte Dokumente

- [[App-Architektur & Tech Stack]]
- [[Datenmodell]]
