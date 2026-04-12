# Cliff Risk Index: The Activity Filter

> CRI = |Δp/Δt| × τ_remaining. Why velocity alone misleads, why the τ multiplier deflates settlement noise, and how three contracts in different states walk through the math.

**Category:** indicator | **Author:** Patrick Liu | **Reading time:** 13 min

---
Every binary prediction-market contract eventually walks off a cliff. At resolution it snaps to either $1.00 or $0.00 with nothing in between. The interesting question is not *whether* the cliff arrives — it always does — but *when* the approach to the cliff begins to dominate the price action.

Some contracts drift sideways for months and then resolve in the final week. Others reprice violently months before expiry as new information arrives. The distinction matters because the first kind is dead and the second kind is alive, and the market scanner I want is one that surfaces the second kind without drowning in the first. CRI is the metric that does that.

## The Formula

```
CRI = | Δp / Δt |  ×  τ_remaining
```

Where Δp is the price change over a recent window, Δt is the window length in days, and τ_remaining is days until resolution. The first term is the absolute price velocity over the chosen window — how fast the market is moving, ignoring direction. The second term is the remaining lifetime of the contract — how much story room is left for the move to have meant something.

The product is a single number per contract. High CRI means active market; low CRI means stuck market. The unit is "cents of price change per day, multiplied by days of remaining life," which simplifies to "expected total cents of price change between now and resolution if the current velocity holds." That framing matches what the metric is actually measuring: a contract with high CRI is on a trajectory where the cumulative move-to-resolution is large, and a contract with low CRI is on a trajectory where it is small.

## Why Velocity Alone Is Wrong

The first version of an "active markets" scan I shipped just used raw |Δp/Δt|. It was useless. The top of the scan was always dominated by markets in the final 24 hours of their lives, where prices were snapping toward $1.00 or $0.00 in 10-cent jumps as they resolved. Those markets were "active" in the sense that prices were moving fast, but the activity was *settlement crystallization* — the natural binary outcome forming up — not new information being priced in.

The mistake was conflating two different sources of price movement. The first source is *thesis update*: the market changes its mind because new information arrived. This is the kind of activity that creates trading opportunities. The second source is *settlement noise*: the market is converging to its terminal value because the resolution is imminent. This is the kind of activity that creates exits, not entries.

Pure velocity does not distinguish between the two. A 5-cent move with 200 days left is almost certainly thesis update (no information would push the market 5 cents in the final 24 hours of a 200-day contract for any reason other than a real new event). A 5-cent move with 1 day left could be either, but the base rate strongly favors settlement noise. You want a metric that demotes the second kind and amplifies the first.

The τ multiplier does exactly that. It scales velocity by remaining lifetime, so a fast move late in life gets demoted (small τ, small product) while a fast move early in life gets amplified (large τ, large product). The economic intuition is: a structural reassessment is rare and worth investigating; a settlement crystallization is common and worth ignoring.

## A History of the Term

CRI does not have a clean parallel in any single existing field, but it borrows from three. From bond trading: *convexity* captures how price changes accelerate as yields move, and the rate of acceleration depends on time to maturity. CRI captures how price changes accumulate as resolution approaches, and the rate depends on remaining time. The math is different but the role is the same — a measurement of how sensitive the instrument is to the path between now and maturity.

From options: *gamma* describes how delta changes as the underlying moves, and gamma is largest near expiry for at-the-money options. The "near expiry, near the money" condition is the moment when small underlying moves cause large price moves. CRI captures the analogous condition in binary contracts: small thesis changes cause large price moves when there is little time left and the price is far from a corner.

From sports betting: *steam moves* are sudden coordinated price shifts in the lines, usually triggered by sharp money. A steam move on a contract days before the event is a signal. A steam move minutes before kickoff is just the line settling. CRI is, in a real sense, a steam-detection metric — it isolates the moves that happen with enough time left for them to be informationally meaningful.

The composite framing — velocity scaled by remaining time — is mine, in the sense that I have not seen it written down in either the bond, options, or sports betting literature in this exact form. The closest is "PV01-times-DTE" approaches in fixed-income desks, which serve a similar purpose. The Cliff Risk Index name was chosen because the binary contract's resolution to $0 or $1 *is* a cliff, and the question CRI answers is "how much real action is happening on the approach."

## Three Contracts Walked Through

I want to walk through three contracts on a typical Monday morning to show what high, medium, and low CRI mean in practice.

**Contract A — Recession 2026 YES.** A long-dated macro contract. 24h Δp = 0.45 → 0.46. τ_remaining = 200 days.

```
CRI_A = |0.46 − 0.45| × 200
      = 0.01 × 200
      = 2.0
```

A 1-cent price move is small in absolute terms. But it happened on a contract with 200 days of story room, which means somebody re-priced their thesis based on something. The base rate of "noise" 1-cent moves on a long-dated macro contract is low — most days that contract doesn't move at all. The CRI of 2.0 puts this contract in the "worth a look" bucket without screaming.

**Contract B — Powell next chair YES.** Medium-dated political contract. 24h Δp = 0.45 → 0.49. τ_remaining = 60 days.

```
CRI_B = |0.49 − 0.45| × 60
      = 0.04 × 60
      = 2.4
```

A 4-cent move with 60 days left. This is the most "live" of the three by CRI, and it should be — a 4-cent move at this horizon is enough to be visible above noise, and 60 days is long enough that the move is *almost certainly* about something other than settlement. Whatever the thing is — a Trump tweet, a leaked shortlist, a polling shift — somebody has new information and is pricing it in. CRI says: investigate first.

**Contract C — Fed cut May YES.** Short-dated rate contract. 24h Δp = 0.45 → 0.61. τ_remaining = 5 days.

```
CRI_C = |0.61 − 0.45| × 5
      = 0.16 × 5
      = 0.8
```

The loudest headline move (16 cents in a day) but the *smallest* CRI of the three. This is the τ scaling working as intended: a 16-cent move with five days left is largely a settlement crystallization. The market is forming up its final answer. There may still be some thesis-update component in there, but the base rate strongly favors "the market is converging to its terminal value." CRI demotes this contract from the active list and frees the attention budget for A and B.

The ordering matters. A pure-velocity scan would have ranked C first (16¢ move!), B second (4¢ move), and A last (1¢ move). The CRI scan ranks B first, A second, and C third — and B is the contract that actually has a story you can engage with. The whole point of CRI is to invert that pure-velocity ordering by accounting for how much room is left.

## Where CRI Is Misleading

Three failure modes worth knowing about, plus a fourth that sneaks in late.

**Failure 1 — One-time news vs structural reassessment.** CRI cannot tell the difference between a single news shock and a sustained re-pricing. A contract that jumps 8 cents on a Tuesday because of one tweet, then drifts back over Wednesday and Thursday, will have a high CRI on Tuesday's reading and a misleading "active" classification. The fix is to use a longer Δt window (3 days instead of 24 hours) or to compute CRI on multiple windows and look for consistency. A contract with high CRI on the 24h, 72h, and 168h windows is structurally active. A contract with high CRI on only the 24h window is news-shocked.

**Failure 2 — Liquidity-driven false moves.** A contract with 12 cents of orderbook depth can move 8 cents on a $50 trade. The CRI scan will show this as a structural reassessment when it is actually just one user with a bigger-than-typical bet. The fix is to gate the CRI scan by [liquidity availability score](/concepts/null-as-signal) — only trust CRI on contracts where the orderbook is deep enough that the velocity reading is not dominated by a single trade. In practice, this means trusting CRI on the warm-cron-covered top 500 markets and treating CRI on long-tail markets as suggestive only.

**Failure 3 — The τ multiplier double-counting.** CRI uses τ_remaining as the multiplier, which means a 200-day contract with the same daily velocity as a 20-day contract gets a CRI 10× larger. That is the right answer when you care about *cumulative* move-to-resolution, but it can over-amplify long-dated contracts that are slowly drifting. A 1-cent daily move that has been sustained for 30 days on a 200-day contract is interesting; a 1-cent daily move that just happened once is noise. The CRI formula treats them identically. The fix is to require sustained velocity over a longer window before trusting the high CRI reading.

**Failure 4 — Resolution-time clustering.** Some events resolve at unusual hours (a court ruling at 10 AM ET, a Fed announcement at 2 PM ET). The 24-hour Δt window catches different parts of the move depending on when you read it. A scan run at 9 AM picks up the previous day's ruling-time move; a scan run at 4 PM picks up nothing. This is not a CRI bug per se, but it means the CRI reading is sampling-time dependent in a way the formula does not expose. The mitigation is to compute CRI on rolling windows aligned to "trading sessions" (which prediction markets do not have, so you have to define your own) or to read CRI as a smoothed average over the last few windows rather than a single snapshot.

## How to Combine CRI With the Other Indicators

CRI is the *activity filter*. It tells you which markets are alive. It does not tell you whether to buy, sell, or pass. The composition I use:

First, filter by [implied yield](/concepts/implied-yield) to find contracts that pay enough to bother with. IY > 100% annualized typically narrows the universe to 100-300 contracts.

Second, filter by CRI to find the active subset. CRI > 1.0 (a number I picked from inspection — your threshold may differ) typically narrows the IY-filtered set to 30 contracts. This is the "high IY *and* moving" intersection, which is the only intersection where the IY actually matters.

Third, check [event overround](/concepts/event-overround) on multi-outcome contracts to see if the sibling set is consistent. EE near 0 means the field is pricing cleanly and the activity has to come from a thesis update, not a structural mispricing. EE > 0 means the field is overconfident and the active contract may be where the slack is.

Fourth, apply judgment on the remaining 5 contracts. Read the orderbook. Check the news flow. Compare cross-venue prices. Decide.

CRI is the second filter, not the first. Putting CRI before IY would give you "moving markets" without regard to whether the move means anything in yield terms. A 5-cent move on a 5-cent contract with 200 days left is structurally interesting (CRI = 1.0) but the IY math is meaningless (you cannot annualize a near-zero-priced contract). Always run IY first.

The full funnel is described in [the valuation funnel concept page](/concepts/the-valuation-funnel) and [the expected edge composition](/concepts/expected-edge), and the live data is on [/screen](/screen).

## A CTA

Run `sf scan --by-cri desc --min-tau 1 --min-iy 1.0` to get the live list of high-CRI contracts that also clear the IY filter. The top of the list is where the actively-deciding markets live. Some are trades. Most are not. The point of CRI is to make sure you are not wasting attention on the dead ones.