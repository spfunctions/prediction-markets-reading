# Resolution Criteria

**Resolution criteria are the specific, pre-defined rules that determine how a prediction market contract settles. They specify the data source, measurement method, and exact conditions for a Yes or No outcome.**


## Explanation

## Why Resolution Criteria Matter

Resolution criteria are the most important thing to read before entering a trade. Ambiguous or misunderstood resolution criteria are the #1 source of unexpected losses for new prediction market traders.

### Anatomy of Resolution Criteria

Every well-designed contract specifies:
1. **The data source**: Which official source determines the outcome (e.g., "BLS CPI report")
2. **The measurement**: What exact number matters (e.g., "year-over-year CPI-U, seasonally adjusted")
3. **The threshold**: The cutoff value (e.g., "above 3.0%")
4. **The timing**: When the data must be published (e.g., "as reported on the initial release")
5. **Edge cases**: What happens if data is revised, delayed, or unavailable

### Common Pitfalls

- **Revision risk**: A GDP contract might resolve on the "advance estimate" (published 30 days after quarter end) vs. the "final revision" (published 90 days later). The numbers can differ by 1+ percentage points.
- **Definition differences**: "Recession" could mean two consecutive quarters of negative GDP, or the official NBER declaration, or something else entirely.
- **Time zone ambiguity**: "Will Bitcoin exceed $100K on January 1?" — in which time zone?

### Reading Criteria in the CLI

When you pull up a market with `sf market`, the resolution rules are displayed alongside the price data. Always read them before placing a trade.


## Example

Kalshi contract: KXCPI-26MAR-T3.5
"Will March 2026 CPI exceed 3.5%?"

Resolution criteria:
  Source: Bureau of Labor Statistics (BLS)
  Measure: CPI-U, all items, 12-month percent change
  Report: The initial release (not revised)
  Threshold: Strictly greater than 3.500%

Note: If BLS reports 3.500% exactly, the contract settles NO.
      Only values strictly above 3.500% trigger YES settlement.


## CLI

```bash
sf market KXCPI-26MAR-T3.5
```


## Related

[settlement](settlement.md), [binary-contract](binary-contract.md), [event-contract](event-contract.md), [expiration](expiration.md)
