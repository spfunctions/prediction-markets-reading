# Why Cycle Clustering Is the Most Prediction-Market-Native Indicator

> Yield-curve thinking borrowed from bond traders is great. CYC is the indicator no equity desk has ever needed, because no equity has the discrete event-resolution structure that makes term-structure trading on a single underlying possible.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 11 min

---
Every other indicator in the stack has a parent in fixed income or sports betting. Implied Yield is a treasury concept. Cliff Risk Index is a flavor of convexity. Event Overround comes from horse racing. Liquidity Availability Score is just bid-ask depth with a haircut. None of these are original to prediction markets — they are imports, dressed in slightly different vocabulary.

Cycle Clustering is the exception, and that is why it is the most prediction-market-native indicator in the stack. There is no equity analog. There is no fixed-income analog. There is barely a sports-betting analog, and the closest one (futures-vs-spot odds on the same team) is so impoverished compared to what CYC makes possible that it does not really count. CYC is the indicator that exists because prediction markets have a feature no other instrument has, and the feature is *discrete event resolution on a continuous calendar*.

## What CYC Actually Does

Cycle Clustering is a regex grouper. It scans the catalog of every active contract on Kalshi and Polymarket and groups them into event families based on slug patterns: `KXIPOSPACEX-26MAY01`, `KXIPOSPACEX-26JUN01`, `KXIPOSPACEX-26JUL01` all belong to the same family because they share the prefix and differ only in the date suffix.

Once contracts are grouped, you can plot them. The x-axis is τ-days. The y-axis is YES probability or — better — implied yield. What you get is a yield curve. Not a yield curve in the metaphorical sense. A literal one. Each point is a binary contract in the same event family. The shape of the curve tells you what the market believes about the timing of the event.

I built the first version of this in October 2025 to look at the SpaceX IPO complex. Polymarket and Kalshi both have a series of contracts asking "will SpaceX IPO by the end of month X?" running from the next month out to roughly two years. There were eighteen contracts in the family. The YES prices ran from 0.02 (next month) to 0.61 (two-year horizon). I plotted them as a curve and the shape was *steep contango with a knee at 14 months*.

That shape is information no single contract on its own can give you. The knee said the market was assigning most of its IPO probability to the 12-18 month window. The steep early section said the near-term contracts were essentially priced as out-of-the-money options. The flatter back section said the market thought 2027 was the year and 2028 was almost a foregone conclusion conditional on it not happening in 2027.

I traded the back-end of that curve. The trade worked. But the thing I want to emphasize is that the trade was *only visible because the curve existed*. Looking at any single SpaceX-IPO contract in isolation gave you a number. Looking at the family gave you a story.

## The Equity Comparison That Doesn't Work

The closest parallel in equity markets is the options-implied volatility surface for a single underlying. You can plot SPX implied vol against strike and expiry and get a surface that has shape, gradients, and inflection points, and the shape tells you what the market thinks about the distribution of future returns.

The IV surface is a great instrument and has been a major edge for derivatives traders for thirty years. But notice what it is *not*. It is not a curve over a single, named, discrete event. The IV surface aggregates *all the things that could move the price of SPX*: earnings, macro data, geopolitics, flows, fed meetings, everything. The surface is a smear over an unknowable number of catalysts.

A SpaceX-IPO yield curve in prediction markets is a curve over *one* catalyst, and the catalyst is named, dated, and verifiable. The market is saying "here is what we think the timing distribution looks like for this single event." That is qualitatively different from anything the IV surface gives you.

The closest fixed-income analog is even worse. A treasury yield curve plots yield against maturity for *the same issuer*, and what changes along the curve is the duration of the rate exposure. There is no "event" being priced. Every point on the curve is the same instrument with a different time horizon, and the shape of the curve tells you about expectations of *rates*, not about expectations of any specific dated thing happening.

CYC gives you a yield curve where every point is a *different thing happening at a different time*. Every contract is a discrete bet on a specific date. The curve aggregates the market's full distribution over when the event will resolve. That is structurally impossible to build with equities or bonds, because neither instrument has discrete dated event resolution as its core mechanic.

## Why the Regex Approach

People ask why CYC is a regex grouper instead of an LLM-based one. The honest answer is that I tried both, and the regex one wins on every dimension that matters.

The 9 patterns in `lib/indicators/cyc-grouper.ts` (by-monthDay-year, iso-date, quarter-year, year-only, before-month, by-end-of-year, month-year, season-year, monthDay-only) catch about 41.4% of the active universe at any given moment. The rest is uncatchable in regex because it is winner-pick events ("who will win the X primary"), unstructured narrative events ("will inflation hit 3% before recession"), or one-off contracts that don't have siblings.

The 41.4% coverage feels low until you realize that the *value* of CYC is concentrated in the high-coverage families: SpaceX IPO timing, Fed meeting cuts, presidential primary cycles, monthly economic data releases. These are the families where the curve has the most points and the shape carries the most information. The 58.6% you cannot regex-cluster is mostly events where there is no curve to draw because there are no siblings.

LLM-based grouping was tempting. It could in principle catch more of the long tail. It also costs about 50× more per scan, runs slower, makes the catalog non-deterministic, introduces false positives that are hard to debug, and forces you to revalidate the cache every time the model updates. For a clustering job that runs every 15 minutes against a 47K-contract universe, the regex approach is the only one that scales economically. Every time I have tried to "improve" CYC with an LLM it has been a regression on at least one of latency, cost, or determinism.

The deeper insight is that the regex approach works *because the venues themselves use templated slug schemas*. Kalshi names contracts the way they do for a reason — the convention is built for machine consumption. CYC is just reading that convention back. An LLM grouper would be solving a problem that the venues already solved.

## The Counterargument

The strongest pushback I get is "yield-curve thinking is great for things that look like bonds, but most prediction markets are one-off binary events with no sibling family. CYC only matters for the small subset of markets that happen to have a templated structure."

This is correct on the data and wrong on the conclusion. Yes, CYC only catches about 41% of the universe. But the 41% it catches is *exactly* the subset of the universe that has tradable curve shape — which is also exactly the subset where the most institutional capital ends up. SpaceX IPO timing is a curve. Every Fed meeting cycle is a curve. Every presidential primary calendar is a curve. Every "will X happen by end of quarter Q" series is a curve. The volume in prediction markets is concentrated in these families precisely *because* they are the contracts that institutional desks understand.

The one-off binary event is the kind of contract retail trades. The curve-shaped event is the kind of contract that justifies a multi-million dollar position. CYC is what makes the second category accessible, and the second category is where the money is.

The other counterargument is "you can already build curves manually by looking at a single event's contracts." Sure, you can. You can also build a treasury yield curve by hand from individual bond quotes. The reason desks don't is that doing it by hand for 200 events at 15-minute intervals is impossible. CYC automates the grouping, which is the whole point.

## Where CYC Breaks

Three caveats.

First, the regex set will misgroup contracts that share a slug prefix but resolve on independent events. The classic case is when a venue reuses a contract template across two different underlying questions. You catch this in code review of the patterns, not at scan time, and the fix is always to tighten the regex.

Second, the curve is meaningless on event families with fewer than three siblings. With one or two contracts, you don't have a shape — you have a line segment. CYC excludes families below a minimum count by default, which is right.

Third, CYC tells you the *shape* of the curve but not whether the shape is mispriced. To turn shape into a trade you need a separate view on what the shape *should* look like, and that view comes from causal reasoning about the event itself, not from CYC. The regex grouper is the substrate; the trade thesis is still on you.

## The Habit

The habit is one query a week. Run `sf scan --cyc-curves --min-siblings 5` and look at every event family that has five or more sibling contracts. For each one, plot the curve. Ask whether the shape makes sense given what you know about the event.

A flat curve says the market has no view on timing. A steep contango says the market thinks the event is more likely far out than near. A backwardated curve says the market thinks the event is more likely soon than later. Each shape is a different trade. None of the shapes is visible from a single contract.

CYC is the indicator that justifies the whole stack. Without it, prediction markets are a long list of binary contracts that look like sports bets with extra steps. With it, prediction markets are a term-structure surface for discrete events, which is an instrument that does not exist anywhere else in finance. The bond desks are not coming for this, because they cannot build it. We are the only ones who can.