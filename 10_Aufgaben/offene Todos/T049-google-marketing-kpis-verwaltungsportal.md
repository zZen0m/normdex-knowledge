# T049 · Google-Marketing-KPIs im Verwaltungsportal integrieren

**Phase:** App / Admin / Verwaltungsportal / Marketing  
**Priorität:** P3 · nach Abschluss des LinkedIn-Rollouts  
**Status:** offen  
**Datum:** 2026-07-19  
**Zuletzt aktualisiert:** 2026-07-19

## Ziel

Der Admin-Bereich `/admin/marketing` wird nach LinkedIn um Google Ads und Google Analytics erweitert. Admins können über den bereits vorbereiteten Kanalumschalter zwischen den drei eigenständigen Auswertungen wechseln. Google-Daten werden erst angezeigt, wenn die jeweilige Integration sicher eingerichtet und der Datenstand eindeutig gekennzeichnet ist.

## Kontext

- [[T048-linkedin-marketing-kpis-verwaltungsportal]] setzt LinkedIn als ersten funktionsfähigen Marketingkanal um.
- Die Marketingseite enthält seit der T048-Nachschärfung sichtbare Kanalziele für LinkedIn, Google Ads und Google Analytics.
- Die Google-Ansichten sind zunächst ausdrücklich als geplant gekennzeichnet und rufen keine Google-Daten ab.
- Google Analytics ist bereits als externer Dienst im Datenschutzprozess dokumentiert; vor einer produktiven API-Anbindung sind Zweck, Berechtigungen und Aufbewahrung erneut zu prüfen.

## Verbindliche Produktentscheidungen

- Google Ads und Google Analytics bleiben getrennte Kanäle mit jeweils passenden KPIs und Statusinformationen.
- Keine erfundenen Nullwerte oder scheinbar verbundenen Zustände: Ohne Integration wird ein klarer Einrichtungszustand angezeigt.
- Live-Abfragen und OAuth-/Service-Credentials sind nur in Produktion zulässig; lokal und auf `dev-server` werden sichere Fixtures verwendet.
- Alle Endpunkte, Credentials und Auswertungen bleiben ausschließlich für Normdex-Admins zugänglich.
- Gemeinsame Zeiträume und kanalübergreifende Vergleiche werden erst eingeführt, wenn die fachlichen Definitionen der Kennzahlen kompatibel und dokumentiert sind.

## Vorgesehener Umfang

### Google Ads

- Kontoverbindung und sicherer Credential-Lifecycle
- Kampagnenstatus, Ausgaben, Impressionen, Klicks, CTR, CPC und definierte Conversions
- Filter nach Zeitraum und Kampagne
- Sync-Status, Fehlerzustände und manuelle Aktualisierung nach dem T048-Muster

### Google Analytics

- GA4-Property-Verbindung und sicherer Credential-Lifecycle
- Nutzer/Sitzungen, Quellen beziehungsweise Kanäle, Landingpages und definierte Schlüsselereignisse
- Filter nach Zeitraum und den fachlich sinnvollen Dimensionen
- Sync-Status, Datenaktualität und transparente Kennzahlendefinitionen

## Akzeptanzkriterien

- [ ] Kanalumschalter zeigt LinkedIn, Google Ads und Google Analytics zugänglich und responsive an.
- [ ] Jeder Google-Kanal hat eine eigene fachliche Spezifikation, API-Schicht, Persistenz und Admin-Ansicht.
- [ ] Unverbundene, laufende, erfolgreiche, veraltete und fehlerhafte Zustände sind eindeutig unterscheidbar.
- [ ] Secrets, OAuth-Tokens und vollständige Provider-Antworten erscheinen weder in Logs noch im Frontend.
- [ ] Lokal und `dev-server` verwenden realistische Fixtures und führen keine Google-Live-Abfragen aus.
- [ ] Automatisierte Backend-/Frontend-Tests, Security-Audits und gestufter Rollout sind dokumentiert und erfolgreich.
- [ ] Datenschutz-, Consent- und Aufbewahrungsanforderungen sind vor Produktivaktivierung geprüft.
- [ ] Nutzerabnahme auf `dev-server` ist erfolgt.

## Abhängigkeiten

- Abschluss und Produktivverifikation von [[T048-linkedin-marketing-kpis-verwaltungsportal]]
- Zugriff auf das passende Google-Ads-Konto und die GA4-Property
- Freigabe der gewünschten Conversion- beziehungsweise Schlüsselereignisse

## Notizen / Fortschritt

### 2026-07-19

- Todo aus der T048-Nutzerabnahme angelegt.
- Der gemeinsame Kanalumschalter wird bereits in T048 vorbereitet; die echten Google-Integrationen bleiben bewusst in diesem separaten Arbeitspaket.
- Vorbereiteter Umschalter auf `dev-server` mit Commit `d1a980f` ausgerollt und im deployten Marketing-Chunk verifiziert.
