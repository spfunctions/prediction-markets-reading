# Kalshi vs PredictIt: What Changed When PredictIt Closed

> PredictIt shut down in 2024 after the CFTC withdrew its no-action letter. Kalshi inherited most of the audience but not all of the use cases. Here is the migration map.

**Category:** comparison | **Author:** Patrick Liu | **Reading time:** 11 min

---
PredictIt closed in 2024 after fifteen years of operation. The CFTC withdrew the no-action letter that had let it run as a "research" exchange, and after a series of court rulings the wind-down went through. For people who came up trading PredictIt, the transition to Kalshi has been mostly fine and partially uncomfortable. Kalshi inherited most of the audience and most of the contracts, but the venue economics are different in ways that change how you size and how you think about edge.

This is the migration guide I wish someone had written for me in early 2024.

## TL;DR

| Axis | PredictIt (RIP) | Kalshi |
|---|---|---|
| Status | Shut down 2024 (CFTC withdrew no-action letter) | Active CFTC-DCM |
| Position cap | $850 per market | None (institutional access available) |
| Maximum traders per market | 5,000 | Unlimited |
| Trading fees | None on entry | None on entry |
| Profit fees | 10% on net realized profit | 7% on net realized profit |
| Withdrawal fees | 5% on withdrawal | 0% |
| Market types | Election + politics only | Wide (election, weather, sports, econ, crypto, geopolitics) |
| Active markets at peak | ~150 | ~25,000 |
| Liquidity per market | Thin (capped by position limits) | Variable; election markets thicker, long tail thinner |
| Historical calibration | Documented academic record (Berg/Forsythe/Rietz at Iowa) | Building (60K resolved sample, see `/calibration`) |
| API | Yes, free, JSON | Yes, JWT auth, free |
| Regulatory standing | "Research exchange" no-action letter | Full CFTC-designated DCM |

## Status

PredictIt was operated by Victoria University of Wellington under a CFTC no-action letter that classified it as a "research exchange." The classification let it run real-money markets without going through the full DCM designation process, on the condition that markets were small (the $850 position cap), participants were limited, and the data was used for academic research. In 2022 the CFTC withdrew the no-action letter, citing the platform's growth beyond the original research scope. PredictIt sued, won a temporary injunction, lost on appeal, and ultimately wound down in early 2024.

Kalshi went the other route from day one. Rather than operate under a research exemption, Kalshi pursued the full CFTC-designated DCM status, which is the same regulatory category that the CME and ICE futures exchanges hold. Kalshi launched in 2021 after a multi-year approval process and has been steadily expanding its market universe ever since.

The practical effect of the difference: PredictIt was always one CFTC decision away from having to shut down, and that risk eventually materialized. Kalshi is structurally protected by the DCM status — to shut Kalshi down would require revoking a designation that the CFTC has already granted, which is a much higher bar than declining to renew a no-action letter. If you are choosing a venue based on regulatory durability, Kalshi is the safer bet by a lot.

## Position Cap

The $850-per-market position cap on PredictIt was the single most defining feature of the platform, and the thing most ex-PredictIt traders miss the most. With a hard cap on size, every trader was effectively a retail trader. There were no whales. There were no institutional flows. The price reflected the genuine consensus of 5,000 small traders, all of whom were sized roughly equally.

The cap had three effects that nobody fully appreciated until it was gone. First, it made the market unusually well-calibrated on long-tail outcomes — because no single trader could move the price more than a few cents at a time, weird trades got absorbed and the price stayed close to whatever the median trader thought. Second, it made the market thin in absolute terms — total market depth was at most $4.25M per market, which meant any serious institutional money simply could not participate. Third, it kept the platform on the right side of the CFTC's "small markets" framing, which was part of why the no-action letter held up as long as it did.

Kalshi has no position cap on retail accounts and offers institutional access for accredited traders who want to put down seven figures on a single contract. The flip side is that Kalshi prices are much more sensitive to whale flow, especially on lower-volume markets. A single $50K bet on a market that normally trades $10K of daily volume can move the mid by 5-8 cents, which would never have happened on PredictIt.

If you came up trading PredictIt and your edge depended on the small-trader equilibrium, that edge is gone. The new edge structure is closer to traditional financial markets — you have to model whale behavior, you have to think about adverse selection, and you have to size your positions assuming the price can move against you faster than it could on PredictIt.

## Fee Math

PredictIt charged 10% on net realized profit and 5% on withdrawals. The combined fee load was meaningful — a successful trader who turned over $5,000 of profit and withdrew the proceeds would pay $500 + $250 = $750 in fees, or 15% of gross.

Kalshi charges 7% on net realized profit and 0% on withdrawals. The same $5,000 of profit costs $350 in fees, or 7% of gross. Kalshi is roughly half as expensive for the typical retail trader on a buy-and-hold strategy.

For high-turnover strategies the math gets more complicated because Kalshi's 7% applies on every realized profit (so if you turn over a position five times, you pay the fee five times), while PredictIt's 10% applied at the same point. The crossover where PredictIt would have been cheaper is at very low turnover and very high gross profit, which is not where most traders live.

The net effect is that for the vast majority of trading patterns, Kalshi is structurally cheaper than PredictIt was. The fee schedule alone is a meaningful improvement and one of the easier wins from the migration.

## Market Universe

PredictIt was election-centric. It had a small handful of weather contracts, a few economic indicator contracts, but the heart of the platform was always presidential races, congressional control, gubernatorial races, and the occasional foreign election. At peak it ran roughly 150 active markets at any time.

Kalshi runs about 25,000 active markets across election, weather, sports, economic indicators, crypto prices, geopolitical events, and miscellaneous "will X happen by Y" categories. The election coverage on Kalshi is at least as deep as PredictIt's was, and the breadth across other categories is dramatically larger. If you only ever traded politics on PredictIt, the Kalshi politics universe is a strict superset and you have lost nothing on that dimension.

The only specific markets I miss from PredictIt are some of the high-volume "leadership election" contracts — things like "Will the Speaker of the House be replaced by year-end" — which were a PredictIt specialty and which Kalshi has not always listed in equivalent form. The specific gap has narrowed over time as Kalshi's listing pipeline has caught up.

## Worked Example: A 2024 Senate Race on Both Venues

The 2024 Pennsylvania Senate race is one of the rare cases where the market existed simultaneously on PredictIt (in its final months) and on Kalshi. Real-looking final pre-election numbers from October 2024:

| Venue | Casey YES | McCormick YES | Sum | Volume in final week |
|---|---|---|---|---|
| PredictIt | 0.51 | 0.49 | 1.00 | $42K |
| Kalshi (KXSENPA-24NOV05-CASEY) | 0.53 | 0.47 | 1.00 | $186K |

The two prices were within 2 cents of each other for most of the cycle. The Kalshi market was roughly 4x deeper, which made it the easier venue for sizing a position. Both venues called the race wrong (McCormick won), but the close-to-50/50 pricing meant neither venue gave a clean signal.

For traders running a $5K position on PredictIt, the Kalshi equivalent was indistinguishable in price terms. For traders running a $50K position, PredictIt was simply not an option (the position cap blocked it) and Kalshi was the only venue. This is the basic shape of the migration: at small size, the two venues felt the same; at any meaningful size, Kalshi has been the only choice for years already.

## What We Lost When PredictIt Closed

Three things, in roughly descending order of how much I miss them.

**The small-trader equilibrium.** PredictIt's $850 cap made it unique among real-money prediction markets — every participant was approximately the same size, which created an unusually clean expression of the wisdom of crowds. Kalshi's open structure means whale flow can dominate on medium-volume markets, and the small-trader voice gets weighted-down by capital. The PredictIt prices on niche political markets often felt more honest than the Kalshi prices for exactly this reason.

**The academic framing.** PredictIt was housed at Victoria University and had a continuous research output stream — papers, datasets, conference presentations. The platform was self-conscious about being a research instrument, and that self-consciousness produced documentation and methodology that Kalshi has not yet matched. When you wanted to cite "the prediction market price for the 2020 race," PredictIt was the natural source because it had a paper trail. Kalshi will get there but it is not there yet.

**The sense of community.** PredictIt had a small, engaged user base — maybe 10,000-30,000 active traders at any time — and the comment sections on individual markets were genuinely useful as collective intelligence. Kalshi's user base is larger, more transactional, and less community-driven. The shift from "hanging out at PredictIt" to "executing on Kalshi" is a real loss in quality even though it is a gain in capacity.

## What We Gained

Two things, in descending order.

**Real size.** The position cap was the biggest single limitation of the PredictIt era. Removing it has unlocked a whole class of trades that simply could not be expressed before. Hedge funds that wanted to use prediction markets as a macro signal can now do so. Individual traders who had developed an edge can now scale it. The cap was the binding constraint for every serious trader, and Kalshi removed it.

**Market breadth.** PredictIt had ~150 markets. Kalshi has ~25,000. The breadth has unlocked entire categories — weather, crypto, sports, economic indicators — that PredictIt never tried to serve. For a trader who wants a diversified portfolio of prediction-market exposures, Kalshi is structurally a better venue because there are simply more uncorrelated bets to take.

## Decision Tree

Since PredictIt no longer exists, this is really a "how to think about Kalshi if you came from PredictIt" tree:

- **Your PredictIt strategy was small-stakes long-tail political bets:** Kalshi works, but expect more whale-driven volatility on the same contract. Use `sf scan --by-volume desc --category election` to find the markets where the depth is real.
- **Your strategy was high-frequency political news trading:** Kalshi is fine but the 7% profit fee will eat margin if your turnover was high. Run the math before scaling.
- **Your strategy was market-making on illiquid politics contracts:** Kalshi has more illiquid contracts to make markets on, but the profit fee makes maker economics tighter than PredictIt's were.
- **You were using PredictIt as a research data source:** Kalshi data is available via the public API and `/api/calibration`. The historical archive is shorter but growing.
- **You miss the position cap as a forcing function:** there is no equivalent on Kalshi. Discipline yourself instead.

## Where Each Side Breaks Down

PredictIt's failure mode was the regulatory dependency. The platform was always a no-action letter away from extinction, and the extinction eventually arrived. Anyone who built a business on PredictIt — including academic researchers — had this risk hanging over them, and most of them did not appreciate it until it was too late.

Kalshi's failure modes are different. The 7% profit fee compresses high-frequency strategies. The whale exposure on low-volume markets makes the price less consensus-driven than PredictIt's was. And the institutional flow that Kalshi has been trying to attract has changed the texture of certain markets in ways that long-time prediction-market traders find unfamiliar.

If you are coming from PredictIt and finding Kalshi uncomfortable, the discomfort is mostly because Kalshi is a real exchange and PredictIt was a research instrument. They were not the same product, even though they looked similar. The migration was always going to involve learning to think like a CLOB trader rather than like a PredictIt regular, and the learning curve is real.

## Live Data Reference

For Kalshi's historical calibration, see `/calibration` (filtered by venue=kalshi). For volume-sorted election markets, use `sf scan --venue kalshi --category election --by-volume desc`. The `/markets` page displays the indicator bundle for any specific Kalshi contract by ticker, and `/concepts/the-valuation-funnel` walks through how to use the indicators to filter the 25K-market universe down to the handful that are worth a closer look.

The maker-economics view of Kalshi specifically lives at `/opinions/automated-market-making-kalshi` and the trading-bot framing at `/opinions/trading-bot-needs-thesis`. The glossary entry for `/learn/event-contract` covers the core CFTC-DCM definition that distinguishes Kalshi's regulatory standing from PredictIt's withdrawn no-action letter.

## The Bottom Line

The migration from PredictIt to Kalshi is mostly a step up. Cheaper fees, larger universe, no position cap, regulatory durability. The things you lose are subtler — the small-trader equilibrium, the academic paper trail, the community texture — and they are real losses but not deal-breakers. Anyone who traded PredictIt successfully can trade Kalshi successfully with maybe two weeks of recalibration.

The thing I would not do is romanticize PredictIt. The $850 cap was not a feature, it was a regulatory constraint that we collectively decided to interpret as a feature. The platform was always small, always under-capitalized, and always one ruling away from closing. Kalshi is the prediction market venue we should have had all along. It just took longer to get here than anyone expected.