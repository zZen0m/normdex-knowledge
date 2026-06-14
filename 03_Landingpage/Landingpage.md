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
| `/newsletter` | Praxisleitfaden ÖNORM M 7140 – Normdex | `https://normdex.at/newsletter` |

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

## Newsletter / Lead-Magnet (Stand 2026-06-13, T027)

Der Hauptanreiz ist der **kostenlose Praxisleitfaden** (Checkliste + Beispielbericht), der 10%-Gutschein ist nur der Bonus.

- **Lead-Magnet:** Praxisleitfaden als PDF, direkter Download auf der Newsletter-Seite und im Erfolgs-State des Formulars. Dateipfad: `public/Normdex_Praxisleitfaden_OENORM_M7140.pdf` (vom Betreiber bereitzustellen).
- Frontend liefert nur UI und sendet die Anmeldung an `POST /newsletter/subscribe`.
- 10%-Gutschein wird weiter über die bestehende Brevo-Double-Opt-in-Strecke per E-Mail versendet (nicht beim Formular-Submit):
  1. Besucher meldet sich auf der Landingpage an und erhält den Leitfaden sofort als Download.
  2. Normdex API startet Brevo Double-Opt-in und speichert einen Pending-Claim.
  3. Besucher bestaetigt die Brevo Double-Opt-in-Mail.
  4. Brevo sendet `list_addition` an Normdex.
  5. Normdex erzeugt einen individuellen 10%-Promotion-Code und sendet ihn per E-Mail.
- Jeder Code ist individuell, einmalig einloesbar und 30 Tage gueltig.

---

## Homepage-Sektionen (Reihenfolge, Stand 2026-06-13, T027)

Reihenfolge in `Index.tsx`: Hero → TrustFactors → TargetAudience → ComparisonSection → NewsletterStrip → Features (`#features`) → HowItWorks (`#how`) → ReportPreview → Pricing (`#pricing`) → CTA.

> Positionierungs-Schwenk (T027): weg von „normkonform" als Hauptversprechen, hin zu „moderne, web-basierte Lösung" (im Browser, im Team, prüffähig). Normkonformität bleibt als Beleg. Verbindlich: kein Mitbewerber namentlich, Du-Form, echte Umlaute, keine Gedankenstriche als Satzzeichen.

### 1. Hero

- **Badge:** *gemäß ÖNORM M 7140* (animierter Puls-Dot, Link zu `/oenorm-m-7140`, pink)
- **H1:** „Im Browser. Im Team. Prüffähig." (pink) + „Wirtschaftlichkeitsberechnungen für Energiesysteme"
- **Unterüberschrift:** *Normdex ist die zeitgemäße Wirtschaftlichkeitsberechnung für Energiesysteme. Ohne Installation, überall im Browser, gemeinsam im Team und mit prüffähigen PDF-Reports. Validiert nach ÖNORM M 7140.*
- **CTAs:** „Kostenlos testen →" (primär, → `#pricing`) | „Beispielbericht ansehen" (sekundär, scrollt zu `#report-preview`)
- **Newsletter-Hook:** *„Kostenlosen Praxisleitfaden sichern — inkl. Checkliste und Beispielbericht."* (Link → `/newsletter`)
- **Trust-Indicators:** ✓ 14 Tage kostenlos beim Erstkauf einer Lizenz · ✓ Keine Abbuchung während der Testphase · ✓ Jederzeit kündbar
- **Validierungs-Badge:** 🛡 *Validiert nach Abschnitt 10 der ÖNORM M 7140*
- **Rechte Seite:** Komponente `HeroAppMock.tsx` — gecodeter Mock der App-Ergebnisseite (Browser-Chrome), Tab **Gesamtkosten** mit gestapeltem Balkendiagramm (Barwert je System: Kapital blau / Verbrauch grün / Betrieb gelb), Wärmepumpe als günstigste Variante markiert + Sieger-Banner. Werte konsistent zum Beispielbericht (WP 2,97 Mio. € < Pellet 3,85 Mio. € < Gas 7,13 Mio. €).
- **Floating-Badge:** „€ 4,16 Mio. günstiger als Gas"

### 2. TrustFactors

Kompakter horizontaler Strip. Kein Eyebrow-Titel. **Styling:** `py-7 bg-white border-y border-border/60`

| Icon | Titel | Sub |
|---|---|---|
| ShieldCheck | ÖNORM-validiert | Nach Abschnitt 10 |
| Globe | Ohne Installation | Direkt im Browser |
| Users | Im Team | Gemeinsame Projekte |
| Lock | Datenschutz | Server in der EU, DSGVO-konform |

### 3. TargetAudience

Strukturell unverändert. Durchgängig **Du-Form**. Icon-Boxen teal (`bg-secondary text-foreground`).

| Zielgruppe | Kernaussage (Du-Form) |
|---|---|
| Energieberater | Du berechnest Rentabilität von PV-Anlagen, Wärmepumpen und Speichern normkonform und dokumentierst die Ergebnisse direkt als professionellen PDF-Report. |
| Planungs- & Ingenieurbüros | Du erstellst Wirtschaftlichkeitsgutachten mit Firmenbranding und validierten Berechnungen für Auftraggeber und Ausschreibungsunterlagen. |
| Technische Sachverständige | Du erhältst nachvollziehbare, normkonforme Ergebnisse gemäß ÖNORM M 7140 für belastbare Gutachten mit vollständiger Dokumentation. |
| Privatpersonen | Du rechnest Sanierungsprojekte oder Energiesystemvergleiche strukturiert und korrekt, auch ohne Fachkenntnisse der Norm. |

### 4. ComparisonSection (NEU, T027)

Sachliche Kategorie-Gegenüberstellung **„klassische Desktop-Software" gegen „moderne Web-Lösung"**, ohne Firmennamen. Zwei Karten (klassisch gedämpft, modern hervorgehoben mit „Normdex"-Badge). Aspekte: Zugang (Installation gegen Browser), Zusammenarbeit (Einzelplatz gegen Team), Verfügbarkeit, Report (starrer Ausdruck gegen prüffähiges PDF), Updates. Komponente `ComparisonSection.tsx`.

### 5. NewsletterStrip (Lead-Magnet, T027)

Inline-Sektion. Hauptanreiz ist jetzt der kostenlose Praxisleitfaden, 10 % nur als Bonus.

- **Links:** EyebrowPill pink „Kostenloser Praxisleitfaden" + H3 „Praxisleitfaden zur ÖNORM M 7140 gratis holen." + Sub (Checkliste + Beispielbericht, plus 10 % Rabatt im ersten Monat).
- **Rechts:** Button „Leitfaden holen →" → `/newsletter`

### 6. Features

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

### 7. HowItWorks

4 Schritte mit nummerierten Teal-Pills und Verbindungslinien. **Du-Form.**

**H2:** In vier Schritten zur **Entscheidung**

| Schritt | Titel | Beschreibung |
|---|---|---|
| 1 | Projekt anlegen | Varianten definieren, Rahmenbedingungen festlegen. |
| 2 | Daten eingeben | Investitionen, Energiepreise, Nutzungsdauer — geführt und validiert. |
| 3 | Rechnen lassen | Kapitalwert, Annuität, Amortisation automatisch nach Norm. |
| 4 | Report exportieren | PDF-Dokumentation mit allen Annahmen und Rechenwegen. |

### 8. ReportPreview (NEU, T027)

Berichtsvorschau auf der Seite (kein Download). Galerie von sechs echten Berichtsseiten als anklickbare Thumbnails, die per Lightbox/Dialog im Browser geöffnet werden. Belegt die Output-Qualität (saubere Darstellung auch großer Beträge) und verweist für den Vollbericht auf den Newsletter-Praxisleitfaden. Komponente `ReportPreview.tsx`, Anker `#report-preview`. Bilder: `public/report-preview/seite-1..6.png` (Deckblatt, Projektdaten, Gesamtkosten, Annuitäten, Kostenverlauf/Amortisation, Resümee). Generiert aus dem kuratierten Beispielbericht via `apps/api/preview_report_demo.py` (Repo normdex-app).

### 9. CTA-Section

- **H2:** Bereit für **ÖNORM-konforme** Berechnungen?
- **Lead:** 14 Tage kostenlos beim Erstkauf einer Lizenz. Bei mehreren Lizenzen einmalig 24,50 € Erstbestellungsrabatt auf die Hauptlizenz.
- **Buttons:** „Jetzt kostenlos testen →" (primary, → `#pricing`) | „Kontakt aufnehmen" (ghost, → `/kontakt`)
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
| Anspruch | Unser Anspruch | Validierte Berechnungslogik nach Abschnitt 10 der ÖNORM M 7140 für belastbare Ergebnisse in professionellen Projektanwendungen. |
| Werte | Unsere Werte | Präzision, Nachvollziehbarkeit und Normkonformität stehen bei uns an erster Stelle, in jeder Berechnung und jedem Report. |

---

## Verwandte Dokumente

- [[Key Messages & CTAs]]
- [[Designsystem & Farben]]
- [[Unternehmensangaben]]
- [[T019-newsletter-gutschein-brevo-webhook-rollout]]
