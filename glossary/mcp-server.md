# MCP Server

**An MCP (Model Context Protocol) server exposes SimpleFunctions' prediction market data and analysis tools as structured endpoints that AI agents (like Claude, GPT, or custom agents) can call directly.**


## Explanation

## What is MCP?

The Model Context Protocol (MCP) is an open standard that lets AI agents call external tools in a structured way. Instead of an agent scraping a website or parsing unstructured text, it calls typed functions with parameters and receives structured responses.

### SimpleFunctions as an MCP Server

SimpleFunctions runs as an MCP server that exposes tools like:
- `scan_markets`: Find markets matching criteria with live prices
- `get_edges`: Get current edge calculations for a thesis
- `evaluate_thesis`: Trigger a thesis evaluation with new information
- `get_orderbook`: Get live orderbook depth for a specific market

### Why This Matters

Any AI agent that supports MCP can now:
1. Research prediction markets by calling `scan_markets`
2. Build and evaluate theses by calling `evaluate_thesis`
3. Find trading opportunities by calling `get_edges`
4. Monitor positions by calling the relevant tools

This turns prediction market analysis from a human-only activity into something any agent can do.

### Setting Up the MCP Server

The SimpleFunctions CLI includes a built-in MCP server:

```bash
sf mcp serve
```

This starts a local MCP server that Claude Desktop, Cursor, or any MCP-compatible agent can connect to. The server handles authentication, rate limiting, and data formatting automatically.

### Common Use Case

Connect SimpleFunctions MCP server to Claude Desktop. Then ask Claude: "What prediction markets have the biggest edge on recession right now?" Claude calls `scan_markets` and `get_edges` under the hood, returning a structured analysis.


## Example

# Start the MCP server
sf mcp serve --port 3847

# In Claude Desktop settings, add:
{
  "mcpServers": {
    "simplefunctions": {
      "command": "sf",
      "args": ["mcp", "serve"],
      "env": {
        "SF_API_KEY": "sf_live_abc123..."
      }
    }
  }
}

# Then in Claude Desktop:
"Find prediction markets where recession probability exceeds 40%"

Claude calls: scan_markets({ category: "economics", minProb: 0.4 })
Returns: Structured list of matching markets with prices, volume, edge


## CLI

```bash
sf mcp serve
```


## Related

[cli](cli.md), [api-key](api-key.md), [delta-api](delta-api.md), [streaming](streaming.md)
