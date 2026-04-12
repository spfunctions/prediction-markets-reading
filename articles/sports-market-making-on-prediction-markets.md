# Sports Market Making on Prediction Markets: Pre-Game, Live, and the $5M Monthly Opportunity

> Polymarket pays millions to market makers who quote sports events. Here is what market making actually means, how it differs from betting, and why the pre-game and live phases require completely different systems.

**Category:** sports | **Author:** SimpleFunctions | **Reading time:** 12 min | **Published:** 2026-04-05

---
Market making is not betting. This distinction matters because most people who look at Polymarket's sports markets think in terms of "who will win." Market makers don't care who wins. They care about quoting both sides of the book — tightly, consistently, and for as long as possible.

## What a market maker actually does

A bettor looks at Liverpool vs Arsenal and thinks: "Liverpool at 55 cents is cheap, I'll buy YES."

A market maker looks at the same game and thinks: "The midpoint is 55 cents. I'll post a bid at 54 cents and an ask at 56 cents. If someone buys my ask, I'm short. If someone hits my bid, I'm long. Either way, I collected 2 cents of spread. And Polymarket pays me a reward for having those orders resting on the book."

The bettor needs Liverpool to win. The market maker needs liquidity to flow through their orders. These are structurally different businesses with different risk profiles and different revenue models.

## The revenue model: liquidity rewards

Polymarket distributes over $5 million per month to market makers on sports events alone. The mechanism:

1. Every minute, the system randomly samples the orderbook
2. Your resting limit orders are scored based on how close they are to the midpoint
3. The scoring function is quadratic: S(v,s) = ((v-s)/v)², where v is the max qualifying spread and s is your distance from mid
4. Your share of the reward pool equals your score divided by all market makers' scores

The reward pools are substantial. A single English Premier League match pays $10,000 to its market makers. A Champions League quarterfinal pays $24,000. NBA games pay $7,700 each.

This is not a rounding error. At 20% market share across EPL and NBA alone, a market maker can earn $50,000-100,000 per month. The capital requirement is $3,000-10,000 in USDC.

## Pre-game vs live: two different systems

### Pre-game

The pre-game period starts when the market opens (usually 24-48 hours before kickoff) and ends when the game begins. During this time:

- Prices move slowly, driven by news (injury reports, lineup announcements, odds movement at traditional bookmakers)
- The orderbook is relatively stable — you can poll every 5 seconds
- Adverse selection risk is low: between information events, your quotes are safe
- The main risk window is lineup announcement (typically 1 hour before kickoff), where prices can jump 5-15 cents in minutes

Pre-game market making is closer to "passive income." Set your quotes near the midpoint, monitor for information events, and let the rewards accumulate. The reward pool is typically 28% of the total per game ($2,800 for an EPL match).

### Live

Live market making operates during the game itself. Everything changes:

- A goal in soccer moves the price 15-30 cents instantly
- You must detect the event and cancel your stale quotes before they get filled at the wrong price
- Polling drops to every 1 second
- A circuit breaker pulls all quotes when a major event is detected, then re-quotes after a cooldown period

Live is harder but pays 2.5-3x more. The same EPL match pays $7,200 for live market making versus $2,800 for pre-game. The math works because adverse selection costs are modest: even if you get hit on every goal (roughly $20-30 per goal, 2-3 goals per game), the reward income dwarfs the losses.

The key insight about live: **you don't need to be faster than the information.** You need to be fast enough to cancel your stale quotes before too much gets filled. A 2-3 second reaction time is sufficient. This is fundamentally different from live betting, where you need sub-second execution to exploit stale prices.

## The two-sided requirement

Polymarket's scoring heavily penalizes single-sided quoting. If you only post bids (or only asks), your score is divided by 3. In extreme price ranges (below 10 cents or above 90 cents), single-sided orders score zero.

This means market makers must always quote both sides. The practical consequence: you will accumulate inventory. If you post a bid at 54 cents and someone sells to you, you're now long. Your ask at 56 cents is still there. Managing this inventory — not letting it grow too large in either direction — is the core operational challenge.

## How this connects to SimpleFunctions

[SimpleFunctions](https://simplefunctions.dev) provides the intelligence layer that makes sports market making more effective:

- **Market scanning**: `sf scan "NBA finals"` finds all relevant markets across Polymarket and Kalshi
- **Edge detection**: `sf edges` identifies mispricings that help calibrate your fair value
- **Execution tracking**: The intent system (`sf intent list`) provides audit trail and Telegram notifications for every fill
- **World context**: `sf world` surfaces macro events that might affect game outcomes

The [sfmm](https://github.com/spfunctions/polymarket-sports-mm) bot (`pip install sfmm`) handles the actual quoting — discovering markets, computing optimal quotes, placing orders via py-clob-client, and managing risk. SimpleFunctions provides the broader intelligence context.

## Who should consider this

Sports market making is attractive if you:

- Have $3,000-10,000 in USDC to deploy on Polymarket
- Can run a process 24/7 (a VPS or always-on machine)
- Want relatively predictable returns that don't depend on picking winners
- Are comfortable with Python and basic trading concepts

It is not for you if you want to "bet on games." Market making and betting are complementary activities, but they require different mindsets, different tools, and different risk management.