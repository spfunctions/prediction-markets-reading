# Market Order

**A market order is an instruction to buy or sell immediately at the best available price. It guarantees execution but not price — you may experience slippage in thin markets.**


## Explanation

## Market Orders in Prediction Markets

A market order says "fill me now at whatever price is available." On the buy side, you pay the current ask. On the sell side, you receive the current bid.

### When to Use Market Orders

Market orders make sense when:
- Speed is more important than price (breaking news events)
- The spread is very tight (1-2 cents) and slippage is minimal
- You're trading small size relative to orderbook depth
- You need to exit a losing position urgently

### When to Avoid Market Orders

- **Wide spreads**: A 10-cent spread means you're overpaying significantly
- **Thin markets**: Your order might fill across many price levels
- **Non-urgent entries**: If you're building a position over days, limit orders save money

### Market Orders on Kalshi

On Kalshi, you submit a market order by placing a limit order at a price guaranteed to cross the spread. For example, to buy immediately, you might submit a buy order at 99 cents — it will fill at the best available ask, not at 99 cents.

### Impact on Your Returns

For a typical prediction market trade with 10 points of edge:
- **Limit order fill**: You capture close to 10 points of edge
- **Market order**: Spread cost eats 3-5 points, leaving 5-7 points of executable edge

Over hundreds of trades, this difference compounds significantly. Professional traders use limit orders for 90%+ of their activity.


## Example

KXRECSSNBER-26 orderbook:
  Best bid: $0.27  (200 contracts)
  Best ask: $0.30  (100 contracts)

Market order to BUY 100 contracts:
  Fills at $0.30 (the ask). Cost: $30.00.

Market order to BUY 250 contracts:
  100 @ $0.30 = $30.00
  150 @ $0.31 = $46.50
  Total: $76.50, average price: $0.306

Compare: Limit order at $0.28, wait for fill:
  250 @ $0.28 = $70.00 (saves $6.50)


## Related

[limit-order](limit-order.md), [slippage](slippage.md), [spread](spread.md), [orderbook](orderbook.md)
