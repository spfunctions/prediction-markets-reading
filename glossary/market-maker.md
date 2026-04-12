# Market Maker

**A market maker is a participant who continuously provides both buy (bid) and sell (ask) orders on a prediction market, earning the spread in exchange for providing liquidity to other traders.**


## Explanation

## Market Making in Prediction Markets

Market makers serve a critical function: they make it possible for other traders to enter and exit positions instantly. Without market makers, you'd have to wait for a specific counterparty who wants the opposite side of your trade.

### How Market Makers Profit

A market maker might post:
- Bid (buy order): 42 cents for "Yes"
- Ask (sell order): 45 cents for "Yes"

The 3-cent gap is the spread. If someone sells to the market maker at 42 cents and someone else buys from them at 45 cents, the market maker earns 3 cents per contract without taking directional risk.

### Market Making on Kalshi vs. Polymarket

**Kalshi** uses a central limit orderbook (CLOB). Market makers post resting limit orders. Kalshi has designated market maker programs with fee incentives.

**Polymarket** uses an automated market maker (AMM) based on a constant-function formula. Liquidity providers deposit funds into pools rather than managing individual orders. The AMM algorithmically adjusts prices based on supply and demand.

### Impact on Traders

Market maker behavior affects your trading:
- **Tight spreads** mean lower costs to enter/exit
- **Deep books** mean you can trade larger sizes without moving the price
- **Thin books** mean your order might cause significant slippage

You can check market maker activity using `sf depth` to see the full orderbook.


## Example

Orderbook for KXRECSSNBER-26:

  BIDS (buyers)         ASKS (sellers)
  100 @ $0.26           50 @ $0.29
  200 @ $0.25           150 @ $0.30
  500 @ $0.24           200 @ $0.31

Spread: $0.29 - $0.26 = $0.03 (3 cents)
Midpoint: ($0.26 + $0.29) / 2 = $0.275

Market makers are likely posting on both sides,
earning ~3 cents per round-trip trade.


## CLI

```bash
sf depth KXRECSSNBER-26
```


## Related

[spread](spread.md), [liquidity](liquidity.md), [orderbook](orderbook.md), [depth](depth.md), [bid-ask](bid-ask.md)
