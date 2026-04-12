# The CRI Test for Stale Positions: When Your Market Hasn't Moved, You're Holding Noise

> High Cliff Risk Index plus a flat P&L is not patience. It is the universe telling you the move is happening to other contracts and you are sitting on the wrong one. Exit.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 9 min

---
I held a Kalshi Powell-pivot contract for nine days last March. The mid never moved more than half a cent on my side of the book. Every other contract in the same Fed-decision family — KXFEDCUTMAY-26MAY07-25, KXFEDCUTJUN-26JUN18-25, the cross-meeting overround — was repricing daily. Mine was a corpse. I told myself I was being patient. I was being noisy.

This is the essay I wish someone had written for me before that nine-day stretch. The thesis is one sentence: high CRI plus a flat position P&L is an exit signal, not a conviction trade.

## What "Stale" Actually Means in CRI Terms

Cliff Risk Index measures activity scaled by time remaining: `CRI = |Δp/Δt| × τ_remaining`. A high CRI on a market means the market itself is alive — prices are moving, volume is real, the orderbook is rotating. A stale position is the case where CRI is high on the *event family* and your specific contract is sitting still.

That gap is the signal. The market is doing work. The work is getting done somewhere. You are not where the work is happening.

Most traders frame "the market is moving but my position isn't" as evidence that they got in early. They will tell you the position is "coiling." They will use the word "patience." They will check the chart every twelve minutes for a week and tell themselves they are being disciplined.

I learned to flip the framing. When the family is hot and your contract is cold, what you are actually holding is a thesis the market has already rejected and is in the process of pricing out by ignoring it. The flat P&L is not a coiled spring. It is the market's way of saying "we considered this and we are buying the next-door contract instead."

## The Worked Example

Here is the trade I would not stop holding. Numbers are exactly what I had in front of me.

| Contract | YES Mid Day 0 | YES Mid Day 9 | τ Day 0 | 24h CRI Day 4 |
|---|:---:|:---:|:---:|:---:|
| A — Powell pivot signal at next FOMC YES | 0.31 | 0.31 | 22 | 0.001 × 18 = 0.018 |
| B — Cut at next FOMC YES | 0.42 | 0.55 | 22 | 0.04 × 18 = 0.72 |
| C — Cut by year-end YES | 0.61 | 0.74 | 198 | 0.018 × 194 = 3.49 |
| D — 50bp cut by year-end YES | 0.18 | 0.27 | 198 | 0.014 × 194 = 2.72 |

I owned contract A. The Powell-pivot signal contract. I had a thesis about Powell's tone at the post-meeting press conference and I was waiting for the market to wake up to it.

Look at the table. Contracts B, C, and D were all moving aggressively. Contract C had a 24h CRI of 3.49 against my contract's CRI of 0.018 — almost two hundred times the activity. The dovish thesis I held was being priced into every adjacent contract on the board. Mine was the one the market was actively *not* repricing.

By day 9 the cumulative move on the family looked like a real Fed pivot was being priced in. My contract had moved zero. Zero. The market had been told my exact thesis, multiple times, by multiple data points, and had chosen to express that thesis on different contracts.

What I held was not a coiled spring. It was a contract that the market had decided was the wrong vehicle for the trade I wanted to express.

## The Pivot to the Exit Rule

The rule I now run is mechanical. Every morning, for any open position, I run `sf scan --by-cri desc --event KXFEDCUT` (or whatever event family the position belongs to). If the position's own CRI is in the bottom quartile of its family while the family median CRI is above 0.5, the position is flagged. Three consecutive flag-days and I exit, full stop, no override.

This is not a stop loss in the price sense. It is a stop loss in the *attention* sense. I am cutting positions that the market has decided to ignore, even if my P&L on them is exactly zero. The cost of holding is not the dollar drawdown — it is the opportunity cost of having a thesis tied to a contract no one else cares about while three sibling contracts run away from me.

If I had run that rule in March, I would have exited contract A on day 5. I would have rotated into contract C on day 5. I would have caught the back half of the 13-cent move on C instead of the zero-cent non-move on A. The exit rule pays for itself the first time you correctly rotate.

## The Counterargument

The strongest pushback is real and I want to address it head-on. The argument goes: "Conviction trades take time. The whole point of having a thesis is that you hold it through periods when the market disagrees. If you exit every time the market ignores your contract for a week, you have no edge — you are just chasing whatever is moving today."

I held this view myself for two years. It is wrong because it conflates two different forms of patience.

The first form of patience is *waiting through a drawdown on a contract the market is actively pricing*. Your contract has a CRI of 1.2, the move is going against you, and you are betting the market is wrong. This is real conviction. The market is engaging with your thesis and rejecting it; you are betting your engagement is more correct than the market's. Hold this trade. Take the drawdown. The convicted size of your bet is exactly what you should be willing to lose.

The second form is what I am calling *staleness*: your contract has a CRI of 0.02, the family CRI is 1.5, and the move is happening on the contracts adjacent to yours. There is no engagement here. The market is not rejecting your thesis. The market is choosing a different *contract* to express your thesis. That is not a drawdown to wait through. That is a vehicle mismatch, and it is solvable today by rotating to where the activity is.

The CRI test separates the two cases. A position taking a drawdown with a high CRI is conviction. A position sitting flat with a low CRI while its family is on fire is staleness. The first you hold; the second you exit.

The other counterargument I get is "but the move on my contract might come last." Maybe, occasionally. But this is asking for a specific kind of luck — that the market both agrees with your thesis *and* eventually lands on the exact contract you happened to pick. Two coincidences at once. The base rate of that happening, in my own logs, is roughly one in twelve.

## Where the Test Breaks

Three caveats before you put this in a script.

First, the CRI test is meaningless on event families with fewer than three sibling contracts. With one contract, there is no comparison. With two, the noise is too high. The test wants at least three siblings, ideally five, to compute a stable family median.

Second, the test breaks on event families where one contract is structurally illiquid relative to its siblings. If your contract's CRI is low because no one trades it (LAS = null, no orderbook depth), that is a different problem from staleness. You are not in a stale position; you are in an illiquid one. The fix is different — exit if the illiquidity makes the position un-actionable, but do not generalize.

Third, the test is one-sided. It only tells you when to *exit* a stale position. It does not tell you what to rotate into. The rotation logic is a separate question and depends on which sibling has the best executable edge after fees, which is its own analysis. CRI gets you out. The next question gets you back in.

## The Habit

The habit is one query a day. Pull every open position. For each one, compute the family CRI and your contract's CRI. Compare. If you are in the bottom quartile of family activity while the family is hot, you are stale. Three days stale and you exit.

This is the cheapest discipline I have ever installed in my trading. It costs nothing to compute, it runs in two seconds, and it has saved me from the same nine-day mistake roughly six times since I built it. The dollar value of the saves is real but the bigger value is what it does to your sense of what "patience" means. Patience is the willingness to take a drawdown on a contract the market is engaging with. It is not the willingness to stare at a stale mid-price for nine days while the rest of the family runs.

The next time you find yourself defending a flat position with the word "patience," check the family CRI before you finish the sentence. If the family is hot and you are cold, you are not patient. You are holding noise.