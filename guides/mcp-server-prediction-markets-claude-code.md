# Connect Claude Code to Prediction Markets: MCP Server Setup Guide

> One command to give your AI agent access to Kalshi and Polymarket data.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 5 min

---
## Give Your AI Agent Access to Prediction Markets

The Model Context Protocol (MCP) lets AI coding assistants like Claude Code, Cursor, and Cline call external tools. SimpleFunctions provides an MCP server with 16 tools for prediction market intelligence.

## Quick Start

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

That's it. Your Claude Code session now has 16 prediction market tools.

## What You Can Do

### Market Research
Ask Claude: "What prediction markets exist for Fed rate decisions?"

Claude will call `scan_markets` and return results from both Kalshi and Polymarket.

### Orderbook Analysis
Ask: "Show me the orderbook for KXWTIMAX-26DEC31-T135"

Claude calls `inspect_book` and returns bid/ask levels, spread, depth, and liquidity score.

### Thesis Management
Ask: "Create a thesis that oil will exceed $120 by end of 2026"

Claude calls `create_thesis`, which:
1. Builds a causal tree decomposing the thesis
2. Scans Kalshi + Polymarket for related markets
3. Identifies mispriced contracts (edges)
4. Sets up 24/7 autonomous monitoring

### Continuous Monitoring
Ask: "What's the latest on my oil thesis?"

Claude calls `get_context` and gets:
- Current confidence level
- Updated causal tree node probabilities
- Fresh edge calculations with orderbook data
- Latest evaluation summary

### What-If Scenarios
Ask: "What if Hormuz shipping drops to 10% probability?"

Claude calls `what_if` to simulate:
- How confidence changes
- Which edges shrink or reverse
- Which positions are at risk

Zero LLM cost — pure computation.

## Available Tools (16)

| Tool | Description |
|------|-------------|
| `get_context` | Thesis snapshot with edges, nodes, evaluation |
| `list_theses` | List all your theses |
| `create_thesis` | Create a new thesis with causal model |
| `inject_signal` | Feed news/observations to thesis |
| `trigger_evaluation` | Force immediate re-evaluation |
| `what_if` | Scenario analysis (zero LLM cost) |
| `scan_markets` | Search Kalshi + Polymarket |
| `inspect_book` | Orderbook depth for specific markets |
| `get_liquidity` | Topic-wide liquidity scan |
| `get_positions` | Kalshi + Polymarket positions |
| `get_balance` | Account balance |
| `get_orders` | Resting orders |
| `get_fills` | Trade history |
| `get_settlements` | Settled contracts |
| `get_milestones` | Upcoming Kalshi events |
| `get_schedule` | Exchange status |

## Also Works With

- **Cursor**: Add MCP server in settings
- **Cline**: Add to MCP config
- **Any MCP client**: Use the URL `https://simplefunctions.dev/api/mcp/mcp`

## CLI Alternative

If you prefer a command-line workflow:

```bash
npm install -g @spfunctions/cli
sf setup
sf scan "oil recession"
sf book KXWTIMAX-26DEC31-T135
sf liquidity oil
```

## Links

- [Documentation](https://simplefunctions.dev/docs)
- [GitHub](https://github.com/spfunctions/simplefunctions-cli)
- [API Reference](https://simplefunctions.dev/llms.txt)