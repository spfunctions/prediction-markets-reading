# Implied Yield vs Raw Probability: Why Bond-Adjacent Prediction Markets Need a Different Lens

> Two Fed-decision contracts at the same mid-price can be wildly different trades. Raw probability hides the difference. Implied yield surfaces it in the unit fixed-income traders already use.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 7 min

---
Two Kalshi contracts cross my screen this morning. They are both about the next Fed decision. They have nearly identical mid prices. They also represent very different trades, and the way I know is that I learned to stop looking at the raw probability.

This is an essay about why fixed-income-adjacent prediction-market contracts need a different lens — and what changes once you build the habit.

## The Two Contracts

Both are Kalshi Fed-decision YES contracts. I have stripped the tickers down to the two attributes that matter:

| Contract | YES Price | τ (days) |
|---|:---:|:---:|
| A — Fed cut at next meeting | $0.42 | 18 |
| B — Fed cut by year-end | $0.44 | 142 |

Same direction. Almost the same mid. Looking at this table, an intuitive trader would say "they are basically the same contract, just at different horizons." That intuition is wrong, and the wrongness is buried in the time axis.

## Apply the Implied Yield Formula

Implied yield is the annualized return on a binary contract held to expiry, computed as:

```
IY = (1 / p) ^ (365 / τ) − 1
```

For contract A:

```
IY_A = (1 / 0.42) ^ (365 / 18) − 1
     = (2.381) ^ (20.28) − 1
     ≈ 8.3 million percent
```

For contract B:

```
IY_B = (1 / 0.44) ^ (365 / 142) − 1
     = (2.273) ^ (2.57) − 1
     ≈ 268%
```

The mid-price tables said these were nearly identical contracts. The IY table says one is paying ~8.3 million percent annualized and the other is paying ~268%. They are not the same trade. They are not even the same *kind* of trade.

## What the Difference Actually Means

The difference is leverage in the time axis. Contract A asks you to take the same conviction and apply it over 18 days; contract B asks you to apply it over 142 days. Conviction held over 18 days compounds dramatically when you express it as an annualized rate. Conviction held over 142 days does not.

Translated into action:

- Contract A is a *high-frequency conviction trade*. If you think the next Fed meeting will see a cut, you are getting paid an enormous premium for the willingness to take that view over a short window. Most of the IY is "coming from" the fact that you are accepting all-or-nothing risk on a near-term event. Sizing this contract on raw probability terms understates the leverage you are taking on; you should size it like a *short-dated option*, not like a long-dated bond.

- Contract B is a *yield trade*. The 268% annualized return looks high — and it is, by treasury standards — but on a 142-day horizon you are getting a much more modest *absolute* payoff in exchange for tying up capital across several Fed meetings. This is a position you can park alongside fixed-income exposures and compare apples-to-apples with credit or convertible yields.

These two framings demand different sizing, different risk management, and different mental models. Raw probability hides all of this. Implied yield exposes it the moment you compute the number.

## Why Fixed-Income Vocabulary Wins

The reason IY works as a framing is that the trader on the other end of a treasury desk *already* speaks this language. They live in basis points, yields, and annualized returns. When you walk into that conversation with "the contract is at 42 cents," you have lost them before the second sentence. When you walk in with "this contract is paying 8.3 million percent annualized," you are speaking their language even if the number itself raises an eyebrow.

The eyebrow is exactly what you want. The eyebrow leads to "wait, that can't be right" leads to "what is τ" leads to "show me the formula" leads to "okay, this is genuinely a short-dated leveraged binary, what is the other side of the trade" — and now you are actually doing risk management together.

Raw-probability framing skips that conversation entirely, because there is nothing in "$0.42" that sounds like anything a fixed-income trader has ever priced. IY gives you the bridge.

## Where the Lens Breaks

IY is not magic. Three things to keep in mind before you start expressing every prediction-market position as a yield:

1. **It assumes you actually win.** IY is the return *conditional on* the contract resolving in your favor. If you have a 60% subjective probability that contract A resolves YES, the *expected* IY is much, much lower than the raw IY. Always carry the conditional in your head.

2. **It explodes near expiry.** As τ → 0, IY → ∞ for any contract not yet pinned to 0 or 1. This is mathematically real and economically meaningless. Stop computing IY on contracts with less than ~5 days left; use raw bid-ask edge instead.

3. **It assumes you can transact at the mid.** IY based on a stale mid-price across a wide bid-ask is fiction. Always pin IY to the side of the book you can actually fill against.

## The Habit

Once you have computed IY a few times on real Fed-decision contracts, the framing becomes automatic. You stop seeing "$0.42 / 18 days" and start seeing "8.3M% annualized over a Fed window I can defend." You also stop sizing the two contracts the same way, because your gut now knows they are not the same animal.

That habit shift — from probability to yield as the default unit on bond-adjacent contracts — is what makes the difference between a prediction-market trader and a *prediction-market quant*. The quant uses IY because it is the only number that lets the rest of the rate stack talk back.