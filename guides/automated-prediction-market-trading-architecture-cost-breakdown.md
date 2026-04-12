# Automated Prediction Market Trading: Architecture and Cost Breakdown

> The real numbers behind running an automated prediction market system. LLM costs per evaluation, Tavily search budgets, Kalshi fees, total cost per thesis, and when each interface (CLI, API, MCP, agent) makes sense.

**Category:** architecture | **Author:** SimpleFunctions | **Reading time:** 10 min

---
Everyone asks "can I build a prediction market bot?" Nobody asks "what will it cost to run?" This article answers the second question with real numbers from running automated thesis monitoring in production.

We'll cover the architecture, break down every cost component, compare building it yourself versus using SimpleFunctions, and explain when each interface (CLI, API, MCP, agent mode) is the right tool.

## Architecture Overview

Here's what a fully automated prediction market system looks like:

```
                        ┌───────────────────────────────────────────┐
                        │              SCHEDULER                    │
                        │  (Vercel Cron / Node cron / systemd)      │
                        │  Triggers heartbeat every 60 min          │
                        └──────────────────┬────────────────────────┘
                                           │
                          ┌─────────────────┴─────────────────┐
                          ▼                                   ▼
                 ┌─────────────────┐               ┌─────────────────┐
                 │  NEWS SERVICE   │               │  PRICE SERVICE  │
                 │  (Tavily API)   │               │  (Kalshi API)   │
                 │                 │               │                 │
                 │  Searches per   │               │  Orderbook      │
                 │  causal node    │               │  snapshots for  │
                 │  ~$0.01/search  │               │  all contracts  │
                 │                 │               │  FREE           │
                 └────────┬────────┘               └────────┬────────┘
                          │                                  │
                          └──────────────┬───────────────────┘
                                         ▼
                              ┌─────────────────────┐
                              │   EVALUATION ENGINE  │
                              │   (LLM via           │
                              │    OpenRouter)        │
                              │                      │
                              │   Node assessment    │
                              │   ~$0.03-0.08/eval   │
                              └──────────┬───────────┘
                                         │
                          ┌──────────────┴──────────────┐
                          ▼                              ▼
                 ┌─────────────────┐          ┌─────────────────┐
                 │  STATE STORE    │          │  NOTIFICATION   │
                 │  (Supabase/     │          │  SERVICE        │
                 │   Postgres)     │          │  (Email/Slack/  │
                 │                 │          │   Webhook)      │
                 │  Thesis state,  │          │                 │
                 │  eval history,  │          │  Fires when     │
                 │  positions      │          │  delta > thresh │
                 └─────────────────┘          └─────────────────┘
```

Five services, three paid APIs (Tavily, OpenRouter, hosting), two free APIs (Kalshi, notification delivery). Let's price each one.

## Cost Component 1: LLM Evaluations

The LLM is the brain of the system. It reads news articles, current market prices, and the causal tree context, then outputs updated confidence levels for each node.

**Per-evaluation cost breakdown:**

```
Input context (typical):
  - Causal tree structure:     ~800 tokens
  - Node history:              ~400 tokens
  - News articles (2-3):       ~1,500 tokens
  - Market data:               ~200 tokens
  - System prompt:             ~500 tokens
  Total input:                 ~3,400 tokens

Output (typical):
  - Updated confidence:        ~300 tokens
  - Evidence citations:        ~200 tokens
  - Uncertainty factors:       ~150 tokens
  Total output:                ~650 tokens
```

**Cost per evaluation by model:**

| Model | Input cost | Output cost | Total per eval |
|-------|:----------:|:-----------:|:--------------:|
| Claude 3.5 Haiku | $0.003 | $0.005 | ~$0.008 |
| Claude 3.5 Sonnet | $0.010 | $0.015 | ~$0.025 |
| GPT-4o mini | $0.002 | $0.003 | ~$0.005 |
| GPT-4o | $0.008 | $0.012 | ~$0.020 |
| Claude Sonnet 4 | $0.010 | $0.015 | ~$0.025 |

We use Claude 3.5 Haiku for routine evaluations (fast, cheap, good enough for "has anything changed?") and Claude Sonnet for significant events (more nuanced reasoning when a node is shifting meaningfully).

**Daily cost per thesis (24 evaluation cycles):**

Not every cycle evaluates every node. Smart scheduling means:

- 24 cycles per day (every 60 minutes)
- Average 3.5 nodes evaluated per cycle (out of ~12 total)
- ~84 node evaluations per day
- At ~$0.05 per evaluation (blended Haiku + occasional Sonnet): **$4.20/day**

**Monthly: ~$126/thesis** at full evaluation frequency. But in practice, many cycles skip evaluation entirely (no new news, no price movement), bringing the actual cost to **$60-90/thesis/month**.

## Cost Component 2: News Search (Tavily)

Tavily provides structured web search results optimized for LLM consumption. Each search returns titles, snippets, URLs, and publication dates.

**Per-search cost:** ~$0.01 on the standard plan.

**Searches per cycle:**

```
Thesis with 12 nodes:
  - 4-6 nodes get dedicated searches per cycle
  - Some searches are batched (related nodes share a query)
  - Typical: 5 Tavily calls per cycle

Per day (24 cycles):    5 * 24 = 120 searches
Per month:              120 * 30 = 3,600 searches

Monthly cost: ~$36/thesis at full frequency
```

**Smart scheduling reduces this significantly.** The system caches news results for 2 hours. If the last search returned nothing relevant, the next cycle skips that node's search. In practice:

```
Actual searches per day: ~40-60 (vs theoretical 120)
Actual monthly cost: ~$12-18/thesis
```

Tavily's $30/month plan includes 3,000 searches. With smart scheduling, that covers 2-3 active theses comfortably.

## Cost Component 3: Kalshi API

Good news: Kalshi API access is free. You can poll orderbooks, check prices, view positions, and place orders without any API fees.

**Trading fees:** Kalshi charges fees on settlement (winning trades), not on entry or exit. The fee schedule is tiered:

```
Monthly volume        Fee per contract (on winning side)
$0 - $999             $0.02 (2¢)
$1,000 - $9,999       $0.015 (1.5¢)
$10,000+              $0.01 (1¢)
```

For a retail trader placing $200-500/month in trades, expect ~2¢ per winning contract in fees. This should be factored into your edge calculation (see the [edge calculation guide](/technicals/edge-calculation-prediction-markets-theory-to-execution)).

## Cost Component 4: Infrastructure

**Option A: Vercel (recommended for simplicity)**

```
Vercel Pro:      $20/month
  - Cron jobs (up to every 1 minute)
  - Serverless functions for evaluation
  - Edge functions for notifications
  - Enough for 5-10 active theses

Supabase Pro:    $25/month
  - Postgres database for thesis state
  - Row-level security
  - Enough storage for years of evaluations
```

**Option B: Self-hosted (cheaper, more work)**

```
VPS (Hetzner/DigitalOcean):  $5-10/month
  - Node.js process with node-cron
  - SQLite for state (no separate DB needed)
  - Manages 20+ theses easily
  - You handle uptime, updates, monitoring
```

**Option C: Local machine (free, least reliable)**

```
Your laptop:     $0/month
  - Runs when your machine is on
  - Stops when you close the lid
  - Good for testing, bad for 24/7 monitoring
  - No costs beyond LLM + Tavily
```

## Total Cost Per Thesis

Here's the full monthly cost for running one thesis with 24/7 monitoring:

```
                         Budget        Standard      Full
                         (min viable)  (recommended)  (max quality)
─────────────────────────────────────────────────────────────
LLM evaluations          $15/mo        $60/mo        $126/mo
  (GPT-4o-mini, skip     (Haiku,       (Sonnet for
   when no change)        smart sched)  all evals)

Tavily searches          $5/mo         $15/mo        $36/mo
  (aggressive caching)   (smart sched) (every cycle)

Infrastructure           $0/mo         $45/mo        $45/mo
  (local machine)        (Vercel+Supa) (Vercel+Supa)

Kalshi fees              ~$2-5/mo      ~$2-5/mo      ~$2-5/mo
  (depends on volume)

TOTAL per thesis         ~$22/mo       ~$122/mo      ~$209/mo
```

The "Standard" tier is what most traders should run: Haiku for routine evaluations with Sonnet for significant events, smart scheduling to reduce unnecessary API calls, and Vercel+Supabase for reliable 24/7 operation.

**For multiple theses**, costs scale sub-linearly:

```
1 thesis:  $122/mo   ($122/thesis)
3 theses:  $230/mo   ($77/thesis)  ← shared infra, cached news
5 theses:  $320/mo   ($64/thesis)  ← more sharing, batched evals
10 theses: $500/mo   ($50/thesis)  ← heavy caching, tiered scheduling
```

The infrastructure costs are fixed (Vercel and Supabase don't change with thesis count), and news searches can be shared across theses with overlapping causal nodes.

## DIY Agent vs. SimpleFunctions

If you're a developer, you might consider building this yourself. Here's an honest comparison:

**Building it yourself:**

```
Development time:     40-80 hours
  - Kalshi API integration:          8 hours
  - Tavily integration:              4 hours
  - LLM evaluation pipeline:         16 hours
  - Causal tree data model:          12 hours
  - Scheduling + monitoring:         8 hours
  - Notification system:             4 hours
  - Dashboard / reporting:           12 hours
  - Testing + debugging:             16 hours

Ongoing maintenance:  2-4 hours/month
  - API changes (Kalshi updates frequently)
  - LLM prompt tuning
  - Bug fixes from edge cases

Total first-year cost:
  Development: 60 hours * $100/hr (your time) = $6,000
  API costs: $122/mo * 12 = $1,464
  Maintenance: 3 hours/mo * $100/hr * 12 = $3,600
  Total: ~$11,064
```

**Using SimpleFunctions CLI:**

```
Setup time:           15 minutes
Ongoing maintenance:  0 hours (updates handled by CLI)

Total first-year cost:
  CLI: free (open source)
  API costs: $122/mo * 12 = $1,464
  Total: ~$1,464
```

The API costs are identical because they're the same APIs. The difference is developer time. If you value your time at $0/hour (side project, learning exercise), build it yourself — you'll learn a lot. If you value your time at any positive number, the math is clear.

## When to Use Each Interface

SimpleFunctions offers four ways to interact: CLI, API, MCP, and agent mode. Each has a sweet spot.

### CLI (`sf` commands)

**Best for:** Individual traders, manual workflow, first-time setup

```bash
sf scan "recession 2026"      # Find opportunities
sf create "thesis statement"   # Build causal tree
sf edges                       # Check edges
sf buy KXTICKER 100 --price 35 # Place orders
sf dashboard                   # Monitor everything
```

**When to use:** You're one person trading your own capital. You want to see everything, control everything, and make every decision yourself. The CLI does the analysis; you do the execution.

**Cost:** Just the API costs ($22-122/mo depending on tier).

### API (REST endpoints)

**Best for:** Developers building custom tools, integrating with existing systems

```bash
curl -X POST https://api.simplefunctions.com/v1/thesis/create \
  -H "Authorization: Bearer sf_..." \
  -d '{"statement": "US recession 2026", "reasoning": "..."}'
```

**When to use:** You want to build a custom dashboard, integrate with your existing trading system, or build automated workflows that trigger based on SimpleFunctions signals.

**Cost:** API costs + your development time.

### MCP (Model Context Protocol)

**Best for:** Using SimpleFunctions inside Claude, Cursor, or any MCP-compatible AI assistant

```
You: "What's the current edge on recession contracts?"
Claude (via MCP): "Your thesis shows 45% vs market 35¢ on KXRECSSNBER-26,
  giving +10¢ theoretical edge. After slippage for 100 contracts, executable
  edge is +8.7¢. Liquidity is HIGH."
```

**When to use:** You want to use natural language to interact with your theses. Ask questions, get answers, adjust nodes, all through conversation. MCP connects SimpleFunctions as a tool that Claude can call directly.

**Cost:** API costs + Claude/Cursor subscription.

### Agent Mode (autonomous monitoring)

**Best for:** 24/7 monitoring with automated signals

```bash
sf agent --thesis recession-2026 --mode monitor
```

**When to use:** You have a thesis and strategy defined, and you want the system to watch the market around the clock. The agent evaluates on schedule, sends notifications when conditions change, and can optionally execute trades when conditions are met.

Agent mode is the most powerful but also the most expensive (highest evaluation frequency). Use it for your highest-conviction theses where missing a signal would cost you real money.

**Cost:** Full API costs ($122/mo per thesis) + infrastructure.

## Scaling: From 1 Thesis to 20

Here's how the architecture evolves as you add theses:

**1-3 theses (hobbyist):**
```
Infrastructure: Vercel free tier + Supabase free tier
Evaluations: Haiku only, skip-when-stable
Total: $30-60/mo
```

**4-10 theses (serious trader):**
```
Infrastructure: Vercel Pro + Supabase Pro
Evaluations: Haiku routine + Sonnet for signals
Shared news cache across theses
Total: $200-400/mo
```

**10-20 theses (fund-level):**
```
Infrastructure: Dedicated VPS + managed Postgres
Evaluations: Tiered (high-conviction theses get Sonnet, others Haiku)
Parallel evaluation pipeline
Shared causal nodes across theses (recession thesis and oil thesis share n1-n2)
Total: $400-800/mo
```

**At 20+ theses**, you're spending $800/mo on monitoring — less than most Bloomberg Terminal users spend on coffee. The question isn't whether the monitoring is worth the cost; it's whether you have enough edge across 20 theses to justify the complexity.

## The Real Cost: Your Attention

The biggest cost of prediction market trading isn't API fees. It's attention. Checking prices, reading news, updating models, second-guessing yourself — this is what consumes your time and energy.

The architecture described here is designed to externalize that cost. Instead of checking five contracts three times a day (45 checks/day, 1,350/month), you get 2-5 notifications when something actually changes. Instead of reading every Iran headline, the system reads them, maps them to causal nodes, and tells you which ones matter.

The financial cost is $60-200/month. The attention savings — getting your evenings, weekends, and 3am back — is the actual value proposition.

Whether you build it yourself or use SimpleFunctions, the architecture is the same. News in, prices in, LLM evaluation, tree propagation, signal out. The question is just how much of your time you want to spend on plumbing versus on thinking about markets.

Spend your time on the thesis. Let the system do the watching.