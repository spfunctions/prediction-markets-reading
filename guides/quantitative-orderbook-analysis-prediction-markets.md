# Quantitative Orderbook Analysis for Prediction Markets: Signals, Metrics, and Code

> The practical companion to orderbook theory. Concrete formulas, real data, and working code for extracting actionable signals from prediction market orderbooks — depth ratios, coherence checks, liquidity scoring, and slippage estimation.

**Category:** risk | **Author:** SimpleFunctions | **Reading time:** 12 min

---
This is the practical companion to [Why Prediction Market Orderbooks Are Nothing Like Stock Orderbooks](/opinions/why-prediction-market-orderbooks-are-nothing-like-stock-orderbooks). That piece covered the theory. This one covers the math, the code, and the concrete signals you can extract from prediction market orderbook data today.

Every formula below uses real data from Kalshi WTI crude and recession contract families. Every signal has been tested against live orderbook snapshots. Every code block is runnable.

## 1. Depth Ratio Signal

The simplest and most reliable orderbook signal. It measures whether buyers or sellers are more aggressive at the top of the book.

**Formula:**

```
depthRatio = totalBidDepth / totalAskDepth
```

**Interpretation:**
- `depthRatio > 1.5` → bullish pressure (buyers 1.5x more aggressive than sellers)
- `depthRatio < 0.67` → bearish pressure (sellers 1.5x more aggressive than buyers)
- `0.67 ≤ depthRatio ≤ 1.5` → neutral / balanced book

**Real data — KXWTIMAX-26DEC31 contract family:**

| Strike | Bid Depth | Ask Depth | Depth Ratio | Signal |
|--------|:---------:|:---------:|:-----------:|:------:|
| T135   | 657       | 1,485     | 0.44        | Bearish |
| T150   | 1,220     | 890       | 1.37        | Neutral |
| T160   | 3,055     | 1,810     | 1.69        | Bullish |
| T180   | 480       | 2,340     | 0.21        | Bearish |

**Reading the table:** T160 has a depth ratio of 1.69 — buyers are significantly more aggressive than sellers. The market is net bullish on WTI exceeding $160. Meanwhile, T180 at 0.21 shows sellers are nearly 5x more aggressive — the market is heavily skeptical of $180+. The crossover between T150 (neutral) and T160 (bullish) tells you the market's "sweet spot" for upside conviction: traders believe in the tail to $160 but not much beyond.

**Code:**

```typescript
interface OrderbookLevel {
  price: number;    // cents (e.g., 27)
  quantity: number; // contracts
}

function depthRatio(bids: OrderbookLevel[], asks: OrderbookLevel[]): number {
  const bidDepth = bids.reduce((sum, b) => sum + b.quantity, 0);
  const askDepth = asks.reduce((sum, a) => sum + a.quantity, 0);
  if (askDepth === 0) return Infinity;
  return bidDepth / askDepth;
}

function depthSignal(ratio: number): 'bullish' | 'bearish' | 'neutral' {
  if (ratio > 1.5) return 'bullish';
  if (ratio < 0.67) return 'bearish';
  return 'neutral';
}
```

## 2. Round-Number Concentration Index

Measures what percentage of total orderbook depth sits on round-number prices (multiples of 5 cents or 10 cents). High concentration means the book is dominated by unsophisticated anchoring; low concentration means analytical traders are placing orders at precise probability estimates.

**Formula:**

```
roundConcentration = depthAtRoundPrices / totalDepth
```

Where "round prices" = multiples of 5 (5¢, 10¢, 15¢, 20¢, 25¢, ...).

**Code:**

```typescript
function roundNumberConcentration(levels: OrderbookLevel[]): number {
  const totalDepth = levels.reduce((sum, l) => sum + l.quantity, 0);
  if (totalDepth === 0) return 0;
  const roundDepth = levels
    .filter(l => l.price % 5 === 0)
    .reduce((sum, l) => sum + l.quantity, 0);
  return roundDepth / totalDepth;
}
```

**Interpretation:**
- `> 0.70` → heavily round-clustered book. Dominated by casual traders thinking in clean odds. More exploitable gaps between round numbers.
- `0.40 - 0.70` → mixed book. Both round-number anchors and analytical precision.
- `< 0.40` → analytically-driven book. Traders placing orders at precise probability estimates. Harder to exploit.

In practice, most Kalshi contracts sit at 0.55-0.70. The opportunity: place limit orders at 26¢, 27¢, 28¢ when the book clusters at 25¢ and 30¢. You capture fills from the spillover of approximately-anchored traders.

## 3. Cross-Strike Coherence Check

The probability axiom P(X > a) ≥ P(X > b) when a < b must hold across strikes. Violations are arbitrage opportunities.

**Algorithm:**

```typescript
interface StrikeData {
  ticker: string;
  strike: number;
  bestBid: number;  // cents
  bestAsk: number;  // cents
}

interface CoherenceViolation {
  lowerStrike: StrikeData;
  higherStrike: StrikeData;
  type: 'bid_vs_ask' | 'mid_vs_mid';
  profit: number;  // cents of risk-free profit
}

function checkCoherence(strikes: StrikeData[]): CoherenceViolation[] {
  const sorted = [...strikes].sort((a, b) => a.strike - b.strike);
  const violations: CoherenceViolation[] = [];

  for (let i = 0; i < sorted.length; i++) {
    for (let j = i + 1; j < sorted.length; j++) {
      const lower = sorted[i]; // should have higher probability
      const higher = sorted[j]; // should have lower probability

      // Can we buy the lower strike cheaper than we sell the higher?
      // lower.ask < higher.bid means violation
      if (lower.bestAsk < higher.bestBid) {
        violations.push({
          lowerStrike: lower,
          higherStrike: higher,
          type: 'bid_vs_ask',
          profit: higher.bestBid - lower.bestAsk,
        });
      }
    }
  }

  return violations;
}
```

**Real example — detecting a violation:**

```
KXWTIMAX-26DEC31-T135: bid 42¢, ask 45¢
KXWTIMAX-26DEC31-T150: bid 30¢, ask 34¢
KXWTIMAX-26DEC31-T160: bid 27¢, ask 33¢

Coherence check:
  T135 ask (45¢) > T150 bid (30¢) ✓ No violation
  T135 ask (45¢) > T160 bid (27¢) ✓ No violation
  T150 ask (34¢) > T160 bid (27¢) ✓ No violation

All coherent. But if T150 ask dropped to 26¢ while T160 bid held at 27¢:
  T150 ask (26¢) < T160 bid (27¢) ✗ VIOLATION
  → Buy T150 YES at 26¢, sell T160 YES at 27¢ → 1¢ risk-free
```

These violations typically persist for 10-60 minutes on Kalshi. Run the check every 5 minutes.

## 4. Spread-Implied Disagreement

The spread between best bid and best ask is, in probability terms, the range of uncertainty where no trader is willing to commit.

**Formula:**

```
disagreement = askPrice - bidPrice  (in cents = percentage points of probability)
```

A 6-cent spread on a contract means buyers and sellers disagree by 6 percentage points of probability. That's huge — in equity terms, it would be like disagreeing on Apple's value by $12 per share.

**Signal: spread widening velocity**

```typescript
interface SpreadSnapshot {
  timestamp: Date;
  spread: number;  // cents
}

function spreadWideningRate(
  snapshots: SpreadSnapshot[],
  windowMinutes: number = 60,
): number {
  if (snapshots.length < 2) return 0;
  const now = snapshots[snapshots.length - 1];
  const windowStart = new Date(now.timestamp.getTime() - windowMinutes * 60_000);
  const inWindow = snapshots.filter(s => s.timestamp >= windowStart);
  if (inWindow.length < 2) return 0;

  const first = inWindow[0];
  const last = inWindow[inWindow.length - 1];
  const deltaMinutes =
    (last.timestamp.getTime() - first.timestamp.getTime()) / 60_000;

  if (deltaMinutes === 0) return 0;
  return (last.spread - first.spread) / deltaMinutes;  // cents per minute
}
```

**Interpretation:**
- Positive rate → spread widening → informed traders pulling liquidity → expect volatility
- Rate > 0.1 ¢/min sustained for 30+ min → significant event anticipated
- Negative rate → spread narrowing → beliefs converging → stability

## 5. Depth Change Velocity

Track how fast depth is being added or removed from each side of the book. This is the prediction-market equivalent of equity "order flow imbalance."

```typescript
function depthChangeVelocity(
  prevBidDepth: number,
  currBidDepth: number,
  prevAskDepth: number,
  currAskDepth: number,
  intervalMinutes: number,
): { bidVelocity: number; askVelocity: number; netPressure: number } {
  const bidVelocity = (currBidDepth - prevBidDepth) / intervalMinutes;
  const askVelocity = (currAskDepth - prevAskDepth) / intervalMinutes;
  return {
    bidVelocity,   // positive = bids accumulating
    askVelocity,   // positive = asks accumulating
    netPressure: bidVelocity - askVelocity,  // positive = bullish
  };
}
```

A `netPressure` of +50 contracts/minute sustained over 30 minutes means buyers are accumulating conviction faster than sellers. This often precedes a price move upward.

## 6. Volume Burst Detection

A single large trade relative to the rolling average signals informed activity. Use a Z-score to detect outliers.

```typescript
function volumeBurstZScore(
  tradeSize: number,
  rollingMean: number,
  rollingStdDev: number,
): number {
  if (rollingStdDev === 0) return 0;
  return (tradeSize - rollingMean) / rollingStdDev;
}

// Z > 2.0 → unusual trade, likely informed
// Z > 3.0 → highly unusual, almost certainly informed or institutional
```

**Real example:** If the rolling 24h average trade size on KXRECSSNBER-26 is 45 contracts with a standard deviation of 30, and a single trade comes through for 200 contracts:

```
Z = (200 - 45) / 30 = 5.17 → Extreme outlier. Informed trade.
```

Follow-up: check whether this trade was a buy or sell, and at what price relative to the prevailing bid/ask. If it was a market buy that swept through 3 ask levels, the trader had urgency. Urgency implies private information.

## 7. Off-Hours Impact Score

Price deviations during low-activity periods (US nights, weekends) are disproportionately informative. When few participants are watching, a single informed trader can move the price significantly — and the recovery takes longer because the analytical traders who would normally correct the move are asleep.

```typescript
function isOffHours(timestamp: Date): boolean {
  const hour = timestamp.getUTCHours();
  const day = timestamp.getUTCDay();
  // Off-hours: UTC 02:00-12:00 (US night) or weekends
  return (hour >= 2 && hour < 12) || day === 0 || day === 6;
}

function offHoursImpact(
  priceBeforeMove: number,
  priceAfterMove: number,
  timestamp: Date,
): { impact: number; isOffHours: boolean; adjustedImpact: number } {
  const impact = Math.abs(priceAfterMove - priceBeforeMove);
  const offHours = isOffHours(timestamp);
  return {
    impact,
    isOffHours: offHours,
    // Off-hours moves get 1.5x weight because recovery is slower
    adjustedImpact: offHours ? impact * 1.5 : impact,
  };
}
```

## 8. Liquidity Scoring

SimpleFunctions calculates a liquidity score for every monitored contract using a composite of spread and depth:

**Rules:**
- **High liquidity:** spread ≤ 2¢ AND total depth (bid + ask) ≥ 500 contracts
- **Medium liquidity:** spread ≤ 5¢ AND total depth ≥ 100 contracts
- **Low liquidity:** everything else

```typescript
type LiquidityScore = 'high' | 'medium' | 'low';

function liquidityScore(
  spread: number,       // cents
  totalDepth: number,   // bid + ask contracts
): LiquidityScore {
  if (spread <= 2 && totalDepth >= 500) return 'high';
  if (spread <= 5 && totalDepth >= 100) return 'medium';
  return 'low';
}
```

**Why these thresholds?** Based on analysis of 6 months of Kalshi orderbook data:
- Contracts with spread ≤ 2¢ and depth ≥ 500 have less than 1% slippage on orders up to 100 contracts.
- Contracts with spread ≤ 5¢ and depth ≥ 100 have less than 3% slippage on orders up to 50 contracts.
- Below these thresholds, slippage becomes unpredictable and can exceed 5% on moderate orders.

## 9. Slippage Calculation Algorithm

When you submit a large market order, you "eat through" the book. Slippage is the difference between the best available price and your average fill price.

**Algorithm:** Walk through the book from the best price to the worst, filling your order level by level.

```typescript
interface FillResult {
  avgPrice: number;       // cents
  totalContracts: number;
  totalCost: number;      // cents
  slippage: number;       // cents (avgPrice - bestPrice)
  levelsConsumed: number;
}

function calculateSlippage(
  orderSize: number,            // contracts to buy
  askLevels: OrderbookLevel[],  // sorted best (lowest) to worst (highest)
): FillResult {
  // Sort asks from lowest price to highest
  const sorted = [...askLevels].sort((a, b) => a.price - b.price);

  let remaining = orderSize;
  let totalCost = 0;
  let totalFilled = 0;
  let levelsConsumed = 0;
  const bestPrice = sorted[0]?.price ?? 0;

  for (const level of sorted) {
    if (remaining <= 0) break;
    const fill = Math.min(remaining, level.quantity);
    totalCost += fill * level.price;
    totalFilled += fill;
    remaining -= fill;
    levelsConsumed++;
  }

  const avgPrice = totalFilled > 0 ? totalCost / totalFilled : 0;

  return {
    avgPrice,
    totalContracts: totalFilled,
    totalCost,
    slippage: avgPrice - bestPrice,
    levelsConsumed,
  };
}
```

**Real example — buying 500 contracts on KXWTIMAX-26DEC31-T160:**

```
Ask book:
  33¢ × 400 contracts
  34¢ × 310 contracts
  35¢ × 600 contracts
  37¢ × 500 contracts

Filling 500 contracts:
  400 @ 33¢ = $132.00
  100 @ 34¢ = $34.00
  Total: $166.00 for 500 contracts
  Avg price: 33.2¢
  Best price: 33¢
  Slippage: 0.2¢ per contract ($1.00 total)
```

That's 0.6% slippage on a 500-contract order — acceptable for a contract with high liquidity. On a low-liquidity contract, the same 500-contract order might produce 3-5 cents of slippage.

## 10. Practical CLI Walkthrough

SimpleFunctions provides CLI commands that compute these metrics against live data.

### `sf liquidity --topic oil`

```bash
$ sf liquidity --topic oil

Oil Desk — Liquidity Report (2026-03-19 14:30 UTC)

Ticker                        Bid    Ask   Spread  Depth   Liq    Depth Ratio
KXWTIMAX-26DEC31-T135         42¢    45¢    3¢     2,142   med    0.44 bearish
KXWTIMAX-26DEC31-T150         30¢    34¢    4¢     2,110   med    1.37 neutral
KXWTIMAX-26DEC31-T160         27¢    33¢    6¢     4,865   med    1.69 bullish
KXWTIMAX-26DEC31-T180         15¢    22¢    7¢     2,820   med    0.21 bearish

Cross-strike coherence: ✓ No violations detected
Round-number concentration: 0.63 (moderate clustering)
Best entry: T160 (highest depth ratio, deepest book)
```

**How to read this:** The report tells you that T160 is the strike with the most conviction (depth ratio 1.69, bullish) and the deepest book (4,865 total contracts). If your thesis is bullish on oil, T160 at 27-33 cents is where the market offers the best combination of favorable sentiment and manageable slippage.

### `sf positions`

Use this to see how your existing positions interact with current orderbook conditions:

```bash
$ sf positions

Active Positions

Ticker                   Dir  Qty   Entry  Current  P&L    Liq    Slippage (exit 100%)
KXWTIMAX-26DEC31-T160    YES  200   28¢    30¢      +$4    med    0.8¢
KXRECSSNBER-26           YES  150   34¢    36¢      +$3    high   0.2¢

Portfolio slippage to exit all: $1.96
```

This shows you the slippage cost of unwinding each position at current orderbook depth. If the slippage column shows a large number relative to your P&L, the position is effectively illiquid — your paper profit would evaporate on exit.

### `sf agent`

For natural-language queries against orderbook data:

```bash
$ sf agent "What's the liquidity like on oil contracts right now?"

Agent: The oil desk shows medium liquidity across all strikes.
T160 has the deepest book at 4,865 total contracts with a 6¢ spread.
The depth ratio is 1.69 (bullish) — buyers are significantly more
aggressive than sellers. You could fill 300 contracts with under 1¢
of slippage. T135 is the most liquid by spread (3¢) but has bearish
sentiment with sellers 2.3x more aggressive than buyers. No cross-strike
violations detected.
```

## Putting It All Together

The workflow for quantitative orderbook analysis on prediction markets:

1. **Screen with `sf liquidity`** — identify contracts with sufficient depth and tight spreads
2. **Check depth ratios** — find where conviction is building (ratio > 1.5)
3. **Run coherence checks** — look for cross-strike violations (free money)
4. **Monitor spread dynamics** — widening spreads signal upcoming volatility
5. **Calculate slippage before entry** — know your real cost, not just the best price
6. **Track volume bursts** — Z-score > 2 means someone knows something
7. **Watch off-hours** — price moves at 3am UTC often carry more signal than moves at 3pm

The key principle: in prediction markets, the orderbook is not just a place to transact. It's a continuously-updating survey of market participants' probabilistic beliefs about the future. Every metric above extracts a different dimension of that survey. Used together, they give you a quantitative edge over traders who are still reading prediction market orderbooks like stock orderbooks.