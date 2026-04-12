# Expected Edge: How to Combine IY, CRI, and EE Into One Action

> The composition rule. Filter by IY, then check CRI, then verify EE. The hierarchy is not optional. A worked walkthrough from 47K markets to one trade.

**Category:** indicator | **Author:** Patrick Liu | **Reading time:** 15 min

---
Each of the indicators in the SimpleFunctions stack does one job. [Implied yield](/concepts/implied-yield) tells you whether a contract pays enough to bother with. [Cliff risk index](/concepts/cliff-risk-index) tells you whether the contract is alive or stuck. [Event overround](/concepts/event-overround) tells you whether the multi-outcome sibling set is pricing cleanly. None of them, on their own, tells you what to trade. The thing that tells you what to trade is the composition.

Expected edge is the name I use for the composed signal — the output you get after running every relevant indicator in the right order against a candidate. It is not a single number. It is a sequence of filters, applied in a specific order, producing a small set of contracts that survive all of them. The hierarchy is not optional, and collapsing it is the most common mistake newcomers to the indicator stack make.

## The Composition Rule

```
expected_edge(market) =
    iy_filter(market)
  ∘ cri_filter(market)
  ∘ ee_check(market.event)
  ∘ judgment(market.context)
```

Each stage takes the output of the previous stage and either passes it through or rejects it. The order matters. Reordering the stages produces wrong answers, and not in a subtle way.

Three properties to notice.

First, the composition is *short-circuiting*. If a contract fails the IY filter, you never compute its CRI. If it passes IY but fails CRI, you never check its EE. The cheap filters run first, the expensive ones run on whatever survives. This is purely a performance optimization — running CRI on 47K markets when 99% of them have IY < 10% is wasted compute — but it has a side benefit: it forces you to think about the funnel as a hierarchy, not as a parallel scan.

Second, the composition is *not commutative*. Running CRI before IY produces "active markets," not "active markets that pay enough to trade." The two sets overlap heavily but are not the same. The active-but-low-yield markets are events the world cares about (sports, weather, election narratives) but where the price has compressed enough that the math doesn't pay. You don't want them in your output. You want them filtered out *before* you spend attention on them, which means IY runs first.

Third, the composition has *judgment as the last stage*. Steps 1-3 are computational — they run on indicator math without any human in the loop. Step 4 is "Patrick reads the orderbook and decides." There is no formula at this stage. The whole point of the funnel is to compress the universe down to a small enough set that judgment is feasible. If you skip the computational filters, you cannot apply judgment to 47K markets in any honest way. If you skip the judgment, you are running on indicators alone, and every false positive in the indicator stack becomes a real trade.

## Why the Order Matters

I want to spend a moment on why "IY first, then CRI, then EE" is the right order and not, say, "CRI first to find active markets, then IY to see which pay."

Suppose you ran CRI first. The top of the CRI scan is contracts where price is moving fast and there is meaningful time left for the move to mean something. That set includes high-IY contracts, but it also includes low-IY contracts where there is genuine activity (a 70-cent contract that's moved to 75 cents in a week with 100 days remaining) but where the math doesn't pay even if you're right. You burn attention on those, and the IY filter at the second stage culls them anyway. Your time was wasted.

Now suppose you ran IY first, then CRI. The top of the IY scan is contracts where the math pays *if* the contract is real. Most of those are not real — they are stale prices on long-tail markets where nobody is paying attention. The CRI filter at the second stage culls those because there is no recent price movement. What survives both filters is "the math pays, *and* the market is moving." That is a much tighter set than either filter alone, and it is the set that justifies the next stage of judgment.

The reason IY-first is the right order is that IY is the *necessary condition* and CRI is the *sufficient condition*. You need both. If you only had to pick one, IY tells you whether to bother, and CRI tells you whether the bother is wasted. Running IY first gives you the smaller candidate set faster, because the IY math is computationally cheap and the IY filter is selective — most markets in the universe have IY below any reasonable threshold.

EE comes third because it is conditional on the contract being part of a multi-outcome event. Single binary contracts (the majority of the Kalshi and Polymarket universes) have no sibling outcomes and therefore no EE to compute. Skipping the EE step on a non-event contract is fine; trying to run it everywhere wastes compute on contracts where the metric doesn't apply.

## The Full Funnel Walked Through

I want to walk through a concrete pass: 47K markets → 1 trade. The numbers are illustrative, not from a specific morning, but the orders of magnitude are correct from production runs of the stack.

**Stage 0 — The universe.** Approximately 47,000 active markets across Kalshi and Polymarket as of early 2026. This number includes everything from heavily-traded political markets to long-tail weather contracts to obscure pop-culture events. About 70% of the universe is on Polymarket, 30% on Kalshi.

**Stage 1 — IY filter.** Run `sf scan --by-iy desc --min-tau 1 --min-iy 1.0`. The min-tau flag excludes contracts in their final day (which would otherwise dominate the IY scan with garbage exponentials). The min-iy flag sets the threshold at 100% annualized — high enough to be interesting compared to a T-bill, low enough that it doesn't filter out the entire universe. This typically produces around 200 candidates. The variance is wide — on a heavy news day with lots of short-term political contracts, it can be 400; on a quiet weekend, 60.

**Stage 2 — CRI filter.** Run `sf scan --by-cri desc --min-tau 1 --min-iy 1.0 --min-cri 1.0`. The min-cri flag sets the activity threshold. Of the 200 IY candidates, typically about 30 pass the CRI filter — meaning the price is actually moving over the recent window. The other 170 are stale: high IY because nobody is paying attention, low CRI because nobody is paying attention.

This is the big compression in the funnel. The 170 you cut here are the markets where the IY scan was a false positive — interesting on paper, dead in practice. The CRI filter is what makes the IY scan trustable. Without it, you would spend your attention on stale markets and wonder why the trades never materialize.

**Stage 3 — EE check on multi-outcome events.** Of the 30 surviving candidates, maybe 12 are part of multi-outcome events (Democratic nomination contracts, sports championship contracts, hurricane category contracts, etc.). Run the EE check on each. The cases that matter:

- EE near zero on the candidate's event: the field is pricing cleanly, and any edge on this contract has to come from a thesis update on the specific sibling.
- EE > 0.04 on the candidate's event with the candidate's price moving aggressively: the field is overconfident, the candidate may be where the slack is, and the trade is "buy YES on the candidate, ride the EE compression."
- EE < −0.04 on the candidate's event: the field is under-pricing somebody, but check liquidity carefully because the under-price is usually a thin orderbook on one outcome.

After the EE check, the 30 candidates compress to about 5. Some of those are dropped because the EE confirms what the IY/CRI already said (a high-IY candidate in an EE-rich event is double-counting the same signal). Others survive because the EE adds new information.

The 18 candidates that did *not* belong to multi-outcome events skip the EE check entirely and move forward. So the actual surviving set after stage 3 is 5 (multi-outcome) + 18 (single binary) = 23. I usually do a quick eyeball on the 18 to drop the ones where the IY-CRI signal is suspicious for any other reason (tickers I don't recognize, venues I don't have positions on, events that look like venue-specific quirks). That brings the total to roughly 5.

**Stage 4 — Judgment.** Now I have 5 contracts. This is the stage where I open the orderbook in the SimpleFunctions UI, read the recent news flow, check cross-venue prices if both venues list the same outcome, look at the historical price chart for the last week, and decide whether to take a position.

For each of the 5, the questions are:
- Does the orderbook actually let me execute? If the bid-ask spread is 8 cents and I need to fill a 200-share position, the displayed mid is fiction.
- Does the recent news support the direction the indicators imply? If CRI is high but every news item points the wrong way, the indicator is reflecting a bad thesis getting priced in, not a good thesis to join.
- Is there a cross-venue price difference I can exploit? If Kalshi and Polymarket show the same outcome at 8 cents apart, I might want both legs.
- Do I have a hard reason for the conviction? "The indicators say so" is not a reason. "The Fed minutes drop tomorrow and the contract is 10 cents below where my thesis says it should be after the minutes" is a reason.

Of the 5, usually 1 — sometimes 0, occasionally 2 — survives this stage and becomes a trade. The other 4 are markets I am paying attention to but not yet in. The one survivor is what I size, what I put on, and what I track as an open position.

## The Numbers

47,000 → 200 → 30 → 5 → 1.

Two orders of magnitude at stage 1 (IY filter). Almost an order of magnitude at stage 2 (CRI filter). About a 6× compression at stage 3 (EE + eyeball). About a 5× compression at stage 4 (judgment).

The total compression ratio is about 47,000:1, which means roughly one trade per scan cycle. On a busy day I might run two or three scans in different categories (rates, politics, sports) and end the day with 2-3 new positions. On a quiet day I run one scan and add nothing. The funnel produces zero trades on a lot of days, and that is the right answer — there is no obligation to trade just because you ran the scanner.

The numbers are illustrative, not exact. On a heavy news week the IY scan can produce 400 candidates and the funnel might produce 3-4 trades a day. On a holiday week the IY scan produces 60 candidates and the funnel produces 0. The point of giving you the numbers is to set the expectation that *most contracts get filtered*, and the survivors are a tiny fraction of the universe.

## What This Doesn't Mean

Three things expected edge is not.

It is not a *single number*. Some readers want a "score" they can sort by — "expected edge = 0.42" or whatever. The composition does not produce a single number, and pretending it does loses information. What it produces is a *sequence of pass-fail filters* with a small surviving set and a judgment step at the end. The surviving set is the output. There is no continuous ranking of the survivors; they are all "candidates worth a look," and the judgment stage chooses among them.

It is not an *automated trading signal*. The judgment stage is not optional. I have shipped versions of the funnel that ran end-to-end without the judgment step, and the results were terrible — most of the false positives in the indicator stack survived all three computational filters and showed up as "trades" that the judgment stage would have rejected. The funnel is a *attention compression tool*, not an alpha generator. It tells you where to look. It does not tell you what to do.

It is not a *backtest framework*. The composition rule is forward-looking: it filters the current universe to a small set of current candidates. Running it historically requires reconstructing the state of the orderbook, the news flow, and the cross-venue prices at every point in time, which is expensive and prone to look-ahead bias. The right way to validate the funnel is to run it forward in production for several months and track the realized PnL, not to run it backward against historical data.

## Where the Composition Breaks

Three failure modes worth knowing about.

**Failure 1 — All four signals align by accident.** Sometimes the funnel produces a small set of "high conviction" candidates where every indicator is screaming in the same direction, and every one of them is wrong. This happens when there is a single underlying systematic factor (a Fed-meeting morning, an election-night moment, a major news shock) that biases all the indicators simultaneously. The indicators are not independent during these moments. The fix is to stop trading the funnel during *acute* events and resume it 24-48 hours later when the systematic factor has dissipated.

**Failure 2 — Liquidity collapse on the surviving set.** The funnel filters by IY, CRI, and EE, none of which directly account for whether the surviving contract is *executable*. A high-IY, high-CRI, EE-favorable contract with 12 cents of total orderbook depth is on paper an opportunity, in practice an unfillable position. The fix is to add a [liquidity availability score](/concepts/null-as-signal) check at the judgment stage — verify the orderbook can support the size you want before committing.

**Failure 3 — The surviving set is just one event.** Sometimes the funnel produces 5 surviving candidates and they are all sibling outcomes of the same multi-outcome event. The "diversification" is fake — they are all driven by the same thesis. The fix is to require *event diversity* in the surviving set: no more than two candidates per event, and no more than three candidates per venue. This is enforced at the judgment stage, not in the computational filters.

## How This Connects to the Rest of the Stack

Expected edge is the integration of the indicators. The indicators themselves are described in [implied yield](/concepts/implied-yield), [cliff risk index](/concepts/cliff-risk-index), and [event overround](/concepts/event-overround). The unit underneath all of them is described in [τ-days](/concepts/tau-days). The structural framework that puts them together is [the valuation funnel](/concepts/the-valuation-funnel) and [the prediction market indicator stack](/concepts/pm-indicator-stack). The "null is signal" recursion that adds Tier B indicators is in [null as signal](/concepts/null-as-signal).

Once the composition becomes habit, the workflow each morning is fast: run `sf scan --by-iy desc --min-tau 1 --min-iy 1.0 --min-cri 1.0`, scroll the output, eyeball the multi-outcome candidates for EE, open the survivors in the SimpleFunctions UI, decide. Total time: 15-20 minutes. Output: 0-3 new positions or watchlist entries. The compression ratio is what makes the workflow feasible. Without it, you would be looking at 47K rows and not trading anything.

## A CTA

The full live-data version of expected edge runs on the [/screen](/screen) page. Sort by IY, filter by CRI > 1, look at the multi-outcome events for EE, and you have the same surviving set the morning workflow produces. The page is the funnel made visible. The trades are still up to you.