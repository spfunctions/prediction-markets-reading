# The Settlement Halo: Microstructure Changes in the Final 24 Hours

> Every binary contract goes through the same predictable shift in the 24 hours before resolution. Spreads compress, volume spikes, makers withdraw, retail piles in. Reading the halo tells you when the indicators stop meaning what they normally mean.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Pull up a Kalshi Fed-decision contract two days before the FOMC announcement and the orderbook looks like every other liquid contract on the platform. The bid-ask is somewhere between four and six cents, the depth on each side is stable, the price is drifting on news. Pull up the same ticker eighteen hours later and the orderbook is unrecognizable. The spread is one cent. The depth is doubled on the side that has been winning. The volume is four times the trailing 30-day average. New retail accounts that have never traded the contract before are putting in market orders for fifty contracts at a time.

This is the settlement halo, and it happens to every prediction market in the final 24 hours of its life. The halo is not noise. It is a structural rearrangement of the orderbook driven by the fact that resolution is now close enough that all of the indicators behave differently than they did a week ago. If you know how to read the halo you can avoid being chopped up by it; if you do not, you will spend years wondering why your stage-1 indicator scans keep returning trades that fail in the final day.

## The Empirical Pattern

I have watched several hundred Kalshi contracts go through the halo. The pattern is consistent enough that I can describe it without referring to a specific ticker. Use the following as a default expectation for any liquid binary contract entering its final day.

**Spread compression.** The bid-ask narrows from a typical 4-6¢ down to 1-2¢ over the course of about twelve hours. This is not because liquidity is improving — it is because the disagreement among makers about fair value is collapsing. As the resolution event approaches, the variance of the prior over outcomes falls, and the spread (which is approximately a measure of that variance, plus an inventory risk premium) falls with it.

**Volume spike.** Daily volume goes from a baseline of, say, 8,000 contracts to something like 32,000 contracts over the final 24 hours, a 4x lift on the trailing average. The lift is not symmetric — it comes from new participants entering the market for the first time, almost all of them retail, almost all of them taking liquidity rather than providing it.

**Maker withdrawal on the losing side.** This one is the most important and the easiest to miss. As the price drifts toward one of the two resolution outcomes, makers stop quoting on the side that is losing. If a Fed-cut contract has drifted from 0.42 to 0.61 over the course of the week, makers will be aggressively bidding the YES side and quoting thin asks. They will not be quoting the NO side at all in the final hours, because the inventory risk of getting filled on a NO position that is about to settle to zero is unhedgeable in any meaningful way.

**Retail flow asymmetry.** The new retail volume that arrives in the halo is overwhelmingly on the side that has already won. People see "Fed cut at 61¢, expected to resolve YES tomorrow" and they buy YES. They are not pricing — they are confirming. This flow chases the existing direction and accelerates the move toward resolution.

The four effects compound. Spreads compress because variance of prior falls. Volume spikes because retail joins. Makers withdraw on the losing side because hedging becomes impossible. Retail flow is one-sided because they are arriving late. By the final eight hours, the contract is no longer a "market" in the sense of two-sided price discovery — it is a one-sided liquidation event with a thin counterflow from arbitrageurs.

## Why It Happens

The halo is not a quirk of any particular venue. It is a consequence of the math of binary settlement.

Binary contracts have one-shot terminal value. The expected value of a YES contract held to settlement is exactly the settlement probability, period. A week before resolution, traders disagree about that probability (some think 40%, some think 60%, the market clears around 50% with a 5¢ spread that captures the disagreement). A day before resolution, most of the disagreement has been resolved by news, by polls, by partial information leaks. The variance of the prior collapses, and the spread collapses with it because makers no longer need to charge as much for being wrong about which side will hit them.

At the same time, the inventory risk for a maker holding the *losing* side of a near-term binary becomes unbounded. If you are a maker bidding the NO side of a Fed-cut contract that is now trading at 0.78 with eight hours to settlement, you are taking on an asymmetric risk: if the cut happens, your inventory snaps to zero and you lose 78¢ per contract. There is no hedge for this. There is no offsetting position. The only sensible inventory management is to stop bidding entirely. So the losing side empties out.

The volume spike is sociological. Resolution is when the prediction market becomes news. People who have never traded prediction markets see "Polymarket says Powell stays at 78%" in a headline and decide they want to either confirm or fade that view. The platform's UX makes it easy: they sign up, deposit, and click YES. None of these flows are price-sensitive. They are sentiment expressions, and they all hit the same side.

I learned to read the halo by losing money to it. I had a stage-1 scan that ranked contracts by IY descending, and one Tuesday morning my scan returned a contract that had been at 0.18 with 36 hours to expiry. The IY was nominally about 4500%. I bought a hundred contracts at the ask, which was 0.20. Eighteen hours later the contract was at 0.04 and the bid had vanished. The "trade" was a textbook halo failure: I had bought the losing side of a contract that was already pricing in the win, and the maker withdrawal on my side of the book meant I could not even exit at a sensible loss. That is when I started writing down what the halo actually does to the indicators.

## Three Trading Implications

**1. Do not enter new positions in the halo unless you are arbing the spread compression.** The halo is the one window in a contract's life where the indicator-driven scan you trust the rest of the time will lie to you systematically. IY explodes near τ → 0 and that is captured by the formula, but the *executable* IY is much lower because the side you can fill has been emptied out by maker withdrawal. The published mid-price is fiction in the halo. Any new entry should require a specific reason that overrides the general "wait for τ > 24h" rule.

**2. The 24-hour spread compression is a free trade for arbitrageurs.** If you are running a cross-venue arb book, the halo is when your edge is largest. The same outcome on Kalshi and Polymarket will both compress, but they will compress at different rates and from different starting spreads. The basis between the two venues is most exploitable in the final 12 hours, because the noise from disagreement among makers has collapsed and what is left is structural friction (fees, bridge costs, dispute risk on Polymarket). This is the one place where I will let `sf scan --by-cri desc` run with τ < 1 day and act on its output, but only on cross-venue pairs.

**3. High CRI in the halo is meaningless.** [Cliff Risk Index](/learn/cliff-risk-index) is calibrated for contracts with structural news flow, where a 1¢ move at high τ is signal. In the halo, every contract has high CRI by construction — the price is converging to its terminal value, so velocity is high and τ is low and the product can look extreme. CRI as an *attention allocator* should turn off below τ = 1 day. A scan that ranks by CRI descending in the halo will return only halo contracts, which is exactly the wrong list to act on.

## Where the Halo Reading Breaks

The halo is a default, not a law. Three places where it does not apply.

**Illiquid contracts that never had a halo to begin with.** A market with $400 of total 24-hour volume will not show the spread compression / volume spike pattern, because there were never enough makers quoting it for "withdrawal" to mean anything. These contracts settle on the last few orders that happen to cross, often at prices wildly disconnected from their fair value. The halo description does not apply here, and neither do the trading implications.

**Contracts with a hard scheduled catalyst inside τ = 24h.** A Fed-decision contract with the FOMC meeting at hour 18 of the halo will have a discontinuity at that hour, not a smooth compression. The pre-meeting halo looks normal; the post-meeting price is the new reality. Reading these as "smooth halos" will mislead you. The right framing is to break the contract into two episodes: the pre-catalyst halo and the post-catalyst settlement window.

**Contracts that pin to 50% by mechanism.** Some daily-settled price-target binaries are designed to be balanced at the close of the trading day. The halo on these does not look like the standard one — see the [tail-of-day-pin-risk](/concepts/tail-of-day-pin-risk) concept page for what to expect instead. The microstructure of a contract that is *supposed* to pin at 50¢ is fundamentally different from a contract that is supposed to resolve to 0 or 1.

A fourth case worth flagging: some contracts have *recurring* halos. A weekly-settled BTC price binary goes through a halo every Friday. The halo behavior is stable across weeks, which means it is partially priced in by makers in advance. You will see the spread start compressing earlier, the volume start lifting earlier, because the weekly cycle has trained the participants. Your halo timing on these contracts should be calibrated against the previous several weeks, not against a generic "24 hours before settlement."

## The sf CLI Read

If you want a one-line check for whether a contract is in its halo, the spread + volume signal is enough. `sf scan --warm --tickers <ticker> --since 24h` returns the trailing 24-hour window of orderbook snapshots, and you can eyeball whether the spread has fallen by more than 50% and the volume has lifted by more than 2x. If both, the contract is in its halo. Skip it for new entries; consider it for arb only.

The deeper read is to compare the contract's current spread to its 30-day rolling average spread, which is what the market regime snapshot substrate (the same data that powers the [orderbook](/learn/orderbook) glossary entry) is built for. A contract whose current spread is below its 7th percentile of the trailing 30 days is almost certainly in the halo, regardless of its absolute τ. This is the version of "halo detection" that does not require you to remember the τ threshold.

## How This Fits the Stack

The halo is the most important *exception* to the [valuation funnel](/concepts/the-valuation-funnel). The funnel assumes that stage-1 indicators (IY, CRI, EE) and stage-2 orderbook reads are produced by markets in their normal regime. The halo is not the normal regime, and any candidate the funnel returns from a contract in its halo should be flagged and re-scrutinized before it goes to stage 3. The simplest implementation is a hard τ filter: `--min-tau 1` on every scan, with a separate halo-aware scan that runs only on cross-venue pairs.

For the broader phenomenon catalog, see [tail-of-day-pin-risk](/concepts/tail-of-day-pin-risk) for how daily-settled contracts compress in their final 30 minutes (a much faster halo), [longshot-bias](/concepts/longshot-bias) for what the halo does to extreme-priced contracts, and [information-latency](/concepts/information-latency) for the news-reaction window that often *creates* the halo in the first place.

The halo is the one phenomenon I make every new prediction-market trader I mentor learn first. Not because it is the most interesting — it is not — but because misreading it is how new traders lose their first $500. Once you can describe what is happening to the orderbook in the final 24 hours, you have the floor under the rest of the indicator stack. Without it, you are running a scanner that returns trades you cannot actually execute.