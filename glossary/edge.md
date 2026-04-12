# Edge

**Edge is the difference between the market price of a prediction market contract and your estimated true probability. Positive edge means you believe the contract is mispriced in your favor.**


## Explanation

## Edge: The Core Concept

Edge is the single most important concept in prediction market trading. It's the answer to the question: "Is this contract priced wrong, and by how much?"

### Calculating Edge

Edge is simple arithmetic:

**Edge = Your Estimated Probability - Market Price**

If you believe there's a 45% chance of a recession and the Kalshi contract trades at 28 cents, your edge is +17 percentage points on the Yes side. If the contract were trading at 55 cents, your edge would be -10 points — meaning the No side has edge.

### Why Edge Matters

Without edge, you're gambling. With edge, you're investing. Every profitable prediction market strategy comes down to finding contracts where the market price diverges from reality, then systematically exploiting that divergence.

### Types of Edge

1. **Informational edge**: You have access to data or analysis the market hasn't priced in yet
2. **Analytical edge**: You model the event more accurately than the crowd (e.g., using causal trees)
3. **Structural edge**: You exploit market mechanics — cross-venue price differences, fee structures, or liquidity imbalances
4. **Temporal edge**: You react faster to new information than other participants

### Finding Edge Systematically

SimpleFunctions automates edge detection. The `sf edges` command scans hundreds of markets, compares current prices against your thesis-implied probabilities, and ranks opportunities by edge size and liquidity. This replaces the manual process of checking markets one by one.

### Edge Decay

Edge is perishable. Once other traders discover the same mispricing, they trade it away. The best edges exist in the window between new information arriving and the market fully pricing it in — often minutes to hours.


## Example

sf edges output for your recession thesis:

Market                          Price   Thesis  Edge    Liq
KXRECSSNBER-26                  $0.28   45%     +17pt   A
KXGDP-26Q2-TNEG                $0.18   30%     +12pt   B
KXUNRATE-26JUN-T5.0            $0.35   42%     +7pt    A
KXCPI-26MAR-T3.5               $0.52   48%     -4pt    A

Interpretation:
  KXRECSSNBER-26 has the largest edge (+17 points) with
  good liquidity (A-rated). This is the highest-priority
  trade opportunity.

  KXCPI contract has negative edge — the market agrees
  with your thesis or is more bullish than you.


## CLI

```bash
sf edges
```


## Related

[executable-edge](executable-edge.md), [thesis-implied-price](thesis-implied-price.md), [edge-detection](edge-detection.md), [spread](spread.md), [implied-return](implied-return.md)
