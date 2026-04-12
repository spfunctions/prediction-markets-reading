# Expected Edge

**Expected Edge is the composite signal that survives all four indicator gates in sequence: implied yield above the floor, cliff risk index above the floor, event overround inside a defensible band, and τ-days inside the tradable window. Expected Edge is not a single number but the *output* of the gate sequence — a contract either passes all four and is "expected edge positive," or it fails one and is excluded. The composition rule is hierarchical because the gates are not independent.**


## Explanation

## Why It Is a Sequence, Not a Sum

The first instinct on combining four indicators is to weight them and produce a single score. That fails for prediction markets in a way worth understanding.

The four indicators measure different things that interact non-linearly. A contract with very high IY and very low CRI is not "average expected edge" — it is a stale pricing that nobody is contesting, which is an entirely different trade than a contract with average IY and average CRI. Averaging the two destroys the information that distinguished them.

The composition rule is therefore *gates in sequence*, not *weights in a sum*. Each gate is binary — either the contract passes the threshold or it does not — and only the contracts that pass all four are surfaced as "expected edge candidates." The order of the gates also matters, because some gates are cheaper to evaluate than others:

```
1. τ-days gate     — cheap, eliminates the pinned and the too-far-out
2. IY gate         — cheap, eliminates the boring
3. CRI gate        — moderate, eliminates the sleepy
4. EE gate         — moderate, eliminates the families with no liquidity
```

A contract that fails the first gate is dropped before any of the later gates compute. This is not a performance optimization in the traditional sense — it is the *correct* hierarchy, because a τ-pinned contract cannot have meaningful IY no matter what the formula says.

## What "Edge" Even Means Here

The word "edge" is doing two jobs in prediction-market trading. Sometimes it means the displayed bid-ask difference between what you can buy and sell at — that is the executable edge. Sometimes it means the difference between your subjective probability and the market's implied probability — that is the thesis edge. Expected Edge in this glossary is neither of those; it is the *composite filter* that turns the universe into a list of contracts where the math says trading is *worth investigating*.

Expected Edge does not say "this contract will make you money." It says "this contract is one of the survivors of all four indicator gates, which means it passed the cheapest possible filter for being interesting, and you should now spend stage 2 and stage 3 of the valuation funnel on it." It is a stage 1 output, not a final answer.

## The Hierarchy Is Not Optional

The most common mistake in expected-edge composition is collapsing the hierarchy: scoring a contract on a weighted average of IY, CRI, EE, and τ, and trading the top 10. This produces a list that is dominated by the indicator with the largest natural variance (usually CRI), and it lets bad scores on one indicator be compensated by good scores on another.

The point of the hierarchy is exactly to prevent that compensation. A contract that is illiquid (LAS = 0) is not a tradable contract no matter how high its IY is. A contract that is pinned (τ → 0) is not a tradable contract no matter how high its CRI is. The gates are independent veto powers. None of them get to be averaged away.


## Example

A scan against the full universe with the default Expected Edge gates:

| Gate | Threshold | Pre-gate | Post-gate |
|---|---|:---:|:---:|
| τ-days | between 5 and 60 days | 46,800 | 12,400 |
| IY | > 100% annualized | 12,400 | 1,820 |
| CRI | > 1.5 | 1,820 | 187 |
| EE (sibling overround) | between −0.05 and +0.05 | 187 | 41 |

The funnel cuts 46,800 contracts down to 41 in four hierarchical gates. Each gate is evaluated only on the survivors of the previous one, which is both faster and more correct than evaluating all four on the full universe.

The 41 survivors are the Expected Edge positive set for this scan. They are not "guaranteed to make money" — they are "passed the cheapest test for being worth a stage-2 orderbook read." From 41, the orderbook stage typically cuts to ~10, and the thesis stage cuts to 1-3. The whole sequence — from 46,800 to a tradable position — runs in roughly 30 minutes of human attention plus a few seconds of compute.

The mistake to avoid: looking at the same data and saying "let me sort the 46,800 contracts by a weighted score of IY × CRI × LAS." That sort would put high-CRI contracts at the top because CRI has the largest natural variance, and most of the top 50 would fail the τ gate (because high CRI clusters near expiry). The hierarchy exists to prevent exactly that failure mode.

A practical CLI invocation: `sf scan --expected-edge`, which applies the four gates in order and returns the survivors. The flag is named for the composition rule, not for any single number.


## CLI

```bash
sf scan --expected-edge
```


## Related

[valuation-funnel](valuation-funnel.md), [implied-yield](implied-yield.md), [cliff-risk-index](cliff-risk-index.md), [event-overround](event-overround.md), [tau-days](tau-days.md), [liquidity-availability-score](liquidity-availability-score.md), [edge](edge.md), [executable-edge](executable-edge.md)
