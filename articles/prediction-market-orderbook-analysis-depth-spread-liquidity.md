# Prediction Market Orderbook Analysis: Reading Depth, Spread, and Liquidity

> Everything you know about stock orderbooks is almost correct for prediction markets. The "almost" will cost you money if you ignore it.

**Category:** markets | **Author:** SimpleFunctions Research | **Reading time:** 12 min | **Published:** 2026-03-24

---
# Prediction Market Orderbook Analysis: Reading Depth, Spread, and Liquidity

*Everything you know about stock orderbooks is almost correct for prediction markets. The "almost" will cost you money if you ignore it.*

---

You have read Level 2 data before. You understand bid-ask spreads, market depth, and the mechanics of limit orders. Good. That foundation transfers to prediction markets, but with structural differences that change how you interpret every signal on the book. A 3-cent spread does not mean the same thing at 50 cents as it does at 5 cents. A deep bid wall does not always mean what it means on equities. And settlement is not a gradual process — it is binary annihilation.

This guide covers how to read prediction market orderbooks as they actually work, not as analogies to stock markets. We will use real output formats from SimpleFunctions tools and real market structures from Kalshi and Polymarket.

## The Four Structural Differences That Change Everything

### 1. Binary Outcome, Not Continuous Price

A stock can close at any price. Apple can be worth $142.37 or $198.51 or anywhere in between. A prediction market contract resolves to one of two values: 0 cents or 100 cents. There is no middle ground. There is no gradual appreciation over time.

This means the orderbook is not pricing a range of future values. It is pricing a single probability. When you see YES trading at 35 cents, the market consensus is a 35% probability of the event occurring. That is the entire informational content of the price.

The practical consequence: there is no "growth stock" equivalent. You cannot buy at 35 cents and hope the market slowly appreciates to 40 cents through general momentum. The price moves only when the market's probability estimate changes based on new information. Every cent of movement represents a discrete shift in assessed likelihood.

### 2. YES + NO Always Sum to $1.00

This is the constraint that makes prediction markets a closed system. If YES is offered at 35 cents, the implied NO price is 65 cents. If you can buy YES at 35c on the ask, someone else can effectively buy NO at 65c.

In equities, there is no structural inverse. Shorting a stock requires borrowing shares, paying interest, and managing margin. In prediction markets, buying NO is a first-class operation with identical mechanics to buying YES.

This creates a mirror-image orderbook:

```
KXWTIMAX — "Will WTI crude close above $90 by June 2026?"

YES Side                          NO Side
─────────────────────────────────────────────────
ASK  37c  ███████  420 contracts
ASK  36c  █████████████  780 contracts
                                  BID  65c  ██████████  600 contracts
BID  34c  ████████████  710 contracts
BID  33c  ██████  350 contracts
                                  ASK  66c  ████████████  720 contracts
                                  ASK  67c  ██████  380 contracts
─────────────────────────────────────────────────
YES Spread: 34c / 36c (2c)       NO Spread: 65c / 66c (1c)
Midpoint: 35c                    Midpoint: 65.5c
```

Notice the spreads are not identical. The YES spread is 2 cents; the NO spread is 1 cent. This asymmetry tells you something about where the liquidity providers are positioned. In this case, the NO side has tighter markets, suggesting more confidence in the NO outcome or simply more active market-making on that side.

### 3. Settlement Is Binary Annihilation

When a stock earnings report comes out, the stock might gap 5% or 10%. The orderbook reprices, but the security continues to exist. When a prediction market settles, every contract is either worth exactly 100 cents or exactly 0 cents. There is no partial outcome.

This means the risk profile near the extremes is fundamentally different from anything in equities. A contract trading at 95 cents has 5 cents of upside and 95 cents of downside. The risk-reward is asymmetric by a factor of 19:1. The orderbook near these extremes behaves differently — spreads widen, depth thins out, and the bid-ask dynamics shift to account for the catastrophic loss scenario.

### 4. Spread as Percentage of Price, Not Absolute Value

This is where most stock traders get burned on their first prediction market trades.

On equities, a 3-cent spread on a $150 stock is 0.02% friction. Irrelevant for any position size. On prediction markets, spread friction scales inversely with price:

```
Price    Spread    Friction (% of price)    Round-trip cost
─────────────────────────────────────────────────────────────
50c      3c        6.0%                     ~6c per contract
35c      3c        8.6%                     ~6c per contract
15c      3c        20.0%                    ~6c per contract
5c       3c        60.0%                    ~6c per contract
3c       3c        100.0%                   ~6c per contract
```

The absolute cost of crossing the spread is the same 6 cents round-trip in every case. But at 5 cents, you are paying 60% of the contract value in friction. You need the event probability to be roughly 8% (not 5%) just to break even after the spread on the YES side.

This is why low-probability markets are structurally different from mid-probability markets. They are not just "cheap contracts." They are high-friction environments where the spread consumes most of your edge.

## Reading the Book: What to Look For

### Bid Depth and Ask Depth

Depth tells you how many contracts you can execute at each price level before moving the market. Here is a real-format example from `sf book`:

```
$ sf book KXWTIMAX

Market: Will WTI crude oil close above $90/barrel by June 30, 2026?
Status: OPEN | Last: 35c | Volume (24h): 12,840 contracts

YES Orderbook
─────────────────────────────────────────────────
ASKS (selling YES / offering to you)
  39c    150 contracts   ░░░░
  38c    280 contracts   ░░░░░░░░
  37c    420 contracts   ░░░░░░░░░░░░
  36c    780 contracts   ░░░░░░░░░░░░░░░░░░░░░░
─── spread: 2c ──────────────────────────────────
BIDS (buying YES / your exit)
  34c    710 contracts   ░░░░░░░░░░░░░░░░░░░░
  33c    350 contracts   ░░░░░░░░░░
  32c    520 contracts   ░░░░░░░░░░░░░░░
  31c    190 contracts   ░░░░░░

Bid Depth (3c): 1,770 contracts ($619.50 notional)
Ask Depth (3c): 1,630 contracts ($603.10 notional)
Spread: 2c (5.7% of midpoint)
Liquidity Score: 7.2 / 10
```

Key metrics to extract:

**Depth within 3 cents of the inside.** This is your actionable liquidity. Anything beyond 3 cents from the best bid or ask is too far from the current price to be relevant for immediate execution. In the example above, 1,770 contracts on the bid side and 1,630 on the ask side within 3 cents. That is a reasonably liquid market.

**The depth ratio.** Bid depth divided by ask depth. Here it is 1,770 / 1,630 = 1.09. Nearly balanced. A ratio above 1.5 or below 0.67 is where you start paying attention to directional pressure.

**Notional depth.** The dollar value, not just the contract count. 1,770 contracts at an average price of 33 cents is about $584 in notional value. On Kalshi, where single contracts represent $1 of risk, this tells you the approximate dollar commitment sitting on the book.

### Spread: The Silent Tax

The spread is the difference between the best bid and best ask. On the KXWTIMAX example, the spread is 2 cents (34c bid / 36c ask).

But the spread alone is meaningless without context. You need to evaluate it as a percentage of the midpoint price:

```
Spread Assessment Framework
─────────────────────────────
Spread / Midpoint     Assessment
─────────────────────────────
< 2%                  Excellent. Institutional-grade liquidity.
2% - 5%               Good. Tradeable for directional positions.
5% - 10%              Moderate. Need strong conviction to justify friction.
10% - 20%             Poor. Only trade with significant edge.
> 20%                 Untradeable. Your order IS the market.
```

At a 35c midpoint, a 2c spread is 5.7% — in the moderate range. Tradeable, but you feel the friction. Compare this to a major stock where the spread might be 0.01% of the price. Prediction markets are structurally wider because the total price range is $0 to $1.00, there are no dedicated market makers with obligations, and volume is orders of magnitude lower.

### Liquidity Score

The SimpleFunctions liquidity score combines spread, depth, volume, and consistency into a single 1-10 metric:

```
Liquidity Score Interpretation
──────────────────────────────
9-10    Deep, tight markets. Multi-hundred contract fills without slippage.
7-8     Solid liquidity. Can trade 50-200 contracts with minimal impact.
5-6     Thin but tradeable. Limit orders only. Expect partial fills.
3-4     Illiquid. Wide spreads, shallow depth. Small positions only.
1-2     Ghost town. Book is decorative. Do not trade.
```

Use `sf liquidity` to scan across categories and find where the actionable markets are.

## Thin Markets: When Your Order IS the Market

A market is thin when the spread exceeds 5 cents and the depth within 3 cents of the inside is below 100 contracts. These markets are traps for anyone using market orders.

```
$ sf book KXTHINEX

Market: Will the FAA approve flying cars by December 2026?
Status: OPEN | Last: 8c | Volume (24h): 47 contracts

YES Orderbook
─────────────────────────────────────────────────
ASKS
  15c     30 contracts   ░░░░░░░░
  12c     15 contracts   ░░░░
  10c     25 contracts   ░░░░░░░
─── spread: 5c ──────────────────────────────────
BIDS
   5c     40 contracts   ░░░░░░░░░░░
   4c     20 contracts   ░░░░░░
   3c     55 contracts   ░░░░░░░░░░░░░░░

Bid Depth (3c): 115 contracts ($13.80 notional)
Ask Depth (3c): 70 contracts ($8.40 notional)
Spread: 5c (71.4% of midpoint at 7.5c)
Liquidity Score: 2.1 / 10
```

The spread is 5 cents, which sounds small until you realize the midpoint is 7.5 cents. That 5 cents is 71.4% of the contract price. If you market-buy at 10c and the event does not occur, you lose 10 cents. If you try to exit by selling at the bid of 5c, you have already lost 50% of your entry price in friction alone.

With only 70 contracts on the ask within 3 cents, placing an order for 100 contracts would eat through the entire visible ask-side depth and push the price to 15c or beyond. Your 100-contract order would fill at a blended price well above 10c. You are not a participant in this market. You are the market.

**Rules for thin markets:**

1. Limit orders only. Never cross the spread.
2. Size your order to no more than 25% of the visible depth on your side.
3. Accept that your fill may take hours or days.
4. Consider whether the wide spread itself is information — maybe there is no edge here, and the market makers have deliberately stepped away.

## Executable Edge: The Only Edge That Matters

Your theoretical edge is the difference between your estimated probability and the market price. If you believe an event has a 45% chance of occurring and YES is trading at 35 cents, your raw edge is 10 cents per contract.

But you do not trade at the midpoint. You trade at the ask (to buy) or the bid (to sell). The spread consumes part of your edge.

**Executable Edge = Raw Edge - (Spread / 2)**

Using the KXWTIMAX example:

```
Your estimate:     45%  (you would pay up to 45c)
Market ask:        36c
Market bid:        34c
Spread:            2c
Midpoint:          35c

Raw edge:          45c - 35c = 10c
Half-spread cost:  2c / 2 = 1c
Executable edge:   10c - 1c = 9c

Verdict: Strong. The spread eats only 10% of your edge.
```

Now consider the same 10-cent raw edge in a thin market:

```
Your estimate:     15%  (you would pay up to 15c)
Market ask:        12c
Market bid:        4c
Spread:            8c
Midpoint:          8c

Raw edge:          15c - 8c = 7c  (raw edge is already smaller)
Half-spread cost:  8c / 2 = 4c
Executable edge:   7c - 4c = 3c

Verdict: Marginal. The spread eats 57% of your edge.
```

And an extreme case where the spread destroys the edge entirely:

```
Your estimate:     10%  (you would pay up to 10c)
Market ask:        9c
Market bid:        1c
Spread:            8c
Midpoint:          5c

Raw edge:          10c - 5c = 5c
Half-spread cost:  8c / 2 = 4c
Executable edge:   5c - 4c = 1c

Verdict: Not worth the risk. 1c of executable edge on a
         contract with 9c of downside is a terrible ratio.
```

The rule of thumb: if the spread consumes more than 40% of your raw edge, the trade is not worth executing. You are paying the market maker more than you are being compensated for your informational advantage.

## Reading Depth Asymmetry

Depth asymmetry — when the bid side is significantly deeper than the ask side or vice versa — is one of the more reliable short-term signals on a prediction market orderbook. It does not predict the outcome of the event. It predicts the near-term price movement.

```
Scenario A: Bid-Heavy Book (accumulation signal)

ASKS
  38c     90 contracts    ░░░
  37c    150 contracts    ░░░░░
  36c    200 contracts    ░░░░░░░
─── spread: 2c ──────────────────
BIDS
  34c    800 contracts    ░░░░░░░░░░░░░░░░░░░░░░░░░░
  33c    650 contracts    ░░░░░░░░░░░░░░░░░░░░░
  32c    500 contracts    ░░░░░░░░░░░░░░░░

Bid depth: 1,950    Ask depth: 440    Ratio: 4.4:1
```

When bid depth is 4x ask depth, someone is accumulating. They have placed large orders on the bid side and the sell-side liquidity is thin. The market can move up quickly because there is little resistance above the current price. One informed buyer lifting the 200 contracts at 36c and the 150 at 37c spends only $56.50 and pushes the price to 38c.

```
Scenario B: Ask-Heavy Book (distribution signal)

ASKS
  38c    750 contracts    ░░░░░░░░░░░░░░░░░░░░░░░░░
  37c    600 contracts    ░░░░░░░░░░░░░░░░░░░░
  36c    500 contracts    ░░░░░░░░░░░░░░░░
─── spread: 2c ──────────────────
BIDS
  34c    120 contracts    ░░░░
  33c     80 contracts    ░░░
  32c     60 contracts    ░░

Bid depth: 260    Ask depth: 1,850    Ratio: 0.14:1
```

Heavy asks with thin bids means someone is distributing — selling into any demand. A single market sell of 120 contracts wipes the bid at 34c and the price drops to 33c. The downside path has very little support.

**Caution:** Depth asymmetry can be manipulated. A participant can place large limit orders with no intention of getting filled, only to pull them ("spoofing") when the price moves toward their real position. This is less common on prediction markets than on crypto exchanges, but it happens. Look for depth that persists over hours, not minutes.

## Cross-Venue Arbitrage: Real Edge or Liquidity Premium?

The same event often trades on multiple platforms. Kalshi, Polymarket, and other venues may all offer contracts on whether the Federal Reserve will cut rates, whether a candidate will win an election, or whether a commodity will hit a price target. The prices do not always agree.

```
Event: "Will US GDP growth exceed 3% in Q2 2026?"

Kalshi (KXGDPQ2):
  YES Ask: 36c    YES Bid: 34c    Midpoint: 35c

Polymarket (same event):
  YES Ask: 39c    YES Bid: 37c    Midpoint: 38c

Price gap: 3c (Kalshi cheaper by 3c at the midpoint)
```

Is this a free 3 cents? Almost never.

**Costs that close the gap:**

1. **Capital lockup.** To arbitrage, you need to buy YES on Kalshi at 36c and sell YES (buy NO) on Polymarket at 61c (100c - 39c = 61c). You have spent 97c to lock in a guaranteed 100c payout — a 3c profit per contract. But your capital is locked until the event resolves, which might be months away. That 3c on 97c of capital is a 3.1% return. Annualized over, say, 4 months, it is about 9.3%. Decent, but not extraordinary, and you bear platform risk the entire time.

2. **Withdrawal timing and fees.** Getting money onto and off of each platform takes time and may incur fees. Polymarket operates on crypto rails, which means gas fees and potential slippage on stablecoin conversions. Kalshi uses traditional banking, which means ACH delays.

3. **Settlement risk.** What if the two platforms resolve the same event differently due to different contract specifications? "GDP growth exceeds 3%" might reference different BEA releases or different revision rounds on different platforms.

4. **Regulatory risk.** Kalshi is CFTC-regulated and US-accessible. Polymarket has different jurisdictional considerations. Holding positions on both platforms may complicate your tax and reporting obligations.

**When cross-venue gaps are real:**

- Gaps exceeding 5 cents on liquid markets (Liquidity Score > 6 on both venues) with resolution within 30 days. The short lockup period improves the annualized return, and the liquidity ensures you can actually execute at the quoted prices.

- Gaps that appear suddenly after a news event, before one platform's participants have updated their orders. These close quickly — you have minutes, not hours.

- Persistent gaps on obscure markets where one venue has structurally different participants. A Kalshi market dominated by US retail traders may price differently from a Polymarket market with international crypto-native participants.

```
Quick Arbitrage Assessment
────────────────────────────────────────────────
Gap    Lockup     Annualized    Verdict
────────────────────────────────────────────────
2c     4 months   ~6%           Not worth the platform risk
3c     1 month    ~36%          Interesting if both sides liquid
5c     2 weeks    ~130%         Execute immediately if depth allows
5c     6 months   ~10%          Marginal after fees
```

## The Liquidity Scanner: Finding Tradeable Markets

Running `sf liquidity` scans across 18 topic categories and surfaces markets where the combination of spread, depth, and volume creates favorable trading conditions.

```
$ sf liquidity --min-score 6 --sort spread

Liquidity Scanner — Markets with Score >= 6.0
Sorted by: tightest spread
────────────────────────────────────────────────────────────────────

Ticker        Event                              Bid   Ask   Sprd  Depth  Score
──────────────────────────────────────────────────────────────────────────────────
KXFEDRATE     Fed funds rate >= 4.5% Jul 2026    52c   53c   1c    2,400  9.1
KXSPXMAX      S&P 500 above 5,800 Jun 2026       61c   62c   1c    1,850  8.8
KXWTIMAX      WTI crude above $90 Jun 2026        34c   36c   2c    1,770  7.2
KXUNEMPLOY    Unemployment below 4.2% Q2          44c   46c   2c    1,200  7.0
KXCPIMAX      CPI above 3.0% May 2026            27c   30c   3c      890  6.4
KXGDPQ2       GDP growth above 3% Q2 2026        34c   37c   3c      650  6.2
KXBTCMAX      BTC above $120K Jun 2026            22c   25c   3c      580  6.1
KXHOUSTART    Housing starts above 1.5M May       38c   41c   3c      420  6.0

Markets scanned: 847 | Meeting criteria: 8 | Avg score (all): 3.4
```

What this tells you at a glance: of 847 active markets, only 8 meet the minimum liquidity threshold of 6.0. The average score across all markets is 3.4, which means the typical prediction market is thin and wide-spread. The tradeable universe is a small subset of the listed universe.

**How to use this data:**

1. Start with the highest-score markets and work down. The top two (KXFEDRATE, KXSPXMAX) have 1-cent spreads and deep books. These are markets where you can express a view with minimal friction.

2. Filter by your areas of knowledge. If you have an informational edge on energy markets, KXWTIMAX at a 7.2 score is your target. If you follow macro closely, KXFEDRATE and KXUNEMPLOY are where you should focus.

3. Revisit daily. Liquidity shifts. A market at 6.0 today might drop to 4.0 after a major news event causes market makers to widen their quotes, or it might jump to 8.0 as resolution approaches and volume increases.

4. Cross-reference with your edge. A liquid market where you have no edge is not useful. An illiquid market where you have a strong edge is frustrating but potentially profitable with patience.

## When NOT to Trade

Some markets are structurally untradeable regardless of your conviction. Recognizing these saves you from capital lockup and unnecessary losses.

### Markets at 0-3 Cents or 97-100 Cents

These are effectively settled. A market at 2 cents means the consensus probability is 2%. To profit on the YES side, you need the event to actually occur — a 50:1 payout. But the spread at this level is catastrophic:

```
Price: 2c    Ask: 3c    Bid: 1c    Spread: 2c

Cost to enter YES: 3c
If event occurs:  +97c profit (you paid 3c, receive $1)
If event fails:   -3c loss

Required probability to break even: 3%
Market estimate: 2%

You need to be 50% more accurate than the market to
break even, AND your edge only yields ~1c of executable
edge after the spread.
```

On the NO side, it is equally bad. Buying NO at 98c for a maximum profit of 2c ties up $0.98 of capital for a $0.02 return. That is a 2% return over whatever time remains until settlement.

The exception: if you have strong, specific information that a near-certain event will not occur (buying NO at 98c when you know something the market does not), the 2c potential profit per contract can be worth it at scale — 10,000 contracts yields $200 on $9,800 of capital. But this is a niche situation.

### Markets with Spread > 10 Cents

```
Price: 40c    Ask: 45c    Bid: 35c    Spread: 10c

Executable edge required just to break even on a
round-trip: 10c (25% of the contract price)
```

A 10-cent spread means you lose 10 cents on every round-trip. To profit, you need a raw edge of at least 10 cents — meaning your probability estimate needs to differ from the market by at least 10 percentage points. That is an enormous informational advantage to claim.

If you find yourself trading markets with 10-cent spreads, ask: do I really know something the market does not, to the tune of 10+ percentage points? If the honest answer is "maybe," the trade is not worth it.

### Markets with No Depth on Your Side

```
ASKS
  42c    500 contracts    ░░░░░░░░░░░░░░░░
  41c    300 contracts    ░░░░░░░░░░
  40c    200 contracts    ░░░░░░░
─── spread: 3c ──────────────────
BIDS
  37c     12 contracts    ░
  35c      5 contracts    
  30c      8 contracts    

You want to buy YES. The ask side has 1,000 contracts
within 3c — you can get in.

But the bid side has 25 contracts within 7 cents. If you
need to exit before settlement, you are selling into a
vacuum. A 100-contract sell would crash the bid to 30c
or lower.
```

Entering a market where you can get in but cannot get out is one of the most common mistakes new prediction market traders make. Always check both sides of the book before entering. If exit liquidity is thin, you are committing to hold until settlement — make sure your thesis and your timeline allow for that.

### The Checklist

Before executing any prediction market trade, run through this:

```
Pre-Trade Checklist
─────────────────────────────────────────────
[ ] Spread < 5% of contract price?
[ ] Depth > 100 contracts within 3c on BOTH sides?
[ ] Liquidity Score > 5?
[ ] Executable edge > 3c after spread cost?
[ ] Exit liquidity exists (bid depth sufficient)?
[ ] Contract price between 5c and 95c?
[ ] Time to settlement > 7 days?
[ ] No conflicting position on another venue?

If any box is unchecked, re-evaluate. If three or more
are unchecked, do not trade.
```

## Putting It Together: A Complete Orderbook Read

Here is how to perform a full analysis using the `sf` toolkit, from market selection to trade decision.

**Step 1: Scan for liquid markets.**

```
$ sf liquidity --min-score 6 --category macro
```

Identify markets where you have both liquidity and informational coverage.

**Step 2: Pull the orderbook.**

```
$ sf book KXFEDRATE
```

Read the full depth. Note the spread, bid/ask ratio, and total depth within 3 cents.

**Step 3: Assess your edge.**

Compare your probability estimate to the midpoint. Calculate the executable edge after subtracting half the spread.

**Step 4: Check the other venue.**

```
$ sf book KXFEDRATE --venue polymarket
```

If the same event is on multiple platforms, compare prices. A gap above 3 cents on a liquid market with short time to resolution is worth investigating for arbitrage.

**Step 5: Check depth asymmetry for timing.**

If you are going to enter, evaluate whether the depth profile suggests imminent price movement. Bid-heavy books may mean the price moves up before you can fill your buy order. Ask-heavy books may mean you get a better entry if you wait.

**Step 6: Place the order.**

Use a limit order at or near the best bid (if buying) or best ask (if selling). Do not cross the spread unless your executable edge is large enough to absorb the friction and you need immediate execution.

**Step 7: Monitor.**

```
$ sf watch KXFEDRATE --alerts spread,depth
```

Set alerts for spread widening (which degrades your executable edge) and depth changes (which affect your ability to exit).

---

Prediction market orderbooks reward patience and penalize urgency. The friction is higher than equities, the liquidity is thinner, and the settlement mechanics create risk profiles that do not exist in traditional markets. But the informational efficiency is lower, which means edges exist for those who can identify and size them correctly.

The book tells you the price. The depth tells you the conviction. The spread tells you the cost of being wrong about both. Read all three before you trade.