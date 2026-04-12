# Limit orders on Kalshi: why thesis-informed makers outperform blind spread collectors

> The edge isn't in being a maker — it's in knowing where to place the bid before the book tells you.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 9 min

---
# Limit orders on Kalshi: why thesis-informed makers outperform blind spread collectors

I've been trading on Kalshi for over a year now, mostly in macro contracts — oil ceilings, recession calls, that kind of thing. Early on I did what most people do: I'd see a contract I liked, hit the market order button, and take whatever the ask was. It worked, sort of. But my fills were consistently worse than they needed to be, and on thinner books I was giving up real edge just to get in fast.

So I switched almost entirely to limit orders. And then I learned something harder: placing limits without a thesis is barely better than taking the market.

This is the piece I wish someone had written when I started. It's about the difference between blind spread-collecting and thesis-informed making, using real Kalshi tickers I actually trade.

## The default behavior and why it's expensive

Most Kalshi traders place market orders. The UX encourages it — you see a price, you click buy, you're in. The platform doesn't make it hard to place limits, but the path of least resistance is taker execution.

The cost of this is invisible until you track it. On a contract like KXWTIMAX-26DEC31-T135 ("Will WTI crude oil reach $135 at any point in 2026?"), the bid-ask might sit at 6¢ / 12¢. You buy at 12¢ thinking you're getting a cheap lottery ticket. But the midpoint is 9¢, and if the "true" price is closer to 8¢, you just paid a 50% premium over fair value to get filled immediately.

On liquid contracts this matters less. On the stuff I trade — tail risk contracts, recession bets, commodity thresholds — spreads are wide and the cost of taking is enormous relative to the contract's edge.

## The blind maker approach

The first-order improvement is to stop taking and start making. Place a limit order inside the spread and wait. This is what most "sophisticated" retail traders do, and it's fine. It's better than market orders. But it has a specific failure mode that most people don't think about.

Here's the failure mode: you look at the book on KXRECSSNBER-26 ("Will the NBER declare a recession starting in 2026?"). Bid is 18¢, ask is 26¢. You want to go long. So you place a limit at 22¢ — right in the middle of the spread. Feels smart. You're "providing liquidity" and getting a "better price."

But ask yourself: is 22¢ actually a good price for this contract? You have no idea. You picked the midpoint of a spread set by other participants whose information and models you know nothing about. Maybe the contract is worth 40¢ and both sides of the book are stale. Maybe it's worth 12¢ and you're about to get filled by someone who knows more than you.

Blind spread-collecting works in market-making when you have speed, hedging, and volume. You don't have any of those on Kalshi. You're a retail trader with one position at a time. The midpoint of the spread is not an edge. It's a guess dressed up as a strategy.

## Thesis-informed making: the actual edge

Here's what I do instead. Before I ever look at the order book, I build a view on what the contract should be worth.

For KXWTIMAX-26DEC31-T135 — will WTI touch $135 at any point this year — I start with a causal tree. What would have to happen for oil to reach $135? A few paths:

1. Major supply disruption (Strait of Hormuz closure, OPEC+ production collapse, major Gulf producer going offline)
2. Demand shock upward (China recovery far exceeding expectations, synchronized global growth)
3. Dollar collapse making dollar-denominated commodities reprice
4. Some combination of moderate supply tightening + moderate demand increase

I assign rough probabilities to each path, accounting for overlap. Say I land at: there's roughly a 7-8% chance WTI touches $135 at some point in 2026. My fair value for this contract is around 7.5¢.

Now I look at the book. Bid 6¢, ask 12¢. Market midpoint is 9¢. A blind maker would bid 9¢ and feel clever.

But I know my fair value is 7.5¢. So I bid 6¢ — right at the current bid. Why? Because 6¢ gives me 1.5¢ of edge below my thesis price. If I get filled, I'm buying below what I believe the contract is worth. If I don't get filled, I haven't lost anything. I'm patient.

Now contrast with KXRECSSNBER-26. My macro model says the probability of an NBER-declared recession starting in 2026 is meaningfully higher than the market implies. My causal tree accounts for the lagged effects of rate policy, leading indicators I track, credit conditions, and the historical base rate of recessions following prolonged restrictive policy. I land at roughly 38% probability.

The book shows bid 18¢, ask 26¢. This is massively mispriced relative to my view. Here I might bid 27¢ — above the current ask. That's aggressive. I'm crossing the spread because my edge is so large (38¢ fair value minus 27¢ entry = 11¢ of expected edge) that I don't want to risk not getting filled. The catalyst window matters: if I think the next jobs report or GDP print could move this contract, I don't have time to sit at 22¢ hoping someone sells to me.

## The math: fill probability vs. entry edge

Let me formalize this, because the tradeoff is concrete and you can actually think about it quantitatively.

For any limit order, you're optimizing across two variables:

- **Entry edge**: the difference between your thesis price and your limit price. More edge = more profit per fill.
- **Fill probability**: how likely your order is to execute. Tighter (more aggressive) limits fill more often.

These are in tension. A bid of 6¢ on a contract you think is worth 7.5¢ gives you 1.5¢ of edge but might only fill 30% of the time. A bid of 7¢ gives you 0.5¢ of edge but fills maybe 60% of the time.

The expected value of the limit order is roughly:

**EV = fill_probability * (thesis_price - limit_price) * contract_size**

For the 6¢ bid: EV = 0.30 * 1.5¢ * $1 = 0.45¢ per contract attempt

For the 7¢ bid: EV = 0.60 * 0.5¢ * $1 = 0.30¢ per contract attempt

The wider limit is actually better on expected value despite filling less often. But this ignores something important: opportunity cost. If the 6¢ bid doesn't fill and the contract moves to 15¢ on a supply shock, you missed the move entirely. You "saved" 1.5¢ and lost 8.5¢ of upside.

This is why catalyst timing matters. If no catalyst is imminent, be patient — post the wider limit, maximize edge per fill. If a catalyst is days away and you think the market hasn't priced it, be aggressive — cross the spread if you have to, because the fill matters more than the entry.

## When to take vs. when to make

I've settled on a rough decision framework that I run through on every trade:

**Take the market when:**
- Your thesis price is more than 2x the current ask (massive mispricing)
- A known catalyst is within 48 hours
- The book is thin and your limit would be the only bid (likely to be picked off)
- You've been sitting unfilled for days and the thesis hasn't changed

**Post a limit when:**
- Your thesis price is 10-50% above the ask (meaningful but not massive edge)
- No near-term catalyst
- The book has some depth and you can place inside the spread
- You're willing to not get filled and that's fine

**Walk away when:**
- Your thesis price is at or below the current ask
- The spread is tight enough that the edge net of fees is negligible
- You don't have a thesis — you just have a "feeling" about the contract

That last point is the most important rule I follow. No thesis, no order. Period. The book alone doesn't tell you anything about where to place a limit. The book tells you where *other people* are willing to transact. Your job is to know whether those people are right or wrong.

## The real orderbook: a worked example

Let me walk through a recent trade on KXRECSSNBER-26 in detail.

My thesis model gave me 38¢ fair value. When I pulled up the contract, the book looked like this:

```
Asks:          Bids:
28¢  (50 lots)    18¢  (30 lots)
30¢  (20 lots)    16¢  (45 lots)
35¢  (10 lots)    14¢  (25 lots)
```

Best ask: 28¢. Best bid: 18¢. Spread: 10¢. Midpoint: 23¢.

A blind maker would bid 23¢ and call it a day.

My analysis: 38¢ fair value means even the best ask (28¢) is 10¢ below my thesis. The edge at 28¢ is 10¢ on a $1 contract — that's enormous. But I don't need to take immediately because the catalyst I'm watching (next GDP revision) is three weeks out.

So I placed a limit at 24¢. Here's why:
- 24¢ is above the midpoint (23¢), so I'm more likely to get filled than a midpoint bid
- 24¢ gives me 14¢ of edge (38¢ - 24¢) — still very large
- If the book tightens and someone sells at 24¢, I'm buying well below my fair value
- If the book moves away from me (asks drop to 25¢), I can reassess

I got filled at 24¢ two days later when someone placed a large market sell. Three weeks after that, the GDP print came in weak, and the contract moved to 33¢. I was up 9¢ on a 24¢ entry — a 37.5% return. If I'd taken at 28¢, I'd have made 5¢ on a 28¢ entry — a 17.9% return. If I'd bid 23¢ and not gotten filled, I'd have made nothing.

The 24¢ limit was the right call because it balanced edge with fill probability given my catalyst timeline. That's what thesis-informed making looks like in practice.

## Common objections

**"I don't have a model — I just read the news."** That's fine. You don't need a quant model. You need a number. Read whatever you read, talk to whoever you talk to, think about it, and write down a probability. It'll be wrong. That's fine too. A wrong-but-thoughtful number is infinitely better than no number, because at least you can track your calibration over time.

**"The spreads are too wide, limits never fill."** This is true on some contracts. If a contract has no depth and a 20¢ spread, you might be better off either taking the ask or skipping the trade entirely. Don't fall into the trap of placing a limit at the midpoint of a dead book and waiting forever. That's just a polite way of not trading.

**"Market makers will front-run my limits."** On Kalshi? No. This isn't an HFT venue. The order book is slow. Your 24¢ bid is going to sit there until someone decides to sell at that price. Nobody is running ahead of you.

## The meta-lesson

The real insight here isn't about limit orders. It's about the difference between process-driven and price-driven trading.

Price-driven: "The contract is at 28¢, that seems cheap, I'll buy it."

Process-driven: "My model says 38¢. The contract is at 28¢. I'll bid 24¢ because my catalyst is three weeks out and I can afford to be patient. If I get filled, I have 14¢ of edge. If I don't, I'll reassess when the book changes."

Same contract. Same direction. Completely different approach. The second version makes explicit what the first version leaves to chance: your entry price, your edge, your catalyst timeline, and your plan if you don't get filled.

Every trade I place now goes through this process. Build the thesis. Get a number. Look at the book. Decide if the edge justifies taking or if I can afford to make. Place the order. Track the outcome.

It's not glamorous. It doesn't involve staring at charts or checking prices every five minutes. Most days I place zero orders. But when I do place one, I know exactly why I'm placing it, at exactly that price, at exactly that time.

That's the edge. Not the limit order itself — the thesis behind it.

---

*If you want to build thesis models like the ones I described — causal trees, scenario probabilities, fair value estimates — I've been building tools for exactly this at [simplefunctions.dev](https://simplefunctions.dev).*