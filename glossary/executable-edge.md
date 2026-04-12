# Executable Edge

**Executable edge is edge that can actually be traded — meaning the market has sufficient liquidity, tight enough spreads, and low enough fees for you to profitably capture the mispricing.**


## Explanation

## Theoretical Edge vs. Executable Edge

Finding a mispriced contract is only half the battle. The other half is being able to trade it profitably after accounting for real-world friction.

### The Friction Stack

When you trade a prediction market contract, you face several costs:

1. **Spread cost**: The difference between bid and ask eats into your edge. A 10-point edge with a 6-cent spread leaves only 4 points of executable edge.
2. **Fees**: Kalshi charges per-contract fees (typically 1-3 cents). This further reduces executable edge.
3. **Slippage**: If you're trading more than the top-of-book quantity, you'll fill at worse prices deeper in the orderbook.
4. **Market impact**: Large orders move the price against you, especially in thin markets.

### Example: Edge That Isn't Executable

Your thesis says a contract should be at 45 cents. It's trading at 30 cents — a 15-point edge. But:
- The ask is at 35 cents (not 30 — the 30 cent price is the last trade)
- Only 20 contracts available at 35 cents
- Next level is 38 cents for 50 contracts
- Kalshi fee: 2 cents per contract
- Your target size: 200 contracts

Your actual average entry would be ~37 cents, plus 2 cents in fees, for an effective cost of 39 cents. Your executable edge is 45 - 39 = 6 points, not 15 points.

### How SimpleFunctions Handles This

The `sf edges` command includes a liquidity score (A through D) alongside each edge. This score factors in spread, depth, and recent volume to indicate how much of the theoretical edge is actually capturable. An edge rated "A" means most of the theoretical edge survives friction. An edge rated "D" means it's likely paper-only.


## Example

Theoretical edge: 15 points (thesis: 45%, market: 30%)

Cost breakdown:
  Spread cost:   -5 points (ask at 35¢, not 30¢)
  Fee:           -2 points (Kalshi per-contract fee)
  Slippage:      -2 points (filling 200 contracts across price levels)

Executable edge: 15 - 5 - 2 - 2 = 6 points
Liquidity score: B (partially executable)

Rule of thumb: Only trade when executable edge > 5 points.


## CLI

```bash
sf edges --min-edge 5
```


## Related

[edge](edge.md), [spread](spread.md), [slippage](slippage.md), [liquidity-score](liquidity-score.md), [depth](depth.md)
