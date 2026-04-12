# MCP Servers for Prediction Markets: Connect Claude Code to Kalshi and Polymarket

> Prediction markets are the sharpest source of real-time probability data on the internet. Here is how to wire them directly into your AI coding agent with one command.

**Category:** tech | **Author:** SimpleFunctions Research | **Reading time:** 10 min | **Published:** 2026-03-24

---
# MCP Servers for Prediction Markets: Connect Claude Code to Kalshi and Polymarket

**Prediction markets are the sharpest source of real-time probability data on the internet. Here is how to wire them directly into your AI coding agent with one command.**

---

## The problem with prediction market data

If you have ever tried to build anything on top of Kalshi or Polymarket, you know the friction. Kalshi has a REST API with OAuth2 authentication, rate limits, and pagination. Polymarket runs on Polygon, so you are dealing with subgraph queries, CLOB API endpoints, and condition ID lookups. Both require polling to stay current. Both return raw order book data that needs transformation before it is useful.

Most developers who want prediction market data in their workflow hit the same wall: they spend more time on API plumbing than on the actual analysis. And if you want data from both platforms normalized into a single view, double the work.

MCP changes this equation entirely.

## What MCP actually is

Model Context Protocol is an open standard that lets AI agents call external tools through a unified interface. Instead of teaching your agent how to make HTTP requests to specific APIs, you give it access to an MCP server that exposes a set of typed tools. The agent sees what tools are available, understands their parameters from the schema, and calls them when relevant.

Think of it as a USB port for AI agents. The agent does not need a custom driver for every service. It just needs a compliant MCP server, and the tools show up automatically.

MCP works with Claude Code, Cursor, Cline, Windsurf, and any client that implements the protocol. The server handles authentication, data normalization, caching, and all the API-specific logic. Your agent just calls tools and gets structured results.

## One-line setup with Claude Code

If you are using Claude Code, the entire setup is a single command:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

That is it. No API keys to configure, no environment variables to set, no packages to install. The next time you start a Claude Code session, it will have access to 18 prediction market tools.

To verify it worked:

```bash
claude mcp list
```

You should see `simplefunctions` in the output with a status of `connected`.

## Setup for Cursor and Cline

If you are using Cursor, add this to your MCP configuration file (`.cursor/mcp.json` in your project root, or the global config):

```json
{
  "mcpServers": {
    "simplefunctions": {
      "url": "https://simplefunctions.dev/api/mcp/mcp"
    }
  }
}
```

For Cline, the configuration goes in your Cline MCP settings:

```json
{
  "mcpServers": {
    "simplefunctions": {
      "url": "https://simplefunctions.dev/api/mcp/mcp"
    }
  }
}
```

Both editors will detect the MCP server on restart and make the tools available in your agent sessions. No API wrapper libraries, no SDK installations, no build step.

## The 18 tools

Here is every tool the SimpleFunctions MCP server exposes, with what each one does:

**Market Discovery**
- `scan_markets` — Search across Kalshi and Polymarket for markets matching a keyword or topic
- `explore_public` — Browse trending and high-volume markets without any specific query
- `get_schedule` — Pull upcoming event schedules that drive market resolution (Fed meetings, earnings dates, elections)
- `get_milestones` — Retrieve key milestone events and deadlines relevant to active markets

**Analysis**
- `create_thesis` — Build a structured investment thesis around a market position
- `get_context` — Fetch background data, news, and historical context for a specific market or topic
- `what_if` — Run scenario analysis on a position (what happens to your edge if conditions change)
- `find_edge` — Identify statistical edges by comparing market prices to model-implied probabilities
- `get_correlation` — Find correlated markets and events that move together

**Portfolio**
- `check_portfolio` — View your current positions across platforms
- `get_pnl` — Pull profit and loss data for your positions
- `risk_check` — Analyze portfolio risk exposure and concentration

**Execution**
- `place_order` — Submit an order to Kalshi or Polymarket
- `cancel_order` — Cancel a pending order
- `get_order_status` — Check the status of a submitted order

**System**
- `heartbeat` — Verify the connection is live and data is fresh
- `get_config` — Retrieve your current configuration and connected accounts
- `help` — Get usage information and examples for any tool

Four of these tools work without any authentication at all: `scan_markets`, `explore_public`, `get_milestones`, and `get_schedule`. You can start pulling live prediction market data into your agent sessions right now, before deciding whether to connect trading accounts.

## Walkthrough: scanning markets

Open Claude Code in any project directory and type a natural language request:

```
scan oil markets
```

Claude Code sees that the `scan_markets` tool is available, recognizes that your request maps to a market search, and calls it automatically. No prompt engineering required. The tool call looks like this under the hood:

```
Tool: scan_markets
Parameters: { "query": "oil" }
```

The response comes back with live market data from both Kalshi and Polymarket:

```
Found 12 markets matching "oil":

Kalshi:
  - "WTI Crude Oil above $80 on March 28" — Yes: 34¢ / No: 66¢
  - "WTI Crude Oil above $75 on March 28" — Yes: 58¢ / No: 42¢
  - "WTI Crude Oil above $70 on March 28" — Yes: 81¢ / No: 19¢
  ...

Polymarket:
  - "Oil price above $80 by end of Q1 2026" — Yes: 28¢
  - "OPEC+ production cut in April 2026" — Yes: 42¢
  ...
```

Each result includes the current price (which is the market-implied probability expressed in cents), volume, and the resolution source. You are looking at a cross-platform snapshot of how the market prices oil-related outcomes, pulled in under two seconds.

You can refine from here. Ask "show me oil markets expiring this week" or "which oil markets have the highest volume" and the agent will either filter the existing results or make additional tool calls as needed.

## Walkthrough: building a thesis

This is where the tools start compounding. Say you have a view on Federal Reserve rate cuts and want to pressure-test it against market data.

```
create a thesis about Fed rate cuts in Q2 2026
```

Claude Code calls `create_thesis` with your topic. The tool does not just search for markets — it builds a structured thesis object that includes:

- **Relevant markets** across Kalshi and Polymarket with current prices
- **Your implied position** based on the thesis framing
- **Key dates** (FOMC meetings, CPI releases, jobs reports) that will move these markets
- **Historical context** on how similar setups have resolved

Now layer on `get_context`:

```
get context for this thesis
```

The agent calls `get_context`, which pulls in recent news, economic data, and commentary relevant to Fed policy. This is not a web search — it is curated context specifically oriented around factors that prediction markets price into rate decisions.

Finally, ask for edges:

```
where are the edges?
```

The agent calls `find_edge`, which compares market-implied probabilities against historical base rates and model estimates. You might see output like:

```
Potential edges identified:

  Market: "Fed cuts rates at June 2026 meeting"
  Current price: 38¢ (38% implied probability)
  Model estimate: 52% based on current dot plot + inflation trajectory
  Edge: +14 points

  Market: "Two or more rate cuts by end of Q2 2026"
  Current price: 22¢ (22% implied probability)
  Model estimate: 18%
  Edge: -4 points (market is slightly rich)
```

Three tool calls. No API code. No data normalization. You went from a vague thesis to a quantified view of where the market might be mispricing outcomes.

## Walkthrough: scenario analysis

The `what_if` tool is particularly useful because it lets you stress-test positions without placing any trades. And it runs with zero LLM cost — the computation happens entirely on the MCP server backend.

```
what if oil drops to $70 by end of March?
```

The agent calls `what_if` with the scenario parameters:

```
Tool: what_if
Parameters: { "scenario": "WTI crude oil drops to $70 by end of March 2026" }
```

The response maps out how this scenario would cascade across related markets:

```
Scenario: WTI crude oil at $70 by March 31, 2026

Direct impact:
  - "WTI above $75 on March 28" — current 58¢ → likely resolution: No
  - "WTI above $70 on March 28" — current 81¢ → likely resolution: At-the-money

Correlated market impact:
  - "US gasoline average below $3.00 in April" — current 35¢ → model moves to ~55¢
  - "Energy sector ETF (XLE) down 5%+ in March" — current 18¢ → model moves to ~40¢
  - "Fed cuts in June" — current 38¢ → slight upward pressure (lower oil = lower inflation expectations)

Portfolio impact (if authenticated):
  - Your long position on "Oil above $75" would face a 58¢ loss per contract
  - Your short position on "Gasoline below $3" would gain ~20¢ per contract
  - Net portfolio impact: -$38 estimated
```

This is scenario analysis that would normally require building a correlation model, pulling data from multiple APIs, and writing custom simulation logic. Here it is a single natural language request.

## Fresh data via backend heartbeat

A common concern with any market data integration: is the data stale? Prediction markets move fast, especially around news events. A 15-minute delay on a market that is repricing a breaking headline is worse than useless.

The SimpleFunctions MCP server runs a backend heartbeat system. The server continuously maintains connections to both Kalshi and Polymarket, keeping market data current. When your agent calls a tool, it is not making a cold API request that might hit a cache from 10 minutes ago. It is reading from a live data layer that is already warm.

You can verify this yourself:

```
check heartbeat
```

The agent calls the `heartbeat` tool, which returns the server status and data freshness timestamps:

```
Server status: connected
Kalshi data age: 4 seconds
Polymarket data age: 7 seconds
Last refresh: 2026-03-23T14:22:08Z
```

This matters most for the analysis tools. When `find_edge` compares market prices to model estimates, those market prices are current. When `what_if` maps scenario cascades, it is working with live order book data, not yesterday's closing prices.

## Why not build your own integration?

You absolutely can. Both Kalshi and Polymarket have public APIs. Here is what that looks like.

**For Kalshi alone**, you need to:

1. Register for API access and get OAuth2 credentials
2. Implement the OAuth2 token flow with refresh handling
3. Build a REST client with retry logic and rate limit handling (Kalshi limits to 10 requests per second)
4. Implement pagination for market listing endpoints
5. Parse the response schemas (which differ between v1 and v2 endpoints)
6. Set up WebSocket connections for real-time price updates
7. Build a polling loop for markets that do not support WebSocket
8. Handle market ID resolution (Kalshi uses ticker strings, not UUIDs)

**For Polymarket**, add:

1. Understanding the CLOB API vs. the Gamma API (different endpoints, different auth)
2. Querying the Polygon subgraph for on-chain settlement data
3. Mapping condition IDs to human-readable market descriptions
4. Handling the CTF (Conditional Token Framework) for position accounting
5. Building a separate order book parser for CLOB data

**To combine both**, add:

1. A normalization layer that maps Kalshi tickers and Polymarket condition IDs to a unified market representation
2. Price reconciliation (Kalshi quotes in cents, Polymarket in token prices that are approximately but not exactly cents)
3. Deduplication logic for markets that exist on both platforms
4. A scheduler to keep both data feeds current without exceeding rate limits

That is a real project. Depending on how polished you want it, budget two to four weeks of development time. And you still need to maintain it when either platform changes their API, which happens regularly.

The MCP alternative:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

One line. All 18 tools. Both platforms. Live data. Done.

This is not to say the DIY approach is wrong — if you need custom execution logic or are building a trading system with specific latency requirements, you should own that infrastructure. But for research, analysis, thesis development, and exploratory work, the MCP server handles the entire data layer so you can focus on the actual decision-making.

## Practical patterns

Once the MCP server is connected, here are patterns that work well in practice.

**Morning scan.** Start a Claude Code session and ask: "What prediction markets moved the most overnight?" The agent uses `explore_public` to pull trending markets and surfaces the biggest movers. Takes about 10 seconds to get a cross-platform briefing.

**Event-driven research.** When news breaks, ask: "How are prediction markets reacting to [event]?" The agent calls `scan_markets` with the event topic and `get_context` for background. You get a real-time probability read on how the market is pricing the news.

**Thesis documentation.** If you keep research notes in markdown (and you should), ask the agent to build a thesis and write the output to a file. The structured thesis from `create_thesis` drops cleanly into a markdown document with markets, prices, key dates, and your edge estimates all formatted and current.

**Portfolio monitoring.** If you have connected trading accounts, ask "how is my portfolio doing" at any point. The agent calls `check_portfolio` and `get_pnl` to give you a live snapshot without opening a browser or switching to a trading terminal.

**Pre-trade checklist.** Before placing a trade, ask: "Run a risk check and tell me what happens if I am wrong." The agent calls `risk_check` for current exposure and `what_if` for the downside scenario. Two tool calls that take seconds and might save you from a correlated blowup.

## Limitations and what to expect

Some honest notes on what this setup does and does not do.

The unauthenticated tools (`scan_markets`, `explore_public`, `get_milestones`, `get_schedule`) give you read-only market data. That is genuinely useful for research, but you cannot see your positions or place trades without connecting your accounts through the SimpleFunctions configuration.

The MCP server is a layer on top of the underlying exchanges. It is not a replacement for direct API access if you are building high-frequency strategies or need sub-second execution. It is optimized for the research and analysis workflow, not for latency-sensitive trading.

Market data coverage depends on what Kalshi and Polymarket list. If a market does not exist on either platform, it will not show up in your scans. That said, between the two platforms you get coverage of politics, economics, crypto, sports, weather, entertainment, and current events — most things worth predicting have a market somewhere.

## Getting started in the next five minutes

Here is the minimum path to pulling live prediction market data into your agent:

```bash
# Install Claude Code if you have not already
npm install -g @anthropic-ai/claude-code

# Add the MCP server
claude mcp add simplefunctions --url https://simplecunctions.dev/api/mcp/mcp

# Start a session and try it
claude

# Then just ask:
# "scan prediction markets for AI regulation"
# "what markets are trending right now?"
# "show me the schedule for upcoming Fed meetings"
```

Four commands. No API keys. No configuration files. You now have an AI agent that can search, analyze, and reason about prediction market data from two major platforms in real time.

The prediction market ecosystem is growing fast. Kalshi now covers hundreds of event categories. Polymarket has become the default venue for crypto-adjacent prediction markets and increasingly for political and economic events. Having both of them wired into your development environment through a single MCP connection is the kind of leverage that compounds — every question you ask gets an answer grounded in what the market actually believes, not what a language model was trained on months ago.

That is the real value here. Prediction markets produce calibrated probability estimates updated in real time. Language models produce plausible-sounding text based on training data with a fixed cutoff. Connecting them through MCP gives you an agent that can tell you not just what might happen, but what the sharpest capital in the world currently thinks will happen, and where they might be wrong.