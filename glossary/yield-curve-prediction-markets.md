# Yield Curve (Prediction Markets)

**A prediction-market yield curve is the term structure of implied yield across all contracts in a single event family, plotted as IY against τ-days for each sibling. The curve is the prediction-market analog of a treasury yield curve, with each "maturity" replaced by a contract resolution date. The curve only exists for event families with two or more sibling contracts, which makes cycle clustering the prerequisite for plotting any curve at all.**


## Explanation

## The Borrowed Vocabulary

A treasury yield curve plots yield-to-maturity against time-to-maturity for a set of bonds with the same issuer (the US Treasury) and different maturities. The shape of the curve tells you what the market thinks about future interest rates: a steep curve means the market expects rates to rise, a flat curve means expectations are stable, an inverted curve means the market expects rates to fall (and is often a recession signal).

A prediction-market yield curve does the same thing for the same reason. Plot implied yield against τ-days for every contract in the same event family — for example, every Fed-decision contract from May to December — and you get a curve whose shape tells you which Fed meetings the market is treating as more or less uncertain. A steep curve means the near-term meetings are paying disproportionately well (the market is anxious about an imminent decision). A flat curve means uncertainty is evenly distributed across meetings. An inverted curve, where far-out contracts pay more than near-term ones, almost always indicates that one specific far-out contract has a structural reason to be high-IY (an event with thin liquidity or known catalyst clustering near its resolution).

## Why CYC Is the Prerequisite

A yield curve requires *at least two contracts in the same family*, plotted against the same time axis. That requires identifying the family in the first place — which is the job of the Cycle Clustering grouper (see `cycle-clustering`). Without CYC, you have 47,000 isolated points, none of which form a curve. With CYC, you have ~2,500 multi-member event families, each of which can be plotted as a curve of varying detail.

The detail of any individual curve depends on how many siblings the family has. A family with 3 members produces a 3-point curve, which is enough to see direction but not shape. A family with 12 members (e.g., monthly Fed-decision contracts across a year) produces a smooth curve with visible local features. The richest families on the platform tend to be calendar-anchored: monthly Fed decisions, quarterly earnings, monthly inflation prints.

## How to Read the Shape

Three patterns to look for. **Steep contango** (near-term low, far-term high) usually means there is no imminent catalyst and the market is pricing the long tail of accumulating uncertainty. **Steep backwardation** (near-term high, far-term low) usually means there is a known imminent catalyst (a meeting, a vote, a deadline) that the market is pricing aggressively, and it dies down once the catalyst is resolved. **Local kink** at a specific maturity usually means a single contract in the family has structural liquidity issues — the IY is high not because the thesis is rich but because the orderbook is thin.

The kink case is the one to watch out for. A naive "buy the highest IY in the family" strategy gets fooled by kinks every time, because the highest IY is often the contract with the worst liquidity, which means the displayed yield is theoretical and the executable yield is much lower. Always pair the curve read with an LAS check on the highest-IY point.


## Example

A SpaceX-IPO yield curve, taken from `/api/public/yield-curves/spacex-ipo-by-end-of` on a recent day:

| Sibling contract | τ-days | YES Price | IY |
|---|:---:|:---:|:---:|
| spacex-ipo-by-end-of-q2-2026 | 80 | 0.08 | 1,460% |
| spacex-ipo-by-end-of-q3-2026 | 170 | 0.18 | 280% |
| spacex-ipo-by-end-of-q4-2026 | 260 | 0.32 | 75% |
| spacex-ipo-by-end-of-q1-2027 | 350 | 0.44 | 28% |
| spacex-ipo-by-end-of-2027 | 540 | 0.62 | 11% |

Plotted, this is a steep contango that flattens out: the near-term contracts are paying enormous IY because the market thinks an imminent IPO is unlikely (8 cents on the Q2 contract), and the far-term contracts are paying low IY because the market is fairly confident an IPO happens by year-end 2027 (62 cents on that contract).

The shape tells you the market's timing distribution: most of the probability mass is between Q4 2026 and year-end 2027, with the Q1 2027 contract at the inflection point. A trader who has private information about the IPO timing — say, a strong view that Q3 2026 is the actual launch window — can take a position on the Q3 contract (currently at 18 cents, IY 280%) and ignore the rest of the curve. A trader who has no view on timing should look at the curve as a whole and ask whether the *shape* makes sense relative to alternative info sources (SpaceX press releases, banker chatter, regulatory filings).

The kink check: are any of these IYs theoretical because the orderbook is thin? Pulling LAS for each point shows that the Q2 and Q1-2027 contracts both have LAS < 30 (very thin), while the Q3 and Q4 contracts have LAS > 200 (executable). The 1,460% IY on Q2 is largely fictional — the spread is wide enough that the executable yield is much lower than the displayed yield. The 280% IY on Q3 is real and tradable.

The composite read: the yield curve shows you the term structure, the LAS check shows you which points on the curve are real, and the combination is the trade.


## CLI

```bash
sf yield-curve <event-family>
```


## Related

[cycle-clustering](cycle-clustering.md), [implied-yield](implied-yield.md), [tau-days](tau-days.md), [liquidity-availability-score](liquidity-availability-score.md), [contagion-velocity-rate](contagion-velocity-rate.md), [event-overround](event-overround.md), [expected-edge](expected-edge.md), [cross-venue](cross-venue.md)
