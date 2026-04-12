# Trading Volume

**Trading volume is the total number of contracts traded in a given time period. High volume indicates active interest and typically correlates with tighter spreads and better liquidity.**


## Explanation

## Trading Volume in Prediction Markets

Volume measures trading activity. A market with 10,000 contracts traded in the past 24 hours is more active than one with 50. But volume alone doesn't tell you everything — you need to consider it alongside open interest and spread.

### Volume Patterns

Volume typically spikes around:
- **Catalysts**: Economic data releases (CPI, jobs numbers, GDP), elections, FOMC meetings
- **Price moves**: A sudden price change attracts attention and trading
- **New information**: Major news events drive volume as traders update their positions

### Volume and Liquidity

High volume markets are generally easier to trade because:
- Spreads tend to be tighter (more competition)
- Large orders fill with less slippage
- Market makers are more active

However, volume can be misleading. A market might have a one-time volume spike from a single large trader, then return to low activity.

### Using Volume in SimpleFunctions

When you run `sf scan`, markets are sorted by a combination of edge, volume, and liquidity. High-edge markets with zero volume are flagged because they may be untradeable — the edge exists on paper but you can't execute it.

Volume data is also critical for cross-venue analysis. A contract with edge on Kalshi but zero volume might have a liquid equivalent on Polymarket.


## Example

24-hour volume comparison across markets:

KXRECSSNBER-26         Volume: 8,400  (high activity)
KXCPI-26MAR-T3.5       Volume: 2,100  (moderate)
KXWTIMAX-26DEC31-T150  Volume: 45     (very thin)

The third market has a 12-point edge detected by sf scan,
but with only 45 contracts traded in 24 hours, you'd likely
move the price significantly trying to fill a 100-contract order.


## CLI

```bash
sf scan --sort volume
```


## Related

[open-interest](open-interest.md), [liquidity](liquidity.md), [liquidity-score](liquidity-score.md), [spread](spread.md)
