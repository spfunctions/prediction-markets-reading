# Event Overround (EE)

**Event overround is the sum of YES prices across all mutually exclusive outcomes in a single prediction-market event, minus one: EE = Σpᵢ − 1. A clean market gives EE = 0. Positive EE means the outcomes collectively overstate certainty (sell-side arbitrage). Negative EE means the outcomes collectively understate it (buy-side arbitrage).**


## Explanation

## A Borrowed Term

"Overround" comes from sports betting, where it describes how much a bookmaker has built into the odds as their margin. If you add up the implied probabilities from a horse race's odds, you get a number greater than 1.0, and the difference is the bookmaker's vig.

Prediction markets do not charge a vig in the same way, but multi-outcome events still have an overround — and tracking it is one of the simplest forms of edge detection. The formula is just:

```
EE = Σᵢ pᵢ − 1
```

Where pᵢ is the YES price of each mutually exclusive outcome, expressed as a probability. The sum is the total probability the market is collectively assigning. Subtract 1 (the true total in any well-formed event) and you get the overround.

## What Each Sign Means

- **EE = 0** → clean market. All outcome prices add up to exactly $1.00. No riskless arbitrage from overround alone.
- **EE > 0** → the sum exceeds 1.0. You can sell YES on every outcome simultaneously and lock in a profit equal to EE per dollar of total notional. The market is overconfident in *something happening*.
- **EE < 0** → the sum is less than 1.0. You can buy YES on every outcome and lock in a profit equal to |EE| per dollar. The market is underconfident in the union — usually because liquidity is thin or one of the outcomes is illiquid.

In practice, raw EE is almost never wide enough to overcome fees and slippage. The interesting use is monitoring *changes* in EE on the same event over time, which often reveals when one outcome is being aggressively repriced relative to its siblings.

## Where It Breaks

EE only makes sense for *mutually exclusive and exhaustive* outcomes. Two common ways the assumption silently fails:

1. The event has an "Other" outcome that is missing from the displayed list. Sum looks low, but the true union (including "Other") is fine.
2. The outcomes overlap. Two YES contracts both win in the same scenario. Sum looks fine, but the true union is broken.

Always verify the outcome set covers the event before treating EE as a signal.


## Example

Polymarket multi-outcome event: "Who will win the 2028 Democratic nomination?"

| Outcome | YES Price |
|---|:---:|
| Newsom | 0.32 |
| Harris | 0.18 |
| Buttigieg | 0.12 |
| AOC | 0.08 |
| Whitmer | 0.06 |
| Other | 0.30 |

Sum of YES prices: **1.06**

```
EE = 1.06 − 1 = +0.06
```

The market is 6 cents overconfident. In theory, selling YES on all six outcomes simultaneously locks in 6 cents of guaranteed profit per dollar of notional, minus fees and the slippage you eat building the position.

In practice on Polymarket, fees and AMM slippage will almost certainly eat the 6 cents. The useful information is that this market is *currently* trading rich on the certainty of the field — and if EE drops to 0.02 next week, that tells you the market is moving toward sharper expectations on a specific candidate.

For the negative case, imagine the same event but with sum = 0.94. EE = −0.06. Buying YES on every outcome locks in 6 cents — and this side of the trade is more often "real" because under-selling typically reflects illiquidity rather than confident pricing.


## CLI

```bash
sf scan --by-overround
```


## Related

[implied-yield](implied-yield.md), [cliff-risk-index](cliff-risk-index.md), [orderbook](orderbook.md), [liquidity](liquidity.md), [binary-contract](binary-contract.md), [cross-venue](cross-venue.md)
