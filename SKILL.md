---
name: idx-watchlist-pro
description: "Build an Indonesia stock (IDX) buy watchlist using multi-factor analysis: technical (1M/1W/1D), fundamentals, broker activity, and recent news. Produces ranked candidates with entry/stop/target plans and risk notes."
version: 1.0.0
author: Hermes
license: MIT
metadata:
  hermes:
    tags: [idx, saham, watchlist, technical, fundamental, broker-activity, news]
---

# IDX Watchlist Pro

Use this skill when user asks for:
- "watchlist saham besok"
- "saham yang worth it dibeli"
- "analisa teknikal + fundamental + broker + berita"

## Goal

Generate a human-in-the-loop watchlist (not auto-trade) with:
1) Technical analysis for 1 bulan, 1 minggu, 1 hari
2) Fundamental quality/valuation snapshot
3) Broker summary (accumulation/distribution context)
4) News catalyst and headline risk
5) Clear trade plan: buy zone, stop loss, target(s), invalidation

## Data sources

Preferred:
- Price/fundamental: Yahoo Finance (`yfinance`, ticker format `XXXX.JK`)
- Broker activity: Stockbit page `https://stockbit.com/broker-activity` (requires logged-in browser/CDP)
- News: Google News RSS (`https://news.google.com/rss/search?q=<query>&hl=id&gl=ID&ceid=ID:id`)

Fallbacks:
- If broker page unavailable, state explicitly: `Broker summary unavailable right now` and continue with other factors.
- If fundamental fields missing/abnormal, mark as `N/A` (never fabricate).

## Workflow

1. Select liquid candidate universe (e.g., 8–15 IDX tickers).
2. Pull 3–6 months daily OHLC and compute:
   - Return 1D / 1W / 1M
   - RSI(14)
   - SMA20 / SMA50 trend relation
3. Pull fundamentals:
   - PE, PB, ROE, Debt/Equity, Profit Margin, Revenue/Earnings growth, Dividend Yield
4. Pull broker-activity snapshots for 1D / 1W / 1M:
   - Extract net values by ticker (B/M/T suffix)
   - Flag persistence of accumulation across periods
5. Pull top 3–5 latest news per ticker from Google News RSS.
6. Build score (transparent and simple):
   - Technical score
   - Fundamental score
   - Broker persistence score
   - News sentiment score
7. Rank and output top picks + secondary list.
8. Provide Monday execution playbook (entry trigger, risk budget, max positions, staggered entry).

## Scoring suggestion

- Technical (0–6):
  - +1 each for positive 1D/1W/1M return
  - +1 close > SMA20
  - +1 close > SMA50
  - +1 RSI in healthy zone (45–70)

- Fundamental (0–6):
  - +1 PE in reasonable range (0–20)
  - +1 PB < 3 (skip for sectors where PB less meaningful)
  - +1 ROE > 12%
  - +1 Debt/Equity acceptable
  - +1 positive revenue growth
  - +1 positive/solid margin

- Broker (0–3):
  - +1 ticker appears with positive net in 1D
  - +1 in 1W
  - +1 in 1M

- News (0–2):
  - Positive catalyst: +2
  - Mixed/neutral: +1
  - Negative dominant: +0

## Output template

Use this exact structure for user-facing result:

1) Konteks Pasar
2) Watchlist Prioritas (Top Picks)
   - Ticker
   - Technical 1M/1W/1D
   - Fundamental ringkas
   - Broker summary 1D/1W/1M
   - News catalyst/risk
   - Buy zone, stop, target
3) Watchlist Sekunder
4) Yang Perlu Dipantau Saat Pre-open/Senin Pagi
5) Rencana Eksekusi (position sizing & risk cap)
6) Disclaimer

## Mandatory safety notes

- This is analysis, not guaranteed profit.
- Never claim certainty.
- Always include invalidation/stop-loss.
- If user asks execution, require explicit approval before placing any order.

## Pitfalls

- Ticker collision in global news (e.g., symbol overlaps non-ID stocks). Add `"saham Indonesia"` in query.
- Some yfinance fields can be missing or anomalous; do sanity checks and mark `N/A`.
- Broker page direct URL variants may 404; prefer opening from Stream menu when needed.

## Quick command checklist

- Build data set: run a Python script in `/tmp` using yfinance + news RSS.
- Broker snapshot: scrape `stockbit.com/broker-activity` periods `1D/1W/1M`.
- Save report JSON for traceability.
- Return concise recommendation with clear risk boundaries.
