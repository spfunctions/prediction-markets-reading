# Cross-Venue Convergence Dynamics: Why Kalshi and Polymarket Converge — and When They Don't

> The same outcome on Kalshi and Polymarket usually trades within 2-5 cents of itself. When the gap widens past that, one of three specific things has happened. Knowing which one tells you whether to arbitrage, hedge, or stay out.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
If you put a Kalshi contract and the equivalent Polymarket contract on the same screen for a week, the most striking thing is how often they agree. Different traders, different jurisdictions, different orderbook structures, different fee schedules — and yet the implied probabilities usually live within a few cents of each other for any market with non-trivial volume. That convergence is not magic. It is the residue of dozens of cross-venue traders running the same arbitrage every time the gap opens.

I track about thirty cross-listed event pairs daily. The empirical pattern is consistent: in any given hour, roughly 80% of the pairs trade within 2-5 cents of each other after fee math, 15% are between 5 and 10 cents apart, and 5% are wider. The wide pairs are the interesting ones. They are either real arbitrage that the cross-venue traders have not gotten to yet, or they are flagging a venue-specific risk that the convergence machinery cannot price.

This page is about how to tell which is which.

## Why Convergence Happens At All

The mechanism is mechanical and unromantic. Every time the price on one venue moves more than ~3 cents away from the price on the other venue for the same outcome, a cross-venue arbitrageur with capital on both venues sells the rich side and buys the cheap side. The two trades create a hedged pair: whatever the outcome resolves to, one position pays $1 and the other pays $0, and the arbitrageur pockets the spread minus fees and bridge friction.

The arbitrage is not free money. There are real costs:

- Fees on both venues. Kalshi charges ~7% of profit on a winning trade. Polymarket has zero trading fees but USDC bridge costs typically $5-15 per round trip.
- Slippage on entry. The orderbooks on both venues are not infinitely deep, so a $10K notional position usually moves the price 1-3 cents while you build it.
- Resolution-rule mismatch risk. The two venues sometimes specify the resolution criteria slightly differently, so a contract that pays YES on Kalshi might not pay YES on Polymarket. This is the killer for naive cross-venue arbitrage.
- UMA dispute risk on Polymarket. The Optimistic Oracle can be challenged by anyone with capital, and the dispute window can delay or alter resolution.

When the price gap exceeds these costs in expectation, the arbitrage triggers. When it does not, the gap persists. The 2-5 cent equilibrium I see in the data is roughly the cost stack — that is the band inside which arbitrage does not pay enough to bother with.

## The Three Ways Convergence Breaks

When the gap widens past the cost stack, it is because one of three specific things has happened. Each one has its own diagnostic signature and its own correct response.

**Break 1 — venue-specific liquidity dry-up.** One venue gets thin while the other stays liquid. The thin venue becomes vulnerable to small flows pushing the price around, and the price drifts away from where the liquid venue is settled. Diagnostic: depth on one venue is 5x or more than the other; the price moves are correlated with small trades on the thin venue. Correct response: arbitrage the spread, but size into the thin venue first (the thin side is the one you can move) and accept that you may eat some slippage.

I see this most often on Polymarket during US trading hours when Kalshi's volume on US-political contracts dominates. The Polymarket equivalent gets quiet, drifts on one or two trades, and a 6-cent gap opens that gets closed an hour later when European volume picks back up on Polymarket.

**Break 2 — regulatory or news shock affecting one venue.** Something happens that changes the operating environment of one venue but not the other. The CFTC issues guidance on Kalshi's treatment of a category of contracts. Polymarket has a wallet incident. A specific market category gets de-listed on one venue. Diagnostic: the gap opens *with no underlying news about the event itself*, and stays open for hours or days. Correct response: do not arbitrage this — the gap reflects a real risk premium that will only resolve when the venue-specific issue resolves.

I learned this one the hard way on a 2024 election market where Polymarket was 5 cents below Kalshi for two days. I assumed it was Break 1 and went long on Polymarket / short on Kalshi (via NO). The next morning Polymarket announced a pause on the contract pending UMA review and the gap widened to 12 cents. I unwound at a loss because I was paying carry on the Kalshi short and the resolution timing was now uncertain.

**Break 3 — UMA dispute or pending resolution risk on Polymarket.** A specific Polymarket contract has an active UMA dispute, or the resolution criteria are ambiguous enough that a dispute is likely. The Polymarket price reflects the dispute risk; the Kalshi price (which has clearer resolution rules and CFTC oversight) does not. Diagnostic: the gap is on a single contract, the contract is approaching resolution, and there is observable activity in the UMA Discord or proposal queue. Correct response: stay out. The gap is real and the resolution is genuinely uncertain.

The way to distinguish Break 3 from Break 1 is to check the UMA proposal queue and the Polymarket Discord. If there is no dispute activity and no rumor of one, the gap is probably Break 1 (liquidity) and tradable. If there is dispute activity, it is Break 3 and you should leave it alone.

## How Fast Is the Convergence?

I have logged the convergence times for about 200 widening events over the last six months. The distribution is skewed but the pattern is reliable: about 80% of gaps wider than 5 cents converge back inside the 2-5 cent band within one hour. About 95% converge within four hours. The remaining 5% either persist for the lifetime of the contract or only resolve at settlement.

The gaps that persist are almost always Break 2 or Break 3. The gaps that converge fast are almost always Break 1. So a simple practical heuristic: if a gap has been wide for less than 30 minutes, treat it as a convergence opportunity (Break 1). If it has been wide for more than 4 hours, treat it as a structural difference (Break 2 or 3) and ask why before trading.

The convergence speed varies a lot by category. Election markets converge fast (lots of cross-venue traders). Sports markets converge fast (the sports-betting overlap is huge). Niche economic indicators converge slowly (small audience, few cross-venue traders watching). Geopolitical markets converge unpredictably (the news flow itself is asymmetric across venues).

## A Worked Convergence Trade

Let me walk through a real-shape example. The contract is "Will the Fed cut rates at the December meeting." It is listed on both Kalshi (as KXFEDDECISION-26DEC) and Polymarket (as POLY-FED-DEC-26).

Monday 9:00 AM ET: Kalshi YES is 0.42, Polymarket YES is 0.41. Gap: 1 cent. Within the band, no trade.

Monday 11:30 AM: A Powell speech leaks dovish commentary. Kalshi YES jumps to 0.48 in the next 15 minutes; Polymarket lags and is at 0.45. Gap: 3 cents. Still inside the cost band, no trade.

Monday 12:15 PM: Kalshi keeps drifting up to 0.51; Polymarket has not moved past 0.45 because European traders are off for the day. Gap: 6 cents. This is now wider than the cost band. I check the UMA queue (clean), the Polymarket Discord (no chatter), and the depth on both venues (Kalshi $4K, Polymarket $1.2K). Diagnostic: Break 1, liquidity asymmetry on Polymarket.

I do the trade. I buy YES on Polymarket at 0.45 for $1000 (filling at an average of 0.46 after slippage). I buy NO on Kalshi at 0.49 for the same $1000 notional (filling at an average of 0.50). My total cost is $0.96 per pair, max payoff is $1.00, expected profit is $0.04 minus fees.

Monday 2:30 PM: European volume picks back up on Polymarket. The price moves to 0.50, narrowing the gap to 1 cent. The convergence is essentially done. I can either hold to resolution and collect the $0.04 minus fees, or I can unwind by selling YES on Polymarket at 0.50 (gain: $0.04) and selling NO on Kalshi at 0.49 (gain: $0.01 from the price move, but I pay 7% of profit on the close). Net: about $0.035 per pair after Kalshi fees.

The trade worked because I diagnosed it correctly as Break 1. If I had done the same trade on a contract with an active UMA dispute (Break 3), the Polymarket leg might have been frozen by the time I tried to unwind, and I would have been stuck with a Kalshi NO position whose other leg was paralyzed.

## Where the Diagnosis Breaks

A few honest caveats on the convergence framework.

**Sometimes the diagnosis is ambiguous.** Break 1 and Break 3 can look identical for the first hour — both produce a wide gap with no obvious news. The way to disambiguate is to check the UMA queue and the venue-specific Discord, but in fast-moving situations you may have to act before the diagnosis is confirmed. The conservative play is "if you cannot rule out Break 3 in 5 minutes, do not trade."

**The fee math is venue-dependent and changes.** Kalshi's 7% on profit is current as of 2026 but has been changed multiple times. Polymarket's bridge costs depend on USDC gas conditions and have varied between $3 and $30 per round trip. The cost band that defines "no trade" shifts as the fee landscape shifts. Re-derive the band on your own venue accounts before relying on the 2-5 cent rule.

**Resolution-rule mismatches are silent killers.** Two contracts that look like the same outcome can have meaningfully different resolution criteria. Kalshi tends to be specific and CFTC-vetted; Polymarket can be more loosely worded. Always read both rule sets in full before treating the pair as the same trade. If you cannot convince yourself the two contracts will resolve the same way in every scenario, do not arbitrage them.

**Cross-venue arbitrage requires capital on both venues.** This is obvious but worth saying. If you only have an account on one venue, the convergence trade is not available to you, and you should ignore the gap entirely rather than half-execute it. Half-execution is how you end up with a directional position you did not intend to take.

## How This Connects to the Stack

The convergence framework lives at the intersection of stage 2 (orderbook reading) and stage 3 (judgment) of the [valuation funnel](/concepts/the-valuation-funnel). The numerical signal is the gap width — easy to compute, easy to scan with `sf scan --cross-venue`. The judgment is which of the three breaks you are looking at, and that requires reading the venue context, not just the price.

The [pm-indicator-stack](/concepts/pm-indicator-stack) does not currently include a cross-venue gap indicator as a Tier A metric, because the gap is per-pair rather than per-contract and the cluster matching is not 100% reliable. But it lives in the same family as the [event-overround](/concepts/event-overround) indicator — both are sums or differences across related contracts that should sum to a known value if the market is internally consistent.

For the philosophical case, see [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) — convergence is the cross-venue version of the yield-curve thinking that page argues for. For the negative case, see the upcoming [from-adr-arbitrage-to-cross-venue-pms](/concepts/from-adr-arbitrage-to-cross-venue-pms) bridge concept, which walks through the analogy with depositary-receipt arbitrage in equities and shows where the analogy holds and where it breaks.

For implementation, the [computing-event-overround-from-multi-outcome-events](/technicals/computing-event-overround-from-multi-outcome-events) technical is structurally similar — both involve summing prices across related contracts and checking for deviations from a known target. A cross-venue convergence detector is essentially a two-element overround calculation.

Convergence is not a primary strategy because the windows are short and the per-trade profit is small. It is, however, one of the most reliable patterns in the market: the same trade works repeatedly with low variance, and the diagnostic categories are stable enough that you can learn to spot them in under a minute. Treat it as a steady supplement to a thesis-driven primary book, not a replacement for one.