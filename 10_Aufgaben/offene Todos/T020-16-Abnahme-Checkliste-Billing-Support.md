# T020-16 · Abnahme-Checkliste Billing-Support

**Bezug:** [[T020-16-Lizenz-und-Billing-Support-Aktionen]]  
**Datum:** 2026-05-30  
**Ziel:** Manuelle Abnahme der Admin-Billing- und Stripe-Support-Funktionen vor Abschluss von T020-16.

## Vorbereitung

- [x] Lokale App starten und als Admin anmelden.
- [x] Einen Testkunden mit Stripe Customer ID öffnen.
- [x] Sicherstellen, dass Stripe-Testmodus verwendet wird, wenn echte Stripe-Aktionen getestet werden.
- [x] Einen Testkunden wählen, der mindestens eine Subscription und eine Rechnung in Stripe hat.
- [x] Optional ein offenes Support-Ticket beim Testkunden bereithalten, um Ticket-Verknüpfung zu prüfen.

## Billing-Diagnose

- [x] Organisationsakte öffnen und Tab `Billing & Stripe` auswählen.
- [x] `Diagnose laden` ausführen.
- [x] Stripe Customer wird angezeigt.
- [x] Stripe Subscriptions werden angezeigt. --> wird angezeigt, aber es steht nur "sub_1TRz4bF05ipkEAzmWvq9cvRa". Ich weiß daher nicht, ob das eine Haupt-oder Zusatzlizenz ist und ob es eine monatl. oder jährliche Lizenz ist.
- [x] Stripe Rechnungen werden angezeigt.
- [x] Lokale Lizenzen werden angezeigt. --> Was sind lokale Lizenzen?
- [x] Offene Rechnungen werden als Hinweis sichtbar.
- [x] Zahlungsprobleme oder Past-Due-Zustände werden als Hinweis sichtbar.
- [x] Abweichungen zwischen lokalen Lizenzdaten und Stripe werden als Diagnose-Warnung sichtbar.

## Stripe-Deep-Links

- [x] `Stripe Customer öffnen` führt zum richtigen Stripe Customer.
- [x] `Stripe` bei einer Subscription führt zur richtigen Stripe Subscription.
- [x] `Stripe` bei einer Rechnung führt zur richtigen Stripe Invoice.
- [x] `Zahlung` bei einer Rechnung führt zur passenden Stripe Payment-/Charge-Ansicht.
- [x] Links öffnen im Testmodus unter `dashboard.stripe.com/test`, wenn ein Test-Secret-Key aktiv ist.

## Kundendaten-Sync

- [x] Aktion `Billing-Adresse synchronisieren` öffnen.
- [x] Vorschau zeigt Vorher/Nachher für Name, E-Mail, Rechnungsadresse und USt-ID.
- [x] Ausführung ohne Grund wird blockiert.
- [x] Ausführung ohne Bestätigung wird blockiert.
- [x] Ausführung mit Grund und Bestätigung synchronisiert nach Stripe.
- [x] Optionales Support-Ticket kann ausgewählt werden.
- [x] Nach erfolgreicher Ausführung wird die Billing-Diagnose aktualisiert.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Zahlungsart / Billing Portal

- [x] Aktion `Zahlungsart aktualisieren` öffnen.
- [x] Ausführung ohne Grund wird blockiert.
- [x] Ausführung ohne Bestätigung wird blockiert.
- [x] Ausführung mit Grund und Bestätigung öffnet eine Stripe-hosted Billing-Portal-Session.
- [x] Rücksprung aus dem Billing Portal führt wieder in die Admin-Organisationsakte.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Kündigung und Reaktivierung

- [x] Bei einer aktiven Lizenz Aktion `Kündigung` öffnen.
- [x] Vorschau zeigt geplantes Ende und Stripe-Bezug.
- [x] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [x] Bestätigte Kündigung setzt lokale Lizenz auf auslaufend.
- [x] Stripe Subscription/Item wird korrekt zum Periodenende behandelt.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.
- [x] Bei einer auslaufenden Lizenz Aktion `Reaktivierung` öffnen.
- [x] Vorschau zeigt bisher geplantes Ende.
- [x] Bestätigte Reaktivierung hebt die Kündigung zurück.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Concurrent-User-Nutzung freigeben

- [ ] Bei einer Lizenz mit aktiver Nutzung Aktion `Freigeben` öffnen.
- [ ] Vorschau zeigt Anzahl aktiver Nutzungen.
- [ ] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [ ] Bestätigte Ausführung beendet aktive Nutzungen.
- [ ] Lizenz ist danach wieder nutzbar.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Stripe-Status-Resync

- [ ] Aktion `Sync` bei einer Stripe-verknüpften Lizenz öffnen.
- [ ] Vorschau zeigt erkannte Abweichungen zwischen Stripe und Normdex.
- [ ] Bei keinen Abweichungen wird das verständlich angezeigt.
- [ ] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [ ] Bestätigte Ausführung übernimmt Status-/Periodenänderungen.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Rabatte / Coupons

- [ ] Bei einer Subscription Aktion `Rabatt` öffnen.
- [ ] Ungültiger oder abgelaufener Promotion Code wird abgelehnt.
- [ ] Gültiger Promotion Code zeigt Coupon-Wert, Dauer und bestehenden Rabatt.
- [ ] Bestehender Rabatt wird als zu ersetzender Discount angezeigt.
- [ ] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [ ] Bestätigte Ausführung wendet den Discount in Stripe an.
- [ ] Aktion `Rabatt entfernen` zeigt bestehende Discounts.
- [ ] Bestätigte Entfernung entfernt den Discount in Stripe.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Refunds

- [ ] Bei einer bezahlten Rechnung ist Aktion `Refund` sichtbar.
- [ ] Refund-Vorschau zeigt Charge, bezahlt, bereits erstattet und offen erstattbar.
- [ ] Stripe-Link in der Refund-Vorschau führt zur passenden Charge.
- [ ] Leerer Refund-Betrag bedeutet vollständige Erstattung.
- [ ] Teilbetrag kann eingegeben werden.
- [ ] Betrag größer als offen erstattbar wird abgelehnt.
- [ ] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [ ] Bestätigte Ausführung erstellt Stripe Refund.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Negative Checks

- [ ] Kunde ohne Stripe Customer ID zeigt verständliche Diagnose.
- [ ] Stripe-Fehler beim Laden der Diagnose wird verständlich angezeigt.
- [ ] Aktionen mit falscher oder fremder Stripe-Zuordnung werden blockiert.
- [ ] Nicht-Admin-Nutzer können die Endpunkte nicht ausführen.

## Abnahmeentscheidung

- [ ] Alle Standard-Supportfälle funktionieren.
- [ ] Komplexe Sonderfälle führen über Stripe-Dashboard-Links statt freier Objektbearbeitung.
- [ ] Pause/Resume ist bewusst nicht Teil des Umfangs: Eine Subscription läuft oder läuft nicht.
- [ ] T020-16 kann nach erfolgreicher manueller Abnahme geschlossen werden.

