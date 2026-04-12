# Contagion Velocity Rate (CVR)

**Contagion Velocity Rate measures how quickly a thesis priced into one prediction-market contract propagates to its semantic neighbors, computed as the lag in hours between a 5¢ move on a parent contract and an equivalent move on a related contract from the same event family. Low CVR means a thesis is spreading fast; null CVR (the common case) means the contract has no detected siblings and the thesis is uncontaminated by neighboring price action.**


## Explanation

## What "Contagion" Means in This Context

A piece of news about the May Fed meeting affects the May Fed-decision contract first. Within minutes, related contracts — the June Fed-decision, the year-end "any cut" contract, the SOFR futures market — start to reprice in the same direction. The speed at which that secondary repricing happens is contagion velocity. Fast contagion means the market is treating the news as a structural reassessment that affects the whole rate stack. Slow contagion means traders are still arguing about whether it matters.

The formula is a measured lag, not a closed form:

```
CVR = median lag (hours) between 5¢ Δp on parent contract and 5¢ Δp on related contract,
      across recent contagion events
```

Where "related contract" comes from the cycle-clustering grouper (see `cycle-clustering`), and "recent" defaults to the last 30 days of history in `market_indicator_history`. The median is the statistic — outliers from one-off news shocks get filtered.

## The "Sparse Coverage" Caveat

CVR is in Tier B of the indicator stack: it requires both the parent and a sibling to have history records, and most contract families do not have enough siblings to compute it at all. For roughly 80% of markets, CVR returns null — because either the contract has no detected sibling family, or the family is too small to produce a meaningful median lag. That null is informative in its own right: a contract with null CVR is *uncontaminated*, in the sense that no one has been pricing related contracts in concert with it.

For the strategies that look for first-mover edge, uncontaminated is the entry condition. A thesis that is not yet priced into the sibling markets is one where you can take a position in the parent before the contagion catches up — and harvest the lag yourself, by taking matching positions in the unmoved siblings as the news propagates. That is the trade. CVR identifies when the lag exists.

## When Low CVR Tells You Something Different

A low CVR (siblings repricing within an hour of the parent) tells you the family is being traded as a single unit by at least one disciplined participant. That is sometimes a tell that there is a coordinated maker on the family, and the apparent dislocation between sibling prices is being held together by their quotes. Trading against that requires more conviction than trading against an uncoordinated retail flow, because the maker has a structural view on the cross-prices that you would have to refute.

High CVR (siblings repricing 8+ hours later) is the opposite: nobody is connecting the dots between contracts, and a fast trader can pick off the lag. That is the traditional cross-market arbitrage opportunity, just expressed as a velocity instead of a static spread.


## Example

Three Fed-related contract families, observed across the last 30 days:

| Parent contract family | Detected siblings | Median lag | CVR | Read |
|---|:---:|:---:|:---:|:---:|
| Fed cut at next meeting | 4 | 0.8 hours | 0.8h | Tightly coordinated, retail can't pick off |
| EU rate decision | 2 | 6.5 hours | 6.5h | Slow contagion, lag-trade opportunity |
| Random commodity event | 0 | n/a | null | No siblings, uncontaminated |

The first family (Fed cut) has CVR = 0.8h. Within an hour of any 5¢ move on the parent, the siblings reprice. There is almost no lag to harvest, and any apparent dislocation is being actively held together by a market maker pricing the family as a unit. Trading against this requires a structural view, not just a velocity view.

The second family (EU rate decision) has CVR = 6.5h. A 5¢ move on the parent typically takes 6+ hours to show up on the sibling markets. That is a window where a fast trader can take a position in the parent and immediately add matching positions in the unmoved siblings — and harvest the lag as it closes. This is the textbook cross-market lag trade.

The third family is the interesting one. CVR is null because the contract has zero detected siblings in the cycle-clustering output. That null is the entry condition for a different strategy: thesis purity. Without sibling contagion, the parent's price reflects only its own thesis — no spillover from adjacent markets, no need to disentangle. For a trader who wants to bet on a single, isolated event without confounders, a null CVR is exactly what to look for.


## CLI

```bash
sf scan --by-cvr asc --warm
```


## Related

[cycle-clustering](cycle-clustering.md), [cliff-risk-index](cliff-risk-index.md), [edge-detection](edge-detection.md), [cross-venue](cross-venue.md), [null-as-signal](null-as-signal.md), [signal](signal.md), [mean-reversion](mean-reversion.md), [event-overround](event-overround.md)
