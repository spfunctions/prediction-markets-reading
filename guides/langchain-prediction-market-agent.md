# Build a Prediction Market Agent with LangChain + SimpleFunctions

> Step-by-step guide to building an autonomous prediction market agent using LangChain and SimpleFunctions API.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 7 min

---
## Build a Prediction Market Agent with LangChain

This guide shows how to build an autonomous agent that monitors prediction markets, detects edges, and suggests trades — using LangChain for orchestration and SimpleFunctions for market intelligence.

## Architecture

```
User → LangChain Agent → SimpleFunctions API → Kalshi + Polymarket
                      → Claude/GPT (reasoning)
                      → Your strategy logic
```

SimpleFunctions handles the hard parts:
- Causal thesis decomposition
- Cross-venue market scanning (Kalshi + Polymarket)
- 24/7 price and news monitoring
- Orderbook depth analysis
- Edge calculation

Your LangChain agent handles:
- User interaction
- Strategy decisions
- When to trade

## Setup

### 1. Get your API key

```bash
npm install -g @spfunctions/cli
sf setup    # interactive wizard
```

Or sign up at [simplefunctions.dev/dashboard](https://simplefunctions.dev/dashboard).

### 2. Install dependencies

```bash
pip install langchain langchain-openai requests
```

### 3. Define tools

```python
import requests
import json

SF_API_KEY = "sf_live_your_key_here"
SF_BASE = "https://simplefunctions.dev"

headers = {
    "Authorization": f"Bearer {SF_API_KEY}",
    "Content-Type": "application/json"
}

def get_context(thesis_id: str) -> dict:
    """Get thesis snapshot: causal tree, edges, confidence, latest evaluation."""
    r = requests.get(f"{SF_BASE}/api/thesis/{thesis_id}/context", headers=headers)
    return r.json()

def inject_signal(thesis_id: str, content: str, signal_type: str = "news") -> dict:
    """Inject a signal (news, observation) into the thesis for next evaluation."""
    r = requests.post(f"{SF_BASE}/api/thesis/{thesis_id}/signal", headers=headers,
                       json={"type": signal_type, "content": content})
    return r.json()

def trigger_evaluation(thesis_id: str) -> dict:
    """Force immediate re-evaluation of thesis with all pending signals."""
    r = requests.post(f"{SF_BASE}/api/thesis/{thesis_id}/evaluate", headers=headers)
    return r.json()

def list_theses() -> list:
    """List all theses."""
    r = requests.get(f"{SF_BASE}/api/thesis", headers=headers)
    return r.json().get("theses", [])

def check_health() -> bool:
    """Check if SimpleFunctions API is up."""
    r = requests.get(f"{SF_BASE}/api/health")
    return r.json().get("status") == "ok"
```

### 4. Create LangChain tools

```python
from langchain.tools import tool

@tool
def get_thesis_context(thesis_id: str) -> str:
    """Get current thesis state including confidence, edges, and latest evaluation."""
    ctx = get_context(thesis_id)
    edges = ctx.get("edges", [])[:5]
    edge_summary = "\n".join([
        f"  {e['market']} | mkt {e['marketPrice']}¢ → thesis {e['thesisPrice']}¢ | edge {e['edge']}¢"
        for e in edges
    ])
    return f"""Confidence: {ctx.get('confidence', '?')}
Top edges:
{edge_summary}
Last eval: {ctx.get('lastEvaluation', {}).get('summary', 'none')}"""

@tool
def inject_market_signal(thesis_id: str, signal: str) -> str:
    """Inject a news signal or observation into the thesis."""
    result = inject_signal(thesis_id, signal)
    return f"Signal injected: {result}"

@tool
def run_evaluation(thesis_id: str) -> str:
    """Trigger deep re-evaluation of thesis."""
    result = trigger_evaluation(thesis_id)
    return f"Evaluation result: {json.dumps(result.get('evaluation', {}), indent=2)}"
```

### 5. Build the agent

```python
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder

llm = ChatOpenAI(model="gpt-4o", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a prediction market analyst.
You monitor theses, detect edges, and suggest trades.
Use tools to get fresh data. Never guess prices."""),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

tools = [get_thesis_context, inject_market_signal, run_evaluation]
agent = create_openai_tools_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# Run
result = executor.invoke({"input": "Check my oil thesis and tell me the top edges"})
print(result["output"])
```

## The Monitoring Loop

For continuous monitoring, use the delta endpoint:

```python
import time

def monitor_loop(thesis_id: str, interval_seconds: int = 300):
    last_check = "2020-01-01T00:00:00Z"

    while True:
        r = requests.get(
            f"{SF_BASE}/api/thesis/{thesis_id}/changes?since={last_check}",
            headers=headers
        )
        data = r.json()

        if data.get("changed"):
            print(f"Confidence changed: {data['confidenceDelta']}")
            # Your agent logic here: re-evaluate, alert, trade
            ctx = get_context(thesis_id)
            # ...

        last_check = data.get("checkedAt", last_check)
        time.sleep(interval_seconds)
```

The delta endpoint returns ~50 bytes when nothing changed — efficient for polling.

## Alternative: MCP (Zero Code)

If you use Claude Code or Cursor, skip the Python entirely:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

Then just ask Claude: "Check my oil thesis and show me the top edges."

## Links

- [API docs](https://simplefunctions.dev/docs)
- [CLI reference](https://github.com/spfunctions/simplefunctions-cli)
- [Full API reference](https://simplefunctions.dev/llms.txt)