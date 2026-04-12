# How Causal Tree Decomposition Works in Prediction Market Trading

> The core methodology behind structured prediction market analysis: decompose a thesis into a tree of testable sub-claims, assign probabilities, propagate them, and find where the market disagrees with your model.

**Category:** patterns | **Author:** SimpleFunctions | **Reading time:** 9 min

---
Most prediction market traders operate on a single number: "I think recession is 50%." They buy at 35¢ and hope. No decomposition, no structure, no way to update when new information arrives.

Causal tree decomposition replaces that single number with a structured model. Instead of "recession is 50%," you get a tree of testable sub-claims, each with its own probability, each mappable to real-world evidence, each connected to specific market contracts.

This article explains exactly how it works, step by step, using a real thesis as the running example.

## The Core Idea

A causal tree takes a complex prediction ("US enters recession in 2026") and breaks it into a chain of upstream causes. Each cause is a node in the tree. Each node has:

1. **A testable claim** — something that can be verified against real-world data
2. **A probability estimate** — how likely this claim is to be true
3. **Children** — sub-claims that support or condition this node
4. **A mapping** — which prediction market contracts correspond to this node

The thesis-implied probability of the terminal node (recession) is computed by propagating probabilities through the tree. The edge is the gap between this computed probability and the market price.

## Building the Tree: Iran War Example

Let's start with a directional view: "The US-Iran conflict will persist, keeping the Strait of Hormuz blocked, oil prices elevated, and the US economy pushed into recession."

This is a causal chain. Each link is a testable claim. Here's how it decomposes:

```
n1    War persists                                   0.88
├── n1.1  US initiated strikes                       0.99
├── n1.2  Iran continues retaliation                 0.85
│   ├── n1.2.1  IRGC has operational capacity        0.95
│   └── n1.2.2  Domestic pressure to respond         0.89
└── n1.3  No diplomatic exit                         0.82
    ├── n1.3.1  US preconditions unacceptable         0.78
    └── n1.3.2  Iran internal politics block deal     0.85

n2    Hormuz blocked                                 0.97
├── n2.1  Mines deployed (confirmed by CENTCOM)      0.99
├── n2.2  IRGC small craft active                    0.90
└── n2.3  Minesweeping takes > 3 months              0.87

n3    Oil stays elevated (>$100/bbl)                 0.64
├── n3.1  SPR drawdown insufficient                  0.95
├── n3.2  OPEC doesn't compensate                    0.68
└── n3.3  No alternative routing at scale            0.71

n4    Fed policy trapped                             0.58
├── n4.1  Inflation above 5% due to oil              0.72
└── n4.2  Rate cuts would worsen inflation           0.80

n5    Recession                                      0.45
├── n5.1  Consumer spending contracts                0.62
└── n5.2  Business investment freezes                0.58
```

### What Makes a Good Node

Each node must satisfy three criteria:

1. **Testable**: You can point to specific data that would confirm or deny it. Node n2.1 ("mines deployed") is testable because CENTCOM publishes mine clearing reports. Node "things feel bad" is not testable.

2. **Independent at the sibling level**: Sibling nodes (children of the same parent) should be as independent as possible. n1.1 (US strikes) and n1.2 (Iran retaliates) are somewhat dependent, but n1.2.1 (IRGC capacity) and n1.2.2 (domestic pressure) are largely independent drivers.

3. **Observable**: There must be a realistic way to monitor this node over time. Tavily news searches, government data releases, satellite imagery reports, API data. If you can't observe it, you can't update it.

## Probability Assignment

Each leaf node gets a probability from the LLM evaluator, grounded in current evidence. The LLM sees:

- The claim text
- Recent news articles related to the claim (via Tavily)
- Current market prices for related contracts
- Historical context

The evaluator doesn't just output a number. It outputs a structured assessment:

```json
{
  "nodeId": "n3.1",
  "claim": "SPR drawdown insufficient to offset Hormuz closure",
  "probability": 0.95,
  "evidence": [
    "SPR at 347M barrels as of March 2026 (DOE weekly report)",
    "Hormuz closure removes ~17M bbl/day, SPR max release is 4.4M bbl/day",
    "At max drawdown rate, SPR lasts ~79 days"
  ],
  "uncertaintyDrivers": [
    "Congress could authorize emergency purchases",
    "IEA coordinated release could supplement"
  ]
}
```

This structure matters because it makes the probability *auditable*. You can look at n3.1 = 0.95 and ask "why?" and get a concrete answer with citations.

## Propagation: How Probabilities Flow

Once every leaf node has a probability, the tree propagates upward. The propagation rule depends on the relationship between children and parent.

### Conjunctive nodes (AND relationship)

When a parent requires ALL children to be true, the parent probability is bounded by the product of independent children:

```
P(n1) = P(n1.1) * P(n1.2) * P(n1.3)
       = 0.99 * 0.85 * 0.82
       = 0.69
```

But wait — the tree shows n1 at 0.88, not 0.69. That's because the children aren't purely conjunctive. The LLM evaluator assigns n1 considering that even if one child weakens, war can persist for other reasons. The propagation uses a weighted combination, not a strict product.

The actual formula used:

```
P(parent) = w_independent * P_independent + w_conditional * P_conditional
```

Where `P_independent` is the LLM's direct assessment of the parent claim, and `P_conditional` is computed from the children. The weights are set by the tree structure: for tightly coupled chains, `w_conditional` dominates; for loosely coupled claims, `w_independent` gets more weight.

### Sequential chain nodes

For the main thesis chain (n1 → n2 → n3 → n4 → n5), each node is conditional on its predecessors:

```
P(recession) = P(n1) * P(n2|n1) * P(n3|n2) * P(n4|n3) * P(n5|n4)
             = 0.88 * 0.97 * 0.64 * 0.58 * 0.45
             ≈ 0.142
```

This gives 14.2% — much lower than the 45% shown for n5. Why? Because n5's probability (0.45) already *incorporates* the upstream conditioning. It represents "probability of recession given oil stays elevated and the Fed is trapped," not the unconditional probability from scratch.

The thesis-implied probability of recession (the number compared against the market) is n5's value: 0.45 or 45%. This represents the model's best estimate given all upstream conditions and their current states.

## The Ripple Effect: What Happens When a Node Changes

This is where the tree becomes powerful. Let's say new intelligence suggests that the SPR drawdown is actually working better than expected. Node n3.1 changes from 0.95 to 0.30.

Here's what ripples through:

**Before the change:**
```
n3.1  SPR insufficient         0.95
n3    Oil stays elevated        0.64
n5    Recession                 0.45
```

**After n3.1 drops to 0.30:**
```
n3.1  SPR insufficient         0.30   ← direct change
n3    Oil stays elevated        0.31   ← cascading update (oil thesis weakens dramatically)
n4    Fed trapped               0.35   ← less oil pressure = less inflation trap
n5    Recession                 0.19   ← downstream collapse
```

The recession probability dropped from 45% to 19%. The market contract is trading at 35¢. Your thesis now says the market is *overpriced* — the edge flipped from +10 points (buy) to -16 points (sell or avoid).

One data point — the SPR drawdown rate — cascaded through four levels of the tree and reversed your position. Without the tree, you'd be staring at a headline ("SPR drawdown exceeds expectations") and guessing whether it matters. With the tree, the impact is computed automatically.

## Mapping Nodes to Market Contracts

Not every node in the tree has a corresponding prediction market contract. But many do:

| Node | Claim | Kalshi/Poly Contract | Market Price |
|------|-------|---------------------|:------------:|
| n1 | War persists | KXIRANWAR-26 | 84¢ |
| n2 | Hormuz blocked | (Polymarket Hormuz) | 82¢ |
| n3 | Oil > $100 | KXOIL100-26Q3 | 41¢ |
| n5 | Recession | KXRECSSNBER-26 | 35¢ |

The edge for each contract is:

```
KXIRANWAR-26:     thesis 0.88 vs market 0.84 = +4¢ edge (small)
Hormuz:           thesis 0.97 vs market 0.82 = +15¢ edge (large)
KXOIL100-26Q3:    thesis 0.64 vs market 0.41 = +23¢ edge (largest)
KXRECSSNBER-26:   thesis 0.45 vs market 0.35 = +10¢ edge (moderate)
```

The tree reveals that the biggest mispricing isn't on the recession contract — it's on the oil contract and the Hormuz contract. These are upstream nodes with fewer dependencies and higher confidence.

## Why This Beats a Single Probability Estimate

A single estimate ("recession is 50%") gives you no way to:

1. **Update incrementally** — when news hits, which part of your thesis does it affect?
2. **Identify the weakest link** — which assumption is most uncertain?
3. **Find the best trade** — which contract in the causal chain has the most edge?
4. **Audit your reasoning** — why do you believe 50% and not 40%?

The causal tree answers all four. It turns a gut feeling into an auditable, updatable, tradeable model.

## Practical Limitations

The tree is not magic. Several real limitations:

**Probability calibration**: The LLM's probability estimates are only as good as its training data and the evidence provided. Systematic bias is possible — LLMs tend to be overconfident on claims with lots of supporting evidence and underconfident on novel scenarios.

**Structural assumptions**: The tree structure itself embeds assumptions. Choosing to make n3 depend on n2 (oil depends on Hormuz) assumes that's the primary channel. If oil rises for unrelated reasons (OPEC cuts, China demand surge), the tree won't capture it unless you add those as separate branches under n3.

**Correlation**: Sibling nodes are assumed to be roughly independent, but in practice they often aren't. n1.2 (Iran retaliates) and n1.3 (no diplomatic exit) are correlated — if Iran escalates, diplomacy becomes less likely. The tree handles this imperfectly.

**Update frequency**: The tree is re-evaluated on a schedule (every 60 minutes in normal mode, 30 minutes during high volatility). Between evaluations, the tree is stale. For fast-moving events, this lag can matter.

Despite these limitations, a structured tree consistently outperforms unstructured estimation. The decomposition forces you to think about *why* you believe something, and the propagation ensures your downstream estimates are consistent with your upstream assumptions.

## Getting Started with Causal Trees

To build your first causal tree:

1. `sf create "your thesis statement"` — the LLM decomposes your thesis automatically
2. Review each node — do you agree with the claims and probabilities?
3. Adjust nodes where your expertise exceeds the LLM's
4. `sf edges` — see where the market disagrees with your model
5. `sf monitor` — watch the tree update as new information arrives

The tree is a living document. It changes as the world changes. That's the point — it's not a prediction, it's a framework for updating predictions systematically.