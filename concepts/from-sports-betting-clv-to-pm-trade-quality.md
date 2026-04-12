# From Sports Betting CLV to Prediction Market Trade Quality

> Closing Line Value is the gold standard for sharp sports bettors. PMs do not have a closing line in the same sense. Here is the PM-native equivalent and how to track it without fooling yourself.

**Category:** theory | **Author:** Patrick Liu | **Reading time:** 12 min

---
If you ask a professional sports bettor how they know they are sharp, they will not tell you about win rate or ROI. They will tell you about CLV — closing line value. CLV is the difference between the price they got and the price the market closed at, and it is the only number that survives the noise of variance well enough to be a real signal. A bettor who consistently beats the closing line is sharp by definition. A bettor who does not is gambling with extra steps.

I came to prediction markets carrying that discipline and immediately ran into the obvious problem: prediction markets do not have a closing line in the same sense. The contract closes at $1.00 or $0.00 — there is no meaningful "fair price at the moment of resolution" to compare your entry against. So I had to invent a PM-native equivalent. This page walks through what CLV is, why it works in sports, why the literal version does not work on PMs, and what the right replacement looks like.

## What CLV Is in Sports Betting

A sports bet is placed at some moment before the event, at some price. The market continues to move after your bet — sometimes toward you (your side gets more expensive), sometimes against you (your side gets cheaper). The price at the moment the market closes — typically the moment the event begins — is the "closing line."

CLV is the difference between your entry price and the closing line, expressed as a probability or implied edge. If you bet a -110 favorite and the closing line was -125, you got CLV of about 4 percentage points. If you bet a -110 favorite and the closing line was +100, you gave up about 10 percentage points of CLV.

The reason CLV is a useful signal — and the reason every sharp bettor obsesses over it — is that *the closing line is the most efficient price the market produces*. By the time the event starts, all the information has been incorporated, all the sharp money has flowed in, and the price reflects the wisdom of every bettor who had an opinion. If you consistently beat the closing line, you are systematically getting prices the market eventually agrees were too good — which is the definition of being a sharp bettor.

Crucially, CLV is *independent of whether you won or lost*. A sharp bettor can have a 10-bet losing streak and still be beating the closing line on every bet, and the variance will eventually reverse and the underlying skill will show through. Tracking CLV separates skill from luck in a way that win rate never can.

## Why the Literal Version Does Not Work on PMs

A prediction market resolves to either $1.00 or $0.00. There is no meaningful "fair price" at the moment of resolution — the contract has crystallized to one of two extreme values, and the market price in the final seconds is just the trader consensus on which way it will land, which is approximately the same as the actual outcome.

You cannot compare "I bought YES at $0.42" against "the contract closed at $1.00" and call the difference CLV — it is just the realized profit from being right, which is a function of both your edge and pure variance. CLV in sports works precisely because the closing line is a *price* and not an *outcome*. The closing line on a -125 favorite still has uncertainty in it — the favorite still loses 44% of the time. That residual uncertainty is what makes CLV a measure of pricing skill rather than win rate.

Prediction markets do not have a residual-uncertainty price at resolution. The "closing line" — the price one second before settlement — is essentially the realized outcome. It is not a fair price in any meaningful sense. So importing CLV literally fails: the closing line is not a useful comparison point because it is degenerate.

## The PM-Native Equivalent: Forward-Window Drift

The replacement I have settled on is what I call *forward-window drift*. Instead of comparing your entry price to the closing line, you compare it to the price at fixed forward windows: 1 hour after entry, 24 hours after entry, 7 days after entry. The windows have to be short enough that the price has not yet been dominated by the resolution outcome (i.e., far enough from τ = 0 that there is still residual uncertainty), and long enough that the market has had time to react to whatever you saw when you entered.

The logic is the same as CLV. If you systematically buy at prices below where the market is one hour later, you are reading the market faster than other traders. If you systematically buy at prices below where the market is 24 hours later, your edge is real on a longer horizon. If you systematically buy at prices below where the market is 7 days later, your thesis is structural and not just a fast-fingers play.

The metric I track is:

```
FWDₚ = (price_at_t+Δ - price_at_entry) / 1.0
```

Where Δ is one of {1h, 24h, 7d}, and `price_at_entry` and `price_at_t+Δ` are both YES prices on the same contract (or equivalent NO prices, with the sign flipped). I track the median across all my entries, segmented by window length and by my own thesis category. If FWD₁ₕ is consistently positive on my "Fed-decision" thesis bucket but consistently zero on my "ballot-initiative" thesis bucket, I have learned something concrete: my Fed reads are sharp and my ballot reads are not.

## Why the Window Length Matters

A 1-hour window catches mostly *fast information*. You read a tweet, you put on the position, the rest of the market reads the same tweet 30 minutes later, your 1-hour FWD is positive. This is real edge but it is fragile and it is heavily dependent on how fast you can read the news. Most people cannot beat the market on a 1-hour window because the market is full of bots and traders who are watching the same feeds.

A 24-hour window catches *thoughtful information*. You read a Bloomberg piece in the morning, you build a thesis over coffee, you put on the position, the rest of the market figures it out by the next morning. This is the "I am sharper than the average trader" version of CLV and it is the most useful FWD for measuring your skill.

A 7-day window catches *structural information*. You build a thesis based on macro analysis or causal-tree decomposition, you put on the position, the market reaches your thesis price over the course of a week. This is the longest-horizon version and it is the one that distinguishes traders who actually have a model from traders who are just reacting to news flow.

I track all three windows separately because they measure different things, and a trader who is good at one is not necessarily good at the others. My own profile is: weak on 1h (I am not a fast-fingers trader), moderate on 24h (I am about average at thoughtful information), strong on 7d (my edge is on structural reads, not fast ones). Knowing this changes how I size positions — I size big when my 7-day FWD has been positive on a thesis bucket and small when it has not.

## A Worked Example

Take a Kalshi Fed-decision contract that I bought at $0.42 with about 30 days to expiry. The price trajectory after my entry was:

| Time | YES Price |
|---|:---:|
| Entry | 0.42 |
| +1h | 0.42 |
| +24h | 0.43 |
| +7d | 0.46 |
| +30d (resolution) | 1.00 |

FWD₁ₕ = +0.00 (no fast read).
FWD₂₄ₕ = +0.01 (slightly positive 24-hour drift).
FWD₇d = +0.04 (meaningful 7-day drift).
Realized return = +138% (paid out fully on resolution).

If I track this contract in my CLV-equivalent log, I see that my entry was *good but not because I read the news faster than anyone* — the 1h drift is zero. The edge showed up over a longer horizon, which is consistent with a structural thesis. The realized 138% return is mostly luck — if the contract had resolved NO, I would have lost the entire $0.42, and the realized PnL would have been -100%, but the *quality* of the entry would have been the same as measured by the forward windows.

This is the same logic CLV uses in sports: the realized outcome is variance, the forward-window drift is skill. Tracking the latter independently of the former is what separates sharps from gamblers.

## How This Avoids the Variance Trap

A typical PM trader looks at a 70% win rate and concludes they are sharp. The problem is that win rate is a function of *what prices they trade at*. A trader who only buys 95¢ contracts will have a 90%+ win rate by definition and zero edge. Win rate is the wrong unit for measuring skill.

Forward-window drift solves this by being *price-independent*. A 4-cent drift over 7 days is a 4-cent drift whether you bought at $0.05 or $0.95. It measures *how the market moved relative to your entry*, not whether you happened to be right about the eventual outcome. The variance reduction is significant — with 100 trades, you can detect a 1-2 cent average drift with high confidence, which is what you need to know whether your edge is real.

## What Does Not Port: Steam Detection

In sports betting, "steam" is the phenomenon of a price moving sharply because of a known sharp bettor or syndicate placing a large bet. Steam detection is the second-most-tracked metric after CLV among professional sports bettors. PMs do not have steam in the same sense — the trader composition on Kalshi and Polymarket is more retail-heavy and sharp money is less concentrated, so price moves rarely exhibit the clean steam pattern. The closest PM analog is the [steam-moves-across-venues](/concepts/steam-moves-across-venues) phenomenon, which is when a price moves on Kalshi shortly before the same outcome moves on Polymarket (or vice versa).

## What Does Not Port: Bookmaker Limits and Exchange Behavior

Sharp sports bettors get banned or limited by sportsbooks once they show too much CLV. Kalshi and Polymarket are exchange venues, not bookmakers, and they do not care whether you are sharp — they make money on transaction fees regardless. The sharp PM trader does not need to hide their action. This is one of the few advantages PMs have over sportsbooks, and it changes the day-to-day operational profile meaningfully.

PMs are also CLOB or AMM systems where you can post your own quote across the spread, which means FWD has to be computed relative to your *fill price*, not the mid-market price at the moment you placed the order. If you posted a maker bid at $0.40 and got filled at $0.40 because someone hit it, your forward window drift starts from $0.40.

## How to Build This for Yourself

The tooling is straightforward. After every trade, log:

- The contract ticker
- The entry price (your fill price, not the mid)
- The entry timestamp
- The thesis category (your own taxonomy: "Fed", "election", "sports", "ballot", whatever)

A cron job pulls the price at 1h, 24h, and 7d after each entry and computes the FWDs. A weekly review aggregates by thesis category and shows the median drift for each window.

I do this with `sf scan --my-trades --window 24h` which gives me a sorted list of my recent entries with the 24-hour drift attached. The interesting line is the median across all my "Fed-decision" trades for the last 30 days. If that median is positive, my Fed reads are sharp this month. If it is negative, I tighten up sizing on Fed contracts until I figure out what changed.

The discipline is exactly the same as CLV tracking in sports. You track the metric, you trust the metric over your subjective sense of how you are doing, and you let the metric tell you when to cut sizing or change your process.

## Where the Bridge Breaks

A few cases where forward-window drift fails as a CLV equivalent.

**Resolution-proximate trades.** If you enter a contract within τ = 5 days of resolution, the forward windows quickly become dominated by the resolution outcome rather than residual uncertainty. The 7-day FWD is meaningless because the contract has already resolved. Use only the 1h and 24h windows for short-tenor trades, and accept that you have less signal.

**Illiquid contracts.** If the contract has LAS = null or very thin depth, the price 1 hour after your entry may be a stale quote rather than a real market price. The drift is then noise. Filter forward-window analysis to contracts with non-null LAS to avoid this.

**Maker fills on stale quotes.** If you post a maker quote and someone hits it 6 hours later because they urgently needed to exit, the entry price you got was *better* than the prevailing market price, and the FWD is artificially positive. This is a real edge — you should not be ashamed of it — but it is a different kind of edge than reading the market correctly. Track maker fills separately from taker fills.

**Selection bias from cancelled orders.** If you cancel orders that did not fill before a positive price move, you only count your entries that *did* fill, and the population is biased toward situations where the market did not move quickly in your favor. The fix is to log cancelled orders separately and audit periodically.

## The Honest Summary

Sports-betting CLV is the single most valuable discipline I imported from my own pre-PM trading life. The PM-native version — forward-window drift — is roughly 90% as good and serves the same purpose: separating skill from variance, identifying which thesis categories you are actually edge-positive on, and resisting the temptation to size up after a lucky win or size down after an unlucky loss.

Track your entries. Compute the 1h, 24h, and 7d drift on every one. Aggregate by thesis category. Trust the metric over your subjective sense of how you are doing. The CLV mindset is the closest thing professional gambling has to a science, and the science ports to PMs with a small adjustment for the absence of a meaningful closing line.

For the broader case that PMs need a fixed-income vocabulary rather than a sports-betting vocabulary, see [/blog/prediction-markets-need-fixed-income-language](/blog/prediction-markets-need-fixed-income-language) and [implied-yield-vs-raw-probability-bond-markets](/opinions/implied-yield-vs-raw-probability-bond-markets). The CLV discipline is one of the few exceptions to that argument — sports betting has built better tooling for measuring trader skill than fixed income has, and that tooling is worth porting even though the rest of the sportsbook vocabulary is not. CLV is a sports-bettor concept that prediction markets should adopt and rename as forward-window drift, not a fixed-income concept that they should ignore.