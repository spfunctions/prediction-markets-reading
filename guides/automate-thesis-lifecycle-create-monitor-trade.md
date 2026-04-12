# Automating thesis lifecycle: create, monitor, evaluate, trade

> The full agentic loop in code: six API calls from raw thesis to executed trade, with complete request/response examples.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 12 min

---
Six API calls. That's the distance between a raw thesis and an executed trade. This guide walks through every one, with complete curl commands, real responses, and the exact tickers you'd use on Kalshi.

No theory. No architecture diagrams. Just the loop.

## Prerequisites

You need two things:

```bash
# 1. SimpleFunctions API key
export SF_API_KEY="sk-sf-..."

# 2. CLI installed (for step 6)
npm install -g @simplefunctions/cli
sf setup  # configures Kalshi keys + API key
```

The base URL for all API calls:

```bash
export SF_BASE="https://api.simplefunctions.com"
```

---

## Step 1: Create the Thesis

`POST /api/thesis/create` takes a raw thesis string and kicks off formation: causal tree decomposition, market scanning across Kalshi and Polymarket, and edge analysis. Two modes: async (default, returns 202) and sync (waits for formation, returns 200).

### Async mode (recommended for automation)

```bash
curl -X POST "$SF_BASE/api/thesis/create" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rawThesis": "Oil prices will exceed $150/barrel by end of 2026 due to prolonged Strait of Hormuz disruption and OPEC+ production cuts failing to compensate for lost Iranian supply",
    "title": "Oil $150 by Dec 2026"
  }'
```

**Response (202 Accepted):**

```json
{
  "id": "a1b2c3d4-5678-9abc-def0-1234567890ab",
  "thesisId": "a1b2c3d4-5678-9abc-def0-1234567890ab",
  "status": "forming",
  "message": "Thesis creation started. Poll GET /api/thesis/:id for status."
}
```

Save the ID. Everything downstream uses it:

```bash
export THESIS_ID="a1b2c3d4-5678-9abc-def0-1234567890ab"
```

### Sync mode (blocks until formation completes)

Append `?sync=true` to wait. Formation typically takes 30-90 seconds depending on the number of markets scanned.

```bash
curl -X POST "$SF_BASE/api/thesis/create?sync=true" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rawThesis": "Oil prices will exceed $150/barrel by end of 2026 due to prolonged Strait of Hormuz disruption and OPEC+ production cuts failing to compensate for lost Iranian supply",
    "title": "Oil $150 by Dec 2026"
  }'
```

**Response (200 OK) — full formation result:**

```json
{
  "success": true,
  "thesisId": "a1b2c3d4-5678-9abc-def0-1234567890ab",
  "confidence": 0.62,
  "processingTimeMs": 47200,
  "causalTree": {
    "rootClaim": "WTI crude oil exceeds $150/bbl by December 2026",
    "nodes": [
      {
        "id": "n1",
        "label": "Strait of Hormuz disruption persists",
        "probability": 0.55,
        "importance": 0.8,
        "children": [
          {
            "id": "n1a",
            "label": "US-Iran military escalation continues",
            "probability": 0.6,
            "importance": 0.9,
            "children": []
          }
        ]
      },
      {
        "id": "n2",
        "label": "OPEC+ fails to offset supply loss",
        "probability": 0.45,
        "importance": 0.7,
        "children": []
      }
    ],
    "generatedAt": "2026-03-28T14:00:00Z",
    "model": "claude-sonnet-4-20250514"
  },
  "edgeAnalysis": {
    "edges": [
      {
        "marketId": "kalshi-KXWTIMAX-26DEC31-T150",
        "venue": "kalshi",
        "marketTitle": "Will WTI crude oil be above $150 on Dec 31, 2026?",
        "marketPrice": 18,
        "thesisImpliedPrice": 34,
        "edgeSize": 16,
        "direction": "yes",
        "confidence": 0.72,
        "rationale": "Market underprices sustained Hormuz disruption scenario. Implied probability of 18% vs thesis-derived 34%.",
        "relatedNodeId": "n1",
        "eventTicker": "KXWTIMAX-26DEC31"
      },
      {
        "marketId": "kalshi-KXRECSSNBER-26",
        "venue": "kalshi",
        "marketTitle": "Will NBER declare a US recession starting in 2026?",
        "marketPrice": 32,
        "thesisImpliedPrice": 44,
        "edgeSize": 12,
        "direction": "yes",
        "confidence": 0.58,
        "rationale": "Oil shock pass-through to recession odds is under-reflected. Energy cost spiral increases recession probability.",
        "relatedNodeId": "n2",
        "eventTicker": "KXRECSSNBER-26"
      }
    ],
    "analyzedAt": "2026-03-28T14:00:45Z",
    "model": "claude-sonnet-4-20250514",
    "totalMarketsAnalyzed": 47
  },
  "suggestedPositions": [
    {
      "venue": "kalshi",
      "externalId": "kalshi-KXWTIMAX-26DEC31-T150",
      "title": "WTI crude oil above $150 on Dec 31, 2026",
      "direction": "yes",
      "entryPrice": 18,
      "edge": 16,
      "rationale": "Largest edge. Thesis implies 34% probability vs market 18%. Positive carry if Hormuz disruption persists."
    }
  ]
}
```

Formation is idempotent. Submitting the same `rawThesis` returns the existing thesis rather than creating a duplicate.

---

## Step 2: Monitor via Delta Endpoint

`GET /api/thesis/:id/changes?since=<ISO timestamp>` is designed for polling. Returns <100 bytes when nothing changed, ~400 bytes when something did. Use this in cron jobs or agent loops instead of fetching the full thesis each time.

```bash
curl -s "$SF_BASE/api/thesis/$THESIS_ID/changes?since=2026-03-28T12:00:00Z" \
  -H "Authorization: Bearer $SF_API_KEY"
```

**Response when nothing changed:**

```json
{
  "changed": false,
  "updatedAt": "2026-03-28T14:00:45Z"
}
```

**Response when confidence moved:**

```json
{
  "changed": true,
  "confidence": 0.68,
  "confidenceDelta": 6.0,
  "evaluationCount": 2,
  "latestSummary": "IAEA report confirms Iran enrichment acceleration. Hormuz insurance premiums up 40bp. Node n1a probability revised from 0.60 to 0.72.",
  "latestEvalAt": "2026-03-28T18:30:00Z",
  "updatedAt": "2026-03-28T18:30:00Z"
}
```

Short IDs work. If your thesis ID is `a1b2c3d4-5678-9abc-def0-1234567890ab`, you can use just `a1b2c3d4`.

### Polling pattern for automation

```bash
#!/bin/bash
# poll-thesis.sh — check every 5 minutes, alert on confidence shift
LAST_CHECK=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

while true; do
  RESP=$(curl -s "$SF_BASE/api/thesis/$THESIS_ID/changes?since=$LAST_CHECK" \
    -H "Authorization: Bearer $SF_API_KEY")

  CHANGED=$(echo "$RESP" | jq -r '.changed')

  if [ "$CHANGED" = "true" ]; then
    DELTA=$(echo "$RESP" | jq -r '.confidenceDelta')
    echo "[$(date)] Confidence shifted by $DELTA"
    echo "$RESP" | jq '.latestSummary'
    LAST_CHECK=$(echo "$RESP" | jq -r '.updatedAt')
  fi

  sleep 300
done
```

---

## Step 3: Inject Signals

`POST /api/thesis/:id/signal` lets you push information into the thesis queue. Signals are consumed on the next monitor cycle (automatic or forced via step 4). This is how external data — webhook payloads, news alerts, manual observations — enters the system.

Four signal types: `news`, `price_move`, `user_note`, `external`.

### Inject a news signal

```bash
curl -X POST "$SF_BASE/api/thesis/$THESIS_ID/signal" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "news",
    "content": "Reuters: Saudi Arabia announces emergency OPEC+ meeting for April 3. Sources indicate kingdom may unilaterally increase production by 500k bpd to stabilize prices.",
    "source": "reuters-webhook"
  }'
```

**Response (202 Accepted):**

```json
{
  "id": "sig-e5f6a7b8-1234-5678-9abc-def012345678",
  "message": "Signal queued. It will be consumed in the next monitor cycle."
}
```

### Inject a price move signal

```bash
curl -X POST "$SF_BASE/api/thesis/$THESIS_ID/signal" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "price_move",
    "content": "KXWTIMAX-26DEC31-T150 moved from 18c to 24c in 2 hours. Volume spike: 12,000 contracts traded vs 800 daily average.",
    "source": "kalshi-price-alert"
  }'
```

### Inject a manual note

```bash
curl -X POST "$SF_BASE/api/thesis/$THESIS_ID/signal" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "user_note",
    "content": "Spoke with shipping analyst. Lloyd list data shows 14 tankers rerouted around Cape of Good Hope this week vs 3 last month. Insurance premiums for Hormuz transit now 2.1% of cargo value."
  }'
```

Signals are fire-and-forget. They queue immediately and get consumed when the monitor agent runs. You can inject multiple signals before triggering evaluation.

---

## Step 4: Force Evaluation

`POST /api/thesis/:id/evaluate` runs the monitor cycle immediately instead of waiting for the next scheduled cron. It consumes all pending signals, re-evaluates each causal node, updates confidence, and returns the full evaluation result.

This is the "what changed?" button. Use it after injecting signals, or whenever you want a fresh read.

```bash
curl -X POST "$SF_BASE/api/thesis/$THESIS_ID/evaluate" \
  -H "Authorization: Bearer $SF_API_KEY"
```

**Response (200 OK):**

```json
{
  "evaluatedAt": "2026-03-28T19:15:00Z",
  "model": "claude-sonnet-4-20250514",
  "previousConfidence": 0.62,
  "newConfidence": 0.54,
  "confidenceDelta": -8,
  "events": [
    {
      "type": "news",
      "content": "Reuters: Saudi Arabia announces emergency OPEC+ meeting for April 3. Sources indicate kingdom may unilaterally increase production by 500k bpd.",
      "source": "reuters-webhook",
      "timestamp": "2026-03-28T19:00:00Z",
      "affectedNodes": ["n2"],
      "impact": "bearish",
      "magnitude": 0.7
    },
    {
      "type": "price_move",
      "content": "KXWTIMAX-26DEC31-T150 moved from 18c to 24c in 2 hours.",
      "source": "kalshi-price-alert",
      "timestamp": "2026-03-28T19:00:01Z",
      "affectedNodes": ["n1"],
      "impact": "bullish",
      "magnitude": 0.4
    }
  ],
  "updatedNodes": [
    {
      "nodeId": "n2",
      "previousProbability": 0.45,
      "newProbability": 0.28,
      "reason": "Saudi unilateral production increase would partially offset Iranian supply loss. 500k bpd covers roughly 40% of disrupted Hormuz volume."
    },
    {
      "nodeId": "n1a",
      "previousProbability": 0.60,
      "newProbability": 0.65,
      "reason": "Price move on KXWTIMAX suggests market participants pricing in continued escalation. Consistent with Hormuz disruption persistence."
    }
  ],
  "positionUpdates": [
    {
      "positionId": "kalshi-KXWTIMAX-26DEC31-T150",
      "previousEdge": 16,
      "newEdge": 7,
      "recommendation": "reduce",
      "reason": "Edge compressed from 16 to 7. Market price moved toward thesis price, but OPEC+ response lowers thesis-implied probability. Consider taking partial profit."
    },
    {
      "positionId": "kalshi-KXRECSSNBER-26",
      "previousEdge": 12,
      "newEdge": 9,
      "recommendation": "hold",
      "reason": "Edge narrowed modestly. Recession probability decreases if oil supply stabilizes, but pass-through effects are lagged. Hold."
    }
  ],
  "summary": "Net bearish. Saudi production increase announcement significantly reduces OPEC-failure node probability (0.45 -> 0.28). Thesis confidence drops 8 points to 0.54. KXWTIMAX edge compressed to 7 — consider trimming. KXRECSSNBER edge at 9, still viable."
}
```

Key fields for automation:
- `confidenceDelta` — magnitude and direction of the shift
- `positionUpdates[].recommendation` — one of `hold`, `increase`, `reduce`, `close`
- `summary` — human-readable synopsis for logging

---

## Step 5: Create a Strategy

`POST /api/thesis/:id/strategies` creates a structured trading strategy tied to the thesis. Strategies define entry conditions, exit conditions, sizing, and stop-losses. They're the bridge between analysis and execution.

```bash
curl -X POST "$SF_BASE/api/thesis/$THESIS_ID/strategies" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "marketId": "kalshi-KXWTIMAX-26DEC31-T150",
    "market": "WTI crude oil above $150 on Dec 31, 2026",
    "direction": "yes",
    "horizon": "9 months",
    "entryBelow": 25,
    "stopLoss": 8,
    "takeProfit": 55,
    "maxQuantity": 500,
    "perOrderQuantity": 50,
    "rationale": "Thesis implies 34% probability vs market 18-24%. Entry below 25c captures minimum 9-point edge. Stop at 8c limits downside to $85/contract.",
    "entry": "Buy YES at limit 25c or below. Scale in: 50 contracts per order, max 500.",
    "exit": "Take profit above 55c. Stop loss at 8c. Re-evaluate if confidence drops below 0.40.",
    "sizing": "Max exposure $125 (500 contracts * 25c). Risk per contract: 17c (25c entry - 8c stop).",
    "priority": 1,
    "createdBy": "api"
  }'
```

**Response (201 Created):**

```json
{
  "id": "strat-f9a8b7c6-5432-1098-7654-abcdef012345"
}
```

### List strategies for a thesis

```bash
curl -s "$SF_BASE/api/thesis/$THESIS_ID/strategies" \
  -H "Authorization: Bearer $SF_API_KEY" | jq
```

```json
{
  "strategies": [
    {
      "id": "strat-f9a8b7c6-5432-1098-7654-abcdef012345",
      "marketId": "kalshi-KXWTIMAX-26DEC31-T150",
      "market": "WTI crude oil above $150 on Dec 31, 2026",
      "direction": "yes",
      "entryBelow": 25,
      "stopLoss": 8,
      "takeProfit": 55,
      "maxQuantity": 500,
      "perOrderQuantity": 50,
      "status": "active",
      "priority": 1
    }
  ]
}
```

Filter by status with `?status=active`.

---

## Step 6: Execute the Trade

The `sf` CLI handles order execution against Kalshi. Strategies give you the parameters; `sf buy` sends the order.

### Buy YES contracts

```bash
# Limit order: 50 contracts at 22c
sf buy KXWTIMAX-26DEC31-T150 50 --price 22 --side yes
```

**CLI output:**

```
  BUY Order
  -----------------------------------
  Ticker:    KXWTIMAX-26DEC31-T150
  Side:      YES
  Quantity:  50
  Type:      limit
  Price:     22¢
  Max cost:  $11.00

  Executing in 3...  (Ctrl+C to cancel)
  ✓ Order placed: ord-abc123
  Status: resting
  Trade logged to ~/.sf/trade-journal.jsonl
```

### Market order (immediate fill)

```bash
# Market order: 25 contracts, fill at best available price
sf buy KXWTIMAX-26DEC31-T150 25 --market --side yes
```

### Buy the recession contract

```bash
# Second leg: recession thesis at 30c
sf buy KXRECSSNBER-26 100 --price 30 --side yes
```

The CLI auto-logs every trade to `~/.sf/trade-journal.jsonl` for audit trail.

---

## Putting It All Together: The Agent Loop

Here's the complete automation script that ties all six steps into a single loop:

```bash
#!/bin/bash
# thesis-agent-loop.sh
# Full lifecycle: create -> monitor -> signal -> evaluate -> strategize -> trade
set -euo pipefail

SF_BASE="https://api.simplefunctions.com"
SF_API_KEY="${SF_API_KEY:?Set SF_API_KEY}"

# ── Step 1: Create thesis (sync mode) ────────────────────────────────
echo "[1/6] Creating thesis..."
CREATE_RESP=$(curl -s -X POST "$SF_BASE/api/thesis/create?sync=true" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rawThesis": "Oil prices will exceed $150/barrel by end of 2026 due to prolonged Strait of Hormuz disruption",
    "title": "Oil $150 by Dec 2026"
  }')

THESIS_ID=$(echo "$CREATE_RESP" | jq -r '.thesisId')
CONFIDENCE=$(echo "$CREATE_RESP" | jq -r '.confidence')
echo "  Thesis: $THESIS_ID (confidence: $CONFIDENCE)"

# ── Step 2: Check for changes (initial baseline) ─────────────────────
echo "[2/6] Setting monitoring baseline..."
SINCE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# ── Step 3: Inject signal ─────────────────────────────────────────────
echo "[3/6] Injecting signal..."
curl -s -X POST "$SF_BASE/api/thesis/$THESIS_ID/signal" \
  -H "Authorization: Bearer $SF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "news",
    "content": "IAEA confirms Iran enrichment at 84%, highest level recorded.",
    "source": "iaea-report"
  }' | jq -r '.message'

# ── Step 4: Force evaluation ──────────────────────────────────────────
echo "[4/6] Evaluating..."
EVAL_RESP=$(curl -s -X POST "$SF_BASE/api/thesis/$THESIS_ID/evaluate" \
  -H "Authorization: Bearer $SF_API_KEY")

NEW_CONF=$(echo "$EVAL_RESP" | jq -r '.newConfidence')
DELTA=$(echo "$EVAL_RESP" | jq -r '.confidenceDelta')
echo "  Confidence: $NEW_CONF (delta: $DELTA)"

# ── Step 5: Create strategy if edge exists ────────────────────────────
TOP_EDGE=$(echo "$CREATE_RESP" | jq -r '.edgeAnalysis.edges[0].edgeSize')
TOP_TICKER=$(echo "$CREATE_RESP" | jq -r '.edgeAnalysis.edges[0].eventTicker')
TOP_PRICE=$(echo "$CREATE_RESP" | jq -r '.edgeAnalysis.edges[0].marketPrice')

if [ "$TOP_EDGE" -gt 5 ]; then
  echo "[5/6] Creating strategy (edge: ${TOP_EDGE})..."
  ENTRY=$((TOP_PRICE + 5))
  STOP=$((TOP_PRICE - 10))

  STRAT_ID=$(curl -s -X POST "$SF_BASE/api/thesis/$THESIS_ID/strategies" \
    -H "Authorization: Bearer $SF_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"marketId\": \"kalshi-${TOP_TICKER}-T150\",
      \"market\": \"WTI crude above 150\",
      \"direction\": \"yes\",
      \"entryBelow\": $ENTRY,
      \"stopLoss\": $STOP,
      \"maxQuantity\": 200,
      \"perOrderQuantity\": 50
    }" | jq -r '.id')
  echo "  Strategy: $STRAT_ID"

  # ── Step 6: Execute ─────────────────────────────────────────────────
  echo "[6/6] Executing trade..."
  sf buy "${TOP_TICKER}-T150" 50 --price "$ENTRY" --side yes
else
  echo "[5/6] Edge too small ($TOP_EDGE). Skipping strategy + trade."
fi

echo "Done. Thesis $THESIS_ID is live."
```

---

## Error Handling Reference

Every endpoint returns consistent error shapes:

```json
{
  "error": "Thesis not found"
}
```

| Status | Meaning | Action |
|--------|---------|--------|
| 400 | Missing required field | Check request body |
| 401 | Invalid or missing API key | Verify `SF_API_KEY` |
| 404 | Thesis not found | Check ID, or thesis belongs to another user |
| 500 | Server error | Retry with backoff |

The `/evaluate` endpoint has a 120-second timeout (`maxDuration: 120`). For theses with many causal nodes and signals, evaluation can take 30-60 seconds. Set your client timeout accordingly.

---

## Key Design Decisions

**Why fire-and-forget signals?** Signals queue instantly (202) so your webhook handler never blocks. The monitor agent batch-processes them, which produces better evaluations than processing signals one at a time — the agent sees all new information together and can assess interactions between signals.

**Why separate create and evaluate?** Formation builds the initial model. Evaluation updates it. They use different prompts, different context windows, and different cost profiles. Formation scans markets; evaluation consumes signals. Keeping them separate means you pay for formation once and evaluation is cheap.

**Why strategies as a separate resource?** Strategies persist independently of evaluations. An evaluation might recommend "reduce," but your strategy defines exactly what "reduce" means: which stop-loss, which take-profit, how many contracts per tranche. The strategy is your execution plan; the evaluation is your intelligence update.

**Short ID support.** Every endpoint that takes a thesis ID accepts prefix matches. Use the first 8 characters instead of the full UUID. The system resolves it server-side.

This is the complete loop. Create, monitor, inject, evaluate, strategize, execute. Everything after step 1 can run unattended.