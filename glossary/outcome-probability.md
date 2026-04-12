# Outcome Probability

**In a prediction market, the price of a contract directly represents the market's consensus estimate of the probability of that outcome occurring. A contract at $0.72 implies a 72% probability.**


## Explanation

## Price as Probability

The fundamental insight of prediction markets is that contract prices are probabilities. A "Yes" contract trading at 72 cents means the market collectively estimates a 72% chance that the event will occur.

### Why This Works

Imagine a contract is trading at 40 cents but the true probability is 60%. A rational trader would buy at 40 cents, expecting an average return of 60 cents — a 50% expected profit. Buying pressure would push the price up toward 60 cents until the opportunity disappears. This arbitrage mechanism keeps prices close to true probabilities.

### Reading Probability from Markets

When you run `sf scan` and see a Kalshi contract at 0.35, that's the market saying "35% chance." But the market can be wrong — and that's where edge comes from. If your analysis says the true probability is 50%, the market is mispricing the contract by 15 percentage points.

### Probability vs. Confidence

Don't confuse market probability with certainty. A contract at 90 cents is not "certain" — it means there's still a 10% chance of failure. Historically, contracts at 90 cents resolve "No" about 10% of the time, which is exactly what the price predicts.

### Calibration

A well-functioning prediction market is "calibrated" — contracts that trade at 70 cents should resolve Yes approximately 70% of the time. Both Kalshi and Polymarket have been shown to be well-calibrated across thousands of settled contracts.


## Example

Kalshi event: "Will CPI exceed 3.5% in March 2026?"

  Yes price: $0.23 → Market says 23% probability
  No price:  $0.77 → Market says 77% probability
  Sum:       $1.00 (always)

Your analysis:
  You estimate 35% probability based on PPI data and Fed comments.
  Edge = 35% - 23% = +12 percentage points on the Yes side.
  This is a potential buy signal.


## CLI

```bash
sf scan --min-edge 5
```


## Related

[prediction-market](prediction-market.md), [edge](edge.md), [calibration](calibration.md), [binary-contract](binary-contract.md)
