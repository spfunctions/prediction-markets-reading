# Liquidity Availability Score (LAS)

**Liquidity Availability Score is a per-market scalar that captures whether a prediction-market orderbook is thick enough to actually trade against, computed as the sum of bid depth and ask depth scaled by the bid-ask spread. LAS is null on roughly 99% of markets, because the warm-regime cron only computes it for the top 500 by 24-hour volume — and that null is itself the most useful signal LAS produces.**


## Explanation

## What the Number Captures

Every other indicator in the stack tells you *what* to trade. LAS tells you *whether you can*. The price can imply a 60% IY and the cliff risk index can flag a structural reassessment, but if the orderbook is twelve cents wide with two contracts on each side, the displayed mid-price is fiction. LAS is the metric that catches that case before you put an order in and discover the spread the hard way.

The formula is intentionally crude:

```
LAS = (bid_depth + ask_depth) / spread
```

Where `bid_depth` and `ask_depth` are the total contract counts at the top N levels of the book (default N = 5), and `spread` is the absolute difference between best bid and best ask in cents. A book with 200 contracts on each side and a 1¢ spread gives LAS = 400. A book with 12 contracts on each side and a 12¢ spread gives LAS = 2. Same headline price, two completely different positions.

## Why Most Markets Have LAS = Null

The warm-regime cron only computes LAS for markets that the top-volume scanner has touched in the last 24 hours. That cap currently sits at 500 markets — a conscious decision to keep the cron job under its compute budget. The other ~46,500 markets in the universe carry `las = null`.

This is the part that confuses new readers, so it is worth being explicit. Null does not mean "data missing." Null means "no one has been looking at this market." Which is exactly the entry condition for the virgin-Polymarket maker strategy: a market that no taker has wandered into yet, where you can post quotes wide and still be the only resting order on the book. LAS = null is a positive selector for that strategy, not a defect.

## How to Use It

Two main reads. First, on the markets where LAS *is* computed, sort descending: high LAS is the universe of contracts you can actually trade larger size in without paying through the spread. Second, on the markets where LAS is null, treat the null itself as the signal — those are the unloved markets where a maker can put up the first quote.

The mistake to avoid is treating LAS as a single number to maximize. It is a *gate*: a contract that fails the LAS gate is one you cannot execute at the displayed price, no matter how attractive everything else looks. Use it to throw out the false positives from your IY/CRI scan, not to rank the survivors.


## Example

Three Kalshi contracts that came up on a recent scan:

| Contract | Bid Depth | Ask Depth | Spread | LAS |
|---|:---:|:---:|:---:|:---:|
| KXFEDDECISION-26MAY-CUT | 380 | 420 | 1¢ | 800 |
| KXSPACEX-26IPO-YES | 45 | 60 | 4¢ | 26 |
| KXOBSCURE-26-EVENT | null | null | null | null |

The first contract has LAS = 800. You can put up 200 contracts at the bid without moving the book. The second has LAS = 26. Your 200-contract order will eat through three price levels and cost you ~3¢ in slippage on top of the 4¢ spread. The third returns null because the warm-regime cron has never warmed this market.

The actionable read on contract three is the opposite of what most people guess. It does not mean "broken data" — it means "this is a market no one has bothered to compute liquidity for in the last 24 hours." If your strategy is to be the first resting quote on an unloved market, that null is exactly the screen you want. If your strategy is to take liquidity, the null tells you to skip and look elsewhere.

The composite read across the three contracts: take size on the first, skip the second, and either skip or maker-quote the third based on which game you are playing.


## CLI

```bash
sf scan --by-las desc --warm
```


## Related

[orderbook](orderbook.md), [depth](depth.md), [spread](spread.md), [slippage](slippage.md), [null-as-signal](null-as-signal.md), [liquidity](liquidity.md), [executable-edge](executable-edge.md), [market-maker](market-maker.md)
