# Why a "DCF" of a Prediction Market Is Mathematically Incoherent

> A discounted cash flow model needs multiple cash flows and a discount rate that compensates the holder for bearing those flows over time. A binary prediction-market contract has exactly one cash flow at exactly one date. There is nothing to discount that is not already inside the implied yield.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 10 min

---
When I tell a former corporate-finance person that you cannot DCF a prediction market, the first reaction is usually "of course you can, every cash flow can be discounted." Then they sit down to write the formula, get to the second line, realize the only cash flow is the resolution payoff, and quietly close the spreadsheet.

The reason DCF does not work on a binary prediction-market contract is not that the math is hard. The reason is that the math gives you back exactly the implied yield you started with, in fewer steps, with more pretentious notation. This page is the long version.

## What DCF Is, Briefly

Discounted cash flow valuation takes a stream of expected future cash flows, discounts each one back to the present at a rate that compensates the holder for time and risk, and sums them. The classical formula is:

```
PV = Σ (CF_t / (1 + r)^t)
```

Where `CF_t` is the cash flow at time `t` and `r` is the discount rate. The framework is great when you have a multi-period stream of cash flows that is uncertain in some calibrated way — corporate earnings, bond coupons, real-estate rents — and you want to back out a fair present value.

DCF rests on two structural features: there are *multiple* cash flows over *multiple* periods, and the *uncertainty* about each cash flow is reflected in the discount rate (not directly in the cash flow numbers). Both features are absent on a binary prediction-market contract.

## A Binary Contract Has One Cash Flow

A binary prediction-market contract pays $1 if the event resolves YES and $0 if it resolves NO. There is one resolution date. There is one payoff. There are no intermediate cash flows whatsoever — no coupons, no dividends, no rebates, nothing. The full cash-flow stream is a single number at a single point in time.

When the cash-flow stream collapses to one element, the DCF formula collapses to:

```
PV = CF_T / (1 + r)^T
```

Where `T` is the time to resolution. There is no sum, because there is only one term. The framework has degenerated into a one-variable equation.

You also have a problem with what to put in for `CF_T`. The expected cash flow is `(probability_yes × 1) + ((1 − probability_yes) × 0) = probability_yes`, which is a quantity strictly between 0 and 1. So the "expected DCF" of a binary contract is:

```
PV = probability_yes / (1 + r)^T
```

And the market price of the contract is, definitionally, what people are paying for it today. So the market price equals `probability_yes / (1 + r)^T`. Solve for `probability_yes` and you get `probability_yes = market_price × (1 + r)^T`.

That is not a valuation. That is an identity. The "DCF model" of a prediction market is just the implied yield equation rearranged. There is nothing the DCF framework adds beyond what you already had.

## The Implied Yield Connection

To make this completely explicit: the implied yield formula is

```
IY = (1 / p)^(365 / τ) − 1
```

Solve this for `p` and you get

```
p = (1 + IY)^(−τ / 365)
```

That is the same shape as the DCF identity above, with `r = IY` and `T = τ / 365`. The "discount rate" in any honest DCF of a binary is the implied yield itself, and once you have the implied yield, the DCF computation is over before it began. You have not added any information; you have just renamed it.

This is the core point of [implied-yield](/concepts/implied-yield) and [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets). The bond traders figured out fifty years ago that for a single-cash-flow zero-coupon instrument, "yield to maturity" *is* the valuation framework. Discounted cash flow and yield-to-maturity are the same machinery, applied at different levels of abstraction. For multi-period instruments, DCF gives you something YTM does not. For single-period instruments, they are the same thing, and the YTM language is more honest because it does not pretend you summed a series.

## What Someone Would Actually Try

Let me play devil's advocate and try to do an actual DCF on a hypothetical Kalshi Fed-cut contract trading at 0.32 with τ = 45 days. Imagine I am the corporate finance person convinced this should work.

Step one: identify the cash flows. There is one. The expected value is `0.32 × $1 = $0.32` if we use the market-implied probability, or some other number if we use a subjective probability.

Step two: pick a discount rate. The risk-free rate over a 45-day horizon is roughly the 1-month T-bill yield, which I will round to 5% annualized for the example. Add some risk premium for the binary outcome uncertainty — let me say another 5% — for a total of 10% annualized.

Step three: discount. `PV = 0.32 / (1 + 0.10)^(45 / 365) = 0.32 / 1.0118 ≈ 0.3163`. The "DCF fair value" of the contract is 31.6 cents.

The contract is currently at 32 cents. By my DCF, it is overvalued by 0.4 cents. Buy NO. Right?

No. Here is what just happened. I picked a 10% discount rate out of the air. I picked it because it sounded reasonable for "binary risk plus risk-free." But the *correct* discount rate, the one that makes the model self-consistent with the market, is the implied yield of the contract itself. And the implied yield of a 0.32 / 45-day contract is about 1,030,200%. That is the actual rate at which the market is discounting the cash flow, and if I had used it I would have gotten back the market price exactly.

The "10% feels right" rate is *my opinion about how risky the contract is*, dressed up as a discount rate. I have not modeled anything. I have asserted that the contract is mispriced at the current 1,030,200% implied rate and should instead be priced at a 10% rate. That is a thesis about the market being wrong. It is not a valuation. The DCF arithmetic is a thin coat of paint over "I think 32 cents is too high."

The deeper problem is that *there is no objective discount rate for a binary contract*. The cash flow's risk is binary outcome risk, which cannot be parameterized as a continuous discount rate the way credit spreads can. You can put a number in for `r` but the number has no anchor. Unlike a corporate bond, where the credit spread is measurable from CDS or from comparable bonds, a binary contract has no comparable instrument whose discount rate you can borrow.

## The Honest Replacement Is Already on Your Screen

The framework that does work for valuing a single-cash-flow binary instrument is not DCF. It is the [pm-indicator-stack](/concepts/pm-indicator-stack), which gives you the indicators that bear on whether the current price is right.

The indicators answer the questions DCF is *trying* to answer, in units that are honest about the binary structure of the underlying.

[Implied yield](/concepts/implied-yield) tells you the annualized return at the current price — the same number a DCF would back out as its discount rate, computed in one step.

[Cliff Risk Index](/concepts/cliff-risk-index) tells you whether the price is moving, which is the closest thing to "risk premium changing" that exists in a discrete-resolution context.

[Event Overround](/concepts/event-overround) tells you, for multi-outcome events, whether the sibling contracts are internally consistent — a check that has no DCF analog because DCF is per-contract.

[Liquidity Availability Score](/learn/liquidity-availability-score) tells you whether the price you see is the price you can transact at, which is upstream of any valuation framework.

Together with stage 3 of [the-valuation-funnel](/concepts/the-valuation-funnel) — the causal tree decomposition — these tools give you the equivalent of the "thesis" that DCF would otherwise embed in its discount rate, but they keep it visible and falsifiable. You can say "I think the implied yield of 1,030,200% is wrong because [specific causal-tree node A] is more likely than the market thinks." That is a thesis. "I think the discount rate should be 10% instead of 1,030,200%" is the same thesis with the labor hidden.

## A Quick CLI Worked Example

The fastest way to feel why DCF is redundant on PMs is to put the same contract through both pipelines. `sf scan --by-iy desc` gives you a sorted list of contracts by implied yield. Pick one off the top of the list — say the hypothetical KXFEDDECISION-26JUN at 0.42 with τ = 42 days — and try to write a DCF for it. You will discover that the only number you can sensibly use as the discount rate is the implied yield itself, which is what `sf scan` already showed you, which means you have done zero additional work.

If you want to add value beyond the stack, the place to add it is in the *causal tree*, not in the discount rate. Build a tree for the Fed-decision contract, identify two nodes you have a different opinion on than the market, compute what the implied probability would be under your tree, and compare to the current price. That is a thesis. `sf bet KXFEDDECISION-26JUN yes 100` is what you do once the thesis survives the comparison. DCF is what you do when you want to feel like you are doing finance work without actually doing any.

## Where This Negation Has Limits

Three corners where something *DCF-shaped* does survive on PMs.

The first is *income-generating PM positions* — namely market-making, where you collect spread over time and the position has a real cash-flow stream from each fill. A market-making position over a long horizon does have multiple cash flows, and you can in principle do a DCF on the expected fill schedule. The math is unstable because the fill rate is a function of conditions you cannot forecast, but the *shape* of the analysis is correct.

The second is *cross-tenor positions on the same event family*. If you are long the December Fed-cut contract and short the May Fed-cut contract, the position generates rolling returns as each leg matures. The combined position has a multi-period cash flow profile that is closer to a bond curve trade than to a single binary, and you can value it with curve-style methods that are DCF-adjacent.

The third is *baskets of binaries that are statistically independent and structured for diversification*. A basket of fifty uncorrelated binaries with similar τ values has an *expected* cash flow profile that looks like a single number, but the *variance* of that profile is bounded enough that you can think of it as an approximate fixed-income position with a known expected yield. See [why-pm-index-funds-are-dubious](/concepts/why-pm-index-funds-are-dubious) for the long version of why this almost-but-not-quite works.

Outside those three corners, DCF on a single binary is the same thing as quoting its implied yield with extra steps. Stop importing equity and corporate-finance frameworks onto an instrument that has its own perfectly good fixed-income vocabulary already.