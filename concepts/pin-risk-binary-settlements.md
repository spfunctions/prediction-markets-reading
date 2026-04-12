# Pin Risk in Binary Settlements: When 0.50 Becomes 0.00 or 1.00

> A binary contract sitting at 50¢ at the moment of resolution settles to either $1.00 or $0.00 with no gradient in between. That snap is real dollar risk, and it dominates the final hours of every active position you are still holding.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
Every binary prediction-market contract eventually pins. At the moment of resolution it stops being a continuous price and becomes a coin flip with a known answer. The most uncomfortable contracts are the ones that approach that moment near the middle of the [0, 1] interval, because the final move is asymmetric: 50¢ goes to either $1.00 or $0.00, and the realized outcome is fully determined by an event you cannot trade against any more.

That asymmetric snap is pin risk, and it is the single most underappreciated cost in active prediction-market trading. This page is the version of the warning I wish someone had given me before my first Fed-decision contract pinned.

## What Pin Risk Actually Is

A standard options trader uses the term "pin risk" in a specific way: it is the risk that an underlying stock closes exactly at the strike price of an option you are short, leaving you uncertain whether to exercise. The prediction-market version is similar in spirit but mathematically simpler. There is no underlying. There is only the binary outcome and the price you are holding at the moment of that outcome.

Concretely: you hold 100 contracts at 50¢ on a Kalshi binary that resolves in 30 minutes. Your position has a cost basis of $50. The market still shows a mid of around 50¢ because nobody has hit a different price recently. In thirty minutes, one of two things happens:

- The contract resolves YES → you receive $100. Your P&L is +$50.
- The contract resolves NO → you receive $0. Your P&L is −$50.

The expected value, given a fair 50/50 prior, is exactly zero. But the *variance* on that expected value is enormous. A position that looks like "$50 of capital tied up at the mid" is actually "$50 of capital that becomes $100 or $0 in the next half hour with no opportunity to bail out."

That is pin risk in one paragraph: the displayed market value is a misleading summary of the *realized* outcome distribution, because the realized outcomes are at the corners of the distribution, not at the mean.

## Why It Dominates the Final Hours

Pin risk grows as the resolution event approaches, for two related reasons.

**First, the time available to exit shrinks.** A position you held for 18 days had 18 days during which the price could move favorably and let you take profit at any point. A position you hold for the final 30 minutes has 30 minutes — and during those 30 minutes, the orderbook is usually thinning out as makers withdraw (see the [maker-taker-regime-in-pms](/concepts/maker-taker-regime-in-pms) page for why). The exit liquidity disappears exactly when you need it most.

**Second, the information you do not have becomes the only thing that matters.** Earlier in the contract's life, your thesis (or the indicator stack, or the news flow) gave you information that mattered. In the final hour, all of those become tied with the actual outcome event. Your edge over the market collapses to zero, because the market is staring at the same final moment you are. Whatever you knew that gave you an edge is now common knowledge or about to be.

The combined effect is that the final hour or two of any binary contract is the worst possible time to be holding it for thesis reasons. You have no edge, you have no exit liquidity, and the realized variance is at its maximum. The math says: close the position, accept whatever P&L you have, and let the contract pin without your money in it.

## A Worked Example

KXFEDDECISION-26MAY01 — "Will the Fed cut rates at the May meeting" YES contract. I bought 200 contracts at 38¢ on April 12th, with the FOMC meeting scheduled for May 14th. My cost basis is $76. The thesis was that the labor market data over April would push the consensus toward a cut.

April 14th: contract drifts up to 42¢. Mark-to-market P&L is +$8.
April 22nd: contract jumps to 51¢ on a soft CPI print. Mark-to-market P&L is +$26.
May 1st: contract sits at 49¢ for ten days as the market waits for the meeting. Mark-to-market P&L is +$22.
May 14th, 12:00 PM: contract is at 50¢. Meeting decision drops at 2:00 PM. Mark-to-market P&L is +$24.

Now I have a decision. I have a $24 mark-to-market gain on a $76 position. The contract is at 50¢ — exactly the worst possible level for pin risk. In two hours, the contract will resolve to either $1.00 (giving me a realized gain of $124) or $0.00 (giving me a realized loss of $76). The expected value is around the mid, but the *variance* is catastrophic.

The right move at 12:00 PM is to close. I can sell my 200 contracts at the bid, which is 49¢ at the moment, and lock in a $22 realized gain. I lose 1¢ to the spread. I forgo the $100 upside and the $74 downside. The remaining $98 of variance gets transferred to whoever crosses my offer.

The wrong move is to hold. The math on holding looks like "I have a 50% conviction the contract resolves YES, so my expected P&L is the same either way." That math is correct in expectation. But it ignores the variance, and the variance is what kills you over a sequence of pinned positions. If I run the same play on twenty FOMC contracts a year, the realized P&L distribution is much wider than the mean implies, and a few catastrophic losses will dominate the year.

Pin risk is the variance you cannot see in the mark-to-market number. The only defense is to close.

## The Math That Justifies Closing

The decision rule is simple. Define:

```
realized_outcome ∈ {0, 1}
position_size = N (number of contracts)
current_mid = p
true_probability = q (your subjective probability of YES)
```

If you close at the mid:

```
guaranteed_PnL = N × (p − cost_basis)
```

If you hold to resolution:

```
E[PnL] = N × (q − cost_basis)
Var[PnL] = N² × q × (1 − q)
```

The closing trade gives you a *certain* P&L equal to the mark-to-market. The holding trade gives you an expected P&L that is higher only if your subjective q is significantly different from p (which it usually is not in the final hour, because you have no new information). And the holding trade gives you variance proportional to N² × q × (1 − q), which peaks at q = 0.5.

The variance term is the killer. For a position at 50¢, variance per contract is at its maximum. For a position at 90¢ or 10¢, variance per contract is much lower because q × (1 − q) shrinks. Pin risk is highest at 50¢ and lowest at the corners. That is why the rule is "close the contracts that pinned near the middle, hold the contracts that pinned near a corner if you want to."

I close any binary that is within 10¢ of 50¢ in the final 6 hours, unless my conviction is so high that q is closer to 1 than the market thinks. That happens maybe one in twenty trades, and even then I usually close half the position to cap the variance.

## Why "Don't Hold to Settlement" Is Not Universal

This is not financial advice and there are real cases where holding to settlement is the right move.

**You have inside information.** If you genuinely know which way the contract is going to resolve and the market does not, the correct move is to hold (and to add to the position, depending on the size of your edge). This is rare and the people who think they have inside information usually do not. Be honest with yourself before assuming this case applies.

**The contract is at a corner already.** A contract pinning at 92¢ has small pin risk because the variance is small (0.92 × 0.08 = 0.074, vs 0.5 × 0.5 = 0.25). You can hold it through resolution if your cost basis was much lower, because the realized outcome is unlikely to be far from the mid.

**The position is small relative to your portfolio.** Pin risk hurts when the position is large enough that the realized variance moves your portfolio P&L meaningfully. If the position is 0.5% of your capital, the pin risk is annoying but not portfolio-altering. You can choose to hold for the small expected upside.

**You are trading the resolution itself.** Some traders explicitly take pin risk on the theory that the realized variance has thicker tails than the implied variance, and they get paid by selling that mispricing. This is a legitimate strategy but it requires a different calibration than thesis trading, and I do not recommend it for anyone who has not specifically built a model for it.

In the absence of one of those four conditions, the default is *close*.

## The Operational Rule

I have a hard cutoff: the position closes 6 hours before resolution unless I have a specific written reason to hold. The 6-hour window is not magic — it is just the point at which the orderbook reliably has enough liquidity for me to exit a normal-sized position cleanly, and after which liquidity starts to evaporate. You can pick a different cutoff for different position sizes; what matters is that you have one and you follow it.

The CLI command I run on the final cutoff every morning is `sf scan --my-positions --tau-max 0.25 --status open`. That gives me the list of contracts I am still holding that resolve in the next 6 hours. I either close them, or I write a one-line note explaining why I am holding (and the note has to invoke one of the four exceptions above; otherwise it does not count).

The discipline is the part that matters. The math is easy. The hard part is overriding the gut feeling that "the contract is at 50¢, my conviction is 50%, holding is neutral." It is not neutral. It is variance, and variance compounds over time in the wrong direction.

## Where the Pin-Risk Frame Breaks

Three failure modes.

**The market does not actually pin near 50¢.** Most binaries do not approach 50¢ at resolution; they drift toward 0 or 1 well before the event, because new information arrives and the price reflects it. Pin risk is concentrated on the *uncertain* contracts, not on all contracts. You only need the closing rule on the small fraction of your portfolio that is still at the middle of the interval in the final hours.

**Settlement timing is fuzzy.** Some contracts have a "resolution period" of hours or days during which the underlying event might or might not have occurred. The pin happens whenever the venue declares the resolution, which can be at an unpredictable time. The fix is to close earlier on contracts with fuzzy resolution timing. See [resolution-risk-premium](/concepts/resolution-risk-premium) for the related question of *why* the resolution timing matters at all.

**Your closing trade itself moves the market.** If you are holding a large position in a thin market, the act of closing eats the spread and can dump the price several cents against you. The defense is to size the position smaller to begin with (so the closing trade is fillable at the displayed spread), or to start unwinding earlier when the orderbook is thicker.

A fourth issue worth noting: the mark-to-market P&L number on the venue dashboard is not the realized P&L if you cannot fill at the mid. The dashboard lies, in the same way Bloomberg lies about a stale corporate bond. Always test the actual exit price by trying to send a small order at the mid before you trust the dashboard.

## How to Practice Closing

Pick one binary you currently hold that resolves in less than 24 hours. Close it now, before reading the next sentence. Watch what your gut does. Most of the time the gut says "but what if it pins my way" — and the gut is right that sometimes it will, and the gut is wrong that the upside justifies the variance.

The habit you want to build is closing first and feeling the regret afterward, not feeling the conviction first and closing too late. Once the closing reflex is the default, the math takes care of itself, and the realized P&L distribution on your binary book gets dramatically tighter.

For the related operational concept on regime, see [maker-taker-regime-in-pms](/concepts/maker-taker-regime-in-pms). For the question of why some contracts have *additional* risk on top of pin risk because the resolution rule itself is fuzzy, see [resolution-risk-premium](/concepts/resolution-risk-premium).