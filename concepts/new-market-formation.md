# New Market Price Formation: The First 24 Hours of a Listed Contract

> When a binary contract first lists, the price is wild. Spreads are 10+ cents wide, depth is in the single digits, and the displayed mid swings 20-40 cents on flows that would barely register on a mature market. The price during this window is not a forecast — it is the venue's makers learning what the contract is worth.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
A new prediction-market contract gets listed at 9:00 AM ET on a Tuesday. By 9:05 AM the YES bid is at 0.18 and the YES ask is at 0.34. By 9:30 AM the mid has wandered to 0.27. By 10:15 AM someone hits the bid for $80 and the price re-marks to 0.22. By noon the spread has tightened to 4 cents around 0.31. By the end of the day the contract is sitting at 0.29 with a healthy bid-ask and modest depth on both sides.

What just happened? The contract did not get any new information between 9:00 AM and 6:00 PM. Nothing about the underlying event changed. The "true" probability of the event was the same all day. What changed is that the venue's makers spent 9 hours figuring out where the contract should clear, and the price evolved through a series of speculative bids, exploratory fills, and revisions until something stable emerged.

This is what new market formation looks like, and the dynamics are predictable enough that you can build them into your trading approach. The first 24 hours of a listed contract are a specific phase of its life with specific risks and specific opportunities. Treating them like the rest of the contract's life is a mistake on both directions.

## Why the First Day Is Different

A mature prediction-market contract has a price that reflects the consensus of dozens or hundreds of traders who have looked at it, the depth that real flow has built up, and indicator values (CRI, IY, EE) that have stabilized into something meaningful. A brand-new contract has none of those things. It has:

- A maker who decided to seed the orderbook (sometimes the venue, sometimes a designated market maker, sometimes a sharp trader who got there first), and whose initial guess at the price is based on nothing more than their own intuition.
- A handful of early traders who saw the listing in a feed and decided to take a position based on whatever their priors were before they had any market data to anchor against.
- An [pm-indicator-stack](/concepts/pm-indicator-stack) that returns null or near-zero for almost everything, because there is no history yet. CRI requires Δp over time. PIV requires the 1¢-delta tracker. CYC requires the contract to be visible to the cycle-clustering grouper. None of these have data yet.

The contract is, in indicator-stack terms, *unobservable* for the first few hours of its life. You can see the price but you cannot see anything about how the price got there or what it means. That blindness is the defining feature of new market formation, and it is what makes the first day risky in a way that is unique to this phase.

## The Typical Pattern

I have logged the first-day price action on about 50 newly listed contracts over the last year. The pattern is consistent enough that I can describe it as a sequence of phases.

**Phase 1 — Maker seeding (first 30-60 minutes).** A single market maker (sometimes Kalshi's designated MM, sometimes a third-party entity, sometimes a trader using maker quotes) puts up a wide two-sided market. Spreads are 10-20 cents wide. The mid is the maker's best guess at the contract's value, which is often within 5-15 cents of where the contract eventually settles but can be wildly wrong on novel categories.

**Phase 2 — Discovery flow (hours 1-6).** Early traders find the listing and put in small positions. Each fill moves the price perceptibly because the depth is so thin. The price wanders within a 20-cent range as different traders pull it in different directions. The orderbook stays wide; the maker is not getting clear signal on which side is more aggressive yet.

**Phase 3 — Tentative consensus (hours 6-24).** Volume picks up as more traders see the listing. The spread tightens to 3-7 cents. The mid stabilizes within a smaller range — usually 5-10 cents wide — and starts to show actual mean reversion (a small move triggers a counter-move from another trader). The maker has gotten enough signal to tighten the quote.

**Phase 4 — Stable pricing (hours 24-72).** The contract starts to look like a normal mature market. Spreads are 1-3 cents. The mid moves only on real news flow, not on discovery flow. The indicator stack starts producing meaningful CRI values. The contract is now tradeable on its merits.

The phases are not crisp boundaries — they overlap and the timings vary by category. Election markets typically run through all four phases in the first 4-8 hours because the audience is large and finds new listings fast. Niche economic indicator markets can stay in phase 2 for days. Sports contracts typically converge fastest because the sports-betting audience overlaps and brings price intuitions from the sportsbooks.

## A Concrete Example

Last quarter Kalshi listed a new contract on whether a specific tech IPO would price by a certain date. I happened to be watching when it went live. Here is what I saw:

- 9:00 AM ET: Initial listing. Bid 0.10, Ask 0.40. Spread: 30 cents. Mid: 0.25.
- 9:14 AM: Someone hits the ask at 0.40 for 50 contracts. The maker re-stacks: Bid 0.18, Ask 0.42. Mid: 0.30. The take pulled the consensus *up*.
- 9:42 AM: Someone hits the bid at 0.18 for 80 contracts. Re-stack: Bid 0.12, Ask 0.36. Mid: 0.24. The opposite take pulled the consensus *down*.
- 10:30 AM: A larger trade — 200 contracts on the ask side at 0.36. Re-stack: Bid 0.22, Ask 0.38. Mid: 0.30. Now the consensus has moved up again.
- 12:15 PM: Spreads have tightened. Bid 0.27, Ask 0.32. Mid: 0.295.
- 3:45 PM: Steady pricing. Bid 0.28, Ask 0.31. Mid: 0.295.
- Next morning: 0.29 with 1-cent spread.

The contract moved through an 18-cent range during the first day. At 9:14 you would have said it was a 0.30 contract. At 9:42 you would have said it was a 0.24 contract. The "true" answer turned out to be 0.29, but you could not have known that until phase 4. Anyone who sized into a directional position in phase 1 or phase 2 was making a bet not on the underlying event but on which way the discovery flow would settle — and the discovery flow is essentially random.

## Three Trading Implications

The pattern is reliable enough to build rules around.

**Implication 1 — Do not enter directional positions in the first 6 hours.** The price during phase 1 and phase 2 is not a forecast. It is the volatility of a small number of discovery trades pulling the mid in random directions. Sizing into this is sizing into noise. Wait until phase 3 (hours 6-24) at minimum before treating the price as actionable.

**Implication 2 — Do not size big in the first 24 hours.** Even after phase 3 begins, the price is still less anchored than a mature market would be. Use small position sizes — 25-50% of what you would normally use — until the contract has had at least one full trading day to stabilize. The discount on size is the cost of accepting that you are trading against a price whose error bars are still wide.

**Implication 3 — The spread itself is a tradeable opportunity for makers.** Phase 1 and phase 2 have 10-30 cent spreads. Any trader willing to put up two-sided maker quotes 3-5 cents inside the existing spread is going to capture flow from any of the discovery traders who decide to cross. The risk is that the discovery flow takes both sides and the maker ends up with adverse selection on whichever side is closer to the eventual fair value. The mitigation is small position sizes and aggressive cancellations whenever the price moves more than 3 cents.

I have made some of my most consistent maker income on phase-1 and phase-2 markets. The trick is to skim the spread while it is wide and exit before phase 3 turns the contract into a normal liquid market where everyone can compete on equal terms. `sf scan --new-listings` gives you a feed of contracts that have been live for less than 24 hours, sorted by current spread. The top of that feed is where the maker opportunity lives.

## Where the Pattern Breaks

The pattern is not universal. A few cases where it doesn't apply.

**High-attention listings.** When the venue announces a new market that everyone is waiting for — a major election contract, a high-profile economic indicator, a viral news event — the audience is already lined up at 9:00 AM and the contract goes through all four phases in the first hour. There is no slow discovery period; the price is anchored by mass attention from the moment of listing. Trade these like mature contracts from the start, but be aware that the first hour can still see 5-10 cent moves on volume.

**Listings with a clear analog.** If the new contract is the next-period version of an existing event family (e.g., the December Fed contract listing while the November contract is already live and mature), the new listing can inherit a price anchor from its sibling. The discovery period is shorter because the maker has a defensible starting price. The convergence to phase 4 can happen in 1-2 hours.

**Cold-listed obscurities.** The opposite case. Some contracts get listed and basically nobody notices. They sit in phase 1 — wide spreads, no flow — for *days*. The price during this extended phase 1 is essentially the maker's solo guess, with no validation from real flow. These are the contracts where the [virgin Polymarket strategy](/opinions/liquidity-availability-as-the-real-edge) lives — find the cold listing, become the only second maker on the book, capture the spread on the trickle of organic flow that eventually shows up.

**Mechanism 2 from the [hazard-rate-anomalies](/concepts/hazard-rate-anomalies) page.** A new sibling listed in an existing event family with a wrong-by-5-cents seed price is a special case of phase 1 where the wrongness creates an immediate cross-venue anomaly with the family's other siblings. The discovery flow has to fix the seed price *and* close the cross-sibling gap simultaneously, which sometimes takes a full week.

## How This Connects to the Stack

New market formation is an explicit failure mode of the [valuation funnel](/concepts/the-valuation-funnel) — specifically, stage 1 cannot scan a market that does not have indicator history yet. The funnel is designed for steady-state markets, and the first 24 hours are not steady state. The mitigation is to have a parallel "new listings" feed (`sf scan --new-listings`) that bypasses stage 1 entirely and routes you directly to manual evaluation of contracts that are too young to be screened.

The [pm-indicator-stack](/concepts/pm-indicator-stack) returns null or near-meaningless values during phase 1 and phase 2 of new formation, which is the [null-as-signal](/concepts/null-as-signal) principle in its starkest form. Null indicators on a brand-new contract do not mean "no signal" — they mean "the contract is in a phase where the only useful signal is the spread itself, and the spread is the maker opportunity."

For the maker side specifically, see the upcoming [maker-taker-regime-in-pms](/concepts/maker-taker-regime-in-pms) concept, which formalizes the regime detection that lets you tell whether you should be a taker or a maker on any given contract. New listings in phase 1 and phase 2 are by definition maker regimes — there is no taker flow yet — and that is the entry condition for the spread-skimming strategy.

For the cross-venue dimension, see [cross-venue-convergence](/concepts/cross-venue-convergence). New listings on one venue often have no equivalent on the other venue yet, which means the cross-venue convergence machinery has nothing to compare to. Wait until both venues list the contract before applying any cross-venue framework to it.

The first 24 hours are short. Most traders just skip them. That is the right call most of the time, because the risks outweigh the opportunities for a directional trader. For a market maker, however, the first 24 hours are the most profitable phase of a contract's life — wide spreads, low competition, and the price discovery is happening *through your quotes* rather than around them.