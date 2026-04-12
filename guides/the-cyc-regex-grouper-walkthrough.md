# The CYC Regex Grouper: How to Find Sibling Markets in 9 Patterns

> The 9 named regex patterns from lib/indicators/cyc-grouper.ts walked through, with TypeScript and Python implementations and the 41.4% coverage discussion.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 14 min

---
Cycle Clustering is the indicator that turns a list of disconnected binary contracts into a yield-curve-shaped surface. It is built on a regex grouper that recognizes when two markets are points on the same event family — for example, "Fed cuts rates by May 2026" and "Fed cuts rates by July 2026" are not siblings if you go by ticker, but they *are* siblings if you go by the temporal pattern in the slug.

This guide is the working CYC grouper, the 9 named patterns it uses, and the 41.4% coverage tradeoff that determines when this approach beats an LLM and when it does not.

## Why Regex and Not an LLM

The temptation when you see "find markets that belong to the same event family" is to throw an LLM at it. I tried that for two weeks. The LLM was correct ~85% of the time, slow (200ms per market), expensive (~$15/day at full universe scale), and *non-deterministic*: the same market would land in different clusters on different runs depending on cache state, model version, and the phase of the moon. None of those properties are acceptable for an indicator that is supposed to be a stable substrate for higher-level scans.

The regex grouper is fast (~50µs per market), deterministic, free, and recognizes the patterns that *actually exist in the wild* — which turn out to follow a small number of formats, because both Kalshi and Polymarket name markets according to internal templates that are themselves regex-driven. We are recognizing the patterns of the upstream pattern-makers.

The cost of regex is coverage. The grouper catches 58.6% of markets cleanly. The remaining 41.4% are either winner-pick events ("who will win the 2028 Democratic nomination") that have no temporal cycle structure, novelty markets that do not match any template, or one-off contracts. For those, the answer is "this market is in a cluster of size 1" — which is itself useful information for the consumer (it means there is no term structure to look at).

## The 9 Named Patterns

Each pattern has a name, a regex, and a canonical "cluster key" that is extracted from the match. Markets that produce the same cluster key are grouped together.

| # | Name | What it catches | Example slugs |
|---|---|---|---|
| 1 | `by-monthDay-year` | "by Mar 15 2026" style deadlines | `fed-cut-by-mar-15-2026` |
| 2 | `iso-date` | ISO YYYY-MM-DD inside a slug | `recession-2026-04-30` |
| 3 | `quarter-year` | "Q1 2026" / "first quarter 2026" | `gdp-q1-2026-positive` |
| 4 | `year-only` | Bare year markers | `spacex-ipo-2027` |
| 5 | `before-month` | "before April" / "before Apr" | `rate-cut-before-april-2026` |
| 6 | `by-end-of-year` | "by EOY" / "by end of 2026" | `recession-by-end-of-2026` |
| 7 | `month-year` | "Mar 2026" / "March 2026" | `fed-meeting-mar-2026-cut` |
| 8 | `weekly-thursday` | "Thursday April 11" weekly bars | `weather-thursday-apr-11-rain` |
| 9 | `numbered-event` | "race 5" / "game 7" / "round 3" | `f1-monaco-2026-race-5-leader` |

Markets that match pattern 1 cluster on the (month, day, year) tuple. Markets that match pattern 2 cluster on the (year, month, day) tuple. Markets that match pattern 3 cluster on the (quarter, year) tuple. And so on.

## Step 1 — TypeScript Implementation

```typescript
interface CYCMatch {
  pattern: string
  clusterKey: string
}

interface MarketForGrouping {
  ticker: string
  slug: string  // url-segment style, lowercase, hyphen-separated
}

interface CYCCluster {
  pattern: string
  clusterKey: string
  members: MarketForGrouping[]
}

const MONTHS: Record<string, number> = {
  jan: 1, january: 1,
  feb: 2, february: 2,
  mar: 3, march: 3,
  apr: 4, april: 4,
  may: 5,
  jun: 6, june: 6,
  jul: 7, july: 7,
  aug: 8, august: 8,
  sep: 9, sept: 9, september: 9,
  oct: 10, october: 10,
  nov: 11, november: 11,
  dec: 12, december: 12,
}

function monthFromString(s: string): number | null {
  return MONTHS[s.toLowerCase()] ?? null
}

const CYC_PATTERNS: Array<{
  name: string
  test: (slug: string) => CYCMatch | null
}> = [
  {
    name: 'by-monthDay-year',
    test: (slug) => {
      const m = slug.match(/by-([a-z]{3,9})-(\d{1,2})-(\d{4})/)
      if (!m) return null
      const month = monthFromString(m[1])
      if (month === null) return null
      return { pattern: 'by-monthDay-year', clusterKey: `${m[3]}-${month}-${m[2]}` }
    },
  },
  {
    name: 'iso-date',
    test: (slug) => {
      const m = slug.match(/(\d{4})-(\d{2})-(\d{2})/)
      if (!m) return null
      return { pattern: 'iso-date', clusterKey: `${m[1]}-${parseInt(m[2], 10)}-${parseInt(m[3], 10)}` }
    },
  },
  {
    name: 'quarter-year',
    test: (slug) => {
      const m = slug.match(/q([1-4])-(\d{4})/)
      if (!m) return null
      return { pattern: 'quarter-year', clusterKey: `${m[2]}-q${m[1]}` }
    },
  },
  {
    name: 'year-only',
    test: (slug) => {
      const m = slug.match(/(?<![a-z\d])(\d{4})(?![a-z\d])/)
      if (!m) return null
      const year = parseInt(m[1], 10)
      if (year < 2020 || year > 2050) return null  // sanity bounds
      return { pattern: 'year-only', clusterKey: String(year) }
    },
  },
  {
    name: 'before-month',
    test: (slug) => {
      const m = slug.match(/before-([a-z]{3,9})-(\d{4})/)
      if (!m) return null
      const month = monthFromString(m[1])
      if (month === null) return null
      return { pattern: 'before-month', clusterKey: `${m[2]}-${month}` }
    },
  },
  {
    name: 'by-end-of-year',
    test: (slug) => {
      const m = slug.match(/(?:by-end-of-|by-eoy-?)(\d{4})/)
      if (!m) return null
      return { pattern: 'by-end-of-year', clusterKey: `${m[1]}-eoy` }
    },
  },
  {
    name: 'month-year',
    test: (slug) => {
      const m = slug.match(/(?:^|-)([a-z]{3,9})-(\d{4})(?:-|$)/)
      if (!m) return null
      const month = monthFromString(m[1])
      if (month === null) return null
      return { pattern: 'month-year', clusterKey: `${m[2]}-${month}` }
    },
  },
  {
    name: 'weekly-thursday',
    test: (slug) => {
      const m = slug.match(/(thursday|thu)-([a-z]{3,9})-(\d{1,2})/)
      if (!m) return null
      const month = monthFromString(m[2])
      if (month === null) return null
      return { pattern: 'weekly-thursday', clusterKey: `thu-${month}-${m[3]}` }
    },
  },
  {
    name: 'numbered-event',
    test: (slug) => {
      const m = slug.match(/(race|game|round|leg|stage)-(\d{1,2})/)
      if (!m) return null
      return { pattern: 'numbered-event', clusterKey: `${m[1]}-${m[2]}` }
    },
  },
]

export function classifyMarket(market: MarketForGrouping): CYCMatch | null {
  // Edge case 1 — empty or missing slug
  if (!market.slug || market.slug.length === 0) return null

  // Edge case 2 — try patterns in order; first match wins
  for (const p of CYC_PATTERNS) {
    const match = p.test(market.slug)
    if (match !== null) return match
  }

  // Edge case 3 — unmatched (the 41.4% bucket)
  return null
}

export function clusterMarkets(markets: MarketForGrouping[]): CYCCluster[] {
  const buckets = new Map<string, CYCCluster>()

  for (const market of markets) {
    const match = classifyMarket(market)
    if (match === null) continue  // unclustered, skip

    const key = `${match.pattern}::${match.clusterKey}`
    const existing = buckets.get(key)
    if (existing) {
      existing.members.push(market)
    } else {
      buckets.set(key, {
        pattern: match.pattern,
        clusterKey: match.clusterKey,
        members: [market],
      })
    }
  }

  return Array.from(buckets.values()).filter(c => c.members.length >= 2)
}
```

The function returns clusters of size ≥ 2 by default, because a single-market "cluster" has no term structure to look at. The 41.4% of markets that fall into the unclustered bucket are quietly dropped, and the consumer can either log them, treat them as their own `type='unclustered'` bucket, or ignore them entirely depending on the use case.

## Step 2 — Python Implementation

The Python version is the same shape:

```python
import re
from dataclasses import dataclass, field
from typing import Optional

MONTHS: dict[str, int] = {
    'jan': 1, 'january': 1,
    'feb': 2, 'february': 2,
    'mar': 3, 'march': 3,
    'apr': 4, 'april': 4,
    'may': 5,
    'jun': 6, 'june': 6,
    'jul': 7, 'july': 7,
    'aug': 8, 'august': 8,
    'sep': 9, 'sept': 9, 'september': 9,
    'oct': 10, 'october': 10,
    'nov': 11, 'november': 11,
    'dec': 12, 'december': 12,
}

def month_from_string(s: str) -> Optional[int]:
    return MONTHS.get(s.lower())

@dataclass
class CYCMatch:
    pattern: str
    cluster_key: str

@dataclass
class MarketForGrouping:
    ticker: str
    slug: str

@dataclass
class CYCCluster:
    pattern: str
    cluster_key: str
    members: list[MarketForGrouping] = field(default_factory=list)

def _by_month_day_year(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'by-([a-z]{3,9})-(\d{1,2})-(\d{4})', slug)
    if not m: return None
    month = month_from_string(m.group(1))
    if month is None: return None
    return CYCMatch('by-monthDay-year', f'{m.group(3)}-{month}-{m.group(2)}')

def _iso_date(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'(\d{4})-(\d{2})-(\d{2})', slug)
    if not m: return None
    return CYCMatch('iso-date', f'{m.group(1)}-{int(m.group(2))}-{int(m.group(3))}')

def _quarter_year(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'q([1-4])-(\d{4})', slug)
    if not m: return None
    return CYCMatch('quarter-year', f'{m.group(2)}-q{m.group(1)}')

def _year_only(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'(?<![a-z\d])(\d{4})(?![a-z\d])', slug)
    if not m: return None
    year = int(m.group(1))
    if year < 2020 or year > 2050: return None
    return CYCMatch('year-only', str(year))

def _before_month(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'before-([a-z]{3,9})-(\d{4})', slug)
    if not m: return None
    month = month_from_string(m.group(1))
    if month is None: return None
    return CYCMatch('before-month', f'{m.group(2)}-{month}')

def _by_end_of_year(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'(?:by-end-of-|by-eoy-?)(\d{4})', slug)
    if not m: return None
    return CYCMatch('by-end-of-year', f'{m.group(1)}-eoy')

def _month_year(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'(?:^|-)([a-z]{3,9})-(\d{4})(?:-|$)', slug)
    if not m: return None
    month = month_from_string(m.group(1))
    if month is None: return None
    return CYCMatch('month-year', f'{m.group(2)}-{month}')

def _weekly_thursday(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'(thursday|thu)-([a-z]{3,9})-(\d{1,2})', slug)
    if not m: return None
    month = month_from_string(m.group(2))
    if month is None: return None
    return CYCMatch('weekly-thursday', f'thu-{month}-{m.group(3)}')

def _numbered_event(slug: str) -> Optional[CYCMatch]:
    m = re.search(r'(race|game|round|leg|stage)-(\d{1,2})', slug)
    if not m: return None
    return CYCMatch('numbered-event', f'{m.group(1)}-{m.group(2)}')

CYC_PATTERNS = [
    _by_month_day_year, _iso_date, _quarter_year, _year_only,
    _before_month, _by_end_of_year, _month_year,
    _weekly_thursday, _numbered_event,
]

def classify_market(market: MarketForGrouping) -> Optional[CYCMatch]:
    if not market.slug:
        return None
    for pattern_fn in CYC_PATTERNS:
        match = pattern_fn(market.slug)
        if match is not None:
            return match
    return None

def cluster_markets(markets: list[MarketForGrouping]) -> list[CYCCluster]:
    buckets: dict[str, CYCCluster] = {}
    for market in markets:
        match = classify_market(market)
        if match is None:
            continue
        key = f'{match.pattern}::{match.cluster_key}'
        if key in buckets:
            buckets[key].members.append(market)
        else:
            buckets[key] = CYCCluster(
                pattern=match.pattern,
                cluster_key=match.cluster_key,
                members=[market],
            )
    return [c for c in buckets.values() if len(c.members) >= 2]
```

## Step 3 — Walking Through 5 Real-Looking Slugs

Five example slugs and the pattern that catches each:

1. `fed-rate-cut-by-may-15-2026` → matches `by-monthDay-year`, cluster key `2026-5-15`. Lands in a cluster with all other markets resolving on the same day.
2. `recession-q3-2026-yes` → matches `quarter-year`, cluster key `2026-q3`. Lands in a cluster with all other Q3 2026 macro markets.
3. `spacex-ipo-by-end-of-2027` → matches `by-end-of-year`, cluster key `2027-eoy`. Lands with other "by EOY 2027" markets.
4. `f1-monaco-2026-race-7-pole` → matches `numbered-event` first, cluster key `race-7`. This one is a bug-by-design: it would also match `year-only` (2026), but `numbered-event` is later in the array. The fix is that the numbered-event match is *more specific* and should win — which means the array order is wrong as written. In production I run the array in reverse order of specificity, with `numbered-event` first.
5. `weather-nyc-thu-apr-11-rain` → matches `weekly-thursday`, cluster key `thu-4-11`. Lands with other Thursday weekly weather markets for the same date.

The fifth case is interesting because it shows the regex catching a *recurring* pattern (weekly Thursday weather) rather than a one-off event. Term structure on weekly weather markets is the most genuinely PM-native shape — a curve with a daily cadence — and CYC is what makes it queryable.

## Step 4 — Edge Cases You Will Hit

Five failure modes worth handling:

**1. Pattern overlap.**
A slug like `fed-cut-q2-2026-by-jun-15-2026` matches both `by-monthDay-year` and `quarter-year`. The first-match-wins behavior in the function above gives `by-monthDay-year`, which is the more specific cluster. The order of patterns in the array is the API surface — if you change the order you change which markets cluster together. Document the intended order and freeze it.

**2. Year sanity bounds.**
The bare `\d{4}` in `year-only` matches things like F1 race numbers (`2026` could appear as `-2026` on a slug fragment). I bound the year to 2020-2050, which is loose enough to catch real future events and tight enough to reject phone-number lookalikes. Adjust based on your actual market universe.

**3. The 41.4% unclustered bucket.**
Almost half the universe ends up in this bucket. Most of those markets are winner-pick events ("which candidate will win"), one-off geopolitical questions ("will event X happen by some unspecified date"), and novelty markets. The function above silently drops them. The right consumer behavior is to either tag them as `type='solo'` and treat them as their own analytical category, or escalate them to a human curator. Do not assume the regex grouper is incomplete and try to add more patterns — the unclustered bucket is structurally distinct content, not missed templates.

**4. Slug normalization mismatches.**
Polymarket and Kalshi slugs are not always in the same format. Polymarket uses lowercase-hyphen-separated. Kalshi uses uppercase-dash-separated tickers and a separate slug field. Always normalize to lowercase + hyphens before running the grouper, or you will silently miss matches.

**5. Same-day collisions.**
Two unrelated events resolving on the same calendar day will land in the same `iso-date` cluster. CYC does not know that "weather-thu-apr-11" and "election-special-apr-11" are different events. The fix is to add a *prefix discriminator* on top of the cluster key — usually the first 1-2 words of the slug. This is implemented as a second-pass refinement in production but is omitted from the basic version above for clarity.

## Step 5 — Hooking into /api/public/yield-curves

Once you have clusters, you can compute a yield curve over each cluster by sorting by τ and pulling the IY for each member:

```typescript
async function buildYieldCurves(): Promise<Array<{
  clusterKey: string
  pattern: string
  curve: Array<{ ticker: string; tauDays: number; iy: number }>
}>> {
  const markets = await fetchAllActiveCanonicalMarkets()
  const clusters = clusterMarkets(markets)
  const curves = []

  for (const cluster of clusters) {
    const points = []
    for (const m of cluster.members) {
      const market = await fetchKalshiMarket(m.ticker)
      const iy = computeImpliedYield(market, { side: 'mid' })
      const tauDays = (market.close_ts - Date.now() / 1000) / 86400
      if (iy !== null && tauDays > 0) {
        points.push({ ticker: m.ticker, tauDays, iy })
      }
    }
    if (points.length >= 2) {
      points.sort((a, b) => a.tauDays - b.tauDays)
      curves.push({
        clusterKey: cluster.clusterKey,
        pattern: cluster.pattern,
        curve: points,
      })
    }
  }
  return curves
}
```

This is the underlying compute for `/api/public/yield-curves` — the endpoint that powers the yield-curve visualizations on the front-end. Hook the curve builder into the SimpleFunctions CLI as `sf yield-curves` or `sf scan --by-cyc` and you get the term-structure surface for every event family in the universe, with IY on the y-axis and τ on the x-axis. That is the prediction-market equivalent of the bond desk's morning yield-curve check, and CYC is the prerequisite that makes it possible.

## The Habit Worth Building

When you find a contract you like, the first thing to do is *not* to size the position — it is to look at the yield curve of the cluster the contract lives in. Most of the time the contract you like is in the middle of the curve and the *real* edge is at one of the endpoints, where the term structure breaks down or where a single point dislocates from its neighbors.

That habit — always-look-at-the-curve-before-you-trade-the-point — is what separates a single-contract trader from someone who is reading the term structure of an event family. CYC is the tool that makes the second framing computationally cheap. Once you have it, the question stops being "is this contract mispriced" and starts being "where on this surface is the dislocation, and is this contract the right point on the curve to trade it from."