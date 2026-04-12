# 5 Patterns That Kill Prediction Market Traders (and How Agents Fix Them)

> Not a textbook. These are real trading mistakes every prediction market trader makes — anchoring, news overload, asymmetric fear, frequency illusion, confirmation bias — and how an automated agent eliminates each one.

**Category:** patterns | **Author:** Patrick Liu | **Reading time:** 1 min

---
Every prediction market trader loses money the same way. Not because they can't read the news. Not because they pick the wrong contracts. Because their brains are running buggy software — cognitive patterns that evolution optimized for survival, not for pricing binary outcomes on Kalshi.

Here are the five patterns that kill prediction market traders, why they're so hard to fix manually, and how an automated agent with a causal model neutralizes each one.

## 1. Anchoring: "I Already Bought at 32¢"

**The pattern:** You bought a recession contract at 32¢. It moves to 35¢. New information comes in — CPI data, a Fed speech, whatever — that your analysis says warrants a price of 45¢. But you don't add. Why? Because you're anchored to your entry price. "It already went up" feels like a reason not to buy more.

This is insane. The market doesn't know your entry price. The edge is between *current price* and *your estimate of true probability*. If your thesis says 45% and the market says 35%, the edge is 10 points regardless of where you bought.

**How the agent fixes it:** SimpleFunctions' strategy engine evaluates edge on every heartbeat. It compares *current market price* against *current thesis-implied probability*. Your entry price doesn't exist in the evaluation function. When the heartbeat runs at 3am and CPI data shifted your causal tree, the agent sees: "thesis says 45%, market says 35%, edge = 10 points, conditions met." It doesn't know or care that you bought at 32¢ yesterday.

The agent has no memory of your P&L. It only sees the current state of the world versus the current price. That's not a bug — it's the entire point.

## 2. News Overload: "Ten Hormuz Headlines and I Don't Know What Matters"

**The pattern:** You're running an Iran thesis. Every day there are 10-15 headlines about Hormuz, IRGC, oil tankers, diplomatic channels, military exercises. Some are noise. Some fundamentally change your causal tree. But you can't tell which is which because you're drowning in the firehose.

So you either (a) ignore all news and miss critical updates, or (b) react to every headline and overtrade. Both lose money.

**How the agent fixes it:** The causal tree is a noise filter. When the monitor service picks up a new headline via Tavily search, it doesn't ask "is this important?" — it asks "which causal node does this affect?"

A headline about Iran testing a new missile? Maps to node n1.2 (Iran continues retaliation) — relevant, triggers re-evaluation. A headline about a US senator's opinion on Iran policy? Doesn't map to any node in the tree — ignored.

The causal tree converts the question from "is this news important?" (subjective, emotional) to "does this change the probability of a specific node?" (structured, answerable). If a headline doesn't touch any node, it gets filtered out automatically. No willpower required.

## 3. Asymmetric Fear: "I'll Take Profits at +5¢ But Hold Losses to -20¢"

**The pattern:** Prospect theory in action. You bought a contract at 40¢. It moves to 45¢ and you sell — "lock in the gains." Then it moves to 60¢ and you watch from the sidelines. Conversely, it drops to 30¢ and you hold — "it'll come back" — then to 20¢, then to 10¢. The pain of realizing a loss is roughly 2x the pleasure of realizing a gain.

The result: you cut your winners short and let your losers run. This is the #1 account killer across all forms of trading.

**How the agent fixes it:** The strategy system uses hard conditions. When you create a strategy, you define:

- **Entry conditions:** What must be true to enter (e.g., "thesis confidence > 65% AND market price < 35¢ AND edge > 8 points")
- **Exit conditions:** What must be true to exit (e.g., "thesis confidence < 40% OR edge < 3 points")
- **Stop conditions:** When to kill the position regardless (e.g., "causal node n1 drops below 50%")

These conditions are evaluated mechanically. There's no "but I feel like it'll come back." When the causal tree updates and node n1 (war persists) drops below 50% because of a surprise diplomatic development, the stop condition triggers. The agent sends you a notification: "Strategy exit signal — n1 dropped below threshold." It doesn't feel bad about the loss. It doesn't wait for a bounce.

The asymmetry is eliminated because the conditions are symmetric. The same logic that tells you to enter tells you to exit. No emotion in either direction.

## 4. Frequency Illusion: "I Should Be Trading Every Day"

**The pattern:** You have a Kalshi account. You log in every morning. You feel like you should be *doing something*. So you find a contract, form a quick opinion, and trade. No thesis. No causal model. Just activity for activity's sake.

Prediction markets are not the stock market. There isn't always an edge. Most contracts are efficiently priced most of the time. The edge appears when new information changes the causal landscape faster than the market prices it in — and that might happen once a week, or once a month.

**How the agent fixes it:** The heartbeat runs every 15 minutes. It scans all active strategies. It evaluates conditions. If no conditions are met, it does nothing. Literally nothing — no trades, no notifications, no activity.

This is the hardest thing for a human trader to do: nothing. Sitting with an open position and not touching it. The agent does this effortlessly because it has no boredom, no FOMO, no urge to "be active."

Your heartbeat might run 96 times in a day and trigger zero actions. Then at 2:47am on a Tuesday, CPI data drops, the causal tree updates, an edge appears, and it fires. One signal in 96 evaluations. That's not a bug — that's discipline that no human can maintain 24/7.

## 5. Confirmation Bias: "The News Supports My Thesis"

**The pattern:** You believe recession is coming. So you read ZeroHedge. You follow the bearish accounts on Twitter. You notice every negative data point and dismiss every positive one. Your information diet is a mirror — it reflects your existing beliefs back at you.

When the Fed says something hawkish, you interpret it as "they know recession is coming." When they say something dovish, you interpret it as "they're panicking because they see what I see." Every piece of evidence confirms your thesis.

Meanwhile, the market is slowly pricing against you, but you don't notice because your news feed is still bearish.

**How the agent fixes it:** The monitor service searches for news related to *causal nodes*, not *conclusions*. It searches for "Iran Hormuz shipping" and "IRGC naval activity" — not for "Hormuz will stay blocked" or "oil will rise."

The search queries are node-adjacent, not conclusion-adjacent. They pull in bullish and bearish information equally because they're searching for facts about the node, not evidence for or against the thesis.

When the monitor finds "Iran signals willingness to negotiate on Hormuz transit" — that's a bearish signal for the Iran thesis. But it gets pulled into the evaluation because the search was "Iran Hormuz" (the node), not "Hormuz will stay blocked" (the conclusion). The evaluation LLM then updates the node confidence downward. No human judgment filtering.

The system is structurally incapable of confirmation bias because the information retrieval layer doesn't know what answer you want.

---

## The Common Thread

All five patterns share one root cause: the human brain processes market positions as extensions of identity. Your thesis becomes *your* thesis. Your P&L becomes *your* P&L. The market becomes personal.

An agent with a causal model doesn't have this problem. The thesis is a data structure. The P&L is a number. The market is a price feed. There's no identity attached to any of it.

This doesn't mean the agent is always right. The causal model can be wrong. The confidence estimates can be off. The LLM can hallucinate. But the agent will never be wrong *because it's scared*, or *because it's bored*, or *because it doesn't want to admit it was wrong*.

And in prediction markets — where the edge is small and the competition is efficient — eliminating systematic emotional errors is often the difference between making money and losing it.

---

## Getting Started

If these patterns sound familiar, they should — every trader has lived through all five.

SimpleFunctions gives you the infrastructure to externalize your thesis into a causal model that an agent can execute without cognitive bias. The `sf scan` command finds contracts. The thesis engine structures your thinking. The strategy system defines entry and exit conditions. The heartbeat monitors 24/7.

Your job becomes what it should have been all along: the *thinking*. Not the watching. Not the clicking. Not the agonizing at 3am about whether to cut a position.

You think. The agent executes.