# Piping prediction market signals into your existing trading system

> Three integration patterns for teams that already have infrastructure: cron polling, agent middleware, and thesis-as-filter.

**Category:** guide | **Author:** Patrick Liu | **Reading time:** 11 min

---
You have a trading system. It ingests signals, runs them through models, and produces orders. You want to add prediction market data to that pipeline without rewriting anything.

This guide covers three patterns, each building on the SimpleFunctions public API. All endpoints are unauthenticated, JSON-over-HTTP, and cached server-side (10-minute TTL on `/context`, 60s stale-while-revalidate on thesis detail). You can hit them from a cron job, a Lambda, or `curl`.

## The API surface

Three endpoints matter:

| Endpoint | What it returns | Cache |
|---|---|---|
| `GET /api/public/context` | Edge movers, macro markets, milestones, liquid markets, evaluation signals | 10 min |
| `GET /api/public/theses` | All public theses with confidence, edge count, implied returns | 60s SWR |
| `GET /api/public/thesis/:slug` | Full thesis: causal tree, edges with orderbook data, confidence history, strategies, track record | 60s SWR |

Every market item includes `venue` (`kalshi` or `polymarket`), a `price` in cents (0-100), and where applicable, `spread`, `volume`, and `edge` (the gap between market price and thesis-implied price, also in cents).

---

## Pattern A: Cron job polling context into your database

**Use case:** You want prediction market snapshots in your own Postgres, queryable alongside your existing signals. Your models already read from the DB. You just need a new table and a writer.

### Step 1: Create the table

```sql
CREATE TABLE pm_signals (
  id          SERIAL PRIMARY KEY,
  snapshot_at TIMESTAMPTZ NOT NULL,
  venue       TEXT NOT NULL,          -- 'kalshi' or 'polymarket'
  ticker      TEXT,                    -- Kalshi ticker, nullable
  slug        TEXT,                    -- Polymarket slug, nullable
  title       TEXT NOT NULL,
  price       INT NOT NULL,            -- cents 0-100
  edge        INT,                     -- cents, from thesis edge analysis
  spread      INT,                     -- cents
  volume      BIGINT,
  change_24h  INT,                     -- cents
  signal_type TEXT NOT NULL,           -- 'edge_mover', 'macro', 'liquid', 'milestone'
  thesis_slug TEXT,                    -- which thesis generated this edge
  raw_json    JSONB,                   -- full item for debugging
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_pm_signals_snapshot ON pm_signals (snapshot_at DESC);
CREATE INDEX idx_pm_signals_venue_ticker ON pm_signals (venue, ticker);
CREATE INDEX idx_pm_signals_edge ON pm_signals (edge DESC NULLS LAST);
```

### Step 2: The polling script

```python
#!/usr/bin/env python3
"""pm_ingest.py — Poll SimpleFunctions context API, write to Postgres.
Run via cron: */10 * * * * /usr/bin/python3 /opt/signals/pm_ingest.py
"""
import json
import urllib.request
import psycopg2
from datetime import datetime, timezone

API_URL = "https://simplefunctions.com/api/public/context"
DSN = "postgresql://signals:password@localhost:5432/trading"

def fetch_context():
    req = urllib.request.Request(API_URL, headers={"Accept": "application/json"})
    with urllib.request.urlopen(req, timeout=15) as resp:
        return json.loads(resp.read())

def ingest(data, conn):
    snapshot_at = data["snapshotAt"]
    rows = []

    # Edge movers — thesis-backed signals with computed edge
    for item in data.get("edges", []):
        rows.append((
            snapshot_at, item["venue"], item.get("ticker"),
            item.get("slug"), item["title"], item["price"],
            item.get("edge"), item.get("spread"), item.get("volume"),
            item.get("change24h"), "edge_mover",
            item.get("thesisSlug"), json.dumps(item),
        ))

    # Movers — largest 24h price changes
    for item in data.get("movers", []):
        rows.append((
            snapshot_at, item["venue"], item.get("ticker"),
            item.get("slug"), item["title"], item["price"],
            None, item.get("spread"), item.get("volume"),
            item.get("change24h"), "macro",
            None, json.dumps(item),
        ))

    # Liquid — high volume, tight spread
    for item in data.get("liquid", []):
        rows.append((
            snapshot_at, item["venue"], item.get("ticker"),
            item.get("slug"), item["title"], item["price"],
            None, item.get("spread"), item.get("volume"),
            item.get("change24h"), "liquid",
            None, json.dumps(item),
        ))

    if not rows:
        return 0

    with conn.cursor() as cur:
        cur.executemany("""
            INSERT INTO pm_signals
                (snapshot_at, venue, ticker, slug, title, price,
                 edge, spread, volume, change_24h, signal_type,
                 thesis_slug, raw_json)
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """, rows)
        conn.commit()
    return len(rows)

if __name__ == "__main__":
    data = fetch_context()
    with psycopg2.connect(DSN) as conn:
        count = ingest(data, conn)
        print(f"[{datetime.now(timezone.utc).isoformat()}] Ingested {count} PM signals")
```

The `/api/public/context` response shape looks like this:

```json
{
  "snapshotAt": "2025-03-15T14:30:00.000Z",
  "cachedUntil": "2025-03-15T14:40:00.000Z",
  "edges": [
    {
      "title": "Fed funds rate above 4.5% end of 2025",
      "venue": "kalshi",
      "ticker": "FED-25DEC-T4.5",
      "price": 62,
      "edge": 14,
      "thesisSlug": "fed-higher-for-longer-2025"
    }
  ],
  "movers": [
    {
      "title": "Oil above $90 by June",
      "venue": "polymarket",
      "slug": "oil-90-june",
      "price": 34,
      "change24h": 8,
      "spread": 2,
      "volume": 142000
    }
  ],
  "milestones": [
    {
      "title": "FOMC Rate Decision",
      "category": "Economics",
      "startsAt": "2025-03-19T18:00:00.000Z",
      "hoursUntil": 28,
      "relatedTickers": ["FED-25MAR"]
    }
  ],
  "liquid": [],
  "signals": [
    {
      "thesisTitle": "Fed Higher for Longer",
      "thesisSlug": "fed-higher-for-longer-2025",
      "summary": "Core PCE print reinforces hawkish hold thesis.",
      "confidence": 78,
      "confidenceDelta": 3,
      "evaluatedAt": "2025-03-15T12:00:00.000Z"
    }
  ],
  "traditionalMarkets": [
    { "symbol": "SPY", "name": "S&P 500 ETF", "price": 562.31, "change1d": -4.2, "changePct": -0.74 }
  ]
}
```

Now your existing models can join `pm_signals` against your internal tables. A simple alpha query:

```sql
-- Markets where thesis edge > 10 cents and 24h movement confirms direction
SELECT ticker, title, price, edge, change_24h
FROM pm_signals
WHERE signal_type = 'edge_mover'
  AND ABS(edge) > 10
  AND SIGN(edge) = SIGN(change_24h)
  AND snapshot_at > NOW() - INTERVAL '1 hour'
ORDER BY ABS(edge) DESC;
```

---

## Pattern B: Agent middleware enriching existing signals

**Use case:** You have an event-driven pipeline. When your system detects a signal (earnings surprise, macro print, geopolitical event), you want to enrich it with current prediction market pricing before it reaches your model.

This middleware sits between your signal detector and your model. It intercepts signals, queries SimpleFunctions for relevant market data, and attaches prediction market context.

```python
"""pm_enricher.py — Middleware that enriches trading signals with PM data.
Designed to slot into an existing signal pipeline.
"""
import json
import urllib.request
import urllib.parse
from dataclasses import dataclass, field, asdict
from typing import Optional

BASE_URL = "https://simplefunctions.com/api/public"

@dataclass
class PMContext:
    markets: list = field(default_factory=list)
    theses: list = field(default_factory=list)
    answer: str = ""
    key_factors: list = field(default_factory=list)
    traditional_markets: list = field(default_factory=list)

@dataclass
class EnrichedSignal:
    original: dict
    pm_context: PMContext
    edge_signals: list = field(default_factory=list)
    confidence_delta: Optional[float] = None

def query_pm(query: str, limit: int = 10) -> PMContext:
    """Hit the /api/public/query endpoint with a natural language query."""
    params = urllib.parse.urlencode({"q": query, "limit": limit})
    url = f"{BASE_URL}/query?{params}"
    req = urllib.request.Request(url, headers={"Accept": "application/json"})
    try:
        with urllib.request.urlopen(req, timeout=10) as resp:
            data = json.loads(resp.read())
            return PMContext(
                markets=data.get("markets", []),
                theses=data.get("theses", []),
                answer=data.get("answer", ""),
                key_factors=data.get("keyFactors", []),
                traditional_markets=data.get("traditionalMarkets", []),
            )
    except Exception as e:
        print(f"[PM Enricher] Query failed: {e}")
        return PMContext()

def get_thesis_edges(slug: str) -> list:
    """Fetch edges for a specific thesis."""
    url = f"{BASE_URL}/thesis/{slug}"
    req = urllib.request.Request(url, headers={"Accept": "application/json"})
    try:
        with urllib.request.urlopen(req, timeout=10) as resp:
            data = json.loads(resp.read())
            return data.get("edges", [])
    except Exception:
        return []

def enrich(signal: dict) -> EnrichedSignal:
    """
    Enrich a trading signal with prediction market context.

    Expects signal to have at least:
      - 'event': str describing the event
      - 'tickers': list of relevant tickers (optional)
    """
    query = signal.get("event", "")
    if not query:
        return EnrichedSignal(original=signal, pm_context=PMContext())

    # Query for relevant prediction markets
    pm = query_pm(query)

    # If theses match, pull their edges for direct pricing data
    edge_signals = []
    for thesis in pm.theses:
        slug = thesis.get("slug", "")
        if not slug:
            continue
        edges = get_thesis_edges(slug)
        for edge in edges:
            edge_size = edge.get("edge", 0)
            if abs(edge_size) >= 5:  # Only meaningful edges
                edge_signals.append({
                    "market": edge.get("market", ""),
                    "venue": edge.get("venue", ""),
                    "market_price": edge.get("marketPrice", 0),
                    "thesis_price": edge.get("thesisPrice", 0),
                    "edge": edge_size,
                    "direction": edge.get("direction", ""),
                    "orderbook": edge.get("orderbook"),
                })

    # Compute aggregate confidence delta from matching theses
    deltas = [
        t.get("confidence", 0) - 50
        for t in pm.theses
        if t.get("confidence") is not None
    ]
    avg_delta = sum(deltas) / len(deltas) if deltas else None

    return EnrichedSignal(
        original=signal,
        pm_context=pm,
        edge_signals=edge_signals,
        confidence_delta=avg_delta,
    )

# ── Integration point ────────────────────────────────────────────────────────
# In your existing pipeline, replace:
#   model.predict(signal)
# with:
#   enriched = enrich(signal)
#   model.predict(enriched)

if __name__ == "__main__":
    # Example: your system detected an FOMC-related signal
    signal = {
        "event": "fed rate inflation",
        "source": "macro_scanner",
        "timestamp": "2025-03-15T14:00:00Z",
        "tickers": ["TLT", "VIXY"],
    }
    enriched = enrich(signal)
    print(json.dumps(asdict(enriched), indent=2, default=str))
```

The enriched output gives your model two new dimensions:

1. **`pm_context.markets`** -- what prediction markets are currently pricing for this event.
2. **`edge_signals`** -- where a quantitative thesis disagrees with the market, with orderbook data to assess executability.

Your model can use the `confidence_delta` as a directional bias: positive means thesis-backed conviction above neutral, negative means below.

The `/api/public/query` response shape:

```json
{
  "query": "fed rate inflation",
  "answer": "Markets are pricing a 62% chance the Fed holds above 4.5% through 2025.",
  "markets": [
    {
      "title": "Fed funds rate above 4.5% end of 2025",
      "venue": "kalshi",
      "ticker": "FED-25DEC-T4.5",
      "price": 62,
      "volume": 89400
    }
  ],
  "theses": [
    {
      "title": "Fed Higher for Longer",
      "slug": "fed-higher-for-longer-2025",
      "confidence": 78,
      "edges": 4
    }
  ],
  "content": [
    {
      "type": "opinion",
      "title": "Why the market is wrong about rate cuts",
      "slug": "market-wrong-rate-cuts",
      "snippet": "Core services inflation remains sticky..."
    }
  ],
  "keyFactors": [
    "Core PCE at 2.8% vs 2.5% expected",
    "Fed dot plot median unchanged at 4.75%",
    "Polymarket rate cut probability dropped 12 points in 7 days"
  ],
  "traditionalMarkets": [
    { "symbol": "TLT", "name": "20+ Year Treasury ETF", "price": 87.42, "change1d": -1.3, "changePct": -1.47 }
  ],
  "sources": ["kalshi", "polymarket", "simplefunctions", "databento"]
}
```

---

## Pattern C: Thesis-as-filter with curl + jq

**Use case:** You have a thesis (a directional view on the world). You want to find prediction markets where your thesis creates a pricing edge, then filter for executability. No Python needed -- just shell scripts and your existing cron infrastructure.

### Step 1: Find theses that match your view

```bash
# List all public theses with their confidence and edge count
curl -s https://simplefunctions.com/api/public/theses | \
  jq '.theses[] | {title, slug, confidence, edgeCount, impliedReturn, winRate}'
```

Output:

```json
{
  "title": "Fed Higher for Longer",
  "slug": "fed-higher-for-longer-2025",
  "confidence": 0.78,
  "edgeCount": 4,
  "impliedReturn": 12.3,
  "winRate": 0.75
}
{
  "title": "Hormuz Strait Disruption",
  "slug": "hormuz-disruption-q2-2025",
  "confidence": 0.45,
  "edgeCount": 7,
  "impliedReturn": 24.1,
  "winRate": 0.57
}
```

### Step 2: Pull edges for a thesis and filter for executability

```bash
# Get all edges where |edge| > 8 cents and spread is tight
curl -s https://simplefunctions.com/api/public/thesis/fed-higher-for-longer-2025 | \
  jq '[.edges[] | select((.edge | length) > 8 and .orderbook.spread <= 3)] |
      sort_by(-.edge) |
      .[] | {
        market,
        venue,
        direction,
        marketPrice,
        thesisPrice,
        edge,
        spread: .orderbook.spread,
        liquidityScore: .orderbook.liquidityScore
      }'
```

Output:

```json
{
  "market": "Fed funds rate above 4.5% end of 2025",
  "venue": "kalshi",
  "direction": "YES",
  "marketPrice": 62,
  "thesisPrice": 78,
  "edge": 16,
  "spread": 1,
  "liquidityScore": "high"
}
{
  "market": "No rate cut before July 2025",
  "venue": "polymarket",
  "direction": "YES",
  "marketPrice": 55,
  "thesisPrice": 68,
  "edge": 13,
  "spread": 2,
  "liquidityScore": "medium"
}
```

### Step 3: Monitor confidence changes

Theses are re-evaluated daily. The confidence history tells you when the thesis is strengthening or weakening.

```bash
# Last 5 evaluations: confidence trend + summaries
curl -s https://simplefunctions.com/api/public/thesis/fed-higher-for-longer-2025 | \
  jq '.confidenceHistory[:5][] | {
    confidence: (.confidence * 100 | round),
    delta: (.delta * 100 | round),
    summary,
    evaluatedAt
  }'
```

### Step 4: Wire it into a daily filter script

```bash
#!/bin/bash
# daily_pm_filter.sh — Find actionable edges across all public theses
# Run: 0 8 * * * /opt/signals/daily_pm_filter.sh >> /var/log/pm_filter.log 2>&1

set -euo pipefail
BASE="https://simplefunctions.com/api/public"
MIN_EDGE=8
MAX_SPREAD=3
OUTPUT="/tmp/pm_opportunities_$(date +%Y%m%d).json"

echo "[$(date -u +%FT%TZ)] Scanning theses for edges > ${MIN_EDGE}c, spread <= ${MAX_SPREAD}c"

# Get all thesis slugs
SLUGS=$(curl -s "${BASE}/theses" | jq -r '.theses[].slug')

echo '[]' > "$OUTPUT"

for slug in $SLUGS; do
  echo "  Checking: $slug"
  EDGES=$(curl -s "${BASE}/thesis/${slug}" | \
    jq --arg slug "$slug" \
       --argjson minEdge "$MIN_EDGE" \
       --argjson maxSpread "$MAX_SPREAD" \
    '[.edges[]
      | select(
          ((.edge // 0) | fabs) > $minEdge
          and (.orderbook.spread // 99) <= $maxSpread
        )
      | . + {thesisSlug: $slug}]')

  # Merge into output
  jq -s '.[0] + .[1]' "$OUTPUT" <(echo "$EDGES") > "${OUTPUT}.tmp"
  mv "${OUTPUT}.tmp" "$OUTPUT"

  sleep 1  # Be polite with rate limits
done

COUNT=$(jq length "$OUTPUT")
echo "[$(date -u +%FT%TZ)] Found ${COUNT} opportunities. Saved to ${OUTPUT}"

# Pipe to your existing alert system
if [ "$COUNT" -gt 0 ]; then
  cat "$OUTPUT"  # Replace with: your_alert_cli send --channel=trading < "$OUTPUT"
fi
```

---

## Choosing between patterns

| | Pattern A (Cron + DB) | Pattern B (Middleware) | Pattern C (Shell filter) |
|---|---|---|---|
| **Infra needed** | Postgres + cron | Python process in your pipeline | cron + curl + jq |
| **Latency** | 10-min snapshots | Real-time per signal (~1-2s) | Daily batch |
| **Best for** | Backtesting, historical analysis, SQL joins | Event-driven enrichment, model input | Screening, daily alerts, manual review |
| **Data depth** | Broad market snapshot | Targeted: query-relevant markets + edges | Deep: full thesis edges + orderbook |
| **Complexity** | Low | Medium | Low |

Most teams start with Pattern A (get data flowing), add Pattern C for thesis-specific screening, and graduate to Pattern B when they want real-time enrichment.

## Rate limits and caching

The public API is unauthenticated and rate-limited:

- `/api/public/context`: 10-minute server cache. Polling more frequently than every 10 minutes just returns the cached result.
- `/api/public/query`: 10 requests per minute per IP (cache misses only -- cached queries are free). 10-minute result cache.
- `/api/public/thesis/:slug`: 60-second stale-while-revalidate. You get instant responses from the edge cache, with background refreshes.
- `/api/public/theses`: Same 60-second SWR.

For Pattern A, a 10-minute cron interval is the natural fit. For Pattern B, the 10 req/min limit on `/query` means you should batch or deduplicate signals before enrichment. For Pattern C, once-daily is fine since thesis evaluations happen on a daily cadence.

All data is from live Kalshi and Polymarket feeds, plus Databento for traditional markets (SPY, TLT, VIXY, GLD, USO). Edges are computed by SimpleFunctions' thesis evaluation pipeline, which cross-references a causal model against live prediction market prices.