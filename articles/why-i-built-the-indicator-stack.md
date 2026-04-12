# Why I Built the Indicator Stack

> A personal account of the frustration that produced IY, CRI, EE, LAS, and CVR — and the rule I locked in early that made the whole thing tractable: pure compute first, language model never.

**Category:** insights | **Author:** Patrick Liu | **Published:** 2026-04-09

---
The first version of my prediction-market workflow was a bookmark folder. I called it "live markets." It had eleven Polymarket links and four Kalshi links and I cycled through it every morning for about three months in the spring of last year. The cycling lasted exactly as long as it took for the bookmark folder to break, which was the morning I added the twelfth link and realized I had no idea what I was supposed to do with the new market other than "open it and stare."

The indicator stack was what I built to replace the bookmark folder. This is the story of why, and of the rule I locked in on day one that made the whole thing tractable.

## The Frustration

By March I had been manually scanning prediction markets for about six months. I had developed rough heuristics: I knew I liked Fed-decision contracts, I knew I distrusted election contracts, I knew I had a soft spot for tech-IPO timing markets even though I did not really have an edge in them. The bookmark folder was the embodiment of those heuristics — it was the markets I had already decided to care about, and my morning routine was "look at each one and see what changed."

The problem with this routine is that it scaled exactly nowhere. The moment I added a new event family — say, the Fed-decision family — I had to manually find every contract in that family, decide which ones were worth bookmarking, and then add to the morning rotation. The cognitive load of just *maintaining the bookmark folder* was eating most of my actual analytical time. I was spending forty minutes a morning looking at maybe fifteen markets and an hour and a half a week curating the list of which fifteen to look at.

The deeper issue was that I was using my own attention as the filter. I was the function that decided which markets were interesting. My eyes, my mood, my tab history — those were the inputs. I was a janitorial filtering layer for the entire prediction-market universe and I was clearly the bottleneck.

I decided in mid-March that the bookmark folder had to die, and that whatever replaced it needed to be a function that I could *run* rather than a list I had to *maintain*.

## The Initial Bad Plan

The first replacement I designed lasted about a week. It was an LLM. I am embarrassed about it now but at the time it seemed obvious — I had been told repeatedly by people with more confidence than judgment that "LLMs are how you find things in unstructured data," and prediction markets are unstructured data, so the obvious move was to dump every market into a context window every morning and ask the model "which of these are interesting and why."

I did this for six days. The output was indistinguishable from what I would have written if I had been forced to write a paragraph about every market I had no opinion on. It was articulate, plausible, completely unable to tell me which market I should size into, and cost me about $40 a day in API fees. The deeper failure was that the model had no way to ground its judgment in the *actual* numbers — it would happily tell me that some contract was "interesting" because the price was "moving" without ever computing whether the move was structurally meaningful or just settlement noise.

By day six I had made the rule that defined everything I built afterward: **filter by pure compute first; only let the language model anywhere near a market that has already passed the compute filter.** No language model in the funnel's narrow end. Compute is cheap, deterministic, debuggable. Compute is what scales to 47,000 markets in 0.3 seconds. Compute is what tells you which markets are worth the language model's attention, which is finite and expensive and easily fooled.

That single rule — *pure compute first, LLM never in the wide end* — is the most important architectural decision I have ever made on this stack. Everything that came after followed from it.

## The Five Numbers

Once I committed to compute-first, the question became: *which* compute? What numbers, computed cheaply across the whole universe, could replace my morning bookmark folder?

I started with one: implied yield. I had been computing it by hand on the back of envelopes since [the morning in October when I stopped trusting raw probabilities](/blog/the-day-i-stopped-trusting-raw-probabilities), and it was the only number I had that consistently surprised me when I looked at it. So I made it the first column. `(1 / p) ^ (365 / τ) - 1`. Vectorize across the universe. Sort descending.

The first day I had IY working as a sortable column, my morning routine collapsed from 40 minutes to 6. I did not need the bookmark folder anymore. I had a list, generated in real time, of every contract on Kalshi and Polymarket whose annualized return was above some threshold I had picked. The interesting contracts mostly *announced themselves*. I had to make essentially zero filtering decisions.

But IY alone has obvious failure modes. A very high IY can be a real opportunity or it can be a market about to expire worthless or it can be a market with no liquidity or it can be a market in the middle of a settlement noise spike. So I needed a second number to *gate* IY, and the obvious one was velocity-of-price-change scaled by remaining time. That became the [cliff risk index](/learn/cliff-risk-index): `|Δp/Δt| × τ_remaining`. CRI tells you whether the IY is on a contract that is *moving* or on a contract that has been stuck for weeks. A high IY with low CRI is usually a bad sign — the market has decided this is a long-shot and you are not seeing fresh information, you are seeing structural pessimism. A high IY with high CRI is an actively-deciding market and is much more likely to be a real opportunity.

Two numbers got me through April. By May I had added the [event overround](/learn/event-overround) — `Σ pᵢ - 1` across mutually exclusive outcomes — for the multi-outcome events where the cross-outcome arithmetic was telling me something the single-market view could not. EE is the most over-celebrated indicator in my stack because newcomers see "arbitrage" and get excited; in practice, raw EE almost never overcomes fees. The interesting use is *changes in EE over time*, which catches the moment when one outcome in a sibling family is being aggressively repriced relative to the rest.

That gave me three numbers. IY, CRI, EE. Tier A. Always available, computed on every market every morning, sorted into a single screener view. These three got me about 80% of the way to the workflow I have now.

The remaining 20% turned out to be the harder problem. Tier A indicators tell you what to *look at*. They do not tell you what you can *trade*. Half the markets that pass the IY filter have no orderbook, half the markets that pass CRI are too thin to fill a position, half the markets that pass EE are on events with overlapping outcomes. I needed indicators that captured *executability* and *liquidity*, not just structural opportunity.

This is where I added [LAS](/learn/liquidity-availability-score) — bid_depth + ask_depth scaled by spread — and PIV (the position-implied velocity from the 1¢-delta tracker) and CVR (cross-market thesis spread). These three became Tier B, and they have one critical property the Tier A indicators do not: they are *sparse*. They are computed only on the top ~500 markets by 24-hour volume, because computing them on the full universe would require pulling orderbooks for every contract every cron tick, which the proxy budget cannot afford.

The sparseness was originally a budget constraint. It became a feature. *Null on Tier B is a positive selector for unloved markets*, and the unloved markets turn out to be the [maker setups I described in the post about empty orderbooks](/blog/when-the-orderbook-is-empty-you-have-information). What started as "we can't afford to compute LAS on every market" became "we have a free signal that tells us which markets nobody else has noticed yet." That reframe — *null is signal, not defect* — happened in late summer and was, I think, the second-biggest conceptual shift after the compute-first rule.

## What the Stack Replaces

The stack does not tell me what to trade. It tells me what to *look at*. The judgment about whether a high-IY high-CRI low-LAS contract is actually a position I want is mine. The stack just makes it possible for me to make that judgment on the right ten contracts a morning instead of having to make it on the wrong fifteen contracts in my bookmark folder.

The five numbers — IY, CRI, EE, LAS, PIV (with CVR as a sixth that I use less often) — are the indicator stack. They live in code at `src/lib/indicators/compute.ts` for the Tier A ones. They feed into the [`sf scan`](/concepts/pm-indicator-stack) command, which is my one-line replacement for the bookmark folder. The default invocation is `sf scan --by-iy desc --min-cri 0.5` and that single command does in 0.3 seconds what the bookmark folder did in 40 minutes, plus it covers the entire Kalshi+Polymarket universe instead of the eleven contracts I had hand-curated.

## The Rule That Held

I want to end this with the one thing I would tell anyone trying to build their own version: keep the language model out of the wide end of the funnel. Forever. The temptation is enormous because LLMs are good at sounding insightful, and "interesting market detection" is exactly the kind of task where sounding insightful is indistinguishable from being insightful — until you backtest and discover it was all noise.

Compute first. Language model only on the narrow end, only on the contracts that have already survived the indicator gates. The compute-first rule is what makes the whole thing reproducible, debuggable, and survivable when the API budget is tight. It is also what made the indicator stack possible at all — without it, I would still be paying $40 a day to read articulate paragraphs about contracts I should have ignored.

A more formal version of all of this lives in [the indicator stack concept page](/concepts/pm-indicator-stack), which goes through the math and the architecture. This essay was about the *story*. The story is short: I had a bookmark folder, the bookmark folder broke, I tried to replace it with an LLM and that failed, and what eventually worked was five numbers and a rule about where compute belongs.

The five numbers are still there. The rule has not budged in over a year.