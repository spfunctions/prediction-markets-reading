# How to Build a World-Aware Claude Agent

> Claude has a knowledge cutoff. Here is how to give it real-time world awareness — calibrated probabilities from prediction markets, injected via system prompt, tool use, or MCP.

**Category:** guides | **Author:** SimpleFunctions | **Reading time:** 7 min | **Published:** 2026-04-02

---
# How to Build a World-Aware Claude Agent

Claude has a knowledge cutoff. Ask it "what's the probability of a US recession?" and it will hedge, give you stale data, or hallucinate a number. This is fixable.

## The Problem

Every LLM has this gap. Web search is the usual workaround, but web search returns narratives:

> "According to recent reports, tensions in the Middle East remain elevated as diplomatic efforts continue..."

Your agent now has vibes, not data. It can't reason precisely over "elevated tensions."

## Three Integration Patterns

There are three ways to give Claude world awareness, from simplest to most powerful.

### Pattern 1: System Prompt Injection (simplest)

One API call fetches ~800 tokens of calibrated world state. Inject it into the system prompt.

```python
import requests
import anthropic

client = anthropic.Anthropic()

# Fetch world state — free, no API key
world = requests.get("https://simplefunctions.dev/api/agent/world").text

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system=f"""You are a macro research analyst with real-time world awareness.

Use the data below as ground truth. Cite specific probabilities and prices.
Do not hallucinate numbers — if it's not in the data, say you don't know.

{world}""",
    messages=[
        {"role": "user", "content": "What's the probability of a US recession in 2026?"}
    ],
)

print(response.content[0].text)
```

Claude now has real numbers to cite. Instead of hedging, it says "recession probability is 33% according to prediction market data."

### Pattern 2: Tool Use (deeper)

Give Claude tools to drill into specific topics mid-conversation.

```python
tools = [
    {
        "name": "get_world_state",
        "description": "Get current world state from prediction markets. Focus on specific topics for deeper coverage.",
        "input_schema": {
            "type": "object",
            "properties": {
                "focus": {
                    "type": "string",
                    "description": "Comma-separated topics: geopolitics, economy, energy, elections, crypto, tech"
                }
            }
        }
    },
    {
        "name": "get_world_delta",
        "description": "Get only what changed since a timestamp. ~30-50 tokens vs 800 for full state.",
        "input_schema": {
            "type": "object",
            "properties": {
                "since": {"type": "string", "description": "Time window: 30m, 1h, 6h, 24h"}
            },
            "required": ["since"]
        }
    },
    {
        "name": "search_prediction_markets",
        "description": "Search for specific prediction market contracts. Returns prices, volumes, spreads.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query, e.g. 'iran oil'"}
            },
            "required": ["query"]
        }
    }
]
```

Then implement a tool-use loop:

```python
import json

def handle_tool(name, input_data):
    if name == "get_world_state":
        url = "https://simplefunctions.dev/api/agent/world"
        if input_data.get("focus"):
            url += f"?focus={input_data['focus']}"
        return requests.get(url).text
    elif name == "get_world_delta":
        return requests.get(
            f"https://simplefunctions.dev/api/agent/world/delta?since={input_data['since']}"
        ).text
    elif name == "search_prediction_markets":
        resp = requests.get(
            f"https://simplefunctions.dev/api/public/scan?q={input_data['query']}&limit=10"
        )
        return json.dumps(resp.json().get("markets", [])[:10], indent=2)

messages = [{"role": "user", "content": "Deep dive on Iran risk and its oil market impact."}]

while True:
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        system=f"You are a macro analyst.\n\n{world}",
        tools=tools,
        messages=messages,
    )

    if response.stop_reason == "tool_use":
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = handle_tool(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result,
                })
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})
    else:
        for block in response.content:
            if hasattr(block, "text"):
                print(block.text)
        break
```

Claude calls `search_prediction_markets("iran")` on its own, gets specific contract data, and produces analysis with real numbers.

### Pattern 3: MCP Server (zero code)

SimpleFunctions exposes 40 tools via MCP. For Claude Code:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

For Claude Desktop, add to your MCP config:

```json
{
  "mcpServers": {
    "simplefunctions": {
      "url": "https://simplefunctions.dev/api/mcp/mcp"
    }
  }
}
```

Now Claude has `get_world_state`, `get_world_delta`, `scan_markets`, and 37 more tools — no code, no API key for public endpoints.

## Focused Mode and Incremental Updates

Two parameters that matter for token-conscious agents:

**Focus**: `?focus=geopolitics,energy` — same ~800 token budget, concentrated on fewer topics. Instead of 4 contracts per topic across 6 topics, you get 10 contracts per topic across 2 topics. More depth, same cost.

**Delta**: `/delta?since=1h` — only what changed. Returns ~30-50 tokens instead of 800. For agents in long sessions, check delta periodically instead of re-reading the full state.

```python
# Full state at start
world = requests.get("https://simplefunctions.dev/api/agent/world").text

# Periodic refresh — only changes
delta = requests.get("https://simplefunctions.dev/api/agent/world/delta?since=1h").text

# Deep dive on one topic
energy = requests.get("https://simplefunctions.dev/api/agent/world?focus=energy").text
```

## Why Prediction Markets Instead of Web Search?

A prediction market price of 53 cents on "Iran invasion" means: someone bet $53 this happens, someone else bet $47 it doesn't. Both have money at risk. The aggregate of all such bets produces a calibrated probability.

A news headline means: an editor thought this would get clicks.

These are fundamentally different information sources. The first is a number your agent can reason over. The second is narrative it has to parse, summarize, and hope is current.

| | Web Search | Prediction Markets |
|---|---|---|
| **Output** | Narrative text | Calibrated probabilities |
| **Token cost** | 2,000-5,000 per search | ~800 for everything |
| **Latency** | 2-5 seconds | ~200ms (cached) |
| **Calibration** | None | Participants lose money when wrong |

## Source

Data: [SimpleFunctions World Model](https://simplefunctions.dev/world) — 9,706 prediction markets from Kalshi (CFTC-regulated) and Polymarket, calibrated by real money. Updated every 15 minutes.