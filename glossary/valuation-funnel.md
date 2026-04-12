# Valuation Funnel

**The valuation funnel is the three-stage hierarchy that turns the universe of 47,000 prediction-market contracts into a single trade: filter by indicator, then read the orderbook, then apply causal reasoning. Each stage feeds the next, each stage is hierarchical (you do not skip ahead), and each stage uses a different cognitive tool — computation at stage 1, mechanical orderbook reading at stage 2, judgment at stage 3.**


## Explanation

## The Three Stages

**Stage 1 — Filter by indicator.** Take the universe (currently ~47,000 active contracts across Kalshi and Polymarket), and apply numerical thresholds on indicators from the stack: implied yield above some floor, cliff risk index above some floor, τ-days inside a tradable window, liquidity availability score above zero. The output is typically 50-200 contracts, depending on how strict the gates are. This stage is pure computation. There is no LLM involvement, no judgment call, no reading of external news. It is the fastest part of the pipeline and runs in seconds via `sf scan` or the `/api/public/screen` endpoint.

**Stage 2 — Read the orderbook.** For each survivor of stage 1, pull the actual orderbook and confirm there is *executable* edge, not just displayed edge. A contract that shows an attractive IY at the displayed mid-price might have 12 cents of bid-ask spread, in which case the IY at the side you can transact at is wildly different. This stage is mechanical — you are looking at depth, spread, and resting volume — but it is not computational, because the question is "can I actually fill this order at this price" and the answer requires looking at the live book. The output is typically 5-20 contracts.

**Stage 3 — Apply causal reasoning.** For each remaining contract, ask the question that no indicator and no orderbook can answer: does the thesis make sense? What is the causal tree that gets you from current state to resolution? Are there known catalysts in the τ window? Is there a confounding event that nobody is pricing in? This stage is judgment, and it is where the LLM (or a human analyst) earns its keep. The output is 1-5 contracts that are worth a position.

## Why the Order Cannot Be Reversed

The most common failure mode in prediction-market trading is putting the LLM at the top of the funnel. "Read all 47K markets and tell me which ones to trade" is a non-starter, both because no LLM has the context window for it and because the LLM has no way to compute IY across the full universe. The numerical filter has to come first, and the LLM has to come last, after the universe has already been narrowed to the contracts where the math says edge exists.

The other common failure mode is skipping stage 2 — running the LLM on the indicator survivors without checking the orderbook. This produces beautiful theses about contracts that turn out to be untradeable at the displayed price, because the spread eats whatever edge the indicator surfaced. Stage 2 is the gate that catches this.

## The Time Allocation

Roughly: stage 1 takes seconds (computational), stage 2 takes minutes per contract (mechanical), stage 3 takes 10-30 minutes per contract (judgment + research). The funnel shape matches the time cost. You want to throw away the most contracts at the cheapest stage, and reserve the expensive thinking for the smallest set.


## Example

A Tuesday morning scan with the default thresholds:

| Stage | Filter | Survivors |
|---|---|:---:|
| Universe | All active Kalshi + Polymarket contracts | 46,800 |
| Stage 1 | IY > 100% AND τ in [5, 60] days AND CRI > 1.0 | 178 |
| Stage 1 (continued) | LAS > 50 (executable book) | 42 |
| Stage 2 | Orderbook spread < 4¢ AND ask depth > 50 contracts | 11 |
| Stage 3 | Thesis defensible AND causal tree complete AND no known confounders | 2 |

The funnel cuts 46,800 down to 2 in three stages. Stage 1 is a single `sf scan --by-iy desc --min-iy 1.0 --tau-min 5 --tau-max 60 --min-cri 1.0 --min-las 50` invocation, runs in about 4 seconds, and returns the 42-contract list. Stage 2 is a manual or semi-automated walkthrough of each contract's book — most of them die here because the displayed mid-price is fictional once you check depth. Stage 3 is the LLM (or a human) writing a thesis for each of the 11 survivors and checking it against the causal tree, news flow, and any cross-venue prices.

The two contracts that come out the bottom are the trades. They might both be wrong, but they are *not* wrong because the math was wrong, the book was thin, or the thesis was sloppy. They are wrong only if the underlying event surprises — which is the only kind of wrongness a prediction-market trader is paid to take.

The compute cost of the funnel is dominated by stage 3 (the LLM call), which is exactly the inverse of the survivor count. That is the funnel working as intended: cheap filtering throws away the noise, expensive thinking focuses on the signal.


## CLI

```bash
sf scan --by-iy desc --min-iy 1.0 --tau-min 5 --tau-max 60 --min-cri 1.0
```


## Related

[expected-edge](expected-edge.md), [implied-yield](implied-yield.md), [cliff-risk-index](cliff-risk-index.md), [liquidity-availability-score](liquidity-availability-score.md), [thesis](thesis.md), [causal-tree](causal-tree.md), [three-data-sources](three-data-sources.md), [edge-detection](edge-detection.md)
