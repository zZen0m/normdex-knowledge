# T018 - Landingpage-Kaufintent in App-Checkout überführen

## Status

abgeschlossen

## Bereich

Landingpage / App / Lizenzen / Registrierung / Stripe

## Ziel

User sollen auf der öffentlichen Landingpage bzw. Preisseite einen Plan auswählen können, ohne anonym direkt bei Stripe zu kaufen. Die Kaufabsicht wird an die App übergeben, nach Registrierung oder Login wieder aufgenommen und in der Lizenzverwaltung als vorbefüllter Checkout fortgesetzt.

## Kontext

Aktuell führt die Landingpage primär zur Registrierung. Die Preisseite zeigt monatliche und jährliche Preise, startet aber keinen echten Kauf-Flow. Der verbindliche Lizenzkauf liegt korrekt in der App unter `/licenses`, weil Stripe-Customer, Organisation, Trial-/Neukundenstatus und Lizenzzuordnung erst nach Account-/Organisationskontext sauber bestimmbar sind.

Die gewünschte Lösung ist ein Intent-Flow:

- Landingpage sammelt nur Kaufabsicht.
- App übernimmt nach Login/Registrierung den Checkout.
- Stripe Checkout wird erst aus der eingeloggten App heraus gestartet.

## Fachliche Entscheidung

Kein anonymer Direktkauf von der Landingpage zu Stripe.

Stattdessen:

1. User wählt auf der Landingpage monatlich oder jährlich und eine Lizenzanzahl.
2. CTA leitet zur App-Registrierung oder zum Login mit Query-Parametern weiter.
3. Nach erfolgreicher Registrierung oder Anmeldung landet der User in der Lizenzverwaltung.
4. Die Lizenzverwaltung öffnet automatisch den Kaufdialog mit vorausgewählten Mengen.
5. Dort sieht der User Preisvorschau, Testzeitraum bzw. Erstbestellungsrabatt und bestätigt den Kauf.
6. Erst danach startet Stripe Checkout oder die direkte Subscription-Erweiterung.

## Vorgeschlagene URLs

Landingpage-CTAs:

- `https://app.normdex.at/auth/register?plan=monthly&qty=1`
- `https://app.normdex.at/auth/register?plan=yearly&qty=1`

Optional für Login/Bestandsuser:

- `https://app.normdex.at/auth/login?next=/licenses?plan=monthly&qty=1&checkout=1`

Interner Zielzustand nach Auth:

- `/licenses?plan=monthly&qty=1&checkout=1`
- `/licenses?plan=yearly&qty=1&checkout=1`

## Technische Umsetzung

### Landingpage

- Preiskarten-CTAs mit konkreten App-Links versehen.
- Monatlich/Jährlich-Auswahl in Query-Parameter übersetzen.
- Optional Anzahlauswahl auf der Preisseite ergänzen übergeben.
- CTA-Texte auf den aktuellen Trial-Flow abstimmen:
  - Einzelkauf: `14 Tage kostenlos testen`
  - Mehrfachkauf: Hinweis auf Erstbestellungsrabatt statt Trial.

### App Registrierung / Login

- Query-Parameter `plan`, `qty` und optional `next` auslesen.
- Kaufintent nach erfolgreicher Registrierung/Login erhalten.
- Nach Registrierung zur bestehenden E-Mail-Verifizierungslogik passend weiterleiten.
- Nach erfolgreichem Login zur Lizenzverwaltung mit Intent-Parametern navigieren.
- Ungültige Werte defensiv ignorieren.

### App Lizenzverwaltung

- `Licenses.tsx` liest beim Laden:
  - `plan`
  - `qty`
  - `checkout=1`
- Kaufdialog automatisch öffnen.
- `qtyMonthly` oder `qtyYearly` entsprechend vorbefüllen.
- Preview automatisch laden.
- URL nach Übernahme des Intents bereinigen, damit Reloads den Dialog nicht wiederholt öffnen.
- Der User muss den Kauf weiterhin bewusst bestätigen.

### Backend

- Keine anonyme Checkout-Erzeugung nötig.
- Bestehende Endpoints `/licenses/checkout/preview` und `/licenses/checkout/create` weiterverwenden.
- Backend validiert weiterhin Neukundenstatus, Trial und Erstbestellungsrabatt auf Organisationsebene.

## Akzeptanzkriterien

- Klick auf monatlichen Landingpage-CTA führt zur App mit monatlichem Kaufintent.
- Klick auf jährlichen Landingpage-CTA führt zur App mit jährlichem Kaufintent.
- Nach Login/Registrierung öffnet sich die Lizenzverwaltung mit vorbefülltem Kaufdialog.
- Bei `qty=1` und Neukundenstatus wird der 14-Tage-Testzeitraum in der Vorschau angezeigt.
- Bei mehreren Lizenzen wird der Erstbestellungsrabatt angezeigt, nicht ein kostenloser Trial.
- Der User wird erst nach expliziter Bestätigung zu Stripe weitergeleitet.
- Abgebrochene oder ungültige Query-Parameter führen nicht zu fehlerhaften Checkout-Sessions.
- Reload der Lizenzseite öffnet den Dialog nicht endlos neu.

## Betroffene Orientierungspunkte

- `D:\Normdex\01_repos\normdex-landingpage\src\pages\Pricing.tsx`
- `D:\Normdex\01_repos\normdex-app\apps\frontend\src\pages\Register.tsx`
- `D:\Normdex\01_repos\normdex-app\apps\frontend\src\pages\Login.tsx`
- `D:\Normdex\01_repos\normdex-app\apps\frontend\src\pages\Licenses.tsx`
- `D:\Normdex\01_repos\normdex-app\apps\api\app\routers\licenses_v2.py`

## Notizen / Fortschritt

- 2026-04-30: Todo angelegt. Entscheidung bestätigt: Landingpage soll Kaufabsicht an die App übergeben; verbindlicher Checkout bleibt im eingeloggten App-Kontext.
- 2026-04-30: Implementiert. Alle vier betroffenen Dateien angepasst (Pricing.tsx, Register.tsx, Login.tsx, Licenses.tsx).
