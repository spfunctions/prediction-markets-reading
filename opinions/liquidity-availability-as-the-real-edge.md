# Liquidity Availability Is the Real Edge in Prediction Markets

> Every other indicator describes which contract to trade. Liquidity Availability Score describes whether you can trade it at all. Most strategies that look beautiful on paper die at LAS, and that is why LAS is the only edge that matters.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 11 min

---
The first 60% implied yield trade I tried to put on had 12 cents of total ask depth. I wanted to size it at $400. I filled $32 and the next ask was three cents above the one I cleared. The remaining $368 of my intended position was a phantom on a backtest that did not understand orderbooks.

I spent the next year rebuilding every screen, every alert, every backtest, around one principle: liquidity is the only edge that survives contact with the venue. The thesis is brutal and I want to state it cleanly. Liquidity Availability Score is the real edge in prediction markets. Everything else is a thought experiment that gets you a screenshot.

## What LAS Measures

Liquidity Availability Score is bid depth plus ask depth, scaled by spread. The formula in the obvious form:

```
LAS = (Σ bid_depth + Σ ask_depth) / (1 + spread_cents)
```

Bid depth is the dollar size you can sell into without moving the market past your reservation price. Ask depth is the dollar size you can buy at the current ask. The spread divisor penalizes wide books — a contract with $400 of depth on each side is worthless if the bid is at 32 and the ask is at 41, because nine cents of slippage destroys most prediction-market theses.

The score is null on roughly 99% of the universe. That is not a bug. It is the warm-regime cron only computing LAS for the top 500 markets by 24-hour volume, because the orderbook fetch is rate-limited and the rest of the universe genuinely has no orderbook to score. The null is information, and I will come back to it.

## Why Every Other Indicator Lies Without LAS

Implied Yield tells you the annualized return you would receive *if you could fill at the displayed mid*. Cliff Risk Index tells you the activity *at the displayed mid*. Event Overround tells you the multi-outcome arbitrage *at the displayed mids*. Notice the pattern. Every indicator in the stack is calibrated against a mid-price that may or may not represent any executable trade.

When LAS is healthy — say, $5,000 of total depth at a one-cent spread — the mid is real. You can transact at it. The indicators are honest.

When LAS is unhealthy — $80 of depth at a six-cent spread — the mid is a polite fiction. The actual trade you can put on is at a price that destroys whatever IY/CRI/EE edge the screen showed you. Your scan said "60% IY, top of the board" and your fill said "30% IY on a tenth of the size you wanted, with the remaining 90% sitting in a queue you will never clear."

This is the gap between paper edge and book edge. LAS is the metric that closes the gap before you put the order in.

## The Worked Example

Two Polymarket contracts crossed my screen in February. Both had IY north of 80% on the displayed mid. Both came up in the same `sf scan --by-iy desc --venue polymarket` output.

| Contract | YES Mid | τ (days) | Displayed IY | LAS |
|---|:---:|:---:|:---:|:---:|
| A — Mayor primary in city of 240k YES | 0.38 | 51 | 89% | 0.04 |
| B — Senate special in mid-size state YES | 0.41 | 47 | 76% | 4.8 |

Contract A had the higher displayed IY (89% vs 76%) and was the more attractive line on a naive scan. Contract B had a much lower displayed IY but a *fundamentally different* LAS — almost 120 times more depth-per-spread.

I put on contract B in size. The fill was within half a cent of the displayed mid. The actual realized IY came in at 73%, three percentage points off the displayed number — basically perfect for an executable trade. The position behaved exactly like the screen said it would.

I tried to put on contract A in the same dollar size. The available ask absorbed about 8% of my intended position before the next price level was four cents higher. To get the rest of the position on at any price, I would have had to walk the book from 38 cents to 45 cents, paying an effective average of 41.5 cents — and at 41.5 cents the IY drops from 89% to 56%. That alone is a third of the apparent edge gone, and I have not even counted the time it takes for the rest of the orders to fill, the front-running risk from posting visible interest in a thin market, or the resolution-day uncertainty.

The displayed IY ranking told me to trade A first. The LAS-aware ranking told me B was a real trade and A was a paper one. The book agreed with LAS.

## Why People Skip This

The reason people skip LAS is that it feels boring. It is not a clever signal. It is not a thesis. It is not the kind of insight you can tweet. LAS is just *can the orderbook handle the trade you want to put on*, and the answer is "no" most of the time, which is unsatisfying.

The other reason is that LAS being null on 99% of markets feels like a coverage problem. Every new user of the system sees LAS = null on the first market they look at and assumes the data is broken. It is not broken. The warm-regime cron has a budget. The budget is spent on the markets where LAS is most likely to matter — the top of the volume distribution. Everything else is intentionally not scored, because there is nothing useful to score.

Once you internalize that the null is a *deliberate signal of un-warmed-ness*, you stop reading it as missing data and start reading it as "this market has not been touched enough to deserve a depth model." That reframing is critical for the next section.

## The Virgin Polymarket Exception

Here is the one case where LAS = null is the entry condition rather than the exit condition. The maker strategy I run on Polymarket — what I call "virgin polymarket" — depends on finding contracts that *no one has been looking at*. The signature is: LAS = null, CYC family of one (no sibling markets to anchor the price), CRI = 0 because nothing is moving.

A market with LAS = null is a market the warm cron has not touched. A market the warm cron has not touched is a market in the bottom 99% of the universe by 24-hour volume. A market in the bottom 99% by volume is a market where, if you post a tight market-maker quote, you may be the only quote on the book for hours. The whole point of the strategy is that you become the price discovery mechanism, not that you trade against an existing one.

In this case, LAS = null is not "I cannot trade this." It is "the orderbook is empty enough that I can become the orderbook." You post bid and ask, you wait, you collect the spread on whatever volume materializes, and you accept that you may sit on the position for a long time.

This is the complete inversion of the LAS-as-edge framing, and it is also the only place where it makes sense to *want* a null score. Outside of that exact maker context, LAS = null is a "do not trade" signal.

## The Counterargument

The strongest objection is from people who run small, slow strategies and never need orderbook depth. They say: "I am putting on $25 positions. I do not need $5,000 of depth. LAS doesn't apply to me."

This is half right. If you genuinely never scale beyond $25 per position, you can ignore LAS *for execution*, because $25 fits inside almost any displayed orderbook. But you still cannot ignore LAS for *exit*. The position you fill at $25 has to be exited at some point, either at resolution or before. If the orderbook on the other side is empty when you want to exit early — say, your thesis broke and you want out — you are stuck holding the position to resolution whether or not that was your plan.

The cleanest version of the counterargument is "LAS only matters if you ever want to scale or ever want to exit early." Both of those are true almost always. The only strategy that genuinely does not care about LAS is "buy and hold to resolution at fixed size below the depth of every venue you trade." That is a tiny strategy, and even then it costs you the optionality to react when the world changes.

## The Two Habits

First habit. Before any trade, look at LAS. If LAS is non-null and below a threshold — I use 1.0 as a default, scaled to the dollar size of the trade — the trade does not happen. I do not write the order. I do not negotiate with myself. The screen says LAS = 0.04 and the trade is dead. There are 47,000 markets and I do not need this one.

Second habit. When LAS is null, ask whether you are running the virgin polymarket strategy or whether you are running anything else. If anything else, the trade does not happen for the same reason. If virgin polymarket, the null is a green light and you post your maker quote. The two cases are mutually exclusive and the framing has to be conscious.

The whole indicator stack collapses to LAS in the limit. Every other number is a function of the displayed mid, and the displayed mid only exists if LAS is healthy. Run `sf scan --by-iy desc` if you want to see the universe of paper edges. Run `sf scan --by-iy desc --las-min 1.0` if you want to see the universe of trades. The second list is two orders of magnitude shorter, and that is the only list that pays.