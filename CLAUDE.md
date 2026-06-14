# CLAUDE.md — Nelli

Guidance for Claude Code working in this repo.

## What this is
**Nelli: Financial Vision Casting** — a single-page React/TypeScript/Vite investment-discovery tool. Quiz → investor archetype → suggested allocation → live-priced custom portfolio builder → 30-year projection (with an insurance ⇄ SBLOC leverage lever) → shareable, client-named summary card. Brand is a celestial horse nebula; light mode is clean (celestial-blue accent), **Celestial dark** makes the image come alive.

Read **`NELLI-OVERVIEW.md`** for full architecture/features and **`ROADMAP.md`** for the running done/todo punchlist. Keep ROADMAP.md updated as work ships.

## Commands
```bash
npm install
npm run dev      # dev server (http://localhost:5173)
npm run build    # tsc -b && vite build  — ALWAYS run before declaring done
npm run lint
npm run preview
```

## Architecture (quick map)
- `src/App.tsx` — phase state machine (`intro → quiz → results → projection`), theme, progress persistence, custom-portfolio state.
- `src/data/questions.tsx` — quiz questions + `visibleQuestions` conditional logic.
- `src/data/assetUniverse.ts` — curated stock/ETF list (live search covers everything else).
- `src/logic/` — `scoring.ts` (archetypes), `allocation.ts` (7 asset classes, ASSET_META, BREAKDOWNS), `projections.ts` (growth + SBLOC math), `customPortfolio.ts` (holdings model, roll-up, `whereToBuy`), `prices.ts` (live prices), `exportSummary.ts` (text export).
- `src/components/` — `IntroScreen`, `QuestionScreen`, `ResultsScreen`, `ProjectionScreen`, `CustomPortfolioBuilder`, `SummaryModal` (hand-drawn Canvas card), `StarfieldBackground`, `ProgressPath`.

## Live prices (`logic/prices.ts`)
- Crypto: CoinGecko (no key). Metals: gold-api.com (no key).
- Stocks/ETFs + ticker search: **Financial Modeling Prep — needs a free API key**, entered in the builder's Stocks tab, stored as `localStorage['nelli-fmp-key']`. `searchStocks()` resolves any listed ticker.

## Conventions / gotchas
- All `localStorage` keys are `nelli-*`. Never hardcode the FMP key in source (repo is public).
- **Idle auto-reset is kiosk-only.** The 20-min "still there?" warning + wipe arms only with `?kiosk` in the URL (`KIOSK` in `App.tsx`); public sessions persist and are never interrupted. `startOver()` clears `nelli-progress-v1` AND `nelli-custom-portfolio`.
- Brand strings live in: `App.tsx`, `IntroScreen.tsx`, `index.html`, `SummaryModal.tsx`, `exportSummary.ts`, `package.json`.
- Summary card is a fixed-width (900px) canvas with **dynamic height** — update `draw()` AND `computeHeight()` together when changing card content.
- Accent: light = celestial blue `#2f6fb0`; dark = gold `#d9a441`. Archetype/donut colors are categorical (leave them).
- Keep this repo on a plain local path (not OneDrive) — cloud sync caused stale/truncated reads.

## Open threads (pick up here)

**▶ Updated 2026-06-14 — start here next session.** **Nelli is LIVE with valid HTTPS at https://nelli.ssopros.com**, served from the droplet's nginx (docroot `site2`) with a real Let's Encrypt cert. Builds/lints green; committed + pushed (`main`); latest build deployed to the droplet. The GitHub Pages cutover was **abandoned** — the domain owner only has the Squarespace *registrar* login, and DNS is delegated to **Google Cloud DNS** (no access), so the nameserver/record change for Pages wasn't possible. Instead we kept nelli on the droplet (no DNS change needed) and solved HTTPS with **standalone certbot** (Plesk's own LE is blocked by the dead license). Likely next moves: (a) confirm the wordmark star on a real iPhone now the inline-block fix is live; (b) add an HTTP→HTTPS redirect for nelli; (c) `dealtracker`/`telos` still renew via Plesk and will break when their certs expire (Aug 6 / Jul 25 2026) — move them to certbot too or install a valid Plesk license.

- [x] Build confirmed green — `tsc -b && vite build` passes clean (only the known >500 kB bundle-size warning). Last verified 2026-06-14.
- [x] **Logo: Playfair Display wordmark + radial star dotting the i** (header, intro hero, summary card; blue star on light / gold on Celestial-dark). Reusable `components/Wordmark.tsx` + a shared `#nelliStar` radial gradient rendered once via `<NelliStarDefs/>` in `App.tsx` (re-tints with `--star-mid`/`--star-edge`/`--star-glow`). Canvas card draws it via `drawWordmark`/`drawSparkle` in `SummaryModal.tsx`, gated on `document.fonts.load`. Playfair loaded via Google Fonts in `index.html`. Placement: `.nelli-i` is `display:inline-block` (a deterministic 1em box, line-height:1) and `.nelli-star { bottom:.61em; width:.36em }` is anchored to it. **Why inline-block:** as a bare inline element the star's `bottom` reference was the inline box edge, which WebKit/iOS computes differently from Blink under line-height:1 + Playfair's tall metrics — so the star sat at its pre-fix spot on iPhones while looking fine in desktop Chrome. The inline-block box is derived purely from font metrics, so it lands identically in both. (`.61em` replaced the old `.76em`, which was tuned to the bare-inline box.) _Verified correct in Chromium at every size/theme; now live at https://nelli.ssopros.com — confirm on a real iPhone._
- [x] **Mobile builder overlap fixed.** The custom-portfolio modal's `@media (max-width:820px)` rule kept `.cpb-body` as a 1-col *grid*; with both items at `min-height:0` the grid sized both rows to fit the body height, compressing the picker so its stock list + API-key box spilled onto the holdings panel + "How G would roll the dice" CTA. Changed to `.cpb-body { display:block }` so sections stack at full height and the body scrolls. Verified at 375px. _Shipped to the live droplet site alongside the inline-block star fix._
- [ ] Paste the free FMP key into the app's Stocks tab to enable live stock prices/search.
- [x] **Ticker coverage — live autocomplete shipped.** FMP `/search` hits are ranked (exact-ticker → prefix, with a major-US-exchange boost) and de-duped across exchanges (foreign/ADR twins folded to the US listing) in `searchStocks()`. The picker is now a proper combobox: keyboard nav (↑/↓/Enter/Esc) with a highlighted option, an exchange badge on live hits, and **price-fill on highlight** — one quote per highlighted/added symbol instead of bulk-fetching every hit, far lighter on FMP's free tier. See `CustomPortfolioBuilder.tsx` (`stockSearch` effect, `results` memo, `ensurePrice`/`onSearchKeyDown`) and `searchStocks()` in `prices.ts`. _Live stock search still needs the FMP key pasted in the Stocks tab to exercise end-to-end._
- [x] **GitHub repo renamed** `pathfinder` → `nelli` — remote is now `git@github.com:gfreesie/nelli.git` (old URL auto-redirects). Local folder also renamed to `…\dev\nelli`.
- [x] **Hosting — LIVE on the droplet over HTTPS via certbot.** `nelli.ssopros.com` is served by the droplet's nginx (Plesk vhost, docroot `/var/www/vhosts/ssopros.com/site2`), A record still → `64.225.114.22` (**no DNS change**). Plesk's own Let's Encrypt is blocked by the dead license (`Install certificate failure: … your license allows hosting only 0 sites`), so the cert is from **standalone certbot** (installed via EPEL) using the webroot challenge: `certbot certonly --webroot -w /var/www/vhosts/default/htdocs -d nelli.ssopros.com` — that webroot is where Plesk's `location ^~ /.well-known/acme-challenge/` points (in both the :80 and :443 server blocks). The cert is wired in by overwriting the Plesk cert-store file the vhost already references (`/usr/local/psa/var/certificates/scfai2j9hpe663ncHIP19r`, holding fullchain+privkey), so it survives Plesk config regen. Auto-renew: `certbot-renew.timer` (enabled + **active** — had to `enable --now`) → deploy hook `/etc/letsencrypt/renewal-hooks/deploy/nelli-plesk.sh` re-syncs that file and reloads nginx; `certbot renew --dry-run` passes. Deploy = `npm run build`, scp `dist/*` to `site2`, then `chown -R ssopros.com_5rn3fk6ybn:psaserv`. GitHub Pages (`deploy.yml`, `public/CNAME`) stays configured but **unused/vestigial**. TODO: HTTP→HTTPS redirect (currently http also serves the app, no redirect); `dealtracker`/`telos` still renew through Plesk → same license wall when they expire.
- [ ] Accessibility pass (see ROADMAP).
