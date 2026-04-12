# The case for agentic market making on Kalshi

> Traditional market makers won't touch prediction markets — but thesis-informed agents with catalyst awareness can provide liquidity and profit from it.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
I've been trading on Kalshi with real money for the past year. Not paper trading, not backtesting — placing limit orders, getting filled, watching contracts resolve. And the single most consistent observation I've had is this: the liquidity is terrible.

Go look at any mid-tier Kalshi market right now. Pull up KXWTIMAX (WTI crude oil max price) or KXGDPNOW (GDP growth). You'll see spreads of 5-15 cents on contracts that should be 1-2 cents wide. You'll see the book is 3-deep on each side if you're lucky. You'll see entire strike ladders where the best bid is 8 cents below the best ask. This isn't an edge case — this is the default state of most Kalshi markets outside the top 10 by volume.

The question is: why? And the answer tells you something important about where prediction market liquidity is actually going to come from.

## Why Traditional Market Makers Haven't Shown Up

In equity and options markets, market making is a well-understood business. Firms like Citadel Securities, Virtu, and Jump run delta-neutral books. They quote both sides of the spread, hedge their exposure continuously, and make money on the bid-ask differential multiplied by massive volume. Speed matters because the game is adversarial — you need to update your quotes faster than informed flow arrives.

None of this works on Kalshi. Here's why:

**Volume is too small.** The median Kalshi market does $10K-$50K in daily volume. A traditional market maker's infrastructure costs more than the entire market's daily revenue. You can't justify a co-located server and a quant team for a market where total daily spread capture might be $200.

**You can't hedge.** If you're making a market on "Will WTI hit $150 in 2026?" and you get filled on the YES side, what do you hedge with? In equities, you hedge with the underlying. In options, you delta-hedge with the stock. In prediction markets, there is no underlying to hedge with. You could try to hedge with crude oil futures, but the correlation between "WTI hits $150" (a binary outcome) and the continuous price of WTI futures is nonlinear and path-dependent. Traditional delta-neutral market making breaks down.

**Speed doesn't help.** On Kalshi's API, order placement latency is measured in hundreds of milliseconds. The matching engine isn't designed for microsecond races. And even if it were, the flow is so sparse that being 10ms faster than the next guy doesn't matter when the next order might arrive in 10 minutes. The speed advantage that drives equity market making revenue simply doesn't exist here.

So the big firms look at this and rationally decide: not worth it. The markets are too small, the risk is unhedgeable, and the speed game is irrelevant. Move on.

This is correct from their framework. It's also an opportunity.

## Thesis-Informed Liquidity Provision

Here's the insight: you don't need speed to make markets on Kalshi. You need *catalyst awareness*.

Traditional market making is about being faster than informed flow. You quote both sides, and when informed flow arrives (someone who knows something you don't), you need to update your quotes before you get picked off. Speed is your defense mechanism.

But in prediction markets, informed flow isn't driven by inside information or faster data feeds. It's driven by *events* — catalysts that everyone can see coming but that arrive at unpredictable times and with unpredictable outcomes. OPEC meetings. CPI releases. Fed decisions. Court rulings. Elections.

If you know which catalysts affect which contracts, and you know when those catalysts are scheduled, you have something more valuable than speed: you have *positioning time*.

## A Concrete Example

Let's walk through this with real numbers.

It's a Tuesday. OPEC is meeting on Friday. You pull up the KXWTIMAX series — WTI crude oil maximum price contracts for 2026. Here's what you see:

```
KXWTIMAX-26DEC31-T120   YES: 68¢ bid / 74¢ ask   (spread: 6¢)
KXWTIMAX-26DEC31-T130   YES: 55¢ bid / 63¢ ask   (spread: 8¢)
KXWTIMAX-26DEC31-T140   YES: 41¢ bid / 52¢ ask   (spread: 11¢)
KXWTIMAX-26DEC31-T150   YES: 30¢ bid / 41¢ ask   (spread: 11¢)
```

These spreads are enormous. In any liquid market, you'd see 1-2 cent spreads. Here, the $150 strike has an 11-cent spread. That's a 26% round-trip cost for anyone who wants to express a view.

Now, you have a thesis. You've been tracking the Iran-oil-recession causal chain. Your model says Hormuz disruption keeps oil elevated. You think the $140 strike is worth about 48 cents (the market's midpoint is 46.5, so you're slightly bullish). And you know OPEC on Friday is a catalyst — depending on the production decision, these contracts will move 3-8 cents.

Here's what you do:

1. **Tuesday:** Place limit orders. Bid 43¢ on the $140 YES (below current bid of 41¢ — wait, actually you're *improving* the bid to 43¢). Ask 51¢ on the $140 YES (below current ask of 52¢). You've tightened the spread from 11¢ to 8¢.

2. **Wednesday-Thursday:** As traders position ahead of OPEC, volume picks up. Some of your orders get filled. You buy some $140 YES at 43¢ and sell some at 51¢. On each round-trip, you capture 8 cents.

3. **Friday:** OPEC announces production cuts. Oil spikes. Flow arrives — people want $140 YES and they want it now. Your remaining ask at 51¢ gets swept. New bid-ask moves to 50/56. You place new bids at 52¢. You're now providing liquidity in the post-catalyst regime.

The key: you placed your orders *three days before the flow arrived*. You didn't need to be fast. You needed to know that OPEC was meeting, that it would drive flow into oil contracts, and roughly where fair value was so you could set your spreads appropriately.

This is thesis-informed market making. Your thesis — oil stays elevated due to geopolitical supply disruption — gives you a fair value estimate. The catalyst calendar — OPEC Friday — tells you when flow will arrive. The combination lets you be the liquidity provider that captures the spread.

## The Contradiction at the Heart of It

Here's where it gets interesting: the same thesis that makes you a better market maker also exposes you to directional risk.

A traditional market maker doesn't care which way the price moves. They're delta-neutral. They make money on the spread regardless of direction. But a thesis-informed maker is *not* neutral. You think oil stays elevated. You think $140 YES is worth 48 cents. This means:

- When you buy $140 YES at 43¢, you're happy — you're buying below fair value *and* getting a good entry on a directional view.
- When you sell $140 YES at 51¢, you're selling above fair value — but you're also reducing a position you believe in.

This isn't a bug. It's the fundamental tradeoff. You're providing liquidity *because* you have a view, and your view makes you willing to hold inventory. A neutral maker wouldn't touch this market — there's nothing to hedge with. A thesis-informed maker can hold the position because they believe in it.

But when your thesis is wrong — when OPEC surprises with a production increase, or Hormuz reopens, or Iran de-escalates — you're holding inventory at the wrong price. You're not a neutral maker who can just widen spreads and wait. You're a directional trader who used market making as an entry mechanism, and now your entry was wrong.

The risk management question becomes: how do you separate your market-making P&L from your directional P&L? The answer is you can't cleanly. They're entangled. And that entanglement is exactly why traditional firms won't do this — it violates the separation of market making and proprietary trading that's fundamental to how they think about risk.

## Why Agents Will Dominate This

Now, here's the punchline. Everything I described above — catalyst awareness, thesis-informed fair values, spread management around scheduled events, inventory tracking, risk monitoring — is a system. It's not intuition. It's rules operating on structured data.

An agentic market maker does this:

1. **Maintain a causal model** of the world relevant to its markets. Iran → Hormuz → oil → specific KXWTIMAX strikes. Fed → rates → recession → KXRECESSION contracts.

2. **Track the catalyst calendar.** OPEC meetings, CPI releases, Fed decisions, earnings, court rulings. Know which catalysts affect which contracts and approximately how much.

3. **Calculate fair values** from the causal model. Not just a point estimate — a distribution. "$140 YES is worth 48¢ ± 5¢ depending on OPEC outcome."

4. **Place limit orders** at fair value minus spread on the bid, fair value plus spread on the ask. Tighten spreads in calm markets, widen them before high-uncertainty catalysts.

5. **Monitor fills and inventory.** Track net position by contract. Know when you're accumulating too much directional risk on one side.

6. **Update the causal model** when catalysts resolve. OPEC cuts production → oil node probability increases → downstream contract fair values update → existing orders get repriced.

7. **Detect thesis-breaking events.** If Iran announces a ceasefire, the agent doesn't just update a probability — it recognizes that the entire causal chain has broken and pulls all orders immediately.

An agent running this loop every 15 minutes across 50 markets simultaneously is providing meaningful liquidity to an ecosystem that desperately needs it. It's tightening spreads, deepening books, and making these markets tradeable for humans who want to express a view without paying a 10-cent spread.

And it's doing this profitably — not by being fast, but by being *informed*. The thesis gives it fair values. The catalyst calendar gives it timing. The causal model gives it update rules. The combination is a market-making strategy that doesn't require speed, hedging, or massive volume.

## What I'm Building Toward

At SimpleFunctions, we've built the infrastructure for thesis-informed trading: causal trees, catalyst monitoring, edge detection, orderbook enrichment. The next step is turning that infrastructure into an agentic liquidity engine.

I don't think the liquidity problem in prediction markets gets solved by traditional market makers showing up. The economics don't work for them. I think it gets solved by hundreds of thesis-informed agents, each running a causal model on a specific domain — oil, elections, macro, tech regulation — and providing liquidity to the contracts they understand.

The agent doesn't need to be right about everything. It needs to be right enough, often enough, that the spread it captures compensates for the directional risk it takes. And because it's operating on a structured causal model (not vibes), it knows *exactly* when to widen spreads (high uncertainty), when to pull orders (thesis-breaking event), and when to lean in (high-confidence catalyst approaching).

This is the future of prediction market liquidity. Not Citadel. Not speed. Informed agents with structured beliefs and the discipline to provide liquidity where it's needed.

If you want to build on this, everything starts at [simplefunctions.dev](https://simplefunctions.dev).