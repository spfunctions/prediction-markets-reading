# Information Latency: How Fast Do Prediction Markets React to News?

> A new piece of information drops at time T. The price reflects it at T + Δ. Δ is your edge window. Three real episodes — Fed decision, geopolitical shock, earnings beat — and what they say about how fast different markets actually react.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 9 min

---
Every trade I have ever made on prediction markets has been a bet that I can act on a piece of information faster than the market can absorb it. That sentence is the entire game. The variable that determines whether the bet works is *information latency*: the time between when the information becomes available and when the price reflects it. If the latency is six seconds, you have no edge as a human; if the latency is six hours, you have a lot of edge.

The latency varies by market, by news type, and by the existing positioning of participants. There is no single number. But there are *typical* numbers for each category, and knowing the typical numbers tells you which markets are worth chasing news on and which ones are already efficient enough that you should not bother. What follows is the framework, three real episodes that anchor the typical numbers, and the failure modes of treating "react fast" as the universal goal.

## The Three Drivers of Latency

Information latency on a prediction market is determined by three variables. None of them is the news itself; all of them are properties of the market.

**Driver 1 — How many sharp traders are watching.** A market with 50 sophisticated participants running scans every minute will absorb a news event in seconds. A market with 3 sophisticated participants checking it occasionally will take hours. The "watcher density" is the most important variable, and it correlates roughly with daily volume, but not perfectly — some high-volume markets have lots of retail volume and few sharp participants, which produces a slow latency despite the volume.

**Driver 2 — How unambiguous the information is.** A Fed decision announcement is unambiguous: the FOMC says "we are cutting rates by 25 basis points" or it does not. A geopolitical event ("there has been an explosion in city X") is more ambiguous: was it a missile, an accident, an earlier event being misreported? The more ambiguous the news, the longer the latency, because traders have to interpret before they can price.

**Driver 3 — What the existing positioning is.** A market that was already heavily one-sided before the news has more "positioning inertia" than a market that was balanced. If a Fed-cut contract is at 0.42 with a balanced book, and the news says "cut," the price moves to 0.85 in seconds. If the same contract is at 0.42 with a heavily YES-biased order book (because everyone *expected* a cut and was already long), the existing longs have to take profits and the price might overshoot to 0.92 before settling at 0.85, which extends the apparent latency.

These three variables compose. A high-watcher-density market with unambiguous news and a balanced book will absorb the news in seconds. A low-watcher-density market with ambiguous news and one-sided positioning can take days to settle into the new price.

## Three Real Episodes

I have a few latency-instrumented episodes that I keep returning to, because they cover the typical spectrum.

**Episode 1 — Fed decision, March 2024.** The FOMC released its statement at 2:00:00 PM ET. The Kalshi "Fed cut March 25bps" contract moved from 0.18 to 0.04 between 2:00:01 and 2:00:14 PM ET. Total latency: 14 seconds. The contract had high watcher density (hundreds of sharp participants), unambiguous news (the statement is binary on the cut question), and roughly balanced positioning. This is the *floor* of latency on Kalshi for this category: 14 seconds is roughly what it takes for the fastest scrapers to read the statement, parse it, and place orders. As a human you cannot beat this. The latency window is too small to enter manually.

**Episode 2 — Geopolitical event, late 2023.** A shock geopolitical headline broke in the late afternoon ET. Several Polymarket contracts on related "Will event X happen by date Y" outcomes started moving within the first ten minutes. Different contracts moved at different speeds: the most-watched ones (Israel-Iran, Russia-Ukraine adjacencies) absorbed the news within 10-15 minutes. Less-watched ones (regional adjacencies, second-order outcomes) took 1-3 hours to fully reprice. The variation across contracts was driven entirely by watcher density — all the contracts had access to the same news, but the ones with active scanners moved first. Latency range: 10 minutes to 3 hours, depending on the contract.

**Episode 3 — Corporate earnings beat.** A major tech company reported earnings after market close. Kalshi had a contract on "Will Q earnings exceed consensus." The earnings release dropped at 4:05 PM ET. The contract — which had been trading at 0.55 — moved to 0.78 over the course of the next 45 minutes, not 45 seconds. Why so slow? Because the contract had low watcher density at 4:05 PM ET (it was a niche corporate market that did not have constant attention), and the existing positioning was thin enough that the moves had to come from new participants entering rather than existing participants flipping. The 45-minute latency is the *ceiling* for a relatively high-attention contract — niche corporate news on Kalshi takes tens of minutes, not seconds.

The takeaway from the three episodes is that latency varies by *two orders of magnitude* depending on the market — from 14 seconds to 3 hours — and the variation is predictable from the three drivers. Knowing which type of market you are looking at is more important than how fast your news feed is.

## Latency = Your Edge Window

Once you know the typical latency for a market, you know your edge window. Your edge window is the gap between when information becomes available and when the market has absorbed it. Inside that window you can act with confidence; outside it you are chasing.

The implication is that a human trader has zero edge on Episode 1-style markets. The 14-second latency on a Kalshi Fed contract is too tight for any human to execute a thesis-driven trade against. By the time you read the FOMC statement, parse it, decide on a direction, and place an order, the price has already moved. The only way to compete in that window is to be a machine, and the machines are already there. So I do not trade Fed decisions on the second of the announcement — I trade the *anticipation* in the days leading up, when the latency window is wide because the news has not happened yet.

A human trader has a meaningful edge on Episode 2-style markets. A 10-30 minute latency on a geopolitical event is well within the time it takes a human to read coverage, build a quick thesis, and place orders. The trick is being *at the screen* when the news breaks. Edge on these contracts is more about availability than about analytical insight — the analysis is rarely the bottleneck, the execution speed is.

A human trader has a *huge* edge on Episode 3-style markets. A 45-minute latency on a corporate earnings contract is enough time to read the press release, run a thesis check, and build a position with care. The contracts with multi-hour latency are the ones where careful human reasoning beats automated systems, because the automated systems do not bother running on niche markets. This is the latency profile of most of my "high-edge" trades.

The general rule: the markets with the longest latency are the ones with the most edge for thesis-aware humans, but they are also the ones where most traders are not paying attention. That is the trade — pay attention to markets where the watcher density is low, because that is where the latency window is wide enough to walk through.

## Where Latency Reading Breaks

**The "first reaction" is not always the right reaction.** A market that absorbs news in 14 seconds is moving on the literal text of the announcement, but the actual implication of the news might require 30 minutes of analysis. The 14-second move can be wrong, and a slower trader who reads the full context can still capture edge by *fading* the first reaction. This is harder than it sounds — the fade has to work in the next several hours, not the next several days, because once analysts catch up, the price will follow them. But the "first reaction is always right" assumption is wrong, and fading the first move is a real strategy on news where the headline is misleading.

**Latency is not constant within a single market.** A contract that has 3-minute latency on most news will have 30-second latency on news that the market was already expecting. The variance within a single contract is large, because the *type* of news matters more than the contract identity. If you want to model latency precisely you have to model it conditional on news type, not just contract identity.

**Speed is not always edge.** A market with very low latency is a market where many people are competing for the same news. Even if you have fast access, you are not the only one. The actual edge is "fast access *and* a thesis nobody else has," and the second condition is much rarer than the first. Buying expensive market data feeds to get a 100ms speed advantage is rarely worth it on prediction markets, because the bottleneck is rarely your raw speed — it is the size of the universe of traders who are also fast.

A fourth caveat: some news is self-fulfilling in a way that makes latency irrelevant. If the news is "Polymarket implied probability moved to X, here is what that means for the campaign," the news *is* the market and the latency framework does not apply — see [reflexivity-loops](/concepts/reflexivity-loops) for that case.

## The sf CLI Read

You can rough-estimate latency on a contract by looking at the [position-implied velocity](/learn/position-implied-velocity) profile in the hour after a known news event. `sf scan --tickers <ticker> --history 60m` returns the trade tape for the trailing hour, and you can see how the contract's price moved in the minutes after the news. If the move was concentrated in the first 60 seconds, the latency is sub-minute. If the move spread across the hour, the latency is several minutes to hours.

For the broader phenomenon catalog, see [reflexivity-loops](/concepts/reflexivity-loops) for the news-cycle case where latency interacts with feedback loops, and [settlement-halo](/concepts/settlement-halo) for the special case of news in the final 24 hours of a contract's life. The halo period has its own latency dynamics that look nothing like the normal-regime numbers above.

Latency is the variable I am most likely to forget in the heat of a trade, and the one that most often determines whether the trade was good or just lucky. The thesis is necessary but it is not sufficient — you need the thesis *and* the time window in which the market has not yet caught up to it. Pick the markets where the time window is wide, accept that you cannot beat the markets where it is narrow, and stop trying to compete with machines on news the machines already cover.