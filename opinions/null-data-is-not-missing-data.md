# Null Data Is Not Missing Data: How to Read EE=null, LAS=null, PIV≈0

> The most common bug in new prediction-market traders is treating a null indicator as a system error. Each null state is a positive entry condition for a specific strategy. Stop filing tickets and start reading the field.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 11 min

---
In the first month of running the indicator stack in production, I got the same Slack message from new users about ten times: "I think your scan is broken, half the markets show LAS = null." It was not broken. The null was the most useful thing on the row. I just had not written down anywhere that the null was *information*, so every new user assumed it was a defect.

This essay is the writeup I wish I had pinned on day one. The thesis is short. Each of the four "null" states in the indicator stack is a positive entry condition for a specific strategy. Treat them as ordinary signals. Better — treat them as the highest-conviction signals in the stack, because they are the cleanest filter for unloved markets, which is exactly the territory where edge accumulates.

## The Four Nulls and What Each One Means

The indicator stack has four sparse fields: LAS, PIV, CVR, EE. Each one can be null, and the null on each one means a different thing. The four meanings are not interchangeable, and you need a habit of reading which null you're looking at before you can decide what to do.

**LAS = null** means the warm-regime cron has not warmed this market. The cron only computes Liquidity Availability Score for the top 500 markets by 24-hour volume. Everything below that line is intentionally not scored, because there is no orderbook depth worth modeling. The signal: this market is in the bottom 99% of the volume distribution. No one is paying attention to it.

**PIV ≈ 0** means there have been no 1-cent-or-more price moves in the trade tape over the last 7 days. Position-Implied Velocity is built on the 1-cent-delta tracker in `market_indicator_history`, and a near-zero value means the price has been pinned. The signal: the market has no aggressive position-builders right now. Either the question is dead, or the question is so consensus that no one is willing to take the other side.

**CVR = null** means no neighbor-market signal propagation has been detected for this market. Contagion Velocity Rate measures how fast a thesis spreads from one market to its semantic neighbors, and a null value means the thesis has not been seen migrating. The signal: this market is *uncontaminated* by the broader conversation. Whatever it is pricing, it is pricing in isolation.

**EE = null** is the simplest of the four. It means the market has no sibling outcomes — the event family has exactly one contract, so there is no overround to compute. The signal: this is a standalone binary, not part of a multi-outcome event. There is no neighbor pricing to anchor against.

These four signals are all different. I have seen new users conflate them — "everything is null, the system is broken" — and the conflation costs them the entire layer of information in the negative space. Read the specific null. There are four cases. Each maps to a different strategy.

## The Mapping to Strategies

This is the part that took me a year to articulate cleanly. Each null is the entry condition for a specific maker strategy. The four strategies in my MM book are:

**Range MM (PIV ≈ 0).** When the price has been pinned for seven days, you post a tight market-maker quote inside the existing range and collect the spread on whatever volume materializes. The PIV near zero is the precondition — it tells you the market is range-bound enough that quoting both sides has positive expected value. If PIV is high (recent aggressive position-building), range MM gets run over. If PIV is near zero, the range is real and your spread capture is real.

You run this on contracts where you have *no view* on the underlying. You are not betting on direction. You are betting on the absence of direction, which is exactly what PIV ≈ 0 confirms.

**Virgin Polymarket (LAS = null).** When LAS is null on a Polymarket contract, the warm cron has not touched it, which means it is in the bottom 99% by volume, which means the orderbook is effectively empty. The strategy is to *become* the orderbook. Post a maker quote 1-2 cents wide, wait for retail flow to find the market, collect the spread on the small volume that materializes.

You run this on Polymarket specifically (not Kalshi) because Polymarket's AMM-plus-orderbook structure means there is always a baseline of automated flow that has to interact with whoever is on the book. If you are the only resting order, you eat that flow for as long as the contract stays in the bottom 99%. Once it climbs into the top 500 — which usually means a news event has happened — you exit, because the warm cron will warm it and other quotes will arrive.

**Standalone Edge (EE = null).** When EE is null because there are no sibling outcomes, you cannot use overround as a signal. What you *can* do is use the standalone binary as an isolated thesis trade, freed from the constraint that the price has to be coherent with neighbors. This is a small but real edge category — there are a handful of contracts on Kalshi every week that are genuine one-offs (an obscure court ruling, a single-question regulatory decision) where the lack of a sibling family means the price drifts on whatever flow happens to find it.

You run this on contracts where you have a strong external thesis (causal-tree work on the underlying event) and the absence of a sibling family means there is no anchor pulling the price toward consensus. You take the directional view, you size it small, you accept the binary outcome.

**Uncontaminated Thesis (CVR = null).** When CVR is null because no neighbor-market signal has propagated to this contract, you are looking at a market the broader conversation has not yet found. This is the highest-conviction null. It means whatever price is on the screen has been set without contamination from the wider thesis-spreading network. If you have an independent view, you are getting the market pre-consensus.

You run this when you have done the causal-tree work yourself, independently, and you can verify that the contract's current price is not yet reflecting the thesis you are about to trade on. The CVR null is the certifying signal that no one else has done the same work and pushed the price.

## The Worked Example

Here is the trade I put on in late February to make the framing concrete. Polymarket contract on a state-level regulatory decision: `will-state-x-regulator-approve-y-by-may-1`. Five days into looking at it, the indicator dashboard showed:

- LAS: null
- PIV: 0.012 (essentially zero)
- CVR: null
- EE: null
- IY at displayed mid: 142% annualized
- τ: 67 days

Three of four nulls, and the one non-null was a near-zero PIV. Every null was telling me the same thing in different vocabulary: this market is in the territory the system specifically does not score, because no one is looking at it. It is bottom-99%-by-volume (LAS = null), it has no recent price aggression (PIV ≈ 0), no sibling outcomes (EE = null), and no thesis contamination from neighbor markets (CVR = null).

I had done the causal-tree work the week before. The regulatory decision had a clear timeline, a clear administrative process, and a clear precedent that suggested the market was mispricing the probability. The IY at the displayed mid said the market was paying me 142% annualized to take the directional view.

I posted a small maker bid 1 cent below the displayed ask. Within four days my bid filled — twice, in two separate fills. The aggregate position was about $180 on what turned out to be a successful resolution. The annualized return on the realized trade came in at 96% (lower than the displayed 142% because of the price I paid versus the mid).

The trade was small and the dollars were not the point. The point is that *every signal that mattered was a null*. If I had filtered the scan on "no nulls," this market would not have appeared. The null itself was the entry condition, and once I learned to read which null was which, I started finding these regularly.

## The Counterargument

The strongest objection is reasonable: "If null means 'we have no data,' then trading on null is trading on absence of evidence, which is not evidence of absence. You are reading information into the gaps that the gaps do not contain."

This objection would be correct if the nulls were random — if the system *failed* to compute LAS on some markets due to bugs or rate limits in a way that was independent of the market's actual properties. But the nulls are not random. They are *correlated with the very property you are using them to find*.

LAS is null specifically when the market is in the bottom 99% by volume. That is the definition. The system did not fail to score it — the system intentionally skipped it because the score would be meaningless. The fact that a market has LAS = null is *the same as* the fact that the market is in the bottom 99% by volume. Reading the null as "this is an unloved market" is not reading information into a gap. It is reading the gap correctly as a label.

Same with EE = null on a standalone binary. The null is not a missing computation. The null is the system reporting "this market has no siblings." That is a fact about the contract, surfaced through the absence of a number. You are not making things up by reading it.

PIV ≈ 0 and CVR = null are the same logic. Each null state is a *consequence* of a specific property of the market, and the property is exactly the property the corresponding strategy wants to find.

The other counterargument is "but you can't run a backtest on null fields." Half right. You can't compute average IY on a null IY field, but you absolutely can compute base rates of resolution on the *subset of markets where IY is null* — and that subset is a perfectly well-defined population. The subset has properties. You can backtest the strategy that takes positions only on that subset. I have. The numbers are good enough that the strategy has been in production for four months.

## Where the Framing Breaks

Two caveats. They matter.

First, the null-as-signal framing only works on indicators that have a *deterministic null condition*. LAS, EE, CVR, and PIV all have explicit, well-defined reasons to be null. If a future indicator gets added that has *random* nulls — say, a metric that fails to compute due to upstream API errors — those nulls are not signals. They are bugs. Read the system documentation for each indicator before treating its null as information.

Second, the strategies that exploit nulls are inherently small-volume strategies. By construction, the markets where the indicators are null are markets nobody is looking at, which means the available volume is tiny. Every null-strategy trade I have run has been in the $50-$500 range. None of the four strategies scales to size. They are valuable as a discipline of finding edge in the unattended corners of the market, not as a primary income source.

If you need a strategy that scales to five-figure positions, you want LAS-positive, PIV-active markets, and you want exactly the opposite of what this essay describes. The null strategies are for the part of the day where you are mining the long tail. They are real, they pay, and they are intellectually satisfying — but they are not the main game.

## The Habit

The habit is to read which null you are looking at every time you see one. Not "is this null." Which one. LAS, PIV, CVR, EE. Each maps to a strategy. The strategy decides whether the row is a trade or a skip.

I run `sf scan --null-mode` every morning, which surfaces just the markets with at least one null indicator and tags which null fired. The list is long — a few hundred contracts on a typical day — and most of them are skips. The ones I trade are the ones where the *combination* of nulls matches a strategy I have a thesis for, and where the residual non-null indicators (the IY at mid, the τ) make the math work.

Stop reporting nulls as bugs. Start reading them as labels. The system is already telling you which markets the rest of the system is not paying attention to. That is exactly the list you wanted.