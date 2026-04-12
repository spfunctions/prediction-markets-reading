# SimpleFunctions vs Oddpool vs Raw Kalshi API — Which Prediction Market Tool Should You Use?

> A practical comparison of three approaches to prediction market tooling: agentic reasoning (SimpleFunctions), data aggregation (Oddpool), and direct exchange access (Kalshi/Polymarket APIs).

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 9 min | **Published:** 2026-04-02

---
Every serious prediction market participant eventually faces the same question: should I build my own pipeline on top of the raw exchange APIs, use a data-centric aggregation service like Oddpool, or go with an end-to-end agentic platform like SimpleFunctions?

The answer depends on what you are trying to do. This post breaks down the three approaches honestly — their strengths, their weaknesses, and when you should reach for each one.

---

## The Three Approaches at a Glance

| Dimension | SimpleFunctions | Oddpool | Raw Kalshi / Polymarket API |
|---|---|---|---|
| **Primary value** | Thesis-driven reasoning + execution loop | Cross-venue data aggregation + whale tracking | Direct exchange access, full control |
| **Data sources** | Kalshi, Polymarket, X/Twitter, news, traditional markets | Kalshi, Polymarket, Opinion | Single exchange per API |
| **AI / reasoning** | Causal tree decomposition, edge detection, scenario analysis | None — pure data pipeline | None |
| **Trading** | Automated order placement via CLI/MCP/REST | No execution — data only | Full exchange order book access |
| **Interface** | CLI, MCP server, REST API, interactive agent | REST API, WebSocket, web dashboard | REST, WebSocket, FIX (Kalshi) |
| **Pricing** | Free explorer tier; paid for thesis engine | Free search; Pro $30/mo; Premium $100/mo | Free (exchange fees on trades) |

---

## SimpleFunctions: Context Flow + Thesis Engine

SimpleFunctions is built around the idea that raw market data is necessary but not sufficient. What matters is *structured context*: data that has been filtered, reasoned about, and connected to a thesis you actually hold.

### How It Works

1. **You state a thesis** in plain English: "Oil stays above $100 for 6 months."
2. The system **decomposes it into a causal tree** — verifiable assumptions, each with a probability and importance weight.
3. It **maps those assumptions to live prediction market contracts** on Kalshi and Polymarket.
4. It computes **edges**: the gap between what the market prices and what your causal model implies.
5. It **monitors 24/7**, re-evaluating every 15 minutes, pulling signals from X/Twitter, news, and traditional markets.

```bash
# Install and scan — no API key needed
npm install -g @spfunctions/cli
sf scan "recession"       # live Kalshi + Polymarket prices
sf query "fed rate cut"   # LLM-enhanced answer + contracts + X sentiment

# Create a thesis (optional — enriches context)
sf create "The Fed will cut rates at least twice by Q3 2026."
sf context <id>           # causal tree, edges, evaluation history
sf edges                  # top mispriced contracts across all theses
```

### Unique Strengths

- **Edge detection**: not just price data but a framework for identifying mispricings relative to your thesis.
- **Causal reasoning**: decomposition into verifiable sub-assumptions means you can pinpoint *which part* of your thesis the market disagrees with.
- **MCP server**: connect directly to Claude Code, Cursor, or any MCP-compatible AI agent. Your agent gets 25+ tools including `scan_markets`, `get_context`, `inject_signal`, `what_if`, and `trigger_evaluation`.
- **Cross-source context**: combines prediction market prices with X/Twitter posts, traditional market data (equities, commodities, rates), and news — all in one call.
- **Trading loop**: once an edge is identified, the system can place orders through your connected exchange accounts.

### Limitations

- Thesis-centric design means the system is most powerful when you have a directional view. Pure exploration mode exists but isn't the core value proposition.
- Not a historical data archive — it's focused on forward-looking context, not backtesting pipelines.
- Newer platform — smaller community than raw exchange SDKs.

---

## Oddpool: The Data Infrastructure Layer

Oddpool occupies a different part of the stack. It is a pure data aggregation and intelligence layer: it collects, normalizes, and enriches prediction market data from Kalshi, Polymarket, and Opinion, then exposes it through clean APIs and dashboards.

### What Oddpool Does Well

- **Cross-venue search**: the only full-text search engine across multiple prediction market exchanges. Search "tariffs" and get matched markets from both Kalshi and Polymarket, side by side with live odds.
- **Whale tracking**: every trade over $500 on both Kalshi and Polymarket is flagged and delivered in real time. This is genuinely valuable signal — large trades often precede price moves.
- **Arbitrage scanning**: automatically finds cross-venue pricing discrepancies and calculates net profit after fees. Traders using Oddpool reportedly spotted 762+ arbitrage opportunities in a single week.
- **WebSocket feeds**: real-time streaming of trades, price changes, and whale alerts via WebSocket — essential for latency-sensitive applications.
- **Clean API design**: well-documented REST and WebSocket endpoints for programmatic access.

### Pricing Tiers

- **Free**: market search, basic dashboards (NFL, elections, Bitcoin).
- **Pro ($30/month)**: live scanners, whale alerts, cross-venue comparisons.
- **Premium ($100/month)**: Arbitrage API, price spread data, full search API, unlimited WebSocket events.

### Limitations

- **No reasoning layer**: Oddpool delivers data, not judgment. It won't tell you *why* a price discrepancy exists or whether a whale trade is informed.
- **No execution**: you can't place trades through Oddpool. It's an intelligence layer, not a trading platform.
- **No thesis framework**: there's no way to express "I believe X" and have the system continuously evaluate that belief against market data.
- **No broader context**: purely focused on prediction market data — no X/Twitter sentiment, no news, no traditional market prices.

---

## Raw Kalshi/Polymarket API: Maximum Control

The raw exchange APIs give you direct, unmediated access to the order book. Kalshi provides REST v2, WebSocket, and FIX 4.4 endpoints. Polymarket exposes its CLOB through REST and WebSocket, with official SDKs in Python, TypeScript, and Rust.

### When Raw API Is the Right Choice

- **Algorithmic trading at scale**: if you're running a systematic strategy that needs sub-second order management, the raw API (especially Kalshi's FIX protocol) provides the lowest latency.
- **Custom backtesting**: building historical datasets and backtesting frameworks that are tailored to your strategy.
- **Full order book control**: limit orders, cancellations, position sizing — you control every parameter.
- **Institutional requirements**: compliance, audit trails, direct custody — the raw API is the only choice.

```python
# Kalshi Python example
from kalshi_python import ApiInstance
api = ApiInstance(host="https://api.kalshi.com/trade-api/v2")
api.login(email="you@example.com", password="xxx")

markets = api.get_markets(status="open", limit=20)
for m in markets:
    print(f"{m.ticker}: {m.yes_ask}c / {m.no_ask}c  vol={m.volume}")
```

```typescript
// Polymarket CLOB example
import { ClobClient } from "@polymarket/clob-client";

const client = new ClobClient("https://clob.polymarket.com");
const markets = await client.getMarkets();
markets.forEach(m => console.log(m.question, m.tokens[0].price));
```

### Limitations

- **You build everything yourself**: normalization, monitoring, alerting, cross-venue matching, context enrichment — all custom code.
- **Auth is non-trivial**: Kalshi tokens expire every 30 minutes requiring re-login; Polymarket uses API key + secret + passphrase with HMAC signing.
- **Single venue**: each API only covers its own exchange. Cross-venue analysis requires integrating both separately.
- **No intelligence layer**: you get prices and order books, not insights or edges.
- **Maintenance burden**: API versions change, endpoints get deprecated, rate limits shift. Ongoing operational cost.

---

## Decision Framework: When to Use What

### Use SimpleFunctions when:
- You have a thesis and want to know if the market agrees with you.
- You want AI-powered analysis that connects prediction market data to news, social media, and traditional markets.
- You want an MCP server for your AI coding agent (Claude Code, Cursor).
- You want edge detection and automated monitoring without building a custom pipeline.
- You're a researcher, analyst, or trader who thinks in narratives and causal chains.

### Use Oddpool when:
- You need cross-venue data aggregation with clean APIs.
- Whale tracking is a core part of your strategy — following large, informed traders.
- You're looking for pure arbitrage opportunities between exchanges.
- You want a real-time WebSocket feed of market events without building the plumbing yourself.
- You're building your own analytics dashboard or trading system and need a reliable data layer.

### Use Raw Kalshi/Polymarket API when:
- You're building a systematic, low-latency trading system.
- You need full order book control for algorithmic strategies.
- You're doing custom academic research or backtesting.
- You have institutional compliance requirements.
- You have the engineering resources to build and maintain a custom pipeline.

### Combine Them

These tools aren't mutually exclusive. A sophisticated setup might look like:

1. **Oddpool** for real-time whale alerts and arbitrage scanning.
2. **SimpleFunctions** for thesis management, edge detection, and AI-powered context.
3. **Raw Kalshi API** for execution when your strategy requires low-latency order management.

---

## Final Thoughts

The prediction market tooling landscape is maturing fast. Two years ago, the raw exchange API was the only option. Today, there are meaningful choices at every layer of the stack.

Oddpool has earned its place as the best pure data aggregation layer — if you need whale tracking, cross-venue search, or arbitrage detection, it's the right tool.

SimpleFunctions is building something different: a reasoning layer on top of the data. The thesis engine, causal decomposition, and edge detection represent a new category — agentic infrastructure for prediction markets. If you think in terms of "I believe X, show me where the market disagrees," SimpleFunctions is the only tool that supports that workflow natively.

And the raw APIs will always have a place for teams that need maximum control and are willing to invest the engineering effort.

Choose based on what you're building, not on hype. The best tool is the one that matches your actual workflow.