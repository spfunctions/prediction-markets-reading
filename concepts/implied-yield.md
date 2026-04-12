# Implied Yield: The Bond-Trader View of a Binary Contract

> IY = (1/p)^(365/τ) − 1, walked through three real Fed and weather contracts, four edge cases, and the framing that turns prediction markets into a credit-risky zero.

**Category:** indicator | **Author:** Patrick Liu | **Reading time:** 14 min

---
A binary prediction-market contract is, mathematically, a credit-risky zero-coupon bond. You buy it at some discount to par. You hold it until a known maturity. At maturity, you receive par or you receive zero, conditional on a single observable event. Bond traders have priced this exact instrument for fifty years.

The number that bond traders use to compare credit-risky zeros to each other and to the rest of the rate stack is *yield*. The number that prediction-market traders use is *price in cents*. Implied yield is the formula that translates between the two.

## The Formula

```
IY = (1 / p) ^ (365 / τ) − 1
```

Where p is the YES price expressed as a probability between 0 and 1, and τ is the days to resolution as a float. The output is a fractional return — multiply by 100 to display as a percentage.

The derivation is one-line. If you put up $p today and receive $1 at resolution τ days from now, your simple return is (1/p) − 1. To annualize, you raise the gross return (1/p) to the (365/τ) power, then subtract 1 to convert from gross to net. That's it. There are no continuously-compounded variants in the production code, because annual compounding matches the conventions of the T-bill desk and that is the comparison set we care about.

Three things follow from the formula structure. First, IY is *not bounded above*. A 1-cent contract with 1 day to resolution has IY = 100^365 − 1, a number with hundreds of digits. This is not a bug; it is an accurate statement that you cannot buy a 1-cent binary that resolves tomorrow without someone disagreeing very strongly with you about what is about to happen. Second, IY is *bounded below at zero*. A contract trading at exactly 1.00 has IY = 0, because there is no remaining yield to extract. Third, IY is *undefined* at p = 0 (you cannot buy a free option) and at τ ≤ 0 (you cannot earn a return after maturity). Both edge cases must be filtered before scanning.

## A History of the Term

Yield as a primitive comes from the bond market, not from prediction markets. The original bond yield is the *current yield*, which is just coupon divided by price. As bonds got more complex, the desk needed a more general number, and yield-to-maturity (YTM) emerged: the discount rate at which the present value of all future cash flows equals the current price. For a coupon bond, this is a polynomial root-finding problem. For a zero-coupon bond, the polynomial collapses to a single power and you get back exactly the formula above.

Sports betting has a parallel concept called *implied probability*, which is the inverse of the decimal odds. A horse priced at 4.0 has an implied probability of 0.25. The implied probability framework treats the bet as a single-shot, no holding period, no annualization. The bond-market framework treats the same instrument as something you hold over time and earn yield against. Prediction markets sit between the two — they have the binary payoff structure of sports betting and the holding-period structure of bonds — and implied yield is the formula that recognizes the second of those two.

The borrowing into prediction markets is recent. I started computing IY on Kalshi contracts in early 2025 because I came from a fixed-income background and the price-in-cents framing felt impoverished. The first version of the formula I shipped used 252 days (the trading-day count) instead of 365, which is wrong because prediction markets resolve on calendar days, not trading days. The fix took 20 minutes. The lesson took longer: the right reference unit for prediction markets is the calendar, not the exchange.

## Three Worked Examples

I want to walk through three real-shaped contracts at different τ and p values to build the intuition.

**Example 1 — Long-dated, mid-priced.** Imagine a Kalshi contract: "Will the Fed cut rates at any meeting in 2026?" Trading at $0.62 with τ = 200 days remaining.

```
IY = (1 / 0.62) ^ (365 / 200) − 1
   = (1.613) ^ (1.825) − 1
   ≈ 2.376 − 1
   ≈ 137.6%
```

A 137.6% annualized return on a contract you believe in. The number is high by any treasury comparison (12-month T-bill is about 5%), and it should be — you are taking binary outcome risk on a Fed-policy thesis. This is in the range I think of as a "yield trade": long enough horizon that you can size it like a fixed-income position, low enough leverage that one wrong decision will not blow you up.

**Example 2 — Short-dated, mid-priced.** Same contract structure but at the next meeting only: "Will the Fed cut rates at the May 2026 meeting?" Trading at $0.42 with τ = 18 days remaining.

```
IY = (1 / 0.42) ^ (365 / 18) − 1
   = (2.381) ^ (20.28) − 1
   ≈ 8,300,000 − 1
   ≈ 8.3 million percent
```

The same conviction, applied over an 18-day window instead of 200, produces an annualized number that is six orders of magnitude larger. Every honest reaction to that number is "that can't be right." It is right. It just means that an 18-day binary at 42 cents is mathematically a leveraged short-dated option, and the IY math is the alarm bell that tells you so. Size this position like a leveraged short-dated option: small, and only when you have a hard reason for the conviction.

**Example 3 — Long-dated, low-priced.** A weather contract: "Will the Atlantic hurricane season produce 20+ named storms in 2026?" Trading at $0.15 with τ = 142 days.

```
IY = (1 / 0.15) ^ (365 / 142) − 1
   = (6.667) ^ (2.570) − 1
   ≈ 113.5 − 1
   ≈ 11,250%
```

This is the example that breaks people new to the formula. A 15-cent contract with about five months to go is paying 11,250% annualized. That sounds insane, but it is structurally normal for low-priced binaries with meaningful holding periods. The IY framing surfaces what the price-in-cents framing hides: this is a contract where being right is worth nearly seven times your stake by maturity, and the yield reflects that convexity.

The pattern across all three: IY is most useful as a *comparison metric* across contracts on similar horizons, less useful as a single absolute number. The 137.6% on contract 1 is more informative when you put it next to a 5% T-bill yield on the same horizon than when you stare at it alone.

## The "Every Contract Is a Credit-Risky Zero" Framing

Once IY is the working unit, every prediction-market contract becomes legible as a fixed-income instrument with three parameters: principal (always $1.00 par), maturity (τ days), and credit (the probability the issuer "pays" — which here means the probability the event resolves YES).

The credit isn't issued by a company; it is issued by the world. There is no balance sheet to read, no rating agency to consult, no covenant to enforce. The credit analyst's job is replaced by *causal reasoning about the event* — who needs to do what, by when, for the contract to resolve in your favor. The IY math is the same; the credit story is different. I write more about that handoff in [the bond-language essay](/blog/prediction-markets-need-fixed-income-language).

What this framing buys you, beyond the comparison-to-T-bills move, is the ability to construct *portfolios* of credit-risky zeros and reason about them as a whole. A weighted-average IY across your prediction-market book tells you the headline yield you are running. A weighted-average τ tells you the duration. A correlation matrix across your contracts tells you whether you are diversified or concentrated. None of this is possible with the price-in-cents framing because price-in-cents is not additive across positions in any meaningful way.

## Four Edge Cases Where IY Breaks

The formula is mathematically simple, but four real-world conditions can make the output meaningless. All four need to be handled in any production scan.

**Edge case 1 — τ → 0.** As discussed in the [τ-days concept page](/concepts/tau-days), every binary contract approaches infinite IY as τ → 0 for any p < 1. This is mathematically real and economically nonsense. The production cutoff I use is τ < 0.25 days (six hours), below which IY is set to null and the contract is excluded from the scan. The cutoff is a convention; you could pick anywhere from 0.1 to 1 day depending on what you trade.

**Edge case 2 — p → 0.** A contract trading at $0.00 has infinite implied yield (because you can buy par for free). The output is undefined at exactly zero and explodes near zero. The production cutoff is p > 0.005, below which IY is set to null. Below that threshold, the orderbook is so thin that the displayed price is almost always wrong, and any IY computation against it is fiction.

**Edge case 3 — p → 1.** A contract trading at $1.00 has IY = 0. Mathematically clean, economically meaningless — you have nothing left to extract. The production behavior is to set IY to null (or 0, depending on the consumer) and let the scan ignore the contract. Above p = 0.995, IY rounding errors start to dominate the signal, so the cutoff in production is p < 0.995.

**Edge case 4 — wide bid-ask spread.** IY computed against the midpoint when the bid is 12¢ and the ask is 38¢ is fiction — there is no real trade at the displayed midpoint. If you care about *executable* IY, always pull from the side of the book you can actually fill against. For long-side scans, that is yes_ask. For short-side scans, that is yes_bid. The midpoint is appropriate only when the spread is tight enough (say, < 5¢) to be a reasonable approximation of the executable price. For wider spreads, IY at mid is a dashboard number, not a trading number.

There is also a softer fifth case worth flagging: *stale prices*. Some Kalshi contracts have not traded in days, and the displayed YES price is whatever the last fill was. IY against a stale mid is meaningless because the market has not priced new information. The fix is to gate the scan on time-since-last-trade, which the warm-cron infrastructure handles for the top 500 markets but does not for the long tail. For long-tail markets, treat IY as suggestive rather than definitive.

## How to Combine IY With the Other Indicators in the Stack

IY alone is not a trade signal. A high-IY contract with no liquidity, no news flow, and no thesis behind it is a number on a screen, not a position. The rest of the indicator stack exists to filter the IY scan into something actionable.

The composition I use, in order:

First, scan by IY. `sf scan --by-iy desc --min-tau 1` returns a sorted list of contracts where IY is high enough to be interesting (say, > 100% annualized) and τ is at least one day (excluding the noise floor). This step is computationally cheap and runs against the full universe of ~47K markets. It typically narrows the universe to 100-300 candidates.

Second, filter by [cliff risk index](/concepts/cliff-risk-index). High IY with low CRI is a *stuck* market — the price is high because nobody is paying attention, not because there is a thesis you can defend. High IY with high CRI is an *active* market — the price is moving, the activity is real, and there is something to engage with. The CRI filter typically narrows 100 candidates to 30.

Third, check [event overround](/concepts/event-overround) on multi-outcome events. If the contract is part of a sibling set, EE tells you whether the sibling pricing is structurally consistent. EE > 0 with rich pricing on your candidate suggests the field is overconfident in *something* and your candidate may be where the slack is. EE near 0 means the sibling set is pricing cleanly and your edge has to come from somewhere other than the overround. The EE filter typically narrows 30 candidates to 5.

Fourth, apply judgment. The remaining 5 are the contracts where you actually engage with the orderbook, read the recent news, check the cross-venue prices, and decide whether to take a position. This is the [valuation funnel](/concepts/the-valuation-funnel) in action, and IY is the first stage.

The hierarchy is not optional. Skipping the IY filter means you're scanning 47K markets by hand. Skipping the CRI filter means you're sizing positions in markets where nothing is happening. Skipping the EE check means you're missing the structural signal that tells you whether the price is fair. Each filter compresses the candidate set by an order of magnitude, and the final five are the ones that get the full attention.

## A CTA

Live IY data for the entire Kalshi and Polymarket universe is on the [/screen](/screen) page. Sort by IY descending, set min-tau to 1, and scroll. The contracts at the top of that list are not all trades — most are noise — but they are the candidates the rest of the stack filters down. The bond-desk view of prediction markets starts here.