# When the Orderbook Is Empty, You Have Information

> An empty orderbook is not missing data. It is one of two specific stories — and the second story is the most reliable maker setup I have found on Polymarket.

**Category:** insights | **Author:** Patrick Liu | **Published:** 2026-04-09

---
For most of my first year staring at prediction-market dashboards, I treated an empty orderbook the way a normal financial trader would: as a defect. As "data quality issue, skip and move on." If I clicked into a Polymarket contract and saw no bids on one side and three stale asks at random round numbers on the other, I would close the tab. Empty orderbook meant the market was untradable and untradable meant uninteresting.

This took me an embarrassingly long time to unlearn. The truth is closer to the opposite: an empty orderbook is one of the most concentrated forms of *information* that prediction markets emit, and the entire third-most-profitable strategy I run depends on reading that information correctly.

This is a short essay on the reframe and on what changed in my workflow once I started treating empty orderbooks as a signal in their own right.

## The Two Stories an Empty Orderbook Can Tell

There are exactly two reasons a Polymarket orderbook can be empty.

**Story one:** Nobody wants this market. The event is degenerate or already resolved or so far in the future that no one has bothered to take the other side. The market is empty because it is *uninteresting to humans*. There is no edge here for anyone, including you. Move on.

**Story two:** Nobody knows about this market. The event is real, the resolution is well-defined, the contract has been listed for two days, and you are one of the first three people to look at it. The market is empty because it is *unmonitored*, not because it is uninteresting. There is enormous edge here for the first patient maker who shows up and posts a wide quote.

Both stories produce identical screen output. Same thin asks at meaningless round numbers, same total absence of bids, same near-zero recent trade activity, same null on the [liquidity availability score](/learn/liquidity-availability-score) the warm-cron uses to decide whether a market is worth caching.

The whole game is figuring out which story you are looking at.

## How I Used to Get This Wrong

For about eight months I had a default sort on my Polymarket screener that excluded any market with fewer than 50 trades in the past 24 hours. The rationale was "filter out the dead markets so I can focus on the live ones." This is the rationale you would use if you came from equities, where an illiquid stock is, with very few exceptions, a *bad* stock.

What that filter was actually doing was throwing away every contract in story two. Every market that had been listed yesterday and not yet been discovered. Every contract in a freshly-launched event tree that had not yet attracted a maker. Every quiet long-tail position that was about to become loud the moment the right news cycle hit it.

I was filtering out exactly the population of markets that had the highest-quality maker setups, because I was using a "quality filter" that mistook *unmonitored* for *unwanted*.

I figured this out in February. I had been running a small cross-venue screener and noticed that one of my Polymarket positions — a quiet contract on a state legislative committee vote, listed less than a week earlier — had been the only seller in the book for three days, then suddenly absorbed about $2,400 of buying volume in a single morning when the relevant committee posted a hearing schedule. My position was up about 9% on the day, which on the size I was running was meaningful.

I went back and looked at what the orderbook had looked like the day I posted my ask. It had been completely empty. Not "thin." Not "wide." Empty. No bid, no ask, no trades for 36 hours. I had only placed the order because the contract slug had popped up in a manual scan of newly-listed markets and the resolution criteria struck me as unusually clean.

That afternoon I sat down and tried to understand what I had stumbled into.

## The Reframe

The reframe is short and it is the entire substance of this essay.

*An empty orderbook is the entry condition for a maker, not an exit condition for a taker.*

If you are trying to take liquidity, an empty book is a wall and you cannot do anything useful with it. But if you are willing to provide liquidity — to be the patient party who sits on the bid or the ask and waits for the first taker to walk into your quote — then an empty book is the *least competitive* environment you will ever encounter. There is literally no one else providing liquidity. You are the entire market on at least one side. Your spread, your size, your timing are all unconstrained by other makers because there are no other makers.

The most reliable version of this setup is what I have started calling the *virgin polymarket* condition: a Polymarket contract that has been listed within the last 7 days, has fewer than 5 historical trades, and has [LAS = null](/learn/liquidity-availability-score) (meaning the warm-regime cron has not yet promoted it into the top-500 by 24-hour volume). All three of those conditions independently mean "no one has been looking at this." Together they mean "you are very probably the first sophisticated participant in this market, and the next person who arrives — even an uninformed one — will be hitting your quote."

I have worked through more of the maker-strategy mechanics in [the post about the indicator stack](/blog/why-i-built-the-indicator-stack), and I am not going to redo that here. What I want to flag in this essay is the *conceptual* shift, which is that "no orderbook" stops being a defect and starts being a screening attribute you actively *want* to find.

## The Filter I Replaced the Old Filter With

The screener now has two default views. The first view is the old default — markets with healthy 24-hour volume, sorted by [implied yield](/learn/implied-yield). This is the view I use when I am running a *taker* strategy. I want to see a deep book and a meaningful spread to harvest, and the volume filter is the right gate for that.

The second view is what I call the *empty book* view. It sorts in the opposite direction. It surfaces markets where:

1. Total 24h volume is less than $500
2. The orderbook depth on either side is less than 100 cents at the touch
3. LAS is null (or below 0.05 if not null)
4. The market has been listed within the last 30 days
5. The resolution criteria parse cleanly via the [CYC regex grouper](/concepts/pm-indicator-stack)

The fifth criterion is the load-bearing one. I do not want every empty book; I want empty books on contracts whose resolution criteria are *clean*, because clean resolution criteria are a proxy for "this contract was set up by someone who knew what they were doing, and the listing was probably driven by a real upcoming event." The CYC grouper catches about 41% of contracts in any given week, and that 41% is the population I am searching within.

When I run this view on a typical Tuesday morning I get back somewhere between 30 and 90 markets. Most of them are in story one and I can dismiss them in five seconds by reading the resolution criteria and asking "would I bet on this if someone offered me 100% odds?" If the answer is "no, the event itself is structurally degenerate," it is story one. If the answer is "yes, this is a real event I just have not heard about," it is story two and I post a quote.

## What I Still Get Wrong

I have become better at telling the two stories apart, but I still misclassify at least a third of the markets I look at. My typical failure mode is treating story one as story two — being optimistic about a contract whose resolution turns out to be either ambiguous or uninteresting in a way I missed on the first pass. I do not lose much on these misclassifications because the maker quote is wide and the size is small, but they are noise on the strategy.

The opposite failure mode — treating story two as story one and walking past a real maker setup — is harder to detect because I never see what I would have made on the contracts I dismissed. I know I am still doing this. I do not yet have a good feedback loop for catching it, because the universe of "things I correctly walked past" is enormous and I cannot possibly inspect it.

## The Open Question

The thing I keep coming back to is a more general version of the same insight: how many other "empty" or "null" or "missing" signals in my workflow am I currently treating as defects when they are actually positive entry conditions?

I have a few candidates. PIV near zero on a wide-spread market. CVR null on a contract whose neighbors are all moving. EE undefined on a market with no listed siblings. Each of these is, I suspect, the same kind of latent maker setup the empty orderbook turned out to be. But I have not yet built the screener views for them, and I have not yet run the equivalent of the February experiment that taught me what an empty book actually meant.

I will get to them. The pattern is clear enough that I trust the *category* of insight even before I have done the work on each specific case. There are things in the data my old framing was telling me to ignore, and most of them are not defects. Most of them are entry conditions in disguise.