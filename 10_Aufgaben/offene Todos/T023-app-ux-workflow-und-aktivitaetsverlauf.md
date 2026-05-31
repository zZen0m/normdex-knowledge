# T023 · App-UX, Workflow-Fehler und Aktivitätsverlauf verbessern

**Priorität:** P2 · App / UX / Workflows / Notifications / Team  
**Status:** offen  
**Datum:** 2026-05-31  

## Ziel

Mehrere aktuell beobachtete UX- und Workflow-Probleme in der Normdex-App sollen gebündelt geprüft und behoben werden. Der Fokus liegt auf kleinen Bildschirmgrößen, klaren Formularabläufen, verständlichen Fehlermeldungen, korrekten Uhrzeiten und lesbaren Aktivitätstexten.

## Kontext

Die Punkte wurden aus der laufenden Nutzung gesammelt und betreffen unterschiedliche App-Bereiche. Sie sollen als ein zusammenhängendes Arbeitspaket behandelt werden, weil mehrere Themen direkte Auswirkungen auf Nutzbarkeit, Supportaufwand und Vertrauen in die App haben.

## Umfang

### Wirtschaftlichkeitsberechnung

- Im Berechnungsformular der Wirtschaftlichkeitsberechnung gibt es aktuell noch responsive Probleme bei kleinen Auflösungen bzw. kleinen Bildschirmen.
- Systeme sollen einfach dupliziert werden können, damit bestehende Eingaben als Ausgangspunkt für Varianten genutzt werden können.

### Projektanlage

- Beim Erstellen eines Projekts soll zuerst die PLZ und danach der Ort eingegeben werden.
- Formularreihenfolge, Labels und Validierung sollen entsprechend angepasst werden.

### Team-Einladungen

- Beim Einladen eines Teammitglieds muss zuerst geprüft werden, ob die E-Mail-Adresse bereits registriert ist.
- Es dürfen nur E-Mail-Adressen eingeladen werden, die noch keinen Account haben.
- Falls die E-Mail-Adresse bereits registriert ist, soll der Workflow blockiert werden.
- Die Fehlermeldung soll als verständlicher Toast angezeigt werden.

### Registrierung

- Wenn im Registrierungsformular Herr/Frau nicht ausgewählt wird, erscheint aktuell eine sehr nutzerunfreundliche Meldung.
- Die Validierung und Fehlermeldung sollen verständlicher und konsistent mit den übrigen Formularen angezeigt werden.

### Uhrzeiten und Zeitzone

- Uhrzeiten stimmen teilweise nicht, vermutlich weil die Zeitverschiebung zur Vienna-Zeitzone nicht berücksichtigt wird.
- Konkret aufgefallen ist das bei Notifications.
- Auch andere Stellen können betroffen sein und sollen geprüft werden.
- Ziel ist eine konsistente Darstellung in der Zeitzone Europe/Vienna.

### Aktivitätsverlauf

- Im Aktivitätsverlauf werden teilweise noch technische Code-Bezeichnungen statt lesbarer Texte angezeigt.
- Beispiel: `Andreas Gruber hat "admin billing license cancelled" durchgeführt.`
- Es soll eine vollständige Liste aller möglichen Aktivitäten erstellt werden.
- Für jede Aktivität soll eine nutzerfreundliche deutsche Übersetzung bzw. Anzeigeform definiert und umgesetzt werden.

## Akzeptanzkriterien

- [ ] Das Berechnungsformular ist auf kleinen Bildschirmgrößen ohne Layoutbrüche nutzbar.
- [ ] Systeme können in der Wirtschaftlichkeitsberechnung dupliziert werden.
- [ ] Die Projektanlage fragt zuerst PLZ und danach Ort ab.
- [ ] Team-Einladungen prüfen vor dem Versand, ob die E-Mail-Adresse bereits registriert ist.
- [ ] Bereits registrierte E-Mail-Adressen können nicht eingeladen werden und zeigen einen verständlichen Toast.
- [ ] Fehlende Auswahl von Herr/Frau im Registrierungsformular zeigt eine nutzerfreundliche Validierungsmeldung.
- [ ] Notifications und weitere relevante Zeitangaben werden korrekt für Europe/Vienna dargestellt.
- [ ] Alle Aktivitätstypen sind inventarisiert.
- [ ] Der Aktivitätsverlauf zeigt keine technischen Code-Bezeichnungen mehr, sondern lesbare deutsche Texte.

## Notizen / Fortschritt

- 2026-05-31: Sammelaufgabe aus Nutzungsfeedback angelegt.
