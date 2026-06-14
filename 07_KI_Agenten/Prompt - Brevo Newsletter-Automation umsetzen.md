# Prompt: Newsletter-Automation in Brevo umsetzen

Diesen Text in ein neues Kontext-Window posten. Er beschreibt vollständig, was zu tun ist.

---

Du hilfst mir, die Newsletter-Nurture-Strecke für Normdex in Brevo umzusetzen. Lies dir zuerst `02_knowledge\normdex-vault\00_Start\AI Kontext - Einstieg.md` durch, danach diese drei Marketing-Dokumente, die die Inhalte und Regeln festlegen:

- `04_Marketing\Newsletter\00 Newsletter-Leitfaden.md` (Einstieg, verweist auf alle Newsletter-Vorgaben)
- `04_Marketing\Newsletter\01 Wording-Vorgaben.md` (verbindliche Sprache, Aufbau, Signatur, CTA-Regeln)
- `04_Marketing\Newsletter\02 Designvorgaben.md` (verbindliche Farben, Schrift, Layout, Footer)
- `04_Marketing\Newsletter\03 Master-Template.html` (fertiges HTML-Gerüst, Basis für jede Mail)
- `04_Marketing\Newsletter\Newsletter-Archiv\Nurture-Strecke nach Lead-Magnet.md` (die fünf Mail-Texte, Betreffzeilen, Preheader, Versandtakt)
- `04_Marketing\Marketingplan 2026 - Erste Kunden.md` (Funnel-Logik, Schreibregeln)
- `01_Produkt\Brand Identity & Voice.md` und `04_Marketing\Key Messages & CTAs.md` (Tonalität, Gutschein-Logik)

## Ziel

Setze die fünf E-Mails der Nurture-Strecke als **on-brand E-Mail-Vorlagen in Brevo** an, sodass die Automation sie nur noch einhängen muss. Die Texte stehen wörtlich in `Nurture-Strecke nach Lead-Magnet.md` (Mail 1 bis 5, plus optionale Mail 6 an Tag 29). Übernimm Betreff, Betreff B, Preheader und Fließtext genau von dort.

## Wichtige Einschränkung (zuerst lesen)

Der Brevo-MCP-Server kann Vorlagen, Listen, Attribute, Kontakte und Coupons anlegen, aber **nicht den Automations-Workflow selbst** (Trigger plus Wartezeiten gibt es nur im Automations-Editor der Brevo-Oberfläche). Plane deshalb so:

1. Du legst die fünf Vorlagen per MCP an (das ist der Hauptteil dieser Aufgabe).
2. Den Workflow (Auslöser, Wartezeiten, Exit) baue ich danach selbst im Brevo-UI. Liefere dazu am Ende eine kurze Klick-Anleitung mit den Wartezeiten Tag 0, 3, 8, 15, 25 (und optional 29).

Sende oder aktiviere nichts ohne meine ausdrückliche Freigabe. Verschicke höchstens Test-Mails an mich (office@normdex.at), und auch das erst nach Rückfrage.

## Brevo-Account, Stand heute

- MCP-Server-UUID: `02096dfb-f479-4478-8b34-1f8f716cbfc7`
- Liste: **Normdex Newsletter**, `listId = 3` (aktuell 3 Kontakte, alle Double-Opt-in bestätigt)
- Absender: **Normdex**, `notify@normdex.at`, `senderId = 2`, Reply-To `office@normdex.at`
- Bestehende Vorlage als **Design-Referenz**: Template-ID 1 ("Standard-Vorlage für Double-Opt-in-Bestätigungen"). Übernimm exakt dieses Layout für die neuen Mails.

## Design-Vorgaben

Verwende als Basis das fertige HTML-Gerüst `04_Marketing\Newsletter\03 Master-Template.html` und die Regeln in `04_Marketing\Newsletter\02 Designvorgaben.md`. Beide sind aus der bewährten Brevo-Vorlage (Template-ID 1) abgeleitet. Die folgenden Eckwerte sind nur die Kurzfassung zur schnellen Kontrolle:

- Hintergrund Seite `#FAFAFA`, Karte `#FFFFFF`, zentriert, `max-width: 600px`, `border: 1px solid #E0E0E0`, `border-radius: 14px`.
- Kopf-Balken `#003C3E` (dunkles Petrol) mit Logo: `https://normdex.at/assets/normdex_logo_horizontal_invert.png`, Breite 190px.
- Akzent: pinke Linie/Aufzählungspunkt `#FF2D58` (z. B. `accent-line`: 3px hoch, 48px breit, unter der H1).
- Textfarbe `#282F3A`, gedämpft `#64748B`, Überschrift `#003C3E`, `font-size: 16px`, `line-height: 1.6`.
- Schrift: `system-ui, -apple-system, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif`.
- Primärer Button: Hintergrund `#003C3E`, Text `#FAFAFA`, `padding: 14px 32px`, `border-radius: 8px`, `font-weight: 700`.
- Footer-Signatur mit Icon `https://normdex.at/assets/normdex_icon.png` (28px, radius 8px) und Zeile "Normdex™ ... Du erhältst diese E-Mail, weil du dich für den Normdex™ Newsletter angemeldet hast."
- Bottom-Zeile mit `© 2026 Normdex™` und Link "Impressum/Datenschutz" auf `https://normdex.at/impressum`.
- Jede Mail braucht einen versteckten Preheader (`<span class="preheader">`) mit dem Preheader-Text aus dem Strecken-Dokument.
- Pflicht-Footer: ein Abmelde-Link. Nutze Brevos Unsubscribe-Platzhalter (`{{ unsubscribe }}` bzw. die Standard-Variable), damit die Mails rechtlich sauber sind.

## Schreibregeln (verbindlich)

Du-Form, aber Wir-Form für Normdex (immer "wir", nie "ich"), direkte Umlaute (ä, ö, ü), keine Gedankenstriche oder Halbgeviertstriche als Satzzeichen (Punkt, Komma, Doppelpunkt oder Klammer stattdessen), Prozent immer als Zeichen **%** (Gutschein-Wert fett), sachlich und ohne übertriebene Werbesprache, kein Mitbewerber namentlich. Anrede immer mit Vorname. Signatur "Dein Normdex-Team". Verbindliche Details in [[01 Wording-Vorgaben]].

## Platzhalter und Links

- Anrede immer mit Vorname, `Hallo {{ contact.VORNAME }},` (Vorname ist Pflichtfeld bei der Anmeldung).
- Gutschein-Code als `{{ contact.COUPON_CODE }}` (Attributname unten klären, siehe offene Punkte).
- Leitfaden-Download-Button (Mail 1): `https://normdex.at/Normdex_Praxisleitfaden_OENORM_M7140.pdf` (bitte vor Verwendung prüfen, ob die PDF unter dieser URL erreichbar ist).
- Mail 3 sendet keinen Beispielbericht (die Landingpage zeigt die Berichtsabbildungen bereits). Der CTA "Normdex ansehen" verweist auf `https://normdex.at`.
- Trial-Start-Button (Mail 4 und 5): `https://app.normdex.at/auth/register`.
- Setze überall sinnvolle UTM-Parameter (`utm_source=brevo`, `utm_medium=email`, `utm_campaign=nurture`, je Mail eine eigene `utm_content`, z. B. `mail1`...`mail5`).

## Aufgaben-Checkliste

1. Vorlage 1 bis 5 (und optional 6) als Brevo-Vorlagen anlegen, Inhalte wörtlich aus `Nurture-Strecke nach Lead-Magnet.md`, Layout aus `03 Master-Template.html` (Platzhalter ersetzen), Regeln aus `01 Wording-Vorgaben.md` und `02 Designvorgaben.md`. Verwende die A-Betreffzeile als Betreff, halte die B-Variante im Notizfeld oder als zweite Vorlage fest, falls A/B getestet werden soll.
2. Absender `senderId = 2` (Normdex, notify@normdex.at), Reply-To `office@normdex.at`.
3. Prüfen, ob das Kontaktattribut für den Gutschein-Code schon existiert; falls nicht, anlegen.
4. Prüfen, ob ein Attribut für die Exit-Bedingung (Trial gestartet) existiert; falls nicht, anlegen (z. B. boolesches `TRIAL_GESTARTET`).
5. Je eine Test-Mail an office@normdex.at senden, erst nach meiner Freigabe, um Darstellung und Platzhalter zu prüfen.
6. Kurze Klick-Anleitung liefern, wie ich im Brevo-Automations-Editor den Workflow zusammenbaue: Auslöser = bestätigte Newsletter-Anmeldung (Double-Opt-in) in Liste 3, dann die fünf Vorlagen mit Wartezeiten Tag 0, 3, 8, 15, 25, Exit-Bedingung "Trial gestartet".

## Offene Punkte, bitte zu Beginn mit mir klären

- **Gutschein:** Ein fester Code für alle (einfach) oder ein individueller Stripe-Promotion-Code je Kontakt (laut `Key Messages & CTAs.md` und Backend-Task `T019-newsletter-gutschein-brevo-webhook-rollout`)? Davon hängt ab, ob die Mail einen festen Code zeigt oder das Attribut `{{ contact.COUPON_CODE }}`.
- **Exit-Bedingung:** Soll der Trial-Start die Strecke automatisch beenden? Wenn ja, wie wird das Attribut gesetzt (App-Webhook in Brevo)? Falls das noch nicht steht, Strecke vorerst ohne Auto-Exit aufsetzen und später nachrüsten.
- **A/B-Betreff:** Sollen die B-Betreffzeilen wirklich als A/B-Test laufen oder nur dokumentiert bleiben?

Frag mich diese drei Punkte, bevor du die Vorlagen final schreibst.
