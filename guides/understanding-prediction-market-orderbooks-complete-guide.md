# Understanding Prediction Market Orderbooks: A Complete Guide

> How to read a Kalshi orderbook from the raw API response to executable trading decisions. Covers yes_dollars vs no_dollars, bid/ask computation, slippage algorithms, depth analysis, and liquidity scoring.

**Category:** risk | **Author:** SimpleFunctions | **Reading time:** 13 min

---
Prediction market orderbooks look simple but hide several quirks that confuse traders coming from equity or crypto markets. The biggest: there are two sides to every contract (YES and NO), and they're mathematically linked. Understanding this relationship is the key to reading the orderbook correctly.

This guide starts from the raw Kalshi API response and builds up to executable trading decisions.

## The Raw API Response

Here's what the Kalshi orderbook API actually returns for a contract:

```bash
GET /trade-api/v2/markets/KXRECSSNBER-26/orderbook
```

```json
{
  "orderbook": {
    "yes": [
      [35, 8],
      [36, 15],
      [37, 22],
      [38, 45],
      [39, 30],
      [40, 80]
    ],
    "no": [
      [65, 12],
      [64, 20],
      [63, 18],
      [62, 35],
      [61, 25],
      [60, 50]
    ]
  }
}
```

Each entry is `[price_in_cents, number_of_contracts]`. The `yes` array contains resting limit orders to sell YES contracts. The `no` array contains resting limit orders to sell NO contracts.

**Critical concept:** A YES contract and a NO contract on the same event always sum to $1.00. If you buy YES at 35¢, someone else effectively has the NO side at 65¢. They're complementary positions.

## Parsing Step by Step

Let's extract the key trading numbers from this raw response.

### Best Ask (YES)

The cheapest available YES contract:

```
yes[0] = [35, 8]
Best YES ask = 35¢, with 8 contracts available
```

If you want to buy YES, the first 8 contracts cost 35¢ each.

### Best Bid (YES)

Here's where it gets unintuitive. There's no explicit "YES bid" in the response. You compute it from the NO side:

```
Best NO ask = no[0] = [65, 12]
Best YES bid = 100¢ - 65¢ = 35¢, with 12 contracts available
```

Why? If someone is willing to buy NO at 65¢, they're implicitly saying they'd sell YES at 35¢. From your perspective as a YES holder wanting to sell: you can sell your YES contract by finding someone who'll buy the NO side. The price they pay for NO (65¢) means you'd receive 100¢ - 65¢ = 35¢ for your YES.

### Spread

```
Spread = Best YES ask - Best YES bid
       = 35¢ - 35¢
       = 0¢
```

This is a tight market. Sometimes the spread will be 1-3¢, which is normal. Spreads above 5¢ indicate a thin or illiquid market.

But wait — sometimes the best YES bid is actually *above* the best YES ask. Consider this orderbook:

```json
{
  "orderbook": {
    "yes": [
      [34, 5]
    ],
    "no": [
      [64, 8]
    ]
  }
}
```

```
Best YES ask = 34¢
Best YES bid = 100¢ - 64¢ = 36¢
Spread = 34¢ - 36¢ = -2¢  (negative!)
```

A negative spread means there's a momentary arbitrage opportunity — you could buy YES at 34¢ and sell at 36¢ simultaneously. In practice, these close within milliseconds. If you see one in the API response, it's likely already gone by the time your order reaches the matching engine.

## Building the Full Depth Table

For serious analysis, convert the raw response into a unified depth table:

```
═══════════════════════════════════════════════════════════
             YES BID SIDE              │           YES ASK SIDE
(computed from NO asks)                │    (direct from YES array)
───────────────────────────────────────┼───────────────────────────
 Price    Contracts   Cumulative       │  Price    Contracts   Cumulative
───────────────────────────────────────┼───────────────────────────
 35¢      12          12              │  35¢      8           8
 36¢      20          32              │  36¢      15          23
 37¢      18          50              │  37¢      22          45
 38¢      35          85              │  38¢      45          90
 39¢      25          110             │  39¢      30          120
 40¢      50          160             │  40¢      80          200
═══════════════════════════════════════════════════════════
```

The bid side shows what you'd receive if selling YES. The ask side shows what you'd pay buying YES. The cumulative column tells you how many contracts you can trade before hitting each price level.

## Slippage Calculation Algorithm

Slippage is the difference between the best available price and the average price you actually pay. Here's the exact algorithm for computing slippage when buying YES contracts:

```
Input:  orderbook.yes (sorted by price ascending), desired_quantity
Output: average_fill_price, total_cost, slippage

function calculateSlippage(asks, quantity):
    filled = 0
    total_cost = 0
    best_price = asks[0][0]  // cheapest available

    for each [price, available] in asks:
        fill_at_this_level = min(available, quantity - filled)
        total_cost += fill_at_this_level * price
        filled += fill_at_this_level

        if filled >= quantity:
            break

    if filled < quantity:
        return ERROR: insufficient depth

    average_fill = total_cost / filled
    slippage = average_fill - best_price

    return { average_fill, total_cost, slippage }
```

**Example with our orderbook, buying 100 YES contracts:**

```
Level 1:  8 @ 35¢ = $2.80   (filled: 8)
Level 2: 15 @ 36¢ = $5.40   (filled: 23)
Level 3: 22 @ 37¢ = $8.14   (filled: 45)
Level 4: 45 @ 38¢ = $17.10  (filled: 90)
Level 5: 10 @ 39¢ = $3.90   (filled: 100)  ← partial fill at this level
──────────────────────────────────────────
Total:   100 contracts = $37.34
Average fill: 37.34¢
Slippage: 37.34¢ - 35¢ = 2.34¢
```

**Selling YES contracts** works the same way but in reverse — you eat through the bid side (NO asks) from high to low:

```
To sell YES, you're effectively letting buyers take the NO side.
Eat NO bids (sorted from highest NO ask price to lowest):

NO ask 65¢: you receive 35¢ per YES (12 available)
NO ask 64¢: you receive 36¢ per YES (20 available)  // Wait — this is wrong.
```

Actually, let's be precise. When you sell YES contracts, someone buying NO at 65¢ is your counterparty. You receive 100¢ - 65¢ = 35¢ per contract. Moving deeper:

```
NO ask at 65¢ → you sell YES at 35¢ (12 contracts)
NO ask at 64¢ → you sell YES at 36¢ (20 contracts)  // Better for you!
```

Wait, this is counterintuitive. Selling deeper into the book gives you a *better* price? Yes — because lower NO ask prices mean higher YES bid prices. The NO side of the book is inverted relative to the YES side.

To sell 50 YES contracts:

```
Level 1: 12 @ 35¢ (from NO ask 65¢)  = $4.20
Level 2: 20 @ 36¢ (from NO ask 64¢)  = $7.20
Level 3: 18 @ 37¢ (from NO ask 63¢)  = $6.66
──────────────────────────────────────────
Total:   50 contracts = $18.06
Average sell price: 36.12¢
```

Hmm, but this isn't right either. Let me correct this.

When selling YES, you traverse NO asks from the *best* (lowest price) up, because the lowest NO ask = highest YES bid. The best price to sell YES is determined by the best NO ask:

```
Best NO ask = 65¢ → YES bid = 35¢ (this is the BEST price to sell YES at)
Next NO ask = 64¢ → YES bid = 36¢ (WRONG — 64¢ NO ask means YES bid = 36¢)
```

No — let's think about this carefully. If someone has placed a limit order to buy NO at 65¢, they're willing to pay 65¢ for the NO side. For you to sell YES, they take the NO and you take 100 - 65 = 35¢. That's the best YES bid.

The next NO order is at 64¢. That person pays 64¢ for NO. You get 100 - 64 = 36¢? No — the NO ask array is sorted by price. Entry `[65, 12]` means there are orders to **sell** NO at 65¢. These aren't bids.

Let me reframe. The `no` array in the API contains orders to **sell NO contracts**. To buy YES, you need YES asks. To sell YES, you need YES bids. YES bids come from people who want to buy YES. In the orderbook, someone selling NO at 65¢ is the same as someone buying YES at 35¢.

The NO sells are sorted ascending: 60, 61, 62, 63, 64, 65. The best (cheapest) NO sell is at 60¢. But for a YES seller, the best counterparty is the one paying the most for NO — which is 65¢ (giving you 35¢ for YES).

**Correct sell-side walk (selling YES):**

Walk the NO array from highest price to lowest (because higher NO ask = worse for you as YES seller):

Actually, NO asks at higher prices are better counterparties for YES buyers (not sellers). Let me use concrete terms:

```
The `no` array: resting orders to SELL NO contracts.
  [65, 12] = 12 contracts offered to SELL NO at 65¢
  [64, 20] = 20 contracts offered to SELL NO at 64¢
  ...

If I want to BUY NO at 65¢, I'm betting AGAINST the event.
  Cost: 65¢. Payout if NO: $1.00.

The YES bid is derived: someone selling NO at X¢ creates
a YES bid at (100-X)¢.
  NO sell at 65¢ = YES bid at 35¢
  NO sell at 64¢ = YES bid at 36¢
  NO sell at 63¢ = YES bid at 37¢
```

So to sell YES, walk from the highest YES bid downward:

```
YES bid 37¢ (from NO sell 63¢): 18 contracts  ← best for seller
YES bid 36¢ (from NO sell 64¢): 20 contracts
YES bid 35¢ (from NO sell 65¢): 12 contracts  ← worst for seller
```

Wait — that means lower NO ask prices give higher YES bids. The NO array sorted ascending gives YES bids sorted descending. The best YES bid comes from the *lowest* NO ask.

**Actually, let's restart cleanly.** The NO array `[60, 50], [61, 25], [62, 35], [63, 18], [64, 20], [65, 12]` represents sell orders for NO contracts at those prices. The lowest NO sell is 60¢, meaning someone will sell you a NO contract for 60¢. This corresponds to a YES bid of 40¢ (100 - 60). The highest NO sell is 65¢, corresponding to YES bid of 35¢.

**Corrected YES bid side (best to worst):**

```
YES bid 40¢ (from NO sell at 60¢): 50 contracts  ← BEST bid
YES bid 39¢ (from NO sell at 61¢): 25 contracts
YES bid 38¢ (from NO sell at 62¢): 35 contracts
YES bid 37¢ (from NO sell at 63¢): 18 contracts
YES bid 36¢ (from NO sell at 64¢): 20 contracts
YES bid 35¢ (from NO sell at 65¢): 12 contracts  ← worst bid
```

Now the spread makes more sense:

```
Best YES ask: 35¢
Best YES bid: 40¢  (from NO sell at 60¢)
Spread: 35¢ - 40¢ = -5¢  (negative → crossed book!)
```

A crossed book means you could buy YES at 35¢ and immediately sell at 40¢ for a 5¢ profit. In practice, the matching engine prevents this — if these orders existed simultaneously, they'd fill against each other immediately. If you see this in an API response, it means orders were placed between the book snapshot and your query, or the response is slightly stale.

In a normal (uncrossed) orderbook, the best YES bid is at or below the best YES ask.

## What Thin vs. Thick Markets Look Like

### Thick Market (High Liquidity)

```
KXRECSSNBER-26 (US Recession 2026):
  Best ask: 35¢  (180 contracts)
  Best bid: 34¢  (150 contracts)
  Spread: 1¢
  Depth within 5¢ (ask): 820 contracts
  Depth within 5¢ (bid): 710 contracts

  → You can fill 500 contracts with ~1.5¢ slippage
  → Spread is tight
  → Reliable price discovery
```

### Thin Market (Low Liquidity)

```
KXOBSCURE-26 (Some niche contract):
  Best ask: 42¢  (3 contracts)
  Best bid: 31¢  (5 contracts)
  Spread: 11¢
  Depth within 5¢ (ask): 15 contracts
  Depth within 5¢ (bid): 12 contracts

  → Buying 20 contracts would cost 45¢+ average (3¢+ slippage)
  → Spread eats most of your edge
  → Price could be anywhere between 31¢ and 42¢
```

Thin markets are dangerous because:

1. The "market price" is misleading — with an 11¢ spread, the true mid is ~36.5¢, not 42¢
2. Your entry moves the price, advertising your position to other participants
3. Exiting is worse than entering — you'll eat an 11¢ spread on the way out
4. Other large orders can gap the price through your position

## SimpleFunctions Liquidity Scoring

The `sf liquidity` command assigns each contract a liquidity score based on two factors:

```
HIGH liquidity:
  - Spread ≤ 2¢
  - Ask depth within 5¢ ≥ 500 contracts
  - You can trade meaningful size without moving the market

MEDIUM liquidity:
  - Spread ≤ 4¢
  - Ask depth within 5¢ ≥ 200 contracts
  - Tradeable but size with care

LOW liquidity:
  - Spread > 4¢ OR depth < 200
  - Thin market. Small positions only.
  - Consider whether the edge justifies the execution cost

UNTRADEABLE:
  - Spread > 10¢ OR depth < 50
  - Don't trade this. The execution costs exceed any reasonable edge.
```

**Example output:**

```bash
sf liquidity --topic recession

Contract                  Spread   Depth(5¢)   Score
KXRECSSNBER-26 YES        1¢       820         HIGH
KXRECSSQ3-26 YES          3¢       340         MEDIUM
KXGDPNEG-26Q2 YES         7¢       85          LOW
KXUNEMPLOY5-26 YES        12¢      30          UNTRADEABLE
```

## Orderbook Patterns to Watch For

### Pattern 1: The Wall

A single large order at a specific price level:

```
35¢:  8 contracts
36¢:  15 contracts
37¢:  500 contracts  ← wall
38¢:  12 contracts
```

A wall at 37¢ means someone is willing to sell 500 contracts at that price. This often acts as resistance — the price is unlikely to move above 37¢ until the wall is eaten. But walls can be pulled (cancelled) at any time, so don't treat them as permanent.

### Pattern 2: The Cliff

Depth drops off sharply after the first few levels:

```
35¢:  200 contracts
36¢:  5 contracts
37¢:  2 contracts
38¢:  0 contracts
```

The first 200 contracts are cheap to fill, but after that, you're paying whatever the next person wants. Be careful with market orders on cliff orderbooks — you could fill at wildly different prices.

### Pattern 3: The Vacuum

No orders within a wide price range:

```
35¢:  50 contracts
(nothing at 36-44¢)
45¢:  30 contracts
```

If someone buys 50 contracts at 35¢, the next fill is at 45¢. A 10¢ vacuum means any moderate-size order will gap the price. This is common on low-activity contracts near resolution dates.

## Practical Takeaways

1. **Always check depth before sizing.** Your target position should be fillable within 2-3¢ of the best ask. If not, reduce size.

2. **Compute your average fill, not just the best ask.** The best ask is advertising. The average fill is reality.

3. **The NO side is your bid side.** Learn to read it inverted. The cheapest NO sell is your best YES bid.

4. **Spreads above 5¢ are a red flag.** Your edge needs to exceed the spread just to break even on a round trip.

5. **Check depth at multiple times.** Orderbooks are thinnest overnight and on weekends. The depth you see at 2pm on Tuesday won't be there at 3am on Sunday.

6. **Use limit orders.** On anything below HIGH liquidity, never use market orders. Set your limit at or near the best ask and wait. Patience saves cents, and cents are edge.

The orderbook is not a static picture — it's a living, breathing representation of what every other market participant is willing to do right now. Learning to read it is the difference between having edge and having the illusion of edge.