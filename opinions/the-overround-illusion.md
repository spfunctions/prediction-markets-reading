# The Overround Illusion: Why EE > 0 Isn't Free Money

> A new prediction-market trader sees Event Overround at +0.06 and thinks they have found a 6% riskless arbitrage. Almost never. Fees, slippage, and the moving target of the orderbook eat the apparent edge before the order gets posted.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 12 min

---
The first time I saw Event Overround at +0.06 on a Polymarket primary contract, I almost put on every YES leg in the family at once. The math was clean. Six cents of overround means six cents of guaranteed profit per dollar of total notional. I had been waiting for a riskless trade my whole prediction-market career. This was it.

It was not it. It was never going to be it. I am writing this so that the next person who sees EE at +0.06 saves themselves the four hours and the $180 in fees I spent learning the lesson directly. The thesis: raw Event Overround almost never overcomes execution costs. The interesting use of EE is the *change* in EE over time, not the level. Stop trading the level.

## The Math, and the Two Things It Hides

Event Overround is the sum of YES prices across the mutually exclusive outcomes of an event, minus 1: `EE = Σ pᵢ − 1`. If you have a six-outcome primary and the YES prices add up to 1.06, the overround is +0.06, and in theory you can sell YES on every outcome, hold to resolution, and pocket 6 cents per dollar of notional regardless of which outcome wins.

The two words doing the hiding are "in theory." What the formula does not capture:

- The fee schedule on the venue
- The slippage you pay walking each outcome's orderbook
- The bid-ask spread on each individual outcome
- The fact that the orderbook moves when you start posting size

Every one of these costs is small in isolation. The problem is that they all stack against you on the same trade, and the stack almost always exceeds the apparent overround.

## The Worked Example

Polymarket multi-outcome event from January: "Who will win the 2028 Democratic nomination?" Six outcomes plus an "Other" basket. The mid prices on the displayed YES side:

| Outcome | YES Mid | Best Ask | Ask Depth |
|---|:---:|:---:|:---:|
| Newsom | 0.32 | 0.34 | $1,200 |
| Harris | 0.18 | 0.20 | $900 |
| Buttigieg | 0.12 | 0.14 | $600 |
| AOC | 0.08 | 0.095 | $400 |
| Whitmer | 0.06 | 0.075 | $300 |
| Other | 0.30 | 0.34 | $800 |

Sum of mids: 1.06. Apparent overround: +0.06. Apparent free money: 6%.

I would have to *sell* YES on every outcome to harvest the overround, which on Polymarket means buying NO on every outcome (the venue does not have a true short-YES primitive — you buy the complementary side). To buy NO at the price I want, I need to hit each outcome's bid, which is on the *other* side of the spread from the displayed mid.

Recompute using the actual bids I would have to lift:

| Outcome | NO Best Bid | NO Bid Depth |
|---|:---:|:---:|
| Newsom | 0.66 | $900 |
| Harris | 0.80 | $700 |
| Buttigieg | 0.86 | $500 |
| AOC | 0.905 | $300 |
| Whitmer | 0.925 | $250 |
| Other | 0.66 | $700 |

Sum of NO bids: 4.81. The cost to buy NO on all six outcomes is $4.81 per unit notional. At resolution, exactly five of the six NO contracts pay $1.00 each (because exactly one outcome wins YES), so I receive $5.00 per unit. Gross profit: $0.19. Per dollar of notional spent: $0.19 / $4.81 = 3.95%.

So the displayed overround was +0.06 = 6%. The actual gross arbitrage available, before any other costs, was 3.95%. The 2 cents of difference is the bid-ask spread of the displayed YES mid against the executable NO bid, on every leg, summed.

Now subtract the costs the formula did not include.

Polymarket fee on each leg: roughly 2% of notional on each fill, depending on tier. Six legs at 2% each, applied to the position notional, takes the gross 3.95% down to roughly 1.9% if the fees don't compound, less if they do.

Slippage walking the book on the larger legs: for a $300 position spread across all six outcomes, I am consuming a meaningful fraction of the displayed depth on the smaller outcomes (Whitmer, AOC). The next price level on Whitmer NO is 0.93, which is 0.5 cents worse. On AOC the next level is 0.91, also 0.5 cents worse. Call this another 0.3% of total notional.

Time risk while filling: by the time I have filled five of the six legs, the sixth leg's orderbook has reacted. Some other arbitrageur or market maker has noticed the imbalance and pulled the favorable price. In my actual experience, completing a six-leg arbitrage on Polymarket takes 15-90 seconds, and the sixth leg is repriced about 60% of the time. Call this another 0.4-0.8% of notional in expected value.

After all costs: 1.9% − 0.3% − 0.6% ≈ 1.0% gross arbitrage on a position you held to resolution, possibly for months. Annualized, depending on τ, this is barely above a treasury bill — and the treasury bill does not have the operational risk of a six-leg multi-venue execution.

The 6% screen number was a lie that hid 5% of execution costs. The actual edge was rounding-error.

## Why the Number Looks Bigger Than It Is

The Event Overround formula is computed against the *displayed mid*, which is the average of bid and ask. The displayed mid is rarely a price you can transact at. In a tight book it is close. In the wide books that characterize most multi-outcome Polymarket events, it is several cents away from any actual trade you can put on.

This is the same problem as Implied Yield computed at mid versus IY computed at the side you can actually fill. The difference is that on a single-contract IY trade, you only pay the bid-ask once. On a six-outcome arbitrage, you pay it six times, on the wrong side of the book each time, because the *direction* of the trade requires you to lift the unfavorable side of every leg simultaneously.

The math compounds against you. Six cents of apparent overround at the mid becomes 1-2 cents of executable overround after spread, becomes essentially zero after fees and slippage. The screen number and the trade number are simply not the same number, and the formula does not warn you about the difference.

## The Useful Thing EE Is Actually For

Raw EE level is a trap. EE *change* over time is a signal.

Here is the version that pays. Pull the EE on a multi-outcome event every hour. Plot the time series. Look for steps. A clean event sits at EE ≈ 0 most of the time. When something happens to one of the outcomes — a candidate drops out, an event resolves earlier than expected, news arrives — the prices adjust, but they do not always adjust *coherently*. For a few hours, sometimes a few days, the sum is off. EE shifts.

The information in the shift is not "there is now arbitrage." The information is "the market is in the middle of a price discovery event on this family." That is a *very* different signal, and you can trade it differently. Instead of harvesting the overround as arbitrage, you trade the *individual leg* whose price is most stale. You take a directional view on the leg the market hasn't repriced yet, and the EE signal is what told you the family was active.

This is the actual use of EE that has paid off for me. It is closer to how Cliff Risk Index works (a measure of activity, not direction) than to how naive arbitrage thinking would frame it. EE going from 0.00 to +0.04 is the family announcing "we are figuring something out right now." The trade is to identify which leg is the laggard and put on a directional position.

## The Counterargument

The strongest pushback is from people who run real multi-leg arbitrage on real venues with sub-cent edge. They will tell me, correctly, that institutional cross-venue arbitrage on prediction markets *is* a real strategy and the overround formula is the right starting point.

I am not arguing with that. I am arguing with the framing that the *displayed* overround on a single venue is a free money signal. Real arbitrage shops handle the costs I described above by:

- Trading on tier-zero fee schedules
- Using market-maker quoting, not lifting the book
- Holding positions across both Polymarket and Kalshi to exploit cross-venue mispricing where the spreads on individual legs are tighter
- Running in size sufficient to amortize the operational cost

If you have all four of those, the overround math becomes tradable in the limit. If you do not have all four — and almost no individual trader does — the overround is theater.

The other counterargument is "just trade smaller and the slippage goes away." Half right. If you trade a $20 position you mostly avoid the depth-eating problem, but you also amplify the fixed-fee component of the cost structure, and on Polymarket the per-trade fee floor is enough that small trades are *worse* per-dollar than medium trades. There is no trade size where naive overround harvesting on a multi-leg event is a clean win for retail.

## The Habit

The habit is to keep EE on the screen but stop staring at the level. Track the *delta*. When EE moves more than 0.02 in a 12-hour window on an event family that normally sits at zero, mark the event as active and look at the *individual legs*, not at the basket. The trade is in the leg. The overround was just the early-warning indicator.

I run `sf scan --by-overround --delta 12h` once a morning. The list usually has three to seven names on it. Of those, maybe one is a real trade. The other six are noise — small price moves on illiquid outcomes that pushed the sum off zero by a cent or two. The signal-to-noise on EE delta is much, much higher than on EE level, and it is the only way I have found to extract usable information from the indicator.

If you remember one thing from this essay: the displayed overround number is not the trade. The displayed overround is the headline of an article. The trade is in the body of the article, which is the individual leg whose price stopped being consistent with its siblings. Read the body, not the headline.