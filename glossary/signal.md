# Signal

**A signal is any piece of new information that could affect a thesis — a news article, a price movement, a data release, a user note, or an external webhook event. The agent evaluates signals against the causal tree.**


## Explanation

## Signals in the SimpleFunctions System

A signal is the raw input to the evaluation cycle. Think of signals as the information stream that the agent processes to keep your thesis current.

### Types of Signals

1. **News signals**: Articles and reports picked up by the search service (Tavily). Automatically generated.
2. **Price signals**: Significant price movements in linked markets. Automatically detected.
3. **User notes**: Information you manually feed to the agent ("I spoke with an industry insider who said...")
4. **Webhook signals**: External data pushed to the agent via webhooks (automated data feeds, custom monitors)

### Signal Processing

Not all signals are equal. The agent evaluates each signal through this pipeline:

1. **Deduplication**: Is this the same information we already processed? (Uses idempotency keys)
2. **Node mapping**: Which causal tree node does this affect? If none, the signal is noise.
3. **Impact assessment**: Does this signal change the node probability? By how much?
4. **Evaluation trigger**: If the impact is significant (>2% probability change), trigger a full evaluation cycle.

### Signal Quality

High-quality signals:
- Come from authoritative sources (official data, established news outlets)
- Map cleanly to a specific causal node
- Provide quantitative information (numbers, not just sentiment)

Low-quality signals:
- Are speculative or opinion-based
- Don't map to any node (noise)
- Duplicate information already processed


## Example

sf signals --thesis recession-2026 --last 24h

Recent signals:
ID      Type       Source    Node    Impact    Status
sig-01  news       tavily   n1.1    +7pt      consumed
sig-02  price      kalshi   —       —         consumed
sig-03  news       tavily   n2.2    +4pt      consumed
sig-04  news       tavily   —       noise     filtered
sig-05  user_note  manual   n3.1    +2pt      consumed
sig-06  news       tavily   n1.1    dup       filtered

6 signals received, 4 consumed, 2 filtered (1 noise, 1 duplicate)


## CLI

```bash
sf signals
```


## Related

[evaluation-cycle](evaluation-cycle.md), [heartbeat](heartbeat.md), [causal-tree](causal-tree.md), [webhook](webhook.md)
