# Monitoring the Situation: From Passive APIs to Proactive Intelligence

> The $886M web scraping industry meets 9,706 prediction market contracts. Nobody is building the cross-reference layer. Until now.

**Category:** markets | **Author:** SimpleFunctions Research | **Reading time:** 10 min | **Published:** 2026-04-02

---
The $886 million web scraping industry is about to collide with the $4.2 billion prediction market ecosystem. And nobody is building the bridge.

Right now, 9,706 prediction market contracts across Kalshi and Polymarket are pricing the future of the world in real time — Iran war probability, Fed rate decisions, AI regulation timelines, oil prices. Meanwhile, every 15 minutes, thousands of agents scrape the web for news, data, and signals. But these two streams of information almost never meet.

That changes today.

## The Problem: Three Signals, Zero Cross-Reference

There are three ways humans form beliefs about the future:

**Belief** — what people bet on with real money. Polymarket prices Iran war probability at 67%. This isn't a poll; it's $56 million in committed capital.

**Sentiment** — what people say publicly. Hacker News, Twitter, Reddit, cable news. 78% of HN commenters on Iran threads expect military escalation. Vocal, passionate, but cheap talk.

**Action** — what institutions actually do. SEC filings, government procurement, carrier rerouting, corporate announcements. The CIA doesn't tweet, but defense contractors file contracts.

**The divergence between these three signals is the most valuable information in the world.** When markets price Iran war at 67% but oil is *down* 2.7%, something is mispriced. When sentiment screams escalation but ceasefire contracts are climbing, someone knows something the crowd doesn't.

Today, nobody synthesizes these three signals at scale. Paradox Intelligence [describes the methodology](https://www.paradoxintelligence.com/blog/how-to-use-alternative-data-predict-polymarket-outcomes) — track behavioral drivers, compare to contract prices, look for divergence — but doesn't offer a product. Dome and PMXT aggregate market data. Firecrawl and Apify scrape the web. SignalHub monitors changes. But nobody connects the dots.

## Why Monitoring Needs Intelligence

The current monitoring landscape is broken in predictable ways.

**Google Alerts** delivers [roughly 10% relevant results, missing 40% of important news](https://getsignalhub.com/blog/the-best-web-monitoring-tools-in-2026-from-google-alerts-to-ai-agents). It's the default option and it's terrible.

**RSS readers** (Feedly, Folo) are great at ingestion but terrible at distribution. Feedly locks Slack integration behind $1,600/month enterprise pricing. Folo has [explicitly rejected webhook support](https://github.com/RSSNext/Folo/issues/3178).

**Page change detectors** (Visualping, Changedetection.io) tell you *that* something changed but not *what it means*. A price moved. A paragraph was edited. So what?

**AI search agents** (Yutori, Ancher) remove user control entirely — you can't specify sources, you can't define extraction schemas, you can't cross-reference with structured data.

The fundamental gap: no tool does **source selection + AI extraction + structured cross-reference + multi-channel push** well at a reasonable price.

## The Three-Layer Architecture

SimpleFunctions' approach to monitoring stacks three layers:

### Layer 1: Universal Source (Firecrawl)

Any URL. Any format. Full Firecrawl passthrough — scrape, crawl, search, map, extract with custom JSON schemas, batch operations. The agent decides what to monitor and how to extract it.

```bash
# Scrape HN front page
curl -X POST simplefunctions.dev/api/monitor-the-situation \
  -d '{"source": {"action": "scrape", "url": "https://news.ycombinator.com"}}'

# Search the web for breaking news
curl -X POST simplefunctions.dev/api/monitor-the-situation \
  -d '{"source": {"action": "search", "query": "iran oil sanctions latest"}}'

# Extract structured data with a custom schema
curl -X POST simplefunctions.dev/api/monitor-the-situation \
  -d '{"source": {"action": "extract", "url": "https://...", "options": {"schema": {"type": "object", "properties": {"claims": {"type": "array"}}}}}}'
```

### Layer 2: Configurable Analysis (Any LLM)

The agent picks the model. Google Gemini Flash for cheap extraction ($0.15/M tokens). Claude Sonnet for nuanced analysis ($3/M). DeepSeek V3.2 for budget bulk processing ($0.28/M). Custom prompts. Custom output schemas.

```json
{
  "analysis": {
    "enabled": true,
    "model": "google/gemini-2.5-flash",
    "prompt": "Extract key claims about Iran sanctions",
    "schema": {
      "type": "object",
      "properties": {
        "claims": {"type": "array", "items": {"type": "string"}},
        "sentiment": {"type": "string", "enum": ["hawkish", "dovish", "neutral"]}
      }
    }
  }
}
```

### Layer 3: Market Cross-Reference (9,706 Contracts)

This is what nobody else can do. For each topic, SimpleFunctions searches across Kalshi and Polymarket, pulls live contract prices, and generates divergence analysis. The SF Index provides macro context: uncertainty (22/100), geopolitical risk (95/100), market momentum.

```json
{
  "enrich": {
    "enabled": true,
    "topics": ["iran", "oil", "energy"],
    "includeIndex": true
  }
}
```

The output isn't a list of markets. It's a **divergence digest** — where the source content disagrees with market prices, what type of divergence it is (price lag, sentiment gap, information asymmetry), and how severe it is.

## The Venezuela Precedent

In January 2026, Polymarket traders priced Maduro's regime collapse hours before any major news outlet reported it. An anonymous trader placed bets worth $400K before US special forces descended, [turning a 376% profit](https://www.coindesk.com/tech/2026/03/15/ai-agents-are-quietly-rewriting-prediction-market-trading). The "Maduro Trade" generated $56.6M in volume.

This is what cross-referenced intelligence looks like in practice. News sentiment was still debating whether intervention was likely. Government actions (troop movements, diplomatic signals) were visible to insiders. And the market priced it in real time — because prediction markets are where belief, sentiment, and action converge into a single number.

SimpleFunctions makes this convergence accessible to every agent, for every topic, at any time.

## The Proactive Push

ChatGPT Pulse tried proactive intelligence in September 2025 — and [OpenAI declared "Code Red" three months later](https://ttms.com/chatgpt-pulse-how-proactive-ai-briefings-accelerate-enterprise-digital-transformation/), pausing the feature. Google CC launched daily AI briefings. Meta leaked plans to message users unprompted.

They all got the UX wrong because they tried to be everything to everyone.

The monitoring paradigm works when it's **narrow-domain + high-stakes + cross-referenced**. A geopolitical analyst doesn't want a general AI summary. They want: "Iran war markets moved 5% today, here's what prediction markets are pricing that news coverage hasn't absorbed yet, and here's where the divergence is actionable."

That's what monitor-the-situation delivers. Not a summary. A **divergence digest**.

## Cost: $0.007 Per Intelligence Cycle

| Component | Cost | Notes |
|-----------|------|-------|
| Firecrawl scrape | 1 credit | Free 500/month |
| LLM extraction | ~$0.002 | Gemini Flash |
| Market scan | $0 | Internal API |
| LLM digest | ~$0.005 | Gemini Flash |
| Webhook push | $0 | Built-in |
| **Total** | **~$0.007** | |

At this cost, an agent can monitor 100 sources every 6 hours for less than $5/month. That's 400 intelligence cycles per day, each cross-referenced against 9,706 prediction market contracts.

## Try It Now

No auth needed. Paste any text and get prediction market cross-reference:

```bash
curl -X POST https://simplefunctions.dev/api/monitor-the-situation/enrich \
  -H "Content-Type: application/json" \
  -d '{"content": "Your text here...", "topics": ["iran", "oil"]}'
```

Or use MCP:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

Then ask: "monitor the situation on iran oil" — the agent will scrape, analyze, cross-reference, and tell you where reality diverges from what markets are pricing.

---

*Built on SimpleFunctions — 9,706 prediction market contracts across Kalshi and Polymarket. Updated every 15 minutes at [simplefunctions.dev](https://simplefunctions.dev).*