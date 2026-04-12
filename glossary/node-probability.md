# Node Probability

**Node probability is the estimated likelihood assigned to a specific node in a causal tree. Changes to leaf node probabilities cascade upward through the tree, updating the root probability and all derived edge calculations.**


## Explanation

## How Node Probabilities Work

Every node in a causal tree has a probability — the estimated chance that the condition described by that node is or will be true.

### Leaf Nodes vs. Branch Nodes

**Leaf nodes** have probabilities you assign directly based on evidence:
- "Unemployment exceeds 5%" — you estimate 35% based on current trends

**Branch nodes** derive their probabilities from their children:
- If children are connected by AND logic: branch prob = child1 × child2
- If children are connected by OR logic: branch prob = 1 - (1-child1) × (1-child2)
- Many branches use weighted combinations

### Updating Node Probabilities

When new information arrives, the agent maps it to the relevant node and adjusts the probability. This update propagates:

1. New data: February unemployment rate comes in at 4.8% (up from 4.5%)
2. Maps to: node n1.1 (unemployment > 5%)
3. Update: n1.1 probability increases from 35% to 42%
4. Propagation: n1 (labor market deterioration) increases
5. Root update: Overall recession probability increases
6. Edge recalculation: Edge on KXRECSSNBER-26 widens

### The Power of Granular Updates

Without a causal tree, you'd have to answer: "How does a 0.3% increase in unemployment affect the overall probability of recession?" That's a hard, fuzzy question.

With a causal tree, you answer: "How does this data affect the probability of unemployment exceeding 5%?" That's much easier — and the math handles the rest.


## Example

Before February unemployment data:
  n1.1: Unemployment > 5%     p = 35%
  n1:   Labor deterioration    p = 50%
  Root: Recession              p = 38%
  Edge on KXRECSSNBER-26:      38% - 28% = +10pt

After data (unemployment = 4.8%, up from 4.5%):
  n1.1: Unemployment > 5%     p = 42%  (+7pt)
  n1:   Labor deterioration    p = 55%  (+5pt)
  Root: Recession              p = 42%  (+4pt)
  Edge on KXRECSSNBER-26:      42% - 29% = +13pt

  (Market moved from 28¢ to 29¢, but thesis moved more)


## CLI

```bash
sf thesis view --tree
```


## Related

[causal-tree](causal-tree.md), [thesis](thesis.md), [confidence](confidence.md), [evaluation-cycle](evaluation-cycle.md)
