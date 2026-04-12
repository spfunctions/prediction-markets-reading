# Implied Yield (IY)

**Implied yield is the annualized return a binary prediction-market contract pays if held to expiry, computed as IY = (1/p)^(365/τ) − 1, where p is the YES price (0–1) and τ is days to resolution. It converts a binary mid-price into the same units fixed-income traders use for treasuries.**


## Explanation

## Why a Binary Contract Has a Yield

A binary prediction-market contract pays $1 if it resolves YES and $0 if it resolves NO. If the current YES price is $0.40 and the contract resolves in 60 days, you put up $0.40 today and — assuming you are right about the outcome — receive $1.00 in 60 days. That is a return, and like any return tied to a holding period, it can be expressed as an annualized yield.

The closed form is:

```
IY = (1 / p) ^ (365 / τ) − 1
```

Where:

- **p** = current YES price, expressed as a probability between 0 and 1
- **τ** = days until the market resolves
- **IY** = the implied annualized yield, treated as a compounding return

A contract at $0.50 with one full year to resolution has IY = 2¹ − 1 = 100%. Same contract priced at $0.25 with one year remaining has IY = 4¹ − 1 = 300%. The same $0.25 contract with only 30 days left has IY = 4^(12.17) − 1 ≈ 16,700,000%. Short-dated, low-priced binaries carry enormous IY because the payoff is fixed at $1.00 and the time window is tiny.

## Why It Matters

Most traders look at a contract trading at 40 cents and think "60 cent edge if I'm right." That framing throws away the holding period. A 60-cent payoff over 60 days is very different from a 60-cent payoff over 600 days, and IY makes the difference visible in a unit traders already understand.

It is also the right number to put next to a treasury bill. If a 90-day contract is trading at an IY of 12% and you can buy a 3-month T-bill at 5%, the prediction-market position pays you seven percentage points more annualized — at the cost of binary outcome risk and zero coupon. That risk-adjusted comparison is the core of every "is this a fair bet" question.

## Where IY Breaks Down

IY assumes the contract resolves at p = 0 or p = 1 cleanly. It is meaningful only on the leg you actually believe in — applying it to both YES and NO at once gives nonsense, because only the side that wins gets paid.

It also approaches infinity as τ → 0 (the formula is undefined at expiry) and behaves erratically when the market is very thin, because the price you can transact at is far from the displayed mid. Treat IY as a comparison metric across contracts on similar horizons, not as a literal forecast of expected return.


## Example

Kalshi contract: a Fed-decision YES at $0.32 with 45 days to resolution.

```
IY = (1 / 0.32) ^ (365 / 45) − 1
   = (3.125) ^ (8.111) − 1
   ≈ 10,303 − 1
   ≈ 1,030,200%
```

The number is absurd because a 45-day binary at 32 cents is inherently high-leverage. The useful framing is "3.1× your money in 45 days if right." IY is most informative when you compare two contracts on the same horizon, not in absolute terms.

For the same contract priced at $0.45 with 90 days to expiry:

```
IY = (1 / 0.45) ^ (365 / 90) − 1
   = (2.222) ^ (4.056) − 1
   ≈ 25.55 − 1
   ≈ 2,455%
```

Still high, but two orders of magnitude lower. The contract has become both more expensive and longer-dated, and IY captures both effects in a single number.


## CLI

```bash
sf scan --by-iy desc
```


## Related

[implied-return](implied-return.md), [edge](edge.md), [cliff-risk-index](cliff-risk-index.md), [event-overround](event-overround.md), [cost-basis](cost-basis.md), [binary-contract](binary-contract.md)
