# Building a prediction market monitoring system: heartbeat architecture for 24/7 edge tracking

> Markets move at 3am and your edge decays while you sleep — here is the architecture for a system that never stops watching.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 11 min

---
I trade prediction markets on Kalshi with real money. Not paper trading, not backtesting — actual positions on recession odds, oil contracts, Fed rate decisions. The single biggest problem I've had isn't finding edges. It's keeping them.

An edge in prediction markets has a half-life. You identify a mispricing on Monday afternoon, and by Tuesday morning the market has moved because a Reuters headline dropped at 2am, or a Polymarket whale shifted the price, or the underlying commodity gapped overnight. You wake up, check your positions, and the edge you carefully constructed is either gone or inverted.

This is the core problem a heartbeat system solves. Not finding edges — monitoring them.

## What happens while you sleep

Prediction markets are 24/7. Kalshi's orderbook doesn't close. Polymarket runs on Polygon and never stops. But the events these contracts price — geopolitical conflicts, economic data releases, policy decisions — happen on their own schedule. A missile strike at 3am Eastern. A surprise PBOC rate cut during the Asian session. An early morning jobs report revision.

If you're running a thesis on the US-Iran conflict cascading into recession (which I am), the causal chain looks like this:

```
War persists → Hormuz blocked → Oil elevated → Consumer pressure → Fed trapped → Recession
```

Each node in this chain can shift at any hour. Hormuz mine activity detected by satellite at midnight. Oil futures gapping on Globex at 4am. A diplomatic backchannel leak hitting X/Twitter at 1am.

Without a monitoring system, you're trading blind for 16 hours a day.

## The heartbeat architecture

A heartbeat is a periodic cycle that runs every 15 minutes, 24/7. Each cycle does the same seven things in the same order:

### 1. News scan

Query Tavily's search API for each thesis you're tracking. Not a broad news sweep — targeted queries built from your causal tree nodes.

For an Iran war thesis, the queries are specific:
- `"Hormuz strait" OR "IRGC navy" OR "mine sweeping" site:reuters.com OR site:apnews.com`
- `"Iran nuclear" OR "JCPOA" OR "diplomatic talks" -opinion`
- `"Strait of Hormuz" shipping insurance rates`

Tavily returns structured results with relevance scores. Cost: ~$0.01 per search. For a thesis with 5 targeted queries, that's $0.05 per cycle.

### 2. Price refresh

Pull current prices from every market your thesis maps to:

```typescript
// Kalshi
const kalshiContracts = [
  'KXRECESSION-26',        // Recession 2026
  'KXWTIMAX-26DEC31-T150', // WTI $150
  'KXGASPRICE-26MAR-T450', // Gas $4.50 March
  'KXFEDRATE-26JUN18',     // Fed rate June meeting
];

// Polymarket
const polyContracts = [
  '0x1234...recession',    // Recession condition ID
  '0x5678...iran-war',     // Iran war continuation
];
```

Both APIs are free for read-only market data. The Kalshi REST API returns mid-market prices, orderbook snapshots, and recent volume. Polymarket's CLOB API gives you the same via their Gamma Markets endpoint.

### 3. Milestone tracking

Check whether any causal tree node has crossed a predefined threshold. Milestones are the structural events that matter:

- Oil crossing $120 (node n3 shifts from "elevated" to "crisis")
- Fed funds futures pricing >75% chance of emergency cut
- Shipping insurance rates for Hormuz transit exceeding 5% of cargo value
- A specific Kalshi contract crossing 80 cents (market is near-certain)

Milestone tracking uses Databento for real-time commodity and futures data. The `GLBX.MDP3` dataset covers CME Globex — WTI futures (CL), Fed funds futures (ZQ), Treasury futures (ZN). Cost: Databento charges per query, roughly $0.002 per snapshot for the symbols I track.

For non-financial milestones, the Tavily results from step 1 feed into a quick LLM classification: "Does this article indicate [milestone X] has occurred?" Binary yes/no. Cheap and fast.

### 4. Adversarial search

This is the step most monitoring systems skip. Actively search for evidence that *contradicts* your thesis.

If your thesis is "war persists → recession," the adversarial search looks for:
- Diplomatic progress signals
- Hormuz reopening indicators
- SPR release announcements that could suppress oil
- Positive economic data that undermines recession probability

This isn't optional. Confirmation bias is the biggest risk in thesis-driven trading. The system should be trying to kill your thesis every 15 minutes. If it can't, the thesis is strong.

### 5. Kill condition check

Every thesis has kill conditions — upstream nodes that, if they flip, invalidate the entire downstream chain. For the Iran war thesis:

```
KILL CONDITIONS:
- Iran ceasefire announced and verified → entire thesis dead
- Hormuz confirmed open and insurance rates normalize → oil chain dead
- Oil drops below $80 sustained 2 weeks → recession chain dead
- n1.2 (Iran retaliates) drops below 30% → war persistence dead
```

The kill condition check compares current node probabilities against these thresholds. If a kill condition triggers, the system doesn't just alert you — it marks the thesis as requiring immediate review and flags all associated positions.

### 6. Evaluation

This is where the LLM does real work. All the data from steps 1-5 feeds into a structured evaluation prompt:

```
You are evaluating thesis [slug] at cycle [timestamp].

Current causal tree state: [JSON]
New signals since last evaluation: [array of news, price changes, milestones]
Adversarial findings: [array]
Kill condition status: [none triggered / WARNING: ...]

For each affected node, output:
- Updated probability (0-100)
- Confidence in the update (high/medium/low)
- One-line reasoning

Then output overall thesis health: STRONG / DEGRADING / CRITICAL / DEAD
```

The LLM sees structured context, not raw data. It's not browsing the internet or making stuff up — it's updating probabilities on a fixed causal tree based on specific, sourced evidence.

Model: Claude Sonnet. Cost per evaluation: ~$0.02-0.04 depending on context size. For a thesis with 8 nodes and 3-5 new signals, the prompt is typically 2000-3000 tokens in, 500-800 tokens out.

### 7. Notification

If the evaluation produced meaningful changes, push them out:

- **Telegram**: Push notification with the delta summary. "Iran thesis: n3 (oil elevated) shifted 64% → 71%. New edge on KXRECESSION-26: +4 cents. No kill conditions triggered."
- **Delta API**: POST the changes to `/api/thesis/{id}/changes`. Agents polling this endpoint get only what changed — typically 50-200 bytes when something moved, 0 bytes when nothing did.
- **Webhook**: For custom integrations.

If nothing changed — which is most cycles at 3am on a Tuesday — the system logs the heartbeat and moves on. No spam.

## The full cycle in numbers

Here's what one heartbeat cycle costs for a single thesis:

| Step | Source | Cost per cycle |
|------|--------|---------------|
| News scan | Tavily (5 queries) | $0.05 |
| Price refresh | Kalshi + Polymarket API | $0.00 |
| Milestone tracking | Databento (4 symbols) | ~$0.008 |
| Adversarial search | Tavily (3 queries) | $0.03 |
| Kill condition check | Local computation | $0.00 |
| Evaluation | Claude Sonnet | ~$0.03 |
| Notification | Telegram API | $0.00 |
| **Total per cycle** | | **~$0.12** |

96 cycles per day (every 15 minutes) = **~$0.61 per thesis per day**.

That's $18.30/month to have an always-on monitoring system for one thesis. If you're running 5 theses simultaneously, it's about $91.50/month. Compare that to the cost of missing a 3am signal that moves your position 15 cents — on a $1,000 position, that's $150 gone.

## What the output looks like

Every heartbeat produces a structured output. Here's a real example from a cycle that detected a meaningful change:

```json
{
  "thesisSlug": "iran-war",
  "cycleTimestamp": "2026-03-28T03:15:00Z",
  "cycleNumber": 4847,
  "signalsIngested": 3,
  "nodesUpdated": [
    {
      "nodeId": "n3",
      "label": "Oil stays elevated",
      "previousProbability": 64,
      "newProbability": 71,
      "reason": "Databento: CL front-month settled $118.40, up $3.20. Tavily: Reuters reports VLCC rates for Hormuz transit doubled this week."
    }
  ],
  "edgesUpdated": [
    {
      "contract": "KXRECESSION-26",
      "previousEdge": 33,
      "newEdge": 37,
      "marketPrice": 35,
      "thesisPrice": 72
    }
  ],
  "killConditions": "none triggered",
  "thesisHealth": "STRONG",
  "costUsd": 0.118
}
```

This is what gets stored. This is what the delta API serves. This is what your agent reads.

## The context snapshot

The heartbeat feeds into a larger structure: the thesis context. When you (or your agent) need the full picture, you call:

```bash
$ sf context iran-war --json
```

This returns everything:

```json
{
  "thesis": {
    "slug": "iran-war",
    "title": "US-Iran War → Recession Cascade",
    "status": "active",
    "health": "STRONG",
    "lastHeartbeat": "2026-03-28T03:15:00Z"
  },
  "causalTree": {
    "nodes": [
      { "id": "n1", "label": "War persists", "probability": 88, "children": ["n1.1", "n1.2", "n1.3"] },
      { "id": "n1.1", "label": "US initiated strikes", "probability": 99 },
      { "id": "n1.2", "label": "Iran continues retaliation", "probability": 85 },
      { "id": "n1.3", "label": "No diplomatic exit", "probability": 82 },
      { "id": "n2", "label": "Hormuz blocked", "probability": 97 },
      { "id": "n3", "label": "Oil stays elevated", "probability": 71 },
      { "id": "n4", "label": "Recession", "probability": 45 }
    ]
  },
  "edges": [
    { "contract": "KXRECESSION-26", "market": 35, "thesis": 72, "edge": 37, "liquidity": "high" },
    { "contract": "KXWTIMAX-26DEC31-T150", "market": 38, "thesis": 75, "edge": 37, "liquidity": "high" },
    { "contract": "KXGASPRICE-26MAR-T450", "market": 14, "thesis": 55, "edge": 41, "liquidity": "medium" }
  ],
  "recentSignals": [
    { "timestamp": "2026-03-28T03:12:00Z", "type": "news", "source": "reuters", "summary": "VLCC rates for Hormuz transit doubled" },
    { "timestamp": "2026-03-28T03:10:00Z", "type": "price", "source": "databento", "summary": "CL front-month $118.40 (+$3.20)" }
  ],
  "killConditions": [
    { "condition": "Iran ceasefire verified", "status": "not triggered" },
    { "condition": "Hormuz open + insurance normalized", "status": "not triggered" },
    { "condition": "Oil below $80 sustained 2 weeks", "status": "not triggered" }
  ]
}
```

This is the structure an AI agent consumes. Not "here's a news feed, figure it out." It's "here's a causal model with current probabilities, here are the contracts where the model disagrees with the market, here's what changed recently, and here are the conditions that would kill the thesis." Structured judgment, not raw data.

## Multi-source ingestion: why each source matters

The heartbeat pulls from four categories of data. Each serves a different purpose.

**News (Tavily)**: Event detection. "Did something happen?" Tavily's search API is the best cost-to-quality ratio I've found for programmatic news retrieval. It returns clean text, relevance scores, and publication dates. $0.01/query.

**Prediction market prices (Kalshi REST API, Polymarket CLOB API)**: Market consensus. "What does the crowd think?" These are free APIs. Kalshi requires authentication for trading but not for market data reads. Polymarket's CLOB runs through their Gamma Markets endpoint — no auth needed for prices.

**Social signals (X/Twitter)**: Early warning. Prediction market prices often lag social media by 30-60 minutes on breaking events. A verified journalist tweeting about Hormuz activity will move the market, but not for another hour. The heartbeat catches it on the next cycle. I use a filtered stream targeting specific accounts and keywords relevant to each thesis.

**Traditional markets (Databento)**: Ground truth for financial nodes. If your causal tree includes "oil stays above $100," you need the actual oil price, not a prediction market proxy. Databento provides tick-level and snapshot data for CME Globex instruments. CL (WTI crude), ZQ (Fed funds futures), ZN (10-year Treasuries), ES (S&P 500 futures). The data is authoritative and timestamped to the microsecond.

## What to do with the output

The heartbeat produces data. The question is how it flows to consumers.

**Delta API**: The primary consumption path. `GET /api/thesis/{id}/changes?since={timestamp}` returns only what changed since the caller's last check. If a node probability shifted, you get the old and new values. If a new signal was ingested, you get the summary. If nothing changed, you get an empty response — 50 bytes. This is designed for agents that poll every few minutes. Minimal bandwidth, minimal processing.

**Telegram push**: For humans. When the evaluation detects a meaningful change (node probability shifted >5%, new edge appeared, kill condition approaching), it pushes a formatted message to a Telegram channel. I have mine set up for my personal trading — I get maybe 3-5 messages per day across all theses, not 96.

**Agent consumption**: An AI agent connected via MCP or REST reads the context snapshot, processes it, and decides whether to act. The agent doesn't need to understand how the heartbeat works — it just reads structured JSON and makes decisions. The `get_context` MCP tool returns exactly the structure shown above.

## Design decisions and tradeoffs

**Why 15 minutes, not 1 minute?** Cost and signal quality. At 1-minute intervals, you're spending $5.86/thesis/day and most cycles produce nothing. News doesn't break every minute. Markets don't reprice every minute. 15 minutes catches 99% of meaningful changes while keeping costs under a dollar.

**Why Tavily instead of scraping RSS?** Reliability and relevance scoring. RSS feeds are noisy — you get every article from a source, not just the ones relevant to your thesis. Tavily lets you query with specific terms and returns ranked results. The 10x cost difference ($0.01/query vs free) is worth it for the signal quality.

**Why not just use a real-time websocket?** I tried this. Kalshi offers websocket feeds for orderbook updates. The problem is volume — on active contracts, you get hundreds of updates per minute. Processing each one through an LLM evaluation is both expensive and pointless. Most orderbook ticks are noise. The 15-minute snapshot captures the net movement without drowning in microstructure.

**Why adversarial search?** Because I've lost money by not doing it. I had a thesis that was "STRONG" for three weeks, with the heartbeat confirming my priors every cycle. The thesis was right on 7 of 8 nodes. The 8th node — which I'd been ignoring because it had low probability — flipped, and the entire downstream chain collapsed. Adversarial search would have caught the early signals on that node.

## Running it yourself

The heartbeat system is what runs behind SimpleFunctions. When you create a thesis through the CLI or MCP, it enters the heartbeat loop automatically. You don't need to manage cron jobs, API keys for each data source, or LLM prompt engineering.

```bash
$ npm install -g @spfunctions/cli
$ sf setup
$ sf thesis create --from-prompt "US-Iran war escalation leads to oil shock and recession"
$ sf heartbeat status

  iran-war  ACTIVE  cycle #4847  last: 3m ago  next: 12m  health: STRONG
```

If you want to build your own, the architecture is straightforward: a cron job (or setInterval in a long-running process) that runs the 7-step cycle. The hard parts are prompt engineering for the evaluation step, handling rate limits across 4+ data sources, and building the delta computation so you're not re-sending the entire state every cycle.

The easy part is deciding to build it. Because if you're trading prediction markets with real money and you're not monitoring 24/7, you're leaving edge on the table every single night.

---

*Patrick builds SimpleFunctions, the thesis engine for prediction market traders. More at [simplefunctions.dev](https://simplefunctions.dev).*