# Why "Prediction Market Index Funds" Are Mathematically Dubious

> Index funds work in equity because constituents share macro exposures and have stable market caps. Binary prediction-market contracts have neither. A naive PM index converges to noise, not to a meaningful return stream.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 11 min

---
Every six months a new pitch deck floats around for a "prediction market index fund." The pitch is some version of "we hold a basket of binary contracts across categories — politics, economy, weather, sports — and offer investors diversified exposure to the prediction-market asset class." It sounds reasonable on the surface and it does not survive contact with the math.

This page walks through why a PM index fund is mathematically dubious as currently described, what the closest *honest* version would actually look like, and what the better alternative is for someone who wants diversified exposure to PMs without picking individual contracts.

## What an Index Fund Is, Briefly

In equity, an index fund holds a basket of stocks weighted by some rule — usually market capitalization — and provides diversified exposure to the underlying constituents. The framework works because three things are true.

First, equity returns are continuous, so the basket return is just the weighted average of constituent returns and is itself a continuous quantity that can be tracked, reported, and benchmarked.

Second, the constituents share *macro factor exposures* — when GDP growth surprises positively, most of the basket benefits, and when interest rates rise, most of the basket suffers. The diversification reduces idiosyncratic risk while preserving exposure to a meaningful underlying factor.

Third, the constituents have *meaningful weights*. Market cap is an objective measurement that ranks Apple as larger than a small-cap industrial in a way that reflects something about the underlying economic exposure. The weighting rule is not arbitrary; it captures real economic significance.

The PM analog fails on all three.

## Failure 1 — Returns Are Discrete and Bounded

A binary prediction-market contract does not have a continuous return distribution. It has a Bernoulli payoff: either $1.00 or $0.00 at resolution, with all the probability mass concentrated at those two points. If you bought a basket of 50 binaries at various prices and held them all to resolution, your "return" would be the sum of (number of YES resolutions × $1) minus the total amount you paid, divided by the amount you paid. That is a real number, but it has weird statistical properties.

The variance of this return is dominated by the *number of correct predictions*, not by any continuous price process. With 50 independent binaries, each at a fair price (i.e., each one resolves YES with probability equal to the price you paid), the *expected* return is exactly zero by construction (you are paying the fair price). The *variance* is determined by the spread of binomial outcomes — sometimes you win 30 out of 50, sometimes 20, and the dispersion is the entire risk.

So the basket return is "flat in expectation, with binomial-shaped noise." That is not the same shape as an equity index, which has a positive expected return (the equity risk premium) and continuous-distribution noise. The PM basket has *no risk premium that the diversification can preserve*, because the diversification is averaging across binaries that are individually priced at their fair value.

## Failure 2 — sqrt(N) on the Wrong Direction

The other reason equity diversification works is that idiosyncratic risk reduces by sqrt(N) as you add uncorrelated constituents, while the systematic factor exposure stays constant. Over enough constituents, the basket converges on the factor return, which is the meaningful thing.

On a basket of fair-priced binaries, the math works similarly but the *meaningful thing* it converges to is zero. Each binary is fairly priced, so its expected return is zero. The variance reduces by sqrt(N) just like in equity, but the expected return reduces by zero (because zero is already zero). What you end up with is a portfolio whose expected return is zero and whose variance shrinks toward zero as you add constituents.

This is not "the index converges to a stable return." This is "the index converges to a flat line near zero." The diversification has not given you exposure to anything; it has averaged away all the variance that any single contract had, *and* it has averaged away all of the expected return (which was already zero by the fair-pricing assumption). The sqrt(N) reduction is happening on both axes simultaneously, and you end up at the origin.

The only way this changes is if the constituent contracts are *systematically mispriced* in the same direction — e.g., the longshot bias from horse racing literature, where contracts under 10 cents tend to resolve at lower than their implied rate. If the basket systematically buys mispriced contracts, the expected return is non-zero. But that requires a *trading strategy*, not an index fund — you have to actively select which contracts to hold based on a model of mispricing, which is the opposite of indexing.

See [longshot-bias](/concepts/longshot-bias) for the mispricing dynamic and what is currently known about it on Polymarket and Kalshi.

## Failure 3 — There Is No Market Cap

The third structural problem is that there is no objective weighting rule. Equity indices use market cap because market cap measures something — the total dollar value of the company — that is comparable across constituents. PM contracts have no analogous quantity.

You could weight by 24-hour volume, but that gives you a basket that is dominated by a small number of very-high-volume political contracts and underweights everything else. You could weight by total notional outstanding, but that gives you a basket that is even more concentrated. You could weight equally, but then a $10,000 position in a Senate contract counts the same as a $10,000 position in a tiny obscure event, which is not what diversification means.

Each of these weighting rules produces a basket with very different return properties, and none of the rules has the same kind of economic justification that market cap has in equities. The weighting is *arbitrary* in the precise sense that no choice is more theoretically defensible than the others. An equity indexer can say "we use market cap because market cap measures economic significance." A PM indexer can only say "we picked this rule because it gave us a number." That is not a defensible foundation for a fund.

## What a Naive PM Index Would Actually Look Like

Let me make this concrete. Suppose we hold a basket of 10 binary contracts, equally weighted at $1,000 each, across categories:

- 2 election contracts (Senate races)
- 2 economic contracts (Fed decisions, recession probability)
- 2 weather contracts (hurricane landfall, temperature targets)
- 2 sports contracts (championship outcomes)
- 2 geopolitical contracts (country-specific elections, conflict outcomes)

Each is currently priced "fairly" — i.e., the market consensus reflects all available information. The expected return on each contract is approximately zero (you pay the fair price; the expected payout equals the price).

What does the basket return look like over a 6-month horizon, assuming the contracts resolve sequentially?

The expected return is approximately zero. The variance is the binomial variance of "how many of the 10 contracts resolved YES versus the implied probability times 10." If each contract is at exactly 0.50, the expected number of YES resolutions is 5 and the standard deviation is about 1.58 contracts (the sqrt of n × p × (1-p) for binomial with n=10, p=0.5). So the basket realized return is 5 ± 1.58 contract-equivalents, against an investment of 10 × $0.50 × $1,000 = $5,000.

If 5 contracts resolve YES, you collect 5 × $1,000 = $5,000, which equals what you paid. Zero return.

If 6 resolve YES, you collect $6,000 against $5,000 paid. +20% return.

If 4 resolve YES, you collect $4,000 against $5,000 paid. -20% return.

The realized return in any single 6-month period is one of a small number of discrete outcomes, depending on how many contracts went your way. The expected value is zero, the variance is binomial, and the dispersion is uncomfortable (a one-standard-deviation move is about ±32% of the basket).

Marketing this as an "index fund" is deeply misleading. Investors who hear "index fund" expect smooth-ish returns with positive expected value. What they would actually get is a binomial lottery with zero expected value and 30%-ish swings. That is not an index fund. That is a casino game with extra steps.

## The Rebalancing Problem

It gets worse when you try to make the basket *continuous* over time. Equity indices rebalance by quietly trading constituents to stay at target weights. The transaction costs are low (basis points) and the rebalancing happens monthly or quarterly.

A PM index has a fundamentally different rebalancing problem: every contract eventually *resolves*, leaving a hole in the basket. To maintain the "10 contracts across 5 categories" structure, you have to constantly find new contracts to add as old ones resolve. Each new addition costs the bid-ask spread plus any slippage, and the spreads on PMs are not basis points — they are typically 50 to 500 basis points per contract on Kalshi, and similar on Polymarket. With 10 contracts and a 6-month average resolution horizon, you are rebalancing roughly twice a year on every contract, and each rebalance eats 100-300 bps of frictional cost.

Over a year, the friction cost of running a 10-contract PM index is on the order of 200-600 bps of net asset value. That is a fee structure that wipes out the (already zero) expected return. The math on this is not subtle: you are paying the venue spread to maintain a position whose expected return is zero, and the spread is much larger than zero.

## The Smarter Alternative: Edge-Weighted Baskets

I do not want to be all negation. There is a coherent version of "diversified exposure to prediction markets," and it looks nothing like an equity-style index fund. The honest version is an *edge-weighted basket* — a portfolio of positions sized by individual conviction, where the weights reflect *how mispriced each contract is* rather than how big the contract is.

Concretely, this is the [expected-edge](/concepts/expected-edge) framework. You scan the universe with [the-valuation-funnel](/concepts/the-valuation-funnel), find contracts where your subjective probability differs from the market price by a defensible amount, size each position by the Kelly fraction of (your edge × your confidence × liquidity factor), and hold the basket. The weights are *derived from edge*, not from any arbitrary capitalization rule.

This basket has positive expected return *because it is selecting on mispricing*, not because of any indexing logic. It is more like an actively managed long-only book than like an index fund. The expected return is the sum of (per-contract expected edge × position size), and the variance is reduced by the diversification across uncorrelated event-specific risks. This is the "dozens of small trades" framing from [thesis-confidence-vs-market-price](/concepts/thesis-confidence-vs-market-price).

The edge-weighted basket has a real Sharpe ratio because it is collecting alpha from mispricing. It has bounded variance because the positions are uncorrelated event-specific risks. It does not converge to noise because the underlying selection process is non-zero. It is also *not an index fund* — it is an actively managed alpha portfolio that happens to use prediction-market contracts as the underlying instrument.

If someone wants to package that as an investable product, they can — but they need to be honest that what they are selling is *active management* with PM contracts as the underlying, not a passive index. The fee structure should reflect that. The marketing should reflect that. The investor expectation should reflect that. Anyone selling a "passive PM index fund" is, at best, confused about what diversification can do on this asset class and, at worst, marketing a product whose math does not work.

## A Quick Sanity Check

If you want to feel the math directly, run the following simulation in your head. Imagine 100 binary contracts, all at fair prices, all resolving in 6 months, equally weighted in a basket. The expected number of YES resolutions is the sum of the implied probabilities, and the realized number is binomial around that expectation.

Over 1000 imaginary 6-month periods, the basket return distribution is centered at zero with a standard deviation of about 5-10% (depending on the price distribution of the constituents). The 95% confidence interval is roughly ±15-20% per period.

Now compare to an equity index over the same horizon. The S&P 500 has a 6-month standard deviation of about 10-12% in normal regimes, but it also has an *expected return* of about 4-5% over 6 months (the equity risk premium scaled to half a year). The Sharpe ratio of the equity index is positive. The Sharpe ratio of the PM basket is zero.

The PM basket is offering you the same volatility as equities with none of the expected return. Whatever it is, it is not a diversified investment vehicle. It is an exposure to a binomial lottery whose expected value is zero by construction.

## Where the Negation Is Too Strong

Two honest caveats so this page is not a strawman.

First, *if* the PM market is systematically mispriced in some direction (longshot bias being the leading candidate), then a basket that exploits the bias has positive expected return *as long as* the bias is actually present in the constituents. That is not an index fund — it is a bias-exploitation strategy — but it is at least a defensible basket. The catch is that the bias on Polymarket and Kalshi is small (the [longshot-bias](/concepts/longshot-bias) page gets into the empirics), and exploiting it still requires active selection.

Second, *correlated baskets* — e.g., a basket of all the Fed-cut contracts at every meeting in the next year — have a different mathematical structure than uncorrelated baskets, because they share systematic exposure to "the Fed cutting." Such a basket is more like a curve trade than a diversified basket, and it has a meaningful expected return profile that can be reasoned about (see [from-bond-ytm-to-pm-implied-yield] which is the existing [prediction-markets-need-fixed-income-language](/blog/prediction-markets-need-fixed-income-language) blog). But that is not an index fund either; it is a curve position.

Outside those two corners, the "PM index fund" pitch is closer to a category error than a financial product. Stop importing equity-style indexing onto an instrument class whose math does not support it. The right alternative is the edge-weighted basket — actively managed, mispricing-driven, sized by Kelly — and that is what [the-valuation-funnel](/concepts/the-valuation-funnel) and [pm-indicator-stack](/concepts/pm-indicator-stack) are designed to build for you.