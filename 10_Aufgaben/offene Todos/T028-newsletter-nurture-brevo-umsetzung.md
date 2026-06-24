# T028 · Newsletter-Nurture-Strecke in Brevo umsetzen

**Phase:** Marketing / Newsletter / Brevo  
**Priorität:** P2 · Lead-Nurture / erste Kunden  
**Status:** in Arbeit  
**Datum:** 2026-06-14

## Ziel

Die fünf E-Mails der Nurture-Strecke nach dem Lead-Magneten liegen als on-brand Vorlagen in Brevo, sodass die Automation sie nur noch einhängen muss. Versand startet erst nach Freigabe und sauberem Gutschein-Code.

## Kontext

Umsetzung nach [[Prompt - Brevo Newsletter-Automation umsetzen]] und [[Nurture-Strecke nach Lead-Magnet]]. Der Brevo-MCP kann Vorlagen, Attribute, Kontakte und Listen anlegen, aber nicht den Automations-Workflow selbst. Trigger und Wartezeiten baut Andreas im Brevo-Automations-Editor. Detail- und Klick-Anleitung in [[04 Brevo-Umsetzung und Workflow-Anleitung]].

Entscheidungen: individueller Gutschein-Code je Kontakt über `{{ contact.COUPON_CODE }}`, vorerst ohne Auto-Exit (Attribut `TRIAL_GESTARTET` vorbereitet), B-Betreffzeilen als A/B-Test je Mail (zweite Vorlage).

## Akzeptanzkriterien

- [x] Kontaktattribute `COUPON_CODE` (Text) und `TRIAL_GESTARTET` (boolean) angelegt.
- [x] Zehn Vorlagen angelegt (Mail 1 bis 5, je Betreff A und B), Sender Normdex (senderId 2), Reply-To office@normdex.at, Layout aus Master-Template, Texte wörtlich aus dem Strecken-Dokument.
- [x] Platzhalter, UTM je Mail, Abmeldelink und Wording-Regeln geprüft.
- [x] Logo-Assets unter `normdex.at/assets/` wieder erreichbar (Assets-Container), Header- und Footer-Logo werden in den Mails angezeigt.
- [x] Leitfaden-PDF unter der in Mail 1 verlinkten URL erreichbar.
- [x] Test-Mails der fünf A-Varianten an office@normdex.at versendet.
- [x] Gutschein-Befüllung verifizieren: liefert der Webhook aus [[T019-newsletter-gutschein-brevo-webhook-rollout]] den Code wirklich in das Attribut `COUPON_CODE`? **Befund 2026-06-24:** Nein — der T019-Webhook erzeugte den Stripe-Code und versendete ihn nur über eine eigene App-Mail, schrieb ihn aber **nie** in das Brevo-Kontaktattribut `COUPON_CODE`. `{{ contact.COUPON_CODE }}` wäre also leer geblieben. **Behoben:** Write-back in den Webhook-Service ergänzt (siehe Notizen 2026-06-24), Code wird jetzt per `PUT /v3/contacts/{email}` nach Brevo gespiegelt. Produktiv-Verifikation mit echter Test-Adresse steht noch aus (Deploy nötig).
- [ ] Automations-Workflow end-to-end testen (Wegwerf-Adresse in Liste 3, Wartezeiten temporär verkürzen, danach zurücksetzen).
- [ ] Auto-Exit bei Trial-Start über `TRIAL_GESTARTET` nachrüsten, sobald die App das Attribut setzt.
- [ ] Optionale Mail 6 (Tag 29) bei Bedarf texten und als Vorlage anlegen.
- [ ] Strecke nach Freigabe aktivieren (erst wenn Gutschein-Code zuverlässig befüllt wird).
- [ ] Testkontakt office@normdex.at und interne Liste „Newsletter Test (intern)" (listId 8) aufräumen oder bewusst behalten.

## Notizen / Fortschritt

- 2026-06-14: Attribute, zehn Vorlagen (IDs 6 bis 15) und Klick-Anleitung erstellt. Vorlagen sind inaktiv, lösen nichts aus.
- 2026-06-14: A/B vorerst nur angelegt, nicht zwingend scharf schalten. Bei aktuell sehr kleiner Liste liefert ein Betreff-Test keine belastbare Aussage, sinnvoll erst ab einigen hundert Anmeldungen.
- 2026-06-14: Logo fehlte zunächst in Header und Footer. Ursache war nicht Brevo, sondern fehlende PNGs im Assets-Container, der `normdex.at/assets/` ausliefert. Nach Bereitstellen der vorhandenen App-PNGs in `/opt/normdex-assets/` werden die Logos korrekt angezeigt. Betraf auch die Transaktionsmails der App, die dieselben URLs nutzen.
- 2026-06-14: Leitfaden-PDF war kurzzeitig nur HTML (SPA-Fallback), nach Landingpage-Update unter der URL als echtes PDF erreichbar.
- 2026-06-14: Test-Mails der fünf A-Varianten an office@normdex.at versendet, Platzhalter über Testwerte (Vorname, Test-Gutscheincode) geprüft.
- 2026-06-24: Namensabgleich `COUPON_CODE` durchgeführt. Ergebnis: Der T019-Webhook befüllte das Brevo-Attribut nie — Code wurde nur via eigener App-Mail (`tpl_newsletter_coupon`) zugestellt, kein `update_contact`/Attribut-Schreibvorgang im gesamten `apps/api`. Damit lief T019 (Sofort-Mail) an T028 (`{{ contact.COUPON_CODE }}`) vorbei.
- 2026-06-24: Option A umgesetzt — `apps/api/app/services/newsletter_coupon_service.py` schreibt den Code nach Erzeugung best-effort per `PUT https://api.brevo.com/v3/contacts/{email}` mit `attributes.COUPON_CODE` zurück, **bevor** die T019-Sofort-Mail rausgeht. Best-effort: Brevo-Fehler werden nur geloggt und blockieren weder Code-Erzeugung noch Mail-Versand. Idempotenz über die bestehende `coupon_sent_at`-Schranke (kein neues DB-Feld, keine Migration nötig). Guard auf `BREVO_API_KEY`. Zwei neue Tests in `tests/test_newsletter.py` (Write-back-Aufruf + Sync-Fehler blockiert Mail nicht), gesamte Newsletter-Suite grün (12 passed).
- 2026-06-24: Offen — Deploy auf Prod + End-to-end-Verifikation mit echter Test-Adresse, dass `COUPON_CODE` im Brevo-Kontakt ankommt und die Nurture-Vorlagen ihn rendern. Bestehende Kontakte, die ihren Code vor diesem Fix erhielten, haben das Attribut nicht gesetzt (ggf. einmaliger Backfill, falls relevant).

## Verwandte Dokumente

- [[04 Brevo-Umsetzung und Workflow-Anleitung]]
- [[Nurture-Strecke nach Lead-Magnet]]
- [[Prompt - Brevo Newsletter-Automation umsetzen]]
- [[T019-newsletter-gutschein-brevo-webhook-rollout]]
