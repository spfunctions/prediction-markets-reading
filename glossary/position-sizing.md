# Position Sizing

**Position sizing determines how many contracts to buy based on your edge size, confidence level, and bankroll. The Kelly criterion is the mathematically optimal approach: bet a fraction of your bankroll proportional to your edge.**


## Explanation

## How Much Should You Bet?

Position sizing is the most underrated skill in prediction market trading. Even with perfect edge detection, bad position sizing destroys returns. Bet too small and you leave money on the table. Bet too big and a few bad trades wipe you out.

### The Kelly Criterion

The Kelly criterion gives the mathematically optimal bet size:

**Kelly fraction = (bp - q) / b**

Where:
- b = odds received (payout / cost)
- p = your estimated probability of winning
- q = 1 - p (probability of losing)

For prediction markets, this simplifies to:

**Kelly fraction = (Your probability - Market price) / (1 - Market price)**

### Fractional Kelly

Full Kelly is too aggressive in practice because probability estimates are uncertain. Most professional bettors use "fractional Kelly" — typically 25-50% of the full Kelly amount. This sacrifices some expected return for dramatically reduced variance.

### Position Sizing in SimpleFunctions

The strategy engine lets you set position sizing through:
- `maxQuantity`: Hard cap on total position
- `perOrderQuantity`: How much to trade in each individual order
- Soft conditions can include portfolio concentration limits

### Practical Guidelines

1. **Never risk more than 5% of bankroll** on a single market
2. **Scale with conviction**: Higher-confidence edges get larger allocations
3. **Account for correlation**: Two recession-related positions are effectively one large bet
4. **Leave cash reserves**: At least 20% of bankroll should be undeployed for new opportunities


## Example

Your bankroll: $5,000
Your thesis: 50% probability on KXRECSSNBER-26
Market price: $0.30 (30%)

Full Kelly fraction = (0.50 - 0.30) / (1 - 0.30) = 0.286 (28.6%)
Full Kelly bet = $5,000 × 0.286 = $1,430

Half Kelly bet (recommended) = $715
At $0.30/contract = 2,383 contracts

But your max comfort position = 5% of bankroll = $250
At $0.30/contract = 833 contracts

Strategy config:
  maxQuantity: 833
  perOrderQuantity: 100
  entryBelow: 32


## CLI

```bash
sf strategies
```


## Related

[edge](edge.md), [risk-concentration](risk-concentration.md), [stop-loss](stop-loss.md), [take-profit](take-profit.md)
