# What-If Analysis

**What-if analysis (scenario analysis) lets you explore how your thesis and edge calculations would change under different assumptions by temporarily modifying causal tree node probabilities.**


## Explanation

## Scenario Planning with What-If Analysis

What-if analysis answers questions like: "If unemployment spikes to 6%, how does my thesis change?" or "What happens to my portfolio if diplomatic talks succeed?"

### How It Works

You temporarily override one or more node probabilities and see how it affects:
1. The root probability (thesis-implied price)
2. Edge calculations across all linked markets
3. Strategy trigger conditions
4. Portfolio-level P&L

### Use Cases

**Pre-event planning**: Before a CPI release, run scenarios for CPI = 3.0%, 3.5%, and 4.0%. Pre-plan your trades for each outcome.

**Risk assessment**: "If my most uncertain node (n3, Fed policy error) turns out to be 0%, what happens to my portfolio?"

**Position stress testing**: "What's my worst-case P&L if nodes n1 AND n2 both go to 0%?"

### What-If in SimpleFunctions

The `sf what-if` command lets you run scenarios interactively:

```
sf what-if --node n1.1 --prob 0.80
```

This temporarily sets "unemployment > 5%" to 80% and shows you the cascading effect on every linked market and strategy.

### Systematic Scenario Generation

For important events, the agent can auto-generate scenarios based on historical data ranges and show you the full spectrum of outcomes. This is especially useful before FOMC meetings, jobs reports, and CPI releases.


## Example

sf what-if --node n1.1 --prob 0.80

Scenario: "Unemployment will exceed 5%" at 80% (current: 42%)

Impact cascade:
  n1:   Labor deterioration    50% → 68%  (+18pt)
  Root: Recession              42% → 55%  (+13pt)

Market impact:
  KXRECSSNBER-26    Edge: +13pt → +26pt  (significantly underpriced)
  KXGDP-26Q2-TNEG   Edge: +8pt  → +18pt
  KXUNRATE-26JUN-T5  Edge: +5pt → +30pt  (now the #1 opportunity)

Strategy triggers:
  ⚠ Strategy S-002 would trigger (edge > 15pt threshold)
  ⚠ Strategy S-004 would trigger (n1 > 65% condition)


## CLI

```bash
sf what-if
```


## Related

[causal-tree](causal-tree.md), [node-probability](node-probability.md), [thesis](thesis.md), [risk-concentration](risk-concentration.md)
