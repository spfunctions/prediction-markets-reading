# Calibration

**Calibration measures how accurate probability estimates are over time. A well-calibrated forecaster or market has events priced at 70% actually occurring about 70% of the time.**


## Explanation

## What is Calibration?

Calibration answers the question: "When I say something is 70% likely, does it actually happen 70% of the time?"

### Perfect Calibration

A perfectly calibrated predictor has this property:
- Events estimated at 10% happen 10% of the time
- Events estimated at 50% happen 50% of the time
- Events estimated at 90% happen 90% of the time

This is plotted on a calibration chart where the x-axis is predicted probability and the y-axis is actual frequency. A perfectly calibrated predictor falls on the diagonal line.

### Common Calibration Errors

**Overconfidence**: Saying things are 90% likely when they only happen 75% of the time. This is the most common error for individual forecasters.

**Underconfidence**: Saying things are 50% likely when they happen 70% of the time. Prediction markets tend toward this — they slightly underweight extreme outcomes.

### Why Calibration Matters for Trading

If you know how a market is miscalibrated, you can exploit it:
- If Kalshi contracts at 85 cents settle Yes 92% of the time, there's systematic edge on the Yes side for high-probability contracts
- If your own estimates at 60% are correct 70% of the time, you're underconfident — you can adjust your sizing upward

### Tracking Your Calibration

Over time, SimpleFunctions tracks how your thesis-implied probabilities compare against actual outcomes. This calibration data helps you improve your estimation process and identify systematic biases.


## Example

Your calibration over the last 50 settled positions:

Predicted  Expected  Actual    Status
10-20%     15%       12%       ✓ Well calibrated
20-30%     25%       22%       ✓ Well calibrated
30-40%     35%       41%       ⚠ Slightly underconfident
40-50%     45%       52%       ⚠ Underconfident (+7pt)
50-60%     55%       58%       ✓ Close
60-70%     65%       71%       ⚠ Underconfident (+6pt)
70-80%     75%       73%       ✓ Close
80-90%     85%       80%       ⚠ Overconfident (-5pt)

Overall: Slight underconfidence in the 30-70% range.
Action: Consider sizing up positions in your "medium confidence" range.


## Related

[outcome-probability](outcome-probability.md), [confidence](confidence.md), [edge](edge.md), [thesis](thesis.md)
