# T044 · Letzter Login in Mitgliederliste des Verwaltungsportals

**Phase:** Admin / Kundenakte / Mitglieder  
**Priorität:** P3 · UX-Verbesserung / Supporteffizienz  
**Status:** erledigt  
**Datum:** 2026-07-07  
**Abgeschlossen:** 2026-07-09

## Ziel

In der Mitgliederliste der Kundenakte (Verwaltungsportal → Kundenakte → Tab „Mitglieder") soll pro Eintrag der Zeitpunkt des letzten Logins des jeweiligen Benutzers angezeigt werden. Damit kann der Admin-Nutzer auf einen Blick erkennen, wann ein Mitglied zuletzt aktiv war — ohne in separate Logs wechseln zu müssen.

## Kontext

Ursprüngliche Annahme war falsch: Die Kundenakte-Seite (`OrganizationCase.tsx`) lädt ihre Daten nicht über `GET /teams/{org_id}/members` (dort liefert `MemberSchema.last_login` bereits alles), sondern über `GET /admin/organizations/{org_id}/case`. Dieser Endpunkt (`get_organization_case` in `apps/api/app/routers/admin.py`) hatte kein `last_login` im Member-Objekt. Es musste daher doch ein Backend-Feld ergänzt werden — die dazu passende Akzeptanzkriterium-Annahme „Kein Backend-Endpunkt muss geändert werden" hat sich nicht bestätigt.

## Umsetzung

- Backend: `apps/api/app/routers/admin.py`, `get_organization_case` — analog zur bestehenden Logik in `teams.py` wird nun eine gebündelte `AuditLog`-Abfrage (`event == "login"`, neuestes Datum je `user_id`) durchgeführt und das Ergebnis als `last_login` in jedes Member-Objekt der `case`-Response geschrieben.
- Frontend: `apps/frontend/src/pages/admin/OrganizationCase.tsx`, Mitgliederzeile im `members`-Tab — zeigt `member.last_login` formatiert (`formatDateTime`) unterhalb der E-Mail-Adresse, mit `Clock3`-Icon (Lucide, bereits importiert) und Fallback „Noch nie eingeloggt".
- Commit: `44a75d2` in `normdex-webapp-dev` (Branch `dev-server`), gepusht.

## Akzeptanzkriterien

- [x] Jede Mitgliederzeile zeigt den letzten Login-Zeitpunkt formatiert an.
- [x] Mitglieder ohne Login-Eintrag zeigen „Noch nie eingeloggt".
- [x] Die Darstellung ist konsistent mit dem bestehenden visuellen Stil der Mitgliederzeile.
- [x] ~~Kein Backend-Endpunkt muss geändert werden~~ — musste doch geändert werden, siehe Kontext.
- [x] Bestehende Badges (Aktiv, Verifiziert, Admin, Rolle) bleiben unverändert.

## Hinweis zur ID

Ursprünglich fälschlich als `T038` angelegt (2026-07-08), was mit dem bereits existierenden, abgeschlossenen `T038-support-ticket-direkt-schliessen-ohne-bounce-loop` kollidierte. Bei Abschluss auf `T044` (nächste freie ID) korrigiert, um die Vault-Konvention „Nummern werden nie wiederverwendet" wiederherzustellen.
