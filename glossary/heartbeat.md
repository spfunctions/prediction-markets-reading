# Heartbeat

**The heartbeat is the automated monitoring engine that periodically scans for new information, runs evaluation cycles, and triggers strategy actions. It keeps your thesis alive and responsive 24/7.**


## Explanation

## The Heartbeat Engine

The heartbeat is what makes SimpleFunctions an agent rather than just an analytics tool. It's the scheduled process that runs continuously, monitoring the world and acting on your behalf.

### What the Heartbeat Does

On each heartbeat tick:
1. **Checks for new signals**: Searches news sources, monitors price changes, checks webhooks
2. **Filters noise**: Maps signals to causal tree nodes, discarding irrelevant information
3. **Runs evaluation**: Updates probabilities, recalculates edge
4. **Checks strategies**: Evaluates entry/exit/stop conditions
5. **Sends notifications**: Alerts you to significant changes

### Heartbeat Frequency

The default heartbeat runs every 30 minutes during active market hours. But frequency can be adjusted:
- **High-frequency**: Every 5-10 minutes during volatile events (FOMC day, CPI release)
- **Low-frequency**: Every 2-4 hours for long-dated theses with slow-moving fundamentals
- **Burst mode**: Continuous scanning when a trigger event occurs

### Why the Heartbeat Matters

Markets move while you sleep. A news event at 3 AM could create a 20-point edge that evaporates by market open. The heartbeat catches these opportunities and can execute strategies automatically.

### Heartbeat Monitoring

You can check heartbeat status with `sf heartbeat status` to see:
- When the last heartbeat ran
- How many signals were processed
- Whether any strategies were triggered
- Current thesis state summary


## Example

sf heartbeat status

Thesis: "US Recession by 2026"
Heartbeat: Active (every 30 min)
Last run: 2 minutes ago
Next run: in 28 minutes

Last 24h summary:
  Heartbeats completed: 48
  Signals ingested: 23
  Signals mapped to nodes: 8
  Probability updates: 4
  Strategy checks: 48
  Strategy triggers: 1 (buy 50 KXRECSSNBER-26 at $0.29)
  Notifications sent: 2

Current state:
  Thesis confidence: 72%
  Root probability: 42%
  Active strategies: 3
  Open positions: 2


## CLI

```bash
sf heartbeat status
```


## Related

[evaluation-cycle](evaluation-cycle.md), [signal](signal.md), [thesis](thesis.md), [edge-detection](edge-detection.md)
