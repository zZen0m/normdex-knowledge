# T017 - Testzeitraum und Erstbestellungsrabatt für Lizenzen

## Status

erledigt

## Abgeschlossen

2026-05-30

## Bereich

App / Lizenzen / Stripe / Landingpage / Rechtliches

## Ziel

Normdex soll Neukunden einen transparenten 14-Tage-Testvorteil anbieten. Beim Erstkauf genau einer Lizenz wird daraus ein echter 14-tägiger kostenloser Testzeitraum. Beim Erstkauf mehrerer Lizenzen wird der Testvorteil in einen einmaligen Erstbestellungsrabatt auf die Hauptlizenz umgewandelt, damit Kunden durch eine größere Erstbestellung nicht benachteiligt werden.

## Kontext

Aktuell werden neue Lizenzen nach erfolgreichem Stripe-Checkout oder direkter Erweiterung einer bestehenden Subscription sofort aktiv geschaltet und abgerechnet. Für Neukunden soll stattdessen ein klar kommunizierter 14-Tage-Testvorteil ergänzt werden:

- Der 14-Tage-Testvorteil gilt nur für Organisationen ohne bestehende aktive, gekündigte, fehlgeschlagene oder bereits laufende Testlizenz.
- Beim Kauf genau einer Lizenz wird der Testvorteil als echter 14-tägiger kostenloser Testzeitraum umgesetzt.
- Beim Kauf mehrerer Lizenzen wird der Testvorteil in einen einmaligen Erstbestellungsrabatt auf die Hauptlizenz umgewandelt.
- Mehrfachkäufe starten sofort aktiv und zahlungspflichtig; der Rabatt muss im Checkout klar erklärt und ausgewiesen werden.
- Während eines echten Trials zählt die Lizenz als voll nutzbare Lizenz für das bestehende Concurrent-Use-/Heartbeat-Locking.
- Nach Ablauf der 14 Tage wird die Testlizenz automatisch in eine normale aktive Lizenz überführt und über Stripe abgebucht.

## Sprachgebrauch / Kommunikation

Die Kommunikation soll konsequent zwischen drei Begriffen unterscheiden:

- `14-Tage-Testvorteil`: Oberbegriff für den Neukunden-Vorteil bei der ersten Bestellung.
- `Kostenloser Testzeitraum`: nur beim Erstkauf genau einer Lizenz.
- `Erstbestellungsrabatt`: Umwandlung des Testvorteils bei einer Erstbestellung mit mehreren Lizenzen.

Kernbotschaft für Landingpage, Kaufdialog, Checkout und Legal:

- „Du kannst 14 Tage kostenlos testen, wenn du eine Lizenz kaufst.“
- „Wenn du direkt mehrere Lizenzen kaufst, wandeln wir diesen Testvorteil automatisch in einen Rabatt auf die Hauptlizenz um.“
- „Weitere Lizenzen werden regulär berechnet.“

Die Darstellung muss einfach, transparent und vor Kaufabschluss sichtbar sein. Kunden sollen verstehen, ob sie gerade einen kostenlosen Testzeitraum oder einen Erstbestellungsrabatt erhalten.

## Fachliche Regeln

- Der 14-Tage-Testvorteil kann pro Organisation nur einmal entstehen.
- Neukunde/qualifizierte Erstbestellung wird technisch als Organisation ohne relevante Lizenzhistorie definiert: keine bestehende oder frühere `active`, `scheduled_end`, `payment_failed`, `pending` oder `trial` Lizenz und kein bereits dokumentierter Testvorteil in Lizenz-/Order-Metadaten.
- Eine Testlizenz darf nur beim Erstkauf einer einzelnen monatlichen oder jährlichen Lizenz entstehen.
- Kauft ein Neukunde mehrere Lizenzen gleichzeitig, gibt es keinen echten Testzeitraum. Stattdessen wird der Testvorteil automatisch in einen Erstbestellungsrabatt auf die Hauptlizenz umgewandelt.
- Bei monatlicher Erstbestellung mit mehreren Lizenzen wird ein einheitlicher Erstbestellungsrabatt von 24,50 EUR auf die Hauptlizenz der ersten Rechnung angewendet. Weitere Lizenzen werden regulär berechnet.
- Bei jährlicher Erstbestellung mit mehreren Lizenzen wird derselbe einheitliche Erstbestellungsrabatt von 24,50 EUR auf die Hauptlizenz der ersten Rechnung angewendet. Weitere Lizenzen werden regulär berechnet.
- Kauft ein Nutzer während einer aktiven Testlizenz weitere Lizenzen, muss die Testlizenz zuerst in eine normale zahlungspflichtige Lizenz überführt werden. Die bis zur automatischen Trial-Überführung verbleibenden Testtage werden dabei aliquot als Gutschrift auf die Hauptlizenz angerechnet. Erst danach darf der Zusatzkauf fortgesetzt werden.
- Die Gutschrift bei Trial-Vorzeitumwandlung richtet sich nach den verbleibenden Trial-Tagen:
  - einheitlich für monatliche und jährliche Lizenzen: `24,50 EUR / 14 * verbleibende Trial-Tage`.
  - Beispiel monatlich: Zusatzkauf nach 7 von 14 Trial-Tagen → 7 Tage verbleiben → 12,25 EUR Gutschrift auf die Hauptlizenz.
- Einzelne Zusatzkäufe bei bestehenden Kunden erhalten keinen neuen Testvorteil.
- Die Landingpage soll monatliche und jährliche Lizenzkäufe unterstützen und erklären, wann ein kostenloser Testzeitraum gilt und wann daraus ein Erstbestellungsrabatt wird.
- Erledigt: Der frühere Marketing-Hinweis „14 Tage kostenlos, ohne Kreditkarte“ ist fachlich nicht mehr korrekt. Aktuelle Formulierung: 14 Tage kostenlos beim qualifizierten Einzel-Erstkauf; Zahlungsdaten werden im Checkout erfasst.

## Technische Umsetzung

### Backend / Datenmodell

- Lizenzstatus um `trial` oder eine gleichwertige eindeutige Trial-Abbildung erweitern.
- Lizenzmodell um Trial-Daten ergänzen, z. B.:
  - `trial_started_at`
  - `trial_ends_at`
  - optional `trial_converted_at`
- API-Responses für Lizenzdetails um Trial-Ende und verbleibende Tage erweitern.
- Preview-/Checkout-Response muss ausgeben:
  - ob ein kostenloser Trial angewendet wird
  - ob ein Erstbestellungsrabatt angewendet wird
  - welcher Rabattbetrag für die Hauptlizenz gilt
  - ob der Testvorteil wegen bestehender Organisation/Lizenz entfällt
- Lizenzzählung, Pool-Logik und Concurrent-Use-Logik müssen `trial` als nutzbare Lizenz berücksichtigen.
- Webhook-/Sync-Logik muss Stripe-Status `trialing` und Trial-Ende korrekt auf Normdex-Lizenzen abbilden.
- Beim Ablauf des Trials muss ein Event geschrieben werden, z. B. `license.trial_converted`.
- Rabattberechnung für den Erstbestellungsrabatt ist einheitlich:
  - monatlich und jährlich: 24,50 EUR Rabatt auf die erste Hauptlizenz der ersten Rechnung
  - Stripe-Gutschein: `ÖNORM M 7140 Basic – Neukundenrabatt`, Coupon-ID `QHQESezY`

### Stripe

- Für qualifizierte Erstkäufe einer einzelnen Lizenz soll Stripe Checkout eine Subscription mit 14 Tagen Trial erzeugen, z. B. über `trial_period_days=14` oder die äquivalente aktuelle Stripe-Konfiguration.
- Stripe muss Zahlungsdaten erfassen, damit die Lizenz nach Ablauf automatisch abgerechnet werden kann.
- Für qualifizierte Erstkäufe mit mehreren Lizenzen soll Stripe keinen Trial starten, sondern den Erstbestellungsrabatt einmalig auf die erste Rechnung bzw. die Hauptlizenz abbilden.
- Beim Kauf weiterer Lizenzen während eines aktiven Trials muss die bestehende Trial-Subscription vor dem Zusatzkauf beendet bzw. sofort zahlungspflichtig gestellt werden.
- Stripe-Testmode-Objekte und Webhook-Verhalten müssen dokumentiert werden.

### Frontend / Lizenzverwaltung

- Die Lizenzverwaltung zeigt für Testlizenzen ein eigenes Badge, z. B. `Test Lizenz · 14 Tage übrig`.
- Die verbleibenden Tage werden dynamisch aus `trial_ends_at` bzw. einem Backend-Feld berechnet/angezeigt.
- Testlizenzen dürfen nicht als „Aktiv“ oder „Ausstehend“ missverständlich dargestellt werden.
- Der Kaufdialog zeigt bei qualifiziertem Einzelkauf den 14-Tage-Testzeitraum transparent an.
- Der Kaufdialog zeigt bei qualifiziertem Mehrfachkauf klar an, dass der 14-Tage-Testvorteil automatisch in einen Erstbestellungsrabatt umgewandelt wird.
- Der Kaufdialog weist den Rabattbetrag für die Hauptlizenz und die regulär berechneten Zusatzlizenzen nachvollziehbar aus.
- Wenn ein User während eines aktiven Trials weitere Lizenzen kaufen möchte, muss die UI vorab erklären, dass der Trial dadurch endet und die erste Lizenz zahlungspflichtig wird.

### Landingpage / Legal

- Landingpage-Preise und CTAs müssen monatliche und jährliche Lizenzkäufe mit 14-Tage-Testvorteil unterstützen.
- Landingpage, Kaufdialog und Checkout müssen den Unterschied zwischen kostenlosem Testzeitraum und Erstbestellungsrabatt verständlich erklären.
- AGB-/Legal-Textentwurf erstellen:
  - 14 Tage kostenloser Testzeitraum beim qualifizierten Einzelkauf
  - Umwandlung des Testvorteils in einen Erstbestellungsrabatt beim qualifizierten Mehrfachkauf
  - monatlicher und jährlicher Mehrfachkauf: einheitlicher Erstbestellungsrabatt von 24,50 EUR auf die Hauptlizenz der ersten Rechnung
  - automatische Umwandlung in eine zahlungspflichtige Lizenz nach Ablauf
  - automatische Abbuchung nach Ablauf über die hinterlegte Zahlungsmethode
  - kein neuer Testvorteil bei bestehenden Kunden
  - Ende des Testzeitraums bei nachträglichem Hinzufügen weiterer Lizenzen
  - aliquote Gutschrift der verbleibenden Trial-Tage bei vorzeitiger Umwandlung wegen Zusatzkauf
- Der Legal-Textentwurf muss ausdrücklich mit dem Hinweis „juristisch prüfen“ markiert werden.

## Betroffene Orientierungspunkte

- `apps/api/app/models.py`
- `apps/api/app/routers/licenses_v2.py`
- `apps/api/app/routers/subscriptions.py`
- `apps/api/app/domain/license_pricing.py`
- `apps/frontend/src/pages/Licenses.tsx`
- `apps/frontend/src/api.ts`
- Vault-Dokumentation zu App-Funktionen, Marketing-/Pricing-Hinweisen und AGB/Legal
- Landingpage-Repo bzw. Landingpage-Dokumentation, sofern die Kauf-CTAs dort gepflegt werden

## Akzeptanzkriterien

- Ein Neukunde kauft genau eine Monatslizenz und erhält eine voll nutzbare 14-Tage-Testlizenz.
- Ein Neukunde kauft genau eine Jahreslizenz und erhält eine voll nutzbare 14-Tage-Testlizenz.
- Nach Ablauf des Trials wird die Lizenz automatisch aktiv und zahlungspflichtig.
- Ein Neukunde kauft mehrere Monatslizenzen und erhält auf die erste Rechnung einmalig 24,50 EUR Erstbestellungsrabatt; Zusatzlizenzen werden regulär berechnet.
- Ein Neukunde kauft mehrere Jahreslizenzen und erhält auf die erste Rechnung einmalig 24,50 EUR Erstbestellungsrabatt; Zusatzlizenzen werden regulär berechnet.
- Mehrfachkäufe zeigen vor Kaufabschluss eindeutig, dass kein kostenloser Testzeitraum startet, sondern der 14-Tage-Testvorteil in einen Rabatt umgewandelt wird.
- Bestehende Kunden erhalten bei weiteren Einzelkäufen keinen neuen Testvorteil.
- Bei Zusatzkauf während eines aktiven Trials wird der Trial zuerst beendet bzw. in eine zahlungspflichtige Lizenz überführt; die verbleibenden Trial-Tage werden als aliquote Gutschrift auf die Hauptlizenz ausgewiesen und angerechnet.
- Die Lizenzverwaltung zeigt ein eigenes Trial-Badge mit verbleibenden Tagen.
- Trial-Lizenzen funktionieren mit dem bestehenden Concurrent-Use-/Heartbeat-Locking.
- Stripe-Checkout erfasst Zahlungsdaten und bucht erst nach Ablauf des Trials ab.
- Webhooks und manuelle Sync-/Confirm-Flows bleiben idempotent.
- Trial-Benefit-Missbrauchsschutz ist wirksam: `organizations.trial_used_at` sperrt den Vorteil dauerhaft pro Organisation, auch wenn Trial-Lizenzen später gelöscht oder beendet werden.
- Parallel gestartete Trial-Checkouts werden über Pending Orders und den frühen Org-Lock blockiert.
- Abgebrochene Trial-Checkouts geben den Trial-Lock nur frei, wenn Stripe keine Subscription erzeugt hat; bei kompletter Stripe-Session oder Stripe-Fehler bleibt der Lock erhalten.
- Migrationen übernehmen bestehende Trial-/Benefit-Historie per Backfill in `organizations.trial_used_at`, damit alte Metadaten nicht durch späteres Löschen der Lizenzhistorie verlorengehen.
- AGB-/Legal-Textentwurf ist erstellt und als juristisch zu prüfen markiert.
- Relevante Vault-Dokumentation ist aktualisiert.

## Tests / Verifikation

- Backend-Test: Neukunde kauft genau eine Monatslizenz → Lizenz wird `trial`, Stripe-Subscription ist `trialing`, keine sofortige Abbuchung.
- Backend-Test: Neukunde kauft genau eine Jahreslizenz → gleicher Trial-Flow mit Jahrespreis nach Ablauf.
- Backend-Test: Neukunde kauft mehrere Monatslizenzen → kein Trial, erste Rechnung erhält einmalig 24,50 EUR Erstbestellungsrabatt, Zusatzlizenzen regulär.
- Backend-Test: Neukunde kauft mehrere Jahreslizenzen → kein Trial, erste Rechnung erhält einmalig 24,50 EUR Erstbestellungsrabatt, Zusatzlizenzen regulär.
- Backend-Test: Preview weist Trial, Erstbestellungsrabatt, Rabattbetrag und reguläre Zusatzlizenzen korrekt aus.
- Backend-Test: Organisation mit bestehender Lizenz kauft eine weitere Lizenz → kein neuer Testvorteil.
- Backend-Test: Organisation mit aktiver Trial-Lizenz will weitere Lizenz kaufen → Trial wird zuerst beendet/konvertiert, verbleibende Trial-Tage werden als Gutschrift berechnet, danach Zusatzkauf möglich.
- Backend-Test: Monatliche Trial-Lizenz wird nach 7 von 14 Tagen wegen Zusatzkauf konvertiert → 7 verbleibende Tage ergeben 12,25 EUR Gutschrift auf die Hauptlizenz.
- Webhook-/Sync-Test: Trial läuft ab → Lizenz wird automatisch `active`, Laufzeitdaten werden synchronisiert, Event wird geschrieben.
- Backend-Test: abgebrochener Trial-Checkout ohne Stripe-Subscription gibt `trial_used_at` wieder frei und verworfene Order-/License-Metadaten blockieren keinen neuen Trial.
- Backend-Test: abgebrochener Checkout mit bereits kompletter Stripe-Session behält den Trial-Lock.
- Backend-Test: verworfene/fehlgeschlagene Checkout-Artefakte mit Trial-Metadaten werden im Legacy-Fallback ignoriert.
- Backend-Test: bestehende Trial-/Benefit-Historie wird durch `trial_used_at`, gleiche `stripe_customer_id`, gleiche `vat_id`, Pending Orders und Legacy-Metadaten erkannt.
- Frontend-Test: Badge zeigt „Test Lizenz“ und verbleibende Tage korrekt.
- Frontend-Test: Mehrfachkauf zeigt klaren Hinweis, dass der Testvorteil in einen Erstbestellungsrabatt umgewandelt wird.
- Frontend-Test: Kaufdialog weist den Erstbestellungsrabatt verständlich und rechnerisch nachvollziehbar aus.
- Smoke-Checks: relevante Backend-Lizenztests, Frontend-Test/build und Stripe-Testmode-Checkout oder dokumentierter Dry-Run.

## Offene Prüfpunkte

- Erledigt: Stripe-Konfiguration für Trial in `licenses_v2.py:1482` umgesetzt (`subscription_data["trial_period_days"] = LICENSE_TRIAL_BENEFIT_DAYS`). Stripe erfasst Zahlungsdaten im Checkout und bucht erst nach Trial-Ablauf ab.
- Erledigt: Stripe-Gutschein für den einheitlichen Erstbestellungsrabatt ist `ÖNORM M 7140 Basic – Neukundenrabatt`, Coupon-ID `QHQESezY`, fixer Betrag 24,50 EUR.
- Erledigt: `trial` wird als eigener Status modelliert, ergänzt um Trial-Felder. Begründung: UI, API, Statuszählung, Webhooks und Audit-Events bleiben dadurch eindeutig.
- Erledigt: Missbrauchsschutz-Grundlage ist implementiert: `organizations.trial_used_at`, Pending-Order-Sperre, Cross-Checks über `stripe_customer_id` und `vat_id`, Checkout-Cancel-Freigabe nur ohne Stripe-Subscription, sowie Backfill-Migration für bestehende Trial-/Benefit-Historie.
- Erledigt: Marketing-Grundformulierung angepasst: kein „ohne Kreditkarte", da Stripe Zahlungsdaten im Checkout erfasst.
- Erledigt: Landingpage-Repo nutzt App-Route `/auth/register?plan=...&qty=...` als Checkout-Einstieg. Die App `/licenses` bleibt zusätzliche Kaufoberfläche für bestehende Organisationen.
- Erledigt: Produktentscheidung gemischte Erstbestellung: erlaubt; `calculate_trial_benefit()` wählt deterministisch den ersten verfügbaren Pool (monatlich vor jährlich), Rabatt wird einmalig in Höhe von 24,50 EUR auf eine Hauptlizenz angewendet. Dokumentiert in [[Funktionen im Detail]] und [[AGB]] §4.2.2.
- Erledigt: Kommunikationsdetail im Kaufdialog bei Zusatzkauf während Trial: Banner „Die Testphase endet bei erfolgreicher Zahlung. X verbleibende Testtage werden als Gutschrift abgezogen." plus separate Zeile „Gutschrift für verbleibende Testtage". Siehe [Licenses.tsx:563-591](apps/frontend/src/pages/Licenses.tsx).
- Juristische Prüfung des AGB-Textentwurfs (§4.2.1 - §4.2.4) einplanen.

## Notizen / Fortschritt

- Angelegt am 2026-04-30 aus Produktanforderung zum 14-tägigen Testzeitraum für Neukunden-Lizenzen.
- 2026-04-30: Produktlogik erweitert: Bei Erstbestellung mehrerer Lizenzen entfällt der Testvorteil nicht ersatzlos, sondern wird als Erstbestellungsrabatt auf die Hauptlizenz umgewandelt. Sprachgebrauch auf `14-Tage-Testvorteil`, `kostenloser Testzeitraum` und `Erstbestellungsrabatt` präzisiert.
- 2026-04-30: Fachliche Neukundenregel bestätigt: Der Testvorteil gilt nur, wenn die Organisation keinerlei relevante Lizenzhistorie (`active`, `scheduled_end`, `payment_failed`, `pending`, `trial`) und keinen bereits dokumentierten Testvorteil hat.
- 2026-04-30: Technischer Start beschlossen: `trial` wird als eigener Lizenzstatus mit `trial_started_at`, `trial_ends_at` und `trial_converted_at` umgesetzt. Erste Implementierungsschritte: Datenmodell/Migration und Preview-/Pricing-Entscheidung.
- 2026-04-30: Datenmodell begonnen: `License` um `trial_started_at`, `trial_ends_at`, `trial_converted_at` erweitert und Alembic-Folgemigration `b7c8d9e0f1a2_add_license_trial_fields.py` angelegt. Migration ist noch nicht lokal angewendet.
- 2026-04-30: Pricing-Domain begonnen: Berechnung für `free_trial` vs. `first_order_discount` ergänzt. Mehrfach-Erstkauf monatlich und jährlich: einheitlich 24,50 EUR Rabatt. Gemischte Erstbestellungen bleiben als fachlicher Prüfpunkt offen.
- 2026-04-30: Preview-API begonnen: `/licenses/checkout/preview` gibt jetzt Trial-Benefit-Metadaten aus (`trial_benefit_kind`, `free_trial_applies`, `first_order_discount_applies`, Rabattbetrag, Begründung). Der Rabatt wird noch nicht in `total_gross` eingerechnet, bis die Stripe-Checkout-Anwendung im Kauf-Flow umgesetzt ist.
- 2026-04-30: Trial-Status teilweise in bestehende Lizenzlogik eingebunden: API-Schemas enthalten Trial-Felder, Pool-Zählung behandelt `trial` als relevante/nutzbare Lizenz, und Heartbeat-/Concurrent-Use-Start erlaubt nicht abgelaufene Trial-Lizenzen.
- 2026-04-30: Erste Tests ergänzt und ausgeführt: `.\venv\Scripts\python -m pytest tests\test_license_pricing.py` ergibt 33/33 grün. Syntaxprüfung der geänderten Backend-Dateien per AST-Parse ist grün. `py_compile` war wegen fehlender Schreibrechte auf `__pycache__` nicht nutzbar.
- 2026-04-30: Alembic-Konsistenz geprüft: `.\venv\Scripts\python -m alembic heads` zeigt `b7c8d9e0f1a2` als einzigen Head.
- 2026-04-30: Regel für Zusatzkauf während Trial angepasst: Die Trial-Lizenz wird weiterhin vor dem Zusatzkauf in eine zahlungspflichtige Hauptlizenz überführt, aber verbleibende Trial-Tage werden aliquot als Gutschrift auf die Hauptlizenz angerechnet und müssen im Kaufdialog transparent kommuniziert werden.
- 2026-04-30: Pricing-Domain für Trial-Vorzeitumwandlung ergänzt: `calculate_trial_conversion_credit()` berechnet die Gutschrift einheitlich aus `24,50 EUR / 14 * verbleibende Trial-Tage`, unabhängig von monatlicher oder jährlicher Lizenz.
- 2026-04-30: Stripe-Konfiguration präzisiert: Der Gutschein heißt `ÖNORM M 7140 Basic – Neukundenrabatt`, hat den fixen Betrag 24,50 EUR und die Coupon-ID `QHQESezY`. Die Development-`.env` enthält nun `STRIPE_COUPON_ID_NEW_CUSTOMER_DISCOUNT=QHQESezY`.
- 2026-04-30: Lokale Webhook-Leitung geprüft: Backend `/subscriptions/config` erreichbar, `/subscriptions/webhook` lehnt GET korrekt mit `405 Method Not Allowed` ab und unsignierte POSTs mit `Ungültige Signatur`. `stripe trigger checkout.session.completed` wurde über Stripe CLI erfolgreich ausgelöst; Backend hat das signierte Event empfangen und erwartbar als `WEBHOOK_WARN` protokolliert, weil das generische Stripe-Fixture keine Normdex-Order/Subscription enthält.
- 2026-04-30: Missbrauchsschutz für Trial-Benefit implementiert. Neues Feld `trial_used_at` (DateTime, nullable) auf `organizations` als dauerhafter Lock, unabhängig vom Lizenzstatus. Alembic-Migration `c1d2e3f4a5b6_add_org_trial_used_at.py` angelegt und auf Dev-DB angewendet. `_has_trial_benefit_history()` um folgende Prüfschichten erweitert: (1) `org.trial_used_at` als primärer permanenter Lock, (2) Cross-Identity-Check `stripe_customer_id` über alle Orgs, (3) Cross-Identity-Check `vat_id` über alle Orgs, (4) Pending Orders mit Trial als Parallelschutz gegen Race Conditions, (5) Legacy-Metadaten-Fallback für Altdaten. `checkout/create` setzt `org.trial_used_at` sofort nach Erstellung der Pending-Lizenzen, vor der Stripe-Session. `checkout/cancel` prüft Stripe-Session-Status und gibt `trial_used_at` nur frei, wenn keine Subscription entstanden ist; bei Stripe-Fehler bleibt der Lock erhalten. `checkout/confirm` setzt `trial_used_at` defensiv als Fallback. Admin-seitiges Löschen von Trial-Lizenzen reaktiviert den Trial nicht, da der Lock auf der Org liegt. 7 neue Unit-Tests für `_has_trial_benefit_history()` ergänzt, 40/40 Tests grün.
- 2026-05-01: Trial-Erinnerungs-E-Mail implementiert. `_handle_subscription_trial_will_end()` in `apps/api/app/routers/subscriptions.py` sendet jetzt eine E-Mail an den Org-Owner (Membership.role="owner"), sobald Stripe den Webhook `customer.subscription.trial_will_end` feuert (fest 3 Tage vor Trial-Ende, nicht konfigurierbar). Neue Templates `tpl_trial_will_end()` und `tpl_trial_will_end_html()` in `apps/api/app/emails.py` ergänzt (Du-Form, Brevo-Branding). Fehler beim E-Mail-Versand werden geloggt, blockieren aber den Webhook-Handler nicht. Hinweis für die FAQ-Kommunikation: „einige Tage vor Ablauf" statt „7 Tage", da Stripe den Zeitpunkt vorgibt.
- 2026-05-01: Missbrauchsschutz nachgeschärft: `checkout/cancel` markiert freigegebene Trial-Benefits jetzt mit `trial_benefit_released_at` und `trial_benefit_release_reason`, wenn Stripe keine Subscription erzeugt hat. `_has_trial_benefit_history()` ignoriert verworfene Pending-Lizenzen mit `checkout_discarded_at` sowie fehlgeschlagene/freigegebene Orders, sodass ein sauber abgebrochener Checkout den Trial nicht dauerhaft blockiert. Bei kompletter Stripe-Session oder Stripe-Fehler bleibt der Org-Lock bestehen.
- 2026-05-01: Backfill-Migration ergänzt: `c1d2e3f4a5b6_add_org_trial_used_at.py` setzt `organizations.trial_used_at` anhand bestehender `licenses.meta` und `license_orders.meta`, ignoriert aber verworfene/fehlgeschlagene Checkout-Artefakte. JSON-Metadaten werden robust als Dict oder JSON-Text verarbeitet.
- 2026-05-01: Verifikation nach Missbrauchsschutz-Fix: `.\venv\Scripts\python -m pytest` in `apps/api` ergibt 127/127 grün. AST-Syntaxcheck für `licenses_v2.py`, neue/angepasste Tests und `c1d2e3f4a5b6_add_org_trial_used_at.py` grün. `py_compile` bleibt wegen `__pycache__`-Rechten ungeeignet.
- 2026-05-30: Landingpage-Update: `Pricing.tsx` zeigt jetzt dynamisch je nach Mengenauswahl entweder den 14-Tage-Trial-Banner oder den Erstbestellungsrabatt-Hinweis (24,50 EUR auf die Hauptlizenz). Zwei neue FAQ-Einträge zum Mehrfachkauf und zum Zusatzkauf während Trial. `Hero.tsx` und `CTA.tsx` enthalten keine fachlich falsche "Keine Kreditkarte erforderlich"-Aussage mehr, sondern erklären Trial vs. Erstbestellungsrabatt.
- 2026-05-30: AGB §4.2 aufgeteilt in §4.2.1 (Kostenloser Testzeitraum beim Einzel-Erstkauf), §4.2.2 (Erstbestellungsrabatt 24,50 EUR bei Mehrfachkauf, inkl. gemischter Erstbestellungen), §4.2.3 (vorzeitige Trial-Umwandlung mit aliquoter Gutschrift `24,50 EUR / 14 * verbleibende Tage`) und §4.2.4 (kein neuer Testvorteil bei bestehenden Kunden). Entwurf ist mit "juristisch prüfen" markiert.
- 2026-05-30: Produktentscheidung gemischte Erstbestellung: erlaubt, einmaliger Rabatt von 24,50 EUR auf eine Hauptlizenz; deterministische Pool-Wahl (monatlich vor jährlich) durch `calculate_trial_benefit()`. Vault-Doku [[Funktionen im Detail]] entsprechend ergänzt.
- 2026-05-30: Frontend-Helfer in `apps/frontend/src/lib/licenseTrial.ts` extrahiert (`trialDaysLeft`, `trialBadgeLabel`, `deriveCheckoutHint`). Zugehöriger Vitest-Test deckt Trial-Badge-Label, Tageszähler-Edge-Cases und Kaufdialog-Hint-Priorität (Trial-Conversion vor Free-Trial vor First-Order-Discount) ab — 12/12 grün. `Licenses.tsx` nutzt nun `trialBadgeLabel` im StatusBadge.
- 2026-05-30: Stripe-Testmode End-to-end-Prüfablauf dokumentiert in [[Stripe Testmode Dry-Run - Trial und Erstbestellungsrabatt]] (7 Szenarien: Einzel-Trial, Mehrfach-Rabatt, gemischter Kauf, Trial-Konvertierung mit Gutschrift, Trial-Abbruch, Bestandskunde, Trial-Erinnerungs-E-Mail).
