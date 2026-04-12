# Prediction Market

**A prediction market is an exchange where participants trade contracts whose payoff depends on the outcome of future events. The price of a contract reflects the crowd's aggregate estimate of the event's probability.**


## Explanation

## How Prediction Markets Work

A prediction market lets you buy and sell contracts tied to real-world outcomes. If you buy a "Yes" contract on "Will the Fed cut rates in June?" at 35 cents, you pay $0.35. If the Fed does cut rates, the contract settles at $1.00 and you profit $0.65. If not, the contract settles at $0.00 and you lose your $0.35.

### Why Prices Equal Probabilities

A contract trading at 35 cents implies a 35% probability. This works because rational traders won't pay more than a contract's expected value. If you think there's a 50% chance the Fed cuts, a contract at 35 cents is a bargain — you'd expect to make $0.50 on average but only pay $0.35.

### Major Prediction Markets

The two largest prediction markets are:

- **Kalshi** — CFTC-regulated, US-based, uses an orderbook model. Contracts on economics, weather, politics, and more. Tickers like KXRECSSNBER-26 for "Will NBER declare a recession by 2026?"
- **Polymarket** — Crypto-based, larger liquidity on political markets. Uses an automated market maker (AMM) model for most markets.

### Why They Matter

Academic research consistently shows prediction markets outperform polls, expert panels, and statistical models at forecasting. They aggregate information from diverse sources in real-time, weighted by how much money people are willing to stake on their beliefs.


## Example

Contract: "Will GDP growth exceed 3% in Q2 2026?"
Current price: 42 cents (Yes)

If you buy 100 Yes contracts at $0.42 each:
  Cost: $42.00
  If GDP > 3%:  Payout = $100, Profit = +$58
  If GDP <= 3%: Payout = $0,   Loss = -$42

The market implies a 42% probability of GDP exceeding 3%.


## CLI

```bash
sf markets
```


## Related

[binary-contract](binary-contract.md), [outcome-probability](outcome-probability.md), [settlement](settlement.md), [market-maker](market-maker.md)
