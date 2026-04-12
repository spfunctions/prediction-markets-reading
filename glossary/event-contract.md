# Event Contract

**An event contract is a type of binary contract tied to a specific real-world event with a defined resolution date. Unlike perpetual financial instruments, event contracts have a built-in expiration.**


## Explanation

## Event Contracts Explained

Event contracts are the basic building blocks of prediction markets. Each contract is tied to a question about the world: "Will X happen by date Y?" The contract has a start date (when trading opens) and an expiration date (when the event must occur for Yes to win).

### How They Differ from Traditional Finance

Traditional options also expire, but event contracts are fundamentally different:
- **Binary payout**: Event contracts pay $1 or $0. Options pay the difference between strike and settlement.
- **No underlying asset**: Event contracts reference real-world events, not tradeable securities.
- **Fixed risk**: You can never lose more than you paid. No margin calls.

### Event Groups

Complex events are often broken into groups. For example, Kalshi might structure a presidential election as:

- KXDEM-28-HARRIS: "Will Kamala Harris win the 2028 Democratic nomination?"
- KXDEM-28-NEWSOM: "Will Gavin Newsom win the 2028 Democratic nomination?"
- KXDEM-28-PRITZKER: "Will J.B. Pritzker win the 2028 Democratic nomination?"

The sum of all Yes prices within a group should equal approximately $1.00 (minus spread costs).

### Time Decay

As the expiration date approaches, contracts tend to move toward either $0 or $1. A contract at 50 cents with 6 months left might still be at 50 cents, but with 1 day left it will likely be near $0.05 or $0.95. This is similar to how options behave, though the dynamics are different since there's no underlying volatility surface.


## Example

Kalshi event group: KXFEDRATE-26JUN
"What will the Fed funds rate target be after the June 2026 FOMC meeting?"

Contracts within the group:
  4.25-4.50%  → $0.08  (8% probability)
  4.00-4.25%  → $0.25  (25% probability)
  3.75-4.00%  → $0.38  (38% probability)
  3.50-3.75%  → $0.22  (22% probability)
  Below 3.50% → $0.07  (7% probability)

Sum: $1.00 (must always sum to ~$1.00)


## CLI

```bash
sf event KXFEDRATE-26JUN
```


## Related

[binary-contract](binary-contract.md), [prediction-market](prediction-market.md), [settlement](settlement.md), [expiration](expiration.md)
