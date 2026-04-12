# Command-Line Interface (CLI)

**The SimpleFunctions CLI is a terminal-based tool for interacting with prediction markets — scanning for edge, managing theses, viewing orderbooks, and executing strategies from the command line.**


## Explanation

## Why a CLI?

GUIs are great for browsing. CLIs are better for systematic trading. The SimpleFunctions CLI gives you:

1. **Speed**: Type a command, get results instantly. No clicking through menus.
2. **Scriptability**: Chain commands, pipe output, build workflows
3. **Agent integration**: The CLI is the primary interface for the agent system
4. **Keyboard-only**: Never leave your terminal

### Core Commands

- `sf scan` — Scan markets with filters (category, probability range, venue)
- `sf edges` — Find edge opportunities based on active theses
- `sf depth` — View live orderbook for a specific market
- `sf market` — Get detailed info on a single market
- `sf thesis create` — Start building a new thesis
- `sf thesis evaluate` — Trigger evaluation with new signals
- `sf strategies` — View and manage trading strategies
- `sf heartbeat status` — Check the monitoring engine
- `sf what-if` — Run scenario analysis

### Installation

```bash
npm install -g simplefunctions
sf auth login
```

### Interactive Mode

The CLI supports an interactive chat mode where you converse with the agent:

```bash
sf chat
```

This opens an interactive session where you can discuss theses, ask questions about markets, and instruct the agent to take actions — all in natural language.


## Example

# Scan for high-edge economic markets
sf scan --category economics --min-edge 5

# View orderbook depth
sf depth KXRECSSNBER-26

# Create a thesis interactively
sf thesis create "Oil prices will exceed $120 by mid-2026"

# Check your portfolio
sf portfolio

# Run a what-if scenario
sf what-if --node n1.1 --prob 0.80

# Start the heartbeat monitor
sf heartbeat start --interval 30m


## CLI

```bash
sf help
```


## Related

[mcp-server](mcp-server.md), [api-key](api-key.md), [edge](edge.md), [thesis](thesis.md)
