# Cost Basis

**Cost basis is your average entry price across all purchases of the same contract. If you buy 100 contracts at 28 cents and 100 more at 32 cents, your cost basis is 30 cents.**


## Explanation

## Cost Basis in Prediction Markets

Your cost basis is the weighted average price you paid for your position. It determines your profit or loss when you exit.

### Why Cost Basis Matters

Cost basis is important for:
1. **P&L tracking**: Your unrealized profit is (current price - cost basis) x quantity
2. **Tax reporting**: Capital gains are calculated from cost basis
3. **Exit decisions**: Knowing your breakeven helps you set stop losses and take-profit levels

### Calculating Cost Basis

**Simple average**: (Total cost / Total contracts)

If you build a position over multiple trades:
- Buy 100 at $0.25 = $25.00
- Buy 200 at $0.30 = $60.00
- Buy 100 at $0.35 = $35.00
- Total: 400 contracts for $120.00
- Cost basis: $120 / 400 = $0.30

### The Anchoring Trap

Psychologically, traders anchor to their cost basis and make irrational decisions:
- "I can't sell below my cost basis" (loss aversion)
- "I should add more to lower my cost basis" (averaging down without new information)

The correct question is never "what is my cost basis?" — it's "what is the current edge?" If your thesis says the contract should be at 50 cents and it's at 40 cents, you have 10 points of edge regardless of whether your cost basis is 20 cents or 45 cents.

### Cost Basis in SimpleFunctions

The strategy engine deliberately ignores cost basis when evaluating edge. It compares current market price against thesis-implied price. Your entry price is irrelevant to the agent's decision-making — and that's the point.


## Example

Position build-up on KXRECSSNBER-26:

Trade 1: Buy 100 @ $0.25   Total: $25.00
Trade 2: Buy 150 @ $0.28   Total: $42.00
Trade 3: Buy 50  @ $0.32   Total: $16.00

Position: 300 contracts
Total cost: $83.00
Cost basis: $83.00 / 300 = $0.2767

Current price: $0.34
Unrealized P&L: (0.34 - 0.2767) × 300 = $19.00 (+22.9%)

If contract settles YES:
  Payout: 300 × $1.00 = $300.00
  Profit: $300 - $83 = $217.00 (+261%)


## Related

[position-sizing](position-sizing.md), [stop-loss](stop-loss.md), [take-profit](take-profit.md), [edge](edge.md)
