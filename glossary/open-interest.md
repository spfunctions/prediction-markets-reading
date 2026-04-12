# Open Interest

**Open interest is the total number of outstanding contracts in a prediction market that have been traded but not yet settled. It measures how many contracts are currently being held by traders.**


## Explanation

## Understanding Open Interest

Open interest counts every contract that exists in the market. When a new buyer and a new seller trade, open interest increases by one contract. When an existing holder sells to another existing holder, open interest stays the same. When two existing holders close their positions against each other, open interest decreases.

### Open Interest vs. Volume

These are often confused:
- **Volume** counts every trade that happens, regardless of whether it creates new contracts or transfers existing ones
- **Open interest** counts only the contracts currently outstanding

A market can have high volume but low open interest if the same contracts are being traded back and forth. Conversely, a market with growing open interest is attracting new capital.

### Why It Matters

Open interest tells you several things:

1. **Market conviction**: Rising open interest with rising prices means new money is betting Yes. Rising open interest with falling prices means new money is betting No.
2. **Liquidity risk**: Low open interest means it may be hard to exit large positions.
3. **Settlement impact**: At settlement, total open interest times price movement equals the total transfer from losers to winners.

### Checking Open Interest

On Kalshi, open interest is displayed on each market page. You can also pull it via `sf market` to see the current number alongside price and volume data.


## Example

Market: KXWTIMAX-26DEC31-T100
"Will WTI oil exceed $100 in 2026?"

Open Interest: 12,450 contracts
Volume (24h):  3,200 contracts
Yes Price:     $0.15

Total capital at risk:
  Yes holders: 12,450 × $0.15 = $1,867.50
  No holders:  12,450 × $0.85 = $10,582.50
  Total locked: $12,450.00 (always = contracts × $1.00)


## CLI

```bash
sf market KXWTIMAX-26DEC31-T100
```


## Related

[volume](volume.md), [liquidity](liquidity.md), [settlement](settlement.md), [prediction-market](prediction-market.md)
