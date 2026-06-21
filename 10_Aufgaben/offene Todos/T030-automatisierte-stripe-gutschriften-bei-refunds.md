# T030 · Automatisierte Stripe-Gutschriften bei Refunds

**Phase:** App-Betrieb / Verwaltungsportal / Stripe / Rechnungslegung  
**Priorität:** P0 · Buchhaltung / Release-Gate  
**Status:** offen  
**Datum:** 2026-06-21

## Ziel

Jede Erstattung einer bereits bezahlten Stripe-Rechnung wird in Normdex vollständig, nachvollziehbar und möglichst ohne manuelle Nacharbeit abgewickelt.

Der Zielprozess umfasst als zusammengehörigen Vorgang:

1. fachliche Ermittlung des zu erstattenden Betrags,
2. eindeutige Zuordnung zur ursprünglichen Stripe-Rechnung und Rechnungsposition,
3. Erstellung einer Stripe Credit Note (Gutschrift),
4. Rückzahlung an das ursprüngliche Zahlungsmittel,
5. Verknüpfung von Credit Note und Refund,
6. automatische Bereitstellung bzw. Zustellung des Gutschrift-PDFs an den Kunden,
7. vollständiges Audit-Logging und
8. automatische Fehlererkennung mit klarer, möglichst handlungsarmer Nachbearbeitung.

Das Verwaltungsportal soll nicht nur Geld zurücküberweisen, sondern den gesamten kaufmännischen Beleg- und Geldfluss korrekt abschließen.

## Leitprinzip: Automation First

Billing-, Rechnungs- und Erstattungsprozesse sollen in Normdex grundsätzlich so automatisiert wie möglich ablaufen.

Das bedeutet insbesondere:

- Ein fachlicher Vorgang wird nach einer bewussten Bestätigung durch den Admin vollständig im Hintergrund ausgeführt.
- Der Admin soll nicht zusätzlich im Stripe-Dashboard eine Gutschrift anlegen, PDFs suchen, E-Mails versenden oder IDs manuell übertragen müssen.
- Technische Teilvorgänge werden idempotent und wiederholbar umgesetzt.
- Fehler werden nicht stillschweigend verschluckt.
- Bei Teilerfolg bleibt der genaue Zustand sichtbar und kann gezielt fortgesetzt werden.
- Manuelle Stripe-Arbeit ist nur ein dokumentierter Ausnahme- und Recovery-Pfad.
- Dieselbe Automationsanforderung gilt künftig generell für vergleichbare Billing-Vorgänge, insbesondere Refunds, Credit Notes, Rechnungszustellung, Kündigungen und notwendige Kundenbenachrichtigungen.

## Warum dieses Todo kritisch ist

Eine Rückerstattung ist nicht dasselbe wie eine Gutschrift:

- Der **Refund** bildet den Geldfluss zurück zum Kunden ab.
- Die **Credit Note** berichtigt die bereits ausgestellte Rechnung und erzeugt den kaufmännischen Beleg.
- Ohne Credit Note kann Geld zurückgezahlt worden sein, während die ursprüngliche Rechnung weiterhin unberichtigt über den vollen Betrag besteht.
- Der Kunde benötigt einen nachvollziehbaren Beleg für seine Buchhaltung und gegebenenfalls für die Korrektur seines Vorsteuerabzugs.
- Normdex benötigt eine nachvollziehbare Grundlage für Erlös-, Umsatzsteuer- und Zahlungsabstimmung.

Das Thema ist vor dem Produktivbetrieb fachlich und technisch abzuschließen. Die konkrete umsatzsteuerliche Ausgestaltung ist vor Live-Freigabe mit Steuerberatung bzw. Buchhaltung zu verifizieren.

## Aktueller Zustand

### Verwaltungsportal

Das Verwaltungsportal unterstützt:

- vollständige und partielle Stripe-Refunds,
- Sofortkündigung von Lizenzen,
- Berechnung eines aliquoten Erstattungsbetrags,
- Auswahl bzw. Eingabe des Refund-Betrags,
- Refund-Grund,
- Pflichtbegründung und optionalen Ticket-Kontext,
- Stripe Refund,
- Audit-Log und UI-Erfolgsmeldung.

### Technische Implementierung

Relevante Codepfade:

- `apps/api/app/routers/admin.py`
  - `admin_billing_payment_refund()`
  - `admin_licenses_cancel_now()`
  - `_admin_licenses_cancel_now_preview()`
  - `_find_refundable_charge_for_subscriptions()`
- `apps/frontend/src/pages/admin/OrganizationCase.tsx`
  - Refund-Workflow
  - „Sofortkündigung mit Aliquot-Erstattung"

Der Backend-Code ruft derzeit direkt `stripe.Refund.create(...)` auf.

Nicht umgesetzt sind:

- Erstellung einer Stripe Credit Note,
- Verknüpfung der Gutschrift mit der ursprünglichen Rechnung,
- Verknüpfung eines bestehenden Refunds mit der Credit Note,
- Erzeugung und Speicherung eines Gutschrift-PDF-Links,
- automatische Gutschrift-E-Mail,
- fachliche Aufteilung auf die ursprünglichen Rechnungspositionen und Steuern,
- eigener persistenter Workflow-Status für den Gesamtvorgang,
- automatische Wiederaufnahme nach Teilfehlern.

### Beobachteter Sandbox-Fall vom 21. Juni 2026

Im Dev-Verwaltungsportal wurde eine Stripe-Lizenz sofort gekündigt und eine aliquote Erstattung von 16,00 EUR ausgelöst.

Beobachtung:

- Stripe Refund über 16,00 EUR erfolgreich.
- Ursprüngliche Zahlung: 49,00 EUR.
- Stripe zeigte das Ereignis „Eine Rückerstattung von 16 Euro aus einer Zahlung über 49 Euro wurde abgeschlossen."
- Es wurde keine neue Credit Note erzeugt.
- Es wurde kein Gutschrift-PDF erzeugt.
- Die zugehörige Rechnung zeigte weiterhin `post_payment_credit_notes_amount = 0`.

Zusätzlich waren auf derselben Sandbox-Zahlung bereits 33,00 EUR erstattet. Mit dem neuen Refund über 16,00 EUR war die Zahlung insgesamt vollständig über 49,00 EUR refundiert. Dieser Testfall muss bei der Bereinigung als zusammenhängender Vorgang betrachtet werden; es darf nicht nochmals Geld ausgezahlt werden.

Die Zahlung war über den Payment Intent fachlich einer Stripe-Rechnung zuordenbar, der Charge selbst enthielt jedoch keine direkte `invoice`-Referenz. Der aktuelle Code orientiert sich primär am Charge und verliert dadurch den Rechnungs- und Belegkontext.

### Bewertung

Der aktuelle Geldfluss funktioniert, der Belegfluss ist unvollständig.

Das ist:

- kein technischer Fehler der Stripe Refund API,
- sondern eine funktionale Lücke in der Normdex-Implementierung,
- ein buchhalterisches Risiko,
- ein P0-Blocker für einen vollständig automatisierten Produktivbetrieb.

## Zielzustand

### Fachlicher Zielprozess

Der Admin startet im Verwaltungsportal einen einzigen geführten Vorgang, zum Beispiel:

> „Lizenz sofort kündigen und aliquot gutschreiben"

Nach der Bestätigung führt Normdex automatisch aus:

1. Betroffene Lizenz und Stripe Subscription eindeutig ermitteln.
2. Ursprüngliche Stripe Invoice und deren Zahlung eindeutig auflösen.
3. Erstattungsbetrag berechnen und gegen bereits erfolgte Refunds und Credit Notes prüfen.
4. Credit Note Preview bei Stripe erstellen bzw. fachlich validieren.
5. Credit Note mit passenden Rechnungspositionen und Steuerinformationen erzeugen.
6. Refund im selben Credit-Note-Vorgang erzeugen oder einen bereits vorhandenen Refund korrekt verknüpfen.
7. Subscription bzw. Subscription Item ohne unerwünschte zusätzliche Stripe-Proration beenden.
8. Lokale Lizenz beenden.
9. Credit-Note-ID, Nummer, PDF-Link, Invoice-ID, Refund-ID und Beträge speichern.
10. Gutschrift automatisch an die Rechnungsadresse bzw. den vorgesehenen Rechnungsempfänger senden.
11. Den Gesamtvorgang im Audit-Log protokollieren.
12. Den finalen Zustand im Verwaltungsportal anzeigen.

### Erwartete UI

Die Vorschau muss mindestens anzeigen:

- ursprüngliche Rechnungsnummer,
- Rechnungsdatum,
- betroffene Rechnungsposition bzw. Lizenz,
- ursprünglicher Bruttobetrag,
- bereits erstatteter Betrag,
- bereits gutgeschriebener Betrag,
- neuer Gutschriftbetrag,
- darin enthaltener Steueranteil bzw. Steuerbehandlung,
- Betrag, der tatsächlich auf das Zahlungsmittel zurückfließt,
- Empfänger der Gutschrift,
- Hinweis, dass Gutschrift und E-Mail automatisch erstellt werden.

Nach Ausführung:

- Status „Gutschrift und Erstattung abgeschlossen",
- Credit-Note-Nummer,
- Link zum Stripe-Gutschrift-PDF,
- Refund-ID,
- Versandstatus,
- Zeitpunkt,
- bei Fehlern ein eindeutiger Teilstatus mit gezielter Wiederholungsaktion.

### Automatisierungsziel

Im Normalfall ist nach der einmaligen Admin-Bestätigung keine weitere manuelle Tätigkeit erforderlich.

## Technisches Sollkonzept

### 1. Rechnung statt Charge als führender Kontext

Für rechnungsbezogene Erstattungen ist die Stripe Invoice das führende Objekt.

Die Auflösung muss robust über folgende Beziehungen funktionieren:

```text
License
  -> Stripe Subscription / Subscription Item
  -> Stripe Invoice
  -> Invoice Payment / Payment Intent
  -> Charge
  -> Refund
  -> Credit Note
```

Fallbacks dürfen nicht einfach den neuesten Customer-Charge verwenden, wenn die Rechnung nicht eindeutig zugeordnet werden kann. Bei Mehrdeutigkeit muss der Vorgang vor einer Geldbewegung stoppen.

### 2. Credit Note und Refund als atomarer fachlicher Workflow

Bevorzugter Neufall:

- Credit Note auf der bezahlten Invoice erzeugen.
- Stripe über `refund_amount` die passende Rückzahlung im selben Vorgang erstellen lassen.
- Credit Note muss die ursprünglichen Rechnungspositionen und Steuerinformationen referenzieren.

Fall mit bereits vorhandenem Refund:

- keine zweite Rückzahlung erzeugen,
- bestehenden Refund über den von Stripe vorgesehenen `refunds`-Parameter mit der Credit Note verknüpfen,
- Gesamtbetrag aus bereits verknüpften Refunds, Credit Notes und neuer Gutschrift vorab validieren.

### 3. Betrags- und Steuerlogik

Vor Ausführung muss geprüft werden:

- Betrag größer 0,
- Betrag nicht höher als der noch gutschreibbare Rechnungsbetrag,
- keine Doppelgutschrift,
- keine Doppelerstattung,
- Summe aus Refunds und Credit Notes überschreitet die ursprüngliche Zahlung nicht,
- Währung stimmt überein,
- Steuerbehandlung wird aus der Originalrechnung übernommen,
- Rundungsdifferenzen werden kontrolliert behandelt,
- Teilgutschriften werden einer konkreten Rechnungsposition oder einer fachlich begründeten Custom Line zugeordnet.

Bei Stripe Tax bzw. Automatic Tax sind die Stripe-Vorgaben für Credit-Note-Lines einzuhalten. Keine manuell erfundene Steueraufteilung verwenden.

### 4. Persistenter Workflow-Status

Ein Refund-/Credit-Note-Vorgang darf nicht nur über lose Audit-Einträge nachvollziehbar sein.

Es ist zu entscheiden, ob:

- eine eigene Tabelle, zum Beispiel `billing_adjustments`, eingeführt wird, oder
- ein bestehendes geeignetes Modell erweitert wird.

Mindestens zu speichern:

- interne Vorgangs-ID,
- Organisation,
- Lizenz(en),
- Stripe Customer,
- Subscription,
- Invoice,
- Payment Intent,
- Charge,
- Refund,
- Credit Note,
- Credit-Note-Nummer,
- PDF-Link,
- Brutto-, Netto- und Steuerbeträge,
- Währung,
- Grund,
- Support-Ticket,
- auslösender Admin,
- Status und Teilstatus,
- Fehlermeldung,
- Versandstatus,
- Erstellungs- und Abschlusszeitpunkte,
- Idempotency Keys.

Empfohlene Statuswerte:

```text
previewed
pending
subscription_adjusted
credit_note_created
refund_created
document_sent
completed
partially_failed
failed
manual_review_required
```

### 5. Idempotenz und Wiederaufnahme

Jeder externe Stripe-Schreibvorgang benötigt einen stabilen Idempotency Key.

Wiederholte Requests, Browser-Reloads, Timeouts oder Worker-Neustarts dürfen nicht zu:

- mehrfacher Kündigung,
- doppeltem Refund,
- doppelter Credit Note oder
- doppeltem E-Mail-Versand

führen.

Bei einem Teilfehler muss Normdex erkennen, welche Schritte bereits erfolgreich waren, und nur die fehlenden Schritte wiederholen.

### 6. Hintergrundverarbeitung

Der Ablauf soll nach Bestätigung als persistenter Hintergrundjob ausgeführt werden.

Anforderungen:

- API-Request erzeugt den Vorgang und startet bzw. queued die Verarbeitung.
- UI erhält eine Vorgangs-ID und zeigt den Status.
- Verarbeitung überlebt API-Neustarts.
- Retry mit begrenzter Anzahl und Backoff bei temporären Stripe-/SMTP-Fehlern.
- Keine rein prozesslokale, flüchtige BackgroundTask für kritische Billing-Schritte.
- Fehlgeschlagene Vorgänge erscheinen im Verwaltungsportal und optional als Admin-Notification.
- Bei dauerhaftem Fehler wird eine klare manuelle Recovery-Anleitung angezeigt.

Vor Umsetzung ist zu entscheiden, ob dafür:

- ein DB-basierter Job-Mechanismus,
- der vorhandene Scheduler mit persistenter Jobtabelle oder
- eine dedizierte Queue

verwendet wird. Für den aktuellen Umfang ist ein robuster DB-basierter Workflow wahrscheinlich ausreichend; die Entscheidung muss dokumentiert werden.

### 7. E-Mail und Dokumentzustellung

Nach erfolgreicher Credit Note:

- Gutschrift-PDF über Stripe bereitstellen,
- Kunden automatisch informieren,
- Rechnungsnummer und Gutschriftnummer nennen,
- Betrag und Grund verständlich anführen,
- PDF direkt anhängen oder über einen stabilen Stripe-Link bereitstellen,
- Versand in Outbox bzw. Audit erfassen,
- E-Mail-Versand idempotent gestalten.

Zu prüfen:

- ob Stripe die Credit Note automatisch zuverlässig an den Kunden sendet,
- ob Normdex zusätzlich eine eigene gebrandete Transaktionsmail senden soll,
- welche Variante für Nachvollziehbarkeit und Zustellsicherheit führend ist.

Es darf nicht unbemerkt zu doppelten E-Mails kommen.

### 8. Webhooks und Reconciliation

Relevante Stripe-Events sind zu prüfen und einzubinden, insbesondere:

- `credit_note.created`,
- `credit_note.updated`,
- `credit_note.voided`,
- `refund.created`,
- `refund.updated`,
- `refund.failed`,
- `charge.refunded`.

Ein regelmäßiger Reconciliation-Job soll offene oder widersprüchliche Vorgänge erkennen:

- Refund ohne Credit Note,
- Credit Note ohne erwarteten Refund,
- fehlgeschlagener Refund,
- fehlendes PDF,
- fehlgeschlagener E-Mail-Versand,
- lokale und Stripe-seitige Beträge stimmen nicht überein.

Das System soll solche Fälle automatisch nachverarbeiten oder als `manual_review_required` markieren.

## Migration und Bereinigung bestehender Fälle

Vor Produktivfreigabe ist eine Bestandsprüfung erforderlich.

### Inventarisierung

Für Dev und Produktion getrennt erfassen:

- alle Refunds,
- zugehörige Charges und Payment Intents,
- zugehörige Invoices,
- vorhandene Credit Notes,
- Differenz zwischen Refund- und Credit-Note-Beträgen,
- bereits versendete Dokumente.

### Klassifizierung

Jeder Fall wird klassifiziert:

- vollständig und korrekt,
- Refund ohne Credit Note,
- Credit Note ohne Refund,
- teilweise verknüpft,
- keine eindeutige Rechnung,
- Betrag bereits vollständig erstattet,
- manuelle Prüfung erforderlich.

### Reparatur

Für Refunds ohne Credit Note:

- ursprüngliche Invoice eindeutig bestimmen,
- bestehenden Refund verknüpfen,
- Credit Note ohne erneute Auszahlung erzeugen,
- PDF bereitstellen bzw. versenden,
- Reparatur protokollieren.

Keine automatisierte Reparatur, wenn:

- die Invoice-Zuordnung nicht eindeutig ist,
- mehrere Rechnungspositionen oder Steuersätze fachlich unklar sind,
- der kombinierte Betrag die Rechnung überschreiten würde,
- bereits externe Buchhaltungsbelege erstellt wurden.

### Konkreter Dev-Testfall

Der Sandbox-Fall mit:

- Rechnung `NDX26-0505`,
- ursprünglichem Betrag 49,00 EUR,
- Refunds über 33,00 EUR und 16,00 EUR,
- insgesamt 49,00 EUR Erstattung,
- bisher 0,00 EUR Credit Notes

ist als Reparaturtest zu verwenden.

Vor einer Änderung ist zu prüfen, ob eine einzige Credit Note über 49,00 EUR oder zwei getrennte Credit Notes über 33,00 EUR und 16,00 EUR fachlich die bessere Abbildung ist. Es darf kein weiterer Refund erzeugt werden.

## Rechtliche und buchhalterische Prüfpunkte

Vor Produktivfreigabe mit Steuerberatung/Buchhaltung bestätigen:

- Anforderungen an österreichische Gutschriften und Rechnungsberichtigungen,
- Pflichtangaben auf der Credit Note,
- Behandlung von Netto, Umsatzsteuer und Brutto,
- Zeitpunkt der Umsatzsteuerberichtigung nach § 16 UStG,
- Auswirkung beim Kunden auf den Vorsteuerabzug,
- Aufbewahrung und Export der Belege,
- Behandlung grenzüberschreitender B2B-Fälle und Reverse Charge,
- Verhalten bei Stripe Tax bzw. unterschiedlichen Steuersätzen.

Technische Umsetzung und Tests ersetzen keine steuerliche Freigabe.

## Sicherheit und Berechtigungen

- Nur berechtigte Admins dürfen den Vorgang starten.
- Pflichtgrund und Bestätigung bleiben erforderlich.
- Optionaler Support-Ticket-Bezug bleibt erhalten.
- Preview ist schreibfrei.
- Ausführung verwendet Idempotency Keys.
- Stripe Live Mode darf nicht versehentlich aus Dev angesprochen werden.
- Dev-Tests laufen ausschließlich mit `sk_test_...`.
- Sensible Stripe-Daten und Kundeninformationen nicht vollständig in Logs schreiben.
- Audit-Einträge müssen unveränderlich und ausreichend detailliert sein.

## Umsetzungspakete

### Paket A · Fachliche Spezifikation und Stripe-Spike

- [x] Stripe Credit Note API mit aktueller Account-API-Version prüfen.
- [x] Neufall „Credit Note erzeugt Refund" in Sandbox testen.
- [x] Fall „bestehenden Refund mit Credit Note verknüpfen" in Sandbox testen.
- [x] Teilgutschrift einer Invoice Line testen.
- [x] Steuer- und Rundungsverhalten testen.
- [x] Ergebnisse und gewählte API-Parameter dokumentieren.

#### Ergebnisse des Sandbox-Spikes (2026-06-21)

Durchgeführt gegen den Dev-Stripe-Account (`acct_1TQLoiF05ipkEAzm`, Test-Mode, `sk_test_...`) mit `stripe` Python SDK 14.0.1, Library-Standard-API-Version `2025-11-17.clover`. Vier reale Szenarien wurden end-to-end gegen die Sandbox ausgeführt (Subscription → Invoice → Payment → Credit Note/Refund), die Testkunden wurden danach wieder gelöscht.

**Kritischer Befund – Invoice/Charge-Verknüpfung hat sich geändert:**

In `2025-11-17.clover` existieren `Invoice.charge` und `Invoice.payment_intent` nicht mehr, und `Charge.invoice` existiert ebenfalls nicht. Der bestätigte, einzig robuste Auflösungsweg ist:

```text
Invoice
  -> stripe.InvoicePayment.list(invoice=invoice.id)   # liefert payment.payment_intent
  -> stripe.PaymentIntent.retrieve(payment_intent_id)
  -> payment_intent.latest_charge                       # liefert Charge-ID
```

Es gibt **keinen Rückweg** von Charge → Invoice. Das bestätigt exakt die im Sandbox-Fall vom 21.6. beobachtete Lücke (Abschnitt „Beobachteter Sandbox-Fall") und das Risiko in Abschnitt „1. Rechnung statt Charge als führender Kontext": `_find_refundable_charge_for_subscriptions()` kann sich nicht auf eine Charge→Invoice-Rückverknüpfung stützen und muss zwingend vorwärts ab der Invoice/Subscription auflösen. Bei mehreren offenen Invoices pro Subscription ist ohne weitere Eingrenzung keine eindeutige Zuordnung möglich – das deckt sich mit der Vorgabe „Bei Mehrdeutigkeit muss der Vorgang vor einer Geldbewegung stoppen."

**Test 1 – Neufall „Credit Note erzeugt Refund automatisch":** ✅ erfolgreich.

```python
stripe.CreditNote.create(
    invoice=invoice.id,
    amount=1600,          # Gutschriftbetrag (netto+brutto je nach Steuer) – Pflichtfeld
    refund_amount=1600,   # Anteil, der per Refund zurückfließt
    reason="order_change", # gültige Werte: duplicate | fraudulent | order_change | product_unsatisfactory ("other" existiert NICHT mehr)
    memo="...",
)
```

Ergebnis: Stripe legt automatisch einen `Refund` an und verknüpft ihn in `credit_note.refunds`. `invoice.post_payment_credit_notes_amount` und `invoice.amount_remaining` werden korrekt aktualisiert. Wichtig: `amount` (oder `lines` oder `shipping_cost`) ist ein **Pflichtfeld** – `refund_amount` allein wird von der API abgelehnt (`Missing required param: amount or lines or shipping_cost`).

**Test 2 – Bestehenden Refund mit Credit Note verknüpfen:** ✅ erfolgreich, keine Doppelauszahlung.

```python
stripe.CreditNote.create(
    invoice=invoice.id,
    amount=1600,
    refunds=[{"refund": existing_refund.id}],   # KEIN "amount"-Subfeld – wird sonst als unknown param abgelehnt
    reason="order_change",
)
```

Nach dem Vorgang existiert weiterhin genau 1 Refund auf der Charge (verifiziert über `stripe.Refund.list(charge=...)`), die Credit Note referenziert ihn korrekt in `credit_note.refunds`. Das ist exakt der Mechanismus für die Migration/Bereinigung des realen NDX26-0505-Falls (Paket F): bestehende Refunds nachträglich verknüpfen, ohne neue Auszahlung.

**Test 3 – Teilgutschrift einer Invoice Line:** ✅ erfolgreich.

```python
stripe.CreditNote.create(
    invoice=invoice.id,
    lines=[{"type": "invoice_line_item", "invoice_line_item": line.id, "amount": 1000}],
    refund_amount=1000,
    reason="order_change",
)
```

Die Credit-Note-Line referenziert korrekt `invoice_line_item`, übernimmt Beschreibung und Bezug zur Originalposition. Damit ist eine fachlich saubere Zuordnung „Gutschrift gehört zu Lizenz/Rechnungsposition X" möglich, wie in Abschnitt 3 (Betrags- und Steuerlogik) gefordert.

**Test 4 – Steuer- und Rundungsverhalten:** ✅ erfolgreich, Steuer wird automatisch korrekt proportional übernommen.

Testaufbau: Invoice-Item netto 33,33 EUR mit `TaxRate` 20 % (exclusive) → Invoice-Total 40,00 EUR (33,33 + 6,67 Steuer, korrekt kaufmännisch gerundet). Teilgutschrift auf 11,11 EUR der Linie:

```python
stripe.CreditNote.create(
    invoice=invoice.id,
    lines=[{"type": "invoice_line_item", "invoice_line_item": line.id, "amount": 1111}],
    refund_amount=1333,
    reason="order_change",
)
```

Ergebnis: Stripe berechnet den Steueranteil der Teilgutschrift automatisch proportional aus dem ursprünglichen `tax_rate` der Invoice-Line (`1111 × 20 % = 222`, korrekt gerundet), Gesamtbetrag der Credit Note = 1111 + 222 = 1333 Cent, exakt passend zum angegebenen `refund_amount`. **Keine manuelle Steueraufteilung nötig**, solange `lines[].invoice_line_item` korrekt referenziert wird – das deckt sich mit der Vorgabe „Keine manuell erfundene Steueraufteilung verwenden."

Hinweis: Im aktuellen Stripe-Account/Produkt-Setup ist für die echten Subscription-Preise (Basic Monthly/Yearly) **kein Stripe Tax bzw. keine `tax_rates`** aktiv – die 49 €/490 € sind Bruttopreise ohne separate USt-Ausweisung auf der Invoice. Vor Produktivfreigabe ist mit Steuerberatung zu klären, ob/wie eine echte Steueraufteilung auf den Live-Rechnungen erfolgen soll; der hier verifizierte Mechanismus deckt den Fall ab, sobald `tax_rates` gesetzt sind.

**Gewählte API-Parameter (Referenz für Paket C):**

| Zweck | Parameter |
|---|---|
| Neuer Refund über Credit Note | `amount` + `refund_amount` + `reason` (kein `"other"`) |
| Bestehenden Refund verknüpfen | `amount` + `refunds=[{"refund": "<id>"}]` (kein `amount`-Subfeld) |
| Teilgutschrift einer Position | `lines=[{"type": "invoice_line_item", "invoice_line_item": "<id>", "amount": <cents>}]` |
| Charge zu Invoice auflösen | `InvoicePayment.list(invoice=...)` → `payment.payment_intent` → `PaymentIntent.latest_charge` |
| Gültige `reason`-Werte | `duplicate`, `fraudulent`, `order_change`, `product_unsatisfactory` |

Spike-Skript (Wegwerf-Code, nicht ins Repo übernommen): `/tmp/t030_paket_a_spike.py` auf dieser Maschine, ausgeführt im Container `normdex-dev-api` gegen `sk_test_...`. Alle erzeugten Testkunden wurden nach dem Lauf wieder gelöscht.

### Paket B · Datenmodell und Workflow

- [x] Persistentes Modell für Billing Adjustments entwerfen.
- [x] Alembic-Migration erstellen.
- [x] Statusmaschine und Idempotency-Konzept umsetzen.
- [x] Recovery- und Retry-Verhalten definieren.

### Paket C · Backend

- [x] Robuste Invoice-Auflösung implementieren.
- [x] Charge-Fallback ohne eindeutige Invoice entfernen oder hart absichern.
- [x] Preview um Invoice-, Credit-Note-, Steuer- und Bestandsdaten erweitern.
- [x] Credit Note plus Refund als gemeinsamen Service implementieren.
- [x] Bestehende Refunds verknüpfen können.
- [x] Sofortkündigung an den neuen Service anbinden.
- [x] Separaten Refund-Workflow an den neuen Service anbinden. *(Jetzt inkl. Einzel-Payment-Refund – siehe Ergebnisse Paket C unten.)*
- [x] Audit und Systemfehler erweitern.
- [x] Hintergrundverarbeitung und Wiederaufnahme umsetzen.
- [x] Reconciliation-Job ergänzen.

#### Ergebnisse Paket B/C (2026-06-21)

Umgesetzt in `normdex-webapp-dev` (Branch `dev-server`), API-Repo:

- **Modell** `BillingAdjustment` (`apps/api/app/models.py`) + Migration `c1a2b3d4e5f6_add_billing_adjustments_table` (Revision-Kette: `ffd3bbde6b6a` → `c1a2b3d4e5f6`). Statusmaschine exakt wie oben spezifiziert (`previewed, pending, subscription_adjusted, credit_note_created, refund_created, completed, partially_failed, failed, manual_review_required`; `document_sent` bleibt als Statuswert reserviert für Paket E, wird in Paket C nicht gesetzt). Migration gegen `dev.db` getestet (`alembic upgrade head` + `alembic downgrade -1` + erneut `upgrade head`, anschließend `dev.db` auf neuem Head committet).
- **Service** `apps/api/app/services/billing_adjustment_service.py`: `resolve_invoice_context()` implementiert exakt den in Paket A verifizierten Pfad `Invoice → InvoicePayment.list → PaymentIntent → latest_charge`; bei mehreren infrage kommenden Invoices wird `AmbiguousInvoiceContextError` geworfen, **kein** Fallback auf den neuesten Customer-Charge mehr (der bisherige Fallback in `_find_refundable_charge_for_subscriptions` ist ersatzlos entfernt – er war die Ursache der Lücke aus dem Sandbox-Fall vom 21.6.). `execute_adjustment()` ist die datengetriebene, idempotente Schritt-Engine (Subscription kündigen → Credit Note + Refund erzeugen/verknüpfen → Lizenzen beenden → completed), mit Backoff (1/5/30/120 Min.) und Eskalation nach 5 Fehlversuchen auf `manual_review_required`. Stripe-Idempotency-Keys sind deterministisch aus der Adjustment-ID abgeleitet.
- **Anbindung bestehender Endpunkte** (`apps/api/app/routers/admin.py`): `admin_licenses_cancel_now` (Sofortkündigung + aliquote Gutschrift) läuft jetzt vollständig über den neuen Service – das ist der im Sandbox-Fall beobachtete Vorgang, der künftig immer eine Credit Note erzeugt. Neue Lese-Endpunkte `GET .../billing/adjustments` und `.../billing/adjustments/{id}` (Backend-Grundlage für die spätere Paket-D-UI).
- **Einzel-Payment-Refund jetzt ebenfalls über Credit Note** (`admin_billing_payment_refund`, Admin gibt Charge-/PaymentIntent-ID frei ein): Gehört die Zahlung zu einer Rechnung, läuft sie nun über denselben `BillingAdjustment`-Service (Credit Note + Refund) wie die Sofortkündigung; nur nicht-rechnungsbezogene Zahlungen (einmalige PaymentIntents ohne Invoice) bleiben ein reiner `stripe.Refund.create()` (zulässiger, dokumentierter Ausnahmegrund laut Akzeptanzkriterien). Neuer Service-Resolver `resolve_invoice_context_from_payment(payment_id)` + Endpunkt-Helper `_resolve_payment_invoice_ctx_for_org`; sowohl `refund-preview` als auch `refund` geben jetzt `mode: "credit_note_refund" | "raw_refund"` zurück. Voraussetzung war ein eigener Sandbox-Spike (siehe nächster Abschnitt).
- **Hintergrundverarbeitung/Reconciliation** (`apps/api/app/services/scheduler.py`): neue Jobs `process_pending_billing_adjustments` (alle 2 Min., Recovery-Netz für Crash/Timeout – Primärpfad bleibt synchron im Request) und `reconcile_billing_adjustments` (alle 30 Min., prüft `completed`-Adjustments der letzten 24h gegen Stripe, eskaliert Abweichungen ohne Auto-Reparatur).
- **Webhooks** (`apps/api/app/routers/subscriptions.py`): Dispatcher um `credit_note.created/updated/voided` und `refund.updated/failed`, `charge.refunded` erweitert; synchronisiert nur bereits bekannte `BillingAdjustment`-Zeilen (PDF-Link, Status bei `voided`/`refund.failed`).
- **Tests** `apps/api/tests/test_billing_adjustment_service.py` (12 Tests, alle grün): Mehrdeutigkeit blockiert vor jeder Geldbewegung, Neufall erzeugt Credit Note + Auto-Refund, bestehender Refund wird verknüpft ohne Doppelauszahlung, abgeschlossene Adjustments sind idempotent (No-Op), Stripe-Fehler erzeugt sichtbaren `failed`-Status mit Backoff, Reconciliation eskaliert echte Inkonsistenzen und lässt konsistente Fälle unverändert; **neu für den Einzel-Payment-Refund:** Auflösung Payment→Invoice via PaymentIntent und via Charge, `None` bei nicht-rechnungsbezogener Zahlung (→ reiner Refund), Stopp bei Mehrdeutigkeit, sowie `execute_adjustment` ohne Lizenzbezug (`license_ids=None` → nur Credit Note, keine Subscription-/Lizenz-Schritte). Die Mocks bilden jetzt die echte verschachtelte `InvoicePayment.payment.payment_intent`-Struktur ab (siehe Bugfix unten). Vollständige Suite im Dev-Container grün: 336 passed, 1 vorbestehender, nicht zusammenhängender Umgebungs-Failure (`test_subscription_portal::...syncs_stripe_customer_from_org` erwartet `return_url=http://localhost:8080/...`, der Container hat `FRONTEND_URL=https://dev.normdex.at` – reiner Env-Mismatch, unabhängig von dieser Änderung).

#### Ergebnisse Paket C – Einzel-Payment-Refund-Spike (2026-06-21)

Eigener Sandbox-Spike (`sk_test_...`, API-Version `2025-11-17.clover`, Dev-Container) für die offene Frage „Payment-ID → Invoice", end-to-end inkl. Credit Note, Testkunden danach gelöscht.

**Kernbefund – einzig robuster Rückweg Payment → Invoice:**

| Versuch | Ergebnis |
|---|---|
| `PaymentIntent.retrieve(pi).invoice` | **existiert nicht** (Attribut fehlt) |
| `Charge.retrieve(ch).invoice` | **existiert nicht** |
| `Invoice.search(query="payment_intent:'…'")` | **nicht unterstützt** (`unsupported search field`) |
| `InvoicePayment.list(payment={"type":"payment_intent","payment_intent": pi})` | ✅ **liefert die Invoice** |

Daraus: `resolve_invoice_context_from_payment` löst aus `ch_` zuerst über `Charge.payment_intent` den PaymentIntent auf und nutzt dann den `payment=`-Filter. 0 Invoices → `None` (reiner Refund), >1 → `AmbiguousInvoiceContextError` (Stopp vor Geldbewegung), genau 1 → `Invoice.retrieve` + Standard-Auflösung. Hinweis: der `payment=`-Filter ist bei brandneuen Zahlungen kurz eventual-consistent (Sekundenbereich); für real erstattete, gealterte Zahlungen irrelevant.

**Dabei aufgedeckter latenter Bug (auch im lizenzbezogenen Pfad):** `_resolve_invoice_payment_context` las den PaymentIntent flach als `InvoicePayment.payment_intent`. In `2025-11-17.clover` steckt er **verschachtelt** unter `InvoicePayment.payment.payment_intent`. Dadurch lieferte die Auflösung gegen die echte API `None` – die gemockten Paket-B/C-Tests hatten das nicht erkannt, weil sie die flache Struktur mockten. Behoben (nested-Zugriff mit Flat-Fallback); Mocks auf die echte Struktur umgestellt. Der Einzel-Payment-E2E gegen die echte Sandbox läuft jetzt grün (Resolver via PI **und** via Charge, Preview, Credit Note + PDF, `invoice.post_payment_credit_notes_amount` korrekt aktualisiert).

### Paket D · Frontend / Verwaltungsportal

- [ ] Begriff „Refund" fachlich zu „Gutschrift und Erstattung" präzisieren.
- [ ] Vorschau um Rechnung, Positionen, Steuer und Dokumentversand ergänzen.
- [ ] Fortschritts- und Teilstatus anzeigen.
- [ ] Credit-Note-PDF verlinken.
- [ ] Sichere Wiederholungsaktion für fehlgeschlagene Teilschritte anbieten.
- [ ] Warnungen bei uneindeutiger Zuordnung und manueller Prüfung anzeigen.

### Paket E · E-Mail

- [ ] Versandstrategie Stripe vs. Normdex festlegen.
- [ ] Gebrandete Gutschrift-E-Mail erstellen, falls Normdex führend versendet.
- [ ] PDF/Link und Pflichtinformationen integrieren.
- [ ] Idempotenten Versand und Outbox-Status umsetzen.

### Paket F · Bestand und Rollout

- [ ] Read-only Inventarskript für bestehende Refunds und Credit Notes erstellen.
- [ ] Dev-Bestand analysieren.
- [ ] Dev-Reparatur mit dem 49-EUR-Testfall durchführen.
- [ ] Produktivbestand vor Änderungen analysieren.
- [ ] Reparaturplan je Produktivfall freigeben.
- [ ] Erst danach automatisierte bzw. kontrollierte Reparatur durchführen.
- [ ] Staging-End-to-end-Test.
- [ ] Steuerliche Freigabe dokumentieren.
- [ ] Produktivdeployment mit Monitoring.

## Tests

### Unit- und Integrationstests

- [ ] Teilrefund erzeugt genau eine passende Credit Note.
- [ ] Vollrefund erzeugt genau eine passende Credit Note.
- [ ] Credit Note referenziert die richtige Invoice.
- [ ] Credit Note übernimmt die richtige Rechnungsposition.
- [ ] Steuerbeträge stimmen mit der Originalrechnung überein.
- [ ] Bereits vorhandener Refund wird verknüpft und nicht erneut ausgezahlt.
- [ ] Wiederholter identischer Request erzeugt keine Duplikate.
- [ ] Timeout nach Credit-Note-Erstellung wird korrekt wiederaufgenommen.
- [ ] Timeout nach Refund-Erstellung wird korrekt wiederaufgenommen.
- [ ] E-Mail wird genau einmal versendet.
- [ ] Fehlgeschlagener Refund führt zu sichtbarem Teilstatus.
- [ ] Fehlgeschlagener E-Mail-Versand kann unabhängig wiederholt werden.
- [ ] Uneindeutige Invoice-Zuordnung blockiert vor jeder Geldbewegung.
- [ ] Betrag über dem noch verfügbaren Gutschriftbetrag wird abgelehnt.
- [ ] Kombination aus früheren Refunds und Credit Notes wird berücksichtigt.

### End-to-end in Stripe Sandbox

- [ ] Monatliche Lizenz teilweise aliquot gutschreiben.
- [ ] Monatliche Lizenz vollständig gutschreiben.
- [ ] Mehrere Lizenzen auf einer Rechnung teilweise gutschreiben.
- [ ] Bereits teilweise erstattete Zahlung weiter gutschreiben.
- [ ] Bestehenden Refund nachträglich mit Credit Note verknüpfen.
- [ ] PDF herunterladen und Inhalte prüfen.
- [ ] Kundenmail erhalten und Inhalte prüfen.
- [ ] Verwaltungsportal zeigt finalen Status und alle Stripe-Referenzen.
- [ ] Stripe-Dashboard zeigt Invoice, Credit Note und Refund nachvollziehbar verknüpft.

## Akzeptanzkriterien

- [ ] Kein neuer produktiver Refund einer rechnungsbezogenen Zahlung entsteht ohne zugehörige Credit Note oder explizit dokumentierten Ausnahmegrund.
- [ ] Im Normalfall genügt eine Admin-Bestätigung; alle Folgeschritte laufen automatisch.
- [ ] Credit Note und Refund sind eindeutig derselben Invoice zugeordnet.
- [ ] Der Kunde erhält automatisch einen Gutschriftbeleg.
- [ ] PDF, Nummer, Betrag, Steuerbehandlung und Referenzrechnung sind nachvollziehbar.
- [ ] Normdex speichert den vollständigen Workflow- und Versandstatus.
- [ ] Teilfehler können ohne Doppelzahlung oder Doppelbeleg fortgesetzt werden.
- [ ] Reconciliation erkennt Refunds ohne Credit Note.
- [ ] Bestehende relevante Dev- und Produktivfälle sind inventarisiert und bereinigt.
- [ ] Sandbox-End-to-end-Tests sind vollständig grün.
- [ ] Steuerberatung/Buchhaltung hat den Zielprozess vor Produktivfreigabe bestätigt.
- [ ] Vault-Dokumentation zu API, Funktionen, E-Mail-System, Datenmodell und Integrationen ist aktualisiert.

## Nicht-Ziele

- Keine eigene vollständige Buchhaltungssoftware in Normdex entwickeln.
- Keine automatische steuerliche Entscheidung bei fachlich uneindeutigen Sonderfällen.
- Keine stillschweigende Bestandsreparatur ohne Preview und Protokoll.
- Keine erneute Auszahlung beim nachträglichen Erstellen einer Credit Note für einen bereits vorhandenen Refund.

## Verwandte Dokumente und Todos

- [[T020-16-Lizenz-und-Billing-Support-Aktionen]]
- [[T020-16-Abnahme-Checkliste-Billing-Support]]
- [[T020-12-Konzept Verwaltungsportal]]
- [[T021-rechnungslegung-individuelle-rechnungen]]
- [[02_App/API-Endpunkte]]
- [[02_App/Funktionen im Detail]]
- [[02_App/E-Mail-System]]
- [[02_App/Datenmodell]]
- [[06_Entwicklung/Integrationen & externe Dienste]]

## Notizen / Fortschritt

- 2026-06-21: Lücke im Dev-Verwaltungsportal anhand eines echten Sandbox-Vorgangs bestätigt. Der bisherige Workflow erstellt nur einen Stripe Refund und keine Credit Note.
- 2026-06-21: Sandbox-Zahlung über 49,00 EUR analysiert. Nach früherem Refund über 33,00 EUR und neuem Refund über 16,00 EUR ist die Zahlung vollständig erstattet; die zugehörige Rechnung enthält weiterhin keine Credit Note.
- 2026-06-21: Produktentscheidung festgehalten: Billing-Vorgänge sollen generell nach einmaliger bewusster Bestätigung möglichst vollständig und resilient im Hintergrund automatisiert werden.
- 2026-06-21: Paket A (Stripe-Spike) vollständig abgeschlossen. Alle vier Sandbox-Szenarien (Neufall, bestehenden Refund verknüpfen, Teilgutschrift einer Invoice Line, Steuer/Rundung) erfolgreich verifiziert, siehe Ergebnis-Abschnitt oben. Kritischer Zusatzbefund: `Invoice.charge`/`Invoice.payment_intent`/`Charge.invoice` existieren in der aktuellen API-Version nicht mehr; korrekter Weg ist `InvoicePayment.list(invoice=...)` → `PaymentIntent` → `latest_charge`. Kein Produktionscode geändert – nächster Schritt ist Paket B/C (Datenmodell + Backend-Service) auf Basis dieser verifizierten API-Parameter.
- 2026-06-21: Paket B und Paket C vollständig umgesetzt (Modell, Migration, Service, Anbindung der Sofortkündigung+Gutschrift, Hintergrundjobs, Reconciliation, Webhook-Sync, Unit-Tests), siehe Ergebnis-Abschnitt oben. Bewusst nicht mitgemacht: Umstellung des separaten Einzel-Payment-Refund-Endpunkts (bräuchte einen eigenen Spike zu `PaymentIntent.invoice`), sowie Paket D (Frontend), E (E-Mail) und F (Bestandsbereinigung/Rollout) – die bleiben offen. `dev.db`-Fixture wurde auf den neuen Migrations-Head aktualisiert. Vor Produktivfreigabe weiterhin nötig: Paket D–F, vollständige Sandbox-End-to-End-Tests, steuerliche Freigabe.
- 2026-06-21 (PAUSE, Nutzungslimit): Paket E (E-Mail/Belegzustellung) vollständig und Paket D im Backend umgesetzt; Pause vor dem Paket-D-Frontend. Commit `e31af5f` (Branch `dev-server`), Backend Python-Syntax geprüft (py_compile OK), Tests/Frontend-UI noch offen.
  - **Paket E (fertig):** Modellfelder `document_sent_at`/`document_recipient`/`document_send_error` + Migration `d2b3c4e5f6a7`. Gebrandete Gutschrift-Mail `tpl_credit_note(_html)` (verlinkt das Stripe-PDF). **Versandstrategie:** Normdex versendet führend (eigene Outbox/Audit), Stripes Auto-Versand wird nicht als führend genutzt. Neuer Service `billing_adjustment_email.send_billing_adjustment_document()` ist idempotent über `document_sent_at`, wirft nicht, hält Fehler in `document_send_error` + Outbox/Audit fest. In `execute_adjustment` ist der Belegversand ein best-effort-Schritt: der Geldfluss (Credit Note + Refund) bleibt gating; ein Mailfehler führt nur in den nicht-terminalen Status `document_sent` (Backoff, Recovery-Job holt nach), nach `BILLING_ADJUSTMENT_MAX_ATTEMPTS` graceful give-up (Geld ist fertig, Beleg bleibt zur manuellen Zustellung sichtbar). Damit wird `document_sent` jetzt tatsächlich genutzt.
  - **Paket D Backend (fertig):** `retry_adjustment()` + Endpoint `POST .../billing/adjustments/{id}/retry` – sichere, idempotente Wiederaufnahme inkl. reinem Beleg-Nachversand und Rücksetzen terminaler `manual_review_required`-Zeilen in einen fortsetzbaren Zustand. `cancel-now` wertet `document_sent` als Refund-Erfolg (kein Geldfehler) und gibt `status`/`credit_note_number` zurück. `serialize_adjustment` um `document_*` + `invoice_number` erweitert. Frontend-`api.ts` um die drei Adjustment-Methoden ergänzt.
  - **Offen (nächste Sitzung):** Paket-D-Frontend in `OrganizationCase.tsx` (Begriff „Gutschrift und Erstattung", erweiterte Vorschau mit Rechnung/Steuer/Beleg, Status-/Teilstatus- und PDF-Anzeige, Wiederholungs-Button, Warnungen bei Mehrdeutigkeit/`manual_review_required`); Unit-Tests für Paket E + retry; vollständige Test-Suite; Migration gegen Dev-DB; danach Paket-D/E-Häkchen setzen und Container/Dev neu laden.
- 2026-06-21: Einzel-Payment-Refund-Spike durchgeführt und Endpunkt umgestellt – Paket C damit vollständig abgeschlossen. Verifizierter Rückweg Payment → Invoice: `InvoicePayment.list(payment={type:payment_intent, payment_intent:…})` (PaymentIntent.invoice/Charge.invoice existieren nicht, Invoice.search nach payment_intent nicht unterstützt). `admin_billing_payment_refund` läuft für rechnungsbezogene Zahlungen jetzt über den BillingAdjustment-Service (Credit Note), nur nicht-rechnungsbezogene bleiben reiner Refund. Dabei latenten Bug im bereits committeten Paket-B/C-Code gefunden und behoben: `_resolve_invoice_payment_context` las `InvoicePayment.payment_intent` flach statt verschachtelt (`.payment.payment_intent`) – betraf auch den cancel-now-Pfad, von gemockten Tests verdeckt. Echte Sandbox-E2E jetzt grün, 12 Unit-Tests grün, Suite 336 passed (1 vorbestehender Env-Failure). Offen bleiben Paket D (Frontend), E (E-Mail), F (Bestand/Rollout), Sandbox-End-to-End-Gesamtdurchlauf und steuerliche Freigabe.
