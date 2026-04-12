# Stop Loss

**A stop loss is a pre-defined exit condition that closes your position when the price moves against you past a threshold. In prediction markets, stops can be based on price, thesis confidence, or causal node changes.**


## Explanation

## Stop Losses in Prediction Markets

A stop loss protects you from catastrophic losses. Instead of holding a losing position all the way to zero, you define conditions under which you'll exit and cut your losses.

### Types of Stops

1. **Price-based stop**: Exit if the contract price drops below X cents. Simple but can be triggered by temporary noise.
2. **Thesis-based stop**: Exit if your thesis confidence drops below a threshold. More intelligent — only exits when your analysis changes.
3. **Node-based stop**: Exit if a specific causal tree node drops below a probability threshold. The most granular approach.

### Setting Stops in SimpleFunctions

The strategy engine supports stop losses through the `stopLoss` parameter and soft conditions:

- `stopLoss: 20` — exit if the bid drops to 20 cents or below
- Soft condition: "Exit if thesis confidence drops below 40%"
- Soft condition: "Exit if node n1 (war persists) drops below 50%"

### The Problem with Manual Stops

Human traders are terrible at executing stops. When a position drops, the emotional impulse is to hold and hope for recovery. Studies show traders hold losing positions 2x longer than winning ones (the disposition effect). Automated stops remove this bias.

### Stop Loss Sizing

A common approach: set your stop so that if it triggers, you lose no more than 2-3% of your total bankroll. Work backward from this to determine position size.

If your bankroll is $5,000 and max acceptable loss is $150 (3%):
- Contract at 40 cents with stop at 30 cents = 10 cents max loss per contract
- Maximum position: $150 / $0.10 = 1,500 contracts


## Example

Strategy for KXRECSSNBER-26:
  Entry: Buy at $0.28 or below
  Target: $0.45 (thesis-implied fair value)
  Stop loss: $0.20

  Risk per contract: $0.28 - $0.20 = $0.08
  Reward per contract: $0.45 - $0.28 = $0.17
  Risk/Reward ratio: 1:2.1

  With 200 contracts:
    Max loss at stop: 200 × $0.08 = $16.00
    Max profit at target: 200 × $0.17 = $34.00


## CLI

```bash
sf strategies
```


## Related

[take-profit](take-profit.md), [position-sizing](position-sizing.md), [risk-concentration](risk-concentration.md), [cost-basis](cost-basis.md)
