# The Valuation Funnel: How to Get From 47,000 Prediction Markets to One Trade

> The 3-stage hierarchy that turns the universe into a position. Filter by indicator, then read the orderbook, then apply causal reasoning. Each layer rejects what the previous layer let through.

**Category:** framework | **Author:** Patrick Liu | **Reading time:** 12 min

---
There are roughly 47,000 active binary contracts on Kalshi and Polymarket combined on any given morning. That number is the problem. A trader who looks at all of them is not a trader, and a trader who looks at none of them is not a trader either. The job is to get from 47,000 to 1 in a way that is repeatable, defensible, and survives a postmortem.

I call the procedure I use the *valuation funnel*, and it has exactly three stages. Each stage takes the output of the previous stage and rejects most of it. The stages are not interchangeable, and the order is not optional — the most common mistake I see is people running them out of order, or skipping the middle one.

## The Three Stages, in Order

**Stage 1 — Filter by indicator.** This is the only stage that touches the full 47,000-market universe. It is pure compute: pull every active market, compute the Tier A indicators (implied yield, cliff risk index, event overround, τ-days), and apply numerical thresholds. The output is a few hundred candidates. No language model touches this stage. No human intuition either. Indicators are the cheap, fast, recall-oriented filter that gets you from 47,000 to ~100 without bias.

**Stage 2 — Read the orderbook.** Stage 1 gives you contracts that *look* good on paper. Stage 2 verifies they are actually tradable. You pull the live bid-ask depth, compute liquidity availability, look at the spread, and ask "if I tried to put on the position the indicator suggested, what would I actually fill at." Most stage-1 candidates die here. A 60% IY contract with three cents of total depth is not a 60% IY trade — it is a 12% IY trade after slippage, if it is anything at all.

**Stage 3 — Apply causal reasoning.** Whatever survives stages 1 and 2 is now a list of maybe five to thirty contracts that have both attractive math and a real orderbook. Stage 3 is where you actually think. You build the causal tree for the event, you check the τ window for known catalysts, you triangulate against reality data and opinion data, and you decide which one of the candidates has a thesis you can defend after the fact. Stage 3 is judgment, and judgment is the part that does not vectorize.

## Why the Order Matters

I have watched a lot of new traders try to run this procedure backwards, usually because they read about a specific event in the news first and *then* go look for a market on it. That is the opposite of the funnel. It is "stage 3 → stage 2 → stage 1," and it almost guarantees you will end up trading the most-discussed market on the platform — which is also the most efficiently priced one, because everyone else read the same news.

The funnel is hierarchical because the indicators in stage 1 are the only thing that can scan the *whole* universe in less than a second. They are how you find the markets that nobody is talking about, where the inefficiency lives. By the time something is news, the market has already moved. By the time you are reading the news, you have ceded the time advantage that the indicator scan was supposed to give you.

The other reason the order matters is cost. Stage 1 is almost free — a SQL query against a snapshot table or a CLI scan. Stage 2 costs an HTTP call per contract and maybe a hundred milliseconds of orderbook parsing. Stage 3 costs you fifteen minutes of focused thinking, and you only have so many of those in a day. If you let stage 3 run on 47,000 markets you go bankrupt on attention before you find any edge.

## Why Putting an LLM in Stage 1 Is Wrong

The most common architectural mistake I see in agent-style trading systems is using a large language model to filter the universe. Someone will say "let's have GPT scan the market list and pick the interesting ones." This sounds reasonable and is wrong on three independent axes.

First, an LLM cannot read 47,000 contracts in a single call. The context window is finite and the cost is linear in tokens, so any LLM-driven scan secretly chunks the universe and runs the filter many times. Each run is non-deterministic, so the ordering of contracts in the chunk biases the output, and two runs will return different "interesting" sets even on the same data. That is not a filter — it is a sampling procedure that has been told to pretend it is a filter.

Second, the LLM does not have access to numbers. It has access to *text representations* of numbers, which it routes through a tokenizer that does not preserve magnitude. Asking an LLM to compare "32¢" and "0.42" and decide which one has higher implied yield is a mathematical operation translated into a guessing game. A SQL query with a `WHERE iy > 0.5` clause does the same thing in microseconds and gets it right.

Third — and this is the one that costs real money — the LLM will reliably pick contracts that *sound* interesting (presidential elections, AI safety, big news events) over contracts that *are* interesting (illiquid sibling markets, range-bound MM opportunities, events with no opinion data flowing through them). The LLM is biased toward narratively rich markets because its training set is narratively rich. The whole point of stage 1 is to find markets with no narrative, because those are the ones with no story-priced bias yet.

So stage 1 is computational. Indicators only. The LLM, if you use one at all, lives in stage 3 — not as a filter, but as a thesis assistant that can read a single contract's full context and help you build the causal tree.

## Worked Example: A Morning Scan

I run `sf scan --by-iy desc --min-tau 14 --min-depth 100` on a typical Tuesday morning. The flags say: rank by descending implied yield, exclude anything with less than 14 days to resolution (so IY does not explode), require at least 100 cents of total orderbook depth (a cheap proxy for stage 2 being passable). The scan touches the whole 47,000-market universe and returns 142 candidates in about 800ms.

That is stage 1 done. I now have 142 markets where the math says there is potentially attractive yield and the depth is at least non-trivial. I have not looked at any of them as humans look at markets. I have not read a single ticker name yet.

I then run `sf scan --warm --tickers <142>` to pull the live orderbooks. About 90 of the 142 had stale depth numbers in the snapshot table (the warm-regime cron only covers the top 500 by 24-hour volume; the rest are from cold scans), and after the warm refresh I lose roughly 60 of them — their actual depth is well below what the snapshot suggested. I am down to 80 candidates. That is stage 2.

Now I look at the 80 ticker names. About 50 are in event families I do not have a thesis for (sports outcomes, foreign elections I cannot evaluate). I drop those. The remaining 30 are all in event families I can reason about. I pick three of those that have the highest IY combined with the lowest CRI (so the market is paying me well *and* it is not actively repricing — meaning my entry is unlikely to be immediately washed out). I build the causal tree for each of the three. I check the news in the τ window for known catalysts. I check the position-implied velocity to see if there is one-sided flow I should worry about.

Of the three, one has a clean causal tree with no obvious catalyst risk. That is the trade. From 47,000 to 1, in about forty minutes, with the 39 minutes of human attention spent only on the contracts that actually deserved attention.

## Where the Funnel Breaks

Three failure modes worth naming.

**Stage 1 misses things.** The indicator filters can only find what is already reflected in the snapshot table. If a market just opened and the indicators have not been computed yet, it is invisible to your scan. The mitigation is to run a manual cold scan periodically, or to have a "new markets" feed that sidelines stage 1 entirely for the first 24 hours of a contract's life.

**Stage 2 is venue-dependent.** The orderbook on Kalshi is a real continuous limit-order book; the orderbook on Polymarket is an AMM with a specific liquidity curve. "Depth" means different things on each, and a stage 2 check that works for one venue does not transfer to the other. You need different stage-2 logic per venue, and in cross-venue arbitrage you need *both* checks before stage 3.

**Stage 3 is the bottleneck and you cannot parallelize it.** The whole point of stage 3 is that it is judgment, and judgment is the part of you that gets tired. If you are running the funnel ten times a day you are not running stage 3 properly on any of them. The funnel is designed to give you three to five high-quality stage-3 candidates per day, not fifty.

The funnel is also incomplete in one specific way: it does not handle the case where the market you want to trade does not exist yet, and the right move is to wait for it to be listed (or to lobby the venue to list it). That is a meta-stage above stage 1 and lives in your head, not in your scan.

## How This Article Connects to the Rest of the Stack

The valuation funnel is the spine of everything else on this site. The [pm-indicator-stack](/concepts/pm-indicator-stack) page enumerates the five indicators that power stage 1. The [endogenous-vs-reality-vs-opinion-data](/concepts/endogenous-vs-reality-vs-opinion-data) page formalizes the three data sources that stage 3 has to reconcile. The [null-as-signal](/concepts/null-as-signal) page explains why the *absence* of data in stages 1 and 2 is sometimes the entry condition rather than a defect. The [prediction-market-valuation-theory](/concepts/prediction-market-valuation-theory) capstone ties all of this back together.

If you want the implementation: [computing-implied-yield-from-kalshi-tickers](/technicals/computing-implied-yield-from-kalshi-tickers) is the code for the stage-1 indicator that does the most work, and [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) is the opinion piece on why that indicator is the right framing in the first place.

The funnel is not original to me. Every fixed-income desk runs some version of it, and every quant trader I have ever worked with has a "scan, then check, then think" procedure. What is original — if anything is — is doing it on prediction markets, where the universe is small enough to scan exhaustively and the contracts are exotic enough that the standard institutional infrastructure does not exist yet. That is the gap the funnel is designed to fill.