# Thesis

**A thesis is a structured investment argument that combines a directional view, a causal model of why the view is correct, and specific market positions that express the view. In SimpleFunctions, a thesis is the core unit of analysis.**


## Explanation

## What is a Trading Thesis?

A thesis is more than an opinion. It's a complete, testable argument about the future that includes:

1. **The claim**: What you believe will happen (e.g., "The US will enter a recession by end of 2026")
2. **The reasoning**: Why you believe it, structured as a causal tree
3. **The evidence**: What data supports each branch of the causal tree
4. **The market expression**: Which contracts are mispriced if your thesis is correct
5. **The invalidation conditions**: What would prove you wrong

### Thesis Lifecycle in SimpleFunctions

```
forming → active → paused → closed
```

- **Forming**: You're building the causal tree and scanning for markets
- **Active**: The agent is monitoring, evaluating signals, and executing strategies
- **Paused**: Monitoring continues but no new trades are placed
- **Closed**: The thesis reached its conclusion (right or wrong)

### Creating a Thesis

When you run `sf thesis create`, the agent guides you through:
1. Stating your thesis in plain language
2. Building a causal tree by decomposing the claim
3. Assigning initial probabilities to each node
4. Scanning for markets that express the thesis
5. Identifying edge opportunities
6. Defining entry/exit strategies

### Why Theses Matter

Without a thesis, you're just looking at contract prices and guessing. With a thesis, every trade has a reason, every price change maps to a causal node, and every exit has a pre-defined condition. The thesis is your decision-making framework — the agent is just the execution layer.


## Example

sf thesis create "US recession by end of 2026"

Agent output:
  ✓ Thesis registered
  ✓ Causal tree generated (3 main branches, 8 leaf nodes)
  ✓ Market scan complete: 12 related contracts found
  ✓ Edge analysis: 4 contracts with >5pt executable edge

  Top opportunity:
    KXRECSSNBER-26 — Market: 28% | Thesis: 42% | Edge: +14pt | Liq: A

  Recommended strategy:
    Buy Yes below $0.30, stop at $0.18, target $0.45
    Max position: 500 contracts ($150 max risk)


## CLI

```bash
sf thesis create
```


## Related

[causal-tree](causal-tree.md), [edge](edge.md), [thesis-implied-price](thesis-implied-price.md), [confidence](confidence.md), [evaluation-cycle](evaluation-cycle.md)
