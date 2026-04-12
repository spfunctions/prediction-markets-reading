# How to Build a World-Aware Agent with the OpenAI Agents SDK

> The OpenAI Agents SDK gives you tools, handoffs, and guardrails. But your agent still doesn't know what day it is. Here is how to fix that.

**Category:** guides | **Author:** SimpleFunctions | **Reading time:** 6 min | **Published:** 2026-04-02

---
# How to Build a World-Aware Agent with the OpenAI Agents SDK

The OpenAI Agents SDK gives you a clean abstraction: agents with instructions, tools, handoffs, and guardrails. But there's a gap in the default setup: your agent's instructions are static. It doesn't know what's happening in the world right now.

## The Gap

```python
from agents import Agent

agent = Agent(
    name="analyst",
    instructions="You are a macro research analyst.",
)
```

Ask this agent "what's the current Iran situation?" and it will either hallucinate, refuse, or give you stale information from its training data.

## The Fix: Dynamic Instructions with World State

```python
import requests
from agents import Agent, Runner

def get_instructions():
    world = requests.get("https://simplefunctions.dev/api/agent/world").text
    return f"""You are a macro research analyst with real-time world awareness.

Use the data below as ground truth. Cite specific probabilities and prices.
Do not hallucinate numbers — if it's not in the data, say you don't know.

{world}"""

agent = Agent(
    name="world_aware_analyst",
    instructions=get_instructions,  # Dynamic — called before each run
)

result = Runner.run_sync(agent, "What's the probability of a US recession?")
print(result.final_output)
```

The `instructions` parameter accepts a callable. Every time the agent runs, it fetches the latest world state — ~800 tokens of calibrated data covering geopolitics, economy, energy, elections, crypto, and tech.

No API key needed. Free. ~200ms latency.

## Adding World State as a Tool

For agents that need to drill into specific topics mid-conversation:

```python
from agents import Agent, Runner, function_tool

@function_tool
def world_state(focus: str = "") -> str:
    """Get current world state. Focus on specific topics for deeper coverage.
    Options: geopolitics, economy, energy, elections, crypto, tech.
    Comma-separated for multiple topics."""
    url = "https://simplefunctions.dev/api/agent/world"
    if focus:
        url += f"?focus={focus}"
    return requests.get(url).text

@function_tool
def world_delta(since: str = "1h") -> str:
    """Get only what changed in the world since the given time.
    Use for periodic refresh — returns ~30-50 tokens, not 800.
    Options: 30m, 1h, 6h, 24h."""
    return requests.get(
        f"https://simplefunctions.dev/api/agent/world/delta?since={since}"
    ).text

@function_tool
def search_prediction_markets(query: str) -> str:
    """Search for specific prediction market contracts.
    Returns prices, volumes, and spreads from Kalshi and Polymarket."""
    import json
    resp = requests.get(
        f"https://simplefunctions.dev/api/public/scan?q={query}"
    )
    return json.dumps(resp.json(), indent=2)

agent = Agent(
    name="world_aware_analyst",
    instructions=get_instructions,
    tools=[world_state, world_delta, search_prediction_markets],
)
```

The agent starts with the full world state in its instructions. When the user asks a specific question, it can:
- Call `world_state(focus="energy,geopolitics")` for 10 contracts per topic instead of 4
- Call `world_delta(since="1h")` to check what changed recently
- Call `search_prediction_markets("iran oil")` to find specific contracts

## Multi-Agent Handoffs with Shared World Context

```python
world = requests.get("https://simplefunctions.dev/api/agent/world").text

SHARED = f"""You have real-time world awareness. Cite these numbers directly.

{world}"""

researcher = Agent(
    name="researcher",
    instructions=SHARED + "\nYou research macro topics. You hand off to the risk analyst when you find concerning data.",
    handoffs=["risk_analyst"],
    tools=[world_state, search_prediction_markets],
)

risk_analyst = Agent(
    name="risk_analyst",
    instructions=SHARED + "\nYou assess portfolio risk based on probability data. You never say 'could' when you have a specific probability.",
    tools=[world_state],
)

result = Runner.run_sync(
    researcher,
    "What should a portfolio manager know about geopolitical risk today?"
)
```

Both agents see the same probabilities. When the researcher says "Iran invasion at 53%" and hands off to the risk analyst, the risk analyst sees the same 53% in its own context. No contradictions.

## Why Not Just Use Web Search?

The OpenAI Agents SDK supports web search tools. Why not use those?

**Token efficiency.** A web search returns 2,000-5,000 tokens of narrative text that your agent has to parse. The world model returns 800 tokens of structured data — numbers with units, ready for reasoning.

**Calibration.** "According to recent reports, tensions remain elevated" is not actionable data. "Iran invasion: 53%, Hormuz disruption: 95%, Geo Risk: 85/100" is. Your agent can compare numbers, detect thresholds, and make conditional decisions.

**Consistency.** Web search results vary by query phrasing and ranking algorithm. The world model returns the same calibrated data every time, from the same source of truth.

**Speed.** 200ms (cached) vs. 2-5 seconds per web search. For agents making multiple reasoning steps, this adds up.

## Source

Data: [SimpleFunctions World Model](https://simplefunctions.dev/world) — 9,706 prediction markets from Kalshi (CFTC-regulated) and Polymarket, calibrated by real money. Updated every 15 minutes.