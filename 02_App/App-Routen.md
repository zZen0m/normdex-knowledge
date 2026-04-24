# App-Routen

## Öffentliche Routen (ohne Login)

| Route | Beschreibung |
|---|---|
| `/impressum` | Impressum / Rechtliche Informationen |
| `/datenschutz` | Datenschutzerklärung |

## Authentifizierungs-Routen

| Route | Beschreibung |
|---|---|
| `/auth/login` | Login-Formular (E-Mail + Passwort) |
| `/auth/register` | Registrierung mit Account-Typ-Auswahl (Privat/Unternehmen) |
| `/auth/forgot` | Passwort-Vergessen: E-Mail-Eingabe |
| `/auth/reset` | Passwort zurücksetzen (mit Token aus E-Mail-Link) |
| `/auth/verify` | E-Mail-Adresse bestätigen (Double-Opt-In) |
| `/auth/delete-confirm` | Account-Löschung bestätigen |
| `/auth/goodbye` | Bestätigungsseite nach Account-Löschung |

## App-Routen (Login erforderlich)

| Route | Beschreibung |
|---|---|
| `/app` | **Dashboard** – Übersicht, Statistiken, Onboarding |
| `/projects` | **Projektliste** – alle Projekte der Organisation |
| `/projects/new` | **Neues Projekt** anlegen |
| `/projects/:projectId` | **Projektdetail** – Metadaten, verknüpfte Berechnungen |
| `/team` | **Teamverwaltung** – Mitglieder einladen, Rollen ändern |
| `/licenses` | **Lizenzverwaltung** – aktive Lizenzen, Nutzung |
| `/whats-new` | **Changelog** – neue Features & Updates |
| `/calculations/economics` | **Wirtschaftlichkeitsrechner** (standalone, ohne Projekt) |
| `/calculations/economics/:projectId` | **Wirtschaftlichkeitsrechner** (projektgebunden) |
| `/calculations/economics/report/:id` | **Berechnungsbericht** (PDF-Export-Ansicht) |
| `/settings/profile` | **Profil-Einstellungen** – Name, Avatar, Passwort |
| `/settings/organization` | **Organisations-Einstellungen** – Adresse, Logo, USt.-ID |
| `/settings/subscription` | **Abo-Verwaltung** – aktuelles Abonnement, Kündigung |
| `/settings/subscription/plans` | **Abopläne** – Preisübersicht, Upgrade |
| `/help` | **Hilfe** – Dokumentation & FAQ |
| `/support` | **Support-Formular** – Ticket erstellen |

## Admin-Routen (nur für Admins)

| Route | Beschreibung |
|---|---|
| `/admin/users` | **Benutzerverwaltung** – Suche, Filter, Bearbeiten |
| `/admin/support` | **Support-Posteingang** – alle Tickets |
| `/admin/support/:ticketId` | **Ticket-Detailansicht** – Antworten, Statusverwaltung |
