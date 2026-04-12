# From Options Skew to Multi-Strike Prediction Market Events

> Options have a vol smile because OTM puts get bid by hedgers. Multi-strike prediction-market events have a structural skew because tail outcomes get bid by lottery buyers. The shape is the same; the mechanism is not.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 12 min

---
Anyone who has looked at an equity options chain has seen the volatility smile — the U-shaped curve of implied volatility plotted against strike price, where OTM puts and OTM calls both have higher implied vol than at-the-money options. The smile is one of the most studied artifacts in derivatives markets and the explanations for its existence range from "rational hedging demand" to "fat-tailed underlying distributions" to "sticky-strike vs sticky-delta dynamics." Whatever the cause, it is a real, persistent feature of every developed options market.

Multi-strike prediction-market events have something that looks visually similar. Plot the YES price against the strike of each outcome in a multi-strike event — say, a Kalshi BTC price-range market with strikes at $40K, $50K, $60K, $70K, $80K — and the curve has a U-shape. The middle strikes are priced low (the market thinks they are unlikely), and the tail strikes are priced higher than a naive interpolation would predict. The shape is similar enough to an options smile that quants who walk onto a PM market for the first time often try to fit a vol surface to it.

The shape is similar. The mechanism is not. This page walks through what an options smile actually is, what a multi-strike PM skew actually is, and why the analogy is more useful than people think *if you understand which parts port and which parts do not*.

## What an Options Smile Is

An equity option's price depends on (at least) the spot, strike, time, risk-free rate, and implied volatility. For a given underlying, the market has implicit beliefs about the volatility — these beliefs are *implied* by the prices of the options, and you can solve Black-Scholes backwards to recover them.

If the market believed the underlying followed a perfect lognormal distribution, every option on the same underlying would imply the same volatility, regardless of strike. The "smile" is the deviation from this flat baseline: in real markets, OTM puts have higher implied vol than at-the-money options, and OTM calls have higher implied vol than at-the-money options, with the result that a plot of implied vol against strike has a smile (or "skew" if it is asymmetric).

The standard explanations for the smile are:

- **Hedging demand**: portfolio managers buy OTM puts to protect against tail downside, bidding up the price and the implied vol.
- **Fat tails**: the actual distribution of returns is fat-tailed (more extreme moves than lognormal predicts), so the market correctly prices OTM options higher than Black-Scholes would.
- **Lottery demand**: retail traders buy OTM calls hoping for a big payoff, bidding up the price.
- **Risk premia**: out-of-the-money options carry insurance-like risk premia that the market charges for taking on tail risk.

All four explanations are partially true, in different proportions for different markets. The combined effect is the smile.

## What a Multi-Strike PM Event Looks Like

A multi-strike prediction-market event is one with multiple mutually exclusive outcomes, each corresponding to a different "strike." The canonical examples are price-range contracts (Kalshi BTC closing price > $X, for various X), volume contracts (next month's COVID cases > Y, for various Y), and election margin contracts (winning candidate's margin > Z%, for various Z).

Take a Kalshi BTC price-range event with five mutually exclusive outcomes:

| Strike | YES Price | Width | Density |
|---|:---:|:---:|:---:|
| BTC < $40K | 0.10 | $∞ to $40K | n/a |
| BTC $40K–$50K | 0.18 | $10K | 0.018 |
| BTC $50K–$60K | 0.30 | $10K | 0.030 |
| BTC $60K–$70K | 0.22 | $10K | 0.022 |
| BTC $70K–$80K | 0.12 | $10K | 0.012 |
| BTC > $80K | 0.08 | $80K to $∞ | n/a |

Sum of YES prices: 1.00 (perfectly summed). Notice the shape: the middle strike ($50K–$60K) is highest, and the tails are lower but *not as low as a strict normal distribution would predict*. If you fit a normal distribution to this, you would expect the wings to drop off faster than they do. They do not. The tails are "fatter" than a normal distribution, *which is exactly the same shape as the options smile*.

Plotted against strike, the implied probability density (after dividing by the bin width to get a per-dollar density) has the same U-shape as an options smile: peaked in the middle, with elevated density in the tails relative to a normal baseline. This is the "PM skew."

## What Generates the PM Skew

The mechanism is *not* the same as the options smile. There is no implied vol on a binary contract — the underlying is discrete and the price is just a probability. So the "smile" you are seeing in the multi-strike event is not an artifact of vol surface dynamics. It is something else.

The actual generator of PM skew is a combination of:

**Lottery demand on the tails.** Retail traders disproportionately buy "BTC > $100K by date X" type contracts hoping for a small position to turn into a big payout. This is exactly the same lottery-demand effect that bids up OTM calls in equity options markets. The mechanism is the same; the market is different.

**Hedging demand on the downside tails.** Crypto holders sometimes buy "BTC < $30K" contracts as cheap insurance against a crash. This is the PM analog of OTM put-buying for portfolio protection. Again, same mechanism, different market.

**Structural overround from MM economics.** Market makers in multi-strike events face adverse-selection costs on each strike, and they charge a premium proportional to the strike's tail risk. The *minimum* overround per strike is roughly proportional to the variance the market thinks the underlying might exhibit, and tail strikes get hit by the variance more heavily than at-the-money strikes. This shows up as a *systematic* elevation in tail YES prices that has nothing to do with retail behavior — it is just MM cost of doing business.

**Fat-tailed underlying.** The same fat-tail explanation that applies to equity options applies to PM events with continuous underlying (BTC, oil, jobs reports). The actual distribution of outcomes is fat-tailed, and the market correctly prices the tail strikes higher than a normal-distribution baseline.

The combination produces a curve that *looks* exactly like an options smile but is generated by a different set of mechanisms. The middle ground — lottery demand and fat tails — is similar in both markets. The hedging demand has a different demographic (crypto holders rather than fund managers). The MM overround is purely a PM artifact.

## What Ports: The Reading

If you understand options skew, you can read PM event skew immediately. The shape is the same and the trading implications are similar:

**Tail strikes are systematically overpriced.** This is true on both options markets (the smile bid) and PM events (lottery + MM overround). If you have a model that says the underlying is well-behaved and the tails are unlikely, you can sell the tails and capture the structural premium. The trade is the same on both markets.

**Middle strikes are sometimes the cheap trade.** When the smile is unusually pronounced, middle strikes are *underpriced* relative to a fair-distribution baseline. This is the "butterfly" trade in options markets and the analogous trade exists on PM events — buy the middle strike, sell the tails, lock in the structural premium.

**Skew changes are informative.** Movements in the smile shape (steeper, flatter, asymmetric) carry information about market sentiment in both markets. A sudden steepening of the BTC PM skew toward the upside is the analog of a sudden steepening of the equity put skew — it is the market pricing in tail risk in real time.

The translation table:

| Options concept | PM concept |
|---|---|
| Implied vol smile | Strike-vs-YES-price curve |
| ATM straddle | At-the-money strike YES |
| OTM put bid | Downside-tail strike YES |
| OTM call bid | Upside-tail strike YES |
| Butterfly trade | Long middle strike, short tails |
| Risk reversal | Long upside tail, short downside tail |
| Vega | None (no analog) |

The first six rows port. The vega row does not, because there is no continuous vol surface to differentiate against — the PM skew is a discrete object that lives at specific strikes, not a continuous function you can take derivatives of.

## What Does Not Port: The Continuous Vol Surface

In options markets, the smile is one slice of a 2D surface (vol vs strike vs tenor). Quants build models of the entire surface (SABR, SVI, Heston) and trade dislocations between adjacent points on the surface. The surface is continuous in both dimensions and the model fitting is a major industry.

PM events have neither dimension as a continuous surface. The strikes are discrete (a finite number per event, set by the venue) and the tenors are discrete (one resolution date per event, with no equivalent of "next-month vs three-month vs LEAPS"). The "surface" is a 2D grid with very few points, and you cannot fit continuous models to it because you do not have enough data.

This is the cleanest example of what does not port. If you came from options vol-surface trading, the literal infrastructure you used — SABR models, smile fitting, vol curves — has no PM equivalent. The conceptual frame (tail strikes priced higher than middle, lottery and hedging demand drive the shape) ports. The technical machinery does not.

## A Worked Multi-Strike Read

Take the BTC price-range event from the table above and ask: is the skew rich or cheap relative to history?

Step 1: compute the implied density per dollar of strike width. The middle strike ($50K–$60K) has density 0.030 per $10K width. The upper tail ($70K–$80K) has density 0.012 per $10K width. The ratio is 2.5.

Step 2: compare to historical readings of the same event. If the typical ratio between center and upper-tail density is 4 (the tails are normally a quarter as dense as the center), and today's ratio is 2.5, then the upper tail is 60% richer than usual. This is a *steep* upper-tail skew, suggesting the market is currently pricing in elevated upside risk.

Step 3: decide whether to fade or follow. If you have a thesis that BTC is *not* about to break out, you can sell the upper tail (KXBTCD-80K-26MAY YES) and lock in the elevated premium. If you have no thesis, you should at least be aware that the skew is rich and adjust your sizing.

This is the same workflow an options trader would do on equity skew. The math is rougher because PM events have fewer strikes than options, but the logic is identical. The interesting thing is that *most PM traders never do this analysis* because they treat each strike as a separate market rather than as part of a multi-strike surface.

The CLI command I run is `sf scan --by-event KXBTCD --skew`, which dumps the strike-vs-YES table for the event and the historical median ratio. If the current ratio is more than 1 standard deviation away from the historical median, the system flags it as a skew dislocation worth investigating.

## What Does Not Port: Time Decay and Vol Risk Premium

In options, theta decay is non-uniform across strikes, which generates a whole sub-discipline of cross-strike time-decay trading. PM events do not have this — all strikes within the same event share the same expiration date and decay at the same rate. Similarly, equity options have a volatility risk premium (implied vol systematically higher than realized vol) that drives "short vol" strategies. PMs do not have a vol risk premium because they do not have a vol. The closest analog is the *resolution-risk premium* on Polymarket and the *MM overround* on multi-strike events, but the mechanism is different and trying to import vol-risk-premium intuition will mislead you.

## Where the Bridge Breaks Hardest

Two specific cases where the analogy fails badly enough to be dangerous.

**Multi-strike events with thin tail liquidity.** The tails of an options smile are usually liquid because OTM options are heavily traded by hedgers. The tails of a multi-strike PM event are often *completely illiquid* — a $90K BTC strike might have $20 of total depth on the orderbook. You can read the skew but you cannot trade it. The PM analog of the smile is a *display artifact*, not a tradable surface, on most events.

**The minimum-overround floor.** Multi-strike events have a minimum overround that scales with the number of strikes. A 10-strike event almost always has Σpᵢ > 1 by 5–9 cents because each strike has its own MM cost of doing business. This is the [the-vig-wall](/concepts/the-vig-wall) phenomenon — you cannot trade EE = 0 in liquid multi-outcome events because the wall is structural. Options markets do not have this artifact because the strikes are not pre-defined by the venue; you can quote any strike you want.

## How This Connects to the Stack

The skew framing connects to the indicator stack via the [event-overround](/concepts/event-overround) and the [the-vig-wall](/concepts/the-vig-wall) phenomenon. Reading a multi-strike event is just an instance of reading the cross-section of YES prices in a single event — the same calculation that produces EE produces the skew shape. For the specific math, see [computing-event-overround-from-multi-outcome-events](/technicals/computing-event-overround-from-multi-outcome-events). For the case that PMs need a fixed-income vocabulary rather than an options vocabulary, see [/blog/prediction-markets-need-fixed-income-language](/blog/prediction-markets-need-fixed-income-language) and [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) — the options skew analogy is one of the few cases where a non-fixed-income framework actually does help.

## The Honest Summary

If you understand options skew, you can read multi-strike PM event skew immediately — the *shape* is the same, even though the *generating mechanism* is different. The shape similarity is enough to import a lot of intuition: tails are systematically overpriced, middle strikes are sometimes the cheap trade, skew changes are informative, butterflies are tradable.

What you cannot import is the technical machinery: vol surface fitting, theta differentials, vega-based strategies, vol risk premium math. None of those have PM equivalents and trying to use them on a PM event will produce models that are fitted to the wrong quantity.

The right move is to use the shape intuition without the machinery. Look at the strike-vs-YES table for any multi-strike event you are considering. Compute the ratio of center-density to tail-density. Compare to historical readings of the same event. If the skew is unusually steep, consider trading against it. If the skew is unusually flat, look for other dislocations. Use `sf scan --by-event <event> --skew` to surface candidates.

Options skew is one of the better-traveled bridges from another field to prediction markets. Most analogies break down quickly; this one holds at the level of *what you observe and how you interpret it* even though it breaks down at the level of *how you would build a model of it*. That is enough to be useful, and it is more than most cross-field analogies deliver.