# AI Agents Don't Need More Data. They Need Judgment.

> The bottleneck for AI agents in financial markets isn't data access — it's the ability to structure beliefs, track causation, and know when they're wrong.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 5 min

---
Every AI agent framework gives you the same pitch: connect your agent to more data sources. More APIs. More feeds. More tools.

The implicit assumption is that the bottleneck is information access. Give the agent enough data and it'll figure out what to do.

This is wrong.

## The Data Surplus Problem

A prediction market agent connected to a news API, a price feed, and a social media stream receives thousands of signals per day. It has more data than any human trader could process. And yet, without a framework for *what to do with that data*, it produces noise.

More specifically, it can't answer:

- **Which signals matter?** A Reuters headline about Hormuz mines matters for an oil thesis. A celebrity tweet about Bitcoin doesn't. The agent needs to know *why* some information is relevant and some isn't.
- **How do signals connect?** Oil above $100 doesn't directly cause recession. It causes consumer spending pressure, which affects Fed policy, which affects credit conditions, which affects recession probability. The agent needs a *causal chain*, not a correlation.
- **When to change its mind?** An agent with a "recession likely" position needs to know what would falsify that belief. Not "if prices go down" but "if n1.2 (Iran retaliates) drops below 40% and n3 (oil stays elevated) drops below 50%, the thesis is dead."

This isn't a data problem. It's a judgment problem.

## Judgment Is Structure

We often think of judgment as mysterious — an experienced trader's "gut feel" that can't be quantified. But judgment is actually quite structural:

1. **Beliefs** — What do you think is true about the world? (thesis)
2. **Causation** — What causes what? (causal tree)
3. **Confidence** — How sure are you about each link? (node probabilities)
4. **Falsification** — What would prove you wrong? (node thresholds)
5. **Update rules** — How does new information change your beliefs? (signal → evaluation)

Each of these is representable, computable, and trackable. You can give an agent a causal tree and say "here's my thesis, here's the structure, here are the probabilities, monitor these nodes." That's structured judgment.

## The Missing Layer

The current AI agent stack looks like this:

```
LLM ← Tools (APIs, data sources) ← User prompt
```

The agent gets a prompt, calls tools to gather data, and generates a response. This works for simple tasks. For financial markets, it produces hallucination dressed up as analysis.

What's missing is a *judgment layer*:

```
LLM ← Judgment Layer (thesis + causal tree + edges) ← Tools ← User prompt
```

The judgment layer provides:

- **Context**: not "here's all the data" but "here's your thesis, here's the causal tree, here's what changed since last check, here are the current edges"
- **Relevance filtering**: only signals that affect nodes in your causal tree
- **Update mechanics**: when a signal shifts a node probability, the tree cascades changes downstream automatically
- **Falsification**: when upstream nodes break, the system tells you your thesis is dead

## Productive Individuals vs Productive Systems

Chamath Palihapitiya recently made the observation that productive individuals don't automatically create productive firms. His 8090 Software Factory is trying to solve this for software engineering — turning individual developer productivity into organizational output.

The same principle applies to AI agents. A powerful LLM is a productive individual. It can analyze a document, generate a hypothesis, write code. But connecting five powerful LLMs to five data sources doesn't give you a productive trading system. It gives you five productive individuals pulling in different directions.

What makes a trading system productive is *shared structure*: a thesis that everyone operates against, a causal model that everyone updates, a set of edges that everyone monitors. The individuals contribute signals and analysis. The structure turns it into coherent action.

## What This Looks Like in Practice

A SimpleFunctions agent operates on a thesis, not a data stream:

```bash
# The agent reads structured context, not raw data
sf context <thesis-id> --json

# It injects observations into the causal tree
sf signal <thesis-id> "Hormuz blockade confirmed by satellite" --type news

# The system evaluates how this shifts the model
sf evaluate <thesis-id>

# Edges update automatically
sf edges
```

The agent's "judgment" isn't a black box. It's a causal tree you can inspect. When the agent says "recession probability increased to 72%," you can see exactly why: which upstream node changed, by how much, and what evidence caused the change.

This is the difference between an agent that trades on vibes and an agent that trades on structured, verifiable, updatable beliefs.

## The Implication

If you're building AI agents for financial markets, the competitive advantage isn't better data access — everyone has the same APIs. It's not a better LLM — they're commoditizing fast. It's the judgment layer: the causal model that turns information into structured beliefs, tracks those beliefs over time, detects when reality diverges from the model, and surfaces the specific contracts where the market disagrees with your analysis.

Data is abundant. Judgment is scarce. Build for judgment.