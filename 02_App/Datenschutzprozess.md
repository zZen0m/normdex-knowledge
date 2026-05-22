# Datenschutzprozess

Stand: 2026-05-22

Dieses Dokument beschreibt den aktuellen operativen Umgang mit Auskunfts-, Datenkopie- und Kontolöschanfragen in der Normdex-App.

## Grundsatz

Normdex bietet aktuell keinen Self-Service-Datenexport in der App an.

Der frühere Export-Button wurde aus der UI entfernt und die API-Endpunkte für den Self-Service-Export sind deaktiviert:

```text
POST /users/me/export/request
GET  /users/me/export/download
```

Beide Endpunkte antworten mit `410 Gone` und verweisen auf den Supportprozess. Grund: Ein unvollständiger automatischer Export wäre riskanter als ein bewusst manueller Prozess.

## Datenkopie / Auskunft

Wenn eine Nutzerin oder ein Nutzer eine Kopie personenbezogener Daten anfordert:

1. Anfrage über Support entgegennehmen, z. B. `office@normdex.at`.
2. Identität prüfen, bevor personenbezogene Daten herausgegeben werden.
3. Betroffene Datenbereiche prüfen:
   - Nutzerprofil und Kontaktdaten
   - Organisation und Mitgliedschaften
   - Projekte und Berechnungen
   - Lizenzen, Lizenzbestellungen und Lizenzereignisse
   - Supporttickets und Supportnachrichten
   - Newsletter-/Gutscheinstatus, falls vorhanden
   - relevante Audit-/Sicherheitsereignisse, soweit herausgabefähig
4. Daten in einem gängigen elektronischen Format bereitstellen, z. B. ZIP mit JSON/CSV/PDF, abhängig vom konkreten Umfang.
5. Daten Dritter, interne Notizen, Geschäftsgeheimnisse und sicherheitsrelevante Details vor Herausgabe prüfen und ggf. schwärzen.
6. Bearbeitung intern dokumentieren: Datum, anfragende Person, geprüfte Identität, bereitgestellte Datenbereiche, Bearbeiter.

## Kontolöschung

Die App-Löschung ist bewusst keine harte physische Löschung des `users`-Datensatzes.

Technischer Zielzustand:

- Login wird deaktiviert.
- Sessions und Tokens werden ungültig.
- Personenbezogene Profilfelder werden entfernt oder anonymisiert.
- Projekte und Berechnungen des Kontos werden gelöscht.
- Mitgliedschaften werden entfernt.
- Aktive Lizenznutzungen werden entfernt.
- Lizenzzuweisungen und Lizenzereignisse werden vom Nutzer entkoppelt.
- Supporttickets und Supportnachrichten werden vom Nutzer entkoppelt und personenbezogene Kontaktdaten werden anonymisiert.
- Newsletter-Gutscheinclaims werden anonymisiert.
- Bestell-, Lizenz-, Audit- und Abrechnungskontext kann erhalten bleiben, sofern er für rechtliche, abrechnungsbezogene, Sicherheits- oder Nachweiszwecke benötigt wird.

Der anonymisierte Nutzer bleibt als technischer Platzhalter bestehen, damit bestehende Fremdschlüssel, Lizenzbestellungen und Auditkontexte konsistent bleiben.

## Owner-Schutz bei Löschung

Ein Konto kann nicht gelöscht werden, wenn es der letzte Owner einer Organisation ist und weitere Mitglieder in dieser Organisation verbleiben.

Vor der Löschung muss in diesem Fall zuerst ein anderer Owner bestimmt werden.

## UI-Kommunikation

Die UI darf nicht behaupten, dass alle verbundenen Daten unwiderruflich gelöscht werden.

Zulässige Kernaussage:

```text
Der Zugang wird deaktiviert. Personenbezogene Kontodaten werden gelöscht oder anonymisiert. Daten, die aus rechtlichen, abrechnungsbezogenen oder sicherheitsrelevanten Gründen aufbewahrt werden müssen, bleiben gemäß Datenschutzerklärung erhalten.
```

## Offene Punkte

- Juristische Prüfung der finalen Datenschutzerklärung.
- Admin-Unterstützung für Datenkopien und DSGVO-Wipe als späterer Verwaltungsportal-Ausbau.
- Dateninventar regelmäßig nach neuen Tabellen/Funktionen aktualisieren.
