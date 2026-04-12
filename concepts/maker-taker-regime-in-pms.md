# Maker / Taker Regime in Prediction Markets: How to Read the Orderbook State

> Every binary market is in one of three regime states at any moment: maker-dominated, taker-dominated, or neutral. The state changes which side of the book you should be on, and reading it correctly is the difference between collecting spread and donating it.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Most of the time when I look at a Kalshi orderbook I am not asking "is this a good price." I am asking a prior question: *who is currently in charge of this book.* The answer is one of three states, and the strategy I deploy from that point on is determined entirely by which state I read.

The three states are maker-dominated, taker-dominated, and neutral. They are not formal categories — there is no flag in the API that says `regime: 'maker'`. They are observable patterns in the spread, the depth, the trade tape, and the rate at which quotes refresh. Every active prediction-market trader is implicitly classifying every contract they look at into one of these three buckets, whether they have a name for it or not. This page is the version of that classification I have written down for myself.

## The Three States

**Maker-dominated.** The book is wide and deep on both sides. Resting orders sit for minutes without being touched. Spread is structurally larger than the trade size — say a 4-cent bid-ask with $400 of depth on each side. Volume is low and lumpy. When trades happen, they cross the spread and are immediately absorbed by maker quotes that refresh almost instantly. The market is being *priced by the makers*; the takers are paying tribute.

**Taker-dominated.** The book is thin and the spread is narrow. Resting orders disappear within seconds of posting. Trade tape is fast and one-sided. Spread is smaller than typical trade size — a 1-cent bid-ask with $80 of depth that gets eaten in the first taker order, leaving a 3-cent gap behind. Quotes do not refresh — makers are *withdrawing* because they cannot keep up with the directional flow. The market is being *priced by the takers*; whoever crosses first sets the next reference.

**Neutral.** Mixed signals. Spread is moderate, depth is moderate, trade tape is intermittent. There is no clear control. Most contracts on Kalshi are in this state most of the day. Neutral is uninteresting *except* as the baseline against which the other two states are visible.

The three buckets are not symmetric. Maker-dominated is the natural resting state of any contract with an institutional MM camped on it. Taker-dominated is what happens when news arrives or a catalyst window opens. Neutral is the in-between zone where neither is true and your strategy has to be regime-agnostic.

## How to Read the State From the API

You do not need a regime classifier to do this. You need three numbers and a couple of seconds of patience.

First, pull the orderbook depth and the spread from the venue API or from the `market_regime_snapshots` table that the SimpleFunctions warm cron populates for the top-volume contracts. The relevant fields are bid_depth, ask_depth, and spreadCents. Second, pull a recent slice of the trade tape — the last fifteen to thirty trades is usually enough. Third, watch the book for ten to twenty seconds and count how often the top-of-book quotes update.

The pattern recognition is:

- **Wide spread + thick depth + slow refresh + low trade frequency** → maker-dominated. Makers have time to think; nothing is forcing their hand.
- **Tight spread + thin depth + fast refresh (or no refresh at all because makers gave up) + high trade frequency** → taker-dominated. Takers are eating faster than makers can replenish.
- **Anything else** → neutral. Trade it like a normal contract; the regime is not telling you anything specific.

The shortcut command I use most mornings is `sf scan --by-spread asc --min-depth 200 --warm` to find taker-dominated candidates, and `sf scan --by-spread desc --min-depth 500 --warm` to find the maker-dominated ones. The first list is where I want to be the *maker* (because the existing makers have withdrawn and there is room to step in). The second list is where I want to be the *taker* (because the existing makers are pricing too defensively and crossing them is cheap).

## Three Real Examples

**Example A — KXFEDDECISION-26MAY01 (Fed cut in May), maker-dominated.** Tuesday morning, no news in the τ window. Bid 41¢, ask 45¢, depth $620 on bid and $580 on ask. Last twelve trades over twenty minutes. Quote refreshes within 200ms of any trade. This is a textbook MM camp — Susquehanna or one of the Kalshi-side market-making specialists is sitting on both sides and absorbing whatever organic flow shows up. My strategy on this contract is *taker if I have a thesis*. I cross the 4-cent spread once because I cannot beat the maker on price. I do not bother to post my own quote inside theirs; they will refresh before I get hit.

**Example B — KXBTCD-26APR07-95K (BTC above $95k by Apr 7 EOD), taker-dominated.** Late afternoon, BTC just printed $94,800. Bid 28¢, ask 30¢, depth $35 on bid and $20 on ask. Last forty trades in eight minutes. Top-of-book is being eaten and the next price level shows up two cents away. The maker that was here this morning has stepped back because the underlying is about to pin. My strategy on this contract is *maker, but only briefly*. I post a passive quote inside the displayed spread, expecting to get filled in the next few minutes by the next taker order, then immediately re-evaluate. I am not trying to camp this contract — I am trying to capture one or two ticks of spread while the regime is still inverted, then exit.

**Example C — KXIPOSPACEX-26JUL01 (SpaceX IPO files S-1 by Jul 1), neutral.** Mid-morning, no SpaceX news at all. Bid 14¢, ask 18¢, depth $90 on bid and $110 on ask. Three trades in the last thirty minutes. Quote refreshes are slow but present. Nothing is happening. Neither side is in clear control. My strategy on this contract is *whatever my thesis says*. The regime is not contributing information. I size based on conviction and let the indicator stack do the work.

## Why the Regime Determines Your Strategy, Not Your Thesis

This is the part most new traders miss. The regime is upstream of your thesis. Even a brilliant thesis on a maker-dominated contract loses to the maker most of the time, because the maker has structurally cheaper access to the spread than you do. Even a weak thesis on a taker-dominated contract can be profitable, because the regime is paying you to provide liquidity that nobody else will.

I size makers wider on contracts I cannot hedge, and I size takers tighter on contracts where the spread is being eaten faster than the makers can refresh. Both rules are about the regime, not about my view on the underlying outcome. When I forget to do this — when I let my conviction override my regime read — I end up paying retail spread on a maker-dominated book and getting exactly the outcome I deserve.

The cleanest way to think about it: the regime tells you *what kind of order you should be sending*. The thesis tells you *which side*. Decoupling the two is one of the more important habits in this kind of trading, because the regime changes faster than the thesis does.

## Cross-Connection to the Indicator Stack

The regime classification is not a replacement for the [pm-indicator-stack](/concepts/pm-indicator-stack); it is a layer on top of it. Specifically, the LAS (liquidity availability score) component of the stack is the closest single number to a regime indicator — high LAS usually means maker-dominated, low or null LAS usually means either taker-dominated or unscanned. But LAS is sparse on most of the universe, so the regime read as I have described it above is the version of the same question you can answer for any contract from public data.

CRI fits in here too. A taker-dominated contract almost always has a rising CRI (because price is moving fast and tau is shrinking). A maker-dominated contract often has CRI near zero. The pair (LAS, CRI) is roughly the same information as the regime classification, expressed as numbers instead of states. If you prefer the numerical view, use the indicator pair. If you prefer the categorical view, use this page. They both work.

## Where the Regime Read Breaks

Three failure modes worth naming.

**The book is fake.** On Polymarket especially, the displayed depth can include AMM-side liquidity that does not behave like maker depth — it has a curve. A "thick" book might be one trade away from showing you a much worse price because the curve flattens fast. The fix is to compute the *executable* depth at the size you actually want to trade, not the displayed depth.

**The regime flipped while you were watching.** Regimes are not stable on news-sensitive contracts. A market that was maker-dominated at 9:30 can be taker-dominated by 9:32 if a wire crosses. The regime read has to be re-done frequently — it is not a label you stamp on a contract once and forget. I re-classify any active position every fifteen to twenty minutes and any contract on my watchlist any time I notice a CRI tick.

**Regime classification on illiquid markets is meaningless.** A contract with three trades a day cannot be classified into any of the three states because there is not enough data. Most of the long tail of the Kalshi universe is in this category. Treat them as "unclassified" and rely on the rest of the indicator stack to decide whether to engage. The maker/taker frame is genuinely useful only on contracts with at least a few dozen trades a day.

A fourth case to flag: the regime tells you nothing about *direction*. A taker-dominated contract is being moved by takers, but the takers might be moving it the wrong way. The regime is about *who controls price discovery*, not about whether the price is going to the right level. Combine the regime read with your thesis; never substitute one for the other.

## How to Practice This

Pick three contracts from different categories — one Fed-decision contract, one BTC daily, one SpaceX IPO market — and watch all three for twenty minutes a day for a week. Write down which regime you think each one is in and why. At the end of the week, compare your reads against the spread and depth history in the snapshot table. You will be wrong on about a third of them at first. After two weeks of this you will be wrong on maybe one in ten, and the regime classification will start happening automatically every time you look at a book.

The goal is not to be right on the regime — the goal is to make the regime read fast enough that it precedes every order you send. Once it does, your strategy will be different on different contracts on the same morning, and the difference will not be arbitrary. It will be a function of which side of the book the contract is currently being priced from.

That is the whole point of this concept page. Most traders in this space have never named the maker/taker regime out loud, and as a result they trade everything as if every contract were neutral. Most contracts are not neutral. The rest of the indicator stack tells you *which* contract to trade; the regime tells you *how* to trade it.

For deeper context: see [liquidity-availability-as-the-real-edge](/opinions/liquidity-availability-as-the-real-edge) for the case that liquidity is the binding constraint on every prediction-market strategy, and see [adverse-selection-on-prediction-markets](/concepts/adverse-selection-on-prediction-markets) for the companion question of *why* the makers withdraw when they do.