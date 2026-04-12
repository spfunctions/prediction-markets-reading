# Kalshi vs Polymarket: Mechanics, Fees, Regulation, Liquidity (2026)

> A side-by-side of the two largest prediction-market venues in 2026. Neither one wins outright. Kalshi wins on legality and tax paperwork; Polymarket wins on fees and listing speed.

**Category:** comparison | **Author:** Patrick Liu | **Reading time:** 11 min

---
Both venues take roughly the same shape from a distance — binary contracts, central limit order books, real money in, real money out — and they end up at very different places once you actually price a trade through each one. Kalshi is the regulated venue. Polymarket is the global one. There is no winner. There is a decision tree, and the right answer depends on three things: where you live, what tax forms you need, and how often you turn over the position.

I run capital on both. The split is roughly 60% Kalshi, 40% Polymarket, and the reason for the split is mostly fee math on the kind of trades I do most often, not any abstract preference for one venue.

## TL;DR

| Axis | Kalshi | Polymarket |
|---|---|---|
| Regulation | CFTC-designated DCM (US-legal) | Polygon-based, offshore, US-restricted |
| Settlement currency | USD via wire / ACH | USDC on Polygon |
| Market types | CLOB only | CLOB with AMM fallback on certain markets |
| Fees | 7% on profit (taker side) | 0% trading fees in 2026 |
| Withdrawal speed | 1-3 business days | Minutes to USDC, then bridge to fiat |
| Active markets | ~25,000 | ~22,000 |
| Daily notional | ~$10M USD | $5M-$30M USDC depending on news cycle |
| Marketwide Brier (`/api/calibration`) | 0.160 | 0.030 |
| API auth | JWT, ~100 req/s, free | Read open, signed wallet for write |
| New market listing | Slow (regulated review) | Fast (anyone can propose, UMA resolves) |
| KYC | Required | None for read; wallet for write |
| Tax reporting | Yes (1099-B) | No (self-report) |

## Regulation

Kalshi is a CFTC-designated contract market. The DCM designation is the same legal status that the CME and ICE futures exchanges hold, and it is the reason Kalshi can list event contracts to US residents and accept ACH deposits from a checking account. Every contract is reviewed by the CFTC before it lists, which is why Kalshi's market universe grows in lurches rather than continuously, and why you will not see a "Will Drake release an album by July?" market on Kalshi any time soon.

Polymarket is built on Polygon and resolves disputes through UMA's optimistic oracle. There is no regulator in the loop. New markets can list within hours of someone proposing them, and the universe is bounded only by what the community is willing to underwrite. The tradeoff is that US residents are technically restricted from trading — Polymarket geofences US IPs after the 2022 CFTC settlement — and the platform has no formal KYC, no SIPC equivalent, and no recourse if Polygon itself fails.

The practical effect: if you live in the US and you need to be able to point at a tax form when something goes wrong, Kalshi is the only option. If you live elsewhere or you are willing to use a non-US wallet and accept the legal ambiguity, Polymarket is open.

## Settlement Currency

Kalshi settles in USD. Money goes in via wire or ACH from a US bank account. Withdrawals come back the same way and clear in 1-3 business days. Every position is denominated in dollars on the screen, every fill is in dollars, every realized P&L is in dollars. If you want to move $50,000 of capital from your trading account to your brokerage account, it is one transfer and one day.

Polymarket settles in USDC on Polygon. To get money in, you bridge USDC from Ethereum mainnet to Polygon (or buy USDC directly on Polygon via a fiat ramp). To get money out, you reverse the bridge and then off-ramp USDC to a fiat account through Coinbase, Kraken, or one of the on-chain ramps. The whole process can take minutes if you already have a Polygon wallet warmed up, or it can take hours and several gas-fee transactions if you are starting from zero.

The hidden cost is bridge friction. A round trip from a US bank account into a Polymarket position and back out is at least four discrete steps (fiat → Coinbase → USDC on mainnet → bridge to Polygon → bridge back → off-ramp), and at each step there is a small fee plus the variance of network conditions. For frequent traders this is a real expense; for occasional traders it is the kind of thing that makes you not bother with positions under $500.

## Market Types

Kalshi runs a pure central limit order book. Every fill is an order matching another order at a specific price. There is no AMM, no LMSR liquidity subsidy, no automated market-making. The orderbook is what the orderbook is, and on thin markets that means the spread is what whichever maker happened to quote it last is willing to provide.

Polymarket runs CLOB primarily, with an AMM fallback on certain markets — typically the lower-volume long-tail markets where no human or bot maker is active. The AMM fallback uses an LMSR-style pricing curve to provide synthetic liquidity, which means you can always get a fill but the price can be substantially worse than the true mid. On the high-volume markets the CLOB dominates and the AMM is irrelevant. On the long-tail it is the difference between "no market at all" and "a market at a 10% spread."

For active traders the difference is mostly invisible — both venues feel like CLOBs because the markets you actually trade are the ones with active makers. For market-makers, the difference matters: on Polymarket you are competing with the AMM curve as your floor, which tightens what spreads you can profitably quote.

## Fees

Kalshi charges 7% on profit on the taker side of every trade. That means if you buy a contract at 30¢ and it resolves YES at $1.00, you pay 7% of the 70¢ profit, or 4.9¢ per contract. Maker rebates are zero — Kalshi does not pay you to provide liquidity. This is the cleanest fee schedule of any prediction market venue, and it has the advantage of being predictable: you always know exactly what your post-fee return is going to be on a winning trade.

Polymarket charges 0% trading fees in 2026. That is not a typo. Both maker and taker pay nothing on trades. The platform earns revenue through other mechanisms (interest on USDC float, eventually sponsored markets) and has chosen to subsidize trading. The catch is that the gas fees on Polygon are not zero — every order consumes a few cents of MATIC — and the bridge friction described above is the real cost.

For a buy-and-hold trade where you enter once and exit once, Kalshi's 7% on profit eats roughly 5% of a typical winning trade. Polymarket's gas + bridge eats maybe 1%. For a high-frequency strategy that turns over 50 times in a week, Kalshi's fee structure is brutal and Polymarket's gas is irrelevant. The crossover point is around 4-6 turnovers per week, and if you are above it, Polymarket pays you back the bridge friction many times over.

## Calibration

`/api/calibration` returns marketwide Brier scores broken down by venue. As of the most recent backfill, marketwide Brier sits at 0.095, with Polymarket at 0.030 and Kalshi at 0.160. Lower is better. The five-times gap is real and worth understanding before you treat one venue's price as more "trustworthy" than the other's.

The Polymarket number is artificially low because of selection bias in the resolved sample — the heavily-traded election markets dominate the resolved set, and election markets are the easiest case for any prediction market because they have the most traders, the most data, and the cleanest resolution criteria. The Kalshi number is higher partly because Kalshi's universe is more diverse (weather, sports, economic indicators, crypto, election) and partly because some of those categories are genuinely harder to calibrate.

Read this carefully: the marketwide Brier difference does NOT mean Polymarket is "more accurate" in a vacuum. It means that on the specific resolved sample we have, Polymarket's price-vs-outcome distance is smaller. For your specific trade in your specific category, look at the by-category Brier breakdown at `/calibration` rather than the headline number.

## Worked Example: The Same Outcome on Both Venues

A US election outcome trades on both venues simultaneously. Real-looking numbers from a typical Tuesday morning:

| Venue | Mid | Bid Depth | Ask Depth | Spread |
|---|---|---|---|---|
| Kalshi | 0.47 | 8,400 contracts | 6,200 contracts | 1¢ |
| Polymarket | 0.49 | 12,000 contracts | 9,800 contracts | 1¢ |

The 2¢ cross-venue spread is the kind of dislocation that happens 5-10 times a day on any liquid election market. To capture it, you would buy YES on Kalshi at 47¢ and sell YES on Polymarket at 49¢ (or buy NO at 51¢, depending on which side you have inventory on). Pre-fee gross is 2¢ per contract.

On Kalshi, the 7% profit fee applies if you eventually resolve the position. If you close on Kalshi rather than holding to resolution, the fee is on the realized P&L, which means the 2¢ fill costs you about 0.14¢ in fees. On Polymarket, gas eats maybe 0.1¢ equivalent. Net edge: ~1.7¢ per contract. On 1,000 contracts, that is $17. On 10,000, that is $170. Whether the trade is worth taking depends on the size of the bridge friction you have to amortize.

## Decision Tree

- **You need US-legal access and a 1099-B at year end:** Kalshi.
- **You need fast new market listings (a contract on a news event from this week):** Polymarket.
- **You live outside the US and cannot legally use Kalshi:** Polymarket only.
- **You run a high-turnover strategy with 4+ entries per week per market:** Polymarket, the fee math wins.
- **You buy-and-hold long-dated contracts and want predictable settlement:** Kalshi.
- **You do cross-venue arbitrage:** both, simultaneously, and watch the cross-venue spread on `/screen` or your own monitor. Use the indicator stack at `/markets` to surface live arb candidates.
- **You care about institutional infrastructure (FIX API, prime brokerage):** Kalshi has the more developed institutional stack.

## Where Each Side Breaks Down

Kalshi's failure modes are mostly around the speed of new market listing. If a piece of news breaks at 9 AM that creates obvious betting interest (a sudden Fed announcement, a celebrity scandal, a court ruling), Kalshi will not have a market on it before the news has stopped being news. The CFTC review process is incompatible with breaking events. If you want to bet on the news cycle in real time, Kalshi structurally cannot serve you.

Kalshi's other failure mode is the 7% profit fee on high-turnover strategies. I have run automated maker strategies on Kalshi where the per-trade gross edge was 0.8¢ and the post-fee net edge was negative. The fee schedule is calibrated to discourage exactly the kind of high-frequency liquidity provision that makes a venue tight. If your business model depends on quoting tight spreads in size, Kalshi punishes you for it.

Polymarket's failure modes are UMA oracle disputes and the legal-ambiguity tail. Roughly once a quarter, a market resolves in a way that the community contests, and UMA's optimistic oracle has to adjudicate. The dispute process can take days, during which your position is frozen. The 2024 Brazil election market is the famous case study — the resolution was ultimately settled in line with the plain reading of the rules, but for 72 hours nobody knew where their capital sat.

The legal-ambiguity tail is harder to quantify. Polymarket has been operating under a 2022 CFTC settlement that prohibits US persons from trading. The geofence catches most US users; some get through. If the CFTC re-engages, the tail risk is that US users lose access to their capital with limited recourse. I do not know how to price this risk and neither does anyone I trust on it. I size Polymarket accordingly — never more than I am willing to write off entirely.

## Live Data Reference

The `/screen` page surfaces cross-venue spread candidates in real time, pulling from `/api/public/scan` filtered by `venue=both` and ranked by spread width. The `/markets` page shows the per-market indicator bundle (IY, CRI, EE, LAS) for both venues side-by-side when the same outcome trades on both. For per-category calibration data, see `/calibration`, which renders the Brier breakdown by category and venue. The yield-trade framing of cross-venue spreads is covered in depth at `/opinions/implied-yield-vs-raw-probability-bond-markets` and `/opinions/automated-market-making-kalshi`, both of which apply directly to the Kalshi vs Polymarket fee math above.

For the framework view of how the indicators feed into venue selection, see `/concepts/the-valuation-funnel` and the `/learn/cross-venue` glossary entry. You can also use `sf scan --venue both --by-cross-spread desc` from the CLI to surface the same data in a sortable list, then `sf bet <ticker> yes 100` to fire the trade once you have picked one.

## The Bottom Line

There is no universal winner. The two venues serve different parts of the market and they will continue to serve different parts of the market for at least the next several years. The right move is to have accounts on both and to route each trade through the venue where the math works. For me that is roughly 60-40 in favor of Kalshi because most of my volume is in long-dated Fed and macro contracts where the 7% fee is amortized across a meaningful time window. Your split will look different if your turnover is different.

The thing I would not do is treat the choice as ideological. "I only trade Kalshi because regulation matters" and "I only trade Polymarket because crypto is the future" are both ways of leaving money on the table. The math is what it is, and the math says route per-trade.