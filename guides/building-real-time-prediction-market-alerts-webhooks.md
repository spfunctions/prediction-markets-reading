# Building Real-Time Prediction Market Alerts with Webhooks

> Polling wastes resources and misses events. Here is how to build a webhook-based alert system for prediction market price moves, confidence shifts, and strategy signals.

**Category:** architecture | **Author:** SimpleFunctions | **Reading time:** 6 min

---
You're running a prediction market thesis. You have positions on Kalshi. You need to know when something changes — a price move, a confidence shift in your causal model, a strategy condition triggering. The obvious approach is polling: check every N seconds. The better approach is webhooks.

## The Problem with Polling

Polling the Kalshi API every 30 seconds for 5 markets means 10 requests per minute, 14,400 per day. Most of those return identical data. You're burning rate limit budget (Kalshi allows ~10 req/s) on requests that tell you nothing new.

Worse, polling at fixed intervals means you miss events between polls. If a market moves 8 cents in 15 seconds and then corrects, your 30-second poll might never see it. The move happened, a window opened and closed, and your system was sleeping.

Polling also scales badly. Monitoring 5 markets is fine. Monitoring 50 markets across 2 venues at 30-second intervals is 200 requests per minute. You'll hit rate limits and still miss fast moves.

## SimpleFunctions Webhook Architecture

SimpleFunctions emits webhooks on three event types:

### 1. Confidence Delta

Fired when your thesis confidence changes by more than a configurable threshold (default: 3 percentage points).

```json
{
  "event": "confidence_delta",
  "thesis_slug": "iran-war",
  "timestamp": "2026-03-15T14:32:00Z",
  "data": {
    "previous_confidence": 0.72,
    "current_confidence": 0.67,
    "delta": -0.05,
    "trigger_node": "n1.2",
    "trigger_node_label": "Iran continues retaliation",
    "node_previous": 0.85,
    "node_current": 0.74,
    "cause": "Diplomatic channel reopened via Oman intermediary"
  }
}
```

This tells you *why* confidence changed, not just *that* it changed. The `trigger_node` field points to the specific causal node that moved. The `cause` field summarizes the information that caused the update.

### 2. Position Update

Fired when a market price crosses a threshold relative to your thesis, creating or destroying edge.

```json
{
  "event": "position_update",
  "thesis_slug": "iran-war",
  "timestamp": "2026-03-15T15:10:00Z",
  "data": {
    "ticker": "KXRECESSION-26",
    "venue": "kalshi",
    "previous_price": 35,
    "current_price": 41,
    "thesis_implied": 45,
    "previous_edge": 10,
    "current_edge": 4,
    "edge_status": "narrowing",
    "alert": "Edge below strategy minimum (5 points). Consider exit."
  }
}
```

### 3. Evaluation Summary

Fired after each complete heartbeat evaluation cycle. This is the "all clear" signal — even if nothing changed, you get a confirmation that the system is running.

```json
{
  "event": "evaluation_summary",
  "thesis_slug": "iran-war",
  "timestamp": "2026-03-15T16:00:00Z",
  "data": {
    "nodes_evaluated": 7,
    "nodes_changed": 1,
    "max_delta": 0.02,
    "markets_checked": 3,
    "signals_fired": 0,
    "next_evaluation": "2026-03-15T18:00:00Z"
  }
}
```

## Setting Up a Webhook Endpoint

You need an HTTPS endpoint that accepts POST requests. Here's a minimal Express.js server:

```javascript
import express from 'express';
import crypto from 'crypto';

const app = express();
app.use(express.json());

const WEBHOOK_SECRET = process.env.SF_WEBHOOK_SECRET;

// Verify the webhook signature
function verifySignature(payload, signature) {
  const expected = crypto
    .createHmac('sha256', WEBHOOK_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
}

app.post('/webhooks/simplefunctions', (req, res) => {
  const signature = req.headers['x-sf-signature'];

  if (!signature || !verifySignature(req.body, signature)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const { event, thesis_slug, data } = req.body;

  switch (event) {
    case 'confidence_delta':
      handleConfidenceDelta(thesis_slug, data);
      break;
    case 'position_update':
      handlePositionUpdate(thesis_slug, data);
      break;
    case 'evaluation_summary':
      handleEvaluationSummary(thesis_slug, data);
      break;
    default:
      console.log('Unknown event:', event);
  }

  res.status(200).json({ received: true });
});

app.listen(3001, () => {
  console.log('Webhook server listening on port 3001');
});
```

Register your endpoint:

```bash
sf webhooks add https://your-server.com/webhooks/simplefunctions
```

## Connecting to Slack

Most traders want alerts in Slack. Here's the handler:

```javascript
async function handleConfidenceDelta(thesisSlug, data) {
  const direction = data.delta > 0 ? 'increased' : 'decreased';
  const emoji = data.delta > 0 ? ':chart_with_upwards_trend:' : ':chart_with_downwards_trend:';

  await fetch(process.env.SLACK_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      text: [
        `${emoji} *${thesisSlug}* confidence ${direction}`,
        `${(data.previous_confidence * 100).toFixed(1)}% → ${(data.current_confidence * 100).toFixed(1)}%`,
        `Trigger: ${data.trigger_node_label}`,
        `Cause: ${data.cause}`,
      ].join('\n'),
    }),
  });
}

async function handlePositionUpdate(thesisSlug, data) {
  if (data.current_edge < 5) {
    await fetch(process.env.SLACK_WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        text: [
          `:warning: *${data.ticker}* edge narrowing`,
          `Price: ${data.previous_price}¢ → ${data.current_price}¢`,
          `Edge: ${data.previous_edge}pts → ${data.current_edge}pts`,
          `Thesis implied: ${data.thesis_implied}%`,
          `${data.alert}`,
        ].join('\n'),
      }),
    });
  }
}
```

## Connecting to Discord

Discord webhooks are nearly identical — different payload shape:

```javascript
async function sendToDiscord(title, description) {
  await fetch(process.env.DISCORD_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      embeds: [{
        title,
        description,
        color: 0xff6600, // orange
        timestamp: new Date().toISOString(),
      }],
    }),
  });
}
```

## Connecting to Telegram

For Telegram, use the Bot API:

```javascript
async function sendToTelegram(message) {
  const TELEGRAM_BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN;
  const TELEGRAM_CHAT_ID = process.env.TELEGRAM_CHAT_ID;

  await fetch(
    `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        chat_id: TELEGRAM_CHAT_ID,
        text: message,
        parse_mode: 'Markdown',
      }),
    }
  );
}
```

## The Delta API: When Webhooks Aren't an Option

Sometimes you can't receive webhooks — firewalled environment, local development, or you just want to pull on your schedule. The delta API gives you efficient polling:

```bash
# Get all changes since a timestamp
sf delta --since 2026-03-15T14:00:00Z --thesis iran-war
```

```json
{
  "since": "2026-03-15T14:00:00Z",
  "until": "2026-03-15T16:30:00Z",
  "changes": [
    {
      "type": "confidence_delta",
      "timestamp": "2026-03-15T14:32:00Z",
      "node": "n1.2",
      "from": 0.85,
      "to": 0.74
    },
    {
      "type": "price_move",
      "timestamp": "2026-03-15T15:10:00Z",
      "ticker": "KXRECESSION-26",
      "from": 35,
      "to": 41
    }
  ]
}
```

The delta API returns only what changed since your last check. No wasted bandwidth, no missed events. Store the `until` timestamp and use it as `since` in your next call — you'll never miss a change and never see a duplicate.

```javascript
// Efficient polling loop
let cursor = new Date(Date.now() - 3600000).toISOString(); // start 1hr ago

setInterval(async () => {
  const delta = await sfClient.delta({
    since: cursor,
    thesis: 'iran-war',
  });

  if (delta.changes.length > 0) {
    for (const change of delta.changes) {
      await processChange(change);
    }
  }

  cursor = delta.until; // advance cursor
}, 60000); // poll every 60 seconds
```

This gives you webhook-like semantics with a polling transport. You lose real-time latency (up to 60 seconds behind) but gain simplicity and firewall compatibility.

## Configuring Alert Thresholds

Not every 1-point move deserves a notification. Configure thresholds per thesis:

```bash
# Only alert on confidence changes > 5 percentage points
sf webhooks config --thesis iran-war --confidence-threshold 0.05

# Only alert on price moves > 3 cents
sf webhooks config --thesis iran-war --price-threshold 3

# Only alert on edge changes that cross your strategy boundary
sf webhooks config --thesis iran-war --edge-threshold 5
```

Start with loose thresholds and tighten them over time. Alert fatigue is real — if you're getting 20 notifications a day, you'll start ignoring all of them. Aim for 2-5 meaningful alerts per day per thesis.

## Production Considerations

**Retry logic.** SimpleFunctions retries failed webhook deliveries 3 times with exponential backoff (10s, 60s, 300s). If all retries fail, the event is logged and available via the delta API. You won't lose events.

**Ordering.** Webhooks are delivered in order per thesis but may arrive out of order across theses. If you're processing events from multiple theses, use the `timestamp` field for ordering, not arrival time.

**Idempotency.** Each webhook includes an `event_id` field. Store processed event IDs and skip duplicates. The retry logic means you might receive the same event twice.

**Latency.** Webhooks are dispatched within 2 seconds of the triggering evaluation. Network latency to your endpoint adds another 50-500ms depending on geography. For prediction markets, where meaningful moves happen over minutes to hours, sub-second latency doesn't matter. If you need faster, you're probably overtrading.

The goal isn't speed. The goal is never missing a signal and never drowning in noise.