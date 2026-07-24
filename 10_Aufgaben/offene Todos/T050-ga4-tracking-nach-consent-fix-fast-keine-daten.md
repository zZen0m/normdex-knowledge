# T050 · GA4 erfasst seit Consent-Fix fast keinen Traffic mehr

**Phase:** Landingpage / Analytics / Datenschutz
**Priorität:** P3 · Datenlage beeinträchtigt, kein Produktionsbug
**Status:** fast abgeschlossen (nur Retention-Einstellung offen)
**Datum:** 2026-07-23
**Zuletzt aktualisiert:** 2026-07-24

## Ziel

Klären, ob und wie die GA4-Datenerfassung auf der Landingpage verbessert werden soll, nachdem der Opt-in-Consent-Fix vom 2026-06-28 dazu geführt hat, dass GA4 im Juli praktisch keine Sessions/Pageviews mehr zeigt.

## Kontext

- Auslöser: Frage im Chat "wie performt die Landingpage in Search Console/GA4 diesen Monat" → GA4-Report für 2026-07-01 bis 2026-07-23 lieferte **0 Zeilen**, während GSC im selben Zeitraum normale Daten zeigt (8 Klicks, ~115 Impressionen über alle Seiten).
- Zum Vergleich: GA4-Report für Juni 2026 zeigte reale Daten (43 Views auf `/`, 10 auf `/newsletter`, 7 auf `/kontakt` etc.).
- Ursache identifiziert: Commit `1eec259` "fix: load analytics after consent" (2026-06-28) in `normdex-landingpage`.
  - Vorher lud `index.html` das gtag.js-Script **immer** und signalisierte nur per Consent Mode v2 `analytics_storage: denied` — Google konnte dadurch über Modeling noch grobe Sessions/Pageviews schätzen.
  - Der Fix entfernte das Script komplett aus `index.html` und verschob es nach [[src/components/GoogleAnalytics.tsx]]. Dort wird `loadGoogleAnalytics()` (Zeile 53) jetzt **nur noch aufgerufen, wenn `localStorage['cookie-consent'].analytics === true`** ist — das gtag.js-Script wird für die meisten Besucher also gar nicht mehr nachgeladen, solange sie nicht aktiv im Cookie-Banner zustimmen.
  - Das ist rechtlich der korrektere Zustand (echtes Opt-in statt "immer laden, nur Signal denied"), führt aber dazu, dass GA4 strukturell nur noch einen Bruchteil des echten Traffics erfasst.
- Der Cookie-Banner ([[src/components/CookieConsent.tsx]]) zeigt drei gleichwertig gestylte Buttons ("Einstellungen", "Nur Notwendige", "Alle akzeptieren") ohne visuelle Hierarchie.
  - Wichtig: Nach EDPB-Guidance zu Dark Patterns muss "Ablehnen" genauso leicht/prominent bleiben wie "Akzeptieren" — den Accept-Button optisch hervorzuheben wäre ein unzulässiges Nudging und würde die Einwilligung rechtlich angreifbar machen. Diese Option scheidet aus.
- Search-Console-Daten sind von diesem Problem nicht betroffen, da sie unabhängig vom Consent-Status direkt von Google erhoben werden.

## Mögliche Optionen (noch nicht entschieden)

1. **Copy im Cookie-Banner verbessern** – konkret erklären, wofür Analytics-Daten genutzt werden (Transparenz statt Nudging), um die Opt-in-Rate organisch zu erhöhen. Rechtlich unproblematisch, Wirkung vermutlich gering (B2B-Fachpublikum, Opt-in-Raten typischerweise 20–40%).
2. **Granularität im Haupt-Banner sichtbar machen**, statt sie hinter "Einstellungen" zu verstecken, um Vertrauen zu schaffen.
3. **Cookieloses Analytics-Tool ergänzen** (z.B. Plausible/Fathom), das ohne personenbeziehbare Cookies und damit ohne Consent-Banner zählt — würde belastbare Zahlen unabhängig von der Opt-in-Quote liefern. Vermutlich der wirksamste Hebel.
4. **So akzeptieren** – GA4 zeigt künftig nur noch "echte Zustimmer", stattdessen stärker auf GSC-Klicks/Impressionen als Performance-Indikator setzen.

## Entscheidung (2026-07-24)

Kombination aus **Option 3** (cookieloses Analytics-Tool) und **Option 4** (GSC stärker als Performance-Indikator nutzen), GA4 bleibt parallel bestehen.

### Umsetzungsplan

- [x] **Tool:** Plausible, self-hosted (eigener Postgres- + ClickHouse-Container im Plausible-Stack, getrennt von der App-DB)
- [x] **Server:** derselbe Server wie `api.normdex.at` / `app.normdex.at` (`normdex-vps`, `/opt/stacks/normdex-analytics`), im bestehenden Traefik-Netzwerk (`proxy`) — Stack läuft (3 Container `Up`, intern via `proxy`-Netzwerk mit 302 auf `/` bestätigt)
- [x] **Subdomain:** `analytics.normdex.at` — DNS A-Record gesetzt, Let's-Encrypt-Zertifikat ausgestellt (gültig bis 22.10.2026), Seite live erreichbar
- [x] **Repo:** neues privates GitHub-Repo [normdex-analytics](https://github.com/zZen0m/normdex-analytics) angelegt (per `gh` CLI), Compose-Stack, ClickHouse-Config, `.env.example` und README gepusht
- [x] **Tracking-Script:** erweiterte Variante (`file-downloads.hash.outbound-links.pageview-props.revenue.tagged-events`) plus `window.plausible`-Queue-Shim in `normdex-landingpage/index.html`, immer geladen, keine Kopplung an `CookieConsent.tsx`
- [x] **GA4:** bleibt bestehen, weiterhin consent-gated wie bisher (`src/components/GoogleAnalytics.tsx`), für Marketing-/Kampagnen-Attribution mit echtem Opt-in
- [x] **Datenschutzerklärung:** Absatz zu Plausible ergänzt (`src/pages/Datenschutz.tsx`, Rechtsgrundlage Art. 6 Abs. 1 lit. f DSGVO, keine Cookies, 26 Monate Speicherdauer)
- [x] **SMTP:** Brevo-Zugangsdaten aus dem laufenden Prod-API-Container wiederverwendet, in `.env` auf dem Server hinterlegt
- [ ] **Data Retention:** 26 Monate — Site `normdex.at` in Plausible angelegt (Admin registriert), Retention-Einstellung unter Site Settings > General noch manuell zu setzen (Nutzer)
- [x] **Deployment:** Stack läuft auf `normdex-vps` unter `/opt/stacks/normdex-analytics` (SSH-Zugriff war entgegen ursprünglicher Annahme doch vorhanden — Host `normdex-vps` in `~/.ssh/config`)
- [x] **Option 4 (GSC):** Praxis-Hinweis im Vault dokumentiert ([[Landingpage]] → Abschnitt „Analytics")

### Offene manuelle Schritte (Nutzer)

1. Data Retention der Site `normdex.at` in der Plausible-UI (Site Settings > General) auf 26 Monate stellen.

## Akzeptanzkriterien

- [x] Entscheidung getroffen, ob und welche der obigen Optionen umgesetzt werden. → Option 3 + 4 kombiniert (siehe oben).
- [x] Plausible self-hosted eingerichtet und unter `analytics.normdex.at` erreichbar. → Live, Admin-Account registriert, Registrierung geschlossen (`ENABLE_REGISTRATION=invite_only`).
- [x] Tracking-Script in der Landingpage integriert (ohne Consent-Kopplung).
- [x] Datenschutzerklärung um Plausible-Absatz ergänzt.
- [x] GSC-Praxis-Hinweis im Vault dokumentiert (primäre SEO-Quelle statt GA4).
- [x] Ergebnis im Vault dokumentiert (dieses Todo laufend aktualisiert).

## Notizen / Fortschritt

### 2026-07-23

- Todo aus Chat-Analyse angelegt, nachdem GSC- und GA4-Performance für Juli verglichen wurden und die Diskrepanz auffiel.
- Root Cause im Code verifiziert (Commit `1eec259`, `src/components/GoogleAnalytics.tsx:53-76`).

### 2026-07-24

- Entscheidung getroffen: Option 3 (Plausible self-hosted) + Option 4 (GSC als primäre SEO-Quelle) kombiniert, GA4 bleibt parallel bestehen.
- Details per Interview geklärt (Hosting, Repo, Consent, SMTP, Retention) — siehe Umsetzungsplan oben.
- Umsetzung (Repo, Compose, Tracking-Script, Datenschutztext) noch offen — nur Todo/Planung, noch keine Implementierung.

### 2026-07-24 (Fortsetzung)

- Repo [normdex-analytics](https://github.com/zZen0m/normdex-analytics) angelegt: `docker-compose.yml` (Plausible + Postgres + ClickHouse, Traefik-Labels für `analytics.normdex.at` im `proxy`-Netzwerk), `.env.example` (inkl. Brevo-SMTP, Retention-Hinweis), ClickHouse-Logging-Configs, README — committed und gepusht.
- `normdex-landingpage`: Plausible-Script (`analytics.normdex.at/js/script.js`, `data-domain="normdex.at"`) fest in `index.html` eingebaut, unabhängig von `CookieConsent.tsx`. Datenschutzerklärung (`src/pages/Datenschutz.tsx`) um eigenen Plausible-Abschnitt ergänzt (vor Google Analytics), nachfolgende Abschnittsnummerierung angepasst. Typecheck grün, lokal committed (noch nicht gepusht — Branch `develop` hatte bereits 2 unabhängige lokale Commits vor dieser Änderung).
- Vault: neuer Abschnitt „Analytics" in [[Landingpage]] mit Stack-Übersicht und GSC-als-primäre-SEO-Quelle-Hinweis.
- **Noch offen:** tatsächliches Deployment von Plausible auf dem Server (`docker compose up -d`, Admin-Account, Site in Plausible-UI mit 26 Monaten Retention anlegen) — manuell durch den Nutzer, da kein SSH-Zugriff für den Agenten. Landingpage-Commit muss noch gepusht werden.

### 2026-07-24 (Deploy)

- Entgegen der ursprünglichen Annahme gab es doch SSH-Zugriff auf den Server (Host `normdex-vps` in `~/.ssh/config`, `deploy@85.215.218.157`) — Deployment komplett durchgeführt statt nur vorbereitet.
- Repo auf dem Server geclont (`/opt/stacks/normdex-analytics`), `.env` mit echten Secrets geschrieben (Postgres-Passwort, `SECRET_KEY_BASE`/`TOTP_VAULT_KEY` generiert, Brevo-SMTP aus dem laufenden Prod-API-Container ausgelesen und wiederverwendet), `docker compose up -d` gestartet.
- Bug gefunden und gefixt: der ursprüngliche Compose-Command rief `/entrypoint.sh db init-admin` auf — dieses Skript existiert im aktuellen Plausible-CE-Image (`ghcr.io/plausible/community-edition:v2`) nicht mehr, Admin-Accounts werden nicht mehr automatisiert über Env-Vars angelegt. Fix: Command auf `createdb && migrate && run` reduziert, `.env.example` und Server-`.env` um `ENABLE_REGISTRATION=public` ergänzt, damit sich der Admin einmalig selbst über `/register` registrieren kann.
- Nutzer hat DNS-A-Record (`analytics.normdex.at` → `85.215.218.157`) gesetzt; initial löste der lokale Fritzbox-Resolver noch NXDOMAIN auf (Google/Cloudflare hatten den Eintrag schon), reines Cache-Timing, kein Fehler — nach kurzem Warten aufgelöst.
- Traefik hat automatisch ein Let's-Encrypt-Zertifikat für `analytics.normdex.at` gezogen (gültig bis 22.10.2026), Seite ist live.
- Nutzer hat sich unter `/register` mit `office@normdex.at` registriert und die Site `normdex.at` in der Plausible-UI angelegt (Tracking-Snippet daraus kopiert).
- `ENABLE_REGISTRATION` danach auf `invite_only` zurückgestellt, Container neu gestartet — Registrierung ist jetzt geschlossen.
- Von Plausible generierter Snippet enthielt mehr als der Standard-Tag: erweiterte Script-Datei mit Tracking-Extensions (`file-downloads.hash.outbound-links.pageview-props.revenue.tagged-events.js`) plus `window.plausible`-Queue-Shim für zukünftige Custom-Events/Conversions — `index.html` entsprechend korrigiert, committed und auf `develop` gepusht (zusammen mit den vorherigen zwei unabhängigen Commits des Nutzers).
- **Einziger offener Punkt:** Data Retention (26 Monate) muss der Nutzer noch manuell in der Plausible-UI unter Site Settings > General setzen — keine Env-Var/API dafür ohne Plausible-API-Key, daher bewusst nicht per DB-Hack gesetzt.
