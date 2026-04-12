# The Vig Wall: Why Multi-Outcome Events Have a Hard Overround Floor

> On a multi-outcome prediction-market event, the sum of YES prices across all outcomes is almost always greater than 1.0 by 2-4 cents. That floor is structural. It is the makers charging for the adverse selection they take on every outcome, and trading event-overround arbs smaller than the wall is just paying makers.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
Every new prediction-market trader who learns about [event-overround](/concepts/event-overround) has the same first thought. "Wait — if the sum of all the YES prices on the candidates exceeds 1.00 by 6 cents, I can just sell YES on every outcome and lock in 6 cents of risk-free profit, right?" Yes in theory, no in practice, and the reason is what I call the vig wall.

The vig wall is the structural minimum overround on a liquid multi-outcome prediction-market event. Empirically it is 2-4 cents on most categories I have measured. You cannot get below it through arbitrage because the makers maintaining the orderbook are *charging it on purpose* to compensate themselves for the adverse selection they eat on each outcome they quote. Any "arbitrage" smaller than the vig wall is paying makers their commission, not capturing edge.

This page is about why the wall exists, how to measure it on different categories, and why the interesting use of EE is the *change* in the overround over time, not the level.

## Where the Wall Comes From

The math is straightforward. A market maker quoting one outcome of a multi-outcome event faces a problem: they are selling YES on a contract that has some probability of resolving YES, and the people most likely to hit their offer are the ones who think the probability is higher than the offer. The maker's expected loss per fill is *the difference between the displayed offer and the true probability*, weighted by the probability of being filled.

To stay solvent, the maker has to charge a spread that covers the expected adverse selection plus their cost of capital plus their inventory risk plus a profit margin. Call this total compensation per outcome the *adverse selection charge*. For a single binary contract on a liquid event, it is typically 1-2 cents wide on each side of the mid (so the bid-ask spread is 2-4 cents).

Now extend to a multi-outcome event with N outcomes. The maker is now quoting N separate contracts, one per outcome. They have to earn the adverse selection charge on each outcome. The aggregate effect on the overround is *the sum of the per-outcome adverse selection charges*, which scales roughly linearly with N (because each outcome has its own adverse selection problem and the makers cannot net them against each other except in trivial cases).

The minimum aggregate charge across N outcomes is the vig wall. For a 5-outcome event with 0.5-cent per-outcome adverse selection (a low-attention market where the makers are not facing aggressive sharps), the wall is about 2.5 cents. For a 10-outcome event with 1-cent per-outcome adverse selection (a politicized market where the makers are nervous about sharps), the wall is closer to 6-8 cents. For a 20-outcome event with 1.5-cent adverse selection per outcome, the wall is in the 15-25 cent range.

The wall is *floor*, not a target. Markets often trade above the wall when the audience is bidding up the certainty of the field beyond what the makers would charge alone. But they essentially never trade *below* it for any sustained period, because makers will widen their quotes (or pull them entirely) if they cannot earn the adverse selection charge.

## What I Have Measured

I have logged the average overround on about 200 multi-outcome events across Kalshi and Polymarket over the last year. The pattern is consistent:

- 2-outcome events (which are mathematically the same as binary contracts): wall ~ 1-2 cents. The "overround" here is essentially the bid-ask spread on the binary, so it is captured by normal stage 2 thinking and is not interesting as a separate concept.
- 3-5 outcome events (Senate races with a few candidates, NFL conference championships, etc.): wall ~ 2-4 cents. The most common case I see in practice.
- 6-10 outcome events (Democratic nomination with the major candidates, Eurovision finals, etc.): wall ~ 4-7 cents. Larger because each additional outcome adds another adverse selection problem.
- 11+ outcome events (Heisman watch lists, all-NFL-playoffs winners, multi-candidate primaries with the long tail): wall ~ 8-15 cents. The tail is expensive because each long-shot outcome is its own per-outcome problem and the maker cannot net them.

The walls are consistent across venues but the *level above the wall* varies. On Kalshi, multi-outcome events with retail attention often trade 3-5 cents above the wall (so total overround 5-9 cents). On Polymarket, the AMM curve effectively bakes in extra overround because the curve gets steep at the tails, so events with fat-tailed outcome distributions trade 2-3 cents above the displayed wall.

## A Worked Example

Take the "2028 Democratic nomination" multi-outcome contract on Polymarket as it stood late last year. The displayed YES prices were:

| Outcome | YES Price |
|---|:---:|
| Newsom | 0.32 |
| Harris | 0.18 |
| Buttigieg | 0.12 |
| AOC | 0.08 |
| Whitmer | 0.06 |
| Other | 0.30 |
| **Sum** | **1.06** |

EE = 1.06 − 1.00 = +0.06 = 6 cents of apparent overround.

The naive arbitrage is "sell YES on every outcome simultaneously, lock in 6 cents per dollar." Let me walk through what actually happens.

To sell YES on each of the 6 outcomes, you have to either find an existing buyer (cross the bid) or place a sell order (offer the ask). On Polymarket's AMM, every "sale" you make moves the AMM curve, which raises the price you can sell at on subsequent fills (and also subsidizes the next buyer). The first contract you sell on Newsom might fill at 0.32, but the second contract fills at 0.319, the tenth at 0.317, the hundredth at 0.310. The AMM is structured so that aggressive selling on one side moves the price down.

So your fills are not at the displayed mid. They are at progressively worse prices. The aggregate slippage cost on selling 6 outcomes simultaneously, on positions large enough to be worth the trade, is typically 3-5 cents — most of which goes to the AMM curve (which is the vig wall in disguise on Polymarket).

After slippage your effective short position is at an aggregate sum of *closer to 1.01-1.02*, not 1.06. Your locked-in profit is 1-2 cents per dollar of notional. Then you pay the gas + bridge costs on the entries, which on a small position is maybe 0.5 cents. Then you pay the bridge cost on the exit when the contract resolves, another 0.5 cents.

Net realized profit on a "6-cent arbitrage": about 0.5-1 cent per dollar, on a position that ties up capital for the lifetime of the contract and exposes you to cluster mis-grouping risk if the "Other" outcome was specified weirdly.

That is the vig wall in practice. The displayed overround was 6 cents. The actual overround net of frictions was 4-5 cents (the wall). The amount you can extract via arbitrage is the spread *above* the wall, not the full displayed spread.

## What the Wall Means for the EE Indicator

The [event-overround](/concepts/event-overround) indicator on the [pm-indicator-stack](/concepts/pm-indicator-stack) is computed as the raw sum minus 1. That number includes the vig wall. To get the *tradable* overround, you have to subtract the wall — which depends on the number of outcomes and the venue and the category.

The simpler use of EE is not "level" but "change." A market with persistent EE = +0.06 is at its normal vig wall and has no arbitrage. A market where EE moves from +0.06 to +0.10 over a few hours is *not* a market where the vig wall is widening — vig walls are sticky. It is a market where the *level above the wall* is widening, which is a real signal: someone is over-pricing the certainty of the field. That is where trades live.

The opposite case — EE moving from +0.06 to +0.02 — is also signal. The level above the wall has narrowed, which means the market has gotten *more efficient* at pricing the field. This often happens around catalysts: the field uncertainty resolves toward a single candidate, and the structural overround goes away.

The [the-overround-illusion](/opinions/the-overround-illusion) opinion piece argues this point in detail. The TL;DR is that you should never look at raw EE as a tradable number. You should look at *EE relative to the vig wall for that contract's category*, and you should look at *changes in EE over time*. Both of those framings respect that the wall is structural and the wall is not the trade.

## Where the Wall Breaks Down

A few cases where the vig wall framework gets fuzzy.

**Illiquid multi-outcome events.** When the orderbook on each outcome has only one or two resting orders, there is no real "maker" charging the wall. The displayed mid is whoever happened to put up the last quote. EE on these markets can be wildly negative or wildly positive, but neither side is tradable because the depth supporting either trade is zero. The wall does not apply because there is no maker to maintain it.

**Brand-new multi-outcome events.** During phase 1 and phase 2 of [new market formation](/concepts/new-market-formation), the wall has not stabilized yet. The makers are still figuring out where each outcome should clear, and the per-outcome adverse selection charges are inflated to cover the discovery uncertainty. Walls of 8-15 cents on small events are common in the first 24 hours and shrink to the 2-4 cent normal range after the contract has had a day to stabilize.

**Mismatched outcome sets across venues.** If Kalshi's version of an event lists 5 outcomes and Polymarket's version lists 8 outcomes (with 3 long-shot candidates added), the walls on the two are not comparable. Kalshi's smaller outcome set will have a smaller wall, which makes it look like Kalshi is "more efficient" — but actually the two contracts have different definitions and the comparison is not meaningful.

**The "Other" outcome can hide all of this.** If the multi-outcome event has an "Other" or "Field" outcome that captures everything not enumerated, the displayed sum should always be very close to 1.0 by construction (because Other catches the remainder). Markets with "Other" have small displayed EE almost regardless of how the named outcomes are priced. The vig wall is buried inside the per-outcome spreads rather than the aggregate sum, so the EE indicator is misleading on these contracts. Look at the spreads, not the sum.

## How This Connects to the Stack

The vig wall is the answer to the most common naive trade in multi-outcome events: "the sum is over 1, free money." The vig wall says: there is a floor below which the sum cannot go, and that floor is the makers' adverse selection charge, and you cannot extract anything below it.

The framework lives at the interface between stage 1 and stage 2 of the [valuation funnel](/concepts/the-valuation-funnel). Stage 1 surfaces multi-outcome events with high displayed EE (`sf scan --by-overround`). Stage 2 is where you check the wall — pull the orderbook, count the outcomes, estimate the per-outcome adverse selection charge, compute the structural wall, and see how much of the displayed EE is above the wall. Most candidates die at this step. The ones that survive are the ones where the level above the wall is wide enough to cover slippage and fees, which is rare.

For the philosophy, see [the-overround-illusion](/opinions/the-overround-illusion). For the per-outcome economics, see the upcoming [maker-taker-regime-in-pms](/concepts/maker-taker-regime-in-pms) and [adverse-selection-on-prediction-markets](/concepts/adverse-selection-on-prediction-markets) concept pages, which formalize the maker compensation that the wall represents. For implementation, see [computing-event-overround-from-multi-outcome-events](/technicals/computing-event-overround-from-multi-outcome-events) — the computation is just a sum, but the *interpretation* is what this page is about.

The vig wall framing changed how I look at multi-outcome events. Before I understood the wall I would chase any cluster with EE > 0 and lose money on slippage. Now I look at the *delta* on EE — is it changing? — and I only trade when the change is decisive enough to mean something beyond the structural floor. That filter eliminates about 95% of the candidates I used to take. The remaining 5% are real, and they pay better because I am not wasting capital on the noise.