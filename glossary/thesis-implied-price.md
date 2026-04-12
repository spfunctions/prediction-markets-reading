# Thesis-Implied Price

**The thesis-implied price is the probability a contract should trade at according to your causal tree model. The difference between thesis-implied price and market price is your edge.**


## Explanation

## How Thesis-Implied Pricing Works

When your causal tree computes a root probability of 42%, that means your model says contracts linked to this event should trade at 42 cents. If the market says 28 cents, you have 14 points of edge.

### From Tree to Price

The thesis-implied price comes from the mathematical combination of node probabilities in your causal tree:

1. Each leaf node has a probability you've estimated
2. Branch nodes combine their children (using AND, OR, or weighted logic)
3. The root node's probability is the thesis-implied price

### Multiple Markets, One Thesis

A single thesis can imply prices across multiple contracts. A recession thesis might simultaneously imply:
- KXRECSSNBER-26 should be at 42 cents (vs. market 28)
- KXGDP-26Q2-TNEG should be at 30 cents (vs. market 18)
- KXUNRATE-26JUN-T5.0 should be at 40 cents (vs. market 35)

Each gap is a separate edge opportunity, all driven by the same underlying analysis.

### Updating Implied Prices

When new information arrives and you update a causal node, the thesis-implied price automatically changes. If a strong jobs report drops unemployment expectations, node n1.1 probability decreases, which decreases the root probability, which decreases the thesis-implied price for recession contracts.

This cascade is automatic in SimpleFunctions — the agent updates the tree and immediately recomputes edge across all linked markets.


## Example

Thesis: "Iran-Israel direct confrontation"
Root probability: 35%

Linked markets and implied prices:
  KXIRAN-STRIKE-26    Thesis: 35¢  Market: 22¢  Edge: +13pt
  KXOIL-WTI-T120      Thesis: 28¢  Market: 15¢  Edge: +13pt
  KXHORMUZ-CLOSE-26   Thesis: 20¢  Market: 08¢  Edge: +12pt

If node n3 (diplomacy fails) drops from 55% → 35%:
  Root probability drops: 35% → 27%
  New implied prices recalculated automatically
  Some edges shrink or disappear


## CLI

```bash
sf edges
```


## Related

[causal-tree](causal-tree.md), [edge](edge.md), [thesis](thesis.md), [node-probability](node-probability.md)
