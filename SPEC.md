# Bloomberg Terminal Clone

## Concept & Vision

A dark, data-dense financial terminal inspired by the iconic Bloomberg Terminal aesthetic. The interface prioritizes information density and rapid data scanning, using a monochromatic green-on-black color scheme with sharp typography. It feels like sitting at a trading desk—serious, professional, and information-rich.

## Design Language

**Aesthetic Direction:** Classic Bloomberg Terminal—dark CRT-style display with phosphor green text, high information density, terminal-like UI patterns.

**Color Palette:**
- Background: `#0a0a0a` (near black)
- Panel Background: `#0d1117` (dark gray)
- Primary Text: `#00ff00` (phosphor green)
- Secondary Text: `#00cc00` (darker green)
- Accent: `#00ff41` (bright green)
- Negative: `#ff4444` (red for losses)
- Positive: `#00ff00` (green for gains)
- Border: `#1a3a1a` (dark green)
- Highlight: `#003300` (selection green)

**Typography:**
- Primary: `IBM Plex Mono` (monospace, terminal feel)
- Fallback: `Consolas, Monaco, monospace`
- Sizes: 11px base for data, 13px for headers, 10px for timestamps

**Spatial System:**
- Dense 2px borders between panels
- 8px padding within panels
- No rounded corners (sharp, technical aesthetic)
- Grid-based layout with fixed-width columns

**Motion Philosophy:**
- Minimal animation—data updates should feel instant
- Subtle blink on new data arrival
- No decorative transitions

**Visual Assets:**
- ASCII-style charts for stock movements
- Box-drawing characters for panel borders
- No icons—text and symbols only

## Layout & Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ BLOOMBERG TERMINAL v1.0          [AI & SEMICONDUCTOR FOCUS]    │
├─────────────────────┬───────────────────────────────────────────┤
│ MARKET INDICES      │ TOP NEWS                                 │
│ ▸ S&P 500    +0.45% │ ▸ NVIDIA reports record earnings...      │
│ ▸ NASDAQ     +0.72% │ ▸ AI chip demand surges...               │
│ ▸ SOX        +1.12% │ ▸ TSMC expands Arizona...                │
├─────────────────────┼───────────────────────────────────────────┤
│ AI STOCKS           │ SEMICONDUCTOR STOCKS                     │
│ NVDA   $875.23 +2.4%│ AAPL   $178.45 +0.3%                     │
│ MSFT   $415.67 +0.8%│ AMD    $165.32 -1.2%                     │
│ GOOGL  $175.89 +1.1%│ INTC   $42.15  -0.5%                     │
│ META   $505.67 +1.5%│ TSMC   $145.67 +2.1%                     │
│ ANTHROPIC    --     │ ASML   $890.12 +1.8%                     │
│ OPENAI       --     │ AMAT   $215.45 +0.9%                     │
├─────────────────────┴───────────────────────────────────────────┤
│ MARKET MOVERS & SENTIMENT                                       │
│ Biggest Gainers: NVDA, AMD, ASML  |  Biggest Losers: INTC, AMD  │
│ VIX: 14.32  |  Fear/Greed: 65 (Greed)                          │
├─────────────────────────────────────────────────────────────────┤
│ [LIVE DATA FEED]  Last updated: 14:32:05 EST                    │
└─────────────────────────────────────────────────────────────────┘
```

**Responsive Strategy:** Fixed minimum width of 1024px. On smaller screens, panels stack vertically with horizontal scroll.

## Features & Interactions

**Core Features:**
1. **Market Indices Panel** - Real-time display of S&P 500, NASDAQ, SOX (semiconductor index)
2. **AI Stocks Panel** - Key AI-related stocks: NVDA, MSFT, GOOGL, META, ANTHROPIC, OPENAI
3. **Semiconductor Stocks Panel** - Key chip stocks: AAPL, AMD, INTC, TSMC, ASML, AMAT
4. **News Feed** - Scrolling financial news related to AI and semiconductors
5. **Market Sentiment** - VIX, fear/greed indicator, top movers

**Data Sources (Public APIs):**
- Stock prices: Yahoo Finance API (via public endpoints)
- News: NewsAPI.org or similar free tier
- Fallback: Realistic sample data if APIs unavailable

**Interactions:**
- Hover on stock row: Highlight with brighter green
- Click on stock: Could expand to show more detail (stretch goal)
- Auto-refresh: Data updates every 60 seconds
- Manual refresh button

**Edge Cases:**
- API failure: Display "DATA UNAVAILABLE" with last known timestamp
- Market closed: Show "MARKET CLOSED" indicator
- Missing data: Display "--" for unavailable values

## Component Inventory

**Header Bar:**
- Terminal title in bright green
- Subtitle showing focus area
- Current date/time display
- States: Normal only

**Data Panel:**
- Box-drawing border (ASCII-style)
- Panel title in bright green
- Data rows with symbol, price, change %
- States: Normal, Loading (blinking "LOADING..."), Error

**Stock Row:**
- Symbol (left-aligned, 8 chars)
- Price (right-aligned, 10 chars)
- Change % with +/- prefix, colored red/green
- States: Normal, Hover (highlighted background)

**News Item:**
- Timestamp
- Headline text
- States: Normal, New (brief blink on update)

**Status Bar:**
- Connection status indicator
- Last update timestamp
- Refresh button

## Technical Approach

**Stack:**
- Single HTML file with embedded CSS and JavaScript
- Vanilla JS (no frameworks)
- Fetch API for data retrieval

**Data Architecture:**
- `window.stockData` - Object storing all stock prices
- `window.newsData` - Array of news articles
- `window.marketIndices` - Index data

**API Strategy:**
- Primary: Yahoo Finance public endpoints
- Secondary: Alpha Vantage demo API
- Fallback: Hardcoded realistic sample data

**Refresh Logic:**
- Initial load fetches all data
- setInterval at 60 seconds for auto-refresh
- Manual refresh button triggers immediate fetch