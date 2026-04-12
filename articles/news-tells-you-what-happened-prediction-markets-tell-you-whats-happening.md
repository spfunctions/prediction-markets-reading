# News Tells You What Happened. Prediction Markets Tell You What's Happening.

> Headlines are past tense. Prices are present tense. If your agent reads news to understand the world, it's always one step behind.

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 7 min | **Published:** 2026-03-31

---
# News Tells You What Happened. Prediction Markets Tell You What's Happening.

There is a grammatical distinction that reveals everything about how information flows in the modern world.

News is written in the past tense: *"The Federal Reserve raised interest rates."* By the time you read this sentence, the decision happened hours ago. The market priced it in minutes after the announcement. The smart money priced it in days before.

A prediction market contract is present tense: *"Federal Reserve rate cut by June: 34 cents."* This is not a report about what happened. It is a continuously-updated statement about what the collective market believes is happening *right now*.

## The Information Delay Stack

Every information source has a latency — the time between an event occurring and you learning about it.

| Source | Latency | What you get |
|--------|---------|-------------|
| Being there | 0 | Direct observation |
| Prediction market price | seconds-minutes | Probability (continuously updated) |
| Wire services (AP, Reuters) | 5-30 minutes | Headline + 2 paragraphs |
| News articles | 30 min - 2 hours | Full report with context |
| Analysis/commentary | 2-24 hours | Expert interpretation |
| Weekly roundups | 1-7 days | Curated summary |

Most agents consume information at the "news articles" to "analysis" tier. They scrape articles, extract entities, summarize text. By the time they've processed the information, the world has moved on.

An agent watching prediction market prices operates at the "seconds-minutes" tier. It doesn't need to parse language, extract sentiment, or assess source credibility. The price *is* the assessment, pre-computed by a market of incentivized participants.

## Past Tense vs. Present Tense vs. Future Tense

This is not just about speed. It's about the *kind* of information each source provides.

**News tells you what happened** (past tense). "The president signed the executive order." Useful for building a historical record. Useless for making decisions about what to do next, because by the time you know, everyone knows.

**Prediction markets tell you what's happening** (present tense). A contract on "Executive order on X by Friday" moving from 30 to 65 cents tells you that something changed — new information entered the system, the probability shifted. You don't need to know the *content* of the information to know that the *state of the world* changed.

**Prediction markets also tell you what will happen** (future tense). A contract on "US-China trade deal by December" at 22 cents tells you: the market thinks this is unlikely. If you disagree, that disagreement is an edge — a quantifiable gap between your model of the world and the market's model.

News gives you none of this. News can't tell you the probability of a future event. It can quote an expert who guesses, but that guess has no accountability, no calibration, and no market feedback mechanism.

## The Inversion: Reading Events Through Prices

Here's the paradigm shift: instead of reading news to understand events, read *price movements* to understand events, then use news to understand *why* the price moved.

Traditional flow:
```
Event → News report → Read article → Update world model → Decide
```

Price-first flow:
```
Event → Price moves → Notice movement → Read news for context → Decide
```

The price-first flow is strictly faster, because the price moves before the news publishes. And it's more honest, because you see the *magnitude* of the change (a 5-cent move vs. a 30-cent move tells you how significant the event is) before any editorial framing colors your interpretation.

A 2-cent price change on a contract means: something happened, but it doesn't change the overall picture much. A 25-cent price change means: the world just shifted. You should pay attention.

No headline comes with this built-in significance scale. "BREAKING" is applied to everything from a cabinet reshuffling to a market crash.

## What This Means for Your Agent

If you're building an AI agent that monitors the world — for trading, for risk assessment, for research, for decision support — the most important question is: what is your agent's *information latency*?

If your agent reads news feeds, its latency is 30 minutes to 2 hours. It knows what happened, but not what's happening.

If your agent watches prediction market prices, its latency is seconds to minutes. It knows what's happening, and it knows the market's current probability estimate for what will happen next.

The practical implications:

1. **Monitor prices first, articles second.** Watch for significant price movements. Use articles to explain why the price moved.
2. **Use deltas, not snapshots.** A price of 65 cents is less informative than "the price moved from 40 to 65 in the last hour." The delta tells you *something just changed*.
3. **Cross-reference multiple contracts.** A single contract is a data point. Ten related contracts moving in the same direction is a signal. Ten contracts moving in opposite directions is a contradiction worth investigating.

The world is not a newspaper. It's a continuous stream of events, probabilities, and beliefs. Prediction markets are the closest thing we have to a real-time API for that stream.

---

*This is part of a series on prediction markets as cognitive tools. Next: [How to Read the World Through Prediction Market Prices](/blog/how-to-read-the-world-through-prediction-market-prices)*
