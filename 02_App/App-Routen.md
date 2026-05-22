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
| `/licenses` | **Lizenzverwaltung** – Pool-Übersicht, Lizenzkauf, Trial-Konvertierung, Sitzungsbereinigung, Kündigung und Stripe-Portal |
| `/whats-new` | **What's New** – editorial aufbereitete Release-Ausgaben mit Story-Abschnitten, Fixes, Roadmap und Archiv |
| `/calculations/economics` | **Wirtschaftlichkeitsrechner** (standalone, ohne Projekt) – Outline-Sidebar, Live-Vorschau und Export-Konfiguration |
| `/calculations/economics/:projectId` | **Wirtschaftlichkeitsrechner** (projektgebunden) – gleiche Rechneroberfläche mit Projektmetadaten |
| `/calculations/economics/report/:id` | **Berechnungsbericht** (PDF-Export-Ansicht) |
| `/settings/profile` | **Profil-Einstellungen** – Name, Avatar, Passwort |
| `/settings/organization` | **Organisations-Einstellungen** – Adresse, Logo, USt.-ID |
| `/settings/subscription` | **Abo-Verwaltung** – aktuelles Abonnement, Kündigung |
| `/settings/subscription/plans` | **Abopläne** – Preisübersicht, Upgrade |
| `/help` | **Hilfe & Dokumentation** – Sticky-TOC + Schnellstart und 6 Bereiche (Team, Lizenz, Projekt/Berechnung, Benutzerkonto, Unternehmen, Support & Community) mit Step-by-Step-Anleitungen. User-Facing Single Source of Truth für die App-Bedienung. |
| `/support` | **Support-Formular** – Ticket erstellen, Kategorie per URL vorauswählen (`?category=feature`), Anhänge hochladen, seitliche Kontakt-/Erwartungsinfos |

## Admin-Routen (nur für Admins)

| Route | Beschreibung |
|---|---|
| `/admin/users` | **Benutzerverwaltung** – Suche, Filter, Bearbeiten |
| `/admin/organizations/:orgId` | **Organisationsakte** – Organisation, Nutzer, Lizenzen, Bestellungen, Projekte, Tickets und Timeline; Zeitstempel werden zeitzonensicher sortiert |
| `/admin/support` | **Support-Posteingang** – alle Tickets |
| `/admin/support/:ticketId` | **Ticket-Detailansicht** – Antworten, Statusverwaltung |
