# How to Read the World Through Prediction Market Prices

> A practical guide to translating prediction market prices into world state. What prices mean, what price changes mean, and how to build a real-time world model from market data.

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 10 min | **Published:** 2026-03-31

---
# How to Read the World Through Prediction Market Prices

A prediction market contract trading at 73 cents is a statement: the collective market believes there is a 73% probability this event will occur. But to build a useful world model from prediction market data, you need to understand what prices mean, what price *changes* mean, and how to combine signals across multiple contracts.

This is a practical guide.

## Rule 1: The Price Is a Probability

A contract on "Will the Federal Reserve cut rates by September?" trading at 34 cents means: the market estimates a 34% probability of a rate cut by September.

This is the foundational translation:
- **Price in cents = probability as a percentage.** 34 cents = 34%. 87 cents = 87%. 3 cents = 3%.
- **The complement tells you the "no" probability.** If "yes" is 34 cents, "no" is 66 cents (100 - 34).
- **Prices near 50 are maximum uncertainty.** The market genuinely doesn't know.
- **Prices near 0 or 100 are high conviction.** But never ignore them — a contract at 3 cents that moves to 15 cents is a massive signal.

## Rule 2: The Delta Matters More Than the Price

A contract at 65 cents tells you the market's current estimate. A contract that *moved* from 40 to 65 cents in the last 6 hours tells you something far more valuable: new information entered the system, and it dramatically increased the probability.

The delta is the rate of change of the world's belief about an event.

| Delta | What it means |
|-------|-------------|
| < 2 cents | Noise. Normal market fluctuation. |
| 2-5 cents | Mild update. New data point, but doesn't change the picture. |
| 5-15 cents | Significant. Something happened. Worth investigating. |
| 15-30 cents | Major shift. Real news, real consequence. |
| > 30 cents | World changed. This event's probability just flipped. |

An agent that only reads prices is partially informed. An agent that tracks *deltas* — the rate of change over time — can detect when the world is shifting before it reads a single article about why.

## Rule 3: Volume Validates the Price

A contract at 73 cents with $2 million in volume means: many people with real money agree on this probability. The signal is strong.

A contract at 73 cents with $500 in volume means: one person placed a bet and nobody disagreed. The signal is weak.

Always weight price by volume. A high-volume contract is a *consensus*. A low-volume contract is a *guess*.

## Rule 4: Spread Measures Confidence

The spread — the gap between the best bid and the best ask — tells you how confident the market is in its price.

- **Tight spread (1-3 cents):** Active market, strong price discovery. The price is reliable.
- **Medium spread (3-8 cents):** Decent market. Price is approximately right but could move.
- **Wide spread (8+ cents):** Thin market, few participants. Price is unreliable. The true probability is somewhere between bid and ask.

For agents: if the spread is wider than your edge, you don't have an edge. The uncertainty in the price exceeds your informational advantage.

## Rule 5: Related Contracts Tell a Story

Individual contracts are data points. Clusters of related contracts tell stories.

Example: If you see these contracts moving simultaneously:
- "US military strikes Iran" — 20 → 35 cents
- "Oil above $120 by June" — 15 → 28 cents
- "Strait of Hormuz disruption" — 25 → 45 cents
- "Fed holds rates through July" — 60 → 72 cents

You don't need to read a single news article to understand: a military escalation with Iran is becoming more likely, the market expects oil price impacts, and the Fed is expected to hold rates in response to geopolitical uncertainty.

Four prices. One story. Built in seconds, not hours.

## Rule 6: Contradictions Are Opportunities

When related contracts *disagree*, something is wrong — and the disagreement is valuable.

Example:
- "Recession by Q4" — 45 cents (market says 45% likely)
- "S&P 500 above 5500 by December" — 62 cents (market says 62% likely)

These are in tension. A recession makes a new S&P high unlikely. Either the recession contract is too high, the S&P contract is too high, or the market is pricing a "mild recession" that doesn't affect equities. Each interpretation has trading implications.

Contradictions across related contracts are where the most information lives. They reveal *what the market hasn't figured out yet*.

## Rule 7: Time Decay Is Information

Contracts have expiration dates. As the date approaches, prices either converge toward 0 or 100.

A contract at 50 cents with 6 months to expiry is genuine uncertainty. A contract at 50 cents with 3 days to expiry means the outcome is literally a coin flip — maximum uncertainty at the last moment.

The *rate* of convergence tells you something: a contract that's been at 50 cents for months and suddenly moves to 70 cents in the last week — that's new information arriving late. It's the most informative kind of price movement.

## Putting It Together: A World Model

Here's how an agent can build a real-time world model from prediction market data:

1. **Define your domains.** What aspects of the world do you care about? Geopolitics? Economics? Technology?
2. **Monitor relevant contracts.** Track prices, deltas, volume, and spread.
3. **Cluster related contracts.** Group by topic. Look for correlated movements.
4. **Watch for delta spikes.** A sudden movement in a cluster of contracts = something is happening.
5. **Identify contradictions.** Disagreements between related contracts = the market is uncertain = edge.
6. **Use news to explain, not to discover.** When you see a price spike, *then* read the news to understand why.

The result: a continuously-updating probability map of the world, built from thousands of incentivized bets, processing information faster than any news feed.

---

*This is part of a series on prediction markets as cognitive tools. Next: [The Most Important Number in a Prediction Market Isn't the Price — It's the Delta](/blog/the-most-important-number-is-the-delta)*
