# How to Build a Prediction Market Trading Bot in 2026

> Binary outcomes, event-driven markets, and settlement dates make prediction market bots fundamentally different from anything you've built for stocks or crypto. Here's how to architect one that actually works.

**Category:** tech | **Author:** SimpleFunctions Research | **Reading time:** 15 min | **Published:** 2026-03-24

---
# How to Build a Prediction Market Trading Bot in 2026

*Binary outcomes, event-driven markets, and settlement dates make prediction market bots fundamentally different from anything you've built for stocks or crypto. Here's how to architect one that actually works.*

---

Prediction markets are not stock markets. If you approach them with the same bot architecture you'd use for equities or crypto, you'll lose money fast. The mechanics are different, the data sources are different, and the failure modes are completely different.

This is a technical guide for TypeScript developers who want to build trading bots for platforms like Polymarket, Kalshi, or Metaculus. We'll cover the architecture from scratch, then walk through three integration paths using SimpleFunctions as the analytical brain — from a CLI-based pipeline to a full autonomous agent.

No filler. No toy examples. Working code you can ship.

## Why Prediction Markets Break Traditional Bot Architecture

If you've built a crypto trading bot, you're used to continuous price signals, technical indicators, and mean-reversion strategies. Prediction markets invalidate most of that playbook. Here's why.

### Binary Outcomes

A prediction market contract resolves to $0 or $1. There is no gradient. The share price at any moment reflects the market's implied probability of an event happening. When you buy a "Yes" share at $0.35, you're betting the true probability is higher than 35%. If you're right and the event occurs, you collect $1. If you're wrong, you get nothing.

This means your entire edge comes from **probability estimation**, not price pattern recognition. Moving averages and RSI are meaningless here. What matters is whether your model of reality is more accurate than the market's.

### Event-Driven, Not Price-Driven

Stock bots react to price movements. Prediction market bots need to react to **events in the real world**. A bill passes committee. A CEO tweets. A jobs report drops. An earthquake hits. These events shift the true probability of an outcome, and your bot needs to detect that shift before the market prices it in.

This means your bot needs to ingest news, parse it, understand causal relationships, and update its probability estimates — all in near real-time.

### Settlement Dates and Illiquidity

Every prediction market contract has a resolution date. Some resolve in hours, some in months. This creates several problems traditional bots don't face:

- **Time decay is nonlinear.** A contract trading at $0.50 with one day left behaves very differently from one at $0.50 with six months left.
- **Liquidity is thin.** You can't always exit a position. Your bot needs to account for the possibility that it's stuck holding until resolution.
- **Correlated markets exist.** If your bot is trading "Will X happen by June?" and "Will X happen by December?", these aren't independent positions. Your architecture needs to model cross-market relationships.

### The Real Architecture You Need

A stock bot's loop is: `poll price → check signal → execute trade → repeat`.

A prediction market bot's loop is fundamentally different:

```
form thesis → build causal model → detect edges → monitor for updates → execute → learn from resolution
```

Every component in that pipeline requires different tooling. Let's build it.

## Architecture: From Thesis to Execution

Here's the full architecture of a prediction market bot that works:

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│  Thesis     │────▶│ Causal Tree  │────▶│ Edge Detection│
│  Formation  │     │ Construction │     │               │
└─────────────┘     └──────────────┘     └───────┬───────┘
                                                 │
                    ┌──────────────┐     ┌───────▼───────┐
                    │  Execution   │◀────│  Monitoring   │
                    │  Engine      │     │  & Heartbeat  │
                    └──────┬───────┘     └───────────────┘
                           │
                    ┌──────▼───────┐
                    │  Track Record│
                    │  & Learning  │
                    └──────────────┘
```

### Thesis Formation

Every trade starts with a thesis: a structured belief about what will happen and why. Not "I think Bitcoin will go up" — that's a stock market thesis. A prediction market thesis looks like:

> "The Federal Reserve will cut rates by at least 25bps at the June 2026 FOMC meeting because core PCE will print below 2.3% for three consecutive months, reducing the Fed's justification for maintaining restrictive policy."

Notice the structure: **outcome** (rate cut), **mechanism** (PCE trend), and **causal chain** (low inflation → policy shift). Your bot needs all three to function.

### Causal Tree Construction

A causal tree maps the dependencies between events that lead to your predicted outcome. For the Fed rate cut thesis:

```
Rate Cut at June FOMC
├── Core PCE < 2.3% (Mar, Apr, May)
│   ├── Shelter inflation cooling
│   ├── Used car prices stabilizing
│   └── Services disinflation continuing
├── Labor market softening
│   ├── NFP < 150k for 2+ months
│   └── Unemployment > 4.2%
├── No external shocks
│   ├── No oil price spike > $100
│   └── No tariff escalation
└── Fed communication shifting dovish
    ├── Dot plot median moves down
    └── Minutes language softens
```

Each node in this tree is something your bot can monitor. When a node's status changes — PCE prints at 2.1%, or oil spikes to $105 — your bot needs to recalculate the probability of the root outcome and compare it to the current market price.

Building these trees by hand is painful. This is where LLMs become genuinely useful — not for trading, but for **structural analysis of causal relationships**.

### Edge Detection

An "edge" exists when your estimated probability diverges meaningfully from the market price. If your causal tree says the rate cut probability is 65% and the market is pricing it at 48%, that's a 17-point edge. Your bot should size the position proportionally to the edge magnitude and your confidence in the model.

The Kelly Criterion gives you a starting framework for position sizing:

```typescript
function kellyFraction(estimatedProb: number, marketPrice: number): number {
  // f* = (p * b - q) / b
  // where b = (1 / marketPrice) - 1, p = estimatedProb, q = 1 - p
  const b = (1 / marketPrice) - 1;
  const f = (estimatedProb * b - (1 - estimatedProb)) / b;
  // Use fractional Kelly (25-50%) to reduce variance
  return Math.max(0, f * 0.25);
}
```

Most practitioners use fractional Kelly (25-50% of the full Kelly bet) to reduce variance. A full Kelly bettor goes bust eventually. A quarter-Kelly bettor survives long enough to learn.

### Monitoring and Heartbeat

This is where most bots die. They form a thesis, place a trade, and then stop paying attention. The market moves against them because the world changed and their model didn't update.

Your bot needs a heartbeat — a periodic process that:

1. Checks every node in every active causal tree
2. Ingests new information (news, data releases, market movements)
3. Recalculates edge estimates
4. Decides whether to add, reduce, or exit positions

The cadence depends on the market's time horizon. A contract resolving tomorrow needs minute-by-minute monitoring. One resolving in six months can check daily.

### Execution

Execution on prediction markets is simpler than on stock exchanges — no HFT, no dark pools, limited order types. The main concerns are:

- **Slippage on thin books.** Always check the order book depth before placing a market order.
- **API rate limits.** Polymarket's CLOB API has rate limits. Batch your operations.
- **Gas costs** (for on-chain markets). Factor transaction costs into your edge calculation.

Now let's build this with real tools.

## Option 1: SimpleFunctions CLI as the Brain

SimpleFunctions handles the hardest parts of the pipeline — thesis formation, causal tree construction, edge detection, and monitoring — so you can focus on execution logic.

### Setup

```bash
npm install -g @spfunctions/cli && sf setup
```

The `sf setup` command walks you through API key configuration and connects to your SimpleFunctions account. It stores credentials locally in `~/.sfconfig`.

### Creating a Thesis

```bash
sf create "The Federal Reserve will cut rates by at least 25bps at the June 2026 FOMC meeting because core PCE will print below 2.3% for three consecutive months"
```

This returns a thesis ID and immediately begins constructing a causal tree. SimpleFunctions uses multiple LLM calls to:

1. Parse the thesis into outcome, mechanism, and timeline
2. Build a causal dependency tree with monitorable nodes
3. Assign initial probability estimates to each node
4. Identify data sources for each node (BLS releases, Fed communications, market data)
5. Calculate an initial edge estimate against current market prices

The output looks like:

```
Thesis created: th_8x2kf9
Causal tree: 4 branches, 12 leaf nodes
Initial estimate: 0.62 (market: 0.48)
Edge detected: +14 points
Monitoring: active (daily heartbeat)
```

### Reading State

To get the current state of a thesis programmatically:

```bash
sf context th_8x2kf9 --json
```

Returns structured data your bot can consume:

```json
{
  "id": "th_8x2kf9",
  "thesis": "Fed rate cut >= 25bps at June 2026 FOMC",
  "estimate": 0.62,
  "confidence": 0.73,
  "market_price": 0.48,
  "edge": 0.14,
  "last_updated": "2026-03-23T14:30:00Z",
  "causal_tree": {
    "root": "rate_cut_june_2026",
    "branches": [
      {
        "node": "core_pce_trend",
        "status": "on_track",
        "probability": 0.71,
        "next_data_point": "2026-03-28T12:30:00Z",
        "source": "BLS PCE Release"
      },
      {
        "node": "labor_market_softening",
        "status": "uncertain",
        "probability": 0.55,
        "next_data_point": "2026-04-04T12:30:00Z",
        "source": "BLS Employment Situation"
      }
    ]
  },
  "signals": [
    {
      "date": "2026-03-21",
      "type": "data_release",
      "description": "Feb core PCE: 2.2% (below 2.3% threshold)",
      "impact": "+0.03 on root estimate"
    }
  ]
}
```

### Pipe-Based Automation

The `sf agent --plain` command outputs machine-readable text, designed for piping into other tools:

```bash
sf agent --plain | while read -r line; do
  # Parse SimpleFunctions output and feed to your execution engine
  echo "$line" >> /var/log/sf-signals.log
done
```

Here's a minimal TypeScript bot that ties it together:

```typescript
import { execSync } from "child_process";

interface ThesisContext {
  id: string;
  estimate: number;
  market_price: number;
  edge: number;
  confidence: number;
  last_updated: string;
}

function getThesisState(thesisId: string): ThesisContext {
  const raw = execSync(`sf context ${thesisId} --json`, {
    encoding: "utf-8",
  });
  return JSON.parse(raw);
}

function shouldTrade(ctx: ThesisContext): {
  action: "buy" | "sell" | "hold";
  size: number;
} {
  const MIN_EDGE = 0.08; // 8 points minimum edge
  const MIN_CONFIDENCE = 0.6;
  const BANKROLL = 1000; // dollars

  if (ctx.confidence < MIN_CONFIDENCE) {
    return { action: "hold", size: 0 };
  }

  if (ctx.edge > MIN_EDGE) {
    const kelly = kellyFraction(ctx.estimate, ctx.market_price);
    return { action: "buy", size: Math.floor(BANKROLL * kelly) };
  }

  if (ctx.edge < -MIN_EDGE) {
    return { action: "sell", size: Math.floor(BANKROLL * 0.5) };
  }

  return { action: "hold", size: 0 };
}

function kellyFraction(prob: number, price: number): number {
  const b = (1 / price) - 1;
  const f = (prob * b - (1 - prob)) / b;
  return Math.max(0, f * 0.25);
}

// Main loop
const THESIS_IDS = ["th_8x2kf9", "th_3m7np2"];
const CHECK_INTERVAL = 60 * 60 * 1000; // hourly

async function run() {
  for (const id of THESIS_IDS) {
    const ctx = getThesisState(id);
    const decision = shouldTrade(ctx);

    if (decision.action !== "hold") {
      console.log(
        `[${id}] ${decision.action} $${decision.size} ` +
        `(edge: ${(ctx.edge * 100).toFixed(1)}%, ` +
        `confidence: ${(ctx.confidence * 100).toFixed(0)}%)`
      );
      // Execute via Polymarket CLOB API, Kalshi API, etc.
      // await executeOrder(id, decision);
    }
  }
}

setInterval(run, CHECK_INTERVAL);
run();
```

This is intentionally minimal. The key insight is that SimpleFunctions handles the hard analytical work (thesis → causal tree → edge estimation → monitoring), and your bot handles the mechanical work (position sizing → order execution → risk management).

## Option 2: MCP Server for Claude Code / Cursor Agents

If you're building inside Claude Code or Cursor, you can connect SimpleFunctions as an MCP (Model Context Protocol) server. This gives your AI coding agent direct access to prediction market analysis tools.

### Setup

One command:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

This registers SimpleFunctions as an MCP server with 18 tools available to your agent. No API keys to manage in your code — the MCP protocol handles authentication.

### What This Enables

Once connected, your Claude Code or Cursor agent can:

- Create and analyze theses in natural language
- Query causal trees and edge estimates mid-conversation
- Feed prediction market context into code generation
- Build monitoring dashboards with live data

The MCP approach is best for **interactive development** — when you're exploring markets, testing thesis ideas, and building tooling iteratively. You talk to your agent, it calls SimpleFunctions tools behind the scenes, and you get structured analysis back in your conversation.

For example, you could tell your agent:

> "Create a thesis about whether the EU AI Act enforcement actions will exceed 5 cases by December 2026, then build me a monitoring script that checks the edge daily."

Your agent would use the MCP tools to create the thesis, retrieve the causal tree, and generate a monitoring script wired to the SimpleFunctions API — all in one conversation.

### Available Tools

The 18 MCP tools cover the full lifecycle:

- **Thesis management:** create, read, update, list, delete
- **Analysis:** get causal tree, get edge estimates, evaluate market pricing
- **Monitoring:** check signals, get heartbeat status, trigger manual refresh
- **Track record:** get historical accuracy, compare to market benchmarks
- **Context:** retrieve full thesis state with all branches and signals

Each tool accepts structured input and returns JSON, so your agent can chain them together programmatically.

## Option 3: REST API for Custom Bots

If you're building a standalone system — a long-running service, a Lambda function, a bot that runs on a schedule — the REST API gives you full control.

### Creating a Thesis

```bash
curl -X POST https://simplefunctions.dev/api/thesis \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "thesis": "SpaceX Starship will complete a successful orbital flight and landing by September 2026",
    "market_url": "https://polymarket.com/event/starship-orbital-2026",
    "monitoring": "daily"
  }'
```

Response:

```json
{
  "id": "th_9k4mw1",
  "status": "processing",
  "estimated_completion": "45s"
}
```

Thesis creation is async — the causal tree construction takes 30-90 seconds depending on complexity. Poll for completion or use a webhook.

### Getting Full Context

```bash
curl https://simplefunctions.dev/api/thesis/th_9k4mw1/context \
  -H "Authorization: Bearer $SF_API_KEY"
```

This returns the same JSON structure shown in the CLI section — thesis state, causal tree, edge estimates, recent signals.

### Submitting External Signals

Your bot might detect relevant events before SimpleFunctions' monitoring catches them. You can push signals manually:

```bash
curl -X POST https://simplefunctions.dev/api/thesis/th_9k4mw1/signal \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "signal": "SpaceX announced Starship Flight 8 scrubbed due to engine valve issue. New launch window TBD.",
    "source": "https://twitter.com/SpaceX/status/...",
    "timestamp": "2026-03-23T18:00:00Z"
  }'
```

SimpleFunctions ingests the signal, updates the relevant causal tree nodes, and recalculates the edge estimate. The next time your bot polls for context, it gets the updated numbers.

### Evaluating a Position

Before executing a trade, get a structured evaluation:

```bash
curl -X POST https://simplefunctions.dev/api/thesis/th_9k4mw1/evaluate \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "market_price": 0.32,
    "position_size": 500,
    "direction": "yes"
  }'
```

Response:

```json
{
  "thesis_id": "th_9k4mw1",
  "our_estimate": 0.45,
  "market_price": 0.32,
  "edge": 0.13,
  "confidence": 0.68,
  "kelly_fraction": 0.11,
  "recommended_size": 110,
  "risk_notes": [
    "Timeline risk: 6 months remaining, multiple launch attempts possible",
    "Correlated positions detected: check portfolio for SpaceX exposure",
    "Liquidity warning: order book thin above $200"
  ]
}
```

### Full TypeScript Bot with REST API

Here's a more complete bot using the REST API directly:

```typescript
const SF_BASE = "https://simplefunctions.dev/api";
const SF_KEY = process.env.SF_API_KEY!;

interface Signal {
  date: string;
  type: string;
  description: string;
  impact: number;
}

interface ThesisState {
  id: string;
  estimate: number;
  confidence: number;
  market_price: number;
  edge: number;
  signals: Signal[];
  last_updated: string;
}

interface Evaluation {
  edge: number;
  confidence: number;
  kelly_fraction: number;
  recommended_size: number;
  risk_notes: string[];
}

async function sfFetch<T>(
  path: string,
  options?: RequestInit
): Promise<T> {
  const res = await fetch(`${SF_BASE}${path}`, {
    ...options,
    headers: {
      Authorization: `Bearer ${SF_KEY}`,
      "Content-Type": "application/json",
      ...options?.headers,
    },
  });
  if (!res.ok) throw new Error(`SF API: ${res.status} ${await res.text()}`);
  return res.json();
}

async function getContext(thesisId: string): Promise<ThesisState> {
  return sfFetch<ThesisState>(`/thesis/${thesisId}/context`);
}

async function submitSignal(
  thesisId: string,
  signal: string,
  source: string
): Promise<void> {
  await sfFetch(`/thesis/${thesisId}/signal`, {
    method: "POST",
    body: JSON.stringify({
      signal,
      source,
      timestamp: new Date().toISOString(),
    }),
  });
}

async function evaluate(
  thesisId: string,
  marketPrice: number,
  positionSize: number
): Promise<Evaluation> {
  return sfFetch<Evaluation>(`/thesis/${thesisId}/evaluate`, {
    method: "POST",
    body: JSON.stringify({
      market_price: marketPrice,
      position_size: positionSize,
      direction: "yes",
    }),
  });
}

// --- Execution engine (Polymarket example) ---

interface TradeDecision {
  action: "buy_yes" | "buy_no" | "sell" | "hold";
  size: number;
  reason: string;
}

function decide(
  state: ThesisState,
  evaluation: Evaluation
): TradeDecision {
  const MIN_EDGE = 0.08;
  const MIN_CONFIDENCE = 0.55;
  const MAX_POSITION = 500;

  if (evaluation.confidence < MIN_CONFIDENCE) {
    return {
      action: "hold",
      size: 0,
      reason: `Confidence too low: ${(evaluation.confidence * 100).toFixed(0)}%`,
    };
  }

  if (evaluation.edge > MIN_EDGE) {
    const size = Math.min(evaluation.recommended_size, MAX_POSITION);
    return {
      action: "buy_yes",
      size,
      reason: `Edge: +${(evaluation.edge * 100).toFixed(1)}pts, ` +
              `Kelly size: $${size}`,
    };
  }

  if (evaluation.edge < -MIN_EDGE) {
    const size = Math.min(
      Math.abs(evaluation.recommended_size),
      MAX_POSITION
    );
    return {
      action: "buy_no",
      size,
      reason: `Negative edge: ${(evaluation.edge * 100).toFixed(1)}pts, ` +
              `counter-trade $${size}`,
    };
  }

  return {
    action: "hold",
    size: 0,
    reason: `Edge too small: ${(evaluation.edge * 100).toFixed(1)}pts`,
  };
}

// --- Main loop ---

interface ActiveThesis {
  id: string;
  market_url: string;
  current_market_price: () => Promise<number>;
}

async function runCycle(theses: ActiveThesis[]) {
  const timestamp = new Date().toISOString();
  console.log(`\n[${timestamp}] Running cycle for ${theses.length} theses`);

  for (const thesis of theses) {
    try {
      const state = await getContext(thesis.id);
      const marketPrice = await thesis.current_market_price();
      const evaluation = await evaluate(thesis.id, marketPrice, 500);
      const decision = decide(state, evaluation);

      console.log(
        `  [${thesis.id}] ${decision.action} — ${decision.reason}`
      );

      if (decision.action !== "hold") {
        // Wire this to your exchange's API
        // await polymarketExecute(thesis.market_url, decision);
      }

      // Log for track record
      if (evaluation.risk_notes.length > 0) {
        console.log(`    Risks: ${evaluation.risk_notes.join("; ")}`);
      }
    } catch (err) {
      console.error(`  [${thesis.id}] Error: ${err}`);
    }
  }
}
```

## The Monitoring Problem

Most prediction market bots fail not because their initial analysis was wrong, but because they **stop updating**. The world changes. New information arrives. The causal tree shifts. And the bot sits there holding a stale position based on a thesis that's no longer valid.

This is the single biggest failure mode in prediction market automation.

### Why Monitoring Is Hard

Stock bots monitor one thing: price. Prediction market bots need to monitor **the entire causal chain that supports their thesis**. For a single thesis about Fed rate cuts, that might mean tracking:

- Monthly PCE releases from BLS
- Weekly jobless claims
- Fed governor speeches and interviews
- FOMC minutes language
- Oil prices (for inflation expectations)
- Geopolitical events that could trigger supply shocks
- Other central bank decisions that might influence Fed thinking

Each of these arrives on a different schedule, from a different source, in a different format. Building a reliable monitoring pipeline for even one thesis is a significant engineering effort. Doing it for a portfolio of 20 theses is a full-time job.

### How SimpleFunctions Solves This

SimpleFunctions runs a heartbeat process for every active thesis. The heartbeat:

1. **Checks all data sources** mapped to causal tree nodes on their respective schedules
2. **Ingests new information** from news feeds, data releases, and market movements
3. **Updates node probabilities** when new evidence arrives
4. **Recalculates the root estimate** and edge
5. **Emits signals** when material changes occur

You don't need to build any of this. Your bot just polls `sf context <id> --json` on whatever cadence makes sense for your strategy, and it gets the latest state.

If you want push notifications instead of polling, you can configure webhooks:

```bash
curl -X POST https://simplefunctions.dev/api/thesis/th_9k4mw1/webhook \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-bot.example.com/signals",
    "events": ["edge_change", "signal_detected", "confidence_shift"],
    "min_edge_change": 0.05
  }'
```

This fires a webhook to your bot whenever the edge estimate changes by more than 5 points, a new signal is detected, or confidence shifts materially. Your bot can sleep until something actually matters.

### Stale Thesis Detection

SimpleFunctions also flags theses that haven't received new information in a while. If a thesis's causal tree nodes haven't been updated and no new signals have been detected, the system reduces confidence automatically and can alert you:

```json
{
  "id": "th_3m7np2",
  "staleness_warning": true,
  "days_since_update": 14,
  "confidence_decay": 0.85,
  "recommendation": "Review thesis — no new evidence in 14 days, confidence decayed to 0.62"
}
```

This prevents the classic failure mode where a bot holds a position based on a thesis that's become irrelevant because nobody was watching.

## The Track Record Feedback Loop

Prediction markets have a massive advantage over stock markets for automated trading: **every bet resolves to a known truth value**. The contract either pays out or it doesn't. This gives you a clean, unambiguous signal for measuring your bot's accuracy.

SimpleFunctions tracks your thesis accuracy over time and uses it to calibrate future estimates:

```bash
sf track-record --json
```

```json
{
  "total_theses": 47,
  "resolved": 31,
  "accuracy": {
    "brier_score": 0.18,
    "calibration": {
      "bucket_60_70": { "predicted": 0.65, "actual": 0.62, "n": 8 },
      "bucket_70_80": { "predicted": 0.74, "actual": 0.71, "n": 6 },
      "bucket_80_90": { "predicted": 0.84, "actual": 0.80, "n": 5 }
    },
    "log_loss": 0.42,
    "roi": 0.23
  },
  "patterns": [
    "Overconfident on technology timelines by ~8%",
    "Well-calibrated on political outcomes",
    "Underestimates tail risks in geopolitical theses"
  ]
}
```

A Brier score of 0.18 is solid — below 0.25 means you're beating naive baselines, and below 0.15 means you're genuinely good at this. The calibration breakdown shows you where your model is over- or under-confident.

The `patterns` field is particularly valuable. SimpleFunctions analyzes your resolved theses and identifies systematic biases. If you're consistently overconfident on technology timelines, your bot can apply a correction factor to future tech-related theses.

### Feeding Track Record Back Into Trading

Your bot can use the track record to adjust its behavior:

```typescript
interface TrackRecord {
  brier_score: number;
  calibration: Record<string, { predicted: number; actual: number; n: number }>;
  patterns: string[];
}

function adjustForCalibration(
  rawEstimate: number,
  trackRecord: TrackRecord
): number {
  // Find the calibration bucket this estimate falls into
  const bucket = Math.floor(rawEstimate * 10) / 10;
  const bucketKey = `bucket_${bucket * 100}_${(bucket + 0.1) * 100}`;
  const cal = trackRecord.calibration[bucketKey];

  if (!cal || cal.n < 5) {
    // Not enough data in this bucket — use raw estimate
    return rawEstimate;
  }

  // Apply calibration correction
  const bias = cal.predicted - cal.actual;
  return Math.max(0.01, Math.min(0.99, rawEstimate - bias));
}
```

Over time, this creates a self-improving system. Your bot makes predictions, the market resolves them, you learn where you're wrong, and the next round of predictions gets better. This feedback loop is what separates profitable prediction market bots from expensive experiments.

## Cost Breakdown

Let's talk real numbers. Running a prediction market bot is not free, and you should know what you're spending before you start.

### SimpleFunctions Costs

SimpleFunctions pricing is usage-based. Here's what a typical thesis costs:

| Operation | Cost | Frequency |
|-----------|------|-----------|
| Thesis creation (causal tree build) | ~$0.15–0.40 in LLM tokens | Once |
| Heartbeat check (monitoring) | ~$0.02–0.08 per check | Daily |
| Signal processing | ~$0.03–0.10 per signal | As needed |
| Context retrieval | Free (cached) | Unlimited |
| Evaluation | ~$0.05–0.15 | Per request |

### Per-Thesis Monthly Cost

For a single thesis with daily monitoring:

- Creation: $0.25 (one-time, amortized over thesis lifetime)
- Daily heartbeat: $0.05 × 30 = $1.50
- Signal processing: ~5 signals/month × $0.06 = $0.30
- Evaluations: ~10/month × $0.10 = $1.00
- **Total: ~$3.00–3.50/month per thesis**

### Portfolio Cost

A portfolio of 15 active theses:

- SimpleFunctions: 15 × $3.25 = ~$49/month
- Your compute (a small VPS for the bot): $5–20/month
- Exchange API costs: Usually free
- **Total: ~$55–70/month**

### Break-Even Analysis

At $65/month in operating costs, you need to generate $780/year in profit to break even. With a $1,000 bankroll and 23% ROI (the track record example above), you'd generate $230/year — not enough.

You need either:

- A larger bankroll ($3,500+ to break even at 23% ROI)
- Better accuracy (higher ROI)
- Fewer theses with higher conviction (lower costs)

The math is straightforward: don't run 15 low-conviction theses. Run 5 high-conviction ones with larger position sizes. Your operating cost drops to ~$35/month, your capital is concentrated where your edge is strongest, and the break-even bankroll drops to ~$1,800.

### Scaling Costs

Once your bot is profitable, scaling is cheap. Going from 5 to 50 theses increases SimpleFunctions costs by roughly $150/month. If each additional thesis generates even small positive expected value, the marginal economics are very favorable.

## Putting It All Together

Here's the complete flow for a production prediction market bot:

**Day 1: Setup**
```bash
# Install CLI
npm install -g @spfunctions/cli && sf setup

# Or connect via MCP for Claude Code
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp

# Create your first thesis
sf create "EU will impose >€10M in AI Act fines by December 2026 due to accelerating enforcement posture following initial violation findings in Q2"
```

**Day 2: Build the execution layer**
- Wire up your exchange API (Polymarket CLOB, Kalshi, etc.)
- Implement position sizing with fractional Kelly
- Set up logging and alerting

**Day 3: Deploy**
- Run the bot on a schedule (cron, PM2, or a simple systemd service)
- Configure webhooks for real-time signal processing
- Set maximum position sizes and drawdown limits

**Ongoing: Monitor and learn**
- Review track record weekly
- Kill theses where confidence has decayed below your threshold
- Add new theses as you spot opportunities
- Adjust calibration based on resolved outcomes

The technical barrier to prediction market trading has dropped dramatically. The analytical work that used to require a team of researchers — building causal models, monitoring dozens of data sources, calibrating probability estimates — can now be handled by an LLM-powered system. Your job as a developer is to build the mechanical layer: execution, risk management, and position sizing.

The bot that wins isn't the fastest or the most sophisticated. It's the one that **updates its beliefs when the world changes** and **sizes its bets according to its actual edge**. Everything else is plumbing.

---

*SimpleFunctions provides the analytical infrastructure for prediction market bots. Start with `npm install -g @spfunctions/cli` or connect via MCP at `https://simplefunctions.dev/api/mcp/mcp`.*