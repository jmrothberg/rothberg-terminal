# JMR's Bloomberg Terminal

A single-file, browser-only clone of the Bloomberg Terminal — phosphor-green monospace, dense data panels, pull-down configuration — powered entirely by **free, public data sources**. No API keys. No backend. No build step.

## ► [LAUNCH THE TERMINAL →](https://jmrothberg.github.io/bloomberg-terminal/)

[![JMR's Bloomberg Terminal](JMRs_Bloomberg.png)](https://jmrothberg.github.io/bloomberg-terminal/)

> Click the screenshot or the launch link above to open the live terminal in your browser. No install, no signup, no keys.

---

```
┌─────────────────────────────────────────────────────────────────┐
│ JMR's BLOOMBERG TERMINAL           [ FREE DATA · CORE MARKETS ] │
├─────────────────────────────┬───────────────────────────────────┤
│ ▸ INDICES ▾                 │ ▸ NEWS · Markets ▾                │
│   ^GSPC   5,423.45  +0.45%  │   12m ago · Reuters               │
│   ^IXIC  17,890.22  +0.72%  │   Fed signals rate hold…          │
│   ^DJI   42,512.10  +0.31%  │                                   │
│   ^VIX      14.32   -2.14%  │   18m ago · CNBC                  │
│                             │   NVIDIA earnings beat…           │
├─────────────────────────────┼───────────────────────────────────┤
│ ▸ WATCHLIST ▾               │ ▸ CRYPTO ▾                        │
│   NVDA    $875.23   +2.4%   │   BTC-USD  $75,496  -0.55%        │
│   MSFT    $415.67   +0.8%   │   ETH-USD   $2,311  -0.34%        │
│   GOOGL   $175.89   +1.1%   │   SOL-USD     $148  +1.82%        │
├─────────────────────────────┼───────────────────────────────────┤
│ ▸ MOVERS & SENTIMENT ▾      │ ▸ COMMODITIES ▾                   │
│   GAINERS: NVDA +2.4%…      │   CL=F  (Oil)    $78.42  +1.2%    │
│   LOSERS:  INTC -1.1%…      │   GC=F  (Gold) $2,412.10 +0.4%    │
│   VIX 14.32  SENTIMENT:     │   SI=F  (Silver)   $28.90 +0.9%   │
│   Greed (67)                │                                   │
└─────────────────────────────┴───────────────────────────────────┘
```

---

## Two differences from the real Bloomberg

1. **All data is free and public.** No Bloomberg subscription. No paid APIs. No keys to obtain. Quotes come from Yahoo Finance (via public CORS proxies), crypto from CoinGecko, forex from open.er-api.com, news from Google News RSS.
2. **Super-easy UI.** Real Bloomberg uses `NVDA <EQUITY> <GO>` keyboard syntax. This clone uses pull-down trays — click a panel title, tap a panel-type chip to change what it shows, type a ticker and press Enter to add it.

---

## Features

### Nine panel types — reassign any panel to any type

| Type | What it shows | Default symbols |
|---|---|---|
| **STOCKS** | Individual equity watchlist | `NVDA MSFT GOOGL META AVGO CRM` |
| **INDICES** | Market indices | `^GSPC ^IXIC ^DJI ^SOX ^VIX ^RUT` |
| **CRYPTO** | Cryptocurrencies | `BTC-USD ETH-USD SOL-USD DOGE-USD` |
| **FOREX** | Currency pairs | `EURUSD=X USDJPY=X GBPUSD=X AUDUSD=X` |
| **COMMODITIES** | Futures contracts | `CL=F GC=F SI=F NG=F HG=F` |
| **TREASURIES** | US bond yields | `^IRX ^FVX ^TNX ^TYX` |
| **NEWS** | Financial headlines by topic | Markets · Tech/AI · Semis · Crypto · Economy · Energy · Politics · World + custom |
| **CALENDAR** | Upcoming US economic releases | Computed (CPI, NFP, FOMC, PCE, GDP…) |
| **MOVERS** | Derived top gainers/losers, VIX, sentiment | From loaded quotes |

### Per-row extras

- **1-day sparklines** — every quote row renders a tiny 64×20 SVG line chart built from the intraday close series. Green if the day is up, red if down.
- **Extended quote columns** — VOL (with K/M/B suffix) plus DAY range and 52W range render under every price.

### Click-a-row drilldown (`[SYM] <GO>`)

Click any equity/index/crypto/commodity/forex/bond row to open a full-screen drilldown:

- **Multi-range chart** — 1D · 5D · 1M · 3M · YTD · 1Y · 5Y, with a dashed `prev close` reference line on the 1D view
- **Fundamentals grid** — P/E (TTM + Forward), Market Cap, EPS, PEG, P/S, Enterprise Value, EV/Rev, Revenue (TTM), Net Income (TTM), 52W High/Low
- **Sector / industry / exchange tags**
- **Per-ticker recent news** — 5 most recent Google News headlines for that company
- **`[×]`** or **`ESC`** closes the drilldown

Data sourced from Yahoo's crumb-free `fundamentals-timeseries` and `v1/search` endpoints (no key required).

### Interactions

- **Click any panel title** → tray slides down in phosphor green
- **Panel-type chips** → reassign that panel to any of the 9 types
- **Type symbol + Enter** → validated against live data and added to the panel
- **Autocomplete** → as you type, the tray suggests matching tickers (via Yahoo search)
- **`[×]` next to a symbol** → remove it
- **Custom news query** → type any search and make your own news topic

### State persistence

Your panel layout, assignments, symbols, and news topic are saved to `localStorage` under `bloomberg_state_v2`. Reload the page — everything stays. Clear the key to reset.

---

## Quick start

### Option A — Live on GitHub Pages (recommended)

Just open the demo link — no install, no setup:

### 🟢 **[► https://jmrothberg.github.io/bloomberg-terminal/ ◄](https://jmrothberg.github.io/bloomberg-terminal/)**

### Option B — Run locally

Clone and open in a browser:

```bash
git clone https://github.com/jmrothberg/bloomberg-terminal.git
cd bloomberg-terminal
open index.html
```

If `file://` gives you CORS trouble in Firefox's strict mode, serve with Python:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/
```

No `npm install` required. `package.json` only exists for an optional Playwright dev dependency.

---

## Keyboard shortcuts

| Key | Action |
|---|---|
| `R` | Refresh all panels |
| `H` | Toggle help |
| `ESC` | Close help / close open tray |
| `Enter` *(in symbol input)* | Add typed symbol |
| `↑` / `↓` *(in symbol input)* | Navigate autocomplete |

---

## Symbol format cheatsheet

The terminal accepts Yahoo Finance ticker syntax across every panel type:

| Asset class | Format | Examples |
|---|---|---|
| US equities | Plain ticker | `NVDA`, `AAPL`, `TSLA`, `BRK-B` |
| Indices | `^` prefix | `^GSPC` (S&P 500), `^IXIC` (NASDAQ), `^DJI` (Dow), `^VIX`, `^RUT` (Russell 2k), `^SOX` (Philadelphia Semi) |
| Crypto | `-USD` suffix | `BTC-USD`, `ETH-USD`, `SOL-USD`, `DOGE-USD`, `ADA-USD` |
| Forex | `=X` suffix | `EURUSD=X`, `USDJPY=X`, `GBPUSD=X`, `AUDUSD=X` |
| Commodity futures | `=F` suffix | `CL=F` (Oil), `GC=F` (Gold), `SI=F` (Silver), `NG=F` (Nat Gas), `HG=F` (Copper) |
| Treasury yields | `^` prefix | `^IRX` (13wk), `^FVX` (5Y), `^TNX` (10Y), `^TYX` (30Y) |
| International stocks | `.XX` suffix | `TSM`, `ASML.AS`, `7203.T` (Toyota, Tokyo) |

Don't know the ticker? Start typing the company name — autocomplete will suggest matches.

---

## Data sources

Every number on the screen is from one of these free, public sources. No API key is required for any of them.

| Panel | Primary source | Fallback | Why |
|---|---|---|---|
| Crypto | [CoinGecko public API](https://www.coingecko.com/en/api) | Yahoo via proxy | CoinGecko is CORS-open — works without a proxy |
| Forex | [open.er-api.com](https://open.er-api.com) | Yahoo via proxy | Also CORS-open |
| Stocks / Indices / Commodities / Bonds | [Yahoo Finance chart endpoint](https://query1.finance.yahoo.com/v8/finance/chart/) | Proxy rotation: allorigins → codetabs → corsproxy | Yahoo has no CORS header, so a public proxy is needed |
| Ticker search / autocomplete | Yahoo Finance search endpoint (via proxy) | — | |
| News | [Google News RSS](https://news.google.com/rss/search) (via proxy) | — | RSS parsed with `DOMParser` |
| Economic calendar | Computed from US release schedule | — | CPI, NFP, FOMC 2026, PCE, GDP etc. derived from date math + known FOMC dates |
| Movers / VIX / Fear-Greed | Derived locally from loaded quotes | — | No network call — free reduction |

### CORS proxies tried in order

The app rotates through these public proxies on failure:

1. `api.allorigins.win/raw`
2. `api.codetabs.com/v1/proxy`
3. `corsproxy.io`

Whichever one succeeds becomes the preferred proxy for subsequent requests in that session.

---

## Architecture

**Single HTML file. Vanilla JS. Zero build step.**

- **UI:** ~400 lines of CSS (phosphor-on-black, IBM Plex Mono), ~80 lines of HTML shell
- **Logic:** ~1000 lines of vanilla JavaScript, no framework
- **State model:**

```js
{
  version: 2,
  panels: {
    p1: { type: 'indices',     symbols: ['^GSPC','^IXIC',…] },
    p2: { type: 'news',        topic: 'market' },
    p3: { type: 'stocks',      symbols: ['NVDA',…] },
    p4: { type: 'crypto',      symbols: ['BTC-USD',…] },
    p5: { type: 'movers' },
    p6: { type: 'commodities', symbols: ['CL=F',…] }
  },
  layout: ['p1','p2','p3','p4','p5','p6'],
  config: { refreshInterval: 60000, newsCount: 10 }
}
```

- **Caching:** runtime `quoteCache` and `newsCache` (TTL-guarded) to avoid hammering proxies on tray toggles.
- **Refresh:** auto-refresh every 60s (configurable), plus manual `[R]` or `[REFRESH]` button.

---

## Known limitations

This is an honest list. The whole point of the project is that everything's free — and free things have costs of their own.

1. **Public CORS proxies are rate-limited.** If `allorigins` is having a bad day, quote and news panels may show `DATA UNAVAILABLE`. The app tries multiple proxies in sequence and caches stale values (dimmed green) rather than showing `--`. Press `R` to retry.
2. **Fundamentals coverage is partial.** Yahoo locked `quoteSummary` behind a cookie+crumb token that a browser-only app can't produce, so the drilldown uses `fundamentals-timeseries` instead. That gives P/E, forward P/E, EPS, market cap, PEG, enterprise value, revenue, and net income — but not beta, dividend yield, price-to-book, next earnings date, or company description. Missing cells show `—`.
3. **Yahoo Finance can throttle aggressive clients.** If you spam the refresh button, you may see temporary blocks. The default 60s refresh interval is already well inside Yahoo's tolerance.
4. **Crypto and forex always work.** CoinGecko and open.er-api.com are CORS-open, so those panels are immune to proxy issues.
5. **Economic calendar uses heuristics.** CPI is assumed mid-month, NFP is first-Friday, FOMC dates are hardcoded for 2026. These rules are accurate most of the time but not authoritative — check the BLS / Fed sites for the final release schedule.
6. **No options chains, no order entry, no intraday charts.** This is a dashboard, not a trading platform.
7. **Single HTML file deliberately.** Simplicity > features. No bundler, no package.json to worry about, just open the file.

---

## Troubleshooting

### "DATA UNAVAILABLE" on stock/indices panels
Public CORS proxies may be down or rate-limiting. Press `R` a few times — the app will try the next proxy in its rotation. If it persists, wait a minute — rate limits usually clear fast.

### News panel is empty
Same cause — Google News RSS has to go through a proxy. Try a different news topic chip, or press `R`.

### Crypto panel shows prices but stocks don't
This is **expected** when CORS proxies are all down. Crypto uses CoinGecko directly (CORS-open), which doesn't need a proxy. Stocks require a proxy because Yahoo doesn't serve CORS headers.

### I want to reset to defaults
In DevTools console:
```js
localStorage.removeItem('bloomberg_state_v2');
location.reload();
```

---

## Project structure

```
bloomberg-terminal/
├── index.html        # The entire app (HTML + CSS + JS in one file)
├── SPEC.md           # Original design spec (aesthetic, layout, color palette)
├── README.md         # This file
├── package.json      # Optional Playwright dev dependency for future tests
└── .gitignore
```

---

## Credits & inspiration

- **Visual design:** Bloomberg Terminal (the real one) — phosphor green on CRT black is the canonical financial terminal aesthetic.
- **Font:** [IBM Plex Mono](https://www.ibm.com/plex/) — hand-designed for dense technical data.
- **Data:** Yahoo Finance (unofficial public endpoints), CoinGecko, open.er-api.com, Google News, BLS/Fed/BEA release schedules.
- **CORS bridging:** the maintainers of allorigins.win, codetabs, and corsproxy.io — free infrastructure that makes single-HTML-file dashboards possible.

---

## License

MIT — do whatever you want with this code. But please don't claim you're affiliated with Bloomberg L.P.; this is a lookalike, not a product.
