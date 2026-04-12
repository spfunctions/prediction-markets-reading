# Prediction Markets Are Already Pricing the Post-Terminal-Value World

> Chamath says terminal value is collapsing. Prediction markets never had terminal value to begin with — every contract has an expiry date. They're the native pricing instrument for a short-duration world.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 4 min

---
Chamath Palihapitiya recently argued that terminal value — the foundation of DCF valuation — is collapsing. AI compresses innovation cycles. Moats erode faster. The 10-year cash flow projection that underpins most equity valuations is increasingly fiction.

He's right. But there's a market that figured this out years ago.

## Prediction Markets Never Had Terminal Value

Every prediction market contract has an explicit expiry date. "Will there be a US recession in 2026?" resolves on December 31, 2026. "Will WTI hit $150?" resolves when it does or when the contract expires. There's no terminal value — no "and then cash flows continue growing at 3% in perpetuity."

This isn't a limitation. It's a feature.

In a world where the future is increasingly uncertain beyond 12-18 months, an instrument that *explicitly* prices short-duration outcomes is more honest than one that pretends to see 10 years ahead.

## DCF vs Causal Models

Traditional equity analysis uses DCF: estimate future free cash flows, discount them back to present value, add a terminal value that typically accounts for 60-80% of the total valuation. The terminal value is an act of faith — "we believe this company will keep generating cash flows similar to today's, growing at some modest rate, forever."

Prediction markets use something closer to causal models. You don't need to know what the world looks like in 10 years. You need to know:

- What are the upstream drivers? (War, oil supply, Hormuz strait)
- How do they causally connect to the outcome? (War → Hormuz closure → oil spike → recession)
- What probability does each link carry?
- Where does the market disagree with your model?

This is fundamentally shorter-duration thinking. And it's more rigorous, because each node in the causal chain is independently verifiable.

## A Live Example

Our [Iran war thesis](/thesis/iran-war) is a real-time causal model tracking how geopolitical conflict flows through to economic outcomes:

```
War persists (99%) → Hormuz blocked (97%) → Oil elevated (100%) → Recession (98%)
```

Each node maps to tradeable contracts on Kalshi and Polymarket. The system finds where market prices diverge from thesis-implied prices — edges. It monitors 24/7 with a 15-minute heartbeat cycle: scanning news, refreshing prices, enriching orderbooks, evaluating new signals against the causal tree.

The implied return updates daily. The confidence score shifts with real-world events. There's no terminal value assumption — just a structured model of how the present leads to near-term outcomes, with specific contracts that will resolve to 0 or 100.

## The Structural Advantage

Chamath's thesis implies that short-duration analysis will increasingly outperform long-duration analysis. If that's true, prediction markets have a structural advantage over equity markets:

**Equity markets** price long-duration cash flows and embed terminal value. When terminal value collapses, the pricing framework breaks. You get the S&P 500 swinging wildly on every new AI model release because no one knows how to value "5 years of cash flows if GPT-7 exists."

**Prediction markets** price specific, short-duration outcomes. There's no terminal value to collapse. "Recession in 2026 YES at 35¢" is a claim about the next 9 months, backed by observable upstream drivers (oil prices, Fed policy, consumer spending). When new information arrives, the model updates and the price moves. Clean, bounded, verifiable.

## What This Means for Traders

If you believe the world is moving toward shorter planning horizons — and the evidence strongly suggests it is — then prediction markets are not a niche. They're the leading indicator of how all markets will eventually price risk.

The traders who win in this environment aren't the ones with better DCF models. They're the ones who:

1. **Think in causal chains**, not terminal value
2. **Update continuously** as new information arrives
3. **Map specific** upstream drivers to downstream outcomes
4. **Trade the edges** where the market hasn't caught up to the causal model

This is what SimpleFunctions is built for. Not to replace your judgment, but to give it structure — a causal tree you can inspect, edges you can quantify, and a monitoring system that tells you when reality diverges from your model.

The terminal value era is ending. The causal model era is just beginning.