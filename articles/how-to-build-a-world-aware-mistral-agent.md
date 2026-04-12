# How to Build a World-Aware Mistral Agent

> Mistral models have a knowledge cutoff. Here is how to give them real-time world awareness with one API call and function calling — calibrated probabilities from 9,706 prediction markets.

**Category:** guides | **Author:** SimpleFunctions | **Reading time:** 6 min | **Published:** 2026-04-02

---
# How to Build a World-Aware Mistral Agent

Mistral models have a knowledge cutoff. This cookbook shows how to give them real-time world awareness — not from web search (narratives), but from calibrated prediction market data (numbers backed by money).

## Setup

```python
import os
import json
import requests
from mistralai import Mistral

client = Mistral(api_key=os.environ["MISTRAL_API_KEY"])
MODEL = "mistral-large-latest"
```

## Step 1: Inject World State

One API call. ~800 tokens. No API key needed.

```python
world = requests.get("https://simplefunctions.dev/api/agent/world").text
print(world)
```

This returns markdown covering geopolitics, economy, energy, elections, crypto, tech — with anchor contracts (recession probability, Fed rate path, Iran invasion) that always appear.

```python
response = client.chat.complete(
    model=MODEL,
    messages=[
        {"role": "system", "content": f"""You are a macro analyst with real-time world data.
Cite specific numbers. Don't hallucinate.

{world}"""},
        {"role": "user", "content": "What's the current geopolitical risk and how does it affect oil?"},
    ],
)

print(response.choices[0].message.content)
```

Instead of "tensions remain elevated," the agent says "Iran invasion at 53%, Hormuz disruption at 95%, oil at $127 (+3.2%), Geo Risk 85/100."

## Step 2: Function Calling for Depth

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_world_state",
            "description": "Get world state from prediction markets. Focus on specific topics for deeper coverage.",
            "parameters": {
                "type": "object",
                "properties": {
                    "focus": {
                        "type": "string",
                        "description": "Comma-separated: geopolitics, economy, energy, elections, crypto, tech"
                    }
                }
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "get_world_delta",
            "description": "Only what changed since a time. ~30-50 tokens vs 800.",
            "parameters": {
                "type": "object",
                "properties": {
                    "since": {"type": "string", "description": "30m, 1h, 6h, 24h"}
                },
                "required": ["since"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_prediction_markets",
            "description": "Search for specific contracts. Returns prices, volumes, spreads.",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "e.g. 'iran oil', 'fed rate cut'"}
                },
                "required": ["query"]
            }
        }
    }
]


def execute_tool(name, args):
    if name == "get_world_state":
        url = "https://simplefunctions.dev/api/agent/world"
        if args.get("focus"):
            url += f"?focus={args['focus']}"
        return requests.get(url).text
    elif name == "get_world_delta":
        return requests.get(
            f"https://simplefunctions.dev/api/agent/world/delta?since={args['since']}"
        ).text
    elif name == "search_prediction_markets":
        resp = requests.get(
            f"https://simplefunctions.dev/api/public/scan?q={args['query']}&limit=10"
        )
        return json.dumps(resp.json().get("markets", [])[:10], indent=2)
    return "Unknown tool"
```

## Step 3: Agentic Loop

```python
messages = [
    {"role": "system", "content": f"You are a macro analyst.\n\n{world}"},
    {"role": "user", "content": "Search for Iran contracts and analyze oil supply risk."},
]

while True:
    response = client.chat.complete(model=MODEL, messages=messages, tools=tools)
    msg = response.choices[0].message

    if msg.tool_calls:
        messages.append(msg)
        for tc in msg.tool_calls:
            args = json.loads(tc.function.arguments) if isinstance(
                tc.function.arguments, str
            ) else tc.function.arguments
            result = execute_tool(tc.function.name, args)
            print(f"Tool: {tc.function.name}({args})")
            messages.append({
                "role": "tool",
                "name": tc.function.name,
                "content": result,
                "tool_call_id": tc.id,
            })
    else:
        print("\n" + msg.content)
        break
```

The agent calls `search_prediction_markets("iran oil")` autonomously, gets live contract prices, and produces analysis grounded in real data.

## Token-Efficient Patterns

| Pattern | When | Tokens |
|---|---|---|
| System prompt with full state | Start of session | ~800 |
| `?focus=energy,geopolitics` | Need depth on specific topics | ~800 (concentrated) |
| `/delta?since=1h` | Periodic refresh mid-session | ~30-50 |
| `search_prediction_markets` | Need specific contract data | Varies |

## Why Not Web Search?

A prediction market price of 53c on "Iran invasion" = aggregate judgment of everyone with money at risk. A news headline = what an editor thought would get clicks.

| | Web Search | Prediction Markets |
|---|---|---|
| **Output** | Narrative | Calibrated probabilities |
| **Tokens** | 2,000-5,000 | ~800 |
| **Latency** | 2-5s | ~200ms |
| **Calibration** | None | Money at risk |

## Source

Data: [SimpleFunctions World Model](https://simplefunctions.dev/world) — 9,706 prediction markets from Kalshi (CFTC-regulated) and Polymarket, calibrated by real money. Updated every 15 minutes.