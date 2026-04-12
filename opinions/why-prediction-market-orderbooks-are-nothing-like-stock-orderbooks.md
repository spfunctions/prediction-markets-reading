# Why Prediction Market Orderbooks Are Nothing Like Stock Orderbooks

> Every price is a probability. Every order is a belief statement. Every spread is a disagreement about the future. Prediction market microstructure operates on fundamentally different logic than equities — and the traders who understand that difference are the ones extracting alpha.

**Category:** analysis | **Author:** SimpleFunctions | **Reading time:** 15 min

---
You open a Kalshi orderbook and see 3,055 contracts resting at 27 cents on KXWTIMAX-26DEC31-T160. If you've traded equities, your instinct is to read this as "support at $0.27." That instinct is wrong — and the mistake will cost you money.

Those 3,055 contracts are not support in the equity sense. They are a collective statement by a group of human beings who believe the probability that WTI crude peaks above $160 by December 31, 2026 is *at least* 27%. Every single one of those contracts represents someone who ran a mental model — geopolitical escalation, Hormuz closure duration, SPR depletion, OPEC capacity constraints — and arrived at a number that justified paying 27 cents for a contract that pays $1 if the outcome occurs and $0 if it doesn't.

This is not how stock orderbooks work. And the differences are not cosmetic. They are structural, pervasive, and exploitable.

## Binary Settlement Changes Everything

A stock's orderbook is denominated in dollars and the price reflects a consensus valuation of future cash flows, discounted by risk. The buyer at $150 and the seller at $151 disagree about valuation by less than 1%. The stock might be worth $140 or $160 — nobody knows — but the orderbook captures a narrow band of disagreement around a shared uncertainty.

A prediction market orderbook is denominated in probability. The buyer at 27 cents and the seller at 33 cents don't disagree about valuation. They disagree about *reality*. The buyer says: "There is at least a 27% chance WTI peaks above $160." The seller says: "There is at most a 33% chance." The spread between them — 6 cents — is not a valuation gap. It is a *belief gap*. A disagreement about the likelihood of a future event.

This distinction cascades through every aspect of orderbook behavior.

## Depth as a Belief Distribution

In equity markets, depth at a price level tells you about liquidity — how many shares you can trade without moving the price. The distribution of depth is shaped by market makers, algorithmic rebate-seekers, and institutional iceberg orders. It tells you almost nothing about what participants *think* the stock is worth.

In prediction markets, depth at a price level tells you about *conviction*. Look at the KXWTIMAX-26DEC31 contract family:

| Strike | Best Bid | Bid Depth | Best Ask | Ask Depth |
|--------|:--------:|:---------:|:--------:|:---------:|
| T135   | 42¢      | 657       | 45¢      | 1,485     |
| T150   | 30¢      | 1,220     | 34¢      | 890       |
| T160   | 27¢      | 3,055     | 33¢      | 1,810     |
| T180   | 15¢      | 480       | 22¢      | 2,340     |

The 3,055 contracts at 27 cents on T160 represent the *mass of belief* at that probability level. These are traders who've concluded, through whatever analysis they've done, that 27% is a fair price for WTI $160. The depth at that level is not just liquidity — it's a census of a particular probabilistic belief.

Compare T150 (bid depth 1,220) with T160 (bid depth 3,055). More traders are willing to commit capital at T160's 27 cents than at T150's 30 cents. Why? Because T160 at 27 cents offers a 3.7:1 payoff ($0.73 profit per $0.27 risked), while T150 at 30 cents offers 2.3:1. The deeper depth at T160 isn't about the price being more "correct" — it's about the payoff structure attracting more participants who see asymmetric upside.

## The Round Number Effect

Pull up any prediction market orderbook and you'll notice something immediately: depth clusters at round-number prices. 25 cents, 30 cents, 50 cents, 75 cents. This isn't random. It's a direct consequence of how humans think about probability.

Nobody thinks "I estimate a 27.3% probability." People think in integer odds:

- **25¢ = 3:1 payoff.** "I'm risking 1 to make 3. That's attractive if I think there's at least a 1-in-4 chance."
- **33¢ = 2:1 payoff.** "I'm risking 1 to make 2. Decent if the odds are roughly 1-in-3."
- **50¢ = 1:1 payoff.** "Even money. I think it's a coin flip."

These clean payoff ratios attract disproportionate order flow. A trader who privately believes the probability is 28% will often place their order at 25 cents rather than 28 cents — because 3:1 is a *concept* they can anchor to, while 2.57:1 is just a number. The result is that round-number prices accumulate depth from traders whose true beliefs span a range around that level.

This creates exploitable dynamics. The gaps *between* round numbers are often thinner than they should be. If you see heavy depth at 25 cents and heavy depth at 30 cents but almost nothing at 27 cents or 28 cents, you're looking at a belief vacuum — a range where few traders have anchored their views. Placing orders in these gaps can capture spread that round-number clustering leaves on the table.

## Settlement Arithmetic and Clean Payoffs

The round-number effect isn't just psychological. It's reinforced by the settlement arithmetic of binary contracts.

When you buy a YES contract at 25 cents:
- You pay $0.25
- If YES, you receive $1.00, profit $0.75 — a **3:1 payoff**
- If NO, you lose $0.25

When you buy at 33 cents:
- Profit if YES: $0.67 — a **2:1 payoff**

When you buy at 50 cents:
- Profit if YES: $0.50 — a **1:1 payoff**

These are the "Schelling points" of prediction market orderbooks. Just as game theory predicts coordination around focal points, prediction market traders coordinate around prices that produce clean payoff ratios. The 25-cent level in particular acts as a magnet because 3:1 is the threshold where many risk-averse traders start to find the bet "worth it" even under significant uncertainty.

This means that depth at 25 cents is systematically *overstated* relative to depth at, say, 23 cents or 28 cents. The belief distribution isn't actually concentrated at 25% probability — it's spread across a range, but the orders cluster at the focal price. Sophisticated traders exploit this by placing orders at 26 cents or 27 cents, capturing fills from the bleedover of traders who are approximately-but-not-exactly anchored at 25 cents.

## Thesis-Driven Clustering

Stock orderbooks are dominated by market makers and algorithms that care about microstructure, not fundamentals. The depth at $150.05 on Apple stock tells you nothing about what anyone thinks Apple is worth as a business.

Prediction market orderbooks are thesis-driven. Groups of traders reading similar analysis — the same Bloomberg article about OPEC supply constraints, the same Argus crude oil report, the same CENTCOM press release about Hormuz mine-clearing operations — arrive at similar probability estimates and cluster their orders at similar prices.

You can sometimes *see* a thesis in the orderbook. When KXWTIMAX-26DEC31-T150 shows a sudden block of 800 new contracts at 32 cents, that's not an algorithm adjusting its hedges. That's a group of traders who probably read the same IEA report that morning, ran the same back-of-envelope supply math, and concluded that 32% is about right for WTI $150.

This thesis-driven clustering means that orderbook depth in prediction markets is *informational*. It tells you what the marginal trader believes and, by extension, what analysis they've consumed. When depth shifts — 500 contracts disappear from the 30-cent bid and reappear at 25 cents — it means the thesis holders just downgraded their probability estimate by 5 points. Something changed in their model of the world.

## Bid/Ask Asymmetry as Directional Sentiment

In equity markets, bid/ask asymmetry is primarily a function of market-maker inventory and order flow imbalances. A heavier ask than bid might mean the market maker is short and needs to sell, or that a large institutional seller is working an order. It's a *flow* signal, not a *belief* signal.

In prediction markets, bid/ask asymmetry is a direct readout of directional sentiment. Look at KXWTIMAX-26DEC31-T135:

- Bid depth: 657 contracts
- Ask depth: 1,485 contracts
- Ratio: sellers are **2.3x more aggressive** than buyers

This means there are 2.3 times more traders willing to sell the "WTI above $135" outcome than to buy it. In probability terms: the marginal seller thinks T135 is overpriced at 45 cents (too high a probability), while relatively few buyers think it's underpriced at 42 cents. The market is directionally bearish on this strike — more participants think WTI will *not* exceed $135 than think it will.

Compare with T150:
- Bid depth: 1,220 contracts
- Ask depth: 890 contracts
- Ratio: buyers are **1.37x more aggressive** than sellers

Here the asymmetry flips. At the T150 level, more traders are willing to commit capital on the YES side at 30 cents than on the NO side at 34 cents. The market is comparatively *more bullish* at this strike than at T135 — which is counterintuitive until you realize that the lower absolute price (30 cents vs 42 cents) offers a much better payoff ratio, attracting conviction buyers who see tail risk.

Reading these asymmetries across the entire strike ladder gives you a directional sentiment map that doesn't exist in equity orderbooks.

## The Spread Is Probability Disagreement

In equities, the spread is a transaction cost. It's the price you pay for immediacy. Tight spreads mean liquid markets. Wide spreads mean illiquid markets. The spread tells you about *market quality*, not about *disagreement*.

In prediction markets, the spread is literally the gap between the most optimistic buyer's belief and the most pessimistic seller's belief. A 6-cent spread on T160 (bid 27, ask 33) means the most aggressive buyer thinks there's at least a 27% chance and the most aggressive seller thinks there's at most a 33% chance. Nobody in the market is willing to assert a probability between 27% and 33%.

This makes spread dynamics uniquely informative:

**Spread narrowing** means beliefs are converging. Buyers and sellers are moving toward consensus on the probability. This typically happens when new information clarifies the situation — an OPEC meeting with a clear outcome, a definitive military action, an unambiguous economic data release.

**Spread widening** means beliefs are diverging. Buyers and sellers are moving *apart* in their probability estimates. This happens before major news events — informed traders withdraw their limit orders, widening the spread, because they don't want to be on the wrong side of an information shock.

Here's the critical insight: **spread widening before news is itself a signal.** When you see the spread on KXRECSSNBER-26 widen from 4 cents to 11 cents 48 hours before a CPI release, informed traders are telling you they expect the CPI number to move this market significantly. They're pulling liquidity because the risk of being adversely selected exceeds the reward of capturing spread. The widening spread is a measure of *expected information content* of the upcoming event.

## Time Decay and the Expiry Effect

As a prediction market contract approaches its settlement date, two opposing forces act on the spread:

**If the outcome is becoming clear:** The spread narrows dramatically. If WTI is at $170 with 2 weeks left, the T160 contract converges toward 95-99 cents with a 1-2 cent spread. The probability is near-certain, and even traders who were skeptical have little basis for disagreement.

**If the outcome remains uncertain:** The spread *widens*. If WTI is at $148 with 2 weeks left, the T150 contract might sit at 45 cents with an 8-cent spread. The outcome is genuinely uncertain, and near-expiry uncertainty amplifies disagreement rather than resolving it.

This is the opposite of options markets, where theta decay is monotonic and predictable. In prediction markets, time decay is path-dependent. The same contract can have a narrowing spread or a widening spread near expiry, depending on the state of the underlying outcome.

## Cross-Strike Coherence (and When It Breaks)

A prediction market with multiple strikes on the same underlying must satisfy basic probability axioms. For WTI max-price contracts:

P(WTI > $135) ≥ P(WTI > $150) ≥ P(WTI > $160) ≥ P(WTI > $180)

If WTI exceeds $160, it has necessarily exceeded $135. So the probability of exceeding a lower threshold must always be greater than or equal to the probability of exceeding a higher threshold.

In practice, momentary violations occur. You might see:
- KXWTIMAX-26DEC31-T150 bid: 30¢
- KXWTIMAX-26DEC31-T135 ask: 28¢

This is a coherence violation: you can buy T135 YES at 28 cents and sell T150 YES at 30 cents. If WTI peaks above $150, both settle YES and you profit 2 cents risk-free. If WTI peaks between $135 and $150, T135 settles YES (you get $1) and T150 settles NO (you lose $0.30), for a net profit of $0.42. The only scenario where you lose is if WTI doesn't exceed $135 — but even then, you lose only 28 cents (on T135) and gain 30 cents (T150 settles NO, you keep the sale proceeds). It's a risk-free arbitrage.

These violations don't persist in equity options because arbitrageurs with co-located servers close them in microseconds. In prediction markets, they can persist for *hours*. The structural arb traders who monitor cross-strike coherence capture risk-free cents repeatedly throughout the day.

## The Participant Hierarchy

Every prediction market orderbook has three layers of participants, and they interact differently than in equity markets:

**Noise traders** arrive in bursts. They saw a headline, formed an opinion, and submitted a market order. They consume liquidity, move the price, and disappear. Their orders are typically small (10-50 contracts), arrive in clusters after news events, and have no persistence. In equity markets, noise traders are swallowed by the ocean of algorithmic liquidity. In prediction markets, where total depth might be 3,000 contracts, a noise trader buying 200 contracts at market can move the price by 3-5 cents.

**Analytical traders** place limit orders based on a thesis. They provide the stable depth that defines the orderbook's structure. Their orders are larger (100-1,000 contracts), persist for days or weeks, and shift only when their underlying analysis changes. These are the traders whose orders create the belief distribution described above.

**Structural traders** don't have a view on the outcome at all. They monitor cross-strike coherence, bid/ask asymmetries, and depth imbalances. They capture risk-free or near-risk-free cents by correcting orderbook inconsistencies. Their presence improves market efficiency but their orders are small and transient.

## The Information Flow Cycle

When new information hits a prediction market, the cycle is distinctive:

1. **News breaks.** A CPI print, a military action, an OPEC announcement.
2. **Noise traders sweep.** Market orders consume the top of the book. Price moves 3-8 cents in seconds.
3. **Spread widens.** Analytical traders pull their limit orders to avoid adverse selection. The spread blows out from 4 cents to 10+ cents.
4. **Analytical traders re-enter.** Over the next 2-6 hours, analytical traders digest the news, update their models, and place new limit orders at updated price levels. The spread narrows, but the price settles at a new level.
5. **Structural arbitrage.** Cross-strike violations created by the uneven repricing get closed by arb traders. Coherence is restored.
6. **New equilibrium.** After 12-48 hours, the orderbook reflects the new information with tight spreads, updated depth, and cross-strike consistency.

In equity markets, this entire cycle takes 5-30 seconds. In prediction markets, it takes *hours to days*. The difference is entirely about participant density and automation. Equity markets have thousands of algorithms processing each piece of information simultaneously. Prediction markets have a few hundred human traders who need to read the news, think about it, update their models, and manually place orders.

## The Key Alpha: Price Impact Recovery Time

This leads to the single most important structural difference between prediction market and equity orderbooks: **price impact recovery time.**

In equity markets, a large market order might push the price 0.1%. Within 5 minutes — often within 30 seconds — the price recovers as market makers and arbitrageurs refill the book. The "half-life" of price impact is measured in seconds.

In prediction markets, a large market order that pushes the price 5 cents might not fully recover for *24-48 hours*. The analytical traders who provide the stable depth aren't sitting in front of screens 24/7. They have jobs. They sleep. They need time to process the new information and decide whether the new price is correct or an overreaction.

This extended recovery time is exploitable in both directions:

- **If the price move is legitimate** (driven by real information), you have a 24-48 hour window to enter before the full repricing is complete.
- **If the price move is noise** (driven by a burst of uninformed market orders), you have a 24-48 hour window to fade the move before the analytical traders come back and restore the previous level.

Either way, the slow recovery is alpha. In equities, this alpha was arbitraged away decades ago. In prediction markets, it persists because the participant base is thin, fragmented, and human.

---

## Implications for Traders

If you're coming from equity trading and moving to prediction markets, the adjustments are fundamental:

1. **Read depth as belief, not liquidity.** A 3,000-contract level isn't "support" — it's a probability consensus.
2. **Watch the spread for information, not just cost.** Widening spreads before events are signals, not just inconveniences.
3. **Monitor cross-strike coherence.** Violations are free money, and they persist far longer than in equities.
4. **Time your entries to the information cycle.** The 24-48 hour recovery window after a price impact is the primary trading opportunity.
5. **Understand the participant hierarchy.** When noise traders sweep the book, the analytical traders haven't changed their minds — they've just temporarily withdrawn.

The prediction market orderbook is not a worse version of an equity orderbook. It's a fundamentally different instrument — one where every price is a probability, every order is a belief, and every microstructure pattern carries information about how the market's collective model of reality is updating.

The traders who understand this are the ones extracting alpha from a market that most equity-trained participants misread completely.