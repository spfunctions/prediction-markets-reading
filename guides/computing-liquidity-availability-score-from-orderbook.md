# Computing Liquidity Availability Score from the Orderbook

> A working TypeScript and Python implementation of LAS = (bid_depth + ask_depth) / spread, including the warm-cron coverage caveat and the null-as-signal interpretation.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 13 min

---
Liquidity Availability Score is the metric that decides whether the trade you found in the IY scan is actually a trade or just a number on a screen. Every other indicator describes WHAT to trade. LAS describes WHETHER you can trade it at all. Most strategies die at LAS, and they die silently because the orderbook has been left out of the loop.

This guide is the working LAS function and the warm-cron context that makes it useful.

## The Formula

```
LAS = (bid_depth + ask_depth) / spread
```

Where **bid_depth** is the total notional sitting on the bid side within N price levels of the top, **ask_depth** is the same on the ask side, and **spread** is the difference between best bid and best ask in cents. The numerator is "how much can I actually fill against," the denominator is "how much will it cost me," and the ratio is dimensioned in dollars-of-depth-per-cent-of-spread.

A LAS of 100 means there are roughly 100 dollars of depth available per cent of spread crossed. A LAS of 2 means the orderbook is a museum exhibit and you should not size a position larger than your morning coffee against it.

The scaling is linear by default. Some implementations use `log(depth) / spread` to flatten the right tail. I use linear in production because the failure mode I care about is "no depth," not "too much depth," and linear keeps the no-depth markets clearly separated from everything else.

## Step 1 — Pull the Orderbook

Hit the Kalshi orderbook endpoint:

```bash
GET https://api.elections.kalshi.com/trade-api/v2/markets/{ticker}/orderbook
```

You get back `yes` and `no` arrays. Each entry is a `[price_cents, size]` pair. The structure is symmetric to the Polymarket `/book` endpoint, which returns `bids` and `asks` instead. Both venues converge on the same shape after normalization.

The trap: the warm-regime cron only fetches orderbooks for the top 500 markets by 24h volume. Everything else is null in our snapshot table. If you query our DB for LAS on a random low-volume market, you will get null. This is not a bug — it is the reason LAS exists as a Tier B indicator. Null LAS is the entry condition for the "virgin polymarket" maker strategy, not a defect to fix.

## Step 2 — Compute Depth and Spread

Pick a depth window — I use the top 5 price levels on each side, which is wide enough to capture realistic fill but tight enough to ignore the noisy long tail. Sum the notional within that window, take the spread as best_ask − best_bid, and you have the inputs to the formula.

## Step 3 — TypeScript Implementation

```typescript
type OrderbookLevel = [priceCents: number, size: number]

interface NormalizedOrderbook {
  ticker: string
  bids: OrderbookLevel[]  // sorted DESCENDING by price (best bid first)
  asks: OrderbookLevel[]  // sorted ASCENDING by price (best ask first)
  fetchedAt: number       // unix seconds, for staleness checks
}

interface LASResult {
  ticker: string
  bidDepth: number
  askDepth: number
  spread: number
  las: number
}

function sumDepth(levels: OrderbookLevel[], maxLevels: number = 5): number {
  let total = 0
  for (let i = 0; i < Math.min(maxLevels, levels.length); i++) {
    const [priceCents, size] = levels[i]
    total += (priceCents / 100) * size  // notional in dollars
  }
  return total
}

export function computeLiquidityAvailabilityScore(
  book: NormalizedOrderbook | null,
  opts: { maxLevels: number; maxStalenessSeconds: number } = {
    maxLevels: 5,
    maxStalenessSeconds: 60,
  },
): LASResult | null {
  // Edge case 1 — no orderbook fetched (null is signal, not defect)
  if (book === null) return null

  // Edge case 2 — book is empty or one-sided
  if (book.bids.length === 0 || book.asks.length === 0) return null

  // Edge case 3 — stale data
  const now = Date.now() / 1000
  if (now - book.fetchedAt > opts.maxStalenessSeconds) return null

  const bestBid = book.bids[0][0]
  const bestAsk = book.asks[0][0]

  // Edge case 4 — crossed book (best bid >= best ask), should never happen
  // but does occasionally on stale snapshots
  if (bestBid >= bestAsk) return null

  const spreadCents = bestAsk - bestBid
  if (spreadCents <= 0) return null  // belt and suspenders

  const bidDepth = sumDepth(book.bids, opts.maxLevels)
  const askDepth = sumDepth(book.asks, opts.maxLevels)

  const las = (bidDepth + askDepth) / spreadCents

  return {
    ticker: book.ticker,
    bidDepth,
    askDepth,
    spread: spreadCents,
    las,
  }
}
```

The function returns null on every failure mode, and returns a structured object on success. The structured shape matters because LAS alone is not always the most interesting thing — sometimes you want to look at bid_depth and ask_depth separately to see whether the book is symmetric.

## Step 4 — Python Implementation

```python
import time
from dataclasses import dataclass
from typing import Optional

@dataclass
class NormalizedOrderbook:
    ticker: str
    bids: list[tuple[int, float]]  # [(price_cents, size), ...] desc by price
    asks: list[tuple[int, float]]  # [(price_cents, size), ...] asc by price
    fetched_at: float

@dataclass
class LASResult:
    ticker: str
    bid_depth: float
    ask_depth: float
    spread: int
    las: float

def sum_depth(levels: list[tuple[int, float]], max_levels: int = 5) -> float:
    total = 0.0
    for price_cents, size in levels[:max_levels]:
        total += (price_cents / 100) * size
    return total

def compute_liquidity_availability_score(
    book: Optional[NormalizedOrderbook],
    max_levels: int = 5,
    max_staleness_seconds: int = 60,
) -> Optional[LASResult]:
    if book is None:
        return None
    if not book.bids or not book.asks:
        return None

    now = time.time()
    if now - book.fetched_at > max_staleness_seconds:
        return None

    best_bid = book.bids[0][0]
    best_ask = book.asks[0][0]
    if best_bid >= best_ask:
        return None

    spread = best_ask - best_bid
    if spread <= 0:
        return None

    bid_depth = sum_depth(book.bids, max_levels)
    ask_depth = sum_depth(book.asks, max_levels)
    las = (bid_depth + ask_depth) / spread

    return LASResult(
        ticker=book.ticker,
        bid_depth=bid_depth,
        ask_depth=ask_depth,
        spread=spread,
        las=las,
    )
```

Same edge cases, same return contract.

## Step 5 — Edge Cases You Will Hit

There are five failure modes worth handling explicitly:

**1. Null orderbook — the warm cron has not warmed this market.**
This is the most important case to handle correctly, because it is the *entry condition* for the virgin-polymarket maker strategy. A market with null LAS is one nobody has been looking at, which means you can post a wide quote and there is nobody else racing you to fill the natural buyers when they show up. Treat null as "no data, but the absence is informative." Do not retry the fetch and do not throw — surface the null and let the consumer interpret it.

**2. Empty or one-sided book.**
A book with bids but no asks (or vice versa) is technically a book, but LAS is undefined because spread is undefined. Return null. The trader can then decide whether to post the missing side as a maker.

**3. Stale snapshot.**
An orderbook fetched 10 minutes ago tells you nothing useful about the *current* trading conditions. The 60-second freshness window in the function above is conservative — for slow-moving markets you can stretch it to 5 minutes, for fast-moving Fed-decision markets you should tighten it to 15 seconds.

**4. Crossed book.**
Best bid ≥ best ask should never happen on a healthy venue, but it does happen on stale snapshots where the best bid was filled and the snapshot has not yet refreshed the asks. Return null and let the next snapshot resolve it.

**5. The "thick top, thin tail" book.**
A book with 1000 dollars of depth at the top of book and 12 dollars below it gives a high LAS that is not representative of how the market will behave when you cross. The function above mitigates this by capping at 5 levels, but the right answer for production is to also check the *shape* of the depth (front-loaded vs evenly distributed) and adjust your sizing accordingly. The simple metric tells you "is there depth at all"; the full check tells you "is the depth where you need it."

A sixth case worth mentioning: Polymarket's AMM markets do not have a discrete orderbook. They have a curve. LAS as defined here does not apply. For AMM markets you compute an effective LAS by simulating a fill of size X and measuring the slippage — that is a different function and lives in a different file.

## Step 6 — Batch Wrapper for sf scan

The batch version is straightforward, with the warm-cron caveat front and center:

```typescript
async function scanByLAS(minLAS: number = 50): Promise<LASResult[]> {
  const tickers = await fetchTopVolumeTickers({ topN: 500 })  // warm cohort
  const results: LASResult[] = []

  for (const ticker of tickers) {
    const book = await fetchOrderbook(ticker)
    const las = computeLiquidityAvailabilityScore(book)
    if (las !== null && las.las >= minLAS) {
      results.push(las)
    }
  }

  return results.sort((a, b) => b.las - a.las)
}

// Inverse scan — find the unloved markets where null LAS is the buy signal
async function scanForVirginMarkets(): Promise<string[]> {
  const allMarkets = await fetchAllActiveCanonicalMarkets()
  const warmCohort = new Set(await fetchTopVolumeTickers({ topN: 500 }))
  return allMarkets
    .map(m => m.ticker)
    .filter(ticker => !warmCohort.has(ticker))
}
```

The first function answers "what are the deepest tradable markets right now." The second answers "where is nobody looking, so I can be the only maker." Both are useful. The second is the one most newcomers miss because it requires reframing null as a signal rather than a missing field.

Hook both into the SimpleFunctions CLI as `sf scan --by-las desc` for the warm version and `sf scan --warm <ticker>` for the on-demand single-market refresh. The on-demand version triggers a cold orderbook fetch outside the warm-cron path, which is the right tool for "I want LAS on a market I think is interesting but the warm cron has not noticed yet."

## The Habit Worth Building

Compute LAS *before* you compute IY, not after. The standard newcomer mistake is to find a 60% IY contract, get excited, and only check the orderbook depth as an afterthought. By that time you have already mentally committed to the trade and the depth check becomes a formality you talk yourself past.

The right order is: filter the warm cohort by LAS, then rank the LAS-passing subset by IY, then read the order book one more time before sizing. The LAS filter throws away 90% of the universe up front and saves you from the discovery that your "edge" lives on a market with 8 cents of total depth. Once you have built that habit, the discipline of "is this even tradable" becomes automatic, and the *un*-tradable markets become a separate strategy bucket (the virgin polymarket maker bucket) instead of a source of disappointment.