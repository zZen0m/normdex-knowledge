# Businessplan Normdex

> Stand: 2026-06-30. Erstellt auf Basis des realen Produktivstands der Webapp (`app.normdex.at`, v0.1.2), der Landingpage (`normdex.at`, v0.0.4) und der Vault-Wissensbasis. Interne Arbeitsgrundlage, kein Investorendokument. Zahlen sind der reale Produktivstand zum Stichtag.

## 1. Executive Summary

Normdex ist eine österreichische B2B-SaaS-Plattform für Wirtschaftlichkeitsberechnungen von Energiesystemen nach ÖNORM M 7140. Die Berechnungsverfahren (Kapitalwert, Annuität, Amortisation) sind nach Abschnitt 10 der Norm validiert. Das Produkt läuft produktiv, ist voll funktionsfähig (Berechnung, PDF-Report, Team-Verwaltung, Lizenz- und Abrechnungssystem über Stripe, Support- und Admin-Portal) und wird von der Permatec e.U. als Einzelunternehmen betrieben.

Die aktuelle Phase ist Markteintritt und Validierung, nicht Skalierung. Ziel bis Jahresende 2026 sind die ersten fünf bis zehn zahlenden Kunden über einen reinen Self-Service-Funnel mit kleinem Budget. Zum Stichtag gibt es einen aktiven externen Zahlkunden, einen weiteren Neuzugang in Anbahnung und eine fertige technische Basis, auf der aufgebaut werden kann.

Der strategische Kern: In einem besetzten Nischenmarkt nicht über Normautorität konkurrieren, sondern über Produkterlebnis. Moderne Web-Lösung statt klassischer Desktop-Software. Im Browser, im Team, mit prüffähigen Reports.

## 2. Produkt und aktueller Stand

### Was Normdex ist

Ein professionelles Berechnungswerkzeug für die wirtschaftliche Bewertung von Energieprojekten und Investitionen über einen mehrjährigen Zeithorizont, ergänzt um Projektverwaltung und Team-Funktionen. Zielgruppen sind Energieberater, Planungs- und Ingenieurbüros, Projektentwickler, technische Sachverständige und Unternehmen mit Teams.

Abgrenzung: kein allgemeines Projektmanagement, keine Buchhaltung, kein Endverbraucher-Tool.

### Technischer Reifegrad (Produktiv)

| Bereich | Stand |
|---|---|
| Webapp | `app.normdex.at`, v0.1.2, React 18 + FastAPI, PostgreSQL in Docker |
| Landingpage | `normdex.at`, v0.0.4, React 18 + Vite, SEO-Grundgerüst solide |
| Kernrechner | Drei Verfahren nach ÖNORM M 7140, validiert nach Abschnitt 10, Variantenvergleich, Sensitivitätsanalyse |
| Output | Prüffähiger PDF-Report, sauber auch bei großen Beträgen |
| Abrechnung | Stripe (Kreditkarte, Klarna), Monats- und Jahrespool, Trial- und Rabattlogik, automatisierte Gutschriften |
| Betrieb | VPS mit Docker-Stacks, Traefik, automatisierter Backup-Service mit Offsite-Upload, Dev- und Prod-Trennung, Alembic-Migrationen |

Das Produkt ist betrieblich abgesichert (Backups, getrennte Umgebungen, dokumentierte Deployment-Workflows). Offene technische Punkte werden über die strukturierten Repo-Audits und das Aufgabensystem im Vault nachgehalten.

### Reale Geschäftskennzahlen (Stichtag 2026-06-30)

| Kennzahl                  | Wert                                                                                   |
| ------------------------- | -------------------------------------------------------------------------------------- |
| Organisationen gesamt     | 3 (davon Permatec als eigene Org)                                                      |
| Externe Organisationen    | 2 (Sarah Illmeier GmbH aktiv (eigener Testaccount), Russ Ingenieure GmbH in Anbahnung) |
| Aktive Lizenzen           | 2 (davon 0 externer Zahlkunde)                                                         |
| Registrierte Nutzer       | 9                                                                                      |
| Angelegte Projekte        | 6                                                                                      |
| Lizenzbestellungen gesamt | 4                                                                                      |

Interpretation: Das Produkt ist am Markt, die ersten echten Transaktionen laufen. Die Basis ist klein und damit voll im Plan der Validierungsphase. Der Engpass ist nicht das Produkt, sondern qualifizierter Traffic und Conversion.

Hinweis zu Russ Ingenieure GmbH: kein klassischer Self-Service-Funnel-Lead, sondern der ehemalige Arbeitgeber von Andreas Gruber (Haustechnik-Planungsbüro), der sich bereit erklärt hat, Normdex zu testen und Feedback zu geben. Einzuordnen als Pilot-/Referenzkunde, nicht als Kaltakquise-Erfolg.

## 3. Markt und Wettbewerb

Der Markt ist eine klar abgegrenzte Nische mit kleinem Suchvolumen und etablierten Anbietern. Die quantitative Marktgröße (Anzahl Büros nach Bundesländern, Betriebsgrößen, TAM/SAM/SOM und Verdrängungsanalyse) ist in der [[Marktanalyse Österreich]] auf Basis der WKO-Branchendaten hergeleitet.

| Anbieter | Profil | Bedeutung |
|---|---|---|
| Pokorny Technologies (Wien) | Desktop-Software nach ÖNORM M 7140, validiert, Variantenvergleich, Sensitivitätsanalyse. Inhaber leitet den Normungsausschuss. | Platzhirsch mit maximaler Normautorität und breitem SEO-/Fachpresse-Cluster |
| Austrian Standards (BVE) | Eigene Lösung im Norm-Shop | Vertrauensanker über die Normorganisation selbst |
| Urban-Energy.at | Kleinerer Mitbewerber | Randwettbewerb |

Konsequenz: Den Wettstreit um Normautorität gewinnt Normdex nicht und führt ihn daher nicht. Frontale SEO auf den Hauptbegriff ist ein langer Kampf gegen jahrelange Domain-Autorität. Die strukturelle Lücke der etablierten Lösungen ist ihr Auslieferungsmodell: Installation per ausführbarer Datei, kein Cloud- oder Teamzugang, fehleranfällige Berichtsausgabe bei großen Beträgen.

Verbindliche Regel: Im öffentlichen Auftritt wird kein Mitbewerber namentlich genannt. Verglichen wird nur die Kategorie (klassische Desktop-Software gegen moderne Web-Lösung). Wettbewerbsanalyse bleibt internen Dokumenten vorbehalten.

## 4. Positionierung und Alleinstellung

Validierung ist Pflicht, nicht Verkaufsargument. Sie wird zur Selbstverständlichkeit erklärt, und die Kaufentscheidung auf das Feld verlagert, auf dem Normdex tatsächlich gewinnt: Bedienung, Reportqualität, Cloud und Team.

Kernpositionierung: **Die zeitgemäße Wirtschaftlichkeitsberechnung nach ÖNORM M 7140. Im Browser, ohne Installation, im Team, mit prüffähigen Reports.**

Alleinstellung gegenüber der Kategorie der Desktop-Software:

- Ortsunabhängig im Browser, ohne Installation, immer aktuell.
- Mehrere Nutzer in einer Organisation, gemeinsame Projekte, Rollen.
- Saubere, prüffähige PDF-Reports, auch bei großen Beträgen.
- Validierung nach Abschnitt 10 sachlich belegt, nicht inszeniert.

## 5. Geschäfts- und Preismodell

Wiederkehrender Umsatz über Software-Abonnements je Lizenz, gebündelt nach Monats- oder Jahrespool. Abrechnung über Stripe.

| Plan | Erste Lizenz | Je weitere Lizenz | Hinweis |
|---|---|---|---|
| Monatlich | 49 €/Monat | 29 €/Monat | Maximale Flexibilität |
| Jährlich | 490 €/Jahr | 290 €/Jahr | 17 % günstiger, entspricht 2 Gratis-Monaten |

Anreiz- und Konditionslogik:

- 14 Tage kostenlos beim qualifizierten Einzel-Erstkauf. Zahlungsdaten werden im Checkout erfasst.
- Bei qualifiziertem Mehrfach-Erstkauf statt Trial einmalig 24,50 € Rabatt auf die erste Rechnung.
- Newsletter-Coupon 10 % im ersten Monat, individuell, einmalig einlösbar, 30 Tage gültig.
- Kündigung zum jeweiligen Laufzeitende, Zusatzlizenzen vor der Hauptlizenz.

Sonderfall Pilotkunden: Für längere kostenlose Nutzungsphasen (zum Beispiel sechs Monate für strategische Pilotkunden) kann der Trial-Zeitraum in Stripe verlängert werden, bevor die erste Rechnung gestellt wird. Das ist aktuell ein manueller Workaround und ein Kandidat für eine eigene Admin-Funktion.

## 6. Go-to-Market

Rahmenbedingungen, die den Plan bestimmen: Kaltstart ohne Branchennetzwerk, Budget unter 2.000 € pro Jahr, Umsetzung solo und nebenbei, reiner Self-Service ohne Demo-Calls, Trial an Zahlungsdaten gebunden.

Funnel-Logik: Die Trial verlangt die Karte und ist damit eine echte Hürde für kalten Verkehr. Davor liegt der Newsletter als Vertrauens- und Auffangschicht. Wer nicht sofort kauft, wird über einen Lead-Magneten (Praxisleitfaden plus Checkliste und Beispielbericht) eingesammelt und über eine Nurture-Strecke an die Trial herangeführt.

| Kanal                                                           | Rolle                                                                  | Priorität |
| --------------------------------------------------------------- | ---------------------------------------------------------------------- | --------- |
| Conversion-Fläche (Landingpage, Beispiel-Report, Methodikseite) | Verkauft im Self-Service allein, muss vor jedem bezahlten Klick stehen | Zuerst    |
| Newsletter und Lead-Magnet                                      | Auffangnetz für alle, die nicht sofort die Karte hinterlegen           | Hoch      |
| Google Ads                                                      | Aktiver Motor auf Begriffe mit klarer Kaufabsicht, kleines Tagesbudget | Hoch      |
| LinkedIn-Markenkanal                                            | Fachposts unter der Marke, führen zu Newsletter oder Beispiel-Report   | Mittel    |
| SEO und Content                                                 | Langfristige, kumulierende Wette auf Long-Tail-Fragen                  | Laufend   |

Verbindliche Schreibregeln für alle öffentlichen Texte: Du-Form, direkte Umlaute, keine Gedankenstriche als Satzzeichen, sachlich statt werblich, kein Mitbewerber namentlich.

## 7. Ziele und Roadmap 2026

Leitziel: bis Jahresende fünf bis zehn zahlende Kunden, erste Referenzen und Testimonials, ein wiederholbarer Akquise-Weg. Umsatz ist in dieser Phase zweitrangig.

Mengenlogik: Bei rund 30 % Trial-zu-zahlend braucht es etwa 25 Trials für acht Kunden, also zwei bis drei Trials pro Monat.

| Phase | Schwerpunkt |
|---|---|
| Fundament | Landingpage schärfen, öffentlichen Beispiel-Report, Methodik- und Validierungsseite, Lead-Magnet und Nurture-Strecke |
| Sichtbarkeit | LinkedIn-Unternehmensseite, Google Ads klein starten, beide Zielseiten testen, wöchentlich messen |
| Kumulieren | Ein Fachbeitrag pro Monat, zwei bis vier LinkedIn-Posts, aktiv Testimonials einsammeln |

Produktseitige Roadmap-Kandidaten: erweiterte Export-Schnittstellen (Excel, GAEB), integrierte CO2-Bilanzierung, weitere Team-Funktionen. Excel-Export ist als Aufgabe bereits dokumentiert.

## 8. Kennzahlen zur Steuerung

Wenige, klare Metriken genügen in dieser Phase:

- Newsletter-Abonnenten pro Monat
- Trial-Starts pro Monat
- Trial-zu-zahlend (Conversion)
- Kosten pro Trial aus den Ads
- Aktive Lizenzen und monatlich wiederkehrender Umsatz (MRR)

## 9. Organisation und Recht

| Feld | Inhalt |
|---|---|
| Betreiber | Permatec e.U., eingetragenes Einzelunternehmen |
| Inhaber | Dipl.-Ing. Andreas Gruber |
| Sitz | Bruck an der Mur, Österreich |
| Marke | Normdex™ ist eine Marke der Permatec e.U. |
| Aufstellung | Solo, nebenberuflich, schlanke Kostenbasis |

Vorteil der Aufstellung: minimale Fixkosten, schnelle Entscheidungen, voller technischer Durchgriff. Grenze: begrenzte Zeit pro Woche und Klumpenrisiko auf einer Person. Daraus folgt die bewusste Wahl wartungsarmer, einmal aufgebauter Kanäle statt dauernd betreuter Vertriebsaktivitäten.

## 10. Risiken und Gegenmaßnahmen

| Risiko | Gegenmaßnahme |
|---|---|
| Etablierter Platzhirsch mit Normautorität und SEO-Vorsprung | Nicht über Autorität konkurrieren, sondern über Produkterlebnis und Kategorie-Argument |
| Kleines Suchvolumen der Nische | Bezahlte Suche mit kleinem Budget plus Newsletter als Auffangnetz, Long-Tail-SEO als Langfristwette |
| Kartenpflicht der Trial bremst kalten Verkehr | Lead-Magnet und Nurture-Strecke als Vertrauensschicht davor |
| Solo-Betrieb, begrenzte Zeit, Personenrisiko | Wartungsarme Kanäle, dokumentierte Workflows, automatisierte Backups, getrennte Umgebungen |
| Technische Release-Qualität (Build, Dependencies, Security) | Strukturierte Repo-Audits, CI-Gates, Aufgabensteuerung im Vault; offene Sicherheitsaufgaben (z. B. Secret-Rotation T026) priorisiert abarbeiten |
| Wenige Referenzen zum Start | Von den ersten Nutzern aktiv Testimonials einsammeln, die mit der Zeit fehlende Autorität ersetzen |

## 11. Finanzplanung: Kosten und Umsatz

> Ergänzt am 2026-07-03. Die Fixkosten unten sind der reale Ist-Stand. Die Umsatzzahlen sind ein Planszenario auf Basis der Ziele aus Abschnitt 7 und des Preismodells aus Abschnitt 5, kein Forecast. Sie stehen und fallen mit dem Aufbau eines funktionierenden Akquise-Motors.

### 11.1 Laufende Fixkosten (Ist-Stand)

Schlanke Kostenbasis, wie sie zum Solo-Betrieb passt. Jährlich abgerechnete Positionen sind auf den Monat umgelegt.

| Position | Anbieter | Abrechnung | Betrag | Betrag mtl. | Betrag p.a. |
|---|---|---|---|---|---|
| Microsoft 365 Business Standard | Microsoft | monatl. | 14,04 € | 14,04 € | 168,48 € |
| ChatGPT Plus | OpenAI | monatl. | 21,00 € | 21,00 € | 252,00 € |
| Claude Pro | Anthropic | monatl. | 21,99 € | 21,99 € | 263,88 € |
| VPS | — | monatl. | 10,00 € | 10,00 € | 120,00 € |
| Gewerbe | — | jährl. | 150,00 € | 12,50 € | 150,00 € |
| SVA | — | monatl. | 13,00 € | 13,00 € | 156,00 € |
| Domain Normdex.at | World4You | jährl. | 36,00 € | 3,00 € | 36,00 € |
| Domain Normdex.com | World4You | jährl. | 24,00 € | 2,00 € | 24,00 € |
| Domain Normdex.de | World4You | jährl. | 10,00 € | 0,83 € | 10,00 € |
| **Summe Fixkosten** | | | | **98,36 €** | **1.180,36 €** |

Marketing und Ads sind hier bewusst nicht enthalten. Dafür gilt der Rahmen aus Abschnitt 6: Budget unter 2.000 € pro Jahr. Ebenfalls nicht in den Fixkosten: Stripe-Transaktionsgebühren, die als variable Kosten mit dem Umsatz mitwachsen (Größenordnung rund 1,5 bis 2,9 % je Zahlung plus Fixanteil).

**Geplante Gesamtkosten 2026:** rund 1.180 € Fixkosten plus bis zu 2.000 € Marketing plus variable Stripe-Gebühren, also grob **bis 3.200 € über das Jahr**.

### 11.2 Absehbare Kostensteigerungen (mittelfristig)

| Auslöser | Effekt | Größenordnung |
|---|---|---|
| Umstieg auf größeren VPS (mehr Kunden, mehr Last) | höhere Betriebskosten | +10 bis +40 € mtl. |
| Umstieg auf größeren Claude-Plan | höhere Toolkosten | ca. 100 € mtl. statt 21,99 €, also rund +78 € mtl. (+936 € p.a.) |
| Steigende Stripe-Gebühren | variabel, wächst mit Umsatz | ca. 1,5 bis 2,9 % vom Umsatz |
| Skalierendes Marketing/Ads | höherer Wachstumshebel | Budget nach Bedarf über die 2.000 € hinaus |

Auch mit diesen Steigerungen bleibt die Kostenbasis im Verhältnis zu den Umsatzzielen klein. Der Solo-Betrieb ohne Personalkosten ist der zentrale Kostenvorteil.

### 11.3 Umsatzplanung 2026 und die nächsten fünf Jahre

Annahmen des Szenarios:

- **Eine Jahreslizenz pro Kunde**, also ARPU **490 € pro Kunde und Jahr**. Das ist die konservative, realistische Annahme: Der durchschnittliche Kunde braucht genau eine Lizenz. Nur wenige sehr große Kunden nehmen eine zweite Lizenz dazu. Diese Ausreißer nach oben werden im Szenario bewusst nicht eingepreist und bilden den Puffer. Monatszahler (588 € p.a.) würden den Schnitt leicht anheben, gleichen also den seltenen Rabattfall aus.
- Kundenzahl zum Jahresende 2026 in der Mitte des Ziels aus Abschnitt 7 (fünf bis zehn), also **acht zahlende Kunden**.
- Zielmarkt ist **ausschließlich Österreich**, weil Normdex die ÖNORM M 7140 abbildet. Das Wachstum ist ambitioniert, aber im österreichischen Markt plausibel, getragen von SEO-Kumulation, Newsletter und bezahlter Suche. Die Marktgröße dazu ist in der [[Marktanalyse Österreich]] hergeleitet.
- ARR = Run-Rate zum Jahresende (Kunden × ARPU). Der realisierte Jahresumsatz liegt darunter, weil Kunden über das Jahr erst dazukommen (Ramp).

| Jahr         | Zahlende Kunden (Jahresende) | ARR (Run-Rate, Jahresende) | Umsatz im Jahr (realisiert, geschätzt) |
| ------------ | ---------------------------- | -------------------------- | -------------------------------------- |
| 2026 (heuer) | 8                            | ~3.900 €                   | ~1.200 €                               |
| 2027         | 20                           | ~9.800 €                   | ~6.900 €                               |
| 2028         | 40                           | ~19.600 €                  | ~14.700 €                              |
| 2029         | 65                           | ~31.900 €                  | ~25.500 €                              |
| 2030         | 95                           | ~46.600 €                  | ~39.200 €                              |
| 2031         | 130                          | ~63.700 €                  | ~54.900 €                              |

### 11.4 Langfristperspektive

Der Zielmarkt Österreich ist klar begrenzt. Die [[Marktanalyse Österreich]] beziffert den bedienbaren Markt (SAM) auf grob **~750 Büros** und den engen Haustechnik-Kern (WKO-Berufszweig Installationstechnik) auf **~640 Büros**. Bei erfolgreicher Etablierung als moderne Web-Alternative zur Desktop-Software sind langfristig **~150 bis 300 zahlende Organisationen** realistisch, was bei einer Jahreslizenz pro Kunde einem **ARR von rund 75.000 bis 147.000 €** entspricht. Die theoretische Nischendecke (nahezu vollständige Durchdringung des Kerns) liegt bei rund 200.000 bis 250.000 € ARR und ist gegen den etablierten Anbieter unrealistisch — sie markiert nur die Obergrenze. Wachstum darüber hinaus bräuchte angrenzende Anwendungsfälle (z. B. CO2-Bilanzierung, weitere Normverfahren, Export-Schnittstellen) oder andere Märkte mit eigenen Normen.

Der 5-Jahres-Zielwert von 130 Kunden entspricht rund 20 % des engen Haustechnik-Kerns und markiert damit das obere Ende des Plausiblen; ein konservativerer Pfad bliebe wirtschaftlich weiterhin klar tragfähig.

Wegen der schlanken Kostenbasis wird der Betrieb schon bei sehr wenigen Kunden deckungsbeitragspositiv: Ab rund **drei Jahreskunden** sind die reinen Fixkosten (~1.180 € p.a.) gedeckt, inklusive des vollen Marketing-Budgets (~2.000 € p.a.) ab rund **sieben Jahreskunden**. Ab dieser Schwelle finanziert jeder weitere Kunde überwiegend Reinvestition und Ertrag.

**Wichtiger Vorbehalt:** Diese Zahlen sind ein Zielszenario, kein Versprechen. Der Engpass bleibt (siehe Abschnitt 2) nicht das Produkt, sondern qualifizierter Traffic und Conversion. Ohne funktionierenden Akquise-Motor verschieben sich alle Jahre nach hinten.

## 12. Verwandte Dokumente

- [[Marktanalyse Österreich]]
- [[Produkt-Übersicht]]
- [[Unternehmensangaben]]
- [[Marketingplan 2026 - Erste Kunden]]
- [[Key Messages & CTAs]]
- [[Landingpage]]
- [[Brand Identity & Voice]]
