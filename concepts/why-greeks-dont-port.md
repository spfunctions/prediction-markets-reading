# Why Greek Letters Mostly Don't Port: Delta, Gamma, Vega, and Binary Contracts

> Options have a Greek apparatus because the underlying is a continuous price process. Binary prediction-market contracts have no underlying spot. Of the five major Greeks, only theta has a meaningful PM analog — and it is better captured by τ-days plus implied yield.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 12 min

---
Options traders who come to prediction markets always try to compute the Greeks first. I understand the impulse. The Greeks are the language of risk in options world, and the muscle memory says "before I size, I want to know my delta and my vega." Then they sit down with a Kalshi binary, try to compute delta, and either get nonsense or quietly redefine the term until it means something it never meant in options.

The honest answer is that four of the five major Greeks do not port to binary prediction-market contracts, one ports trivially in a way that is not very useful, and one — theta — has a real analog that is better captured by other tools the prediction-market world already has. This page is the long version of that claim.

## The Greeks, Briefly

Before we kill them, let me make sure we agree on what they mean in options world.

**Delta** is the first derivative of the option price with respect to the price of the underlying. If a call option has delta of 0.6, a $1 move in the underlying produces approximately a $0.60 move in the option price. Delta is also commonly read as "the equivalent number of shares of underlying" — a 0.6-delta call is "60 shares' worth" of long exposure for hedging purposes.

**Gamma** is the second derivative of the option price with respect to the underlying. It captures how delta itself changes as the underlying moves. High gamma means your delta is unstable; you have to rehedge constantly. Gamma matters most for at-the-money options near expiry, where small moves in the underlying produce large changes in delta.

**Vega** is the sensitivity of the option price to *volatility* of the underlying. Higher implied volatility means higher option premium, all else equal. Vega is not technically a Greek letter — it is a Latin V — but everyone calls it one.

**Theta** is the sensitivity of the option price to time. Specifically, it is the daily decay of option value, holding everything else constant. Long options bleed theta; short options collect it.

**Rho** is the sensitivity to interest rates. It matters in long-dated options and in environments where rates are volatile.

The five together let an options trader decompose any position into "exposure to spot, exposure to vol, exposure to time, exposure to rates" and reason about each axis separately. The decomposition is the entire point of the framework.

## Why Delta Doesn't Port

A binary prediction-market contract pays $1 if the event resolves YES and $0 if it resolves NO. There is no continuous underlying spot price. The "underlying" is an event whose state is either pre-resolution (price floats) or post-resolution (price is exactly 0 or 1). The closest thing to "delta" you can write down is the partial derivative of the contract price with respect to your subjective probability of YES — but that is not what delta means in options. That is just "how much would my fair value change if I changed my opinion."

If you push the analogy harder, you can define a kind of *effective delta* on a binary as follows: if you bought YES and the contract resolves YES, your "delta" was 1 the whole time, because you collected the full $1 payoff. If you bought YES and it resolves NO, your delta was 0 the whole time, because you collected nothing. There is no continuous spectrum. The delta is a posterior label, not a forward-looking sensitivity.

This is why options traders' delta-hedging instincts misfire on PMs. There is nothing to hedge against. An option position is exposed to spot moves and you can offset that exposure with shares of the underlying. A PM position is exposed to one binary outcome and there is no continuous instrument that tracks the same outcome — you can only short the opposite side of the same contract, which is not a hedge, it is just *exiting* the position.

The closest you can get to delta-style hedging across PMs is a *cross-market hedge*: long a Fed-cut contract at the next meeting, short a Fed-cut contract at the year-end horizon, betting that the curve flattens. That is a valid trade, but the right framing for it is *yield curve trading*, not delta hedging. See the [yield-curve-prediction-markets](/learn/yield-curve-prediction-markets) glossary entry and [the-shape-of-a-prediction-market-yield-curve](/blog/the-shape-of-a-prediction-market-yield-curve) blog post for the right framing.

## Why Gamma Doesn't Port

Gamma exists because delta is a continuous, differentiable function of the underlying price. On a binary, "delta" is not differentiable in any useful sense — it is a step function that jumps from "whatever you want to call your forward-looking exposure" to "1 or 0" at the moment of resolution. There is no smooth curve to take a second derivative of.

Some people try to define gamma as the second derivative of the binary price with respect to the *probability*. The math actually works out — for a binary contract priced at p, the price function is itself the probability, so the first derivative is 1 and the second derivative is 0. Gamma equals zero by construction. That is not "gamma does not exist," it is "gamma exists and is identically zero," which is even less useful, because the whole point of gamma is to flag positions whose hedging requirements are unstable, and "identically zero" carries no information.

The thing gamma was invented to capture — *convexity in the price-versus-underlying curve* — is captured on a PM by the [Cliff Risk Index](/concepts/cliff-risk-index), which measures how fast the contract is approaching its eventual cliff at 0 or 1. CRI is computed as `|Δp/Δt| × τ_remaining`, and it gives you the prediction-market answer to "how convex is my position right now," in a way that gamma does not because gamma does not exist on a step function.

## Why Vega Doesn't Port

Vega prices the cost of "the underlying might move around a lot before expiry." On an option, this matters because more underlying volatility means more chance of finishing in the money. The price of an option contains an embedded forecast of future volatility, and vega is the sensitivity to that forecast.

A binary PM contract has no continuous underlying. There is nothing to be volatile. The "volatility" of a binary contract is just *price noise around the consensus probability*, and that price noise does not represent uncertainty about an underlying that might or might not finish in the money. The contract finishes in the money based on a single binary event, not based on whether some price process happens to land above or below a strike at expiry. Volatility *of the contract price* is real (and CRI captures it), but volatility *of the underlying* is not even definable because there is no underlying.

If you try to define vega anyway — say, as the sensitivity of the contract price to the standard deviation of recent price moves — you end up with a number that mostly tracks "how much news has been hitting this market lately." That is not vega. That is realized volatility on the contract price itself, which is the same thing CRI is built around. Save yourself the renaming and just use CRI.

## Why Rho Doesn't Port (Mostly)

Rho is the only Greek that has a real analog, but it is a small one. Holding a long PM position with τ = 200 days is opportunity-cost-equivalent to having that capital out of treasuries for 200 days. If treasury rates move, the *yield comparison* between your PM position and the risk-free rate moves. That is rho-shaped behavior.

The catch is that rho is usually small enough to ignore unless you are running a very large book at very long horizons. Most PM contracts have τ under 90 days, and the rho on a 90-day position is a second-order effect on top of the first-order question of "is my thesis right." I track it on long-dated positions and ignore it on anything under three months.

## Why Theta *Does* Port — But There is a Better Tool

Theta is the only Greek that has a clean PM analog. An option loses value over time as the optionality erodes. A binary PM contract has its own form of decay: the implied yield collapses toward zero as τ shrinks, because the same payoff over a shorter window is worth less in annualized terms (or, equivalently, because the market is moving toward its eventual snap to 0 or 1 and you have less time for new information to push the price).

You could compute "theta" on a PM as the expected change in mid-price per day, holding everything else constant, and you would get a real number. The problem is that you already have a better tool for the same job: the combination of [τ-days](/concepts/tau-days) and [implied yield](/concepts/implied-yield). τ-days is the time-to-resolution itself; IY is what you are getting paid for that time. Together they decompose the same risk theta is trying to capture, in units that are native to the prediction-market ecosystem and that put you on the same yield curve as fixed income.

The reason τ + IY beats theta on PMs is that they handle the exponential time-to-resolution dependence correctly. A binary contract's value is much more time-sensitive at short τ than at long τ, and theta-as-a-flat-daily-decay misses that nonlinearity. IY exponentiates over τ, which means the same per-day price change implies a wildly different annualized yield depending on whether τ is 5 days or 500 days. That is the right behavior for an instrument whose payoff is binary at a single horizon.

## What to Use Instead — the Indicator Stack

If you came from options world expecting a Greek apparatus, the right replacement on PMs is not "compute the Greeks anyway." It is the [pm-indicator-stack](/concepts/pm-indicator-stack), which gives you five numbers that decompose risk along axes that *do* exist on PMs.

For "convexity / gamma" — use [Cliff Risk Index](/concepts/cliff-risk-index). High CRI flags markets that are actively repricing, which is the prediction-market equivalent of "your position is at the steep part of the price curve."

For "carry / theta" — use the combination of [implied yield](/concepts/implied-yield) and [τ-days](/concepts/tau-days). IY is the annualized return on the position; τ is how long you have to hold it. Together they tell you what you are being paid per unit of time.

For "delta / hedging exposure" — there is no clean replacement, because there is no continuous underlying. The closest thing is *cross-tenor positioning* on a yield curve or *multi-outcome positioning* on a sibling-event basket, which is captured by [Event Overround](/concepts/event-overround) and the cycle-clustering tooling.

For "vega / volatility exposure" — this just does not exist. Stop looking for it. The thing you are trying to express is usually CRI or [Liquidity Availability Score](/learn/liquidity-availability-score), depending on whether you mean "the market is moving" or "the market is illiquid."

## A Worked Example: Translating an Options-Trader Pitch into PM-Native

Imagine the following pitch from someone who came from an options desk: "I want long delta on the Fed cut, short vega on rates volatility, neutral theta, and small positive gamma." That sentence is meaningful on options. On PMs it is gibberish. Here is what it translates to.

"Long delta on the Fed cut" → buy YES on the relevant Kalshi Fed-cut contract. There is no partial-delta version; you are either in or out.

"Short vega on rates volatility" → does not exist. There is no continuous rates volatility input to the PM contract; the contract resolves on a single Fed decision date and the *volatility* of the rates path between now and then is not separately priced.

"Neutral theta" → does not exist as such. The PM analog would be choosing a τ that gives you the carry profile you want, which is a *position-level* decision, not a Greek you can dial in independently.

"Small positive gamma" → high CRI. You want a contract that is actively repricing. `sf scan --by-cri desc` will give you a sorted list.

The translation is not one-to-one. Two of the four legs simply have no PM analog, and the two that do are better expressed as direct selections in the indicator stack rather than as Greek-style sensitivities. That is the punchline: the Greek apparatus was built for a different instrument shape, and trying to port it generates more confusion than clarity.

## Where This Negation Itself Has Limits

I do not want to be doctrinaire about this. There are two specific places where Greek-style thinking does survive on PMs, and I want to name them so the page is not a strawman.

The first is *delta hedging across venues* — if the same outcome trades on Kalshi and Polymarket, you can long one and short the other and the position is approximately delta-hedged in the sense that "any move in the underlying belief moves both legs equally and your P&L is the spread." That is a genuine hedge, and it has an analog in cross-listing arbitrage that bond and equity traders run constantly. See [from-adr-arbitrage-to-cross-venue-pms](/concepts/from-adr-arbitrage-to-cross-venue-pms) for the longer discussion.

The second is *time decay on multi-outcome events*. The vig wall on a multi-outcome event (the practical floor on Event Overround, see [event-overround](/concepts/event-overround)) does erode predictably over time, and the erosion is theta-shaped. A trader who wants to capture the erosion is doing something that genuinely looks like collecting theta. That is one place the analog is real and you can use it without renaming it.

Outside those two cases, the Greeks belong on options and the indicator stack belongs on PMs. Stop trying to use the wrong toolbox. The five numbers in [pm-indicator-stack](/concepts/pm-indicator-stack) are designed for the instrument you are actually trading.