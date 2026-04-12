# Implied Return

**Implied return is the expected profit as a percentage of capital invested, calculated from the edge and the contract price. It converts edge points into a return metric traders can compare across opportunities.**


## Explanation

## From Edge to Returns

Edge tells you how many cents of mispricing exist. Implied return tells you how profitable that mispricing is relative to your capital deployed.

### Calculating Implied Return

**Implied return = Edge / Entry Price**

If your thesis says a contract should be at 45 cents and it's trading at 30 cents:
- Edge = 15 points
- Entry price = 30 cents
- Implied return = 15 / 30 = 50%

This means if your thesis is correct, you expect a 50% return on invested capital.

### Why This Matters

Two trades can have the same edge but very different returns:
- Trade A: Edge = 10pt, entry at 20 cents → Return = 50%
- Trade B: Edge = 10pt, entry at 80 cents → Return = 12.5%

Trade A is 4x more capital-efficient. If you have limited capital (most traders do), implied return should influence which opportunities you prioritize.

### Risk-Adjusted Returns

Implied return doesn't account for the probability of being wrong. A more complete metric is:

**Expected return = (thesis_prob × (1 - entry_price) - (1 - thesis_prob) × entry_price) / entry_price**

This weights the upside by the probability of winning and the downside by the probability of losing.

### Comparing Across Markets

The `sf edges` output can be sorted by implied return instead of raw edge, giving you a capital-efficiency perspective on your opportunities.


## Example

Three opportunities with different return profiles:

Market            Entry  Thesis  Edge   Implied Return
KXRECSSNBER-26    $0.28  42%     +14pt  50.0%
KXGDP-26Q2-T2.0  $0.62  72%     +10pt  16.1%
KXWTIMAX-T150     $0.05  12%     +7pt   140.0%

By edge alone:   KXRECSSNBER-26 wins (14pt)
By return alone: KXWTIMAX-T150 wins (140%)
By expected value: KXRECSSNBER-26 likely still wins
  (higher confidence, better liquidity)

Risk-adjusted expected return for KXRECSSNBER-26:
  (0.42 × $0.72 - 0.58 × $0.28) / $0.28 = 50% expected return


## CLI

```bash
sf edges --sort return
```


## Related

[edge](edge.md), [position-sizing](position-sizing.md), [cost-basis](cost-basis.md), [executable-edge](executable-edge.md)
