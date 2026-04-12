# Heartbeat architecture: how to monitor 50+ prediction markets in real-time

> Inside the 10-step monitoring loop that watches Kalshi, Polymarket, and traditional markets on a 15-minute cycle for $0.61/thesis/day

**Category:** architecture | **Author:** Patrick Liu | **Reading time:** 12 min

---
# Heartbeat architecture: how to monitor 50+ prediction markets in real-time

We run ~50 active theses at any time across Kalshi and Polymarket. Each thesis has a position, an entry price, a set of kill conditions, and a probability estimate we believe the market is mispricing. The job of the heartbeat is simple: every 15 minutes, decide if anything has changed enough to notify us.

This article walks through the full 10-step monitoring loop, the design decisions behind it, and the actual JSON flowing through each stage.

## System overview

The heartbeat is a single async function that runs on a cron schedule. One invocation processes one thesis. With 50 active theses and a 15-minute cycle, we process ~200 thesis evaluations per hour.

```typescript
// cron: */15 * * * *
export async function heartbeat(thesisId: string): Promise<HeartbeatResult> {
  const thesis = await db.select().from(theses).where(eq(theses.id, thesisId)).limit(1);
  const pipeline = [
    newsScan,        // Step 1
    priceRescan,     // Step 2
    tradMarkets,     // Step 3
    socialSignals,   // Step 4
    milestoneCheck,  // Step 5
    killCondCheck,   // Step 6
    signalAgg,       // Step 7
    llmEval,         // Step 8
    deltaCalc,       // Step 9
    notify,          // Step 10
  ];
  return runPipeline(thesis, pipeline);
}
```

Each step receives the accumulating `HeartbeatContext` and appends its signals. Let's walk through each one.

---

## Step 1: News scan (Tavily)

We use Tavily's search API to find recent news relevant to the thesis. The query is constructed from the thesis title plus key entities extracted at thesis creation time.

```typescript
async function newsScan(ctx: HeartbeatContext): Promise<void> {
  const query = `${ctx.thesis.title} ${ctx.thesis.entities.join(' ')}`;
  const results = await tavily.search({
    query,
    search_depth: 'advanced',
    max_results: 5,
    include_answer: true,
    topic: 'news',
    days: 1,  // only last 24 hours
  });
  ctx.signals.push({
    source: 'tavily_news',
    timestamp: new Date().toISOString(),
    data: {
      answer: results.answer,
      articles: results.results.map(r => ({
        title: r.title,
        url: r.url,
        snippet: r.content.slice(0, 300),
        score: r.score,
      })),
    },
  });
}
```

Output JSON at this stage:

```json
{
  "source": "tavily_news",
  "timestamp": "2026-03-28T14:15:00Z",
  "data": {
    "answer": "The Federal Reserve held rates steady at 4.25-4.50%...",
    "articles": [
      {
        "title": "Fed holds rates, signals June cut unlikely",
        "url": "https://reuters.com/...",
        "snippet": "Federal Reserve officials voted unanimously to hold...",
        "score": 0.94
      }
    ]
  }
}
```

Why Tavily over a raw Google search? Tavily returns relevance scores and a synthesized answer. The answer field alone saves us from needing a separate summarization step. Cost: ~$0.01 per search.

---

## Step 2: Price rescan (Kalshi + Polymarket)

We fetch the current market price from whichever venue(s) the thesis tracks.

```typescript
async function priceRescan(ctx: HeartbeatContext): Promise<void> {
  const prices: PricePoint[] = [];

  if (ctx.thesis.kalshiTicker) {
    const kalshi = await kalshiClient.getMarket(ctx.thesis.kalshiTicker);
    prices.push({
      venue: 'kalshi',
      ticker: ctx.thesis.kalshiTicker,
      yesPrice: kalshi.yes_price,
      noPrice: kalshi.no_price,
      volume24h: kalshi.volume_24h,
      spread: kalshi.yes_price + kalshi.no_price > 100
        ? kalshi.yes_price + kalshi.no_price - 100 : 0,
    });
  }

  if (ctx.thesis.polymarketConditionId) {
    const pm = await polyClient.getMarket(ctx.thesis.polymarketConditionId);
    prices.push({
      venue: 'polymarket',
      ticker: ctx.thesis.polymarketConditionId,
      yesPrice: Math.round(pm.outcomePrices[0] * 100),
      noPrice: Math.round(pm.outcomePrices[1] * 100),
      volume24h: pm.volume24hr,
      spread: Math.abs(pm.outcomePrices[0] + pm.outcomePrices[1] - 1) * 100,
    });
  }

  ctx.signals.push({
    source: 'price_rescan',
    timestamp: new Date().toISOString(),
    data: { prices, previousPrice: ctx.thesis.lastPrice },
  });
}
```

Output JSON:

```json
{
  "source": "price_rescan",
  "timestamp": "2026-03-28T14:15:01Z",
  "data": {
    "prices": [
      {
        "venue": "kalshi",
        "ticker": "FED-RATE-25JUN",
        "yesPrice": 32,
        "noPrice": 70,
        "volume24h": 45200,
        "spread": 2
      }
    ],
    "previousPrice": 28
  }
}
```

---

## Step 3: Traditional markets (Databento)

This is the step most prediction market monitors skip. We pull relevant traditional market data because prediction markets don't exist in a vacuum. A thesis about a Fed rate cut correlates with Treasury yields. A thesis about oil prices correlates with CL futures.

At thesis creation time, we map each thesis to 1-3 traditional market instruments.

```typescript
async function tradMarkets(ctx: HeartbeatContext): Promise<void> {
  if (!ctx.thesis.tradMarketSymbols?.length) return;

  const snapshots = await Promise.all(
    ctx.thesis.tradMarketSymbols.map(async (sym) => {
      const bar = await databento.getLatestBar(sym);
      return {
        symbol: sym,
        last: bar.close,
        change1d: bar.change_1d_pct,
        change5d: bar.change_5d_pct,
        volume: bar.volume,
      };
    })
  );

  ctx.signals.push({
    source: 'trad_markets',
    timestamp: new Date().toISOString(),
    data: { snapshots },
  });
}
```

Why include traditional markets? Because they move first. When 10Y Treasury yields spike 15bps in an hour, the Kalshi Fed rate market might not reprice for another 30 minutes. Traditional markets give us a leading indicator for prediction market price discovery lag.

---

## Step 4: X/Twitter signals

We scan X for high-signal posts from curated lists of domain experts. Not firehose -- targeted accounts.

```typescript
async function socialSignals(ctx: HeartbeatContext): Promise<void> {
  const query = ctx.thesis.socialQuery; // pre-built at thesis creation
  const tweets = await xClient.search({
    query,
    max_results: 10,
    recency: 'recent',
    sort_order: 'relevancy',
  });

  ctx.signals.push({
    source: 'x_social',
    timestamp: new Date().toISOString(),
    data: {
      tweets: tweets.map(t => ({
        author: t.author_username,
        text: t.text.slice(0, 280),
        likes: t.public_metrics.like_count,
        retweets: t.public_metrics.retweet_count,
        timestamp: t.created_at,
      })),
    },
  });
}
```

Output JSON:

```json
{
  "source": "x_social",
  "timestamp": "2026-03-28T14:15:02Z",
  "data": {
    "tweets": [
      {
        "author": "NickTimiraos",
        "text": "Fed's Waller says June cut is 'not off the table' if labor market weakens further...",
        "likes": 2340,
        "retweets": 812,
        "timestamp": "2026-03-28T13:42:00Z"
      }
    ]
  }
}
```

---

## Step 5: Milestone check

Each thesis has milestones -- specific events that would confirm or invalidate the thesis. The milestone check compares current signals against these predefined checkpoints.

```typescript
async function milestoneCheck(ctx: HeartbeatContext): Promise<void> {
  const milestones = ctx.thesis.milestones; // e.g., [{event: "CPI print < 2.5%", hit: false}]
  const newsText = ctx.signals
    .filter(s => s.source === 'tavily_news')
    .map(s => s.data.answer)
    .join(' ');

  const hit = milestones.filter(m => !m.hit).map(m => ({
    ...m,
    possibleHit: newsText.toLowerCase().includes(m.keyword),
  }));

  ctx.signals.push({
    source: 'milestone_check',
    timestamp: new Date().toISOString(),
    data: { milestones: hit },
  });
}
```

---

## Step 6: Kill condition check

Kill conditions are the inverse of milestones -- they define when we should exit a position unconditionally. These are set at thesis creation and never overridden by the LLM evaluation.

```typescript
async function killCondCheck(ctx: HeartbeatContext): Promise<void> {
  const kills = ctx.thesis.killConditions;
  // e.g., [{condition: "price > 85", type: "price"}, {condition: "resolution date passed", type: "time"}]

  const currentPrice = ctx.signals
    .find(s => s.source === 'price_rescan')
    ?.data.prices[0]?.yesPrice;

  const triggered = kills.map(k => ({
    ...k,
    triggered: evaluateKillCondition(k, { currentPrice, now: new Date() }),
  }));

  ctx.signals.push({
    source: 'kill_condition',
    timestamp: new Date().toISOString(),
    data: { conditions: triggered },
  });

  // If any kill condition fires, short-circuit the pipeline
  if (triggered.some(t => t.triggered)) {
    ctx.killTriggered = true;
  }
}
```

Output when a kill fires:

```json
{
  "source": "kill_condition",
  "timestamp": "2026-03-28T14:15:03Z",
  "data": {
    "conditions": [
      {
        "condition": "price > 85",
        "type": "price",
        "triggered": true
      },
      {
        "condition": "resolution date passed",
        "type": "time",
        "triggered": false
      }
    ]
  }
}
```

---

## Step 7: Signal aggregation

This is where deduplication happens. Multiple sources often report the same event. A Reuters article about the Fed appears in Tavily results, gets quoted on X, and moves Treasury yields. Without dedup, the LLM sees three signals that are actually one.

```typescript
async function signalAgg(ctx: HeartbeatContext): Promise<void> {
  const allSignals = ctx.signals;

  // Extract text content from all signals for embedding
  const textChunks = allSignals.flatMap(extractTextChunks);

  // Cosine similarity dedup: if two chunks > 0.85 similarity, merge them
  const deduped = await deduplicateByEmbedding(textChunks, {
    threshold: 0.85,
    model: 'text-embedding-3-small',
  });

  // Classify signal strength
  const classified = deduped.map(chunk => ({
    ...chunk,
    strength: classifyStrength(chunk), // 'high' | 'medium' | 'low'
  }));

  ctx.aggregatedSignals = {
    total: allSignals.length,
    dedupedCount: classified.length,
    highStrength: classified.filter(c => c.strength === 'high'),
    mediumStrength: classified.filter(c => c.strength === 'medium'),
    lowStrength: classified.filter(c => c.strength === 'low'),
  };
}
```

Output JSON:

```json
{
  "total": 14,
  "dedupedCount": 8,
  "highStrength": [
    {
      "text": "Fed holds rates, Waller signals openness to June cut",
      "sources": ["tavily_news", "x_social"],
      "strength": "high"
    }
  ],
  "mediumStrength": [
    {
      "text": "10Y yield down 8bps to 4.18%",
      "sources": ["trad_markets"],
      "strength": "medium"
    }
  ],
  "lowStrength": []
}
```

---

## Step 8: LLM evaluation

This is the core of the heartbeat. We construct a prompt that gives Claude the thesis, the current position, all aggregated signals, and asks for an updated probability estimate.

```typescript
async function llmEval(ctx: HeartbeatContext): Promise<void> {
  if (ctx.killTriggered) {
    ctx.evaluation = { action: 'EXIT', reason: 'Kill condition triggered', confidence: 1.0 };
    return;
  }

  const prompt = buildEvalPrompt(ctx);
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [{ role: 'user', content: prompt }],
  });

  ctx.evaluation = parseEvalResponse(response);
}
```

The evaluation prompt is carefully constructed. Here's the actual template:

```typescript
function buildEvalPrompt(ctx: HeartbeatContext): string {
  return `You are evaluating a prediction market thesis.

THESIS: ${ctx.thesis.title}
POSITION: ${ctx.thesis.position} @ ${ctx.thesis.entryPrice}c
CURRENT PRICE: ${ctx.currentPrice}c
TIME TO RESOLUTION: ${ctx.thesis.daysToResolution} days

YOUR PREVIOUS ESTIMATE: ${ctx.thesis.lastEstimate}%

NEW SIGNALS (deduplicated, ranked by strength):
${JSON.stringify(ctx.aggregatedSignals, null, 2)}

MILESTONES:
${JSON.stringify(ctx.thesis.milestones, null, 2)}

INSTRUCTIONS:
1. Analyze each high-strength signal for thesis impact
2. Consider base rates and prior probability
3. Apply adversarial reasoning: what would a smart bear/bull argue?
4. Output JSON with: newEstimate (0-100), confidence (0-1), reasoning (string), action (HOLD|EXIT|ADD)`;
}
```

Critical design decision: **adversarial reasoning is mandatory**. The prompt explicitly forces the model to argue against the thesis before reaching a conclusion. Without this, LLMs exhibit confirmation bias -- they anchor to the existing estimate and find reasons to confirm it.

Output JSON:

```json
{
  "newEstimate": 38,
  "confidence": 0.72,
  "reasoning": "Waller's dovish signal is meaningful but one voice. Base rate for June cuts when March hold occurs is ~35%. Adversarial: hawks still outnumber doves on the committee, and next CPI could reverse this. Adjusting estimate up from 32 to 38.",
  "action": "HOLD"
}
```

---

## Step 9: Delta calculation

Compare the new evaluation against thresholds to determine if this heartbeat produces an actionable event.

```typescript
async function deltaCalc(ctx: HeartbeatContext): Promise<void> {
  const prev = ctx.thesis.lastEstimate;
  const curr = ctx.evaluation.newEstimate;
  const priceDelta = Math.abs(ctx.currentPrice - ctx.thesis.lastPrice);
  const estimateDelta = Math.abs(curr - prev);

  ctx.delta = {
    estimateChange: curr - prev,
    estimateDeltaAbs: estimateDelta,
    priceChange: ctx.currentPrice - ctx.thesis.lastPrice,
    priceDeltaAbs: priceDelta,
    isSignificant: estimateDelta >= 5 || priceDelta >= 5 || ctx.killTriggered,
    edgeChange: (curr - ctx.currentPrice) - (prev - ctx.thesis.lastPrice),
  };
}
```

Output JSON:

```json
{
  "estimateChange": 6,
  "estimateDeltaAbs": 6,
  "priceChange": 4,
  "priceDeltaAbs": 4,
  "isSignificant": true,
  "edgeChange": 2
}
```

The `isSignificant` threshold of 5 points was tuned empirically. At 3, we got too many false positives (noisy updates). At 10, we missed real moves. Five points hits the sweet spot.

---

## Step 10: Notification

If the delta is significant, we fire notifications. If not, we silently log and move on.

```typescript
async function notify(ctx: HeartbeatContext): Promise<void> {
  // Always persist the heartbeat result
  await db.insert(heartbeats).values({
    thesisId: ctx.thesis.id,
    signals: ctx.signals,
    evaluation: ctx.evaluation,
    delta: ctx.delta,
    runAt: new Date(),
  });

  // Update thesis with latest values
  await db.update(theses).set({
    lastPrice: ctx.currentPrice,
    lastEstimate: ctx.evaluation.newEstimate,
    lastHeartbeatAt: new Date(),
  }).where(eq(theses.id, ctx.thesis.id));

  if (!ctx.delta.isSignificant) return;

  // Significant change: notify via multiple channels
  const notification = {
    thesisTitle: ctx.thesis.title,
    action: ctx.evaluation.action,
    estimateChange: `${ctx.thesis.lastEstimate}% → ${ctx.evaluation.newEstimate}%`,
    priceChange: `${ctx.thesis.lastPrice}c → ${ctx.currentPrice}c`,
    reasoning: ctx.evaluation.reasoning,
    killTriggered: ctx.killTriggered || false,
  };

  await Promise.all([
    sendSlackNotification(notification),
    sendEmailDigest(notification),
    updateDashboard(notification),
  ]);
}
```

---

## Design decisions

### Why 15 minutes, not 1 minute?

Prediction markets are not equity markets. Kalshi's most active contracts see ~500 trades per day. Polymarket's liquid markets do maybe 2,000. At this volume, a 1-minute poll produces 14 identical readings for every 1 that contains new information.

More importantly, the LLM evaluation (Step 8) is the bottleneck. At ~$0.003 per evaluation, running every minute would cost $4.32/thesis/day instead of $0.29. Multiply by 50 theses and you're at $216/day for marginal benefit.

The 15-minute window also maps well to news cycles. Tavily's results meaningfully change every 10-20 minutes. Running faster just means paying for the same search results twice.

### Why adversarial search is mandatory

In our first version, the LLM evaluation had no adversarial requirement. The result: **72% of evaluations confirmed the prior estimate within 2 points.** The model was rubber-stamping.

After adding mandatory adversarial reasoning ("argue the strongest bear/bull case before concluding"), estimate changes became more distributed and, critically, more predictive. Our backtest showed a 14% improvement in calibration when adversarial prompting was included.

The cost is negligible -- about 100 extra tokens per evaluation, or $0.0003.

### Why traditional markets are included

We tested three configurations:

| Config | Calibration (Brier) | Avg lead time |
|--------|-------------------|---------------|
| News + prediction only | 0.218 | baseline |
| News + prediction + social | 0.201 | +8 min |
| News + prediction + social + trad markets | 0.183 | +22 min |

Traditional markets gave us the best marginal improvement. The signal propagation chain is: institutional traders move traditional markets first, news picks it up, retail moves prediction markets last. By including traditional market data, we're 22 minutes ahead of a pure prediction market monitor.

### Signal deduplication

Without deduplication, the LLM overweights events covered by multiple sources. A single Fed announcement might appear as:
- A Tavily news article (Reuters)
- A Tavily news article (Bloomberg)
- 3 X posts quoting the same announcement
- A Treasury yield move reflecting the announcement

That's 6 signals for 1 event. The embedding-based dedup (Step 7) collapses these to 1-2 signals, preventing the LLM from treating one event as six independent data points.

We use `text-embedding-3-small` with a 0.85 cosine similarity threshold. Below 0.85, legitimately different signals start getting merged. Above 0.90, obvious duplicates slip through.

### Evaluation prompt construction

The prompt template went through 23 iterations. Key learnings:

1. **Include the previous estimate.** Without it, the model has no anchor and estimates swing wildly between heartbeats.
2. **Include days to resolution.** Time decay matters enormously in prediction markets. A 60% estimate 30 days out means something different than 60% 2 days out.
3. **Force structured JSON output.** Free-text evaluations were inconsistent and hard to parse. Structured output made delta calculation deterministic.
4. **Separate the action from the estimate.** The model might raise its estimate from 30% to 45% (bullish) but recommend HOLD because the current price already reflects 44%. Edge = estimate - price.

---

## Cost breakdown

Per thesis, per day (96 heartbeats at 15-min intervals):

| Component | Cost/run | Daily (x96) |
|-----------|----------|-------------|
| Tavily search | $0.010 | $0.096 |
| Kalshi API | $0.000 | $0.000 |
| Polymarket API | $0.000 | $0.000 |
| Databento | $0.002 | $0.192 |
| X API | $0.001 | $0.096 |
| Embedding (dedup) | $0.0002 | $0.019 |
| Claude Sonnet eval | $0.003 | $0.288 |
| **Total** | **$0.006** | **$0.61** |

At 50 theses: **~$30.50/day** or **~$915/month**.

The Claude Sonnet evaluation is nearly half the cost. We considered using Haiku for routine heartbeats and Sonnet only when signals are strong, but the calibration difference was significant enough to keep Sonnet for all evaluations. Haiku's Brier score was 0.23 vs Sonnet's 0.18 -- the $0.19/day savings per thesis wasn't worth the accuracy loss when real money is on the line.

---

## Full heartbeat JSON

Here's the complete output of a single heartbeat run:

```json
{
  "thesisId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "runAt": "2026-03-28T14:15:00Z",
  "durationMs": 3420,
  "signals": [
    { "source": "tavily_news", "count": 5 },
    { "source": "price_rescan", "venues": ["kalshi"] },
    { "source": "trad_markets", "symbols": ["ZN", "ZQ"] },
    { "source": "x_social", "count": 7 },
    { "source": "milestone_check", "hits": 0 },
    { "source": "kill_condition", "triggered": false }
  ],
  "aggregation": {
    "totalSignals": 14,
    "dedupedSignals": 8,
    "highStrength": 1,
    "mediumStrength": 3,
    "lowStrength": 4
  },
  "evaluation": {
    "previousEstimate": 32,
    "newEstimate": 38,
    "confidence": 0.72,
    "action": "HOLD",
    "reasoning": "Waller's dovish signal is meaningful but one voice..."
  },
  "delta": {
    "estimateChange": 6,
    "priceChange": 4,
    "isSignificant": true,
    "edgeChange": 2
  },
  "notified": true
}
```

Runtime averages 3.2 seconds per thesis. The LLM evaluation (Step 8) accounts for ~2.1 seconds. Everything else is parallel I/O.

---

## What we'd change

Two things we're actively working on:

1. **Adaptive frequency.** High-volatility theses (earnings events, geopolitical crises) should heartbeat every 5 minutes. Low-vol theses (will X happen by year end?) could run every hour. We're building a volatility classifier that adjusts the cron interval dynamically.

2. **Multi-thesis correlation.** Right now each thesis is evaluated independently. But theses are correlated -- a Fed rate thesis and a mortgage rate thesis share signals. We're exploring a graph-based approach where shared signals propagate to all related theses in a single evaluation pass.

The heartbeat architecture is simple by design. Ten steps, clear data flow, predictable costs. The complexity lives in the prompt engineering (Step 8) and the signal quality (Steps 1-4), not in the orchestration.