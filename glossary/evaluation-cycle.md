# Evaluation Cycle

**An evaluation cycle is the automated process by which the agent reviews new signals, updates causal tree probabilities, recalculates edge, and determines whether any strategy conditions are met.**


## Explanation

## The Heartbeat Evaluation Loop

The evaluation cycle is the agent's core loop. It runs on a schedule (the "heartbeat") and on-demand when important signals arrive.

### What Happens in Each Cycle

1. **Signal ingestion**: Collect new signals since the last evaluation — news articles, price changes, user notes, webhook data
2. **Node mapping**: Map each signal to the relevant causal tree node(s)
3. **Probability update**: Adjust node probabilities based on new information
4. **Root recomputation**: Cascade updates through the tree to compute new thesis-implied prices
5. **Edge calculation**: Compare updated thesis prices against current market prices
6. **Strategy evaluation**: Check if any strategy entry/exit/stop conditions are now met
7. **Notification**: Send alerts for significant changes (edge found, confidence shift, strategy trigger)

### Evaluation Frequency

- **Scheduled**: Every 30-60 minutes during market hours
- **Event-triggered**: Immediately when a high-priority signal arrives (e.g., economic data release)
- **Manual**: You can force an evaluation with `sf thesis evaluate`

### What the Agent Produces

Each evaluation generates:
- Updated confidence score and delta (e.g., "confidence changed from 68% to 72%")
- Summary of what changed and why
- Updated edge snapshot across all linked markets
- Any triggered strategy actions

### Evaluation History

All evaluations are stored as an append-only log. You can review past evaluations to understand how your thesis evolved over time and whether the agent's updates were accurate.


## Example

sf thesis evaluate

Evaluation #47 for "US Recession by 2026"
  Signals processed: 3
    • BLS: February unemployment rate 4.8% (was 4.5%)
    • Reuters: Consumer confidence index drops to 88.3
    • Kalshi: KXRECSSNBER-26 moved from 28¢ to 30¢

  Node updates:
    n1.1: Unemployment > 5%     35% → 42%  (BLS data)
    n2.2: Consumer confidence < 80   48% → 52%  (confidence data)

  Thesis confidence: 68% → 72% (+4pt)
  Root probability:  38% → 42% (+4pt)

  Edge changes:
    KXRECSSNBER-26: +10pt → +12pt  (market moved +2¢ but thesis moved +4pt)

  Strategy check:
    S-001: Entry condition met (edge > 10pt, price < 32¢) ✓
    Action: Recommend buy 50 contracts at $0.30


## CLI

```bash
sf thesis evaluate
```


## Related

[heartbeat](heartbeat.md), [signal](signal.md), [causal-tree](causal-tree.md), [thesis](thesis.md), [edge-detection](edge-detection.md)
