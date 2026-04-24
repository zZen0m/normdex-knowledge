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
