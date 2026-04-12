# Computing Cliff Risk Index from Kalshi Tickers, Step by Step

> A working TypeScript and Python implementation of |Δp/Δt| × τ for Kalshi binary contracts, including the empty-history edge case and a batch wrapper for sf scan.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 11 min

---
Cliff Risk Index measures how loudly a Kalshi binary contract is talking right now, scaled by how much story room is left before it walks off the cliff at resolution. The formula is one line. The work is in pulling a clean Δp from a price history that arrives in 47 different shapes, depending on which endpoint you hit and how old the market is.

This guide is the working version of the function I run on every Kalshi market in the warm universe.

## The Formula

```
CRI = | Δp / Δt | × τ_remaining
```

Where **Δp** is the absolute change in YES price over a recent window, **Δt** is the length of that window in days, and **τ_remaining** is the days until the contract resolves. The first ratio is raw velocity. The τ multiplier deflates near-expiry settlement noise so that an early structural move outranks a late mechanical one.

A 1-cent move with 200 days remaining is a fresh thesis being priced. A 1-cent move with 5 days remaining is a quote refresh. CRI says the first one is 40 times more interesting (200 / 5 = 40), and that ratio is the entire reason the metric exists.

## Step 1 — Pull the Price History

Hit the Kalshi candlestick endpoint. The series you want is hourly bars over the last 24 hours, which gives you ~24 datapoints to work from:

```bash
GET https://api.elections.kalshi.com/trade-api/v2/series/{series}/markets/{ticker}/candlesticks?period_interval=60&start_ts=...&end_ts=...
```

You get back an array of bars, each with `yes_bid`, `yes_ask`, `open_interest`, and `price` (last trade) for the bar's interval. The two we care about are the first bar's mid-price and the last bar's mid-price. Their difference, divided by the elapsed time in days, is Δp/Δt.

The trap here is that the series is sparse for low-volume contracts. You can hit a 24h window and get back 3 bars. You can hit it on a brand-new market and get back zero. Both cases are handled below.

## Step 2 — Compute τ_remaining

Same trick as IY. Pull `close_ts` from the market endpoint, subtract `now`, divide by 86400 to get days. Keep it as a float. A market closing in 30 hours has τ_remaining = 1.25, and the 0.25 part is what makes CRI rank it correctly against a market closing in 18 hours.

## Step 3 — TypeScript Implementation

```typescript
interface CandlestickBar {
  end_period_ts: number  // unix seconds
  yes_bid: { close: number }   // cents, 0–100
  yes_ask: { close: number }   // cents, 0–100
}

interface KalshiMarket {
  ticker: string
  close_ts: number  // unix seconds
}

function midFromBar(bar: CandlestickBar): number {
  return ((bar.yes_bid.close + bar.yes_ask.close) / 2) / 100
}

export function computeCliffRiskIndex(
  market: KalshiMarket,
  bars: CandlestickBar[],
  opts: { windowHours: number } = { windowHours: 24 },
): number | null {
  const now = Date.now() / 1000
  const tauDays = (market.close_ts - now) / 86400

  // Edge case 1 — already past resolution time
  if (tauDays <= 0) return null

  // Edge case 2 — fewer than 2 bars means we cannot compute Δp at all
  if (bars.length < 2) return null

  // Sort by time ascending so the first/last accessors are stable
  const sorted = [...bars].sort((a, b) => a.end_period_ts - b.end_period_ts)
  const first = sorted[0]
  const last = sorted[sorted.length - 1]

  // Edge case 3 — window collapsed to a single instant
  const elapsedDays = (last.end_period_ts - first.end_period_ts) / 86400
  if (elapsedDays <= 0) return null

  const pFirst = midFromBar(first)
  const pLast = midFromBar(last)

  // Edge case 4 — pinned bar (book is one-sided, mid is meaningless)
  if (pFirst <= 0 || pFirst >= 1 || pLast <= 0 || pLast >= 1) return null

  const velocity = Math.abs(pLast - pFirst) / elapsedDays
  return velocity * tauDays
}
```

The function returns CRI as a unit-bearing number (cents per day, multiplied by days remaining), which is dimensionally a fractional move scaled by horizon. Higher is "more active." There is no natural cap.

## Step 4 — Python Implementation

The Python version is the same shape:

```python
import time
from typing import Optional

def mid_from_bar(bar: dict) -> float:
    return ((bar['yes_bid']['close'] + bar['yes_ask']['close']) / 2) / 100

def compute_cliff_risk_index(
    market: dict,
    bars: list[dict],
    window_hours: int = 24,
) -> Optional[float]:
    now = time.time()
    tau_days = (market['close_ts'] - now) / 86400

    if tau_days <= 0:
        return None
    if len(bars) < 2:
        return None

    sorted_bars = sorted(bars, key=lambda b: b['end_period_ts'])
    first = sorted_bars[0]
    last = sorted_bars[-1]

    elapsed_days = (last['end_period_ts'] - first['end_period_ts']) / 86400
    if elapsed_days <= 0:
        return None

    p_first = mid_from_bar(first)
    p_last = mid_from_bar(last)
    if p_first <= 0 or p_first >= 1 or p_last <= 0 or p_last >= 1:
        return None

    velocity = abs(p_last - p_first) / elapsed_days
    return velocity * tau_days
```

Same logic, same edge cases, same return contract. Use whichever one matches your stack.

## Step 5 — Edge Cases You Will Hit

There are four failure modes that crash a naive CRI implementation. All four are handled in the code above. The reasons matter, because skipping any of them poisons your scan output:

**1. Brand-new markets (zero or one bar in the window).**
A market that opened 90 minutes ago has either no candlestick history or a single bar. Both states make Δp uncomputable. Treat CRI as null. Do not fall back to "use the market open price as Δp = 0" — that gives you a CRI of 0 for the most volatile slice of the universe, which is exactly backwards.

**2. Window collapse.**
Sometimes the API returns multiple bars all stamped at the same epoch second (this happens when the warm cron repacks history). The elapsed window is 0 seconds, the velocity is undefined, the function must return null. The check is one line and it has saved me from a divide-by-zero in production at least three times.

**3. Pinned bars.**
When the YES book is one-sided (no bid, only ask, or vice versa), the mid-price computation produces 0 or 1. These are not real prices. Treat any pinned bar as null and skip the computation. The downstream consumer can decide whether to retry on the next cron pass.

**4. Settlement crystallization in the final hours.**
The CRI formula does not break here, but the *interpretation* does. A 16-cent move on the day before resolution is mostly a quote pinning to the actual outcome — it is not new information. I clamp τ_remaining to a minimum of 1.0 day in scan output for this reason: it pulls these contracts out of the "actively deciding" bucket where they do not belong. The clamp is a convention, not a law.

A fifth case worth flagging: the candlestick endpoint sometimes returns bars *outside* the requested window because the warm cron extended the bar boundaries. Always sort by `end_period_ts` and trust the data, not the request.

## Step 6 — Batch Wrapper for sf scan

Once the per-market function works, vectorize it across the warm universe:

```typescript
async function scanByCliffRisk(minCRI: number = 1.0): Promise<Array<{ ticker: string; cri: number }>> {
  const markets = await fetchAllActiveKalshiMarkets()
  const results: Array<{ ticker: string; cri: number }> = []

  for (const m of markets) {
    const bars = await fetchCandlesticks(m.ticker, { intervalMinutes: 60, lookbackHours: 24 })
    const cri = computeCliffRiskIndex(m, bars)
    if (cri !== null && cri >= minCRI) {
      results.push({ ticker: m.ticker, cri })
    }
  }

  return results.sort((a, b) => b.cri - a.cri)
}
```

In practice you do not run this synchronously across 4,000 markets every time you scan — the candlestick fetches dominate the wall clock. The warm-regime cron pre-computes a 24h CRI for the top 500 markets by volume and writes it to `market_regime_snapshots`, and the on-demand scanner reads from that table. Hook the cached version into the SimpleFunctions CLI as `sf scan --by-cri desc` and you get a one-line command for ranking the universe by activity, with the τ scaling already baked in.

If you need fresh CRI on a market that is not in the warm cohort, `sf scan --warm <ticker>` triggers an on-demand candlestick pull and recomputes the value. That is the standard workflow for "I think this contract is active, why is CRI null in the snapshot."

## The Habit Worth Building

Run CRI as a primary filter before you read any orderbook. The metric tells you which contracts are *moving* — and the moving ones are the ones where there is actually a story to read. Most of the time you will look at a contract with a fascinating headline and discover its CRI is 0.2: the headline is real, but the market has already digested it, and there is nothing left to trade. CRI is the alarm bell that tells you when the conversation is happening *now*, not yesterday.

Once you have that habit, the workflow inverts. Instead of "read the news, find the contract," you do "rank by CRI, ask why each top entry is moving." The second workflow is faster, broader, and catches stories your news feed has not surfaced yet. CRI is what makes that inversion possible.