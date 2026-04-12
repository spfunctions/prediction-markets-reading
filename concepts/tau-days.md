# τ-days: The Continuous Time Unit Underneath Every Indicator

> Calendar days to resolution, expressed as a float, drives every other indicator in the stack. The unit looks trivial. Getting it wrong corrupts every downstream number.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 12 min

---
Every indicator in the SimpleFunctions stack — implied yield, cliff risk, expected edge, even the warm-cron prioritization — is multiplied by, divided by, or scaled against τ. If τ is wrong by 30%, every dollar of edge you compute is wrong by 30%. If τ is rounded to an integer, the short end of the curve becomes nonsense. If τ is computed against the wrong timezone, the whole Fed-decision book mis-prices for the entire week.

τ is the unit underneath the unit. It is the most boring number in the indicator stack and the one I have shipped the most bug fixes against.

## What τ Actually Is

τ is the continuous time-to-resolution of a binary contract, expressed in days, as a floating-point number:

```
τ_days = (close_ts − now_unix_seconds) / 86400
```

That is the entire definition. Three things to notice.

The first is that τ is a *float*, not an integer. A market that closes at 6 PM today and is being scored at 9 AM today has τ = 0.375. A market that closes tomorrow morning at the open has τ = 0.62. Rounding τ to integer days throws away the entire intraday curve and replaces it with a step function that snaps to 1 the moment the calendar flips. Every short-dated IY computation becomes wrong by the time-of-day fraction. This sounds pedantic until you watch the same Fed-meeting contract flip its IY by an order of magnitude depending on whether the cron ran at 8 AM or 8 PM, all because somebody thought τ was `floor((close_ts − now) / 86400)`.

The second is that τ is *signed*. A market past its close timestamp has τ < 0. The right behavior is to treat negative τ as "undefined for indicator purposes" and exclude the market from the scan. The wrong behavior is to silently take `Math.max(τ, 0)` and let a stale market sit at IY = ∞. Negative τ is not an edge case to coerce; it is a signal that the contract should not be in the scan set at all.

The third is that τ is a *moving target*. Every second that passes, τ shrinks by 1/86400. The cliff-risk-index time term and the implied-yield exponent both depend on τ in ways that compound: as τ shrinks, IY grows (because the same payoff is being earned over a smaller window) and CRI grows (because the same velocity is being scaled against a smaller remaining window). Both numbers want to explode near expiry. The job of the τ implementation is to stay honest about that explosion rather than smoothing it away.

## Why "Float Not Int" Matters in Practice

A Kalshi Fed-decision contract closes at 2:00 PM ET on the announcement day. If your scan runs at 9:00 AM ET on the close day, the calendar says "today." The integer-day implementation says τ = 0. Plug that into IY:

```
IY = (1 / 0.42) ^ (365 / 0)  →  divide by zero
```

The float-day implementation says τ = 5/24 ≈ 0.208. Plug that in:

```
IY = (1 / 0.42) ^ (365 / 0.208) − 1
   = (2.381) ^ (1755) − 1
   ≈ a number with hundreds of digits
```

That second number is also wrong, in the sense that no human will ever interpret it. The difference is that the second number is wrong in a way the *system* can detect and filter (with a τ < 0.25 cutoff, for example). The first number crashes the cron job and corrupts the whole batch.

The pattern repeats at every short horizon. A weather contract that closes in 6 hours has τ = 0.25. A movie box-office contract that closes Sunday at midnight, scored on Saturday afternoon, has τ ≈ 1.4. A pre-market sports contract that closes 90 minutes before tip-off has τ ≈ 0.06. None of these can be expressed in integer days without losing the entire intraday curve.

## The close_ts → τ Derivation, Step by Step

Kalshi exposes close_ts as a Unix epoch in seconds, in UTC. Polymarket exposes a similar field but in milliseconds. The right-hand side of the formula is "now," and now is also UTC seconds. The conversion to days is exactly division by 86400 (seconds per day, ignoring leap seconds — which don't matter at this resolution).

Three corruption modes you should be alert to. The first is *unit confusion*: I have shipped two bugs where milliseconds were treated as seconds and the resulting τ was 1000× too big. Every market suddenly looked like it had thousands of years to resolution. Every IY collapsed toward 0%. The error was visible only at the global level — a single market looked plausible. The fix was a one-line clamp that detected close_ts > some-far-future-bound and rejected it.

The second is *timezone drift*. Some upstream APIs return close_ts in the venue's local timezone (Eastern for Kalshi, UTC for Polymarket) but with the same unit. The two are off by 4-5 hours depending on daylight saving. For a 200-day contract, this is invisible. For a 6-hour contract, it inverts whether the market is even still open.

The third is *clock skew between machines*. The cron job runs on Vercel; the data lives in Postgres; the user opens the page on their phone. If any of these clocks disagree about "now" by more than a few minutes, the τ values displayed in the UI will not match the τ values used by the cron when it computed the indicators. The fix is to compute τ on the *server side at fetch time*, never trust client-side τ, and recompute on every read for short-dated markets.

## Where τ → 0 Breaks Every Indicator

Every Tier A indicator in the stack has a known failure mode at the short end:

**Implied Yield.** IY = (1/p)^(365/τ) − 1. As τ → 0, the exponent goes to infinity, and IY explodes for any p < 1. Mathematically real. Economically meaningless. The cutoff I use in production is τ < 0.25 days (six hours), below which IY is set to null. Six hours is the convention I picked because it cleanly separates "still tradable for a thesis" from "settlement noise during the resolution window."

**Cliff Risk Index.** CRI = |Δp/Δt| × τ_remaining. Notice the τ multiplier. As τ → 0, the multiplier shrinks and CRI deflates. This is by design — a 10-cent move with 2 hours left is not a structural reassessment; it is the market crystallizing toward the resolution. Without the τ multiplier, every contract about to settle would dominate the CRI scan. With it, the loudest near-expiry moves get correctly demoted.

**Event Overround.** EE = Σpᵢ − 1. EE does not depend on τ directly, but it does depend on the orderbooks of every sibling outcome being live and quoted. As any one of those siblings approaches τ → 0, its quote starts to crystallize, and the sum stops representing a coherent market view. EE on a multi-outcome event with one sibling in its final hour is fiction; the sibling has effectively already resolved.

**Liquidity Availability Score.** LAS does not use τ in the formula but is gated by the warm-regime cron, which prioritizes markets by remaining τ within an active window. As τ → 0, the cron pulls fresh orderbook data more aggressively, but the window is narrow enough that the orderbook is often empty. LAS converges to null at expiry, which is the correct behavior — the question "is this tradable" becomes degenerate when there are no more trades to make.

The pattern across all four: τ → 0 is the noise floor of the indicator stack. Every formula has a cutoff somewhere in the 0.1 to 1 day range below which the output stops being meaningful. The job of the τ implementation is to expose that cutoff cleanly so the downstream code can apply it.

## τ_remaining vs τ_total: Why Both Matter

Sometimes you want τ_remaining (days left until close). Sometimes you want τ_total (days from market open to close). Confusing the two is a category error.

τ_remaining is the right input to IY (because the yield is computed against the holding period until *resolution*, not the holding period since the market opened). τ_remaining is also the right input to CRI's τ multiplier (because what matters is how much story room is left for the move to mean something).

τ_total shows up when you want to *normalize* a metric across markets of different lifetimes. For example: "the price has moved 18 cents since the market opened" is more interesting on a market that has only been open for 3 days than on one that has been open for 300. A normalized version of velocity uses Δp / τ_total in the denominator. CRI in its production form uses τ_remaining, but a related metric you might compute in research uses τ_total.

The two are not interchangeable. A market with τ_total = 365 days and τ_remaining = 7 days is in a very different state than a market with τ_total = 7 days and τ_remaining = 7 days, even though they share the same τ_remaining. The first is a long-running thesis market about to resolve. The second is a short-window news market that just opened. The IY math treats them identically. The CRI math treats them identically. But your sizing and your conviction should not.

I do not bake τ_total into the production indicator stack because the data is harder to source cleanly (open_ts is sometimes missing or repurposed by the venue). But it shows up in any honest research notebook.

## Why τ Is the Right "Duration" Analog

Bond traders compute *duration* — the price-weighted average time to receive cash flows from a bond. Duration captures how exposed a bond is to interest-rate changes: longer duration means more sensitivity. A 30-year zero-coupon Treasury has duration ≈ 30. A 1-year zero has duration ≈ 1.

Binary prediction-market contracts are zero-coupon by construction. The only cash flow is the $1 payment (or $0) at maturity. There is nothing in the middle to weight. So the price-weighted average time collapses to *just* the time to maturity. τ_remaining is the duration of a binary contract, in the same units bond traders use, with no weighting machinery needed.

This matters because once you have duration, the rest of the bond-desk vocabulary unlocks. You can talk about a portfolio's *weighted average τ*. You can plot IY against τ to get a yield curve. You can find dislocations on the curve where one maturity is paying disproportionately well. You can compute a *roll-down* trade by buying a long-τ contract and watching τ shrink toward a lower-yielding short-τ point on the curve. All of this is bond-desk machinery that translates directly because τ is the right primitive.

The thing that doesn't translate is *convexity*. A real bond has a non-linear price-yield relationship; a binary contract has a linear price-payoff relationship at any single yield level. The closest analog is the cliff-risk index, which captures how sharply price reacts to the approach of expiry, but it is not the same animal. I write more about that mapping in [the prediction markets need fixed-income language essay](/blog/prediction-markets-need-fixed-income-language).

## Where the τ Lens Breaks

Three ways τ as a primitive falls down. None of them are fatal, but all of them are real.

The first is *resolution uncertainty*. Some prediction markets do not have a hard close timestamp — they resolve "when the event happens" with no guarantee that the event happens by close_ts. A "Will Putin meet with Zelensky in 2026?" market technically closes Dec 31, 2026, but the answer is informed every time a meeting does or does not happen during the year. τ to the *informational* resolution is not the same as τ to the *administrative* close. The IY math uses the administrative close, which understates the rate at which information is arriving.

The second is *path dependence*. Some markets have early-resolution clauses. A "Will Bitcoin hit $200K in 2026?" contract resolves YES the moment Bitcoin touches $200K, not on Dec 31. The effective τ is much shorter than the close_ts implies for the YES side, and not for the NO side. IY with raw τ overstates yield on the YES side and understates it on the NO side. The fix is to use barrier-option-style adjustments, which I have not implemented in production because the venues do not consistently expose the early-resolution rules.

The third is *holiday and weekend effects*. A 30-day contract that spans a Fed meeting is not the same as a 30-day contract over a quiet news month. τ as a calendar number is blind to event density. The right analog is *event-time τ*, where you count news-bearing days rather than wall-clock days, but I have not built that primitive yet because the data on "what counts as a news day" is not clean enough to scan.

## The Habit

Compute τ as a float. Recompute it every time you read a market. Reject negative τ from any indicator scan rather than coercing to zero. Cut off any indicator that explodes below τ = 0.25 days (and pin the cutoff explicitly — never let the formula run free into the noise floor).

Then, once the unit is honest, the rest of the stack — [implied yield](/concepts/implied-yield), [cliff risk index](/concepts/cliff-risk-index), [event overround](/concepts/event-overround), and the [expected edge composition](/concepts/expected-edge) — becomes a stack of formulas you can actually trust. None of those formulas work without τ. All of them are wrong if τ is wrong.

Run `sf scan --by-iy desc --min-tau 1` to get a sorted list of contracts where τ is at least one day, excluding the noise floor. The min-tau flag is the production guardrail for everything τ touches. Live data is on the [/screen](/screen) page if you want to see the τ values for the current scan.