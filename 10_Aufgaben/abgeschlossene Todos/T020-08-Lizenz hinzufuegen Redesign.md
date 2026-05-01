# T020-08 · "Lizenz hinzufügen"-Dialog Redesign + Gutscheincode

**Phase:** 3 (Mittleres Feature)
**Priorität:** P2
**Parent:** [[T020-allgemeine Todos]]

## Beschreibung
Der aktuelle "Lizenz hinzufügen"-Dialog wirkt rudimentär, bietet keine Möglichkeit zur Eingabe eines Gutscheincodes und ist optisch nicht ansprechend. Der Dialog wird visuell modernisiert und um ein Code-Feld erweitert.

## Betroffene Dateien
- Frontend: `apps/frontend/src/pages/Licenses.tsx:235-589` — `BuyDialog`.
- Backend: `apps/api/app/routers/subscriptions.py` — neuer Endpoint `POST /subscriptions/validate-promotion-code`.
- Backend Checkout: `apps/api/app/routers/licenses_v2.py:1204-1235` — `discounts` Parameter erweitern.

## Umsetzung
### UI Redesign
- Klarere Trennung monatlich/jährlich (Reiter mit Preisanzeige).
- Verbesserte Quantity-Steuerung (Stepper, Min/Max, sofortige Preisvorschau).
- Strukturierter Preview-Bereich (Subtotal, Rabatt, Total).

### Gutscheincode
- Eingabefeld "Gutscheincode" mit "Anwenden"-Button.
- Validierung gegen neuen Endpoint, der via `stripe.PromotionCode.list()` prüft.
- Bei Erfolg: Rabatt in Vorschau einrechnen und visuell anzeigen.
- Bei Fehler: Inline-Fehlermeldung.
- Beim Submit: Promotion-Code-ID an `discounts` der Checkout-/Subscription-Anlage durchreichen.

## Akzeptanzkriterien
- [ ] Dialog visuell modernisiert, konsistent mit Brand.
- [ ] Code-Feld vorhanden und funktional.
- [ ] Gültiger Code wird validiert und Rabatt in Vorschau angezeigt.
- [ ] Ungültiger Code zeigt Inline-Fehler.
- [ ] Code wird beim finalen Kauf an Stripe übergeben.

## Verifikation
1. Test-Promotion-Code in Stripe anlegen.
2. Dialog öffnen, Code eingeben → Rabatt in Vorschau.
3. Submit → Stripe-Subscription mit angewandtem Rabatt.
4. Ungültiger Code → klare Fehlermeldung.
