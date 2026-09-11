# Broadcast Studio

An announcement composer for a workplace platform. You build one announcement — image, heading, body, CTA — and it renders on all three screens the product lives on: **mobile, web, and signage (TV)**.

Built for the Freespace UI/UX assignment (brief given 7 Sept 2026).

**Figma file:** https://www.figma.com/design/UwBeREMnMmg8XBLrO2L8kX — output frames for all three screens, the editor UI, and the system board (breakpoint table, palettes, rationale).

## Run it

Open `index.html` in a browser, or `npm start` to serve it locally on :4173. No build step, no dependencies, no bundler — the only network request is the Google Fonts stylesheet.

## Deploy to Cloudflare Pages

It's a static site, so there is nothing to build.

**From the dashboard (auto-deploys on every push)**
1. Cloudflare → Workers & Pages → Create → Pages → Connect to Git, pick this repo (authorise the private repo when prompted).
2. Framework preset **None**, Build command **empty**, Build output directory **`/`**.
3. Save and Deploy. Pushes to `main` redeploy; other branches get preview URLs.

**From the CLI**

```
npx wrangler login
npm run deploy
```

`npm run deploy` is `wrangler pages deploy . --project-name broadcast-studio --branch main`; `npm run preview` publishes to a preview branch instead.

### What ships

| File | Why |
|---|---|
| `index.html` | The whole app — markup, styles, logic |
| `favicon.svg` | Tab icon |
| `_headers` | Cloudflare Pages headers: CSP (allows Google Fonts, inline styles/script, `data:` images for uploads), `nosniff`, `X-Frame-Options: SAMEORIGIN`, referrer and permissions policy, cache rules |
| `wrangler.toml` | `pages_build_output_dir = "."` so Wrangler knows the root is the site |
| `package.json` | The deploy/serve scripts — no dependencies to install |

No environment variables, no secrets, no server. Uploaded images are read with `FileReader` and never leave the browser.

## What the brief asked for, and where it is

| Brief | In the app |
|---|---|
| Image, heading, body, CTA | The four blocks in the left rail, plus an eyebrow for message type |
| Drag or select an image, then stylise it | Image block: six generated art fields, or drag/drop and upload your own. Fit, focal point, scrim, blur |
| Stylise heading / body / CTA | Contextual inspector on the right — type scale, weight, alignment, colour, button style, size, radius, full-width behaviour |
| Output on mobile, web and signage | Top-bar screen switcher. Each is a real device frame with the announcement inside a mock product, not a floating rectangle |
| Announcement as a widget or full screen | Placement switcher: in-app widget vs full screen, for both mobile and web |
| "Not the boring square announcement" | Four layouts (full-bleed hero, split, colour field, ticker strip), a palette per announcement, and a carousel that cross-fades colour as you swipe — the Amazon/Myntra behaviour named in the brief |
| Responsive | One content model, a breakpoint table in the renderer. The size slider scales a step, it never sets a pixel value |

## Decisions worth defending

- **The announcement is authored once, not three times.** Everything screen-specific lives in a breakpoint table (`CFG`) — slot size, safe padding, type steps, minimum tap target. The author gets one size control; the renderer resolves it per screen. (Tesler's Law: the complexity has to live somewhere, so it lives in the system, not the author's head.)
- **Signage is not a big phone.** No input device, viewed from 4–6 metres. So the CTA becomes a QR block with a "scan to open" line, the headline steps up to 104px, and safe padding goes to 84px.
- **Contrast is computed, not hoped for.** The status bar shows live WCAG ratios for the headline and the button against the scrimmed background, flagged against AA 4.5:1 — a white headline over a bright photo is how every announcement tool fails.
- **The inspector only shows the selected block's controls** (Hick's Law), the editor uses the layout people already know from Figma and Canva (Jakob's Law), CTAs are held at ≥44px and go full-width on mobile (Fitts's Law), and one accent per announcement is spent on the button (Von Restorff).
- **Edit where you look.** Headline and body are editable directly on the canvas; every change renders instantly, nothing sits behind a Preview button (Doherty Threshold).

Turn on **Rules** in the top bar to see the responsive decisions annotated on the canvas. **Export** gives the JSON payload, a drop-in embed snippet, and the full design rationale.

## Out of scope, on purpose

Scheduling, audience targeting, approvals and analytics belong to a publishing layer around this composer, not inside it. Image library is generated (CSS/SVG) so the tool runs offline; uploaded images stay local to the browser.
