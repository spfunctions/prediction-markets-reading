# How to Scan Prediction Market Orderbooks: Spread, Depth, and Liquidity Analysis

> A practical guide to reading and analyzing orderbook data from Kalshi and Polymarket.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 6 min

---
## Why Orderbook Analysis Matters for Prediction Markets

Edge detection is not enough. You can find a market where your thesis implies 65¢ but the market trades at 40¢ — a theoretical edge of +25¢. But if the spread is 15¢ and depth is 50 contracts, your actual fill will be terrible.

**Orderbook analysis tells you whether an edge is executable.**

## Key Metrics

### Spread
The difference between best bid and best ask. In prediction markets:
- **≤ 2¢**: Tight spread, high conviction market
- **3-5¢**: Moderate spread, tradeable
- **> 5¢**: Wide spread, illiquid — proceed with caution

### Depth
Number of contracts on the top 3 price levels. More depth = more contracts you can trade without moving the price.

### Liquidity Score
We score markets as:
- **High**: Spread ≤ 2¢ AND depth ≥ 500 contracts
- **Medium**: Spread ≤ 5¢ AND depth ≥ 100 contracts
- **Low**: Everything else

### Executable Edge
The real edge after accounting for the spread:
```
YES buyer: executableEdge = thesisPrice - askPrice
NO buyer:  executableEdge = bidPrice - thesisPrice
```

A market with +25¢ theoretical edge but 10¢ spread has only +15¢ executable edge.

## Scanning Orderbooks with the CLI

### Single Market Deep Dive

```bash
sf book KXWTIMAX-26DEC31-T135
```

Output:
```
KLSH Will the maximum WTI front month settle price reach $135.01 by Dec 31, 2026?
KXWTIMAX-26DEC31-T135

  Last 47¢  Bid 48¢  Ask 49¢  Spread 1¢  Liq high
  Vol24h 3K  OI 31K  Expires 2026-12-31

  BID                 ASK
  ──────────────────────────────────────
    48¢       59    49¢      527
    47¢      578    50¢      332
    45¢       10    51¢      400
    44¢      194    52¢      746
    37¢       11    53¢      500
  depth: 14406 bid / 13262 ask
```

### Topic-Wide Liquidity Scan

```bash
sf liquidity oil          # All oil-related markets
sf liquidity crypto       # All crypto markets
sf liquidity geopolitics  # Iran, Hormuz, etc.
sf liquidity              # List all 18 available topics
```

### Polymarket Markets

```bash
sf book --poly "oil price above 100"
sf book --poly "iran war ceasefire"
```

### Cross-Venue Comparison

```bash
sf scan "oil" --venue all  # Search both Kalshi + Polymarket
```

## Reading the Signals

### Spread Widening
If a market's spread was 2¢ yesterday and is 8¢ today, market makers are withdrawing — uncertainty is rising.

### Depth Imbalance
If bid depth is 3x ask depth, buyers are more aggressive. The market may be about to move up.

### Volume Spike Without Price Move
High volume but stable price means large orders are being absorbed — someone is providing serious liquidity at current levels.

## Automating Orderbook Monitoring

SimpleFunctions' heartbeat engine monitors orderbooks every 15 minutes for all your thesis edges:
- Persists snapshots to database for time-series analysis
- Generates commentary: "spread widening", "buyers aggressive", "sell-side depth dropped 40%"
- Recalculates executable edge from live orderbook data

```bash
# Set up autonomous monitoring
sf create "Oil will exceed $120 by end of 2026"
# Heartbeat engine automatically scans orderbooks every 15 min
sf context <id> --json  # See latest orderbook signals in edges
```

## Tools

- **sf book**: Deep dive on specific markets
- **sf liquidity**: Topic-wide liquidity scan
- **sf scan**: Cross-venue market search
- **MCP**: `claude mcp add simplefunctions` for AI agent access