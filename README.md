# Stock Technical Analyzer

A single-file, no-build web app that analyzes a stock by ticker or company name using four
technical indicators — **RSI**, **MACD**, **ADX**, and **CMF** — and produces a plain-English
summary plus an overall Buy/Sell verdict.

Live demo: _(add the GitHub Pages URL here once deployed)_

## Features

- Look up a stock by ticker (`AAPL`) or company name (`Apple`)
- Computes RSI(14), MACD(12,26,9), ADX(14) with +DI/-DI, and CMF(20) entirely client-side from
  raw daily OHLCV data — no server-side indicator library involved
- Weighted composite score → verdict (Strong Buy → Strong Sell)
- MACD/CMF confirmation check: when price momentum (MACD) and volume-based money flow (CMF)
  disagree, the composite score is damped and a caveat is shown, instead of silently averaging
  the disagreement away
- 6M / 1Y / 2Y lookback selector
- Light/dark theme, responsive layout (desktop + mobile browser)

## Getting started

1. Get a free API key from [Twelve Data](https://twelvedata.com/pricing) (no card required,
   800 requests/day, 8/min — and their free Basic plan provides **real-time US equity data**,
   not delayed).
2. Open `index.html` in any browser (double-click it, or serve it — no build step needed).
3. Paste your API key into the Settings panel (gear icon). It's stored only in that browser's
   `localStorage` and is sent only to Twelve Data — never to any other server.
4. Enter a ticker or company name and click Analyze.

## Architecture

This is intentionally a **static, backend-free app**:

- All logic lives in `index.html` (HTML + CSS + vanilla JS, no framework, no build tooling)
- Twelve Data's API sends `Access-Control-Allow-Origin: *`, so the browser can call it directly
  — no CORS proxy, no server component
- Each visitor brings their own free API key (see [Design decisions](#design-decisions) below
  for why this was chosen over a shared backend key)

This means it can be hosted on any static host (GitHub Pages, Netlify, Vercel, S3, etc.) for
free, with zero ongoing infrastructure to maintain.

## Deploying to GitHub Pages

```bash
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

Then in the repo on GitHub: **Settings → Pages → Source → Deploy from branch → main → / (root)**.
The app will be live at `https://<your-username>.github.io/<repo-name>/`.

## Indicator methodology (for future reference)

- **RSI(14)** — Wilder's smoothing. `<30` oversold (bullish), `>70` overbought (bearish),
  `45–55` neutral, else mild momentum in that direction.
- **MACD(12,26,9)** — standard EMA-based MACD/signal/histogram. Bullish when MACD line is above
  both the signal line and zero; bearish when below both; "improving"/"weakening" in between.
- **ADX(14)** — Wilder's double-smoothed ADX with +DI/-DI. `<20` = "No trend" (direction from
  RSI/MACD/CMF is less trustworthy in this regime), `20–25` emerging, `25–40` strong, `40+` very
  strong. Direction comes from whichever of +DI/-DI is larger.
- **CMF(20)** — Chaikin Money Flow: `Σ(Money Flow Volume, 20 bars) / Σ(Volume, 20 bars)`.
  `≥0.05` buying pressure, `≤-0.05` selling pressure, else "Balanced". This is the only
  volume-based indicator of the four — it's used as a confirmation check against MACD (see below).
- **Composite score** — weighted average (`rsi:1, macd:1.2, adx:1, cmf:0.8`), then damped by the
  MACD/CMF confirmation check: full agreement = no damping, CMF "Balanced" while MACD leans a
  direction = 12% damping ("unconfirmed"), CMF pointing the opposite way from MACD = 30% damping
  ("opposing"). Mapped to a verdict: `≥0.5` Strong Buy, `≥0.15` Buy, `>-0.15` Hold, `>-0.5` Sell,
  else Strong Sell.
- Recommended reading order when interpreting results: **ADX first** (sets the regime/context),
  then **MACD** (trend momentum), then **RSI** (timing within that context), then **CMF**
  (volume confirmation).

## Design decisions

- **Twelve Data over Yahoo Finance-via-CORS-proxy**: the app originally scraped Yahoo Finance
  through free public CORS proxies (corsproxy.io, allorigins.win, thingproxy). All three broke
  or became unreliable within the same week (plan changes, discontinued service, flakiness).
  Twelve Data sends proper CORS headers directly, so no proxy is needed at all.
- **Twelve Data over FMP**: data quality is equivalent between the two (verified by cross-checking
  OHLC values), but FMP gates its technical-indicator endpoints behind a paid plan and has no
  MACD/CMF endpoints anyway, and has no shared demo key for testing. Not a clear win either way,
  so no reason to add the complexity of switching.
- **Bring-your-own API key over a shared backend key**: a shared key would need a real backend
  (GitHub Pages can't run server code or hold secrets), plus it would expose one shared
  800-req/day quota to abuse by any visitor of a public URL. Keeping it backend-free was chosen
  deliberately to keep hosting free and zero-maintenance.
- **No PWA / install support (for now)**: the app already works fine as a responsive mobile
  website; PWA (installable, home-screen icon) was considered and intentionally deferred — see
  Roadmap.

## Roadmap / possible future enhancements

- [ ] Reorder `buildSummary()`'s prose to walk through ADX → MACD → RSI → CMF explicitly
      (currently ADX leads and CMF trails, but RSI/MACD are bundled into one sentence)
- [ ] PWA support (manifest + service worker) for "Add to Home Screen" installability
- [ ] Optional shared-backend-key mode (would require pairing static hosting with a small
      serverless function, e.g. Cloudflare Workers, to hold the secret — considered and deferred,
      see Design decisions)
- [ ] Multi-symbol comparison / watchlist view
- [ ] Persist recent searches
- [ ] Additional indicators (Bollinger Bands, OBV, Stochastic)

## Disclaimer

Educational technical-analysis tool only. Not financial advice. Verify independently before
trading.
