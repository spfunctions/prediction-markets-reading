# Settlement

**Settlement is the process by which a prediction market contract is resolved to its final value — either $1.00 (Yes) or $0.00 (No) — once the outcome of the underlying event is determined.**


## Explanation

## How Settlement Works

When a prediction market contract reaches its resolution date or its triggering event occurs, the exchange settles the contract. All "Yes" holders receive $1.00 per contract if the event happened, or $0.00 if it didn't. "No" holders receive the opposite.

### Settlement on Kalshi

Kalshi uses official data sources for settlement. For example:
- **Economic data contracts** settle based on official government releases (BLS, BEA, Census)
- **Weather contracts** settle based on NOAA data
- **Political contracts** settle based on official election results or government actions

Settlement typically happens within 24 hours of the resolution source publishing data. Kalshi specifies the exact resolution source in each contract's rules.

### Settlement on Polymarket

Polymarket uses a decentralized oracle system (UMA) for settlement. Token holders vote on the outcome, with economic incentives to vote honestly. This process can take 2-4 days for disputed outcomes.

### Early Settlement

Some contracts can settle early if the outcome becomes certain before the expiration date. For example, a contract on "Will the S&P 500 close above 5,000 at any point in 2026?" would settle immediately to $1.00 the first time it crosses 5,000, even if the expiration is December 31.

### Important: Read the Rules

Different contracts have different settlement criteria. Always check the resolution source and exact conditions before trading. A contract about "GDP growth" might settle on the advance estimate, the second estimate, or the final revision — each gives a different number.


## Example

Contract: KXWTIMAX-26DEC31-T100
"Will WTI crude oil exceed $100/barrel at any point in 2026?"

Resolution source: CME WTI Crude Oil front-month settle price
Settlement: Immediate to $1.00 if WTI touches $100 at any point.
            Settles to $0.00 on Dec 31, 2026 if it never touches $100.

If you hold 200 Yes contracts and WTI hits $100.05:
  Payout: 200 × $1.00 = $200.00
  Settlement is automatic — no action needed.


## Related

[binary-contract](binary-contract.md), [resolution](resolution.md), [expiration](expiration.md), [prediction-market](prediction-market.md)
