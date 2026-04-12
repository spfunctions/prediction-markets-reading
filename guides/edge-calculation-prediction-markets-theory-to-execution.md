# Edge Calculation in Prediction Markets: From Theory to Execution

> Theoretical edge means nothing if you can't execute it. This article covers the full edge stack: theoretical edge, spread cost, slippage, depth-adjusted edge, and when to walk away from a trade entirely.

**Category:** risk | **Author:** SimpleFunctions | **Reading time:** 9 min

---
Every prediction market trader talks about "edge." Few can tell you their actual edge after execution costs. This gap between theoretical edge and executable edge is where most traders quietly lose money without realizing it.

This article walks through the full edge calculation stack — from the number your model produces to the number your account actually sees after a trade is filled.

## Level 1: Theoretical Edge

Theoretical edge is the simplest calculation. Your model says a contract is worth X. The market says it's worth Y. The difference is your edge.

**Example:**

```
Contract:     KXRECSSNBER-26 (US recession 2026)
Market price: 35¢ (YES side, best ask)
Thesis price: 72¢ (your causal model's output)

Theoretical edge = thesis price - market price
                 = 72¢ - 35¢
                 = +37¢
```

A 37-cent edge on a contract that costs 35¢ looks enormous. If you're right, you're buying something worth 72¢ for 35¢. That's a 106% return on capital if it settles YES.

But theoretical edge is a fantasy number. You can't trade at the theoretical edge. Here's why.

## Level 2: Spread-Adjusted Edge

The "market price" of 35¢ is the best ask — the cheapest price someone is willing to sell YES contracts to you. But there's also a best bid — the highest price someone is willing to buy YES contracts from you.

**Real orderbook snapshot:**

```
KXRECSSNBER-26 Orderbook (YES side):
  Best bid:  33¢  (12 contracts)
  Best ask:  35¢  (8 contracts)
  Spread:    2¢
```

When you buy, you pay the ask (35¢). When you sell, you receive the bid (33¢). The spread is the market maker's cut.

**Spread-adjusted edge:**

```
To enter:   you pay 35¢ (the ask)
To exit:    you receive 33¢ (the bid) — if you need to sell before settlement
Round-trip spread cost: 2¢

Spread-adjusted edge (if held to settlement) = 72¢ - 35¢ = +37¢
Spread-adjusted edge (if exiting early)      = 72¢ - 35¢ - 2¢ = +35¢
```

If you plan to hold the contract to settlement (it resolves YES or NO), the spread only costs you on entry. If you might exit early, the spread costs you on both sides.

For most thesis-driven trades, you're holding to settlement or near settlement. So the spread cost is 0¢ on the exit side. But you should always know the spread because it tells you about market liquidity — more on that below.

## Level 3: Slippage

Slippage is what happens when you want to buy more contracts than the best ask has available. You "eat" through the orderbook, filling at progressively worse prices.

**Full orderbook depth (YES ask side):**

```
Price    Contracts Available
35¢      8
36¢      15
37¢      22
38¢      45
39¢      30
40¢      80
```

If you want to buy 100 contracts, here's what happens:

```
Fill 1:   8 contracts @ 35¢  =  $2.80
Fill 2:  15 contracts @ 36¢  =  $5.40
Fill 3:  22 contracts @ 37¢  =  $8.14
Fill 4:  45 contracts @ 38¢  = $17.10
Fill 5:  10 contracts @ 39¢  =  $3.90   (partial fill to reach 100)
────────────────────────────────────────
Total:  100 contracts          = $37.34
Average fill price:              36.5¢  (not 35¢!)

Slippage = average fill - best ask
         = 36.5¢ - 35¢
         = 1.5¢
```

**Slippage-adjusted edge:**

```
Theoretical edge:            +37¢  (thesis 72¢ vs best ask 35¢)
After slippage (100 contracts): +35.5¢  (thesis 72¢ vs avg fill 36.5¢)
```

For 100 contracts, slippage costs 1.5¢. Not catastrophic. But try 500 contracts:

```
Fill through 35¢ to 40¢:  200 contracts filled
Remaining 300 contracts: would need to fill at 41¢+ (if depth exists)

At 500 contracts, average fill might be 39¢+
Slippage = 4¢+
```

Slippage scales nonlinearly with order size. Double the order doesn't mean double the slippage — it usually means 3-4x the slippage because the deeper levels of the orderbook are thinner.

## Level 4: Depth-Adjusted Edge

This is the number that actually matters. Depth-adjusted edge answers: "Given how much I want to trade, what's my real edge after all costs?"

**The formula:**

```
Depth-adjusted edge = thesis price - average fill price - fees

For our 100-contract example:
  = 72¢ - 36.5¢ - 0¢ (Kalshi charges no maker/taker fees on most contracts)
  = +35.5¢ per contract
  = $35.50 total edge on 100 contracts
```

Note: Kalshi charges fees on settlement, not on entry. If your contract settles YES, Kalshi takes a percentage of the payout. As of 2026, the fee schedule is tiered based on volume. For a retail trader, expect roughly 1-2¢ per winning contract in settlement fees.

**Including settlement fees:**

```
Depth-adjusted edge (net of all costs):
  = thesis price - avg fill - settlement fee (if winning)
  = 72¢ - 36.5¢ - 1.5¢ (estimated settlement fee)
  = +34¢ per contract

Expected value per contract:
  = (probability of YES * net payout if YES) - (probability of NO * cost)
  = (0.72 * (100¢ - 36.5¢ - 1.5¢)) - (0.28 * 36.5¢)
  = (0.72 * 62¢) - (0.28 * 36.5¢)
  = 44.64¢ - 10.22¢
  = +34.42¢ expected profit per contract
```

## When NOT to Take an Edge

Having a positive edge doesn't mean you should trade. Here are four situations where you should pass:

### 1. Thin Market (Depth < 200 Contracts)

If the total ask-side depth within 5¢ of the best ask is less than 200 contracts, the market is thin. Problems with thin markets:

- Your entry will move the price significantly
- If you need to exit, the bid side is probably even thinner
- Other participants can see your large order and front-run you
- Price discovery is unreliable — the "market price" might not reflect real demand

**Rule of thumb:** If you can't fill your target position within 3¢ of the best ask, the market is too thin for that size.

### 2. Event Imminent (< 48 Hours to Resolution)

When a contract is about to resolve, the edge calculation changes. The market price is converging on the true probability. Any remaining edge is likely either:

- **Genuine** — you have information the market doesn't (rare and possibly illegal depending on the information)
- **Liquidity premium** — the last few cents of convergence happen right at resolution, and you're taking settlement timing risk

Unless you have a specific informational advantage, avoid entering positions within 48 hours of resolution. The edge is usually a mirage.

### 3. Correlated Positions

If you already hold KXOIL100-26Q3 (oil > $100) and you're considering KXRECSSNBER-26 (recession), check the causal tree. These contracts share upstream nodes (n1 war persists, n2 Hormuz blocked). They're correlated bets.

Adding the recession contract doesn't diversify your portfolio — it doubles down on the same upstream risk. If n1.3 (no diplomatic exit) breaks and a deal is reached, both positions lose simultaneously.

**The check:** If two contracts share more than 50% of their causal tree ancestry, treat the edge calculation as a portfolio problem, not an individual contract problem. The combined edge might still be positive, but the risk is concentrated.

### 4. Model Uncertainty Is High

Your causal model says 72%, but how confident are you in that number? If the key upstream nodes have wide uncertainty ranges:

```
n3.2 OPEC doesn't compensate: 68% (but honestly could be 40-90%)
n4.1 Inflation above 5%:      72% (but depends on oil price path)
```

When multiple nodes have wide ranges, the compounding uncertainty makes your terminal probability unreliable. A thesis price of "72%" that's really "somewhere between 30% and 90%" isn't a 37¢ edge — it's a coin flip.

**The check:** If more than two nodes in your critical path have confidence ranges wider than ±20 percentage points, cut your position size in half or pass entirely.

## The Edge Calculation Workflow

In practice, the `sf edges` command runs this entire stack automatically:

```bash
sf edges --thesis iran-war

Thesis: iran-war
Updated: 12 minutes ago

Contract               Thesis   Market   Theo.Edge   Exec.Edge(100)   Depth    Liquidity
KXRECSSNBER-26 YES     72¢      35¢      +37¢        +35.5¢            420      HIGH
KXOIL100-26Q3 YES      64¢      41¢      +23¢        +21.8¢            380      HIGH
KXFEDCUT-26JUN NO      58¢      35¢      +23¢        +19.2¢            180      MEDIUM
KXIRANWAR-26 YES       88¢      84¢      +4¢         +2.1¢             95       LOW
```

The columns tell you everything:

- **Theo.Edge**: The fantasy number (thesis - best ask)
- **Exec.Edge(100)**: What you'd actually get buying 100 contracts (after slippage)
- **Depth**: Total contracts available within 5¢ of best ask
- **Liquidity**: HIGH (spread ≤2¢ + depth ≥500), MEDIUM (spread ≤4¢ + depth ≥200), LOW (everything else)

The recession contract has the best combination: large executable edge, good depth, high liquidity. The Iran war contract has almost no executable edge despite the thesis being highly confident — the market already agrees with you.

## The Mental Model

Think of edge as a funnel:

```
Theoretical edge:     +37¢     ← What your model says
After spread:         +35¢     ← What you can buy and sell at
After slippage:       +35.5¢   ← What you actually fill at (for your size)
After fees:           +34¢     ← What hits your account
After model error:    ???       ← The part you can't calculate
```

Every layer of the funnel reduces your edge. The first three layers are calculable and mechanical. The fourth — model error — is the one that determines whether you actually make money. That's why the causal tree exists: to make your probability estimate as rigorous as possible, so that the gap between "after fees" and "after model error" is as small as possible.

The traders who blow up are the ones who only look at the top of the funnel. "37 cents of edge!" They ignore the spread, the slippage, the fees, and most critically, the possibility that their 72% estimate is wrong.

Calculate your executable edge. If it's still attractive after every cost layer, take the trade. If it erodes to single digits by the time you account for execution reality, pass and find a better opportunity.