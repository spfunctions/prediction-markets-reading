# Edge Detection

**Edge detection is the systematic process of scanning prediction markets to find contracts where the market price diverges from your thesis-implied fair value by more than the cost of execution.**


## Explanation

## Systematic Edge Detection

Edge detection is what separates systematic trading from casual speculation. Instead of browsing markets and making gut-feel decisions, you define your view through a thesis, compute implied prices, and let the system find mismatches.

### The Edge Detection Pipeline

1. **Thesis computation**: Your causal tree produces thesis-implied prices for the root event
2. **Market mapping**: The system identifies all contracts related to your thesis (across Kalshi and Polymarket)
3. **Price comparison**: Current market prices are compared against thesis-implied prices
4. **Friction adjustment**: Spread, fees, and estimated slippage are subtracted from theoretical edge
5. **Ranking**: Opportunities are ranked by executable edge × liquidity score

### Cross-Venue Edge Detection

One of the most powerful forms of edge detection is cross-venue — finding the same event priced differently on Kalshi vs. Polymarket. If Kalshi prices a recession at 28% and Polymarket at 35%, there's at least a 7-point arbitrage opportunity (minus cross-venue friction).

### Edge Decay

Edges are perishable. The typical lifecycle:
1. New information creates a mispricing (minutes)
2. Active traders spot and start trading it (minutes to hours)
3. Price converges toward fair value (hours to days)
4. Edge disappears completely

The heartbeat's frequency determines how quickly you catch edges. Faster heartbeats catch shorter-lived opportunities.

### Filters and Thresholds

Not every edge is worth trading. The `sf edges` command supports filters:
- `--min-edge 5`: Only show edges above 5 points
- `--min-liquidity B`: Only show markets with B-grade or better liquidity
- `--category economics`: Only show economic contracts


## Example

sf edges --min-edge 5 --min-liquidity B

Your thesis: "US Recession by 2026" (root: 42%)

  Market                    Mkt    Thesis  Edge   Liq  Venue
  KXRECSSNBER-26           28¢    42¢     +14pt   A   kalshi
  KXGDP-26Q2-TNEG          18¢    30¢     +12pt   B   kalshi
  recession-2026-poly       35¢    42¢     +7pt    A   polymarket
  KXUNRATE-26JUN-T5.0       35¢    42¢     +7pt    A   kalshi
  KXCPI-26MAR-T3.5          52¢    48¢     -4pt    A   kalshi (NO side edge)

Cross-venue opportunity:
  KXRECSSNBER-26 (kalshi) at 28¢ vs recession-2026-poly at 35¢
  Same event, 7¢ difference. Possible arb after fee adjustment.


## CLI

```bash
sf edges
```


## Related

[edge](edge.md), [executable-edge](executable-edge.md), [cross-venue](cross-venue.md), [thesis-implied-price](thesis-implied-price.md), [heartbeat](heartbeat.md)
