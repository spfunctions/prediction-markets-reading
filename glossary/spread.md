# Bid-Ask Spread

**The bid-ask spread is the difference between the highest price a buyer will pay (bid) and the lowest price a seller will accept (ask). It represents the cost of trading immediately.**


## Explanation

## Understanding Spreads

The spread is the toll you pay for instant execution. If the best bid is 42 cents and the best ask is 46 cents, the spread is 4 cents. You can:

1. **Buy at the ask** (46 cents) for immediate execution
2. **Sell at the bid** (42 cents) for immediate execution
3. **Post a limit order** between bid and ask and wait for a fill

### Spread as a Cost

If you buy at 46 cents and immediately sell, you'd get 42 cents — losing 4 cents per contract. This means you need at least 4 points of edge just to break even on a round-trip trade.

### What Drives Spread Width

- **Liquidity**: More liquid markets have tighter spreads (1-3 cents). Illiquid markets can have 5-15 cent spreads.
- **Volatility**: Spreads widen when the market is moving fast, because market makers face more risk.
- **Event proximity**: Spreads often narrow as settlement approaches because uncertainty decreases.
- **Time of day**: Kalshi spreads are often tightest during US market hours (9 AM - 4 PM ET).

### Reading Spreads in the CLI

The `sf depth` command shows you the full orderbook including the current spread. When `sf scan` reports edge, it always accounts for the spread — showing you the executable edge after spread costs.

### Spread Strategy

For markets with wide spreads, posting limit orders inside the spread is often better than crossing the spread. You sacrifice speed for a better price. For fast-moving markets (like during CPI releases), crossing the spread is necessary to get filled.


## Example

sf depth KXRECSSNBER-26:

  BID (buy)              ASK (sell)
  150 contracts @ $0.27  80 contracts @ $0.30
  300 contracts @ $0.26  200 contracts @ $0.31
  500 contracts @ $0.25  400 contracts @ $0.32

Spread: $0.30 - $0.27 = $0.03 (3 cents)
Midpoint: $0.285

To buy 80 contracts immediately: pay $0.30 (cross the spread)
To buy 80 contracts cheaply: post bid at $0.28 and wait


## CLI

```bash
sf depth
```


## Related

[bid-ask](bid-ask.md), [slippage](slippage.md), [liquidity](liquidity.md), [depth](depth.md), [market-maker](market-maker.md)
