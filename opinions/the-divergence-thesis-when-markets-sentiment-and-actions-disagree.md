# The Divergence Thesis: When Markets, Sentiment, and Actions Disagree

> Every edge in prediction markets comes from disagreement between signal types. Here's the framework for finding them.

**Category:** analysis | **Author:** Patrick Liu | **Reading time:** 8 min

---
Every edge in prediction markets comes from the same source: disagreement between signals.

Not disagreement between traders — that's just a spread. The real alpha is when *types of information* disagree. When what people bet on (belief) contradicts what people say (sentiment) or what institutions do (action).

This is the divergence thesis. And it's the only sustainable edge in a market where [14 of the top 20 most profitable wallets are bots](https://www.financemagnates.com/trending/prediction-markets-are-turning-into-a-bot-playground/).

## The Three Signals

**Belief** is the prediction market price itself. Polymarket, Kalshi, Manifold — real money, real conviction. When "US forces enter Iran by April 30" trades at 67 cents, that's not an opinion. That's $44 million saying "more likely than not."

**Sentiment** is the conversation. X threads, Hacker News comments, Reddit discussions, cable news chyrons, newsletter takes. Sentiment is free to produce and consume. It costs nothing to tweet "Iran war is inevitable." This makes sentiment data noisy, but also fast — sentiment moves before markets.

**Action** is what actually happens. SEC filings. Government contracts. Carrier rerouting. CEO stock sales. Military deployments. Actions are expensive, irreversible, and high-signal. You can't fake a $2 billion defense contract.

## Why Divergence Matters

In efficient markets, these three signals converge. Sentiment moves first (someone tweets about a troop deployment), markets follow (prices adjust), and eventually actions confirm or deny (the troops deploy or don't).

But prediction markets are *not* efficient. They're young, they're illiquid, and participants are biased. [Only 7-13% of human traders achieve positive P&L](https://www.coindesk.com/tech/2026/03/15/ai-agents-are-quietly-rewriting-prediction-market-trading). This means the convergence process is slow and uneven — which creates exploitable divergences.

**Type 1: Price Lag.** News breaks. Sentiment shifts. But the market hasn't moved yet. This happens constantly in thin markets. A SEC filing drops at 4pm and the relevant Kalshi contract doesn't move until the next morning when traders log in.

**Type 2: Sentiment Gap.** Markets price X but the crowd believes Y. Right now, 78% of Hacker News commenters expect Iran military escalation, but ceasefire contracts have been quietly climbing. Either the crowd is wrong (and the market is absorbing diplomatic signals the crowd can't see) or the market is wrong (and sentiment will eventually force a correction).

**Type 3: Information Asymmetry.** Actions reveal what sentiment and markets haven't priced. When defense contractors file unusually large procurement contracts with expedited timelines, that's information the market might take weeks to absorb. The Venezuela collapse was priced on Polymarket [hours before CNN reported it](https://sites.lsa.umich.edu/mje/2026/03/14/prediction-markets-as-truth-machines/) because someone with access to action-level intelligence was betting.

**Type 4: Consensus (anti-divergence).** When all three signals agree, there's no edge. Markets at 67%, sentiment at 78%, and government actions consistent with escalation — this is priced. Don't trade consensus.

## The Cross-Reference Framework

Paradox Intelligence [published the methodology](https://www.paradoxintelligence.com/blog/how-to-use-alternative-data-predict-polymarket-outcomes):

1. Identify the behavioral driver behind the contract
2. Find signals that track that driver (search trends, social volume, app downloads, financial flows)
3. Compare behavioral evidence to the contract price
4. When multiple independent signals disagree with the price, you have a candidate edge
5. Rule out known explanations before sizing the trade

What they didn't build: the infrastructure to do this at scale, across thousands of contracts, continuously.

That's what SimpleFunctions does. The `monitor-the-situation` API scrapes any source (Layer 1: action data), analyzes it with configurable LLMs (Layer 2: sentiment extraction), and cross-references with 9,706 live prediction market contracts (Layer 3: belief prices). The divergence detection is automated.

## Real Example: The Oil-Equity Divergence

As of April 2, 2026:

- **SPY** (S&P 500): +0.81%
- **Oil** (CL): -2.7%
- **Gold** (GLD): +1.82%
- **Iran war probability**: 67% (Polymarket)
- **Gas price above $6 by year-end**: 95% (Kalshi)

Stocks up, oil down, gold up, during active geopolitical escalation. That's three divergences in one session:

1. **Energy-equity split**: If Iran war is 67%, energy should be bid, not offered. Either SPY is ignoring Iran risk or oil is over-discounting ceasefire probability.
2. **Stocks-and-gold both up**: Classic pre-volatility signal. The market is simultaneously risk-on and hedging. Historically, this resolves with a sharp directional move within 5 trading days.
3. **Gas $6 at 95% but $6.80 at 10%**: Markets see a ceiling. But if Hormuz disrupts, that ceiling breaks. The $6.80 contracts might be significantly underpriced.

None of these are predictions. They're observations about where signals disagree. The divergence thesis doesn't tell you which signal is right — it tells you where to look.

## Why Bots Can't Do This (Yet)

The 14 bot wallets dominating Polymarket profit from speed and arbitrage — not from cross-signal divergence detection. They're fast, but they're narrow. They exploit the spread between Yes/No on the same contract, or between Polymarket and Kalshi prices on equivalent events.

Cross-signal divergence detection requires:
1. Consuming unstructured data (news, tweets, government filings)
2. Extracting structured claims with semantic understanding
3. Mapping those claims to relevant prediction market contracts
4. Assessing whether the divergence is meaningful or noise
5. Doing this continuously across hundreds of topics

This is an intelligence problem, not an arbitrage problem. And it's exactly where LLM-powered agents have an edge over pure-quant bots.

## The Actionable Takeaway

Stop looking at prediction markets in isolation. A price is just a number. A price *that disagrees with what you can see happening in the world* is an opportunity.

Use SimpleFunctions' divergence detection to find these disagreements automatically:

```bash
# Cross-reference any text with 9,706 markets
curl -X POST https://simplefunctions.dev/api/monitor-the-situation/enrich \
  -H "Content-Type: application/json" \
  -d '{"content": "your research here", "topics": ["iran", "oil"]}'
```

Or let an agent do it continuously — scrape, extract, enrich, push. $0.007 per cycle. 400 cycles a day for $5/month.

The edge isn't in being faster. It's in seeing what the market hasn't seen yet.

---

*All prices from SimpleFunctions — 9,706 markets across Kalshi and Polymarket. April 2, 2026.*