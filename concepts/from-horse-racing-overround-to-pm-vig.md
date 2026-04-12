# From Horse Racing Overround to Prediction Market Vig

> Sports betting has priced overround for two centuries. The math ports to prediction markets directly. The venue economics do not, and the difference between a sportsbook’s 4.5% on price and Kalshi’s 7% on profit is bigger than it sounds.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 12 min

---
Sports bettors have been doing the same calculation for two hundred years, and they call it overround. Add up the implied probabilities from a bookmaker's odds, get a number greater than 1.0, the difference is the bookmaker's margin. The math is so basic that it is one of the first things any sharp bettor learns. It is also the most directly transferable piece of intuition from sports betting to prediction markets, and it is the one I most often see new PM traders fumble.

This page walks through what an overround actually is in horse racing, how the math ports to prediction markets, where the venue economics diverge, and what a sports bettor should know before they assume their overround intuition will save them on Kalshi or Polymarket.

## What Overround Is in Horse Racing

A traditional horse race has N horses. The bookmaker offers odds on each horse — say 5/1 on Horse A, 3/1 on Horse B, 7/2 on Horse C, and so on. Implied probability for each horse is `1 / (odds + 1)` (using the fractional odds convention). For Horse A at 5/1, the implied probability is `1/6 ≈ 0.167`. For Horse B at 3/1, it is `1/4 = 0.25`. And so on.

If you sum the implied probabilities across all horses in the race, you should get 1.0 in a fair market. In practice you get something like 1.10 to 1.20, depending on the bookmaker. The excess — the 10% to 20% — is the overround, and it is the bookmaker's expected margin. Spread across a balanced book, the overround is the bookmaker's profit in expectation.

The formal definition is:

```
Overround = Σᵢ (1 / (decimal_oddsᵢ)) - 1
```

For decimal odds. Or equivalently in terms of implied probabilities:

```
Overround = Σᵢ pᵢ - 1
```

That second form is identical to the [event-overround](/concepts/event-overround) formula on prediction markets. *The math is literally the same.*

## How the Math Ports

A multi-outcome prediction market is structurally identical to a horse race. Take a Kalshi or Polymarket event with K mutually exclusive outcomes. The YES price on each outcome, expressed as a probability, is `pᵢ`. The sum of all `pᵢ` should be 1.0 for a fair market. The deviation is the overround, and the formula is the same one a horse-racing bettor has been using since the 1800s.

The "Who wins the 2028 Democratic nomination?" Polymarket event from earlier in this site:

| Outcome | YES Price |
|---|:---:|
| Newsom | 0.32 |
| Harris | 0.18 |
| Buttigieg | 0.12 |
| AOC | 0.08 |
| Whitmer | 0.06 |
| Other | 0.30 |

Sum = 1.06. Overround = +0.06. In horse-racing language, this is a market with a 6% overround on the field. A sharp horse bettor reading this as a horse race would say: "the bookmaker is getting 6% margin; if I want to make money I need at least 6 cents of edge per dollar to break even before fees."

That is exactly the right intuition. The math ports directly. If you can compute overround on a horse race, you can compute it on a Kalshi multi-outcome event using the same formula and the same units. The output — a small positive number representing the structural margin — means the same thing in both venues.

## Where the Venue Economics Diverge

This is the part sports bettors get wrong. The math ports, but the *venue charges the overround differently*, and the difference is bigger than it looks.

A traditional sportsbook takes the overround out of the *price you pay*. When you bet $100 on Horse A at 5/1, you receive $500 in winnings (plus your $100 stake back) if Horse A wins. The bookmaker has built their margin into the odds themselves — the "true" probability of Horse A might be 18% but they price it as 16.7% (5/1). You are paying the margin every time you bet, on the price.

A prediction market like Kalshi takes a different approach. Kalshi charges a *fee on profit*, not on price. The fee structure (as of 2026) is roughly 7% on net profit at settlement, with no fee on losing trades and no fee on the price itself. So if you buy YES at $0.42 and the contract resolves YES, you receive the full $1.00 in payoff, then Kalshi takes 7% of your $0.58 profit — which is about 4.1¢. Your net is $0.96 instead of $1.00.

Polymarket is different again. Polymarket has historically had near-zero fees on trade execution but has the AMM-implied slippage cost (which scales with position size and orderbook depth) and the resolution risk premium (the implicit discount for UMA dispute risk). The "fee" you pay on Polymarket is mostly invisible until you hit a large position, and then it suddenly becomes the dominant cost.

## Translating Sportsbook Vig to Prediction-Market Fees

A sports bettor used to a 4.5% bookmaker margin needs to translate that into the prediction-market world by asking: what would an equivalent total cost be on Kalshi or Polymarket for the same trade?

The naive answer is "Kalshi takes 7% on profit so it is worse." The actual answer is more complicated because the *base* the percentage is taken on is different.

Consider a binary trade where you buy YES at $0.50 and your subjective fair value is $0.55 (a 5¢ edge). You expect to win 55% of the time.

- *Sportsbook (4.5% on price)*: you bet $50 to win $50 in profit. Your effective expected value per $50 risked is something like `(0.55 × 50) - (0.45 × 50) - 0.045 50 = 5 - 2.25 = $2.75`. Sharper bettors would express this differently but the rough math holds.
- *Kalshi (7% on profit)*: you buy 100 contracts at $0.50, paying $50. If you win, you get $100 - $7 fee = $93. Net profit = $43 (instead of $50). EV = `(0.55 × 43) - (0.45 × 50) = 23.65 - 22.5 = $1.15`. Significantly less than the sportsbook EV.
- *Polymarket (near-zero fee, 5¢ of slippage on a $50 size)*: you pay $0.55 effective entry instead of $0.50, get $100 if you win. EV = `(0.55 × 45) - (0.45 × 55) = 24.75 - 24.75 = $0.00`. The slippage exactly ate the edge.

Three different venue structures, three different effective costs, on the same underlying trade. The sportsbook with the higher headline overround actually delivers the best expected value here because it taxes the price (which compresses the spread between bet and payoff) rather than the profit (which compresses the asymmetry between win and loss).

This is the part that surprises sports bettors. "Kalshi takes 7% on profit" sounds tolerable until you do the EV math and realize that taxing profit is a much more aggressive form of vig than taxing price, *for any trade with positive expected return*. The 7% on profit only feels light if you are a 50/50 bettor, where the wins and losses cancel and the fee is a small fraction of nothing.

## What Carries Over: The Workflow

The actual workflow of a sharp horse bettor ports almost completely to a sharp prediction-market trader. The steps are:

1. Compute the implied probabilities of every outcome.
2. Sum them, get the overround, identify the structural margin.
3. Identify outcomes where your subjective probability is higher than the implied probability by more than the per-outcome share of the overround.
4. Size positions based on Kelly or fractional Kelly, accounting for the overround as a fixed cost.
5. Track *closing line value* (more on this in [from-sports-betting-clv-to-pm-trade-quality](/concepts/from-sports-betting-clv-to-pm-trade-quality)) to verify your edge is real over time.

Steps 1–3 and step 4 port directly. Step 5 needs adaptation because PMs do not have a "closing line" in the same sense (the price at resolution snaps to 0 or 1, which is not a meaningful "closing line" for measuring your entry quality).

The thinking is identical. The arithmetic is identical. The mental model of "I am paying a structural margin to the venue, and my edge has to overcome that margin before I make money" is identical.

## What Does Not Carry Over: The Liquidity Profile

A horse race lasts five minutes and the entire orderbook clears at the moment of the race. There is no "stale price" problem because the bookmaker is the counterparty to every trade and they cannot get stuck holding a losing position they cannot exit — they just balance the book.

A prediction market is a continuous limit order book (or AMM, in Polymarket's case) that lives for weeks or months, and the orderbook is full of people who could leave at any moment. If a sports bettor is used to "the bookmaker is always there to take the other side of my trade," they are in for a surprise on PMs, where the orderbook can be 12 cents wide and 50 cents deep — meaning there is no realistic way to put on the position in size without moving the price five percent against yourself.

This is what the [liquidity-availability-score](/learn/liquidity-availability-score) is measuring, and it is the indicator that has no horse-racing analog. A horse-racing bettor never had to think about depth because the bookmaker provided unlimited depth at the posted odds (within the limits of their book). PMs do not have that property and the missing intuition is one of the most expensive things sports bettors learn the hard way.

## What Does Not Carry Over: Resolution Risk

Horse races resolve when a horse crosses a finish line. The judging is contested only in extreme cases (photo finishes, disqualifications) and the resolution lag is minutes. Prediction markets resolve through processes that range from "Kalshi reads a Reuters headline" (low risk) to "UMA voters dispute the meaning of a press release" (high risk). The resolution risk on PMs is real and it has no horse-racing analog.

The translation is: a sharp horse bettor's intuition about "I will get paid if my horse wins" needs to be replaced, on PMs, with "I will get paid if the contract resolves correctly *and* the venue resolves it correctly *and* no UMA dispute changes the outcome." For Kalshi, the second clause is essentially always satisfied. For Polymarket, the third clause is real and you should price it into your edge calculation.

## What Carries Over Beautifully: Closing Line Dynamics

Sharp sports bettors track closing line value (CLV) religiously — the difference between the price you bet at and the price the market closes at. CLV is the gold standard for measuring whether your edge is real, because it is *independent of whether you won or lost any specific bet*. If you consistently get prices that are better than the closing line, you are sharp. If you consistently get prices that are worse than the closing line, you are not, and no amount of variance will save you over time.

This intuition ports directly to PMs, with one adjustment. PMs do not have a "closing line" because the price at resolution is just 0 or 1. The PM-native equivalent is "where did the price move at fixed intervals after I entered" — your entry at $0.42, the price one hour later at $0.43, the price 24 hours later at $0.45, the price 7 days later at $0.50. If your entries are systematically below the 24-hour-later price, you are reading the market faster than other traders, and you have something resembling CLV.

The full translation is in [from-sports-betting-clv-to-pm-trade-quality](/concepts/from-sports-betting-clv-to-pm-trade-quality). For now: a sports bettor's CLV intuition is 80% transferable, and it is the single best discipline you can bring with you from sportsbooks.

## The Honest Summary

A sports bettor's overround intuition is roughly 80% transferable to prediction markets. The math is identical. The thinking is identical. The workflow is identical. What changes is the venue economics: Kalshi's 7%-on-profit is a different shape than a sportsbook's 4.5%-on-price, and the difference is meaningful for any trade with positive expected return. Polymarket's slippage-and-resolution-risk model is different again, and a sports bettor needs to learn to price both.

If you came from sportsbooks, keep your overround math, your Kelly sizing, your CLV discipline, and your "I am paying for the venue's existence" mental frame. Update your fee math to account for profit-based vig instead of price-based vig. Add the [liquidity-availability-score](/learn/liquidity-availability-score) as a depth check that horse racing never required. Add resolution risk as a credit-risk overlay that horse racing never had. Run `sf scan --by-overround` to find the multi-outcome events with the highest structural margins, then look for outcomes within those events where your subjective probability disagrees with the implied probability by more than the share of overround you would be paying.

For the longer treatment of when raw overround is and is not actionable, see [the-overround-illusion](/opinions/the-overround-illusion) — the short version is that raw EE is rarely wide enough to overcome fees, and the interesting use is *changes* in EE over time. For the broader argument that PMs should be talked about in the language of fixed income rather than sports betting, see [/blog/prediction-markets-need-fixed-income-language](/blog/prediction-markets-need-fixed-income-language) and [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) — sports-betting vocabulary is fine for the multi-outcome arbitrage layer, but it does not extend cleanly to single-contract pricing the way fixed-income vocabulary does.