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

## Tests

`apps/api/tests/test_user_privacy.py` – neue Fälle: Übertragung an verbleibenden
Owner, Solo-Team-Löschung, Legacy-Projekt ohne `organization_id`, `updated_by`-Null.
Alle 16 Tests grün.
