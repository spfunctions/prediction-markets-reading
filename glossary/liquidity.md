# Liquidity

**Liquidity is the ease with which you can buy or sell a prediction market contract without significantly affecting its price. Liquid markets have tight spreads, deep orderbooks, and high trading volume.**


## Explanation

## Liquidity in Prediction Markets

Liquidity is one of the most important practical considerations in prediction market trading. A market can have the best edge in the world, but if it's illiquid, you can't capture it.

### Components of Liquidity

Liquidity has three dimensions:
1. **Spread**: How much it costs to cross from buy to sell
2. **Depth**: How many contracts you can trade at reasonable prices
3. **Resilience**: How quickly the orderbook replenishes after a large trade

### Liquidity Varies Widely

On Kalshi, top political and economic markets (presidential elections, CPI, Fed rates) might have $100K+ of depth. Niche markets (specific weather events, obscure economic data) might have $500 or less. The difference is enormous.

### Why Prediction Markets Are Less Liquid Than Traditional Markets

Prediction markets are younger, have fewer participants, and often have regulatory restrictions that limit market maker activity. This creates both a challenge and an opportunity:
- **Challenge**: You can't always trade the size you want
- **Opportunity**: Illiquidity itself creates edge. Prices in illiquid markets are less efficient, meaning they deviate further from true probabilities.

### Checking Liquidity

The CLI provides liquidity data in several places:
- `sf scan` includes a liquidity score (A-D) for each market
- `sf depth` shows the full orderbook
- `sf market` shows 24h volume and open interest


## Example

Liquidity comparison:

Market                  Spread  5-level Depth  24h Volume  Score
KXPRESWIN-28            $0.01   45,000         12,400      A
KXRECSSNBER-26          $0.03   2,500          3,200       A
KXCPI-26MAR-T3.5        $0.03   800            1,100       B
KXWTIMAX-26DEC31-T150   $0.08   80             45          D

Score A: Trade up to 500 contracts with minimal impact
Score B: Trade up to 200 contracts with moderate impact
Score C: Trade up to 50 contracts
Score D: Effectively untradeable at size


## CLI

```bash
sf scan
```


## Related

[depth](depth.md), [spread](spread.md), [volume](volume.md), [liquidity-score](liquidity-score.md), [market-maker](market-maker.md)
