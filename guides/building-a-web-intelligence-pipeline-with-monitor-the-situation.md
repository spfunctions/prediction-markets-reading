# Building a Web Intelligence Pipeline with monitor-the-situation

> From scraping any URL to divergence-aware market intelligence — in one API call. Complete guide with examples.

**Category:** guide | **Author:** SimpleFunctions Research | **Reading time:** 10 min

---
This guide walks through building a complete web intelligence pipeline using SimpleFunctions' `monitor-the-situation` API. From scraping any URL to generating divergence-aware market intelligence — in a single API call.

## Architecture

```
Source (Firecrawl)  →  Analysis (LLM)  →  Enrichment (Markets)  →  Webhook
     ↓                      ↓                    ↓                    ↓
  scrape/crawl/       Any OpenRouter         9,706 contracts       HTTPS + HMAC
  search/map/          model + custom        Kalshi + Polymarket    Slack/Discord/
  extract/batch        JSON schema           SF Index               Custom
```

Each step is optional. You can use just the source (Firecrawl-as-a-service), just the enrichment (paste text, get market intelligence), or the full pipeline.

## Quick Start: Enrich Any Text

The simplest endpoint. No auth, no Firecrawl, no setup:

```bash
curl -s -X POST https://simplefunctions.dev/api/monitor-the-situation/enrich \
  -H "Content-Type: application/json" \
  -d '{
    "content": "3 front-page HN articles about Iran oil sanctions. 847 comments. 78% sentiment expects military escalation.",
    "topics": ["iran", "oil"]
  }'
```

Response includes:
- `findings[]` — divergence analysis with severity ratings
- `headline` — one-line summary
- `brief` — 2-3 sentence summary
- `tweetable` — tweet-ready version with data points
- `markets[]` — matching prediction market contracts with live prices
- `index` — SF Index (uncertainty, geopolitical risk, momentum)

## Full Pipeline: Scrape → Analyze → Enrich → Push

```bash
curl -s -X POST https://simplefunctions.dev/api/monitor-the-situation \
  -H "Content-Type: application/json" \
  -d '{
    "source": {
      "action": "scrape",
      "url": "https://news.ycombinator.com",
      "options": {
        "formats": ["markdown"],
        "onlyMainContent": true
      }
    },
    "analysis": {
      "enabled": true,
      "model": "google/gemini-2.5-flash",
      "prompt": "Extract the top 5 discussion topics. For each: topic name, comment count, overall sentiment (bullish/bearish/neutral), and any claims about future events.",
      "schema": {
        "type": "object",
        "properties": {
          "topics": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "name": {"type": "string"},
                "comments": {"type": "number"},
                "sentiment": {"type": "string"},
                "futureClaims": {"type": "array", "items": {"type": "string"}}
              }
            }
          }
        }
      }
    },
    "enrich": {
      "enabled": true,
      "topics": ["AI", "iran", "regulation"],
      "includeIndex": true
    },
    "webhook": {
      "url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
      "format": "brief",
      "secret": "your-hmac-signing-key"
    }
  }'
```

## Firecrawl Actions

The `source.action` field controls what Firecrawl does. All Firecrawl options are passed through — you get the full SDK.

### scrape — Single URL to Markdown

```json
{
  "source": {
    "action": "scrape",
    "url": "https://reuters.com/world/middle-east",
    "options": {
      "formats": ["markdown", "screenshot"],
      "onlyMainContent": true,
      "mobile": false,
      "blockAds": true
    }
  }
}
```

### search — Web Search

```json
{
  "source": {
    "action": "search",
    "query": "iran oil sanctions 2026 latest",
    "options": {"limit": 5}
  }
}
```

### crawl — Multi-Page

```json
{
  "source": {
    "action": "crawl",
    "url": "https://www.sec.gov/cgi-bin/browse-edgar",
    "options": {
      "limit": 20,
      "maxDepth": 2,
      "includePaths": ["/Archives/edgar/data/"],
      "scrapeOptions": {"formats": ["markdown"]}
    }
  }
}
```

### extract — Structured Data with Schema

```json
{
  "source": {
    "action": "extract",
    "urls": ["https://finance.yahoo.com/quote/USO"],
    "options": {
      "schema": {
        "type": "object",
        "properties": {
          "price": {"type": "number"},
          "change": {"type": "number"},
          "volume": {"type": "string"}
        }
      },
      "prompt": "Extract the current oil ETF price and daily change"
    }
  }
}
```

### map — Discover Site Structure

```json
{
  "source": {
    "action": "map",
    "url": "https://www.congress.gov/legislation"
  }
}
```

### batch_scrape — Parallel URLs

```json
{
  "source": {
    "action": "batch_scrape",
    "urls": [
      "https://news.ycombinator.com",
      "https://reddit.com/r/geopolitics",
      "https://www.bbc.com/news/world"
    ],
    "options": {"formats": ["markdown"], "onlyMainContent": true}
  }
}
```

## LLM Model Selection

The `analysis.model` field accepts any OpenRouter model ID. Choose based on your tradeoff:

| Model | Cost/M tokens | Best for |
|-------|---------------|----------|
| `google/gemini-2.5-flash` | $0.15 / $0.60 | Fast extraction, structured output |
| `deepseek/deepseek-v3.2` | $0.28 / $0.40 | Budget bulk processing |
| `anthropic/claude-sonnet-4.6` | $3 / $15 | Nuanced analysis |
| `anthropic/claude-opus-4.6` | $5 / $25 | Complex reasoning |
| `openai/gpt-5.1` | $1.25 / $10 | Balanced quality/cost |

If you provide a `schema` (JSON Schema), the API uses structured output (`generateObject`). Otherwise it returns free-form text.

## Webhook Configuration

Webhooks are HTTPS-only with SSRF protection. Private IPs, localhost, and cloud metadata endpoints are blocked.

### HMAC-SHA256 Signing

When you provide a `secret`, every webhook delivery includes an `X-SF-Signature` header:

```
X-SF-Signature: sha256=a1b2c3d4...
```

Verify on your server:

```javascript
const crypto = require('crypto')
const expected = crypto.createHmac('sha256', secret).update(body).digest('hex')
const received = req.headers['x-sf-signature'].replace('sha256=', '')
if (expected !== received) throw new Error('Invalid signature')
```

### Webhook Formats

- `full` — Complete result: source summary, analysis, markets, digest, index
- `brief` — Headline + brief summary + timestamp
- `tweetable` — Single text field, tweet-ready

## Agent Integration: MCP

Connect via Model Context Protocol:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

Two tools available:

**`monitor_the_situation`** — Full pipeline. Pass source, analysis, enrich, webhook as parameters.

**`enrich_content`** — Quick cross-reference. Pass content + topics.

Example agent prompt: *"Scrape news.ycombinator.com, extract the top AI discussions, and cross-reference with prediction markets. Push a brief digest to my Slack webhook."*

## Agent Integration: Python

```python
import requests

# Full pipeline
result = requests.post("https://simplefunctions.dev/api/monitor-the-situation", json={
    "source": {"action": "scrape", "url": "https://news.ycombinator.com"},
    "analysis": {
        "enabled": True,
        "model": "google/gemini-2.5-flash",
        "prompt": "Summarize the top 5 stories"
    },
    "enrich": {"enabled": True, "topics": ["AI", "tech"], "includeIndex": True}
}).json()

print(result["digest"]["headline"])
print(result["digest"]["tweetable"])
for f in result["digest"]["findings"]:
    print(f"  [{f['severity']}] {f['topic']}: {f['divergence']}")
```

## Agent Integration: Periodic Monitoring

Set up a cron job or agent loop to monitor continuously:

```python
import time, requests

SOURCES = [
    {"action": "scrape", "url": "https://news.ycombinator.com"},
    {"action": "search", "query": "iran oil latest"},
    {"action": "scrape", "url": "https://www.bbc.com/news/world"},
]

while True:
    for source in SOURCES:
        result = requests.post("https://simplefunctions.dev/api/monitor-the-situation", json={
            "source": source,
            "enrich": {"enabled": True, "topics": ["iran", "oil", "energy"], "includeIndex": True},
            "webhook": {"url": "https://hooks.slack.com/...", "format": "brief"}
        }).json()

        if result.get("digest", {}).get("findings"):
            high = [f for f in result["digest"]["findings"] if f.get("severity") == "high"]
            if high:
                print(f"HIGH DIVERGENCE: {result['digest']['headline']}")

    time.sleep(6 * 3600)  # Every 6 hours
```

Cost: 3 sources x 4 cycles/day x $0.007 = **$0.084/day** ($2.52/month).

## Response Shape

```json
{
  "ok": true,
  "source": {
    "action": "scrape",
    "success": true,
    "data": { "...raw Firecrawl response..." },
    "markdown": "# Hacker News..."
  },
  "analysis": {
    "model": "google/gemini-2.5-flash",
    "result": { "topics": [...] },
    "usage": { "inputTokens": 1234, "outputTokens": 567, "estimatedCostUsd": 0.002 }
  },
  "markets": {
    "matches": [
      { "title": "US forces enter Iran by April 30", "venue": "polymarket", "price": 67 }
    ],
    "index": { "uncertainty": 22, "geopolitical": 95, "momentum": 0.09 },
    "queriedTopics": ["iran", "oil"]
  },
  "digest": {
    "headline": "HN AI buzz vs market skepticism",
    "brief": "Despite strong sentiment...",
    "tweetable": "Markets say X, crowds say Y...",
    "findings": [
      {
        "topic": "AI regulation",
        "sourceSignal": "HN: 3 posts, 400 comments",
        "marketSignal": "Kalshi: 21c by 2027",
        "divergence": "Sentiment overestimates near-term regulation",
        "divergenceType": "sentiment_gap",
        "severity": "medium",
        "relatedMarkets": [...]
      }
    ]
  },
  "webhook": { "delivered": true, "statusCode": 200 },
  "meta": { "processingTimeMs": 12000, "firecrawlCreditsUsed": 1, "llmCostUsd": 0.007 }
}
```

## Docs & Discovery

| Resource | URL |
|----------|-----|
| API docs (self-describing) | `GET /api/monitor-the-situation` |
| Agent discovery manifest | `GET /api/tools` |
| MCP server | `https://simplefunctions.dev/api/mcp/mcp` |
| LLM reference | `https://simplefunctions.dev/llms.txt` |
| Full reference | `https://simplefunctions.dev/llms-full.txt` |

---

*SimpleFunctions — 9,706 prediction market contracts. Firecrawl-powered scraping. Any LLM. Cross-referenced intelligence.*