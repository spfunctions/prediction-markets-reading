# Why P/E Ratios Don't Port to Prediction Markets — and What Does

> Equity P/E rests on an unbounded earnings stream and a continuous price. Binary prediction-market contracts have neither. The right analog is yield-to-maturity on a credit-risky zero, not P/E on a stock.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 11 min

---
Every few weeks I see a thread on X where someone fresh from equity research tries to apply a P/E framing to a prediction market. The pitch usually sounds reasonable. "This contract is trading at 30 cents and the implied earnings are 70 cents — that is a P/E of 0.43, the cheapest equity I have ever seen." A few replies down, someone else asks "what is the cheap one's P/E next year?" and the thread ends in mutual confusion.

The reason it ends in confusion is that P/E does not port to prediction markets at all. Not in a "be careful, the analogy has limits" way. In a "the formula is undefined and the intuition leads you straight off a cliff" way. This page is about exactly why, and what to use instead.

## The Steelman: Why People Reach for P/E

I want to be fair to the impulse. P/E exists because equity has two features that make a ratio of price to earnings informative.

First, an equity has an *earnings stream*. A company generates revenue every quarter, deducts costs, and reports a number that the market decides is worth a multiple. The multiple compresses two pieces of information into one — what you are paying today, and what you expect to receive periodically going forward. If the multiple is low relative to peers or history, you are paying less for the same income, and the trade is "buy the cheap one."

Second, equity prices are *continuous*. A stock at $30 can trade to $35 or $25 tomorrow, and the multiple updates smoothly with the price. The P/E framework lives in a world where partial conviction maps to partial position size and where a 10% return is a reasonable thing to want.

Both of these features are present in equity markets. Neither is present in a binary prediction-market contract. P/E was invented for one set of mechanics and people import it into the other set because, on the surface, both look like "buy a price, hope it goes up." The shapes are not the same.

## Why P/E Is Undefined on a Binary Contract

A binary prediction-market contract pays $1 if the event resolves YES and $0 if it resolves NO. There is one cash flow, ever, at one point in time. There is no quarterly earnings number. There is no "next year's E" because there is no next year — by next year the contract has resolved and become either dollar bills or paper.

Set aside the missing time series for a second and try to write down the formula. P/E = price divided by earnings per share. On a binary contract, the price is whatever you paid (say 0.32 today) and the "earnings" is the resolution payoff. The resolution payoff is either 0 or 1, and you do not know which one until the event happens. Plug in either value:

- If you assume the contract resolves YES, P/E = 0.32 / 1.00 = 0.32. "P/E of 0.32, looks cheap."
- If you assume it resolves NO, P/E = 0.32 / 0.00 = undefined. Division by zero.

The framework breaks at the second case for a reason that is not a glitch — it is the entire structure of the instrument. Equity P/E assumes a *positive earnings stream you can divide by*. A binary contract has a payoff that is, fifty percent of the time on average, exactly zero. There is no number to put in the denominator.

People who do this calculation usually skip the second case. They quietly assume the contract resolves YES, compute "P/E = 0.32," and go to bed feeling clever. They have not computed P/E. They have computed *one over the assumed-success payoff*, which is a number with no relationship to the equity multiple they think they are mimicking.

## What the P/E Framing Costs You

There are three concrete losses from importing P/E into PMs, and they are all ones I have watched a real person make.

The first is *wrong direction on cheapness*. In equity, low P/E means cheap, which usually means worth buying. In a PM, "low price" can mean cheap or it can mean correctly identified as unlikely. A contract trading at 5 cents is not 5x cheaper than one trading at 25 cents — it is 20 percentage points less likely to resolve YES, which is a property of the underlying event, not a discount the market is offering you. The P/E intuition trains you to buy low prices, which on a PM is the same as systematically buying longshots, which is a documented losing strategy in horse racing literature going back fifty years.

The second is *wrong sizing*. P/E framing makes you size by "how mispriced is the multiple," which on a binary contract maps to "how far from your fair value is the price." That is not a wrong question, but it leaves out the time axis entirely. A contract priced at 30 cents that you think should be at 50 cents is a different sized trade depending on whether resolution is in 5 days or 500 days, and P/E gives you no way to feel that difference. Implied yield does — see [implied-yield](/concepts/implied-yield) for the math.

The third is *wrong comparison set*. P/E lets you compare across companies, sectors, eras. The whole point of the multiple is to be commensurable. The PM analog of P/E is *not* commensurable across contracts on different events — there is no "earnings" you can normalize to. The right commensurable unit is annualized yield, because every binary at every horizon can be put on the same yield curve. P/E lets you compare AAPL to MSFT. It does not let you compare a Fed-decision contract to a Senate race.

## What Does Port: Yield-to-Maturity on a Credit-Risky Zero

The framing that *does* work, and that I think prediction-market traders should adopt aggressively, comes from fixed income. A binary prediction-market contract is mathematically the same instrument as a credit-risky zero-coupon bond.

Look at the cash flows. You pay 0.32 today for a piece of paper. At a known maturity date, the paper pays you par (1.00) if some condition is met, or zero if it is not. The "credit" is whether the issuer defaults; the prediction-market "credit" is whether the event resolves YES. The math of pricing the two is identical, and bond traders have been doing it for fifty years.

The right number to compute is implied yield: `IY = (1/p)^(365/τ) − 1`. This is the annualized return on the contract held to expiry, and it has every property P/E does not. It is defined for any p strictly between 0 and 1. It collapses the price and the time-to-resolution into a single number you can rank across contracts. It is the same unit a treasury desk uses for T-bills, so you can put a Fed-decision PM next to a 3-month treasury and decide whether the prediction market is paying you enough to justify the binary outcome risk.

The full derivation, three worked examples, and the four edge cases are on the [implied-yield](/concepts/implied-yield) concept page. The opinion piece [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) walks through why the IY framing wins specifically on Fed-decision contracts. The technical [computing-implied-yield-from-kalshi-tickers](/technicals/computing-implied-yield-from-kalshi-tickers) is the working code.

## A Side-by-Side

Let me put the two framings on the same contract so the contrast is concrete. Take a hypothetical Kalshi Fed-cut contract trading at 0.32 with τ = 45 days.

The P/E framing computes 0.32 / 1.00 = 0.32 and says "P/E of 0.32, very cheap, buy." That sentence carries no information about the holding period, no comparison to any benchmark, and no way to compare against a Senate-race contract that happens to also trade at 0.32 with τ = 220 days.

The IY framing computes (1 / 0.32) ^ (365 / 45) − 1 ≈ 1,030,200%, which is a number whose first reaction is "that is enormous and probably wrong." But the number is correct, and the correct interpretation is "this is a short-dated leveraged binary that pays out roughly 3.1× your money in 45 days if you are right." That framing tells you the trade is more like a short-dated option than a long-bond yield trade, which tells you to size it like an option (small fraction of capital, conviction-driven), not like a bond (larger fraction, yield-driven). The 220-day Senate contract at the same 0.32 price computes to IY ≈ 412%, which is far smaller and points to a yield-style position rather than a leveraged conviction trade. P/E says they are the same. IY says they are not.

## A Quick Sanity Check at the CLI

The fastest way to internalize the difference is to put the IY column on your screen for two weeks and try to forget the price column exists. `sf scan --by-iy desc --min-tau 14 --min-depth 100` ranks the active universe by yield with a sane lower bound on time-to-resolution and depth, and the resulting list is dominated by contracts you would have ignored if you were ranking by raw price. The contracts at the top of the IY list are usually unloved sub-50-cent positions on events nobody is talking about, which is precisely where the inefficiency lives — see [the-valuation-funnel](/concepts/the-valuation-funnel) for why stage 1 is supposed to surface exactly that.

## Where the IY Framing Itself Breaks

I do not want to leave the impression that IY is universally applicable. It fails in three places that are worth naming, because if you do not name them you end up writing the same essay six months later under a different title.

IY explodes as τ approaches zero. Mathematically the formula gives larger and larger numbers as the time-to-resolution shrinks, and any contract under about a day has an IY that is basically meaningless. Use a τ floor of 5 days or so when you are scanning, and switch to raw bid-ask edge for anything closer to expiry.

IY is a *conditional* return — it is what you make if the contract resolves in your favor. The expected return is (your subjective probability times the IY), which is a much smaller number. People who quote IY without the conditional are doing the prediction-market equivalent of quoting "this stock will return 200% if my thesis is right" without mentioning the probability of the thesis being right.

IY assumes you can transact at the displayed price. On a thin orderbook with a 12-cent bid-ask, the IY computed off the midpoint is fiction. The honest version uses the side of the book you would actually fill against, which is what stage 2 of [the-valuation-funnel](/concepts/the-valuation-funnel) is for and what [reading-prediction-market-orderbooks](/technicals/reading-prediction-market-orderbooks) walks through in detail.

None of these caveats apply to the equity P/E. They are real costs of the IY framing. They are also costs you can manage by just being aware of them, which is the sense in which IY is workable and P/E is not even definable.

## The Punchline

P/E is a great metric for an instrument with a continuous earnings stream and a continuous price. A prediction market contract is neither of those things. The right multiple is yield, the right curve is the term structure across τ, and the right comparison set is the rest of your fixed-income book. Stop importing equity vocabulary onto an instrument that already has its own vocabulary borrowed from a field that has been pricing this exact cash-flow shape for half a century. The bond traders did the work. The math is sitting there. Use it.