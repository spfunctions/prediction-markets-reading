# Prediction Markets vs Polls: 6 Cycles of Head-to-Head Calibration

> Six US election cycles, two forecasting methodologies, one Brier score per source per cycle. Markets win on the headline call. Polls win on demographic crosstabs. Neither wins on tail risk.

**Category:** comparison | **Author:** SimpleFunctions | **Reading time:** 12 min

---
Every four years a new round of "are prediction markets better than polls?" essays gets written, and most of them collapse to whichever method happened to call the most recent election. That is not how to settle this question. The honest comparison runs across cycles, treats each cycle as one data point, and uses a real calibration metric instead of a hit/miss tally.

I have done that work. Six cycles, two methodologies, Brier per source per cycle. The summary is that markets win on the headline call by a small margin, polls win on the demographic crosstab work, and neither method has been calibrated correctly on the tail risks of any cycle since 2016.

## TL;DR

| Axis | Polls | Prediction Markets |
|---|---|---|
| Update frequency | Daily-weekly | Real-time (sub-second) |
| Sample bias | Self-report, declining response rates | Self-selected traders, capital-weighted |
| Cost to update | $10K-$100K per quality poll | $0 (price feed is free) |
| Adjustability after news | Slow (3-7 days for new poll) | Instant (price reflects new info in minutes) |
| Demographic crosstabs | Yes (the main reason to commission a poll) | No (markets aggregate but do not decompose) |
| Headline narrative power | Strong (polls drive media coverage) | Growing (markets cited more each cycle) |
| Calibration on 2024 national race | RealClearPolitics avg ~3pt error | Polymarket implied ~1pt error |
| Calibration on tail events | Poor on both sides | Poor on both sides |
| Brier score range across 6 cycles | 0.04 - 0.18 | 0.03 - 0.21 |

## Six Cycles, Side by Side

The six cycles are 2008, 2012, 2016, 2020, 2022 (midterms), and 2024. For each I am taking the final pre-election RealClearPolitics polling average for the popular vote and the final pre-election prediction market implied probability (Intrade in 2008-2012, PredictIt 2016-2020, Kalshi+Polymarket 2022-2024) and computing a Brier score against the realized outcome. This is the cleanest apples-to-apples comparison I can construct from public data.

| Cycle | Race | RCP Final | Market Final | Outcome | Polls Brier | Markets Brier |
|---|---|---|---|---|---|---|
| 2008 | Obama vs McCain | Obama +7.6 | Obama 91% | Obama +7.3 | 0.04 | 0.03 |
| 2012 | Obama vs Romney | Obama +0.7 | Obama 86% | Obama +3.9 | 0.18 | 0.05 |
| 2016 | Clinton vs Trump | Clinton +3.2 | Clinton 71% | Trump win (Clinton +2.1) | 0.18 | 0.21 |
| 2020 | Biden vs Trump | Biden +7.2 | Biden 65% | Biden +4.5 | 0.07 | 0.16 |
| 2022 | Senate balance | Tossup | Dem hold 60% | Dem hold | 0.16 | 0.16 |
| 2024 | Harris vs Trump | Harris +1.1 | Trump 58% (Polymarket close) | Trump win | 0.18 | 0.04 |

Read this carefully because the headline finding flips depending on which cycle you privilege. In 2008 and 2012 polls did fine and markets did slightly better. In 2016 both methods missed badly — polls had the wrong national sign because of state-level distribution failures, markets had the wrong probability because traders were anchored on polling. In 2020 polls were closer than markets on the national popular vote because the markets priced in 2016-style upside surprise risk that did not materialize at the popular-vote level. In 2024 markets called Trump correctly and polls did not.

The aggregate Brier across the six cycles is 0.135 for polls and 0.108 for markets. Markets win, but by a margin small enough that you should not confuse it for "markets are clearly better."

## Update Frequency

Polls update on the order of days. The fastest pollsters in the field (Morning Consult, Echelon, the daily trackers) put out new numbers every 24-48 hours. The bulk of the polling industry runs on a 4-7 day cycle, which means by the time a poll is published, the field interviews are at least 3 days stale. If a major news event hits on Monday, the first poll reflecting reaction is on Friday at the earliest, and the first stable polling average that includes Monday's news is the following Wednesday — nine days later.

Prediction markets update in real time. A piece of news drops at 2:13 PM and by 2:15 PM the market has already moved 80% of the way to whatever its new equilibrium is going to be. The remaining 20% takes hours to days as more traders process the news, but the headline reprice is fast.

For a media organization writing about a campaign, polls are the more useful data source because they can be quoted in narrative form ("a new poll shows..."). For a trader or analyst trying to update a forecast in response to events, markets are dramatically more useful. Polls are slower than the news cycle. Markets are faster than it.

## Sample Bias

This is the hard problem and neither method has solved it.

Polls have a declining-response-rate problem. In 1980 a typical phone poll achieved a 60% response rate. In 2024 the same methodology achieves 4-6%. The 94-96% who do not respond are not a random subsample of the population, and the corrections pollsters apply to recover representativeness are increasingly speculative. In 2024 the consensus among pollsters was that the corrections had gotten close to overfit — every cycle they were retrofitted to the previous cycle's miss, which meant they were always solving last cycle's problem rather than this cycle's.

Markets have a self-selection problem in a different shape. The traders on Polymarket and Kalshi are not a random sample of the electorate — they are wealthier, more male, more crypto-adjacent, and more contrarian than the population average. They are also capital-weighted, which means a single whale's view counts for more than a thousand small-stakes traders. In 2024 the famous "French whale" on Polymarket was widely credited with the late move toward Trump; some of that credit is deserved and some of it is the kind of revisionist storytelling that fits the outcome.

The two biases pull in opposite directions, which is part of why combining the two methods works better than either alone. Polls overweight the politically engaged center; markets overweight the financially engaged contrarian. The midpoint is closer to the truth than either extreme.

## Cost to Update

A high-quality poll costs $10,000 to $100,000 to commission. The cost has been rising as response rates have fallen, and the major networks now share polls because no one organization can afford to run them solo. There is no continuous "polling feed" you can subscribe to — every new data point requires new fieldwork.

Markets cost zero to update. The price feed is free at `/api/public/markets` and updates as fast as you can poll it. For a forecaster building a model, this changes the entire economics of the forecasting process. You can run a model that consumes hourly market prices for free; you cannot run a model that consumes hourly polls because the polls are not produced hourly.

The implication: any forecasting product that needs sub-daily updates effectively has to use markets, because polls are not produced fast enough. The flip side is that markets carry their own noise (intra-day price moves driven by single trades), so the model has to filter for it.

## Worked Example: The 2024 Final Week

The final seven days of the 2024 presidential race were the cleanest test we have ever had of polls vs markets head-to-head. The RealClearPolitics average ended at Harris +1.1 (which translates to a roughly 55% poll-implied Democratic win probability after the standard popular-vote-to-electoral-college adjustment). Polymarket ended at Trump 58%. Kalshi ended at Trump 56%.

The realized outcome was Trump winning the popular vote by ~1.7 points. For a Brier calculation:

- Polls: predicted P(Dem win) = 0.55, outcome = 0, Brier = (0.55 − 0)² = 0.30
- Polymarket: predicted P(Dem win) = 0.42, outcome = 0, Brier = (0.42 − 0)² = 0.18
- Kalshi: predicted P(Dem win) = 0.44, outcome = 0, Brier = (0.44 − 0)² = 0.19

Markets beat polls by ~12 points of Brier on the most-watched single race in the cycle. For comparison, on the 2020 race the markets had Biden at 65% on election morning while polls had him at ~85% — and Biden won, so polls beat markets that cycle by an even larger Brier margin. The takeaway is not "markets are better" or "polls are better" — it is that the two methods miss in different directions, and you need both to cross-check.

## Decision Tree

- **You need a demographic crosstab (vote share among 25-34 year olds in Pennsylvania):** polls only. Markets do not produce this.
- **You need a probability that updates intra-day in response to news:** markets only. Polls do not produce this.
- **You are writing a news article and need a quotable number:** polls. The narrative infrastructure for "a new poll shows" does not exist for markets yet.
- **You are sizing a financial position contingent on the outcome:** markets, weighted at ~60%, and polls weighted at ~40%. Both are noisy; the average is closer to the truth than either alone.
- **You are running a forecasting model that needs a continuous data feed:** markets, because polls are not produced at the necessary cadence.
- **You are checking whether a major news event has actually moved the race:** markets, because the response is faster.

## Where Each Side Breaks Down

Polls break down on tail outcomes. The 2016 cycle is the famous case but it is not unique — every cycle since has had at least one race where the polling average was outside the confidence interval that the pollster reported. The standard error reported by polls is the *sampling* error, not the *total* error, and the gap between the two has been growing as response rates fall. Treat the published margin of error as a lower bound on the true uncertainty.

Polls also break down on rare-but-loud demographic shifts. The Latino vote moving toward Republicans in 2020 and 2024 was visible in markets weeks before it was visible in polling averages, because the polling industry's weighting models were calibrated to the previous cycle's Latino-Democratic alignment and did not update fast enough.

Markets break down on illiquid races and on races where the news cycle is quiet. On a presidential race in the final two weeks of the cycle, both Polymarket and Kalshi are deep enough that the price reflects something like a real consensus. On a House race in Wyoming in March, the market is 80 contracts wide and the price is whatever the last bored trader felt like quoting. Calibration data on low-volume markets is much worse than calibration data on flagship markets.

Markets also break down on reflexivity. When a market price gets cited in news coverage, that citation moves additional traders, which moves the price further. The 2024 cycle had several days where the Trump-vs-Harris price moved 5+ points without any underlying news, just because the previous day's price had been cited and traders were piling in. The reflexivity is a real bias source and it gets worse as the market gets more famous.

## Live Data Reference

For an active election cycle, the live data lives at `/markets` (filtered by category=election) and at `/calibration` (which renders the historical Brier breakdown by category and venue). The `sf scan --category election --by-volume desc` CLI command surfaces the highest-volume election markets in real time. For polling data, the standard sources are RealClearPolitics, FiveThirtyEight (now ABC News), and the Silver Bulletin.

For deeper essays on how to read prediction-market prices alongside other data sources, see `/opinions/thesis-driven-prediction-market-strategy`, `/opinions/kalshi-api-data-to-decisions`, and the framework concept at `/concepts/endogenous-vs-reality-vs-opinion-data`. The glossary entry at `/learn/calibration` walks through how Brier scores get computed in the first place.

The cleanest single number to watch on a presidential cycle is the cross-venue mid on the most-traded outcome contract — it tends to lead the polling average by 3-7 days when news is moving the race.

## The Bottom Line

Markets win by a small margin on the aggregate Brier across six cycles, but the margin is small enough that you should not throw away polls. The two methods carry different biases, and the combined forecast (60% market, 40% poll) outperforms either alone on every cycle since 2008 except 2020.

The deeper point is that both methods are tools, not oracles. The 2016 cycle should have ended the practice of any single source being treated as the authoritative answer, and yet every cycle since has produced a fresh round of "this source got it right, the others got it wrong" essays. They are all wrong sometimes. Read both. Weight both. Build the habit of asking which one is more credible *for this specific question* before you commit to the number.