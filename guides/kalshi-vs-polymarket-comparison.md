# Kalshi vs Polymarket: A Developer's Comparison of APIs, Orderbooks, and Liquidity

> A data-driven comparison of the two largest prediction markets from a developer and trader perspective.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 8 min

---
## Kalshi vs Polymarket: Which Prediction Market Should You Build On?

If you're building a trading bot, analysis tool, or AI agent for prediction markets, your first decision is: Kalshi or Polymarket? Here's a data-driven comparison based on actually building on both APIs.

## Architecture

| Feature | Kalshi | Polymarket |
|---------|--------|------------|
| **Regulation** | CFTC-regulated (US) | Unregulated (crypto) |
| **Settlement** | USD (bank account) | USDC on Polygon |
| **Auth** | RSA-PSS signed requests | EIP-712 + HMAC-SHA256 |
| **Orderbook** | Central limit order book | Central limit order book (CLOB) |
| **API Style** | REST + WebSocket + FIX | REST + WebSocket |

## API Comparison

### Market Discovery

**Kalshi** organizes markets as Series → Events → Markets. You search by series ticker (e.g., `KXWTIMAX`) to find all related markets.

```bash
# Kalshi: ~9000 series, search by keyword
sf scan "oil recession"

# Polymarket: search by natural language
sf scan --venue polymarket "oil price above 100"
```

**Polymarket** uses Events → Markets with tag-based categorization. The Gamma API (`gamma-api.polymarket.com`) handles discovery, while the CLOB API (`clob.polymarket.com`) handles pricing.

### Orderbook Data

Both exchanges expose full orderbook depth. Here's how to compare them side by side:

```bash
# Compare orderbooks across venues
sf book KXWTIMAX-26DEC31-T135 --history
sf book --poly "oil price above 100"
```

**Key difference**: Kalshi's orderbook uses YES/NO dollar prices (e.g., `yes_bid_dollars: "0.48"`). Polymarket's CLOB returns decimal prices (e.g., `price: "0.48"`) that represent probability directly.

### Liquidity

In our testing across 18 market categories:

- **Crypto markets**: Polymarket has 5-10x more depth
- **Macro/geopolitics**: Roughly equal, Kalshi slightly ahead on US-centric events
- **Sports**: Kalshi dominates (90% of their volume)

```bash
# Scan liquidity across both venues
sf liquidity crypto    # Compare crypto orderbooks
sf liquidity oil       # Compare oil/energy orderbooks
```

### Price History

**Kalshi**: Authenticated candlestick API with OHLC data (1min to daily intervals). Requires API keys.

**Polymarket**: Public OHLC endpoint (`/ohlc`) with 1m to 1w intervals. Also has a simpler `/prices-history` endpoint for time series. No auth required.

### Rate Limits

| Endpoint Type | Kalshi | Polymarket |
|---------------|--------|------------|
| General | ~100 req/10s | 15,000 req/10s |
| Orderbook | ~50 req/10s | 1,500 req/10s |
| Trading | ~20 req/10s | 3,500 req/10s |

Polymarket is significantly more generous with rate limits.

## When to Use Which

**Use Kalshi when:**
- You need CFTC regulation and USD settlement
- Trading sports or US-centric events
- You want FIX protocol for low-latency trading

**Use Polymarket when:**
- You need higher liquidity on crypto/geopolitical events
- You want simpler API access (no auth for most data)
- You're building for a global audience

**Use both when:**
- You want cross-venue edge detection
- You're building an arbitrage strategy
- You want the best price across venues

## Tools for Building on Both

```bash
# Install SimpleFunctions CLI — works with both venues
npm install -g @spfunctions/cli
sf setup

# Scan both venues at once
sf scan "oil recession iran"

# Deep orderbook analysis
sf book KXWTIMAX-26DEC31-T135

# Liquidity comparison
sf liquidity oil

# Connect to Claude Code via MCP
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

## Conclusion

Neither platform is strictly better. Kalshi wins on regulation and US market depth; Polymarket wins on API ergonomics, rate limits, and crypto/geopolitical liquidity. The best tooling works with both.