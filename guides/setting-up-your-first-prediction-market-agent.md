# Setting Up Your First Prediction Market Agent with SimpleFunctions

> From zero to a running agent in 15 minutes. MCP configuration, your first scan, your first thesis, your first edge — with real screenshots and every decision point explained.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 1 min

---
This isn't API documentation. This is a person sitting down, opening a terminal, and having a running prediction market agent 15 minutes later.

No filler. Every step does something. Every decision point is explained.

## Prerequisites

You need three things:

1. **Node.js 18+** — `node --version` should show 18 or higher
2. **A Kalshi account** — free at kalshi.com, you need the API keys from Settings → API
3. **An OpenRouter API key** — for the LLM that powers thesis analysis

That's it. No Docker, no databases, no cloud setup.

## Step 1: Install the CLI

```bash
npm install -g @spfunctions/cli
```

Verify the installation:

```bash
sf --version
# @spfunctions/cli 1.2.x
```

The `sf` command is now available globally. Everything runs through this single binary.

## Step 2: Configure Your Keys

```bash
sf setup
```

This launches an interactive setup that asks for:

- **Kalshi API Key & Secret** — from your Kalshi dashboard under Settings → API Keys
- **OpenRouter API Key** — from openrouter.ai/keys

Keys are stored locally in `~/.sfrc`. They never leave your machine. The CLI calls Kalshi and OpenRouter directly — there's no SimpleFunctions server in between.

## Step 3: Your First Scan

```bash
sf scan
```

This is where it gets interesting. The scan command:

1. Pulls all active markets from Kalshi and Polymarket
2. Groups them by event (e.g., all recession contracts together)
3. Runs a fast LLM pass to identify which markets have potential structural mispricing

You'll see output like:

```
Scanning 847 active markets...
Grouped into 234 events.

Found 12 candidates with potential edge:

  1. US Recession 2026      Kalshi  YES @ 35¢   Candidate: causal chain analysis suggests mispricing
  2. Fed Rate Cut Jun 2026   Kalshi  YES @ 22¢   Candidate: market lagging employment data
  3. Oil > $90 by Sep 2026   Poly    YES @ 41¢   Candidate: supply constraint underpriced
  ...
```

**Why 12 out of 847?** Because the scan isn't random filtering. It's running a quick causal plausibility check: "Is there a reason the market price might be wrong?" Most markets are efficiently priced. The scan surfaces the ones where a causal argument exists for mispricing.

## Step 4: Build a Thesis

Pick a candidate from the scan. Let's say "US Recession 2026" caught your eye. Now build a thesis:

```bash
sf thesis create
```

The agent mode kicks in. It asks you for your directional view and why. You might say:

> "I think the US-Iran conflict will persist, keeping Hormuz blocked, oil elevated, and pushing the economy toward recession."

The agent then:

1. Decomposes your view into a **causal tree** — a chain of testable sub-claims, each with a probability
2. Maps each node to verifiable real-world signals
3. Identifies which prediction market contracts correspond to which nodes
4. Calculates the **thesis-implied probability** vs **market price** for each contract

The output is a structured thesis with nodes like:

```
n1  War persists                     88%
├── n1.1  US initiated strikes       99%
├── n1.2  Iran continues retaliation 85%
└── n1.3  No diplomatic exit         82%
n2  Hormuz blocked                   97%
n3  Oil stays elevated               64%
n4  Recession                        45%

Thesis-implied: 45%  |  Market: 35%  |  Edge: +10 points
```

## Step 5: Detect Edge

The edge isn't "I think recession will happen." The edge is the gap between what your causal model implies and what the market is pricing.

```bash
sf edge --thesis iran-war
```

This compares every node in your causal tree against current market prices and shows you where the divergence lives:

| Contract | Your Thesis | Market Price | Edge |
|----------|:----------:|:----------:|:----:|
| Recession 2026 YES | 45% | 35% | +10 |
| Oil > $90 Q3 YES | 64% | 41% | +23 |
| Fed Cut Jun NO | 78% | 65% | +13 |

The key insight: **the edge is distributed across the causal chain.** The recession contract has 10 points of edge, but the oil contract has 23 points. Your thesis says oil is the bigger mispricing — and it's upstream in the causal chain, meaning it has fewer dependencies.

## Step 6: Create a Strategy

```bash
sf strategy create --thesis iran-war
```

A strategy is a set of conditions that define when to enter, when to exit, and what to size. The agent helps you define:

- **Entry conditions:** "thesis confidence > 65% AND market price < 40¢ AND edge > 8 points"
- **Exit conditions:** "thesis confidence < 40% OR edge < 3 points"
- **Position sizing:** Kelly criterion or fixed fractional (the agent walks you through this)

The strategy is stored and ready for monitoring.

## Step 7: Start the Heartbeat

```bash
sf monitor --strategy iran-recession
```

Now your agent is alive. Every 15 minutes it:

1. Fetches latest market prices
2. Searches for news related to your causal tree nodes
3. Re-evaluates node confidences based on new information
4. Checks strategy conditions
5. Sends you a notification if any condition triggers

You'll get a notification like:

```
[SIGNAL] Strategy iran-recession triggered
  Reason: CPI data exceeded expectations, n3 confidence updated 64% → 71%
  Edge: thesis 48% vs market 36% = +12 points
  Action: Consider entry per strategy conditions
```

The agent monitors. You decide whether to execute. The division of labor is clean: it watches 24/7, you make the final call.

## What You Have Now

In 15 minutes you've set up:

- A **scanner** that identifies mispriced prediction market contracts
- A **thesis engine** that structures your thinking into a testable causal model
- An **edge detector** that quantifies the gap between your model and market prices
- A **monitoring agent** that watches 24/7 and alerts you to opportunities

This isn't a trading bot. It's an intelligence system that does the hard work of watching, thinking, and quantifying — so you can focus on the decisions that matter.

## Next Steps

- Read [Position Sizing for Prediction Markets](/technicals/position-sizing-kelly-criterion-prediction-markets) to understand how to size your trades using Kelly criterion with causal model confidence
- Read [5 Patterns That Kill Prediction Market Traders](/technicals/5-patterns-that-kill-prediction-market-traders) to understand the cognitive biases the agent protects you from
- Read [Running a 24/7 Trading Agent](/technicals/running-247-trading-agent-architecture-costs) for the full operational picture