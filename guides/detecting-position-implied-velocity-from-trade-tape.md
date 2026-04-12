# Detecting Position-Implied Velocity from the Trade Tape

> A working TypeScript and Python implementation of PIV from the 1¢-delta tracker in market_indicator_history, with the 7-day rolling window and the why-1¢ rationale.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 13 min

---
Position-Implied Velocity is the metric that tells you whether positions are being added, removed, or flipped on a market — *separately from* the headline price. A market can have a flat price and a screaming PIV (lots of two-sided turnover with no net direction), or a moving price and a quiet PIV (one big flow that priced the market and then went home). The two cases require different responses, and the only way to tell them apart is to look at the position substrate.

This guide is the working PIV function, the substrate it reads from, and the rationale for the two magic constants in the formula.

## What PIV Actually Measures

PIV is built on top of a 1¢-delta tracker. Every time a market's mid-price moves by at least 1¢ from its previous tracked level, the tracker writes a row to `market_indicator_history` with the new mid, the timestamp, and a derived `position_intensity` field that estimates how much notional was needed to move the price by that 1¢.

PIV over a window is the rolling sum of position_intensity over that window, divided by the window length. The result is in units of "dollars per day of position activity that the market actually absorbed." It is the closest thing prediction markets have to a real-time tape velocity metric.

The formula:

```
PIV(t, window_days) = Σ(position_intensity_i for i in window) / window_days
```

Where the sum is over every 1¢-delta event within `window_days` of `t`. The 1¢ threshold and the 7-day default window are both deliberate, and the rationale matters for when to deviate.

## Why 1¢ and Not 0.5¢ or 2¢

The 1¢ threshold is the smallest price quantum that Kalshi exposes natively (Kalshi prices are in integer cents, and Polymarket rounds to the same precision after normalization). Going finer than 1¢ would invent precision that the venue does not expose. Going coarser than 1¢ — say, 2¢ — would silently drop signal on the most liquid markets, where 1¢ moves are common and structurally meaningful.

I tried 0.5¢ for a week using interpolated mid-prices and the noise floor swamped the signal. I tried 2¢ for a week and missed several Fed-decision repricings that mattered. 1¢ is the sweet spot.

## Why 7 Days and Not 1d or 30d

The 7-day window is empirically tuned. A 1-day window is too noisy — a single Tuesday morning flow shows up as a screaming PIV that has decayed to nothing by Wednesday. A 30-day window is too smoothed — a fresh repricing that started yesterday is averaged with three weeks of nothing and shows up as a yawn.

7 days is long enough to span a typical news cycle and short enough to detect a fresh trend within a day or two of its onset. For event-window markets (Fed days, earnings releases), I drop the window to 3 days. For long-dated speculative markets (2028 election), I keep it at 7 or even stretch to 14.

## Step 1 — Read from market_indicator_history

The substrate table has the schema:

```sql
CREATE TABLE market_indicator_history (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  ticker text NOT NULL,
  recorded_at timestamptz NOT NULL,
  mid_price numeric(5,4) NOT NULL,           -- 0.0000 to 1.0000
  delta_from_prev numeric(5,4) NOT NULL,     -- signed
  position_intensity numeric(12,2) NOT NULL, -- dollars
  source text NOT NULL                       -- 'warm-cron' | 'on-demand'
);

CREATE INDEX idx_mih_ticker_recorded ON market_indicator_history (ticker, recorded_at DESC);
```

The cron writes a row every time `abs(mid_price - last_recorded_mid) >= 0.01`. Reads use the index for fast time-bound queries.

## Step 2 — TypeScript Implementation

```typescript
interface IndicatorHistoryRow {
  ticker: string
  recordedAt: Date
  midPrice: number          // 0–1
  deltaFromPrev: number     // signed
  positionIntensity: number // dollars
  source: 'warm-cron' | 'on-demand'
}

interface PIVResult {
  ticker: string
  windowDays: number
  totalIntensity: number
  rowCount: number
  piv: number
}

export async function computePositionImpliedVelocity(
  ticker: string,
  fetchHistory: (ticker: string, sinceUnix: number) => Promise<IndicatorHistoryRow[]>,
  opts: { windowDays: number } = { windowDays: 7 },
): Promise<PIVResult | null> {
  const now = Date.now() / 1000
  const sinceUnix = now - opts.windowDays * 86400

  const rows = await fetchHistory(ticker, sinceUnix)

  // Edge case 1 — no history at all
  if (rows.length === 0) return null

  // Edge case 2 — only a single row, cannot establish a flow
  if (rows.length < 2) return null

  // Edge case 3 — all rows older than the window edge (defensive)
  const recentRows = rows.filter(r => r.recordedAt.getTime() / 1000 >= sinceUnix)
  if (recentRows.length < 2) return null

  const totalIntensity = recentRows.reduce(
    (acc, r) => acc + Math.abs(r.positionIntensity),
    0,
  )

  const piv = totalIntensity / opts.windowDays

  return {
    ticker,
    windowDays: opts.windowDays,
    totalIntensity,
    rowCount: recentRows.length,
    piv,
  }
}
```

The fetcher is injected so you can swap between a direct DB call (Drizzle, Prisma, raw `pg`) and an HTTP call to `/api/public/market/[ticker]/history` without changing the function. The absolute value on `positionIntensity` matters: PIV is a *velocity*, not a *direction*. Two large flows in opposite directions still produce a high PIV, and that is the desired behavior.

## Step 3 — Python Implementation

```python
import time
from dataclasses import dataclass
from datetime import datetime
from typing import Callable, Optional

@dataclass
class IndicatorHistoryRow:
    ticker: str
    recorded_at: datetime
    mid_price: float
    delta_from_prev: float
    position_intensity: float
    source: str  # 'warm-cron' | 'on-demand'

@dataclass
class PIVResult:
    ticker: str
    window_days: int
    total_intensity: float
    row_count: int
    piv: float

def compute_position_implied_velocity(
    ticker: str,
    fetch_history: Callable[[str, float], list[IndicatorHistoryRow]],
    window_days: int = 7,
) -> Optional[PIVResult]:
    now = time.time()
    since_unix = now - window_days * 86400

    rows = fetch_history(ticker, since_unix)
    if len(rows) == 0:
        return None
    if len(rows) < 2:
        return None

    recent_rows = [
        r for r in rows
        if r.recorded_at.timestamp() >= since_unix
    ]
    if len(recent_rows) < 2:
        return None

    total_intensity = sum(abs(r.position_intensity) for r in recent_rows)
    piv = total_intensity / window_days

    return PIVResult(
        ticker=ticker,
        window_days=window_days,
        total_intensity=total_intensity,
        row_count=len(recent_rows),
        piv=piv,
    )
```

Same logic, same edge cases. The Python version uses a callable for the fetcher so you can plug in psycopg or HTTPX equivalently.

## Step 4 — Edge Cases You Will Hit

There are five failure modes worth handling:

**1. Brand-new markets — no history rows.**
A market that opened 6 hours ago has at most 5 rows in `market_indicator_history`, and most of them are warmup noise. Return null. PIV is an *established* velocity metric and demands a minimum history depth of ~24 hours. Below that, the only honest answer is "we do not know yet."

**2. Sparse history — fewer than ~5 rows in the window.**
The 1¢ threshold means very stable markets accumulate few rows. A market that has only moved 2¢ over 7 days will have 2 rows, and PIV will be a meaningful but very low number. The function returns it correctly. The caller should interpret a low PIV with a low row count as "this market is asleep" rather than "we have no data" — both are valid signals but they mean different things downstream.

**3. PIV ≈ 0 with a thick orderbook — entry condition for range MM.**
A market with LAS = 200 (deep book) and PIV ≈ 0 (nobody is moving the price) is the textbook range-making setup. The orderbook is thick enough to support quoting, and the absence of position flow means your wide-spread quotes will not get run over. This is the case where PIV ≈ 0 is a *positive* signal, not a defect. Code consumers should not filter out low-PIV markets indiscriminately.

**4. Cron gap.**
The warm-regime cron sometimes misses a 5-10 minute window when the proxy redeploys. During that gap, no rows are written, and PIV computed across the gap is artificially low. The fix is to track the cron health separately and refuse to compute PIV when the gap exceeds 15 minutes inside the window. This is a production concern more than a math concern, but it has bitten me twice.

**5. Stale tracker — the last row is more than 24 hours old.**
A market whose last-tracked row is from 26 hours ago has not moved by 1¢ in 26 hours, which is itself a signal (the market is dormant), but PIV computed naively will look like a normal medium value because the rows from 5 days ago are still in the window. I add a "last_row_age" check to the result and the caller can decide whether to trust the PIV or treat the market as dormant.

## Step 5 — Batch Reads via /api/public/market/[ticker]/history

For cross-market scans, the per-ticker DB query approach is too slow. The faster path is the public history endpoint, which accepts a comma-separated ticker list and returns batched results in a single round-trip:

```typescript
interface HistoryBatchResponse {
  results: Record<string, IndicatorHistoryRow[]>
}

async function fetchHistoryBatch(
  tickers: string[],
  sinceUnix: number,
): Promise<HistoryBatchResponse> {
  const url = new URL('https://simplefunctions.com/api/public/market/history')
  url.searchParams.set('tickers', tickers.join(','))
  url.searchParams.set('since', String(sinceUnix))
  const res = await fetch(url, { headers: { 'accept': 'application/json' } })
  if (!res.ok) throw new Error(`fetchHistoryBatch failed: ${res.status}`)
  return res.json()
}

async function scanByPIV(tickers: string[], minPIV: number = 100): Promise<PIVResult[]> {
  const since = Date.now() / 1000 - 7 * 86400
  const batch = await fetchHistoryBatch(tickers, since)
  const results: PIVResult[] = []

  for (const ticker of tickers) {
    const rows = (batch.results[ticker] ?? []).map(r => ({
      ...r,
      recordedAt: new Date(r.recordedAt),
    }))
    const piv = await computePositionImpliedVelocity(
      ticker,
      async () => rows,
    )
    if (piv !== null && piv.piv >= minPIV) {
      results.push(piv)
    }
  }

  return results.sort((a, b) => b.piv - a.piv)
}
```

The batch endpoint pulls 100 tickers worth of history in one request, which is roughly 50× faster than the per-ticker approach. Hook it into the SimpleFunctions CLI as `sf scan --by-piv desc` and you get a one-line ranker for "where is the position activity right now," which is the question that matters when you are trying to ride a fresh flow rather than fight an established one.

## The Habit Worth Building

Always read PIV alongside CRI. The two metrics answer adjacent but distinct questions. CRI tells you the price has moved meaningfully relative to the time horizon. PIV tells you whether real positions were the cause, or whether the move was a thin-book quote shuffle that nobody actually traded into.

A high CRI + high PIV is a real flow you might want to follow. A high CRI + low PIV is a quote shuffle you should distrust. A low CRI + high PIV is a market where positions are churning without producing price movement — usually a sign of two-sided liquidity provision and a candidate for the range MM strategy. A low CRI + low PIV is a sleeping market.

The four-quadrant view becomes second nature once you have looked at it for a week, and it makes the price-only intuition you started with feel like a single-dimensional shadow of the full picture.