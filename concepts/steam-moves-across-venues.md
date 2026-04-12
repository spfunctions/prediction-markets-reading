# Steam Moves Across Venues: When Sharp Money Hits Both Books at Once

> Sports bettors have a name for the simultaneous coordinated movement of two books that signals real money: steam. Prediction markets have steam too, and it is the highest-quality signal on the screen because it is the only one that is hard to fake.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Sports bettors have a vocabulary for the moment when two books move in the same direction at the same time, and they have it for a reason: steam is the closest thing to a guaranteed signal that exists in pari-mutuel sports. When the line on a Sunday NFL game moves from -3 to -3.5 on Pinnacle and DraftKings within the same five-minute window, that is steam. It means a sharp player (or a syndicate) just put real money on the same side at multiple venues, and the books are responding to the same flow.

Steam works as a signal because it is *expensive to fake*. Moving one book costs you a little bit of capital. Moving two books in the same five-minute window costs you twice as much capital, and there is no easy way to do it without actually putting on the position. The signal is informative because the cost of generating it is roughly equal to the cost of acting on it.

Prediction markets have steam too. This essay is about how to detect it, why it is the highest-quality cross-venue signal that exists, and how to trade around it without getting whiplashed by the noisy version.

## The PM Definition of Steam

A steam move on prediction markets is *the same outcome moving in the same direction on both Kalshi and Polymarket within a five-minute window, with the move on each venue exceeding a venue-specific noise threshold, and with the post-move spread on both venues remaining tight*.

That last clause is the one new traders skip and it is the most important. A move that shows up on both venues but blows out the spread on one of them is *not* steam — it is one venue catching up to the other after a delayed maker reaction, which is a different pattern. Real steam keeps the books tight on both sides because the makers are absorbing real flow without panicking.

The five-minute window is conservative. On a fast-moving outcome (election night, breaking geopolitical news), real steam can hit both books inside thirty seconds. On a slow market (long-tenor economic indicators), the propagation can take fifteen minutes. The right window depends on the latency of the slowest sharp player who would plausibly be hitting both books, and that latency in turn depends on whether the venues are operationally connected for the trader (most are not — you have to log into Kalshi separately from Polymarket and the workflow is meaningfully slower).

## Why Cross-Venue Steam Is the Highest-Quality Signal

Three reasons.

**One — it is hard to fake.** Faking a single-venue move requires putting on a position equal to the displayed move size. Faking a cross-venue move requires putting on the position *twice*, which doubles your capital exposure for the same display effect. For a sophisticated faker the math of "is this fake worth doing" gets a lot worse the moment you have to mirror the trade on a second book. Most of the noise in single-venue tape is from one trader fishing for momentum; cross-venue steam survives that filter because the cost of fishing is too high.

**Two — it requires shared information.** A genuine steam move means *the same trader (or coordinated traders) acted on the same information at both venues*. That information has to be real enough to motivate paying the spread on both books, which filters out a lot of the noisy thesis-chasing. Steam carries an implicit information richness that single-venue moves do not.

**Three — it is observable in real time without model.** The other high-quality cross-venue signal — convergence after divergence — is a slow phenomenon that you have to compute over an hour or more. Steam is fast (5-minute window) and the detector is a one-line query: "show me outcomes where Kalshi mid moved by more than 2 cents and Polymarket mid moved by more than 2 cents in the same direction in the last 5 minutes, with both spreads still under 4 cents." That query runs cheap, runs continuous, and returns hits that are usually actionable.

By contrast, single-venue moves require model context to interpret. A 4-cent move on Kalshi alone might be noise, might be a sharp, might be a maker recalibration, might be a fat finger. You cannot tell from the move itself; you need the surrounding tape and an opinion about the trader population. Cross-venue steam removes most of those alternative explanations because they do not coordinate across venues.

## Detecting Steam in Practice

The detector I use is simple. For every outcome that has a Kalshi listing and a Polymarket listing (the cross-venue universe is on the order of 2,000 outcomes at any time), compute a 5-minute price delta on each venue. Filter for cases where both deltas have the same sign and both exceed 2 cents in magnitude. Then layer a spread check: the post-move spread on both venues must be under 4 cents (i.e., the makers did not panic).

The remaining hits are candidate steam events. On a typical day this returns somewhere between zero and twelve events. Half of those are real steam (the kind you should follow), and the other half are noise — usually one venue updating to match the other after a minute of lag, which technically passes the filter but is not "two traders acting on the same information." The way I distinguish the two: on real steam, the *taker volume on both venues* in the 5-minute window is meaningfully above baseline. On the lag pattern, only one venue has elevated taker volume and the other is just maker quote drift.

A worked example. Last week I caught a steam event on a Polymarket / Kalshi pair on the same recession-2026 outcome. The Polymarket mid moved from 0.31 to 0.36 in about 90 seconds; the Kalshi mid moved from 0.33 to 0.37 in about 4 minutes. Both spreads stayed under 3 cents on both venues. Taker volume on Polymarket in that window was about 8x baseline; on Kalshi about 5x baseline. That is real steam — there was new information about the macro outlook (a credible recession call from a major bank had just dropped on the wire), and sharp traders were positioning at both venues simultaneously.

The right play in that situation was to *follow the steam* — get in on whichever venue had the better current spread, in the same direction as the move. I was on Kalshi for that one and the entry was at about 0.365. Two days later both contracts had drifted to about 0.41, which was a clean win on a low-effort signal.

## Why You Should Follow Steam, Not Fade It

The temptation when a market moves fast is to fade — to sell into the buying or buy into the selling — because the move "looks overdone." That instinct is correct for a lot of single-venue moves, especially the ones driven by retail momentum. It is *wrong* for steam moves because the cost-to-fake argument means the people generating the move have skin in the game.

Faders survive on single-venue noise. Followers survive on cross-venue steam. The two strategies look superficially similar (you are reacting to a price move) but they are mathematically opposite, and the difference is whether the move was generated by one trader or by multiple traders at multiple venues.

The sports betting heuristic, copy-pasted: *steam is for following*. Sharps move books because they have information; if you are not the sharp, the right play is to recognize that and ride along. The bad outcome is that the steam was wrong (information turned out to be false) and you lose with the sharps. The much worse outcome is that you faded the steam, the sharps were right, and you got run over because you were on the opposite side of an informed trade.

## The Two Failure Modes

Steam detection has two failure modes that I have learned about the hard way.

**Failure mode one — false steam from latency.** Sometimes Kalshi and Polymarket move "together" not because two traders acted on shared information but because one venue updated and a maker on the other venue manually mirrored the update. This produces a pattern that looks like steam — both venues move in the same direction in the same window — but the *causation* is one-way (venue A → maker on venue B), not two-traders-on-shared-information. The taker-volume check usually catches it: if only one venue has elevated taker volume, the other is mirroring, not absorbing real flow.

**Failure mode two — false steam from a single large trader at two venues.** A single trader who happens to have accounts at both Kalshi and Polymarket can generate a "cross-venue" move single-handedly by hitting both books in sequence. The detector will register this as steam because the criteria are mechanical, but the underlying signal is one trader, not many. This is rarer than failure mode one, but harder to filter because the taker volume check passes (real volume on both venues, just from the same source). The defense is to look at the *number of distinct trades* in the window, not just the dollar volume. A single trader generates a small number of large fills; a steam event from multiple traders generates a larger number of mid-sized fills.

## Where Steam Detection Breaks

Beyond the two false-positive modes, there are three structural cases where steam detection does not apply.

**Outcomes that exist on only one venue.** A Kalshi-exclusive contract (most KX-prefixed weather contracts) has no Polymarket counterpart, so there is no cross-venue check to run. Steam is undefined for these. You are stuck with single-venue analysis.

**Outcomes where the venues use different resolution criteria.** "Will Powell stay Fed Chair through 2026" can resolve differently on Kalshi vs Polymarket if one venue settles based on the official appointment letter and the other settles based on the inauguration date. When the resolution rules differ, the same outcome is technically two different contracts, and "steam across venues" is not a meaningful concept because the contracts are not the same instrument. See [the resolution-ambiguity-score](/concepts/resolution-ambiguity-score) page for how to detect these mismatches.

**Markets in the middle of a UMA dispute.** Polymarket contracts that are actively in dispute can show wild price moves that are not reflected on Kalshi (which has no UMA dependency). These look like one-sided steam but are actually venue-specific resolution risk pricing in. Filter out any Polymarket contract that is within the UMA challenge window before running the steam detector.

## How This Connects to the Rest of the Stack

Steam moves are the highest-frequency form of cross-venue signal. The lower-frequency form is convergence-divergence dynamics covered in [cross-venue-convergence](/concepts/cross-venue-convergence). The two together cover most of the cross-venue work that matters. For the implementation, see [cross-venue-edge-detection-kalshi-polymarket](/technicals/cross-venue-edge-detection-kalshi-polymarket), which has the working code for the kind of detector this essay describes.

The steam concept also intersects with [catalyst-driven-spread-compression](/concepts/catalyst-driven-spread-compression): in the minutes after a catalyst, you often see a coordinated steam move on both venues as sharp traders position around the new information. Watching for both signals together — pre-catalyst compression + post-catalyst steam — is how you get a high-confidence read on what just happened and what the market thinks about it.

For the philosophical case that endogenous market data (which is what steam is, fundamentally) only becomes a signal when triangulated against the other data axes, see [endogenous-vs-reality-vs-opinion-data](/concepts/endogenous-vs-reality-vs-opinion-data).

The habit to build is this: after every major news event, run `sf scan --steam --window 5m` and look at what the cross-venue tape is saying. The names that come back are the contracts where money has moved in the last five minutes. Half the time you do not have a thesis on those contracts. The other half, the steam is telling you exactly which thesis just got validated by people with skin in the game.