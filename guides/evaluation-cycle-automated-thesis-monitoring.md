# The Evaluation Cycle: How Automated Thesis Monitoring Works

> Inside the heartbeat loop that powers continuous thesis monitoring: news scanning, price refreshes, milestone checks, LLM evaluation, confidence updates, and smart scheduling that adapts to market volatility.

**Category:** architecture | **Author:** SimpleFunctions | **Reading time:** 10 min

---
A causal tree is only as good as its last update. If you built a thesis on Monday and never re-evaluated it, by Friday the world has moved and your model is stale. News broke, prices shifted, diplomatic meetings happened, data was released. Your 72% confidence in recession might be 50% now — or 85% — but you wouldn't know without re-evaluation.

The evaluation cycle solves this. It's a recurring loop — a "heartbeat" — that automatically scans for new information, checks market prices, re-evaluates your causal tree nodes, and alerts you when something meaningful changes.

This article explains exactly how the loop works, what happens at each stage, and how smart scheduling keeps costs down without missing critical events.

## The Heartbeat Loop

Every evaluation cycle follows the same six steps:

```
┌──────────────┐
│  1. NEWS     │  Tavily search for each causal node
│     SCAN     │  "Iran Hormuz shipping", "OPEC production", etc.
└──────┬───────┘
       ▼
┌──────────────┐
│  2. PRICE    │  Kalshi API: fetch current orderbook
│     REFRESH  │  for all mapped contracts
└──────┬───────┘
       ▼
┌──────────────┐
│  3. MILESTONE│  Check if any node has a pending
│     CHECK    │  resolution event (data release, vote, etc.)
└──────┬───────┘
       ▼
┌──────────────┐
│  4. LLM      │  Send node + news + prices to evaluator
│     EVALUATE │  Get updated confidence for each node
└──────┬───────┘
       ▼
┌──────────────┐
│  5. CONFIDENCE│ Propagate node changes through tree
│     UPDATE   │  Recalculate thesis-implied probabilities
└──────┬───────┘
       ▼
┌──────────────┐
│  6. NOTIFY   │  If delta exceeds threshold, fire
│     / WEBHOOK│  notification or webhook
└──────────────┘
```

Let's walk through each step with a real example.

### Step 1: News Scan (Tavily)

The monitor constructs search queries from the causal tree node descriptions. For the Iran war thesis, it generates queries like:

```
Node n1.2  "Iran continues retaliation"  → search: "IRGC retaliation Hormuz 2026"
Node n1.3  "No diplomatic exit"           → search: "Iran US diplomatic negotiations"
Node n3.1  "SPR drawdown insufficient"    → search: "strategic petroleum reserve drawdown rate"
Node n3.2  "OPEC doesn't compensate"      → search: "OPEC production decision output"
```

Tavily returns structured results: title, snippet, URL, and publication date. The monitor filters for recency (last 2 hours for normal cycles, last 30 minutes for high-volatility cycles) and deduplicates against previously seen articles.

A typical scan returns 2-5 relevant articles across all nodes. Most cycles return nothing new — and that's fine. The system is watching for changes, not generating activity.

**Cost per scan:** Each Tavily search costs approximately $0.01. With 4-6 searches per thesis per cycle, that's $0.04-0.06 per cycle. At 24 cycles per day (one every 60 minutes), that's roughly $1.00-1.50 per thesis per day, or about $30-45 per thesis per month.

### Step 2: Price Refresh (Kalshi Orderbook)

The monitor hits the Kalshi API for current orderbook data on every mapped contract:

```
GET /trade-api/v2/markets/KXRECSSNBER-26/orderbook

Response:
{
  "orderbook": {
    "yes": [
      [35, 8],    // 8 contracts available at 35¢
      [36, 15],
      [37, 22]
    ],
    "no": [
      [65, 12],   // 12 contracts available at 65¢ (= YES bid at 35¢)
      [64, 20],
      [63, 18]
    ]
  }
}
```

The monitor extracts:
- **Best ask (YES):** 35¢
- **Best bid (YES):** 35¢ (computed as 100¢ - best NO ask of 65¢)
- **Spread:** 0¢ (tight market)
- **Depth (YES, within 5¢):** 45 contracts on ask side
- **Price delta since last check:** +1¢ (was 34¢ last cycle)

The price refresh is free — Kalshi doesn't charge for API access. But the monitor uses a delta approach to avoid unnecessary processing: if no price has moved more than 1¢ since the last cycle, the price data is flagged as "stable" and downstream processing is lighter.

### Step 3: Milestone Check

Some causal tree nodes have associated milestones — specific dates or events that will significantly update confidence:

```
n3.1 (SPR drawdown): milestone = "DOE Weekly Petroleum Status Report" every Wednesday 10:30am ET
n4.1 (Inflation):    milestone = "BLS CPI Release" on March 12, 2026
n1.3 (Diplomacy):    milestone = "UN Security Council meeting" on March 15, 2026
```

If the current time is within 2 hours of a milestone, the monitor escalates the evaluation priority for that node. It also adjusts the Tavily search to specifically look for the milestone event results.

On milestone days, the evaluation cycle runs more frequently for the affected nodes — every 15 minutes instead of every 60 minutes — because the information environment is changing rapidly.

### Step 4: LLM Evaluation

This is the core of the cycle. The evaluator sends each affected node to the LLM with full context:

**Input to LLM:**

```
You are evaluating a causal tree node for a prediction market thesis.

Node: n3.2 "OPEC doesn't compensate for Hormuz disruption"
Current confidence: 0.68
Last evaluated: 2 hours ago

Recent news (from Tavily):
1. "Saudi Arabia signals willingness to discuss emergency production increase"
   - Source: Reuters, 45 minutes ago
   - Key quote: "Saudi energy minister said 'all options are on the table' regarding
     Hormuz disruption impact"

2. "OPEC+ emergency meeting scheduled for March 22"
   - Source: Bloomberg, 1 hour ago

Current market prices:
- Oil > $100 Q3 2026: 41¢ (was 43¢ yesterday)
- OPEC production increase contract: 38¢ (was 32¢ yesterday)

Evaluate: Should the confidence for this node change? Provide your updated
probability and cite specific evidence.
```

**Output from LLM:**

```json
{
  "nodeId": "n3.2",
  "previousConfidence": 0.68,
  "updatedConfidence": 0.55,
  "confidenceDelta": -0.13,
  "summary": "Saudi signaling willingness to increase production meaningfully reduces confidence that OPEC won't compensate. The emergency meeting scheduling suggests concrete action is being considered, not just rhetoric. However, OPEC+ has historically been slow to agree on increases, and Saudi spare capacity may be insufficient to fully offset Hormuz volumes (~17M bbl/day). Dropping from 0.68 to 0.55 to reflect the increased probability of at least partial compensation.",
  "evidenceCited": [
    "Saudi energy minister statement (Reuters, 45min ago)",
    "OPEC+ emergency meeting scheduled March 22 (Bloomberg)"
  ],
  "uncertaintyFactors": [
    "Meeting scheduled but outcome unknown",
    "Saudi spare capacity is ~2-3M bbl/day vs 17M bbl/day Hormuz flow",
    "OPEC+ consensus historically difficult"
  ]
}
```

**Cost per evaluation:** Using Claude Haiku via OpenRouter, each node evaluation costs approximately $0.03-0.05 depending on context size. A full thesis with 12 nodes costs $0.36-0.60 per complete evaluation. With 24 evaluations per day, that's $8.64-14.40 per thesis per day.

In practice, not every node is re-evaluated every cycle. If no new news affects a node and no price has moved, the node is skipped. A typical cycle evaluates 3-5 nodes out of 12, costing $0.09-0.25 per cycle.

### Step 5: Confidence Update and Propagation

When node confidences change, the tree propagates the changes downstream:

```
n3.2 updated: 0.68 → 0.55 (Δ = -0.13)

Propagation:
  n3 (Oil stays elevated): recalculated 0.64 → 0.54
  n4 (Fed trapped):        recalculated 0.58 → 0.51
  n5 (Recession):          recalculated 0.45 → 0.37

Edge update:
  KXRECSSNBER-26: thesis 0.37 vs market 0.35 = +2¢ edge (was +10¢)
```

The propagation is purely mathematical — no LLM involved. It follows the tree structure and the conditional probability relationships defined when the tree was created. This step is instantaneous.

### Step 6: Notification / Webhook

The monitor checks if any delta exceeds the notification threshold. Default thresholds:

- **Node delta > 0.10:** Notify (a single node changed by more than 10 percentage points)
- **Thesis delta > 0.08:** Notify (the terminal node changed by more than 8 points)
- **Edge sign flip:** Always notify (your edge went from positive to negative or vice versa)
- **Strategy condition triggered:** Always notify

For our example, n5 dropped from 0.45 to 0.37 (Δ = -0.08), hitting the thesis delta threshold. The notification fires:

```
[THESIS UPDATE] iran-war
  Trigger: OPEC compensation signals weakened oil thesis
  n3.2 (OPEC doesn't compensate): 68% → 55%
  n5 (Recession): 45% → 37%
  Edge on KXRECSSNBER-26: +10¢ → +2¢

  Action consideration: Edge approaching zero. Consider whether
  thesis still warrants position.
```

You receive this via your configured channel (email, webhook, Slack, or in-app notification). You review the reasoning, check the cited evidence, and decide whether to adjust your position.

## Smart Scheduling

Running a full evaluation every 15 minutes for every thesis would be expensive and wasteful. Smart scheduling adapts the cycle frequency based on market conditions:

**Normal mode (60-minute cycles):**
- No significant price movements (≤3¢ change on any mapped contract)
- No milestone events within 24 hours
- No breaking news detected
- Cost: ~$1.50 per thesis per day

**High-volatility mode (30-minute cycles):**
- Any mapped contract moved >3¢ in the last hour
- Milestone event within 6 hours
- Breaking news detected on a critical node
- Cost: ~$3.00 per thesis per day

**Spike mode (15-minute cycles):**
- Any mapped contract moved >8¢ in the last hour
- Milestone event happening now (e.g., CPI release)
- Multiple nodes affected simultaneously
- Cost: ~$6.00 per thesis per day

**Backlog skip:**
If the previous evaluation is still running when the next cycle is scheduled, the new cycle is skipped. This prevents queue buildup during complex evaluations and avoids double-counting news articles. The skip is logged so you can see if evaluations are taking too long (which usually means the LLM context is too large — consider pruning low-impact nodes).

## The Delta API for Efficient Polling

The monitor doesn't re-fetch everything on every cycle. It uses a delta approach:

**News delta:** Tavily searches include a `search_depth: "basic"` parameter for routine scans and `search_depth: "advanced"` only when a node has changed or a milestone is approaching. The `days` parameter is set to 1 for normal mode, narrowing to the last few hours during high-volatility mode.

**Price delta:** The Kalshi orderbook is fetched every cycle (it's free), but downstream processing only triggers if a price has moved more than 1¢ since the last check. Stable prices mean the existing node evaluations are still valid.

**Evaluation delta:** Only nodes that have new information (news or price movement) are re-evaluated. Nodes with no new inputs keep their current confidence. This cuts LLM costs by 60-70% compared to evaluating every node every cycle.

## A Complete Cycle in Practice

Here's what a single evaluation cycle looks like in the system logs:

```
[2026-03-19T14:30:00Z] Heartbeat #1847 starting for thesis: iran-war
[2026-03-19T14:30:01Z] News scan: 5 queries dispatched to Tavily
[2026-03-19T14:30:03Z] News scan: 2 new articles found
  - Reuters: "Saudi signals production flexibility" (affects n3.2)
  - Bloomberg: "OPEC+ emergency meeting March 22" (affects n3.2)
[2026-03-19T14:30:04Z] Price refresh: 4 contracts fetched
  - KXRECSSNBER-26: 35¢ (unchanged)
  - KXOIL100-26Q3: 41¢ (−2¢)
  - KXFEDCUT-26JUN: 65¢ (unchanged)
  - KXIRANWAR-26: 84¢ (unchanged)
[2026-03-19T14:30:04Z] Milestone check: none within 24h
[2026-03-19T14:30:05Z] Evaluation: 2 nodes queued (n3.2, n3 parent)
[2026-03-19T14:30:08Z] n3.2 evaluated: 0.68 → 0.55 (Δ = -0.13)
[2026-03-19T14:30:09Z] Propagation: n3 0.64→0.54, n4 0.58→0.51, n5 0.45→0.37
[2026-03-19T14:30:09Z] Edge update: KXRECSSNBER-26 +10¢ → +2¢
[2026-03-19T14:30:09Z] NOTIFICATION FIRED: thesis delta -0.08 exceeds threshold
[2026-03-19T14:30:10Z] Heartbeat #1847 complete. Duration: 10.2s. Cost: $0.08
```

Total cycle time: 10 seconds. Total cost: 8 cents. The system found two relevant articles, re-evaluated one node, propagated the change through the tree, detected a meaningful shift, and notified you — all while you were having lunch.

## Why This Matters

Manual monitoring fails in predictable ways:

1. **You sleep.** The most market-moving news often breaks outside US market hours.
2. **You get bored.** Checking five contracts across three exchanges every two hours is tedious. You stop doing it within a week.
3. **You miss context.** A headline about Saudi production doesn't obviously connect to your recession thesis unless you're actively tracing the causal chain.
4. **You overreact or underreact.** Without a structured evaluation, you either panic at every headline or ignore them all.

The evaluation cycle replaces all of this with a mechanical process. It watches when you can't. It connects news to causal nodes automatically. It quantifies the impact instead of leaving it to your gut. And it only bothers you when something actually matters.

The cycle doesn't make trading decisions. It makes *awareness* decisions: what changed, how much, and why. Your job is to decide what to do with that awareness.