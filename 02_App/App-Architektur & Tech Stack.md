# App-Architektur & Tech Stack

## Architektur-Übersicht

```
Normdex
├── Frontend  (React + TypeScript + Vite, Port 8080)
│   ├── SPA mit React Router v6
│   ├── Tailwind CSS + Radix UI + shadcn/ui
│   └── Auth via HttpOnly JWT Cookie
│
└── Backend  (FastAPI + Python, Port 8000)
    ├── REST API
    ├── PostgreSQL (Prod) / SQLite (Dev) via SQLAlchemy
    ├── Alembic Migrations
    ├── SMTP (Brevo) für E-Mails
    ├── Stripe für Zahlungen
    ├── Microsoft Graph API (Support-Mailbox)
    └── APScheduler für Hintergrund-Jobs
```

**Monorepo-Struktur:**
```
apps/
  api/       → FastAPI Backend
  frontend/  → React Frontend
docs/
  brand/     → Markenrichtlinien (Farben, E-Mail-Theme)
deploy/      → Deployment-Konfiguration
```

---

## Tech Stack Frontend

| Technologie | Zweck |
|---|---|
| React 18.2 | UI Framework |
| TypeScript | Typsicherheit |
| Vite 7.2 (SWC) | Build-Tool |
| React Router v6 | Client-seitiges Routing |
| Tailwind CSS v3.3 | Styling |
| Radix UI | Accessible UI Primitives |
| shadcn/ui | UI-Komponenten auf Radix-Basis |
| Tremor React | Charts, Tabellen, Tabs |
| Recharts | Diagramme in Berechnungsreports |
| React Hook Form | Formular-Verwaltung |
| Zod | Schema-Validierung |
| Lucide React | Icons |
| Sonner | Toast-Benachrichtigungen |
| react-to-print | PDF-Export via Browser-Print |
| react-easy-crop | Avatar-Zuschnitt |
| next-themes | Dark Mode |
| Inter Variable | Schriftart |

---

## Tech Stack Backend

| Technologie | Zweck |
|---|---|
| FastAPI | REST API Framework (async) |
| Python 3.10+ | Sprache |
| Uvicorn | ASGI Server |
| SQLAlchemy 2.0+ | ORM |
| Alembic | Datenbank-Migrationen |
| PostgreSQL | Produktionsdatenbank |
| SQLite | Entwicklungsdatenbank |
| Argon2 | Passwort-Hashing |
| jose | JWT (JSON Web Tokens) |
| SlowAPI | Rate Limiting |
| APScheduler | Hintergrund-Jobs (async) |
| Pillow | Bild-Verarbeitung (Avatar-Resize) |
| httpx | Async HTTP Client (Graph API, Webhooks) |
| Stripe SDK | Zahlungsintegration |

---

## Versionierung & Deployment

**Semantic Versioning** (`MAJOR.MINOR.PATCH`):
- **Patch** (`v0.1.1`): Bugfixes
- **Minor** (`v0.2.0`): Neue Features (abwärtskompatibel)
- **Major** (`v1.0.0`): Große Änderungen / Breaking Changes

Die Versionsnummer liegt in `apps/frontend/package.json` und wird via Vite in die App injiziert (Sidebar, PDF-Reports).

**Deployment:**
- Docker-Container-basiertes Deployment
- Backend: Port 8000
- Frontend: Port 8080 (Dev), statisches Build für Produktion
- Datenbank-Migrationen: `alembic upgrade head`

**Wichtige Environment-Variablen (Produktion):**
```env
DATABASE_URL=postgresql+psycopg://user:pass@host:5432/normdex
JWT_SECRET=<starkes-zufälliges-secret>
FRONTEND_ORIGIN=https://app.normdex.at
BACKEND_ORIGIN=https://api.normdex.at
COOKIE_SECURE=true
ACCESS_TTL_MIN=60

SMTP_HOST=smtp-relay.brevo.com
SMTP_PORT=587
SMTP_FROM_EMAIL=notify@normdex.at
SMTP_FROM_NAME=Normdex

STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

MS_TENANT_ID=...
MS_CLIENT_ID=...
MS_CLIENT_SECRET=...
MS_SHARED_MAILBOX=support@normdex.at

N8N_SUPPORT_WEBHOOK_ENABLED=true
N8N_SUPPORT_WEBHOOK_URL=https://...
N8N_SUPPORT_WEBHOOK_SECRET=...
```

---

## Verwandte Dokumente

- [[App-Routen]]
- [[Authentifizierung & Sicherheit]]
- [[Datenmodell]]
- [[Integrationen & externe Dienste]]
