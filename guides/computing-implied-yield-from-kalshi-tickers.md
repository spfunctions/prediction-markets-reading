# Computing Implied Yield from Kalshi Tickers, Step by Step

> A working TypeScript and Python implementation of the implied-yield formula for Kalshi binary contracts, including the τ-days edge case at expiry and a batch wrapper for sf scan.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 9 min

---
Implied yield is the annualized return a Kalshi binary contract pays if held to resolution, expressed as the unit a fixed-income trader would recognize. The formula is two lines of math and forty lines of code, including the edge cases. This guide walks through both.

## The Formula

```
IY = (1 / p) ^ (365 / τ) − 1
```

Where **p** is the current YES price (between 0 and 1) and **τ** is the days remaining until the market resolves. The formula gives you a fractional return — multiply by 100 if you want a percentage.

That is the whole math. The work is in pulling `p` and `τ` cleanly from the Kalshi API and handling the failure modes.

## Step 1 — Pull the Market

Hit the Kalshi REST endpoint:

```bash
GET https://api.elections.kalshi.com/trade-api/v2/markets/{ticker}
```

You get back a market object with the fields you need. The two we care about:

- `yes_bid`, `yes_ask`, or `last_price` — these are integer cents (0 to 100)
- `close_ts` — Unix epoch seconds at which the market resolves

Pick one of the price fields based on what you want IY to represent. If you want IY *for the side you can actually buy at*, use `yes_ask` and divide by 100. If you want a midpoint estimate, average `yes_bid` and `yes_ask` and divide by 100. The midpoint version is what most dashboards display.

## Step 2 — Compute τ

τ is just `close_ts − now` converted from seconds to days:

```
τ_days = (close_ts − now_unix_seconds) / 86400
```

Be careful with the rounding here. You want a *float*, not an integer. A market closing in 30 hours has τ = 1.25, not τ = 1, and the difference matters when IY is exponentiating.

## Step 3 — TypeScript Implementation

```typescript
interface KalshiMarket {
  ticker: string
  yes_bid: number  // cents, 0–100
  yes_ask: number  // cents, 0–100
  last_price: number  // cents, 0–100
  close_ts: number  // unix seconds
}

export function computeImpliedYield(market: KalshiMarket, opts: { side: 'mid' | 'ask' | 'last' } = { side: 'mid' }): number | null {
  const now = Date.now() / 1000
  const tauDays = (market.close_ts - now) / 86400

  // Edge case 1 — market is already past resolution time
  if (tauDays <= 0) return null

  // Edge case 2 — market is so close to expiry that IY explodes
  if (tauDays < 0.25) return null  // less than 6 hours; meaningless

  let priceCents: number
  switch (opts.side) {
    case 'ask': priceCents = market.yes_ask; break
    case 'last': priceCents = market.last_price; break
    case 'mid': priceCents = (market.yes_bid + market.yes_ask) / 2; break
  }
  const p = priceCents / 100

  // Edge case 3 — pinned to either resolution
  if (p <= 0 || p >= 1) return null

  return Math.pow(1 / p, 365 / tauDays) - 1
}
```

This returns a fractional return. Multiply by 100 to display as a percent.

## Step 4 — Python Implementation

The Python version is the same shape:

```python
import time
from typing import Literal, Optional

def compute_implied_yield(
    market: dict,
    side: Literal['mid', 'ask', 'last'] = 'mid',
) -> Optional[float]:
    now = time.time()
    tau_days = (market['close_ts'] - now) / 86400

    if tau_days <= 0:
        return None
    if tau_days < 0.25:  # less than 6 hours
        return None

    if side == 'ask':
        price_cents = market['yes_ask']
    elif side == 'last':
        price_cents = market['last_price']
    else:  # mid
        price_cents = (market['yes_bid'] + market['yes_ask']) / 2

    p = price_cents / 100
    if p <= 0 or p >= 1:
        return None

    return (1 / p) ** (365 / tau_days) - 1
```

## Step 5 — Edge Cases You Will Hit

There are three failure modes that crash a naive implementation. All three are handled in the code above. The reasons matter:

**1. τ ≤ 0 — market is past resolution time but not yet settled.**
This happens routinely when Kalshi has not yet pushed the resolution event but the close_ts has already passed. Treat IY as undefined; the market should be excluded from any scan.

**2. τ < 0.25 days — market is in the final hours.**
Mathematically the formula still works, but the output is a number with twelve digits and zero economic meaning. Pinning the cutoff at 6 hours is a convention I picked because it cleanly separates "still tradable for a thesis" from "settlement noise."

**3. p ≤ 0 or p ≥ 1 — market is pinned.**
A market trading at 100 cents has zero remaining yield. A market trading at 0 cents has infinite yield (and zero practical interest). Both are meaningless from an IY perspective and should be excluded.

A fourth case worth noting: very wide bid-ask spreads. IY based on a midpoint when the bid is 12¢ and the ask is 38¢ is fiction — there is no real trade at that midpoint. If you care about *executable* IY, always pull from `yes_ask` instead of mid.

## Step 6 — Batch Wrapper for sf scan

Once the per-market function works, you can vectorize it across the full Kalshi universe:

```typescript
async function scanByImpliedYield(minIY: number = 0.5): Promise<Array<{ ticker: string; iy: number }>> {
  const allMarkets = await fetchAllActiveKalshiMarkets()
  const results = []
  for (const m of allMarkets) {
    const iy = computeImpliedYield(m, { side: 'ask' })
    if (iy !== null && iy >= minIY) {
      results.push({ ticker: m.ticker, iy })
    }
  }
  return results.sort((a, b) => b.iy - a.iy)
}
```

Run this against a 4000-market universe and you get a sorted list of "tradable IY > 50% annualized" — which is the practical input to a yield-driven scan strategy. Hook it into the SimpleFunctions CLI as `sf scan --by-iy desc` and you have a one-line command for finding bond-adjacent contracts that are paying enough to compete with the rest of your fixed-income stack.

## The Habit Worth Building

Compute IY on every Kalshi contract you look at, even when you do not plan to trade it on yield grounds. The number gives you a sanity check that raw probability cannot: a 32-cent contract with 9 days remaining is *not* a casual position, no matter what your mid-price intuition says. The IY formula is the alarm bell that tells you when your intuition has missed the time axis.