# Liquidity Score

**A liquidity score is a composite rating (typically A through D) that grades a market's tradeability based on spread, orderbook depth, recent volume, and historical fill rates.**


## Explanation

## How Liquidity Scores Work

SimpleFunctions computes a liquidity score for every market it scans. This score combines multiple factors into a single letter grade, making it easy to quickly identify tradeable markets.

### Scoring Components

The score is calculated from:
1. **Spread** (30% weight): Tighter spreads score higher. Under 3 cents is excellent.
2. **5-level depth** (30% weight): More contracts available near the midpoint scores higher.
3. **24h volume** (20% weight): Higher recent volume indicates active participation.
4. **Open interest** (10% weight): More outstanding contracts suggest a mature market.
5. **Fill rate** (10% weight): How often posted orders actually fill (measured historically).

### Grade Definitions

- **A**: Spread < 3 cents, depth > 1,000 contracts. You can trade aggressively.
- **B**: Spread < 5 cents, depth > 200 contracts. Trade with limit orders.
- **C**: Spread < 10 cents, depth > 50 contracts. Small positions only.
- **D**: Spread > 10 cents or depth < 50 contracts. Edge likely not executable.

### Using Scores in Practice

When `sf edges` returns results, always filter by liquidity score first. A 20-point edge with a D-grade liquidity score is worth less than a 5-point edge with an A-grade score, because you can actually execute the latter.

The `--min-liquidity` flag lets you filter: `sf edges --min-liquidity B` shows only markets where you can reasonably execute.


## Example

sf edges --min-liquidity B output:

Market                    Edge   Liq   Executable Edge
KXRECSSNBER-26           +17pt   A     ~14pt (spread + fee adjusted)
KXGDP-26Q2-TNEG         +12pt   B     ~8pt
KXUNRATE-26JUN-T5.0      +7pt   A     ~5pt

Filtered out (below B):
KXWTIMAX-26DEC31-T150   +15pt   D     ~2pt (edge exists but can't execute)


## CLI

```bash
sf edges --min-liquidity B
```


## Related

[liquidity](liquidity.md), [depth](depth.md), [spread](spread.md), [executable-edge](executable-edge.md)
