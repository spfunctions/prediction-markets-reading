# Prediction market liquidity: why depth matters more than volume for serious traders

> Volume tells you how many people showed up; depth tells you whether you can actually trade.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
I've been trading on Kalshi with real money for about two years. In that time I've made plenty of mistakes, but the most expensive lesson was about liquidity — specifically, the difference between what the market *says* its volume is and what actually happens when you try to move size through it.

This article is the guide I wish I had when I started. No marketing, no hype about "democratizing finance." Just how orderbooks actually work on prediction markets, and a framework for deciding whether a market is liquid enough to trade.

## Volume Is a Vanity Metric

When Kalshi or Polymarket promote a market, the headline number is always volume. "$50M traded on the presidential election!" "$2M volume on recession markets this week!"

Volume tells you one thing: people have traded this contract before. It tells you nothing about whether *you* can trade it *right now* at a reasonable price.

Here's why. A market can have $5M in lifetime volume but a current orderbook that looks like this:

```
--- KXINFLCPIY-26MAR-T3.5 (CPI YoY > 3.5%) ---
Bids                    Asks
10 contracts @ 32¢      5 contracts @ 41¢
25 contracts @ 28¢      15 contracts @ 45¢
50 contracts @ 22¢      30 contracts @ 52¢
```

That's a 9-cent spread on a contract worth maybe 35 cents. If you want to buy 50 contracts at market, you're paying a blended price of roughly 44 cents — nearly 30% above where the "last trade" happened. That $5M volume number didn't help you at all.

Conversely, a market with $200K in volume but tight spreads and deep resting orders might let you move 200 contracts with 1 cent of slippage. That's the market you actually want to trade.

## Three Concepts You Need: Depth, Spread, Slippage

Let me define these precisely in the context of prediction markets, because they work slightly differently than equities.

**Spread** is the gap between the best bid and best ask. On Kalshi, contracts trade between 1 and 99 cents. A 1-cent spread is tight. A 5-cent spread is wide. Anything above 10 cents is essentially untradable for active strategies.

The spread is your *minimum* cost of entering and exiting a position. If you buy at 52 and the spread is 5 cents, you need the contract to move at least 5 cents in your direction before you can exit at breakeven. On a contract trading at 50 cents, that's a 10% hurdle rate before you've even accounted for Kalshi's exchange fees.

**Depth** is the total quantity of resting orders at each price level. A market with 500 contracts on the bid at 50 cents has more depth than one with 10 contracts at the same price. Depth determines how much size you can move without pushing the price.

On prediction markets, depth is often shockingly thin. I regularly see Kalshi markets where the entire visible orderbook has maybe 200 contracts on each side. If you want to take a $5,000 position, that's 100 contracts at 50 cents — and you'll eat through three or four price levels to get filled.

**Slippage** is the difference between the price you expected and the price you actually got. It's the direct consequence of insufficient depth. If the best ask is 50 cents but you need 100 contracts and only 20 are available at 50, you're going to pay 51, 52, 53 for the rest. Your average fill might be 52 cents — 2 cents of slippage, or about 4% of your position value.

On a binary contract, 4% slippage is devastating. Your theoretical edge might only be 5-8%. Slippage eats it alive.

## How to Read a Kalshi Orderbook

Let me walk through a real example. Here's what an orderbook looks like for a WTI crude oil max price contract:

```
--- KXWTIMAX-26DEC31-T135 (WTI Max Price > $135 in 2026) ---

Asks (sellers)                          Bids (buyers)
  150 contracts @ 18¢                     80 contracts @ 12¢
   75 contracts @ 17¢                    120 contracts @ 11¢
   40 contracts @ 16¢                    200 contracts @ 10¢
   20 contracts @ 15¢                     50 contracts @  9¢
                                         100 contracts @  8¢

Spread: 3¢ (best bid 12¢, best ask 15¢)
Mid: 13.5¢
Depth (3 levels): 135 contracts ask-side, 400 contracts bid-side
Last trade: 13¢
24h volume: 342 contracts
```

What can we read from this?

First, the spread is 3 cents on a contract trading around 13 cents. That's a 23% round-trip cost if you cross the spread both ways. You do *not* want to be a market-taker on this contract.

Second, the depth is asymmetric — much more depth on the bid side than the ask side. This usually means there are patient buyers accumulating but sellers are thin. If you want to sell into this market (maybe you think oil won't hit $135), you can actually get reasonable fills. If you want to buy, you'll push the price up fast.

Third, the 24h volume of 342 contracts is modest. But the depth on the bid side (400 contracts in 3 levels) is actually decent. This is a case where depth is better than volume suggests.

Now compare that to a recession market:

```
--- KXRECSSNBER-26 (NBER Recession in 2026) ---

Asks (sellers)                          Bids (buyers)
  300 contracts @ 28¢                    250 contracts @ 25¢
  200 contracts @ 29¢                    200 contracts @ 24¢
  150 contracts @ 30¢                    300 contracts @ 23¢
  100 contracts @ 32¢                    150 contracts @ 22¢
  200 contracts @ 35¢                    400 contracts @ 20¢

Spread: 3¢ (best bid 25¢, best ask 28¢)
Mid: 26.5¢
Depth (3 levels): 650 contracts ask-side, 750 contracts bid-side
Last trade: 26¢
24h volume: 1,847 contracts
```

This is a much healthier book. The spread is 3 cents on a 26-cent contract (about 12% round-trip, still not great but workable). Depth is deep and relatively symmetric. You can move 200+ contracts on either side without significant slippage.

The key difference: the recession contract has both higher volume *and* better depth. But they don't always correlate.

## The Liquidity Score Framework

After getting burned enough times, I built a simple framework for scoring market liquidity before I enter any position. It's four grades:

### Grade A — Tradable at Size

- Spread: 1-2 cents
- Depth: 500+ contracts within 3 cents of mid on each side
- Slippage for 100 contracts: < 1 cent
- Example: Major political markets near election day, high-profile Kalshi flagship markets

These are the markets where you can actually run a strategy. You can enter and exit without the orderbook moving against you. Limit orders get filled in reasonable time. You can adjust positions intraday.

Grade A markets on Kalshi are rare. You find them on presidential elections, maybe major economic indicators during high-news periods, and sometimes on weather markets during hurricane season.

### Grade B — Tradable with Patience

- Spread: 2-4 cents
- Depth: 200-500 contracts within 3 cents of mid on each side
- Slippage for 100 contracts: 1-2 cents
- Example: KXRECSSNBER-26 during normal news flow

Most of the "liquid" prediction markets fall here. You can trade these, but you need to use limit orders almost exclusively. Crossing the spread is expensive. You should plan to hold positions for days or weeks, not hours.

My recession and oil positions mostly live in Grade B markets. I enter with limit orders, set them a penny inside the spread, and wait. Sometimes I wait hours. That's fine — the edge is big enough to justify the patience.

### Grade C — Tradable with Caution

- Spread: 4-8 cents
- Depth: 50-200 contracts within 3 cents of mid on each side
- Slippage for 100 contracts: 3-5 cents
- Example: Most mid-tier Kalshi markets, many Polymarket markets outside the top 20

These markets are tradable if your edge is large (>10%) and you're willing to hold to expiration. Don't plan on exiting early — you'll give back your edge in slippage. Only enter with limit orders. Size down. These are "conviction trade" markets where you have a strong thesis and you're willing to be patient.

### Grade D — Avoid

- Spread: 8+ cents
- Depth: < 50 contracts within 3 cents of mid
- Slippage for 100 contracts: 5+ cents
- Example: Long-tail Kalshi markets, obscure contracts, anything expiring in 2+ years with no news catalyst

I don't trade Grade D markets, period. The spread alone kills any edge you might have. These markets exist on Kalshi's website and they technically have prices, but they're decorative. You'll see "32 cents" and think that's tradable, but the actual bid is 20 and the ask is 40. The "price" is a fiction.

Surprisingly, some high-volume markets are Grade D. How? All the volume happened in one burst — maybe a news event drove a flurry of trades, but then the market makers pulled their orders and left. The volume stays on the ticker forever. The liquidity does not.

## Why High-Volume Markets Can Be Illiquid

This is the single most counterintuitive thing about prediction markets, and it trips up almost everyone who comes from equity trading.

In equities, high volume almost always correlates with tight spreads and deep books. Market makers are incentivized to provide liquidity in high-volume names because they earn the spread on more trades.

Prediction markets don't work this way. Here's why:

**No dedicated market makers.** Kalshi has some market-making programs, but they're thin compared to equity markets. Most liquidity comes from retail traders placing limit orders. When those traders get filled or lose interest, the liquidity disappears.

**Event-driven volume spikes.** A single news event can generate 80% of a market's lifetime volume in one day. The CPI release drops, everyone rushes to trade inflation contracts, volume spikes to 10,000 contracts. Next day? The book is back to 50 contracts on each side.

**Concentrated price levels.** Sometimes all the volume in a market has happened at one price. If CPI inflation has been trading at 35 cents for weeks with volume accumulating, that tells you people agree it's worth 35 cents. It doesn't tell you there's depth at 35. All those trades might have been 5 contracts here, 10 contracts there, with the book refilling slowly.

**Binary contract dynamics.** As a binary contract approaches expiration and the outcome becomes clearer, liquidity concentrates at the extremes (near 0 or near 100). The middle of the book empties out. A contract that was liquid at 50 cents three months ago might be illiquid at 15 cents today, even though volume has increased.

I learned this the hard way on a geopolitics market. Volume was over 5,000 contracts, which looked great. But when I tried to sell 150 contracts, I slipped 7 cents. The volume was all from one day when the news broke. By the time I was trading, the book was a ghost town.

## Using SimpleFunctions to Check Liquidity

I built tooling for this exact problem. The `sf` CLI can pull orderbook snapshots so you don't have to eyeball the Kalshi web UI, which honestly makes it hard to see depth.

Here's what it looks like:

```bash
$ sf book KXWTIMAX-26DEC31-T135

  KXWTIMAX-26DEC31-T135 — WTI Crude Max Price > $135 (2026)
  Mid: 13.5¢ | Spread: 3¢ | Liquidity Grade: C+

  ASKS                              BIDS
  150 @ 18¢  ████████              80 @ 12¢  ████
   75 @ 17¢  ████                 120 @ 11¢  ██████
   40 @ 16¢  ██                   200 @ 10¢  ██████████
   20 @ 15¢  █                     50 @  9¢  ██
                                  100 @  8¢  █████

  3-level depth: 135 ask / 400 bid
  Est. slippage (100 contracts): ~3.2¢ buy / ~1.4¢ sell

$ sf book KXRECSSNBER-26

  KXRECSSNBER-26 — NBER Recession Declaration 2026
  Mid: 26.5¢ | Spread: 3¢ | Liquidity Grade: B

  ASKS                              BIDS
  300 @ 28¢  ██████████           250 @ 25¢  ████████
  200 @ 29¢  ███████              200 @ 24¢  ███████
  150 @ 30¢  █████                300 @ 23¢  ██████████
  100 @ 32¢  ███                  150 @ 22¢  █████
  200 @ 35¢  ███████              400 @ 20¢  █████████████

  3-level depth: 650 ask / 750 bid
  Est. slippage (100 contracts): ~1.1¢ buy / ~0.9¢ sell
```

The visual bars make depth asymmetry immediately obvious. The liquidity grade saves me from having to do mental math every time. I check this before entering any position.

## Practical Rules I Follow

After two years of this, here's my operating playbook:

**1. Always check depth before you check price.** The price is meaningless if you can't trade at it. I've passed on contracts where I had a 15-cent theoretical edge because the slippage would eat 10 of those cents.

**2. Use limit orders exclusively on Grade B and below.** I know this means I sometimes don't get filled. That's fine. Missing a trade is free. Getting slipped is not.

**3. Size to the book, not to your conviction.** If the book only has 100 contracts within 2 cents of mid, I'm not putting in 300. I'll do 50-80 and see if the book refills. Sometimes I scale into a position over 2-3 days.

**4. Watch for depth replenishment patterns.** Some markets have "natural" depth that refills throughout the day — probably from market-making bots. Others fill up once in the morning and drain by afternoon. Trade the ones that refill.

**5. Don't trade the first hour after a news event.** This is when spreads blow out and depth disappears. Everyone is rushing to reprice and nobody wants to provide liquidity. Wait for the book to settle. The opportunity will still be there in an hour, usually at a better price.

**6. Monitor multiple time horizons for the same underlying.** Kalshi often has the same question at different expirations. The near-term contract might be Grade D while the far-term one is Grade B, or vice versa. Trade the one where you can actually get executed.

## The Bottom Line

Volume is what prediction markets advertise. Depth is what actually determines whether you can trade profitably. Every serious trader I know has learned this the hard way — by getting slipped on a position they thought was liquid.

The fix is simple: look at the book before you trade. Score the liquidity. Size appropriately. Use limits. Be patient.

Prediction markets are still early. Liquidity will improve as more participants enter and market-making infrastructure matures. But right now, in 2026, the orderbooks are thin enough that understanding depth is the difference between making money and donating it to the spread.

If you want to check orderbook depth and liquidity grades programmatically, I built [SimpleFunctions](https://simplefunctions.dev) for exactly this. It pulls real-time book data from Kalshi and scores markets so you can focus on the ones that are actually tradable.