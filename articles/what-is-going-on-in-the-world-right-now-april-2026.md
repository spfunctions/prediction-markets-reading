# What Is Going On in the World Right Now — April 2026

> Iran at 67%, Ukraine ceasefire at 30%, Taiwan at 10%, Democrats eyeing Senate flip. What 9,706 prediction market contracts are telling us about the state of the world.

**Category:** geopolitics | **Author:** SimpleFunctions Research | **Reading time:** 8 min | **Published:** 2026-04-02

---
The news tells you what happened yesterday. Prediction markets tell you what's happening right now — and what's about to happen next. As of April 2, 2026, here is what 9,706 prediction market contracts across Kalshi and Polymarket are telling us about the state of the world.

The SimpleFunctions Geopolitical Risk Index stands at 94/100 — the highest we've tracked. Uncertainty is low at 22/100 (markets have strong opinions), but the direction of those opinions is alarming.

## The Iran War: 67% Chance US Forces Enter by April 30

This is the dominant story in prediction markets right now.

**US forces enter Iran by April 30: 67¢** on Polymarket. By year-end: 76¢. The market is pricing a near-certain US military engagement with Iran before 2027.

On the diplomatic side, ceasefire contracts tell a grimmer story:
- **US-Iran ceasefire by April 7: 2¢** — markets see virtually no chance of a near-term resolution
- **Ceasefire by April 30: 25¢**
- **Ceasefire by June 30: 58¢**

The spread between "forces enter" (67¢) and "ceasefire by April 30" (25¢) implies markets expect a military phase followed by negotiation — the pattern of every US engagement in the Middle East since 2003.

The Strait of Hormuz contracts are flashing: SimpleFunctions edge detection flags an 85¢ model edge on warship transit contracts. Oil (USO) is down 2.7% today, but gas price contracts are pricing $6+ by year-end at 95% confidence. The energy market hasn't fully absorbed the Iran risk premium yet.

**What this means for your portfolio:** The equity-oil divergence (SPY +0.81% vs Oil -2.7%) is unusual. Either stocks are ignoring Iran risk, or oil is over-discounting ceasefire probability. One of them is wrong.

## Russia-Ukraine: Ceasefire by 2026 at Just 30%

The war grinds on. Markets are not optimistic about a resolution this year.

- **Russia-Ukraine ceasefire by end of 2026: 30¢** on Polymarket
- **Ceasefire by June 2026: 9¢** — essentially zero near-term hope
- **Ceasefire by end of 2027: 54¢** — coin flip even with 20 more months

One striking contract: **Russia-Ukraine ceasefire before US-Iran ceasefire: 12¢**. Markets believe the Iran conflict will resolve before the Ukraine war — a reversal from six months ago when Ukraine was the focus.

On the ground, Polymarket prices Russia capturing Kostyantynivka by June 30 at 50¢ (down 6¢ recently) — suggesting a slowing Russian offensive but not a Ukrainian breakthrough.

**Putin out by end of 2026: 11¢.** Regime change is not what markets are betting on.

## Taiwan: Low Probability, But Watch the Trend

- **China invades Taiwan by end of 2026: 10¢** on Polymarket
- **Military clash before 2027: 13¢**
- **China blockade by June 30: 5¢**

Taiwan risk is priced low, but the numbers are higher than they were a year ago. The 10¢ invasion probability might seem small, but for a tail-risk event with catastrophic consequences for semiconductor supply chains and global trade, it represents significant concern.

The longer-dated contracts tell the real story: **invasion by end of 2027 at 21¢** — roughly 1 in 5. That's not a black swan. That's a known risk with a timeline.

## US Midterm Elections: Democrats Eyeing a Comeback

The 2026 midterms are shaping up to be the most consequential since 2018.

Republican Senate seats after the midterms:
- **≤47 seats (Democrats take majority): 27¢** — the single most likely outcome bucket
- **50 seats (tie): 12¢**
- **51 seats (bare majority): 14¢**

Markets are pricing roughly a **40% chance Democrats flip the Senate**. Combined with the House (where the president's party historically loses seats), this would be a significant check on the Trump administration's policy agenda.

The tariff contracts add context: **China tariff rate 10-20% by July 1: 70¢**. Markets expect tariffs to moderate from current levels, possibly because traders are pricing in Congressional pushback if Democrats gain seats.

## Energy: The Squeeze Nobody's Talking About

Oil is down 2.7% today, but the forward contracts tell a different story:

- **Gas prices above $6.00 by year-end: 95¢** — near certainty
- **Above $6.80: only 10¢** — markets see a ceiling
- **Russia crude exports below 4.0 mbpd in April: 74¢**

The Iran war is the key variable. If Hormuz shipping disrupts (our edge detection flags this at 85¢ model-implied probability), the gas price ceiling breaks. The current 95¢ / 10¢ split between $6.00 and $6.80 represents a narrow band of consensus that assumes the conflict stays contained. If it doesn't, the $6.80 contracts are significantly underpriced.

Meanwhile, gold (GLD) is up 1.82% today alongside stocks — an unusual risk-on-plus-haven-bid pattern that historically precedes volatility spikes.

## AI: The Quiet Revolution Continues

While geopolitics dominates the market, AI contracts remain active:

- **AI regulation by 2027: 21¢** on Kalshi — the market sees it as unlikely but possible
- **OpenAI announces AGI creation: 18¢ (near-term), 35¢ (mid-term), 56¢ (by 2027)**

The AI regulation contract at 21¢ is interesting in the context of the midterms. If Democrats gain Congressional seats, the probability of regulation legislation increases significantly — but the current contract doesn't fully reflect that conditional dependency. This is the kind of cross-market edge that automated causal models detect but human traders miss.

## The Macro Picture

The Fed is holding steady. **No change in April: 98¢**. But inflation concerns are rising: **inflation above 6% in 2026: 22¢** (up 8¢ recently). The tariff regime, energy prices, and geopolitical risk premium are all inflationary.

**IMF declares global recession before 2027: 29¢** — just under 1 in 3. That's high. Historically, when recession probability crosses 30% in prediction markets, the actual recession follows within 12 months about 40% of the time.

## What the Divergences Tell Us

The SimpleFunctions divergence detection engine is flagging three anomalies right now:

1. **Energy sector split** — some energy contracts rising while others falling. This usually resolves with a sharp move in one direction.
2. **Stocks and gold both up** (SPY +0.81%, GLD +1.82%) — classic pre-volatility signal. The market is simultaneously risk-on and hedging.
3. **Equity-oil divergence** (SPY +0.81% vs Oil -2.7%) — one of these is mispriced. Historical precedent: when SPY and oil diverge by more than 3% in a single day during geopolitical escalation, oil catches up within 5 trading days 68% of the time.

## How to Track This

These aren't opinions or forecasts. They're real-money prices from traders with skin in the game. You can track all of this live:

```bash
# Install the CLI
npm i -g @spfunctions/cli

# Get the world state — updated every 15 minutes
sf query "what's happening in the world"

# Watch specific topics in real-time
sf watch "iran" cross-venue
sf watch "ukraine" orderbook

# See what's mispriced
sf edges

# Or just ask
sf agent
```

Or use the API directly:

```bash
# World state in 800 tokens
curl https://simplefunctions.dev/api/agent/world

# Search any topic
curl "https://simplefunctions.dev/api/public/scan?q=iran"

# Cross-market contagion detection
curl "https://simplefunctions.dev/api/public/contagion?window=24h"
```

The world is moving fast. Prediction markets are the fastest way to track it.

---

*All data sourced from SimpleFunctions World Model — 9,706 markets across Kalshi and Polymarket. Prices as of April 2, 2026. Updated every 15 minutes at [simplefunctions.dev](https://simplefunctions.dev).*