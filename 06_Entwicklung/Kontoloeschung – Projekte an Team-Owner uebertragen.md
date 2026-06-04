---
tags: [entwicklung, datenschutz, kontoloeschung, bugfix]
status: behoben
datum: 2026-06-04
schwere: kritisch
---

# Kontolöschung – Projekte an Team-Owner übertragen

## Befund (Audit 2026-06-04)

Beim **Self-Service-Löschen des eigenen Kontos** wurden **alle Projekte und
Berechnungen** des Users **hart gelöscht**. Da Projekte **organisationsweit
geteilt** sind (Sichtbarkeit über `organization_id`, siehe
`apps/api/app/project_visibility.py`), riss ein ausscheidendes Team-Mitglied
gemeinsam genutzte Projektdaten mit – die übrigen Team-Mitglieder verloren ihre
Daten. **Kritischer Datenverlust.**

- Betroffener Pfad: `apps/api/app/routers/users.py` → `_anonymize_user_account`.
- Der **Admin-Löschpfad** (`apps/api/app/routers/admin.py` → `delete_user`) war
  nicht betroffen – er blockiert die Löschung (HTTP 409), wenn der User Projekte
  besitzt.

## Behebung

Statt zu löschen werden Projekte (und deren Berechnungen) eines ausscheidenden
Mitglieds an einen **verbleibenden Owner/Admin der jeweiligen Organisation
übertragen** (neue Hilfsfunktionen `_resolve_org_recipient` und
`_reassign_or_delete_owned_data` in `users.py`).

- Empfänger: bevorzugt Owner/Admin (`TEAM_ADMIN_ROLES`), Fallback anderes Mitglied.
- **Sonderfall Solo-Team:** Ist der User das einzige verbliebene Mitglied seiner
  Organisation, existiert kein Empfänger → Projekte + Berechnungen werden gelöscht
  (DSGVO-konform, niemand sonst hatte Zugriff). Durch die Block-Logik (letzter Owner
  mit weiteren Mitgliedern) abgesichert.
- `Project.updated_by`-Referenzen des Users werden wie bisher auf `NULL` gesetzt.
- Keine DB-Migration nötig: Der User-Datensatz wird anonymisiert (nicht gelöscht),
  der `ondelete="CASCADE"` auf `projects.user_id` feuert in diesem Flow nicht.

## Vereinheitlichung beider Lösch-Pfade (Audit-Finding 2)

Im Anschluss wurde die gesamte Lösch-/Anonymisierungslogik in den gemeinsamen
Service `apps/api/app/services/account_deletion.py` (`anonymize_user_account` +
Helfer) extrahiert. **Beide** Pfade nutzen jetzt dieselbe, einzige Strategie:

- **Self-Service** (`users.py` → `delete_confirm`) ruft `anonymize_user_account`.
- **Admin** (`admin.py` → `delete_user`) ruft denselben Service und **anonymisiert
  ebenfalls** statt hart zu löschen. Der bisherige HTTP-409-Eigentums-Blocker
  (Projekte/Bestellungen) entfällt, weil org-gebundene Projekte übertragen werden
  und der User-Record anonymisiert erhalten bleibt — keine NOT-NULL-Referenz kann
  mehr verwaisen.

Konsequenz der Anonymisierung (statt Hard-Delete) im Admin-Pfad: `AuditLog` und
`Notification` des Users bleiben am anonymisierten Datensatz erhalten (vorher hart
gelöscht); der DB-`ondelete="CASCADE"` auf `projects.user_id` feuert in keinem Pfad.

## Tests

- `apps/api/tests/test_user_privacy.py` – Übertragung an verbleibenden Owner,
  Solo-Team-Löschung, Legacy-Projekt ohne `organization_id`, `updated_by`-Null.
- `apps/api/tests/test_admin_delete_user.py` – auf Anonymisierungs-Semantik
  umgestellt (User anonymisiert statt gelöscht, Projekt-Übertragung, Billing-Snapshot).
- `apps/api/tests/test_avatar_delete.py` – Admin-Avatar-Löschung an Anonymisierung
  angepasst.
- Gesamte Backend-Suite **297 grün**.
