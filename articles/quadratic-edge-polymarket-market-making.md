# The Quadratic Edge: Why Tighter Beats Bigger in Polymarket Market Making

> Polymarket scores market makers with a quadratic function. Most people miss what this means: being 2x closer to the midpoint is worth 4x the score. The optimal strategy is not what you think.

**Category:** sports | **Author:** SimpleFunctions | **Reading time:** 10 min | **Published:** 2026-04-05

---
Polymarket's liquidity reward program uses a quadratic scoring function. Most market makers understand this intellectually but fail to internalize what it means for their strategy. The result: they leave money on the table by quoting too wide or by putting too much capital into size instead of tightness.

## The formula

Every minute, Polymarket samples your resting orders and scores each one:

```
S(v, s) = ((v - s) / v)²

v = max_incentive_spread (set per market, typically 3-5 cents)
s = distance from the size-cutoff-adjusted midpoint
```

The total score for a sample is: S × order_size. Your reward share is your total score divided by all market makers' total scores.

## Why quadratic matters

Linear scoring would mean: 2x tighter = 2x the score. Quadratic means: 2x tighter = 4x the score.

Here are the actual numbers for a market with v = 3 cents:

| Distance from mid | Score per contract | Relative to 1 cent |
|---|---|---|
| 0 cents (at mid) | 1.000 | 2.25x |
| 0.5 cents | 0.694 | 1.56x |
| 1 cent | 0.444 | 1.00x (baseline) |
| 1.5 cents | 0.250 | 0.56x |
| 2 cents | 0.111 | 0.25x |
| 2.5 cents | 0.028 | 0.06x |
| 3 cents (at v) | 0.000 | 0.00x |

The steepness is extreme. An order at 2 cents from mid scores 4x less than an order at 1 cent. An order at 2.5 cents is essentially worthless — it scores 16x less than one at 1 cent.

## The counterintuitive implication

Most people's instinct: "I have $500 in capital. I should quote 500 contracts at a comfortable spread."

The quadratic-optimal strategy: "I should quote the minimum qualifying size at the tightest possible spread. 100 contracts at 1 cent from mid scores more than 400 contracts at 2 cents."

The math:

```
Option A: 100 contracts at 1 cent (s=0.01)
  Score = 0.444 × 100 = 44.4

Option B: 400 contracts at 2 cents (s=0.02)
  Score = 0.111 × 400 = 44.4

Same score. But Option A uses 4x less capital.
```

At equal capital, tighter always wins. Spare capital should go to quoting additional markets, not to increasing size on one market.

## The adjusted midpoint trap

Your orders are scored against the "size-cutoff-adjusted midpoint" — not the raw orderbook midpoint. The adjusted midpoint filters out orders below the market's min_incentive_size before computing the mid.

This matters because small orders (say, 10 contracts from a retail trader) near the mid don't count. The adjusted midpoint might be 1-2 cents different from what you see on the raw orderbook. If you blindly quote at raw_mid ± 1 tick, you might actually be 2-3 ticks from the adjusted mid. That's a 4-9x score reduction.

To compute the adjusted midpoint yourself:

```python
from sfmm.core.scoring import adjusted_midpoint

# Filter the book, then compute mid
adj_mid = adjusted_midpoint(
    bids=[(0.54, 200), (0.53, 100), (0.52, 50)],
    asks=[(0.56, 150), (0.57, 100)],
    min_size=50,  # market's min_incentive_size
)
# This is the midpoint Polymarket scores you against
```

## Two-sided balance: the Q_min bottleneck

Your score isn't the sum of both sides. It's the minimum:

```
Q_min = max(min(Q_one, Q_two), max(Q_one, Q_two) / 3)
```

If your bid side scores 200 and your ask side scores 50, your Q_min is 66.7 (the larger side divided by 3, which is better than the raw minimum of 50). But you're still leaving 133.3 points on the table compared to having both sides at 200.

The optimization target is not "maximize total score" but "maximize the minimum of the two sides." In practice this means keeping bid and ask sizes roughly balanced. If you skew sizes to express a view, keep the ratio within 2:1 at most.

## Prices for score, sizes for view

This leads to the core principle of quadratic-optimized market making:

**Never move prices to express a directional view. Always move sizes.**

If you think a market is underpriced (fair value > midpoint), the temptation is to bid higher. But bidding 2 cents from mid instead of 1 cent costs you 4x the score. Instead:

```
Bid price = mid - 1 tick    (always, regardless of view)
Ask price = mid + 1 tick    (always, regardless of view)

Bid size = base × (1 + skew)   (bigger if bullish)
Ask size = base × (1 - skew)   (smaller if bullish)
```

Your score stays near-maximum (both sides are 1 tick from mid). Your inventory naturally drifts toward your view (more bids getting filled than asks). And the Q_min stays balanced because the price-based scores are symmetric.

## Practical implementation

The [sfmm](https://github.com/spfunctions/polymarket-sports-mm) package implements this optimization:

```bash
pip install sfmm
sfmm run --dry-run   # See the computed quotes without placing orders
```

The quote engine always places orders at mid ± 1 tick and adjusts sizes based on a blend of directional view (70%) and inventory mean-reversion (30%). Both sides are always above min_incentive_size to preserve the 3x two-sided bonus.

## The uptime dimension

One more thing quadratic scoring implies: **every minute matters equally.** 10,080 samples per epoch, each weighted the same. Missing 1 hour = losing 0.6% of weekly revenue. Missing a day = losing 14%.

The most common failure mode isn't bad quoting — it's downtime. A bot that quotes at 2 cents from mid but runs 24/7 will outperform one that quotes at 0 cents from mid but crashes for 6 hours a day.

Reliability is the unsexy edge.