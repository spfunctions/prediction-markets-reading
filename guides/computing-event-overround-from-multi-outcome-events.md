# Computing Event Overround Across Sibling Markets

> A working TypeScript and Python implementation of Σpᵢ − 1 across mutually exclusive outcomes, including the sibling-grouping logic and the missing-Other-bucket trap.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 12 min

---
Event overround is the simplest arbitrage signal in prediction markets and the easiest one to compute wrong. The math is one subtraction. The work is in identifying which markets actually belong in the same event — and noticing when one of them is silently missing.

This guide walks through the working version: from the raw `canonical_markets` table to the final EE number, including the two ways the assumption silently fails.

## The Formula

```
EE = Σᵢ pᵢ − 1
```

Where **pᵢ** is the YES price of the i-th mutually exclusive outcome, expressed as a probability between 0 and 1. The sum is the total probability the market is collectively assigning to the event happening at all. Subtract 1 (the true total in any well-formed event) and you get the overround.

The interpretation:

- **EE = 0** — clean market. Outcomes sum to exactly 1.0. No arbitrage from overround alone.
- **EE > 0** — outcomes overstate certainty. Selling YES on every outcome locks in EE per dollar of total notional, modulo fees.
- **EE < 0** — outcomes understate certainty. Buying YES on every outcome locks in |EE| per dollar, modulo fees.

In practice raw EE rarely overcomes fees and slippage. The interesting use is monitoring *changes* in EE on the same event over time, which is how you spot one outcome being aggressively repriced relative to its siblings.

## Step 1 — Identify the Sibling Set

This is the part most implementations get wrong. Two markets are siblings if and only if they belong to the same event and the outcomes are mutually exclusive. In our DB the join key is `venueEventId` for Kalshi (a Kalshi event ticker like `KXPRES-28`) and `eventId` for Polymarket (a Polymarket event slug). Both venues normalize to a canonical `event_id` in the `canonical_markets` table.

You almost never want to group by venue event ticker alone, because the venue groups markets in ways that are too narrow (one event per Fed meeting) or too broad (all 2028 election contracts under one umbrella). Group by canonical event ID and verify the outcome set covers the event.

## Step 2 — TypeScript Implementation

```typescript
interface CanonicalMarket {
  ticker: string
  eventId: string
  outcomeName: string  // e.g., "Newsom", "Harris"
  yesPriceMid: number  // 0–1, already normalized
  closeTs: number      // unix seconds
}

interface EventOverroundResult {
  eventId: string
  outcomes: Array<{ name: string; price: number }>
  sum: number
  ee: number
  hasOtherBucket: boolean
}

export function computeEventOverround(
  markets: CanonicalMarket[],
): EventOverroundResult | null {
  // Edge case 1 — fewer than 2 outcomes is not a multi-outcome event
  if (markets.length < 2) return null

  // Edge case 2 — outcomes must all belong to the same event
  const eventId = markets[0].eventId
  if (!markets.every(m => m.eventId === eventId)) {
    throw new Error(`computeEventOverround: mixed eventIds in input`)
  }

  // Edge case 3 — drop pinned outcomes (book is one-sided)
  const valid = markets.filter(m => m.yesPriceMid > 0 && m.yesPriceMid < 1)
  if (valid.length < 2) return null

  // Edge case 4 — drop expired markets from the sum
  const now = Date.now() / 1000
  const live = valid.filter(m => m.closeTs > now)
  if (live.length < 2) return null

  const outcomes = live.map(m => ({ name: m.outcomeName, price: m.yesPriceMid }))
  const sum = outcomes.reduce((acc, o) => acc + o.price, 0)
  const ee = sum - 1

  // Detect the missing-Other-bucket case
  const hasOtherBucket = live.some(m =>
    /^(other|none of the above|field|any other)$/i.test(m.outcomeName)
  )

  return {
    eventId,
    outcomes,
    sum,
    ee,
    hasOtherBucket,
  }
}

export function groupMarketsByEvent(
  markets: CanonicalMarket[],
): Map<string, CanonicalMarket[]> {
  const byEvent = new Map<string, CanonicalMarket[]>()
  for (const m of markets) {
    const arr = byEvent.get(m.eventId) ?? []
    arr.push(m)
    byEvent.set(m.eventId, arr)
  }
  return byEvent
}
```

The two functions split cleanly: `groupMarketsByEvent` does the indexing, `computeEventOverround` does the math on a single group. That separation lets you cache the grouping and recompute EE on a tick without rescanning the universe.

## Step 3 — Python Implementation

The Python version uses dataclasses and the same logic:

```python
import time
import re
from dataclasses import dataclass
from typing import Optional

@dataclass
class CanonicalMarket:
    ticker: str
    event_id: str
    outcome_name: str
    yes_price_mid: float
    close_ts: float

@dataclass
class EventOverroundResult:
    event_id: str
    outcomes: list[tuple[str, float]]
    sum: float
    ee: float
    has_other_bucket: bool

OTHER_PATTERN = re.compile(r'^(other|none of the above|field|any other)$', re.IGNORECASE)

def compute_event_overround(
    markets: list[CanonicalMarket],
) -> Optional[EventOverroundResult]:
    if len(markets) < 2:
        return None

    event_id = markets[0].event_id
    if not all(m.event_id == event_id for m in markets):
        raise ValueError("compute_event_overround: mixed event_ids in input")

    valid = [m for m in markets if 0 < m.yes_price_mid < 1]
    if len(valid) < 2:
        return None

    now = time.time()
    live = [m for m in valid if m.close_ts > now]
    if len(live) < 2:
        return None

    outcomes = [(m.outcome_name, m.yes_price_mid) for m in live]
    total = sum(p for _, p in outcomes)
    ee = total - 1

    has_other_bucket = any(OTHER_PATTERN.match(name) for name, _ in outcomes)

    return EventOverroundResult(
        event_id=event_id,
        outcomes=outcomes,
        sum=total,
        ee=ee,
        has_other_bucket=has_other_bucket,
    )

def group_markets_by_event(
    markets: list[CanonicalMarket],
) -> dict[str, list[CanonicalMarket]]:
    by_event: dict[str, list[CanonicalMarket]] = {}
    for m in markets:
        by_event.setdefault(m.event_id, []).append(m)
    return by_event
```

Same shape, same edge cases, same return contract.

## Step 4 — Edge Cases You Will Hit

There are five failure modes here, and four of them are silent. The dangerous ones are the silent ones, because the function returns a number that *looks* sensible:

**1. The "Other" bucket is missing from the displayed list.**
This is the most common silent failure. A Polymarket "2028 Democratic nominee" event has six named candidates and an implicit Other. If the API returns only the named six and the sum is 0.94, EE looks negative and arbitrageable. It is not. The true total (including Other) is 1.00, and the 0.06 you would "buy" by going long the named six is the implied price of Other. The `hasOtherBucket` flag in the result is the warning. If it is false on a clearly multi-outcome event, do not trust the EE number — go look at the venue and verify the outcome set.

**2. Outcomes overlap.**
Some venues list "Trump wins" and "Republican wins" as separate markets in the same event. They are not mutually exclusive. The sum looks fine (or even slightly low), but the math is broken. Always verify outcomes are mutually exclusive *before* computing EE — the function above does not catch this case and will silently return a meaningless number.

**3. One outcome is illiquid and pinned.**
A 6-outcome event where one market is at 0.99 (pinned by a single ask) gives a sum > 1 that has nothing to do with overround. The pinned market is not actually trading at 0.99 — there is no real bid there. The `yesPriceMid > 0 && yesPriceMid < 1` filter drops these from the sum, but be aware that a near-pinned market (say 0.97) will pass the filter and still distort the total.

**4. Cross-venue mixing.**
Computing EE across a Kalshi market and a Polymarket market in the "same event" is almost always wrong, because the resolution criteria are subtly different. Stick to within-venue groups unless you have manually verified that the resolution sources match.

**5. Stale data.**
EE on a 30-minute-stale snapshot is a curve through old quotes. The arbitrage you "see" has already evaporated. Always carry a freshness timestamp with the result and reject anything older than your acceptable staleness window (~60 seconds for actively traded events).

## Step 5 — Putting It Together

The full pipeline from canonical markets to EE for every event:

```typescript
async function scanByOverround(minAbsEE: number = 0.02): Promise<EventOverroundResult[]> {
  const markets = await fetchAllCanonicalMarkets({ active: true })
  const grouped = groupMarketsByEvent(markets)
  const results: EventOverroundResult[] = []

  for (const [, group] of grouped) {
    if (group.length < 2) continue
    const result = computeEventOverround(group)
    if (result === null) continue
    if (Math.abs(result.ee) >= minAbsEE) {
      results.push(result)
    }
  }

  return results.sort((a, b) => Math.abs(b.ee) - Math.abs(a.ee))
}
```

Hook the result into the SimpleFunctions CLI as `sf scan --by-overround desc` and you have a one-line ranker for events with the largest absolute overround. Combine with a freshness filter (`--max-age 60s`) and a fee guard (`--min-net 0.01`) to get a list that is actually executable rather than just numerically interesting.

## The Habit Worth Building

Compute EE on every multi-outcome event you watch, but never trade on raw EE alone. Track the *delta* — what was EE on this event yesterday, what is it now, what is the trajectory. A 6-cent overround that holds steady for a week is just the resting state of an illiquid event. A 6-cent overround that opened up overnight from a 1-cent baseline is the market actively repricing one outcome relative to its siblings, and *that* is the actionable signal.

The static reading of EE is a trap that costs newcomers their first week's P&L. The dynamic reading of ΔEE is the actual edge. Compute both, log both, and only pull the trigger when the delta moves and you can name which outcome is doing the moving.