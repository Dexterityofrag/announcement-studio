# Broadcast Studio

An announcement composer for a workplace platform. You build one announcement — image, heading, body, CTA — and it renders on all three screens the product lives on: **mobile, web, and signage (TV)**.

Built for the Freespace UI/UX assignment (brief given 7 Sept 2026).

**Figma file:** https://www.figma.com/design/UwBeREMnMmg8XBLrO2L8kX — output frames for all three screens, the editor UI, and the system board (breakpoint table, palettes, rationale).

## Run it

Open `public/index.html` in a browser, or `npm start` to serve it locally on :4173. No build step, no dependencies, no bundler — the only network request is the Google Fonts stylesheet.

## Deploy to Cloudflare

Static site, nothing to build. It ships as a **Worker with static assets** — assets-only, so there is no Worker script; Cloudflare just serves `public/`.

**From a connected repo (Workers Builds)**
- Build command: *empty*
- Deploy command: `npm run deploy`
- That runs `wrangler deploy`, which the build environment's token is scoped for.

**From your machine**

```
npx wrangler login
npm run deploy
```

**If you would rather use Cloudflare Pages** — connect the repo, set framework preset **None**, build command **empty**, build output directory **`public`**, and leave the deploy command blank. Pages uploads the directory itself; don't call wrangler from a Pages build.

> Why the first build failed: the deploy command ran `wrangler pages deploy`, which hits the **Pages** API. The token injected into the build container (`CLOUDFLARE_API_TOKEN`) is scoped for Workers, so it came back `Authentication error [code: 10000]` — being account Super Administrator doesn't matter, the token's own permissions do. `wrangler deploy` uses the Workers API instead and goes through. Deploying by hand with your own logged-in token would have worked either way.

### What ships

| File | Why |
|---|---|
| `public/index.html` | The whole app — markup, styles, logic |
| `public/favicon.svg` | Tab icon |
| `public/_headers` | CSP (allows Google Fonts, inline styles/script, `data:` images for uploads), `nosniff`, `X-Frame-Options: SAMEORIGIN`, referrer and permissions policy, cache rules |
| `wrangler.toml` | `[assets] directory = "./public"` — the whole deploy config |
| `package.json` | `deploy` / `dev` / `start` scripts, no dependencies to install |

Keeping the site in `public/` means the README, configs and `.git` are never uploaded. No environment variables, no secrets, no server — uploaded images are read with `FileReader` and never leave the browser.

## Decisions worth defending

- **The announcement is authored once, not three times.** Everything screen-specific lives in a breakpoint table (`CFG`) — slot size, safe padding, type steps, minimum tap target. The author gets one size control; the renderer resolves it per screen. (Tesler's Law: the complexity has to live somewhere, so it lives in the system, not the author's head.)
- **Signage is not a big phone.** No input device, viewed from 4–6 metres. So the CTA becomes a QR block with a "scan to open" line, the headline steps up to 104px, and safe padding goes to 84px.
- **Contrast is computed, not hoped for.** The status bar shows live WCAG ratios for the headline and the button against the scrimmed background, flagged against AA 4.5:1 — a white headline over a bright photo is how every announcement tool fails.
- **The inspector only shows the selected block's controls** (Hick's Law), the editor uses the layout people already know from Figma and Canva (Jakob's Law), CTAs are held at ≥44px and go full-width on mobile (Fitts's Law), and one accent per announcement is spent on the button (Von Restorff).
- **Edit where you look.** Headline and body are editable directly on the canvas; every change renders instantly, nothing sits behind a Preview button (Doherty Threshold).

Turn on **Rules** in the top bar to see the responsive decisions annotated on the canvas. **Export** gives the JSON payload, a drop-in embed snippet, and the full design rationale.

## Out of scope, on purpose

Scheduling, audience targeting, approvals and analytics belong to a publishing layer around this composer, not inside it. Image library is generated (CSS/SVG) so the tool runs offline; uploaded images stay local to the browser.
