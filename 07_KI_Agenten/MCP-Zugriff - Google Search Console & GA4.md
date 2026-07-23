# MCP-Zugriff: Google Search Console & GA4

Seit 2026-07-23 hat Claude (Claude Code) über einen MCP-Server namens **`normdex-seo`** direkten Lesezugriff auf Google Search Console und Google Analytics 4. Damit lassen sich SEO- und Traffic-Fragen ("wie performt die Landingpage in Search Console diesen Monat", "welche Seiten laufen in GA4 am besten") ohne manuellen Export direkt im Chat beantworten.

## Verfügbare Werkzeuge

| Tool | Zweck |
|---|---|
| `gsc_search_analytics` | Klicks, Impressionen, CTR, Position — gruppierbar nach `query`, `page`, `country`, `device`, `date`, `searchAppearance` |
| `gsc_inspect_url` | URL-Inspection: Indexierungsstatus, Crawling, Canonical, Mobile Usability für eine einzelne URL |
| `gsc_sitemaps_list` | Eingereichte Sitemaps und deren Verarbeitungsstatus |
| `gsc_list_sites` | Listet alle Search-Console-Properties, auf die das zugrunde liegende Service-Account Zugriff hat (nützlich zur Prüfung der exakten `siteUrl`-Schreibweise) |
| `ga4_run_report` | GA4-Report über beliebige Dimensionen (`pagePath`, `sessionDefaultChannelGroup`, `deviceCategory`, `country` etc.) und Metriken (`screenPageViews`, `sessions`, `activeUsers`, `conversions`, `bounceRate` etc.) |
| `ga4_run_realtime_report` | Aktive Nutzer der letzten ~30 Minuten |

## Bekannte Default-Werte

- **Search-Console-Property:** `https://normdex.at/` (Default, falls `siteUrl` nicht angegeben wird)
- **GA4-Property-ID:** `528084185` (Default, falls `propertyId` nicht angegeben wird)

## Funktionsweise (Stand dieser Erkenntnis)

- Der Zugriff läuft laut Tool-Beschreibung über ein **Google Service Account**, das in der Search Console als Nutzer auf die Property `normdex.at` freigegeben wurde und in GA4 Leserechte auf die Property `528084185` hat.
- Der MCP-Server selbst (`normdex-seo`) läuft außerhalb dieses Repos/Vaults — Konfiguration, Service-Account-Credentials und genaue Berechtigungen sind hier nicht einsehbar und müssten bei Bedarf direkt in der `claude mcp`-Konfiguration bzw. den hinterlegten Secrets geprüft werden.
- Der Zugriff ist **lesend**. Es gibt keine Tools zum Ändern von GSC- oder GA4-Einstellungen.

## Bekannte Einschränkung: GA4 zeigt seit 2026-06-28 kaum noch Daten

Bei der ersten Nutzung dieses Zugriffs (2026-07-23) fiel auf, dass GA4 für Juli 2026 fast keinen Traffic zeigt, während Search Console normale Werte liefert. Ursache ist ein Consent-Fix in der Landingpage (Commit `1eec259`), der das GA4-Script erst nach aktivem Cookie-Opt-in lädt. Details, Root Cause und Optionen: [[T050-ga4-tracking-nach-consent-fix-fast-keine-daten]].

## Verwandte Dokumente

- [[T050-ga4-tracking-nach-consent-fix-fast-keine-daten]]
- [[AI Kontext - Einstieg]]
