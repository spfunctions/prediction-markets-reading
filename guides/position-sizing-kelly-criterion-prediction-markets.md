# Position Sizing for Prediction Markets: Kelly Criterion Meets Causal Models

> Prediction market contracts have a $1 cap, binary settlement, and clear expiry. Kelly criterion applies directly — but the critical input is your estimated true probability. Here's how causal model confidence feeds into the formula.

**Category:** risk | **Author:** Patrick Liu | **Reading time:** 1 min

---
Position sizing in prediction markets is simultaneously simpler and more dangerous than in equities. Simpler because the math is cleaner. More dangerous because the contracts have a hard cap at $1 — meaning you can lose 100% of your position.

Let's work through the math, then apply it to a real thesis.

## Why Prediction Markets Are Perfect for Kelly

The Kelly criterion was designed for exactly this type of bet:

- **Binary outcome** — the contract settles at $1 or $0
- **Known payoff structure** — you know exactly what you win and what you lose
- **Repeatable** — you can make many independent bets over time
- **Edge is estimable** — you have a probability estimate that differs from the market

The formula is straightforward:

```
f* = (bp - q) / b

Where:
  f* = fraction of bankroll to bet
  b  = odds received (payout / risk)
  p  = your estimated probability of winning
  q  = 1 - p (probability of losing)
```

For a prediction market contract at price c (in dollars), buying YES:

- You pay c per contract
- You receive $1 if YES, $0 if NO
- Your profit if YES: (1 - c)
- Your loss if NO: c
- So b = (1 - c) / c

## A Real Example: The Iran Recession Thesis

Let's use real numbers from an active thesis.

**The setup:**
- Contract: "US enters recession in 2026" on Kalshi
- Market price: 35¢ (market says 35% probability)
- Your thesis confidence: 72% (based on the causal tree: war persists → Hormuz blocked → oil elevated → recession)

**Calculating Kelly:**

```
b = (1 - 0.35) / 0.35 = 1.857
p = 0.72
q = 0.28

f* = (1.857 × 0.72 - 0.28) / 1.857
f* = (1.337 - 0.28) / 1.857
f* = 1.057 / 1.857
f* = 0.569 → 56.9% of bankroll
```

Full Kelly says bet 56.9% of your bankroll. This is where most people stop and say "Kelly is crazy." And they're right — **you should never use full Kelly.**

## Why Full Kelly Is Wrong (And What to Use Instead)

Full Kelly assumes your probability estimate is *exactly correct*. But your estimate has uncertainty. The causal tree says 72%, but that number itself has a confidence interval. Maybe the real probability is somewhere between 55% and 85%.

This is where prediction markets differ from coin flips. With a fair coin, you know p = 0.5 exactly. With a causal model, your p = 0.72 is an estimate with its own uncertainty.

**The fix: fractional Kelly.**

| Approach | Formula | Our Example | Risk Level |
|----------|---------|:-----------:|:----------:|
| Full Kelly | f* | 56.9% | Suicidal |
| Half Kelly | f*/2 | 28.5% | Aggressive |
| Quarter Kelly | f*/4 | 14.2% | Conservative |
| Eighth Kelly | f*/8 | 7.1% | Cautious |

**Why half Kelly is the sweet spot for most traders:**

- You capture ~75% of the growth rate of full Kelly
- Your drawdowns are roughly half as severe
- You can be wrong about your probability estimate by a meaningful margin and still not blow up

For our recession thesis at half Kelly: bet 28.5% of your bankroll. If your bankroll is $10,000, that's $2,850 on recession contracts at 35¢ — roughly 81 contracts.

## The Critical Input: Where Does p Come From?

This is the part most Kelly discussions skip. They assume you have a probability estimate and move on. But *where does that number come from*?

In SimpleFunctions, p comes from the **causal tree aggregation**. Each node in the tree has a confidence estimate:

```
n1  War persists                     88%
n2  Hormuz blocked                   97%
n3  Oil stays elevated               64%
n4  Recession (given oil elevated)   78%
```

The thesis-implied probability of recession is the product of the conditional chain: the probability that *all upstream conditions hold* AND the terminal node fires. The exact calculation depends on the causal structure — some nodes are independent, some are conditional — but the output is a single number: your thesis-implied p.

**This is why the causal model matters for position sizing.** Without it, your p is a gut feeling. With it, your p is decomposed into testable components. And when one component changes — say, a diplomatic breakthrough drops n1 from 88% to 50% — the entire chain updates, your p drops, and your Kelly fraction drops with it.

## Practical Position Sizing Rules

Based on running real strategies, here are the rules that work:

### Rule 1: Never More Than 25% on a Single Thesis

Even if Kelly says 40%, cap it at 25%. Prediction markets have liquidity risk — you might not be able to exit at the price you want. The 25% cap accounts for this.

### Rule 2: Scale by Confidence Interval Width

If your causal tree nodes are tightly estimated (small range), use half Kelly. If they're loosely estimated (wide range), use quarter Kelly. The wider your uncertainty, the smaller your bet.

### Rule 3: Reduce Position Size as Expiry Approaches

As a contract approaches its resolution date, the theta decay accelerates. If your thesis hasn't played out with 2 weeks to expiry, reduce to quarter Kelly regardless of what the model says.

### Rule 4: Correlation Across Theses

If you're running multiple theses, check for correlation. If your recession thesis and your oil thesis share upstream nodes (they share "Hormuz blocked"), they're not independent bets. Size them as a portfolio, not individually. A simple rule: if two theses share more than 50% of their causal tree, treat them as one position for sizing purposes.

## The SimpleFunctions Workflow

When you create a strategy with `sf strategy create`, the agent walks you through position sizing:

1. It calculates full Kelly based on your thesis confidence vs. market price
2. It suggests half Kelly as the default
3. It shows you the maximum drawdown at that sizing
4. It lets you adjust up or down based on your risk tolerance

The strategy then monitors your position relative to the target size. If the edge widens (market drops but your thesis confidence holds), it signals to consider adding. If the edge narrows, it signals to consider trimming.

Position sizing isn't a one-time calculation. It's a continuous process that updates as your causal model updates. The agent handles the math; you handle the judgment calls.

---

## Key Takeaways

1. **Kelly criterion works perfectly for prediction markets** — binary outcomes, known payoffs, estimable edge
2. **Never use full Kelly** — your probability estimates have uncertainty, use half Kelly or less
3. **The causal model is the probability source** — decompose your belief into testable nodes, don't guess
4. **Size positions as a portfolio** — correlated theses share risk, treat them accordingly
5. **Update continuously** — as the causal tree changes, so should your position sizes