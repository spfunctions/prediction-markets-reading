# Causal Tree

**A causal tree is a structured probabilistic model that breaks down a complex event into a hierarchy of independent, verifiable sub-conditions. Each node has a probability, and the tree computes the overall probability of the root event.**


## Explanation

## Causal Trees: SimpleFunctions' Core Analytical Framework

A causal tree is how SimpleFunctions thinks about prediction markets. Instead of asking "what's the probability of X?" directly, we decompose the question into smaller, more answerable components.

### Structure

A causal tree has:
- **Root node**: The ultimate question (e.g., "Will Iran-Israel conflict escalate to direct confrontation?")
- **Branch nodes**: Conditions that contribute to the root outcome
- **Leaf nodes**: Observable, verifiable facts that can be tracked

### Example: Iran Conflict Thesis

```
Root: Iran-Israel direct confrontation (p = 35%)
├── n1: Iran retaliates beyond proxies (p = 45%)
│   ├── n1.1: Missile/drone strike on Israeli territory (p = 60%)
│   └── n1.2: Naval confrontation in Strait of Hormuz (p = 30%)
├── n2: US military involvement (p = 25%)
│   ├── n2.1: US bases in region attacked (p = 40%)
│   └── n2.2: Congressional authorization (p = 15%)
└── n3: Diplomatic channels fail (p = 55%)
    ├── n3.1: UN Security Council deadlocked (p = 70%)
    └── n3.2: Back-channel negotiations collapse (p = 45%)
```

### Why Trees Beat Single Estimates

1. **Decomposition makes estimation easier**: Estimating "probability of WWIII" is impossible. Estimating "probability of UN Security Council deadlock on Iran resolution" is tractable.
2. **Information maps to nodes**: A headline about Iranian missile tests maps directly to node n1.1. Without the tree, you'd have to reason about how it affects the overall probability — with the tree, you update one node and the math propagates.
3. **Confidence tracking**: Each node has both a probability and a confidence level. If n3.2 has low confidence, you know where to focus research.

### The Agent Uses Trees for Everything

When the heartbeat service runs, it:
1. Scans for new information (news, price changes)
2. Maps each piece to a causal node
3. Updates node probabilities
4. Recomputes the root probability
5. Compares against market prices to find edge
6. Triggers strategy actions if conditions are met


## Example

sf thesis view --tree

Thesis: "US enters recession by end of 2026"
Root probability: 42%

Causal tree:
n1: Labor market deterioration        p=55%  conf=72%
  n1.1: Unemployment > 5%             p=35%  conf=68%
  n1.2: Non-farm payrolls < 100K/mo   p=40%  conf=65%
n2: Consumer spending decline          p=45%  conf=70%
  n2.1: Real retail sales negative     p=38%  conf=75%
  n2.2: Consumer confidence < 80       p=52%  conf=60%
n3: Fed policy error                   p=30%  conf=55%
  n3.1: Overtightening (rates > 5.5%)  p=20%  conf=50%
  n3.2: Too late to cut                p=40%  conf=58%

Root = weighted combination of branches
Edge on KXRECSSNBER-26: 42% thesis - 28% market = +14pt


## CLI

```bash
sf thesis view --tree
```


## Related

[thesis](thesis.md), [node-probability](node-probability.md), [confidence](confidence.md), [thesis-implied-price](thesis-implied-price.md), [edge-detection](edge-detection.md)
