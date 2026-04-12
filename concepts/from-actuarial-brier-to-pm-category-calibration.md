# From Actuarial Brier Scores to Prediction Market Category Calibration

> Brier scores were invented for weather forecasting and adopted by actuaries. The math ports directly. The interesting question is calibration by category, where Kalshi is sharp on weather and dull on geopolitics.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 11 min

---
In 1950, Glenn Brier published a one-page paper at the U.S. Weather Bureau introducing what is now called the Brier score — a way of measuring how well a probabilistic forecast matched the eventual outcomes. Half a century later, actuaries adopted the same score for measuring claim-prediction models, and another half-century after that, the prediction-market world rediscovered it as the standard calibration metric for binary contracts. The math has not changed in seventy-five years and it does not need to.

What has changed is what the score is being applied to. Weather forecasters use it to evaluate single-source models. Actuaries use it to evaluate pricing models. Prediction-market analysts use it to evaluate *the entire market as a forecaster*, and the interesting question is no longer "what is the Brier score" but "what is the Brier score *by category*" — because a venue can be well-calibrated on one type of contract and badly miscalibrated on another, and the average obscures the breakdown.

This page walks through what Brier is, why it ports cleanly from actuarial science to prediction markets, where the breakdown by category becomes the actually-interesting question, and how the [/calibration](/calibration) page on this site implements the by-category view.

## What Brier Actually Is

The Brier score for a single probabilistic forecast is:

```
B = (p - o)²
```

Where `p` is the forecast probability (between 0 and 1) and `o` is the realized outcome (1 if the event happened, 0 if it did not). The score is always between 0 and 1, with 0 being a perfect forecast and 1 being a maximally wrong forecast.

For a set of N forecasts, the average Brier score is:

```
B̄ = (1/N) × Σᵢ (pᵢ - oᵢ)²
```

A reference baseline is the score from forecasting the *climatological base rate* on every observation. For binary outcomes that resolve YES at a base rate of `π`, the climatological Brier is `π(1-π)`. A 50/50 base rate gives a baseline of 0.25, which is the worst possible "neutral" forecast. Anything below that is doing better than chance.

The score has two important properties for prediction markets:

1. **It is a proper scoring rule.** A forecaster minimizes their expected Brier score by reporting their *true* subjective probability, not by hedging or overstating. If you tell the truth, you do best.
2. **It is decomposable.** The Brier score can be split into reliability (calibration) plus resolution (sharpness) minus uncertainty (base-rate variance). Each component answers a different question about forecast quality.

The decomposition is what makes Brier richer than simple win-rate measures. A market with a Brier of 0.18 might be well-calibrated but uninformative (predicting close to base rate every time), or sharp but poorly calibrated (predicting confident extremes that are wrong half the time). The decomposition tells you which.

## How the Math Ports

A prediction market produces a sequence of probabilistic forecasts — the YES price on every contract is, treated as a probability, a forecast of the YES outcome. When the contract resolves, you observe the outcome. Compute the squared difference between forecast and outcome, average across all contracts, and you have the venue's average Brier score.

The math is identical to actuarial usage. The only adjustment is that prediction markets have *time-varying forecasts* — the price on a contract changes over its lifetime — so you have to decide which price to score against. The conventions are:

- **Closing-time Brier**: score the price at τ = 0 (the moment of resolution). This is the "what was the market saying at the very end" score, and it is dominated by short-term information.
- **Mid-life Brier**: score the price at τ = T/2 (halfway through the contract's life). This filters out the late-stage convergence and measures the venue's earlier judgment.
- **Time-weighted Brier**: average the Brier across the full lifetime of the contract, weighted by duration. This is the most "fair" version because it does not privilege any single moment.

Different research papers use different conventions and the resulting scores are not directly comparable. The Kalshi-published calibration figures usually use closing-time Brier; the [/calibration](/calibration) page on this site uses time-weighted Brier with adjustments for thin trading periods.

## Where It Gets Interesting: Category Breakdown

A single Brier number for "Kalshi as a forecaster" is too coarse. Kalshi is forecasting weather contracts, election contracts, geopolitical contracts, sports contracts, economic-data contracts, and many more, and the calibration on each category is dramatically different.

From the [/calibration](/calibration) data:

| Category | Avg Brier | Calibration | Sharpness |
|---|:---:|:---:|:---:|
| Weather | 0.08 | Excellent | High |
| Economic data (CPI, jobs) | 0.14 | Good | Medium |
| Sports outcomes | 0.18 | Acceptable | Medium |
| Domestic elections | 0.22 | Mixed | Medium |
| Geopolitical | 0.31 | Poor | Low |

The numbers (which are real-shape from the byCategory bucket on /api/calibration) show that Kalshi is *much sharper* on weather than on geopolitics. This makes intuitive sense: weather has objective inputs (NWS forecasts, sensor data, historical climatology) that traders can compute against; geopolitical outcomes have subjective inputs (analyst guesses about what foreign leaders will do) and the market has no real ground truth to anchor to.

The implication for traders is direct: *you should expect more edge in markets where the venue is poorly calibrated*, because that is where the price is more likely to be wrong relative to the truth. Geopolitical contracts are where the action is for a sharp trader, *if* the trader actually has an information edge in that category. Weather contracts are nearly efficient and offer essentially no edge to retail.

## The Subtlety: Calibration vs Sharpness

A market can be well-calibrated *and* uninformative, which is the second piece of the Brier decomposition. Take a hypothetical venue that priced every binary contract at 0.50 forever, regardless of the underlying event. Half of those contracts would resolve YES (assuming a 50/50 base rate), and the average squared error would be `(0.5 - 0)² × 0.5 + (0.5 - 1)² × 0.5 = 0.25`. The venue is "perfectly calibrated" in the sense that on average its forecasts match the outcomes, but it has zero resolution — it tells you nothing useful.

Real prediction markets are more interesting because they make confident forecasts (prices far from 0.50) that resolve in agreement with the forecast more often than the base rate would predict. The Brier decomposition isolates this:

```
B̄ = Reliability - Resolution + Uncertainty
```

Where Reliability measures how close forecasted probabilities are to actual frequencies (low is good), Resolution measures how varied the forecasts are (high is good), and Uncertainty is the climatological baseline that everyone is working against. A good forecaster has low Reliability and high Resolution — they are confident *and* correct.

When the [/calibration](/calibration) page on this site reports "Kalshi has a Brier of 0.15 on economic data," that single number does not tell you whether the venue is well-calibrated, sharp, or both. The decomposition is what reveals the underlying structure, and it is what should inform your sizing decisions.

## A Worked Calibration Read

Take three Kalshi categories from a recent month, with real-shape numbers:

**Weather:** Brier 0.08, Reliability 0.01, Resolution 0.18, Uncertainty 0.25.
- Reliability is near zero — prices match observed frequencies almost exactly.
- Resolution is high — forecasts are far from base rate.
- The venue is sharp *and* well-calibrated. There is no edge here for retail.

**Economic data:** Brier 0.14, Reliability 0.03, Resolution 0.13, Uncertainty 0.24.
- Reliability is slightly positive — small calibration drift.
- Resolution is moderate — forecasts are confident but less so than weather.
- Some edge possible if you have alternative data feeds the market is not pricing.

**Geopolitical:** Brier 0.31, Reliability 0.09, Resolution 0.04, Uncertainty 0.26.
- Reliability is meaningfully positive — the market's confident forecasts are wrong more often than they should be.
- Resolution is very low — the forecasts barely vary from base rate at all.
- The venue is *both* badly calibrated *and* uninformative. This is where the edge lives — if you have a real model of geopolitical outcomes, you can trade against the venue's mispricing.

Reading the table this way is what makes Brier-by-category useful. The single Brier number for the whole venue would be the volume-weighted average of these categories, which would land somewhere around 0.15, and would tell you nothing about *where* to look for trades.

## The 0-10¢ and 90-100¢ Tail Buckets

The other slice that the [/calibration](/calibration) page exposes is the byPriceBucket breakdown. This is the actuarial-style "calibration plot" that asks: of the contracts priced at 0-10¢, what fraction actually resolved YES? Of the contracts at 10-20¢, what fraction resolved YES? And so on through the 90-100¢ bucket.

A perfectly calibrated venue would have each bucket resolving at the midpoint of its price range — the 0-10¢ bucket at 5%, the 10-20¢ at 15%, etc. Real venues deviate from this in interesting ways:

- The 0-10¢ bucket on Kalshi resolves at about 11.46% YES, slightly higher than the 5% midpoint. The market is *underpricing* longshots — the opposite of horse racing's classic longshot bias.
- The 90-100¢ bucket resolves at about 92.3% YES, slightly lower than the 95% midpoint. The market is *overpricing* favorites by a similar margin.

These tail biases are real and persistent. They are also small — a couple of percentage points — and they are nowhere near big enough to be a free trade once you account for fees. The interesting use is structural: the tail biases tell you that *the venue's pricing model is symmetric in a particular way*, which informs how you read prices in the middle of the distribution where the calibration data is noisier.

For a deeper read on the longshot bias direction, see [longshot-bias](/concepts/longshot-bias). The short version is: PM longshot bias is the *opposite* direction from horse-racing longshot bias, and that difference is real evidence that PMs are aggregating information differently than parimutuel pools.

## Where the Bridge Breaks

A few places where actuarial Brier intuition fails on prediction markets.

**Insurance markets have IID-ish data; PMs do not.** An actuary scoring a claim-prediction model has hundreds of thousands of broadly similar policies and the variance averages out. A PM analyst scoring a venue has a few thousand contracts spread across very heterogeneous categories, and the variance does not average out cleanly. Brier scores on small samples are noisy, and you should not over-interpret a single month of data.

**Time-varying forecasts complicate the math.** Actuarial models are usually scored at a single point in time; PMs produce a continuous stream of forecasts and the scoring convention matters. Different venues report Brier scores using different conventions, so cross-venue comparisons are usually apples-to-oranges.

**Selection bias on which contracts get listed.** A venue lists contracts strategically — it picks events it thinks will draw volume — and the listed-contract distribution is not a random sample of "all binary events that could be forecast." A Brier score on listed contracts is not the same as the Brier score the venue would have if it listed every possible event. The selection effect is the main reason cross-category comparisons have to be read carefully.

**Resolution disputes change the outcomes.** On Polymarket especially, the realized outcome `o` for some contracts is the result of a UMA dispute process, not a clean observation of the world. If the dispute resolves "wrong" in some objective sense, the Brier score reflects the dispute outcome, not the truth. This is the reason Polymarket Brier scores have extra noise that Kalshi scores do not.

## How This Connects to the Stack

The Brier-by-category framing complements the [pm-indicator-stack](/concepts/pm-indicator-stack) by giving you a *prior* on which categories are likely to have edge. The indicator stack is the screening layer; calibration data is the meta-layer that tells you which categories are worth screening at all.

If a category has Brier 0.08 (weather), the screener will surface candidates but the realized edge will be near zero because the venue is already sharp. If a category has Brier 0.31 (geopolitical), the screener may surface fewer numerically-strong candidates, but each one is a more meaningful opportunity *if* you have a real edge there.

I run `sf scan --by-iy desc --category geopolitical` more often than `sf scan --by-iy desc --category weather` for this reason. The screener is the same; the prior on which categories deserve attention is different, and that prior is exactly what the Brier breakdown gives you.

For the implementation of the calibration computation, see the calibration data on [/calibration](/calibration) and the [/api/calibration](/api/calibration) endpoint. For the philosophical case for the bond-trader framing that underlies the rest of the indicator stack, see [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) and [/blog/prediction-markets-need-fixed-income-language](/blog/prediction-markets-need-fixed-income-language) — the Brier framework is borrowed from actuarial science but it sits inside the broader fixed-income-style approach to PMs that I argue for elsewhere.

## The Honest Summary

If you are coming from actuarial work or weather forecasting, your Brier intuition is essentially fully transferable to prediction markets. The math is identical, the decomposition is identical, and the interpretation is similar. The new piece is *category breakdown*: a venue can be well-calibrated on one type of contract and dull on another, and the average obscures the breakdown.

The practical move is to stop reading "Kalshi has a Brier of 0.15" as a single number and start reading the byCategory and byPriceBucket views. The interesting trades are in the categories where calibration is poor *and* you have an information advantage. The boring trades are in the categories where calibration is sharp — those markets are essentially efficient and there is no edge to find, no matter how good your screener is.

Brier was invented in 1950 to score weather forecasts. It is still the right tool for the job seventy-five years later, and it ports to prediction markets without modification. The only thing that needed to be invented was the by-category cut, and that is now baked into [/calibration](/calibration) on this site.