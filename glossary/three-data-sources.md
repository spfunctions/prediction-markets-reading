# Three Data Sources

**Three Data Sources is the framework axis that says every prediction-market thesis must distinguish reality data (what is actually happening in the world), endogenous data (what the prediction market itself is showing), and opinion data (what other humans are saying about the event). A thesis that uses one source is brittle; a thesis that triangulates all three holds up under contradiction. The most common failure mode is over-reliance on endogenous data — assuming that because the price moved, the underlying thesis has changed.**


## Explanation

## The Three Sources, Defined

**Reality data** is the authoritative record of what is happening in the actual world. BLS jobs reports, Kalshi resolution APIs, weather sensor readings, court filings, election commission counts. This data is slow, expensive to gather, and definitionally correct — it is the ground truth that the prediction market eventually has to settle against. A thesis that ignores reality data is a thesis built on price moves and rumor.

**Endogenous data** is what the prediction market itself shows. Prices, volumes, orderbook depth, position flows, the indicator stack (IY, CRI, EE, LAS, PIV, CVR). This data is fast and cheap, and it is *recursive* — the market reacts to its own price, so endogenous signals contain a feedback loop with no external referent. Endogenous data is the most accessible source and the most seductive trap, because it can produce a coherent-looking thesis with zero external validation.

**Opinion data** is what other humans are saying about the event. X posts, news articles, expert commentary, Discord channels, Substack newsletters, polling aggregators. This data is noisy and fast, and it sometimes leads the market (when an expert source has private information) and sometimes lags it (when commentary is reacting to a price move that has already happened).

Each source has a failure mode: reality data is slow, endogenous data is recursive, opinion data is noisy. A thesis that triangulates all three uses each source to check the failure modes of the other two.

## Why Endogenous-Only Is the Common Trap

A trader who watches only the price chart sees movement and infers cause. "The contract moved from 32 to 38, so the underlying thesis has improved." This is sometimes true and sometimes a hallucination. The price might have moved because:

- A real piece of reality data arrived (correct inference)
- A different price moved and the contagion propagated (no new information)
- A single large taker hit the book without any underlying view (no new information)
- A market-making bot widened its quotes (no new information)

The endogenous-only trader cannot distinguish these four cases. The trader who also pulls reality data and opinion data can: if no real news has hit and no expert is talking about it, the price move is *probably* not a reassessment.

## The Triangulation Discipline

The discipline is to require at least two of the three sources to point in the same direction before acting on a thesis. A signal from endogenous data alone is a question, not an answer. The question is "is this price move backed by reality or opinion?" — and the answer comes from the other two sources.

This is also the reason a prediction-market trader needs to be reading news flow, even though news is noisy. News is the opinion-data input, and it is the cheapest of the three sources to monitor at scale. Reality data is hard to gather at high frequency. Endogenous data is automatic. Opinion data is the swing source, and skipping it means you are working with two sources instead of three.


## Example

A specific morning a few weeks ago. The Kalshi contract for "Will the May Fed meeting cut rates?" was at 32¢ at 9am. By 11am it was at 41¢. The endogenous-data signal is unambiguous: a 9¢ move in two hours, with PIV spiking to 40+ in the same window. Something is happening.

| Source | What it said |
|---|---|
| Endogenous | Price moved 32 → 41, PIV spiked, CRI surged |
| Reality | No new BLS data, no new FOMC minutes, no new statement from any Fed governor |
| Opinion | A widely-followed Fed-watch newsletter had published a piece at 9:15am arguing the dot plot would shift |

The triangulation: endogenous and opinion both pointed in the direction of "thesis improving," reality was silent. The 9¢ move was *probably* a real reassessment driven by the newsletter, not a stale flow. The trade-off question becomes "do I trust the newsletter?" rather than "is the price move noise?" — which is a much sharper question to answer.

If reality had been silent *and* opinion had been silent, the move would have been one source signaling alone, and the right action would have been to fade the move (because endogenous-only signals revert more often than they confirm). The triangulation is what separated the two cases.

The CLI surfaces this by integrating opinion-data feeds via the news-vector API and reality-data feeds via the resolution-endpoint cron. Both are first-class inputs to `sf research <market>`, which returns a thesis pre-populated from all three sources rather than from price alone.


## CLI

```bash
sf research <market-ticker>
```


## Related

[valuation-funnel](valuation-funnel.md), [thesis](thesis.md), [causal-tree](causal-tree.md), [edge-detection](edge-detection.md), [signal](signal.md), [null-as-signal](null-as-signal.md), [expected-edge](expected-edge.md), [cliff-risk-index](cliff-risk-index.md)
