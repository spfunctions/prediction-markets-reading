# I automated my Kalshi thesis with a causal tree. Here's what I learned in 3 months.

> Externalizing your thesis into a trackable causal structure changes how you think — not just how you trade.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
In January, Iran got bombed. I had a view. I wanted to track it.

Not "track" in the sense of checking Kalshi every morning with coffee. Track in the sense of: I have a directional thesis about a geopolitical event cascading into oil prices cascading into recession probability, and I want a machine to tell me when my thesis is wrong, when the edge has moved, and when I should get out.

I'm Patrick. I built SimpleFunctions. I trade on Kalshi with real money. This is what happened when I tried to automate my own thesis — what worked, what didn't, and the one insight that made it worth doing.

## The impulse

The Iran strikes hit in late December 2025. Within 48 hours I had a view: the conflict would persist, Hormuz would stay disrupted, oil would stay elevated, and the US would tip into recession by late 2026. I started buying YES on recession contracts and oil ceiling contracts.

The problem was obvious within a week. I was checking five different Kalshi tickers manually, reading Reuters and Bloomberg for updates, mentally adjusting my confidence, and then... not adjusting my positions. The thesis lived in my head. Updates arrived through vibes. Position sizing was whatever felt right at the time.

I had edge — I genuinely believed the market was underpricing recession risk given the Hormuz disruption. But I had no system for tracking whether that edge still existed tomorrow, or next week, or after the next diplomatic signal.

## Building the causal tree

The first thing I did was decompose my thesis into a tree. Not a fancy model — just the causal chain written out with probabilities:

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

Each node has a probability. Each node is causally linked to the nodes below it. If n1.2 drops — say Iran signals de-escalation — everything downstream updates: Hormuz reopens faster, oil drops, recession probability falls.

This was the first thing that changed my thinking. When you write the tree out, you immediately see which nodes you're most uncertain about. I was 99% sure the strikes happened (obviously) and 99% sure mines were deployed (satellite imagery). But n3 — oil stays elevated — was only 64%. That's where my thesis was weakest. That's where I should be paying the most attention.

## Mapping to real contracts

The tree is abstract. The contracts are concrete. I mapped each relevant node to Kalshi tickers:

- **n3 (Oil stays elevated):** `KXWTIMAX-26DEC31-T135` — WTI crude maximum price above $135 in 2026
- **n4 (Recession):** `KXRECSSNBER-26` — US recession as declared by NBER in 2026
- **Gas downstream:** `KXAAAGASM` — National average gas price contracts

The edge calculation was straightforward: my thesis implies a probability for each contract. The market has its own price. The difference is the raw edge.

```
$ sf edges

  KXRECSSNBER-26 YES     35¢  thesis: 72¢  edge: +37  exec: +36  spread: 1¢  liq: high
  KXWTIMAX-26DEC31-T135  38¢  thesis: 75¢  edge: +37  exec: +36  spread: 1¢  liq: high
  KXAAAGASM YES           14¢  thesis: 55¢  edge: +41  exec: +39  spread: 3¢  liq: med
```

The gas contract had the highest raw edge (+41) but only medium liquidity and a 3-cent spread. The recession contract had slightly lower raw edge (+37) but a 1-cent spread and high liquidity. In practice, recession was the better entry.

I put real money on both.

## The heartbeat

Every 15 minutes, the system runs a heartbeat cycle. It scans news sources for signals relevant to the causal tree, refreshes contract prices, enriches orderbook depth, and evaluates whether any node probabilities need updating.

Here's what a real heartbeat output looks like when something actually happens:

```
[2026-01-18T14:30:00Z] heartbeat
  scan: 3 new signals
    Reuters: "IRGC claims second wave of Hormuz mine deployment"
    Bloomberg: "SPR drawdown pace exceeds 2022 levels"
    Kalshi: KXWTIMAX-26DEC31-T135 moved 38¢ → 42¢
  evaluation:
    n2.1 (mines deployed) confirmed → 99% (no change)
    n2.3 (minesweeping 3+ mo) → 91% (was 87%, second wave extends timeline)
    n3   (oil elevated) → 68% (was 64%, SPR depletion reduces buffer)
  edges recalculated:
    KXRECSSNBER-26: +37 → +34 (market caught up 3¢)
    KXWTIMAX-26DEC31-T135: +37 → +33 (market moved 4¢)
  positions:
    KXRECSSNBER-26 YES: 200 contracts @ avg 34¢, mark 35¢, P&L +$2.00
    KXWTIMAX-26DEC31-T135 YES: 150 contracts @ avg 37¢, mark 42¢, P&L +$7.50
```

Nothing dramatic. The system saw signals, updated two nodes slightly, recalculated edges, and reported that the market was catching up to my thesis on oil. The edge was shrinking but still substantial. No action required.

Most heartbeats look like this — nothing changed, edges stable, positions fine. That's the point. The system watches so I don't have to.

## What went wrong: confirmation bias

About three weeks in, I realized the causal tree had a problem. Every signal that confirmed my thesis got incorporated. Signals that challenged it got... not exactly ignored, but under-weighted.

Here's a concrete example. In early February, there was a back-channel diplomatic signal — reported by FT, citing unnamed sources — that Iran might be open to Hormuz de-escalation in exchange for sanctions relief. My n1.3 ("no diplomatic exit") should have dropped from 82% to maybe 65%. Instead, I found myself reading the article skeptically: "unnamed sources," "might be open to," "back-channel." I left n1.3 at 78%.

The problem was structural, not psychological. The tree had no built-in mechanism for falsification. Every node had a probability, but no node had a "what would make me revise this below 50%?" threshold. I was looking for confirmation because the tree didn't force me to look for disconfirmation.

## Adding adversarial search

The fix was adding adversarial search to the heartbeat cycle. For each node above 70%, the system now actively searches for evidence *against* the claim. Not just "what happened related to Hormuz" but "what evidence exists that Hormuz will reopen sooner than expected?"

This changed things meaningfully. The adversarial search surfaced a Lloyd's of London assessment in mid-February that minesweeping technology had improved since 2022 and that the 3+ month timeline might be optimistic. My n2.3 dropped from 91% to 78%. Downstream, oil-elevated dropped slightly. Recession dropped a point.

Small moves. But the right direction. Without adversarial search, I would have missed the Lloyd's report entirely — it wasn't in my usual Reuters/Bloomberg scan, and it challenged my thesis, so I wouldn't have gone looking for it.

## What didn't work: coarse nodes

Some nodes were too coarse. "Oil stays elevated" (n3) was doing too much work. Elevated relative to what? $80? $100? $120? The node was trying to capture a continuous variable as a binary, and it made edge calculations fuzzy.

I eventually split n3 into three sub-nodes:

```
n3  Oil stays elevated
├── n3.1  WTI stays above $100     82%
├── n3.2  WTI hits $120+           71%
└── n3.3  WTI hits $150+           38%
```

Each sub-node mapped cleanly to a specific Kalshi strike. The edge calculations became sharper. I could see that the market was pricing $100+ oil about right but significantly underpricing the $120-$135 range.

Lesson: if a node doesn't map to a specific contract, it's probably too abstract. Split it until each leaf corresponds to something you can trade.

## Thesis confidence lagged reality

The biggest systematic problem was latency. My thesis confidence updated on a 15-minute heartbeat, but reality moved in seconds. When a major diplomatic development dropped — say, a UN Security Council emergency session — the market moved immediately. My thesis confidence adjusted on the next heartbeat cycle, 8 minutes later. By then, the edge had already compressed.

This isn't really fixable without making the system event-driven rather than polling-based. I chose to accept the latency. The thesis is a medium-frequency instrument. If you're trying to capture intra-minute moves, you need a different system. The causal tree is for finding 20-40 cent edges and riding them for weeks, not for scalping.

I lost roughly $40 on two occasions where the market moved against my thesis faster than the heartbeat could update. Annoying but not structural. The system made back multiples of that on edges it found that I never would have found manually.

## Three months of P&L

I'm not going to show exact numbers because I don't think P&L is the interesting part. What I will say:

- The thesis was net profitable over three months.
- The biggest winner was KXRECSSNBER-26, which I entered at 34 cents and which moved to the high 60s as the oil situation persisted.
- The biggest loser was a gas contract (KXAAAGASM series) where I overestimated the pass-through speed from crude to retail gas prices. The causal link was real but slower than I modeled.
- The adversarial search prevented at least two bad entries where I would have doubled down on weakening nodes.

The dollar amounts matter less than this: every decision was traceable. I could look at any position and see exactly why I entered, which nodes supported it, what changed, and whether the edge still existed. No vibes. No "I just felt like recession was coming." Everything was in the tree.

## The real insight

Three months in, here's what I actually learned: the causal tree isn't primarily a trading tool. It's a thinking tool.

When your thesis lives in your head, you don't know what you believe. You have a vague sense that "things are bad" or "recession is coming" but you can't articulate the causal chain, you can't identify the weakest link, and you can't tell when you're wrong.

When your thesis lives in a tree — with explicit nodes, explicit probabilities, explicit causal links — you are forced to think clearly. You see where you're most uncertain. You see where the market disagrees with you. You see which upstream events would kill your thesis.

This changes how you think, not just how you trade. I found myself reasoning more carefully about geopolitics in general because I had a framework for decomposing complex events into testable claims. Friends would say "I think the war is going to escalate" and I'd ask "which specific node? Why? What would change your mind?" Annoying at dinner parties. Useful for trading.

## What I'd do differently

If I started over:

1. **Finer-grained nodes from the start.** Don't put "oil stays elevated" as a single node. Decompose immediately.
2. **Adversarial search from day one.** Don't wait until you catch yourself confirmation-biasing. Build disconfirmation into the system.
3. **Accept the latency.** The 15-minute heartbeat is fine for thesis-level trading. Don't try to make a causal tree trade like a market maker.
4. **Track node accuracy.** I wish I'd logged every probability estimate alongside what actually happened, so I could calibrate my own judgment over time. (This is something I'm building into SF now.)

## The tool

I built SimpleFunctions to do this. Not because I wanted to build a product — I wanted to track my own thesis. The tool exists because the problem is real: unstructured thesis → biased updates → bad sizing → confused exits.

If this resonates, you can try it at [simplefunctions.dev](https://simplefunctions.dev).