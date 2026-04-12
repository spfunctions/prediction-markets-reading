# Resolution Ambiguity Score: Quantifying Rule-Risk Per Market

> Every prediction market has a measurable "ambiguity score" based on how many edge cases the rule mentions, the venue history of disputes on similar markets, and the source-of-truth specificity. Multiply your expected edge by (1 − ambiguity/10) to get the rule-adjusted edge.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 12 min

---
The price of a prediction-market contract reflects two probabilities at once: the probability that the underlying event happens, and the probability that *the market resolves the way the trader expects given that the event happens*. Most analysis treats these as the same number. They are not. The gap between them is the *resolution risk* — the chance that the contract pays the wrong way for reasons that have nothing to do with whether the underlying event occurred.

The previous concept page on [resolution-risk-premium](/concepts/resolution-risk-premium) introduces the idea: when the resolution rule is fuzzy, the market price IS the rule's expected interpretation, not the outcome's probability. This essay does the next step — quantify the ambiguity of a specific market on a 0 to 10 scale you can multiply against expected edge.

I call that number the *resolution ambiguity score* (RAS). It is heuristic, hand-tuned, and the most useful pre-trade check I run on any contract whose payout depends on a non-trivial source-of-truth.

## The Three Components

RAS has three additive components. Each contributes 0 to ~3 points to the final score. The maximum is roughly 10, the minimum is 0, and most contracts I look at land between 1 and 4.

**Component one — rule-text edge case count (0 to 3 points).** Read the resolution rule on the venue (the "Rules" tab on Kalshi, the "Resolution criteria" field on Polymarket). Count the number of explicit edge cases the rule mentions. These are sentences of the form "if X happens, then Y" or "in the event of Z, the market resolves to W". Each edge case the rule has to spell out is one place where someone designing the contract anticipated trouble.

Zero edge cases means the rule is "X happens, contract resolves YES." That is a clean rule, contributing 0 to the ambiguity score. One edge case ("if X is delayed past the close date, contract resolves NO") is normal and contributes about 0.5 points. Five edge cases with their own sub-clauses ("if X is partially delayed, but Y happens before the close date, and Z is reported by the official source, then the contract resolves YES, unless A is true, in which case...") is a contract that is *expecting* trouble, and contributes 2.5 to 3 points.

The reading is fast — under two minutes per contract — and the count is mostly objective. The component captures the *anticipation of dispute* by the rule-writer, on the principle that rules with more anticipated edge cases get triggered into edge cases more often.

**Component two — venue history of disputes on similar markets (0 to 4 points).** This is the most predictive component and also the most expensive to compute. For each candidate market, identify the recent venue history of *similar* markets — same category, same source-of-truth, same general structure. Then count the dispute rate. On Polymarket, "dispute" means a UMA challenge in the resolution window. On Kalshi, "dispute" is rarer because Kalshi is the resolver, but it shows up as rule clarifications, settlement delays, or community complaints in the resolution period.

A category with zero historical disputes contributes 0 points. A category with disputes in 5-10% of recent resolutions contributes 1 point. A category with disputes in 20-30% contributes 2 points. A category with chronic disputes (>30% on recent markets) contributes the full 3-4 points. Polymarket's "presidential election in Country X" category has historically been in the 10-15% dispute range; Polymarket's "what will the US president say at the next press conference" category has been at >40%, because the source-of-truth is a *transcript* and the disputes are over phrasing.

I track this with a manual spreadsheet of categories I have traded and the dispute rate in the last six months. Two minutes a week to maintain. The payoff is putting a real number on dispute risk for any new contract that falls into a known category.

**Component three — source-of-truth specificity (0 to 3 points).** Read the rule's source-of-truth clause. The rule will say "the market resolves based on [source]." The specificity of that source is the third component.

Maximum specificity is something like "the official close-of-business price as published by the BLS at https://bls.gov/news.release/empsit.htm". That is a public, automated, single-source-of-truth that anyone can verify in five seconds. Contributes 0 points.

Medium specificity is "the official announcement by the Federal Reserve at the FOMC press conference." That is a single-source but it requires *interpretation* (which sentence in the announcement counts? Does a verbal qualifier change the answer? What if the announcement is delayed?). Contributes 1 to 2 points depending on how unambiguous the announcement format is.

Low specificity is "a credible news source reports that X has happened." That is multi-source, interpretation-required, and the rule does not even specify which sources count. Contributes 2 to 3 points. Most of the highest-RAS contracts I have ever seen had a low-specificity source clause, because the dispute then becomes "did this source actually report this?" and that question itself is not resolvable by the rule.

Add the three components. The result is the resolution ambiguity score, on a 0-10 scale. In practice the scores cluster between 1 and 6 — anything above 6 is a contract I will not trade at all unless the RAS-adjusted edge is huge.

## Three Real-Shape Examples

Three contracts I have actually evaluated, walked through end to end.

**Example one — Polymarket UMA dispute pattern.** A Polymarket contract listed a few months ago: "Will Country X hold its presidential election by December 31, 2026?" The rule looked simple at first glance ("YES if the election is held by the date, NO otherwise"). The edge case count was actually 4: delays past the close date, runoffs, disputed results, post-vote rescheduling. Component one score: about 2.5.

Component two — venue history. The "Will country X hold an election" category has had at least three recent UMA disputes on similar contracts. Two of the disputes were over the runoff edge case (does a runoff count as the same election or a separate one?). The dispute rate in the category was about 20%, contributing 2 points.

Component three — source-of-truth. The rule said "credible reporting from international news sources." That is the lowest-specificity clause possible. Contributes the full 3 points.

Total RAS: 2.5 + 2 + 3 = 7.5. A contract I would not trade on conviction grounds at all. Even if I had a strong view on the deadline, the contract was not pricing that view — it was pricing my view *plus* the dispute risk, and the two were inseparable.

Ten weeks later the contract resolved with a UMA challenge. The challenge took five days to resolve, the resolution went against what most traders thought was the "obvious" outcome, and the conviction holders lost money on a technicality. That is what RAS is supposed to flag.

**Example two — Kalshi recession-definition pattern.** Last year there was a Kalshi contract for "Will the US enter a recession by year-end?" The rule said the contract would resolve based on whether the NBER (National Bureau of Economic Research) declared a recession by the close date. The edge case count was modest — 3 points worth, mostly around the timing of the NBER declaration relative to the close.

Component two — Kalshi historically has few disputes on macro contracts because Kalshi is the resolver and adheres rigidly to the rule text. There had been one prior incident where the NBER had not declared anything as the close approached, leading to community complaints. Dispute rate was effectively 0% but the *near-miss rate* was meaningful. I scored component two at 1 point.

Component three — the source-of-truth was the NBER, a single named authoritative source. But the *NBER definition of recession* is itself fuzzy (committee judgment, not a numerical formula). Specific about *who* resolves it; not specific about the source's own decision process. Component three: 1.5 points.

Total RAS: 3 + 1 + 1.5 = 5.5. Mid-range. A contract I would trade, but only with strong conviction on both the underlying *and* the NBER timing. What actually broke this contract was not a dispute — the NBER did not declare recession by the close date even though the economy was widely considered to be in one. The contract resolved NO. Traders who were "right" about the economy were wrong about the rule. RAS was 5.5 because the timing of the source was vague, and the timing was the binding constraint.

**Example three — a clean Kalshi weather contract.** Take "Will the high temperature at JFK on April 15 exceed 65°F?" The rule resolves based on the official NWS observation at the JFK reporting station. Edge case count is 1 — a clean numeric tie-breaker for "exactly 65°F". Component one: 0.5.

Component two — Kalshi has resolved hundreds of these with zero disputes. Category dispute rate: 0%. Component two: 0.

Component three — the source-of-truth is the NWS automated observation feed at a single named station, the most specific clause possible. Component three: 0.

Total RAS: 0.5. The cleanest possible contract from a rule-risk perspective. Whatever conviction you have on the temperature outcome translates almost perfectly into an edge calculation. If you have a good weather model, the trade is exactly what your model says — no haircut for resolution risk.

## The Trading Rule for RAS

The way I use RAS in practice is as a multiplier on expected edge. If your raw expected edge is 10 cents on a contract with RAS = 2, your rule-adjusted edge is 10 × (1 − 2/10) = 8 cents. If RAS is 6, the rule-adjusted edge is 10 × (1 − 6/10) = 4 cents. If RAS is 9, the rule-adjusted edge is 1 cent — too small to trade against fees.

The formula is heuristic. I am not claiming the linear haircut is mathematically optimal. What it is is *honest* — it forces me to pay attention to rule risk before sizing into a contract, and it forces me to size *down* on contracts where the rule risk is non-trivial. The alternative — ignoring rule risk because the *price* is what I have data on — is how people lose money in the chronic-dispute categories.

I also use RAS as a hard threshold. I do not trade contracts with RAS > 7 on conviction grounds at all, regardless of the edge calculation. If I want that kind of exposure, I find a cleaner-rule contract on another venue, or go without.

## Where RAS Breaks

A few honest caveats.

**The score is heuristic, not measured.** I have not built a regression that fits RAS to historical outcomes. The component weights are hand-tuned. A more rigorous approach would calibrate the weights against actual dispute rates and rule-mismatch outcomes — and someone should do that. For now, RAS is a checklist that I treat as a *pre-trade discipline*, not a quantitative model.

**The venue history component is venue-specific.** A category that has been clean on Kalshi may have chronic disputes on Polymarket, and vice versa. The component-two score has to be computed per-venue, because the resolution mechanisms are completely different (Kalshi is centralized; Polymarket uses UMA). Cross-venue arbs have to compute RAS on both legs and take the *higher* of the two.

**Brand-new categories have no history.** When a venue lists a category that has never existed before — say, a new format of NBA contract — there is no dispute history to score component two against. I default to scoring those at 1 point of dispute risk until enough history accumulates. This is conservative and probably wrong some of the time, but it has saved me from several rough listings.

**The rule can change after listing.** This is the worst failure mode. A few times a year, a venue updates the rule on a listed contract — usually because a clarification is needed, sometimes because the original rule was found to be incoherent. When the rule changes mid-contract, your RAS computation is stale and your sizing is wrong. The defense is to re-read the rule before any large position add, and to size smaller on contracts where rule changes are even theoretically possible.

## How This Connects to the Rest of the Stack

RAS is the sister concept to [resolution-risk-premium](/concepts/resolution-risk-premium), which describes the *price-level* effect of resolution risk. RAS is the *per-market* score that drives that effect. They are the same phenomenon at two different scales: RAS is the score on one contract, and resolution risk premium is the price compression across all contracts in a category that the market has correctly identified as ambiguous.

For the broader case that endogenous market data alone is not enough — that you have to look at the rule and the source-of-truth and the venue history — see [endogenous-vs-reality-vs-opinion-data](/concepts/endogenous-vs-reality-vs-opinion-data). The rule text is *reality data* (it is what the contract will actually resolve against), the dispute history is *opinion data* (it is what other humans think about how disputes have been handled), and the price is *endogenous data*. RAS is a way of forcing all three into a single score that you can use at trade time.

For the implementation of pulling the rule text and computing the heuristic, the right place to start is the existing [reading-prediction-market-orderbooks](/technicals/reading-prediction-market-orderbooks) walk-through plus the rule-text scraping endpoints (Kalshi has a public market-detail endpoint that includes the full rule; Polymarket has the resolution_criteria field on the market metadata). Once you have the rule text in a pipeline, the component-one and component-three scores can be partly automated; component two requires the manual category-history spreadsheet.

`sf scan --max-ras 3` is the flag I use to filter for rule-clean contracts at scan time. The default RAS threshold of 3 excludes contracts where the rule risk is high enough to materially distort the edge calculation, and it leaves about 70% of the universe still in the candidate pool. The other 30% is the part where you need to do the per-contract reading before sizing.

The big takeaway is that *the rule is the contract*. The rule is what gets enforced at resolution. Whatever the price says about your edge, the rule is what determines whether the edge is real. RAS is the discipline of taking the rule seriously at the same level of attention you give the price.