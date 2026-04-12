# Running a 24/7 Trading Agent: Architecture, Costs, and What to Watch

> The real operational picture. Heartbeat cron jobs, Tavily news search costs, OpenRouter LLM spend, Kalshi API quirks, and why this whole system runs for ~$100/month vs. a quant fund's $50K/month data bill.

**Category:** architecture | **Author:** Patrick Liu | **Reading time:** 1 min

---
This is the post nobody writes: what it actually costs and looks like to run a 24/7 prediction market agent in production. Not the theory. Not the architecture diagram. The real numbers, the real bugs, and the real operational lessons.

## Architecture Overview

The system has four moving parts:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Vercel Cron     │────▶│  Monitor        │────▶│  Tavily         │
│  (every 15 min)  │     │  Service        │     │  (news search)  │
└─────────────────┘     └───────┬─────────┘     └─────────────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
            ┌───────────┐ ┌─────────┐ ┌──────────┐
            │ OpenRouter │ │ Kalshi  │ │ Supabase │
            │ (LLM eval)│ │ (prices)│ │ (state)  │
            └───────────┘ └─────────┘ └──────────┘
```

**Vercel Cron** triggers the heartbeat every 15 minutes. The monitor service fetches prices, searches for news, evaluates the causal tree, checks strategy conditions, and writes results to the database. If a signal fires, it sends a notification.

## Real Monthly Costs

Here's what we actually pay running 3 active theses with continuous monitoring:

| Service | What It Does | Monthly Cost |
|---------|-------------|:------------:|
| Vercel Pro | Hosting + cron jobs (96/day) | $20 |
| Supabase Pro | Postgres database + auth | $25 |
| Tavily | News search API (~2,000 searches/mo) | $30 |
| OpenRouter | LLM calls (~500 evaluations/mo) | $15-25 |
| Kalshi API | Market data + orderbook | Free |
| **Total** | | **~$90-100/mo** |

For context: a quantitative trading fund pays $50,000-200,000/month for data feeds alone. Bloomberg Terminal is $24,000/year per seat. We're running a causal model + 24/7 monitoring + real-time edge detection for the price of a nice dinner.

### Cost Breakdown Details

**Tavily ($30/month):** Each heartbeat triggers 1-3 Tavily searches per active thesis. With 3 theses and 96 heartbeats per day: ~288-864 searches/day, ~8,640-25,920/month. The $30 plan gives us enough headroom. The searches are targeted at causal tree nodes, not broad market news — this keeps them focused and cheap.

**OpenRouter ($15-25/month):** Each evaluation sends the causal tree + news context + market data to an LLM for node confidence updates. We use Claude Haiku for routine evaluations (fast, cheap) and Claude Sonnet for significant events (more accurate). Average cost per evaluation: ~3-5¢. With ~500 evaluations/month: $15-25.

**Kalshi API (free):** Kalshi provides free API access for market data and orderbook. No websocket costs, no data fees. This is a massive advantage over traditional markets where data feeds cost thousands.

## Operational Lessons (Things That Broke)

### Lesson 1: Tavily Rate Limits Are Real

We hit Tavily rate limits twice in the first month. The heartbeat was running searches too aggressively — every 15 minutes for every node. The fix: batch node searches and cache results for 2 hours unless a significant price movement triggers a re-search.

Now the cadence is:
- **Every 15 min:** Price check only (free, Kalshi API)
- **Every 2 hours:** Full news search + LLM evaluation
- **On significant price move (>3%):** Immediate full evaluation regardless of schedule

This cut our Tavily usage by 60% without meaningfully reducing signal quality.

### Lesson 2: Kalshi Price Fields Can Be Null

We discovered this the hard way: the Kalshi API sometimes returns `null` for the `yes_price` field on low-liquidity contracts. Our evaluation code assumed a number and crashed.

The fix was simple — default to the last known price and flag the contract as "stale data" — but it took down monitoring for 6 hours before we caught it. Now we have explicit null checks on every price field.

### Lesson 3: LLM Hallucination in Soft Conditions

The LLM evaluates "soft conditions" — qualitative assessments like "has there been a significant diplomatic development?" Sometimes it hallucinates. Once it claimed there was a ceasefire announcement when there wasn't — based on a headline about "diplomatic channels remaining open" which it over-interpreted.

The fix: every soft condition evaluation now includes a verification step. The LLM sees the raw news text and must cite the specific sentence that supports its conclusion. If it can't cite, the condition isn't triggered. This reduced false positives from ~8% to <1%.

### Lesson 4: Vercel Cron Cold Starts

Vercel serverless functions have cold starts. The first heartbeat after an idle period takes 3-5 seconds to initialize the database connection. This isn't a correctness issue — it's a latency issue. If a major news event breaks and the cron triggers during a cold start, you lose a few seconds.

The practical impact: negligible. Prediction markets move slowly. A 5-second delay on a 15-minute heartbeat cycle doesn't matter. But it's worth knowing.

### Lesson 5: Signature Issues with Kalshi API

Kalshi uses HMAC-SHA256 signature authentication. The timestamp must be within 5 seconds of server time. On Vercel serverless functions, the system clock is occasionally a few seconds off after a cold start.

We added NTP clock checking and a retry with fresh timestamp on 401 errors. Happens maybe once a week.

## What to Monitor

If you're running this system, here's your dashboard:

### 1. Tavily Budget Consumption

Track daily search count vs. plan limit. Set an alert at 80% of monthly limit. If you're burning through searches, your evaluation cadence is too aggressive.

### 2. LLM Evaluation Cost Per Thesis

Track the average cost per evaluation and compare across theses. If one thesis is significantly more expensive, it probably has too many causal nodes or the context window is too large. Consider pruning.

### 3. Signal-to-Noise Ratio

Track how many heartbeats result in an actual signal vs. "no action needed." A healthy ratio is 1-5% — you want the agent watching quietly 95%+ of the time and only signaling when something actually changes. If you're getting signals on >10% of heartbeats, your conditions are too loose.

### 4. Price Staleness

Track the age of the latest price for each monitored contract. If a price is more than 30 minutes old, something is wrong with the Kalshi API connection. Alert on this.

### 5. Causal Tree Stability

Track how much each node's confidence changes per evaluation cycle. Nodes should be relatively stable (±2-3%) with occasional jumps when real news hits. If a node is swinging ±10% on every evaluation, the LLM is being inconsistent — consider pinning that node or using a more deterministic evaluation.

## Scaling Considerations

The current architecture handles 3-5 active theses comfortably. At 10+ theses, you'd want to:

1. **Parallelize evaluations** — evaluate theses concurrently instead of sequentially
2. **Shared news cache** — multiple theses often share upstream nodes; cache news searches across theses
3. **Tiered evaluation** — not every thesis needs evaluation every 2 hours; low-conviction theses can run on 6-hour cycles

At 50+ theses, you'd probably want to move off Vercel cron to a dedicated worker process. But honestly, if you have 50 active theses, you have bigger problems than infrastructure.

## The Bottom Line

Running a 24/7 prediction market agent costs ~$100/month and takes about 2 hours to set up. The ongoing operational work is minimal — mostly watching for the anomalies described above and occasionally adjusting evaluation cadence.

The value isn't in the infrastructure. It's in the *discipline*. The agent watches when you sleep. It evaluates without emotion. It signals based on conditions, not feelings. For the price of a streaming subscription, you have a system that a quantitative fund would charge 2-and-20 to provide.

The only expensive component is your thesis. The causal model. The thinking. And that's exactly where your time should go — not staring at price charts at 3am.