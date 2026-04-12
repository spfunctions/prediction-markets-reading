# Resolution Risk Premium: Pricing the Rule, Not the Outcome

> When the resolution rule is fuzzy, the displayed market price is not the probability of the outcome — it is the market's best guess at how the rule will be interpreted. Three famous cases show the gap, and the discount you should apply.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
Every binary prediction-market contract has two layers of uncertainty stacked on top of each other. The first layer is the obvious one: will the event happen. The second layer is less obvious and often more important: *will the resolution authority decide that the event happened*. The two are not the same, and the gap between them is the resolution risk premium.

This is the concept that distinguishes a sophisticated PM trader from a naive one. Naive traders price the outcome. Sophisticated traders price the rule. On contracts with clean rules the two collapse to the same number, and the distinction does not matter. On contracts with fuzzy rules — which is more contracts than people think — the distinction is the entire reason the market is mispriced.

## Why Rules Are Fuzzy

A prediction-market rule is a piece of natural-language text that tries to specify, in advance, what set of real-world outcomes count as YES. The text is written by humans before the future is known, and humans cannot anticipate every edge case. Three things go wrong:

1. **The rule depends on a source that becomes unreliable.** "Resolves YES if the BLS publishes a non-farm payrolls number above 200,000." What if the BLS revises the number? What if the BLS releases a flash estimate and a final estimate that disagree?

2. **The rule depends on a definition that is contested.** "Resolves YES if the U.S. enters a recession by Q3 2026." Defined how — NBER announcement, two consecutive quarters of negative GDP, the Sahm rule, something else?

3. **The rule depends on a scenario the writer did not imagine.** "Resolves YES if Brazil holds a presidential election by November 2026." What if the election is held but a court invalidates the result a week later?

When any of these go wrong, the resolution authority (a UMA panel on Polymarket, the Kalshi compliance team on Kalshi) has to decide. The decision is a coin flip from the outside, and the market prices it accordingly.

## Three Famous Cases

**Polymarket Brazil 2022 election dispute.** Lula won the runoff against Bolsonaro on October 30, 2022. The contract "Will Lula win the 2022 Brazilian presidential election" should have resolved YES at 100¢. It did, eventually, but in the days after the election there was a small but real probability that Bolsonaro would refuse to concede and the courts would intervene. The market traded at 96-97¢ for several days *after* Lula was confirmed as the winner, because the resolution risk premium reflected the small probability that the dispute would lead UMA to delay or contest resolution. The 3-4¢ gap was not "pricing the outcome" — the outcome was settled. It was pricing the rule.

**Kalshi recession definition.** A recurring class of Kalshi contracts asks "will the U.S. be in a recession by date X." Kalshi specifies the resolution as "based on NBER announcement," but NBER announces recessions with a lag of 6-12 months. So the contract is *technically* about NBER's eventual decision, not about the underlying economic conditions. This means that during the 2022 GDP slowdown — when GDP printed two consecutive negative quarters and most journalists were calling it a recession — the Kalshi contract still traded at around 30¢, because the NBER had not announced and the market was pricing the announcement, not the GDP. The naive trader looked at the GDP prints and bought aggressively. The sophisticated trader read the rule and waited for NBER signals.

**Polymarket "Russia controls Bakhmut by date X."** A contract about whether Russia would control the city of Bakhmut by a specific date. The fighting was protracted, both sides made conflicting claims, and at the deadline there was no clean fact that everyone agreed on. The contract traded in the 40-50¢ range for weeks because the resolution outcome depended entirely on which sources UMA decided to trust. The "outcome" — who actually controlled Bakhmut — was never going to be a single number. The price was the market's estimate of UMA's eventual judgment call.

In all three cases, the displayed price was the *rule's expected interpretation*, not the *event's probability*. They are different things, and treating them as the same is the most common error new sophisticated traders make.

## The Discount Math

Once you accept that the rule has uncertainty, you can write down a simple model. Let:

```
p_outcome = your true probability the underlying event occurs
p_rule = the probability the resolution authority decides "YES" given the event occurred
displayed_price ≈ p_outcome × p_rule
```

The displayed price is the *product* of the outcome probability and the rule-interpretation probability. If you have an edge on p_outcome but the rule is uncertain, your edge is multiplied by p_rule before it shows up in P&L.

Concrete numbers: I think the underlying event has a 75% probability. The market is at 60¢. Naively, I have a 15¢ edge. But if p_rule is only 0.85 (because the resolution rule is fuzzy enough that there is a 15% chance the authority decides against the literal reading), my expected payout is 0.75 × 0.85 = 0.638 × $1.00 = $63.75 per dollar. My realistic edge is 3.75¢, not 15¢. The fuzzy-rule discount has eaten 75% of my expected edge.

This is why sophisticated PM traders apply a *resolution discount* to every position before they size it. The discount comes from a per-contract estimate of p_rule, which depends on:

- Source specificity (a rule that names a specific URL or document is much more reliable than one that says "based on news reports")
- Historical UMA dispute rate for similar contracts (the [resolution-ambiguity-score](/concepts/resolution-ambiguity-score) Batch L concept will formalize this)
- The political sensitivity of the outcome (politically charged contracts get more disputes)
- The clarity of the deadline (a clean deadline is reliable; "by year-end 2026" is not)
- The number of edge cases the rule explicitly mentions (more edge cases = the rule writer was thinking carefully = more reliable)

I keep a rough scorecard for each contract I look at, mostly in my head, and I multiply my edge by a discount factor between 0.7 and 1.0 depending on where the contract scores. Anything below 0.7 I do not trade at all unless the spread is wide enough to absorb the risk.

## How the Premium Shows Up in the Price

The resolution risk premium is observable as the *gap* between (a) what the market price implies about the outcome, and (b) what your independent analysis says the outcome probability is. If the market is consistently 5-10¢ below your outcome probability on contracts where you have no other reason to think you are right, that gap is usually the resolution premium working in the price.

You can verify this by comparing the same outcome on Kalshi vs Polymarket. Kalshi has cleaner resolution rules (a U.S.-regulated exchange with a compliance team and explicit rule documents). Polymarket has fuzzier rules (UMA disputes, source-of-truth ambiguity, political sensitivity). The same outcome often trades a few cents apart, and the cross-venue spread is mostly the resolution risk premium difference. See the upcoming [from-adr-arbitrage-to-cross-venue-pms](/concepts/from-adr-arbitrage-to-cross-venue-pms) page for the cross-venue spread framing in detail.

The premium is usually small for routine contracts (Fed decisions, labor market reports, scheduled elections in stable democracies) and large for contested ones (geopolitical events, contested elections, novel outcomes the platform has never resolved before). The variance in the premium is the variance in *how much you should discount your edge*.

## A Worked Example

KXRECESSION-26Q3 — "Will the U.S. be in recession by Q3 2026 per NBER" YES contract. Currently trading at 22¢ in March 2026. My analysis of GDP, labor market, and Sahm-rule indicators says the underlying probability of a recession occurring in the relevant window is roughly 35%. Naive edge: 13¢.

But the rule says "per NBER," and NBER announces recessions with a 6-12 month lag. By Q3 2026, NBER will not have ruled on a recession that started in mid-2026 even if one occurred. So the rule effectively asks "will NBER announce a recession by Q3 2026," and that is a much harder bar than "will the economy be in recession." I estimate p_rule at maybe 0.5 — half the time, even if a recession occurs in the relevant window, NBER will not have announced it by the deadline.

Adjusted edge: 0.35 × 0.5 = 0.175 vs market price 0.22. Now the trade looks slightly *negative*. My naive analysis said this was a 13¢ edge to the upside; my rule-aware analysis says it is a 4.5¢ edge to the *downside*. The right move is either to short the contract (buy NO) or to skip it entirely. The naive long that looked obvious is the wrong trade.

This is the kind of mistake that does not show up in your P&L until contract resolution, at which point the contract resolves NO (because NBER did not announce in time) and you discover that your "edge" was a misread of the resolution rule. The fix is to do the rule-aware analysis *before* sizing the position, not after.

The CLI shortcut for this on the SimpleFunctions side is `sf scan --by-iy desc --rule-discount auto`, which applies a heuristic discount factor to every IY computation based on the rule text and the venue. The auto-discount is rough but it is much better than 1.0 (which is what you get if you do not apply any discount at all).

## Where the Frame Breaks

Three failure modes worth naming.

**You over-discount.** Sometimes the rule is genuinely clean and you are seeing fuzziness that does not exist. A scheduled FOMC contract with a specific meeting date and a specific decision criterion does not need a 15% rule discount. The discount should be calibrated to the actual fuzziness, not to a generic anxiety. The fix: keep a log of contracts you discounted heavily that resolved cleanly anyway, and tighten your discount factor over time on similar rules.

**The rule changes mid-contract.** Both Kalshi and Polymarket have, on rare occasions, updated the resolution language of a contract after it was already trading. This is supposed to be rare but it is not zero. The fix: check the rule document on the venue page for any contract you hold older than a month, and refresh your discount estimate if anything has been edited.

**The resolution authority itself becomes the news.** If UMA has a string of disputed resolutions and the prediction-market community starts losing trust in the dispute mechanism, the resolution risk premium widens for *every* contract on Polymarket simultaneously. This is a form of platform risk that is correlated across the whole book and cannot be diversified away by holding many different contracts. The defense is to cap your total Polymarket exposure as a fraction of your portfolio, treating UMA risk as a single concentrated factor.

A fourth issue: the resolution risk premium is not stable over time. A contract that had a 0.85 p_rule when you bought it can drop to 0.6 if a dispute breaks out on a sister contract or if the venue announces ambiguous policy. Your discount factor has to be re-evaluated when conditions change, and sometimes the right move is to close a position not because your outcome probability changed but because your *rule probability* changed.

## The Habit Worth Building

Read the resolution rule before you read the price. Every time. Even on contracts that look obvious. The first thing you should know about a binary prediction-market contract is *what specifically counts as YES*, and the price comes second.

The discipline is annoying — most rules are routine and the answer is "it counts as YES if the event happens, duh." But the routine cases are not the ones you lose money on. You lose money on the 5-10% of contracts where the rule has a wrinkle you did not catch, and the only way to avoid those is to read every rule like it might be the wrinkled one.

I size makers wider on contracts where the rule has any ambiguity I cannot resolve in 60 seconds of reading. I avoid takers entirely on contracts where the rule depends on a source that has had any historical dispute. The rule is the trade. The outcome is the consequence of the trade. Treat them in that order.

For the related question of how to score the ambiguity systematically, see the upcoming [resolution-ambiguity-score](/concepts/resolution-ambiguity-score) Batch L page. For the indicator that measures *whether* the resolution event is approaching, see [tau-days](/concepts/tau-days). For the broader category of things that go wrong in the final hours of a contract, see [pin-risk-binary-settlements](/concepts/pin-risk-binary-settlements).