# T038 · Support-Ticket direkt auf "Geschlossen" setzen (Bounce-Teufelskreis vermeiden)

**Phase:** App / Support / Ticketsystem / E-Mail
**Priorität:** P2 · UX-Bug / Support-Workflow
**Status:** erledigt
**Datum:** 2026-07-03
**Abgeschlossen:** 2026-07-04

## Ziel

Im Ticket-Detail-View soll ein zusätzliches Dropdown-Element ergänzt werden, mit dem ein Support-Mitarbeiter ein Ticket direkt auf den Status **"Geschlossen"** setzen kann — unter Umgehung des regulären "Gelöst"-Flows. Da dies bewusst die reguläre Funktionalität (Kundenbenachrichtigung + automatisches Wiederöffnen bei Antwort) aushebelt, muss vor Ausführung ein Bestätigungsdialog erscheinen.

## Problembeschreibung

### Was passiert heute

Setzt ein Support-Mitarbeiter ein Ticket auf **"Gelöst"**, durchläuft es folgenden Flow (`support_admin.py:359-367`):

1. Status wird sofort auf `resolved_pending` gesetzt, `auto_close_at = now + 1h`.
2. Der Scheduler-Job (`scheduler.py`, Funktion für `resolved_pending`-Tickets, stündlich) setzt das Ticket nach Ablauf der Stunde auf `resolved` und verschickt eine Benachrichtigungsmail an `ticket.requester_email`.
3. Existiert diese E-Mail-Adresse nicht, bounct der Mailserver eine Unzustellbarkeits-Antwort zurück.
4. Der eingehende Antwort-Handler (`support.py:639-644`) reagiert auf jede eingehende Nachricht zu einem Ticket im Status `resolved` oder `waiting_on_customer` und setzt es automatisch zurück auf `open`.
5. Damit ist das Ticket wieder offen, obwohl es de facto erledigt ist — bei jedem erneuten "Lösen" wiederholt sich der Kreislauf (**Teufelskreis**).

### Warum "Geschlossen" das Problem löst

Der Reopen-Handler (`support.py:639-644`) prüft explizit:

```python
# Only reopen if resolved or waiting_on_customer. Closed tickets stay closed.
if ticket.status in ["resolved", "waiting_on_customer"]:
    ...
```

Tickets im Status `closed` werden also **nie** durch eine eingehende Antwort (inkl. Bounce) wieder geöffnet. Ein direkter Wechsel auf `closed` statt `resolved` umgeht damit zuverlässig den Bounce-Teufelskreis.

### Backend ist bereits vorbereitet

Der PATCH-Endpunkt `update_ticket` in `support_admin.py` unterstützt den direkten Statuswechsel auf `closed` bereits vollständig (Zeile 372-373, 408-409):

- Setzt `ticket.closed_at`
- Verschickt eine "closed"-Benachrichtigungsmail (`send_status_email(..., "closed", ...)`) — falls die Adresse ungültig ist, bounct diese zwar ebenfalls, löst aber **kein Reopen** aus (siehe oben).
- Kein Timer/Delay wie bei `resolved` — die Statusänderung ist sofort wirksam.

**Der fehlende Teil ist rein im Frontend:** Im Ticket-Detail-Dropdown wird `closed` aktuell bewusst aus den wählbaren Status-Optionen herausgefiltert:

```tsx
// SupportTicketDetail.tsx:596-598
{Object.entries(statusLabels)
    .filter(([key]) => key !== 'triaged' && key !== 'resolved_pending' && key !== 'closed')
    ...
```

`closed` wurde bisher offenbar bewusst als "nur automatisch erreichbarer" Zwischenstatus behandelt (analog zu `resolved_pending`), nicht als manuell wählbare Option.

### Betroffene Dateien

- `apps/frontend/src/pages/admin/SupportTicketDetail.tsx` — Status-Dropdown (Zeile ~585-604), Bestätigungsdialog ergänzen
- `apps/api/app/routers/support_admin.py` — PATCH `/tickets/{ticket_id}` (kein Änderungsbedarf, unterstützt `closed` bereits)
- `apps/api/app/routers/support.py` — Reopen-Logik bei eingehender Antwort (Zeile 639-644, zur Absicherung/Referenz, kein Änderungsbedarf)

## Lösungsweg

### Schritt 1 — `closed` im Dropdown wieder zulassen

In `SupportTicketDetail.tsx` den Filter anpassen, sodass `closed` als wählbare Option erscheint (weiterhin ohne `triaged` und `resolved_pending`, da diese echte Zwischenzustände ohne manuellen Zugriff bleiben sollen).

### Schritt 2 — Bestätigungsdialog ergänzen

**Designsystem-Check (durchgeführt 2026-07-03):** Es gibt keine dedizierte `AlertDialog`/`ConfirmDialog`-Komponente. Das etablierte Muster im Frontend ist der generische `Dialog` aus `components/ui/dialog.tsx` (`Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogDescription`, `DialogFooter`), lokal pro Seite mit eigenem Confirm-State instanziiert — siehe Referenzimplementierung in `apps/frontend/src/pages/Team.tsx` (Zeile ~16, ~94, ~568-584: lokaler `confirm`-State mit `title`/`description`/`onConfirm`, gerendert über `Dialog open={confirm.open} ...`). Für T038 dieses Muster 1:1 in `SupportTicketDetail.tsx` übernehmen, keine neue globale Komponente bauen.

Beim Auswählen von `closed` im Dropdown (`handleStatusChange`) nicht den bestehenden Inline-Save/Cancel-Mechanismus (Häkchen/X-Icons neben dem Badge) greifen lassen, sondern den lokalen Confirm-Dialog öffnen, generischer Text (keine Erwähnung des E-Mail-Sonderfalls):

> "Ticket wirklich direkt auf **Geschlossen** setzen? Der reguläre Ablauf über 'Gelöst' (inkl. Wartezeit und automatischem Wiederöffnen bei Kundenantwort) wird dabei übersprungen."

Erst nach Bestätigung `updateStatus("closed")` auslösen; bei Abbruch bleibt der bisherige Status unverändert (`pendingStatus` zurücksetzen).

### Schritt 3 — Tests / manuelle Prüfung

- Ticket mit gültiger E-Mail direkt auf `closed` setzen → Bestätigungsdialog erscheint, nach Bestätigung Status `closed`, Benachrichtigungsmail wird verschickt.
- Ticket mit ungültiger/nicht existierender E-Mail direkt auf `closed` setzen → Bounce-Antwort trifft ein → Ticket bleibt `closed` (kein Reopen).
- Dialog abbrechen → Status bleibt unverändert.
- Bestehender "Gelöst"-Flow (`resolved_pending` → `resolved` nach 1h) bleibt unverändert funktionsfähig.

## Akzeptanzkriterien

- [x] Dropdown im Ticket-Detail bietet "Geschlossen" als manuell wählbare Option an.
- [x] Auswahl von "Geschlossen" löst einen Bestätigungsdialog aus, der die Konsequenz (Umgehung des regulären Gelöst-Flows) klar benennt.
- [x] Nach Bestätigung wird der Status korrekt auf `closed` gesetzt (bestehender Backend-Endpunkt, keine Änderung nötig).
- [x] Bei Abbruch des Dialogs bleibt der ursprüngliche Status erhalten.
- [ ] Ticket mit ungültiger E-Mail-Adresse, das direkt auf `closed` gesetzt wird, öffnet sich bei einer Bounce-Antwort **nicht** erneut. (bereits durch Backend-Reopen-Logik abgedeckt, manuell nicht erneut verifiziert)

## Notizen / Fortschritt

- 2026-07-03: Todo angelegt. Auslöser: Wiederkehrender Teufelskreis bei Tickets mit ungültiger Kunden-E-Mail — "Gelöst" → Benachrichtigungsmail nach 1h → Bounce → automatisches Reopen. Backend unterstützt direkten `closed`-Wechsel bereits vollständig; Lösung ist im Wesentlichen ein reiner Frontend-Task (Dropdown-Filter + Bestätigungsdialog).
- 2026-07-04: Umgesetzt im Repo `normdex-webapp-dev` (`apps/frontend/src/pages/admin/SupportTicketDetail.tsx`): `closed` aus dem Dropdown-Filter entfernt (bleibt wählbar), `handleStatusChange` verzweigt bei `closed` auf einen lokalen Confirm-Dialog (Muster aus `Team.tsx` übernommen) statt auf den Inline-Save/Cancel-Mechanismus. Bei Bestätigung wird `updateStatus("closed")` ausgelöst, bei Abbruch bleibt der Status unverändert. `tsc --noEmit` lief fehlerfrei durch. Keine Backend-Änderungen nötig (wie vorgesehen).
