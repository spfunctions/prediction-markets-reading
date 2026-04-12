# How to Build a World-Aware LangChain Agent

> Your LangChain agent has no idea what happened today. Here is how to give it real-time world awareness with one API call — no news scraping, no search, no extra tokens wasted.

**Category:** guides | **Author:** SimpleFunctions | **Reading time:** 7 min | **Published:** 2026-04-02

---
# How to Build a World-Aware LangChain Agent

Your LangChain agent has a knowledge cutoff. Ask it "what's the probability of a US recession?" and it will either hallucinate a number, hedge with "I don't have access to real-time data," or give you a stale figure from its training data.

This is a solvable problem.

## The Problem

Every agent framework gives you tools for *doing things* — searching, browsing, calling APIs. But none of them solve the more basic problem: your agent doesn't know what day it is, what oil costs, or whether Iran just invaded somewhere.

Web search is the usual fix. But web search returns narratives, not data:

> "According to recent reports, tensions in the Middle East remain elevated as diplomatic efforts continue..."

Your agent now has vibes, not numbers. It can't reason precisely over "elevated tensions."

## The Solution: A Calibrated World Model

What if your agent started every conversation knowing:

- Iran invasion probability: 53% (+5c in 24h)
- US recession in 2026: 33%
- Fed rate cuts this year: 0 (98% probability)
- Oil: $127/barrel (+3.2%)
- Geopolitical Risk Index: 85/100

These aren't opinions. They're prices from prediction markets — 9,706 contracts where people with real money at stake are voting on what will happen. Get it wrong, lose money. That punishment mechanism produces calibrated probabilities.

## Implementation: 3 Lines

```python
import requests
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage

# Fetch the world model — ~800 tokens, updated every 15 min
world = requests.get("https://simplefunctions.dev/api/agent/world").text

llm = ChatOpenAI(model="gpt-4o")
response = llm.invoke([
    SystemMessage(content=f"You are a research analyst.\n\n{world}"),
    HumanMessage(content="Should I be worried about oil prices?"),
])
print(response.content)
```

No API key needed. Free. The response is ~800 tokens of markdown covering geopolitics, economy, energy, elections, crypto, and tech — with anchor contracts (recession, Fed, Iran invasion) that always appear regardless of daily noise.

## Making It a Tool

If you want the agent to refresh its world state mid-conversation:

```python
from langchain_core.tools import tool

@tool
def get_world_state(focus: str = "") -> str:
    """Get the current world state. Optionally focus on specific topics:
    geopolitics, economy, energy, elections, crypto, tech.
    Comma-separated for multiple topics."""
    url = "https://simplefunctions.dev/api/agent/world"
    if focus:
        url += f"?focus={focus}"
    return requests.get(url).text

@tool
def get_world_delta(since: str = "1h") -> str:
    """Get only what changed in the world since the given time.
    Much smaller than full state — use for periodic refresh.
    Examples: since='1h', since='6h', since='24h'."""
    return requests.get(
        f"https://simplefunctions.dev/api/agent/world/delta?since={since}"
    ).text
```

Bind these to your agent:

```python
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(
    llm,
    tools=[get_world_state, get_world_delta],
    prompt=f"You are a macro research analyst.\n\n{world}",
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "What's the Iran situation and how does it affect oil?"}]
})
```

The agent starts with the full world state in its system prompt. If the user asks about something specific, it can call `get_world_state(focus="geopolitics,energy")` for a deeper view — same token budget, concentrated on fewer topics.

For long-running agents, `get_world_delta(since="1h")` returns only what changed — typically 30-50 tokens instead of 800. Your agent stays current without re-reading the entire world state.

## Why This Is Better Than Web Search

| | Web Search | News API | World Model |
|---|---|---|---|
| **Output** | Narrative text | Raw JSON | Calibrated probabilities |
| **Signal quality** | Mixed, needs parsing | Headlines, no judgment | Numbers with money behind them |
| **Token cost** | 2,000-5,000 per search | 500-1,000 per call | ~800 for everything |
| **Latency** | 2-5 seconds | 500ms | 200ms (cached) |
| **Coverage** | Whatever Google ranks | Whatever the API has | 9,706 markets, 6 topics |

The key difference: a prediction market price of 53 cents on "Iran invasion" encodes the judgment of everyone with money at risk on that question. A news headline encodes what an editor thought would get clicks.

## Going Deeper

Once your agent has world awareness, you can add depth:

```python
@tool
def scan_markets(query: str) -> str:
    """Search prediction markets for contracts related to a query."""
    return requests.get(
        f"https://simplefunctions.dev/api/public/scan?q={query}"
    ).json()

@tool
def get_contagion(window: str = "6h") -> str:
    """Find cross-market anomalies: what should have moved but hasn't.
    The gap between expected and actual movement is a potential edge."""
    return requests.get(
        f"https://simplefunctions.dev/api/public/contagion?window={window}"
    ).json()
```

The world state is level 1 — knowing what's happening. Market scanning is level 2 — finding what's mispriced. Contagion detection is level 3 — finding what *should* be happening but isn't yet.

## Source

Data: [SimpleFunctions World Model](https://simplefunctions.dev/world) — 9,706 prediction markets from Kalshi (CFTC-regulated) and Polymarket, calibrated by real money. Updated every 15 minutes.