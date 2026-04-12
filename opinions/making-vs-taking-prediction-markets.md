# Making vs taking in prediction markets: two completely different games

> Most traders don't realize they're playing the wrong game — market making and market taking in prediction markets require opposite personalities, opposite edges, and opposite relationships with time.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 9 min

---
I've been trading prediction markets on Kalshi for over a year now with real money. Somewhere around month three, I noticed something that completely changed how I think about this space: there aren't prediction market traders. There are prediction market *takers* and prediction market *makers*. And they're playing entirely different games.

Most people never make this distinction. They open Kalshi, see a contract like KXRECSSNBER trading at 35 cents, form an opinion, and either buy or sell. That's taking. But there's a whole other game happening underneath — the people posting those limit orders at 34 and 36, maintaining a spread, adjusting continuously. That's making. Same market, completely different activity.

## The hunter and the farmer

The best mental model I've found: takers are hunters, makers are farmers.

A hunter waits. Surveys the landscape. When they spot prey — a mispriced contract — they strike with conviction and then they're done. A single kill can feed them for weeks. The hunter's edge is *judgment*: seeing something the market doesn't see.

A farmer tends crops every day. They don't need any single harvest to be extraordinary. They need the process to be reliable across hundreds of cycles. The farmer's edge is *process*: capturing small amounts of value consistently across time.

These aren't just different strategies. They're different cognitive modes. Different relationships with uncertainty. Different definitions of what "winning" means.

## What taking actually looks like

When I'm taking on Kalshi, here's what happens. I spend days building a causal model around some thesis — say, the probability that WTI crude hits a specific max level. I'm looking at KXWTIMAX contracts. I decompose the thesis into sub-claims: OPEC production decisions, geopolitical risk in the Strait of Hormuz, US strategic petroleum reserve levels, demand forecasts from China.

Each sub-claim gets a probability. I propagate those through the causal tree. I end up with a thesis-implied price for a specific contract.

Then I check the market. If my thesis says KXWTIMAX-120 ("WTI crude above $120 at any point this quarter") should be 45 cents and it's trading at 28, I have an edge. I buy. If it should be 45 and it's at 43, I pass.

The key features of taking:

- **Discrete**: I enter a position and I'm done. Maybe I'll adjust later if my thesis changes, but the act of entry is a single decision.
- **High-conviction**: I need to believe I'm right *and* that the market is wrong. That's a high bar.
- **Low-frequency**: I might make one or two thesis-driven trades per week. Some weeks zero.
- **Edge from judgment**: My alpha comes from understanding the world better than the current price reflects.

When I nail a take, it feels like solving a puzzle. The market was wrong about something specific, I identified it, and I got paid for being right.

## What making actually looks like

Making is nothing like this.

When I'm making markets, I'm posting bids and offers on both sides of a contract. Say KXRECSSNBER — the Kalshi recession contract based on the NBER declaration. It's trading around 30 cents. I might post a bid at 28 and an offer at 32. If both get filled, I've captured 4 cents of spread regardless of whether a recession happens.

That sounds simple. It's not. Here's what's actually going on:

- **Continuous management**: I need to watch those orders constantly. If recession risk suddenly spikes — a bad jobs report, a bank failure — I need to pull my bid at 28 before someone takes it and I'm underwater.
- **Inventory risk**: If my bid at 28 gets filled but my offer at 32 doesn't, I'm now long the recession contract. I didn't want a position. I wanted a spread. But the market moved, and now I'm a reluctant directional trader.
- **Width vs. volume**: Wider spreads mean more profit per fill but fewer fills. Tighter spreads mean more fills but less profit each time — and more risk of adverse selection (getting picked off by someone who knows something I don't).
- **Low conviction per trade**: I don't need to know whether a recession is happening. I need to know what the *fair spread* is around the current price.

The key features of making:

- **Continuous**: It's not a decision, it's a process. You're always on.
- **Low-conviction**: You don't need strong views on the outcome. You need accurate views on the spread.
- **High-frequency**: Dozens of fills per day across multiple contracts.
- **Edge from process**: My alpha comes from correctly pricing the spread and managing inventory, not from being right about the world.

When I nail a making session, it doesn't feel like solving a puzzle. It feels like a good day of farming. Nothing dramatic happened. The crops grew.

## Why prediction market making is fundamentally different

If you've read about market making in equities or crypto, you might think you already know this game. You don't. Prediction markets break several assumptions that traditional market making relies on.

**Episodic liquidity, not continuous**. In equities, AAPL trades all day, every day. Liquidity is roughly continuous. In prediction markets, liquidity is episodic. A contract like KXWTIMAX might see zero volume for hours, then a burst of activity when oil data drops, then nothing again. Your making strategy has to handle this. You can't just set-and-forget a spread.

**No speed race**. Traditional market making on equities has devolved into a latency arms race. Firms spend hundreds of millions to shave microseconds off their execution time. Prediction markets don't have this problem — yet. The edge isn't in speed, it's in *catalyst awareness*. If you know that the EIA petroleum status report drops at 10:30 AM ET every Wednesday, and you widen your spread on KXWTIMAX at 10:29, you've just avoided the adverse selection that kills naive makers. No co-location required.

**Binary outcomes with known resolution**. Every prediction market contract resolves to 0 or 100. There's no dividend, no earnings surprise, no stock split. This means the distribution of outcomes is fundamentally different. As a maker, you're not dealing with fat tails in the traditional sense — you're dealing with a Bernoulli distribution whose parameter (the true probability) is drifting over time as information arrives.

**Catalyst-driven price action**. In equities, prices move semi-continuously based on a massive flow of micro-information. In prediction markets, prices move in steps driven by identifiable catalysts. The NBER recession contract doesn't drift — it jumps when GDP data comes out, when unemployment numbers drop, when the NBER committee actually makes an announcement. As a maker, you need a calendar of catalysts for every contract you're quoting.

## The coupling

Here's what makes prediction markets intellectually interesting: making and taking aren't independent. They're coupled through the same causal tree.

When I'm building a thesis around oil markets, the causal model I construct — OPEC decisions, geopolitical risk, demand forecasts — serves both modes. As a taker, that model tells me which contracts are mispriced. As a maker, that same model tells me when to widen spreads (before a catalyst hits a node in the tree) and when to tighten them (when the tree is stable).

The same causal tree, different execution modes.

This coupling means the best prediction market participants do both — but at different times, on different contracts, with different mental modes. I might be making markets on KXRECSSNBER while simultaneously holding a directional take on KXWTIMAX. The recession-making is a process; the oil-taking is a position. They coexist because they draw on related but different parts of the same world model.

## The personality mismatch problem

Here's where most people go wrong: they apply the wrong personality to the wrong game.

**Hunters trying to farm.** This is the most common failure mode I see. Someone who's great at forming contrarian views and making big directional calls tries to make markets. They can't help themselves — they let their directional bias leak into their quotes. They make the bid too wide on one side because they "feel" the contract should go up. They hold inventory too long because they've convinced themselves the position will work out. They turn every making session into a series of small directional bets, which is the worst possible version of taking: low-conviction *and* high-frequency.

**Farmers trying to hunt.** Less common but equally destructive. Someone who's good at process-driven, systematic work tries to make directional bets. They size too small because they're used to thinking in terms of spread capture, not conviction sizing. They exit too early because they're trained to manage inventory, not hold positions. They update too frequently on noise because they're monitoring continuously, which is good for making but terrible for taking. Every minor data release shakes them out of a position that required patience.

I've made both mistakes. Early on, I'd try to make markets while having strong directional views. I'd consistently get picked off on one side because I was shading my quotes. I was neither a good maker nor a good taker — I was a confused mess.

The fix was simple in theory, hard in practice: when I'm making, I force myself to be symmetric. No directional bias in my quotes. If I have a strong view, I *take* a position separately and keep my making book clean. When I'm taking, I force myself to stop watching the order book tick by tick. I set my entry, set my exit conditions, and walk away.

## The infrastructure gap

One reason I built SimpleFunctions was exactly this distinction. Most prediction market tools don't acknowledge that making and taking are different activities. They give you a price chart and an order book and leave you to figure it out.

For taking, you need thesis infrastructure: causal trees, probability propagation, edge calculation against live market prices. For making, you need spread management: catalyst calendars, inventory tracking, fill monitoring, real-time spread width adjustment.

Same data feeds, different abstractions on top.

I've found that just having the mental model — "am I hunting or farming right now?" — improves my trading more than any specific tool. But having the right tools makes the mental discipline easier to maintain.

## The bottom line

If you trade prediction markets, ask yourself: are you a taker or a maker? Not as an identity — you can be both. But at any given moment, on any given contract, you should know which game you're playing.

If you're taking: do you have a thesis? Can you write down the causal tree? Do you know what evidence would falsify your position? If not, you're not taking — you're gambling.

If you're making: are your quotes symmetric? Are you tracking your inventory? Do you know when the next catalyst drops? If not, you're not making — you're providing liquidity as charity.

The people who consistently make money in prediction markets are the ones who know which game they're playing and play it well. The people who lose are the ones who mix the two — hunting when they should be farming, farming when they should be hunting.

Two games. Same market. Choose wisely.

---

*I'm Patrick. I build [SimpleFunctions](https://simplefunctions.dev) — prediction market research tools for people who take this stuff seriously.*