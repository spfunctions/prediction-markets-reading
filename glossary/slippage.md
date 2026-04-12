# Slippage

**Slippage is the difference between the expected execution price and the actual price at which a trade fills. It occurs when you trade larger quantities than available at the best price.**


## Explanation

## What Causes Slippage

Slippage happens when your order size exceeds the liquidity at the best price level. If the ask is 30 cents for 50 contracts but you want 200 contracts, you'll fill 50 at 30 cents, then fill deeper in the orderbook at 31, 32, 33 cents — paying more on average than you expected.

### Types of Slippage

1. **Size slippage**: Your order is larger than the top-of-book quantity. This is predictable — you can see the orderbook before trading.
2. **Speed slippage**: The market moves between when you see a price and when your order arrives. More common during volatile events.
3. **Impact slippage**: Your order itself moves the market. Other participants see your buying and adjust their prices upward.

### Measuring Slippage

Slippage is usually measured as the difference between the midpoint price and your average fill price.

### Minimizing Slippage

- **Use limit orders**: Set a maximum price you're willing to pay. You might not fill entirely, but you control your worst-case execution.
- **Scale into positions**: Instead of buying 500 contracts at once, buy 50-100 at a time over hours or days.
- **Check depth first**: Always run `sf depth` before placing a trade to understand the orderbook shape.
- **Time your entries**: Trade during high-volume periods when orderbooks are deepest.

### Why SimpleFunctions Tracks Slippage

The strategy engine uses `perOrderQuantity` to control how much you trade in each execution. If your target position is 500 contracts, it might fill 50 at a time to minimize market impact. The `maxQuantity` parameter caps total position size.


## Example

You want to buy 300 Yes contracts on KXRECSSNBER-26.

Orderbook (ask side):
  50 @ $0.29
  100 @ $0.30
  150 @ $0.31
  200 @ $0.32

Expected fill:
  50 × $0.29 = $14.50
  100 × $0.30 = $30.00
  150 × $0.31 = $46.50
  Total: $91.00 for 300 contracts
  Average price: $0.3033

Slippage = $0.3033 - $0.29 (best ask) = $0.0133 per contract
Total slippage cost: $0.0133 × 300 = $4.00


## CLI

```bash
sf depth
```


## Related

[spread](spread.md), [depth](depth.md), [liquidity](liquidity.md), [market-order](market-order.md), [limit-order](limit-order.md)
