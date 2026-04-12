# Orderbook

**An orderbook is a real-time list of all outstanding buy and sell orders for a prediction market contract, organized by price level. It shows the supply and demand at every price point.**


## Explanation

## Reading an Orderbook

The orderbook is the most important data structure in trading. It shows you exactly who wants to buy and sell, and at what prices.

### Structure

An orderbook has two sides:
- **Bids (left/buy side)**: Orders from people wanting to buy. Sorted highest price first. The best bid is the highest price anyone will pay.
- **Asks (right/sell side)**: Orders from people wanting to sell. Sorted lowest price first. The best ask is the lowest price anyone will accept.

### What It Tells You

1. **Current price**: The midpoint between best bid and best ask
2. **Spread**: The gap between best bid and best ask
3. **Depth**: How many contracts are available at each price level
4. **Support/resistance**: Large resting orders act as barriers to price movement
5. **Imbalance**: If bids are much larger than asks (or vice versa), it suggests directional pressure

### Orderbook vs. AMM

Kalshi uses an orderbook model where every trade matches a specific buyer with a specific seller. Polymarket uses an AMM (Automated Market Maker) for most markets, where you trade against a liquidity pool rather than specific counterparties. The CLI normalizes both into a consistent format.

### Using the Orderbook for Edge Detection

The orderbook reveals more than just prices. A thin orderbook with a big ask wall at 35 cents might indicate a large seller who needs to exit — potentially creating an opportunity if your thesis says fair value is higher.


## Example

sf depth KXCPI-26MAR-T3.5:

  BIDS                    ASKS
  ----                    ----
  200 @ $0.50             100 @ $0.53
  400 @ $0.49             250 @ $0.54
  150 @ $0.48             500 @ $0.55  ← ask wall
  600 @ $0.47             100 @ $0.56

Best bid: $0.50  |  Best ask: $0.53
Spread: $0.03    |  Midpoint: $0.515

The 500-contract ask wall at $0.55 suggests a large
seller is defending that level. To push the price above
$0.55, buyers would need to absorb 850 contracts.


## CLI

```bash
sf depth KXCPI-26MAR-T3.5
```


## Related

[bid-ask](bid-ask.md), [depth](depth.md), [spread](spread.md), [market-maker](market-maker.md), [liquidity](liquidity.md)
