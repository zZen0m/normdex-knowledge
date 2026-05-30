# Toast-Benachrichtigungen

## Zweck

Toast-Benachrichtigungen geben kurzes, nicht-blockierendes Feedback zu abgeschlossenen oder fehlgeschlagenen Aktionen in der Normdex App. Sie ersetzen keine Formularvalidierung im Feld und keine Bestätigungsdialoge für irreversible Aktionen.

## Abgrenzung: Toasts vs. persistente Benachrichtigungen

Toasts sind **flüchtige UI-Signale**: sie reagieren auf die unmittelbare Aktion des Nutzers und sind nach wenigen Sekunden weg. Persistente **In-App-Benachrichtigungen** (siehe [[Funktionen im Detail#Benachrichtigungen]]) sind dagegen Datensätze in der Datenbank, die auch nach Reload oder Re-Login sichtbar bleiben und ungelesen/gelesen-Status tragen.

Bei einer Live-Notification (über SSE empfangen) wird **zusätzlich** ein `notify.info`-Toast angezeigt, damit der User die neue Notification sofort wahrnimmt — der Toast ist hier das Aufmerksamkeitssignal, die persistente Notification der eigentliche Inhalt.

## Technische Grundlage

- Toast-System: Sonner
- Zentrale Toaster-Konfiguration: `apps/frontend/src/components/ui/sonner.tsx`
- Zentraler Standard-Helper: `apps/frontend/src/lib/notify.tsx`
- Standardaufrufe erfolgen über `notify.success`, `notify.error`, `notify.warning`, `notify.info` oder `notify.loading`

Direkte Standardaufrufe wie `toast.success(...)` oder `toast.error(...)` sollen nicht mehr verwendet werden. Fachliche Custom-Toasts mit eigener Interaktion dürfen Sonner direkt verwenden.

## Position und Verhalten

- Desktop: rechts unten
- Mobile: weiterhin unten, mit maximaler Breite innerhalb des Viewports
- Toasts sind manuell schließbar
- Automatisch verschwindende Standard-Toasts zeigen einen Ablaufbalken
- Bei `duration: Infinity` wird kein Ablaufbalken angezeigt
- Bei `prefers-reduced-motion: reduce` wird die Ablaufanimation deaktiviert

## Varianten

| Variante | Zweck | Standarddauer | Farbe | Icon |
|---|---:|---:|---|---|
| `success` | Aktion wurde abgeschlossen | 7 Sekunden | `#16a34a` | `CheckCircle2` |
| `info` | neutrale Systeminformation | 7 Sekunden | `#3b82f6` | `Info` |
| `warning` | handlungsrelevanter Hinweis | 7 Sekunden | `#f59e0b` | `TriangleAlert` |
| `error` | Aktion ist fehlgeschlagen | 7 Sekunden | `#ff2d58` | `XCircle` |
| `loading` | laufende Aktion | dauerhaft | `#3b82f6` | `Loader2` |

Die Dauer kann pro Toast überschrieben werden, wenn die Meldung länger ist oder besondere Aufmerksamkeit braucht.

## Textregeln

- Erfolgsmeldungen kurz und bestätigend formulieren.
- Fehlermeldungen lösungsorientiert formulieren.
- Du-Form verwenden, wenn der Nutzer direkt angesprochen wird.
- Beschreibung nur ergänzen, wenn sie zusätzliche Handlungssicherheit gibt.
- Keine technischen Rohfehlermeldungen anzeigen, wenn sie für Nutzer nicht verständlich sind.
- Direkte Umlaute verwenden.

Beispiele:

```ts
notify.success("Projekt wurde gespeichert.");
notify.error("Speichern fehlgeschlagen.", {
  description: "Bitte versuche es erneut."
});
notify.warning("Lizenz endet bald.");
notify.info("Änderungen werden verarbeitet.");
```

## Ausnahmen

Der Support-Countdown-Toast in `apps/frontend/src/pages/admin/SupportTicketDetail.tsx` bleibt eine bewusste Ausnahme. Er ist ein fachlicher Workflow mit eigener Interaktion, eigenem Fortschrittsbalken und Abbrechen-Aktion.
