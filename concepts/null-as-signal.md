# Null Is a Signal, Not a Defect: Reading Missing Data on Prediction Markets

> When LAS is null, when EE is null, when PIV is near zero — the absence of data is sometimes the entry condition. Four null patterns and the four maker strategies they unlock.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
When I first wired up the indicator stack, every new user complained about the same thing. "Why are half the LAS values null? Is the cron broken?" "Why does this market show CVR = null when other markets in the same family have CVR values?" "Why is PIV exactly zero on contracts that I know are trading?" The default assumption was always that the system was broken. The default fix was always "we should fill in the nulls."

That fix would have destroyed the most useful thing the indicator stack does. Half the strategies in my book live inside the null states — and the moment you mistake the null for a defect, you lose the information that is hiding in the absence.

This essay is about the four null patterns, what they actually mean, and why each one is the entry condition for a specific maker strategy.

## The General Principle

Every numerical indicator can return one of three things: a value, a sentinel (zero, infinity, negative), or null. Most data systems treat null as a degraded value — something to be filled, imputed, or treated as missing. Most quant pipelines have explicit null-handling logic that quietly turns nulls into zeros or last-known values, on the theory that "more data is better than less data."

That theory is wrong for prediction-market indicators. Null on a Tier B indicator does not mean "the value would have been low if we had computed it." It means "we did not compute the value, and the reason we did not compute it is itself a signal." The null is an *index into a state machine*, and the state machine has a name for each null condition.

Concretely, in the [pm-indicator-stack](/concepts/pm-indicator-stack):

- **LAS = null** → the warm-regime cron has not warmed this market, which means it is below the top-500-by-volume threshold, which means *no one has been looking at it.*
- **EE = null** → the event has no clustered sibling outcomes, which means either the event is genuinely binary or *the cycle-clustering pass has not grouped its siblings yet*.
- **PIV ≈ 0** → the 1¢-delta tracker has registered no qualifying moves over the rolling window, which means the *price has been stationary*. This is technically a real value, not a null, but it acts like one.
- **CVR = null** → no semantic-neighbor graph has been built for this market, which means the *thesis has not propagated* and the market is uncontaminated.

Each of these states is an entry condition. Reading them as defects throws away the entry.

## Pattern 1 — LAS Null and the Virgin Polymarket Strategy

The warm-regime cron computes LAS for the top 500 markets by 24-hour volume on each venue. Those 500 markets are the loud ones: presidential elections, big sports events, major Fed contracts, viral news markets. Everything else — the other ~46,500 active markets — has LAS = null because the cron has never run on them.

The standard interpretation of "null LAS" is: this market is illiquid and you should avoid it. The interpretation that pays is: this market is being ignored by the people who scan for liquid markets, and *if you put a maker quote on it, you are alone on the book*.

The virgin Polymarket strategy is the named version of this. Find a Polymarket market with LAS = null, verify by hand that the orderbook is sparse (one or two stale resting orders, no one within 5¢ of the mid), and then put up a tight two-sided maker quote. If you get filled on either side, the small amount of organic flow that does pass through the market gives you the spread. If you do not get filled, you do not lose anything but cancel-cost.

The strategy works because the market has a structural property: a small amount of organic flow keeps trickling in (people actually do trade these markets, just slowly), and you are the only person providing the liquidity that flow needs to cross. The fills are infrequent but the spread is wide and the inventory risk is low because the market is not actively repricing — by definition, a market with LAS = null has no one rushing in or out. You are the orderbook.

I have made consistent low-double-digit returns on this strategy over the last six months, and the entry filter is *the explicit null check on LAS*. If I imputed the null with a low-LAS estimate, I would still find these markets — but I would also pull in dozens of low-LAS-but-not-null markets where the orderbook is *almost* empty rather than actually empty, and the strategy economics break down because there is already a maker present. The null is the precise marker for "no one is here."

## Pattern 2 — EE Null and the Uncategorized-Event Edge

EE (event overround) is the sum of YES prices across mutually exclusive outcomes in a single event. It requires the system to know that two markets are siblings — that they belong to the same event family. This is what the cycle-clustering grouper does (the [the-cyc-regex-grouper-walkthrough](/technicals/the-cyc-regex-grouper-walkthrough) technical explains the regex set).

The grouper's coverage is roughly 41%. The other 59% of the universe is markets that the grouper has not (or cannot) categorize into an event family. For all of those markets, EE is null — not because the math fails, but because there is no sibling set to sum over.

Most users see "EE = null" and assume the market is broken or unclassified. The trader-friendly read is different: a market with EE = null is a market where *no one has been doing the cross-outcome arbitrage check*, because the system that would surface the arbitrage cannot see the siblings. If you can manually identify the sibling set — by reading the market description, by knowing the event structure from outside the venue, by being a domain expert — you have an information advantage over every other trader who is filtering on "show me events with non-null EE."

The strategy here is: scan the EE = null universe for events you happen to recognize, manually compute the sum across siblings, and act on the dispersion. The system will eventually catch up (the grouper improves over time), and the moment it does, the EE value becomes visible to everyone and the edge dissipates. Until then, your domain knowledge is the bridge across the null.

I have caught structural dislocations this way maybe a dozen times in the last year. Each one was a small profit, none of them were huge — but each one was alpha I would not have had if the system had imputed EE rather than left it null.

## Pattern 3 — PIV Near Zero and the Range MM Strategy

PIV (position-implied velocity) measures how aggressively positions are being added or removed, derived from the 1¢-delta tracker over a rolling 7-day window. A value near zero means the price has not crossed the 1¢ threshold in either direction often enough to register meaningful flow. That can mean two very different things:

1. The market is dead — no one is trading it, the price is stuck because there is no participation.
2. The market is *consolidating* — there is participation, but the buy and sell flows are roughly balanced, the price is range-bound, and the 1¢ moves are happening within a tight band that the tracker classifies as noise.

Both states give you PIV ≈ 0. The strategy lives in case 2.

A market that is consolidating in a tight range is the textbook setup for a range-bound market-making strategy. You quote both sides of the range, expecting the price to bounce between your bid and your ask, capturing the spread on each round trip. The entry filter is exactly "low PIV with non-null other indicators" — non-null IY, non-null CRI, non-null EE — because non-null other indicators tell you the market is being observed and traded, and low PIV tells you the trading is balanced.

Distinguishing case 1 from case 2 requires a quick sanity check: is there *any* trade volume in the last 24 hours? If yes, you are in case 2. If no, you are in case 1 and you should walk away (or, if it is on Polymarket, treat it as a virgin-market candidate from pattern 1 instead).

The thing the PIV-near-zero filter does that nothing else does is *find the markets that look stable from indicator velocity but have actual underlying flow*. A lot of markets are stable because they are abandoned. A few are stable because the buyers and sellers are perfectly matched. The second set is where the range-MM strategy makes money, and the PIV indicator is the only way I know to surface them efficiently.

## Pattern 4 — CVR Null and the Uncontaminated Thesis

CVR (contagion velocity rate) measures how fast a thesis is spreading from one market to its semantic neighbors. It depends on a graph of semantically related markets, which the system builds for major event clusters but not for everything. CVR = null means the system has not constructed the graph for this market — typically because the market is in a domain the contagion crawler does not cover, or because it is too new to have neighbors yet.

The trader interpretation: a market with CVR = null is *uncontaminated* by thesis flow from elsewhere. Whatever thesis is going to drive its price has not yet propagated through the network. If you have a thesis from outside the system — from your own research, from a specific source you trust, from an event you witnessed firsthand — you can enter a CVR = null market *before* the thesis arrives via the normal information channels.

This is the rarest of the four patterns and the most fragile. Two specific risks: (a) "uncontaminated" can also mean "nobody cares," in which case you are right but never get paid because the thesis never propagates; (b) the moment any contagion does happen, CVR becomes non-null and the edge is by definition consumed, so you have to be in the position before the indicator updates.

The way I use this filter is as a tiebreaker. When I have two otherwise-similar candidates after stages 1 and 2 of the [valuation funnel](/concepts/the-valuation-funnel), and one has CVR = null and the other has CVR > 0, I prefer the null one because it has more thesis room ahead of it. The null is a *temporal* signal — it tells me how much of the run is still in front of the price.

## The Meta-Pattern

The four patterns share a structure. In each case, the indicator returns null because the system did not perform a computation that *most other traders are filtering on*. Most other traders have a "show me only markets with valid data" toggle on by default. By switching that toggle off and *specifically* filtering for the null states, you are looking at exactly the universe that everyone else has hidden from themselves.

That is the moat. The moat is not in the indicators themselves — every serious trader will eventually compute IY and CRI. The moat is in *what you do with the absence of indicators*. The absence is invisible to anyone whose system is designed to fill it in, and it is highly visible to anyone whose system is designed to surface it.

The [pm-indicator-stack](/concepts/pm-indicator-stack) is built around this principle. The Tier B indicators are sparse on purpose. The warm-cron coverage is intentionally limited. The grouper's 41% coverage is intentionally not 100%. Each of these intentional gaps creates a null state that becomes a tradable filter.

## Where This Reasoning Breaks

A few honest caveats.

**Null can also be a real bug.** Sometimes the cron really did break, sometimes the grouper really did fail in a way that should be fixed. The way to distinguish "principled null" from "bug null" is to verify that the null state is consistent with the documented coverage rules. If a market that *should* be in the warm-cron cohort (top 500 by volume) is showing LAS = null, that is a bug. If a market that is well below the top 500 is showing LAS = null, that is correct. Always check before treating null as signal.

**Some null states are unstable.** A market might have CVR = null today and CVR = 0.4 tomorrow because the contagion crawler caught up. Strategies that rely on null states as entry conditions need to monitor for the null collapsing into a real value, and the collapse is the natural exit signal.

**Null-state strategies are low-throughput.** The whole point of these patterns is that they live in the long tail. You are not going to find a hundred virgin-Polymarket setups per day. Maybe two or three. The economics depend on you being patient and only taking the cleanest setups, not on volume.

**The indicators that return null are not random.** They are specifically the Tier B indicators that depend on cron coverage or graph construction. Tier A indicators (IY, CRI, EE_when_siblings_exist) are always computable from raw price data, so you will never see them null on a market that has any history at all. If you are seeing Tier A nulls, that is a real bug.

## How This Connects to the Stack

The null-as-signal principle is one of the four pillars of the framework, alongside the [valuation funnel](/concepts/the-valuation-funnel), the [pm-indicator-stack](/concepts/pm-indicator-stack), and the [endogenous-vs-reality-vs-opinion-data](/concepts/endogenous-vs-reality-vs-opinion-data) three-source axis. The capstone [prediction-market-valuation-theory](/concepts/prediction-market-valuation-theory) explains how all four interlock.

In implementation terms: you will see the null-handling logic show up in [computing-implied-yield-from-kalshi-tickers](/technicals/computing-implied-yield-from-kalshi-tickers) (where IY returns null on edge cases), in the upcoming [computing-liquidity-availability-score-from-orderbook](/technicals/computing-liquidity-availability-score-from-orderbook) (which explicitly returns null when the warm cron has not run), and in the corresponding upcoming opinion piece [null-data-is-not-missing-data](/opinions/null-data-is-not-missing-data) where I argue this principle in less technical terms.

The CLI handle for filtering on null states is currently `sf scan --las null` and `sf scan --cvr null`. Both flags are intentional — they filter *for* the null, not against it — and they are how I find most of the maker setups I run in any given week.

The framework I keep coming back to is this: if your data system silently fills in nulls, your data system is robbing you of half its potential information. The right design is to let nulls propagate, surface them as filterable states, and treat them as entry conditions for strategies that are *only* visible when most other traders have looked away.