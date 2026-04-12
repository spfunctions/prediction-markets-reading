# LMSR vs CLOB vs Continuous Double Auction: Prediction Market Architectures

> Three liquidity models, three trade-offs. LMSR for cold-start, CLOB for active markets, CDA for specific event-driven cases. Why Polymarket runs both at the same time.

**Category:** comparison | **Author:** Patrick Liu | **Reading time:** 12 min

---
Three different ways to run a prediction market, three different liquidity stories. LMSR is the academic answer, CLOB is the financial-markets answer, and CDA is the messy middle ground that some venues use for specific event-driven cases. The choice of architecture is not just an implementation detail — it changes who can participate, how prices form, and what edge looks like.

Most active prediction-market traders only ever interact with CLOB markets and never have to think about LMSR or CDA. That is fine for trading, but it leaves the architectural question on the table, and the architectural question matters when you are evaluating whether a venue is the right place for your specific strategy.

## TL;DR

| Axis | LMSR | CLOB | CDA |
|---|---|---|---|
| Liquidity provider | Automated formula (logarithmic scoring rule) | Human or bot makers posting limit orders | Human or bot makers in batch auctions |
| Initial liquidity required | Operator subsidy (LMSR loses bounded amount) | Zero (empty book until first quote) | Zero |
| Price discovery | Continuous via formula | Discrete via order matching | Periodic via batch auctions |
| Slippage | Predictable, logarithmic | Depends on book depth | Depends on batch size |
| Worst-case loss for operator | Bounded by liquidity parameter b | Zero (operator does not provide liquidity) | Zero |
| Best for | Cold-start markets, low volume | High-volume markets with active makers | Event-driven markets with predictable trading windows |
| Used by | Hanson 2003 paper, Manifold (play money), Augur v1 | Kalshi, Polymarket CLOB, traditional financial exchanges | Some sports books, some experimental venues |
| Computational complexity | O(1) per trade | O(log n) per match | O(n log n) per auction |
| Composability with on-chain settlement | Excellent (formula is deterministic) | Good (order matching is deterministic) | Good |

## LMSR — Logarithmic Market Scoring Rule

LMSR was introduced by Robin Hanson in a 2003 paper titled "Combinatorial Information Market Design." The core insight is that you can implement a prediction market without any human market-makers by having an automated formula price every trade. The formula is:

```
C(q) = b × log(Σ exp(q_i / b))
```

Where q is a vector of outstanding share quantities for each outcome, b is a liquidity parameter chosen by the operator, and C is the operator's cumulative cost function. The market price for outcome i at any moment is the partial derivative of C with respect to q_i, which works out to:

```
p_i = exp(q_i / b) / Σ exp(q_j / b)
```

That is the entire pricing logic. There is no orderbook. There is no bid-ask spread. There is just the formula, and every trade moves the prices according to the formula.

The properties that make LMSR useful:

- **Cold-start problem solved**. You can launch a market with zero traders and the formula will quote a price. The first trader gets a fill at the initial price (50/50 by default), and subsequent traders get prices that adjust based on the running book of shares.
- **Bounded operator loss**. The maximum the operator can lose is b × log(N) where N is the number of outcomes. This is critical for any operator who wants to subsidize liquidity without unbounded risk.
- **Predictable slippage**. A trader who buys X shares of an outcome moves the price by a known amount given by the formula. This is in contrast to CLOB markets where slippage depends on what is on the book at the moment.
- **Deterministic and on-chain friendly**. The formula is a pure function. Augur v1 used LMSR specifically because it could be implemented as a smart contract without needing off-chain order matching.

The properties that make LMSR limiting:

- **Worse pricing as volume grows**. Once a market has active human or bot makers, those makers can quote tighter spreads than the LMSR formula because they can incorporate information that the formula does not. LMSR markets at high volume are systematically worse priced than CLOB markets at the same volume.
- **Subsidy required**. The operator has to put up the b parameter as a real-money commitment to absorb potential losses. For a venue running thousands of markets, the cumulative subsidy is substantial.
- **Liquidity parameter is a guess**. Setting b too low means the market is too sensitive to small trades; too high means the operator's potential loss is too large. Tuning b for thousands of markets is a real engineering problem.

LMSR is the right answer for the cold-start case — markets with zero traders that need to bootstrap. It is the wrong answer for any market that has reached active trading volume.

## CLOB — Central Limit Order Book

A central limit order book is the architecture every traditional financial exchange uses. Traders post buy and sell limit orders at specific prices and quantities, the venue maintains a sorted book, and trades happen when a buy and sell cross. The price you see on the screen is the mid between the best bid and the best ask.

The properties that make CLOB the right default for active markets: tight spreads when makers are present (1-2 cents on a 0-100 scale, dramatically better than LMSR can produce), information aggregation across every resting order, unbounded scaling to high volume, and a profit motive for liquidity provision because makers can earn the spread by posting orders. The venue does not provide liquidity, so the venue does not lose money when no one trades.

The properties that make CLOB limiting: cold-start failure (new markets have empty books and the first trader has nothing to trade against), fragility to maker withdrawal (when a maker pulls quotes during news, the book collapses), and a real software-engineering cost — a CLOB matching engine is a project, not a one-line formula.

CLOB is what you want once a market has volume. It is what Kalshi runs exclusively and what Polymarket runs on its high-volume markets.

## CDA — Continuous Double Auction (and Batch-Auction Variants)

A continuous double auction is closely related to a CLOB but with a different matching cadence. In a true batch CDA, orders are collected for a short window (often 1-10 seconds) and then matched at a single clearing price. This is the "uniform-price auction" model that some traditional exchanges use for opening and closing crosses.

In the prediction-market context, CDA is rare. A few experimental venues have used batch matching for in-play sports markets where traders need to react to game state without being picked off by faster bots. The benefits are real (latency arbitrage disappears, price discovery is cleaner during volatile windows) but the costs are also real (the semantics confuse retail traders, the matching algorithm is more complex, and orders submitted early in the window leak information). Most venues that have experimented with CDA have ended up reverting to CLOB.

## Why Polymarket Runs Both LMSR and CLOB

Polymarket originally launched on an LMSR-style automated market maker, which was the right call for a venue with no makers and lots of cold-start markets. As volume grew, Polymarket added a CLOB layer for the high-volume markets and kept the LMSR fallback for the long tail.

The current architecture: every Polymarket market has both an LMSR-style AMM and a CLOB. Traders interact with whichever one offers better pricing for their order, and on the high-volume markets the CLOB almost always wins because human and bot makers quote tighter spreads than the formula can. On the long-tail markets, the AMM provides the only liquidity.

This dual architecture is a smart design choice. It solves the cold-start problem (the AMM is always there) without sacrificing the tight spreads that come from human market-making (the CLOB takes over when makers show up). The cost is implementation complexity — running two matching engines and routing orders between them — but the user experience is a single market with continuously available liquidity at competitive prices.

Kalshi made the opposite choice: pure CLOB, no AMM fallback. The bet is that if the venue has enough volume and enough institutional makers, the cold-start problem solves itself because makers will quote new markets within hours of listing. This has been mostly true on the major Kalshi markets and partially false on the long tail, where some Kalshi contracts have empty orderbooks for weeks at a time.

## Worked Example: Slippage Under LMSR vs CLOB

A trader wants to buy 1,000 contracts of YES on a binary market. The mid is 50¢. Compare slippage under each architecture.

**Under LMSR with b = 100 (typical low-volume parameter):**

The cost of buying 1000 shares from a book at mid 50/50 is given by C(q) - C(0) = 100 × [log(exp(1000/100) + 1) - log(2)] ≈ 100 × [10 - 0.693] ≈ 930. So the average price per share is 0.93 — almost twice the mid. Slippage is 43¢ per share. This is brutal because b is low.

**Under LMSR with b = 5000 (typical high-volume parameter):**

C(q) - C(0) = 5000 × [log(exp(1000/5000) + 1) - log(2)] ≈ 5000 × [log(2.221) - 0.693] ≈ 5000 × [0.798 - 0.693] ≈ 525. Average price 0.525. Slippage is 2.5¢. Still meaningfully worse than mid but tolerable.

**Under CLOB with a thick book:**

Suppose the book has 1500 contracts at 50¢, 800 at 51¢, 500 at 52¢. Buying 1000 contracts crosses the 50¢ level (filling 1000 of the 1500 available) at zero slippage. Average fill 0.50. Slippage 0¢.

**Under CLOB with a thin book:**

Suppose the book has 200 at 50¢, 100 at 52¢, 50 at 55¢, 50 at 60¢, then nothing until 70¢. Buying 1000 contracts fills 200 at 50, 100 at 52, 50 at 55, 50 at 60, and the remaining 600 at 70. Average fill ≈ 0.633. Slippage 13¢.

The CLOB outperforms LMSR when the book is thick, and LMSR with high b outperforms CLOB when the book is thin. Polymarket's dual architecture makes the user-side choice automatic — whichever quotes a better price wins. From the trader's perspective, you do not need to know which engine handled your order; you just need the order to fill at a good price.

## Decision Tree

- **You are building a new prediction market venue from scratch with zero traders:** start with LMSR, add CLOB later when volume justifies it.
- **You are building a venue that expects institutional traders and high volume from day one:** pure CLOB. The institutional traders will not tolerate LMSR's spreads.
- **You are building a venue for a specific event-driven use case (in-play betting):** consider CDA for the event window only, with CLOB the rest of the time.
- **You are choosing between Kalshi and Polymarket as a trader:** for high-volume markets, the architecture is invisible; pick on fees and regulation. For long-tail markets, Polymarket's AMM fallback often gives you a fill where Kalshi has an empty book.
- **You are evaluating a venue you have not heard of before:** ask which architecture they run. If LMSR-only and they claim high volume, they are subsidizing more than they should be. If CLOB-only and they have a cold-start problem, they are leaving long-tail markets unserved.
- **You are running `sf scan --by-las desc`:** the LAS column is computed from CLOB depth, so it is undefined on pure-LMSR markets and meaningful only on CLOB or hybrid venues.

## Where Each Side Breaks Down

LMSR breaks down at high volume because the formula cannot compete with human makers on spread tightness. Once a market has >5,000 contracts of daily volume, the LMSR formula is consistently worse than what a competent maker could quote. This is why Polymarket transitioned its high-volume markets to CLOB and why no major venue uses pure LMSR anymore.

CLOB breaks down at low volume because the orderbook is empty and there is no liquidity. The cold-start problem is real and structural. Kalshi handles it by relying on professional makers to seed new markets, but on the long tail this seeding does not happen and the markets sit empty for days. The Liquidity Availability Score (LAS) at `/learn/liquidity-availability-score` exists precisely to identify these orphaned markets, and the "null LAS = entry condition for virgin market making" framing is the practical workaround.

CDA breaks down on retail UX. The batch-auction semantics confuse traders who expect immediate fills, and the implementation complexity is not justified by the marginal benefit on most prediction markets. CDA is a niche solution for a niche problem (latency arbitrage) and most venues do not have that problem badly enough to justify the architecture.

## Live Data Reference

For a live look at orderbook depth on Kalshi (CLOB) and Polymarket (CLOB + AMM fallback), use `/markets` and inspect the per-market indicator bundle. The `las` field is the executable-depth proxy and is null when no maker is providing liquidity. For cold-start markets where you want to find LMSR-style auto-quoted prices, Polymarket's long tail is the place to look — those markets fall back to AMM pricing when no human maker is active.

The `/concepts/pm-indicator-stack` page covers how the indicators are computed and where each one breaks down by venue architecture. For the underlying market microstructure, `/concepts/the-valuation-funnel` walks through how to filter the universe down to markets where the architecture supports the trade you want to make. The `/learn/market-maker` glossary entry covers the maker role across architectures, and the practitioner essays at `/opinions/automated-market-making-kalshi` and `/opinions/liquidity-availability-as-the-real-edge` apply directly to the CLOB-vs-LMSR question for active strategies.

## The Bottom Line

The architecture choice is settled at the venue level by the venue operator, and as a trader you mostly do not need to think about it. What you do need to think about is whether the venue you are using is the right architecture for your strategy. If you trade long-tail markets, you want a venue with LMSR fallback. If you trade high-volume markets, you want a venue with deep CLOB liquidity. If you do both, you want Polymarket's hybrid model, which is the only major venue that supports both modes natively.

The deeper point is that the architecture is invisible until it isn't. When the long-tail market you are trying to trade has a 200-contract empty book on Kalshi and a 5¢-spread AMM quote on Polymarket, you are seeing the architectural choice in action. That is the moment when "which venue do I use" stops being a fee question and becomes a liquidity question, and the answer depends on what your strategy needs from the underlying matching engine.