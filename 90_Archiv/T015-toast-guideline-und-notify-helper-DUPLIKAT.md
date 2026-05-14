# T015 – Toast-Guideline und zentraler Notify-Helper

## Status

offen

## Bereich

App / Designsystem / Frontend

## Ziel

Toast-Benachrichtigungen in der Normdex App sollen einheitlich aussehen und sich konsistent verhalten. Dafür soll eine dokumentierte Toast-Guideline im Vault entstehen und anschließend ein zentraler Frontend-Helper eingeführt werden, über den Standard-Toasts ausgelöst werden.

## Kontext

Aktuell nutzt die App Sonner für Toast-Benachrichtigungen. Es gibt bereits eine zentrale `Toaster`-Konfiguration in:

- `apps/frontend/src/components/ui/sonner.tsx`

Viele Stellen rufen jedoch direkt `toast.success(...)`, `toast.error(...)`, `toast.custom(...)` oder Varianten davon auf. Dadurch können Farbe, Format, Dauer, Textstruktur und Verhalten uneinheitlich wirken.

Im Vault existieren bereits allgemeine Vorgaben:

- `01_Produkt/Brand Identity & Voice.md`
  - sachlich, klar, hilfreich
  - Fehlermeldungen lösungsorientiert
  - Erfolgsmeldungen kurz und bestätigend
  - direkte Umlaute verwenden
- `01_Produkt/Designsystem & Farben.md`
  - Erfolg: `#16a34a`
  - Warnung: `#f59e0b`
  - Fehler: `#ff2d58`
  - Info: `#3b82f6`
- `02_App/App-Architektur & Tech Stack.md`
  - Sonner ist als Toast-System festgelegt

Was fehlt, ist eine konkrete Toast-Spezifikation für Layout, Dauer, Progress-Bar, Icons, Varianten und Einsatzregeln.

## Wichtige Ausnahme

Der bestehende Custom-Toast in der Support-Ticket-Detailansicht soll nicht umgebaut werden.

Ausgenommen bleiben:

- `apps/frontend/src/pages/admin/SupportTicketDetail.tsx`
- insbesondere der `ResolutionCountdown`-Toast für `resolved_pending`

Grund: Dieser Toast ist fachlich ein spezieller Countdown-Workflow mit eigenem Fortschrittsbalken und Abbrechen-Aktion. Er ist bewusst anders gestaltet und soll unverändert bleiben, sofern nicht später ausdrücklich anders entschieden.

## Gewünschte Toast-Guideline

Eine neue oder passende bestehende Vault-Dokumentation soll festlegen:

1. Position
   - Standardposition: rechts unten
   - Verhalten auf Mobile

2. Dauer
   - Standarddauer für Erfolg, Info, Warnung und Fehler
   - längere Dauer für wichtige/mehrzeilige Meldungen
   - wann ein Toast manuell schließbar sein soll

3. Varianten
   - `success`
   - `error`
   - `warning`
   - `info`
   - optional `loading`

4. Farben
   - Verwendung der bestehenden semantischen Farben aus `Designsystem & Farben`
   - klare Unterscheidung, ohne zu starke visuelle Brüche
   - Dark-Mode-Kompatibilität prüfen

5. Textstruktur
   - kurze Titel oder Einzeiler für Standardfälle
   - optionale Beschreibung nur bei zusätzlichem Nutzen
   - keine technischen Rohfehlermeldungen, wenn sie nicht nutzerverständlich sind
   - konsistente Du-Form

6. Icons
   - einheitliche Lucide-Icons je Status
   - keine wechselnden Icon-Stile je Seite

7. Ablaufbalken
   - Standard-Toasts sollen einen horizontalen Ablaufbalken haben
   - Balken läuft über die Toast-Dauer ab
   - bei `prefers-reduced-motion` muss die Animation reduziert oder deaktiviert werden
   - bei `duration: Infinity` kein Ablaufbalken

8. Einsatzregeln
   - Erfolg: Aktion wurde abgeschlossen
   - Fehler: Aktion ist fehlgeschlagen und Nutzer braucht klare nächste Information
   - Warnung: Aktion möglich, aber Zustand ist kritisch oder handlungsrelevant
   - Info: neutrale Systeminformation

## Technische Umsetzung

Ein zentraler Helper soll eingeführt werden, z. B.:

- `apps/frontend/src/lib/notify.tsx`

Mögliche API:

```ts
notify.success("Projekt wurde gespeichert.");
notify.error("Speichern fehlgeschlagen.", {
  description: "Bitte versuche es erneut."
});
notify.warning("Lizenz endet bald.");
notify.info("Änderungen werden verarbeitet.");
```

Der Helper soll intern Sonner verwenden, aber die Darstellung zentral kontrollieren:

- einheitliches Layout
- Status-Icon
- Statusfarbe
- Dauer
- optionaler Beschreibungstext
- Ablaufbalken
- einheitliche `toast.dismiss`-Integration

Für Standardfälle soll `toast.custom(...)` oder eine gleichwertige Sonner-Integration genutzt werden, weil Sonner in der aktuell verwendeten Version keinen fertigen Progress-Bar-Parameter bereitstellt.

## Migrationsumfang

Nach Einführung des Helpers sollen direkte Standard-Toast-Aufrufe schrittweise ersetzt werden:

- `toast.success(...)`
- `toast.error(...)`
- `toast.warning(...)`
- `toast.info(...)`

Nicht automatisch ersetzen:

- fachliche Custom-Toasts mit eigener Interaktion
- `SupportTicketDetail.tsx` / `ResolutionCountdown`
- Spezialfälle, die bewusst `duration: Infinity` oder eigene JSX-Struktur brauchen

## Akzeptanzkriterien

- Es gibt eine dokumentierte Toast-Guideline im Vault.
- Standard-Toasts verwenden einen zentralen Helper statt verstreuter Direktaufrufe.
- Erfolg, Fehler, Warnung und Info sehen konsistent aus.
- Standard-Toasts haben einen Ablaufbalken, sofern sie automatisch verschwinden.
- Die Toast-Dauer ist zentral definiert und überschreibbar.
- Die Support-Ticket-Detail-Custom-Toasts bleiben unverändert.
- `npm run build` läuft erfolgreich.
- Bestehende relevante UI-Flows zeigen keine Layout-Regressionen.

## Offene Entscheidungen

- Exakte Standarddauer je Variante, z. B. 4s Erfolg/Info, 6s Warnung/Fehler.
- Ob Fehler-Toasts immer einen Close-Button bekommen sollen.
- Ob Toasts bei Hover pausieren sollen oder nur Sonner-Standardverhalten genutzt wird.
- Ob die Guideline in `01_Produkt/Designsystem & Farben.md` ergänzt oder als eigenes Dokument unter `02_App/` angelegt wird.

## Notizen / Fortschritt

- Angelegt am 2026-04-27 aus Feedback zur uneinheitlichen Toast-Darstellung.
- Bestehender Support-Countdown-Toast wurde ausdrücklich als gewollte Ausnahme markiert.
