# Hazard Rate Anomalies in Temporal Series: When the Yield Curve Lies

> Cycle-clustered prediction markets form a hazard-rate curve over time. Sometimes the curve has anomalies that violate basic monotonicity. Those anomalies are arbitrage and they are sitting on the orderbook waiting for someone to read them.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Some mornings the yield curve on a prediction-market event family does not make sense. A contract resolving by November 30 is priced higher than the same-event contract resolving by December 31. That is mathematically impossible if the contracts are correctly specified — anything that resolves YES by November 30 also resolves YES by December 31, so the longer-dated contract has to price *at least as high*. When the data says otherwise, the data is telling you one of two things: a real arbitrage exists, or one of the two contracts is mis-cataloged.

The first time I noticed one of these anomalies on a SpaceX IPO event family I assumed it was a bug in the cycle-clustering grouper. Two hours of poking later, the bug was on the venue. Someone had crossed an orderbook by accident on the November contract, the price had stuck, and there was no incentive for any other trader to fix it because the November contract had almost no flow. That arbitrage paid 4 cents per dollar of notional on the side I cared about, and it sat there for the better part of a day.

This page is about how those anomalies form, how to detect them with the indicators already on the [pm-indicator-stack](/concepts/pm-indicator-stack), and how to actually trade them without getting stuck on the wrong side.

## What a Hazard Rate Curve Looks Like in the First Place

The cycle-clustering grouper (the [the-cyc-regex-grouper-walkthrough](/technicals/the-cyc-regex-grouper-walkthrough) technical walks through the regex set) takes a universe of binary contracts and identifies the ones that belong to the same event family. A Fed-decision family has a contract for each FOMC meeting. A SpaceX IPO family has a contract for each calendar quarter or year. A "Trump indicted by date X" family has a contract for each candidate cutoff date. The slugs change but the structure repeats: an event with a single underlying outcome, observed at multiple resolution dates.

If you sort the family's contracts by close timestamp and plot the YES price against the resolution date, you get a curve. The curve has a name borrowed from actuarial science — the *hazard rate* — and it represents the cumulative probability that the event will have happened by each date. By construction, the curve has to be monotonically non-decreasing. If P(event happens by Nov 30) is some value, then P(event happens by Dec 31) has to be at least that value, because every November-resolution YES is also a December-resolution YES.

You can pull the full curve through `/api/public/yield-curves/[event]` or compute it on the fly via `sf scan --event spacex-ipo --curve`. The endpoint returns one row per cluster member with the YES price, the close timestamp, the days remaining, the implied yield, and the orderbook depth at the moment of the snapshot. Plotted, it usually looks like a smooth contango curve — prices climb from near zero on the soonest-dated contracts to whatever the long-term consensus probability is on the furthest-dated contracts.

When the curve has a kink, a dip, or an outright inversion between two adjacent points, you have a hazard rate anomaly. That is the entry condition for everything in this page.

## How Anomalies Form

I have catalogued four mechanisms over the last year. They are not equally common, and they do not have equally tradable arbitrage windows.

**Mechanism 1 — illiquid sibling drift.** The most common case. A short-dated sibling has decent volume because the resolution is imminent and people are taking last-minute positions; the long-dated sibling has almost no volume because resolution is months away. The short-dated sibling has been re-marked by recent trading, the long-dated one is sitting at a stale price from when it was first listed. Over time the short-dated price drifts up on real flow while the long-dated price stays where the original makers parked it. Eventually you get crossed: short-dated > long-dated.

**Mechanism 2 — listing-time mismatch.** A new sibling gets added to an existing family weeks after the original siblings. The maker who lists it has a guess at where to seed the price, and that guess is sometimes wrong by 5+ cents. If the guess seeds the new contract too low, you have an instant inversion against any earlier sibling that has been re-marked by real trading. The mismatch usually fixes itself within 24-72 hours as flow finds the new contract, but in low-attention families it can persist for weeks.

**Mechanism 3 — crossed orderbook from a fat finger.** Someone sweeps through a thin sibling's orderbook with a market order that is disproportionate to the depth, and the resulting fill prints a price that does not reflect any real reassessment. The orderbook stays crossed because the affected contract is too illiquid for anyone to push it back. This is the "the November SpaceX contract just got hit for $400" case I described above.

**Mechanism 4 — venue-side resolution rule changes.** Rare but interesting. A venue clarifies the resolution rule on one sibling without updating the others. The clarified sibling reprices to reflect the new rule; the unclarified siblings stay at the old price. If the clarification effectively narrows what counts as YES, the clarified sibling drops; if it broadens, the clarified sibling rises. Either way, the cluster is now arbitrage because the contracts no longer have the same definition.

Mechanisms 1 and 2 produce arbitrage that you can trade with normal liquidity-taking orders. Mechanism 3 produces arbitrage that disappears the moment anyone larger than you tries to capture it (the orderbook re-stacks). Mechanism 4 is *not really arbitrage* — it is a definitional gap, and trading it without understanding the rule change is how you take a position you cannot justify.

## Detecting Anomalies in Practice

The [pm-indicator-stack](/concepts/pm-indicator-stack) does not have an "anomaly score" indicator, but you can derive one cheaply from the existing data. The procedure I use:

1. Pull the cluster via `/api/public/yield-curves/[event]`. You get back N rows, sorted by close timestamp.
2. For each adjacent pair (i, i+1), compute `anomaly_i = price_i − price_{i+1}`. A positive number means the earlier-dated contract is priced higher than the later-dated one — an anomaly.
3. Filter to anomalies above a threshold. I use 2 cents because anything smaller gets eaten by fees. A 4-cent anomaly is interesting; an 8-cent anomaly is suspicious enough that I want to verify the cluster definition before trading.
4. For each surviving anomaly, pull the orderbook on both contracts. Verify the depth supports an actual fill at the displayed prices. This is where most "anomalies" die — the displayed mid is fictional because there is no real bid-ask within 10 cents of it.

You can run the whole loop in about a minute on a cluster of 8-12 siblings via `sf scan --event spacex-ipo --check-anomalies`. The output is the list of crossed pairs that survived the depth check. On a typical day the SpaceX cluster has zero anomalies. On a bad day for one of the siblings it has one or two, and they are usually mechanism 1 or mechanism 2.

The harder detection problem is *false positives from cluster mis-grouping*. The CYC regex grouper is right about 41% of the time on uncatchable winner-pick events; the rest get either mis-grouped or left as singletons. If the grouper has put two contracts in the same family that *do not actually have the same definition*, you will see what looks like a violation of monotonicity but is really two unrelated contracts. Always sanity-check the grouping by reading both ticker descriptions before you trade.

## Trading Anomalies Without Getting Stuck

The naive arbitrage trade on a hazard rate anomaly is: short the over-priced sibling, long the under-priced sibling, hold to whichever resolves first, collect the difference. In theory this is a riskless trade because the long position covers the short position by construction.

In practice it is not riskless, and the reason is venue-specific.

**On Kalshi**, you cannot short directly — you have to express the short by buying the NO side of the contract. NO buyers pay for the NO side, which is priced at 100 − YES price. So shorting the over-priced YES at 0.55 means buying NO at 0.45. You have a long position in NO on the November contract (cost: 0.45 per contract) and a long position in YES on the December contract (cost: 0.51 per contract, in the example where Nov is 4¢ over Dec). Total cost per pair: 0.96. Maximum payoff: 1.00 (one of the two positions wins exactly $1). Realized profit: 0.04 minus fees.

**On Polymarket**, the math is the same in spirit but the AMM curve makes the entry costlier than the displayed mid suggests. A 4-cent anomaly on the displayed mid prices is often a 2-cent anomaly after AMM slippage on both legs. Anomalies under 5 cents are rarely worth trading on Polymarket purely because of the AMM friction.

**On both venues**, the position has *one risk you have to actually price*: the November contract resolves first. If it resolves NO (which is what you are betting on by holding NO) and the December contract is still trading, you have to wait for December to resolve to collect on the YES leg. During that wait, the December contract can move against you on news that affects only December, not November. The arbitrage is only riskless if the two contracts have the same underlying outcome — which they should by construction, but in mechanism 4 cases (resolution rule change) they no longer do.

The procedural rule I follow is: never trade an anomaly without first reading both ticker descriptions in full and confirming the resolution criteria are identical. Mechanism 4 anomalies are not arbitrage; they are paying you 4 cents to take definitional risk. The other three mechanisms are arbitrage and they pay reliably, but only on small position sizes. Trying to size a $10K trade into a $400 anomaly is how you become exit liquidity for the other side.

## Where the Lens Breaks

Three honest caveats.

**The curve assumes the family is actually a family.** If the cycle-clustering grouper has put two unrelated contracts in the same cluster (mechanism 4 in disguise), every "anomaly" you see is fake. Cluster verification is not optional. The grouper covers about 41% of event families cleanly; the rest have some mis-grouping risk that you have to absorb manually.

**Anomalies under 2 cents are noise.** Bid-ask spreads on most siblings are 1-3 cents; the displayed mid swings within that range without anyone trading. Filtering at 2 cents is the minimum threshold to avoid chasing spread micro-structure. Fees and slippage will eat anything smaller before you can collect.

**The anomaly detector is not the [cliff-risk-index](/learn/cliff-risk-index) detector.** Both look for "things that look unusual" on the indicator stack, but they are measuring different things. CRI flags markets that are *moving fast*. The anomaly detector flags clusters that are *internally inconsistent*. A cluster can have high CRI on every member (everyone is moving) and zero anomalies (the moves are coordinated). Conversely, a cluster can have low CRI on every member (nothing is moving) and a persistent anomaly (one member has been mis-priced for weeks). Read both signals and treat them as orthogonal.

## How This Connects to the Stack

Hazard rate anomalies are the most concrete example of why the cycle-clustering grouper exists in the first place. Without CYC, you cannot construct the curve, and without the curve you cannot detect the anomalies. The [why-cycle-clustering-is-the-most-pm-native-indicator](/opinions/why-cycle-clustering-is-the-most-pm-native-indicator) opinion piece argues that this is the indicator that justifies the rest of the stack — and the trading examples on this page are the proof of that argument.

The detection workflow lives inside stage 1 of the [valuation funnel](/concepts/the-valuation-funnel): you scan the universe of clustered events, you compute the anomaly score per cluster, you filter to anomalies above the 2-cent threshold, you pull the orderbooks for the survivors. That is roughly 47,000 markets compressed into 3-10 candidate trades in about ninety seconds. From there it is stage 2 (the depth check) and stage 3 (the cluster verification).

For implementation, see the [computing-cliff-risk-index-step-by-step](/technicals/computing-cliff-risk-index-step-by-step) and [computing-event-overround-from-multi-outcome-events](/technicals/computing-event-overround-from-multi-outcome-events) technicals — the second one is structurally identical to anomaly detection because both are sums or differences across sibling sets. For the philosophy, see [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets), which argues that prediction markets should be priced with bond-desk vocabulary, and a hazard rate curve is exactly that vocabulary applied to event-driven contracts.

Anomalies are not common enough to be a primary strategy. They show up on maybe 1-2% of clusters in any given week, and most of them are too small or too thin to trade. But when one is real, the math is clean and the payoff is calculable in advance. They are also one of the few places in prediction markets where you get to use the word "arbitrage" without a footnote.