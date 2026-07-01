# T036 · Dashboard – Erste Schritte Popup merkt sich Zustand

**Phase:** App / Frontend / UX  
**Priorität:** P3 · UX-Verbesserung  
**Status:** offen  
**Datum:** 2026-07-01

## Ziel

Das „Erste Schritte"-Popup im Dashboard soll seinen ein- oder ausgeklappten Zustand im `localStorage` des Browsers speichern. Beim nächsten Seitenaufruf wird dieser gespeicherte Zustand wiederhergestellt, statt das Popup immer standardmäßig ausgeklappt anzuzeigen.

## Kontext

Aktuell öffnet sich das Erste-Schritte-Panel bei jedem Seitenaufruf automatisch ausgeklappt. Nutzer, die das Panel bewusst eingeklappt haben, müssen es bei jedem Besuch erneut schließen. Das ist störend, sobald die Onboarding-Schritte bereits bekannt sind.

## Akzeptanzkriterien

- [ ] Nutzer klappt das Popup ein → Zustand wird in `localStorage` gespeichert
- [ ] Nutzer klappt das Popup aus → Zustand wird in `localStorage` gespeichert
- [ ] Beim nächsten Seitenaufruf: Panel startet im zuletzt gespeicherten Zustand
- [ ] Kein gespeicherter Zustand vorhanden (Erstbesuch / localStorage leer) → Panel startet ausgeklappt (bisheriges Standardverhalten)
- [ ] Zustand ist nutzer- bzw. browserspezifisch (kein serverseitiges Speichern notwendig)
