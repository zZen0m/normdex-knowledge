# Brevo-Umsetzung und Workflow-Anleitung (Nurture-Strecke)

Diese Datei dokumentiert, was für die Nurture-Strecke in Brevo bereits per MCP angelegt wurde, und enthält die Klick-Anleitung, mit der du den Automations-Workflow im Brevo-UI zusammenbaust. Grundlage sind [[Prompt - Brevo Newsletter-Automation umsetzen]] und [[Nurture-Strecke nach Lead-Magnet]].

Stand: 14.06.2026.

## Was bereits angelegt ist

### Kontaktattribute (neu)

- **COUPON_CODE** (Text): hält den individuellen Stripe-Promotion-Code je Kontakt. Wird in den Mails als `{{ contact.COUPON_CODE }}` ausgegeben. Muss vom Backend je Kontakt befüllt werden, siehe [[T019-newsletter-gutschein-brevo-webhook-rollout]].
- **TRIAL_GESTARTET** (boolean): vorbereitet für die spätere Exit-Bedingung. Aktuell noch ohne Auto-Exit, siehe unten.

### E-Mail-Vorlagen (10 Stück, je Mail Betreff A und B)

Alle Vorlagen: Absender Normdex (senderId 2, notify@normdex.at), Reply-To office@normdex.at, Layout aus [[03 Master-Template]], Inhalte wörtlich aus [[Nurture-Strecke nach Lead-Magnet]]. Sie sind als inaktiv angelegt und versenden nichts von selbst.

| Mail | Versand | Vorlage A (Betreff A) | ID | Vorlage B (Betreff B) | ID |
|---|---|---|---|---|---|
| 1 | Tag 0 | Dein Leitfaden zur ÖNORM M 7140 ist da | 6 | Da ist dein Praxisleitfaden | 7 |
| 2 | Tag 3 | Der häufigste Fehler in Wirtschaftlichkeitsberechnungen | 8 | Ein Detail, das viele Vergleichsrechnungen verzerrt | 9 |
| 3 | Tag 8 | Geht das nicht auch in Excel? | 10 | Excel oder Werkzeug, wann lohnt sich was | 11 |
| 4 | Tag 15 | Normkonform, im Browser, im Team | 12 | So startest du ohne Risiko | 13 |
| 5 | Tag 25 | Dein Gutschein gilt noch wenige Tage | 14 | Bald läuft dein Rabatt aus | 15 |

Personalisierung in den Mails: Anrede `{{ contact.VORNAME }}`, Gutschein `{{ contact.COUPON_CODE }}` (Mail 1, 4, 5), Abmeldelink über `{{ unsubscribe }}`. UTM je Mail gesetzt (`utm_source=brevo`, `utm_medium=email`, `utm_campaign=nurture`, `utm_content=mail1` bis `mail5`). Mail 2 ist eine reine Wertmail ohne Button.

## Offener Punkt vor Aktivierung: Leitfaden-PDF

Der Download-Button in Mail 1 zeigt auf `https://normdex.at/Normdex_Praxisleitfaden_OENORM_M7140.pdf`. Diese URL liefert derzeit **kein PDF**, sondern die HTML-Startseite (SPA-Fallback). Vor dem Scharfschalten der Strecke entweder die PDF unter genau dieser URL bereitstellen oder den Link in Vorlage 6 und 7 auf den tatsächlichen Speicherort anpassen.

## Test-Mails (erst nach deiner Freigabe)

Auf Wunsch verschicken wir je eine Test-Mail an office@normdex.at, um Darstellung und Platzhalter zu prüfen. Hinweis: In Test-Mails bleiben `{{ contact.VORNAME }}` und `{{ contact.COUPON_CODE }}` ohne hinterlegte Kontaktdaten leer. Für einen echten Platzhalter-Test entweder den eigenen Kontakt mit Vorname und Testcode in COUPON_CODE pflegen oder im Automations-Test einen Beispielkontakt nutzen.

## Klick-Anleitung: Workflow im Brevo-Automations-Editor

Der Automations-Workflow selbst lässt sich nur im Brevo-UI bauen (Trigger und Wartezeiten gibt es nicht über die API).

### 1. Automation anlegen

1. In Brevo auf **Automations** gehen, **Create an automation**, dann **Start from scratch**.
2. Name vergeben, zum Beispiel "Nurture nach Lead-Magnet".

### 2. Auslöser (Entry point)

1. Als Trigger **Contact added to a list** wählen (Kontakt zu einer Liste hinzugefügt).
2. Liste **Normdex Newsletter** (listId 3) auswählen. Da die Anmeldung mit Double-Opt-in läuft, landet ein Kontakt erst nach bestätigter Anmeldung in dieser Liste, der Trigger feuert also genau zum richtigen Zeitpunkt.

> **Dev/Prod-Hinweis (2026-06-24):** Dev- und Prod-Backend nutzen denselben Brevo-Account. Solange `BREVO_LIST_ID` in `.env.api.dev` und `.env.api.prod` auf dieselbe Liste zeigt, lösen Test-Anmeldungen auf dem Dev-Server **auch** den Prod-Webhook (und damit einen echten Stripe-Gutschein) aus, da Brevo-Webhooks account-weit auf Events abonniert sind, nicht pro Liste gefiltert. Geplante Trennung über eine eigene Dev-Liste: siehe [[T033-brevo-dev-prod-listentrennung]].

### 3. Mailfolge mit Wartezeiten

Baue diese Abfolge auf. Die Wartezeiten sind kumulativ relativ zum Eintritt gedacht, deshalb stehen unten die Abstände zwischen den Schritten.

1. **Mail 1 senden**, sofort (Tag 0). Send an email, Vorlage 6 (oder A/B, siehe unten).
2. **Wait** 3 Tage.
3. **Mail 2 senden** (Tag 3), Vorlage 8.
4. **Wait** 5 Tage.
5. **Mail 3 senden** (Tag 8), Vorlage 10.
6. **Wait** 7 Tage.
7. **Mail 4 senden** (Tag 15), Vorlage 12.
8. **Wait** 10 Tage.
9. **Mail 5 senden** (Tag 25), Vorlage 14.

Optional, falls die sechste Erinnerung gewünscht ist: danach **Wait** 4 Tage und eine kurze sechste Mail an Tag 29. Für Mail 6 liegt aktuell noch kein finaler Text vor, sie müsste vorher als Vorlage ergänzt werden.

### 4. A/B-Test der Betreffzeilen

Die B-Varianten (Vorlagen 7, 9, 11, 13, 15) haben denselben Inhalt wie A, nur eine andere Betreffzeile. Um je Mail A gegen B zu testen, an der jeweiligen Sendestelle statt eines einzelnen Sendeschritts so vorgehen:

1. Eine **A/B Split** beziehungsweise prozentuale Aufteilung einfügen (50/50).
2. Im einen Zweig **Send an email** mit Vorlage A, im anderen Zweig **Send an email** mit Vorlage B.
3. Nach beiden Zweigen wieder zum gemeinsamen **Wait**-Schritt zusammenführen.

Wer den A/B-Test erst später oder nur für einzelne Mails will, nutzt zunächst nur die A-Vorlagen und ergänzt die Splits bei Bedarf.

### 5. Exit-Bedingung (vorerst ohne Auto-Exit)

Aktuell läuft die Strecke ohne automatischen Abbruch bei Trial-Start. Das Attribut **TRIAL_GESTARTET** ist bereits angelegt, damit du den Exit nachrüsten kannst, sobald die App den Trial-Start an Brevo meldet (Webhook oder API, der das Attribut auf true setzt).

Zum späteren Nachrüsten:

1. Vor jedem Sendeschritt (oder als globale Bedingung) einen **If/Else**-Schritt einfügen, der prüft, ob `TRIAL_GESTARTET = true`.
2. Trifft das zu, den Kontakt über **Exit the automation** aus der Strecke nehmen, damit keine weiteren Werbe-Mails dieser Serie folgen. Der Kontakt geht dann ins Produkt-Onboarding über.

### 6. Aktivieren

Erst nach deiner Freigabe und nach Klärung der PDF-URL die Automation auf **aktiv** schalten. Vorher die Vorlagen über je eine Test-Mail prüfen.

## Verwandte Dokumente

- [[00 Newsletter-Leitfaden]]
- [[01 Wording-Vorgaben]]
- [[02 Designvorgaben]]
- [[03 Master-Template]]
- [[Nurture-Strecke nach Lead-Magnet]]
- [[Key Messages & CTAs]]
- [[Prompt - Brevo Newsletter-Automation umsetzen]]
