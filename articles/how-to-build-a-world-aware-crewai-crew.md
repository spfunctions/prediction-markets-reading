# How to Build a World-Aware CrewAI Crew

> A CrewAI crew where every agent shares the same real-time world context. The research analyst, the risk officer, and the writer all see the same probabilities — no contradictions.

**Category:** guides | **Author:** SimpleFunctions | **Reading time:** 8 min | **Published:** 2026-04-02

---
# How to Build a World-Aware CrewAI Crew

Multi-agent systems have a consistency problem. Your research agent says "recession probability is low." Your risk agent says "recession probability is elevated." They're not disagreeing — they just have different training data cutoffs and different hallucinated numbers.

The fix is obvious once you see it: give all agents the same source of truth.

## The Problem with Multi-Agent World Knowledge

In a CrewAI crew, each agent is a separate LLM call. Each call has:
- Its own knowledge cutoff
- Its own tendency to hallucinate numbers
- Its own interpretation of "recent events"

Result: your research analyst says "oil is around $80" while your risk officer says "oil prices have been volatile around $120." Both are hallucinating. Neither knows the current price.

## The Fix: One World Model, All Agents

```python
import requests
from crewai import Agent, Task, Crew

# One API call. Every agent gets the same reality.
world = requests.get("https://simplefunctions.dev/api/agent/world").text

SHARED_CONTEXT = f"""
You have access to real-time world data. Use the numbers below as ground truth.
Do NOT hallucinate probabilities — cite these figures directly.

{world}
"""
```

Now build your crew with this shared context:

```python
researcher = Agent(
    role="Macro Research Analyst",
    goal="Analyze geopolitical and economic developments using real-time probability data",
    backstory=SHARED_CONTEXT + "\nYou are a senior macro analyst. You cite specific probabilities and prices. You never say 'approximately' when you have exact numbers.",
    verbose=True,
)

risk_officer = Agent(
    role="Risk Officer",
    goal="Identify portfolio risks based on current world conditions",
    backstory=SHARED_CONTEXT + "\nYou are a risk officer. You flag when probabilities cross thresholds that matter for portfolio exposure. You reference specific contract prices.",
    verbose=True,
)

writer = Agent(
    role="Briefing Writer",
    goal="Produce a concise morning briefing for the investment committee",
    backstory=SHARED_CONTEXT + "\nYou write crisp, data-driven briefings. Every claim has a number attached. No hedging language.",
    verbose=True,
)
```

All three agents see the same Iran invasion probability, the same recession odds, the same oil price. When the risk officer flags "geopolitical risk at 85/100," the writer cites the same number. No contradictions.

## Adding Depth Tools

For agents that need to drill deeper:

```python
from crewai.tools import tool

@tool("get_focused_world_state")
def get_focused_world_state(topics: str) -> str:
    """Get world state focused on specific topics.
    Same token budget, deeper coverage per topic.
    topics: comma-separated, e.g. 'geopolitics,energy'"""
    return requests.get(
        f"https://simplefunctions.dev/api/agent/world?focus={topics}"
    ).text

@tool("get_market_detail")
def get_market_detail(ticker: str) -> str:
    """Get detailed data for a specific prediction market contract.
    Includes price, spread, volume, orderbook depth."""
    return requests.get(
        f"https://simplefunctions.dev/api/public/market/{ticker}?depth=true"
    ).text

@tool("search_markets")
def search_markets(query: str) -> str:
    """Search prediction markets across Kalshi and Polymarket."""
    resp = requests.get(
        f"https://simplefunctions.dev/api/public/scan?q={query}"
    )
    return resp.text
```

## The Full Crew

```python
research_task = Task(
    description="Analyze the current geopolitical situation and its implications for energy markets. Use the world state data — cite specific probabilities and prices.",
    expected_output="A structured analysis with specific probability citations",
    agent=researcher,
    tools=[get_focused_world_state, search_markets],
)

risk_task = Task(
    description="Based on the research analysis, identify the top 3 portfolio risks right now. For each risk, cite the specific prediction market probability.",
    expected_output="3 risks with probability levels and recommended actions",
    agent=risk_officer,
    tools=[get_focused_world_state, get_market_detail],
)

briefing_task = Task(
    description="Write a 200-word morning briefing summarizing the analysis and risks. Every sentence must contain a number.",
    expected_output="A concise morning briefing with data citations",
    agent=writer,
    context=[research_task, risk_task],
)

crew = Crew(
    agents=[researcher, risk_officer, writer],
    tasks=[research_task, risk_task, briefing_task],
    verbose=True,
)

result = crew.kickoff()
print(result)
```

## What the Output Looks Like

Instead of:

> *"Geopolitical tensions remain elevated. Oil prices could be affected by Middle East developments. We recommend monitoring the situation."*

You get:

> *"Iran invasion probability stands at 53% (+5c in 24h, $225K volume). Hormuz transit disruption at 95%. Oil at $127/barrel (+3.2%). With geopolitical risk at 85/100 and energy sector contracts showing divergence, the primary portfolio risk is unhedged energy exposure. Recession probability at 33% remains stable."*

Every number is real. Every number comes from someone with money on the line.

## Keeping Long-Running Crews Current

For crews that run continuously:

```python
@tool("get_world_changes")
def get_world_changes(since: str = "1h") -> str:
    """Get only what changed in the world since the given time.
    Use '1h', '6h', or '24h'. Returns ~30-50 tokens, not 800."""
    return requests.get(
        f"https://simplefunctions.dev/api/agent/world/delta?since={since}"
    ).text
```

The delta endpoint returns only changes — index shifts, market moves, new alerts. ~30-50 tokens instead of 800. Your crew stays current without burning context window on repeated full state reads.

## Source

Data: [SimpleFunctions World Model](https://simplefunctions.dev/world) — 9,706 prediction markets from Kalshi (CFTC-regulated) and Polymarket, calibrated by real money. Updated every 15 minutes.