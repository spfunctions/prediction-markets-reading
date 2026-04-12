# Liquidity Migration Across Resolution: Where the Money Goes When a Market Closes

> When a high-volume prediction market resolves, the capital that was in it has to go somewhere. The migration patterns are predictable enough that the receiving markets are often a better trade than the resolving one was.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
A presidential-election market resolves on a Tuesday in November. By Wednesday morning the contract is settled, the trades have cleared, and the traders who held positions are now sitting on cash. That cash does not go back into a savings account. It goes into the next market that looks like it scratches the same itch — the same urgency, the same scale, the same kind of edge.

I have watched this happen every cycle for the last three years, and it is reliable enough to build a strategy around. When a flagship market closes, the secondary markets in the same family see a noticeable bump in volume and depth within 24-48 hours. The mid prices move differently because the new entrants are *already calibrated* — they had been scanning the secondary markets while their primary capital was tied up — and they enter with conviction.

This page is about the migration patterns, how to detect them, and why the migration markets are often a better trade than the flagship was.

## The Basic Pattern

Capital moves in three observable ways after a flagship market resolves.

**Migration type 1 — same-category next-period.** The most common case. Election markets close after Election Day; the capital flows to the next election cycle (gubernatorial races, midterms, the next presidential cycle). Fed-decision markets close after the meeting; the capital flows to the next FOMC meeting's contract. NFL playoff contracts close after the championship; the capital flows to the next season's futures. The receiving market is structurally identical to the closed one — same category, same kind of question — just one cycle ahead.

**Migration type 2 — adjacent-category capital reallocation.** Less common but interesting. A flagship resolves and the freed capital goes to a *different* category that shares the same audience. Election capital sometimes flows to economic-indicator markets because political traders are also macro-curious. Crypto-event capital flows to AI-event capital because the audiences overlap. The receiving category sees a temporary volume bump that lasts 1-3 weeks.

**Migration type 3 — exit to cash.** The traders who held the flagship position decide they are done with prediction markets for now and pull their capital off the venue entirely. This shows up as a temporary depth reduction across many markets simultaneously, lasting 2-4 weeks before the audience rebuilds. It is the migration pattern you most need to know about because it is the one where *the markets you would normally trade get worse, not better*, and the strategies that depended on the old depth stop working.

The relative mix of the three migrations varies by venue and event type. On Kalshi, after the 2024 election, I logged about 60% type 1, 25% type 2, and 15% type 3 over the first month. On Polymarket the type-3 share was higher because the political audience there was more committed to the single flagship and less interested in adjacent contracts.

## Why Migration Is Predictable

The reason this pattern works is that prediction-market traders are *category-loyal*. Most of them have a set of categories they understand — politics, sports, economics, crypto, geopolitics — and they trade within those categories. When their primary market in a category closes, they look for the next thing in the same category before they look at anything else. The migration is mechanical because the alternative (re-learning a new category from scratch) is expensive in time and attention.

The other reason is that the receiving market is *already on the trader's screen*. A trader who held a $5K position in the November Fed contract is also watching the December and January contracts as a courtesy — they are in the same dashboard, the same scan, the same mental model. When the November position resolves, the December contract is the path of least resistance for the freed capital.

This is why the [cycle-clustering](/learn/cycle-clustering) grouper matters so much for migration detection. The grouper is what tells you which markets are in the same family as the one that just closed. A trader who is loyal to the "Fed decisions" category will migrate to other contracts in the Fed-decisions cluster; the cluster is the unit of loyalty. The grouper covers about 41% of event families cleanly, which means you can predict migration for 41% of resolutions and have to manually identify the destination for the rest.

## How to Detect Migration in Real Time

The migration shows up in two indicators on the [pm-indicator-stack](/concepts/pm-indicator-stack), with a lag of 24-72 hours after the flagship resolves.

**Indicator 1 — depth on receiving markets.** The bid-ask depth on the next-period contract in the same family will increase noticeably. I usually see depth go from 100-500 cents pre-migration to 800-2000 cents post-migration on the receiving contract. This shows up as an increase in the [liquidity-availability-score](/learn/liquidity-availability-score) for that contract on the next warm-cron pass — usually within 6 hours of the migration starting.

**Indicator 2 — price drift with tight spreads.** The receiving market's mid price will start drifting in whichever direction the migrated capital is biased toward. The drift is usually small (1-3 cents) but it happens with *tight spreads*, because the new entrants are sized and patient and they are not crossing the spread aggressively. The combination of moving mid + tight spreads is the signature of informed flow, which is exactly what migration is.

The detection workflow I use: every Wednesday morning, I run `sf scan --recently-resolved --within 7d` to get the list of high-volume markets that closed in the last week. For each one, I look up the cluster siblings via the cycle-clustering grouper. For each sibling, I pull the LAS and the price velocity. I sort by the LAS *increase* over the last week. The top of that sort is the migration destinations.

On a typical week the top of the sort has 3-5 contracts that are seeing real migration flow. They are usually next-period versions of the just-resolved flagship plus 1-2 adjacent-category contracts that share the same audience. The price action on these contracts in the following week is, in my experience, more predictable than the typical contract because the flow is informed and patient.

## A Worked Migration Trade

Here is a real-shape walkthrough. The flagship is the November 2024 presidential election contract, which I will call POLY-PRES-2024 for shorthand. It resolved on November 5-6, 2024.

Wednesday Nov 6, 8:00 AM ET: Flagship resolved overnight. I run my migration scan. The cluster grouper has the following sibling families on Polymarket:
- Senate races (about 20 contracts, mostly already resolved)
- Gubernatorial 2025 (3 contracts, lightly traded pre-migration)
- 2028 Democratic nomination (1 multi-outcome contract, modestly traded)
- 2028 Republican nomination (1 multi-outcome contract, modestly traded)
- Midterms 2026 (2 contracts, lightly traded)

Of these, the two 2028 nomination markets are the most plausible migration destinations because they are the same category (presidential politics), the same scale (multi-million-dollar potential volume), and the same kind of question (who wins).

Thursday Nov 7: I check the LAS on both 2028 contracts. The Democratic nomination contract has gone from LAS 12 to LAS 47 in 36 hours — a 4x jump. The Republican nomination contract has gone from LAS 9 to LAS 34, also a 4x jump. The depth is real and the spreads have tightened. Migration is confirmed.

I do not have a strong directional thesis on either 2028 contract yet. What I have is the *structural* observation that these contracts now have meaningful depth and meaningful flow, which means stage 2 of the [valuation funnel](/concepts/the-valuation-funnel) is now passable on them in a way it was not a week ago. I add both to my watchlist for stage 3 evaluation. Over the following week I build small positions on the candidates I have a thesis for, knowing that the depth will support exits at reasonable slippage.

Friday Nov 15: One week after the flagship resolved. The Democratic nomination contract has seen the YES on Newsom drift from 0.32 to 0.36 with a 3-cent spread the whole way. The Republican nomination contract has seen the YES on a few candidates move similarly. The migrated capital is establishing positions, and the price action reflects sized, patient flow rather than emotional bursts.

The trades I put on in the first week of migration outperformed the trades I put on in the same contracts a month later, after the migration had stabilized and the contracts were back to their normal pre-migration flow rates. The migration window is a structural opportunity that closes when the new equilibrium is reached.

## Migration Type 3 — The Bad Case

Type 3 migration is the one that hurts you. After a flagship resolves, sometimes the audience just leaves. Polymarket saw this after the 2022 midterms — the political audience was exhausted, the next-period contracts were too far in the future to be interesting, and the venue's overall depth dropped by something like 30% for about a month.

The diagnostic for type 3 migration is: the LAS on the *next-period sibling* does not increase. In fact, it sometimes decreases, because the makers who were camped on those contracts leave too (their flow has dried up along with everyone else's). The cluster grouper looks for migration destinations and finds nothing. The recently-resolved scan returns lots of dead clusters in a row.

When I see type 3 migration patterns, I do two things:

1. **Pull the strategies that depended on the old depth.** Range-MM strategies, virgin-Polymarket maker strategies, and cross-venue convergence strategies all degrade when the underlying depth drops. They do not stop working entirely, but their expected value gets cut in half and the variance triples. Better to scale them down and wait.
2. **Look for re-entry signals on the next migration cycle.** Type 3 migrations end when a new flagship-class market lists and starts attracting attention. The next presidential cycle, the next major sports event, the next viral economic contract. Track the new flagships and resume the strategies when one of them starts pulling depth back into the venue.

Type 3 is the migration pattern that most traders miss because they are looking for *opportunities*, not *risks*. The opportunity is to scale down before the depth drop hurts you, which is harder to value than scaling up before a known good trade.

## Where the Framework Breaks

A few honest caveats.

**Migration takes 24-72 hours, sometimes longer.** The detection lag is real. If you wait until the LAS jump is unambiguous, you have already missed some of the earliest entries. But if you trade on a 12-hour LAS bump, you are sometimes trading on noise. The right tradeoff depends on your appetite for false positives and your position-sizing tolerance.

**Cluster mis-grouping affects migration detection.** The cycle-clustering grouper covers about 41% of event families cleanly. The other 59% have either incomplete clusters (siblings that the regex missed) or bad clusters (unrelated contracts grouped together). Migration detection inherits this 41% ceiling. For the events the grouper does not catch, you have to identify the migration destinations manually, which is slower and less reliable.

**Not every flagship has a clean next-period sibling.** Some flagships are one-time events with no successor. The 2024 SVB-collapse contract had no obvious next-period version. After it resolved, the capital dispersed across many small adjacent contracts rather than concentrating in a single migration destination. The framework does not help much in these cases — you have to track the dispersal manually if you want to follow the capital.

**Migration is not a directional signal.** Knowing that capital is flowing into a contract does not tell you which side of the contract the capital is taking. The detection framework tells you which contracts are about to have *more flow*. The directional question — what side that flow is taking — still needs stage 3 thinking. Treat migration as a signal that *attention* is about to move, not as a signal that *YES* is about to be the right side.

## How This Connects to the Stack

Migration is one of the few cases where the [pm-indicator-stack](/concepts/pm-indicator-stack) Tier B indicators (LAS specifically) are doing the most useful work. The Tier A indicators (IY, CRI, EE) tell you about static contract properties. LAS tells you about *flow*, and flow is what migration is. The reason LAS matters here even though it is null on 99% of markets is that the ones it is *not* null on are exactly the ones that just received a migration — the warm cron picks up the volume bump on the next pass and the LAS becomes computable.

For the cluster identification, see the [cycle-clustering](/learn/cycle-clustering) glossary entry and the [the-cyc-regex-grouper-walkthrough](/technicals/the-cyc-regex-grouper-walkthrough) technical. Migration detection is downstream of correct cluster identification — if the grouper misses the family, the migration destination is invisible to the scan.

For the philosophy, see [liquidity-availability-as-the-real-edge](/opinions/liquidity-availability-as-the-real-edge), which argues that *whether you can trade* is more important than *what the math says you should trade*. Migration is exactly the case where a market that was untradable yesterday becomes tradable today, purely because flow has arrived. That is the reverse of the usual liquidity decay story and it is one of the more reliable structural patterns in the market.

The migration framework does not make me money on every flagship resolution. It makes me money about half the time, because the other half are either type 3 (no destination) or the destination is in a cluster the grouper missed. But the half it works on is reliable enough that I always run the scan on Wednesday mornings, and the times I have skipped it are the times I have noticed I missed an obvious entry a week later.