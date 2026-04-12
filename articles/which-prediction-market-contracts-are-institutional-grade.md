# Which Prediction Market Contracts Are Institutional-Grade

**Category:** insights | **Author:** Patrick Liu | **Reading time:** 12 min | **Published:** 2026-04-12

---
A friend from a multi-strat fund pinged me last month asking if we had a screener he could point his PM at. Not for fun — for the book. He wanted to know which prediction market categories could absorb a $50K position without moving the market more than a penny, and which ones were basically arcade games with a Bloomberg terminal plugged in.

I told him I would get back to him after I ran the numbers. This is that email, cleaned up.

## The Question Behind the Question

When a fund asks "which prediction markets can we trade," they are not asking about prediction markets. They are asking about five things at once:

1. **Can I get size on?** — Depth. Can I place K+ without eating through three levels of the book.
2. **Can I get out?** — Holding period. Is there a secondary market, or am I locked until resolution?
3. **Do I have edge?** — Information asymmetry. Is this a market where my infrastructure (data feeds, models, analysts) gives me a structural advantage over the marginal counterparty?
4. **Can I tell compliance?** — Regulatory defensibility. Can I write a one-paragraph justification that my chief compliance officer won't send back?
5. **Can I build a portfolio?** — Market count. Are there enough correlated instruments in this category to construct a diversified position, or is it a single-name bet?

I ran every active contract on Kalshi and Polymarket through our [indicator stack](/concepts/pm-indicator-stack) and grouped by category. Here is what the numbers say.

## Elections and Politics: The Only Category That Passes All Five Tests

I will just say it upfront: this is the one.

Kalshi has 46 election contracts with daily volume above ,000. Average bid-ask spread: 0.8 cents. Average bid-side depth: ,000. Average ask-side depth: ,000. On Polymarket, the politics and geopolitics buckets together account for 434 liquid markets and roughly  million in combined daily volume.

Those are not Sports-tier numbers in raw volume — Sports does  million a day on Kalshi alone. But Elections clears the institutional bar in ways Sports never will:

**Depth is real.** A K market order on CONTROLH-2026-D moves the price less than a cent. Try that on an NBA first-half spread contract and you will eat three levels.

**Tau is long.** 221 out of 235 liquid election markets resolve in 3 months or more. That is a holding period an allocator can underwrite. Compare that to Sports, where 430 out of 1,895 liquid contracts expire within a week — those are day trades, not positions.

**Information edge is quantifiable.** Polling aggregation, campaign finance data, demographic modeling, turnout estimation — these are statistical problems. A fund with a quant team and a data budget has a structural advantage over the retail bettor who is trading on vibes and cable news. This is not true in Sports, where the marginal counterparty is a professional sports bettor with fifteen years of closing-line value data.

**Compliance can write the memo.** The CFTC approved Kalshi's election contracts. "Event risk hedging" is a recognized use case. A corporate treasury hedging tariff exposure through a trade-policy contract has a cleaner compliance story than most structured credit positions I have seen in fund decks.

**Portfolio construction works.** You can build a diversified book across presidential, congressional, gubernatorial, and policy-outcome contracts. The correlation structure is rich enough to hedge within the category — long one party's Senate odds, short their House odds, and you have a spread trade that isolates the chamber-specific signal.

Cross-venue arbitrage is also concentrated here. We track 21 Kalshi-Polymarket pairs in Elections, more than every other category combined. When the same outcome is priced at 47 cents on one venue and 44 on another, that is a mechanical  edge per contract with no model risk. Our [cross-venue scan](/api/public/screen?has_orderbook=true&sort=las) surfaces these daily.

## Economics and Financials: Good Edge, Thin Books

This is the second-best category, but with a caveat that changes the entire trading strategy.

Economics has 136 liquid contracts. Average spread: 2.5 cents. Average bid-side depth: ,000. [LAS](/learn/liquidity-adjusted-spread) (spread as a fraction of price): 1.3 — wide. Financials has the tightest LAS on the platform at 0.023, but bid-side depth of only ,100.

The information edge here is arguably the strongest of any category. CPI prints, NFP revisions, Fed decisions — these are markets where a fund with a nowcast model and a Bloomberg terminal has a massive structural advantage over retail. The edge is not subtle: it is the difference between someone who has read the BLS methodology document and someone who has not.

But you cannot be a taker in these markets. The books are too thin. A K market order on a Financials contract will blow through every level. The strategy that works here is **making markets**: posting limit orders on both sides, collecting the spread, and using your information edge to skew the quotes. This is exactly what our [regime system](/api/public/regime/scan?label=maker) is designed to identify — markets in a "maker regime" where the adverse selection risk is low enough that providing liquidity is profitable.

83 out of 136 liquid Economics contracts expire within a month. That means high turnover, frequent settlement, and a short feedback loop on whether your model is right. For a systematic strategy, this is attractive. For a discretionary PM who wants to put on a position and check it next quarter, it is not.

## Crypto: Conditional on Who You Are

Crypto has 221 liquid contracts. Average spread: 1.5 cents. Depth: K bid. The tau distribution is extreme — 111 out of 221 expire within a week.

The institutional play here is not directional prediction. It is **cross-venue arbitrage** between the prediction market contract and the underlying spot price. When a Kalshi contract asks "Will BTC be above ,000 on Friday?" and the spot market implies a different probability than the contract price, that is a mechanical edge.

But crypto prediction markets layer two compliance risks: the crypto exposure and the prediction market exposure. Many fund compliance departments will reject this on sight, regardless of the edge. If your fund already has a crypto mandate, this is worth exploring. If it does not, the compliance cost of getting approval exceeds the edge.

## Everything Else: No

**Sports** (M daily volume): Highest raw liquidity but worst adverse selection. Your counterparty is a sharp bettor with decades of closing-line-value data. A quant fund has no structural edge here.

**Climate and Weather** (.8M volume): Insurance companies have information edge but the depth is catastrophic — ,400 average bid. You cannot get size on.

**Entertainment and Mentions**: Pure attention markets. Information is not quantifiable. No model can predict what Elon Musk will tweet.

**Companies, Science, World**: Fewer than 40 liquid markets each. Not enough to build a portfolio.

## The Punchline

If I had to write the one-paragraph version for my friend's PM, it would be this:

Elections and political-outcome contracts are the only prediction market category that simultaneously clears the five institutional hurdles: depth, holding period, quantifiable edge, regulatory defensibility, and portfolio construction. Economics and Financials are the second tier, but only for market-making strategies — the books are too thin for directional taking. Everything else is either too illiquid, too adversely selected, or too compliance-toxic for institutional capital.

The screener data behind this analysis is live at [sf screen](/api/public/screen?sort=iy). The regime labels are at [regime scan](/api/public/regime/scan). If you want to drill into a specific contract, [sf inspect](/api/agent/inspect/CONTROLH-2026-D) gives you the full dossier — indicators, regime, orderbook, cross-venue pair, and a suggested next action.

I will update this when the numbers change.