# Thesis Confidence Score

**Confidence is a 0-100% score that measures how certain you are about your probability estimates, not the probability itself. High confidence means your estimates are well-researched; low confidence means they're speculative.**


## Explanation

## Confidence vs. Probability

This distinction trips up new traders constantly:

- **Probability**: "I think there's a 40% chance of recession"
- **Confidence**: "I'm 75% sure that my 40% estimate is close to correct"

You can have high confidence in a low probability ("I'm very sure this is unlikely") or low confidence in a high probability ("I think this is likely but I haven't done enough research").

### Why Confidence Matters

Confidence should drive position sizing, not probability. A 40% probability with 90% confidence warrants a larger position than a 60% probability with 30% confidence.

### Confidence in SimpleFunctions

Each causal tree node has both a probability and a confidence score:

- **High confidence (>70%)**: The node probability is based on hard data (official statistics, confirmed events). The agent sizes positions larger.
- **Medium confidence (40-70%)**: Reasonable estimates based on analysis, but could shift with new information.
- **Low confidence (<40%)**: Speculative estimates. The agent flags these nodes for additional research and reduces position sizes.

### How Confidence Changes

Confidence increases when:
- Official data confirms your estimate
- Multiple independent sources agree
- Historical base rates support your number

Confidence decreases when:
- New contradictory information emerges
- Your estimate is based on a single source
- The situation is genuinely unprecedented (no base rate)

The evaluation cycle updates both probabilities and confidence scores when new signals arrive.


## Example

Causal tree nodes with confidence:

n1: Labor market deterioration
  Probability: 55%
  Confidence:  72%
  Basis: BLS data, weekly jobless claims, JOLTS

n2: Consumer spending decline
  Probability: 45%
  Confidence:  70%
  Basis: Retail sales data, credit card spending

n3: Fed policy error
  Probability: 30%
  Confidence:  45%  ← LOW CONFIDENCE
  Basis: Speculation about Fed reaction function

Action: Node n3 has low confidence. The agent recommends
researching Fed communication more before sizing positions
that depend on this branch.


## CLI

```bash
sf thesis view
```


## Related

[thesis](thesis.md), [causal-tree](causal-tree.md), [node-probability](node-probability.md), [evaluation-cycle](evaluation-cycle.md)
