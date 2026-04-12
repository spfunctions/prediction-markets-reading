# The Day I Stopped Trusting Raw Probabilities

> A specific morning, two Kalshi Fed-decision contracts at almost the same mid, and the realization that the cents on the screen had been hiding the trade from me for months.

**Category:** insights | **Author:** Patrick Liu | **Published:** 2026-04-09

---
Last fall I had two Kalshi tickers pinned to a sticky note on the side of my monitor. KXFEDDECISION-25NOV01-Y at 0.42, and KXFEDDECISION-26MAY01-Y at 0.44. Both were Fed-cut contracts. Both YES. Both at almost exactly the same mid. I had been staring at the sticky note for about a week, telling myself I would size into one of them once I worked out which was the better trade.

I never sized into either one. I would sit down, look at the two prices, decide they were "basically the same," shrug, and go work on something else. The decision sat in the same place for six trading days. By the time I finally moved, the November contract had already crossed 0.55 and the spread between them had blown out to a place where I could no longer pretend they were the same animal.

This is the story of the morning that broke that habit.

## What I Was Actually Looking At

On Tuesday October 28 I came back from making coffee, sat down, and looked at the sticky note again. Two contracts. 0.42 and 0.44. I remember this date because I had a tab open with a draft email to a friend on a bond desk who had asked me to walk him through "what these prediction-market things even are." I was procrastinating on the email by re-staring at my own portfolio.

That morning I finally did the dumb thing I should have done weeks earlier. I opened a terminal and typed two formulas in by hand. Not in a script. Not in the dashboard. With a pencil, on the back of a printed-out cron schedule that was lying on the desk.

The November contract had τ ≈ 18 days to resolution. The May contract had τ ≈ 200 days. Both at roughly 0.43 mid.

I wrote out:

```
IY_nov = (1 / 0.42) ^ (365 / 18) - 1
IY_may = (1 / 0.44) ^ (365 / 200) - 1
```

And then I evaluated them.

The November contract came out to something on the order of 8.3 million percent annualized. The May contract came out to roughly 80%. Both numbers are sketched, both round in the third digit, neither survives an executable-edge check against the orderbook. But the *ratio* between them was the part that finally cracked my head open. The two contracts I had been treating as interchangeable were not separated by a 5% difference in expected return. They were separated by five orders of magnitude in implied annualized yield.

I sat there for maybe ninety seconds with my coffee getting cold and just kept staring at the two numbers I had written down.

## The Thing That Was Hiding

Raw probability had been hiding the time axis from me for the entire week. The cents-on-the-screen view treated the τ-days field as decoration. Every time I looked at the two prices side by side, my eye landed on "42 vs 44" and my brain returned "approximately equal." It never went on to ask the next question, which is *equal in what unit*.

Once I expressed both contracts in [implied yield](/learn/implied-yield) — in the unit a fixed-income trader would use to compare two zero-coupon notes with different maturities — the two contracts stopped being approximately equal and started being utterly different instruments. One was a short-dated, high-leverage, all-or-nothing bet that paid like a lottery if I was right and like a brick if I was wrong. The other was something I could park alongside a treasury ladder and compute a portfolio yield against.

These are not the same trade. They never were. Raw probability had been telling me they were because I had been looking at the wrong number.

## What Changed That Afternoon

The first thing I did, after I finished the second cup of coffee, was open the screener and add a column. Not "yield." A specific named column called `iy_ask_45d` which I defined as the implied yield computed against the ask side of the orderbook with a τ floor of 6 hours. I made it the default sort.

The second thing I did was rewrite the email to the bond-desk friend from scratch. The first draft had been about "how a binary contract is priced in cents and how the price reflects implied probability." That whole framing went in the trash. The new draft started with "imagine a credit-risky zero-coupon note with a 60-day duration paying 4,710% annualized to maturity, and the credit event is whether the FOMC cuts rates by 25 bps at the next meeting." His reply landed three hours later and said: "ok, now I understand what you have been talking about."

I have a [blog post about that exact vocabulary upgrade](/blog/prediction-markets-need-fixed-income-language) and I wrote it directly out of this morning. It is the only piece of writing I have ever published that came from a single 90-second moment of staring at a sticky note. Most insights are slow. This one was abrupt.

The third thing I did was put a Python helper into my evaluation harness that refused to display a Kalshi contract without also displaying its IY at three different sides of the book (bid, mid, ask). The function lives at `sf scan --by-iy desc` now. I did not build it that day — I built it the following weekend — but the *decision* to build it was made on the morning of October 28 with a pencil and an old cron printout.

## The Habit That Replaced the Old One

I have not framed a binary contract in raw probability terms since.

That is not a stylistic preference. It is a load-bearing change. Raw probability is fine for a yes/no question that has no time axis — "will it rain in Manhattan tomorrow," sure, the time axis is fixed at one day, the unit doesn't matter. But the moment a contract has a meaningful holding period, raw probability becomes a *lossy compression* of the trade. It throws away the information about how long you have to wait for the answer, and that information turns out to be the most important thing about the position.

I now do the IY conversion before I do *anything* else with a contract. Before I check the [orderbook](/learn/orderbook). Before I check the [thesis](/learn/thesis). Before I check the [cliff risk index](/learn/cliff-risk-index). The IY number is the first thing my eye lands on, and it has reordered the way I rank contracts in a way that the old probability-sort never could.

The deeper move underneath all of this is that I started treating prediction markets as a *fixed-income surface* rather than a betting platform. Every contract is a point on a curve. Every curve is a term structure. Every term structure has a shape. The shape is the trade. And the only way to see the shape is to plot the right axis on the right unit, which is yield against tau, not cents against count.

I have a [longer essay on the indicator stack](/concepts/pm-indicator-stack) that came out of the same project. The IY was the first piece. Everything else followed.

## What I Still Wonder About

I do not know how many other days like October 28 are still ahead of me — moments where some unit I am taking for granted turns out to be hiding the trade. The sticky note is gone. The cron printout I wrote on it is gone. But the question it left me with is still very much open: what other axis am I currently flattening?

I have a list of suspects. I will get to them.