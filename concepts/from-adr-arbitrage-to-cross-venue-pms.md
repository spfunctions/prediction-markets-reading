# From ADR Arbitrage to Cross-Venue Prediction Markets

> American Depositary Receipts trade in two markets simultaneously and converge via arbitrage. The same outcome on Kalshi and Polymarket is the prediction-market version. The math is similar; the credit-risk profile is not.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 12 min

---
An American Depositary Receipt is a stub of a foreign-listed equity that trades on a U.S. exchange. The same underlying company trades in two markets simultaneously — say, Toyota in Tokyo and Toyota's ADR in New York — and the prices converge through arbitrage. When the ADR trades at a discount to the underlying, an arb desk buys the ADR, shorts the underlying, waits for convergence, and pockets the spread minus borrow costs and fx hedging. This trade is one of the oldest cross-listed-equity strategies on Wall Street and the math is well understood.

Cross-venue prediction markets are the same shape. The same outcome — say, "Will the Fed cut rates at the November meeting?" — trades on both Kalshi and Polymarket simultaneously. When the prices diverge, you can buy the cheap leg, sell the expensive leg, and wait for convergence. The trade looks like ADR arb with the symbols changed.

The math is similar but the credit-risk profile is genuinely different, and the difference is the part that gets sharp ADR traders into trouble when they walk onto PMs and assume their existing arb intuition will save them.

## What ADR Arbitrage Actually Is

An ADR is issued by a U.S. depositary bank — BNY Mellon, Citibank, JPMorgan — against a deposit of the underlying foreign shares. One ADR might represent one foreign share, or two, or half, depending on the issuance ratio. The depositary bank ensures the ADR is fungible with the underlying via a creation/redemption mechanism: an authorized participant can deliver foreign shares to the bank and receive new ADRs, or surrender ADRs and receive shares back.

This creation/redemption process is what enforces the price relationship. If the ADR trades at a 2% premium to the fx-converted price of the underlying, an arb participant can buy the underlying in Tokyo, deliver it to the depositary bank, receive the ADR, sell it in New York, and pocket the spread. The mechanism is mechanical, the spread is normally a few basis points, and the trade is essentially riskless if you can size it right and hold for the convergence window.

The risks that *do* exist are well-understood:

- **fx risk** during the conversion window
- **Settlement risk** between the two exchanges (T+2 in NY, T+3 in Tokyo, with the gap covered by financing)
- **Borrow risk** if you are shorting one leg and the borrow becomes expensive
- **Conversion fees** charged by the depositary bank, typically a few cents per ADR

These are real costs but they are knowable and they net out to a small structural friction. ADR arb is the canonical example of "boring riskless arbitrage that pays a small but reliable spread."

## How Cross-Venue PM Arbitrage Looks Similar

The same outcome listed on both Kalshi and Polymarket is the PM version of cross-listing. Take a contract on "Will the Fed cut rates at the November meeting?":

- Kalshi: KXFEDNOV YES at $0.42
- Polymarket: POLY-FED-NOV YES at $0.39

The 3-cent spread is the divergence. If the two contracts will eventually resolve to the same value (both YES, both NO), then *exactly one* will pay $1.00 and the other will pay $0.00 — because the contracts are economically identical, they share the same underlying event.

The arb is: buy the cheap leg (POLY YES at $0.39), sell or short the expensive leg (KXFED YES at $0.42, which on a binary venue means buying NO at $0.58 since YES + NO = $1.00). Total cost: $0.97 per pair. Total payoff: exactly $1.00 regardless of how the contract resolves — because if the event happens, POLY pays $1.00 and KXFED NO pays $0.00 (you lose $0.58); if the event does not happen, POLY pays $0.00 and KXFED NO pays $1.00 (you lose $0.39). Either way, your gross is $1.00 against a cost of $0.97.

The risk-free spread is $0.03, or about 3.1% of the cost basis. On an annualized basis with a 30-day holding period, that is roughly 47% IRR. This is exactly the kind of math an ADR arb desk would recognize.

## Where the Math Diverges

The above analysis assumed the two contracts are perfectly equivalent. In ADR arb, that assumption is enforced by the depositary bank's creation/redemption mechanism — there is a literal physical process that converts one instrument into the other and guarantees fungibility. In cross-venue PM arb, *no such mechanism exists*. The two contracts only converge if the resolution decisions on both venues agree, and there is no enforcement that they will.

Three concrete ways the assumption breaks down:

**Resolution timing.** Kalshi might resolve a contract within hours of the event. Polymarket might take days because of the UMA dispute window. During that gap, you have one leg paid and the other still open, and you cannot close the position cleanly.

**Resolution criteria.** Kalshi and Polymarket sometimes have slightly different wordings of "the same" contract. "Will the Fed cut at November's FOMC?" on Kalshi might resolve based on the official statement; the equivalent on Polymarket might resolve based on the federal funds target range moving. In most cases these are the same event. In edge cases they are not.

**Resolution disputes.** Polymarket's UMA process can resolve a contract in a way that disagrees with the obvious answer. If a Polymarket contract resolves NO when Kalshi resolves YES, your "arbitrage" loses both legs. This is a real failure mode, not a theoretical one, and it has happened on contracts where the wording was contested.

These risks have no ADR-arb analog. ADR arb has fx risk and settlement risk, but those are *operational* risks that net out over many trades. Resolution-criteria risk on cross-venue PMs is a *credit* risk — a small probability of total loss on the trade — and credit risk does not net out the same way operational risk does.

## Pricing the Resolution-Risk Premium

The 3-cent spread between the two venues is not pure inefficiency. Some of it is *priced compensation for the resolution-risk premium*. When you buy the cheap Polymarket leg, you are accepting a small probability that UMA disputes the contract and you lose the full position. The sustainable cross-venue spread reflects the market's collective estimate of how often that happens.

A rough framework: if the resolution-risk premium on Polymarket is 2 cents (i.e., the market collectively thinks there is a ~2% chance of an adverse dispute outcome on the average contract), then a 3-cent cross-venue spread has only 1 cent of "real" arbitrage in it after compensating for that risk. The remaining 1 cent has to overcome execution costs (fees, slippage, position-sizing constraints), and what is left over — typically zero or small — is the actual riskless return.

This is the part that ADR arb desks underestimate. They look at a 3-cent spread and see "47% IRR" without pricing in the credit-risk overlay that the spread is partially compensating for. The actual riskless return is much lower than the spread suggests, and the residual is risk premium that you are taking on when you put on the trade.

## A Worked Cross-Venue Trade

Take a real-shape trade. KXFEDNOV YES at $0.42, POLY-FED-NOV YES at $0.39. You decide to put on a one-pair arb at notional $100 per leg.

Buy 100 POLY YES at $0.39 = $39 cost. Buy 100 KXFED NO at $0.58 = $58 cost. Total cost: $97. Expected payoff at resolution: $100 (regardless of outcome). Expected gross profit: $3 on $97 cost.

But wait, fees:

- Polymarket has near-zero trading fees, but slippage on a $39 trade in a thinner POLY market is about 2 cents on the price, so your effective entry is $0.41 per contract instead of $0.39. New POLY cost: $41.
- Kalshi charges 7% on profit at settlement. If POLY pays $100 and KXFED NO pays $0, you net $100 - $41 - $58 = $1 gross from POLY and lose the full $58 on Kalshi. Wait, let me redo this.
- Actually: at settlement, *one* leg pays $100 and the other pays $0. The winning leg gets the 7% Kalshi fee taken if it is the Kalshi leg.

Recompute:

- Case 1: Event YES. POLY YES pays $100, KXFED NO pays $0. Profit on POLY: $100 - $41 = $59. Loss on KXFED NO: $0 - $58 = -$58. Net: $1. No Kalshi fee because the Kalshi leg lost.
- Case 2: Event NO. POLY YES pays $0, KXFED NO pays $100. Loss on POLY: $0 - $41 = -$41. Profit on KXFED NO: $100 - $58 = $42, minus 7% Kalshi fee on profit = $42 - $2.94 = $39.06. Net: -$1.94.

So the trade pays $1 if YES happens and loses $1.94 if NO happens. The expected value depends on the probability of YES, which the market is currently pricing at roughly 0.41 (the average of the two venues). At p = 0.41, EV = `0.41 × 1 + 0.59 × (-1.94) = 0.41 - 1.14 = -0.73`. Expected loss of 73 cents per pair.

The trade that looked like a 3-cent arb is actually a *negative-EV trade* once you account for fees and slippage. The whole 3-cent spread was eaten by the fee asymmetry between the two venues plus the slippage on the thinner leg.

This is the kind of math an ADR arb desk does in their head before putting on the trade. The PM equivalent requires the same discipline and the answer is often "the spread is not big enough to be a real arb, even though it looks like one." The interesting question becomes: *how big does the spread have to be for the trade to be positive-EV?*

For this contract, with the fee structure above, the breakeven spread is about 5¢ — you need POLY at $0.37 (or KXFED at $0.44, or some combination) before the trade pays after fees. Below that, you are paying the venues to take risk for them.

## What Carries Over Beautifully: The Workflow

Once you accept the credit-risk overlay, the rest of the ADR arb workflow ports cleanly:

1. **Universe scan**: identify pairs of contracts on Kalshi and Polymarket that represent the same outcome. This is the analog of identifying ADR/underlying pairs. The PM-native version is in [computing-event-overround-from-multi-outcome-events](/technicals/computing-event-overround-from-multi-outcome-events) — the same logic that finds sibling outcomes on a single venue can find equivalent outcomes across venues.

2. **Spread tracking**: for each pair, track the price spread continuously and alert when it exceeds a threshold. ADR desks have run this kind of monitoring on Bloomberg terminals for decades. The PM version is `sf scan --cross-venue --min-spread 0.05` (or the equivalent in your tooling).

3. **Execution**: when a spread exceeds threshold, execute both legs simultaneously. ADR desks have algorithmic execution for this; PMs require manual coordination because there is no shared book.

4. **Position management**: hold until convergence (which on PMs means resolution, since there is no creation/redemption mechanism to force convergence pre-resolution). Track the spread daily; close out if the spread widens further (indicating a real divergence that might persist).

The workflow is identical. The only difference is the credit-risk overlay at every step, which means the *thresholds* are higher than ADR arb thresholds and the *position sizes* are smaller.

## What Does Not Carry Over: The Convergence Mechanism

ADR arb works because the depositary bank's creation/redemption mechanism *guarantees* convergence at any time. Cross-venue PMs have no such mechanism. The two contracts only converge at resolution, and resolution is weeks or months away. You hold both legs for the full duration and your effective IRR depends on the path the prices take.

This is the biggest difference and it changes the strategy. ADR arb is a "scalp the spread, exit immediately" strategy. Cross-venue PM arb is a "lock in the spread, hold to maturity, accept the credit risk" strategy. The right comparison is not ADR arb but rather buying the cheap leg of a CDS basis trade and holding to default. A trader who comes from ADR arb without internalizing the credit-arb mindset will keep getting blown up.

## What Does Not Carry Over: Borrow Costs

ADR arb often involves shorting the more-expensive leg, which requires borrowing the security. PMs do not have borrow costs in this sense. You can buy NO on Kalshi as a "synthetic short" of YES, and the cost is just the NO price (which is `1 - YES_price`). No third-party borrow mechanism. This is one of the few ways PMs are *cleaner* than ADR arb.

## Where the Bridge Breaks Hardest

The cleanest case where the analogy fails is when one venue lists a contract and the other does not. In ADR arb, this never happens — the depositary bank guarantees a tradable instrument on each side. In PMs, plenty of Kalshi contracts have no Polymarket equivalent, and vice versa. The mitigation is to focus the strategy on contracts that are *structurally listed on both venues* — major elections, Fed decisions, big macro events — though those also tend to have tighter spreads and less arb opportunity. The sweet spot is the *medium-volume* tier where both venues list but neither has aggressive market makers.

## How This Connects to the Stack

The cross-venue framing connects to several pieces of the indicator stack. The [pm-indicator-stack](/concepts/pm-indicator-stack) is where you identify candidates with attractive math; the [valuation-funnel](/concepts/the-valuation-funnel) is where you screen them down to executable candidates. The cross-venue check is a *fourth filter* you apply on top of the funnel: after you have a candidate that survives stages 1–3, you check whether there is a cross-venue equivalent and whether the spread is wider than the resolution-risk premium would justify.

The [cross-venue-convergence](/concepts/cross-venue-convergence) phenomenon page describes the empirical patterns of when and why convergence happens. The [steam-moves-across-venues](/concepts/steam-moves-across-venues) page describes the related phenomenon of sharp money moving between venues. Together they form the cross-venue layer of the framework.

For the philosophical case that PMs should be talked about in fixed-income vocabulary rather than equity-arb vocabulary, see [/blog/prediction-markets-need-fixed-income-language](/blog/prediction-markets-need-fixed-income-language) and [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) — the cross-venue spread is closer to a credit basis trade than an equity arb, and treating it as such is what keeps you out of trouble.

## The Honest Summary

ADR arb intuition is roughly 60% transferable to cross-venue prediction markets. The workflow is identical. The math for spread sizing is similar. The pre-trade screening is similar. What is *not* transferable is the assumption of instant convergence — PMs have no creation/redemption mechanism, so you hold both legs to resolution, and during the holding period you take on the credit-risk overlay that comes from each venue's resolution process.

The right mental model is *credit arb*, not ADR arb. Size positions based on the credit-risk premium you are taking on, not the headline spread. Focus on liquid contracts where both venues list. Verify the contract wordings actually match. Account for the fee asymmetry between the two venues, which can quietly turn a positive-spread trade into a negative-EV trade. Run `sf scan --cross-venue --min-spread 0.05` to find candidates, but treat the output as a starting point, not a list of free trades.

The spread is real. The Polymarket leg has tail risk. Size accordingly.