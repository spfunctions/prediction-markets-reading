# Mean Reversion

**Mean reversion is the tendency for prediction market prices to return toward their long-term average after temporary spikes or drops caused by overreaction to news events.**


## Explanation

## Mean Reversion in Prediction Markets

When a news event causes a prediction market contract to spike from 30 cents to 50 cents in minutes, does it stay at 50? Often, no. The price frequently reverts partially — settling at perhaps 38-42 cents as the market digests the information more carefully.

### Why Prices Overreact

1. **Emotional trading**: Headlines trigger fear or euphoria, pushing prices past fair value
2. **Thin liquidity**: A few aggressive market orders in a thin orderbook can move prices dramatically
3. **Information ambiguity**: Initial reports are often incomplete or misleading

### Mean Reversion Patterns

- **News spikes**: Prices jump 10-20 points on a headline, then revert 30-50% of the move within hours
- **Data releases**: Economic data (CPI, jobs) causes initial overreaction, followed by a more measured repricing
- **Weekend gaps**: Markets sometimes gap on Monday open and partially revert by Tuesday

### Trading Mean Reversion

This is a counter-trend strategy:
1. Wait for a sharp price move (>10 points in <1 hour)
2. Assess whether the move is justified by your causal tree
3. If the tree says the move is larger than warranted, fade the overreaction
4. Set tight stops in case the move is genuine

### Risks

Mean reversion is not guaranteed. Some price moves are fully justified — and fading them loses money. The causal tree is your guide: if the news genuinely changes a node probability by the same magnitude as the price move, it's not overreaction and you shouldn't fade it.


## Example

KXRECSSNBER-26 price action after strong jobs report:

Time       Price   Event
9:30 AM    $0.32   Pre-report price
8:30 AM    $0.22   Jobs report: +350K (much stronger than expected)
9:00 AM    $0.20   Panic selling — "no recession possible"
10:30 AM   $0.25   Partial reversion as market digests
12:00 PM   $0.27   Further reversion
EOD        $0.26   Settles ~6 cents below pre-report

Your causal tree analysis:
  Jobs report affects n1.1 (unemployment) → decrease by 5pt
  This should move root from 38% to 35% → contract should be ~$0.28
  Market at $0.20 was an overreaction by ~8 cents
  Buying at $0.22 captured ~5 cents of mean reversion


## Related

[edge](edge.md), [what-if-analysis](what-if-analysis.md), [signal](signal.md), [spread](spread.md)
