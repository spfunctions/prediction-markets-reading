# How We Bet on Peru's Presidential Election with an AI Agent

> 35 candidates. 40% undecided voters. 48 hours to go. We used prediction market tools to find an 11-cent mispricing window, designed a two-phase Taker+Maker strategy, and deployed $1,000 — all with an AI agent doing the legwork.

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 14 min | **Published:** 2026-04-10

---
Peru's presidential election on April 12, 2026 has 35 candidates competing for two runoff spots. Forty percent of voters are undecided. The prediction market contracts are live on Kalshi.

We found a systematic mispricing, designed a strategy, and executed real trades — $1,000 deployed in under two hours. Here's exactly how.

## Step 1: Find the Market

There are 47,498 active contracts on Kalshi. You can't read them all. You need a filter.

We used `screen_markets` to scan for election contracts with meaningful volume. The KXPERUPRES-26 series popped up — a set of contracts on who wins Peru's first-round presidential election, one contract per candidate.

This is the starting point for any prediction market trade: you don't begin with an opinion. You begin with a screen.

## Step 2: Check the Math

The first thing you do with a multi-candidate market is add up all the YES prices.

In a well-functioning market, the sum should equal 100 cents. There is exactly one winner, so all probabilities must sum to 100%. Any deviation from 100 is either the market's vig (overround) or a structural mispricing.

Peru's market summed to **107 cents**.

That 7-cent overround tells you two things:

1. **Someone is overpaying.** The market as a whole is pricing in more probability than exists. Some candidates are overpriced.
2. **The overround isn't distributed evenly.** If one candidate absorbs most of the excess, the others might be underpriced relative to their true odds.

This is the edge. Not a gut feeling about who wins. A mathematical fact about how prices are distributed.

## Step 3: Find the Mispricing

We pulled the orderbook for every candidate with meaningful volume and compared market prices against polling data. Here's what the numbers looked like:

| Candidate | Market Price | Polling | Our Estimate | Expected Value |
|-----------|-------------|---------|-------------|---------------|
| Keiko Fujimori | 30¢ | ~18% | 25% | **Negative** |
| Ricardo Belmont | 29¢ | ~5% | 5% | **Negative** |
| Carlos Álvarez | 17¢ | ~17% | 18% | Neutral |
| **López Aliaga** | **16¢** | ~17% | **22%** | **+37%** |
| **Roberto Sánchez** | **7¢** | ~7% | **12%** | **+71%** |
| Jorge Nieto | 4¢ | ~3% | 3% | Neutral |
| Alfonso López Chau | 2¢ | <2% | 1% | Neutral |

Two candidates jumped out.

**Keiko Fujimori was overpriced.** The market gave her 30 cents — a 30% implied probability. But polling had her around 18%, and she carries a well-documented electoral ceiling. She's made two runoffs and lost both. Her unfavorability rating is the highest in the field. The market was pricing name recognition, not win probability.

**López Aliaga and Sánchez were underpriced.** López Aliaga at 16 cents implied a 16% win probability, but polling and momentum indicators pointed to 22%. That's a +37% expected value. Sánchez at 7 cents with an estimated 12% true probability offered +71% expected value.

Where did the mispricing come from? Keiko absorbed the overround. International bettors who've heard of exactly one Peruvian politician — Fujimori — bid her up. That excess flowed out of candidates they'd never heard of.

This is a pattern in multi-candidate elections: **familiarity premium, obscurity discount.** The well-known candidate trades above fair value. The less-known candidates trade below.

## Step 4: Design the Strategy

We didn't just buy the underpriced candidates and wait. We designed a two-phase strategy that captures edge from two different sources.

### Phase 1: Taker (Pre-Election)

The Taker phase is a directional bet. You believe the market is wrong, and you buy at the current price.

| Contract | Direction | Contracts | Price | Cost |
|----------|-----------|-----------|-------|------|
| KXPERUPRES-26-RALI (López Aliaga) | YES | 1,562 | 16¢ | $249.92 |
| KXPERUPRES-26-RPAL (Sánchez) | YES | 2,142 | 7¢ | $149.94 |

Total deployed: $400. Maximum loss: $400. Maximum gain if López Aliaga wins: $1,312 profit. Maximum gain if Sánchez wins: $1,993 profit.

### Phase 2: Maker (Post-Election)

This is the part most people miss.

After the first-round results come in, the losing candidates' contracts need to go to zero. But prediction markets don't update instantly. There's a 2-12 hour window where eliminated candidates are still trading at 2-5 cents instead of zero.

During this window, you can **sell YES (or buy NO) on eliminated candidates** at near-zero risk. The candidate has already lost. The contract will settle at zero. But someone on the other side is either asleep, not paying attention, or running a bot that hasn't updated.

We reserved $500 for this phase. The target: sweep NO contracts on eliminated candidates, starting with Belmont (who was trading at 29 cents YES with only ~5% true probability).

The Maker phase doesn't require an opinion about who wins. It requires being awake when the results come in and having capital ready.

### Why Two Phases?

| | Taker | Maker |
|---|---|---|
| **Edge source** | Information advantage (better pricing model) | Speed advantage (faster than market update) |
| **Risk** | Candidates lose, you lose $400 | Near zero (candidate already eliminated) |
| **Timing** | Pre-election | Post-result, 2-12 hour window |
| **Skill** | Analysis | Execution |

The combination is more robust than either alone. If our directional Taker bets lose, the Maker phase partially recovers the loss. If the Taker bets win, the Maker phase adds incremental profit.

## Step 5: Stress-Test with Adversarial Review

This is where the AI agent earns its keep — not by making predictions, but by finding holes in your reasoning.

Before executing, we ran three adversarial checks:

**Discipline check:** Are we falling for sunk cost bias? Do we have concentrated risk? The agent flagged that our existing WTI and recession positions had exactly these problems. But the Peru position — fresh, uncorrelated, sized at $1,000 — passed clean.

**Pre-check:** What's the strongest argument against this trade? The agent identified: (1) polling in Peru is unreliable, with 40% undecided voters making any estimate fragile; (2) Keiko has institutional support that polls don't capture; (3) the 7-cent overround might be fair compensation for multi-candidate uncertainty, not a mispricing. We accepted these risks and sized accordingly.

**Situation report:** What external factors could blow up the thesis? The agent checked for last-minute candidate withdrawals, endorsements, or scandals. None found. The trade was clean.

The value of the agent isn't "it told us who would win." The agent has no idea who will win. The value is: **it forced us to articulate why we might be wrong before we put money down.** That's discipline, not prediction.

## Step 6: Execute and Automate

We placed the Taker orders at 18:26 UTC on April 10, 2026 — 48 hours before polls opened.

- López Aliaga YES: 1,562 contracts at 16¢ → $249.92, **filled immediately**
- Sánchez YES: 2,142 contracts at 7¢ → $149.94, **resting** (limit order)

For the Maker phase, we scheduled a wake-up call. At 12:33 UTC on April 12 — roughly when first-round results would be available — the agent would automatically:

1. Check which candidates were eliminated
2. Scan their orderbooks for residual YES liquidity
3. Buy NO contracts on eliminated candidates
4. Execute up to $500 in Maker trades

No human intervention needed. The agent handles the 3 AM execution window that humans sleep through.

## The Expected Value Math

| Scenario | Probability | Taker P&L | Maker P&L | Net |
|----------|------------|-----------|-----------|-----|
| López Aliaga wins | 22% | +$1,312 | +$65 | **+$1,377** |
| Sánchez wins | 12% | +$1,993 | +$65 | **+$2,058** |
| Both lose, Maker works | 59% | -$400 | +$65 to +$200 | **-$200 to $0** |
| Both lose, Maker fails | 7% | -$400 | $0 | **-$400** |
| **Weighted expected value** | | | | **+$390 (+39%)** |

The strategy has positive expected value even though both candidates are more likely to lose than win. That's because the payoff when they win is large enough to compensate for the frequency of loss.

This is the core insight of prediction market trading: **you don't need to be right most of the time. You need to be right more often than the price implies.**

## What This Illustrates

### 1. Multi-candidate elections are structurally inefficient

In a two-candidate race, the market has one number to get right. In a 35-candidate race, the market has to correctly price all 35 — and the sum has to equal 100. This is a much harder coordination problem. Mispricings are not bugs; they're structural features of complex markets.

### 2. Taker and Maker are complementary, not competing

Most people think of trading as one thing: buy low, sell high. But Taker (directional) and Maker (spread/latency) are fundamentally different profit sources. Combining them in a single trade plan gives you multiple ways to win.

### 3. The agent's value is discipline, not prediction

We called roughly 60 tools over two hours: screening markets, pulling orderbooks, running Kelly criterion calculations, executing adversarial reviews, placing orders, scheduling automated follow-up. No human could context-switch across these tasks as quickly. But the agent didn't predict anything. It structured the analysis so that the human could make a better decision.

### 4. The settlement delay is a tradeable pattern

After any binary event resolves — an election, a policy decision, a deadline — prediction market prices take time to update. This delay is predictable and repeatable. The Maker phase exploits it. Every election, every resolution, every expiry creates a brief window where you can sell certainty at a discount.

---

## Tools Used

The entire analysis-to-execution pipeline ran through SimpleFunctions:

- **`screen_markets`** — scanned 47,498 contracts, filtered to the KXPERUPRES-26 series
- **`inspect_book`** — pulled real-time orderbooks for 7 candidates
- **`calculate`** — Kelly criterion sizing, expected value computation, overround decomposition
- **Adversarial skills** — `/discipline`, `/precheck`, `/sitrep` for stress-testing
- **`place_order`** — executed Taker trades on Kalshi
- **`schedule_wake`** — programmed automated Maker execution for post-result window

60 tool calls. 2 hours from first screen to last fill. $1,000 deployed with +39% expected value.

---

*This is a live case study. Results will be updated after Peru's first-round vote on April 12, 2026. For more on prediction market strategy, see [Prediction Markets Are the Best Real-Time Sensor for World Events](/blog/prediction-markets-are-the-best-real-time-sensor-for-world-events) and [The Most Important Number Is the Delta](/blog/the-most-important-number-is-the-delta).*
