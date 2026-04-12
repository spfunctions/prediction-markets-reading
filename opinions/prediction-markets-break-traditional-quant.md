# Why prediction markets break traditional quant models — and what works instead

> Statistical models that crush equities fall apart on prediction markets — because there's no history, no continuity, and exactly one instance of every event.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
Last month I tried to backtest a simple question: "Will the US enter a recession in 2026?"

On Kalshi, the contract is KXRECSSNBER-26. It trades around 40¢. I wanted to know: historically, when this contract has traded at 40¢, how often does the event actually occur? What's the calibration curve? What does the price path look like in the months before resolution?

I pulled the data and stared at it. There's exactly one KXRECSSNBER-26 contract. It was created in early 2025. It will resolve once, in late 2026 or early 2027, when the NBER makes its call. There is no second instance. There is no backtest.

This is the moment where traditional quant breaks.

## Why Factor Models Fail Here

In equities, quantitative strategies work because you have abundant, repeated data. You can backtest momentum across 5,000 stocks over 30 years. You can compute Sharpe ratios. You can build factor models that decompose returns into value, momentum, size, quality. The statistical machinery is powerful because the underlying assumption holds: the future will look roughly like the past, and you have enough past to estimate from.

Prediction markets violate every one of those assumptions:

**No history.** KXRECSSNBER-26 has existed for about a year. There's no equivalent contract from 2024 to compare against. Even if Kalshi had offered KXRECSSNBER-25, the macro environment was completely different — different president, different trade policy, different Fed posture. The label is the same; the underlying reality shares almost nothing.

**Non-continuous.** Equity prices are quasi-continuous — AAPL moves in small increments driven by earnings, macro, sentiment. Prediction market contracts are event-driven. KXRECSSNBER-26 might trade at 40¢ for three weeks, then jump to 65¢ overnight because the April jobs report was catastrophic. There's no smooth price process to model.

**No cross-section.** Factor models work because you can compare many assets at the same time. "Value" means something because you can rank 3,000 stocks by price-to-book. In prediction markets, each contract is a unique proposition. You can't rank "recession 2026" against "Trump wins 2028" on any common factor. They're not comparable along any statistical axis.

**Binary settlement.** Every contract resolves to 0 or 100. There's no partial outcome, no dividend yield, no earnings surprise that's "a little better than expected." The payoff is discontinuous. This means the entire return distribution is bimodal, which breaks most risk models built for continuous returns.

**Reflexivity is the norm.** In prediction markets, the event being priced can be influenced by the price itself. A recession contract trading at 80¢ signals that the market expects a recession, which affects consumer confidence, which affects actual recession probability. This feedback loop doesn't exist in the same way for equities (except in extreme cases like bank runs). It means the price is both a signal about reality and a causal input to reality.

I've talked to quant traders who moved from equities to prediction markets. The smart ones all hit the same wall. Their backtesting frameworks are useless. Their factor libraries are irrelevant. Their risk models assume a return distribution that doesn't exist here.

## What About Market Microstructure?

Some quant approaches do transfer. Orderbook analysis works — bid-ask spread, depth, queue position. Market-making strategies can work if you're careful about inventory risk (though the binary settlement makes inventory management harder). Statistical arbitrage between correlated contracts on different venues has real alpha.

But these are execution strategies, not prediction strategies. They tell you how to trade, not what to trade. The core question — "is this contract mispriced?" — requires something entirely different.

## Causal Reasoning as the Only Viable Approach

If you can't backtest, you need to forward-test. If you can't use statistical patterns, you need to use causal structure. The question isn't "what did this price do historically?" but "what real-world mechanisms drive this outcome, and what do they currently imply?"

Take KXRECSSNBER-26. You can't look at past recession contracts. But you can decompose the question:

- What causes recessions? Demand shocks, supply shocks, financial crises, policy errors.
- Which of those are currently active? The Iran-Hormuz situation is creating a supply shock in oil.
- How severe? Strait of Hormuz closure takes ~20% of global oil transit offline. That's not marginal.
- What are the second-order effects? Oil spike → consumer spending drop → corporate margin compression → layoffs → recession.
- What contracts map to each link in this chain?

Now you have something you can work with. Not a backtest — a causal tree.

## A Real Causal Tree

Here's a simplified version of a thesis I'm actually trading:

```
n1  Iran-US conflict persists           88%
├── n1.1  US strikes continue           99%
├── n1.2  Iran retaliates               85%
└── n1.3  No diplomatic off-ramp        82%
n2  Strait of Hormuz disrupted          75%
├── n2.1  IRGC naval operations active   90%
├── n2.2  Insurance rates spike          85%
└── n2.3  Tanker rerouting > 30 days     70%
n3  Oil stays above $110                64%
├── n3.1  SPR insufficient               80%
├── n3.2  OPEC can't compensate          75%
└── n3.3  Demand doesn't collapse        85%
n4  US recession in 2026                45%
├── n4.1  Consumer spending contracts    60%
├── n4.2  Fed can't cut (inflation)      70%
└── n4.3  Credit conditions tighten      55%
```

Each of these nodes maps to observable reality. n2.2 (insurance rates spike) — you can check Lloyd's of London War Risk premiums. n3.1 (SPR insufficient) — the SPR is at historically low levels, publicly reported weekly. n4.2 (Fed can't cut) — the Fed's own dot plot and the KXFEDDECISION contracts on Kalshi price this directly.

The tree gives you something a backtest never could: a mechanistic explanation of *why* you believe what you believe, with each link independently verifiable.

## From Causal Tree to Tradeable Edge

Once you have a causal tree, you multiply down the chain to get an implied probability for each leaf node. If your tree implies a 45% recession probability, but KXRECSSNBER-26 is trading at 40¢, that's a 5-cent edge. Not huge. But if KXWTIMAX-26DEC31 (WTI above $150) is trading at 38¢ and your model implies 64%, that's a 26-cent edge on the same thesis.

The point is: you're not looking at each contract in isolation. You're looking at the whole causal chain and finding where the market is most wrong about a specific link.

This is where I built a tool to track this. I was doing it in spreadsheets at first — manually updating node probabilities, recalculating implied prices, checking Kalshi orderbooks. It took hours per day and I still missed things. So I built [SimpleFunctions](https://simplefunctions.dev) to automate the mechanical parts: scanning markets, mapping contracts to causal nodes, calculating edges, monitoring for signals that should update the tree.

The tool doesn't tell me what to believe. It tells me: given what I believe, where are the best trades, and what would have to change for me to stop believing it.

## The Update Problem

Here's where causal models have another structural advantage over statistical models: they handle updates naturally.

A statistical model trained on historical data doesn't know what to do with "Reuters reports IRGC deployed new mines in the Strait of Hormuz." It's a novel input that doesn't map to any feature in the training set.

A causal model handles it directly: this is evidence for n2 (Hormuz disrupted), specifically for n2.1 (IRGC naval operations). Increase n2.1 from 90% to 97%. That cascades: n2 goes up, n3 goes up, n4 goes up. Every downstream contract's implied price shifts. New edges appear.

The update is traceable. You can look at the tree and see exactly why recession probability went from 45% to 52% — because one upstream node moved, and the causal chain propagated the change.

Compare this to a neural net that updates its recession prediction from 45% to 52%. Why? "The model's internal weights shifted." Not useful for trading, and not useful for knowing when to exit.

## Knowing When You're Wrong

The most underrated feature of causal models is falsification. Each node has a kill condition. If Iran agrees to a ceasefire (n1.3 drops to near zero), the entire downstream chain collapses. You don't need to wait for your P&L to tell you — the tree tells you before the market fully prices it in.

I've had trades where I exited within hours of a diplomatic signal because the causal tree told me the thesis was dead, while the market was still processing the headline. That's not speed — I'm not a fast trader. It's structure. I knew in advance what would kill the thesis, so when it happened, there was no deliberation.

Statistical models don't have this. They have stop-losses, which are price-based. A stop-loss triggers after the market has already moved against you. A causal kill condition triggers when the reason for your trade changes, which often precedes the price move.

## Limitations

I want to be honest about what doesn't work.

Calibrating the individual node probabilities is hard. When I say "Iran retaliates: 85%," that's my subjective estimate. It's informed by reading, but it's not rigorous in the way a statistical estimate is. Two people with the same information might write 85% and 65% and there's no way to say who's right ex ante.

The causal structure itself is a model, and all models are wrong. I might miss a causal link (e.g., China mediates a deal — is that under n1.3 or is it a separate node?). The tree is only as good as my understanding of the domain.

Liquidity on many prediction market contracts is thin. A 26-cent edge doesn't help if you can only fill $500 before moving the price. Kalshi's orderbooks are getting deeper, but they're still nowhere near equity markets.

And the fundamental issue: you're making a small number of large, concentrated bets. There's no diversification in the way equity factor portfolios diversify across thousands of names. If your causal model is wrong — if you misidentified the key upstream driver — you lose your entire thesis.

## The Structural Implication

Here's the thing that keeps me up at night (in a good way): prediction markets might be the first financial domain where agentic reasoning has a structural advantage over traditional quantitative methods.

In equities, a statistical model with enough data will beat a human analyst most of the time. The data exists, the patterns are real, and machines are better at finding them.

In prediction markets, the data doesn't exist. There's no history to mine. Every event is effectively unique. The only way to price these contracts is to reason about causal mechanisms — to understand *why* things happen, not just *that* they happen.

This is exactly what LLMs are good at. Not in the sense of having perfect judgment, but in the sense of being able to read 50 news articles about Iran, synthesize them into a causal model, compare that model against market prices, and flag discrepancies. The reasoning is the edge, not the data.

I don't think traditional quant shops will dominate prediction markets the way they dominate equities. The toolkit is wrong. The alpha is in causal reasoning, domain expertise, and structured belief updating — not in backtests, factor models, and statistical arbitrage.

And if that's true, then we're watching the emergence of a new kind of quantitative trading: one where the "quant" part isn't statistical analysis, but structured causal reasoning, executed by humans and agents working together.

That's what I'm building toward at [simplefunctions.dev](https://simplefunctions.dev).