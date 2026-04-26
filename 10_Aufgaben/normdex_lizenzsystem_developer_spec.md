# Developer Spec – Normdex Lizenzsystem (Pool-Modell mit Sammelabrechnung)

## Zweck
Dieses Dokument beschreibt die Soll-Architektur für die Umstellung des Normdex-Lizenzsystems auf ein Pool-Modell mit Sammelabrechnung je Intervall, Einzellizenzsicht in der Lizenzverwaltung, individueller Mindestlaufzeit je Lizenz, gestaffelten Preisen, Admin-only-Kündigung und optionalem Rabattcode-/Promotionsystem.

---

## 1. Projektziel

Das bestehende Lizenzsystem soll so umgestellt werden, dass:

1. jede einzelne Lizenz in Normdex separat sichtbar und separat verwaltbar ist,
2. jede Lizenz eine individuelle Mindestlaufzeit besitzt,
3. neue Lizenzen sofort aktiviert und sofort anteilig verrechnet werden,
4. Kündigungen nur durch Admins/Owner erlaubt sind,
5. bei Kündigung der Zugriff bis Laufzeitende aktiv bleibt,
6. bei Reduktion zuerst immer eine Add-on-Lizenz gekündigt wird,
7. Stripe möglichst eine Sammelrechnung und Sammelabbuchung je Intervall erzeugt,
8. monatliche und jährliche Lizenzen rabattlogisch getrennte Pools sind,
9. Staffelpreise pro Pool gelten,
10. zusätzliche Promotion-/Rabattcodes möglich sind,
11. beim Kauf mehrere Lizenzen gleichzeitig ausgewählt werden können,
12. beim Kauf Preisstaffelungen live sichtbar sind.

---

## 2. Fachliches Zielbild

### 2.1 Pools
Pro Organisation existieren maximal zwei Abrechnungspools:

- `monthly`
- `yearly`

Diese Pools sind fachlich und abrechnungstechnisch getrennt.

### 2.2 Staffelpreise
#### Monatlicher Pool
- 1. Lizenz: **49,00 € / Monat**
- jede weitere Lizenz: **29,00 € / Monat**

#### Jährlicher Pool
- 1. Lizenz: **490,00 € / Jahr**
- jede weitere Lizenz: **290,00 € / Jahr**

### 2.3 Rabattlogik
Rabatte durch Staffelpreise gelten **nur innerhalb desselben Pools**:
- Monatslizenzen beeinflussen den Jahrespool nicht
- Jahreslizenzen beeinflussen den Monatspool nicht

### 2.4 Einzellizenzprinzip
In Normdex ist **jede Lizenz ein eigener Datensatz** mit:
- eigenem Startdatum
- eigener Preiszuordnung
- eigenem Mindestlaufzeitende
- eigenem Status
- eigener Kündigungsinformation

Jede Lizenz erlaubt genau **einen gleichzeitigen aktiven Nutzer** (Concurrent-Use-Locking via Heartbeat-Session). Die statische Benutzerzuweisung (`assigned_user_id`) ist im Datenmodell vorhanden, wird aber in der UI nicht exponiert.

### 2.5 Sammelabrechnung
In Stripe soll **nicht pro Lizenz eine eigene Subscription** existieren, sondern:
- eine gemeinsame monatliche Pool-Subscription pro Organisation
- eine gemeinsame jährliche Pool-Subscription pro Organisation

Dadurch entstehen im Regelfall:
- eine monatliche Sammelrechnung
- eine jährliche Sammelrechnung

Die eigentliche Einzellizenzlogik wird in Normdex geführt.

---

## 3. Abgrenzung / Nicht-Ziele

Nicht Teil dieses Projekts:

- Änderung der bestehenden Concurrent-Use-/Heartbeat-Logik als Grundprinzip
- vollständige Neugestaltung des Team-/Organisationsmodells
- Einführung neuer Produktlinien außerhalb von `economics_v1`
- Vereinheitlichung von monatlicher und jährlicher Rechnung in **eine** Gesamtrechnung
- rechtliche Ausformulierung finaler AGB-Texte durch Juristen

---

## 4. Muss-Anforderungen

### 4.1 Lizenzverwaltung

#### 4.1.1 Einzellizenzdarstellung
Die Lizenzverwaltung muss jede einzelne Lizenz separat anzeigen.

Für jede Lizenz müssen mindestens sichtbar sein:
- Lizenztyp: **Basislizenz** oder **Add-on-Lizenz**
- Pool: **monatlich** oder **jährlich**
- Preis
- Startdatum
- Laufzeitende / nächstes Vertragsende
- Status
- aktive Sitzung (Nutzername + Uhrzeit, falls jemand die Lizenz gerade verwendet)
- Kündigungsstatus inkl. Enddatum

#### 4.1.2 Statusanzeige
Mindestens folgende Status müssen unterstützt und in der UI verständlich dargestellt werden:
- Aktiv
- Gekündigt zum Laufzeitende
- Beendet
- Zahlung fehlgeschlagen
- Ausstehend / Pending

Beispielanzeige:
- `Aktiv`
- `Gekündigt – endet mit Ablauf des 14.09.2026`
- `Beendet am 14.09.2026`

### 4.2 Rechte / Berechtigungen

#### 4.2.1 Lizenzkündigung
Nur Benutzer mit Admin-/Owner-Rechten der Organisation dürfen Lizenzen kündigen.

#### 4.2.2 Lizenzkauf
Nur Benutzer mit Admin-/Owner-Rechten der Organisation dürfen neue Lizenzen kaufen.

#### 4.2.3 Members
Normale Members dürfen:
- Lizenzen sehen
- ggf. ihre Zuweisung sehen

Normale Members dürfen nicht:
- kündigen
- neue Lizenzen kaufen
- Rabattcodes anwenden
- Preise ändern

### 4.3 Kündigungslogik

#### 4.3.1 Reihenfolge
Wenn eine Lizenz reduziert / gekündigt werden soll, muss **zuerst immer eine Add-on-Lizenz** gekündigt werden.

Die Basislizenz darf erst gekündigt werden, wenn:
- keine aktive Add-on-Lizenz mehr vorhanden ist
**oder**
- der Pool aufgelöst werden soll

#### 4.3.2 Kündigungswirkung
Bei Kündigung gilt:
- Zugriff bleibt bis Laufzeitende aktiv
- Lizenzstatus wechselt auf `scheduled_end`
- Enddatum wird in Normdex gespeichert
- UI zeigt die Kündigung transparent an

#### 4.3.3 Kündigungsfrist / Laufzeitende
Die gekündigte Lizenz endet frühestens zu ihrem individuellen Mindestlaufzeitende bzw. zum Ende der aktuellen vertraglichen Periode.

#### 4.3.4 Anzeige in der UI
Nach Kündigung muss in der Lizenzverwaltung klar sichtbar sein:
- dass die Lizenz gekündigt wurde
- wann sie endet

Pflichttext sinngemäß:
> Diese Lizenz wurde gekündigt und endet mit Ablauf des DD.MM.YYYY.

### 4.4 Neue Lizenzen

#### 4.4.1 Sofortige Aktivierung
Neu gekaufte Lizenzen müssen nach erfolgreichem Kauf sofort aktiv sein.

#### 4.4.2 Sofortige anteilige Verrechnung
Neu gekaufte Lizenzen sollen sofort anteilig im jeweiligen Pool verrechnet werden.

#### 4.4.3 Individuelle Mindestlaufzeit
Trotz sofortiger anteiliger Verrechnung muss Normdex pro Lizenz individuell speichern:
- Startdatum
- Mindestlaufzeitende
- aktuelles Vertragsende

### 4.5 Mehrfachkauf

#### 4.5.1 Kauf mehrerer Lizenzen auf einmal
Ein Benutzer soll nicht nur eine Lizenz nach der anderen kaufen können, sondern mehrere gleichzeitig.

Beispiele:
- 4 monatliche Lizenzen
- 2 jährliche Lizenzen
- 4 monatliche + 2 jährliche Lizenzen in einem Bestellvorgang in der UI

#### 4.5.2 Live-Preisberechnung
Im Kaufdialog muss live erkennbar sein:
- wie viele Basislizenzen anfallen
- wie viele Add-on-Lizenzen anfallen
- wie sich der Gesamtpreis zusammensetzt

Beispiel:
- 1 × 49 €
- 3 × 29 €
- 1 × 490 €
- 1 × 290 €

### 4.6 Rabattcodes / Promotions

#### 4.6.1 Zusätzliche Rabattcodes
Zusätzlich zu den gestaffelten Preisen müssen Rabattcodes / Promotions unterstützt werden.

#### 4.6.2 Zielgruppen
Es sollen mindestens folgende Anwendungsfälle möglich sein:
- öffentlicher Launch-Rabatt
- private / limitierte Rabattaktionen
- Testuser / Pilotkunden
- manuelle Sonderrabatte

#### 4.6.3 Kombinationsregel
Die Staffelpreise bleiben das Grundpreismodell.
Rabattcodes wirken zusätzlich auf den berechneten Warenkorb / Kaufpreis.

#### 4.6.4 Testlizenzen
Für Testuser soll zusätzlich die Möglichkeit bestehen, kostenlose oder rabattierte Lizenzen auch direkt intern in Normdex zu vergeben, ohne ausschließlich auf Stripe-Promotion-Codes angewiesen zu sein.

---

## 5. Soll-Datenmodell

### 5.1 Tabelle `licenses`
Eine Zeile = genau eine Lizenz.

Pflichtfelder:
- `id`
- `organization_id`
- `product_key`
- `billing_pool` (`monthly`, `yearly`)
- `license_kind` (`base`, `addon`)
- `status` (`pending`, `active`, `scheduled_end`, `ended`, `payment_failed`)
- `started_at`
- `current_term_start`
- `current_term_end`
- `committed_until`
- `cancel_requested_at` nullable
- `scheduled_end_at` nullable
- `price_amount_gross`
- `currency`
- `stripe_subscription_id`
- `stripe_subscription_item_id`
- `stripe_price_id`
- `assigned_user_id` nullable
- `created_by_user_id`
- `meta` JSON nullable

### 5.2 Tabelle `license_orders`
Abbildung eines internen Bestellvorgangs.

Felder:
- `id`
- `organization_id`
- `created_by_user_id`
- `promotion_code` nullable
- `subtotal_gross`
- `discount_total_gross`
- `total_gross`
- `currency`
- `status`
- `created_at`
- `stripe_checkout_session_id` nullable

### 5.3 Tabelle `license_order_items`
Abbildung der Preiszeilen einer Bestellung.

Felder:
- `id`
- `license_order_id`
- `billing_pool`
- `license_kind`
- `quantity`
- `unit_price_gross`
- `line_total_gross`

### 5.4 Tabelle `license_events`
Audit- und Historieneinträge.

Beispiele:
- `license.created`
- `license.activated`
- `license.cancel_requested`
- `license.ended`
- `license.user_assigned`
- `license.user_unassigned`
- `license.rebased_to_base`

### 5.5 Tabelle `license_usage`
Die bestehende Nutzungstabelle kann weiterverwendet werden für Heartbeat / aktive Nutzung.

---

## 6. Stripe-Modell

### 6.1 Stripe-Prices
Es sind vier Stripe-Prices anzulegen:
- `economics_basic_monthly_base`
- `economics_basic_monthly_addon`
- `economics_basic_yearly_base`
- `economics_basic_yearly_addon`

### 6.2 Pool-Subscriptions
Pro Organisation maximal:
- eine monatliche Pool-Subscription
- eine jährliche Pool-Subscription

### 6.3 Einzellizenzen in Stripe
Einzellizenzen werden im jeweiligen Pool über Subscription Items abgebildet.

### 6.4 Sammelrechnung
Die monatliche Pool-Subscription erzeugt eine monatliche Sammelrechnung.
Die jährliche Pool-Subscription erzeugt eine jährliche Sammelrechnung.

---

## 7. Geschäftsregeln

### 7.1 Preisfindung beim Kauf
Bei jeder neu gekauften Lizenz muss Normdex vor Kaufabschluss prüfen:

#### Für den jeweiligen Pool:
- Wenn im Pool noch keine aktive oder noch laufende Lizenz existiert → `base`
- Wenn mindestens eine aktive oder noch laufende Lizenz existiert → `addon`

Wichtig:
Auch `scheduled_end`-Lizenzen zählen noch mit, solange sie noch nicht beendet sind.

### 7.2 Mehrfachkauf
Wenn ein Käufer mehrere Lizenzen gleichzeitig auswählt, muss die Preislogik für die Menge in Reihenfolge angewendet werden.

Beispiel Monats-Pool, Kauf von 4 Lizenzen bei leerem Pool:
- Lizenz 1 = base
- Lizenz 2–4 = addon

Beispiel Jahres-Pool, Kauf von 2 Lizenzen bei leerem Pool:
- Lizenz 1 = base
- Lizenz 2 = addon

Beispiel Monats-Pool, es existieren bereits 2 Monatslizenzen und es werden 3 weitere gekauft:
- alle 3 neuen = addon

### 7.3 Rebasierung
Wenn die aktuelle Basislizenz endet und im Pool noch Add-ons aktiv bleiben, muss eine verbleibende Add-on-Lizenz zur neuen Basislizenz werden.

Empfohlene Regel:
- die älteste verbleibende aktive Add-on-Lizenz wird zur neuen Basislizenz

Dabei muss:
- `license_kind` angepasst werden
- Preis für zukünftige Perioden angepasst werden
- ein `license_event` erzeugt werden

---

## 8. API-Anforderungen

### 8.1 Lesen
- `GET /licenses/pools`
- `GET /licenses/pools/{pool}/items`
- `GET /licenses/{license_id}`
- `GET /licenses/{license_id}/history`

### 8.2 Kauf
- `POST /licenses/checkout/preview`
- `POST /licenses/checkout/create`
- `POST /licenses/checkout/confirm`

### 8.3 Verwaltung
- `POST /licenses/{license_id}/cancel`
- `POST /licenses/{license_id}/undo` — Direktaktivierung rückgängig machen (10-Minuten-Fenster)
- `POST /licenses/{license_id}/force-release` — Admin: aktive Sessions einer Lizenz sofort beenden
- `POST /licenses/{license_id}/assign-user` *(Backend vorhanden, UI nicht exponiert)*
- `POST /licenses/{license_id}/unassign-user` *(Backend vorhanden, UI nicht exponiert)*

### 8.4 Promotions
- `POST /licenses/promotions/validate`
- `POST /licenses/promotions/apply`
- optional intern:
  - `POST /admin/licenses/grant-complimentary`

---

## 9. UI-Anforderungen

### 9.1 Lizenzverwaltung
Die Lizenzverwaltung soll gegliedert sein in:
- monatliche Lizenzen
- jährliche Lizenzen

Jede Lizenz muss als eigene Zeile/Karte sichtbar sein.

#### Je Eintrag:
- Typ: Basis / Add-on
- Preis
- Status
- Aktive Sitzung (Avatar + Name + „Aktiv seit HH:MM" wenn belegt, sonst „Keine aktive Sitzung")
- Startdatum
- Laufzeitende
- Kündigungsstatus
- Aktionen (Admin/Owner: Kündigen, Sitzungen bereinigen, Kauf rückgängig)

#### 9.1.1 Rückgängig-Machen-Dialog
Wenn ein Admin/Owner den „Rückgängig"-Button im 10-Minuten-Fenster klickt, muss ein Bestätigungsdialog erscheinen, der darüber informiert, dass die Lizenz mit sofortiger Wirkung vollständig entfernt wird. Die Aktion darf erst nach expliziter Bestätigung ausgeführt werden.

#### 9.1.2 Lizenzauswahl in `useLicenseLock`
Die Hook-Logik muss alle aktiven Lizenzen des passenden Produkts der Reihe nach probieren (nicht nur die erste). Bei 409 (Lizenz bereits belegt) wird die nächste Lizenz versucht. Erst wenn alle Lizenzen belegt sind, wird dem Nutzer ein Fehler angezeigt.

### 9.2 Kaufdialog
Der Kaufdialog muss enthalten:
- Anzahl monatliche Lizenzen
- Anzahl jährliche Lizenzen
- Eingabefeld Rabattcode
- Live-Zusammenfassung
- Gesamtpreis
- ggf. Hinweis auf getrennte monatliche/jährliche Abrechnung

### 9.3 Kündigungsdialog
Wenn der Admin kündigt:
- UI zeigt, welche Lizenz gekündigt wird
- Standardmäßig wird zuerst eine Add-on-Lizenz gewählt
- UI weist darauf hin, dass Zugriff bis Laufzeitende aktiv bleibt
- UI nennt konkretes Enddatum

---

## 10. Zustände / State Machine

### 10.1 Lizenzstatus
#### `pending`
Lizenz ist angelegt, aber noch nicht final aktiviert.

#### `active`
Lizenz ist aktiv und nutzbar.

#### `scheduled_end`
Lizenz ist gekündigt, bleibt aber bis `scheduled_end_at` aktiv.

#### `ended`
Lizenz ist beendet und nicht mehr nutzbar.

#### `payment_failed`
Abrechnung fehlgeschlagen.

---

## 11. Webhooks / Hintergrundlogik

### Relevante Stripe-Webhooks
- `checkout.session.completed`
- `customer.subscription.updated`
- `invoice.paid`
- `invoice.payment_failed`

### Erwartetes Verhalten
#### `checkout.session.completed`
- interne Bestellung bestätigen
- Lizenzen auf `active` setzen
- Stripe-IDs speichern

#### `customer.subscription.updated`
- Periodenende aktualisieren
- ggf. Status synchronisieren

#### `invoice.paid`
- Zahlungserfolg dokumentieren

#### `invoice.payment_failed`
- betroffene Lizenzen auf `payment_failed`
- ggf. Grace-Period-Logik

---

## 12. Nichtfunktionale Anforderungen

- klare Auditierbarkeit
- idempotente Webhook-Verarbeitung
- serverseitige Rechteprüfung
- konsistente Preislogik auch bei Parallelkäufen
- saubere Transaktionen beim Kauf
- Historisierung wichtiger Lizenzereignisse
- verständliche deutsche UI-Texte

---

## 13. TODO-Liste für Implementierung

### Phase 1 – Architektur & Datenmodell
1. Bestehendes Lizenzmodell analysieren
2. Neues Einzellizenzschema definieren
3. Migration für `licenses` erstellen
4. Tabellen `license_orders`, `license_order_items`, `license_events` anlegen
5. Rebasierungslogik fachlich finalisieren

### Phase 2 – Stripe
6. Vier Prices in Stripe anlegen
7. Pool-Subscription-Konzept implementieren
8. Mapping `license <-> subscription item` implementieren
9. Rabattcode-/Promotion-Code-Handling definieren
10. Complimentary/Testlizenzen-Konzept ergänzen

### Phase 3 – Backend
11. Preview-Endpunkt für Kauf bauen
12. Kauf-/Checkout-Endpunkte bauen
13. Admin-only-Checks für Kauf und Kündigung implementieren
14. Kündigungslogik `addon zuerst` implementieren
15. Sofortige Aktivierung + individuelle Mindestlaufzeit implementieren
16. Rebasierung nach Beendigung implementieren
17. Lizenzhistorie / Events implementieren

### Phase 4 – Webhooks & Synchronisierung
18. Webhook-Handler erweitern
19. Idempotenz absichern
20. Statussynchronisierung Stripe <-> Normdex testen
21. Fehlerfälle `invoice.payment_failed` abfangen

### Phase 5 – Frontend
22. Neue Lizenzverwaltung bauen
23. Monats-/Jahresblöcke getrennt anzeigen
24. Lizenzstatus-Badges und Hinweise bauen
25. Kaufdialog mit Live-Kalkulation bauen
26. Rabattcode-Feld integrieren
27. Kündigungsdialog bauen
28. Rechteabhängige Buttons für Admin/Member umsetzen

### Phase 6 – Tests
29. Unit-Tests Preislogik
30. Unit-Tests Kündigungsreihenfolge
31. Integrationstests Kauf mehrerer Lizenzen
32. Integrationstests monatlich + jährlich gemischt
33. Tests für Rebasierung
34. Tests für UI-Hinweise gekündigter Lizenzen
35. Tests für Promotions und Complimentary Lizenzen

### Phase 7 – Rollout
36. Migration bestehender Kunden planen
37. Bestandsdaten in neues Modell überführen
38. interne Dokumentation aktualisieren
39. Support-/FAQ-Texte ergänzen
40. Launch-Kommunikation vorbereiten

---

## 14. KI-gerechte Arbeitsanweisung

### Arbeitsauftrag an die KI
Implementiere ein neues Lizenzsystem für Normdex mit folgenden Regeln:

1. Es gibt zwei getrennte Billing-Pools pro Organisation: `monthly` und `yearly`.
2. In Normdex ist jede Lizenz ein eigener Datensatz.
3. In Stripe werden Lizenzen je Pool zu einer Sammelabrechnung gebündelt.
4. Staffelpreise:
   - monthly base = 49
   - monthly addon = 29
   - yearly base = 490
   - yearly addon = 290
5. Neue Lizenzen werden sofort aktiviert.
6. Neue Lizenzen werden sofort anteilig verrechnet.
7. Normdex speichert trotzdem pro Lizenz individuell:
   - `started_at`
   - `current_term_start`
   - `current_term_end`
   - `committed_until`
8. Nur Admin/Owner darf Lizenzen kaufen oder kündigen.
9. Bei Kündigung bleibt Zugriff bis Laufzeitende aktiv.
10. Bei Kündigung muss zuerst immer eine Add-on-Lizenz gekündigt werden.
11. Gekündigte Lizenzen erhalten Status `scheduled_end`.
12. Die UI muss jede Lizenz einzeln anzeigen.
13. Die UI muss für gekündigte Lizenzen einen deutlichen Hinweis mit Enddatum anzeigen.
14. Beim Kauf muss der Nutzer mehrere monatliche und/oder jährliche Lizenzen gleichzeitig auswählen können.
15. Der Kaufdialog muss die Staffelpreise live und transparent anzeigen.
16. Rabattcodes zusätzlich zu Staffelpreisen müssen unterstützt werden.
17. Es muss zusätzlich die Möglichkeit geben, kostenlose oder rabattierte Testlizenzen intern zu vergeben.
18. Wenn die Basislizenz endet und Add-ons verbleiben, muss automatisch eine Add-on-Lizenz zur neuen Basislizenz werden.

---

## 15. Empfohlene Reihenfolge für eine KI

1. Datenmodell anpassen
2. Preislogik als reine Domänenlogik implementieren
3. Lizenzkauf-Preview implementieren
4. Lizenzkauf-Checkout implementieren
5. Kündigungslogik implementieren
6. Rebasierung implementieren
7. Webhook-Synchronisierung implementieren
8. Lizenzverwaltungs-UI implementieren
9. Rabatt-/Promotion-Logik ergänzen
10. Tests schreiben

