# τ-days

**τ-days is the continuous time-to-resolution of a prediction-market contract, computed as (close_ts − now_unix) / 86400 and held as a floating-point value. The unit is intentionally continuous, not integer, because every other indicator in the stack — implied yield, cliff risk, expected edge — exponentiates against τ, and a 6-hour rounding error in the input becomes a 100% rounding error in the output.**


## Explanation

## Why a Float, Not an Int

A naive implementation of "days remaining" rounds. A market that resolves in 30 hours becomes "1 day remaining." A market that resolves in 36 hours also becomes "1 day remaining." The two markets get the same τ even though one is twice as far from resolution as the other.

That rounding is fine for human display ("about a day left") and catastrophic for computation. Implied yield is `(1/p)^(365/τ) − 1`. The exponent term `365/τ` is wildly sensitive to small changes when τ is small. At τ = 1.0 the exponent is 365. At τ = 1.25 it is 292. At τ = 0.75 it is 487. The IY for the same price moves by a factor of two between those points. Rounding to integers throws all of that away.

The convention is therefore to keep τ as a float at all times in computation, and only round at the moment of human display. The formula is one line:

```
τ_days = (close_ts − now_unix_seconds) / 86400
```

Where 86400 is the number of seconds in a day. A market closing in exactly 36 hours has τ = 1.5. A market closing in 6 hours has τ = 0.25. A market that has already passed its close timestamp has τ ≤ 0 and is excluded from any indicator computation.

## τ_remaining vs τ_total

Two related measurements that get confused. τ_remaining is what is left from now until resolution — the live measurement that drives implied yield and cliff risk. τ_total is the original window from the contract's creation to its resolution — useful for normalizing how much of the contract's life has been priced. Most indicators want τ_remaining; a few (notably the "fraction of life remaining" gauge in the indicator stack) want τ_total or the ratio τ_remaining / τ_total.

When in doubt, use τ_remaining. It is the term that appears in IY and CRI, and it is what `market_indicator_history` records at each snapshot.

## The Bond-Desk Analog

Fixed-income traders call the equivalent quantity "duration," and they care about it for the same reason: every yield calculation exponentiates against it, and every price-sensitivity calculation differentiates against it. A binary prediction-market contract is, mathematically, a zero-coupon bond with binary recovery, so duration and τ-days are computing the same thing for the same reason: what is the time axis on which this position is exposed?

The simpler vocabulary in prediction markets is on purpose. Bond duration has Macaulay, modified, key-rate, and effective variants because bonds have intermediate cash flows (coupons) that need to be weighted. A prediction-market contract has one cash flow at maturity. The simpler version of duration — calendar days to that single payment — is τ-days. No weighting required.


## Example

A Kalshi market with `close_ts` = 1748736000 (an arbitrary Unix timestamp in mid-2025), evaluated at `now_unix` = 1748580000:

```
τ_days = (1748736000 − 1748580000) / 86400
       = 156000 / 86400
       ≈ 1.8056
```

So τ ≈ 1.81 days. About 43 hours from resolution. If you naively rounded to "2 days," you would compute IY at the wrong exponent. The actual exponent in the IY formula is `365 / 1.81 ≈ 201.7`, not `365 / 2 = 182.5`.

For a YES contract at p = 0.40, the rounded IY would be:

```
IY_rounded = (1 / 0.40) ^ 182.5 − 1  ≈  3.6 × 10^54%
```

The float-precise IY is:

```
IY_float = (1 / 0.40) ^ 201.7 − 1  ≈  3.4 × 10^60%
```

Both are absurdly high (because a 40-cent contract with two days left is inherently absurd-yield), but they differ by six orders of magnitude. Neither is useful for absolute return forecasting; both are useful for sorting contracts against each other on the same horizon, and that sort is meaningless if you have rounded τ before computing.

The practical convention in the codebase: keep τ as a JavaScript `number` (always a 64-bit float), never round in computation, only format for display. Every indicator in `src/lib/indicators/compute.ts` follows this rule.


## CLI

```bash
sf scan --max-tau 30 --by-iy desc
```


## Related

[implied-yield](implied-yield.md), [cliff-risk-index](cliff-risk-index.md), [expected-edge](expected-edge.md), [expiration](expiration.md), [resolution](resolution.md), [binary-contract](binary-contract.md), [event-contract](event-contract.md), [valuation-funnel](valuation-funnel.md)
