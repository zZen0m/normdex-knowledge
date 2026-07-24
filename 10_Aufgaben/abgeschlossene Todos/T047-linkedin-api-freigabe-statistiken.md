# T047 · LinkedIn API-Freigabe für Unternehmensseiten-Statistiken beantragen

**Phase:** Marketing / LinkedIn-Markenkanal / Reporting
**Priorität:** P3 · Nice-to-have für Reporting, kein Blocker für laufenden Betrieb
**Status:** erledigt (Statistik-Abruf funktioniert; Weiterverarbeitung/Reporting-Darstellung ist eigenes Folge-Thema)
**Datum:** 2026-07-12

## Ziel

Offiziellen API-Zugriff auf die Statistiken der LinkedIn-Unternehmensseite (Follower-Entwicklung, Post-Performance, Impressions, Engagement) erhalten, damit diese Daten automatisiert ausgelesen und ausgewertet werden können (z. B. über n8n oder direkt für Reporting), statt sie nur manuell im LinkedIn-Interface einzusehen.

## Hintergrund

Ausgangsfrage war, ob es einen MCP-Server oder eine Automatisierung (n8n, analog zu Make) gibt, um auf die LinkedIn-Unternehmensseite zuzugreifen und Statistiken auszulesen. Ergebnis der Recherche:

- Es gibt keinen offiziellen MCP-Server von Anthropic für LinkedIn. Community-Projekte sind entweder Scraper (ToS-Risiko, Account-Sperre möglich) oder setzen selbst offiziellen API-Zugang voraus.
- **Posten** auf die Unternehmensseite ist unkompliziert über n8n möglich (nativer LinkedIn-Node, OAuth2, einfache API-Freigabe).
- **Statistiken auslesen** (Follower, Post-Performance, Impressions) erfordert dagegen Zugriff auf die **Community Management API** (organische Page-Insights) bzw. **Marketing Developer Platform / Ads API** (Kampagnendaten) — das ist eine LinkedIn-seitige Partner-Freigabe, kein Tool-Problem. n8n oder Make können die Daten danach gleichwertig über HTTP-Request-Node + OAuth2-Token abfragen.
- Bewusst keine provisorische Zwischenlösung (z. B. manueller CSV-Export über n8n) gewählt — der Nutzer will direkt die offizielle Freigabe beantragen.

Kontext: [[Marketingplan 2026 - Erste Kunden]], Phase 2 „Sichtbarkeit" — LinkedIn-Unternehmensseite ist seit 04.07.2026 angelegt (https://www.linkedin.com/company/normdex), zwei Posts/Monat geplant.

## Beantragungsprozess (Stand der Recherche 2026-07-12)

1. **LinkedIn Developer Portal** (`developer.linkedin.com`) — Anmeldung mit dem LinkedIn-Account, der **Admin-Rechte auf der Normdex-Unternehmensseite** hat. Ohne Admin-Rolle auf der Page ist kein Antrag möglich.
2. **App anlegen** im Developer Portal und mit der Unternehmensseite verknüpfen (Voraussetzung für jeden weiteren Produkt-Antrag).
3. **Produkt beantragen**:
   - **Community Management API** — für organische Page-Statistiken (Follower, Post-Reichweite, Engagement), ohne Ads-Bezug. Vermutlich das relevante Produkt für dieses Todo.
   - **Marketing Developer Platform / Ads API** — nur nötig, falls später auch Kampagnen-/Ads-Daten ausgelesen werden sollen (aktuell kein aktives Ads-Budget bekannt, daher nicht Kern dieses Todos).
4. **Review durch LinkedIn**: manuelle Prüfung, Use-Case-Beschreibung erforderlich. Erfahrungsgemäß Tage bis Wochen Bearbeitungszeit. Für manche Produkte (v. a. Marketing/Ads API) wird teils ein Mindest-Track-Record bei Ads-Ausgaben vorausgesetzt — Voraussetzungen ändern sich gelegentlich, daher vor dem Antrag im Portal die aktuell gültigen Bedingungen für das jeweilige Produkt prüfen.

## Offene Fragen

- Wie lange dauert die Freigabe in der Praxis (bisher keine eigene Erfahrung damit)?
- Nach Freigabe: Soll der Abruf über n8n (HTTP-Request-Node + OAuth2) oder direkt aus der Normdex-App erfolgen?

## Nächste Schritte

- [x] Bei developer.linkedin.com als Page-Admin anmelden und App anlegen (erledigt 2026-07-12)
- [x] App mit LinkedIn-Unternehmensseite „normdex" verknüpft
- [x] Community Management API beantragen — **genehmigt** (neue App, siehe Notizen 2026-07-19)
- [x] Alte App A (Sign In with LinkedIn / Events / persönliches Profil) gelöscht (2026-07-19, vom Nutzer im Portal entfernt) — kein Dual-App-Setup nötig
- [x] OAuth-Consent-Flow für App B abgeschlossen, Access-/Refresh-Token gesichert (2026-07-19)
- [x] Organisations-Zugriff verifiziert (`organizationAcls`-Endpunkt liefert ADMINISTRATOR-Rolle zurück)
- [x] Statistik-Endpunkte angebunden und verifiziert (2026-07-19): Follower-Statistik und aggregierte Post-Statistik liefern echte Daten
- [x] Einzelpost-Statistiken ergänzt (2026-07-19) — siehe Notizen
- [x] Regelmäßiger Abruf + Ablage für Follower-Verlaufshistorie eingerichtet (2026-07-19) — siehe Notizen
- [ ] Folge-Thema (separates Todo bei Bedarf): Anzeige der Statistiken im Normdex-Verwaltungsportal (Marketing-Reiter/KPIs) — noch nicht entschieden, siehe Notizen

## Notizen / Fortschritt

**2026-07-12 — App erstellt, OAuth-Verbindung für Basis-Scopes erfolgreich getestet**

- Entscheidung: Integration läuft **nicht über n8n**, sondern direkt lokal in der Claude-Code-Umgebung auf dem VPS, damit Claude selbst Statistiken auslesen und Fragen dazu beantworten kann. n8n war nur als Ausweichoption gedacht, falls der Developer-Weg nicht klappt.
- App im LinkedIn Developer Portal angelegt, mit der Unternehmensseite verknüpft. Freigegebene Produkte bislang: **Sign In with LinkedIn using OpenID Connect**, **Share on LinkedIn**, ein **Events**-Produkt. **Community Management API steht dort aktuell nicht als beantragbar zur Verfügung** — der eigentliche Kern dieses Todos (Statistik-Zugriff) ist damit weiterhin blockiert.
- Freigegebene Scopes: `r_verify`, `openid`, `profile`, `r_events`, `w_member_social`, `email`, `r_profile_basicinfo`, `rw_events`. Keine davon deckt Organisations-Statistiken ab (dafür nötig: `r_organization_social`, `rw_organization_admin`).
- 3-legged-OAuth2-Flow einmalig manuell durchgeführt (Consent im Browser, Code aus Redirect-URL kopiert) und serverseitig gegen Access-/Refresh-Token getauscht.
- Verbindung erfolgreich getestet: `GET https://api.linkedin.com/v2/userinfo` liefert Profildaten (Andreas Gruber) zurück — technischer OAuth-Unterbau funktioniert.
- Zugangsdaten (Client ID/Secret, Access-/Refresh-Token) liegen lokal auf dem VPS unter `/home/deploy/tools/linkedin-stats/credentials.json` (Dateirechte 600, nicht in einem Git-Repo, nicht versioniert). Hilfsskripte dort: `get_auth_url.py`, `exchange_token.py`.
- Access-Token gültig bis 10.09.2026, Refresh-Token bis 2027 — Statistik-Scopes können bei Bedarf per erneutem Consent-Flow nachgezogen werden, sobald Community Management API im Portal beantragbar ist.
- **Damit nutzbar:** eigenes Profil abrufen, Organisation-Events lesen/verwalten, Posts im eigenen Namen erstellen. **Weiterhin nicht möglich:** Follower-Statistiken, Post-Performance/Impressions der Unternehmensseite.

**2026-07-19 — Zweite App angelegt, Community Management API genehmigt; Fokus klargestellt: nur Unternehmensseite**

- Grund für die zweite App: LinkedIn lässt pro App offenbar nur eine der beiden Produktlinien zu — entweder Events-Produkt (wie bei der ersten App beantragt) oder Community Management API, nicht beides in derselben App. Daher neue App angelegt und dort gezielt nur die Community Management API beantragt — **wurde genehmigt**.
- Damit existieren jetzt zwei LinkedIn-Apps mit getrennten Client-ID/Secret-Paaren:
  - **App A** (alt, 2026-07-12): Sign In with LinkedIn (OpenID Connect), Share on LinkedIn, Events-Produkt. Scopes für persönliches Profil, persönliches Posten, Events.
  - **App B** (neu, 2026-07-19): Community Management API — Scopes für Organisations-Statistiken (`r_organization_social`, `rw_organization_admin`) und vermutlich auch Posten im Namen der Unternehmensseite (`w_organization_social`).
- Klarstellung vom Nutzer: **Nur die Unternehmensseite soll verwaltet werden, persönliches Profil ist explizit nicht im Fokus.** Damit ist offen, ob App A (Events, persönliches Profil) überhaupt noch gebraucht wird — aktuell kein bekannter Use-Case dafür. App B allein deckt das eigentliche Ziel dieses Todos ab.
- Damit ist die frühere offene Frage „Reicht die Community Management API allein?" geklärt: ja, für den festgelegten Scope (nur Unternehmensseite) reicht App B.
- Nächster Schritt: OAuth-Consent-Flow für App B durchführen (analog zum bisherigen Vorgehen), damit Access-/Refresh-Token für die Statistik-Scopes vorliegen.
- App A wurde vom Nutzer im Developer Portal gelöscht (2026-07-19) — es gibt nur noch eine App/einen Credential-Satz.
- Zugangsdaten für App B liegen lokal auf dem VPS unter `/home/deploy/tools/linkedin-orgpage/credentials.json` (Rechte 600, nicht in einem Git-Repo, nicht versioniert). Hilfsskripte dort: `get_auth_url.py`, `exchange_token.py` (gleiches Muster wie zuvor bei App A).
- Redirect-URI für App B: `http://localhost:8765/callback`.
- Angeforderte Scopes bewusst auf reine Organisations-Scopes beschränkt (User will nur die Unternehmensseite verwalten, kein persönliches Profil): `r_organization_followers`, `r_organization_social`, `rw_organization_admin`, `r_organization_social_feed`, `w_organization_social`, `w_organization_social_feed`. App B hätte zusätzlich auch persönliche Member-Scopes zur Verfügung (`r_member_postAnalytics`, `w_member_social`, `r_member_profileAnalytics`, `r_basicprofile`, `w_member_social_feed`, `r_1st_connections_size`) — diese wurden bewusst nicht angefordert.
- Auth-URL generiert und bereit für den Consent-Schritt im Browser.
- **Consent-Flow abgeschlossen:** Im normalen Chrome-Profil hing die LinkedIn-Auth-Seite dauerhaft (vermutlich Extension-/Cookie-Konflikt), im Inkognito-Fenster funktionierte es sofort. Code aus Redirect-URL gegen Access-/Refresh-Token getauscht (Access-Token gültig bis 17.09.2026, Refresh-Token bis 2027).
- Zugriff verifiziert: `GET https://api.linkedin.com/v2/organizationAcls?q=roleAssignee` liefert `ADMINISTRATOR`-Rolle für `urn:li:organization:135245244` zurück — Community Management API ist einsatzbereit.
- **Statistik-Endpunkte angebunden (2026-07-19):** Skript `/home/deploy/tools/linkedin-orgpage/fetch_stats.py` ruft zwei funktionierende Endpunkte ab (beide auf der Legacy-`/v2/`-API, nicht der neueren versionierten `/rest/`-API — letztere lieferte `426 NONEXISTENT_VERSION` und wurde nicht weiterverfolgt, da `/v2/` funktioniert):
  - `GET /v2/organizationalEntityFollowerStatistics?q=organizationalEntity&organizationalEntity={orgURN}` — Follower-Zahl (organisch/bezahlt), aufgeschlüsselt nach Branche/Seniorität/Firmengröße etc.
  - `GET /v2/organizationalEntityShareStatistics?q=organizationalEntity&organizationalEntity={orgURN}` — aggregierte Post-Statistik über alle Posts (Impressions, Klicks, Likes, Kommentare, Engagement-Rate). **Nur aggregiert, keine Einzelpost-Aufschlüsselung** — Versuche, einzelne Posts zu listen (`/v2/shares?q=owners`), scheiterten am Parameterformat und wurden nicht weiterverfolgt, da für den aktuellen Bedarf nicht nötig.
  - `fetch_stats.py` erneuert das Access-Token automatisch per Refresh-Token, wenn es in unter 5 Minuten abläuft.
  - Testlauf (2026-07-19, Page ist noch sehr jung): 1 organischer Follower, 31 Impressions/12 unique Impressions/3 Klicks/3 Likes über alle bisherigen Posts aggregiert.
- Für eine echte Follower-Verlaufshistorie (Zeitreihe) müsste der Abruf regelmäßig laufen und die Werte irgendwo abgelegt werden (z. B. Cronjob + einfache Datei/DB) — aktuell liefert der Endpunkt nur den Snapshot zum Abrufzeitpunkt, keine Historie rückwirkend.

**2026-07-19 — Einzelpost-Statistiken ergänzt**

- Zusätzlich zur aggregierten Post-Statistik jetzt auch Statistik pro einzelnem Post in `fetch_stats.py`.
- **Postliste:** `GET /rest/posts?q=author&author={orgURN}` — das ist die neuere versionierte REST-API (nicht `/v2/`), die einen `LinkedIn-Version`-Header im Format `YYYYMM` braucht. Ältere/zu alte Werte liefern `426 NONEXISTENT_VERSION`; aktuell funktioniert `202507` (als `REST_VERSION`-Konstante im Skript hinterlegt, muss künftig ggf. nachgezogen werden, wenn LinkedIn die Version abschaltet).
- Kurioser Fund: Das `total`-Feld im Paging-Objekt dieses Endpunkts war unzuverlässig (zeigte `total: 2`, obwohl nur 1 Post existiert und auch Folgeseiten denselben einen Post zurückgaben) — das Skript verlässt sich deshalb nicht auf `total`, sondern paginiert, bis eine Seite keine neuen Post-IDs mehr liefert.
- **Statistik pro Post:** `GET /v2/organizationalEntityShareStatistics?q=organizationalEntity&organizationalEntity={orgURN}&shares=List({postURN},...)` — Achtung bei der URL-Konstruktion: Die URNs innerhalb von `List(...)` müssen URL-kodiert werden (Doppelpunkte → `%3A`), die Klammern selbst dürfen **nicht** kodiert werden. Deshalb wird diese Anfrage manuell als Query-String gebaut statt über das `requests`-`params`-Dict (das hätte auch die Klammern kodiert und zu `400 ILLEGAL_ARGUMENT` geführt). Mehrere Post-URNs lassen sich kommasepariert in einem einzigen Request abfragen.
- Testlauf: liefert für den einen bisherigen Post Datum, Textanfang und Einzelstatistik (Impressions, Klicks, Likes, Kommentare, Engagement-Rate) separat von der Gesamtsumme.

**2026-07-19 (Fortsetzung) — Ursache für „total: 2 trotz nur 1 Post" geklärt: geplante Posts werden mitgezählt, aber nicht mitgeliefert**

- Nutzer-Hinweis führte auf die richtige Spur: Er hatte zu diesem Zeitpunkt einen Post für 21.07. geplant (später einen zweiten für 04.08.) — `total` im Paging-Objekt zählt auch nicht veröffentlichte Posts, der `q=author`-Finder liefert standardmäßig aber **nur `PUBLISHED`-Posts** in `elements` zurück.
- Fix: zusätzlicher Parameter `viewContext=AUTHOR` am `/rest/posts`-Aufruf — damit erscheinen auch Posts mit `lifecycleState: PUBLISH_REQUESTED` (= geplante Posts) in der Liste. Verifiziert: jetzt korrekt 3 Elemente (1× `PUBLISHED`, 2× `PUBLISH_REQUESTED`), passend zu den beiden vom Nutzer geplanten Posts (21.07., 04.08.).
- Einschränkung: LinkedIn liefert für `PUBLISH_REQUESTED`-Posts keinen tatsächlichen geplanten Veröffentlichungszeitpunkt über die API — nur `createdAt` (Anlagezeitpunkt des Entwurfs). Der eigentliche Schedule-Termin ist über diesen Endpunkt nicht auslesbar.
- Statistik-Abfrage (`share_stats_for_posts`) wird jetzt nur noch für `PUBLISHED`-Posts aufgerufen, nicht für geplante — die Einzelpost-Übersicht zeigt geplante Posts trotzdem mit an, aber ohne Statistik-Zeile.

**2026-07-19 (Fortsetzung) — Bewusste Fokus-Entscheidung: nur Statistik, keine Schreib-Automatisierung**

- Schreib-Scopes (`w_organization_social`, `w_organization_social_feed`) sind vorhanden und genehmigt — Post erstellen/bearbeiten/löschen, kommentieren, reagieren wären technisch möglich, wurden aber bewusst **nicht** implementiert oder getestet.
- Entscheidung des Nutzers: Posts plant er weiterhin selbst manuell in der LinkedIn-UI (Volumen ist überschaubar). Das eigentliche Ziel ist **Analyse**, nicht Automatisierung: Statistiken sollen helfen zu verstehen, wie sich mehr Follower gewinnen lassen.
- Konsequenz für die Weiterentwicklung: Der nächste sinnvolle Schritt ist nicht Content-Automatisierung, sondern der bereits oben notierte Punkt „regelmäßiger Abruf + Ablage für echte Follower-Verlaufshistorie" — ohne Zeitreihe lässt sich nicht auswerten, welche Posts/Zeitpunkte tatsächlich zu Follower-Wachstum geführt haben.

**2026-07-19 (Fortsetzung) — Historisierung eingerichtet**

- Neues Skript `/home/deploy/tools/linkedin-orgpage/snapshot.py` (importiert Funktionen aus `fetch_stats.py`) ruft Follower-Zahl, aggregierte Post-Statistik und Einzelpost-Statistik ab und hängt sie als eine JSON-Zeile an `/home/deploy/tools/linkedin-orgpage/history.jsonl` an (Rechte 600).
- Cronjob als `deploy`-User eingerichtet: täglich 05:00 Uhr, Log unter `/home/deploy/tools/linkedin-orgpage/cron.log`. Bestehender Cronjob (`update-landingpage-repo-develop.sh`, 06:00 Uhr) wurde nicht verändert.
- Erste Historien-Zeile manuell erzeugt und verifiziert (2026-07-19).
- Offene Frage/Idee des Nutzers: Statistiken zusätzlich im Normdex-Verwaltungsportal anzeigen (z. B. eigener „Marketing"-Reiter mit LinkedIn-KPIs). Dafür noch keine Entscheidung getroffen — würde eine eigene Datenhaltung in der App (DB-Tabelle + Migration) oder einen Backend-Endpunkt bedeuten, der die JSONL-Historie vom VPS-Tool-Ordner einliest. Separates Thema, noch nicht umgesetzt.
