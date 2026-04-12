# Congruence Classes and Signal: Modular Arithmetic as Attention Compression

> The most powerful operation in number theory is also the most violent: division with remainder. What you throw away defines what you can see.

**Category:** insights | **Author:** SimpleFunctions Research | **Reading time:** 10 min | **Published:** 2026-03-26

---
# Congruence Classes and Signal: Modular Arithmetic as Attention Compression

*The most powerful operation in number theory is also the most violent: division with remainder. What you throw away defines what you can see.*

## The Modular Reduction You Already Perform

Every morning, you open your prediction market portfolio. Polymarket shows 847 active markets. Kalshi lists another 300. Your RSS feed has 200 unread items. Twitter has been running all night.

You do not read everything. You cannot. So you perform an operation that would be immediately recognizable to Gauss: you take a modulus. You compress an infinite stream of information into a finite number of categories — "relevant," "noise," "maybe later" — and you discard the rest. The specific tweet is gone. The structural relationship it carried — bullish, bearish, irrelevant — is preserved.

This is not a metaphor. It is a precise structural correspondence. And understanding it formally changes how you build systems to process prediction market signals.

## Z/nZ: The Integers, Compressed

Take the integers: ..., -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7, ... An infinite set. Now pick a modulus, say n = 3. Every integer gets mapped to its remainder when divided by 3:

- 0, 3, 6, 9, 12, ... → class [0]
- 1, 4, 7, 10, 13, ... → class [1]
- 2, 5, 8, 11, 14, ... → class [2]

The result is Z/3Z, a set with exactly three elements. You have compressed infinitely many integers into three residue classes. This is violent. The number 7 and the number 1 are now the same object. You cannot recover which one you started with.

But here is what survives: *structure*. If a ≡ b (mod 3) and c ≡ d (mod 3), then a + c ≡ b + d (mod 3) and a × c ≡ b × d (mod 3). Addition and multiplication — the fundamental operations — are preserved. Divisibility relations are preserved. The compression is lossy on values but lossless on algebraic structure.

This is the core tradeoff. You choose a modulus. You lose specific numbers. You keep the relationships between them. The question that matters is: *which modulus preserves the structure you actually need?*

## Information Flow as an Infinite Stream

Now map this to prediction markets. Consider a single market: "Will the Fed cut rates at the June 2026 FOMC meeting?" The information relevant to this market is, in principle, infinite. Every economic data release, every Fed governor speech, every inflation print, every employment report, every foreign central bank decision, every geopolitical event that might affect monetary policy — these arrive continuously, at irregular intervals, with wildly varying signal density.

Call this information stream I. It is the integers — unbounded, infinite, arriving sequentially. You cannot store all of it. You cannot process all of it. You must take a modulus.

Your modulus is your *attention frequency*: how often you sample the stream, and at what granularity. This maps I into a finite set of attention classes, exactly as Z maps to Z/nZ. The specific data point vanishes. Its class membership — its residue — is what you trade on.

The deep question is the same one a number theorist faces: what is the right n?

## When the Modulus Is Too Small

Set n = 2. In modular arithmetic, Z/2Z has two elements: even and odd. It is the coarsest nontrivial compression possible. You can tell whether a number is divisible by 2, and nothing else. You cannot distinguish 3 from 7 from 101 — they are all just "odd."

In information processing, this corresponds to checking the news twice a day, or once every twelve hours. Your attention classes are too coarse. A Fed governor speaks at 10 AM, bond yields spike 15 basis points, prediction market prices shift 4 cents, and by the time you sample at 6 PM, the move has been fully absorbed. You see the closing price. You do not see the trajectory. Worse, you cannot distinguish between a market that moved 4 cents on genuine new information and one that moved 4 cents on a rumor that was immediately retracted — because both events fall into the same residue class: "stuff that happened today."

This is not just a speed problem. It is a *structural* problem. In Z/2Z, you have lost all information about divisibility by 3, 5, 7, or any other prime. The congruence structure that would let you decompose a signal into its constituent factors is simply gone. You are trading blind to the internal structure of the information you receive.

Concretely: if you sample a market like "Will Biden pardon Hunter?" every 12 hours during a fast-moving news cycle, you will see the same residue class — "price moved" — whether the move was driven by a credible legal filing, a speculative opinion column, or a bot-driven pump. The modulus is too small to distinguish the generating factors.

## When the Modulus Is Too Large

Now set n = 10,000. In Z/10000Z, you have ten thousand residue classes. You can detect divisibility by every prime up to 10,000. You have extraordinary resolution. You have also made computation nearly impossible.

In information terms, this is tracking every tweet, every order book update, every tick of every related market, every second. Your attention classes are so fine-grained that each one contains at most a single data point. You have not compressed at all. The "residue class" of each piece of information is just... that piece of information. You have performed the identity map and called it analysis.

This fails for two reasons. First, computation: the human brain, or any finite processing system, cannot perform arithmetic in Z/10000Z at the speed the stream demands. You drown. Second, and more fundamentally: *most of those ten thousand classes are structurally identical*. The difference between checking the Fed funds futures at 10:32:07 and 10:32:08 is, for any tradeable purpose, zero. You have created thousands of residue classes that contain no new congruence information. The marginal class is pure noise.

This is the curse of high-frequency data without high-frequency structure. The information production rate of most prediction markets is nowhere near one meaningful signal per second. Sampling at that rate does not give you more structure. It gives you more noise organized into more buckets.

## The Optimal Modulus: Matching Information Production Rate

Here is the key insight: the right modulus is the one where *each residue class contains meaningfully distinct structural information, and adding one more class would only subdivide noise.*

In number theory, this has a precise analog. Consider the problem of detecting whether a number is a perfect square. Working modulo 4, you can immediately reject any number congruent to 2 or 3 (mod 4) — squares are always 0 or 1 (mod 4). Increasing to mod 8 gives you more information (squares are 0, 1, or 4 mod 8). But increasing to mod 1,000,000 gives you diminishing returns relative to the computational cost. There is an optimal modulus for each structural question.

For prediction markets, the structural question is: "Has the probability of this event meaningfully changed?" The information production rate of the underlying event determines how often that question's answer flips from "no" to "yes."

SimpleFunctions' Tavily integration scans news sources on a frequency between 15 minutes and 2 hours. This is not an arbitrary engineering choice. It is a judgment about the information production rate of the events being tracked. For a market like "Will Congress pass the spending bill by March 31?", the relevant information — committee votes, leadership statements, procedural motions — arrives at a rate where 15-minute scans capture each distinct signal as a distinct residue class. Scanning every 30 seconds would create dozens of empty classes between signals. Scanning every 6 hours would collapse multiple distinct signals into one class.

The range of 15 minutes to 2 hours represents a modulus band: n is not fixed but adapts based on the information density of the event. A breaking crisis gets n = 15 minutes. A slow-moving regulatory process gets n = 2 hours. In both cases, the modulus is calibrated so that residue classes map roughly one-to-one onto structurally distinct states of the world.

## The Chinese Remainder Theorem: Multi-Source Reconstruction

Now consider a harder problem. You are tracking a single event — say, "Will the US impose new tariffs on Chinese semiconductors by Q3 2026?" — across multiple information sources. Kalshi has a market. Polymarket has a market. Databento feeds you order flow data. Tavily pulls news articles. Each source provides a partial view: a residue modulo a different prime.

The Chinese Remainder Theorem (CRT) states: if you know a number's residue modulo several pairwise coprime integers, you can uniquely reconstruct that number modulo their product. Formally, if n₁, n₂, ..., nₖ are pairwise coprime, then knowing x mod n₁, x mod n₂, ..., x mod nₖ determines x mod (n₁ × n₂ × ... × nₖ) uniquely.

The prediction market analog is direct. Each information source compresses reality through a different modulus — a different attention structure with different blind spots. Polymarket's order book reflects retail sentiment and crypto-native traders. Kalshi captures regulated US bettors. Databento provides institutional flow data. News articles carry narrative and official statements. Each is a residue: a lossy compression of the same underlying reality through a different modulus.

Individually, each source has blind spots. Polymarket cannot tell you what is happening in closed-door congressional meetings. News articles cannot tell you about quiet institutional positioning. But if the moduli are *coprime* — if the blind spots do not overlap — then the CRT guarantees you can reconstruct far more than any single source provides.

This is exactly what SimpleFunctions' multi-source architecture does. It collects residues from Kalshi, Polymarket, and Databento, along with Tavily news scans, and combines them. The system does not simply average prices. It performs something structurally closer to CRT reconstruction: each source's signal is treated as information modulo that source's particular compression function, and the combination recovers information that no individual source contains.

The critical requirement of CRT — that the moduli be pairwise coprime — maps to the requirement that information sources be *genuinely independent*. Two sources that follow each other (a news aggregator that just repackages Reuters, two prediction markets with heavy arbitrage between them) are like taking residues mod 6 and mod 3. Since 3 divides 6, the second residue provides zero new information. The mod-3 residue is already determined by the mod-6 residue. You need sources whose compression functions are structurally different, not just differently branded.

## Adaptive Modulus: The Heartbeat Engine

In pure mathematics, you pick a modulus and it stays fixed. In signal processing for live markets, the information production rate is nonstationary. A market can sit quiet for three weeks and then produce more signal in one afternoon than in the prior month.

SimpleFunctions' heartbeat engine addresses this by making the modulus adaptive. The scan frequency — the interval between Tavily queries, the polling rate for market prices — adjusts based on detected volatility. When a market's price has been stable within a 2-cent band for a week, the modulus is large (scan every 2 hours). When price moves exceed a threshold within a scan window, the modulus shrinks (scan every 15 minutes).

This is the information-theoretic analog of working in Z/nZ where n is chosen dynamically based on the density of interesting residues. If you are studying a function and discover that its behavior mod 7 reveals important structure, you refine your analysis to mod 49 or mod 7² to see finer detail within that prime's contribution. If mod 11 is showing nothing, you stop computing it. The modulus is a resource allocation decision, and the optimal allocation changes as the structure of the problem reveals itself.

Practically, this means the system's attention is not uniform across markets. A quiet market for Swiss GDP growth gets sampled infrequently — a large modulus that compresses many hours into a single "no change" class. A volatile market for a contested election gets sampled every 15 minutes — a small modulus that preserves the fine structure of rapid information arrival. The total computational budget is fixed (you have finite API calls, finite processing time), so the system performs a continuous reallocation, shrinking moduli where information is dense and expanding them where it is sparse.

## Quotient Structures and Trader Attention

There is one more algebraic concept worth extracting. When you form Z/nZ, you are constructing a *quotient group* — you are dividing Z by the subgroup nZ (all multiples of n). The resulting structure is not just a set; it inherits the group operation from Z. You can add residue classes, and the result is well-defined.

The trader analog: your attention classes are not just categories. They support operations. You can combine two signals — "Fed governor hawkish" + "employment data strong" — and the result ("rate cut probability decreasing") is well-defined within your attention structure. This works precisely because your modulus preserves the additive structure. If your compression were arbitrary — if your categories were random subsets of information rather than congruence classes — then combining signals within categories would not produce meaningful results.

This is why principled compression matters more than aggressive compression. A bad modulus (categories that cut across the natural structure of the information) destroys the ability to reason compositionally. A good modulus (categories aligned with the information's algebraic structure) lets you perform valid inference within the compressed representation. The residue class is a legitimate object of computation, not just a label.

## The Fundamental Theorem of Arithmetic, Restated

The Fundamental Theorem of Arithmetic says every integer greater than 1 factors uniquely into primes. Primes are the atoms. Modular arithmetic lets you study one prime factor at a time.

The analog for signal processing: every market-moving event has underlying causal factors — economic data, political dynamics, legal rulings, market microstructure. These are the primes. Your multi-source architecture (CRT) lets you isolate them. Your adaptive modulus (heartbeat engine) lets you allocate attention to the factors that are currently active. And the algebraic structure of your residue classes (quotient group) lets you recombine partial information into tradeable conclusions.

The entire pipeline — from Tavily scan to Kalshi order to portfolio adjustment — is, at its core, a carefully chosen chain of modular reductions. Infinite information, finite attention, preserved structure.

The modulus is the mechanism. Choose it well.