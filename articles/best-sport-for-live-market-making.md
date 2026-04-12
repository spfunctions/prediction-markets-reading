# Why Soccer Is the Best Sport for Live Market Making (And Basketball Is the Worst)

> Not all sports are equal for live market making. The math comes down to event frequency × price impact × time between events. Soccer wins by a wide margin.

**Category:** sports | **Author:** SimpleFunctions | **Reading time:** 9 min | **Published:** 2026-04-05

---
Live market making during a sporting event means quoting both sides of the book while the game is happening. Events — goals, baskets, knockouts — move the fair price. If your quotes are still resting at the old price when the event happens, you get adversely selected: someone fills your stale order at a price that no longer reflects reality.

Different sports have radically different event profiles. This makes some sports excellent for live market making and others borderline impossible. The difference is quantifiable.

## The framework: safe minutes vs dangerous minutes

For each sport, we can decompose a game into:

- **Safe minutes**: no game-changing events. Your quotes sit on the book, accumulate reward score, and face minimal adverse selection risk.
- **Dangerous minutes**: a game-changing event just happened. Your quotes are stale. You need to cancel them before they get filled.

The ratio of safe to dangerous minutes determines how profitable live market making is for that sport.

## Soccer: 86 safe minutes, 4 dangerous

A soccer match lasts 90+ minutes. The average number of goals is 2.5. Each goal creates roughly 90 seconds of danger (time for the event to register, circuit breaker to fire, quotes to be pulled, market to stabilize, and new quotes to be placed).

```
Total: 95 minutes (including stoppage time)
Dangerous: 2.5 goals × 1.5 min = 3.75 minutes
Safe: 91.25 minutes (96% of the match)

Revenue per safe minute (EPL, 20% share):
  $7,200 × 20% / 95 minutes = $15.16 per minute

Adverse selection per goal:
  100 contracts × 20 cent impact = $20

Net per game:
  Revenue: $1,440
  Adverse selection: 2.5 × $20 = $50
  Profit: $1,390
```

Soccer is ideal because goals are rare, high-impact, and clearly detectable. Between goals, the market barely moves. You're earning $15/minute doing almost nothing.

Other soccer events (red cards, penalties, injuries) matter too, but their price impact is smaller (5-10 cents vs 15-30 cents for goals) and they're even rarer.

## Basketball: no safe minutes

An NBA game has roughly 200 scoring events across 48 minutes of play. That's one every 14 seconds. Each basket moves the price 1-2 cents — small individually, but the cumulative effect means the midpoint is constantly shifting.

```
Total: 48 minutes of play (~2.5 hours elapsed)
Scoring events: ~200
Average time between events: 14 seconds
Price impact per event: 1-2 cents

Problem: polling at 1 second means you see ~14 stale
orderbook states between updates. By the time you react
to a scoring run, 3-4 baskets may have already changed
the fair price.
```

With continuous small events, there are no long safe periods. Your quotes are slightly stale almost all the time. The adverse selection per event is small, but it adds up over 200 events.

NBA games are still profitable — the $5,550 live pool is large — but the profit margin per game is lower and the operational complexity is higher. A basketball market making bot needs sub-second responsiveness and sophisticated microstructure handling that a soccer bot doesn't.

**Recommendation**: start with soccer. Add basketball once your infrastructure is proven.

## Tennis: extreme swings, short games

Tennis is the opposite extreme. Individual points don't matter much, but break points and tiebreaks can swing the match probability by 10-20 cents in a single point.

```
A break of serve in a close match:
  Price impact: 10-15 cents
  Frequency: 5-8 per match
  Duration of danger: ~30 seconds per break point

A tiebreak:
  Each point: 5-10 cent swing
  7-12 points in rapid succession
  Essentially un-makeable — pure adverse selection
```

Tennis is playable during regular service games but lethal during break points and tiebreaks. A sport-aware circuit breaker that widens spreads approaching break points would help, but the reward pools for tennis ($1,050-1,450 per match) may not justify the engineering effort.

## CS2/Esports: the underrated opportunity

Counter-Strike 2 has a distinctive structure: 24-30 rounds of 2 minutes each, with clear round-end events. Between round-ends, the price barely moves (teams are executing strategies). At round-end, the price adjusts based on which team won.

```
Round duration: ~2 minutes
Price impact per round: 2-4 cents
Halftime (side switch): 5-8 cent impact
Map point / overtime: 10-15 cent impact

A-tier CS2 reward: $5,500 per match ($3,950 live)
```

CS2 is attractive because:
1. Round-ends are perfectly discrete (no ambiguity about when the event happened)
2. Between rounds, the market is completely stable (players are buying weapons, not shooting)
3. The reward pools are comparable to top soccer leagues

The main challenge: CS2 maps can last 30-60 minutes, and a best-of-3 can go to 3 maps. The total quoting duration can exceed 3 hours — longer than a soccer match. More time quoting = more reward, but also more capital locked up.

## UFC/MMA: don't bother

A UFC fight can end in a single punch. The probability of "Fighter A wins" can go from 45% to 0% in under one second. No polling-based bot can react to a knockout.

```
UFC reward pool: $4,250 per event (main card)
Knockout frequency: ~25% of fights
Price impact of knockout: 50-100 cents (yes, the full range)
```

The adverse selection on a single knockout can exceed the entire reward pool. Unless you have a live video feed with sub-second processing (which puts you in a different category of infrastructure), avoid UFC for live market making. Pre-game only.

## The ranking

Based on reward-to-risk ratio for live market making:

| Rank | Sport | Why |
|---|---|---|
| 1 | Soccer (EPL, CL) | 96% safe time, huge pools, rare events |
| 2 | CS2 (A-tier) | Discrete rounds, high pools, stable between rounds |
| 3 | Cricket (IPL) | Long games, wickets are infrequent, $4,500 pools |
| 4 | Baseball (MLB) | Discrete at-bats, moderate event frequency |
| 5 | Hockey (NHL) | Like soccer but faster and lower pools |
| 6 | Basketball (NBA) | Constant small events, but $5,550 pools justify the effort |
| 7 | Tennis | Break points are killers, modest pools |
| 8 | UFC/MMA | Knockout risk makes live untenable |

## Implementation

The [sfmm](https://github.com/spfunctions/polymarket-sports-mm) bot has sport-specific circuit breaker thresholds built in:

```python
# From sfmm/risk/circuit.py
JUMP_THRESHOLDS = {
    "soccer":     0.08,   # goal detection
    "basketball": 0.03,   # unusual event
    "tennis":     0.10,   # break of serve
    "cs2":        0.06,   # round end
    "ufc":        0.15,   # knockout
}
```

Start with `sfmm run` on soccer and CS2. These two sports alone cover $15,000+ per game day in live reward pools across the major leagues. Add basketball and other sports as your confidence in the infrastructure grows.