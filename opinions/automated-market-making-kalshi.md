# The Case for Automated Market Making on Kalshi

> Most Kalshi markets have wide spreads because nobody is making them. That is both a problem and an opportunity.

**Category:** technical | **Author:** SimpleFunctions | **Reading time:** 7 min

---
Open any Kalshi market outside the top 10 by volume and you'll see the same thing: a bid at 25 cents, an ask at 40 cents, and nothing in between. That 15-cent spread is a tax on every participant. The taker pays it. The market suffers from it — wider spreads mean fewer trades, which means less information aggregation, which means the price is less useful as a probability estimate.

This is the core liquidity problem of prediction markets. And unlike equity or crypto markets, almost nobody is solving it.

## Why the Spreads Are Wide

In traditional markets, market makers are everywhere. Citadel, Jane Street, Virtu — they quote tight spreads on thousands of instruments because the economics work: high volume, rebates from exchanges, hedgeable inventory, and statistical edge from being faster than the next guy.

Kalshi markets have none of these advantages:

**Low volume.** Even popular Kalshi markets trade a few million dollars in notional. The median market trades a few thousand. There's not enough flow to sustain a market maker on volume alone.

**No hedging.** If you're making a market in SPY options, you can delta-hedge with the underlying. If you're making a market in "US Recession 2026 YES," there's no liquid hedge. You own the binary risk outright.

**Event risk.** Binary contracts go from 35 cents to 100 or 0 on a single event. A market maker quoting around 35 cents can lose 65 cents per share instantaneously if they're on the wrong side when the event resolves. In traditional markets, this kind of gap risk is rare; in prediction markets, it's the fundamental structure.

**No maker rebates.** Kalshi charges fees to both sides. There's no rebate for providing liquidity, which is the bread and butter of equity market makers.

Given all this, it's rational that professional market makers haven't shown up. The economics don't work for the traditional approach. But they might work for a *different* approach.

## Informed Market Making

The traditional market making model is delta-neutral: quote both sides, collect the spread, hedge away the directional risk. This doesn't work on Kalshi because there's nothing to hedge with.

But there's another model: *informed* market making. Instead of being delta-neutral, you have a view on the fair value of the contract. You quote around that fair value, leaning your quotes based on your thesis.

Here's the difference:

**Uninformed market maker** on "Recession 2026 YES" at 35 cents:
- Bid 30, Ask 40 (10-cent spread, centered on market price)
- No view on whether 35 cents is correct
- Gets picked off when informed traders arrive

**Informed market maker** with a thesis that fair value is 55 cents:
- Bid 45, Ask 58 (13-cent spread, centered on *thesis* price)
- Actively tightening the spread closer to where they believe fair value is
- Picking up 10-20 cents of edge when fills happen on the bid side

The informed market maker is doing two things simultaneously: providing liquidity (tightening spreads, improving market quality) and expressing a directional thesis (buying cheap relative to their model). This is economically viable even in low-volume markets because the edge per trade is much larger than a traditional market maker's fraction-of-a-penny spread.

## Capital Requirements and Inventory Risk

Let's talk numbers. Suppose you want to make markets across 20 Kalshi contracts with an average position limit of $5,000 per contract. That's $100K of capital deployed. Your average spread capture is 8 cents per share, and you turn over the inventory once per week.

At $5,000 per contract at an average price of 40 cents, that's 12,500 shares. Turning that over weekly at 8 cents capture: $1,000 per contract per week, or $20,000 across 20 contracts. Against $100K of capital, that's a 20% weekly return before losses.

But here's the catch: *losses are binary*. If one of your 20 contracts resolves against you and you're holding max inventory, you lose $5,000 instantly. If three resolve against you in a bad week, that's $15,000 — wiping out most of your spread revenue.

This means portfolio construction matters enormously:

1. **Diversify across uncorrelated events.** Don't make markets in "Recession 2026" and "S&P down 20%" simultaneously — they're correlated. A recession causes both to resolve YES, doubling your loss.

2. **Size positions by confidence.** If your thesis model gives high confidence on fair value, quote tighter and hold larger inventory. If confidence is low, quote wider and keep inventory small.

3. **Manage expiry risk.** As a contract approaches resolution, spreads should widen because event risk increases. Don't be the market maker quoting a tight spread on a contract that resolves tomorrow based on a single data point.

4. **Set hard inventory limits.** Never hold more than X% of your capital in a single contract. The temptation to load up on "obvious" mispricings is how market makers blow up.

## The Quoting Problem: Where Is Fair Value?

Here's the hard part: to be an informed market maker, you need to know where fair value is. In equity markets, fair value is derived from complex models, but fundamentally it's anchored by observable quantities — earnings, book value, comparable transactions.

In prediction markets, fair value for "Recession 2026 YES" isn't derivable from any single observable. It's a function of:

- Fed policy path
- Oil prices
- Consumer spending trends
- Credit conditions
- Labor market data
- Geopolitical risk
- And dozens of other variables, each with their own uncertainty

This is where causal models become essential. A causal tree decomposes "recession probability" into its upstream drivers, assigns probabilities to each, and propagates them through to a fair value estimate. Without this structure, you're guessing. And guessing as a market maker is how you provide free money to informed traders.

SimpleFunctions' causal tree gives you exactly this framework. You define the upstream nodes — war risk, oil supply disruption, Fed response, consumer resilience — assign probabilities based on evidence, and the system calculates the implied fair value for downstream contracts. Your quotes are centered on this model-implied fair value, not on the last traded price.

When new information arrives — a news headline, a price move in oil, a Fed statement — the causal tree updates. The implied fair value shifts. Your quotes adjust automatically. You're not reacting to noise; you're repricing based on structural changes in the upstream drivers.

## Practical Architecture

A thesis-driven market making system on Kalshi needs:

1. **Causal model** producing fair value estimates for each contract
2. **Quoting engine** that places/adjusts orders around fair value with configurable spread
3. **Inventory manager** that tracks current positions, enforces limits, and adjusts quotes based on current exposure
4. **Signal processor** that ingests news and price data, updates the causal tree, and triggers requoting
5. **Risk monitor** that kills quotes when things look wrong — abnormal volume, rapid price movement, approaching resolution

The signal processing loop is the key differentiator. Traditional market makers requote based on *price* signals — the underlying moved, so the option price moves. Informed prediction market makers requote based on *causal* signals — an upstream driver changed, so the downstream contract's fair value changes.

```
News: "Fed signals pause on rate cuts"
  → Causal node: Fed_hawkish probability rises from 60% to 75%
  → Downstream: Recession probability increases from 35% to 42%
  → Fair value shifts: Recession YES from 35¢ to 42¢
  → Quotes adjust: new bid 37¢, new ask 47¢
  → Old bid at 30¢ cancelled (now below updated fair value)
```

This cascade happens in the causal tree, not in a trader's head. It's systematic, auditable, and consistent.

## The Opportunity Window

Prediction markets are still early. Kalshi's volume has grown significantly but remains a fraction of traditional markets. As volume increases, professional market makers will eventually arrive. But right now, the spreads are wide, the competition is thin, and an informed market maker with a structured thesis has a genuine edge.

The window won't last forever. The time to build is now.

```bash
$ npm install -g @spfunctions/cli
$ sf setup
$ sf context <thesis-id> --json   # Get fair value estimates for your contracts
```