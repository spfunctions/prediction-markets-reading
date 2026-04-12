# From Thesis to Execution: How SimpleFunctions Manages the Full Trading Loop

> The complete walkthrough. From "I think Iran will cause a recession" to "my agent detected CPI data at 3am and updated the causal tree." Every step is a product feature wrapped in a real trading decision.

**Category:** workflow | **Author:** Patrick Liu | **Reading time:** 1 min

---
This page walks through the entire SimpleFunctions trading loop using a real thesis. Not in theory — in practice. Every step corresponds to a command you run and a decision you make.

The thesis: **US-Iran war persists → Hormuz stays blocked → oil stays elevated → recession.**

Let's follow it from formation to live monitoring.

## Phase 1: Scan — Finding the Opportunity

```bash
sf scan --desk geopolitics
```

The scan pulls in 200+ geopolitical contracts from Kalshi and Polymarket. It groups them by event and runs a quick causal plausibility check. Out of 200+ contracts, it flags 8-12 with potential structural mispricing.

**What you see:**

```
Scanning geopolitics desk...

Found 11 candidates:

  US Recession 2026            Kalshi   YES @ 35¢
  Oil > $90 by Sep 2026        Poly     YES @ 41¢
  Fed Rate Cut Jun 2026        Kalshi   NO  @ 65¢
  Hormuz Shipping Disruption   Poly     YES @ 82¢
  Iran Ceasefire by Jul 2026   Kalshi   NO  @ 71¢
  ...
```

**The decision point:** You see the recession contract at 35¢ and you have a thesis about why it should be higher. The scan didn't tell you this — your analysis of the Iran situation did. The scan just showed you that the market has priced recession at 35%, which you think is too low.

## Phase 2: Thesis Formation — Structuring Your Thinking

```bash
sf thesis create --name "iran-war"
```

The agent enters interactive mode. It asks: "What's your directional view and why?"

You explain: "Iran war persists. Hormuz stays blocked because minesweeping takes months. Oil stays above $100 because SPR can't compensate and OPEC won't increase production. The Fed is trapped — raise rates into a supply shock or let inflation run. Either way, recession."

The agent decomposes this into a causal tree:

```
n1  War persists                          88%
├── n1.1  US initiated strikes            99%  (already happened)
├── n1.2  Iran continues retaliation      85%
└── n1.3  No diplomatic exit              82%
n2  Hormuz blocked                        97%
├── n2.1  Mines deployed                  99%  (confirmed)
├── n2.2  IRGC small craft active         90%
└── n2.3  Minesweeping > 3 months         87%
n3  Oil stays elevated (>$100)            64%
├── n3.1  SPR insufficient                73%
├── n3.2  OPEC doesn't compensate         68%
└── n3.3  No alternative supply           71%
n4  Fed trapped                           58%
n5  Recession                             45%
```

**Why this matters:** Each node is independently verifiable. Node n2.1 (mines deployed) is 99% because it's been confirmed by CENTCOM. Node n3.2 (OPEC doesn't compensate) is 68% because it depends on Saudi political decisions that could go either way. The tree makes your uncertainty *explicit* and *decomposed*.

## Phase 3: Edge Detection — Where Are You Smarter Than the Market?

```bash
sf edge --thesis iran-war
```

The edge detector maps your causal tree nodes to market contracts and calculates the divergence:

| Contract | Your Thesis | Market | Edge | Causal Path |
|----------|:----------:|:------:|:----:|:-----------:|
| Recession 2026 YES | 45% | 35% | +10 | n1→n2→n3→n4→n5 |
| Oil > $90 Q3 YES | 64% | 41% | +23 | n1→n2→n3 |
| Fed Cut Jun NO | 58% | 35% | +23 | n1→n2→n3→n4 |
| Hormuz Disruption YES | 97% | 82% | +15 | n1→n2 |

**The insight:** The biggest edge isn't on the recession contract — it's on the oil contract and the Fed contract. These are upstream in your causal chain, meaning they have fewer dependencies and higher confidence. The recession contract has the most dependencies (everything has to go right), so even though your thesis implies 45%, the uncertainty compounds.

## Phase 4: Strategy Creation — Defining When to Act

```bash
sf strategy create --thesis iran-war --contract "recession-2026-yes"
```

The agent helps you define mechanical conditions:

**Entry conditions:**
- Thesis confidence for n5 (recession) > 40%
- Market price < 38¢
- Edge > 7 points
- Causal tree upstream health: n1, n2, n3 all > 60%

**Exit conditions:**
- Thesis confidence for n5 < 30%
- Edge < 3 points
- OR any upstream node < 50%

**Position sizing:**
- Half Kelly based on current edge
- Max 20% of bankroll
- Scale down if any upstream node confidence drops below 70%

These conditions are stored. The agent will evaluate them on every heartbeat.

## Phase 5: Monitoring — The 24/7 Watch

```bash
sf monitor --strategy iran-recession
```

Now the system is live. Here's what happens on a typical cycle:

**Heartbeat at 02:15 UTC:**
1. Fetch latest prices: Recession YES @ 36¢, Oil > $90 YES @ 43¢
2. Search Tavily for news on causal nodes: "Iran Hormuz shipping," "IRGC naval," "oil supply disruption," "Fed interest rate"
3. Found 3 relevant articles — one about IRGC naval exercises (reinforces n2.2), one about Saudi production meeting (relevant to n3.2), one about US diplomatic backchannel (relevant to n1.3)
4. Re-evaluate affected nodes: n2.2 stays at 90%, n3.2 updated 68% → 65%, n1.3 stays at 82%
5. Recalculate thesis-implied recession: 45% → 44%
6. Check strategy conditions: edge = 44% - 36% = 8 points > 7 ✓, all upstream > 60% ✓
7. No action needed — conditions still met but no new trigger

**Heartbeat at 14:30 UTC (CPI data release day):**
1. Fetch prices: Recession YES @ 37¢
2. Search finds CPI came in at 6.2% vs 5.8% expected
3. This maps to n4 (Fed trapped) — inflation data exceeding expectations strengthens the case that the Fed can't cut into this
4. n4 updated: 58% → 67%
5. Downstream cascade: n5 (recession) updated: 45% → 52%
6. Strategy evaluation: thesis 52% vs market 37% = 15 points edge
7. **SIGNAL FIRED:** "Edge widened to 15 points. CPI exceeded expectations, strengthening Fed-trapped thesis. Consider adding to position."

You get a notification. You review the reasoning. You decide to add.

## Phase 6: Signal Injection — When the World Changes

The causal tree isn't static. Sometimes a single event fundamentally changes the picture.

**Scenario: Surprise diplomatic breakthrough**

News breaks: Iran and US agree to temporary Hormuz transit corridor. This is a direct hit to node n1.3 (no diplomatic exit).

The monitor picks this up on its next cycle:

1. Tavily search finds the diplomatic headline
2. LLM evaluation: n1.3 drops from 82% to 35%
3. Upstream cascade: n1 (war persists) drops from 88% to 62%
4. n2 (Hormuz blocked) drops from 97% to 58%
5. n3 (oil elevated) drops from 64% to 38%
6. n5 (recession) drops from 52% to 22%
7. **EXIT SIGNAL:** "Thesis confidence collapsed. n1.3 (no diplomatic exit) dropped to 35%. Recession implied probability now 22%, below 30% threshold. Strategy exit conditions met."

You get the signal. You exit the position. The causal tree told you *exactly why* — not "the price went down" but "a diplomatic breakthrough undermined the upstream assumption that was supporting the entire chain."

## The Full Loop

```
scan → thesis → causal tree → edge detection → strategy → monitor → signal → action
  ↑                                                                            |
  └────────────────────────── new thesis ──────────────────────────────────────┘
```

After exiting, you're back at the beginning. Maybe the diplomatic breakthrough creates a *new* thesis: "temporary corridor collapses, war re-escalates." Or maybe you scan for entirely different opportunities. The loop continues.

## What This Is Not

This is not automated trading. SimpleFunctions doesn't execute trades for you. It:

- **Structures** your thinking (causal tree)
- **Quantifies** your edge (thesis vs. market)
- **Monitors** 24/7 (heartbeat + news search)
- **Signals** when conditions change (notifications)

You make every execution decision. The agent does the work that humans are bad at — watching continuously, updating without emotion, maintaining discipline at 3am.

## Why This Matters

Most prediction market traders operate on vibes. They have a feeling about recession, they buy a contract, they check the price three times a day, they panic when it moves against them, they hold when they should sell.

The full trading loop replaces vibes with structure. Every belief is decomposed. Every assumption is testable. Every update is traced to a specific causal node. Every action is triggered by a defined condition.

It's not about being smarter than the market. It's about being *more disciplined* — and discipline is exactly what an agent provides.