# T050 · GA4 erfasst seit Consent-Fix fast keinen Traffic mehr

**Phase:** Landingpage / Analytics / Datenschutz
**Priorität:** P3 · Datenlage beeinträchtigt, kein Produktionsbug
**Status:** offen
**Datum:** 2026-07-23
**Zuletzt aktualisiert:** 2026-07-23

## Ziel

Klären, ob und wie die GA4-Datenerfassung auf der Landingpage verbessert werden soll, nachdem der Opt-in-Consent-Fix vom 2026-06-28 dazu geführt hat, dass GA4 im Juli praktisch keine Sessions/Pageviews mehr zeigt.

## Kontext

- Auslöser: Frage im Chat "wie performt die Landingpage in Search Console/GA4 diesen Monat" → GA4-Report für 2026-07-01 bis 2026-07-23 lieferte **0 Zeilen**, während GSC im selben Zeitraum normale Daten zeigt (8 Klicks, ~115 Impressionen über alle Seiten).
- Zum Vergleich: GA4-Report für Juni 2026 zeigte reale Daten (43 Views auf `/`, 10 auf `/newsletter`, 7 auf `/kontakt` etc.).
- Ursache identifiziert: Commit `1eec259` "fix: load analytics after consent" (2026-06-28) in `normdex-landingpage`.
  - Vorher lud `index.html` das gtag.js-Script **immer** und signalisierte nur per Consent Mode v2 `analytics_storage: denied` — Google konnte dadurch über Modeling noch grobe Sessions/Pageviews schätzen.
  - Der Fix entfernte das Script komplett aus `index.html` und verschob es nach [[src/components/GoogleAnalytics.tsx]]. Dort wird `loadGoogleAnalytics()` (Zeile 53) jetzt **nur noch aufgerufen, wenn `localStorage['cookie-consent'].analytics === true`** ist — das gtag.js-Script wird für die meisten Besucher also gar nicht mehr nachgeladen, solange sie nicht aktiv im Cookie-Banner zustimmen.
  - Das ist rechtlich der korrektere Zustand (echtes Opt-in statt "immer laden, nur Signal denied"), führt aber dazu, dass GA4 strukturell nur noch einen Bruchteil des echten Traffics erfasst.
- Der Cookie-Banner ([[src/components/CookieConsent.tsx]]) zeigt drei gleichwertig gestylte Buttons ("Einstellungen", "Nur Notwendige", "Alle akzeptieren") ohne visuelle Hierarchie.
  - Wichtig: Nach EDPB-Guidance zu Dark Patterns muss "Ablehnen" genauso leicht/prominent bleiben wie "Akzeptieren" — den Accept-Button optisch hervorzuheben wäre ein unzulässiges Nudging und würde die Einwilligung rechtlich angreifbar machen. Diese Option scheidet aus.
- Search-Console-Daten sind von diesem Problem nicht betroffen, da sie unabhängig vom Consent-Status direkt von Google erhoben werden.

## Mögliche Optionen (noch nicht entschieden)

1. **Copy im Cookie-Banner verbessern** – konkret erklären, wofür Analytics-Daten genutzt werden (Transparenz statt Nudging), um die Opt-in-Rate organisch zu erhöhen. Rechtlich unproblematisch, Wirkung vermutlich gering (B2B-Fachpublikum, Opt-in-Raten typischerweise 20–40%).
2. **Granularität im Haupt-Banner sichtbar machen**, statt sie hinter "Einstellungen" zu verstecken, um Vertrauen zu schaffen.
3. **Cookieloses Analytics-Tool ergänzen** (z.B. Plausible/Fathom), das ohne personenbeziehbare Cookies und damit ohne Consent-Banner zählt — würde belastbare Zahlen unabhängig von der Opt-in-Quote liefern. Vermutlich der wirksamste Hebel.
4. **So akzeptieren** – GA4 zeigt künftig nur noch "echte Zustimmer", stattdessen stärker auf GSC-Klicks/Impressionen als Performance-Indikator setzen.

## Akzeptanzkriterien

- [ ] Entscheidung getroffen, ob und welche der obigen Optionen umgesetzt werden.
- [ ] Falls Option 3 (cookieloses Tool): Tool ausgewählt, DSGVO-/TKG-Einordnung geprüft, Integration umgesetzt.
- [ ] Falls Option 1/2: Banner-Copy bzw. -Layout angepasst, ohne gegen Dark-Pattern-Vorgaben zu verstoßen.
- [ ] Ergebnis im Vault dokumentiert (z.B. unter `06_Entwicklung` oder `08_Entscheidungen`, falls Grundsatzentscheidung).

## Notizen / Fortschritt

### 2026-07-23

- Todo aus Chat-Analyse angelegt, nachdem GSC- und GA4-Performance für Juli verglichen wurden und die Diskrepanz auffiel.
- Root Cause im Code verifiziert (Commit `1eec259`, `src/components/GoogleAnalytics.tsx:53-76`).
