# Three Data Sources That Tell You What the World Thinks, What the World Is Doing, and What the World Is Feeling

> Prediction markets are belief. Traditional markets are action. Social media is sentiment. Each alone is incomplete. Together, they form the most complete real-time picture of the world available to any agent.

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 8 min | **Published:** 2026-03-31

---
# Three Data Sources That Tell You What the World Thinks, What the World Is Doing, and What the World Is Feeling

There are three public data sources that, taken together, give you the most complete real-time picture of the world available to any observer.

Each one captures a different dimension of reality:

1. **Prediction markets** tell you what the world *thinks* will happen. (Belief)
2. **Traditional markets** tell you what the world *is doing* about it. (Action)
3. **Social media** tells you what the world *feels* about it. (Sentiment)

Each alone is incomplete. Together, they triangulate reality.

## Prediction Markets: The Belief Layer

A prediction market contract is a crystallized belief with money behind it. "Iran sanctions by April: 73 cents" means: the weighted consensus of people willing to risk capital says there's a 73% chance this happens.

Prediction markets are uniquely valuable because:
- **They're forward-looking.** They tell you about events that haven't happened yet.
- **They're calibrated.** A 70% contract resolves "yes" roughly 70% of the time.
- **They're incentive-aligned.** Wrong beliefs cost money. Right beliefs make money.

But prediction markets have blind spots:
- **Low-liquidity contracts are unreliable.** A $500-volume contract is one person's guess.
- **They can't tell you *why*.** A price moved 15 cents. Why? The market doesn't explain itself.
- **They don't capture urgency or emotion.** A contract can move slowly toward 90% while the world is panicking.

This is where the other two sources fill the gaps.

## Traditional Markets: The Action Layer

When the S&P 500 drops 3% in a day, that's not a belief — it's an action. Billions of dollars are being moved. Portfolios are being reallocated. Risk is being repriced.

Traditional market prices (equities, bonds, commodities, currencies, volatility indices) tell you what the world is *doing* with its money right now:

- **VIX spikes** = fear. The cost of hedging is rising. People are buying protection.
- **Gold rises** = flight to safety. Capital is moving from risk assets to stores of value.
- **Oil spikes** = supply disruption or demand surge. Real-economy impact incoming.
- **Treasury yields fall** = money flowing into safe bonds. Risk-off sentiment.
- **Dollar strengthens** = global capital seeking safety in the reserve currency.

These are not opinions. They are *actions at scale*. When a pension fund moves $500 million into treasuries, that's a stronger signal than any op-ed about market risk.

The limitation of traditional markets: they tell you *what* people are doing, but not *why*. Is the S&P dropping because of Iran? Because of earnings? Because of a technical breakdown? The price doesn't say.

## Social Media: The Sentiment Layer

X (Twitter), Reddit, and other social platforms capture the raw emotional state of the world's discourse.

When a geopolitical event happens:
- **Volume of discussion** tells you: how many people care about this.
- **Sentiment of discussion** tells you: are people scared, angry, hopeful, confused?
- **Velocity of discussion** tells you: is attention accelerating or fading?
- **Key voices** tell you: who's shaping the narrative?

Social media is the fastest signal — even faster than prediction markets for breaking events, because it captures the *reaction* before anyone has time to *price* the reaction.

But social media is also the noisiest signal:
- **It's not incentive-aligned.** Outrage gets engagement. Calm analysis doesn't.
- **It's not calibrated.** People who are loudly wrong face no financial consequence.
- **It's manipulable.** Bots, campaigns, and viral dynamics distort the signal.
- **Volume ≠ importance.** A celebrity scandal generates more discussion than a trade policy change, but the trade policy affects more people.

## The Trinity: Belief + Action + Sentiment

Each data source has a superpower and a blind spot:

| Source | Superpower | Blind spot |
|--------|-----------|-----------|
| Prediction markets | Calibrated probability estimates | Can't explain *why* |
| Traditional markets | Massive-scale capital flows | Can't distinguish causes |
| Social media | Raw speed + emotional texture | Noisy, manipulable, uncalibrated |

The magic happens when you combine them:

**Prediction market moves + no social media volume** = informed money is moving quietly. The signal is strong because it's not driven by crowd panic.

**Social media explosion + no prediction market movement** = the crowd is panicking, but the market doesn't agree. The panic is probably overblown.

**All three moving in the same direction** = the event is real, significant, and the world is both believing it, acting on it, and feeling it. This is the strongest signal.

**Traditional markets moving but prediction markets stable** = the market is pricing a risk that isn't a specific event. Systematic risk, not event risk.

## Building a Perception Engine

An agent that monitors all three sources can build a *perception engine* — a real-time model of the world's state that combines belief, action, and sentiment:

```
Input Layer:
  Prediction markets → probability estimates for 1000+ events
  Traditional markets → SPY, VIX, gold, oil, bonds, currencies
  Social media → volume, sentiment, velocity, key voices

Processing:
  Cross-reference: Are all three sources aligned?
  Detect divergences: Where do sources disagree?
  Prioritize: Which signals are high-delta + high-volume?

Output:
  World state vector: a structured, real-time model of
  what's happening, how likely outcomes are, and how
  the world feels about it.
```

No single source gives you this. News is too slow. Expert analysis is too infrequent. Government reports are too delayed. But prices + actions + sentiment, updated continuously, give you the closest thing to a real-time world model that currently exists.

---

*This is part of a series on prediction markets as cognitive tools. Next: [Orderbooks Are Fossilized Beliefs](/blog/orderbooks-are-fossilized-beliefs)*
