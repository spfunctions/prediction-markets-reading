# Event Overround: Multi-Outcome Arbitrage in Practice

> EE = Σpᵢ − 1. The borrowed term from sports betting, the Newsom/Harris/Buttigieg example walked through fully, and the two ways the assumption silently fails.

**Category:** indicator | **Author:** Patrick Liu | **Reading time:** 13 min

---
Multi-outcome prediction markets — the kind where five candidates fight for one nomination, or three teams compete for one championship — have a structural property that single binary contracts do not. The YES prices across all the outcomes should sum to exactly $1.00, because exactly one of them must happen. When they don't, the difference has a name: event overround.

This is the simplest form of edge detection in prediction markets. It is also the most over-promised. Beginners look at a multi-outcome event with a sum of 1.06 and think "free 6%." It is almost never free. But the metric is still worth tracking, because the *changes* in EE over time tell you when the market is structurally re-pricing the field — and that is genuinely actionable.

## The Formula

```
EE = Σᵢ pᵢ − 1
```

Where pᵢ is the YES price of each mutually exclusive outcome in the event, expressed as a probability between 0 and 1, and the sum runs across all outcomes. The output is a fractional overround — multiply by 100 to display as a percent.

Three sign cases:

EE = 0 means the market is clean. The sum of YES prices across all outcomes is exactly $1.00, which is what a well-formed mutually exclusive and exhaustive event should always produce. No structural mispricing exists at this snapshot.

EE > 0 means the market is *overconfident in something happening*. The total notional you would need to sell YES on every outcome is larger than the $1.00 you would receive at resolution. In theory, selling YES on every outcome simultaneously locks in a profit equal to EE per dollar of notional. In practice, fees and slippage usually eat the trade.

EE < 0 means the market is *underconfident in the union*. The total cost to buy YES on every outcome is less than the $1.00 you would receive at resolution. In theory, buying YES on every outcome locks in a profit equal to |EE|. In practice, this side of the trade is more often "real" because under-selling typically reflects illiquidity rather than confident pricing — but the same fee and slippage caveats apply.

## A History of the Term

"Overround" comes from horse racing and football betting, where it is also called "vig" or "juice." A bookmaker's job is to quote odds that, when summed, exceed 1.0 by a margin that represents the bookmaker's profit. A horse race with 8 horses and an overround of 12% means the implied probabilities sum to 1.12, and the bookmaker pockets the 12% as compensation for taking the other side of every bet.

The metric has been part of bookmaker risk management since at least the 19th century. The math is identical to what prediction markets need. The only difference is that prediction markets do not have a single counterparty setting the line — the overround emerges from the collective behavior of the orderbook participants. In a perfectly efficient market, the overround should be exactly zero (no party has the incentive to leave money on the table). In practice, it is not, because prediction markets have idiosyncrasies (different liquidity per outcome, missing "Other" categories, time-of-day effects) that horse-race books do not.

The borrowing into prediction markets is fairly recent. Polymarket introduced multi-outcome events around 2021; Kalshi added them later. The "overround" framing as a structured metric came from the small group of traders who had backgrounds in sports books and brought the vocabulary with them. The name "Event Overround" is what I picked when I added the metric to the SimpleFunctions indicator stack, because plain "overround" was overloaded in the existing literature and I wanted a name that bound it specifically to the *event* (the set of mutually exclusive outcomes) as the unit of computation.

## The Newsom/Harris/Buttigieg Example, Walked Through

Polymarket multi-outcome event: "Who will win the 2028 Democratic nomination?" Imagine the orderbook one morning shows:

| Outcome | YES Price |
|---|:---:|
| Newsom | 0.32 |
| Harris | 0.18 |
| Buttigieg | 0.12 |
| AOC | 0.08 |
| Whitmer | 0.06 |
| Other | 0.30 |

Sum = 0.32 + 0.18 + 0.12 + 0.08 + 0.06 + 0.30 = **1.06**

```
EE = 1.06 − 1 = +0.06
```

The market is 6 cents overconfident in the Democratic nomination being decided by *somebody* in this list. In theory, you could sell YES on all six outcomes simultaneously and lock in 6 cents of profit per dollar of total notional, because exactly one of them is going to win and pay you $1.00, but you collected $1.06 to take that obligation.

In practice, on Polymarket, here is what happens to the trade. Polymarket charges 2% fees on each side of each outcome, plus AMM-style slippage that gets worse as you push the order through the book. To collect the full 6 cents you have to sell into all six order books simultaneously. The first five outcomes have decent depth (Newsom, Harris, Buttigieg) so the slippage is maybe 0.5 cents per side. The last two (AOC, Whitmer) and "Other" have thin books, and the slippage on those is 1.5 to 2 cents per side. Add 2% fees on each leg.

Net per dollar of notional: 6 cents headline overround minus ~3 cents of total slippage minus ~1 cent of fees. You are left with maybe 2 cents of edge if the trade goes perfectly, and a downside of catastrophic loss if any one of the outcomes resolves YES while you have an open short on it. The "free" 6% is gone before you even include the cost of the capital you are tying up across six positions for a year.

This is why the standard advice on EE is: "the raw number is rarely the trade." The math is correct, but the trade implementation costs eat it.

## What Actually Works: Tracking *Changes* in EE

The interesting use of EE is not the snapshot. It is the time series. Watching how EE moves on the same event over time tells you when the market is structurally re-pricing the field.

Suppose the same Polymarket event has these EE readings over a week:

| Day | EE |
|---|:---:|
| Monday | +0.06 |
| Tuesday | +0.05 |
| Wednesday | +0.04 |
| Thursday | +0.02 |
| Friday | +0.01 |

EE is dropping by about 1 cent per day. The interpretation is that the market is moving toward sharper, more confident pricing on a specific candidate. The total field is becoming more efficient as one outcome firms up and others compress.

If you can identify *which* outcome is firming up (it is usually the one whose YES price is rising fastest), you have a directional signal that doesn't depend on harvesting the overround at all. The EE compression tells you that consensus is forming. You then look for the candidate where the price is moving most aggressively, and that is the candidate the market is collectively concluding is the most likely winner.

Conversely, if EE is *expanding* over time — say, going from 0.01 to 0.05 to 0.10 — the market is becoming less efficient. Liquidity is fleeing or the field is fragmenting. Either way, the conditions are right for somebody to step in and provide structure (which is part of the [virgin polymarket maker strategy](/concepts/null-as-signal)).

This is the pattern that matters: EE as a derivative, not EE as a level. Raw EE on any single morning is rarely actionable. EE *changes* across mornings are the leading indicator of structural moves in the multi-outcome event.

## Two Ways the Assumption Silently Fails

EE is mathematically simple, but it depends on a strong assumption: the displayed outcomes are *mutually exclusive and exhaustive*. Two ways that assumption fails in practice, and both are dangerous because the formula keeps producing a clean-looking number in either failure mode.

**Failure 1 — Missing "Other" outcome.** Some events have an "Other" or "None of the above" outcome that captures the residual probability. If "Other" is missing from the displayed list — either because the venue doesn't expose it, or because the user pulled the data from an outcome-only endpoint that filters it out — the sum looks artificially low. For example, if the Newsom/Harris/Buttigieg event above had no "Other" row, the sum would be 0.76, and EE would compute to −0.24. That looks like a massive arbitrage opportunity. It is a data error.

The fix is to verify that the outcome set covers the event. On Polymarket, the event API exposes a flag for whether the event has "the residual outcome." On Kalshi, the convention is to include an explicit "no candidate" market in the event. On both venues, the safe approach is to compute EE only on events where the venue confirms the outcome set is exhaustive.

**Failure 2 — Overlapping outcomes.** Some events have outcomes that are not actually mutually exclusive. A "Will Bitcoin hit $200K in 2026?" market and a "Will Bitcoin hit $300K in 2026?" market both resolve YES if Bitcoin hits $300K. Adding their YES prices and computing EE is meaningless — the "or" operation in the union is broken because the events overlap. EE will produce a number, but the number doesn't refer to anything real.

The fix here is harder because the venue does not always tag the event semantics clearly. The practical approach is to only compute EE on events that are explicitly tagged as "winner-take-one" categorical outcomes (presidential nominations, sports championships, weather buckets) and to refuse to compute it on threshold-style events even when the venue groups them together. The CYC regex grouper that powers the cycle clustering metric has a category filter for exactly this reason.

There is also a softer third failure mode worth mentioning: *liquidity asymmetry*. If five of six outcomes have deep books and the sixth has 12 cents of total depth, the EE reading is dominated by the sixth outcome's stale price. EE = +0.04 in that case might really be +0.10 if you tried to actually trade against the orderbook. The fix is to weight or filter outcomes by their orderbook depth before summing — though in practice, I just mark the EE reading "low confidence" when any one outcome's depth is below a threshold and let the user decide.

## How to Combine EE With the Other Indicators

EE is the *structural consistency check*. It tells you whether the multi-outcome event is pricing cleanly or whether there is an internal contradiction. It does not tell you which outcome to buy.

The composition I use:

First, filter by [implied yield](/concepts/implied-yield) to find candidates that pay enough to engage. IY > 100% on at least one outcome in the event narrows the universe.

Second, filter by [cliff risk index](/concepts/cliff-risk-index) to find events where activity is happening. High CRI on at least one sibling is enough to flag the event as live.

Third, check EE. Three actionable cases. If EE > 0.04, the field is structurally rich and one of the IY-passing siblings may be where the slack is — investigate which sibling has the most aggressive recent price move. If EE < −0.04, the field is structurally cheap and the cheapest outcome may be the trade — but check liquidity carefully because under-pricing usually reflects orderbook thinness. If |EE| < 0.02, the field is pricing cleanly and any edge has to come from a thesis update on the specific sibling, not from structural mispricing.

Fourth, apply judgment. The remaining few candidates get the orderbook read, the news check, and the size decision.

EE alone is rarely the trade. EE in combination with IY and CRI surfaces a small set of candidates where multiple signals converge. That convergence is what makes the indicator stack worth running. The full composition is in [the expected edge concept page](/concepts/expected-edge).

## A CTA

Live event overround data across Polymarket and Kalshi multi-outcome events is on the [/screen](/screen) page. Filter by EE > 0.03 or EE < −0.03 to find the events where the overround is large enough to be worth investigating. Most are noise. The ones that aren't are where the structural-vs-thesis distinction starts to pay.