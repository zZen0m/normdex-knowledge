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

Erledigt am 26.07.2026. Google-Ads-Konto (unter Permatec e.U.) angelegt, GA4-Property `normdex.at` (528084185) verknüpft, Status „Verknüpft", beide Datenfreigabe-Schalter („App- und Webmesswerte importieren", „Google Analytics-Zielgruppen importieren") aktiv.

## Schritt 3: Conversions aus GA4 importieren

Erledigt am 03.08.2026 (via Chrome-Steuerung). Beide GA4-Events waren inzwischen synchronisiert und importierbar. Ablauf im neuen Google-Ads-Setup-Flow:

1. Zielvorhaben → Conversions → „Neue Conversion-Aktion".
2. Datenquelle „Conversions auf einer Website" (GA4-Property normdex.at, 528084185) aktiv gelassen, „Conversions aus Anrufen" abgewählt.
3. Auf der Seite „Conversions gruppieren" NICHT die Kategorie-Kacheln oben (die legen ein neues Event an), sondern unten „Mehrere Conversion-Aktionen aus einem verknüpften Konto erstellen" → „Auswählen" → Property normdex.at → in der Checkbox-Tabelle `trial_start_click` und `newsletter_signup` angehakt.
4. Kategorien pro Zeile: `trial_start_click` → „Anmeldung", `newsletter_signup` → „Lead-Formular senden".
5. In den Einstellungen je Conversion: Wert auf „keinen Wert verwenden" (kein erfundener Wert, Start mit Klicks maximieren), Zählmethode „Nur genau eine Conversion" (= Einmal).
6. `trial_start_click` bleibt Primäre Aktion. Für `newsletter_signup` ist die Primär/Sekundär-Wahl im Wizard ausgegraut — stattdessen nach dem Wizard unter Zielvorhaben → „Lead-Formular senden" → „Zielvorhaben bearbeiten" → „Kontostandard" den Schalter „Als Standardzielvorhaben auf Kontoebene festlegen" AUSgeschaltet. Damit ist newsletter_signup nur Beobachtung, nicht gebotsrelevant.

Ergebnis in der Conversion-Aktionsliste bestätigt: `trial_start_click` „In Zielvorhaben auf Kontoebene einbezogen: Ja", `newsletter_signup: Nein". Beide Tracking-Status „Keine kürzlich erfassten Conversions" — das ist normal und wechselt erst nach dem ersten echten Klick auf „aktiv erfassend".
##### EDIT: DONE

## Schritt 4: Kampagne anlegen

Erledigt am 03.08.2026 (via Chrome-Steuerung). Kampagne „ÖNORM M 7140 Suche AT" (Campaign-ID 24100511423) wurde veröffentlicht und sofort pausiert.

Wichtige Erkenntnis: Der Google-Ads-Erstellungswizard für neue Kampagnen erlaubt nur eine einzige Anzeigengruppe und bietet auch keine Stelle für Negativ-Keywords. Beides ist erst nach der Veröffentlichung über die normale Kampagnenverwaltung möglich. Ablauf war daher:

1. Im Wizard: Kampagnentyp Suche, nur Suchnetzwerk (Partner und Display abgewählt), Standort Österreich, Sprache Deutsch, Gebotsstrategie „Klicks maximieren" mit Obergrenze 2,00 €, Tagesbudget 7,00 €/Tag, Zielvorhaben „Registrierungen". AI Max für die Kampagne deaktiviert gelassen (Textanpassung und Erweiterung der finalen URL beide aus – Achtung, die Überprüfen-Seite zeigte hier fälschlich „aktiviert" an, das ist ein reiner Anzeigefehler der Zusammenfassung, die tatsächlichen Schalter waren korrekt aus).
2. Anzeigengruppe 1 (ÖNORM M 7140 Kern) komplett im Wizard erstellt: Keywords, RSA-Anzeige mit allen 15 Überschriften/4 Textzeilen (Testphase-Set), Sitelinks, Callouts und Snippet auf Kampagnenebene (gelten automatisch für alle Anzeigengruppen). Beim Snippet-Typ gibt es kein exaktes „Funktionen"-Pendant in der deutschen Auswahlliste, „Ausstattung" als nächstliegende Option gewählt.
3. Kampagne veröffentlicht, danach sofort über das Status-Dropdown links oben auf „Pausiert" gesetzt (lief kurz als „Aktiviert" im Lernmodus, bevor die Pausierung gegriffen hat).
4. Anzeigengruppe 2 (Wirtschaftlichkeit Energiesysteme, Software) über Kampagnen → Anzeigengruppen → „+" nachträglich angelegt: gleiche Ziel-URL wie Gruppe 1 (`app.normdex.at/auth/register`), gleiche 5 Keywords aus dem Referenzdokument. Google Ads hat hier automatisch die komplette Anzeige (15 Überschriften, 4 Textzeilen) von Gruppe 1 übernommen, da dieselbe Ziel-URL erkannt wurde – Inhalte 1:1 geprüft, stimmen mit dem Testphase-Set aus dem Referenzdokument überein.
5. Anzeigengruppe 3 (Methodik und Lebenszykluskosten, informierend) ebenso nachträglich angelegt: Ziel-URL `normdex.at/newsletter`, 4 Keywords aus dem Referenzdokument. Achtung: Hier hatte Google Ads beim automatischen Vorschlag fälschlich die Ziel-URL und alle Anzeigentexte von Gruppe 1/2 übernommen (falsche URL `app.normdex.at/auth/register` statt Newsletter-Seite). Manuell korrigiert: Ziel-URL auf `normdex.at/newsletter` gesetzt und alle 15 Überschriften sowie 4 Textzeilen einzeln durch das Lead-Magnet-Set aus dem Referenzdokument ersetzt.
6. Negativ-Keywords (alle 20 aus dem Referenzdokument) über Zielgruppen, Keywords und Inhalte → Keywords → Tab „Auszuschließende Keywords" → „Hinzufügen zu: Kampagne" ergänzt.
7. Ad-Gruppen-Ebene-Sitelink-Vorschläge (in Anzeigengruppe 2 und 3 leer angeboten) bewusst nicht ausgefüllt, da die Kampagnen-Sitelinks aus Schritt 2 (Anzeigengruppe 1) automatisch für die ganze Kampagne gelten.

Ergebnis: Alle drei Anzeigengruppen, alle Keywords, Anzeigentexte, Erweiterungen und Negativ-Keywords stehen wie im Referenzdokument beschrieben. Kampagnenstatus „Pausiert" – noch nicht live.

Optionaler Zusatzschritt nach Veröffentlichung übersprungen: Google Ads bot direkt nach dem Publizieren an, ein zusätzliches globales Google-Tag (gtag.js) auf der Website einzubinden. Das wurde nicht gemacht, da die Conversion-Erfassung schon über die importierten GA4-Events läuft (siehe Schritt 3) und das Einbinden von Tracking-Code eine Website-Code-Änderung wäre, die nicht in dieser Session gehört.
##### EDIT: DONE

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
