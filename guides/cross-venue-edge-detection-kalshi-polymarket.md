# Cross-Venue Edge Detection: Kalshi vs Polymarket

> The same event priced differently across venues. Why it happens, how to detect it programmatically, and why thesis-informed cross-venue trading beats pure arbitrage.

**Category:** patterns | **Author:** SimpleFunctions | **Reading time:** 7 min

---
On March 4, 2026, the Kalshi market for "US Recession in 2026" was trading at 34 cents YES. At the same time, the closest equivalent on Polymarket was at 39 cents. A 5-point spread on the same underlying event.

This happens more often than you'd think. And understanding *why* it happens — not just *that* it happens — is the difference between losing money on fake arbitrage and making money on real cross-venue edge.

## Why Prices Diverge Across Venues

The textbook answer is "arbitrage keeps prices aligned." The real answer is that prediction market arbitrage is structurally hard, so prices diverge regularly. Here's why:

### Different User Bases

Kalshi skews US-regulated, KYC-verified, relatively conservative. Polymarket skews crypto-native, global, more speculative. These populations have different information sets, different risk tolerances, and different behavioral biases.

When a geopolitical event breaks, the crypto-native Polymarket crowd reacts faster (they're online 24/7) but often overreacts. The Kalshi crowd reacts slower but with more measured moves. This creates a predictable divergence pattern: Polymarket spikes first, Kalshi catches up over hours.

### Fee Structure Differences

Kalshi charges a fee on settlement (currently 1 cent per contract on the winning side). Polymarket uses AMM pools with spread embedded in the pricing. These structural costs mean the "true" price on each venue includes different overhead.

A Kalshi YES at 34 cents and a Polymarket YES at 36 cents might actually represent the same expected probability once you account for fees — there's no real edge. You have to net out venue-specific costs before comparing.

### Liquidity Asymmetry

Kalshi markets are order-book based. Polymarket markets are AMM-based (or hybrid). An AMM will always show a price — but the slippage on a large order can be enormous. The posted price on Polymarket means nothing if trying to fill 500 contracts moves the price 3 points.

When comparing prices, you need to compare *executable prices at your intended size*, not midpoint prices.

### Settlement Timing and Resolution

The same "event" may settle differently on different venues. "US enters recession in 2026" on Kalshi settles based on the NBER dating committee. On Polymarket, the resolution criteria might reference two consecutive quarters of GDP decline. These are correlated but not identical — and the market knows it.

## Detecting Divergences Programmatically

The naive approach is to pull prices from both venues and alert when the spread exceeds a threshold. This generates a lot of false positives. Here's a better framework:

### Step 1: Event Matching

The hardest part. Kalshi ticker `KXRECESSION-26` and a Polymarket market titled "Will the US be in a recession by end of 2026?" are about the same event but may have different settlement criteria. You need a mapping layer.

SimpleFunctions maintains a cross-venue event map that pairs markets across Kalshi and Polymarket by underlying event. The matching considers:
- Event description similarity (semantic, not string matching)
- Settlement date alignment (within 30 days)
- Resolution criteria comparison (flagging any differences)

### Step 2: Normalized Price Comparison

Once events are matched, normalize prices:

```
adjusted_kalshi = kalshi_yes_price + settlement_fee
adjusted_poly   = polymarket_yes_price + estimated_slippage(your_size)
spread           = adjusted_poly - adjusted_kalshi
```

A spread of 2 points or less is probably noise. 3-5 points is interesting. 5+ points is either a real divergence or the markets have different settlement criteria that you missed.

### Step 3: Historical Spread Analysis

Divergences are meaningful relative to their history. If Kalshi-Polymarket spread on recession markets is *always* 3-4 points (because of structural fee differences), then a 4-point spread isn't a signal. But if the historical spread is 1-2 points and suddenly it's 6, something changed.

SimpleFunctions tracks cross-venue spread history and alerts on z-score deviations — spreads that are statistically unusual for that particular event pair.

## Why Pure Arbitrage Doesn't Work

On paper, you'd buy YES on the cheap venue and sell YES (buy NO) on the expensive venue. Lock in the spread. Free money.

In practice:

**Capital lockup.** Both positions tie up capital until settlement. A 5-point spread on a market that settles in 8 months means your annualized return is about 7.5%. Not terrible, but not the "free money" it looks like.

**Settlement risk.** If the venues resolve differently (one says YES, the other says NO — it happens with ambiguous events), you lose on both sides.

**Counterparty risk.** Polymarket operates offshore. Regulatory changes could freeze funds. Kalshi is US-regulated but could face its own issues. Having capital locked on two venues doubles your counterparty exposure.

**Execution risk.** By the time you detect the spread, execute on venue A, and execute on venue B, the spread may have closed. Especially on Polymarket where AMM prices update instantly on each trade.

## What Works Instead: Thesis-Informed Cross-Venue Trading

Pure arbitrage tries to be direction-neutral. Thesis-informed cross-venue trading accepts directionality and uses the spread to choose *where* to express a view.

Here's the framework:

**You have a thesis.** Your causal model says recession probability is 45%. Kalshi is at 34 cents, Polymarket is at 39 cents. You want to buy YES.

**Choose the venue with more edge.** Buy on Kalshi at 34 cents. Your edge is 11 points (45% - 34%). On Polymarket your edge would be 6 points (45% - 39%). Same thesis, same direction, but Kalshi gives you nearly 2x the edge on this particular trade.

**Monitor both venues for exit signals.** If Polymarket moves to 50 cents while Kalshi is at 42 cents, Polymarket is giving you information — its user base (faster-reacting, more speculative) already priced in something that Kalshi hasn't. This might be your signal to add on Kalshi before it catches up, or it might be a warning that the Polymarket crowd is overreacting.

**Use divergence as a confidence check.** If both venues move in the same direction by similar amounts, the signal is confirmed. If they diverge (Kalshi drops while Polymarket rises), something is wrong — either with the market or with your thesis. Divergence after a news event is a red flag that demands thesis re-evaluation.

## SimpleFunctions Cross-Venue Scanning

The `sf scan` command pulls from both venues simultaneously and shows you cross-venue spreads:

```bash
sf scan --cross-venue
```

Output includes:

```
Cross-venue spreads (>3 points):

  Recession 2026          Kalshi 34¢  |  Poly 39¢  |  Spread: 5pts
  Fed Rate Cut Jun        Kalshi 22¢  |  Poly 25¢  |  Spread: 3pts
  Oil >$90 Sep            Kalshi 41¢  |  Poly 38¢  |  Spread: -3pts (Kalshi higher)
```

When you're running a thesis, `sf edge` automatically shows you which venue offers better entry:

```
Thesis: iran-war
  Recession 2026: thesis 45% | Kalshi 34¢ (edge +11) | Poly 39¢ (edge +6)
  → Best entry: Kalshi
```

## Historical Divergence Patterns Worth Knowing

From tracking cross-venue spreads over the past year, a few patterns emerge consistently:

**News-driven spikes converge within 4-12 hours.** When breaking news causes a spike on one venue, the other catches up. Polymarket moves first ~70% of the time. If you see Polymarket spike and Kalshi hasn't moved, you have a window — but it closes fast.

**Weekend divergences are noise.** Kalshi volume drops on weekends. Polymarket (crypto-native, global) stays active. Spreads that appear on Saturday afternoon are usually gone by Monday morning. Don't trade on weekend spreads unless the underlying event is itself weekend-news-driven.

**Divergences widen before settlement.** In the final two weeks before a market settles, spreads between venues tend to widen, not narrow. This is counterintuitive but makes sense: as settlement approaches, each venue's specific resolution criteria matter more, and any differences in those criteria get priced in.

**Liquidity predicts direction of convergence.** When a spread exists, the higher-liquidity venue is usually closer to "right." The lower-liquidity venue adjusts toward the higher-liquidity one about 65% of the time. This isn't a strong enough signal to trade on alone, but it's a useful tiebreaker.

## The Bottom Line

Cross-venue price differences are real and frequent. But they're not free money — they're information. The divergence tells you something about how different populations are pricing the same event. Use that information to make better entry decisions, not to construct fragile arbitrage positions.

Your thesis is the edge. The venue comparison is the execution optimization.