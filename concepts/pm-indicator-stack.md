# The Prediction Market Indicator Stack: Five Numbers That Actually Matter

> IY, CRI, EE, LAS, CVR. The five indicators that turn a binary price into a decision. Tier A is always available; Tier B is sparse and that sparsity is itself the entry condition for half the strategies in the stack.

**Category:** framework | **Author:** Patrick Liu | **Reading time:** 13 min

---
Every prediction-market trader I have met has a moment where they realize "the price is not enough." A market is at $0.42. So what. The number is necessary but it is not sufficient. You cannot size a position from $0.42 alone, you cannot compare two contracts from $0.42 alone, you cannot tell whether to trade or wait from $0.42 alone. Something else has to be on the screen.

The indicator stack is what I put on the screen instead. Five numbers, in two tiers. None of them are exotic, none of them require a model, all of them can be computed from data the venue already publishes. The point of the stack is not that these are the only useful things you can compute — it is that these five, taken together, change the question from "what is this contract worth" to "what kind of contract is this and what should I do about it."

## The Five, By Name

The stack is broken into Tier A (always available, computed from any market with any history at all) and Tier B (sparse, available only on markets that the warm-regime cron has covered, plus the ones I bother to manually warm).

**Tier A — always on:**

1. **Implied Yield (IY)** — `(1/p)^(365/τ) − 1`. The annualized return on the contract held to expiry. This is the bond-trader view of a binary, and it is the unit that lets you compare a Kalshi Fed contract to a treasury bill in the same column of a portfolio sheet.

2. **Cliff Risk Index (CRI)** — `|Δp/Δt| × τ_remaining`. How fast the contract is repricing, scaled by how much story room is left. High CRI means the market is actively making up its mind. Low CRI means the market is asleep.

3. **Event Overround (EE)** — `Σpᵢ − 1` summed across mutually exclusive outcomes in a multi-outcome event. Borrowed from sports betting. EE > 0 means the market is collectively over-pricing the certainty of *something* happening; EE < 0 means it is under-pricing.

**Tier B — sparse, warm-cron-gated:**

4. **Liquidity Availability Score (LAS)** — bid_depth + ask_depth scaled by spread. This is your "can I actually trade" metric. It is null on roughly 99% of the universe, because the warm-regime cron only computes it for the top 500 markets by 24-hour volume. That null is a feature, which I will get to.

5. **Contagion Velocity Rate (CVR)** — how fast a thesis is propagating from one market to its semantic neighbors. Sparse because it requires a graph of related markets, which only exists for events the system has clustered.

A sixth indicator — Position-Implied Velocity (PIV) — sits at the boundary between Tier A and Tier B because the substrate (the 1¢-delta tracker in `market_indicator_history`) is built incrementally and only has useful history once a market has been observed for at least seven days. I treat PIV as Tier B for any market younger than a week.

## What Each One Adds Beyond Price

The reason the stack is five numbers and not one is that each indicator answers a different question. None of them substitute for any of the others.

**IY answers "is this paying enough?"** A 32-cent contract with 9 days to expiry is paying ~8.3 million percent annualized if you are right. A 32-cent contract with 200 days to expiry is paying ~108% annualized. The mid-price is identical. The trades are not the same trade. IY surfaces the time axis that raw probability hides.

**CRI answers "is anything happening?"** Two contracts can have identical IYs and look like the same opportunity, but if one of them has been at the same price for forty days and the other moved 8 cents this morning, you are looking at very different things. CRI ranks them. The high-CRI contract is *live* and deserves stage-3 attention. The low-CRI contract might be a sleeper or might just be dead — that is what stage 3 is for.

**EE answers "is the event consistent with itself?"** In a multi-outcome event, the outcome prices have to sum to 1.0 (or the union has to cover the event space cleanly). EE measures the deviation. A persistently positive EE on the same event for days at a time often points to one outcome being structurally overpriced relative to its siblings, which is information you cannot get from looking at any single outcome in isolation.

**LAS answers "can I actually fill the order?"** Indicators 1–3 are about *whether the math says there is edge*. LAS is about *whether there is an orderbook to capture it from*. A 60% IY contract with $0.12 of total depth is paying 60% on a position size you cannot enter. LAS is the sanity check that prevents stage-1 alpha from being theoretical alpha.

**CVR answers "is this thesis already priced in?"** Information travels through markets at a measurable speed. If a thesis hits Polymarket and the same thesis is already starting to bend the price on Kalshi six hours later, you have an upper bound on how much edge is left. High CVR on a market means the thesis has propagated; low CVR means the market is still uncontaminated and you have time.

The thing none of the five indicators answer is "is the thesis correct." That is the whole job of stage 3 in the [valuation funnel](/concepts/the-valuation-funnel). The indicators tell you which markets are worth applying judgment to. They do not substitute for the judgment.

## Why These Five and Not Others

I get this question constantly. Why not realized volatility? Why not order-flow imbalance? Why not put-call skew (whatever the binary analog of that would be)?

The honest answer is that I built the stack incrementally over about eighteen months of trading prediction markets full-time, and these are the five that survived. Every other indicator I tried either reduced to one of these five (most "volatility" measures collapse to CRI), or did not add information beyond what I already had, or required data the venues do not publish.

The stack is also constrained by what is *cheap to compute*. Stage 1 of the funnel has to scan 47,000 markets in less than a second, and you cannot do that with anything that requires a non-trivial calculation per market. IY is one division and one exponent. CRI is one subtraction and one multiplication. EE is a group-by sum. LAS is a few additions. CVR is the only one that requires a graph traversal, and that is why it lives in Tier B with the warm cron.

Five numbers also fits on a screen. I have tried stacks with eight and ten indicators and they all suffered from the same problem: too many lights to look at means you stop looking at any of them. Five is the maximum I can keep in my head while reading a market description.

## The Null-Is-Signal Principle

Tier B is sparse because the warm-regime cron only covers ~500 markets at a time. The other ~46,500 markets have null values for LAS, CVR, and full-history PIV. The first instinct of every new user is "this is a bug, why is the data missing." The framing I want to introduce here, and which the [null-as-signal](/concepts/null-as-signal) concept page expands on in full, is that *the null is the entry condition for half the strategies in the stack*.

Concretely: a market with LAS = null is a market that has been below the top-500 volume threshold for at least one full warm-cron interval. Translated: *no one is trading it*. Translated again: *no maker is camping it*. Translated again: *if you put up a maker quote, you are alone on the book*. That is exactly the entry condition for the "virgin Polymarket" strategy in my MM book — find the markets nobody is making, become the only maker, capture the spread on the trickle of organic flow.

Similarly, EE = null on a multi-outcome event means there are no sibling outcomes — either the event is genuinely binary or the system has not clustered the siblings yet. Either case is information. CVR = null means the thesis has not propagated, which is the entry condition for getting in *before* the contagion happens. Each null state is, somewhere, an entry condition for a real strategy.

Treat null as a value, not as a defect. The indicator stack returns null on purpose. Filter for null when null is what you want.

## A Worked Cross-Section

Here are five contracts I pulled off the screen one Wednesday morning, with their full indicator stack. I have anonymized the tickers but the numbers are real-shape.

| Ticker | YES Price | τ | IY | CRI | EE | LAS | CVR |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| KXIPOSPACEX-26MAY01 | 0.18 | 73 | 1,142% | 0.6 | -0.04 | null | null |
| KXFEDDECISION-26JUN | 0.42 | 42 | 1,290% | 2.1 | +0.02 | 18.4 | 0.31 |
| POLY-SENATE-CO-26 | 0.55 | 220 | 56% | 0.4 | null | null | null |
| KXNFLAFC-26W12 | 0.62 | 4 | n/a | n/a | -0.01 | 91.2 | null |
| POLY-AI-AGI-2027 | 0.08 | 612 | 56% | 0.1 | null | null | 0.04 |

Reading the row by row: SpaceX IPO is paying high yield, has light activity (low CRI), the sibling cluster is slightly under-priced (negative EE), and the orderbook is unwarmed (LAS null). That is a maker setup if I am willing to hold τ = 73 days. The Fed-decision contract is paying similar yield with much higher CRI — the market is actively repricing — and LAS is healthy, so this is a *taker* setup if my thesis disagrees with the direction of the move.

The Senate contract has long τ, modest IY, and is in the "no warm coverage" category — interesting only if I have a long-horizon thesis that does not need execution speed. The NFL contract is too close to expiry for IY to mean anything (n/a), but EE tells me the sibling structure is fine and LAS says the orderbook is liquid, so this is a stage-3 only situation (i.e., do you actually have an opinion on the AFC outcome). The AGI contract is the long-tail extreme: tiny price, tiny CRI, nothing happening — a position you take and forget, not a position you scan for.

Five contracts, five different decisions. None of them readable from price alone.

## Where the Stack Breaks

A few honest caveats.

**Stale snapshots.** All the Tier B numbers (and the substrate for PIV) come from cron jobs. If the cron has been broken for an hour, your stack is reading stale data. Always check the snapshot freshness before treating any LAS or CVR number as live. The `sf scan --warm` flag forces a re-pull on the candidates you care about.

**Non-Tier-A markets are second-class.** The stack assumes Tier A indicators are computable, which means the market has at least a few hours of price history. Brand-new markets — first day of listing — have no CRI or PIV substrate yet. They are visible in stage 1 only by their IY, which on a brand-new market is more rumor than signal.

**Cross-venue inconsistency.** "Depth" on Kalshi is order book depth in cents; "depth" on Polymarket is AMM liquidity at the current curve point. LAS computed naively from the two looks like the same number but represents different things. If you are doing cross-venue work, normalize before comparing.

**The stack does not include everything that matters.** No volatility surface, no implied correlation, no flow-based indicators that would require trade-tape access. That is on purpose — the goal is "five numbers I can compute from public endpoints in milliseconds," not "every number that has ever been useful." If you need the seventh thing, you will know, and you should add it as a sixth tier with its own dedicated cron.

## How This Connects to the Rest of the Stack

The indicator stack feeds directly into the [valuation funnel](/concepts/the-valuation-funnel). Stage 1 is "filter by indicator," and the indicators it filters by are exactly the ones on this page. Each indicator also has a long-form concept page in batch B — see the upcoming [implied-yield](/concepts/implied-yield), [cliff-risk-index](/concepts/cliff-risk-index), [event-overround](/concepts/event-overround), [tau-days](/concepts/tau-days), and [expected-edge](/concepts/expected-edge) for the per-indicator deep dives.

For implementation: [computing-implied-yield-from-kalshi-tickers](/technicals/computing-implied-yield-from-kalshi-tickers) is the working code for IY, and the four sibling technicals in batch D do the same for CRI, EE, LAS, and PIV. For the philosophical case for the stack as a whole, see [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets) and the upcoming [why-i-built-the-indicator-stack](/blog/why-i-built-the-indicator-stack) blog post.

The stack is the spine of the screening workflow. Once the stack is on your screen and you have used it for two weeks, going back to staring at raw mid-prices feels like trying to read a book with one eye closed. The whole point of having indicators at all is that you are leaving information on the table by not computing them — and the cost of computing them is, at this scale, basically zero.