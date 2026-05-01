# Landingpage

**URL:** `https://normdex.at`

## Zweck

Die Normdex-Landingpage ist die öffentliche Marketing-Website mit folgenden Zielen:

1. **Informieren:** Was ist Normdex? Wofür ist es gut?
2. **Überzeugen:** Warum ÖNORM-konform, validiert, professionell?
3. **Konvertieren:** Besucher zu Trial-Registrierungen führen
4. **SEO:** Bei relevanten Suchanfragen auffindbar sein

**Kein Dark Mode** – ausschließlich Light Mode.

---

## Tech Stack

| Technologie | Zweck |
|---|---|
| React 18 | UI Framework |
| TypeScript | Typsicherheit |
| Vite | Build-Tool |
| React Router v6 | Client-seitiges Routing |
| Tailwind CSS v3 | Styling |
| Radix UI / shadcn/ui | UI-Komponenten |
| React Helmet Async | SEO (Title, Meta, Canonical) |
| Sonner | Toast-Notifications |
| Google reCAPTCHA v2 | Spam-Schutz Kontaktformular |
| Lucide React | Icons |
| Inter Variable | Schriftart |

**Dev-Server:** Port 8081 (`npm run dev`) | **Paketmanager:** npm

---

## Routen & SEO

| Pfad | SEO-Titel | Canonical |
|---|---|---|
| `/` | Normdex – ÖNORM M 7140 Wirtschaftlichkeitsberechnung \| Permatec e.U. | `https://normdex.at/` |
| `/preise` | Preise – Normdex Wirtschaftlichkeitsberechnung \| ÖNORM M 7140 | `https://normdex.at/preise` |
| `/features` | Funktionen – Normdex Wirtschaftlichkeitsberechnung für Energiesysteme | `https://normdex.at/features` |
| `/oenorm-m-7140` | ÖNORM M 7140 – Wirtschaftlichkeitsberechnung für Energiesysteme \| Normdex | `https://normdex.at/oenorm-m-7140` |
| `/ueber-uns` | Über Normdex – Berechnungsplattform für Energiesysteme \| Permatec e.U. | `https://normdex.at/ueber-uns` |
| `/kontakt` | Kontakt – Normdex Support \| Permatec e.U. | `https://normdex.at/kontakt` |
| `/impressum` | Impressum – Normdex \| Permatec e.U. | `https://normdex.at/impressum` |
| `/datenschutz` | Datenschutzerklärung – Normdex \| Permatec e.U. | `https://normdex.at/datenschutz` |
| `/agb` | AGB – Allgemeine Geschäftsbedingungen \| Normdex | `https://normdex.at/agb` |
| `/newsletter` | Newsletter & Updates – Normdex | `https://normdex.at/newsletter` |

---

## Header & Footer

### Header (Stand 2026-05-01)

- **Styling:** Helles Glas-Header — `bg-white/85 backdrop-blur-md border-b border-border/60`, sticky, `h-[72px]`
- **Logo:** `/Normdex_Logo_Horizontal.svg` (nicht invertiert, h-8)
- **Navigation:** Features · So funktioniert's (`/#how`) · Preise · ÖNORM M 7140
- **Buttons:** „Anmelden" (Ghost) → `https://app.normdex.at/auth/login` | „Kostenlos testen →" (Primary) → `https://app.normdex.at/auth/register`
- **Mobile Sheet:** Dunkler Teal-Hintergrund (`bg-foreground text-background`)

### Footer

**Beschreibungstext:**
> *Professionelle ÖNORM M 7140 Wirtschaftlichkeitsberechnung für technische Investitionen und Anlagen. Normdex™ wurde in Österreich für österreichische Anforderungen entwickelt.*

- **Kontakt:** `office@normdex.at` | 8600 Bruck an der Mur, Österreich
- **Link-Gruppen:** Produkt (Features, Preise) / Unternehmen (Über uns, Kontakt) / Legal (Datenschutz, AGB, Impressum, Cookies, Cookie-Einstellungen)
- **Copyright:** `© 2026 Permatec e.U. Alle Rechte vorbehalten.`

---

## Newsletter-Gutschein

- Landingpage kommuniziert den 10%-Gutschein fuer Newsletter-Anmeldung.
- Frontend liefert nur UI und sendet die Anmeldung an `POST /newsletter/subscribe`.
- Gutschein wird nicht direkt beim Formular-Submit versendet.
- Ablauf:
  1. Besucher meldet sich auf der Landingpage zum Newsletter an.
  2. Normdex API startet Brevo Double-Opt-in und speichert einen Pending-Claim.
  3. Besucher bestaetigt die Brevo Double-Opt-in-Mail.
  4. Brevo sendet `list_addition` an Normdex.
  5. Normdex erzeugt einen individuellen 10%-Promotion-Code und sendet ihn per E-Mail.
- Jeder Code ist individuell, einmalig einloesbar und 30 Tage gueltig.

---

## Homepage-Sektionen (Reihenfolge, Stand 2026-05-01)

### 1. Hero

- **Badge:** *gemäß ÖNORM M 7140* (animierter Puls-Dot, Link zu `/oenorm-m-7140`, pink)
- **H1:** ÖNORM-konforme Wirtschaftlichkeitsberechnungen für Energiesysteme
- **Unterüberschrift:** *Normdex unterstützt die strukturierte Durchführung betriebswirtschaftlicher Vergleichsrechnungen für Energiesysteme — nachvollziehbar, validiert und auf professionelle Projektanwendungen ausgerichtet.*
- **CTAs:** „Kostenlos testen →" (primär) | „Features ansehen" (sekundär, → `/features`)
- **Newsletter-Hook:** *„Newsletter abonnieren — 10 % Rabatt im ersten Monat sichern."* (Link → `/newsletter`, zwischen CTAs und Trust-Indicators)
- **Trust-Indicators:** ✓ 14 Tage kostenlos · ✓ Keine Kreditkarte erforderlich · ✓ Jederzeit kündbar
- **Validierungs-Badge:** 🛡 *Validiert nach Abschnitt 10 der ÖNORM M 7140*
- **Rechte Seite:** Coded Mock-Dashboard (Browser-Chrome + Sidebar + 3 KPI-Karten + Balkendiagramm) — kein Foto
- **Floating Cards:** Variantenvergleich / Beispiel Kapitalwert €45.200,-

### 2. TrustFactors

Kompakter horizontaler Strip. Kein Eyebrow-Titel. **Styling:** `py-7 bg-white border-y border-border/60`

| Icon | Titel | Sub |
|---|---|---|
| ShieldCheck | ÖNORM-validiert | Nach Abschnitt 10 |
| BookOpen | Nachvollziehbar | Transparente Rechenwege |
| FileSearch | Prüfbar | Vollständige Dokumentation |
| Lock | Datenschutz | DSGVO-konform, AT-Server |

### 3. TargetAudience

Strukturell unverändert. Durchgängig **Du-Form**. Icon-Boxen teal (`bg-secondary text-foreground`).

| Zielgruppe | Kernaussage (Du-Form) |
|---|---|
| Energieberater | Du berechnest Rentabilität von PV-Anlagen, Wärmepumpen und Speichern normkonform — und dokumentierst die Ergebnisse als professionellen PDF-Report. |
| Planungs- & Ingenieurbüros | Du erstellst Wirtschaftlichkeitsgutachten mit Firmenbranding und validierten Berechnungen — für Auftraggeber und Ausschreibungsunterlagen. |
| Technische Sachverständige | Du erhältst nachvollziehbare, normkonforme Ergebnisse gemäß ÖNORM M 7140 — für belastbare Gutachten mit vollständiger Dokumentation. |
| Privatpersonen | Du rechnest Sanierungsprojekte oder Energiesystemvergleiche strukturiert und korrekt — ohne Fachkenntnisse der Norm. |

### 4. NewsletterStrip (NEU, 2026-05-01)

Inline-Sektion zwischen TargetAudience und Features. Primärer Newsletter-Touchpoint mit 10%-Hook.

- **Links:** EyebrowPill pink „10 % Rabatt" + H3 „Newsletter abonnieren — 10 % im ersten Monat sichern." + Sub „Code per Mail nach Bestätigung. Einlösbar einmalig im Stripe-Checkout."
- **Rechts:** Button „Jetzt 10 % sichern →" → `/newsletter`

### 5. Features

Gleichmäßiges 3-Spalten-Grid (6 Karten, alle gleich groß). Durchgängig **Du-Form**. Icon-Boxen teal.

**H2:** Was Normdex **für dich leistet**

| Nr. | Titel | Beschreibung |
|---|---|---|
| 01 | Alle drei Verfahren | Kapitalwert-, Annuitäten- und Amortisationsmethode — nach ÖNORM M 7140 implementiert. |
| 02 | Varianten vergleichen | Mehrere Energiesysteme nebeneinander rechnen und die wirtschaftlichste Option identifizieren. |
| 03 | Dynamische Auswertung | Kapitalwert-, Cashflow- und Amortisationsdiagramme aus deinen Projektdaten. |
| 04 | PDF-Report | Prüfungsfähige Dokumentation mit allen Annahmen und Rechenwegen — nach Norm-Vorgabe. |
| 05 | Sensitivitätsanalyse | Energiepreise, Zinsen, Nutzungsdauer flexibel variieren. |
| 06 | Team-Workspace | Mehrere Nutzer:innen in einer Organisation, gemeinsame Projekte, Rollenverwaltung. |

### 6. HowItWorks

4 Schritte mit nummerierten Teal-Pills und Verbindungslinien. **Du-Form.**

**H2:** In vier Schritten zur **Entscheidung**

| Schritt | Titel | Beschreibung |
|---|---|---|
| 1 | Projekt anlegen | Varianten definieren, Rahmenbedingungen festlegen. |
| 2 | Daten eingeben | Investitionen, Energiepreise, Nutzungsdauer — geführt und validiert. |
| 3 | Rechnen lassen | Kapitalwert, Annuität, Amortisation automatisch nach Norm. |
| 4 | Report exportieren | PDF-Dokumentation mit allen Annahmen und Rechenwegen. |

### 7. CTA-Section

- **H2:** Bereit für **ÖNORM-konforme** Berechnungen?
- **Lead:** 14 Tage kostenlos — keine Kreditkarte erforderlich.
- **Buttons:** „Jetzt kostenlos testen →" (primary, → `https://app.normdex.at/auth/register`) | „Kontakt aufnehmen" (ghost, → `/kontakt`)
- **Newsletter-Hook:** *„Lieber erst informieren? Newsletter abonnieren und 10 % Rabatt im ersten Monat sichern."*

---

## Preise (`/preise`)

### Preismodell

| Plan | Preis | Ab 2. Lizenz | Besonderheit |
|---|---|---|---|
| **Monatlich** | 49 €/Monat | 29 €/Monat | Maximale Flexibilität — ideal für Projektphasen |
| **Jährlich** | 490 €/Jahr | 290 €/Jahr | −17% ggü. monatlich, entspricht 40,83 €/Monat |

**Enthaltene Funktionen (beide Pläne):** ÖNORM-konforme Wirtschaftlichkeitsberechnung / Automatisierte Berechnungen und Reports / Datenverwaltung und Export-Funktionen / E-Mail Support / Regelmäßige Updates / Sichere Cloud-Speicherung / Multi-Projekt-Verwaltung / PDF-Export der Berechnungen

### UX-Konzept der Preisseite (Stand 2026-05-01)

Die Preisseite setzt auf ein **Hybrid-Modell**: Plan-Karten zum Vergleich + dauerhaft sichtbarer „Deine Auswahl"-Panel.

**Plan-Karten (klickbare Radio-Flächen):**
- Monatlich / Jährlich als nebeneinander liegende Karten
- Klick auf eine Karte → Karte wird aktiv (Ring + Shadow), andere wird inaktiv
- Yearly-Karte hat „Beliebteste Wahl"-Badge und „17% günstiger"-Chip mit TrendingDown-Icon
- Monatlich-Karte zeigt: „Mit dem Jahresplan sparst du 98€/Jahr pro Lizenz"
- Am Kartenende: ÖNORM-Validierungs-Badge (ShieldCheck + „Validiert gemäß Abschnitt 10")
- Kein Mengenregler in den Karten (wurde entfernt)

**„Deine Auswahl"-Panel (immer sichtbar, unterhalb der Karten):**
- Trial-Banner: pulsierender Dot + „14 Tage kostenlos testen — keine Abbuchung während der Testphase"
- Tabs-Toggle: „Monatlich" / „Jährlich — 17% sparen" (synchronisiert mit Kartenwahl)
- Mengenregler: große Buttons (h-10 w-10), Anzeige: Zahl + „Lizenz/en", Minimum 1, Maximum 20
- Live-Preisanzeige: Gesamtbetrag + bei Jährlich: „entspricht X€/Monat im Schnitt"
- Ersparnis-Chip (nur bei Jährlich): „Du sparst X€/Monat gegenüber dem Monatsabo" (grün)
- CTA: „Jetzt kostenlos starten" (volle Breite, immer sichtbar)
- Trust-Zeile: 🔒 Kein Risiko · Jederzeit kündbar · Deine Karte wird erst nach 14 Tagen belastet

**Checkout-URL-Schema (via T018):**
- `https://app.normdex.at/auth/register?plan=yearly&qty=2`
- `https://app.normdex.at/auth/register?plan=monthly&qty=1`

**Sprache:** Durchgängig Du-Form (gemäß Brand Identity & Voice)

### FAQ auf /preise

1. Was passiert nach den 14 Tagen?
2. Kann ich jederzeit wechseln?
3. Gibt es eine kostenlose Testphase?
4. Wie funktioniert die Abrechnung?
5. Kann ich jederzeit kündigen?
6. Wie funktioniert das Lizenzsystem?

---

## Roadmap (öffentlich kommuniziert)

| Status | Feature |
|---|---|
| Geplant | Erweiterte Export-Schnittstellen (GAEB, Excel) |
| Geplant | Integrierte CO2-Bilanzierung |
| Geplant | Team-Collaboration Features |

---

## Über uns (`/ueber-uns`)

| Card | Titel | Text |
|---|---|---|
| Mission | Unsere Mission | Wirtschaftlichkeitsberechnungen für Energiesysteme strukturiert, nachvollziehbar und normkonform gemäß ÖNORM M 7140 durchführbar zu machen. |
| Anspruch | Unser Anspruch | Validierte Berechnungslogik nach Abschnitt 10 der ÖNORM M 7140 — für belastbare Ergebnisse in professionellen Projektanwendungen. |
| Werte | Unsere Werte | Präzision, Nachvollziehbarkeit und Normkonformität stehen bei uns an erster Stelle — in jeder Berechnung und jedem Report. |

---

## Verwandte Dokumente

- [[Key Messages & CTAs]]
- [[Designsystem & Farben]]
- [[Unternehmensangaben]]
- [[T019-newsletter-gutschein-brevo-webhook-rollout]]
