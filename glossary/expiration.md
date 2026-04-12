# Contract Expiration

**Expiration is the date and time by which a prediction market contract must resolve. If the event has not occurred by expiration, the contract settles to $0 for Yes holders and $1 for No holders.**


## Explanation

## How Expiration Works

Every prediction market contract has an expiration date. This is the deadline for the event to occur. After expiration:
- If the event happened: Yes = $1.00, No = $0.00
- If the event didn't happen: Yes = $0.00, No = $1.00

### Expiration Types

**Fixed date**: The contract expires on a specific date regardless of what happens. Example: "Will there be a government shutdown before July 1, 2026?" expires on July 1.

**Touch/trigger**: Some contracts expire early if the condition is met. Example: "Will Bitcoin exceed $200K at any point in 2026?" settles to $1.00 immediately upon touching $200K.

**Rolling**: Some markets have daily, weekly, or monthly contracts that roll over. Kalshi's weather markets often have daily contracts.

### Time Value and Decay

A contract far from expiration has more "time value" — more time for the event to happen. As expiration approaches:
- Contracts near 50 cents tend to stay volatile
- Contracts near 0 or 100 cents tend to become more certain
- The rate of time decay accelerates in the final days

### Strategy Implications

Long-dated contracts give your thesis more time to play out but tie up capital longer. Short-dated contracts are more capital-efficient but require precise timing. The `sf strategies` command lets you set expiration-aware entry conditions.


## Example

Contract: KXFEDRATE-26JUN "Will Fed cut by 25bp+ at June FOMC?"
Expiration: June 18, 2026 (FOMC meeting date)

Current date: March 2026
Days to expiration: ~90 days
Current price: $0.38

Same contract with 7 days left:
  If strong employment data released → price drops to $0.12
  If CPI surprise lower → price jumps to $0.65

Time value compresses — prices move more violently near expiration.


## Related

[settlement](settlement.md), [binary-contract](binary-contract.md), [event-contract](event-contract.md), [resolution](resolution.md)
