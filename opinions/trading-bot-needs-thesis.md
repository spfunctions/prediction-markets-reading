# Why Your Trading Bot Needs a Thesis, Not Just a Signal

> Signal-chasing bots lose money in prediction markets because they confuse price movement with probability changes. Here is the fix.

**Category:** essay | **Author:** SimpleFunctions | **Reading time:** 7 min

---
The standard trading bot architecture looks like this: ingest data, compute signals, execute trades. Signal goes up, buy. Signal goes down, sell. The logic is clean, the code is simple, and the backtest looks great.

Then you deploy it on a prediction market and it loses money.

Not because the signals are wrong — they're often directionally correct. But because prediction markets are not continuous price discovery instruments. They're binary outcome instruments trading in a thin orderbook. The dynamics are fundamentally different, and signal-chasing bots break against these dynamics.

## The Failure Mode

Here's a concrete example. You build a bot that trades "Recession 2026 YES" on Kalshi. Your signal is a composite of macro indicators: unemployment claims, yield curve, PMI, consumer confidence. When the composite deteriorates, the bot buys. When it improves, the bot sells.

Monday morning, the ISM Manufacturing PMI comes in at 47.2 — below the 48.0 consensus. Your signal ticks bearish. The bot buys Recession YES at 36 cents.

Tuesday, a Reuters headline reads: "US-Iran tensions ease as diplomatic channel opens." The Recession YES contract drops to 31 cents — not because the economy improved, but because geopolitical risk (an upstream driver of oil prices, which drive recession probability) just decreased. Your macro signal didn't change. PMI is still at 47.2. But the contract moved 5 cents against you.

Wednesday, Iran launches a retaliatory strike and the headline reverses. Recession YES spikes back to 38 cents. Your bot, which sold at 31 after seeing the "dip," just round-tripped and lost 5 cents plus fees.

This is the failure mode: the bot tracks *price* signals but doesn't understand *why* the price moved. It treats a 5-cent drop driven by geopolitical de-escalation the same as a 5-cent drop driven by economic improvement. They're not the same. One is a change in an upstream causal driver. The other is a change in the outcome variable itself.

## Price Moved vs Probability Changed

This distinction is everything in prediction markets:

**Price moved** = the contract's last trade or best bid/ask changed. This can happen because of:
- Informed trading (someone with real information entered the market)
- Noise trading (someone bought/sold for non-informational reasons)
- Liquidity events (a large order consumed the book temporarily)
- Correlated asset moves (oil moved, so recession reprices)

**Probability changed** = the actual likelihood of the underlying event shifted. This happens because:
- A causal driver changed (war escalated, Hormuz closed, oil spiked)
- Base rates updated (new economic data released)
- The event space narrowed (time passed, ruling out some scenarios)

A good bot needs to distinguish between these. When the probability *actually* changed — because an upstream causal driver shifted — the bot should update its position. When the price moved due to noise or liquidity, the bot should either do nothing or fade the move.

## Why Context Beats Speed

In equity and crypto markets, speed matters because you're competing to react to the same information. A millisecond advantage in processing a Fed announcement translates to a real edge.

In prediction markets, speed is mostly irrelevant. Why? Because:

1. **The orderbook is thin.** Moving fast to hit a 100-lot offer doesn't generate meaningful P&L. There isn't a 10,000-lot resting order to race for.

2. **Events resolve over days/weeks, not milliseconds.** "Recession 2026" is a 9-month bet. Being first to react to a single headline by 30 seconds doesn't compound into a meaningful edge.

3. **The edge is in interpretation, not reaction.** Two traders see the same Reuters headline about Iran diplomacy. One sells Recession YES because "tensions eased." The other checks whether the diplomatic channel actually affects the causal chain (Does it change war probability? Does it change Hormuz closure probability? Does it change oil supply?) and concludes: no, this is a gesture, not a resolution. The second trader's advantage isn't speed — it's *context*.

This is why thesis-driven bots outperform signal-chasing bots in prediction markets. The signal bot reacts to the headline. The thesis bot evaluates whether the headline changes the causal model.

## A Concrete Comparison

Let's compare two bots processing the same headline: "White House announces emergency SPR release of 50M barrels."

**Signal bot:**
- Sees headline → oil-bearish signal fires
- Checks "WTI $150 YES" contract → sells at 38 cents
- Result: locked in a sale based on headline sentiment

**Thesis bot:**
- Sees headline → checks causal tree
- Node: "SPR compensates for Hormuz disruption" — current probability 15%
- Analysis: 50M barrels is roughly 2.5 days of US consumption. Hormuz disruption removes ~17M barrels/day from global supply. The SPR release is a rounding error on the supply gap.
- Conclusion: causal tree node doesn't meaningfully shift. Fair value for WTI $150 unchanged.
- Action: no trade. Or better: buy if the signal bot's selling pushed the price below fair value.

The signal bot sold into a headline. The thesis bot checked whether the headline actually changed the underlying probability. In this case, it didn't — the SPR release was theater, not substance. The signal bot gave away edge to anyone paying attention to the causal chain.

## Building Thesis Infrastructure

What does a thesis-driven bot actually need?

**1. A causal model, not a signal feed.** Instead of subscribing to a price stream and computing technical indicators, the bot operates on a causal tree. Each node has a probability, an evidence base, and upstream/downstream links.

**2. A signal evaluation layer.** When a new piece of information arrives (headline, data release, price move), the bot doesn't trade immediately. It first asks: "Does this change any node in my causal tree?" If yes, which node? By how much? What does the downstream cascade look like?

**3. A staleness detector.** Most signals are noise. The causal tree provides a natural filter: if an incoming signal doesn't map to any node, it's irrelevant. If it maps to a node but doesn't change the probability by more than a threshold (say, 2%), it's below the noise floor.

**4. A position manager that thinks in thesis terms.** Instead of "I'm long 500 shares of Recession YES," the bot thinks: "I'm expressing a thesis that oil-driven inflation leads to recession, with 72% implied probability vs 35% market price. My position is sized to the edge and my confidence in the upstream drivers."

This is what SimpleFunctions provides as infrastructure. The causal tree is the thesis. Signals are evaluated against the tree. Edges are computed from the divergence between thesis-implied probabilities and market prices. The bot doesn't chase signals — it operates on structured, verifiable beliefs about the world.

```bash
# The bot reads structured context, not raw prices
$ sf context <thesis-id> --json

# New information evaluated against the tree
$ sf signal <thesis-id> "SPR release of 50M barrels announced" --type news

# Tree updates only if the signal is causally relevant
$ sf edges  # Fair values recomputed from updated tree
```

## The Takeaway

If you're building a bot for prediction markets, the competitive advantage isn't faster data feeds or more signals. It's a thesis — a structured causal model that tells the bot *why* it believes what it believes, *what* would change its mind, and *when* to ignore the noise.

Signals are easy to generate and impossible to interpret without context. A thesis provides the context. Build the thesis first. The signals will know where to go.