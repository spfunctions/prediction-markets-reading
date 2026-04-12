# Candlestick Data

**Candlestick data represents price movement over a time period using four values: open, high, low, and close (OHLC). In prediction markets, candlesticks show how contract prices evolve over hours or days.**


## Explanation

## Candlestick Charts for Prediction Markets

Candlestick charts are borrowed from traditional finance. Each "candle" represents a time period and shows:
- **Open**: The price at the start of the period
- **Close**: The price at the end of the period
- **High**: The highest price during the period
- **Low**: The lowest price during the period

### Reading Candlesticks

- **Green/bullish candle**: Close > Open (price went up)
- **Red/bearish candle**: Close < Open (price went down)
- **Long wicks**: The market tested extreme prices but didn't hold them
- **Small body**: Indecision — opening and closing prices are close

### Candlesticks in Prediction Markets

Prediction market candlesticks are useful for:
1. **Identifying overreactions**: Long lower wicks after news events suggest panic selling that was bought up
2. **Volume confirmation**: Big candles with high volume confirm genuine moves; big candles with low volume suggest manipulation
3. **Pattern recognition**: While traditional candlestick patterns (doji, hammer, etc.) are less reliable in prediction markets, basic trend analysis works

### Accessing Candlestick Data

Kalshi provides candlestick data via their API. SimpleFunctions normalizes this into a consistent format:

```bash
sf candles KXRECSSNBER-26 --period 1h --count 24
```

This returns the last 24 hours of hourly candles. Useful for understanding recent price action before entering a trade.


## Example

sf candles KXRECSSNBER-26 --period 4h --count 6

Time          Open   High   Low    Close  Volume
Mar 19 00:00  0.28   0.29   0.27   0.28   420
Mar 19 04:00  0.28   0.28   0.27   0.27   180
Mar 19 08:00  0.27   0.31   0.27   0.30   1,850  ← News event
Mar 19 12:00  0.30   0.31   0.29   0.30   620
Mar 19 16:00  0.30   0.30   0.28   0.29   340
Mar 19 20:00  0.29   0.29   0.28   0.29   210

The 08:00 candle shows high volume with a big range
(0.27 to 0.31) — likely triggered by economic data release.
The close at 0.30 (upper half of range) suggests buyers won.


## CLI

```bash
sf candles
```


## Related

[volume](volume.md), [orderbook](orderbook.md), [mean-reversion](mean-reversion.md)
