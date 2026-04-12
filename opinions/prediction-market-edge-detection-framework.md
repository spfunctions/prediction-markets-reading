# Prediction market edge detection: a practical framework for finding mispriced contracts

> Most prediction market traders have opinions but no framework for measuring whether those opinions are worth trading — here is a systematic approach to finding and sizing edge.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 11 min

---
I have been trading on Kalshi with real money for over a year. In that time I have learned one thing that matters more than anything else: having an opinion is free, but having a *measurable edge* is the only thing that makes money.

Most prediction market traders skip the measurement step. They see a contract at 35 cents, feel like the probability should be higher, and buy. Maybe they are right. Maybe they just anchored on a scary headline. Without a framework, they cannot tell the difference — and neither can their P&L over any meaningful sample.

This piece lays out the framework I actually use. Real tickers, real numbers, real spreads.

## Defining Edge Precisely

Edge is not "I think this contract is wrong." Edge is a number:

```
edge = thesis_implied_probability - market_price
```

That is it. If my causal model says recession probability is 55% and KXRECSSNBER-26 is trading at 35 cents, my edge is +20 cents. If my model says CPI will come in hot at 72% and KXCPI is trading at 70 cents, my edge is +2 cents.

The first edge is interesting. The second is noise. But you cannot distinguish them without writing both numbers down.

The thesis-implied probability comes from a structured model — a causal tree that decomposes the outcome into upstream drivers, assigns probabilities to each, and propagates them forward. You do not have to use software for this. A spreadsheet works. A napkin works. But you need *something* that forces you to decompose your belief into falsifiable components.

When I say "recession probability is 55%," that is shorthand for: I think the Fed stays restrictive (70%), labor market softens meaningfully by Q3 (60%), consumer spending contracts (45% conditional on the first two), and the NBER calls it within the calendar year (85% conditional on actual contraction). Multiply those conditional probabilities through the tree and you get a number. That number is your thesis-implied probability. The gap between that number and the market price is your edge.

## Four Types of Edge

Not all edge is created equal. I have found it useful to classify mispricings into four categories, because each one has different risk characteristics and different decay profiles.

### 1. Consensus Gap

The market price reflects the consensus view, but the consensus is wrong about a factual input. This is the cleanest form of edge.

Example: In early 2026, KXWTIMAX-26DEC31-T135 ("Will WTI oil hit $135 at any point in 2026?") was trading at 8 cents. The consensus view was that Hormuz stays open, OPEC maintains discipline, and global demand stays tepid. My model had Hormuz closure probability at 35% (based on escalation dynamics that the market was underweighting), and conditional on closure, WTI above $135 at ~80%. That alone implied a fair value around 28 cents before considering other paths to $135. The edge was +20 cents.

Consensus gaps are the highest-conviction edges because they are based on the market being wrong about something specific and verifiable. They also tend to persist because the consensus only updates when the evidence becomes undeniable.

### 2. Attention Gap

The market is not wrong — it is just not paying attention. Thin markets with low volume often drift away from fair value simply because nobody is actively quoting them.

I see this constantly on Kalshi. A contract listed under an obscure category gets 200 shares of volume per day. The last trade was three days ago. The bid-ask spread is 20 cents wide. The "price" is wherever the last noise trader happened to fill.

Attention gaps are easy to find and easy to exploit, but they come with a catch: if nobody is paying attention, nobody will trade against you either. You can be right and still not get filled.

### 3. Timing Gap

The market price reflects the correct long-run probability but fails to account for when information will arrive. This is common in contracts with known catalyst dates.

Example: KXFEDDECISION trades around each FOMC meeting. Two weeks before the meeting, the contract might reflect a 25% chance of a cut. But if you track the Fed funds futures curve, the implied probability is 40%. The prediction market is slow to update because its participant base does not monitor fixed income markets in real time. The 15-cent gap closes over the next week as the meeting approaches and more informed traders enter.

Timing gaps are short-lived but repeatable. They show up on every data release, every scheduled event, every known catalyst.

### 4. Risk Premium

The market price is "correct" in expected value terms but embeds a premium for the risk of being wrong. This is not technically a mispricing — it is compensation for bearing risk.

Tail-risk contracts often trade below fair value because most traders do not want to tie up capital in a low-probability bet that might take months to resolve. A contract that should trade at 15 cents based on base rates might trade at 8 cents because the opportunity cost of holding it is high and the resolution timeline is uncertain.

Risk premiums are real edge, but they require patience and capital. You are getting paid to wait.

## Executable Edge: Adjusting for the Spread

Raw edge is not the same as executable edge. If my model says fair value is 55 cents and the market mid is 35 cents, my raw edge is 20 cents. But I cannot trade at the mid. I trade at the ask (if I am buying) or the bid (if I am selling).

```
executable_edge = raw_edge - (spread / 2)
```

If the ask is 40 cents and the bid is 30 cents, the spread is 10 cents. My executable edge on a buy is:

```
executable_edge = (55 - 35) - (10 / 2) = 20 - 5 = 15 cents
```

That 15 cents is what I actually capture if I lift the ask at 40 and the contract resolves at my thesis-implied value. In practice, I use a slightly more conservative formula that accounts for the probability I am wrong:

```
expected_edge = executable_edge × confidence - (1 - confidence) × cost_if_wrong
```

Where confidence is my meta-probability that the thesis model itself is correct. If I am 70% confident in my model and the cost of being wrong (the ask price I paid) is 40 cents:

```
expected_edge = 0.15 × 0.70 - 0.30 × 0.40 = 0.105 - 0.12 = -0.015
```

Wait — negative expected edge? Yes. If I am only 70% confident in a model that gives me 15 cents of executable edge, the trade is actually slightly negative EV after accounting for model uncertainty. I need either more edge or more confidence to make this work.

This is why writing the numbers down matters. Without this framework, I would have looked at a 20-cent raw edge and thought "great trade." The framework told me: not yet.

## Walking Through a Real Edge Table

Here is an edge table from my actual trading notes, using `sf edges` output from SimpleFunctions. The tickers are real Kalshi contracts:

```
$ sf edges

Ticker                      Market  Thesis  Raw     Spread  Exec    Liq     Signal
                            Price   Fair    Edge           Edge    Score
─────────────────────────────────────────────────────────────────────────────────────
KXRECSSNBER-26              0.35    0.55    +0.20   0.10    +0.15   7/10    consensus_gap
KXWTIMAX-26DEC31-T135       0.08    0.28    +0.20   0.15    +0.13   4/10    consensus_gap
KXFEDDECISION-26MAR        0.25    0.40    +0.15   0.04    +0.13   9/10    timing_gap
KXCPI-26MAR-T0.4            0.52    0.58    +0.06   0.06    +0.03   8/10    timing_gap
KXGASMAX-26DEC31-T4.5       0.14    0.45    +0.31   0.22    +0.20   2/10    consensus_gap
KXUNRATE-26MAR-T4.2         0.40    0.44    +0.04   0.08    +0.00   7/10    noise
```

Let me walk through what this table is actually telling me.

**KXRECSSNBER-26** — Recession in 2026 per NBER dating. Market says 35%. My model says 55%. Executable edge after spread is 15 cents. Liquidity is decent (7/10). This is a real trade. The edge type is consensus gap — the market is underweighting the lagged effects of restrictive monetary policy. I am long this contract.

**KXWTIMAX-26DEC31-T135** — Will WTI hit $135 at any point in 2026? Market says 8%. My model says 28%. Raw edge is huge at 20 cents. But the spread is 15 cents wide, chopping executable edge down to 13 cents. Liquidity score is 4/10 — thin book, hard to get filled at size. I have a small position here. The edge is real but the execution cost is painful.

**KXFEDDECISION-26MAR** — Will the Fed cut at the March 2026 meeting? Market says 25%. Fed funds futures imply 40%. The spread is tight at 4 cents (this is a high-attention contract near its catalyst date). Executable edge is 13 cents. Liquidity is 9/10. This is the best risk-adjusted trade on the board. Timing gap: the prediction market is lagging the rates market. I am long and sized up.

**KXCPI-26MAR-T0.4** — Will month-over-month CPI exceed 0.4% in March? Market says 52%. My model (based on components tracking: shelter, used cars, energy) says 58%. Executable edge after spread is only 3 cents. This is technically positive but not worth the transaction costs and capital tie-up. I pass.

**KXGASMAX-26DEC31-T4.5** — Will natural gas hit $4.50 in 2026? Market says 14%. My model says 45%. That is a 31-cent raw edge — the largest on the table. But the spread is 22 cents. After spread adjustment, executable edge is 20 cents, which is still attractive. The problem is the 2/10 liquidity score. I can get maybe 200 shares filled before I move the market against myself. At $200 of exposure, even a 20-cent edge is $40 of expected profit. Not worth the monitoring cost.

**KXUNRATE-26MAR-T4.2** — Will unemployment exceed 4.2% in March? Market says 40%. My model says 44%. Raw edge is 4 cents. The spread is 8 cents. Executable edge rounds to zero. This is not a trade. My model barely disagrees with the market, and the spread eats whatever edge exists.

## When Edge Is Actionable vs When It Is a Trap

The edge table makes the decision framework mechanical. A trade needs to pass three filters:

**Filter 1: Executable edge > 10 cents.** Below this threshold, model uncertainty and transaction costs eat the profit. I have tried trading 5-cent edges and the win rate does not compensate for the times the model is simply wrong.

**Filter 2: Liquidity score >= 5/10.** Below this, I cannot get meaningful size. A 30-cent edge on a contract where I can only fill 100 shares is $30 of expected profit. Not worth the cognitive overhead of tracking the position.

**Filter 3: Time to resolution > 7 days.** Contracts approaching expiry have a different dynamic. The edge might be real, but event risk is concentrated. A contract that is "mispriced" at 35 cents with 2 days to go might be correctly priced given the uncertainty of the resolution mechanism. NBER recession dating, for instance, happens with a lag of 6-12 months. A contract that "expires" in December might not resolve until July of the following year.

The traps I have fallen into:

**Thin liquidity masquerading as edge.** A contract at 8 cents with a fair value of 25 cents looks like 17 cents of edge. But if the bid is 3 cents and the ask is 13 cents, I am buying at 13 and my executable edge is 12 cents — still looks good. Then I try to sell a week later and the bid has dropped to 1 cent. Nobody is on the other side. I hold to expiry and the contract resolves NO. My 17-cent "edge" was a 13-cent loss.

**Stale thesis masquerading as conviction.** I built a model in January that said recession probability was 55%. By March, three months of strong payrolls data had come in. I did not update the model. The market moved from 35 to 28 cents. I bought more, thinking the edge had *increased*. It had not — my model was stale. The market was right to reprice. I was anchored to an outdated thesis.

**Calendar spread confusion.** KXRECSSNBER-26 and KXRECSSNBER-26Q2 are different contracts. The annual contract includes all four quarters. If I am long the annual at 55% thesis probability, that does not mean each quarterly contract is mispriced by the same amount. Most of my recession probability mass is in Q3-Q4. The Q2 contract at 15 cents might be correctly priced even though the annual contract at 35 cents is cheap.

## The Process

Every Sunday I run `sf edges` and rebuild the table. I update the causal tree nodes with any new data from the past week. I check whether my thesis-implied probabilities have changed and by how much. I compare the new edge table to the previous week.

If a new edge appears (a contract moved but my model did not), I investigate why. Sometimes the market got new information I missed and I need to update. Sometimes it is noise — a large order pushed the price and it will revert.

If an existing edge disappeared (my model and the market converged), I check whether to take profit. If the convergence happened because the market came to me, that is P&L realized. If it happened because I updated my model toward the market, that is a thesis revision — different thing entirely.

The framework is boring. It is a spreadsheet and a weekly review. But it is the difference between trading on vibes and trading on measured disagreement with the market.

## What This Framework Does Not Do

It does not predict the future. My model is wrong regularly. The edge table told me KXFEDDECISION was a great trade last January and the Fed held steady — I lost the premium. The point is not to be right every time. The point is to ensure that when I am right, I am sized appropriately, and when I am wrong, I know why.

It does not remove the need for judgment. The causal tree requires me to assign probabilities to upstream nodes, and those assignments are subjective. What the framework does is make the subjectivity *explicit* and *decomposed*. Instead of "I feel like recession is likely," I have to say "I think the Fed stays restrictive at 70% and labor markets soften at 60%." Each of those numbers is individually debatable and updatable.

It does not work on every contract. Some prediction markets are purely sentiment-driven (politics, celebrity events) where causal modeling does not apply. I stick to economic and financial contracts where the upstream drivers are observable and quantifiable.

## Getting Started

If you want to try this yourself:

1. Pick three contracts you have an opinion on.
2. For each, write down your thesis-implied probability as a single number.
3. Check the current market price and the bid-ask spread.
4. Calculate executable edge: (thesis - market) - (spread / 2).
5. If executable edge is greater than 10 cents and liquidity is reasonable, consider the trade.
6. If it is less than 10 cents, you do not have a trade — you have an opinion.

The gap between opinion and trade is measurement. The framework is the measurement.

I built [SimpleFunctions](https://simplefunctions.dev) to automate steps 2-4 and surface the edge table programmatically. But the framework works with a pen and paper. The tool is optional. The discipline is not.