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

- [x] Bei einer Lizenz mit aktiver Nutzung Aktion `Freigeben` öffnen.
- [x] Vorschau zeigt Anzahl aktiver Nutzungen.
- [x] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [x] Bestätigte Ausführung beendet aktive Nutzungen.
- [x] Lizenz ist danach wieder nutzbar.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Stripe-Status-Resync

- [x] Aktion `Sync` bei einer Stripe-verknüpften Lizenz öffnen.
- [x] Vorschau zeigt erkannte Abweichungen zwischen Stripe und Normdex.
- [x] Bei keinen Abweichungen wird das verständlich angezeigt.
- [x] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [x] Bestätigte Ausführung übernimmt Status-/Periodenänderungen.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Rabatte / Coupons

- [x] Bei einer Subscription Aktion `Rabatt` öffnen.
- [x] Ungültiger oder abgelaufener Promotion Code wird abgelehnt.
- [ ] Gültiger Promotion Code zeigt Coupon-Wert, Dauer und bestehenden Rabatt. EDIT: 23.06.2026: Nein, es wird nichts angezeigt: Lesbare Rabattdetails = leer
- [ ] Bestehender Rabatt wird als zu ersetzender Discount angezeigt. EDIT: 23.06.2026: Nein, auch wenn ein bestehender Rabatt bereits existiert wird dieser in der Vorschau nicht angezeigt.
"Rabatt-Vorschau Subscriptionsub_1Tg6qEF05ipkEAzmYUNmT0Uq
Stripe-Status active
Aktuelle Rabatte —
Ersetzt bestehende Rabatte Nein"
- [x] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [x] Bestätigte Ausführung wendet den Discount in Stripe an.
- [ ] Aktion `Rabatt entfernen` zeigt bestehende Discounts. EDIT: 23.06.2026: Nein, auch wenn ein bestehender Rabatt bereits existiert wird dieser in der Vorschau nicht angezeigt.
"Rabatt-Vorschau Subscriptionsub_1Tg6qEF05ipkEAzmYUNmT0Uq
Stripe-Status active
Aktuelle Rabatte —
Ersetzt bestehende Rabatte Nein"
- [ ] Bestätigte Entfernung entfernt den Discount in Stripe. EDIT: 23.06.2026: Im Testfall wurde der Rabatt in Normdex entfernt, die Toastmeldung kam als Bestätigung. Aber der Discount ist in Stripe immer noch vorhanden. (In Stripe: Testgutschein 25 % Rabatt, unbegrenzt)
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Refunds

- [x] Bei einer bezahlten Rechnung ist Aktion `Refund` sichtbar.
- [x] Refund-Vorschau zeigt Charge, bezahlt, bereits erstattet und offen erstattbar.
- [x] Stripe-Link in der Refund-Vorschau führt zur passenden Charge.
- [x] Leerer Refund-Betrag bedeutet vollständige Erstattung.
- [x] Teilbetrag kann eingegeben werden.
- [x] Betrag größer als offen erstattbar wird abgelehnt.
- [x] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [x] Bestätigte Ausführung erstellt Stripe Refund.
- [x] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Negative Checks

- [x] Kunde ohne Stripe Customer ID zeigt verständliche Diagnose.
- [x] Stripe-Fehler beim Laden der Diagnose wird verständlich angezeigt.
- [x] Aktionen mit falscher oder fremder Stripe-Zuordnung werden blockiert.
- [x] Nicht-Admin-Nutzer können die Endpunkte nicht ausführen.

## Abnahmeentscheidung

- [x] Alle Standard-Supportfälle funktionieren.
- [x] Komplexe Sonderfälle führen über Stripe-Dashboard-Links statt freier Objektbearbeitung.
- [x] Pause/Resume ist bewusst nicht Teil des Umfangs: Eine Subscription läuft oder läuft nicht.
- [x] T020-16 kann nach erfolgreicher manueller Abnahme geschlossen werden.

