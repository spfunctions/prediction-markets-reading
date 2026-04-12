# Cliff Risk Index (CRI)

**The cliff risk index measures how fast a binary prediction-market contract is approaching resolution, defined as the absolute price velocity multiplied by the days remaining: CRI = |Δp/Δt| × τ. High CRI flags markets that are actively deciding; low CRI flags markets that are stuck.**


## Explanation

## Why "Cliff"

Every binary prediction-market contract eventually walks off a cliff. At resolution it snaps to either $1.00 or $0.00 with nothing in between. The interesting question is *when* the cliff approach starts. Some contracts drift sideways for months and then resolve in the final week. Others reprice violently months before expiry as new information arrives. CRI is the metric that separates the two.

The formula is:

```
CRI = | Δp / Δt |  ×  τ_remaining
```

Where:

- **Δp** = price change over a recent window (e.g., 24 hours)
- **Δt** = the window length, in days
- **τ_remaining** = days to resolution

The first term — absolute velocity — captures *how much* the market moved. The second term — days remaining — captures *how much story room* is left for the move to mean something. The product is the metric.

## What High vs Low CRI Means

A high-CRI contract is one where the market is actively making up its mind right now. A low-CRI contract is asleep. Use CRI to allocate attention: which contracts are *active*, regardless of which direction the headline price is going?

CRI is not a directional signal. It does not tell you whether the move is bullish or bearish, only that there *is* a move. Combine it with other indicators (your thesis, news flow, divergence from related markets) to decide what to do about the activity.

## Why the τ Scaling Matters

Without the τ multiplier, you would just have raw velocity, and a contract about to expire would always look like the most active thing on the board (because the snap to 0 or 1 produces the largest possible move). The τ scaling deflates near-expiry noise and surfaces *structural* moves earlier in the contract's life.

A 1-cent move with 200 days remaining is a structural reassessment. A 1-cent move with 5 days remaining is settlement noise. CRI says the first one is 40× more interesting (200/5 = 40), and that ratio is what makes the metric useful for allocation.


## Example

Three contracts on a typical Monday morning:

| Contract | 24h Δp | τ remaining | CRI |
|---|:---:|:---:|:---:|
| A — Recession 2026 YES | 0.45 → 0.46 | 200 days | 0.01 × 200 = **2.0** |
| B — Powell next chair YES | 0.45 → 0.49 | 60 days | 0.04 × 60 = **2.4** |
| C — Fed cut May YES | 0.45 → 0.61 | 5 days | 0.16 × 5 = **0.8** |

Contract C has the loudest headline move (16¢ in a day) but the *smallest* CRI of the three. That is the scaling working as intended: a 16¢ move with five days left is largely a settlement crystallization, not a structural shift.

Contract A has the smallest headline move (1¢) but the second-highest CRI, because the move happened with 200 days of story room remaining — a tiny shift that actually represents a fresh thesis being priced in.

Contract B sits in the middle and is the most "live" of the three by this metric: a meaningful move (4¢) at a meaningful horizon (60 days). CRI tells you to look at B first.


## CLI

```bash
sf scan --by-cri desc
```


## Related

[implied-yield](implied-yield.md), [event-overround](event-overround.md), [volatility](volatility.md), [edge-detection](edge-detection.md), [binary-contract](binary-contract.md), [expiration](expiration.md)
