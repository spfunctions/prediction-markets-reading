# How to Backtest a Prediction Market Strategy

> Binary outcomes and clear settlement make prediction markets unusually good for backtesting. Here is how to build a calibration curve, avoid common pitfalls, and use settlement data to track realized returns.

**Category:** workflow | **Author:** SimpleFunctions | **Reading time:** 8 min

---
Prediction markets are unusually good for backtesting. In equities, you're testing against continuous price movements with ambiguous "was I right?" criteria. In prediction markets, every contract settles at exactly $1 or exactly $0. You were right or you were wrong. No interpretation needed.

This clarity makes it possible to build something most traders never build: a real calibration curve that tells you whether your confidence estimates actually mean anything.

## Why Settlement Data Is Gold

Every settled Kalshi contract gives you three data points:
1. The price history (what the market thought over time)
2. The settlement outcome (YES = $1 or NO = $0)
3. Your thesis confidence at each point in time (if you were tracking it)

With enough settlements, you can answer the most important question in trading: **when I say 70%, am I right 70% of the time?**

## Getting Settlement Data

```bash
# Pull all your settled positions
sf settlements --from 2025-01-01

# Output:
# Ticker           Side  Entry  Exit  Settlement  P&L    Thesis Conf.
# KXRECESSION-25   YES   32¢    -     NO ($0)     -$320  68%
# KXFEDRATE-25SEP  YES   45¢    -     YES ($1)    +$550  72%
# KXCPI-25JUN      NO    62¢    -     NO ($1)     +$380  55%
# ...
```

```bash
# Pull historical market data for any ticker (including settled)
sf history KXRECESSION-25

# Returns daily OHLC + volume + your thesis confidence snapshots
```

The `sf settlements` command pulls every contract where you had a position and it has since settled. The `sf history` command gives you the price timeline for any market, including the thesis confidence your agent assigned at each evaluation point.

## Building a Calibration Curve

A calibration curve answers: "When my model says X%, does the event happen X% of the time?"

Here's how to build one from your settlement data:

```python
import json
import subprocess
from collections import defaultdict

# Pull settlements via CLI
result = subprocess.run(
    ['sf', 'settlements', '--from', '2025-01-01', '--format', 'json'],
    capture_output=True, text=True
)
settlements = json.loads(result.stdout)

# Bucket by confidence level (round to nearest 10%)
buckets = defaultdict(lambda: {'total': 0, 'correct': 0})

for s in settlements:
    # Round thesis confidence to nearest 10%
    bucket = round(s['thesis_confidence'] * 10) * 10
    buckets[bucket]['total'] += 1

    # Was the thesis correct?
    if s['side'] == 'YES' and s['settlement'] == 1.0:
        buckets[bucket]['correct'] += 1
    elif s['side'] == 'NO' and s['settlement'] == 0.0:
        buckets[bucket]['correct'] += 1

print("Calibration Curve:")
print(f"{'Confidence':>12} {'Total':>6} {'Correct':>8} {'Actual %':>9}")
print("-" * 40)

for conf in sorted(buckets.keys()):
    b = buckets[conf]
    actual = (b['correct'] / b['total'] * 100) if b['total'] > 0 else 0
    bar = '█' * int(actual / 5)
    print(f"{conf:>10}%  {b['total']:>5}  {b['correct']:>7}  {actual:>7.1f}%  {bar}")
```

Example output:

```
Calibration Curve:
  Confidence  Total  Correct  Actual %
----------------------------------------
        30%      8        2     25.0%  █████
        40%     12        4     33.3%  ██████
        50%     15        8     53.3%  ██████████
        60%     18       12     66.7%  █████████████
        70%     14       10     71.4%  ██████████████
        80%      9        8     88.9%  █████████████████
        90%      4        4    100.0%  ████████████████████
```

This curve tells you: when your model says 70%, the event happens 71.4% of the time. That's almost perfectly calibrated. When it says 40%, events happen 33.3% of the time — your model is slightly overconfident at lower confidence levels.

**A perfectly calibrated model falls on the diagonal:** 30% confidence → 30% actual, 50% → 50%, 80% → 80%. Deviations from the diagonal tell you how to adjust.

## Tracking Realized Returns

Calibration tells you if your probabilities are right. Returns tell you if your trading is right. These are different things — you can be perfectly calibrated but still lose money if your position sizing is wrong or your entry timing is bad.

```bash
sf settlements --from 2025-01-01 --summary

# Output:
# Total positions:      80
# Win rate:             58.8%
# Total P&L:           +$2,340
# Average edge at entry: 8.2 points
# Average fill vs mid:   -1.3 points (slippage)
# Realized edge:         6.9 points
# Kelly fraction used:   0.35 (conservative)
```

The key metric is **realized edge** — the average difference between your thesis confidence and the settlement outcome, adjusted for entry price. If your realized edge is positive, you're genuinely identifying mispriced contracts. If it's negative, your thesis engine is worse than the market.

## The What-If Engine

Forward-looking analysis is harder than backtesting, but SimpleFunctions provides a what-if engine for scenario analysis:

```bash
sf whatif --thesis iran-war --node n1.2 --set 0.50

# Output:
# Scenario: n1.2 (Iran continues retaliation) set to 50% (currently 74%)
#
# Impact on thesis:
#   Overall confidence: 67% → 48%
#   Node n2 (Hormuz blocked): 97% → 71%
#   Node n3 (Oil elevated): 64% → 42%
#   Node n4 (Recession): 45% → 28%
#
# Impact on positions:
#   KXRECESSION-26: edge 10pts → edge -7pts (market 35¢, new thesis 28%)
#     → EXIT SIGNAL: thesis now below market price
#   KXOIL90-26Q3:   edge 23pts → edge 1pt (market 41¢, new thesis 42%)
#     → HOLD but edge nearly gone
```

This lets you stress-test your thesis before events happen. "What if diplomacy succeeds?" "What if oil drops below $70?" "What if the Fed cuts early?" For each scenario, you see exactly how your positions would be affected.

```bash
# Run multiple scenarios at once
sf whatif --thesis iran-war --scenarios scenarios.json
```

Where `scenarios.json`:

```json
[
  {
    "name": "Diplomatic breakthrough",
    "changes": { "n1.2": 0.30, "n1.3": 0.40 }
  },
  {
    "name": "Escalation",
    "changes": { "n1.2": 0.95, "n2": 0.99 }
  },
  {
    "name": "Oil supply shock",
    "changes": { "n3": 0.90 }
  }
]
```

This is forward-looking backtesting — you're not testing against historical outcomes but against hypothetical future states. It forces you to think about scenarios you might be ignoring.

## Pitfalls: What Will Fool You

### 1. Survivorship Bias in Market Selection

You remember your wins because you traded them. You forget the markets you looked at, decided not to trade, and turned out to be right about ignoring. When backtesting, you need to include *all* markets your system evaluated, not just the ones you traded.

```bash
# Include all evaluated markets, not just traded ones
sf settlements --from 2025-01-01 --include-passed

# This shows markets where your thesis had an opinion but you didn't trade
# because conditions weren't met. You can check: if you HAD traded, would
# you have made money?
```

If your "passed" trades would have performed better than your "traded" trades, your entry conditions are filtering wrong. You're trading the wrong subset of your good ideas.

### 2. Overfitting to Known Outcomes

The most dangerous backtest is one you run *after* you know the outcome. "My model would have predicted that CPI print!" Sure — because you unconsciously calibrated it to the number you already know.

The fix is to track your predictions *before* settlement. SimpleFunctions timestamps every thesis confidence snapshot. When you backtest, you're comparing your *at-the-time* confidence against the *later* settlement — not your *in-hindsight* confidence.

```bash
# Show confidence at time of each evaluation, not current confidence
sf history KXRECESSION-25 --thesis iran-war --show-snapshots

# Output shows your confidence over time:
# 2025-03-15  68%  (initial thesis)
# 2025-04-01  72%  (after CPI data)
# 2025-05-15  65%  (after diplomatic signal)
# 2025-06-30  58%  (thesis weakening)
# 2025-09-30  SETTLED: NO
```

Your thesis confidence declined from 68% to 58% before settlement — the model was learning. But you entered at 68% confidence. The backtest should use 68%, not the later 58%.

### 3. Small Sample Sizes

With 20 settled positions, your calibration curve is meaningless statistically. You need at least 50-100 settlements per confidence bucket to draw conclusions. At 10 trades per month, that's 5-10 months of data before your calibration curve stabilizes.

This is frustrating but real. Don't over-optimize based on small samples. Track the data, wait for statistical significance, then adjust.

### 4. Changing Market Regimes

A strategy that worked in low-volatility 2024 markets might fail in high-volatility 2026 markets. Prediction market microstructure — spreads, depth, participant behavior — changes over time. Your backtest on 2025 data might not predict 2026 performance.

The mitigation: backtest on rolling windows, not the full history. Compare performance in the most recent 3 months against the prior 3 months. If performance is degrading, your model or your strategy needs updating.

## Putting It All Together

Here's the complete backtest workflow:

```bash
# 1. Pull all settlements
sf settlements --from 2025-01-01 --format json > settlements.json

# 2. Pull all evaluations (including markets you didn't trade)
sf settlements --from 2025-01-01 --include-passed --format json > all_evals.json

# 3. Build calibration curve
python3 calibration.py settlements.json

# 4. Calculate realized returns
sf settlements --from 2025-01-01 --summary

# 5. Run what-if scenarios on current positions
sf whatif --thesis iran-war --scenarios scenarios.json

# 6. Compare against passed-on trades
python3 compare_traded_vs_passed.py settlements.json all_evals.json
```

The goal of backtesting isn't to prove your strategy works. It's to find out *where* it breaks. Every strategy has failure modes. The ones you discover in backtesting cost you nothing. The ones you discover in live trading cost you money.

Prediction markets give you the cleanest possible backtesting environment: binary outcomes, exact settlement, timestamped confidence snapshots. Use it. Most prediction market traders never look at their own track record. The ones who do — and who adjust based on what they find — are the ones who compound returns over years instead of blowing up in months.