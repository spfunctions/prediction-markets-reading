# Why "Thesis Confidence" Is Not the Same as Market Price

> A 70% subjective conviction about an outcome and a 70-cent market price are not the same number. The market price is the capital-weighted aggregate of every trader who has put real money on it. Your conviction is one input among thousands. Conflating them is the most expensive mistake new PM traders make.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
Pick any prediction-market beginner's discussion thread and you will eventually see the following argument. "I am 70% confident Powell stays as Fed Chair. The market is at 78 cents. So the market is 8 cents overvalued and I should sell." A few replies later someone else writes the symmetric version. "I am 70% confident Powell stays. The market is at 62 cents. So I have an 8-cent edge and I should buy."

Both of these arguments are wrong in the same way. They treat *subjective conviction* and *market price* as values on the same scale, with the implication that the difference between them is straightforwardly your edge. That implication only holds in a very specific set of circumstances, and most beginners are not in those circumstances when they make the trade.

This page is about the gap between thesis confidence and market price, when you can treat the gap as edge, when you cannot, and how to size into the difference when it is real.

## The Steelman: Subjective Edge Is Real

I want to make sure I am not strawmanning this. Subjective probability is a valid way to express belief, and the difference between your subjective probability and the market's implied probability is, in some sense, the only place edge can come from. If you and the market agreed perfectly on every number, there would be nothing to trade. Disagreement is the prerequisite for any trade.

The issue is not whether subjective edge exists. The issue is what the market price actually represents and how your subjective probability should be combined with it.

## What the Market Price Is

The market price of a binary prediction-market contract is, in the limit, the *capital-weighted aggregate of every active trader's subjective probability*. Each participant brings some amount of capital and some belief, and the equilibrium price clears so that the marginal trader is indifferent between buying and selling at that level. The price is not "what the market thinks." The price is "the level at which buyers and sellers are exactly balanced, given how much money everyone is willing to put behind their views."

This has three implications that are uncomfortable for beginners.

First, the price already incorporates the beliefs of every trader who has been active on the contract. Including traders with much more information than you. Including traders with much more capital than you. Including traders who have spent their careers studying exactly this kind of event. If your view is "Powell stays at 70%," that view is already represented in the price *if anyone with the same view has put money on the contract*. The price is not "the market does not know about your view." It is "the market has aggregated views similar to yours along with all the other views, and this is where they balance."

Second, the *mean* belief and the *price* are not necessarily equal. If sentiment is bimodal — half the traders think Powell stays at 95%, half think he leaves at 95% — the mean belief is 50%, but the equilibrium price could clear anywhere between depending on capital distribution. The price is an aggregate, not an average. Pretending the price is "what the average trader believes" is mathematically wrong.

Third, the price is *sensitive to capital distribution*, not just to beliefs. A small number of well-capitalized traders can move the price away from the population mean. This is the wisdom-of-crowds caveat — crowds are wise when the participants are diverse and roughly equally weighted, and they are dumb when a few participants dominate. PMs are sometimes one and sometimes the other, depending on the contract. See [prediction-markets-vs-polls](/opinions/prediction-markets-vs-polls) for the long version of when prediction markets beat polls and when they do not.

## When Your Conviction Is Not the Same as the Price

Once you internalize what the market price actually represents, the "I think it's 70% so the price should be 70¢" framing becomes obviously incomplete. The right framing is closer to: "the market price reflects everyone else's view weighted by their capital. My view is one more input. The question is how much weight I should give my view relative to everyone else's."

In most cases, the answer is "less weight than you instinctively want to give it." Here are the three scenarios where your conviction diverges from the market and how to think about each.

**Scenario 1: You have private information.** You know something the market does not know yet. A friend at the Fed told you Powell is committing today. You saw a news leak before the wire picked it up. Your model identifies a base rate the market is missing. In this scenario, your conviction is genuinely a different number than the price, *because* the market does not have your information. Trade hard. This is the original case for why information traders make money on prediction markets.

**Scenario 2: You think the market is processing public information badly.** You see the same news everyone else sees, but you think most traders are anchoring on the wrong reference point or weighting recent events too heavily. This is sometimes a real edge — markets really do overreact and underreact — but it is much harder to monetize than the private-information case, because you are not bringing new information; you are bringing better processing. Better processing only matters if you can act before other better-processing traders also notice. The window is short and the edge is thin.

**Scenario 3: You just feel strongly about it.** You have no private information and no specific claim about market processing — you just have a high subjective probability because the outcome feels obvious to you. This is the most common case, and it is the case where subjective conviction is *least* a reason to trade. The market has aggregated thousands of "feels obvious" votes already. Your additional vote does not move the price unless you put real capital behind it, and putting real capital behind a "feels obvious" view is how beginners learn the wisdom-of-crowds lesson the expensive way.

## The Sizing Rule

Once you have decided your conviction is a real edge (scenario 1 or scenario 2, not scenario 3), the question is how to size the position. The naive rule is "edge equals your_p minus market_p, position size is proportional to edge." This is approximately right but missing a critical multiplier.

The full version is something like:

```
edge = your_p − market_p
risk_adjusted_edge = edge × confidence_in_your_p × (1 − ambiguity_discount) × liquidity_factor
position_size = risk_adjusted_edge × kelly_fraction × bankroll
```

The terms after `edge` matter as much as `edge` itself.

`confidence_in_your_p` is between 0 and 1, and it answers "how sure are you that your subjective probability is the right one." If you would not bet your house on the value, it is below 0.5. Most actual subjective probabilities have confidence well below 0.5 and the trader has not introspected.

`ambiguity_discount` is the haircut for resolution risk — how likely is it that the market resolves in some weird way that does not match your interpretation of the question. UMA disputes on Polymarket, edge-case rule interpretations on Kalshi. This is the [resolution-ambiguity-score](/concepts/resolution-ambiguity-score) territory.

`liquidity_factor` is the haircut for "even if my edge is real, can I express it." A 10-cent edge on a contract with 12 cents of total depth is a 1.2-cent edge in practice. This is the [liquidity-availability-as-the-real-edge](/opinions/liquidity-availability-as-the-real-edge) territory and the [liquidity-availability-score](/learn/liquidity-availability-score) glossary entry.

`kelly_fraction` is the position-sizing multiplier from the Kelly criterion — see [position-sizing-kelly-criterion-prediction-markets](/technicals/position-sizing-kelly-criterion-prediction-markets). Most professional traders use a fraction well below the full Kelly value because full Kelly is only correct under assumptions that are usually violated in practice.

When you put all these multipliers together, the size of the position you should actually take is *much smaller* than the naive "I think it's 70% and the market is at 60% so I have a 10-cent edge" framing would suggest. By the time you have honestly haircut for confidence, ambiguity, liquidity, and Kelly fractionality, the 10-cent "edge" has shrunk to maybe a 1- or 2-cent expected value per unit, and the position size is bounded by liquidity rather than by the headline edge.

## A Worked Example

Take a hypothetical Kalshi contract on "Powell stays as Fed Chair through 2026" trading at 0.62. Your subjective probability is 0.72. Naively that is a 10-cent edge, and the naive trade is "buy YES at 0.62, hope it goes to 0.72."

Step 1 — confidence in your number. You came up with 0.72 by reading three news articles and trusting your gut. Confidence is maybe 0.4. Apply the haircut: `risk_adjusted_edge_so_far = 0.10 × 0.4 = 0.04` (4 cents).

Step 2 — ambiguity discount. The contract is a Kalshi contract with a clear rule ("Powell holds the position on December 31, 2026"). Ambiguity is low, maybe 0.05. Apply: `risk_adjusted_edge_so_far = 0.04 × (1 − 0.05) = 0.038` (3.8 cents).

Step 3 — liquidity factor. The orderbook has $400 of depth on the YES side at 0.62, with the next level at 0.64. If you try to put on a position larger than $400, you eat 2 cents of slippage. For a $400 position, the liquidity factor is 1.0; for a $1000 position it is closer to 0.5. Let's say you size for $400. `risk_adjusted_edge_so_far = 0.038 × 1.0 = 0.038` (3.8 cents).

Step 4 — Kelly fraction. Half-Kelly on a 3.8-cent expected value at a 62-cent price is on the order of `0.038 / (0.62 × 0.38) × 0.5 ≈ 0.08` of bankroll. On a $10,000 bankroll, that is $800 — but the orderbook only takes $400 cleanly, so the binding constraint is liquidity, not Kelly.

Final position: $400 long YES at 0.62, with an honest expected return of about 3.8 cents per dollar — in the neighborhood of $15 of expected value on the position. The naive "10-cent edge" framing would have suggested a much larger position with a much larger expected payoff. The honest framing tells you the trade is small, modest, and worth doing only because the indicator stack lets you find dozens of similar small trades and the aggregate is what matters.

## Where the Gap Really Is Edge

I do not want to dismiss subjective conviction as a source of trade ideas. It is not zero — it is just smaller than beginners think, and it is more expensive to monetize than the headline number suggests.

The two cases where the gap genuinely is edge are: information asymmetry (you know something the market does not know yet, and you can act before the asymmetry closes) and slow news / illiquid markets (the news has hit but the price has not adjusted because there are not enough traders watching). Both of these cases are about *time*, not about strength of belief — the question is whether the market is going to discover what you know, not whether you happen to disagree with the current price.

The case where the gap is *not* edge is the case where you and the market are looking at the same information and you simply disagree about how to weight it. In efficient markets with deep books, you should default to the market's interpretation rather than your own, because the market's interpretation has been tested against thousands of other people's interpretations and yours has not. This is the wisdom-of-crowds principle, and it applies more often to PMs than beginners want to admit.

## The Practical Takeaway

The right mental model for "my conviction vs the market price" is not "my number minus their number equals my edge." It is closer to "my number is one input among thousands. The market price already incorporates the inputs from people with similar views to mine. My number only adds value if I have information or processing the market lacks. And once I have decided my number is a real edge, I have to haircut it for confidence, ambiguity, and liquidity before I size the trade."

The discipline this imposes is that you stop trading every contract you have a strong opinion on, and start trading the smaller set of contracts where you have a *defensible reason to believe the market is missing something*. That set is smaller than your gut tells you it is, and the trades within it are smaller than the naive sizing would suggest. Both of those facts feel uncomfortable. Both of them are also correct, and ignoring them is the most expensive habit in prediction-market trading.

`sf scan --by-iy desc` plus the rest of the [pm-indicator-stack](/concepts/pm-indicator-stack) is the antidote to the "I have a strong opinion" failure mode, because it surfaces contracts on the basis of mathematics rather than gut reaction. The gut reaction is the part that should be applied last, in stage 3 of [the-valuation-funnel](/concepts/the-valuation-funnel), to the small number of candidates that have already passed the indicator filter and the orderbook check.