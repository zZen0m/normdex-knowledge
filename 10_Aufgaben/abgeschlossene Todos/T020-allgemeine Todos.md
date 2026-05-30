# T020 - Allgemeine Todos (Sammelaufgabe)

**Status:** erledigt
**Abgeschlossen:** 2026-05-30
**Bereich:** App

Es gibt noch allgemeine Dinge, die mir aufgefallen sind und die ich hier nun einzeln aufführe. Diese Punkte sollen dann nach der Reihe abgearbeitet werden.

Alle Sub-Todos T020-01 bis T020-12 sind abgeschlossen. Der weitere Verwaltungsportal-Ausbau wird über separate Folge-Todos wie [[T020-16-Lizenz-und-Billing-Support-Aktionen]] geführt.

---

## Aufteilung in Sub-To-dos & Reihenfolge

Die ursprünglichen Punkte wurden in zwölf einzelne, atomar umsetzbare Sub-To-dos zerlegt und priorisiert.
Plan-Datei: `C:\Users\Andreas\.claude\plans\lies-dir-das-to-do-virtual-matsumoto.md`

### Phase 1 — Quick Wins (UI-Polish, niedriges Risiko)
1. [[T020-01-Economics-Label entfernen]]
2. [[T020-02-Spilling-Portal neuer Tab]]
3. [[T020-03-Projekttabelle Layout]]
4. [[T020-04-Char-Counter Projekt bearbeiten]]
5. [[T020-05-3 Karten Polish]]

### Phase 2 — Funktionale Lücken (Bug-artig)
6. [[T020-06-Stripe Adress-Sync]]
7. [[T020-07-Testlizenz kuendigen]]

### Phase 3 — Mittlere Features
8. [[T020-08-Lizenz hinzufuegen Redesign]]
9. [[T020-09-Berechnungen-Karte Hero]]
10. [[T020-10-Export-Tab Umbau]]
11. [[T020-11-Resuemee-Tab]]  *(abhängig von T020-10)*

### Phase 4 — Konzept
12. [[T020-12-Konzept Verwaltungsportal]]  *(Konzept abgeschlossen; Implementierungs-To-dos folgen separat)*

---

## Original-Notizen




## Lizenzverwaltung
### Weiterleitung via neuer Tab
![[Pasted image 20260501161134.png]]
Wenn man das Spilling-Portal öffnet, sollte ein neuer Tab geöffnet werden, und man soll nicht direkt in das Spilling-Portal switchen, sondern immer über einen neuen Tab.


### Kündigung der Testlizenz
Es soll die Möglichkeit geben, auch die Testlizenz über einen Button zu kündigen. 

Wenn man die Lizenz kündigt, soll die Testlizenz trotzdem bis zum Ende des Testzeitraums weiterlaufen. Sie soll jedoch nicht in eine aktive Hauptlizenz überführt werden, sondern nach Ablauf des Testzeitraums einfach entfernt werden.


### UI "Lizenz hinzufügen"
![[Pasted image 20260501161435.png]]
Ich finde dieses Pop-up zum Hinzufügen von Lizenzen sehr rudimentär und optisch nicht ansprechend. Zudem fehlt die Möglichkeit, Gutscheincodes einzugeben. Da man beim Erwerb von Zusatzlizenzen nicht zu Stripe weitergeleitet wird, hat man faktisch nie die Gelegenheit, einen Code einzulösen. Das ist aus meiner Sicht ein Manko.

Auch die grafische Übersicht ist nicht besonders gut gelungen. Insgesamt hätte ich hier eine professionellere Lösung erwartet; das Ganze sollte optisch ansprechender gestaltet sein.

## Projektseite
### Projektübersicht / Projektdetails

![[Pasted image 20260501161647.png]]
![[Pasted image 20260501161658.png]]

Die Tabelle mit den einzelnen Projekten sollte so breit sein wie die Projektdetailseite. Die Überschrift sollte zudem linksbündig ausgerichtet werden, sodass sie eine vertikale Linie mit dem Rest bildet.

Außerdem finde ich die Projektdetailseite aktuell noch nicht so ansprechend; diese sollte man optisch besser gestalten.

Hinsichtlich der Berechnungen:
1. Die Karte mit den Berechnungen ist aktuell darauf ausgelegt, dass man hier mehrere verschiedene Berechnungen durchführen könnte.
2. Momentan gibt es jedoch nur die Ö-Norm (die Wirtschaftlichkeitsberechnung).
3. Da nur diese eine Option existiert, wirkt die Karte unten irgendwie deplatziert. Hier sollten wir uns eine bessere Lösung einfallen lassen.

Was mir noch aufgefallen ist: Auf der Projektdetailseite steht unter dem Projekttitel noch ein Label mit "Economics". Das kannst du komplett entfernen.




### Projekt bearbeiten
![[Pasted image 20260501161847.png]]

![[Pasted image 20260501162003.png]]
Auf der Seite, wo man das Projekt bearbeiten kann, fehlt die Anzeige für die maximale Anzahl an Zeichen. Es gibt diese Begrenzung zwar, aber sie wird nicht eingeblendet – zum Beispiel bei der Beschreibung. In anderen Bereichen, etwa wenn man den Support kontaktiert, gibt es bei der Problembeschreibung und beim Betreff wird eine maximale Zeichenanzahl angezeigt, die man eingeben kann. So etwas fehlt mir hier.

Mir gefällt auch die Art und Weise nicht, wie die ganzen Elemente aktuell angezeigt werden. Das kann man meiner Meinung nach noch etwas hübscher gestalten, zum Beispiel die drei Karten mit Projektadresse, Auftraggeber und Sachbearbeitung. Da könnte man bestimmt noch bessere Lösungen finden und beispielsweise mehr mit Icons arbeiten.

## Wirtschaftlichkeitsberechnung

### Übersichtsseite
![[Pasted image 20260501162250.png]]
Entferne auch hier bitte das Label Economics.


### Bericht exportieren
![[Pasted image 20260501162349.png]]

Hier gefällt mir die Aufteilung generell noch nicht so. Oben wird der Tab als Bericht bezeichnet, aber ich finde, Export würde hier besser passen.

Bezüglich der Berichtsinhalte wäre Folgendes gut:
1. Alles sollte standardmäßig eingeklappt sein.
2. Alle Optionen sollten standardmäßig angehakt sein.
3. Diese Häkchen sollen gespeichert werden, sodass beim nächsten Export automatisch wieder die genauen Einstellungen gesetzt sind.

Außerdem fehlt noch der Haken, wo man das Resümee ein- und ausblenden kann. Das würde ich im Punkt "Allgemein" hinzufügen.

Erstelle dort bitte noch einen weiteren Haken für das Resümee, damit man wählen kann, ob man das Resümee mit exportieren möchte oder nicht.

### Resümee
Und es bräuchte noch einen weiteren Tab. Diesen kannst du zwischen den Ergebnissen und dem Tab Bericht (der dann auf Export geändert wird) einfügen.

Der neue Tab erhält den Namen Resümee. Dort können wir zwei Elemente integrieren:

1. Eine Auswahlmöglichkeit zwischen den Systemen (eine Art Dropdown-Feld), in dem festgelegt wird, welches System seitens des Sachbearbeiters wirklich empfohlen wird.
2. Ein Textfeld für das Resümee, in dem der Kunde einen Text schreiben kann, der später im Bericht vermerkt wird.

So hat man eine Grundlage, um den Text und die Auswahl auf der entsprechenden Resümee-Seite im Bericht einzufügen.



## Verwaltung
![[Pasted image 20260501162843.png]]

Als Betreiber dieses Services und dieser Software möchte ich umfassenden Kundensupport leisten können. Das kann ich einerseits mit Hilfe des Ticketsystems tun, da ich Zugriff auf das Ticketportal habe. Andererseits möchte ich aber auch einzelne Details von Kunden sehen und bearbeiten können.

Aktuell gibt es eine Verwaltungsseite, auf der man alle registrierten Kunden, alle Daten und so weiter sieht. Diese Seite soll jedoch noch umfangreicher gestaltet werden, damit ich:
1. Daten auf Kundenwunsch hin ändern kann.
2. Alle Einstellungen anpassen kann.
   (a) Wenn ein Kunde beispielsweise eine Lizenz gekündigt oder einen Rabatt haben möchte, möchte ich die volle Kontrolle haben und alle Einstellungen treffen können.
   (b) Wenn ein Kunde seine gesamten Daten gelöscht haben möchte, soll ich hier alles löschen können.
3. Hilfestellung bei der Wirtschaftlichkeitsberechnung geben kann, also bei spezifischen Projekten unterstützen kann.

Bitte hilf mir hier einen Plan zu erstellen, welche Funktionen dieses Verwaltungsportal haben muss, damit ich ordentlichen Kundensupport durchführen kann.



## Rechnungsadresse

Aktuell gibt es bei den Unternehmenseinstellungen die Unternehmensadresse und die Rechnungsadresse. Ich bin mir jedoch nicht sicher, welche Adresse auf der Rechnung erscheint, wenn der Kunde diese Daten ausfüllt und eine Stripe-Subscription abschließt.

Ich will natürlich, dass die Rechnungsadresse auf der Rechnung erscheinen wird. Wenn der Kunde nachträglich (also während einer bestehenden Subskription) die Rechnungsadresse ändert, dann soll bei der nächsten Abrechnung die aktuelle Adresse verwendet werden. Das ist mir gerade eingefallen, und ich bin mir gar nicht sicher, ob das überhaupt implementiert ist.
