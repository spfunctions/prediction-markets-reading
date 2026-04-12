# Causal trees for prediction markets: turning macro intuition into tradeable structure

> A practical walkthrough of building hierarchical probabilistic models that map directly to binary contracts on Kalshi and Polymarket.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
I've been trading prediction markets on Kalshi with real money for the past year. The single biggest improvement to my PnL wasn't a better data feed or a faster bot. It was learning to write down what I actually believe about the world — in a specific, structured way — before I put money on anything.

That structure is a causal tree.

## What is a causal tree

A causal tree is a hierarchical probabilistic model of reality. You start with a thesis — a directional claim about how the world works — and decompose it into a tree of sub-claims, each with an estimated probability. Parent nodes depend on their children. Change a child's probability and the parent updates.

It's not a fancy concept. You already think this way when you reason about geopolitics or economics. "If Iran keeps retaliating, then Hormuz stays closed. If Hormuz stays closed, oil stays high. If oil stays high, recession becomes likely." That's a causal chain. A causal tree just writes it down explicitly, with numbers.

Here's the key insight: every node in a causal tree can potentially map to a binary contract on Kalshi or Polymarket. "War persists" maps to conflict-duration contracts. "Oil stays elevated" maps to KXWTIMAX series. "Recession in 2026" maps to KXRECESSION-26. The tree isn't an abstraction you build and then separately go trade. The tree *is* the trading plan.

## Why it works for prediction markets

Prediction markets trade binary contracts. YES or NO, resolves to $1 or $0. This looks simple but it creates a hard problem: how do you decide what a contract is worth?

Most traders use vibes. They see "Recession 2026 YES at 35 cents" and think "that feels low" and buy it. No framework for why. No way to update. No way to know when the trade is dead.

A causal tree solves this because it gives you a *derived* price for every contract. You don't estimate the recession probability directly. You estimate the upstream drivers — war duration, oil supply disruption, Fed response — and the recession probability falls out of the tree. This is better than gut feel for three reasons:

1. **Decomposition makes estimation easier.** You probably can't accurately estimate P(recession). But you can estimate P(Hormuz stays blocked | war continues) with reasonable confidence, because it depends on naval logistics and mine-clearing timelines, not the entire global economy.

2. **Updates are mechanical.** When a Reuters headline says "satellite imagery confirms new mines in Strait of Hormuz," you know exactly which node to update (n2.1, mines deployed) and the cascade is automatic. You don't have to re-think the entire thesis from scratch.

3. **Falsification is explicit.** If Iran stops retaliating (n1.2 drops from 85% to 20%), you can see immediately that your entire downstream chain collapses. You exit before the loss compounds.

## Building a causal tree from a thesis statement

Let me walk through a real example. My thesis for the past several months has been:

> "The US-Iran conflict will persist and escalate. Hormuz will remain disrupted. Oil will stay elevated through 2026. The US economy will enter recession."

That's four claims chained together. Each one causes the next. Let me decompose them.

**Node 1: War persists.** This is the root of the upstream chain. What needs to be true for the war to persist? The US initiated strikes (already happened, 99%). Iran continues retaliating (assessment: 85%). No diplomatic off-ramp emerges (82%). I assign the composite node 88%.

**Node 2: Hormuz blocked.** Given the war persists, what keeps Hormuz closed? Mines deployed (confirmed, 99%). IRGC small craft harassment continues (90%). Mine-clearing operations take 3+ months (based on historical precedent from 1988 tanker war, 87%). Composite: 97%.

**Node 3: Oil stays elevated.** Given Hormuz disruption, does oil stay above $100? This depends on SPR release capacity, OPEC+ spare capacity, and demand destruction timeline. I had this at 64% when I first built the tree. It's moved since.

**Node 4: Recession.** Given sustained high oil prices, what's the recession probability? This is the final downstream node, dependent on consumer spending pressure, Fed policy response, credit conditions, and corporate earnings impact. Initially 45%, but the tree-implied probability — after propagating all upstream nodes — came out much higher.

Here's what the tree looks like when you run `sf context --tree`:

```
Iran War Thesis — Causal Tree
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

n1  War persists                        88%
├── n1.1  US initiated strikes          99%  (confirmed)
├── n1.2  Iran continues retaliation    85%
│   ├── n1.2.1  IRGC doctrine intact    92%
│   └── n1.2.2  Domestic pressure       78%
└── n1.3  No diplomatic exit            82%
    ├── n1.3.1  US election dynamics     88%
    └── n1.3.2  Iran hardliner control   85%

n2  Hormuz blocked                      97%
├── n2.1  Mines deployed                99%  (confirmed)
├── n2.2  IRGC small craft active       90%
└── n2.3  Minesweeping takes 3+ mo      87%

n3  Oil stays elevated (>$100)          64%
├── n3.1  SPR insufficient              72%
├── n3.2  OPEC+ can't compensate        68%
└── n3.3  Demand destruction slow        61%

n4  Recession 2026                      45%  (direct)
    Thesis-implied:                     72%  (propagated)
```

That last line is where the money is. My direct estimate of recession probability is 45% — roughly what the market says. But when I propagate through the tree, the thesis-implied probability is 72%. If my upstream estimates are right, the market is underpricing recession by 37 cents.

## How probabilities propagate

The propagation logic is straightforward. Each parent node's probability is a function of its children. The simplest model: multiply the child probabilities along a chain.

For node 4 (Recession), the chain is:

```
P(recession | thesis) = P(war persists) × P(hormuz | war) × P(oil elevated | hormuz) × P(recession | oil elevated)
                      = 0.88 × 0.97 × 0.64 × P(recession | oil > $100)
```

The conditional probability P(recession | oil > $100) is where your macro knowledge matters. Based on historical base rates — every sustained period of oil above $100 in the past 50 years has been associated with recession — I estimate this at around 0.80.

So: 0.88 × 0.97 × 0.64 × 0.80 = 0.437 at the most conservative reading. But the tree has sub-nodes that reinforce each other. When you account for the feedback loops (recession fears reduce oil demand, but Hormuz blockade constrains supply more), the propagated number pushes higher.

In practice, SimpleFunctions runs a more sophisticated propagation that handles conditional dependencies, feedback loops, and correlation between sibling nodes. But the mental model is: multiply along the chain, adjust for conditionals.

## How node updates cascade

This is where causal trees become genuinely useful for trading. When new information arrives, you update a single node and watch the cascade.

**Example: Diplomatic breakthrough.**

Suppose tomorrow Reuters reports that backchannel negotiations between the US and Iran are making progress. You update n1.3 (No diplomatic exit) from 82% down to 40%.

The cascade:

```
n1.3  No diplomatic exit:    82% → 40%   (-42 pts)
n1    War persists:          88% → 62%   (-26 pts)
n2    Hormuz blocked:        97% → 68%   (-29 pts)
n3    Oil stays elevated:    64% → 45%   (-19 pts)
n4    Recession (implied):   72% → 38%   (-34 pts)
```

Every downstream node shifts. Your recession edge evaporates — the thesis-implied price drops from 72 cents to 38 cents, which is basically where the market already trades. Time to exit.

Conversely, if Iran launches a major retaliatory strike, n1.2 moves from 85% to 95%, and every downstream node tightens upward. Your edge grows.

The point is: you never have to re-estimate the entire model. You update the node that the new information directly affects, and the tree handles the rest.

## From tree to trades

Once you have the tree, finding trades is mechanical. For every leaf or intermediate node that maps to a tradeable contract, compare the thesis-implied price to the market price:

```
$ sf edges

Contract                Market   Thesis   Edge   Exec    Spread  Liq
─────────────────────── ──────── ──────── ────── ─────── ─────── ──────
KXRECESSION-26 YES      35¢      72¢      +37    +36     1¢      high
KXWTIMAX-26DEC31-T150   38¢      75¢      +37    +36     1¢      high
KXGASPRICE-26MAR-T450   14¢      55¢      +41    +39     3¢      med
KXFEDX-26JUN YES        42¢      68¢      +26    +24     2¢      high
```

Raw edge is (thesis price - market price). Executable edge accounts for the spread — what you'd actually pay to get filled. Liquidity grade tells you whether the orderbook can absorb your size.

The gas contract (KXGASPRICE-26MAR-T450) has the highest raw edge at +41 cents, but only medium liquidity and a 3-cent spread. The recession contract (KXRECESSION-26) has +37 raw edge but a 1-cent spread and high liquidity. In practice, recession is the better trade because you can get in and out cleanly.

This is the entire workflow: thesis → tree → propagation → edges → execution. No vibes. Every number traces back to an explicit belief about a specific node in the tree.

## The discipline of writing it down

I want to be honest about something. The causal tree doesn't make you right. I've had nodes that I estimated at 90%+ flip to near-zero when reality surprised me. Iran has done things I didn't model. The Fed has responded in ways I didn't anticipate.

What the tree does is make you *precisely wrong* instead of *vaguely wrong*. When a thesis breaks, you can see exactly which node broke and why. You know whether it was a failure of your upstream model (you misjudged Iran's strategic calculus) or a downstream surprise (oil stayed high but consumers didn't cut spending the way you expected). This matters because it's how you get better. Vague wrongness teaches you nothing. Precise wrongness teaches you where your model of the world is broken.

The other discipline: the tree forces you to quantify things you'd rather leave fuzzy. "Iran will probably keep fighting" is comfortable because it has no commitment. "Iran continues retaliation: 85%" is uncomfortable because it's a specific claim you can be wrong about. That discomfort is the point. If you're not willing to put a number on it, you probably don't understand it well enough to trade it.

## Practical tips from a year of doing this

**Start with 5-8 nodes.** More than that and you spend all your time updating the tree instead of trading. You can always add nodes later when you notice a gap.

**Separate confirmed from estimated nodes.** In my tree, "US initiated strikes" is at 99% and marked confirmed. I don't spend time re-estimating it. This keeps the cognitive load on the nodes that actually move.

**Update weekly, not daily.** Unless there's a major event, updating every day adds noise without adding signal. I do a weekly review where I go through each non-confirmed node and ask: has anything changed?

**Watch the edges, not the nodes.** The tree is an intermediate structure. The thing that matters for PnL is the edge — the gap between your thesis-implied price and the market price. If the market moves toward your thesis, your edge shrinks even if nothing changed in the tree. That's fine. Take profit and look for the next edge.

**Kill the thesis when it's dead.** The hardest part. When your upstream nodes break, the thesis is dead. Don't hold and hope. The tree makes this visible — use it.

## Building this yourself vs using SimpleFunctions

You can absolutely build a causal tree in a spreadsheet. I did that for my first three months. It works. The pain points that eventually pushed me to build tooling around it:

- Manual propagation is error-prone when you have 8+ nodes with conditional dependencies
- Matching nodes to live contract prices requires checking Kalshi every time you update
- Orderbook data (spread, depth, liquidity) changes intra-day and matters for execution
- Tracking which signals affected which nodes over time is tedious but important for post-mortems

SimpleFunctions automates the mechanical parts: propagation, market scanning, orderbook enrichment, signal tracking. You still provide the thesis and the probability estimates. The tool doesn't think for you — it just makes the thinking process faster and less error-prone.

If you want to try it, the CLI handles the full workflow:

```bash
$ npm install -g @spfunctions/cli
$ sf setup
$ sf thesis create "US-Iran conflict drives 2026 recession"
$ sf context <thesis-id> --tree
$ sf edges
```

Or connect it to Claude Code via MCP and let an agent monitor your tree on a 15-minute cycle:

```bash
claude mcp add simplefunctions --url https://simplefunctions.dev/api/mcp/mcp
```

The thesis is yours. The tree is yours. The tool just keeps it updated.

---

*I trade on Kalshi with real money using the approaches described here. This isn't financial advice — it's a description of how I structure my own thinking. If you want to try this workflow, start at [simplefunctions.dev](https://simplefunctions.dev).*