# How to Build an OpenClaw Prediction Market Bot with SimpleFunctions

> A step-by-step technical guide to connecting OpenClaw agents with live prediction market data from Kalshi, Polymarket, and Databento — without writing a single scraper.

**Category:** tech | **Author:** SimpleFunctions Research | **Reading time:** 12 min | **Published:** 2026-03-26

---
# How to Build an OpenClaw Prediction Market Bot with SimpleFunctions

*A step-by-step technical guide to connecting OpenClaw agents with live prediction market data from Kalshi, Polymarket, and Databento — without writing a single scraper.*

---

## What Is OpenClaw and Why Should You Care

OpenClaw is an autonomous AI agent framework created by Peter Steinberger that has quietly become one of the most capable agent runtimes available. At its core, OpenClaw gives you a general-purpose AI agent that can be extended with skills — modular capabilities that let the agent interact with external systems, APIs, and data sources. As of March 2026, the ClawHub registry hosts over 13,700 skills covering everything from file manipulation to Kubernetes cluster management.

The architecture is straightforward. You define an agent configuration, attach skills, and the OpenClaw runtime handles tool selection, multi-step reasoning, and execution. The agent operates in a loop: observe, reason, act. Skills are the "act" part — they give the agent hands.

```yaml
# openclaw.config.yaml
agent:
  name: market-analyst
  model: claude-sonnet-4-20250514
  max_iterations: 25
  skills:
    - web_search
    - calculator
    - file_ops
```

This is fine for general tasks. But if you want your agent to do anything useful with prediction markets, you hit a wall.

## The Problem: Prediction Market Data Is a Mess

Prediction markets are fragmented. Kalshi runs regulated contracts in the US. Polymarket operates on Polygon with an order book model. PredictIt still exists for political markets. Each platform has its own API, its own data format, its own authentication scheme, and its own quirks.

If you want an OpenClaw agent to answer "What does the market think about Fed rate cuts in June?", the agent needs to:

1. Know which platforms carry that contract
2. Authenticate with each platform's API
3. Parse different response formats (Kalshi returns cents, Polymarket returns token prices between 0 and 1)
4. Normalize the data into something comparable
5. Handle edge cases like illiquid markets, expired contracts, and duplicate listings

You could write a ClawHub skill for each platform. People have tried. The results are brittle — APIs change, rate limits vary, and you end up maintaining three separate integrations that break at different times.

There's also the data depth problem. Raw contract prices tell you what the market thinks right now, but they don't tell you *why*. To build a real trading edge, you need to cross-reference market prices with underlying data — economic indicators, earnings reports, weather data, satellite imagery. That means more APIs, more authentication, more parsing.

This is the kind of problem that looks simple on the surface and turns into a maintenance nightmare six weeks in.

## SimpleFunctions as the Bridge

SimpleFunctions provides a hosted API layer that aggregates prediction market data from multiple sources and exposes it through three clean tool endpoints designed specifically for AI agent consumption. Instead of writing and maintaining platform-specific integrations, you give your OpenClaw agent three tools and let it reason about markets directly.

The three endpoints:

| Endpoint | Purpose | What It Returns |
|----------|---------|-----------------|
| `context` | Market discovery and situational awareness | Structured overview of active markets matching a query, with prices from all available platforms |
| `query` | Deep research on a specific market or question | Detailed analysis with historical data, related contracts, and underlying data from Databento |
| `thesis` | Create and track a specific market thesis | Thesis object with entry points, monitoring criteria, and edge detection |

Each endpoint returns structured JSON that an OpenClaw agent can parse and reason about without any post-processing. The API handles normalization, deduplication, and cross-platform price comparison internally.

The key design decision here is that SimpleFunctions doesn't just proxy API calls. It enriches them. When you query for "Fed rate cut June 2026", you get back Kalshi contract prices, Polymarket token prices, *and* relevant Databento economic data — Fed funds futures, Treasury yields, recent economic releases — all in a single response.

## Setting Up the Integration

### Step 1: Get Your API Key

Sign up at simplefunctions.com and grab your API key. The free tier gives you 15 million tokens, which is enough to build and test a complete trading bot before you spend anything.

```bash
export SIMPLEFUNCTIONS_API_KEY="sf_live_..."
```

### Step 2: Define the Tools in OpenClaw Format

OpenClaw skills are defined as tool specifications. Here are the three SimpleFunctions tools in OpenClaw's skill format:

```yaml
# skills/simplefunctions_context.yaml
skill:
  name: sf_market_context
  description: >
    Get prediction market context for a topic. Returns active contracts
    from Kalshi, Polymarket, and related market data. Use this for
    initial market discovery and situational awareness.
  endpoint:
    url: https://api.simplefunctions.com/v1/context
    method: POST
    headers:
      Authorization: "Bearer ${SIMPLEFUNCTIONS_API_KEY}"
      Content-Type: application/json
  parameters:
    - name: query
      type: string
      required: true
      description: Natural language query about a market or event
    - name: platforms
      type: array
      required: false
      description: Filter to specific platforms (kalshi, polymarket)
    - name: include_data
      type: boolean
      required: false
      default: true
      description: Include underlying data from Databento
```

```yaml
# skills/simplefunctions_query.yaml
skill:
  name: sf_market_query
  description: >
    Deep research query on a specific prediction market question.
    Returns detailed analysis including historical price data,
    related contracts, volume analysis, and relevant economic
    or event data. Use this after context to drill into specifics.
  endpoint:
    url: https://api.simplefunctions.com/v1/query
    method: POST
    headers:
      Authorization: "Bearer ${SIMPLEFUNCTIONS_API_KEY}"
      Content-Type: application/json
  parameters:
    - name: question
      type: string
      required: true
      description: Specific prediction market question to research
    - name: depth
      type: string
      required: false
      enum: [shallow, standard, deep]
      default: standard
      description: Analysis depth level
    - name: time_horizon
      type: string
      required: false
      description: Relevant time horizon (e.g., "2026-06-18")
```

```yaml
# skills/simplefunctions_thesis.yaml
skill:
  name: sf_market_thesis
  description: >
    Create or retrieve a market thesis. A thesis combines a directional
    view on a market with entry criteria, monitoring conditions, and
    edge detection. Use this to formalize a trading idea and track it.
  endpoint:
    url: https://api.simplefunctions.com/v1/thesis
    method: POST
    headers:
      Authorization: "Bearer ${SIMPLEFUNCTIONS_API_KEY}"
      Content-Type: application/json
  parameters:
    - name: action
      type: string
      required: true
      enum: [create, get, update, list]
      description: Thesis operation
    - name: thesis
      type: object
      required: false
      description: Thesis definition (for create/update)
    - name: thesis_id
      type: string
      required: false
      description: Thesis ID (for get/update)
```

### Step 3: Register the Skills

```bash
openclaw skills add ./skills/simplefunctions_context.yaml
openclaw skills add ./skills/simplefunctions_query.yaml
openclaw skills add ./skills/simplefunctions_thesis.yaml
```

Verify they're loaded:

```bash
openclaw skills list --filter simplefunctions
```

```
NAME                  STATUS    SOURCE
sf_market_context     active    local
sf_market_query       active    local
sf_market_thesis      active    local
```

### Step 4: Configure the Agent

Update your agent configuration to include the new skills:

```yaml
# openclaw.config.yaml
agent:
  name: market-analyst
  model: claude-sonnet-4-20250514
  max_iterations: 25
  system_prompt: |
    You are a prediction market analyst. You have access to live market
    data from Kalshi and Polymarket, plus underlying economic data from
    Databento. Your job is to identify mispriced contracts and generate
    actionable trading theses.

    When analyzing markets:
    1. Start with sf_market_context for broad discovery
    2. Use sf_market_query to drill into specific contracts
    3. Use sf_market_thesis to formalize and track ideas

    Always compare prices across platforms. A >3% spread between
    Kalshi and Polymarket on the same event is worth investigating.
  skills:
    - sf_market_context
    - sf_market_query
    - sf_market_thesis
    - calculator
    - file_ops
```

## What the Agent Sees: Structured Responses

When the agent calls `sf_market_context` with a query like "Fed rate cuts 2026", it gets back structured JSON that looks like this:

```json
{
  "query": "Fed rate cuts 2026",
  "timestamp": "2026-03-25T14:30:00Z",
  "markets": [
    {
      "event": "Federal Reserve June 2026 FOMC Decision",
      "resolution_date": "2026-06-18",
      "contracts": [
        {
          "platform": "kalshi",
          "contract_id": "FED-26JUN-T4.25",
          "title": "Fed funds rate below 4.25% after June FOMC",
          "yes_price": 0.42,
          "no_price": 0.58,
          "volume_24h": 185000,
          "open_interest": 2400000
        },
        {
          "platform": "polymarket",
          "contract_id": "0x7a3b...f291",
          "title": "Fed cuts rates in June 2026",
          "yes_price": 0.45,
          "no_price": 0.55,
          "volume_24h": 312000,
          "liquidity": 890000
        }
      ],
      "spread": {
        "yes_price_diff": 0.03,
        "direction": "polymarket_higher",
        "note": "3% spread may indicate arbitrage opportunity"
      }
    }
  ],
  "underlying_data": {
    "source": "databento",
    "fed_funds_futures": {
      "june_2026": 4.18,
      "implied_probability_cut": 0.68
    },
    "recent_releases": [
      {
        "indicator": "CPI YoY",
        "date": "2026-03-12",
        "actual": 2.8,
        "expected": 2.9,
        "prior": 3.0
      },
      {
        "indicator": "Nonfarm Payrolls",
        "date": "2026-03-07",
        "actual": 175000,
        "expected": 200000,
        "prior": 215000
      }
    ],
    "treasury_yields": {
      "2y": 3.92,
      "10y": 4.15,
      "2s10s_spread": 0.23
    }
  }
}
```

Notice what's happening here. The agent gets normalized prices from both platforms in a single call. It sees the 3-cent spread between Kalshi (0.42) and Polymarket (0.45) on essentially the same event. It also sees that Fed funds futures imply a 68% probability of a cut — meaningfully higher than what either prediction market is pricing.

That gap between the futures-implied probability and the prediction market prices is exactly the kind of edge a trading bot should investigate. The agent can now use `sf_market_query` to dig deeper.

## Security: Why Hosted Beats ClawHub for Financial Data

In January 2026, the OpenClaw security team purged 2,419 malicious skills from ClawHub. These ranged from credential-harvesting skills disguised as API wrappers to skills that exfiltrated conversation context to third-party servers. The full incident report is worth reading if you haven't — it's a case study in supply chain attacks against agent systems.

For general-purpose skills (file operations, text processing, math), ClawHub's post-purge verification system is probably fine. For anything touching financial data or trading credentials, the calculus is different.

When you install a ClawHub skill, you're running someone else's code in your agent's execution context. That code has access to everything the agent can see, including:

- Your API keys for trading platforms
- Your portfolio positions and order history
- Your trading strategy logic
- Any PII in the conversation context

A hosted API like SimpleFunctions inverts this trust model. Your agent sends a natural language query and gets structured data back. The API never sees your trading platform credentials. It never sees your strategy logic. It never runs code in your agent's context.

```
ClawHub Skill Model:
  Agent → [Skill Code Runs Locally] → External API
  Risk: Skill code has full agent context access

Hosted API Model:
  Agent → [HTTPS Request] → SimpleFunctions → [Market Data]
  Risk: Limited to data in the query
```

This isn't theoretical paranoia. If you're building a bot that will eventually place real trades, the attack surface matters. A compromised ClawHub skill could modify your agent's reasoning to place disadvantageous trades, exfiltrate your strategy to competing traders, or drain your trading account directly.

The hosted model has its own risks — you're trusting SimpleFunctions to return accurate data and not go down — but the blast radius of a compromise is categorically smaller.

## Advanced: Building a Thesis-Driven Trading Bot

The real power of this integration shows up when you move beyond ad-hoc queries and start building persistent trading theses. Here's how to architect a bot that identifies edges, tracks them, and flags opportunities.

### Creating a Thesis

A thesis is a structured market view with specific entry and exit criteria. Here's how the agent creates one:

```python
# This is what happens inside the agent's reasoning loop
# when it decides to formalize a trading idea

thesis_request = {
    "action": "create",
    "thesis": {
        "title": "June FOMC rate cut underpriced in prediction markets",
        "direction": "yes",
        "target_contracts": [
            "FED-26JUN-T4.25",  # Kalshi
            "0x7a3b...f291"      # Polymarket
        ],
        "rationale": (
            "Fed funds futures imply 68% probability of June cut. "
            "Prediction markets pricing 42-45%. CPI trending down, "
            "labor market softening. 20+ point gap suggests markets "
            "haven't fully priced in recent data."
        ),
        "entry_criteria": {
            "max_yes_price": 0.50,
            "min_spread_to_futures": 0.15,
            "min_volume_24h": 100000
        },
        "exit_criteria": {
            "take_profit": 0.65,
            "stop_loss": 0.30,
            "time_exit": "2026-06-17"
        },
        "monitoring": {
            "check_interval": "6h",
            "alert_on": [
                "price_move_gt_5pct",
                "volume_spike_gt_3x",
                "new_economic_release",
                "fomc_minutes_release"
            ]
        }
    }
}
```

The agent sends this to the `sf_market_thesis` endpoint, which returns a thesis ID and starts tracking the specified conditions.

### Monitoring Loop

Here's where you wire the thesis into an OpenClaw automation:

```yaml
# openclaw.automation.yaml
automations:
  - name: thesis-monitor
    schedule: "0 */6 * * *"  # Every 6 hours
    agent: market-analyst
    prompt: |
      Check all active theses for updates. For each thesis:
      1. Get current market prices using sf_market_context
      2. Check if any entry criteria are now met
      3. Check if any exit criteria are triggered
      4. Check if any alert conditions fired
      5. Write a brief update to ./thesis_logs/{thesis_id}/{date}.md

      If entry criteria are met and no position exists, output a
      structured trade recommendation to ./signals/{thesis_id}.json

  - name: data-watchdog
    schedule: "*/30 * * * *"  # Every 30 minutes
    agent: market-analyst
    prompt: |
      Quick scan: use sf_market_context to check for any prediction
      market contracts with >5% cross-platform spread. Log any
      findings to ./spreads/latest.json
```

### Signal Output

When the monitoring loop detects that entry criteria are met, the agent writes a structured signal:

```json
{
  "thesis_id": "th_8f3a2b",
  "signal": "entry",
  "timestamp": "2026-03-25T20:00:00Z",
  "contracts": [
    {
      "platform": "kalshi",
      "contract_id": "FED-26JUN-T4.25",
      "current_yes_price": 0.44,
      "recommended_action": "buy_yes",
      "size_suggestion": "2% of bankroll",
      "reasoning": "Below max entry of 0.50, spread to futures at 0.24"
    },
    {
      "platform": "polymarket",
      "contract_id": "0x7a3b...f291",
      "current_yes_price": 0.46,
      "recommended_action": "buy_yes",
      "size_suggestion": "1.5% of bankroll",
      "reasoning": "Slightly higher price, lower size. Liquidity adequate."
    }
  ],
  "portfolio_context": {
    "existing_exposure": "none",
    "correlation_warning": "Correlated with existing Treasury long thesis"
  }
}
```

This signal file is the handoff point. You can wire it into an execution layer — a separate script or agent that places actual orders — or use it as input for manual trading decisions. Keeping signal generation and execution separate is a deliberate architectural choice: you want to be able to audit the reasoning chain before money moves.

### Cross-Platform Arbitrage Detection

One of the more practical applications is automated spread detection. Because SimpleFunctions normalizes prices across Kalshi and Polymarket, your agent can systematically scan for arbitrage opportunities:

```python
# Agent reasoning pattern for arb detection

# 1. Get context for a broad category
context = sf_market_context(query="US politics 2026")

# 2. For each event with contracts on multiple platforms
for event in context["markets"]:
    if len(event["contracts"]) > 1:
        prices = {c["platform"]: c["yes_price"] for c in event["contracts"]}
        spread = max(prices.values()) - min(prices.values())

        if spread > 0.05:  # 5% or more
            # 3. Deep dive on this specific market
            details = sf_market_query(
                question=event["event"],
                depth="deep"
            )

            # 4. Check if spread is real or due to
            #    different resolution criteria
            if details["resolution_match"]["identical"]:
                # Real arb — create a thesis
                sf_market_thesis(
                    action="create",
                    thesis={
                        "title": f"Arb: {event['event']}",
                        "type": "arbitrage",
                        "buy_platform": min(prices, key=prices.get),
                        "sell_platform": max(prices, key=prices.get),
                        "spread": spread,
                        # ...
                    }
                )
```

The resolution match check is critical. A 10% spread between Kalshi and Polymarket on what looks like the same event might just mean they have different resolution criteria. The `sf_market_query` endpoint flags these discrepancies so your agent doesn't walk into false arbitrage.

## Cost and Scaling

The free tier gives you 15 million tokens, which covers roughly:

- 500 `context` calls (broad market scans)
- 300 `query` calls (deep dives)
- 200 `thesis` operations (creates, updates, retrievals)

For a bot running the monitoring loop described above (checking every 6 hours with periodic broad scans), you're looking at approximately 8-12 million tokens per month depending on the number of active theses and the depth of analysis.

After the free tier, pricing is $2 per million tokens. A moderately active trading bot — 10 active theses, hourly spread scans, deep research on new opportunities — runs about 30-40 million tokens per month, or $60-80. That's less than most market data terminal subscriptions and you're getting cross-platform aggregation, normalization, and underlying data included.

```
Monthly cost estimate:
  Casual (manual queries):      ~5M tokens   → Free tier
  Active monitoring bot:        ~30M tokens  → ~$60/month
  Multi-strategy, high-freq:    ~100M tokens → ~$200/month
```

Token counts scale linearly with the number of markets you're tracking and the depth of analysis. The `shallow` depth option on `sf_market_query` uses roughly 40% fewer tokens than `deep` — use it for routine checks and save deep analysis for when the agent spots something interesting.

## What This Doesn't Do

This setup gives you a research and signal generation system. It does not place trades. Execution — actually buying and selling contracts on Kalshi or Polymarket — requires platform-specific integration with proper authentication, position management, and risk controls.

That's intentional. The hardest part of building a prediction market bot isn't placing orders. It's identifying which orders to place. The research and signal generation layer is where the alpha lives, and it's where most people's homebrew solutions break down because they can't keep up with data from multiple platforms.

If you want to add execution, the signal JSON output is designed to be consumed by a separate execution agent or script. Keep the boundary clean. Your research agent should never have direct access to your trading credentials.

## Getting Started

```bash
# 1. Install OpenClaw if you haven't
brew install openclaw

# 2. Clone the starter config
git clone https://github.com/simplefunctions/openclaw-prediction-markets
cd openclaw-prediction-markets

# 3. Set your API key
export SIMPLEFUNCTIONS_API_KEY="sf_live_..."

# 4. Register the skills
openclaw skills add ./skills/*.yaml

# 5. Run your first market scan
openclaw run --prompt "Scan prediction markets for US economic events \
  in the next 30 days. Identify any contracts where the prediction \
  market price diverges more than 10% from what underlying data suggests."
```

From there, iterate. Start with manual queries to build intuition for what the agent finds useful. Add the monitoring automation once you've identified a few theses worth tracking. Add the arbitrage scanner once you understand how cross-platform spreads behave in practice.

The prediction market ecosystem is still inefficient enough that a well-configured agent with good data access can find real edges. The window won't last forever — but in March 2026, it's still wide open.