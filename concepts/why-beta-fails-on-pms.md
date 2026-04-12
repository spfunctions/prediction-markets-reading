# Why Beta-to-S&P Doesn't Make Sense for Prediction Markets

> Beta requires a continuous return distribution and a meaningful market portfolio. Prediction markets have neither. The few correlations that exist thrash around faster than any rolling window can stabilize.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 11 min

---
Every once in a while a former equity quant pings me and asks for my "PM beta book." They want a list of prediction-market contracts ranked by their beta to the S&P 500, the way you would build a long-short equity book ranked by beta to a sector benchmark. The request makes complete sense in the world the asker came from. It also has no clean answer in the world the asker is now in.

This page walks through why beta is structurally the wrong measurement on prediction-market contracts, what an honest empirical attempt looks like, and what to do instead when you actually want to decompose the risk of a PM position into market-level and idiosyncratic components.

## What Beta Is, in Two Sentences

In equity finance, beta is the slope of the regression of an asset's returns on the market's returns over some window. A stock with beta of 1.2 moves, on average, 1.2 times as much as the market — same direction, larger amplitude — and the residual after the market move is "alpha" plus noise.

The framework rests on three assumptions: returns are continuous and well-defined, there is a single meaningful "market portfolio" to regress against, and the relationship between asset and market is stable enough over the regression window that the slope is informative going forward. All three assumptions hold reasonably well for liquid US equities. None of them hold cleanly for prediction-market contracts.

## Assumption 1: Continuous Returns

Equity returns are computed as `(p_t / p_{t-1}) − 1` and they are well-defined as long as the price is not zero. A US stock at $100 yesterday and $102 today produced a 2% return, and the next day's number sits on the same number line as the previous one.

Binary prediction-market contracts have no comparable concept. The "return" on a contract is the change in the implied probability, but that change is bounded — a contract can only ever go from 0.42 to 0.50, not from $42 to $50, and the move is structurally constrained to the [0, 1] interval. The structure is more like a saturating function than a continuous price process.

You could try to define a return as `p_t − p_{t-1}`, treating each move in the implied probability as a "return." Some people do this. The problem is that the resulting numbers are not commensurable across contracts because they do not account for the time-to-resolution. A 4-cent move on a 200-day contract and a 4-cent move on a 4-day contract are very different events; the second one is approaching the cliff and the first one is structural reassessment. The same numerical "return" carries totally different information depending on τ.

You could try to fix this by switching to log returns or to implied-yield deltas instead of price deltas, but at that point you are no longer computing beta in the equity sense — you are computing beta in some bespoke unit that does not have a comparable equity benchmark, which defeats the purpose of borrowing the term in the first place.

## Assumption 2: A Meaningful Market Portfolio

Equity beta works because there *is* a meaningful market portfolio — the S&P 500 or the Russell 3000 or the MSCI World, depending on the strategy. The portfolio represents "the market," it has thousands of constituents, it is liquid enough that the daily return is essentially a clean signal, and the market portfolio is conceptually the *aggregate* of the asset class you are trading. Beta is meaningful because the regression has both sides on the same conceptual surface.

Prediction markets have nothing analogous. There is no "PM market portfolio." If you tried to construct one, you would have to decide whether to weight by 24-hour volume (which would be dominated by a handful of high-volume political contracts) or by total capital at risk (which is even more concentrated) or by number of contracts equally (which gives an average that is dominated by the long tail of essentially-untraded markets). Each of these gives a different "portfolio," and none of them has any economic meaning the way the S&P 500 does.

Some people regress PM contracts against the S&P 500 directly, on the theory that "the PM contract pays in dollars and dollars correlate with the equity market." This is a category error. The dollars at risk in a PM are dollars locked into a specific binary outcome that has no mechanical relationship with the S&P. A Powell-stays-Fed-Chair contract and the S&P move on the same news flow occasionally — Powell announcements are also macro news — but the relationship is *coincidence* through the news, not a structural exposure to the same factor.

The closest you can get to a "PM market portfolio" is *the indicator stack itself* — see [pm-indicator-stack](/concepts/pm-indicator-stack). The reason that framing works is that the indicators are commensurable across contracts in a way that prices are not. You can ask "which contracts have IY in the top 10% of the universe today" and that is a meaningful cross-section. Beta-to-IY-percentile, if you wanted to invent it, would be a real number. Beta-to-S&P is not.

## Assumption 3: Stability Over the Regression Window

Equity beta only works if the underlying relationship is stable enough that the past regression slope predicts the future slope. For most US large-caps, this is roughly true over windows of a few months to a few years — beta drifts but slowly, and you can refresh it quarterly.

Prediction-market contracts have a fundamental problem here: their *news regime* changes faster than any rolling window can stabilize. A Fed-decision contract is exposed to one set of news for the first ten days of its life (general economic data), a different set for the next ten days (FOMC commentary), and a third set in the final 48 hours (the actual statement and dot plot). The "factor exposures" of the contract are not stable across those phases. A 30-day rolling beta against any factor would average across three different regimes and produce a number that does not describe any of them.

I have actually run this experiment. Take five Kalshi contracts that you would expect to have the most equity-like exposure — election outcomes, recession probability contracts, Fed-decision contracts — and run a 60-day rolling beta against SPY. The output is not a stable beta with a slowly-drifting trend. It is a beta that thrashes around between, say, +0.4 and −0.6 over the course of a single quarter, without any systematic pattern, because each rolling window catches a different news regime.

When I show that rolling beta to the equity quant who asked for the PM beta book, they usually say "the noise is just because the windows are too short." So I show them the 180-day version. Same problem, slightly slower thrashing. The issue is not window length; the issue is that PM contracts do not *have* stable factor exposures because the underlying events do not have stable factor structures. Each event is its own thing, and the news flow that moves it is largely orthogonal to the news flow that moves the S&P.

## The One Place It Almost Works

There is one corner where a beta-style regression does produce something halfway meaningful: contracts on broad macro outcomes (US recession, US default, S&P-level price targets) over windows where the market regime is stable. A "US recession in the next 12 months" contract really does have a roughly negative correlation with SPY, and over a quarter of stable monetary policy you can fit a regression that is not pure noise.

The catch is that this works only when the regime is stable, and the regime is not stable most of the time. The window in which you can confidently say "this PM contract has beta x to SPY" is narrower than the window in which you would actually want to use the number. By the time you have collected enough data to compute the beta with any confidence, the underlying news regime has moved on and the beta is stale.

The other catch is that even in this corner, the beta you compute is not very useful for *decision-making*. Knowing that the recession contract has beta of −0.3 to SPY does not tell you how to trade the contract — it tells you that if SPY drops 1%, the contract will tend to move up 0.3% in implied probability terms. That is a hedge ratio, not an alpha signal, and the hedge ratio is only stable over short horizons. You are doing a lot of regression work to compute a number you can only use for the next two weeks.

## What to Use Instead: Event-Specific Risk Decomposition

The framing that does work on PMs is *event-specific risk decomposition* — basically, building a causal tree for each contract and asking "what are the discrete things that have to go right for this to resolve YES, and what is the conditional probability of each?"

This is the [causal tree](/learn/causal-tree) approach, and it is documented in detail in [how-causal-tree-decomposition-works-prediction-market-trading](/technicals/how-causal-tree-decomposition-works-prediction-market-trading). The short version: for each contract, you write down the chain of events that have to happen in order, you assign rough probabilities to each link, and you compute the joint probability. The factors in the decomposition are *event-specific* — they are not general macro factors, they are things like "does the FOMC vote 9-3 or 8-4 on cuts," "does the BLS revision exceed two standard deviations," "does the candidate concede before midnight ET."

This decomposition gives you something beta cannot give you: a *story* about why the contract should be where it is. Beta tells you that the contract co-moves with some external factor by some amount; the causal tree tells you what would have to change in the world for the contract to be wrong about its current price. The first is a statistical observation; the second is an actionable thesis.

When you actually want to talk about "what is my market exposure across my PM book," the right answer is not a portfolio beta. It is a *category exposure* — how many of your positions are in election contracts, how many in Fed contracts, how many in geopolitical, how many in sports — combined with a check that you do not have multiple positions whose causal trees share critical nodes. If three of your contracts all depend on "Fed cuts in May," you are not diversified; you are triple-long the same factor under different names. Beta does not catch that. A causal-tree audit does.

## A Quick Worked Example

Take a hypothetical book of five Kalshi positions: a Fed-cut-in-May contract at 0.45, a recession-by-year-end at 0.32, a Powell-stays-Fed-Chair at 0.78, a Senate-control-2026 at 0.55, and an SPX-above-5500-by-July at 0.40. The naive thing is to compute a 90-day rolling beta of each against SPY and call the result the book's market exposure.

When I run this exercise on the actual Kalshi history, the betas come out at roughly +0.6, −0.3, +0.1, +0.05, and +0.7 respectively. The "market exposure" of the book is some weighted average of those five, which depends on position sizes. Those five numbers also happen to be from the past 90 days; the same numbers from the previous 90 days are very different (think +0.2, −0.1, −0.1, +0.0, and +0.5), and the numbers from the 90 days before that are different again. The beta picture does not stabilize.

Now do the causal-tree audit instead. Two of the five positions (Fed-cut and SPX-above-5500) share a critical node: "Fed signals dovish at next meeting." If the Fed disappoints, both move down hard. The recession contract has a *negative* exposure to that same node. The Powell contract is mostly about a discrete personnel decision and is largely orthogonal. The Senate contract is its own world. The audit tells you the book has a concentrated bet on Fed dovishness with one offsetting hedge — a fact that is *immediately useful* for sizing and that no amount of beta computation would have surfaced cleanly.

The causal-tree view is the right replacement for beta on prediction markets. Beta forces you to express risk as "co-movement with a single benchmark," which assumes the wrong shape for the underlying. The causal tree lets you express risk as "shared dependence on a specific node," which is the actual structure of how PM positions correlate when they correlate at all.

## Where This Negation Is Too Strong

Two honest caveats.

First, *implied correlation across cross-listed contracts* is real and can be regressed cleanly. The same outcome on Kalshi and Polymarket should have correlation near 1 in changes, and the residual is a tradeable spread. That is genuinely a beta-style relationship, and the cross-venue arbitrage stack uses it constantly. See [from-adr-arbitrage-to-cross-venue-pms](/concepts/from-adr-arbitrage-to-cross-venue-pms) for the depositary-receipt analog.

Second, *category-level co-movement* exists on short horizons. When OpenAI announces something, every "AI" contract on Polymarket moves together for a few hours. That is a real correlation, and it is similar in spirit to sector beta in equities. The difference is that the correlation is fragile — see [pm-narrative-beta](/concepts/pm-narrative-beta) for the detailed treatment — and using it requires constant rebuilding of the category groupings, which makes it more of a *narrative beta* than a stable factor exposure.

Outside those two cases, beta-to-S&P or beta-to-anything is the wrong measurement on PMs. The right measurement is event-specific risk decomposition through causal trees, and that is the framework you should be using.