# Google Ads: Setup-Anleitung (Erstinbetriebnahme)

Konkrete Klick-für-Klick-Schritte, um die fertige Kampagne aus [[Google Ads - Keywords und Anzeigen]] live zu schalten. Diese Anleitung ist manuell auszuführen (Google Ads Konto, Google Analytics Konto) — kann nicht per Code oder Agent ausgeführt werden.

## Voraussetzung: Code-Deployment (zuerst erledigen)

Am 26.07.2026 wurden im Repo `normdex-landingpage` zwei GA4-Conversion-Events ergänzt (bisher gab es nur Pageview-Tracking, keine Events):

- `trial_start_click` — feuert beim Klick auf „Jetzt kostenlos starten" auf `/preise` (`src/pages/Pricing.tsx`), mit Parametern `plan` und `qty`. Das ist das harte Ziel.
- `newsletter_signup` — feuert nach erfolgreicher Anmeldung im Newsletter-Formular (`src/components/NewsletterForm.tsx`), mit Parameter `herkunft`. Das ist das weiche Ziel.

Beide Events laufen über die bestehende, consent-gated GA4-Integration (`GoogleAnalytics.tsx`, Property `G-WGPWWKYJRW`) — sie feuern nur, wenn der Besucher Analyse-Cookies akzeptiert hat, genau wie die bisherigen Pageviews.

**Vor Schritt 1 unten:** Änderungen committen, nach `normdex-landingpage` deployen (Build war lokal bereits erfolgreich getestet), und per Klick auf der Live-Seite (Realtime-Bericht in GA4) verifizieren, dass beide Events tatsächlich ankommen.
##### EDIT: DONE
### Troubleshooting-Notiz vom 26.07.2026: „Datenerhebung nicht aktiv" trotz korrektem Code

Nach dem Deployment zeigte GA4 dauerhaft „Für Ihre Website ist die Datenerhebung nicht aktiv" und der Echtzeit-Bericht blieb bei Testklicks auf 0, obwohl mehrfach mit erteiltem Analyse-Consent getestet wurde. Ausführliche Diagnose ergab:

- `window.dataLayer` im Browser enthielt alle korrekten Einträge (`consent default`, `consent update` auf granted, `js`, `config`, das eigene `newsletter_signup`-Event) — der Code ist korrekt.
- Die echte Google-Bibliothek lädt und läuft (automatische Enhanced-Measurement-Instrumentierung wie `gtm.formInteract`, `gtm.click-v2` erschien im dataLayer).
- Googles eigener Tag Assistant bestätigte „Google-Tag gefunden", meldete aber „Von diesem Tag wurden keine Treffer gesendet" — und das über drei unabhängige Testumgebungen hinweg (Desktop mit vollständig deaktiviertem AdGuard, Handy im Mobilfunknetz).
- Direkter Server-zu-Server-Test per Measurement Protocol (`POST https://www.google-analytics.com/mp/collect?measurement_id=G-WGPWWKYJRW&api_secret=…`, `204`-Antwort) landete sofort und korrekt im Echtzeit-Bericht. **Die Property selbst ist also nachweislich gesund.**
- Fazit: Die Blockade liegt spezifisch in der Testumgebung des Nutzers (vermutlich netzwerk-/DNS-seitig, tiefer als die AdGuard-Browsererweiterung — z. B. Router- oder systemweite DNS-Konfiguration), nicht im Code oder in der GA4-Konfiguration. Für echte Website-Besucher außerhalb dieses speziellen Setups ist keine Beeinträchtigung zu erwarten.
- Für zukünftige Property-Gesundheitschecks: ein Measurement-Protocol-Test-Hit (siehe oben) ist der schnellste, browserunabhängige Weg, um „Property kaputt" von „Client-Problem" zu unterscheiden. Das dafür angelegte API-Secret sollte nach Gebrauch in GA4 unter Verwaltung → Datenstreams → Measurement Protocol API-Secrets wieder gelöscht werden.

## Schritt 1: GA4-Events als Key Events markieren

Erledigt am 26.07.2026. Da eigene Testklicks aus dem lokalen Netzwerk nachweislich nie bei Google ankamen (siehe Troubleshooting-Notiz oben), wurden die Key Events manuell registriert, ohne auf einen empfangenen Testklick zu warten:

1. In Google Analytics 4 (Property zu `G-WGPWWKYJRW`) einloggen.
2. Verwaltung → Datenanzeige → **Ereignisse** → Tab **„Schlüsselereignisse"** → „Schlüsselereignis erstellen" → „Ereignis erstellen".
3. Ereignisname exakt eintragen (`trial_start_click` bzw. `newsletter_signup`), „Als Schlüsselereignis markieren" aktivieren.
4. Bei „Standardwert des Schlüsselereignisses" **„Keinen Standardwert für Schlüsselereignis festlegen"** wählen (kein fixer Dollarwert, da unterschiedliche Plan-/Mengenpreise und Start mit „Klicks maximieren" statt wertbasierten Geboten).
5. Bei „So soll ein Ereignis erstellt werden" **„Mit Code erstellen"** wählen (nicht „Ohne Code erstellen" — der Code existiert bereits im Repo, es muss nur die Definition angelegt werden, kein URL-Trigger nötig).
6. Erstellen, für beide Events wiederholen.

Damit sind beide Events jetzt als Schlüsselereignisse registriert, unabhängig davon, ob bereits echte Daten dafür vorliegen — sie zählen automatisch, sobald ein Besucher sie auslöst.

## Schritt 2: Google Ads mit GA4 verknüpfen

1. In Google Ads: Tools und Einstellungen → Einrichtung → Verknüpfte Konten.
2. Google Analytics (GA4) auswählen, Property verknüpfen, die auch für `normdex.at` läuft.
3. Verknüpfung bestätigen.

## Schritt 3: Conversions aus GA4 importieren

1. In Google Ads: Tools und Einstellungen → Messung → Conversions.
2. „+ Neu" → „Google Analytics (GA4)-Property importieren".
3. Beide Key Events (`trial_start_click`, `newsletter_signup`) auswählen und importieren.
4. Für `trial_start_click`: Kategorie „Kauf/Anmeldung" (hartes Ziel), Zählmethode „Einmal" pro Klick.
5. Für `newsletter_signup`: Kategorie „Lead" (weiches Ziel), Zählmethode „Einmal".
6. Optional, aber empfohlen: `trial_start_click` als primäre Conversion für die Gebotsstrategie markieren, `newsletter_signup` als sekundär (nur zur Beobachtung, nicht gebotsrelevant) — sonst optimiert Google Ads am Ende auf das leichtere Ziel statt auf zahlungsrelevante Klicks.

## Schritt 4: Kampagne anlegen

Alle Inhalte (Keywords, Anzeigentexte, Erweiterungen, Budget) stehen fertig in [[Google Ads - Keywords und Anzeigen]]. Ablauf:

1. Neue Kampagne, Ziel „Website-Traffic" oder ohne Zielvorgabe (nicht „Umsatz", da keine E-Commerce-Conversion-Werte hinterlegt sind).
2. Kampagnentyp: Suche. Netzwerke: nur Suchnetzwerk, Displaynetzwerk und Suchnetzwerk-Partner abwählen.
3. Standort: Österreich. Sprache: Deutsch.
4. Gebotsstrategie: „Klicks maximieren" mit Ober­grenze 1,50–2,50 € (siehe Kampagnen-Setup-Tabelle im Referenzdokument).
5. Tagesbudget: 5–10 € zum Start.
6. Negativ-Keywords auf Kampagnenebene aus dem Referenzdokument eintragen.
7. Drei Anzeigengruppen anlegen exakt wie im Referenzdokument beschrieben (ÖNORM M 7140 Kern, Wirtschaftlichkeit Energiesysteme, Methodik/Lebenszykluskosten informierend), jeweils mit den dort gelisteten Keywords (Wortgruppe/genau passend wie angegeben) und Ziel-URLs.
8. Responsive Suchanzeigen mit den vorformulierten Überschriften und Textzeilen aus dem Referenzdokument anlegen (Gruppe 1+2 → Testphase-Texte, Gruppe 3 → Lead-Magnet-Texte).
9. Anzeigenerweiterungen (Sitelinks, Callouts, Snippet) aus dem Referenzdokument hinzufügen.

## Schritt 5: Vor dem Start prüfen

- Conversion-Tracking-Status in Google Ads zeigt „Aktiv erfassend" (nicht mehr „Kein aktuelles Erfassen") — sonst noch nicht live schalten.
- Landingpages sind fertig (bereits verifiziert am 26.07.2026: `/oenorm-m-7140`, `/preise`, `/newsletter` laden alle unter 500 ms, Viewport-Meta korrekt, Trial-CTA verlinkt korrekt zu `app.normdex.at/auth/register?plan=…&qty=…`, Lead-Magnet- und Beispielbericht-PDF erreichbar).
- Kampagne zunächst pausiert lassen, erst nach diesem Check aktivieren.

## Schritt 6: Nach dem Start

- Nach 2–3 Wochen laut Referenzdokument schwache Keywords/Überschriften aussortieren.
- Wöchentlich kurz messen: Kosten pro Newsletter-Anmeldung, Kosten pro Trial-Start, welche Anzeigengruppe liefert (siehe Abschnitt „Messung" in [[Marketingplan 2026 - Erste Kunden]]).
- Sobald genug Conversions vorliegen, auf „Conversions maximieren" mit Ziel-CPA wechseln.

## Verwandte Dokumente

- [[Google Ads - Keywords und Anzeigen]]
- [[Marketingplan 2026 - Erste Kunden]]
- [[Landingpage]]
