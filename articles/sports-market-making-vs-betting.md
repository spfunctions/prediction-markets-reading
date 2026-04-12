# Sports Market Making vs Sports Betting: $5M/Month in Rewards Most People Ignore

> 99% of Polymarket sports participants are betting. Almost nobody is making markets. The reward pool is $5M/month with minimal competition.

**Category:** sports | **Author:** SimpleFunctions | **Reading time:** 8 min | **Published:** 2026-04-05

---
There's a structural opportunity on Polymarket that most participants don't see. Almost everyone trading sports markets is betting — taking directional positions based on who they think will win. Almost nobody is providing liquidity on both sides of the book. Polymarket pays $5 million per month in rewards for the latter, and the competition is thin.

## The fundamental difference

| | Betting | Market Making |
|---|---|---|
| Revenue source | Directional profit (I think Lakers win, buy YES, collect if right) | Liquidity rewards + spread capture |
| Core skill | Better probability estimation than the market | Tighter quotes + higher uptime than other market makers |
| Information need | Must know something the market doesn't | Don't need information advantage — just need to not be slower than everyone else |
| When you lose | Your prediction is wrong | Adverse selection (someone fills your stale quote after an event) |
| Time commitment | Analyze games → place bets → wait for results | Run a bot 24/7 that quotes automatically |
| Capital efficiency | High variance — individual bets win or lose 100% | Low variance — small gains per minute, compounding |

A bettor who is wrong about the Lakers loses their stake. A market maker who is "wrong" about the Lakers just has an inventory position that the next trade might reverse — and they've been collecting rewards the entire time.

## Why the opportunity exists

Polymarket's sports market making requires:

1. A Polygon wallet with USDC
2. py-clob-client credentials (EIP-712 signing)
3. A bot that polls orderbooks and places/cancels orders
4. Understanding of the quadratic scoring function

This is a real technical barrier. Most sports participants are retail users who click buttons on the Polymarket webapp. They don't run Python bots. They don't know what EIP-712 signing is. They don't compute adjusted midpoints.

The result: reward pools that are generous relative to the competition.

Consider an EPL match with a $10,000 reward pool. If 5 market makers compete with roughly equal scores, each earns $2,000 per game. There are 10 EPL games per week. That's $20,000/week from one league — before touching NBA, Champions League, or any other sport.

## The adverse selection math

The main risk of market making is adverse selection: a goal is scored, the price should move 20 cents, but your old quote at the pre-goal price gets filled before you can cancel it.

Quantified for a typical EPL match:

```
Average goals: 2.5 per game
Price impact per goal: 15-25 cents
Your exposure per quote: 100 contracts

Worst case per goal: 100 × 0.25 = $25 loss
Worst case per game: 2.5 × $25 = $62.50

Reward income (20% share): $2,000
Net: $2,000 - $62.50 = $1,937.50
```

Adverse selection costs 3% of revenue. This ratio is why sports market making works — the rewards massively overcompensate for the risk.

In practice, the cost is even lower:
- Not every goal hits your quotes (the book may have other orders ahead of yours)
- Circuit breakers cancel quotes within 2-3 seconds, limiting fill size
- Price impact varies — some goals change the price 10 cents, not 25

## Market making with a view

Pure market making quotes symmetrically: equal size on both sides, no directional opinion. This maximizes your reward score (because Q_min = min of both sides, and symmetry means the minimum equals both).

But you can blend in a view if you have one. Use [SimpleFunctions](https://simplefunctions.dev) for market intelligence:

```bash
sf edges
  Lakers YES: market 42¢, SF-implied 51¢, edge +9¢
```

Then skew your sizes — not your prices:

```
Bid price = 41¢ (mid - 1 tick)   # unchanged
Ask price = 43¢ (mid + 1 tick)   # unchanged

Bid size = 150 (bullish → buy more)
Ask size = 80  (bullish → sell less)
```

Your reward score barely changes (prices are still 1 tick from mid). But your inventory naturally accumulates in the direction of your view. You earn rewards AND express a directional opinion. This is the best of both worlds.

## Getting started

```bash
pip install sfmm

# See what's available
sfmm discover

# Test without real orders
sfmm run --dry-run

# Go live (requires Polymarket credentials)
sfmm run --mode pre
```

The [sfmm](https://github.com/spfunctions/polymarket-sports-mm) bot handles market discovery, quote optimization, order management, and risk controls. SimpleFunctions provides the intelligence layer if you want edge-informed quoting on top.

Start with pre-game on a single league. The risk is minimal, the capital requirement is low, and the rewards are real.