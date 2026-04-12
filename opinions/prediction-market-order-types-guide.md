# The complete guide to prediction market order types: market, limit, and thesis-informed

> How I decide between market and limit orders on Kalshi, and why a causal model changes the math on both.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
# The complete guide to prediction market order types: market, limit, and thesis-informed

I've placed a few hundred orders on Kalshi at this point. Most of them were wrong in some small way — not wrong on direction, wrong on *how I entered*. I'd nail a thesis on recession probability and then fumble the execution by market-buying into a wide spread, or I'd set a limit order so tight it never filled while the contract moved 15 cents against me.

Order type matters more in prediction markets than in equities. The spreads are wider, the liquidity is thinner, and the contracts are binary — they settle at $1 or $0, period. That means every cent you give up on entry is a cent off your max profit, and there's a hard ceiling on what you can make.

This is everything I've learned about order types, when to use each, and how to let a causal model do the deciding.

## Market orders: you're buying certainty of execution

A market order says: fill me now, at whatever the best available price is. You guarantee execution. You do not guarantee price.

On Kalshi, this means you're hitting the best ask (if buying) or the best bid (if selling). If the order book is thin, you might get slipped — your 200-lot fills the first 50 contracts at 34 cents and the next 150 at 37 cents. Your average entry is 36.25 cents, not the 34 you saw on the screen.

I use market orders in three situations:

**1. High conviction + tight spread.** If the bid-ask spread on a contract is 1-2 cents and I have strong conviction, I'm not going to haggle. The cost of crossing the spread is small relative to my expected edge. On KXRECSSNBER-26, when I had a causal model saying recession probability was around 38% and the contract was trading at 25 cents with a 24/26 spread, I just bought at market. Two cents of slippage on a 13-cent expected edge is noise.

```bash
sf buy KXRECSSNBER-26 300
```

No `--price` flag means market order. Three-second countdown, then it fills at best available.

**2. Time-sensitive catalyst.** When a jobs report drops and you have a pre-formed thesis about what the number means for KXRECSSNBER-26, you don't have time to finesse a limit order. The market will reprice in seconds. Market order, get your position, then manage it.

**3. Exiting a position.** If I'm up on a trade and want to lock in profit, I'll often market-sell rather than trying to get the last cent on a limit. A bird in hand.

The failure mode of market orders is overpaying for entry on wide spreads. If the spread on KXWTIMAX is 28/35 — a 7-cent gap — a market buy at 35 means you're immediately underwater by 7 cents if you wanted to exit. That's a big deal on a contract that might only have 15 cents of edge.

## Limit orders: you're buying certainty of price

A limit order says: fill me at this price or better, or don't fill me at all. You guarantee price. You do not guarantee execution.

```bash
sf buy KXWTIMAX-26MAY 200 --price 30
```

This places a limit order to buy 200 contracts at 30 cents or better. If nobody is selling at 30, the order sits on the book and waits. Maybe it fills in an hour when someone decides to sell. Maybe it never fills.

I use limit orders in three situations:

**1. Wide spread, patient entry.** KXWTIMAX contracts can have spreads of 5-8 cents. I'm not crossing a spread that wide. I'll place a limit order inside the spread — if the bid is 28 and the ask is 35, I might bid 31. That gives sellers a better price than the current best bid, so I'm more likely to get filled, but I'm not paying the full ask.

**2. No imminent catalyst.** If there's no data release or event in the next few days that would move the contract, there's no rush. A limit order lets me define my entry price and wait. On KXRECSSNBER-26, if we're in a quiet week between economic data releases, I'll set a limit bid a few cents below the mid and let it sit.

**3. Scaling into a position.** I might want 500 contracts but not all at once. I'll ladder limit orders:

```bash
sf buy KXRECSSNBER-26 150 --price 26
sf buy KXRECSSNBER-26 150 --price 24
sf buy KXRECSSNBER-26 200 --price 22
```

If the contract dips, I accumulate at progressively better prices. If it never dips, I only have partial (or no) fill, but I haven't overpaid.

The failure mode of limit orders is missing the trade entirely. I've set limit orders on contracts that then moved 20 cents in the direction I expected, and I got zero fill because I was trying to save 2 cents on entry. That's a classic penny-wise, pound-foolish mistake.

## The spread tells you which order type to use

Here's a simple heuristic I use:

| Spread (cents) | Default order type | Reasoning |
|---|---|---|
| 1-2 | Market | Cost of crossing is trivial |
| 3-4 | Limit at midpoint | Split the difference |
| 5+ | Limit inside spread | Wide spreads eat your edge |

But this heuristic ignores the most important variable: *how much edge do you actually have?*

## Thesis-informed orders: let the model decide

This is where it gets interesting. If you have a causal model that estimates the true probability of an event, you can derive the optimal order type, price, and size mathematically. No vibes required.

A thesis-informed order works like this:

1. Your causal model outputs a probability estimate (e.g., 38% chance of recession by end of 2026)
2. You compare that to the market price (e.g., KXRECSSNBER-26 trading at 25 cents = 25% implied probability)
3. The difference is your edge (38% - 25% = 13 percentage points)
4. You use the edge to determine order type, limit price, and position size

The `sf strategies` command automates this. It reads your thesis, checks current market prices, and generates a strategy:

```bash
sf strategies KXRECSSNBER-26
```

This outputs something like:

```
Thesis: RECESSION_2026 (p=0.38, confidence=medium)
Market: KXRECSSNBER-26 @ 25¢ (implied p=0.25)
Edge: +13pp

Recommended strategy:
  Order type: LIMIT
  Entry price: 27¢ (inside spread, 11pp edge preserved)
  Quantity: 180 contracts (half-Kelly)
  Stop loss: 18¢
  Take profit: 45¢
```

The logic behind the recommendation:
- **Order type** is limit because the spread is wide enough that crossing it would cost more than 15% of the expected edge
- **Entry price** is set inside the spread at a level that still preserves at least 80% of the edge
- **Quantity** is derived from Kelly criterion (more on this below)
- **Stop and take-profit** are set based on the model's confidence interval

## Kelly criterion for prediction markets

Kelly criterion tells you the optimal fraction of your bankroll to bet, given your edge and the odds. For binary prediction market contracts, the math simplifies nicely.

The standard Kelly formula for a binary bet:

```
f* = (p * b - q) / b
```

Where:
- `f*` = fraction of bankroll to bet
- `p` = your estimated probability of winning
- `q` = 1 - p (probability of losing)
- `b` = net odds (profit per dollar risked if you win)

For a prediction market contract priced at `c` cents:
- If you buy Yes at `c` cents, you profit `(100 - c)` cents if it settles Yes
- Your net odds `b = (100 - c) / c`

So for KXRECSSNBER-26 at 25 cents, with my model's 38% probability:
- `p = 0.38`, `q = 0.62`
- `b = (100 - 25) / 25 = 3.0`
- `f* = (0.38 * 3.0 - 0.62) / 3.0 = (1.14 - 0.62) / 3.0 = 0.173`

Kelly says bet 17.3% of my bankroll. But full Kelly is aggressive — it assumes your probability estimate is perfectly calibrated, which it never is. I use half-Kelly (8.65%) or quarter-Kelly (4.3%) depending on my confidence in the model.

With a $2,000 bankroll at half-Kelly:
- Bet size = $2,000 * 0.0865 = $173
- At 27 cents per contract = 640 contracts
- I'd round down and maybe buy 500-600 contracts in the strategy

The key insight: Kelly naturally scales your bet size down when your edge is small and up when your edge is large. If the contract were at 35 cents instead of 25 (only 3pp of edge), Kelly would recommend a much smaller position.

## Real scenario walkthrough: KXRECSSNBER-26

Let me walk through an actual decision I faced.

**Setup:** My recession model tracks leading indicators — yield curve inversion duration, ISM manufacturing, initial jobless claims trend, credit spreads. In February 2026, the model was outputting 38% recession probability. KXRECSSNBER-26 ("Will NBER declare a recession starting in 2026?") was trading at 25 cents.

**Spread check:** Bid 23, ask 27. That's a 4-cent spread.

**Edge calculation:** 38% - 25% = 13pp. At 27 cents (the ask), edge is 38% - 27% = 11pp. Still substantial.

**Order type decision:** 4-cent spread, 11pp edge at the ask. The spread cost (4 cents) is about 36% of the edge (11 cents). That's enough to warrant a limit order rather than a market order. I set a limit at 26 cents — inside the spread, giving me 12pp of edge.

```bash
sf buy KXRECSSNBER-26 300 --price 26
```

**Result:** Filled 220 of the 300 over the next two hours as sellers came in. The remaining 80 never filled because an employment report came out stronger than expected and the contract dipped to 22 before I could adjust.

Lesson: partial fills on limit orders are normal. I got 220 contracts at my target price, and the 80 I missed would have been better to get at 22 than 26 anyway. No regret.

## Real scenario walkthrough: KXWTIMAX-26MAY

Different contract, different dynamics.

**Setup:** KXWTIMAX-26MAY ("Will WTI crude hit $X by May 2026?") was priced at 32 cents. My energy model, which tracks OPEC+ production decisions, US inventory trends, and geopolitical risk, estimated a 48% probability.

**Spread check:** Bid 28, ask 35. That's a 7-cent spread — wide.

**Edge calculation:** 48% - 32% = 16pp. But at the ask (35 cents), it's 48% - 35% = 13pp. Still good, but 7 cents of that 13pp edge evaporates if I market-buy.

**Order type decision:** This has to be a limit order. 7 cents is more than half the edge at the ask. I set a limit at 31, one cent below the mid.

```bash
sf buy KXWTIMAX-26MAY 200 --price 31
```

**Size decision:** My Kelly calculation at 31 cents entry:
- `b = 69/31 = 2.23`
- `f* = (0.48 * 2.23 - 0.52) / 2.23 = (1.07 - 0.52) / 2.23 = 0.247`
- Half-Kelly: 12.4% of bankroll

With a $2,000 bankroll, that's $248, or about 800 contracts at 31 cents. But I only placed 200 — deliberately undersizing because the energy model has wider confidence intervals than the recession model. When you're less sure about your probability estimate, size down further. Quarter-Kelly or less.

**Result:** Filled 200 contracts over about six hours. The contract eventually moved to 44 as oil inventory data came in bearish. I sold 100 at 42 (market order — tight spread, wanted to lock profit) and held the rest.

## The decision framework, summarized

```
Spread < 2 cents AND edge > 10pp  →  Market order, full Kelly sizing
Spread < 2 cents AND edge 5-10pp  →  Market order, half Kelly sizing  
Spread 2-5 cents AND edge > 10pp  →  Limit at midpoint, half Kelly sizing
Spread 2-5 cents AND edge 5-10pp  →  Limit inside spread, quarter Kelly sizing
Spread > 5 cents AND any edge     →  Limit inside spread, quarter Kelly or less
Edge < 5pp                        →  No trade (edge too thin for the uncertainty)
```

This is the framework `sf strategies` implements. It's not magic — it's just the spread/edge ratio determining order type, and Kelly criterion determining size, with a confidence discount.

## Common mistakes I've made

**Market ordering into a wide spread.** Early on, I'd get excited about a thesis and just hit the ask on a contract with a 6-cent spread. You're starting every trade in a hole.

**Setting limit orders too tight.** I went through a phase of always trying to buy at the bid. This means you almost never get filled, because you're competing with other buyers. You need to improve on the best bid by at least a cent to have a realistic chance of filling.

**Ignoring order book depth.** A tight spread on 10 contracts doesn't mean the same thing as a tight spread on 500 contracts. If you want 300 contracts and there are only 20 at the best ask, you're going to blow through multiple levels. Check depth before choosing your order type.

**Oversizing when the model is uncertain.** My KXWTIMAX position was deliberately small relative to Kelly because the model's confidence was only medium. Full Kelly on an uncertain probability estimate is a recipe for blowing up.

**Not adjusting after partial fills.** If I get 220 out of 300 contracts filled and the market moves, I need to re-evaluate. Is the thesis still intact? Has the edge changed? Should I chase the remaining 80 at a worse price or let them go? I used to just leave stale orders on the book.

## Tools I use

All of this is run through the SimpleFunctions CLI. The key commands:

- `sf buy <ticker> <qty> --price <cents>` — limit order at the specified price
- `sf buy <ticker> <qty>` — market order (no price flag)
- `sf strategies <ticker>` — generates order type, price, and size from your thesis
- `sf strategies` — lists all active strategies across your thesis portfolio

The thesis engine does the probability estimation. The strategies engine translates that into concrete orders. I review and approve — the 3-second countdown on every order is there so you can cancel if something looks off.

## Final thought

The order type is not a trivial implementation detail. On wide-spread, binary contracts with hard ceilings on profit, how you enter determines a meaningful chunk of your P&L. A good thesis with bad execution is a mediocre trade. A good thesis with good execution — right order type, right price, right size — is how you compound edge over time.

If you want to try this yourself, the thesis engine and strategies framework are at [simplefunctions.dev](https://simplefunctions.dev).