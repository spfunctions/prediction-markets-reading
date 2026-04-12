# Take Profit

**A take-profit level is a pre-defined exit condition that closes your position when the price reaches your target. It locks in gains before the market can reverse.**


## Explanation

## Taking Profits in Prediction Markets

Take-profit orders are the mirror image of stop losses. While stops protect your downside, take-profit levels secure your upside.

### When to Take Profits

There are several approaches:

1. **Thesis-implied target**: Exit when the market price reaches your estimated fair value. If your thesis says 50% and you bought at 30 cents, take profits at 50 cents.
2. **Partial scaling**: Take profits in stages. Sell 1/3 at 40 cents, 1/3 at 50 cents, hold the final 1/3 for settlement.
3. **Edge depletion**: Exit when the edge shrinks below a minimum threshold (e.g., below 3 points).

### The Challenge of Prediction Markets

Unlike stocks, prediction market contracts have a hard ceiling at $1.00 and floor at $0.00. This bounded payoff means:
- Your maximum possible profit is $1.00 minus your entry price
- Contracts above 80 cents have limited upside remaining
- The reward/risk profile changes as the price moves

### Take Profit in SimpleFunctions

The strategy engine's `takeProfit` parameter sets a price level at which the system will look to exit. Combined with soft conditions, you can create nuanced exit strategies:

- `takeProfit: 55` — sell when bid reaches 55 cents
- Soft: "Scale out 50% at target, hold remainder for settlement"

### Don't Hold to Settlement

Unless your thesis strongly suggests the event will happen (>80% confidence), consider taking profits before settlement. Contracts can whipsaw violently in the final days, and the time value of capital matters.


## Example

Entry: Bought 300 contracts at $0.30 (cost: $90)
Thesis target: 50% ($0.50)

Scaling plan:
  At $0.40: Sell 100 contracts → Profit: 100 × $0.10 = $10
  At $0.50: Sell 100 contracts → Profit: 100 × $0.20 = $20
  At $0.60: Sell 100 contracts → Profit: 100 × $0.30 = $30

Total profit if fully scaled: $60 (67% return on $90)

Alternative: Hold all to settlement
  If settles Yes: 300 × $0.70 = $210 profit (233% return)
  If settles No:  300 × $0.30 = $90 loss (100% loss)
  Expected value at 50%: $210 × 0.5 - $90 × 0.5 = $60


## CLI

```bash
sf strategies
```


## Related

[stop-loss](stop-loss.md), [position-sizing](position-sizing.md), [edge](edge.md), [cost-basis](cost-basis.md)
