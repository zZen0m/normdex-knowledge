# Designsystem & Farben

## Konzeptionelle Markenfarben

Beide Produkte (App und Landingpage) verwenden dasselbe konzeptionelle Farbschema, unterscheiden sich aber in den CSS-Variablen-Namen:

| Konzept          | Farbe                           | Hex / HSL                       | Verwendung                                        |
| ---------------- | ------------------------------- | ------------------------------- | ------------------------------------------------- |
| **Dunkel-Teal**  | Primary (App) / Foreground (LP) | `#003c3e` / `hsl(182 100% 12%)` | Navigation, Header, Footer, Hauptelemente         |
| **Magenta/Pink** | Accent (App) / Primary (LP)     | `#ff2d58` / `hsl(343 100% 59%)` | Call-to-Actions, Highlights, fokussierte Zustände |
| **Hellcyan**     | Secondary (LP)                  | `hsl(182 50% 96%)`              | Sektionshintergründe auf der Landingpage          |
| **Off-White**    | Background                      | `#fafafa` / `hsl(0 0% 98%)`     | Seitenhintergrund                                 |

---

## App-Designsystem (Design Tokens)

### Core Brand

| Token | Hex | Verwendung |
|---|---|---|
| `ndx.core.primary` | `#003c3e` | Navigation, Header, Hauptelemente – Deep Teal |
| `ndx.core.primary_foreground` | `#fafafa` | Text auf Primary-Hintergründen |
| `ndx.core.accent` | `#ff2d58` | Call-to-Actions, Highlights – Vibrant Pink |
| `ndx.core.accent_foreground` | `#ffffff` | Text auf Accent-Hintergründen |

### Neutrals

| Token | Hex | Verwendung |
|---|---|---|
| `ndx.neutral.background` | `#fafafa` | Haupt-Seitenhintergrund |
| `ndx.neutral.surface` | `#ffffff` | Karten- und Modal-Hintergründe |
| `ndx.neutral.border` | `#e0e0e0` | Rahmen, Trennlinien |
| `ndx.neutral.text` | `#282f3a` | Primärer Fließtext |
| `ndx.neutral.muted_text` | `#64748b` | Sekundärtext, Platzhalter |

### Semantische Farben (Status)

| Token | Hex | Verwendung |
|---|---|---|
| `ndx.semantic.success` | `#16a34a` | Erfolgsmeldungen, gültige Zustände |
| `ndx.semantic.warning` | `#f59e0b` | Warnungen, handlungsrelevante Hinweise |
| `ndx.semantic.error` | `#ff2d58` | Fehler, destruktive Aktionen (= Accent-Farbe) |
| `ndx.semantic.info` | `#3b82f6` | Infomeldungen, Links |

### Dark Mode (App)

| Element | Wert |
|---|---|
| Hintergrund | `hsl(200 10% 8%)` – sehr dunkles Blau-Grau |
| Vordergrund | `hsl(210 40% 98%)` – fast weiß |
| Primary | `hsl(182 100% 22%)` – helleres Teal |

### Schatten (App)

| Level | CSS-Wert |
|---|---|
| Soft | `0 4px 20px -4px rgba(0,0,0,0.1)` |
| Medium | `0 8px 32px -8px rgba(0,0,0,0.15)` |
| Strong | `0 16px 48px -12px rgba(0,0,0,0.2)` |

---

## Landingpage-Designsystem (CSS-Variablen, HSL)

### Basis-Tokens

| CSS-Variable | HSL-Wert | Verwendung |
|---|---|---|
| `--primary` | `hsl(343 100% 59%)` | Magenta/Pink – CTAs, Akzente |
| `--primary-foreground` | `hsl(0 0% 98%)` | Text auf Primary-Hintergründen |
| `--secondary` | `hsl(182 50% 96%)` | Hellcyan – Sektionshintergründe |
| `--foreground` | `hsl(182 100% 12%)` | Dunkles Teal – Haupttext, Header, Footer |
| `--background` | `hsl(0 0% 98%)` | Off-White – Seitenhintergrund |
| `--card` | `hsl(0 0% 100%)` | Karten-Hintergründe |
| `--muted` | `hsl(182 20% 94%)` | Gedämpfte Hintergründe |
| `--muted-foreground` | `hsl(182 30% 45%)` | Sekundärtext |
| `--border` | `hsl(220 13% 91%)` | Rahmen, Trennlinien |
| `--radius` | `1rem` | Abgerundete Ecken (App: `0.5rem`) |

### Gradienten & Muster

| Token | Wert | Verwendung |
|---|---|---|
| `--gradient-hero` | `linear-gradient(135deg, hsl(0 0% 98%) 0%, hsl(182 50% 96%) 100%)` | Hero-Hintergrund |
| `--gradient-card` | `linear-gradient(180deg, hsl(0 0% 100%) 0%, hsl(182 20% 98%) 100%)` | Karten-Hintergrund |
| `--gradient-dark` | `linear-gradient(135deg, hsl(182 100% 10%) 0%, hsl(182 80% 16%) 100%)` | CTA-Section (dunkel) |
| `--gradient-primary-ring` | `linear-gradient(135deg, hsl(343 100% 59%) 0%, hsl(343 70% 75%) 100%)` | Gradient-Border-Ringe |
| `--pattern-dot-grid` | `radial-gradient(circle, hsl(182 100% 12% / 0.07) 1px, transparent 1px)` | Atmosphärisches Hintergrundmuster |

### Schatten (Landingpage)

| Token | Wert |
|---|---|
| `--shadow-soft` | `0 4px 20px -4px hsl(182 30% 45% / 0.08)` |
| `--shadow-medium` | `0 8px 32px -8px hsl(182 30% 45% / 0.12)` |
| `--shadow-strong` | `0 16px 48px -12px hsl(182 30% 45% / 0.18)` |
| `--shadow-card-hover` | `0 12px 40px -8px hsl(343 100% 59% / 0.15)` |

> **Hinweis:** `--destructive` ist bewusst auf die Primary-Farbe gesetzt (bekannter Bug, noch nicht behoben).

---

## Typografie & Assets

| Eigenschaft | App | Landingpage |
|---|---|---|
| **Schriftart** | Inter Variable | Inter Variable |
| **Icon-Bibliothek** | Lucide React | Lucide React |
| **Radius** | `0.5rem` (8 px) | `1rem` (16 px) |
| **Logo (auf dunkel)** | — | `/Normdex_Logo_Horizontal_invert.svg` |
| **Sidebar-Breite (App)** | 20 rem / 4.5 rem (eingeklappt) | — |

---

## E-Mail-Farbschema (Outlook-kompatibel)

Für HTML-E-Mails wird ein vereinfachtes Subset der Markenpalette verwendet.

| Variable | Hex | Verwendung |
|---|---|---|
| `emailPrimary` | `#003c3e` | Haupt-Header, Buttons |
| `emailText` | `#282f3a` | Fließtext |
| `emailBg` | `#fafafa` | E-Mail-Hintergrund |
| `emailSurface` | `#ffffff` | Content-Container |
| `emailBorder` | `#e0e0e0` | Trennlinien |
| `emailLink` | `#003c3e` | Links (unterstrichen) |
| `emailAccent` | `#ff2d58` | Highlights, kritische Hinweise |

**Regeln für E-Mail-Templates:**
1. Immer **Inline-CSS** – keine externen Stylesheets
2. Immer **6-stellige Hex-Codes** – kein RGB/HSL (Kompatibilität)
3. Keine Gradienten oder Hintergrundbilder (Outlook-Einschränkungen)
4. Kein Dark-Mode-Targeting – hoher Kontrast im Light-Theme testen

---

## Verwandte Dokumente

- [[Brand Identity & Voice]]
- Farb-Tokens (JSON): `D:\Normdex\01_repos\normdex-app\docs\brand\colors.json`
