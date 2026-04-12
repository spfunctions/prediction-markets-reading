# Cross-Venue Analysis

**Cross-venue analysis compares pricing of equivalent events across different prediction market platforms (like Kalshi and Polymarket) to find arbitrage opportunities or the best execution venue.**


## Explanation

## Why Cross-Venue Matters

The same event can trade at different prices on different platforms. This happens because:
- Different user bases have different information and biases
- Liquidity varies by platform
- Fee structures differ
- Regulatory differences affect participation

### Kalshi vs. Polymarket

The two largest prediction markets serve different audiences:
- **Kalshi**: US-regulated, attracts institutional and data-driven traders, strong in economic markets
- **Polymarket**: Crypto-based, attracts crypto-native and political traders, strong in political markets

Because the audiences differ, prices can diverge — especially for complex or niche events.

### Types of Cross-Venue Opportunities

1. **Pure arbitrage**: The same event is priced at 30 cents on Kalshi and 40 cents on Polymarket. Buy on Kalshi, sell on Polymarket, lock in a 10-cent risk-free profit.
2. **Best execution**: Your thesis has 12 points of edge on Kalshi but 15 points on Polymarket. Trade on the venue with more edge.
3. **Liquidity comparison**: A market is illiquid on Kalshi but liquid on Polymarket (or vice versa).

### Challenges

Cross-venue trading has friction:
- Capital is locked on each platform separately
- Fee structures differ (Kalshi has per-contract fees, Polymarket has different mechanisms)
- Settlement timing can differ
- You need accounts on both platforms

### Cross-Venue in SimpleFunctions

The `sf cross-venue` command automatically matches equivalent contracts across platforms and shows price differentials, liquidity comparisons, and net-of-fees edge.


## Example

sf cross-venue --thesis recession-2026

Same-event comparison:
Event: "US enters recession by end of 2026"

             Kalshi          Polymarket      Diff
Price:       $0.28           $0.35           7¢
Spread:      3¢              2¢
Volume 24h:  3,200           8,400
Depth:       2,500           12,000
Fee:         ~2¢/contract    ~0.5¢

Your thesis: 42%
Edge (Kalshi):     42-28 = +14pt, net of fees: ~12pt
Edge (Polymarket): 42-35 = +7pt,  net of fees: ~6.5pt

Recommendation: Kalshi has better edge despite lower liquidity.
  But if size >500 contracts, Polymarket may provide better fills.


## CLI

```bash
sf cross-venue
```


## Related

[edge-detection](edge-detection.md), [edge](edge.md), [liquidity](liquidity.md), [executable-edge](executable-edge.md)
