# T020-02 · Billing-Portal in neuem Tab öffnen

**Status:** erledigt  
**Phase:** 1 (Quick Win)  
**Priorität:** P0 · niedriges Risiko  
**Parent:** [[T020-allgemeine Todos]]  
**Abgeschlossen:** 2026-05-01

## Ergebnis

In `apps/frontend/src/pages/Licenses.tsx` (Zeile 1255, Funktion `handleManageBilling`) wurde `window.location.href = res.url` durch `window.open(res.url, '_blank', 'noopener,noreferrer')` ersetzt.

Keine weiteren `<a>`-Tags oder Buttons gefunden, die direkt auf die Portal-URL verlinken — die einzige Einstiegsstelle war `handleManageBilling`.

## Akzeptanzkriterien

- [x] Klick auf "Billing-Portal öffnen" öffnet neuen Tab.
- [x] Normdex-App bleibt im Original-Tab geöffnet.
- [x] Kein Reload, kein History-Eintrag der Portal-URL.
