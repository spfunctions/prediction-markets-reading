# Rate Limiting

**Rate limiting restricts how many API requests you can make in a given time window. Kalshi allows about 10 requests per second; SimpleFunctions handles rate limit management automatically.**


## Explanation

## Why Rate Limits Exist

API rate limits protect servers from being overwhelmed and ensure fair access for all users. Every prediction market API has them:

- **Kalshi**: ~10 requests/second for authenticated endpoints
- **Polymarket**: Varies by endpoint, generally 60 requests/minute
- **SimpleFunctions**: Tiered by plan (free: 100/min, pro: 1000/min)

### What Happens When You Hit a Limit

When you exceed the rate limit, the API returns a 429 (Too Many Requests) response with a `Retry-After` header telling you how long to wait.

### How SimpleFunctions Handles Rate Limits

The CLI and API client handle rate limiting automatically:
1. **Request queuing**: If you fire 20 requests, the client queues them and sends at the maximum allowed rate
2. **Backoff**: On a 429 response, the client waits the specified duration before retrying
3. **Batching**: Multiple small requests are combined into batch requests to reduce total request count
4. **Caching**: Recent responses are cached to avoid redundant requests

### Rate Limits and the Heartbeat

The heartbeat engine is designed to operate well within rate limits. A 30-minute heartbeat cycle uses approximately 5-10 API calls per thesis, well within any rate limit tier.

### Monitoring Your Usage

```bash
sf rate-limits
```

Shows your current rate limit usage, remaining quota, and reset time for each API.


## Example

sf rate-limits

API             Limit       Used    Remaining   Resets
SimpleFunctions 1000/min    42      958         in 18s
Kalshi          10/sec      3       7           in 0.3s
Polymarket      60/min      12      48          in 45s

Heartbeat usage (last cycle):
  SimpleFunctions: 8 requests
  Kalshi: 4 requests
  Polymarket: 2 requests
  Total: 14 requests per 30-minute cycle

At this rate, you can run ~70 theses simultaneously
before approaching rate limits.


## CLI

```bash
sf rate-limits
```


## Related

[api-key](api-key.md), [batch-api](batch-api.md), [delta-api](delta-api.md), [heartbeat](heartbeat.md)
