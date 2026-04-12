# Batch API

**A batch API endpoint accepts multiple requests in a single HTTP call, reducing overhead and latency. SimpleFunctions supports batch market queries, batch edge calculations, and batch price checks.**


## Explanation

## Why Batch APIs?

When you need data on 50 markets, making 50 separate API calls is slow and wasteful. A batch API lets you send all 50 requests in one call and get all 50 responses back at once.

### Benefits of Batching

1. **Reduced latency**: One round-trip instead of fifty
2. **Lower rate limit usage**: One request instead of fifty against your quota
3. **Atomic responses**: All data comes from the same point in time
4. **Connection efficiency**: Reuses a single HTTP connection

### Batch Endpoints in SimpleFunctions

- `POST /api/batch/markets` — Get prices for multiple markets
- `POST /api/batch/edges` — Calculate edge for multiple thesis-market pairs
- `POST /api/batch/depth` — Get orderbook snapshots for multiple markets

### Usage Pattern

```json
POST /api/batch/markets
{
  "markets": [
    "KXRECSSNBER-26",
    "KXCPI-26MAR-T3.5",
    "KXGDP-26Q2-T2.0"
  ]
}
```

Response returns data for all three markets in a single object.

### CLI Batch Operations

The CLI uses batch APIs automatically. When you run `sf scan` or `sf edges`, it batches market queries behind the scenes for optimal performance. A scan that would take 30 seconds with individual requests completes in 2-3 seconds with batching.


## Example

# CLI batches automatically:
sf scan --category economics
# → Internally batches 200+ market queries into 4 batch requests
# → Returns results in ~2 seconds

# Direct API usage:
POST /api/batch/markets
Body: {
  "markets": ["KXRECSSNBER-26", "KXCPI-26MAR-T3.5", "KXGDP-26Q2-T2.0"]
}

Response: {
  "results": [
    { "market": "KXRECSSNBER-26", "bid": 0.27, "ask": 0.30, "volume": 3200 },
    { "market": "KXCPI-26MAR-T3.5", "bid": 0.50, "ask": 0.53, "volume": 1100 },
    { "market": "KXGDP-26Q2-T2.0", "bid": 0.62, "ask": 0.65, "volume": 890 }
  ],
  "timestamp": "2026-03-19T14:30:00Z"
}


## Related

[delta-api](delta-api.md), [rate-limit](rate-limit.md), [api-key](api-key.md)
