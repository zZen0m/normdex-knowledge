# T051 · Beispielbericht öffentlich downloadbar machen

**Phase:** Landingpage / Marketing / Conversion
**Priorität:** P1 · Conversion – Trust-Signal
**Status:** offen
**Datum:** 2026-07-24

## Ziel

Den neu erstellten Beispielbericht (`04_Marketing/Beispielbericht.pdf` im Vault) als eigenständige, öffentlich und ohne Anmeldung herunterladbare PDF auf der Landingpage bereitstellen. Ersetzt die aktuell falsche Behauptung, der vollständige Bericht sei Teil des Praxisleitfaden-Downloads.

## Kontext

Laut [[Marketingplan 2026 - Erste Kunden]] (Abschnitt "Conversion-Fläche") ist der öffentliche Beispiel-Report als PDF ohne Anmeldung explizit als eigenständiges, vom Newsletter/Lead-Magnet getrenntes Trust-Signal vorgesehen: *"Das ist das stärkste Trust-Signal ganz ohne persönliches Auftreten."* Der Punkt stand in der Phase-1-Checkliste bisher offen ("Status bei letztem Stand nicht bestätigt").

Aktueller Implementierungsstand im Repo `normdex-landingpage`:
- `src/components/ReportPreview.tsx` zeigt bereits 6 Vorschaubilder (`public/report-preview/seite-1.png` bis `seite-6.png`) in einer Lightbox, frei sichtbar, kein Download.
- Der Text darunter behauptet aktuell: *"Den vollständigen Bericht als PDF bekommst du kostenlos über unseren Praxisleitfaden."* Das stimmt nicht: `LEAD_MAGNET_URL` (`src/components/NewsletterForm.tsx`) verweist ausschließlich auf `public/Normdex_Praxisleitfaden_OENORM_M7140.pdf`, eine eigene Beispielbericht-PDF gibt es im Download-Flow bisher nicht.
- Mehrere Stellen bewerben den Praxisleitfaden fälschlich als Bundle "Checkliste + Beispielbericht": `src/components/Hero.tsx:90`, `src/components/NewsletterStrip.tsx:31`, sowie `src/pages/Newsletter.tsx` (Beschreibungstext) und die Doku in [[Landingpage]] (Abschnitte "Newsletter / Lead-Magnet" und "NewsletterStrip").

Klärung mit Andreas (2026-07-24):
- Bericht ist komplett fiktiv/anonymisiert, keine echten Kundendaten → unbedenklich für öffentliche Veröffentlichung.
- Download soll ungated sein, nicht über den Newsletter-Flow.
- Download-Link/Button ausschließlich unter der ReportPreview-Sektion, kein zusätzlicher Button in der Lightbox, Hero-CTA "Beispielbericht ansehen" bleibt unverändert (scrollt weiterhin zur Sektion, lädt nicht direkt herunter).

## Umsetzung

### 1. PDF ins Repo bringen
- `04_Marketing/Beispielbericht.pdf` aus dem Vault nach `normdex-landingpage/public/Normdex_Beispielbericht_OENORM_M7140.pdf` kopieren (Namensschema konsistent zu `Normdex_Praxisleitfaden_OENORM_M7140.pdf`).

### 2. `ReportPreview.tsx` anpassen
- Den Satz "Den vollständigen Bericht als PDF bekommst du kostenlos über unseren Praxisleitfaden." ersetzen durch einen direkten Download-Button/Link auf `/Normdex_Beispielbericht_OENORM_M7140.pdf` (mit `download`-Attribut).
- Bestehenden Link zum Praxisleitfaden (`/newsletter/`) an dieser Stelle entfernen, da nicht mehr zutreffend.

### 3. Bundle-Sprache korrigieren (Beispielbericht ≠ Teil des Leitfadens)
- `src/components/Hero.tsx:90`: "inkl. Checkliste und Beispielbericht" → nur noch Checkliste erwähnen, Beispielbericht separat referenzieren oder Satz streichen.
- `src/components/NewsletterStrip.tsx:31`: "Checkliste und Beispielbericht, direkt als PDF" → analog anpassen.
- `src/pages/Newsletter.tsx`: Beschreibungstext ("kompakt erklärt, mit Checkliste und Beispielbericht") entsprechend korrigieren, ggf. mit Verweis auf die öffentliche Beispielbericht-Seite/-Sektion.

### 4. SEO
- Neue URL `/Normdex_Beispielbericht_OENORM_M7140.pdf` in `public/sitemap.xml` aufnehmen.

### 5. Doku aktualisieren
- [[Landingpage]]: Abschnitt 8 (ReportPreview) und Abschnitt "Newsletter / Lead-Magnet" korrigieren, damit sie den neuen, ungated Download korrekt beschreiben statt der bisherigen Bundle-Behauptung.
- [[Marketingplan 2026 - Erste Kunden]]: Phase-1-Checkliste-Punkt "Beispiel-Report als öffentliches PDF erstellen. (Status bei letztem Stand nicht bestätigt)" auf erledigt setzen.

## Akzeptanzkriterien

- [ ] `Normdex_Beispielbericht_OENORM_M7140.pdf` liegt in `public/` und ist über einen direkten Link/Button erreichbar, ohne Anmeldung oder Weiterleitung zum Newsletter.
- [ ] Download-Button erscheint ausschließlich unter der ReportPreview-Sektion (kein zusätzlicher Button in der Lightbox, Hero-CTA unverändert).
- [ ] Kein Text auf der Landingpage behauptet mehr, der Beispielbericht sei Teil des Praxisleitfaden-Downloads.
- [ ] `/Normdex_Beispielbericht_OENORM_M7140.pdf` ist in `sitemap.xml` gelistet.
- [ ] [[Landingpage]] und [[Marketingplan 2026 - Erste Kunden]] spiegeln den neuen Stand korrekt wider.

## Verifikation

- Lokaler Build (`npm run build` in `normdex-landingpage`) enthält die PDF unter `dist/`.
- Manueller Klick-Test: von der Landingpage aus lässt sich der Beispielbericht ohne Anmeldung herunterladen.
- `sitemap.xml` im Build enthält die neue PDF-URL.

## Notizen / Fortschritt

- 2026-07-24: Todo angelegt nach Grill-Session mit Andreas. Hintergrund: neuer Beispielbericht wurde erstellt, bisherige Landingpage-Umsetzung behauptete fälschlich, er sei Teil des Praxisleitfadens, obwohl laut Marketingplan ein eigenständiges, öffentliches Trust-Signal vorgesehen war.
