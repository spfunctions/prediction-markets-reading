# Webhook

**A webhook is an HTTP callback that sends real-time notifications to your application when specific events occur — such as price alerts, strategy triggers, or thesis confidence changes.**


## Explanation

## Webhooks for Prediction Market Automation

Webhooks let you receive instant notifications when something important happens, without constantly polling for updates.

### What Triggers Webhooks

SimpleFunctions can send webhooks for:
- **Price alerts**: A contract crosses a threshold you set
- **Edge alerts**: New edge detected above your minimum
- **Strategy triggers**: A strategy entry or exit condition is met
- **Evaluation updates**: Thesis confidence changes significantly
- **Heartbeat summaries**: Periodic summaries of monitoring activity

### Webhook Payload

Each webhook sends a JSON payload with structured data:

```json
{
  "type": "strategy_trigger",
  "thesisId": "...",
  "strategy": "S-001",
  "action": "entry",
  "market": "KXRECSSNBER-26",
  "currentPrice": 0.29,
  "thesisImplied": 0.42,
  "edge": 0.13,
  "timestamp": "2026-03-19T14:30:00Z"
}
```

### Common Integrations

- **Slack/Discord**: Forward webhook payloads to a channel for team visibility
- **Telegram**: Get mobile notifications via a Telegram bot
- **Custom bots**: Trigger automated trading workflows
- **Zapier/Make**: Connect to any service without code

### Setting Up Webhooks

You can set a webhook URL per-thesis:

```bash
sf thesis update --webhook "https://your-app.com/webhook/sf"
```

The agent also supports webhooks as a signal input — external services can push signals to your thesis via webhook, which the evaluation cycle processes like any other signal.


## Example

# Set up a webhook for your thesis
sf thesis update recession-2026 --webhook "https://hooks.slack.com/services/..."

# When a strategy triggers, Slack receives:
{
  "type": "strategy_trigger",
  "thesis": "US Recession by 2026",
  "action": "BUY",
  "market": "KXRECSSNBER-26",
  "quantity": 50,
  "price": "$0.29",
  "edge": "+13pt",
  "reason": "Edge exceeds threshold, price below entry condition"
}


## CLI

```bash
sf thesis update --webhook
```


## Related

[heartbeat](heartbeat.md), [signal](signal.md), [api-key](api-key.md), [evaluation-cycle](evaluation-cycle.md)
