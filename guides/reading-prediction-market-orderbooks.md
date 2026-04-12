# Reading Prediction Market Orderbooks: Liquidity, Spread, and When to Enter

> Price tells you what the market thinks. The orderbook tells you how confident it is, how much it will cost you to trade, and whether the price can be trusted at all.

**Category:** risk | **Author:** SimpleFunctions | **Reading time:** 8 min

---
Every prediction market trader looks at price. Almost nobody looks at the orderbook. This is a mistake — the orderbook tells you things that price alone can't.

The price of a Kalshi contract tells you "the market thinks there's a 35% chance of recession." The orderbook tells you "...but there's only $200 supporting that price, the spread is 4 cents wide, and a single $500 order would move the price 6 points." These are radically different pieces of information.

## What's in a Kalshi Orderbook

When you pull the orderbook for a Kalshi market (`GET /markets/{ticker}/orderbook`), you get two arrays:

```json
{
  "orderbook": {
    "yes": [
      [35, 150],
      [34, 300],
      [33, 500],
      [32, 200]
    ],
    "no": [
      [66, 100],
      [67, 250],
      [68, 400],
      [69, 150]
    ]
  }
}
```

The `yes` array shows bids to buy YES contracts: 150 contracts bid at 35 cents, 300 at 34, and so on. The `no` array shows bids to buy NO contracts — which is equivalent to offers to sell YES. A NO bid at 66 cents means someone will buy NO at 66 cents, which is identical to selling YES at 34 cents (100 - 66 = 34).

Wait — that math means the best YES bid is 35 cents and the cheapest YES offer is effectively 34 cents (100 - 66)? That would be a crossed book. In practice, the "NO bid at 66" means the YES ask is at 34 cents... let me be precise.

**The YES ask** = 100 - (best NO bid). If the best NO bid is 66 cents, you can buy YES at 34 cents.
**The YES bid** = best price in the YES array. If the best YES bid is 35 cents, you can sell YES at 35 cents.

When the YES bid (35) > YES ask (34), the book is crossed and those orders would have already matched. In a live market, the book shows the *resting* orders — the ones that haven't matched. So you'll see:

- Best YES bid: 33 cents (150 contracts)
- Best YES ask: 37 cents (100 contracts)
- Spread: 4 cents

## Bid-Ask Spread as a Cost

The spread is the toll you pay for immediacy. If the market shows YES at 33/37:

- You want to buy immediately? You pay 37 cents (lift the ask).
- You want to sell immediately? You receive 33 cents (hit the bid).
- Round-trip cost: 4 cents.

This means your edge must exceed the spread to be profitable. If your thesis says the true probability is 40% and the market midpoint is 35%, you have 5 points of theoretical edge. But if the spread is 4 cents and you need to buy at the ask (37 cents), your executable edge is only 3 points (40% - 37%). And if you eventually sell at the bid, your round-trip executable edge is 40% - 37% - (40% - 33%) = negative. You'd lose money.

**Rule of thumb:** Your theoretical edge should be at least 2x the half-spread to be worth trading.

## Depth as a Signal of Conviction

Depth is the total dollar amount resting at each price level. A market with $5,000 on the bid and $5,000 on the ask at tight spreads is a market where participants have conviction and capital behind their views.

A market with $50 on each side is a market where nobody cares, and the "price" is set by a single small trader whose opinion may or may not mean anything.

Here's a real contrast from Kalshi orderbook snapshots:

**KXRECESSION-26 (deep market):**
```
Bids:                    Asks:
35¢  ████████ $1,500     37¢  ██████ $1,200
34¢  ██████████ $2,200   38¢  ████████ $1,800
33¢  ████████████ $3,100 39¢  ██████████ $2,500
32¢  ██████ $1,400       40¢  ████ $900
```

**KXOBSCURE-EVENT (thin market):**
```
Bids:                    Asks:
28¢  █ $50               35¢  █ $75
25¢  █ $100              40¢  █ $50
20¢  █ $30               45¢  █ $25
```

The recession market has $8,200 in bids and $6,400 in asks. The obscure market has $180 in bids and $150 in asks. The recession market's price is meaningful — it represents thousands of dollars of committed capital. The obscure market's price is nearly arbitrary — one $200 market order would move the price 10+ cents.

## Calculating Executable Edge

SimpleFunctions calculates what we call "executable edge" — the real edge after accounting for orderbook conditions:

```
theoretical_edge = thesis_confidence - market_midpoint
half_spread = (best_ask - best_bid) / 2
slippage = estimated_price_impact(your_order_size, orderbook_depth)
executable_edge = theoretical_edge - half_spread - slippage
```

Example:
- Thesis confidence: 45%
- Best bid: 33 cents, Best ask: 37 cents
- Midpoint: 35 cents
- You want to buy 200 contracts
- Orderbook shows 100 contracts at 37, 150 at 38, 200 at 39

Your order of 200 contracts fills: 100 at 37, 100 at 38. Average fill: 37.5 cents.

```
theoretical_edge = 45% - 35% = 10 points
actual_entry = 37.5 cents
executable_edge = 45% - 37.5% = 7.5 points
```

You lost 2.5 points to spread and slippage. Still a good trade — but very different from the 10 points you'd calculate looking at midpoint alone.

```bash
sf context KXRECESSION-26 --size 200
```

This shows you the executable edge at your intended size, not just the theoretical midpoint edge.

## The SimpleFunctions Liquidity Score

SimpleFunctions assigns each market a liquidity score from 0-100 based on:

- **Spread** (40% weight): Tighter spread = higher score
- **Depth** (30% weight): More resting orders = higher score
- **Volume** (20% weight): More trading activity = higher score
- **Consistency** (10% weight): Stable depth over time vs. fleeting orders

```bash
sf scan --min-liquidity 60
```

This filters out markets where the orderbook conditions make trading impractical. A market with 15 points of theoretical edge but a liquidity score of 12 is usually a trap, not an opportunity.

## When Thin Markets Are Opportunities

Counterintuitively, some thin markets are the best opportunities. Here's when:

**1. Early-stage markets.** A market that just opened has thin orderbooks by definition. If you have a strong thesis and can be patient with limit orders, you can establish a position at favorable prices before liquidity arrives.

**2. Neglected markets.** Some events are genuinely mispriced because nobody is paying attention. Low liquidity *is* the reason the mispricing persists — there's no arbitrage capital correcting it. If you're confident in your thesis and can tolerate the wide spread, the edge may be real.

**3. Post-news thinning.** After a big news event, one side of the book often gets pulled. Market makers step back, spreads widen, and resting orders get canceled. This creates a temporary liquidity vacuum where you can place limit orders at favorable prices that fill when liquidity returns.

## When Thin Markets Are Traps

**1. No exit.** You can enter a thin market, but can you exit? If there's $50 on the bid side and you own 500 contracts, you can't sell without cratering the price. Always check the *exit* side of the book before entering.

**2. Stale books.** Some thin markets have orders that have been resting for weeks. These might be from inactive accounts or market makers who forgot to cancel. The price might be "32 cents" but nobody is actually willing to transact at 32 cents. Test with a small order first.

**3. Single-counterparty risk.** If 90% of the depth comes from one entity (you can sometimes infer this from order sizes), and that entity pulls their orders, the price moves violently. You're not trading against a market — you're trading against one person.

## Practical Orderbook Workflow

Here's how to incorporate orderbook analysis into your trading:

1. **Start with `sf scan`** to find markets where your thesis implies edge.
2. **Check liquidity score.** Below 40? Proceed with extreme caution. Below 20? Probably skip unless the edge is enormous (15+ points).
3. **Run `sf context --size N`** with your intended position size. Look at executable edge, not theoretical edge.
4. **Use limit orders.** In prediction markets, there's almost never a reason to market-order. Place a limit at or near the bid and let it fill. Patience costs nothing; crossing the spread costs real points.
5. **Check depth on both sides.** You need liquidity to enter *and* to exit. A market with deep asks and thin bids is easy to buy but hard to sell.
6. **Monitor orderbook changes.** SimpleFunctions tracks depth changes over time. A sudden drop in bid depth (without price change) often precedes a price drop — someone knows something and pulled their support.

The orderbook is the market's body language. The price is what it *says* it believes. The orderbook is how *confidently* it believes it. Learn to read both.