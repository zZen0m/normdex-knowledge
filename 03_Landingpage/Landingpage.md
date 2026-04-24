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

---

## Header & Footer

### Header

- **Styling:** Dunkler Teal-Hintergrund (`bg-foreground`), heller Text, sticky
- **Logo:** `/Normdex_Logo_Horizontal_invert.svg` (h-12)
- **Navigation:** Features / Preise / Über uns / Kontakt
- **Buttons:** „Anmelden" (Ghost) → `https://app.normdex.at/auth/login` | „Registrieren" (Primary) → `https://app.normdex.at/auth/register`

### Footer

**Beschreibungstext:**
> *Professionelle ÖNORM M 7140 Wirtschaftlichkeitsberechnung für technische Investitionen und Anlagen. Normdex™ wurde in Österreich für österreichische Anforderungen entwickelt.*

- **Kontakt:** `office@normdex.at` | 8600 Bruck an der Mur, Österreich
- **Link-Gruppen:** Produkt (Features, Preise) / Unternehmen (Über uns, Kontakt) / Legal (Datenschutz, AGB, Impressum, Cookies, Cookie-Einstellungen)
- **Copyright:** `© 2026 Permatec e.U. Alle Rechte vorbehalten.`

---

## Homepage-Sektionen (Reihenfolge)

### 1. Hero

- **Badge:** *gemäß ÖNORM M 7140* (animierter Puls-Dot, Link zu `/oenorm-m-7140`)
- **H1:** ÖNORM-konforme Wirtschaftlichkeitsberechnungen für Energiesysteme
- **Unterüberschrift:** *Normdex unterstützt die strukturierte Durchführung betriebswirtschaftlicher Vergleichsrechnungen für Energiesysteme — nachvollziehbar, validiert und auf professionelle Projektanwendungen ausgerichtet.*
- **CTAs:** „Kostenlos testen" (primär) | „Features ansehen" (sekundär)
- **Trust-Indicators:** ✓ Keine Kreditkarte | ✓ 14 Tage kostenlos
- **Validierungs-Badge:** 🛡 *Validiert nach Abschnitt 10 der ÖNORM M 7140*
- **Floating Cards:** Variantenvergleich / Beispiel Kapitalwert €45.200,-

### 2. TrustFactors

**Eyebrow:** „Verlässliche Basis"

| Icon | Titel | Beschreibung |
|---|---|---|
| ShieldCheck | Validiert nach Abschnitt 10 | Die Berechnungsergebnisse werden anhand der Validierungsvorgaben in Abschnitt 10 der ÖNORM M 7140 geprüft und bestätigt. |
| BookOpen | Gemäß ÖNORM M 7140 | Alle Kalkulationsverfahren — Kapitalwertmethode, Annuitätenmethode, Amortisationsrechnung — folgen den definierten Berechnungsvorschriften der Norm. |
| FileSearch | Nachvollziehbare Ergebnisdokumentation | Jede Berechnung wird vollständig dokumentiert: Eingabeparameter, Zwischenwerte und Ergebnisse sind transparent und reproduzierbar dargestellt. |

### 3. Features

**H2:** Was Normdex für Ihre **Wirtschaftlichkeitsberechnung** leistet

| Nr. | Titel | Beschreibung |
|---|---|---|
| 01 | Alle drei Berechnungsverfahren | Kapitalwert, Annuität und Amortisation sind vollständig implementiert — inklusive dynamischer Preisentwicklung und Kalkulationszinssatz. |
| 02 | Strukturierter Variantenvergleich | Vergleichen Sie beliebig viele Energiesystem-Varianten direkt nebeneinander — mit einheitlicher Kostenstruktur und normkonformer Bewertung. |
| 03 | Sensitivitätsanalyse | Untersuchen Sie die Auswirkungen veränderter Energiepreise und Zinssätze auf Ihre Ergebnisse — systematisch und reproduzierbar. |
| 04 | Normkonforme PDF-Reports | Erstellen Sie auf Knopfdruck vollständige Berechnungsberichte mit allen Eingabeparametern, Zwischenergebnissen und Kennzahlen. |
| 05 | Gesamtkostenübersicht auf einen Blick | Visualisieren Sie kapital-, betriebs- und verbrauchsgebundene Kosten über die gesamte Nutzungsdauer — aufgeschlüsselt und vergleichbar. |
| 06 | Validierte Berechnungslogik | Die Berechnungsergebnisse werden gemäß Abschnitt 10 der ÖNORM M 7140 validiert — für nachvollziehbare und belastbare Projektergebnisse. |

### 4. HowItWorks

**H2:** Einwandfrei in 3 Schritten starten

| Schritt | Titel | Beschreibung |
|---|---|---|
| 1 | Einfacher Projektstart | Legen Sie Ihr erstes Projekt an. Normdex führt Sie durch die notwendigen Basisdaten und gibt typische Kalkulationszinssätze der ÖNORM M 7140 vor. |
| 2 | Beliebig viele Varianten | Ergänzen Sie strukturierte Kosten (Investition, Betrieb, Verbrauch) pro Variante. Normdex berechnet Kapitalwert, Annuität und Amortisation automatisch im Hintergrund. |
| 3 | Fertiger Report | Ein Klick genügt und Sie erhalten einen vollständig formatierten PDF-Report für Ihre Auftraggeber oder Geschäftsführung. |

### 5. CTA-Section

- **H2:** Bereit für normkonforme Wirtschaftlichkeitsberechnungen?
- **Benefits:** 14 Tage kostenlos / Keine Kreditkarte / Vollzugriff / Kostenloser Support
- **Button:** „Jetzt kostenlos testen" → `https://app.normdex.at/auth/register`

---

## Preise (`/preise`)

| Plan | Preis | Besonderheit |
|---|---|---|
| **Monatlich** | 49 €/Monat/Benutzer | Maximale Flexibilität, monatlich kündbar |
| **Jährlich** | 490 €/Jahr/Benutzer (= 40,83 €/Monat) | -17%, 2 Monate geschenkt |

**Enthaltene Funktionen (beide Pläne):** ÖNORM-konforme Wirtschaftlichkeitsberechnung / Automatisierte Berechnungen und Reports / Datenverwaltung und Export-Funktionen / E-Mail Support / Regelmäßige Updates / Sichere Cloud-Speicherung / Multi-Projekt-Verwaltung / PDF-Export der Berechnungen

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
