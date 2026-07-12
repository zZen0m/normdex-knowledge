# T047 · LinkedIn API-Freigabe für Unternehmensseiten-Statistiken beantragen

**Phase:** Marketing / LinkedIn-Markenkanal / Reporting
**Priorität:** P3 · Nice-to-have für Reporting, kein Blocker für laufenden Betrieb
**Status:** offen
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

- Reicht die Community Management API allein, oder wird für bestimmte Kennzahlen (z. B. Impressions pro Post) doch die Marketing Developer Platform benötigt?
- Wie lange dauert die Freigabe in der Praxis (bisher keine eigene Erfahrung damit)?
- Nach Freigabe: Soll der Abruf über n8n (HTTP-Request-Node + OAuth2) oder direkt aus der Normdex-App erfolgen?

## Nächste Schritte

- [ ] Bei developer.linkedin.com als Page-Admin anmelden und App anlegen
- [ ] App mit LinkedIn-Unternehmensseite „normdex" verknüpfen
- [ ] Community Management API beantragen (Use-Case: automatisiertes Reporting der Page-Statistiken für internes Marketing-Tracking)
- [ ] Nach Freigabe: Abrufweg entscheiden (n8n vs. direkte Integration) und Todo entsprechend fortschreiben oder neues Umsetzungs-Todo anlegen
