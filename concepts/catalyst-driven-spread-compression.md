# Catalyst-Driven Spread Compression: Reading the Tape Before News

> Before any scheduled catalyst — Fed announcement, jobs report, election night — the spreads on the relevant prediction markets compress in a predictable shape. Knowing the shape lets you front-run volatility and avoid being caught flat-footed.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Every scheduled catalyst leaves a fingerprint on the orderbook before it lands. The shape is predictable, the timing is predictable, and most retail traders never learn to read it because they are watching the mid-price instead of the spread. This essay is about the spread, not the price — and about what the spread tells you in the hour before a Fed decision or a BLS print that the price will not say until ten minutes after the headline.

The phenomenon I want to name is *catalyst-driven spread compression*. It is the second-most-useful microstructure pattern I know on prediction markets, after [the settlement halo](/concepts/settlement-halo). It is also the easiest one to read off a screen, because the only data you need is the bid and ask, and both venues publish those for free.

## What the Pattern Looks Like

Take a Kalshi Fed-decision contract on the morning of an FOMC announcement. The decision lands at 2:00 PM ET. At 9:30 AM ET — four and a half hours out — the contract has its normal trading-day spread, which on a typical Fed contract is about 5 cents wide. By noon the spread is 3 cents. By 1:00 PM it is 2 cents. By 1:45 PM it is 1 cent or tighter. The orderbook depth on each side has roughly doubled at every step.

Then at 2:00:00 PM the spread blows out — 8 cents, 12 cents, sometimes more — and the depth on both sides evaporates as makers pull every quote within a hundred milliseconds of the headline. By 2:02 PM the new spread settles in at maybe 3 cents around a new mid that is twenty cents from where it started. By 2:30 PM the spread is back to where it was at 9:30 AM and trading is normal again.

That whole arc — the gradual compression in the four hours before the catalyst, the violent blowout at the moment of the print, the rapid normalization in the next twenty minutes — is the catalyst-driven spread compression pattern. I have seen the same shape on every Fed contract I have ever watched, on every NFP-print contract, on every CPI contract, and on every election-night contract. The amplitude varies. The shape does not.

## Why Spreads Compress Before News

The compression has two mechanical drivers and one game-theoretic one.

The first mechanical driver is *positioning flow*. As a known catalyst approaches, traders who have a thesis on the outcome want to be in the market, and they want to be in *cleanly* — meaning they want a tight spread to enter against. That demand pulls makers in. Each maker who sees inbound flow tightens their quote slightly to capture more of it. The spread compresses incrementally as the book gets contested. You can watch this happen in real time on the [marketRegimeSnapshots](/concepts/maker-taker-regime-in-pms) data: the spreadCents field on a typical Fed contract walks down monotonically from 9:30 AM through about 1:50 PM.

The second mechanical driver is *taker exhaustion*. The traders who wanted to take liquidity early — the morning crowd, the news-driven entries — have already done so by midday. The remaining flow in the final hour is increasingly maker-on-maker, which is to say tight, small, and polite. There is no big takeout pressure to widen the book against, so the equilibrium spread keeps drifting tighter.

The game-theoretic driver is the more interesting one. Every maker in the book knows the catalyst is at 2:00 PM. They also know that at 2:00:00.500 PM their quote becomes adverse-selected against by the news itself — anyone who reads the headline a hundred milliseconds before they can pull will pick them off. So makers face a choice: keep tight quotes up until the announcement (and accept the pickoff risk), or pull the quotes a few seconds before the announcement (and miss the last few minutes of pre-news flow). Most makers split the difference by staying tight until ~1:55 PM and then pulling abruptly.

That coordinated pull is what creates the *blowout* at the moment of the news. It is not random; it is the equilibrium response of every maker doing the same expected-value math at the same time. The spread does not gradually widen — it snaps from "1 cent" to "10 cents" in about a hundred milliseconds, because every maker pulls within the same one-second window.

## Reading the Compression Off the Screen

Once you know the shape, you can read the live tape and tell exactly where you are in the cycle. I run `sf scan --by-spread asc --filter catalyst-window` about an hour before any scheduled print, and the scan returns the contracts whose spread has compressed the most in the previous two hours. That set is almost always the contracts the market has identified as "the ones that will move." Sometimes it surfaces a contract I did not expect — a sibling market in the same event family, or a Polymarket contract on the same outcome that I had not been watching.

The comparison I find most useful is the spread *trajectory*, not the absolute spread. A contract that has gone from 5 cents to 1 cent in three hours is more informative than a contract that has been at 1 cent all morning. The trajectory says "this is the contract makers are competing for right now," which is the leading indicator that real money is positioning. The marketRegimeSnapshots table stores enough history to compute that trajectory cheaply.

A worked example. Last NFP morning I watched two Kalshi contracts on payroll outcomes side by side. Contract A — "non-farm payrolls > 200K" — opened at a 4 cent spread and tightened to 1.5 cents by 7:45 AM ET (15 minutes before the print). Contract B — "non-farm payrolls > 250K" — opened at a 6 cent spread and was still at 5 cents at 7:45 AM. The compression on A was the market saying "this is the strike that matters this month." The flat spread on B was the market saying "no one cares about the upside tail." The print landed at 215K. The 200K contract moved violently from 0.55 to 0.78 in the first 30 seconds; the 250K contract moved from 0.18 to 0.22 and then stayed there. Compression predicted which contract would carry the action.

## Three Trading Implications

The pattern is not just descriptive — it carries three concrete implications for how to trade around scheduled catalysts.

**One — compression is a leading indicator of which contracts will move.** If you are looking at five sibling contracts in the same event family and the spread on one of them is compressing dramatically while the others stay wide, that is the contract the market has decided is the "fulcrum" of the catalyst. Allocate attention there. Your indicator scan will not pick it up because indicator scans (IY, CRI) are price-based; the compression signal is in the spread, which lives one layer down.

**Two — entries are cheaper in the compression window than after the news.** If you have a thesis on the catalyst and you want to be in the market, the right entry window is roughly 60 to 30 minutes before the announcement. The spread is tight, the book is deep, and the maker pickoff hasn't started yet. Entries inside the final 15 minutes are riskier because the makers are about to pull and you are essentially racing them. Entries after the news has already landed pay the post-blowout spread, which is wider than the compression-window spread by a factor of 5 to 10.

**Three — flat spreads in the catalyst window are themselves a signal.** If a contract you expected to be in the catalyst window is *not* compressing, the market is telling you it does not see the catalyst as relevant to that contract. Either your thesis about the relevance is wrong, or the contract is so illiquid that nobody is positioning around the catalyst at all. Both are useful information. The flat-spread contract is *not* the one you want to be in for a catalyst trade — but it might be the one you want to make markets in afterward, because the post-catalyst flow will be one-sided and predictable.

## Where the Pattern Breaks

A few honest caveats.

**Unscheduled catalysts do not compress.** The whole pattern depends on the catalyst being on a known calendar — FOMC, NFP, CPI, election night, scheduled court rulings. Unscheduled events (a Fed emergency cut, a surprise resignation, a flash crash) do not get pre-positioned, so there is no compression to read. The compression signal lives on the *known unknowns*, not the unknown unknowns.

**Thinly-traded markets do not show the pattern cleanly.** A Kalshi contract with $200 of total daily volume is not getting maker competition before any catalyst. The spread will look the same all day because there is one maker camping a wide quote and nobody contesting it. The pattern requires at least Tier B liquidity (LAS in the warm-cron-covered range) to be readable. On illiquid contracts, the spread is just whatever the lone maker decided to set this morning.

**Cross-venue compression is asymmetric.** Kalshi compresses harder than Polymarket on US-regulated catalysts, because the Kalshi maker pool is more sophisticated and competes more aggressively. Polymarket compresses harder on global political events that the Kalshi user base does not follow as closely. If you are tracking compression as a signal, do it per-venue, not pooled.

**The compression can be faked.** I have seen one instance where a single maker walked down their own spread quote across the morning to *create* the appearance of compression, presumably to bait other makers into following. The bait failed because the depth on the other side did not move (which is the giveaway — real compression has both sides thickening). If you see compression with one-sided depth, treat it as suspicious. Real compression is two-sided.

## How This Connects to the Rest of the Stack

Catalyst-driven spread compression is the *intra-day* analog of the [settlement halo](/concepts/settlement-halo), which is the *final-24-hours* analog of the same idea. Both are microstructure phenomena that happen because makers and takers update their behavior in response to a known event approaching. Both are readable from the [marketRegimeSnapshots](/concepts/maker-taker-regime-in-pms) data with no model.

The compression signal is also adjacent to [steam moves across venues](/concepts/steam-moves-across-venues): when compression on Kalshi *and* Polymarket happens simultaneously on the same outcome, that is the cross-venue version of the same coordinated behavior, and it is one of the highest-quality signals on the entire screen.

For the underlying methodology — why "spreads compress when something is about to happen" is a real phenomenon and not just a folk theory — see the [pm-indicator-stack](/concepts/pm-indicator-stack) page on why microstructure indicators matter, and [orderbooks-are-fossilized-beliefs](/blog/orderbooks-are-fossilized-beliefs) for the broader case that the orderbook tells you more than the headline price.

The thing to take away from this essay is the habit shift: when a catalyst is on your calendar, watch the spread for the four hours before it, not the price. The price will tell you what already happened. The spread will tell you what is about to.