# Build a Prediction Market Research Crew with CrewAI + SimpleFunctions

> Use CrewAI multi-agent architecture to build a prediction market research and trading team.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 6 min

---
## Multi-Agent Prediction Market Research with CrewAI

CrewAI lets you build teams of specialized agents. Here we build a 3-agent crew for prediction market intelligence:

1. **Researcher** — scans news and injects signals
2. **Analyst** — evaluates thesis and edges
3. **Trader** — recommends positions based on edges + liquidity

## Setup

```bash
pip install crewai requests
npm install -g @spfunctions/cli  # for CLI access
sf setup                          # configure API key
```

## Define SimpleFunctions Tools

```python
from crewai.tools import tool
import requests

SF_API_KEY = "sf_live_your_key_here"
SF_BASE = "https://simplefunctions.dev"
headers = {"Authorization": f"Bearer {SF_API_KEY}", "Content-Type": "application/json"}

@tool("Get thesis context")
def get_context(thesis_id: str) -> str:
    """Get thesis snapshot with confidence, causal tree, and top edges."""
    r = requests.get(f"{SF_BASE}/api/thesis/{thesis_id}/context", headers=headers)
    return str(r.json())

@tool("Inject signal")
def inject_signal(thesis_id: str, content: str) -> str:
    """Inject news or observation into thesis."""
    r = requests.post(f"{SF_BASE}/api/thesis/{thesis_id}/signal", headers=headers,
                       json={"type": "news", "content": content})
    return str(r.json())

@tool("Trigger evaluation")
def trigger_eval(thesis_id: str) -> str:
    """Force thesis re-evaluation with pending signals."""
    r = requests.post(f"{SF_BASE}/api/thesis/{thesis_id}/evaluate", headers=headers)
    return str(r.json())

@tool("Check delta")
def check_changes(thesis_id: str, since: str) -> str:
    """Check if thesis changed since timestamp. Lightweight poll."""
    r = requests.get(f"{SF_BASE}/api/thesis/{thesis_id}/changes?since={since}", headers=headers)
    return str(r.json())
```

## Define the Crew

```python
from crewai import Agent, Task, Crew

THESIS_ID = "f582bf76"  # your thesis ID

researcher = Agent(
    role="Market Researcher",
    goal="Find breaking news that could affect the thesis",
    backstory="Expert at finding market-moving information fast.",
    tools=[inject_signal],
)

analyst = Agent(
    role="Thesis Analyst",
    goal="Evaluate thesis confidence and identify tradeable edges",
    backstory="Quantitative analyst who evaluates prediction market edges.",
    tools=[get_context, trigger_eval],
)

trader = Agent(
    role="Position Manager",
    goal="Recommend which edges to trade based on liquidity and risk",
    backstory="Experienced trader who sizes positions carefully.",
    tools=[get_context],
)

# Define tasks
research_task = Task(
    description=f"Search for recent news about oil markets and Iran conflict. Inject any relevant findings as signals into thesis {THESIS_ID}.",
    agent=researcher,
    expected_output="List of signals injected",
)

analysis_task = Task(
    description=f"Get thesis {THESIS_ID} context. Trigger evaluation if there are pending signals. Report confidence, top edges, and any concerning changes.",
    agent=analyst,
    expected_output="Analysis report with confidence level and top 3 edges",
)

trading_task = Task(
    description=f"Review the analysis and recommend: which edges to enter, position size, and stop-loss levels. Consider liquidity scores.",
    agent=trader,
    expected_output="Trading recommendations with risk assessment",
)

crew = Crew(
    agents=[researcher, analyst, trader],
    tasks=[research_task, analysis_task, trading_task],
    verbose=True,
)

result = crew.kickoff()
print(result)
```

## Running as a Scheduled Job

```python
import schedule
import time

def daily_analysis():
    result = crew.kickoff()
    print(f"Daily analysis complete: {result}")

schedule.every().day.at("09:00").do(daily_analysis)

while True:
    schedule.run_pending()
    time.sleep(60)
```

## Alternative: CLI Pipe Mode

For simpler setups, pipe CLI output to your scripts:

```bash
sf context f582bf76 --json | python my_strategy.py
sf edges --json | python find_best_edge.py
sf book KXWTIMAX-26DEC31-T135 --json | python check_liquidity.py
```

## Links

- [SimpleFunctions docs](https://simplefunctions.dev/docs)
- [CrewAI docs](https://docs.crewai.com)
- [CLI GitHub](https://github.com/spfunctions/simplefunctions-cli)