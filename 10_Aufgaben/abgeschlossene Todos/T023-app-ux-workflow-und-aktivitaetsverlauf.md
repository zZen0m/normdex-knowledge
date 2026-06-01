# T023 · App-UX, Workflow-Fehler und Aktivitätsverlauf verbessern

**Priorität:** P2 · App / UX / Workflows / Notifications / Team  
**Status:** erledigt  
**Datum:** 2026-05-31  
**Abgeschlossen:** 2026-06-01  

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

- [x] Das Berechnungsformular ist auf kleinen Bildschirmgrößen ohne Layoutbrüche nutzbar.
- [x] Systeme können in der Wirtschaftlichkeitsberechnung dupliziert werden.
- [x] Die Projektanlage fragt zuerst PLZ und danach Ort ab.
- [x] Team-Einladungen prüfen vor dem Versand, ob die E-Mail-Adresse bereits registriert ist.
- [x] Bereits registrierte E-Mail-Adressen können nicht eingeladen werden und zeigen einen verständlichen Toast.
- [x] Fehlende Auswahl von Herr/Frau im Registrierungsformular zeigt eine nutzerfreundliche Validierungsmeldung.
- [x] Notifications und weitere relevante Zeitangaben werden korrekt für Europe/Vienna dargestellt.
- [x] Alle nutzerrelevanten Aktivitätstypen sind inventarisiert.
- [x] Der Aktivitätsverlauf zeigt keine technischen Code-Bezeichnungen mehr, sondern lesbare deutsche Texte.

## Notizen / Fortschritt

- 2026-05-31: Sammelaufgabe aus Nutzungsfeedback angelegt.
- 2026-05-31: Schnelle Wins umgesetzt:
  - **Projektanlage (PLZ vor Ort):** Feldreihenfolge in `NewProject.tsx` und im Bearbeiten-Formular `ProjectDetail.tsx` jeweils für Projektadresse, Auftraggeber und Sachbearbeitung auf PLZ → Ort umgestellt. (`ProjectSection.tsx` zeigte PLZ Ort bereits korrekt an.)
  - **Registrierung (Anrede):** In `Register.tsx` das Zod-Schema von `required_error` auf `errorMap` umgestellt, sodass auch die leere Auswahl die freundliche Meldung „Bitte wähle eine Anrede aus." zeigt statt der technischen Enum-Fehlermeldung.
  - **Team-Einladungen:** Backend `teams.py` (`create_invite`) blockiert jetzt Einladungen an bereits registrierte E-Mail-Adressen (nicht nur bestehende Mitglieder) mit klarer deutscher Meldung; Frontend `Team.tsx` zeigt die Server-Meldung als Toast (`getApiErrorMessage`) statt eines generischen Texts.
  - Damals offen geblieben: Responsive-Fixes + System-Duplizierung (Wirtschaftlichkeitsberechnung), Zeitzone Europe/Vienna, Aktivitätsverlauf-Übersetzungen.
- 2026-05-31: Wirtschaftlichkeitsberechnung Punkt 1 umgesetzt:
  - Systeme können im Systeme-Tab im Bearbeitungsmodus dupliziert werden. System-Metadaten und alle zugehörigen Kostenpositionen werden mit neuer System-ID übernommen; maximal drei Systeme bleiben unverändert.
  - Die Kostenbereiche der Systemeingabe wechseln auf kleinen Bildschirmen auf mobile Karten und bleiben ab Tablet/Desktop als Tabellen erhalten.
  - Frontend-Build erfolgreich geprüft mit `npm run build` in `apps/frontend/`.
  - Danach offen geblieben: Zeitzone Europe/Vienna vollständig prüfen, Aktivitätsverlauf-Übersetzungen.
- 2026-05-31: Aktivitätsverlauf bereinigt:
  - Admin-/Support-Audit-Events (`admin_*`) werden im normalen Dashboard-Feed herausgefiltert, weil sie keinen Nutzen für Kund:innen im App-Feed stiften.
  - Nutzerrelevante fehlende Events (`register`, `profile_updated`, `account_delete_requested`, `email_change_blocked`) wurden im Feed ergänzt.
  - Der Fallback zeigt keine technischen Event-Codes mehr an.
  - ActivityFeed nutzt für Zeitangaben die zentrale Europe/Vienna-Datumslogik.
  - Geprüft mit `npm run build` in `apps/frontend/` und `.\venv\Scripts\python -m py_compile app\routers\dashboard.py` in `apps/api/`.
  - Offen verbleibt: Zeitzone Europe/Vienna an weiteren relevanten Stellen vollständig prüfen.
- 2026-06-01: Zeitzonen-Sweep abgeschlossen:
  - Zentrale Datumslogik in `apps/frontend/src/lib/datetime.ts` erweitert (`parseServerDate`, `formatServerDate`, `formatDateTime`, `formatDate`, `getServerTime`, `VIENNA_TZ`).
  - Relevante Datumsanzeigen in Dashboard, Projekten, Lizenzen, Support-Inbox, Support-Ticket-Detail, Organisationscase und Projektauswahl auf zentrale UTC/Vienna-Logik umgestellt.
  - Sortierungen nach Server-Zeitstempeln nutzen nun den zentralen Parser, damit naive UTC-Werte nicht als Browser-Lokalzeit interpretiert werden.
  - Geprüft mit `npm test` und `npm run build` in `apps/frontend/`.
