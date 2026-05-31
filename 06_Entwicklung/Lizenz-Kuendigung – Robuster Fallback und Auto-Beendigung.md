# Lizenz-Kündigung – Robuster Fallback und Auto-Beendigung

Beschreibt zwei Robustheitsmechanismen im Kündigungs-Workflow der Lizenzverwaltung, die sicherstellen, dass Kündigungen auch dann korrekt ablaufen, wenn Stripe-Webhooks verzögert eintreffen oder ausbleiben (z. B. Stripe-Störung, falsch konfigurierter Endpoint, lokale Dev-Umgebung ohne Webhook-Forwarding).

Eingeführt am 2026-05-31.

## Hintergrund / Problem

Im Pool-Modell werden zwei Dinge normalerweise über Stripe-Webhooks (`customer.subscription.updated` / `.deleted`) gesteuert:

1. **Term-Verlängerung:** Bei jeder Abrechnung rollt der Webhook `current_term_end` weiter (z. B. 26.05. → 26.06.).
2. **Beendigung:** Eine gekündigte Lizenz wechselt von `scheduled_end` auf `ended`, sobald die Abrechnungsperiode über `scheduled_end_at` hinausläuft.

Bleibt der Webhook aus, entstehen zwei Fehlbilder:

- **Veralteter Term:** `current_term_end` bleibt eingefroren. Eine Kündigung fiel dann auf `scheduled_end = now` zurück (Fallback im alten Code) — der Kunde verlor **sofort** den Zugriff auf eine bereits bezahlte Periode, statt zum nächsten Periodenende.
- **Hängende Lizenz:** Eine `scheduled_end`-Lizenz, deren Enddatum erreicht ist, blieb für immer in diesem Status und belegte weiter einen Pool-Slot.

In der Dev-Umgebung treten beide Effekte garantiert auf, da keine echten Stripe-Webhooks auf `localhost` ankommen.

## Lösung 1 – Robuster Kündigungs-Fallback

Datei: `apps/api/app/routers/licenses_v2.py`, Funktion `cancel_license`.

Das geplante Enddatum (`scheduled_end_at`) wird so bestimmt:

- **Trial:** endet sofort (`now`) — es gibt keine bezahlte Periode.
- **Es existiert ein Laufzeitende in der Zukunft** (`committed_until` / `current_term_end`): das späteste davon wird verwendet (wie bisher).
- **Kein Zukunfts-Termin, aber ein bekanntes (veraltetes) Laufzeitende:** der Helfer `_next_period_boundary_after()` rollt das letzte bekannte Ende in **echten Abrechnungsschritten** (monatlich/jährlich) vor, bis es in der Zukunft liegt.
- **Gar keine Termin-Info:** `now + eine Abrechnungsperiode`.

Neue Helfer:

- `_add_billing_period(start, pool)` — addiert exakt eine Periode, mit korrekter Monatsende-Behandlung (31.01. → 28.02., Schaltjahr 29.02. → 28.02.).
- `_next_period_boundary_after(base_end, pool, now)` — rollt vor bis zur nächsten zukünftigen Periodengrenze (Hard-Cap gegen Endlosschleifen).

> Effekt: Eine Kündigung beendet den Zugriff nie sofort, sondern immer zu einer echten Periodengrenze. Beispiel: Term-Ende 26.05., Kündigung am 31.05. → `scheduled_end_at = 26.06.`

## Lösung 2 – Zeitbasierter Auto-Beendigungs-Job

Datei: `apps/api/app/services/scheduler.py`.

Neuer **stündlicher** Job `expire_scheduled_licenses` (registriert in `start_scheduler`, `IntervalTrigger(minutes=60)`):

- Findet alle Lizenzen mit `status == "scheduled_end"` und `scheduled_end_at <= now`.
- Setzt sie auf `ended`, schreibt ein `LicenseEvent` (`reason: "scheduled_end_passed_fallback"`).
- Führt das Add-on-Rebasing aus (ältestes aktives Add-on wird zur Base befördert, falls die Base wegfällt) — identisch zur Webhook-Logik.

Die Kernlogik liegt in `_expire_due_licenses(db)` (synchron, idempotent, transaktions-neutral — Commit übernimmt der async-Wrapper). Das spiegelt das bestehende Webhook-Handler-Pattern und ist dadurch direkt testbar.

> Sicherheitsnetz: Selbst ohne jeden Stripe-Webhook werden gekündigte Lizenzen spätestens innerhalb einer Stunde nach Ablauf sauber beendet.

## Zusammenspiel mit „Kündigung zurückziehen"

Der Button „Kündigung zurückziehen" (Frontend `Licenses.tsx`, `canReactivateCancel`) erscheint nur, solange `scheduled_end_at` **in der Zukunft** liegt; das Backend (`reactivate_cancelled_license`) lehnt ein abgelaufenes Enddatum ab. Durch Lösung 1 liegt `scheduled_end_at` nach einer Kündigung verlässlich in der Zukunft, wodurch das Zurückziehen im vorgesehenen Zeitfenster funktioniert.

## Tests

- `apps/api/tests/test_license_cancel_reactivation.py`
  - `test_cancel_with_stale_term_schedules_to_next_period_boundary`
  - `test_cancel_without_term_dates_falls_back_to_one_period`
- `apps/api/tests/test_license_expiry_job.py` (neu)
  - beendet fällige Lizenzen, überspringt zukünftige, idempotent, befördert Add-on bei Base-Ende.

## Verwandte Dokumente

- [[Stripe Testmode Dry-Run - Trial und Erstbestellungsrabatt]]
- [[Verwaltungsportal-Billing-und-Abo-Support]]
