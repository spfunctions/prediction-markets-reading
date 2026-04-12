# Brier Score vs Log Loss vs Quadratic Score: Picking a Calibration Metric

> Three proper scoring rules, three different jobs. Brier for public reporting, log loss for ML training, quadratic for cross-distribution comparison. The same five forecasts scored under each.

**Category:** comparison | **Author:** Patrick Liu | **Reading time:** 11 min

---
Picking a calibration metric is the kind of decision that feels like it should be "use whichever, they all measure the same thing" and is actually load-bearing on what your dashboard ends up communicating. Brier, log loss, and quadratic score are all proper scoring rules, which means they all incentivize honest forecasting. They are not interchangeable. The differences matter when you publish a number that other people are going to use to compare forecasters.

The short version: Brier is the right default for public reporting on prediction markets, log loss is the right loss function for training ML forecasters, and quadratic score is the right metric when you are comparing forecasters who use very different probability distributions. `/api/calibration` returns Brier because Brier is the right choice for the use case it serves.

## TL;DR

| Axis | Brier Score | Log Loss | Quadratic Score |
|---|---|---|---|
| Formula | (p − a)² | -log(p) if a=1 else -log(1-p) | -2pa + p² + (1-a)² |
| Bounded? | Yes (0 to 1 for binary) | No (∞ if you predict 0 and outcome is 1) | Yes (0 to 2 for binary) |
| Sensitivity to extreme errors | Linear in squared error | Punishes very-confidently-wrong much harder | Linear in squared error |
| Defined at p=0 or p=1? | Yes | No (-∞ if you are wrong) | Yes |
| Decomposable into calibration + resolution + uncertainty | Yes (Murphy decomposition) | Yes (entropy decomposition) | Yes |
| Used by | Most prediction-market dashboards, weather forecasting | ML training (Kaggle, sklearn, PyTorch) | Older forecasting literature, game theory |
| Used by us (`/api/calibration`) | Yes | No | No |
| Strict propriety | Yes | Yes | Yes |
| Median computation cost (per forecast) | One subtraction, one square | One division, one log | One multiply, one square |

## Brier Score

The Brier score for a binary forecast is (p − a)² where p is the predicted probability and a is the realized outcome (0 or 1). For multi-class forecasts the formula generalizes to the sum of squared differences across all outcomes.

The properties that make Brier the right default for public reporting:

- **Bounded**. A binary Brier is between 0 and 1. The worst possible score is 1 (you predicted 0 and the outcome was 1, or vice versa). This is the property that lets you put Brier on a chart without worrying about a single bad forecast blowing up the y-axis.
- **Linear in squared error**. The penalty for being wrong by 0.2 is four times the penalty for being wrong by 0.1. This is the natural shape that most readers intuit when they see a "calibration error" number.
- **Murphy decomposition**. The Brier score decomposes cleanly into three components: calibration (how well do your stated probabilities match realized frequencies?), resolution (how much does your forecast vary across outcomes?), and uncertainty (the irreducible variance of the outcome itself). This decomposition is what `/calibration` uses to show the per-category breakdown.
- **Defined at p=0 and p=1**. If you are willing to bet the farm on a forecast, you can express that as p=1 and the Brier score will tell you how it played out. Log loss cannot do this without infinite penalties.

The marketwide Brier on `/api/calibration` for our resolved sample is 0.095 (Polymarket 0.030, Kalshi 0.160). For interpretation: a perfectly random forecaster on a 50/50 outcome scores 0.25 in expectation, so 0.095 is meaningfully better than chance, and the gap between Polymarket and Kalshi reflects sample composition more than intrinsic accuracy.

## Log Loss

Log loss (also called cross-entropy loss) is -log(p) when the outcome is YES and -log(1-p) when the outcome is NO. It is the natural likelihood-based scoring rule and the standard loss function for training probabilistic ML models.

The properties that make log loss the right choice for ML training:

- **Smooth gradient**. The derivative of log loss with respect to the predicted probability is well-behaved everywhere except at p=0 or p=1, which makes it work cleanly with stochastic gradient descent. Brier's derivative is also smooth, but the gradient signal is weaker for very confident-and-wrong forecasts, which makes optimization slower.
- **Information-theoretic interpretation**. Log loss is the expected number of bits required to encode the realized outcome under your forecast distribution. This connects directly to entropy, mutual information, and KL divergence — all of which matter for ML model design but not for public reporting on prediction markets.
- **Punishes over-confident wrongness very hard**. A forecast of p=0.99 that turns out to be wrong gets a log loss of -log(0.01) ≈ 4.6, compared to a Brier of (0.99 − 0)² = 0.98. The log-loss penalty is about 4.7x the Brier penalty for the same error. This is a feature for ML training (you want gradients that strongly punish overconfidence) and a bug for public reporting (one bad forecast can dominate a 1000-forecast average).

The catastrophic failure mode of log loss in public reporting is what happens when a forecaster predicts p=0 and the outcome is 1. Log loss = -log(0) = +∞. Your average over 1000 forecasts is +∞ regardless of how good the other 999 were. This is why no prediction-market dashboard uses log loss as the headline metric — a single oops blows up the entire history.

## Quadratic Score

The quadratic score for a binary forecast with predicted probability p and outcome a is -2pa + p² + (1-a)². It is closely related to Brier — in fact, it is almost the same thing, just expressed in a different normalization. The traditional quadratic score is bounded between 0 and 2 (rather than Brier's 0 and 1) and is more often used in game theory and decision theory than in applied forecasting.

The properties that make quadratic score occasionally useful:

- **Cross-distribution comparability**. When you are comparing forecasters who use very different underlying probability distributions (one forecaster is a beta-distribution Bayesian, another is a frequentist point estimator, a third is a market price), quadratic score has properties around scale invariance that make the comparison cleaner than Brier in some edge cases.
- **Zero-sum interpretation**. The traditional quadratic scoring rule can be set up so that the sum of scores across all forecasters in a tournament is constant, which makes it useful for forecasting competitions where you want a clean ranking.
- **Historical literature**. A lot of the older forecasting literature (Brier 1950, Savage 1971, Selten 1998) uses quadratic score as the canonical example, so if you are reading those papers you need to know how it relates to Brier. Practically, quadratic score reduces to a linear transformation of Brier in most binary cases, so the rankings produced by the two metrics are identical even though the absolute numbers differ.

For prediction-market reporting, quadratic score has no real advantage over Brier and several disadvantages (the unfamiliar normalization is the main one). I have never seen a prediction-market dashboard report quadratic score, and I do not think anyone ever should.

## Worked Example: Five Forecasts, Three Metrics

The same five forecasts, scored under all three metrics. Outcomes are realized; predicted probabilities are what the forecaster said.

| # | Predicted p | Outcome | Brier (p-a)² | Log loss | Quadratic |
|---|---|---|---|---|---|
| 1 | 0.80 | 1 (YES) | 0.04 | 0.223 | 0.08 |
| 2 | 0.50 | 0 (NO) | 0.25 | 0.693 | 0.50 |
| 3 | 0.95 | 1 (YES) | 0.0025 | 0.0513 | 0.005 |
| 4 | 0.10 | 1 (YES) | 0.81 | 2.303 | 1.62 |
| 5 | 0.70 | 1 (YES) | 0.09 | 0.357 | 0.18 |

Average: Brier 0.239, log loss 0.725, quadratic 0.477.

Notice forecast #4 — the bad one. The forecaster said 10% and the outcome happened. Under Brier, the penalty is 0.81. Under log loss, the penalty is 2.303. Under quadratic, the penalty is 1.62. Log loss assigns the worst forecast roughly 2.85 times the penalty Brier does relative to the average — log loss makes the bad forecast dominate the average more strongly.

If forecast #4 had been p=0.01 instead of p=0.10, Brier would give 0.9801 (still bounded), log loss would give 4.605, and quadratic would give 1.9602. The log-loss penalty has more than doubled while the Brier penalty has barely moved. This is the property that makes log loss great for ML training (the gradient strongly pulls the forecaster away from over-confident wrong predictions) and dangerous for public reporting (a single very confident wrong forecast can dominate the average).

## Decision Tree

- **You are publishing a calibration number on a public dashboard:** Brier. It is bounded, it does not blow up on tail bets, and it is the metric your readers expect.
- **You are training a probabilistic ML model with stochastic gradient descent:** log loss. The gradient signal is what you want for optimization, and the unbounded penalty is a feature in training even though it is a bug in reporting.
- **You are running a forecasting competition where rankings matter and you want the rankings to be invariant to monotonic transformations:** quadratic, but Brier produces the same rankings on binary outcomes so it does not really matter.
- **You are decomposing a forecast quality number into calibration / resolution / uncertainty components:** Brier with the Murphy decomposition. Log loss has its own decomposition (calibration / refinement / entropy) but Brier's decomposition is more interpretable for non-specialists.
- **You are computing a forecaster's expected payoff under a proper scoring rule contract:** any of the three. They are all proper, which is the only property the contract needs.
- **For our use case at SimpleFunctions:** Brier, every time. `/api/calibration` returns Brier because Brier is the right metric for the kind of public-facing prediction-market reporting we do.

## Where Each Side Breaks Down

Brier breaks down when you actually need to penalize over-confident wrongness more heavily than its quadratic shape allows. If you are scoring a forecaster who occasionally bets the farm at p=0.99, Brier treats those bets as only marginally worse than p=0.95 bets when wrong, and that under-weights the catastrophic risk. Log loss handles this case better. The fix on the Brier side is to weight by stake size or to report a tail-conditional Brier alongside the average — but neither is a clean solution.

Brier also breaks down when the underlying distribution is genuinely multi-modal. A forecaster who says "either 0.05 or 0.95 with equal probability" should not be scored the same as one who says "0.50 with high confidence," and Brier can collapse the two into similar scores depending on outcome distribution. This matters less for binary outcomes than for continuous ones.

Log loss breaks down — catastrophically — on any p=0 or p=1 forecast that turns out to be wrong. The fix is to clip predictions away from the boundaries (e.g., min(max(p, 0.001), 0.999)), but every clip is a hack and the choice of clip threshold becomes its own bias source. In practice, anyone using log loss on prediction-market data has to clip, and the clipping makes the score sensitive to the clip parameter in ways that Brier is not.

Quadratic score breaks down by being unfamiliar. The math is fine but no one reads the result correctly without translation, and the translation is back to Brier anyway. Use Brier and skip the indirection.

## Live Data Reference

The Brier scores reported by `/api/calibration` use the formula and decomposition described above, applied to the resolved subset of markets in our database. For per-category Brier breakdowns, see `/calibration`. For the per-market historical price data that goes into the calculation, use `sf scan --resolved --by-brier desc` from the CLI to see the worst-calibrated resolved markets in descending order.

For the framework view of how calibration metrics fit into a trading strategy, see `/concepts/pm-indicator-stack` and the glossary entry at `/learn/calibration`. The IY-vs-raw-probability framing at `/opinions/implied-yield-vs-raw-probability-bond-markets` covers a related "pick the right unit before you score the forecast" argument, and `/opinions/ai-agents-need-judgment` discusses why metric choice matters for any forecaster (human or LLM) trying to learn from outcomes.

The marketwide Brier of 0.095 hides a lot of variance — sports markets calibrate around 0.04, election markets around 0.07, weather around 0.02, and the long-tail "miscellaneous" category sits at 0.18. The category breakdown is much more useful than the headline number for picking which categories to trust prices on.

## The Bottom Line

Use Brier for everything except ML training. Brier is bounded, decomposable, and is the standard that the prediction-market community has converged on. Log loss has its place in model training and nowhere else. Quadratic score has no place outside of the historical literature.

If someone hands you a prediction-market dashboard reporting log loss as the headline metric, ask whether they have clipped extreme predictions, what threshold they used, and how the clipping affects the historical comparison. The answer will reveal whether the dashboard is built by someone who knows the metric is fragile or by someone who picked it without thinking. In either case, ask for the Brier number alongside it — that is the comparison you can actually trust.