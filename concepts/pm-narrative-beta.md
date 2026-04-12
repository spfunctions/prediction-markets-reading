# The Prediction Market "Narrative Beta": When News Cycles Move Multiple Markets at Once

> Some prediction markets are narratively correlated even when their underlying outcomes are independent. Every "AI" market moves together when OpenAI is in the news, even when the contracts have nothing in common mathematically. That correlation is real, fragile, and tradeable for short windows.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Equity traders have a vocabulary for sector beta — the idea that a stock can move because its sector is moving, even if the company-specific fundamentals are unchanged. A semiconductor stock catches a bid when NVIDIA reports good earnings, not because the company being bid on has any direct exposure to NVIDIA, but because the *narrative* about semiconductors got better. The capital that flows into the sector flows into all of its constituents indiscriminately, and only later sorts itself out.

Prediction markets have an analogous phenomenon, and almost nobody who is not explicitly looking for it has noticed it yet. I call it *narrative beta*, and it shows up most cleanly on event clusters that share a topic but no underlying mathematical dependence. Every "AI" market moves together when there is an OpenAI news cycle. Every "Russia" market moves together when there is a Ukraine headline. Every "election" market moves together when a major poll drops, even when the elections are in different countries with different candidates.

This essay is about why narrative beta exists, how to measure it, why it is fragile in a specific way, and how to trade it inside its short useful window.

## The Phenomenon

Pick any concept that has multiple prediction markets in its orbit. "AI" is a good one because it has at least a dozen Polymarket contracts with the prefix or theme: "OpenAI IPO 2027", "AGI by 2030", "GPT-5 released by Q2 2026", "Anthropic valuation > $100B", "AI causes recession by 2028", and so on. These contracts have nothing in common mathematically. The OpenAI IPO does not cause AGI. AGI does not cause GPT-5. GPT-5 does not cause an Anthropic valuation. Each one has a separate causal tree, separate set of catalysts, separate resolution criteria.

Now wait for any major OpenAI news to drop — a product announcement, a leadership change, a fundraising round, a safety incident — and watch what happens to all twelve contracts in the next hour. They all move. Not by the same amount, and not always in the same direction, but they all move. The correlation in the price changes during the news window is meaningfully higher than the correlation outside the window. The contracts are tied together by narrative even though they are independent by mechanism.

I have measured this on rolling 30-day windows for several event clusters and the pattern is consistent: the cross-market correlation inside news windows averages around 0.4 to 0.6, while the baseline correlation outside news windows averages around 0.05 to 0.15. That gap is the narrative beta. It is real, it is large enough to trade, and it is the kind of structural feature that an indicator-only screen will not surface because it is a *cross-market* phenomenon, not a per-market one.

## Why Narrative Beta Exists

Two reasons, both behavioral.

The first reason is *attention compounding*. When a news cycle hits a topic, every trader who follows that topic refreshes their screen for every contract in the cluster. The attention is shared across the contracts, so any flow that the news generates gets distributed across all of them. A trader who reads "OpenAI launches GPT-5" might immediately update their position on five different contracts, not because the news bears on all five but because they were all on the same screen and the trader processed them as a batch. The flow is correlated because the attention is correlated.

The second reason is *thesis bundling*. A lot of traders hold theses that are framed at the level of the topic, not the level of the individual contract. A trader who is "long AI" expresses that thesis by buying YES on multiple AI contracts. When new information arrives that updates the topic-level thesis, the trader rebalances all the topic-level positions in the same direction. From the outside, this looks like the contracts are correlated. From the trader's perspective, they are expressing one thesis with multiple instruments, the same way an equity investor might be "long semiconductors" by holding NVIDIA + AMD + INTC.

Both mechanisms are *real* but neither produces a stable correlation. Neither one creates a structural reason for the contracts to move together over the long run; both create a temporary co-movement during news windows that decays back to baseline within a few hours after the news fades.

## Measuring It

The measurement is straightforward in principle and finicky in practice. You need:

1. A grouping function that maps each contract to its narrative cluster (AI, election, Russia, recession, climate, etc.). This is the part where humans do better than rules — the grouping is semantic, not mechanical.

2. A rolling correlation matrix across the contracts in each cluster, computed on hourly returns over a trailing window of (typically) 7 days.

3. A news-window detector. Either you supply this manually (the OpenAI news dropped at 14:00 ET on April 3) or you derive it from a spike in cross-cluster mentions in opinion data.

The narrative beta is the difference between the correlation inside news windows and the correlation outside them, averaged across the cluster. A cluster with high narrative beta has a big gap. A cluster with low narrative beta has a small gap.

The thing to watch out for in the measurement is the *baseline*. Some clusters look correlated all the time because they share an underlying mechanical driver. "Recession 2026" and "S&P 500 down 10% by year-end" are not just narratively correlated — they are causally correlated, because both depend on the same macro conditions. The narrative beta on those is artificially high because you cannot subtract the causal piece. The cleanest measurements are on clusters where the contracts are *demonstrably* causally independent, like the AI cluster I described above. There, the baseline is genuinely near zero and the correlation spikes during news windows are unambiguous.

## A Worked Example

Here is one I caught about three weeks ago. The narrative cluster was "Tesla / Musk", which on Polymarket has a handful of contracts with the prefix or theme: "Tesla deliveries Q2 > 500K", "Musk sells X (Twitter) in 2026", "Tesla market cap > $1T by year-end", "Cybertruck production > 100K in 2026". These contracts share Musk-as-CEO as a common thread but the mechanical drivers are independent — Q2 deliveries are about manufacturing, X sale is about Musk's personal capital, market cap is about overall investor sentiment, Cybertruck production is about a specific factory.

A Tesla earnings report dropped on a Thursday morning. The earnings beat was material — revenue 8% above consensus, deliveries 12% above consensus. Within the next two hours, *all four contracts* moved. The deliveries contract moved from 0.42 to 0.58 (intuitive — direct exposure). The market cap contract moved from 0.33 to 0.47 (also intuitive). The X-sale contract moved from 0.18 to 0.21 (less intuitive — a strong Tesla report should not change the probability of Musk selling X). The Cybertruck contract moved from 0.39 to 0.46 (mildly intuitive — Tesla strength bleeds into Cybertruck expectations).

The X-sale move is the giveaway for narrative beta. There is no mechanical reason a Tesla earnings beat should change the probability of Musk selling X. The contract moved because the *narrative around Musk* got more bullish, and traders updated all their Musk-themed positions in the same direction. That 3-cent move on the X-sale contract was pure narrative beta — no underlying causal change, just attention-driven flow.

Two days later, the market re-sorted. The deliveries contract held its move (the news was real and durable). The market-cap contract gave up half the move (the macro winds had shifted). The X-sale contract drifted back to 0.19 — the narrative-driven move *fully decayed* once the attention faded. That is the typical pattern for narrative beta moves: the ones that are mechanically grounded persist, the ones that are pure narrative bleed back to baseline within 48 to 72 hours.

## Trading Around Narrative Beta

Three concrete plays.

**Play one — fade the narrative-driven moves on contracts with no mechanical exposure.** When the X-sale contract moved from 0.18 to 0.21 because of Tesla earnings, the right play was to short the move. The contract had no mechanical reason to move; the move was pure attention spillover; the move would decay. I sized small (this is a low-conviction edge by construction) and made about 2 cents per contract over 48 hours. Nothing huge, but the win rate on this kind of trade is high enough that it compounds with volume.

**Play two — anticipate narrative beta on upcoming catalysts.** If you know an OpenAI announcement is coming on Friday, you can pre-position on the AI cluster contracts that are *most* prone to narrative-driven moves (the ones with the longest tau and the weakest mechanical link to the announcement). When the announcement drops and the cluster moves together, the contracts you pre-positioned on will move with the cluster, and you can exit into the narrative-driven flow before it decays.

**Play three — use cluster correlation as a sanity check on single-contract theses.** If you are about to put on a single-contract trade and the entire narrative cluster around that contract has been quiet, you have less narrative-tailwind risk than if the cluster has been trending. If the cluster has been moving against you for a week, your single-contract thesis is fighting the narrative beta and you should size smaller. The cluster behavior is a regime indicator for the contract.

## Why Narrative Beta Is Fragile

The fragility is the most important property of the phenomenon and the easiest one to underestimate.

Narrative beta breaks the moment a *contract-specific* shock hits one of the cluster constituents. The example: the AI cluster moves together until OpenAI specifically gets sued, at which point the OpenAI-specific contracts crash and the rest of the cluster moves *opposite* (because the bad news for OpenAI is good news for Anthropic, etc.). The correlation flips sign within minutes. Any position that was sized assuming continued narrative correlation gets whipsawed.

The fragility comes from the same source as the beta itself. Narrative beta exists because traders treat the cluster as a single thing for attention purposes. The moment the cluster becomes *internally differentiated* by news, the attention re-fragments, and the within-cluster correlation collapses or flips. A trader who fades narrative beta and forgets that the correlation is conditional will lose money on the day a contract-specific shock rotates the cluster.

The defensive heuristic: never hold a narrative-beta trade across a known contract-specific catalyst for any constituent of the cluster. If OpenAI has an earnings call on Tuesday, close your AI-cluster narrative trades on Monday. The news that creates the contract-specific shock is also the news that breaks the correlation you were trading.

## Where the Concept Breaks

Three honest caveats.

**Cluster definition is subjective.** There is no canonical "AI cluster" — different traders will group contracts differently, and the narrative beta you measure depends on the grouping. The cluster is in the eye of the trader, and the trader is wrong some of the time. Always re-test the cluster definition before sizing any narrative-beta trade.

**Low-volume markets do not show the pattern cleanly.** Narrative beta requires *flow* to express itself. A cluster of low-volume contracts where nobody is trading will not show the within-window correlation, because there are no trades happening to generate the correlation. The pattern is most readable on clusters where every constituent has at least Tier B liquidity coverage.

**The decay window is variable.** I said "48 to 72 hours" above, but I have seen narrative beta moves that decayed in 6 hours and others that persisted for two weeks. The decay rate depends on whether the originating news got refreshed by follow-on news. A trade that assumes a specific decay window and gets the timing wrong loses on theta even when the directional thesis was right.

## How This Connects to the Rest of the Stack

Narrative beta is the *cross-market* analog of [reflexivity-loops](/concepts/reflexivity-loops): both are phenomena where market behavior is driven by something other than the underlying outcome's probability. Reflexivity is single-market and self-referential; narrative beta is cross-market and attention-driven. They often co-exist on the same contract — a heavily-trafficked election contract is both reflexive (its price feeds back into news) and a narrative-beta carrier (it moves with the broader election cluster).

The narrative beta concept also interacts with [endogenous-vs-reality-vs-opinion-data](/concepts/endogenous-vs-reality-vs-opinion-data). Narrative beta is fundamentally an opinion-data phenomenon — the correlation comes from how traders process news, which is shaped by the opinion ecosystem around the topic. Markets with rich opinion data have stronger narrative beta. Markets with sparse opinion data (niche economic indicators, esoteric weather contracts) have essentially none.

For the implementation of cluster correlation, see [the-cyc-regex-grouper-walkthrough](/technicals/the-cyc-regex-grouper-walkthrough), which is the tooling for grouping contracts into clusters mechanically (and which underestimates the narrative dimension — narrative clusters are looser than CYC clusters and have to be defined semantically).

The right way to use narrative beta in practice is as a regime overlay on your existing screening. Run the indicator stack as normal, and then for each candidate trade, ask: "is this contract part of a cluster that is currently in a news window?" If yes, your edge is potentially boosted (or eroded) by the narrative beta and you should adjust the sizing accordingly. If no, the trade is on its own merits and the cluster behavior does not enter the calculation. `sf scan --cluster-aware` is the flag I use to surface this information at scan time; the cluster annotations come from the same CYC grouper that drives the term-structure analysis.

The big lesson is that prediction markets are not always priced as independent contracts. Sometimes they are priced as *baskets* — implicit, attention-driven, fragile baskets — and the trader who can read the basket structure has an edge over the trader who reads each contract in isolation.