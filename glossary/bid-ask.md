# Bid and Ask Prices

**The bid price is the highest price a buyer is currently willing to pay for a contract. The ask price is the lowest price a seller is currently willing to accept. Together they define the tradeable price range.**


## Explanation

## Bid and Ask Explained

When you look at a prediction market contract, you don't see one price — you see two:

- **Bid**: "I'll buy at this price." This is what you get if you sell immediately.
- **Ask**: "I'll sell at this price." This is what you pay if you buy immediately.

### The "Last Price" Trap

Many platforms prominently display the "last traded price." This can be misleading because:
- It might be hours or days old
- It might have been a single small trade
- The current bid/ask might be far from the last price

Always check the live bid and ask, not just the last price. The CLI shows you real-time bid/ask data.

### Buying and Selling

- **To buy**: You either (a) pay the ask price for instant execution, or (b) post a bid at a lower price and wait
- **To sell**: You either (a) accept the bid price for instant execution, or (b) post an ask at a higher price and wait

### Bid-Ask and Yes/No

Remember: every prediction market contract has a Yes and No side. If the Yes bid is 42 cents, the No ask is 58 cents (they must sum to $1.00). When you buy Yes at 42 cents, someone is effectively selling No at 58 cents.


## Example

Contract: KXGDP-26Q2-T2.0 "Will Q2 2026 GDP exceed 2.0%?"

  Yes bid: $0.62    Yes ask: $0.65
  No bid:  $0.35    No ask:  $0.38

These are equivalent:
  Buying Yes at $0.65  ↔  Selling No at $0.35
  Selling Yes at $0.62 ↔  Buying No at $0.38

Midpoint: $0.635 (market-implied probability: ~63.5%)


## CLI

```bash
sf market KXGDP-26Q2-T2.0
```


## Related

[spread](spread.md), [orderbook](orderbook.md), [market-order](market-order.md), [limit-order](limit-order.md)
