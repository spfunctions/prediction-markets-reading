# Tail-of-Day Pin Risk: Why Daily-Settled Contracts Move at 3:55 PM ET

> Daily-settled price-target binaries see a violent shift in the final 30 minutes of the trading day. Makers cannot hedge a binary that is pinning to {0,1} in real time, so the orderbook becomes a one-sided liquidation. For sharp traders, the close is the most reliable maker opportunity on the platform.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
Some Kalshi contracts settle daily, not at the end of an event window measured in weeks. The KXBTCD-style price-target binaries (Bitcoin closing above some level by 4:00 PM ET) and the equivalent S&P, Nasdaq, and FX contracts all share this property: every weekday, the contract resolves at 4:00 PM ET against a tape print, and a new contract for the following day lists at the same time. The settlement is fast, mechanical, and governed by a single-second observation.

That structure does something to the final 30 minutes that does not happen on weekly or event-driven contracts. The microstructure inverts. Makers stop being makers. Spreads blow out. Volume spikes asymmetrically toward the side closer to the settlement print. And the price action between 3:55 PM ET and 4:00 PM ET is consistently the most violent five minutes in the entire trading day, even when the underlying tape is barely moving.

I trade these contracts for the close. They are one of two or three places on the prediction-market landscape where a sharp human providing liquidity in the final ten minutes can earn a real edge against everyone else who is panicking out of positions. What follows is what to look for and why the structure produces the opportunity.

## The Empirical Pattern

Take a representative weekday for a KXBTCD-style "BTC > $X by close" binary. At 2:00 PM ET the contract is trading at, call it, 0.62 with a 1.5¢ spread and orderbook depth of about 1,800 contracts on each side. Volume so far on the day is around 14,000 contracts. The microstructure is normal — this looks like any other liquid binary.

At 3:00 PM ET, with one hour to settlement, the spread has widened slightly to 2¢ but the depth has stayed roughly the same. Volume is now 19,000. Still normal.

At 3:45 PM ET, with fifteen minutes to settlement, something starts to change. The bid depth has dropped to about 900 contracts but the ask depth has fallen to 400. The spread has widened to 3.5¢. Volume has lifted to 26,000 — most of the new trades have hit on the YES side, suggesting flow is leaning into the "BTC will close above $X" outcome.

At 3:55 PM ET, with five minutes to settlement, the orderbook is essentially gone on the side that is losing. If BTC has drifted higher and the contract is now at 0.74, the NO side has 80 contracts of depth and the YES side has 1,400. The spread is 6¢. Volume in the final five minutes will end up being 8,000 contracts, all of which fill on the thin side and chase the price to either 0.85 or 0.55 within the final two minutes.

At 4:00 PM ET the contract resolves to 0 or 1 against the official close print. The trades that filled at 0.85 in the final minute either gave up everything or doubled. There is no in-between.

## Why Makers Can't Stay In

A market maker on a continuous-strike binary like this is providing two-sided liquidity in exchange for capturing the spread. The math works as long as the inventory risk is bounded — if you accumulate 200 contracts of YES at $0.62, you can hedge by selling a small futures position, and the linear delta of the contract roughly tracks the linear delta of the underlying.

Inside the final minutes, this stops working. The contract is no longer linear in the underlying — it is a binary that will resolve to 0 or 1 in seconds. The "delta" of a binary contract that is going to settle in 90 seconds with the underlying near the strike is undefined in any practical sense, because the realized P&L is bimodal. A maker who is short 200 contracts of YES on a price-target binary that is 1¢ above the strike with 60 seconds to go has no hedge. There is no futures position that protects them. Their entire short position will either expire worthless (good for them) or settle at $1.00 (a $200 loss). There is no in-between, and no way to lock in the current price exposure.

The rational maker response is to stop quoting. Pull all bids on the side they are short, pull all asks on the side they are long, leave whatever inventory they have to its fate. The orderbook empties out on whichever side the underlying is moving against. The remaining traders are takers — and now the spread is whatever the few remaining makers feel like charging, which is large.

The asymmetry is the load-bearing detail. Makers do not pull *both* sides equally. They pull the losing side first, because that is where the adverse-selection risk is concentrated. The winning side stays at least partially populated by retail flow chasing the existing move. The result is a one-sided book where the offer is large and the bid is empty, or vice versa, and trades fill at increasingly extreme prices as the few remaining contracts get hit.

## Three Implications for Traders

**1. Do not hold positions through the close unless you have already won.** If you are long YES at 0.62 and the contract is now at 0.74 with five minutes to settlement, the math says you should let it ride — but the orderbook says you cannot exit at 0.74 because the bid is 0.65 and the depth is 80 contracts. The realized exit price for a meaningful position is several cents below the displayed mid. The general rule: if you would not be willing to take the contract to zero or par, exit by 3:30 PM ET. The final 30 minutes are not a "trading window" — they are a settlement window that you have to be all-in or all-out on.

**2. Provide liquidity only on the side that pays you to be there.** The opportunity for sharp traders is providing maker quotes in the final ten minutes, but only on the side where the structural maker withdrawal has created an unjustified spread. If the contract is at 0.74 and the bid depth is 80 contracts at 0.68, you can quote a bid at 0.71 and immediately become the top of the book. You will get filled by retail panic-sellers and arbitrageurs unwinding the cross-venue spread. Your edge is the 3-4¢ between your bid and the eventual settlement, scaled by the probability the contract actually pins above the strike — which, since you are the marginal liquidity, you can compute from the underlying tape directly.

**3. The 3:55-4:00 PM window is the highest-quality maker opportunity on the platform.** Most maker strategies on prediction markets struggle because the warm-regime cron only covers the top 500 contracts by 24-hour volume, leaving 99% of the universe with [LAS = null](/learn/liquidity-availability-score). The daily-settled contracts at the close are the exception: they are always in the top 500, the orderbook is always live, and the structural maker withdrawal in the final minutes guarantees there is real spread to capture. I run a small maker strategy on these every weekday and it is the one part of my book that has positive carry without active thesis work.

I want to be clear about what "real money-making opportunity" means here. The strategy is not free. You have to be at the screen, you have to have intuition about the underlying tape (you cannot just blind-quote), and you have to size small enough that a single bad pin does not blow up your week. But the structural reason the spread is there — makers cannot hedge a binary in its final minutes — is not going to go away. As long as Kalshi runs daily-settled price-target binaries, the close will pay sharp humans a few basis points to take the inventory risk that sophisticated makers will not.

## Where the Pin Risk Reads Differently

**Contracts with very wide strikes.** A "BTC > $20,000 by close" binary when BTC is at $74,000 will not have a pin window, because the contract is already at 0.99 and there is nothing for makers to withdraw from. The pin window is concentrated on contracts that are within 1-2% of their strike at 3:00 PM ET. Out-of-money strikes settle quietly.

**Contracts with thin all-day liquidity.** Some daily strikes are listed but get almost no volume during the trading day. These do not have a pin window because they never had a populated orderbook to withdraw from. They settle on a few stale orders and the price action looks like a dotted line all day. The pin pattern requires a contract that was actively quoted and is now being abandoned — not one that was never populated to begin with.

**News-driven pins.** If a Fed announcement or BLS print drops at 3:30 PM ET, the entire pin window dynamic gets overwritten by the news event. The orderbook withdrawal is no longer driven by the binary's internal mechanics — it is driven by uncertainty about what the news means for the underlying. These pins look superficially like normal pins but the maker provision strategy described above does not work on them, because the inventory risk on the *post-news* underlying is also unbounded.

A fourth caveat worth mentioning: not every venue runs daily-settled binaries the same way. Polymarket's prediction-market analogs to these are usually weekly or event-driven, not daily. The pin-risk dynamics described here are mostly a Kalshi phenomenon. If you are trading the Polymarket equivalents, the timing and the magnitudes are different and you should not import the 3:55 PM ET intuition wholesale.

## The sf CLI Read

The simplest way to spot the pin window forming is to track the spread of a daily-settled contract minute-by-minute in the final hour. `sf scan --tickers <KXBTCD-ticker> --warm --interval 1m` will give you the live snapshots, and the moment you see the spread double from its 1-hour-trailing average, you are in the pin window. From there, the side with the thinner depth is the one to provide liquidity on, sized to whatever your risk tolerance for a wrong-side pin is.

For the broader pattern, see the [settlement-halo](/concepts/settlement-halo) page — the daily pin is essentially a 30-minute version of the 24-hour halo, with the same maker-withdrawal mechanism but a much faster timescale. The [adverse-selection-on-prediction-markets](/concepts/adverse-selection-on-prediction-markets) page covers the related question of *who* you are trading against when you provide liquidity in these windows. They are usually retail panic-sellers and a few arbitrageurs unwinding the cross-venue book — exactly the counterparties you want.

The pin window is the most reliable structural edge I have found on Kalshi. It is not glamorous, it requires being at the screen at the close, and it caps out at a few thousand dollars a week of edge before you start affecting the market you are trying to make. But it is one of the few maker strategies that does not require warm-cron coverage, does not require a thesis, and pays you for absorbing inventory risk that the sophisticated makers explicitly cannot take. That is a structural edge worth showing up for.