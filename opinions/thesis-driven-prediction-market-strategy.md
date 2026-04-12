# How to Build a Thesis-Driven Prediction Market Strategy

> Not how to build a bot. How to structure your thinking about a prediction market bet — from causal tree to executable edge.

**Category:** tutorial | **Author:** Patrick Liu | **Reading time:** 4 min

---
Most prediction market content tells you how to call an API or build a bot. This article is different. It's about how to think — how to structure a prediction market bet so you know *why* you're entering, *what* needs to be true for you to win, and *where* the market is mispricing reality.

## The Problem with Unstructured Bets

Most prediction market traders operate on vibes. They see a contract — "Recession in 2026? YES at 35¢" — and think "yeah, probably" and buy it. No framework for *why* they believe that. No way to update when new information arrives. No way to know when the bet is dead.

This is how you lose money systematically.

## Start with a Thesis, Not a Contract

A thesis is a directional claim about the world with a causal structure. Not "I think recession will happen" but:

> "The US-Iran war will persist. Hormuz stays closed. Oil stays above $100. The SPR can't compensate. The Fed is trapped between inflation and recession."

This is a *chain* of claims. Each one causes the next. And each one can be independently verified or falsified.

## Decompose into a Causal Tree

Once you have a thesis, decompose it into a tree of testable sub-claims, each with an estimated probability:

```
n1  War persists                     88%
├── n1.1  US initiated strikes       99%
├── n1.2  Iran continues retaliation 85%
└── n1.3  No diplomatic exit         82%
n2  Hormuz blocked                   97%
├── n2.1  Mines deployed             99%
├── n2.2  IRGC small craft active    90%
└── n2.3  Minesweeping takes 3+ mo   87%
n3  Oil stays elevated               64%
n4  Recession                        45%
```

These nodes are causally linked — upstream changes cascade downstream. If Iran stops retaliating (n1.2 drops), the whole tree updates: Hormuz reopens, oil drops, recession probability falls.

## Find Where Markets Disagree

Now scan prediction markets for contracts that map to your causal tree nodes. The edge is where *your* thesis-implied price diverges from the *market* price:

| Contract | Thesis Price | Market Price | Edge |
|----------|:----------:|:----------:|:----:|
| Recession 2026 YES | 72¢ | 35¢ | **+37¢** |
| WTI $150 YES | 75¢ | 38¢ | **+37¢** |
| Gas $4.50 Mar YES | 55¢ | 14¢ | **+41¢** |

A 37-cent edge on Recession means: if your causal model is right, the market is massively underpricing this contract. That's where you trade.

## The Edge Is Not the Spread

Raw edge isn't enough. You need *executable* edge — the edge after accounting for the orderbook spread and liquidity:

```
$ sf edges

  Recession 2026 YES    35¢  72¢  +37  +36  1¢  high   f582bf76
  WTI $150 YES          38¢  75¢  +37  +36  1¢  high   641ba280
  Gas $4.50 Mar YES     14¢  55¢  +41  +39  3¢  med    641ba280
```

The gas contract has the highest raw edge (+41) but only medium liquidity and a 3¢ spread. The recession contract has slightly lower raw edge (+37) but a 1¢ spread and high liquidity. In practice, the recession bet is the better entry.

## Update Continuously

A thesis isn't a one-time bet. It's a living model. When new information arrives — a news article, a price move, a diplomatic signal — you feed it into the causal tree:

```
$ sf signal f582bf76 "Reuters: Hormuz mines confirmed by satellite imagery" --type news
```

The system updates affected node probabilities, recalculates edges, and tells you whether your positions are still good or if something has changed.

## Know When You're Wrong

The most important part of a thesis is knowing when to abandon it. If Iran starts de-escalating and your n1.2 ("Iran continues retaliation") drops from 85% to 30%, the entire downstream chain collapses. Oil drops, recession probability falls, your gas and recession bets are dead.

A causal tree makes this visible. You're not staring at a PnL wondering why you're losing — you can see exactly which upstream node broke and what it means for every downstream contract.

## The Workflow

1. **Write** a thesis in plain English
2. **Decompose** into a causal tree with probabilities
3. **Scan** prediction markets for contracts that map to your nodes
4. **Identify** edges where market price diverges from thesis price
5. **Execute** on the highest executable edges
6. **Monitor** 24/7 — inject signals, update probabilities, recalculate edges
7. **Exit** when upstream nodes falsify your thesis

This is what SimpleFunctions automates. You provide the thesis. The engine builds the causal tree, scans Kalshi and Polymarket, detects edges, and monitors everything with a 15-minute heartbeat cycle.

```
$ npm install -g @spfunctions/cli
$ sf setup
$ sf context <thesis-id> --json
```

The thesis is yours. The infrastructure is ours.