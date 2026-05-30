# T020-16 · Abnahme-Checkliste Billing-Support

**Bezug:** [[T020-16-Lizenz-und-Billing-Support-Aktionen]]  
**Datum:** 2026-05-30  
**Ziel:** Manuelle Abnahme der Admin-Billing- und Stripe-Support-Funktionen vor Abschluss von T020-16.

## Vorbereitung

- [ ] Lokale App starten und als Admin anmelden.
- [ ] Einen Testkunden mit Stripe Customer ID öffnen.
- [ ] Sicherstellen, dass Stripe-Testmodus verwendet wird, wenn echte Stripe-Aktionen getestet werden.
- [ ] Einen Testkunden wählen, der mindestens eine Subscription und eine Rechnung in Stripe hat.
- [ ] Optional ein offenes Support-Ticket beim Testkunden bereithalten, um Ticket-Verknüpfung zu prüfen.

## Billing-Diagnose

- [ ] Organisationsakte öffnen und Tab `Billing & Stripe` auswählen.
- [ ] `Diagnose laden` ausführen.
- [ ] Stripe Customer wird angezeigt.
- [ ] Stripe Subscriptions werden angezeigt.
- [ ] Stripe Rechnungen werden angezeigt.
- [ ] Lokale Lizenzen werden angezeigt.
- [ ] Offene Rechnungen werden als Hinweis sichtbar.
- [ ] Zahlungsprobleme oder Past-Due-Zustände werden als Hinweis sichtbar.
- [ ] Abweichungen zwischen lokalen Lizenzdaten und Stripe werden als Diagnose-Warnung sichtbar.

## Stripe-Deep-Links

- [ ] `Stripe Customer öffnen` führt zum richtigen Stripe Customer.
- [ ] `Stripe` bei einer Subscription führt zur richtigen Stripe Subscription.
- [ ] `Stripe` bei einer Rechnung führt zur richtigen Stripe Invoice.
- [ ] `Zahlung` bei einer Rechnung führt zur passenden Stripe Payment-/Charge-Ansicht.
- [ ] Links öffnen im Testmodus unter `dashboard.stripe.com/test`, wenn ein Test-Secret-Key aktiv ist.

## Kundendaten-Sync

- [ ] Aktion `Billing-Adresse synchronisieren` öffnen.
- [ ] Vorschau zeigt Vorher/Nachher für Name, E-Mail, Rechnungsadresse und USt-ID.
- [ ] Ausführung ohne Grund wird blockiert.
- [ ] Ausführung ohne Bestätigung wird blockiert.
- [ ] Ausführung mit Grund und Bestätigung synchronisiert nach Stripe.
- [ ] Optionales Support-Ticket kann ausgewählt werden.
- [ ] Nach erfolgreicher Ausführung wird die Billing-Diagnose aktualisiert.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Zahlungsart / Billing Portal

- [ ] Aktion `Zahlungsart aktualisieren` öffnen.
- [ ] Ausführung ohne Grund wird blockiert.
- [ ] Ausführung ohne Bestätigung wird blockiert.
- [ ] Ausführung mit Grund und Bestätigung öffnet eine Stripe-hosted Billing-Portal-Session.
- [ ] Rücksprung aus dem Billing Portal führt wieder in die Admin-Organisationsakte.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

## Kündigung und Reaktivierung

- [ ] Bei einer aktiven Lizenz Aktion `Kündigung` öffnen.
- [ ] Vorschau zeigt geplantes Ende und Stripe-Bezug.
- [ ] Ausführung ohne Grund oder Bestätigung wird blockiert.
- [ ] Bestätigte Kündigung setzt lokale Lizenz auf auslaufend.
- [ ] Stripe Subscription/Item wird korrekt zum Periodenende behandelt.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.
- [ ] Bei einer auslaufenden Lizenz Aktion `Reaktivierung` öffnen.
- [ ] Vorschau zeigt bisher geplantes Ende.
- [ ] Bestätigte Reaktivierung hebt die Kündigung zurück.
- [ ] Audit-/Timeline-Eintrag ist nachvollziehbar.

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

