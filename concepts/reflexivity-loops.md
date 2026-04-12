# Reflexivity Loops in Election Markets: When Price → Consensus → Price

> Soros named it. Election markets live it. The implied probability becomes the news, the news shapes new positioning, the new positioning shapes the price. In a reflexive market your edge erodes faster than the indicators say it does.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
George Soros has been writing about reflexivity since the 1980s. The core idea, stripped of the philosophical scaffolding, is that some markets are not just *measuring* the world they are also *shaping* it. The price is information about the underlying — but the price is also itself an input to the underlying, because participants react to the price. The feedback loop runs both ways, and in reflexive markets the loop dominates the simple "market reflects fundamentals" model that finance textbooks start with.

Election markets are the most reflexive prediction markets I know of. The implied probability of a Polymarket or Kalshi presidential contract gets quoted in news articles, cited by pundits, embedded in campaign donor outreach, mentioned by the candidates themselves. Every one of those mentions is a piece of information that flows back into the market and moves the price further. The feedback is not a metaphor — you can watch it run in real time during an election cycle. What follows is what the loop looks like, what to do about it, and where the reflexivity reading breaks down.

## The Loop, Step by Step

A US presidential election contract on Polymarket sits at 0.58 for candidate A. Some piece of news arrives — a poll, a debate moment, a fundraising number — and the market moves to 0.62. Within the hour, the new price is on the front page of three financial news sites and in two political newsletters with subject lines like "Polymarket now gives candidate A a 62% chance." Some readers see those headlines and think "the smart money is converging on A." A subset of those readers open a Polymarket account and buy YES on candidate A. That flow moves the price to 0.64. The new price gets cited in an evening news segment. More flow arrives. By the end of the day, the contract is at 0.67 and the original news event has been amplified into a five-cent move.

That is the loop in its compressed form: news → price → coverage → flow → price → coverage → flow. Each arrow is real and each one happens fast.

I want to be careful about the mechanism, because reflexivity gets used loosely. The loop requires three specific conditions:

1. The market price is **observable to non-traders** and gets cited in coverage they read.
2. The non-traders can **act on the price quickly** — open an account, deposit funds, place a position.
3. There is **enough flow** from non-trader entry to move the price meaningfully.

Election markets satisfy all three. Most other prediction markets do not. A Kalshi contract on "January CPI year-over-year above 3.2%" satisfies condition 1 partially (the price is in the data, if you go look) but not 2 (most CPI traders are already in the market) and not 3 (retail flow on CPI contracts is small). The CPI contract is *not* reflexive in the same sense, even though it is in the same orderbook system as the election contract.

## The 2024 Cycle as a Case Study

The 2024 US presidential election was the most reflexive cycle I have watched in real time on Polymarket. Through October, the contract on the eventual winner moved from around 0.45 to 0.60 in a sequence of five separate events — debate performances, polling shifts, momentum stories — and after every move, the new implied probability was cited within hours by the political press as "Polymarket says X now has Y% chance." The citations were not framed as "Polymarket's price is consistent with Y%" — they were framed as "the prediction market thinks X has Y% chance," which is a much stronger ontological claim, treating the price as a source of belief rather than a result of trading.

Once the press started treating the Polymarket implied probability as a source, the loop accelerated. Donations followed the price. Endorsements followed the price. Pundits adjusted their commentary in the direction of the price. Retail traders, opening accounts in volumes well above the venue's typical onboarding rate, kept pushing the price further in the same direction. By late October the contract had moved well outside the band that the polling data alone would have justified, and it stayed there until election night.

The reflexivity was empirically visible in the data. The five-day rolling correlation between the Polymarket implied probability and the headline poll average detached during the final two weeks. Polls were at one number; the prediction market was at a meaningfully different number; the gap closed only on election night when reality broke the loop.

This is the canonical case study for what reflexivity does to a market. It is not a story about the market being "wrong" — the market was probably right about the *direction* of the eventual outcome. It is a story about the *magnitude* being amplified by the feedback loop, in a way that made the implied probability poorly calibrated as a forecast of "what would polls say if there were no Polymarket." The price was simultaneously a measurement and a cause.

## The Practical Rule

You cannot trade against a reflexive loop with the same sizing you trade non-reflexive markets. Three rules I follow.

**Lag your sizing.** When a reflexive market moves, the temptation is to fade the move on the assumption that the underlying fundamentals have not actually changed. Sometimes you are right about the fundamentals and the move was pure reflexivity. The problem is the loop has not finished running yet. The price that looked too high yesterday will look even higher tomorrow as the news cycle absorbs the move and the next round of retail flow arrives. Your fade is correct in direction but wrong in timing, and the timing kills you. Lag your sizing — wait for the loop to exhaust itself, which usually takes two to four days, before adding to a fade position.

**Accept that your edge erodes faster.** In a non-reflexive market, you can find a 6¢ edge and reasonably expect it to stay open for a few hours while you build the position. In a reflexive market, the same 6¢ edge is closing in real time as the loop runs. By the time you have built the position, the edge is 3¢, and by the time you have sized to your target, the edge is 1¢. Position planning has to account for the fact that the entry is more expensive than the indicator suggests. I size reflexive trades at maybe 60% of what I would size a comparable non-reflexive trade, and I take the entry in two halves rather than one.

**Use the news cycle as a tape, not just as a fundamental.** In a reflexive market, the news *is* the trade in a literal sense — the news flow is what moves the price, regardless of whether the underlying reality is changing. Watching the news cycle is not optional. I keep a separate tab with the political newsletter that I trust the most to *anticipate* what will be in tomorrow's coverage, and I plan entries around when the coverage cycle is going to push the price further. This is the opposite of trading on fundamentals; it is trading on *what other traders will read tomorrow*.

## Where Reflexivity Reading Breaks

**Reflexivity is real but it is not unbounded.** The loop only runs as long as there is fresh retail attention to feed it. By the late stages of an election cycle, retail attention saturates — everyone who was going to open a Polymarket account already has — and the loop's amplification falls off. The reflexive amplification of a poll move in March is much larger than the reflexive amplification of the same poll move in October, because the marginal new participant in October is rarer.

**Reflexivity does not mean the market is wrong.** A reflexive market with a price that is "amplified" relative to fundamentals can still be right about the outcome. The 2024 example is instructive: the Polymarket implied probability was higher than polling data would have justified, *and* the eventual winner actually won. Reflexivity changes the size and timing of moves; it does not by itself create directional errors. Be careful about the assumption that "this market is reflexive, therefore the price is inflated, therefore I should fade." All three claims need separate support.

**Not every news-citation creates a loop.** The mention of a Polymarket price in a news article creates a loop only if readers can act on it. A New York Times article about prediction markets that does not link to the venue, does not explain the trading mechanism, does not provide an onboarding path, generates almost no inflow. The loop requires actionability, not just visibility. For non-US markets where Polymarket is not the dominant venue, even high-visibility coverage does not move the price in the same way.

A fourth caveat: reflexivity can run in either direction. The loop amplifies *whichever direction the news pushes the price*, not just up. A negative news cycle for a candidate creates the same loop in the opposite direction — price drops, coverage notes "Polymarket now gives X only 32%," more selling flow arrives. The loop does not have a directional preference.

## The sf CLI Read

There is no clean indicator for reflexivity, because the feedback loop is mediated through external coverage which is not in the market data. The closest proxy I use is the 24-hour [position-implied velocity](/learn/position-implied-velocity) (PIV) on the contract: when PIV is sustained at a high level for more than three days in a row on a politically prominent contract, the contract is almost certainly running in a reflexive loop. `sf scan --by-piv desc --category election` will surface these.

For the broader phenomenon catalog, see [longshot-bias](/concepts/longshot-bias) for the related question of how mass participation systematically distorts pricing, and [information-latency](/concepts/information-latency) for the time scale at which news reaches the price in *non*-reflexive markets. Reflexive markets have a different latency profile because the news cycle and the price cycle are coupled.

Reflexivity is the one phenomenon in this catalog that I actively struggle to trade. It does not respond to the indicators in the way the rest of the stack assumes, the loop runs faster than my position-building, and the news cycle drives the move more than the fundamentals do. Election markets are where my edge is smallest, and I think a lot of that is reflexivity. Soros built a career off recognizing this dynamic in equity markets; I do not claim to have his edge in prediction markets. But naming the phenomenon is the first step toward not trading it badly.