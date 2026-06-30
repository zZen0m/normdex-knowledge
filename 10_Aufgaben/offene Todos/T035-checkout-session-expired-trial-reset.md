# T035 · Checkout-Abbruch: Trial-Benefit automatisch zurücksetzen

**Phase:** App / Lizenzen / Stripe / Checkout  
**Priorität:** P1 · Produktionsbug / UX-kritisch  
**Status:** offen  
**Datum:** 2026-06-30

## Ziel

Bricht ein Nutzer den Stripe-Checkout ab oder lässt ihn ablaufen, muss der `trial_used_at`-Flag auf der Organisation automatisch zurückgesetzt und die zugehörigen Pending-Datensätze bereinigt werden. Der Nutzer soll danach nahtlos einen neuen Checkout starten und seinen Trial regulär in Anspruch nehmen können.

## Problembeschreibung

### Was passiert heute

Beim Start des Checkout-Prozesses (`POST /licenses/checkout/create`) setzt die App **sofort** den `trial_used_at`-Timestamp auf der Organisation und legt Pending-Datensätze an:

```python
# licenses_v2.py:1450-1453
# Permanently lock trial benefit on org as soon as pending records exist.
# This prevents race conditions (concurrent sessions) and survives license deletion.
if trial_benefit.kind:
    org.trial_used_at = now_utc_naive()
```

Der Checkout wird dann in Stripe abgeschlossen. Erst wenn Stripe das Event `checkout.session.completed` schickt, wird die Lizenz von `pending` auf `trial` gesetzt und der Trial aktiv.

**Das Problem:** Der Webhook-Handler (`subscriptions.py`) behandelt `checkout.session.expired` nicht. Läuft die Checkout-Session ab (Stripe-Standard: 24 Stunden) oder bricht der Nutzer den Checkout manuell ab, passiert serverseitig nichts:

- `trial_used_at` bleibt permanent gesetzt
- Pending-Lizenz und Pending-Order bleiben in der Datenbank
- Der Nutzer bekommt im normalen Checkout-Flow keinen neuen Trial mehr (`_has_trial_benefit_history()` gibt `True` zurück)
- In der App erscheint eine ausstehende Lizenz ohne aktiven Subscription-Stand

### Konsequenz für den Nutzer

Der Nutzer landet in einer Sackgasse: Der Trial gilt als verbraucht, obwohl er nie eine Lizenz aktiviert hat. Ohne manuellen Admin-Eingriff (wie am 2026-06-30 für Russ Ingenieure GmbH nötig) kann er nie mehr einen Trial starten.

### Betroffene Dateien

- `apps/api/app/routers/subscriptions.py` — Webhook-Handler (fehlendes Event)
- `apps/api/app/routers/licenses_v2.py` — Checkout-Create, `trial_used_at`-Setzung
- Stripe-Dashboard — Webhook-Konfiguration (fehlendes Event registriert)

## Ursachenanalyse

Die Entscheidung, `trial_used_at` beim Checkout-Start zu setzen, ist prinzipiell korrekt: Sie verhindert Race-Conditions (mehrere Tabs, parallele Sessions). Der Fehler liegt darin, dass kein Kompensationsmechanismus für den Abbruchfall existiert. Stripe liefert mit `checkout.session.expired` das nötige Signal — es wird nur nicht verarbeitet.

## Lösungsweg

### Schritt 1 — Stripe-Webhook-Event registrieren

Im Stripe-Dashboard unter **Developers → Webhooks → Endpoint bearbeiten** das Event `checkout.session.expired` für den Produktions-Webhook-Endpunkt (`https://app.normdex.at/subscriptions/webhook`) hinzufügen.

### Schritt 2 — Handler in `subscriptions.py` ergänzen

Im Webhook-Router den neuen Event-Typ einbinden:

```python
# subscriptions.py, im Webhook-Dispatcher (ca. Zeile 445ff.)
elif event_type == "checkout.session.expired":
    _handle_checkout_session_expired(obj, db)
```

### Schritt 3 — Handler-Funktion implementieren

```python
def _handle_checkout_session_expired(session, db: Session) -> None:
    """
    Wenn eine Checkout-Session abläuft ohne abgeschlossen zu werden:
    Pending-Lizenz und Pending-Order löschen, trial_used_at zurücksetzen,
    damit der Nutzer einen neuen Checkout starten kann.
    """
    session_id = session.get("id")
    if not session_id:
        return

    # Order über Checkout-Session-ID finden
    order = db.query(models.LicenseOrder).filter(
        models.LicenseOrder.stripe_checkout_session_id == session_id,
        models.LicenseOrder.status == "pending",
    ).first()

    if not order:
        return  # Bereits abgeschlossen oder unbekannt

    org = db.query(models.Organization).filter(
        models.Organization.id == order.organization_id
    ).first()

    if not org:
        return

    # Sicherheitscheck: Nur zurücksetzen wenn wirklich keine aktive Lizenz existiert
    active_license_exists = db.query(models.License).filter(
        models.License.organization_id == org.id,
        models.License.status.in_(("active", "trial", "scheduled_end")),
    ).count() > 0

    if active_license_exists:
        # Org hat bereits eine aktive Lizenz — nichts zurücksetzen
        _record_event(db, None, "checkout.expired_ignored_active_license", {
            "session_id": session_id,
            "order_id": order.id,
            "organization_id": org.id,
        })
        return

    # Pending-Lizenzen dieser Order löschen (cascadet license_events via ORM)
    pending_licenses = db.query(models.License).filter(
        models.License.organization_id == org.id,
        models.License.status == "pending",
    ).all()
    for lic in pending_licenses:
        db.delete(lic)

    # Order löschen (cascadet license_order_items via ORM)
    db.delete(order)

    # trial_used_at zurücksetzen — Nutzer kann neu starten
    org.trial_used_at = None

    db.flush()

    _record_event(db, None, "checkout.expired_trial_reset", {
        "session_id": session_id,
        "organization_id": org.id,
    })

    db.commit()
```

### Schritt 4 — Bestehende `_record_event`-Signatur prüfen

Falls `_record_event` zwingend eine `license_id` erwartet, einen separaten System-Log-Eintrag (`log_system_error` oder eigene Tabelle) verwenden oder die Funktion um einen optionalen Parameter erweitern.

### Schritt 5 — Tests ergänzen

- Checkout-Session abgelaufen, Org hatte noch keinen aktiven Trial → `trial_used_at` wird zurückgesetzt, Pending-Datensätze gelöscht
- Checkout-Session abgelaufen, aber Org hat bereits eine aktive Lizenz (aus einem anderen Pool) → nichts wird verändert
- Checkout-Session abgelaufen, `stripe_checkout_session_id` ist unbekannt → kein Fehler, kein Effekt
- Race-Condition: Zwei gleichzeitige Abläufe (Session-Expire + manueller Reset) → idempotentes Verhalten

### Schritt 6 — Manuelle Bereinigung bestehender Pending-Datensätze (optional)

Ggf. einmalige DB-Abfrage auf Organisationen prüfen, bei denen `trial_used_at` gesetzt ist, aber alle Lizenzen `pending` sind und der zugehörige Stripe-Checkout seit mehr als 24 Stunden abgelaufen sein müsste.

## Akzeptanzkriterien

- [ ] `checkout.session.expired` ist im Stripe-Webhook-Endpunkt registriert (Prod + Dev).
- [ ] Handler in `subscriptions.py` ist implementiert und eingebunden.
- [ ] Bricht ein Nutzer den Checkout ab und läuft die Session ab: `trial_used_at = NULL`, Pending-Lizenz und Pending-Order sind gelöscht.
- [ ] Hat die Org bereits eine aktive/trial-Lizenz: Kein Eingriff durch den Handler.
- [ ] Alle Tests grün, kein Regressionsrisiko für `checkout.session.completed`.
- [ ] Stripe-Event in Dev-Umgebung über Stripe CLI testbar: `stripe trigger checkout.session.expired`.

## Notizen / Fortschritt

- 2026-06-30: Bug in Produktion aufgetreten. Russ Ingenieure GmbH (Kundennummer 660686) hat Checkout gestartet und abgebrochen. `trial_used_at` war gesetzt, keine aktive Lizenz vorhanden. Manueller Reset durch Admin erforderlich (Backup `20260630_125432`). Todo angelegt.
