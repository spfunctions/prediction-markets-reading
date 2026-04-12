# How I track my macro thesis across 49 Kalshi contracts without checking the screen

> A causal tree, 12 edges, and a heartbeat that runs every 15 minutes so I don't have to.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 9 min

---
I have a thesis: the US ends up in a shooting war with Iran before December 2026. If that happens, oil clears $135/barrel, the Fed pauses or reverses rate cuts, CPI re-accelerates, and the probability of an NBER-declared recession roughly doubles.

That's four causal links. Each link touches multiple Kalshi contracts. In total, my thesis maps to 49 markets. I used to check them one by one, tabbing between browser windows, copying prices into a spreadsheet. That lasted about a week before I missed a move on KXCPI that would have changed my position sizing on KXWTIMAX-26DEC31-T135.

So I built something. This is a walkthrough of how it works.

## The thesis as a causal tree

I use SimpleFunctions to create and manage theses. The CLI command is:

```bash
sf create thesis \
  --name "Iran War → Oil → Recession" \
  --root "US-Iran military conflict by Dec 2026" \
  --edges 12
```

A thesis in SimpleFunctions is a directed acyclic graph. The root node is the core claim. Child nodes are downstream consequences. Edges are causal links with weights. Here's the tree for this thesis:

```
Iran War (root)
├── Oil > $135/bbl
│   ├── Energy sector earnings surge
│   ├── Consumer spending compression
│   └── CPI re-acceleration
│       ├── Fed holds or hikes
│       │   ├── Housing market freeze
│       │   └── Credit tightening
│       └── Real wage decline
├── Defense spending increase
│   └── Deficit expansion
├── Supply chain disruption (Strait of Hormuz)
│   ├── Shipping cost spike
│   └── Manufacturing slowdown
└── Recession (NBER)
    ├── Unemployment rise
    └── GDP contraction
```

12 edges. Each edge has a conditional probability I estimated from historical base rates and current conditions. For example, P(Oil > $135 | Iran war) = 0.78, based on the 1990 Gulf War spike pattern adjusted for current spare capacity.

## 49 markets mapped

Each node maps to one or more Kalshi contracts. Here's a subset of the edge table:

```
┌─────────────────────────────────┬────────────────────────────────────┬───────┬──────────┐
│ Edge                            │ Kalshi Ticker                      │ Side  │ Last     │
├─────────────────────────────────┼────────────────────────────────────┼───────┼──────────┤
│ Oil > $135/bbl                  │ KXWTIMAX-26DEC31-T135              │ YES   │ $0.08    │
│ Recession (NBER)                │ KXRECSSNBER-26                     │ YES   │ $0.41    │
│ Fed Dec decision                │ KXFEDDECISION-26DEC17-T450         │ YES   │ $0.22    │
│ CPI YoY > 4%                   │ KXCPI-26DEC-T4.0                   │ YES   │ $0.05    │
│ Gas > $4/gal avg                │ KXAAAGASM-26AUG-T4.00              │ YES   │ $0.12    │
│ Unemployment > 5%              │ KXUNRATE-26DEC-T5.0                │ YES   │ $0.18    │
│ GDP Q3 negative                 │ KXGDP-26Q3-TNEG                   │ YES   │ $0.14    │
│ S&P 500 < 4800                  │ KXSPX-26DEC31-T4800               │ NO    │ $0.89    │
└─────────────────────────────────┴────────────────────────────────────┴───────┴──────────┘
```

The full table has 49 rows. Some nodes map to multiple contracts at different strike prices—I hold oil at $120, $125, $130, and $135 because the payoff structure is different at each level, and the implied probability curve tells me where the market is pricing risk versus where I think it should be.

## The heartbeat

Once the thesis is created and markets are mapped, I turn on the heartbeat:

```bash
sf heartbeat start \
  --thesis "iran-war" \
  --interval 15m \
  --notify telegram
```

Every 15 minutes, the heartbeat does four things:

1. **News scan**: Pulls recent headlines from 40+ sources filtered to the thesis keywords. Scores relevance using an LLM. Anything above 0.7 relevance gets flagged.

2. **Price check**: Fetches current prices for all 49 contracts via the Kalshi API. Computes delta from last check and from my entry price.

3. **Adversarial search**: This is the one most people skip. The agent actively searches for evidence that *disconfirms* my thesis. It looks for peace talks, diplomatic signals, OPEC spare capacity announcements, anything that would weaken the causal chain. I don't want an agent that confirms my bias. I want one that tries to kill my thesis every 15 minutes.

4. **Kill condition check**: Each thesis has explicit kill conditions. For this one: "US and Iran announce bilateral talks with EU mediation" or "OPEC demonstrates 4M+ bbl/day spare capacity deployment within 30 days." If a kill condition triggers, the agent flags it immediately.

Here's what an actual heartbeat output looks like:

```
[2026-03-28T14:30:02Z] HEARTBEAT iran-war (cycle #4,217)

📡 NEWS (3 relevant / 41 scanned)
  [0.91] Reuters: IRGC naval exercises expand to full Strait of Hormuz width
  [0.82] FT: Saudi Aramco delays maintenance on Khurais field
  [0.74] AP: Pentagon repositions Eisenhower carrier group to Gulf of Oman

📊 PRICES (7 moved >2% since last cycle)
  KXWTIMAX-26DEC31-T135   $0.08 → $0.09  (+12.5%)  ⚡
  KXRECSSNBER-26          $0.41 → $0.43  (+4.9%)
  KXAAAGASM-26AUG-T4.00   $0.12 → $0.13  (+8.3%)
  KXFEDDECISION-26DEC17   $0.22 → $0.22  (0.0%)
  KXCPI-26DEC-T4.0        $0.05 → $0.05  (0.0%)
  KXUNRATE-26DEC-T5.0     $0.18 → $0.19  (+5.6%)
  KXGDP-26Q3-TNEG         $0.14 → $0.15  (+7.1%)

🔍 ADVERSARIAL SEARCH
  No disconfirming signals found this cycle.
  Last disconfirm: 2026-03-26 (Oman backchannel report — assessed low credibility)

🛑 KILL CONDITIONS: CLEAR
  [✗] Bilateral talks w/ EU mediation
  [✗] OPEC 4M+ bbl/day spare demo

⚖️  THESIS CONFIDENCE: 0.34 → 0.36 (+0.02)
  Trigger: IRGC exercise scope + carrier repositioning
  Edge update: P(conflict | current posture) adjusted 0.31 → 0.33
```

That confidence number—0.36—is not my probability of Iran war. It's the composite confidence that my entire causal chain holds. The root node might be at 0.33 probability, but the downstream conditional probabilities compound. 0.36 means "the expected value of the full position, weighted by edge probabilities, is positive at current Kalshi prices."

When that number changes by more than 0.03 in either direction, I get a Telegram notification:

```
🔔 [iran-war] Confidence Δ +0.04 (0.32 → 0.36)
Primary driver: IRGC exercise escalation
Suggested action: Review KXWTIMAX-26DEC31-T135 position (currently 200 contracts YES @ $0.06 avg)
Edge table: https://app.simplefunctions.dev/thesis/iran-war/edges
```

## What I still check manually

I want to be honest about this. The heartbeat handles monitoring, but I still do several things by hand:

**Order execution.** I don't auto-trade. The Kalshi API supports it, and I could wire up SimpleFunctions to place orders when conditions are met. I choose not to. My thesis involves tail risk events, and I don't trust any automated system to size positions correctly in a crisis. When the heartbeat tells me to act, I open Kalshi and place the order myself. This takes 2-3 minutes. That's fine.

**Edge weight calibration.** Every Sunday, I spend about 30 minutes reviewing the conditional probabilities in my causal tree. The agent suggests adjustments based on the week's data, but I make the final call. These are judgment calls about geopolitics and macro, and I'm not ready to outsource them entirely. Maybe with a longer track record I will.

**New market discovery.** When Kalshi lists a new contract that's relevant to my thesis—say a new CPI bracket or a new GDP quarter—I add it manually. The agent flags candidates ("New market KXCPI-26SEP-T3.5 may be relevant to node CPI re-acceleration"), but I decide whether to include it and where it fits in the tree.

**Cross-platform comparison.** I also trade on Polymarket for markets Kalshi doesn't cover. The heartbeat can monitor both, but reconciling positions across platforms is something I do in a spreadsheet. I know. I should automate this. I haven't.

## The math of not watching

Before the heartbeat, I spent roughly 2 hours per day monitoring my positions. Checking news, refreshing prices, scrolling Twitter for geopolitical takes, re-running my spreadsheet calculations.

Now I spend about 20 minutes per day. The Sunday calibration session adds 30 minutes per week. That's roughly 90% less time on monitoring, which I've reallocated to research and finding new theses.

More importantly, I don't miss moves. The KXWTIMAX-26DEC31-T135 contract went from $0.04 to $0.09 over three days in early March when the Houthi escalation news broke. The heartbeat caught it on the first 15-minute cycle after the Reuters wire. I was on it within 20 minutes of the first price move. Before, I might have seen it 6-8 hours later when I sat down for my evening check.

## Why this is different from a screener

You could build some of this with a price alert on Kalshi's app. Set alerts on 49 contracts, get notified when they move. That handles price monitoring.

What it doesn't handle:

- **Causal reasoning.** A screener tells you KXWTIMAX moved. It doesn't tell you *why*, and it doesn't connect that move to your recession thesis. The heartbeat knows the causal tree. When oil moves, it automatically re-evaluates the downstream nodes.

- **Adversarial search.** No screener is actively trying to kill your thesis. This is the feature I value most. Every 15 minutes, the agent tries to prove me wrong. It has found three credible disconfirming signals in the past month that I would have missed.

- **Kill conditions.** Hard stops. If my thesis is dead, I want to know immediately, not when I notice the prices have already moved against me.

- **Composite confidence.** A single number that tells me whether my overall position has positive expected value at current market prices, updated every 15 minutes. No spreadsheet required.

## The "Claude Code for Kalshi traders" explanation

People ask me what SimpleFunctions is. The most accurate description: it's Claude Code, but for prediction market traders.

Claude Code gives developers an AI agent that understands their codebase, runs in the terminal, and does real work—not just autocomplete, but actual multi-step reasoning about code. SimpleFunctions does the same thing for market theses. It understands your causal model, runs continuously, and does real work—monitoring, adversarial search, confidence updates—not just price alerts.

The thesis is your codebase. The heartbeat is your CI pipeline. The kill conditions are your test suite. The edge table is your dependency graph.

If that analogy makes sense to you, you're the target user.

## Getting started

I built SimpleFunctions because I needed it for my own trading. The Iran thesis is one of seven I currently run. The others cover Fed policy, China-Taiwan, US housing, AI regulation, crypto ETF flows, and the 2026 midterms. Each has its own causal tree, its own markets, its own heartbeat.

If you trade on Kalshi and spend too much time staring at prices, the tool is at [simplefunctions.dev](https://simplefunctions.dev).