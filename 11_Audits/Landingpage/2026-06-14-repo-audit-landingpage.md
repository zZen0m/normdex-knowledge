# Repo-Audit - Landingpage - 2026-06-14

## 1. Geprüfter Stand

- Repository: `D:\Normdex\01_repos\normdex-landingpage`
- Branch: `develop`
- Commit: `3478813`
- Version laut `package.json`: `0.0.4` (Release-Kandidat im Arbeitsbaum)
- Letzter Git-Tag: `v0.0.3`
- Audit-Datum: `2026-06-14`
- Speicherort: `D:\Normdex\02_knowledge\normdex-vault\11_Audits\Landingpage\2026-06-14-repo-audit-landingpage.md`
- Vorheriger Audit: `docs/repo-audits/landingpage-repo-audit-2026-05-09.md`

## 2. Audit-Abdeckung

- Statische Codeanalyse der vollständigen uncommitteten T027-Änderung
- Prüfung von Homepage, neuen Komponenten, Newsletter-Flow, Berichts-Lightbox und ÖNORM-Seite
- Routen-, SEO-, Accessibility- und Conversion-Prüfung
- Abgleich mit dem Vorbericht vom 2026-05-09
- Produktionsbuild mit Vite 8
- ESLint-Prüfung
- Dependency-Audit mit `npm audit`
- Lokaler HTTP-Smoke-Test für Homepage, Newsletter, ÖNORM-Seite, Vorschau-PNGs und Praxisleitfaden-PDF
- Kein visueller Browser-Test möglich, da der In-App-Browser in dieser Sitzung nicht verfügbar war
- Kein echter Newsletter-Submit ausgeführt, um keine Testdaten an den produktiven API-Flow zu senden

## 3. Fortschritt seit letztem Audit

- Vorheriger Audit: 2026-05-09
- Findings im Vorbericht: 8
- Davon behoben: 7
- Davon weiterhin offen: 1
- Davon nicht abschließend verifizierbar: 0
- Regressionen: 0

### Behobene Findings

- Finding A - Newsletter-Opt-in im Kontaktformular: behoben (`newsletter` wird im Request übertragen).
- Finding B - SEO-Metadaten nicht durchgängig: behoben (Helmet und Canonical auf den betroffenen Routen vorhanden).
- Finding C - Großes Initial-Bundle: behoben (Routen werden mit `React.lazy` geladen; Build erzeugt getrennte Route-Chunks).
- Finding D - Footer-Navigation mit harten Reloads: behoben (interne Ziele verwenden `Link`).
- Finding F - `FORMSPREE_ID` Dead Code: behoben.
- Finding G - Siezen in `Cookies.tsx`: behoben.
- Finding H - `/newsletter` fehlt in Sitemap: behoben.

### Weiterhin offene Findings

- Finding E - TypeScript-Strictness: teilweise verbessert, aber `strict` und `noImplicitAny` bleiben deaktiviert.

### Regressionen

- Keine Regressionen festgestellt.

## 4. Confidence

- Confidence: hoch

Die statischen Befunde sind direkt im Repository belegt. Build, Lint, Dependency-Audit und lokale HTTP-Aufrufe wurden erfolgreich ausgeführt. Lediglich die visuelle Prüfung im echten Browser und ein produktiver Formular-Submit blieben aus.

## 5. Kurzfazit

Der Release-Kandidat `0.0.4` setzt die neue Marketingpositionierung konsistent um und ergänzt belastbare Conversion-Elemente: App-Mock, Vergleichssektion, Berichtsvorschau und Praxisleitfaden. Während des Audits wurden ein neues Lint-Warning, fehlende Fokuszustände, eine inkonsistente CTA-Botschaft sowie zwei moderate React-Router-Schwachstellen behoben. Zusätzlich wurde die nicht benötigte und verwundbare `lovable-tagger`-Entwicklungsabhängigkeit entfernt und der React-SWC-Pluginstand mit Vite 8 kompatibel gemacht. `npm audit` meldet 0 Schwachstellen, der Produktionsbuild ist erfolgreich und alle geprüften lokalen URLs liefern HTTP 200. Der Trend gegenüber dem Vorbericht ist klar positiv. Es bestehen keine kritischen oder hohen Release-Blocker.

## 6. Wichtigste Findings

1. **[mittel][Improvement] TypeScript-Strictness bleibt teilweise deaktiviert.**
2. **[mittel][Risk] Es gibt keine automatisierten Komponenten- oder Flow-Tests.**
3. **[niedrig][Risk] Beispielberichtswerte sind im Hero-Mock manuell dupliziert.**
4. **[niedrig][Improvement] Sieben bestehende Fast-Refresh-Lint-Warnungen in shadcn-Basiskomponenten.**

## 7. Detaillierte Findings je Punkt

### Finding 1 - TypeScript-Strictness bleibt teilweise deaktiviert

- Kategorie: Improvement
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: persistent (seit 2026-04-08)
- Betroffene Datei(en) oder Pfade: `tsconfig.app.json`
- Evidenz: `strict: false` und `noImplicitAny: false`; `noUnusedLocals` und `noUnusedParameters` sind inzwischen aktiviert.
- Beschreibung des Problems: Wichtige Typprüfungen für Nullwerte und implizite `any`-Typen greifen weiterhin nicht.
- Warum das relevant ist: Formular-, Consent- und Routing-Code kann bei Refactorings Fehler enthalten, die der Compiler nicht erkennt.
- Business Impact: Erhöhtes Regressionsrisiko und langfristig höhere Wartungskosten.
- Konkrete Handlungsempfehlung: Zuerst `strictNullChecks`, danach `noImplicitAny` und abschließend `strict` in getrennten, kleinen Änderungen aktivieren.

### Finding 2 - Keine automatisierten Komponenten- oder Flow-Tests

- Kategorie: Risk
- Priorität: mittel
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `package.json`, `src/`
- Evidenz: Es existiert kein Test-Script und keine erkennbare Test-Suite für Newsletter, Pricing, Anker-Navigation oder Lightbox.
- Beschreibung des Problems: Conversion-relevante Flows werden aktuell nur über Build, Lint und manuelle Prüfung abgesichert.
- Warum das relevant ist: Änderungen an Router, Dialog, Formularstatus oder Pricing können unbemerkt funktionale Regressionen verursachen.
- Business Impact: Potenziell verlorene Leads oder fehlerhafte Kaufweiterleitungen nach späteren Änderungen.
- Konkrete Handlungsempfehlung: Kleine Vitest-/Testing-Library-Suite für Newsletter-Erfolgsstate, Report-Lightbox und Pricing-URL ergänzen.

### Finding 3 - Beispielberichtswerte im Hero-Mock manuell dupliziert

- Kategorie: Risk
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `src/components/HeroAppMock.tsx`, `public/report-preview/`
- Evidenz: Kostenwerte und Vergleichsbetrag sind als Konstanten im TSX-Code hinterlegt und stammen aus einem separat generierten Bericht.
- Beschreibung des Problems: Bei einer späteren Neugenerierung des Beispielberichts können Hero-Grafik, Screenshots und PDF auseinanderlaufen.
- Warum das relevant ist: Widersprüchliche Zahlen schwächen das zentrale Vertrauenssignal der Landingpage.
- Business Impact: Glaubwürdigkeits- und Conversion-Risiko bei inkonsistenten Beispielwerten.
- Konkrete Handlungsempfehlung: Beispieldaten künftig aus einer gemeinsamen JSON-Datei generieren oder beim Erstellen der Screenshots automatisiert abgleichen.

### Finding 4 - Bestehende Fast-Refresh-Lint-Warnungen

- Kategorie: Improvement
- Priorität: niedrig
- Verifizierungsstatus: statisch verifiziert
- Kontinuität: neu
- Betroffene Datei(en) oder Pfade: `src/components/ui/badge.tsx`, `button.tsx`, `form.tsx`, `navigation-menu.tsx`, `sidebar.tsx`, `sonner.tsx`, `toggle.tsx`
- Evidenz: ESLint meldet sieben `react-refresh/only-export-components`-Warnungen, aber keine Fehler.
- Beschreibung des Problems: Komponenten und Hilfsexporte liegen in denselben shadcn-Dateien.
- Warum das relevant ist: Fast Refresh kann während der Entwicklung für diese Module weniger zuverlässig sein.
- Business Impact: Kein Produktionsrisiko; geringe Reibung im Entwicklungsprozess.
- Konkrete Handlungsempfehlung: Warnungen bei einer späteren shadcn-Aktualisierung bereinigen oder gezielt für unveränderte Basiskomponenten konfigurieren.

## 8. Quick Wins

- Einen minimalen Lightbox-Test für Öffnen, Vor/Zurück und Schließen ergänzen.
- Die Hero-Beispieldaten in eine eigene, gemeinsam nutzbare Datendatei verschieben.
- Browserslist-Daten im nächsten Dependency-Wartungslauf aktualisieren.

## 9. Strategische Empfehlungen

- Conversion-Flows mit einer kleinen automatisierten Regressionstest-Suite absichern.
- TypeScript schrittweise auf strikten Modus umstellen.
- Generierte Marketing-Assets und die dazugehörigen Zahlen aus einer gemeinsamen Quelle ableiten.

## 10. Empfohlene nächste Aktion

Version `0.0.4` veröffentlichen und die automatisierten Conversion-Tests im nächsten Sprint einplanen.

## 11. Offene Unsicherheiten / Punkte zur manuellen Prüfung

- Visuelle Prüfung der neuen Homepage-Sektionen auf echten Desktop- und Mobil-Viewports.
- Reale Newsletter-Anmeldung inklusive Brevo-Double-Opt-in und Gutscheinversand.
- Downloadverhalten des Praxisleitfadens auf Safari/iOS.
- Farbkontrast und Lesbarkeit der kleinsten Texte im Hero-App-Mock auf kleinen Displays.
