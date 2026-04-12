# How to Actually Make Money on Kalshi (Not the Advice You'll Find on Reddit)

**Category:** markets | **Author:** Patrick Liu | **Reading time:** 14 min | **Published:** 2026-04-12

---
Every Kalshi guide on the internet says the same thing: "find an event you know about, buy YES or NO, wait." That advice will lose you money at exactly the rate of the bid-ask spread plus fees, which on most contracts is 3-7 cents per round trip.

I have been trading prediction markets for two years and building tools for them for one. Here is what I wish someone had told me on day one.

## Step 0: Stop Thinking in Probabilities

The single most expensive habit in prediction market trading is looking at a contract at 42¢ and thinking "do I believe this has a 42% chance?" That framing makes you a pollster, not a trader. And pollsters do not have a P&L.

Instead, convert to [implied yield](/learn/implied-yield):

```
IY = ((1 - price) / price) × (365 / days_to_resolution)
```

A contract at 42¢ resolving in 18 days has an annualized implied yield of roughly 28,000%. The same contract resolving in 200 days yields 80%. These are not the same trade. The probability is identical. The *trade* is completely different.

I wrote the [full story of this realization](/blog/the-day-i-stopped-trusting-raw-probabilities). It is the single most important thing I have learned.

## Step 1: Screen, Don't Browse

Kalshi has roughly 50,000 active contracts. Browsing the homepage is like trying to find a stock to buy by scrolling the NYSE ticker tape.

Use a screener. Here are the three queries I run every morning:

**High yield, short duration:**
```
sf screen --iy-min 200 --tau-max 30 --venue kalshi
```
Or via API: [/api/public/screen?iy_min=200&tau_max_days=30&venue=kalshi](/api/public/screen?iy_min=200&tau_max_days=30&venue=kalshi&sort=iy)

This surfaces contracts where the market is offering >200% annualized yield on instruments that resolve within a month. Most of these are low-probability contracts (5-15¢) where a small edge produces enormous returns.

**Liquid with orderbook:**
```
sf screen --has-orderbook --las-max 0.05 --sort las
```
Or: [/api/public/screen?has_orderbook=true&las_max=0.05&sort=las](/api/public/screen?has_orderbook=true&las_max=0.05&sort=las)

[LAS](/learn/liquidity-adjusted-spread) (Liquidity-Adjusted Spread) measures the spread as a fraction of the price. Lower is better. LAS < 0.05 means the spread is less than 5% of mid — these are the contracts where you can get in and out without the spread eating your edge.

**Unloved markets (no one is watching):**
```
sf screen --without-thesis --venue kalshi --tau-min 7
```
Or: [/api/public/screen?no_thesis=true&venue=kalshi&tau_min_days=7](/api/public/screen?no_thesis=true&venue=kalshi&tau_min_days=7&sort=iy)

These are markets with no thesis coverage — meaning no one in our system has an active model on them. [Null is a signal](/concepts/null-as-signal): the absence of attention is itself an edge. These are the markets where a retail trader with domain knowledge can genuinely beat the field.

## Step 2: Check the Orderbook Before You Click Buy

The price on the contract page is the last trade. The price you will *get* is the ask. On illiquid contracts, these can be 5-10 cents apart.

Before entering any position, check:
- **Spread:** bid-ask gap in cents. Under 3¢ is good. Over 5¢ means your entry cost alone eats most of your edge.
- **Depth:** how many dollars sit at each price level. If there is $50 at the ask and you want to buy $500, you are going to walk the book.
- **LAS:** the ratio. Under 0.05 is liquid. Over 0.15 is illiquid.

You can pull this for any ticker: [/api/public/market/KXFEDDECISION-26JUN18-Y?depth=true](/api/public/market/KXFEDDECISION-26JUN18-Y?depth=true)

Or use the CLI: `sf inspect KXFEDDECISION-26JUN18-Y`

## Step 3: Size by Regime

Not every market is safe to trade at every moment. Our [regime system](/concepts/maker-taker-regime-in-pms) classifies each market into three states:

- **Maker regime:** Low adverse selection. The orderbook is stable, depth is growing, no one with inside information is aggressively taking liquidity. Safe to post limit orders.
- **Taker regime:** High adverse selection. Someone knows something. Depth is evaporating. Flow is directional. If you are not the informed trader, you are the liquidity being consumed.
- **Neutral:** Insufficient signal.

The rule is simple: **size normally in maker, size small in neutral, don't trade in taker unless you are the one with the edge.**

Check any market's regime: [/api/public/regime/scan?label=maker&sort=score](/api/public/regime/scan?label=maker&sort=score)

## Step 4: Multi-Outcome Events Are Where the Free Money Lives

When an event has three or more outcomes (e.g., "Who will win California governor?"), the YES prices should sum to roughly 100¢. When they sum to more, the house is taking a cut ([overround](/concepts/event-overround)). When they sum to less, you are getting free expected value.

Screen for it:
```
sf screen --or-min 0.05
```
[/api/public/screen?or_min=0.05&sort=or](/api/public/screen?or_min=0.05&sort=or)

An overround of 0.05 means the YES legs sum to 105¢. If you buy a basket of all outcomes, you are guaranteed to pay $1.05 for a $1.00 payout — that is the vig. But if you sell the most overpriced leg, or buy the most underpriced one, the overround tells you exactly how much structural edge is in the event.

## Step 5: Don't Hold to Resolution

The biggest mistake I see new traders make: buying at 30¢, watching it go to 50¢, holding to see if it resolves YES. That is not trading. That is gambling with a slow feedback loop.

Prediction markets have continuous secondary trading. If your 30¢ contract moves to 50¢, you have a 67% return. Take it. Redeploy the capital into the next highest-IY opportunity. The annualized return of capturing 20¢ moves across multiple positions vastly exceeds the return of holding one position to binary resolution.

The [world state](/api/agent/world) updates every 15 minutes with the latest edges, movers, and divergences. Use it as your rebalancing signal.

## The Setup

Install the CLI:
```bash
npm install -g @spfunctions/cli
sf login
sf screen --iy-min 200 --tau-max 30
```

Or use the API directly from any agent or script. The [screener](/api/public/screen), [inspect](/api/agent/inspect/CONTROLH-2026-D), and [world state](/api/agent/world) are all public endpoints.

The tools are free. The edge is in how you use them.