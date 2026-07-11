# T046 · Static Site Generation der Landingpage mit vite-react-ssg

**Phase:** Landingpage / Technisches SEO / Rendering
**Priorität:** P2 · Größter struktureller SEO-Hebel – mittelfristig
**Status:** offen
**Datum:** 2026-07-10

## Ziel

Die Landingpage (`normdex.at`) so umbauen, dass **jede Route ihren vollständigen sichtbaren Inhalt (Body) bereits im ausgelieferten HTML** enthält — nicht erst nach JavaScript-Ausführung im Browser. Umsetzung via **Static Site Generation (SSG)** mit `vite-react-ssg`. Kein Laufzeit-Server nötig, nginx bleibt Auslieferer statischer Dateien.

## Ausgangslage / Problem (verifiziert)

Die Seite ist aktuell eine **reine Client-Side-App (SPA)**. Das ausgelieferte HTML enthält nur einen leeren Container — der gesamte sichtbare Inhalt wird erst durch React im Browser erzeugt.

Beweis aus dem aktuellen Build (`dist/oenorm-m-7140/index.html`):

```html
<body>
  <div id="root"></div>
</body>
```

Das bestehende Prerender-Script `scripts/generate-route-html.mjs` befüllt **nur den `<head>`** (Title, Meta, Canonical, JSON-LD) pro Route — nicht den Body.

**Warum das ein Problem ist:**
- Google indexiert JS-Content nur in einer zweiten, verzögerten Rendering-Welle (Crawl-Budget-Kosten, langsamere/unzuverlässigere Indexierung für eine junge Domain).
- Alle Consumer ohne JS-Ausführung sehen eine leere Seite: **AI-/Answer-Engines** (ChatGPT-, Perplexity-Crawler etc.), viele Preview-Bots. OG-Unfurls funktionieren zwar (die lesen den `<head>`), aber der Fachinhalt ist unsichtbar.
- Das per Helmet zur Laufzeit injizierte **FAQPage-Schema** (auf `/oenorm-m-7140/`) steht ebenfalls nicht im rohen HTML.

Kontext: Diese Aufgabe ist die im Commit `bee87bb` (siehe [[2026-07-10-gsc-seo-optimierung-landingpage]]) als „größter struktureller Hebel" markierte offene Empfehlung. Die On-Page-SEO-Basis (Canonicals, Marken-Entität, FAQ-Schema) steht bereits; SSG hebt sie ins rohe HTML.

## ⚠️ Vorabklärung (Hard Gate, zwingend zuerst)

Das Repo nutzt laut Build-Output **Vite 8 / rolldown-vite** (`package.json`: `"vite": "^8.0.10"`; Build zeigt `rolldown:vite-resolve`, `[PLUGIN_TIMINGS]`). Vor jeglicher Umsetzung:

1. Prüfen, ob die aktuelle `vite-react-ssg`-Version **Vite 8 / rolldown** unterstützt (README + Peer-Dependencies + Issues).
2. In einem Wegwerf-Branch minimal verifizieren, dass `vite-react-ssg build` mit der bestehenden `vite.config.ts` überhaupt durchläuft.

**Wenn inkompatibel:** Auf **Fallback B (react-snap)** ausweichen (siehe Abschnitt „Fallback") ODER Vite/rolldown-Version für die Landingpage anpassen — Entscheidung dokumentieren, dann diese Aufgabe entsprechend umschreiben. **Nicht** ohne bestätigte Kompatibilität mit dem Hauptumbau beginnen.

## Rahmenbedingungen

- **Eigener Feature-Branch** (z. B. `feat/ssg-vite-react-ssg`), getrennt vom SEO-Commit `bee87bb`.
- **Keine sichtbare Änderung** an Layout, Design, Texten oder Routen-Pfaden. Rein technischer Umbau.
- **Alle bestehenden Routen** bleiben unter identischen Pfaden erreichbar (inkl. Trailing-Slash-Canonicals aus `bee87bb`).
- Backend/`VITE_API_URL` und Docker/nginx-Auslieferung bleiben unverändert.

## Umsetzung (Phasenplan)

### Phase 0 – Vorbereitung
- Feature-Branch anlegen.
- `vite-react-ssg` als Dependency installieren.
- Kompatibilitäts-Gate (oben) bestanden.

### Phase 1 – Layout extrahieren
Die aktuelle `src/App.tsx` umschließt `<Routes>` mit Providern + `Header`/`Footer`/`CookieConsent`/`GoogleAnalytics`/`ScrollToTop`. Diesen Rahmen in eine **Layout-Komponente** überführen, die `<Outlet/>` statt `<Routes>` rendert:

```tsx
// src/Layout.tsx (neu)
import { Outlet } from "react-router-dom";
import { HelmetProvider } from "react-helmet-async";
import { QueryClientProvider, QueryClient } from "@tanstack/react-query";
import { TooltipProvider } from "@/components/ui/tooltip";
import { Toaster } from "@/components/ui/toaster";
import { Toaster as Sonner } from "@/components/ui/sonner";
import { Header } from "@/components/Header";
import { Footer } from "@/components/Footer";
import { ScrollToTop } from "@/components/ScrollToTop";
import CookieConsent from "@/components/CookieConsent";
import GoogleAnalytics from "@/components/GoogleAnalytics";

const queryClient = new QueryClient();

export default function Layout() {
  return (
    <HelmetProvider>
      <QueryClientProvider client={queryClient}>
        <TooltipProvider>
          <Toaster />
          <Sonner />
          <ScrollToTop />
          <Header />
          <Outlet />
          <Footer />
          <CookieConsent />
          <GoogleAnalytics />
        </TooltipProvider>
      </QueryClientProvider>
    </HelmetProvider>
  );
}
```

> Hinweis Provider-Platzierung: Falls `vite-react-ssg` im Router-Modus verlangt, dass App-weite Provider (v. a. `HelmetProvider`) außerhalb des Routers liegen, stattdessen in den Entry-Callback (Phase 3, zweites Argument von `ViteReactSSG`) verschieben. Genaue Vorgabe der installierten Version aus deren README übernehmen.

### Phase 2 – Routen als Array definieren
Die JSX-`<Routes>`/`lazy()`-Definition aus `App.tsx` in ein Routen-Array überführen (Layout als Root-Route mit `children`):

```tsx
// src/routes.tsx (neu)
import type { RouteRecord } from "vite-react-ssg";
import Layout from "./Layout";

export const routes: RouteRecord[] = [
  {
    path: "/",
    element: <Layout />,
    children: [
      { index: true, lazy: () => import("./pages/Index") },
      { path: "preise", lazy: () => import("./pages/Pricing") },
      { path: "kontakt", lazy: () => import("./pages/Contact") },
      { path: "ueber-uns", lazy: () => import("./pages/About") },
      { path: "features", lazy: () => import("./pages/FeaturesPage") },
      { path: "oenorm-m-7140", lazy: () => import("./pages/OenormM7140") },
      { path: "newsletter", lazy: () => import("./pages/Newsletter") },
      { path: "impressum", lazy: () => import("./pages/Impressum") },
      { path: "datenschutz", lazy: () => import("./pages/Datenschutz") },
      { path: "agb", lazy: () => import("./pages/AGB") },
      { path: "cookies", lazy: () => import("./pages/Cookies") },
      { path: "cookie-einstellungen", lazy: () => import("./pages/CookieSettings") },
      { path: "*", lazy: () => import("./pages/NotFound") },
    ],
  },
];
```

> Wichtig – genaue API prüfen: Feldname/Signatur für Code-Splitting (`lazy` vs. `Component`/`element` vs. `entry`) und das erwartete Modul-Export-Format (Default-Export vs. `{ Component }`) **exakt an die installierte `vite-react-ssg`-Version anpassen**. Die Seiten nutzen aktuell `export default` — ggf. auf das von vite-react-ssg erwartete Schema anheben. Ziel bleibt unverändert: jede Route = ein `RouteRecord`, Layout als Root mit `<Outlet/>`.
>
> **Alle 12 Routen** müssen übernommen werden (Abgleich mit `src/lib/seo-routes.json`) plus die Catch-all-Route `*` → `NotFound`.

### Phase 3 – Entry umbauen
`src/main.tsx` von SPA-Mount auf den vite-react-ssg-Entry umstellen:

```tsx
// src/main.tsx
import "./index.css";
import { ViteReactSSG } from "vite-react-ssg";
import { routes } from "./routes";

export const createRoot = ViteReactSSG({ routes });
```

Das alte `createRoot(document.getElementById("root")!).render(<App />)` entfällt. `src/App.tsx` wird durch `Layout.tsx` + `routes.tsx` ersetzt (App.tsx entfernen oder auf Re-Export reduzieren; verwaiste Importe bereinigen).

### Phase 4 – Build-Pipeline umstellen
`package.json`:
- `"build": "vite build && node scripts/generate-route-html.mjs"` → `"build": "vite-react-ssg build"`
- `"postbuild:routes"`-Script entfernen.
- `scripts/generate-route-html.mjs` **löschen** (Head-Erzeugung übernimmt jetzt der Helmet-Render zur SSG-Zeit).
- Neues Script `"verify:prerender": "node scripts/verify-prerender.mjs"` ergänzen (Phase 6).

`vite.config.ts`: bestehende Optionen (`assetsDir: "static_assets"`, Alias `@`, `plugin-react-swc`) beibehalten. Falls `vite-react-ssg` zusätzliche `ssgOptions` benötigt (z. B. `script: "async"`, `formatting`, `dirStyle: "nested"` für Trailing-Slash-Verzeichnisse), dort ergänzen. **`dirStyle`/Ausgabe muss weiterhin `dist/<route>/index.html` erzeugen**, damit die Trailing-Slash-Canonicals aus `bee87bb` passen.

### Phase 5 – Head-Management (react-helmet-async beibehalten)
Die `<Seo>`-Komponente (`react-helmet-async`) und `src/lib/seo.ts` / `seo-routes.json` **bleiben unverändert** und liefern Title/Meta/Canonical/JSON-LD pro Route.
- `HelmetProvider` muss den gerenderten Baum umschließen (Phase 1 / Entry-Callback).
- **Verifizieren**, dass `vite-react-ssg` den Helmet-Head in das statische HTML serialisiert (Phase 6-Check „Head").
- **Fallback, falls Helmet-Head nicht automatisch serialisiert wird:** entweder die Helmet-Integration von vite-react-ssg aktivieren (gemäß deren Doku) **oder** als dokumentierte Übergangslösung ausschließlich für die Head-Injektion ein reduziertes Post-Build-Script behalten. Body-SSG bleibt in jedem Fall erhalten.

`index.html` auf ein **minimales Template** reduzieren: `charset`, `viewport`, `link rel=icon`, `google-site-verification` behalten; die route-spezifischen Tags (Title, Description, Canonical, OG/Twitter, JSON-LD) **entfernen**, da sie nun aus Helmet kommen (verhindert doppelte/veraltete Tags im finalen HTML).

### Phase 6 – Verifikation & Tests (siehe eigener Abschnitt)

## Browser-only-Code Audit (SSG rendert einmalig ohne DOM)

Alles, was `window`/`document`/`localStorage`/`matchMedia`/`IntersectionObserver` **zur Render-Zeit** (nicht in `useEffect`/Event-Handlern) liest, würde den SSG-Build zum Absturz bringen. Bestandsaufnahme:

| Datei | Zugriff | Status |
|---|---|---|
| `components/GoogleAnalytics.tsx` | `window`, `localStorage`, `document` | ✅ nur in `useEffect` → sicher |
| `components/ScrollToTop.tsx` | `window.history`, `scrollTo` | ✅ nur in `useEffect` → sicher |
| `components/Header.tsx` | `window` Scroll-Listener | ✅ nur in `useEffect` → sicher |
| `lib/scroll.ts` | `document`, `window.scrollY` | ✅ nur aus Handlern/Effekten aufgerufen → sicher |
| `hooks/use-mobile.tsx` | `window.matchMedia` | ✅ in `useEffect`, Initialwert `undefined` → sicher (Server rendert „nicht mobil") |
| `components/RevealOnScroll.tsx` | `IntersectionObserver` | ✅ rendert `{children}` immer, togglet nur CSS-Klasse → Inhalt landet im HTML |
| `pages/Contact.tsx` | reCAPTCHA / `grecaptcha` | ⚠️ **prüfen**: Falls `<ReCAPTCHA>` zur Render-Zeit `window` berührt → in client-only-Wrapper (`useEffect`-gated Mount) oder dynamischen Import kapseln |
| `components/NewsletterForm.tsx` | Formular/API | ⚠️ **prüfen** auf Render-Zeit-Browserzugriffe |

**Aufgabe:** Vor dem Build alle `src/**`-Dateien auf Render-Zeit-Browserzugriffe grep-prüfen (`window.`, `document.`, `localStorage`, `matchMedia`, `navigator`) und jeden Treffer bestätigen, dass er in Effekt/Handler liegt. Die ⚠️-Fälle explizit absichern.

## Teststrategie

### 1. Bestehende Unit-Tests (müssen grün bleiben)
`npm test` (Vitest, aktuell 12 Tests). `src/lib/seo.test.ts` prüft weiterhin die SEO-Datenbasis (unabhängig vom Rendering-Weg) — muss ohne Änderung bestehen.

### 2. Neuer Render-Smoke-Test (fängt Render-Zeit-Crashes ab)
Vitest, der **jede Seiten-Komponente** unter statischem Router (`MemoryRouter` + `HelmetProvider` + `QueryClientProvider`) rendert und sicherstellt, dass kein Fehler / kein `window`-Zugriff beim reinen Render auftritt. Vorbild: bestehende `pages/FeaturesPage.test.tsx`. Für alle 12 Routen ergänzen (mind. für `/`, `/oenorm-m-7140`, `/kontakt`, `/newsletter` — die mit Interaktion).

### 3. Neuer Post-Build-Prerender-Check (Kern-Akzeptanz)
Script `scripts/verify-prerender.mjs`, das nach dem Build für definierte Routen das erzeugte `dist/<route>/index.html` liest und **hart fehlschlägt** (Exit ≠ 0), wenn Erwartungen verletzt sind:

```js
// scripts/verify-prerender.mjs (Skizze)
import { readFile } from "node:fs/promises";

const checks = [
  { file: "dist/index.html",
    mustContain: ["Wirtschaftlichkeitsberechnung", 'rel="canonical" href="https://normdex.at/"', '"@type": "Organization"'] },
  { file: "dist/oenorm-m-7140/index.html",
    mustContain: ["ÖNORM M 7140", "Häufige Fragen", "Kapitalwertmethode",
      'href="https://normdex.at/oenorm-m-7140/"', '"@type":"FAQPage"'] },
];

let failed = false;
for (const { file, mustContain } of checks) {
  const html = await readFile(file, "utf8");
  const bodyEmpty = /<div id="root">\s*<\/div>/.test(html);
  if (bodyEmpty) { console.error(`❌ ${file}: <body> ist leer (kein SSG)`); failed = true; }
  for (const needle of mustContain) {
    if (!html.includes(needle)) { console.error(`❌ ${file}: fehlt „${needle}"`); failed = true; }
  }
  if (!failed) console.log(`✅ ${file}`);
}
process.exit(failed ? 1 : 0);
```

Als `npm run verify:prerender` nach dem Build ausführen (lokal + in CI, falls vorhanden).

### 4. Lint
`npm run lint` sauber (keine verwaisten Importe aus dem App.tsx-Umbau).

### 5. Manuelle Verifikation nach Deployment
- `curl https://normdex.at/oenorm-m-7140/` bzw. Browser „Seitenquelltext anzeigen": H1 + FAQ-Text müssen im **rohen HTML** stehen.
- Google **Rich Results Test** für `/oenorm-m-7140/`: FAQPage + Organization werden erkannt.
- Social-Preview (LinkedIn Post Inspector) unverändert korrekt.
- GSC URL-Prüfung („Live testen") zeigt gerenderten Inhalt ohne JS-Abhängigkeit.

## Akzeptanzkriterien

- [ ] Vite-8/rolldown-Kompatibilität von `vite-react-ssg` bestätigt (oder dokumentierter Fallback gewählt).
- [ ] `dist/<route>/index.html` enthält für **alle** 12 Routen den vollständigen Body-Inhalt (H1 + Sektionstexte) im rohen HTML.
- [ ] `<body>` ist bei keiner Route mehr nur `<div id="root"></div>`.
- [ ] Head pro Route korrekt: genau **ein** Title, korrektes Canonical (Trailing-Slash), JSON-LD `@graph` (Organization + WebSite + Seiten-Node) — **keine Duplikate**.
- [ ] FAQPage-Schema steht im rohen HTML von `/oenorm-m-7140/`.
- [ ] `scripts/generate-route-html.mjs` entfernt; `npm run build` = `vite-react-ssg build`.
- [ ] Ausgabepfad weiterhin `dist/<route>/index.html` (Trailing-Slash-kompatibel).
- [ ] `npm test` grün (bestehende 12 + neuer Render-Smoke-Test).
- [ ] `npm run verify:prerender` grün.
- [ ] `npm run lint` sauber.
- [ ] Keine visuelle/funktionale Regression (Navigation, Cookie-Consent, Newsletter-Formular, Kontakt-Formular inkl. reCAPTCHA, Scroll-to-Hash).
- [ ] Docker-Build (`docker build`) läuft durch; nginx liefert alle Routen mit 200 aus.

## Fallback B – react-snap (nur falls vite-react-ssg inkompatibel)

Kein Router-Umbau nötig: `react-snap` läuft **nach** dem regulären Build, rendert die Seiten in Headless-Chromium und schreibt das gerenderte HTML zurück.
- `react-snap` installieren, `"postbuild": "react-snap"` ergänzen, `reactSnap`-Config (Routenliste, `source: "dist"`) setzen.
- Mount in `main.tsx` auf `hydrateRoot` bei vorhandenem Markup umstellen.
- **Nachteil:** Chromium im Build nötig (im `node:22-alpine`-Docker-Image fehlt Chromium → zusätzliche Installation/Anpassung des Dockerfiles erforderlich). Deshalb nur Fallback.

## Rollback

Isolierter Feature-Branch. Bei Problemen Branch verwerfen / Revert — der `main`/`develop`-Zustand (SPA + `generate-route-html.mjs`) bleibt unberührt. Erst nach vollständig grünen Akzeptanzkriterien mergen und deployen.

## Risiken

- **Vite 8 / rolldown ↔ vite-react-ssg** (primär) — Hard Gate vorab.
- **Helmet-Head-Serialisierung** unter vite-react-ssg — Verifikation + dokumentierter Fallback.
- **Render-Zeit-`window`-Zugriff** (v. a. reCAPTCHA in Contact) — Audit + client-only-Wrapper.
- **Hydration-Mismatch** (z. B. `useIsMobile` Initialwert, `RevealOnScroll`-Sichtbarkeit) — Server und erster Client-Render müssen übereinstimmen; im Zweifel Initialzustand server-konform wählen.
- **Build-Dauer** steigt (Rendering pro Route) — akzeptabel bei 12 statischen Routen.

## Notizen / Fortschritt

- 2026-07-10: Todo angelegt als Folge der GSC-/SEO-Optimierung (Commit `bee87bb`, [[2026-07-10-gsc-seo-optimierung-landingpage]]). SSG ist der dort identifizierte „größte strukturelle SEO-Hebel". Ergänzt die Content-/Keyword-Todos [[T042-seo-analyse-landingpage]], [[T043-oenorm-seite-inhaltlicher-ausbau]] und [[T037-fachbeitrag-content-struktur-landingpage]] auf technischer Ebene. Umlaut-/Schreibregeln beachten: [[feedback_german_umlauts]], [[feedback_oenorm_uppercase]].
