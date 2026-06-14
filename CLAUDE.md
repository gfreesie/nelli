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
- Brand strings live in: `App.tsx`, `IntroScreen.tsx`, `index.html`, `SummaryModal.tsx`, `exportSummary.ts`, `package.json`.
- Summary card is a fixed-width (900px) canvas with **dynamic height** — update `draw()` AND `computeHeight()` together when changing card content.
- Accent: light = celestial blue `#2f6fb0`; dark = gold `#d9a441`. Archetype/donut colors are categorical (leave them).
- Keep this repo on a plain local path (not OneDrive) — cloud sync caused stale/truncated reads.

## Open threads (pick up here)
- [x] Build confirmed green — `tsc -b && vite build` passes clean (only the known >500 kB bundle-size warning). Last verified 2026-06-13; covers favicon/OG image, social meta, light-mode blue retint, AMC/GME tickers.
- [ ] Paste the free FMP key into the app's Stocks tab to enable live stock prices/search.
- [x] **Ticker coverage — live autocomplete shipped.** FMP `/search` hits are ranked (exact-ticker → prefix, with a major-US-exchange boost) and de-duped across exchanges (foreign/ADR twins folded to the US listing) in `searchStocks()`. The picker is now a proper combobox: keyboard nav (↑/↓/Enter/Esc) with a highlighted option, an exchange badge on live hits, and **price-fill on highlight** — one quote per highlighted/added symbol instead of bulk-fetching every hit, far lighter on FMP's free tier. See `CustomPortfolioBuilder.tsx` (`stockSearch` effect, `results` memo, `ensurePrice`/`onSearchKeyDown`) and `searchStocks()` in `prices.ts`. _Live stock search still needs the FMP key pasted in the Stocks tab to exercise end-to-end._
- [x] **GitHub repo renamed** `pathfinder` → `nelli` — remote is now `git@github.com:gfreesie/nelli.git` (old URL auto-redirects). Local folder is still `…\dev\pathfinder`; rename/move it whenever and reopen from the new path (nothing references the absolute path).
- [ ] Deploy (e.g., GitHub Pages) so the OG share image unfurls — update absolute URLs in `index.html` and set Vite `base` if served from a subpath.
- [ ] Accessibility pass (see ROADMAP).
