# Connecting your AI agent to prediction market data in 5 minutes

> Three integration paths — MCP, REST, CLI — each with working code you can ship today.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 9 min

---
# Connecting your AI agent to prediction market data in 5 minutes

Prediction markets are the highest-signal source of real-time probabilistic data on the internet. Polymarket, Kalshi, and Metaculus collectively price thousands of events — elections, recessions, Fed decisions, AI milestones — updated every second by traders with skin in the game.

SimpleFunctions gives your AI agent direct access to this data through three integration paths. Pick the one that fits your stack.

---

## 1. MCP — Model Context Protocol (Claude Code / Cursor)

If you're building with Claude Code or Cursor, MCP is the fastest path. One command, zero config.

### Setup

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

That's it. Your agent now has access to live prediction market tools.

### What happens under the hood

The MCP server exposes these tools to your agent:

- **`get_context`** — Returns the current macro context: top edges, trending markets, notable moves
- **`scan_markets`** — Search markets by keyword across all venues
- **`get_market_detail`** — Deep dive on a specific market: price history, orderbook, volume
- **`get_edges`** — Cross-venue arbitrage and mispricing signals

### Example: agent conversation

Here's what an actual agent interaction looks like after adding the MCP server:

**User:**
```
What are current recession odds?
```

**Agent (internal tool call):**
```json
{
  "tool": "scan_markets",
  "arguments": {
    "query": "recession",
    "limit": 5
  }
}
```

**Tool response:**
```json
{
  "markets": [
    {
      "title": "US recession in 2026?",
      "venue": "polymarket",
      "probability": 0.38,
      "volume_24h": 245000,
      "last_updated": "2026-03-28T14:22:00Z"
    },
    {
      "title": "Will NBER declare a recession starting in 2026?",
      "venue": "kalshi",
      "probability": 0.31,
      "volume_24h": 89000,
      "last_updated": "2026-03-28T14:20:00Z"
    },
    {
      "title": "US GDP negative in Q2 2026?",
      "venue": "kalshi",
      "probability": 0.22,
      "volume_24h": 34000,
      "last_updated": "2026-03-28T14:18:00Z"
    }
  ]
}
```

**Agent response:**
> Recession odds are split across venues. Polymarket prices a 2026 US recession at **38%**. Kalshi's NBER-specific contract is lower at **31%** — the gap likely reflects different resolution criteria. GDP-specific contracts (Q2 negative growth) sit at **22%**. Volume is concentrated on Polymarket ($245k/24h), suggesting that's where price discovery is happening.

The agent didn't just fetch a number. It compared venues, identified the spread, and explained why the prices diverge. That's what happens when your agent has structured market data instead of scraped web pages.

### Cursor setup

For Cursor, add to your `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "simplefunctions": {
      "url": "https://simplefunctions.dev/api/mcp/mcp"
    }
  }
}
```

Restart Cursor. The tools appear in the agent's tool palette automatically.

---

## 2. REST API — Python and JavaScript

For programmatic access — trading bots, dashboards, data pipelines — use the REST API directly.

### Context endpoint

The `/api/public/context` endpoint returns the current macro snapshot: top edges, trending markets, and notable price moves. This is the single best endpoint to call if you want to orient your agent quickly.

**Python:**

```python
import requests

ctx = requests.get("https://simplefunctions.dev/api/public/context").json()

# Top edges (cross-venue mispricings)
for edge in ctx.get("edges", []):
    print(f"{edge['title']}: {edge['edge']}% edge — "
          f"{edge['venue_a']} {edge['price_a']}¢ vs "
          f"{edge['venue_b']} {edge['price_b']}¢")

# Notable moves
for move in ctx.get("moves", []):
    print(f"{move['title']}: {move['delta']:+.1f}% in {move['period']}")
```

**JavaScript / TypeScript:**

```javascript
const ctx = await fetch("https://simplefunctions.dev/api/public/context")
  .then(r => r.json());

// Top edges
for (const edge of ctx.edges ?? []) {
  console.log(
    `${edge.title}: ${edge.edge}% edge — ` +
    `${edge.venue_a} ${edge.price_a}¢ vs ${edge.venue_b} ${edge.price_b}¢`
  );
}

// Notable moves
for (const move of ctx.moves ?? []) {
  console.log(`${move.title}: ${move.delta > 0 ? '+' : ''}${move.delta}% in ${move.period}`);
}
```

### Scan endpoint

Search markets by keyword:

```python
import requests

results = requests.get(
    "https://simplefunctions.dev/api/public/scan",
    params={"q": "fed rate cut", "limit": 10}
).json()

for market in results["markets"]:
    print(f"[{market['venue']}] {market['title']}: {market['probability']*100:.0f}%")
```

### Response shape

The context endpoint returns this structure:

```json
{
  "edges": [
    {
      "title": "US recession in 2026",
      "edge": 7.2,
      "venue_a": "polymarket",
      "price_a": 38,
      "venue_b": "kalshi",
      "price_b": 31,
      "category": "economics"
    }
  ],
  "moves": [
    {
      "title": "Fed cuts in June 2026",
      "delta": -12.5,
      "period": "24h",
      "venue": "kalshi",
      "current_price": 44
    }
  ],
  "trending": [
    {
      "title": "AI model passes ARC-AGI-2",
      "venue": "metaculus",
      "probability": 0.15,
      "volume_24h": 0,
      "comment_count": 342
    }
  ],
  "generated_at": "2026-03-28T15:00:00Z"
}
```

### Building an agent with the REST API

Here's a minimal Python agent that uses the context endpoint to answer questions:

```python
import requests
import json

def get_market_context():
    """Fetch current prediction market context."""
    return requests.get("https://simplefunctions.dev/api/public/context").json()

def search_markets(query: str, limit: int = 5):
    """Search prediction markets by keyword."""
    return requests.get(
        "https://simplefunctions.dev/api/public/scan",
        params={"q": query, "limit": limit}
    ).json()

# These functions map directly to tool definitions
# for any LLM framework: LangChain, CrewAI, OpenAI function calling, etc.
TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "get_market_context",
            "description": "Get current prediction market macro context: edges, moves, trending",
            "parameters": {"type": "object", "properties": {}}
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_markets",
            "description": "Search prediction markets by keyword",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string"},
                    "limit": {"type": "integer", "default": 5}
                },
                "required": ["query"]
            }
        }
    }
]
```

Drop those tool definitions into any framework. The functions work as-is.

---

## 3. CLI — Terminal workflows and shell scripts

The `sf` CLI is built for terminal-native workflows: piping into `jq`, cron jobs, quick lookups.

### Install

```bash
npm install -g simplefunctions-cli
```

### Get context

```bash
sf context --json | jq '.edges[] | select(.edge > 20)'
```

This filters to only high-conviction edges (>20% cross-venue mispricing). Output:

```json
{
  "title": "Trump wins 2028 GOP nomination",
  "edge": 24.5,
  "venue_a": "polymarket",
  "price_a": 72,
  "venue_b": "kalshi",
  "price_b": 48
}
```

### Scan markets

```bash
sf scan "recession" --json
```

Output:

```json
[
  {
    "title": "US recession in 2026?",
    "venue": "polymarket",
    "probability": 0.38,
    "volume_24h": 245000
  },
  {
    "title": "Will NBER declare a recession starting in 2026?",
    "venue": "kalshi",
    "probability": 0.31,
    "volume_24h": 89000
  }
]
```

### Compose with other tools

The CLI shines when piped:

```bash
# Alert on large moves
sf context --json | jq '.moves[] | select(.delta > 10 or .delta < -10)'

# Feed into an LLM
sf context --json | llm -s "Summarize the current prediction market landscape"

# Cron job: log edges every hour
*/60 * * * * sf context --json >> /var/log/market-edges.jsonl
```

---

## 4. System prompt snippet — any agent framework

If you're building a custom agent (LangChain, CrewAI, AutoGen, raw API calls), paste this into your system prompt to give it prediction market awareness:

```text
## Prediction Market Tools

You have access to real-time prediction market data via SimpleFunctions.

Available endpoints:
- GET https://simplefunctions.dev/api/public/context
  Returns: { edges, moves, trending, generated_at }
  Use for: macro overview, cross-venue edges, notable price movements

- GET https://simplefunctions.dev/api/public/scan?q={query}&limit={n}
  Returns: { markets: [{ title, venue, probability, volume_24h }] }
  Use for: searching specific topics

When answering questions about probabilities, forecasts, or event likelihood:
1. Call the relevant endpoint first
2. Compare prices across venues when available
3. Note volume as a signal of price reliability
4. Flag large recent moves (>5% in 24h) as context
5. Always cite the venue and current price

Do NOT hallucinate probabilities. If you cannot find market data, say so.
```

This snippet works with any model (GPT-4, Claude, Gemini, Llama) and any framework. The key instruction is the last line: it prevents the agent from making up numbers when it should be fetching them.

### Why this matters

Without this system prompt, an agent asked "what are recession odds?" will either hallucinate a number or say it doesn't know. With it, the agent will call the scan endpoint, get real prices, and cite its sources. The difference is the gap between a chatbot and an analyst.

---

## Choosing your integration path

| Method | Best for | Setup time | Auth required |
|--------|----------|------------|---------------|
| **MCP** | Claude Code, Cursor, MCP-compatible agents | 30 seconds | No |
| **REST** | Custom apps, bots, dashboards, pipelines | 2 minutes | No |
| **CLI** | Terminal workflows, shell scripts, cron jobs | 1 minute | No |

All three hit the same underlying data. No API key required for public endpoints. Rate limits are generous (100 req/min for public tier).

---

## What you can build

Once your agent has market data, the possibilities compound:

- **Research agents** that ground every claim in market-implied probabilities
- **Trading bots** that monitor edges and execute when spreads widen
- **News analysis** that enriches headlines with real-time odds ("Fed to cut rates — markets price this at 44%, down 12.5% in 24h")
- **Risk dashboards** that track tail events across all venues
- **Slack bots** that alert on large market moves

The common thread: your agent stops guessing and starts citing.

---

## Next steps

Full API reference at [simplefunctions.dev/docs/guide](https://simplefunctions.dev/docs/guide).