# Why Parimutuel Intuition Fails on Order-Book Prediction Markets

> Horse racing pools dilute payouts as more bets stack on a winner. Kalshi and Polymarket continuous limit order books do not. The payoff is a fixed dollar; the thing that shrinks is the edge, not the payout.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 10 min

---
Sports bettors who come to prediction markets carry a specific intuition with them: "if everyone is on the same side, the payout shrinks." That intuition was built in horse-racing parimutuel pools, where it is mathematically correct. It is also wrong on Kalshi and Polymarket, where the mechanics are completely different.

This page is about the difference between parimutuel and continuous-limit-order-book pricing, why the intuition does not transfer, and what the right mental model is for "the crowd is on my side" on a CLOB prediction market.

## How Parimutuel Pools Actually Work

In a horse-racing parimutuel pool, every dollar wagered on every horse goes into a common pot. After the race, the track takes its cut (the takeout, typically 15-20%), and the rest of the pot is divided among the people who bet on the winning horse, in proportion to how much they bet. Your payout is:

```
payout = (your_bet / total_bets_on_winner) × (pool_size − takeout)
```

The more dollars are stacked on the winning horse, the smaller your share of the pot. If $100 is bet on Horse A and Horse A wins, and the pool minus takeout is $1,000, every dollar bet on A pays out $10. If $500 is bet on A instead, the pool stays at $1,000 and every dollar pays out $2. The payoff on a parimutuel bet is *inversely proportional to how much capital is on the same side*. This is the dilution effect, and it is the reason horse-racing bettors care about being on a horse that is *not* a public favorite — being on a horse that is loaded with public money makes the payout smaller even when you win.

Parimutuel pools have one more property worth noting: the price (the implied odds) is not stable. As more money flows in, the displayed odds update continuously, and the final odds you bet against are not necessarily the odds that displayed when you bet. There is a "morning line" estimate but the actual payoff depends on the *closing* pool composition, not on the pool composition at the moment of your bet.

This dynamic builds a specific intuition: "if everyone piles onto my side, my payout gets crushed." For a horse bettor, this is real, mechanical, and correct.

## How a Continuous Limit Order Book Actually Works

A Kalshi or Polymarket binary contract trades on a continuous limit order book (CLOB), which has nothing in common with a parimutuel pool. The mechanics are:

There is an order book with bids and asks at various price levels. A bid at 38¢ is "I will buy 100 contracts at 38 cents apiece, putting up $38, and if the contract resolves YES I get $100." An ask at 42¢ is the opposite. When a buyer at 42¢ meets a seller at 42¢, a trade happens at 42¢ for some quantity.

Crucially, the *payout* on a winning contract is fixed at $1.00 per contract, regardless of how many other people also bought. If you bought 100 contracts at 42¢ and the contract resolves YES, you get $100. Period. The fact that 10,000 other people also bought the same contract does not change your payout one cent. The pool is not shared. The contract is a *standalone instrument* with a fixed payoff.

What does change as more buyers come in is the *price*. If the contract was at 42¢ when you bought and the next thousand buyers all bid up to 50¢, the price is now 50¢. The next person to buy pays 50¢ for the same $1.00 payoff. Your payoff is still $1.00. Their payoff is $0.50 if it resolves YES (because they paid 50¢ to get there).

This is the structural difference: in parimutuel, the *payout* dilutes as more people are on your side. In a CLOB, the *entry price* moves as more people are on your side, but the payoff for any given purchase is locked in at the moment of purchase. The crowd does not eat your payoff. The crowd eats your *entry edge*, by moving the price toward consensus before you can act.

## The Practical Difference

In horse racing, "everyone is on my horse" means "my payout is going to be small." In a CLOB PM, "everyone is on my side" means "I had better have gotten in early, because the price has moved and the next buy is at a worse level than the first buy."

These two statements look similar at first glance and they are completely different in practice. The horse-racing version is about *outcome dilution* — the winning bettors have to share a fixed pot. The CLOB version is about *price discovery* — each contract pays a fixed amount but the price you pay to enter is rising. In horse racing, you can be the millionth person on the winning horse and your payout will still be defined (small but defined) by your share of the pot. In a CLOB, you cannot be the millionth person to enter at the original price, because by the time the millionth person tries to enter, the price has moved to the level where supply meets demand, and that level is no longer the original price.

The implication for trading is exactly opposite in spirit. A horse-racing bettor wants to find horses that the crowd is *not* on, because being early gives you a price advantage *and* a payout advantage (the pool has not been diluted by public money yet). A CLOB PM trader wants to find contracts that the crowd is not yet on for a different reason — being early gives you a price advantage *only*, because the payout is fixed regardless. The crowd-avoidance instinct is similar; the underlying mechanic is different.

## Where the Confusion Costs Money

I have watched the parimutuel intuition cost money in two specific ways.

The first is *over-fading consensus contracts*. A horse-racing person sees a Kalshi contract at 78¢ where "everyone is on YES" and instinctively wants to take the other side, because in horse racing the lopsided side is the bad bet. On a CLOB PM, the lopsided side is just the side the market thinks is more likely, and unless you have a specific reason to disagree, taking the other side is just betting against the consensus for no reason. The price is 78¢ because the aggregate of all participants thinks the YES probability is around 78%. There is no payoff dilution to take advantage of by going against the crowd. There is only the standard "is the market wrong" question, which has nothing to do with parimutuel dynamics.

The second is *misunderstanding the meaning of volume*. In horse racing, "high volume on this horse" is information about *pool dilution*. In a CLOB PM, "high volume on this contract" is information about *liquidity and price discovery* — high volume means the price has been actively negotiated and is more likely to be efficient, not that your payout is going to be smaller. Treating high-volume contracts as "diluted" leads people to favor low-volume contracts on a parimutuel-style intuition, which is exactly backwards on a CLOB — low-volume contracts are *less* well-priced because fewer people have negotiated against them, not more rewarding because the pool is less crowded.

## The Real Risk on a CLOB: Edge Decay

The right thing to be afraid of on a CLOB PM is not pool dilution but *edge decay through price discovery*. Here is the dynamic.

Suppose you have a real edge — say, you have a model that says a contract is currently mispriced at 40¢ and should be at 60¢. If you act fast, you buy at 40¢ and hope the price moves to 60¢ over the next few days, capturing 20 cents of edge. But if you wait, other traders with similar models or similar information do the same thing, and the price moves to 50¢ before you act. Now your edge is 10 cents instead of 20. If you wait longer, the price moves to 58¢ and your edge is 2 cents. By the time the consensus has fully formed, the price is at your fair value and there is no edge left to capture.

This is the CLOB version of "the crowd hurts you" — but it hurts you by closing the *price gap* you were trying to exploit, not by diluting a shared pot. The asymmetry matters because the right defenses are different. Against parimutuel dilution, you want a horse the crowd is not on. Against CLOB edge decay, you want to *be* the early money — the first person to act on the information, because by the time the next person acts the price has moved. Speed and information edge matter; crowd avoidance is incidental.

This is also what I mean when I say things like "the price is moving at high CRI and the trade is getting away from you." See [cliff-risk-index](/concepts/cliff-risk-index) for the metric and [the-cri-test-for-stale-positions](/opinions/the-cri-test-for-stale-positions) for the opinion piece on how to use it. High CRI on a contract you wanted to trade *against* is the signal that consensus is forming and the edge is decaying. Low CRI is the signal that consensus has not yet formed and you may still have a window.

## A Quick Worked Example

Take a hypothetical Polymarket contract on "AI agent benchmarks beat human level by end of 2026" trading at 0.18 with τ = 270 days. Your model says it should be at 0.32. Naive 14-cent edge.

In the parimutuel mental model, you would worry "if everyone gets on this trade, my payout will dilute, so I want a contract where the crowd is not on my side yet." That worry, on a CLOB, is misdirected — the *payout* is locked in at the moment you buy, regardless of who else buys later.

In the CLOB mental model, you would instead think: "the price is at 0.18 because the active orderbook participants currently aggregate to around 18% probability. If my model is right and a few other modelers notice, the price will move toward 0.32 over the next several days or weeks. I want to be the early money. My edge is the difference between 0.18 and where the price is when I'd have to exit if my model is wrong, or where the price is when consensus catches up if I am right."

The trade is the same (buy YES at 0.18), but the *reason for being early* is different (price discovery, not pool dilution), and the right metric to monitor afterwards is different too (CRI for "is the price actively moving" and PIV from [position-implied-velocity](/learn/position-implied-velocity) for "is positioning shifting"), not "how many other people have bet on my side."

## What Actually Does Port from Sports Betting

I want to be careful not to overcorrect. Sports betting has produced a lot of intuition that *does* port to PMs, even though the parimutuel-specific intuition does not.

[Closing Line Value](/concepts/from-sports-betting-clv-to-pm-trade-quality) — the discipline of measuring whether the price moved in your favor after you bet — is one of the most useful concepts a sharp sports bettor can bring to PMs. CLV translates because it is about *price discovery*, which exists on both market structures.

The general idea of *avoiding markets the public is heavily on without specific reason* also ports, but for a different reason — the CLOB version of "public money is on the wrong side" is "the price has moved past where the math says it should be, and you have an opportunity to take the opposite side." That is just standard mean reversion, not parimutuel dilution.

The concept of *steam moves* — sharp money hitting a market in a coordinated way — also ports, and is the topic of [steam-moves-across-venues](/concepts/steam-moves-across-venues). Sharp money moves the price in a CLOB just like it moves the line in a sportsbook; the mechanics differ but the signal is similar.

## The Punchline

Continuous limit order book prediction markets are not parimutuel pools. The payoff is fixed; the price is what moves. "Everyone is on my side" hurts you by closing the price gap before you can act, not by diluting a shared pot. The right defense is speed and information edge, not crowd avoidance. And if you came from sports betting, the most valuable things to import are *closing line value* and *steam* — not the pool-dilution instinct, which simply does not apply.