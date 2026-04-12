# The Prediction Market Data Stack: From Raw Prices to Actionable Intelligence

> The six layers of the prediction market data stack — from raw exchange ticks to executed trades — and how to build or buy each one.

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 11 min | **Published:** 2026-04-02

---
A prediction market price is just a number. 37 cents. 62 cents. 84 cents.

To turn that number into a decision — whether to trade, hedge, alert, or adjust a portfolio — you need layers of processing that transform raw exchange data into actionable intelligence. This post maps the full prediction market data stack, from the exchange's matching engine to a reasoned trading decision, and shows how each layer works.

---

## The Six Layers

```
Layer 6: Execution           →  Place orders, manage risk, track P&L
Layer 5: Edge Detection      →  Where does the market disagree with reality?
Layer 4: Causal Reasoning    →  Why should this price be different?
Layer 3: Context Enrichment  →  What does the world say about this event?
Layer 2: Normalized Data     →  Clean, cross-venue, comparable prices
Layer 1: Raw Exchange Data   →  Order books, trades, ticks from each venue
```

Most prediction market tools operate at layers 1-2. A few reach layer 3. SimpleFunctions is designed to cover all six.

Let's walk through each layer.

---

## Layer 1: Raw Exchange Data

This is the foundation — the direct feed from each exchange's matching engine.

### What It Includes

- **Order book snapshots**: bid/ask prices and sizes at each level.
- **Trade prints**: every executed trade with timestamp, price, size, and side.
- **Market metadata**: ticker, title, description, expiry, settlement rules.
- **Account data**: your positions, balances, order status (authenticated).

### Sources

- **Kalshi**: REST v2 (`api.kalshi.com/trade-api/v2`), WebSocket (`wss://api.kalshi.com/trade-api/ws/v2`), FIX 4.4 for institutional access.
- **Polymarket**: CLOB REST (`clob.polymarket.com`), Gamma API for market discovery, WebSocket for streaming.

### The Challenge

Raw data from each exchange is siloed, uses different schemas, different auth mechanisms, and different market identifiers. "Will the Fed cut rates in June?" might be `KXFEDRATE-26JUN` on Kalshi and a different condition ID on Polymarket. There's no shared key.

```python
# Raw Kalshi data — useful but venue-specific
{
    "ticker": "KXFEDRATE-26JUN-T4.25",
    "title": "Fed funds rate below 4.25% at June meeting",
    "yes_ask": 42,
    "yes_bid": 40,
    "volume": 18420,
    "open_interest": 94000
}
```

### Building It Yourself

This layer is straightforward if you're integrating a single exchange. Most teams can get basic market data flowing in a day using the official SDKs. The complexity arrives when you need reliability (reconnection logic, token refresh, rate limit handling) and when you add a second venue.

**Estimated build effort**: 1-3 days for a single venue. 1-2 weeks for both, with proper error handling.

---

## Layer 2: Normalized Cross-Venue Data

This is where raw exchange data becomes comparable. The same event — "Fed rate cut in June 2026" — needs to be matched across venues so you can see both prices side by side.

### What Normalization Involves

- **Entity resolution**: matching Kalshi tickers to Polymarket condition IDs. This is harder than it sounds — titles don't always match exactly, and market structures differ (Kalshi's binary yes/no vs. Polymarket's token pairs).
- **Price normalization**: converting to a common format (typically cents, 0-100 scale representing implied probability).
- **Volume normalization**: Kalshi reports in contracts, Polymarket in USDC. Converting to comparable units.
- **Timestamp alignment**: syncing data from venues with different latency profiles.

### Who Does This

- **Oddpool** is the strongest option here. Its cross-venue search and arbitrage scanner handle entity resolution, price normalization, and spread calculation across Kalshi, Polymarket, and Opinion.
- **SimpleFunctions** normalizes across Kalshi and Polymarket as part of its scan and query pipeline, with the added benefit of including non-prediction-market data sources.

### Building It Yourself

Entity resolution is the hard part. Fuzzy string matching on market titles gets you 70% of the way. The remaining 30% — markets with different granularity, different expiry dates, or structured differently — requires manual mapping or an LLM-based matching step.

**Estimated build effort**: 2-4 weeks for robust cross-venue normalization. It's ongoing work as new markets launch daily.

---

## Layer 3: Context Enrichment

Raw prices exist in a vacuum. Context enrichment connects prediction market data to the broader information landscape.

### Data Sources for Enrichment

- **X/Twitter**: real-time sentiment, breaking news, expert commentary. A tweet from a Fed governor moves rate markets within minutes.
- **News feeds**: Reuters, AP, Bloomberg — for verified factual updates that drive event outcomes.
- **Traditional markets**: S&P 500, oil, gold, Treasury yields — correlated assets that provide context for prediction market prices.
- **Government data**: FOMC minutes, BLS reports, executive orders — primary sources for policy-driven markets.
- **On-chain data**: for crypto-related prediction markets, on-chain flows and whale wallet movements.

### Why This Layer Matters

A prediction market contract on "US recession by Q4 2026" trading at 37 cents tells you what the crowd thinks. But to evaluate whether 37 cents is right, you need to know:

- What do the latest employment numbers say? (Government data)
- What's the yield curve doing? (Traditional markets)
- What are Fed officials saying on X? (Social media)
- Did the tariff situation escalate or de-escalate today? (News)

Without this context, you're trading on vibes.

### How SimpleFunctions Handles It

The `query` endpoint demonstrates the full context enrichment pipeline:

```bash
curl "https://simplefunctions.dev/api/public/query?q=recession+probability+2026&mode=full"
```

This single call returns:

- **Kalshi contracts**: KXRECSSNBER-26 at 37c, plus related rate and GDP markets.
- **Polymarket markets**: "US Recession 2026" at 34c.
- **X/Twitter posts**: recent tweets about recession indicators, Fed policy, labor market.
- **Traditional markets**: S&P 500, 2Y/10Y spread, unemployment claims.
- **Synthesized answer**: an LLM-generated analysis connecting all sources.

### Building It Yourself

You'll need API access to X/Twitter (tiered pricing, $100-5000/month depending on volume), a news API (various providers), and market data feeds (free delayed, paid real-time). The integration isn't hard — it's the *relevance filtering* that's challenging. Not every tweet about "recession" is useful. Filtering signal from noise at scale requires either careful keyword engineering or an LLM classification step.

**Estimated build effort**: 2-4 weeks for basic integration. Months for robust relevance filtering.

---

## Layer 4: Causal Reasoning

This is where the stack shifts from data to judgment. Causal reasoning asks: *given all the context, what should this price be?*

### From Correlation to Causation

Traditional data analysis finds correlations: "oil prices and recession contracts move together." Causal reasoning asks *why* — and builds a model of the mechanism:

```
Thesis: "Tariff escalation leads to recession by Q4 2026"

Causal tree:
├── Tariffs increase consumer prices (0.85)
│   ├── Broad tariffs remain in effect (0.70)
│   └── Retaliation from trade partners (0.75)
├── Higher prices reduce consumer spending (0.65)
│   ├── Savings rate is already low (0.80)
│   └── No fiscal stimulus to offset (0.60)
├── Reduced spending causes GDP contraction (0.55)
│   ├── Services sector weakens (0.50)
│   └── Manufacturing already contracting (0.70)
└── GDP contraction meets NBER recession criteria (0.70)

Implied probability: ~18%
Market price: 37%
```

### What This Enables

- **Disagreement identification**: the market says 37%, your causal model says 18%. That's not just an abstract number — you can trace *which assumptions* drive the disagreement.
- **Scenario analysis**: "What if tariffs are rolled back?" Adjust the node and see how the overall probability changes.
- **Signal injection**: when new information arrives, you update the relevant node rather than re-evaluating everything from scratch.

### How SimpleFunctions Handles It

The thesis engine automates causal decomposition:

```bash
sf create "Tariff escalation leads to US recession by Q4 2026"
# → Generates causal tree with 8-15 nodes
# → Assigns initial probabilities based on available data
# → Maps nodes to relevant Kalshi/Polymarket contracts

sf what-if <thesis-id> "Tariffs are rolled back to pre-2025 levels"
# → Adjusts relevant nodes
# → Recalculates overall confidence
# → Shows impact on detected edges
```

### Building It Yourself

Causal tree decomposition requires either significant domain expertise (manually building the tree) or an LLM with careful prompt engineering and structured output. The decomposition itself is the easier part — maintaining it over time (updating probabilities as new data arrives, adding new nodes, pruning irrelevant ones) is where the operational complexity lives.

**Estimated build effort**: 1-3 months for a basic thesis engine. Ongoing iteration to improve decomposition quality.

---

## Layer 5: Edge Detection

With a causal model and live market prices, edge detection becomes mechanical: where does the market price diverge from the model's implied probability?

### How Edges Are Calculated

```
Market price (Kalshi KXRECSSNBER-26):  37 cents
Model implied probability:              18 cents
Raw edge:                               19 cents

Bid/ask spread:                         4 cents
Executable edge (after crossing):       17 cents

Confidence-weighted edge:
  = executable edge * model confidence
  = 17 * 0.72
  = 12.2 cents
```

### What Makes a Good Edge

Not all edges are tradeable:

- **Size**: is there enough liquidity to trade at this price? A 20-cent edge on a market with $500 daily volume isn't useful.
- **Duration**: how long until expiry? Edges on markets expiring tomorrow are riskier than those with months of runway.
- **Confidence**: how certain are you in your causal model? A 20-cent edge with 50% model confidence is less compelling than a 10-cent edge with 90% confidence.
- **Correlation**: are multiple edges pointing the same direction? Correlated edges amplify risk.

### How SimpleFunctions Handles It

```bash
sf edges
# → Returns ranked list of edges across all theses
# → Sorted by confidence-weighted executable edge
# → Includes liquidity, time to expiry, and correlation flags

sf context <thesis-id>
# → Shows edges specific to one thesis
# → Includes track record: how often have similar edges been profitable?
```

### Building It Yourself

If you've built layers 1-4, edge detection is mostly math. The hard part is calibration: is your model actually well-calibrated? Without a track record, you don't know whether a "19-cent edge" is real or just model error. Building a backtesting framework to validate your edge detection is a project in itself.

**Estimated build effort**: 1-2 weeks for basic edge calculation. Months for calibration and validation.

---

## Layer 6: Execution

The final layer: turning a detected edge into an actual position.

### What Execution Involves

- **Order placement**: market or limit orders on the relevant exchange.
- **Position sizing**: Kelly criterion or fixed-fraction sizing based on edge magnitude and confidence.
- **Risk management**: maximum position size, correlation limits, stop-loss rules.
- **Monitoring**: tracking fills, unrealized P&L, and changes to the underlying edge.
- **Settlement tracking**: prediction markets settle at 0 or 100. Tracking outcomes and updating your track record.

### How SimpleFunctions Handles It

SimpleFunctions connects to your Kalshi and Polymarket accounts for execution:

```bash
sf setup                    # connect exchange credentials
sf get_positions            # current positions across venues
sf get_balance              # available buying power
# Edge detection → thesis engine identifies opportunities
# Execution happens through the connected exchange APIs
```

The system surfaces edges; you decide whether to act. Execution through the CLI or MCP interface keeps you in control.

### Building It Yourself

Execution against exchange APIs is well-documented. The challenge is building the operational infrastructure: position tracking across venues, P&L calculation, settlement reconciliation, and risk monitoring. This is standard trading system engineering, but it's non-trivial.

**Estimated build effort**: 2-4 weeks for basic order placement. Months for a full position management and risk system.

---

## The Full Stack: Build vs. Buy

| Layer | Build Yourself | SimpleFunctions | Oddpool |
|---|---|---|---|
| 1. Raw Exchange Data | 1-3 days | Included | Included |
| 2. Normalized Data | 2-4 weeks | Included | Included |
| 3. Context Enrichment | 2-4 weeks | Included (X, news, traditional) | Not included |
| 4. Causal Reasoning | 1-3 months | Included (thesis engine) | Not included |
| 5. Edge Detection | 1-2 weeks + calibration | Included | Arbitrage only |
| 6. Execution | 2-4 weeks | Included | Not included |
| **Total** | **3-7 months** | **1 minute setup** | **30 min setup (layers 1-2 only)** |

---

## Where Each Tool Fits

**Oddpool** is the best choice for layers 1-2 if you're building your own reasoning and execution on top. Its cross-venue normalization, whale tracking, and arbitrage detection are best-in-class for the pure data layer.

**SimpleFunctions** covers all six layers. If you want the full stack — from raw data to executed trade, with AI-powered reasoning in between — it's the most complete option available. The thesis engine (layers 4-5) is its unique differentiator.

**Raw exchange APIs** are necessary when you need capabilities that no intermediary provides: FIX protocol for low-latency trading, custom historical datasets, or institutional compliance requirements.

---

## Practical Architecture

Here's how a production prediction market stack might look:

```
                        Your Application / Agent
                               │
                    ┌──────────┼──────────┐
                    │          │          │
              SimpleFunctions  │      Oddpool
              (layers 1-6)    │    (layers 1-2)
              thesis engine   │    whale alerts
              edge detection  │    arb scanning
              context flow    │
                    │          │          │
                    └──────────┼──────────┘
                               │
                    ┌──────────┼──────────┐
                    │                     │
                 Kalshi API          Polymarket API
                 (execution)         (execution)
```

SimpleFunctions handles the intelligence loop. Oddpool provides supplementary real-time data feeds. Direct exchange APIs handle execution when you need low-level control.

---

## Conclusion

The prediction market data stack is deeper than most participants realize. Getting a price is layer 1. Knowing what to do with that price requires layers 2 through 6.

You can build each layer yourself — and for some use cases (institutional trading, custom research), that's the right choice. But for most traders, analysts, and AI agents, the layers above raw data are where the value lives. That's where prices become context, context becomes reasoning, and reasoning becomes edge.

SimpleFunctions was built to handle that full transformation. From raw exchange data to an actionable edge, in one call. Not because the individual layers are impossibly hard, but because connecting them — and keeping them connected, 24 hours a day, across venues, across data sources — is where the real complexity lives.

The tools exist. The data is accessible. The question is how much of the stack you want to own versus how much you want to focus on the part that actually matters: your thesis about the future.