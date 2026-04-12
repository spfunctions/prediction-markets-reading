# Limit Order

**A limit order is an instruction to buy or sell a contract at a specified price or better. Unlike market orders, limit orders guarantee your price but not execution — your order only fills if the market reaches your price.**


## Explanation

## Limit Orders in Prediction Markets

A limit order lets you set the price you're willing to trade at. The order sits in the orderbook until someone takes the other side, or you cancel it.

### How Limit Orders Work

**Buy limit at 28 cents**: Your order rests on the bid side. If someone sells at 28 cents or lower, your order fills. If the market never trades that low, your order remains unfilled.

**Sell limit at 45 cents**: Your order rests on the ask side. If someone buys at 45 cents or higher, your order fills.

### Advantages

- **Price control**: You never pay more (or receive less) than your specified price
- **Lower costs**: Limit orders avoid crossing the spread, saving you the spread cost
- **Patience premium**: In prediction markets, patient limit orders often capture better prices than aggressive market orders

### Disadvantages

- **No fill guarantee**: The market might never reach your price
- **Opportunity cost**: While waiting for a fill, the market might move away from you
- **Partial fills**: You might get filled on only part of your order

### Using Limit Orders Strategically

On Kalshi, most sophisticated traders use limit orders exclusively. The strategy engine places limit orders based on your entry conditions:

- `entryBelow: 30` — places a buy limit at 30 cents
- `entryAbove: 70` — places a sell limit at 70 cents

The heartbeat service monitors these and adjusts as needed.


## Example

You want to buy KXRECSSNBER-26 but the current ask is $0.31.
Your thesis supports buying at $0.28 or below.

Place limit order: Buy 200 @ $0.28

Scenario 1 — Fill: New economic data pushes price down.
  Someone sells at $0.28, your order fills. Cost: $56.00.

Scenario 2 — No fill: Market stays above $0.28.
  Your order sits unfilled. No cost, no position.

Scenario 3 — Partial fill: Only 80 contracts available at $0.28.
  You get 80 contracts filled, 120 remain as resting order.


## CLI

```bash
sf strategies
```


## Related

[market-order](market-order.md), [bid-ask](bid-ask.md), [orderbook](orderbook.md), [spread](spread.md)
