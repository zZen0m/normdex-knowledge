### Im Registrierungsformular

Wenn eine E-Mail bereits existiert kommt "Email already registered" als Text. Hier wäre eine Toast-Meldung eher angebracht mit einer deutschen Übersetzung.













**Vorbereitung**

- Öffne https://app.normdex.at in einem frischen Browser/Inkognito-Fenster.
- Lege dir 2-3 Test-E-Mail-Adressen bereit, idealerweise echte Postfächer.
- Halte Stripe Dashboard, Brevo Dashboard, Microsoft 365/Outlook Support-Postfach und n8n Executions offen.
- Nutze für Zahlungen Stripe-Testdaten nur dann, wenn die Keys im Live-Modus testfähig/absichtlich live sind. Aktuell sind Live-Keys gesetzt, also Vorsicht mit echten Zahlungen.

**1. Basis-Check**

- App lädt ohne Basic Auth.
- Keine weiße Seite, keine offensichtlichen Konsolenfehler.
- Login-Seite lädt.
- Registrierung-Seite lädt.
- Impressum/Datenschutz/öffentliche Seiten laden.
- API Health prüfen: https://api.normdex.at/health zeigt {"status":"ok"}.

**2. Registrierung & E-Mail**

- Neuen Account mit Test-E-Mail registrieren.
- Prüfen: Verifizierungs-E-Mail kommt an.
- Link in der Mail anklicken.
- Prüfen: Account wird verifiziert.
- Danach einloggen.
- Passwort-vergessen testen.
- Prüfen: Reset-Mail kommt an.
- Reset-Link öffnen und Passwort ändern.
- Mit neuem Passwort einloggen.

**3. Organisation & Team**

- Nach Login Organisation/Profil vervollständigen.
- Organisationseinstellungen öffnen und speichern.
- Team-Mitglied einladen.
- Prüfen: Einladungs-E-Mail kommt an.
- Einladung in zweitem Browser/Inkognito öffnen.
- Registrierung/Beitritt über Invite testen.
- Prüfen: Mitglied erscheint im Team.
- Rollen/Berechtigungen grob testen: Owner darf Billing/Team, normaler User nicht.

**4. Newsletter & Brevo**

- Landingpage oder Newsletter-Formular öffnen.
- Mit frischer Test-E-Mail anmelden.
- Prüfen in der App: Erfolgsmeldung erscheint.
- Prüfen in Brevo: Kontakt wurde angelegt.
- Prüfen: Kontakt ist in der richtigen Liste.
- Prüfen: Attribute wie Vorname, Nachname, Unternehmen, Rolle, Herkunft sind korrekt.
- Prüfen: Double-Opt-in bzw. Bestätigungsmail kommt, falls so konfiguriert.
- Prüfen: Rabattgutschein/Newsletter-Coupon-Mail kommt an.
- Prüfen: Coupon-Link führt zu /licenses bzw. korrekter App-Seite.
- Gleiche E-Mail nochmal anmelden.
- Erwartung: keine kaputte Doppelanlage; idealerweise saubere “bereits angemeldet”-Meldung oder idempotentes Verhalten.

**5. Stripe Lizenzkauf**

- Als Organisations-Owner einloggen.
- Seite Lizenzen/Subscription öffnen.
- Prüfen: Monats- und Jahrespreise werden angezeigt.
- Prüfen: Preis entspricht gewünschtem Modell.
- Testlizenz/Trial beanspruchen.
- Prüfen: Stripe Checkout öffnet sich korrekt.
- Prüfen: Success-Redirect geht zurück zu https://app.normdex.at/..., nicht localhost.
- Nach Checkout warten und Seite neu laden.
- Prüfen: Lizenzstatus aktiv/trialing sichtbar.
- Prüfen im Stripe Dashboard: Customer, Subscription, Trial, Coupon/Discount korrekt.
- Prüfen in App: Lizenz entsperrt die Wirtschaftlichkeitsberechnung.
- Falls möglich: Billing Portal öffnen.
- Prüfen: Portal öffnet mit richtigem Customer.
- Cancel/Zurück-Flow testen.

**6. Newsletter-Coupon in Stripe**

- Mit Newsletter-Testaccount den Gutschein erhalten.
- Lizenzkauf mit Coupon starten.
- Prüfen im Stripe Checkout: Rabatt wird angewendet.
- Prüfen im Stripe Dashboard: Coupon/Promotion Code am Checkout bzw. Abo hängt.
- Prüfen: Coupon kann nicht mehrfach missbraucht werden, falls so vorgesehen.
- Prüfen: Ablaufdatum/Frist des Coupons ist korrekt.

**7. Lizenzlogik in der App**

- Ohne aktive Lizenz eine lizenzpflichtige Berechnung öffnen.
- Erwartung: Sperre/Hinweis erscheint.
- Mit aktiver Trial/Lizenz öffnen.
- Erwartung: Berechnung ist nutzbar.
- Zusatznutzer-Limit testen: zweiten User in Organisation einladen.
- Prüfen: Zugriff verhält sich passend zur Lizenzanzahl.
- Lizenzübersicht öffnen.
- Prüfen: Status, Zeitraum, Nutzeranzahl, Billing-Periode stimmen.

**8. Wirtschaftlichkeitsberechnung**

- Neues Projekt anlegen.
- Pflichtfelder prüfen.
- Wirtschaftlichkeitsberechnung starten.
- Beispielwerte eingeben.
- Berechnung speichern.
- Ergebnisbericht öffnen.
- PDF/Export erzeugen.
- Prüfen: Logo, Zahlen, Tabellen, Seitenumbrüche, Umlaute, Währung, Datum.
- Projekt erneut öffnen.
- Prüfen: gespeicherte Daten bleiben erhalten.
- Sensitivitätsanalyse testen, falls vorhanden.

**9. Support, MS Graph & n8n**

- Über App ein Support-Ticket erstellen.
- Prüfen in App: Ticket-ID wird angezeigt.
- Prüfen im Support-Postfach: Auto-Reply kommt beim Kunden an.
- Prüfen im Admin-Supportbereich: Ticket erscheint.
- Prüfen in n8n: Execution für ticket.created läuft.
- Prüfen n8n Payload: Ticket-ID, Betreff, Kategorie, E-Mail, Firma korrekt.
- Prüfen n8n Signatur/Header, falls Workflow das validiert.
- Auf Ticket antworten bzw. Status ändern.
- Prüfen: E-Mail geht über MS Graph raus.
- Von extern an support@normdex.at schreiben.
- Prüfen: Graph-Webhook verarbeitet eingehende Mail.
- Prüfen: Ticket oder Nachricht erscheint im Supportbereich.
- Prüfen: keine Mail-Schleife entsteht.

**10. Kontaktformular Landingpage**

- Kontaktformular auf normdex.at testen.
- Prüfen: erstellt ein Public Support Ticket.
- Prüfen: n8n erhält Event.
- Prüfen: Auto-Reply kommt an.
- Falls Newsletter-Checkbox im Kontaktformular existiert: testen, ob sie erwartungsgemäß behandelt wird.

**11. Admin & Monitoring**

- Als Admin einloggen.
- Admin-Dashboard öffnen.
- Nutzerliste prüfen.
- Organisationen prüfen.
- Support-Inbox prüfen.
- Systemfehler/Logs prüfen, falls UI vorhanden.
- Fehlerhafte Aktionen provozieren, z.B. ungültiger Checkout ohne Owner.
- Erwartung: saubere Fehlermeldung, kein Crash.

**12. Security & Session**

- Logout testen.
- Nach Logout geschützte Seite direkt aufrufen.
- Erwartung: Redirect/Login.
- Cookie prüfen: Secure sollte aktiv sein.
- Seite in HTTP statt HTTPS aufrufen, falls möglich.
- Erwartung: HTTPS/Traefik greift.
- Passwort ändern testen.
- Danach mit altem Passwort einloggen.
- Erwartung: geht nicht.

**13. Mobile/Browser**

- App auf Desktop Chrome testen.
- App auf Safari/Firefox testen, falls verfügbar.
- Mobile View testen.
- Login, Lizenzen, Berechnungsformular, Report grob prüfen.
- Keine überlappenden Buttons/Tabellen auf Mobile.

**Besonders Wichtig**  
Die kritischsten Tests sind in dieser Reihenfolge:

1. Registrierung + E-Mail-Verifizierung.
2. Login + Session/Cookies.
3. Newsletter → Brevo → Coupon-Mail.
4. Stripe Trial/Checkout → Webhook → Lizenz aktiv.
5. Lizenz entsperrt Berechnung.
6. Supportticket → MS Graph Auto-Mail → n8n Event.
7. Eingehende Support-Mail über MS Graph Webhook.

Wenn du nur eine kompakte Smoke-Test-Runde machen willst, nimm genau diese sieben Punkte.