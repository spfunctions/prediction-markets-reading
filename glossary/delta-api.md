# Delta API

**The Delta API is a lightweight polling endpoint that returns only data that has changed since your last request. It minimizes bandwidth and processing by sending incremental updates rather than full snapshots.**


## Explanation

## Why Delta APIs?

When monitoring hundreds of prediction market contracts, fetching complete snapshots every 30 seconds is wasteful. Most prices haven't changed. The Delta API solves this by returning only what's different.

### How It Works

1. You make an initial request and receive a full snapshot plus a cursor (timestamp or sequence number)
2. Subsequent requests include the cursor: `GET /api/delta?since=cursor`
3. The API returns only markets where price, volume, or status changed since that cursor
4. You update your local state with the deltas

### Benefits

- **Lower bandwidth**: A delta response for 500 markets might be 2KB instead of 200KB
- **Faster processing**: Only process changed data
- **Lower rate limit usage**: Fewer heavy requests
- **Real-time feel**: Poll every 5 seconds without performance issues

### Delta API in SimpleFunctions

The heartbeat engine uses the Delta API internally to monitor market prices efficiently. Instead of fetching every market's full orderbook on each heartbeat, it pulls deltas and only re-evaluates theses when relevant prices change.

### For API Users

If you're building custom tooling on top of SimpleFunctions, the Delta API is the recommended way to stay synchronized. It's especially useful for dashboards, monitoring scripts, and external alert systems.


## Example

# Initial full snapshot
GET /api/delta
→ { markets: [...500 markets...], cursor: "2026-03-19T14:00:00Z" }

# 30 seconds later, poll for changes
GET /api/delta?since=2026-03-19T14:00:00Z
→ {
    changes: [
      { market: "KXRECSSNBER-26", price: 0.29, volume24h: 3250 },
      { market: "KXCPI-26MAR-T3.5", price: 0.51, volume24h: 1150 }
    ],
    cursor: "2026-03-19T14:00:30Z",
    unchanged: 498
  }

Only 2 markets changed — saved 99.6% bandwidth.


## Related

[api-key](api-key.md), [rate-limit](rate-limit.md), [heartbeat](heartbeat.md), [batch-api](batch-api.md)
