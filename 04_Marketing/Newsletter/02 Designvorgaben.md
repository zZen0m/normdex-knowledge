# Newsletter: Designvorgaben

Diese Datei legt das verbindliche Layout für alle Normdex-Newsletter und automatisierten Marketing-Mails fest. Grundlage ist die in Brevo bestehende und bewährte Vorlage (Template-ID 1, Double-Opt-in-Bestätigung). Jede neue Mail nutzt dasselbe Gerüst, das fertige HTML steht in [[03 Master-Template]].

## Grundprinzip

Eine schmale, zentrierte Karte auf hellem Hintergrund. Dunkler Petrol-Kopf mit Logo, ein pinker Akzent, ruhiger Textkörper, klarer Button, dezenter Footer mit Signatur und Pflichtangaben. Schlicht und seriös, passend zur Zielgruppe.

## Farben

| Element | Farbe |
|---|---|
| Seitenhintergrund | `#FAFAFA` |
| Karte | `#FFFFFF` |
| Kopf-Balken | `#003C3E` (dunkles Petrol) |
| Überschrift (H1) | `#003C3E` |
| Akzent (Linie, Aufzählungspunkt) | `#FF2D58` (Pink) |
| Fließtext | `#282F3A` |
| Gedämpfter Text (Footer, Hinweise) | `#64748B` |
| Rahmen und Trennlinien | `#E0E0E0` |
| Buttontext | `#FAFAFA` |

## Schrift

- Familie: `system-ui, -apple-system, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif`.
- Fließtext 16px, Zeilenhöhe 1.6.
- H1 etwa 22px, fett (700).
- Footer-Text 11 bis 12px.

## Layout

- Karte zentriert, `max-width: 600px`, `border: 1px solid #E0E0E0`, `border-radius: 14px`, Inhalt nicht randlos (Innenabstand etwa 24 bis 32px).
- Kopf-Balken `#003C3E`, linksbündig, mit Logo `https://normdex.at/assets/normdex_logo_horizontal_invert.png`, Breite 190px.
- Unter der H1 eine kurze Akzentlinie in `#FF2D58` (3px hoch, 48px breit, abgerundet).
- Versteckter Preheader direkt nach dem `<body>` (`<span class="preheader">`), Text aus dem jeweiligen Mail-Eintrag.

## Button

- Hintergrund `#003C3E`, Text `#FAFAFA`, `padding: 14px 32px`, `border-radius: 8px`, `font-weight: 700`, zentriert.
- Pro Mail nur ein primärer Button (siehe CTA-Regel in [[01 Wording-Vorgaben]]).

## Footer (Pflicht)

- Signaturzeile mit Icon `https://normdex.at/assets/normdex_icon.png` (28px, `border-radius: 8px`) und Hinweis, warum die Person die Mail erhält.
- Bottom-Zeile mit `© 2026 Normdex™` und Link "Impressum/Datenschutz" auf `https://normdex.at/impressum`.
- **Abmeldelink** über Brevos Unsubscribe-Platzhalter, rechtlich verpflichtend in jeder Marketing-Mail.
- Markenname immer als Normdex™ (mit hochgestelltem TM beim ersten Vorkommen).

## Technische Hinweise

- Tabellenbasiertes HTML für E-Mail-Clients, Inline-Styles bevorzugen.
- Bilder mit `alt`-Text und fester Breite.
- Auf Dark-Mode-Verträglichkeit achten (kein reines Schwarz, definierte Hintergründe).
- Vor Aktivierung Test-Mail an office@normdex.at senden und Darstellung prüfen.

## Verwandte Dokumente

- [[00 Newsletter-Leitfaden]]
- [[01 Wording-Vorgaben]]
- [[03 Master-Template]]
- [[Brand Identity & Voice]]
