# The Shape of a Prediction Market Yield Curve

> The first time I plotted implied yield against tau across an event family, the curve had the same steep contango shape as a freshly-issued credit-risky bond ladder. Notes from the morning that happened.

**Category:** insights | **Author:** Patrick Liu | **Published:** 2026-04-09

---
Sometime around January I sat down with a coffee and a single sheet of graph paper and tried to draw, by hand, the prediction-market equivalent of a yield curve. I knew what a treasury yield curve looked like — anyone who has worked near a fixed-income desk has the rough shape burned into their visual cortex. I wanted to know what the same plot looked like for a family of binary contracts on the same underlying event at different time horizons.

The result, which I kept on my desk for weeks afterward, was the moment I realized the term-structure thinking I had been borrowing piecemeal from bond traders was actually a coherent framework for prediction markets — not a metaphor. This is what I drew that morning and what it taught me.

## The Event I Picked

I picked the SpaceX-IPO family on Polymarket. It is a useful event family for this exercise because there are at least five contracts on it at different horizons, the contracts share clean resolution criteria (same definition of "IPO," same definition of "first day of trading"), and the [CYC regex grouper](/concepts/pm-indicator-stack) catches all of them under one event tag, which means I can pull them as a coherent family with one query rather than hand-collecting them.

The contracts I had open that morning, with prices and τ-days as of that day:

| Contract | YES Price | τ (days) |
|---|:---:|:---:|
| SpaceX IPO by end of Q2 2026 | 0.04 | 60 |
| SpaceX IPO by end of 2026 | 0.18 | 240 |
| SpaceX IPO by end of Q1 2027 | 0.32 | 330 |
| SpaceX IPO by end of 2027 | 0.51 | 600 |
| SpaceX IPO by end of 2028 | 0.69 | 965 |

Five contracts. Five horizons. Same underlying event. The probability of "Elon takes SpaceX public by date D" is monotonically increasing in D, because the set of futures in which an IPO happens grows as the deadline pushes out. All five prices reflect the market's collective belief about *when* the IPO is, decomposed into a series of "by date X" cumulative probabilities.

This is, structurally, exactly the same shape as a series of zero-coupon bonds with different maturities on the same issuer. And like a series of zeros, the right way to look at the relationship between them is not the prices themselves but the *yields*.

## Computing the Curve

I computed the [implied yield](/learn/implied-yield) for each contract, by hand, on the same sheet of graph paper. Formula: `IY = (1 / p) ^ (365 / τ) - 1`.

| Contract | p | τ | IY (annualized) |
|---|:---:|:---:|:---:|
| Q2 2026 | 0.04 | 60 | (1/0.04)^(365/60) - 1 ≈ 5.9 trillion% |
| End 2026 | 0.18 | 240 | (1/0.18)^(365/240) - 1 ≈ 1,160% |
| Q1 2027 | 0.32 | 330 | (1/0.32)^(365/330) - 1 ≈ 252% |
| End 2027 | 0.51 | 600 | (1/0.51)^(365/600) - 1 ≈ 38% |
| End 2028 | 0.69 | 965 | (1/0.69)^(365/965) - 1 ≈ 14.8% |

The Q2 2026 number is silly in the way that all very-near-dated low-priced binaries are silly. I crossed it out with a pencil and made the curve start from the second point. But the *rest* of the curve, with the absurd left tail removed, was actually shaped like something I recognized: a steep declining yield against horizon. The classic *contango* shape from commodity futures, or the "credit-risky" steep zero-coupon shape from a junk-bond ladder right after issuance.

I drew it, with τ on the x-axis and IY on the y-axis (log scale on y because the numbers were spanning several orders of magnitude even after I dropped the Q2 point). The shape was unmistakable. A steep down-slope, flattening as τ pushed out, asymptoting toward something like 10–15% in the multi-year tail.

I sat there with the graph paper for a while.

## What the Shape Says

A bond trader looking at this curve would describe the shape in one sentence: *the market is paying you a lot to be patient, and most of the patience premium is concentrated in the front end.*

In bond-desk vocabulary, this is exactly what a freshly-issued, medium-credit-quality zero-coupon ladder looks like in its first weeks of trading: very high implied yields on the short-dated maturities (because the issuer's near-term credit story has not yet been fully digested), tapering quickly to a flat back end (because the long-dated maturities are dominated by the *probability* of eventual recovery rather than by *timing* uncertainty).

In prediction-market terms, the same shape is saying: *the market thinks the SpaceX IPO is much more likely to happen in 2027 than in early 2026, but it has not yet crystallized which specific 2027 quarter, so the gap between the 2027 price and the 2028 price is large compared to the gap between Q2 2027 and Q3 2027*. The steep front end is uncertainty about *when* in the multi-year window. The flat back end is increasing confidence that the event has happened by *some* point in the window.

That decomposition is invisible if you look at the contracts one at a time. You need the whole curve.

## The Bond-Desk Vocabulary Mapping

Once I had the shape, I started writing in the margins of the graph paper, trying to map what I was seeing onto the language a bond trader would use. It came together pretty fast because the math is genuinely the same:

The *term structure* is the curve itself — IY plotted against τ for the family. The *front end* is the short-dated portion; the *back end* is the long-dated portion. The *butterfly* is the curvature: how much the middle maturity sags relative to the line connecting the two ends. The *roll-down* is what happens to a position on the curve as time passes and a long-dated contract gradually becomes a shorter-dated contract — its IY mechanically rises if the price stays put, which is the prediction-market equivalent of "earning the carry." A *steepener* is a trade that profits when the front end moves up relative to the back end. A *flattener* is the opposite.

Every one of these terms has a direct, mechanical translation to a position you can take across the SpaceX IPO contracts. A flattener trade — sell the steepest part of the front end, buy the flat back end — is a position on the thesis that the *crystallization* of the IPO timing is going to happen sooner than the market expects. The market is currently uncertain about *which year*; if news arrives that pulls forward the expected timing, the front-end IYs collapse and the back-end IYs are roughly unchanged, which is exactly what a flattener wants.

I had been making trades like this for months without realizing I was doing the prediction-market equivalent of a bond curve trade. The vocabulary simply was not there. Once I had the vocabulary, the trades made more sense and I could explain them to other people in less than five minutes instead of half an hour of hand-waving.

## The CYC Prerequisite

I would not have been able to do any of this without the [cycle clustering](/concepts/pm-indicator-stack) regex grouper, because before CYC I had no way to programmatically identify "all the contracts in the same event family." I would have had to hand-collect them, and in practice that means I would have done it once for SpaceX as a one-off and never built the systematic version.

CYC is what lets me say "give me the term structure for any event family" as a single query. It is also why CYC is the indicator I think is *most* prediction-market-native — it has no clean equity-market analog, because equities do not have the discrete event-resolution structure that creates contract families in the first place. You can build a yield curve from corporate bonds because the same issuer issues debt at multiple maturities. You can build a yield curve from SpaceX IPO contracts because the same event has multiple "by date" cumulative resolutions. The mechanism is structurally the same, even though the underlying instrument is different.

I have a longer essay sitting half-written about why this CYC-enabled curve view is the core thing prediction markets do that no other instrument does, but that one is for another day.

## The Page That Will Eventually Show This

The next step on this whole project is to put the curve view on a public-facing page. Right now I generate it as an ad hoc plot in a notebook. The plan is for /yield-curves/[event] to take an event slug, pull the family via CYC, compute the IY for each contract in the family, and render the curve with a few obvious views: log-y, linear-y, butterfly, roll-down forecast. Same thing a bond desk's terminal would show, except for prediction-market families instead of treasury maturities.

That page is not live yet. It is the thing the [CYC grouper](/concepts/pm-indicator-stack) was secretly built for, and it is the natural endpoint of [the indicator stack post](/blog/why-i-built-the-indicator-stack) — the place where the five numbers stop being a screener and start being a *visualization layer*.

## What I Still Don't Know

The thing I keep wondering about is whether the *shape* of these curves carries information beyond what each individual point's IY does. In bond markets, the *slope* of the front end is itself a signal — it predicts changes in central bank policy with measurable accuracy. The *curvature* (the butterfly) is a signal too. These are not just visual properties; they are tradeable information that does not exist in any single bond's yield.

I do not yet know whether prediction-market term structures carry the same kind of meta-information. I suspect they do, because the same math is operating on the same kind of instrument. But I have not done the work to test it, and I have a sneaking feeling that the answer is going to require a lot more event families than the SpaceX IPO one — something like a year of weekly snapshots across a hundred event families before the patterns become statistically legible.

That is the project I am circling now. The curve shape on graph paper that morning was the first hint that there was a real research question here. The single sheet of paper is still in a drawer somewhere. I should put it back on the desk.