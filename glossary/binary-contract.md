# Binary Contract

**A binary contract is a financial instrument that pays out a fixed amount (typically $1) if a specified condition is met, and nothing otherwise. Prediction markets are built on binary contracts.**


## Explanation

## Binary Contracts in Prediction Markets

A binary contract has exactly two outcomes: Yes or No. There is no in-between. The contract either settles at $1.00 (Yes wins) or $0.00 (No wins). This simplicity is what makes prediction markets powerful — every contract maps directly to a probability.

### Structure

On Kalshi, binary contracts trade in cents from $0.01 to $0.99. When you buy a "Yes" contract at 65 cents, you're simultaneously selling the "No" side at 35 cents (since Yes + No always equals $1.00).

### Types of Binary Contracts

- **Event contracts**: Will X happen? (e.g., "Will there be a government shutdown?")
- **Range contracts**: Will X be above/below Y? (e.g., "Will WTI crude oil exceed $100/barrel?")
- **Date-bound contracts**: Will X happen before date Y?

### Multi-Outcome Events

Some events have more than two outcomes (e.g., "Who will win the presidential nomination?"). These are structured as multiple binary contracts — one per candidate — where the set of Yes prices should sum to approximately $1.00. On Kalshi, these appear as event groups with tickers like KXDEM-26.

### Key Properties

Binary contracts have bounded risk. You can never lose more than you paid for the contract, and your maximum profit is $1 minus your entry price. This makes position sizing straightforward.


## Example

Kalshi contract: KXRECSSNBER-26 "Will NBER declare a recession by Dec 31, 2026?"

Buy 50 Yes contracts at $0.28
  Max loss:   50 x $0.28 = $14.00
  Max profit: 50 x $0.72 = $36.00

The risk/reward ratio is 36:14, or about 2.6:1.
This is attractive IF you believe the true probability exceeds 28%.


## CLI

```bash
sf market KXRECSSNBER-26
```


## Related

[prediction-market](prediction-market.md), [settlement](settlement.md), [event-contract](event-contract.md), [outcome-probability](outcome-probability.md)
