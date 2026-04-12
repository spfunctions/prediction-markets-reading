# Kalshi API: From Data to Decisions (Not Just Another Wrapper)

> Every Kalshi API article stops at "here's how to call the endpoint." This one starts there.

**Category:** technical | **Author:** Patrick Liu | **Reading time:** 4 min

---
There are plenty of articles on how to authenticate with the Kalshi API, fetch market data, and place orders. This isn't one of them.

This article is about what happens *after* you get the data. Because data isn't a signal. A price feed isn't a trading strategy. And a wrapper library isn't alpha.

## The Data-to-Decision Gap

Here's what most Kalshi API integrations look like:

```python
# Step 1: Get data
markets = kalshi.get_markets(series="KXWTIMAX")
for m in markets:
    print(f"{m.ticker}: {m.yes_price}¢")

# Step 2: ???
# Step 3: Profit
```

Step 2 is where everyone gets stuck. You have prices. Now what?

- Which contracts are mispriced?
- Mispriced relative to *what*?
- How do you know your "relative to what" is right?
- When does your edge expire?
- How do you update when reality changes?

These aren't API questions. They're intelligence questions. And they require a layer between raw data and your trading decisions.

## Price Is Not Signal

A contract trading at 35¢ tells you one thing: the market's consensus probability is 35%. That's a fact about the market, not about reality. To turn that into a signal, you need:

1. **A model of reality** — what *you* think the probability is, and why
2. **A comparison** — where your model disagrees with the market
3. **Context** — orderbook depth, spread, liquidity, recent volume
4. **Time dynamics** — is the edge growing or shrinking? Is the event approaching?

This is the chain: **data → model → comparison → context → decision**.

## The SimpleFunctions Approach

SimpleFunctions sits in the gap between data and decision. The pipeline:

```
Thesis → Causal Tree → Market Scan → Edge Detection → Orderbook Enrichment → Signal
```

### 1. Scan: Find relevant contracts

```bash
$ sf scan "oil prices" --limit 10

KXWTIMAX — WTI Crude Oil Maximum Price
  KXWTIMAX-26DEC31
  ├── WTI $120 YES    72¢    vol 2.1M
  ├── WTI $130 YES    63¢    vol 1.8M
  ├── WTI $135 YES    56¢    vol 890K
  ├── WTI $140 YES    50¢    vol 1.2M
  ├── WTI $150 YES    38¢    vol 654K
  └── WTI $160 YES    29¢    vol 312K
```

### 2. Formation: Build a causal model

Not "I think oil goes up." Instead: *why* would oil go up? What's the causal chain? Which upstream events drive which downstream outcomes?

SimpleFunctions decomposes your thesis into a tree. Each node has a probability. Nodes are causally linked — change an upstream probability and everything downstream updates.

### 3. Evaluation: Find the mispricing

```bash
$ sf edges

  Recession 2026 YES    35¢  72¢  +37  +36  1¢  high
  WTI $150 YES          38¢  75¢  +37  +36  1¢  high
  Gas $4.50 Mar YES     14¢  55¢  +41  +39  3¢  med
```

The system calculates: thesis-implied price, market price, raw edge, executable edge (accounting for spread), and liquidity grade.

### 4. Context: Orderbook depth

A 37-cent edge means nothing if the orderbook is empty. The system enriches each edge with bid/ask depth, spread, and volume to give you *executable* edges.

## For AI Agents: One-Line MCP Setup

If you're building an agent that trades prediction markets, you don't need to implement any of this yourself. Connect via MCP:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

Your agent gets 15 tools:

| Tool | Description |
|------|-------------|
| `get_context` | Full thesis snapshot — tree, edges, orderbook, evaluation |
| `scan_markets` | Search Kalshi markets by keyword (no auth needed) |
| `inject_signal` | Push news or observations into a thesis |
| `trigger_evaluation` | Run deep analysis on accumulated signals |
| `create_thesis` | Build a new thesis with causal decomposition |
| `what_if` | Scenario analysis — "what if node n2 drops to 30%?" |

The agent reads context, injects signals it discovers, and triggers evaluation when enough has accumulated. The heavy lifting — news scanning, price tracking, orderbook enrichment, LLM evaluation — runs on a 15-minute heartbeat cycle in the background.

## REST API: Same Pipeline

Everything available via MCP is also available via REST:

```bash
# Get thesis state with edges and orderbook
curl -H "Authorization: Bearer sf_live_xxx" \
  https://simplefunctions.dev/api/thesis/<id>/context

# Lightweight delta check (~50 bytes when nothing changed)
curl -H "Authorization: Bearer sf_live_xxx" \
  "https://simplefunctions.dev/api/thesis/<id>/changes?since=2026-03-15T00:00:00Z"
```

The delta endpoint is designed for agents that poll. If nothing changed since your last check, you get back 50 bytes. Efficient.

## The Point

The Kalshi API gives you data. That's necessary but not sufficient. What you need is the intelligence layer between data and decision — a system that turns prices into edges, news into signals, and hunches into structured, verifiable theses.

That's what we built.

```bash
$ npm install -g @spfunctions/cli && sf setup
```