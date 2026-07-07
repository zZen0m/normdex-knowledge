# T038 · Letzter Login in Mitgliederliste des Verwaltungsportals

**Phase:** Admin / Kundenakte / Mitglieder  
**Priorität:** P3 · UX-Verbesserung / Supporteffizienz  
**Status:** offen  
**Datum:** 2026-07-07

## Ziel

In der Mitgliederliste der Kundenakte (Verwaltungsportal → Kundenakte → Tab „Mitglieder") soll pro Eintrag der Zeitpunkt des letzten Logins des jeweiligen Benutzers angezeigt werden. Damit kann der Admin-Nutzer auf einen Blick erkennen, wann ein Mitglied zuletzt aktiv war — ohne in separate Logs wechseln zu müssen.

## Kontext

Das Backend liefert `last_login` bereits vollständig: `MemberSchema` (apps/api/app/routers/teams.py, Zeile 34) enthält das Feld, und der Endpunkt `GET /teams/{org_id}/members` befüllt es über eine dedizierte Log-Abfrage (teams.py, Zeilen 258–279). Die Änderung beschränkt sich ausschließlich auf die **Frontend-Darstellung**.

## Umsetzung

- Datei: `apps/frontend/src/pages/admin/OrganizationCase.tsx`
- Im `members`-Tab (ca. Zeile 2043 ff.) wird jede Mitgliederzeile gerendert.
- `member.last_login` ist im API-Response bereits vorhanden, wird aber bisher nicht angezeigt.
- Das Datum soll lesbar formatiert werden (z. B. `dd.MM.yyyy, HH:mm` — analog zu anderen Zeitstempeln in der App).
- Ist kein Login vorhanden (`null`), soll ein Fallback-Text angezeigt werden, z. B. „Noch nie eingeloggt".
- Platzierung: unterhalb oder neben der E-Mail-Adresse, dezent als sekundärer Text (`text-muted-foreground text-xs`), mit einem kleinen Uhren- oder Kalender-Icon (Lucide, bereits im Projekt).

## Akzeptanzkriterien

- [ ] Jede Mitgliederzeile zeigt den letzten Login-Zeitpunkt formatiert an.
- [ ] Mitglieder ohne Login-Eintrag zeigen „Noch nie eingeloggt".
- [ ] Die Darstellung ist konsistent mit dem bestehenden visuellen Stil der Mitgliederzeile.
- [ ] Kein Backend-Endpunkt muss geändert werden.
- [ ] Bestehende Badges (Aktiv, Verifiziert, Admin, Rolle) bleiben unverändert.
