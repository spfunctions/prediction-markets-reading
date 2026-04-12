# How Prediction Markets Are Pricing the Strait of Hormuz Crisis — Live Data

**Category:** geopolitics | **Author:** Patrick Liu | **Reading time:** 11 min | **Published:** 2026-04-12

---
There are currently eleven distinct prediction market contracts tracking whether shipping through the Strait of Hormuz returns to normal. Combined volume across Kalshi and Polymarket: over $1.5 million in the last 24 hours.

I have been tracking the Iran cluster in our [world state](/api/agent/world) for three weeks. What started as a single Polymarket contract — "Iran x Israel/US conflict ends by April 7" — has metastasized into a full term structure of contracts spanning military operations, regime change, oil infrastructure, and shipping normalization, with resolution dates from next week to next year.

This is the most complete pricing surface the prediction market ecosystem has ever produced for a single geopolitical event. Here is what it looks like from the inside.

## The Term Structure

Start with the Hormuz shipping contracts:

| Contract | Venue | Price | 24h Volume | τ days |
|----------|-------|-------|------------|--------|
| Hormuz normal by end of April | Polymarket | 17¢ | $865K | 18 |
| Hormuz above 60 transits before May 1 | Kalshi | 14¢ | $55K | 19 |
| Hormuz above 60 transits before May 15 | Kalshi | 25¢ | $38K | 33 |
| Hormuz above 60 transits before June 1 | Kalshi | 40¢ | $35K | 50 |

This is a yield curve. The market is pricing a 17% chance of normalization by end-April, rising to 40% by June 1. The slope of this curve tells you the market's implied hazard rate for resolution — roughly 0.5% per day, accelerating slightly in the May-June window.

But Hormuz is downstream of the military and political contracts:

| Contract | Price | Volume |
|----------|-------|--------|
| Iran x Israel/US conflict ends by April 7 | 47¢ | $1.1M |
| Trump ends military operations by April 15 | 7¢ | $773K |
| Kharg Island no longer under Iranian control by April 15 | 3¢ | $725K |
| US invades Iran before 2027 | 34¢ | $678K |
| Iranian regime falls by June 30 | 9¢ | $638K |
| Iranian regime falls by April 30 | 3¢ | $1.0M |

The market is saying: the conflict has a coin-flip chance of ending soon (47¢ on the April 7 contract), but almost zero chance of escalating to regime change (3¢ on April 30 regime fall) or Kharg Island seizure (3¢). The base case is a negotiated de-escalation that takes weeks to restore shipping, not a decisive military victory.

## Cross-Venue Gaps

The Hormuz contracts exist on both Kalshi and Polymarket with slightly different specifications. Kalshi uses the IMF PortWatch 7-day moving average of transit calls with a threshold of 60. Polymarket uses "returns to normal" without a specific numeric threshold.

The price gap is material: Kalshi at 14¢ vs. Polymarket at 17¢ for roughly the same end-of-April resolution. The 3-cent difference reflects either the specification risk (what counts as "normal"?) or a genuine arbitrage. Our [cross-venue scan](/api/public/screen?has_orderbook=true&sort=las) monitors these daily.

## What the Oil Market Is Not Pricing

Here is what surprised me most when I pulled the data. The prediction market term structure implies that Hormuz disruption has a >80% chance of persisting through April. But WTI crude is at levels that suggest the physical oil market has largely shrugged this off.

Either the oil market knows something the prediction markets don't (alternative supply routes, strategic reserve releases, demand destruction), or the prediction markets are the leading indicator and oil has not yet repriced the tail risk.

For a trader, this divergence is the entire trade. If you believe prediction markets are more informationally efficient than commodity futures on geopolitical catalysts — and [there is evidence they are](/blog/prediction-markets-are-the-best-real-time-sensor-for-world-events) — then the oil market is mispriced.

## How to Track This

The full Iran/Hormuz cluster is visible in the [screener](/api/public/screen?keyword=iran&sort=volume) and the [world state](/api/agent/world). Individual contract dossiers are available via [inspect](/api/agent/inspect/KXHORMUZNORM-26MAR17-B260501) — you will get regime label, orderbook depth, related markets, and a suggested next action.

The resolution dates on these contracts range from 3 days to 263 days. The short-dated ones will resolve or expire quickly. The long-dated ones (US invasion by 2027 at 34¢, regime fall by June at 9¢) are the instruments where structural positions make sense. The IY on a 9¢ contract with 79 days to resolution is astronomical — but that is the point. These are deep out-of-the-money geopolitical options, and they are priced accordingly.