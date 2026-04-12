# Market Depth

**Market depth is the total quantity of resting orders at each price level in the orderbook. Deep markets can absorb large trades with minimal price impact; shallow markets cannot.**


## Explanation

## What is Market Depth?

Depth measures how much liquidity exists beyond the best bid and ask. A market might have a tight 2-cent spread, but if there are only 10 contracts at the best price, it's shallow — a 100-contract order would blow through multiple price levels.

### Reading Depth

When you run `sf depth`, you see the full orderbook with quantities at each level. The key metrics are:

- **Top-of-book**: Contracts at the best bid and best ask
- **5-level depth**: Total contracts within 5 cents of the midpoint
- **Total depth**: All resting orders on both sides

### Depth vs. Volume

A market can have high depth but low volume (lots of resting orders, few trades happening). This is common in markets waiting for a catalyst — market makers post orders but nobody is taking them.

Conversely, a market can have low depth but high volume during a news event — orders are being filled and replenished rapidly.

### Why Depth Matters for Edge

The SimpleFunctions liquidity score combines depth with spread to grade how executable an edge is. A 10-point edge on a market with only 20 contracts of depth is much less valuable than a 5-point edge on a market with 5,000 contracts of depth.

### Depth and Position Sizing

Your position size should never exceed a small fraction of total depth. A common rule: don't try to fill more than 10% of the visible bid-side depth in a single order, or you'll cause excessive slippage.


## Example

Two markets with the same edge but different depth:

Market A: KXRECSSNBER-26
  Edge: +8 points  |  5-level depth: 2,500 contracts
  You can comfortably take a 250-contract position.

Market B: KXWTIMAX-26DEC31-T150
  Edge: +8 points  |  5-level depth: 80 contracts
  A 250-contract order would move the price ~10 cents.
  Maximum practical position: ~20 contracts.

Edge × executable size = expected dollar profit.
Market A: 8pt × 250 = $20 expected profit
Market B: 8pt × 20  = $1.60 expected profit


## CLI

```bash
sf depth
```


## Related

[orderbook](orderbook.md), [liquidity](liquidity.md), [liquidity-score](liquidity-score.md), [slippage](slippage.md)
