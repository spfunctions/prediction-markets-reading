# Risk Concentration

**Risk concentration measures how much of your portfolio depends on the same underlying thesis or event. Correlated positions amplify both gains and losses, creating hidden portfolio risk.**


## Explanation

## The Hidden Risk in Prediction Markets

New traders often think they're diversified because they hold five different contracts. But if all five are recession-related, they're really holding one giant bet:

- KXRECSSNBER-26 (recession declaration)
- KXGDP-26Q2-TNEG (negative GDP)
- KXUNRATE-26JUN-T5.0 (high unemployment)
- KXCPI-26MAR-T3.5 (high inflation)
- KXFEDRATE-26JUN (rate cut)

If the economy stays strong, all five positions lose simultaneously. That's not five independent bets — it's one concentrated macro bet.

### Measuring Concentration

SimpleFunctions tracks risk concentration at two levels:

1. **Thesis level**: How much capital is deployed on a single thesis?
2. **Node level**: How much capital depends on a specific causal node?

If node n1 (labor market deterioration) drives 80% of your portfolio's expected value, and new data invalidates n1, you lose 80% of your expected profits in one event.

### Managing Concentration

Rules of thumb:
- No single thesis should represent more than 30% of deployed capital
- No single causal node should drive more than 20% of portfolio expected value
- Uncorrelated theses provide better risk-adjusted returns

### Concentration vs. Conviction

High conviction in a thesis is fine — but express it through position sizing on your best edge, not through adding every marginally related contract. Five contracts with 14-point edge each are worse than two contracts with 14-point edge and two uncorrelated contracts with 8-point edge.


## Example

sf portfolio risk

Portfolio: $2,500 deployed across 8 positions

Risk concentration analysis:
  Thesis: "Recession 2026"    Exposure: $1,800 (72%)  ← TOO HIGH
  Thesis: "Iran escalation"    Exposure: $500 (20%)   ✓
  Thesis: "Fed policy shift"   Exposure: $200 (8%)    ✓

Causal node concentration:
  n1 (labor deterioration)     Drives: 55% of portfolio EV ← HIGH
  n2 (consumer spending)       Drives: 25% of portfolio EV
  n3 (Fed error)               Drives: 12% of portfolio EV

Recommendation:
  ⚠ Reduce recession thesis to <50% of portfolio
  ⚠ Consider hedging n1 exposure with a counter-position


## CLI

```bash
sf portfolio risk
```


## Related

[position-sizing](position-sizing.md), [causal-tree](causal-tree.md), [thesis](thesis.md), [stop-loss](stop-loss.md)
