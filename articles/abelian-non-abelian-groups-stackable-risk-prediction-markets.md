# Abelian and Non-Abelian Groups: Stackable Risk vs Non-Stackable Risk

> When the order of events changes the outcome, every model that assumes otherwise is lying to you.

**Category:** insights | **Author:** SimpleFunctions Research | **Reading time:** 10 min | **Published:** 2026-03-26

---
# Abelian and Non-Abelian Groups: Stackable Risk vs Non-Stackable Risk

*When the order of events changes the outcome, every model that assumes otherwise is lying to you.*

## The Algebra You Forgot Matters

A group is one of the simplest structures in abstract algebra: a set equipped with a binary operation, an identity element, and inverses. You learned this in undergraduate algebra and probably filed it away as pure abstraction. But groups describe symmetry, and symmetry — or its absence — governs how risk factors combine.

An **abelian group** is one where the group operation commutes. For any two elements *a* and *b* in the group, *a* \cdot *b* = *b* \cdot *a*. The integers under addition are the canonical example: 3 + 5 = 5 + 3. It does not matter which operand comes first. The result is identical. You can stack the operations in any order and arrive at the same place.

A **non-abelian group** is one where this commutativity fails. The symmetric group S3 — permutations of three objects — is the smallest finite non-abelian group. If you compose two permutations, the order in which you apply them generally changes the result. Rotate then flip is not the same as flip then rotate. The matrix group GL(n, R) of invertible n-by-n real matrices is non-abelian for n >= 2: matrix multiplication does not commute. The path you take through the group determines where you end up.

This distinction — stackable versus non-stackable, order-independent versus order-dependent — is not a curiosity of pure mathematics. It is the dividing line between risk models that work and risk models that quietly break down precisely when you need them most.

## Why Traditional Finance Lives in an Abelian World

Modern portfolio theory begins with a simple premise: the return of a portfolio is the weighted sum of the returns of its constituent assets. If asset A contributes a return r_A and asset B contributes r_B, the portfolio return is w_A \cdot r_A + w_B \cdot r_B. Addition is commutative. It does not matter whether you "add" the risk contribution of asset A first or asset B first. The result is the same.

This is not an accident. It is an architectural choice baked into the mathematics at the foundation. Markowitz optimization, the Capital Asset Pricing Model, and every linear factor model that descends from them operate in vector spaces — which are, by definition, abelian groups under addition. The risk factors live in R^n. You project returns onto factor loadings. You compute covariance matrices. Everything superposes linearly.

Value at Risk (VaR) compounds this assumption. VaR asks: "What is the maximum loss at a given confidence level over a given horizon?" The standard parametric approach assumes returns are drawn from a multivariate normal distribution. The covariance matrix — a symmetric, order-independent object — encodes all pairwise relationships between risk factors. There is no slot in the covariance matrix for "A happens before B." The matrix element sigma_{AB} is the same as sigma_{BA}. Symmetry is assumed. Commutativity is hard-coded.

Even more sophisticated approaches — copulas, GARCH models, regime-switching frameworks — tend to preserve this abelian structure at the level of risk factor combination. They may allow for fat tails, time-varying volatility, and non-linear dependence. But the fundamental question they answer is still: "Given that these risk factors are all present, what is the distribution of outcomes?" Not: "Given that these risk factors arrived in this specific sequence, what is the distribution of outcomes?"

The entire quantitative finance apparatus is, in the language of algebra, an abelian group acting on a portfolio space. And for many applications — diversified equity portfolios, interest rate hedging, routine market-making — this works tolerably well. The risk factors (earnings surprises, rate changes, sector rotations) are approximately independent, approximately simultaneously present, and approximately order-insensitive. The abelian assumption is not exactly correct, but it is close enough.

Until it is not.

## The Non-Abelian Reality of Geopolitical Risk

Consider two events: the imposition of comprehensive sanctions on Iran, and the outbreak of a military conflict involving Iran. Both affect oil prices, regional stability, alliance structures, and refugee flows. A linear risk model would assign each event a "shock" and sum them. Sanctions contribute +$20/barrel to oil. Conflict contributes +$40/barrel. Together: +$60/barrel. The order is irrelevant.

But the order is not irrelevant.

**Sanctions first, then war.** Sanctions are imposed. Over months, Iran's oil exports decline. Alternative supply chains develop. Countries find substitute suppliers or draw down strategic reserves. Diplomatic channels remain open; there is a theory of the case that economic pressure will produce a negotiated outcome. When war eventually erupts, it erupts in a context where markets have partially adjusted, where supply chains have partially rerouted, and where the narrative frame is "diplomacy failed." The oil price shock of war is real but partly cushioned by prior adaptation.

**War first, then sanctions.** A military conflict erupts without the preparatory phase of economic isolation. Oil markets spike immediately and violently. There are no substitute supply chains because there was no incentive to build them in advance. Sanctions imposed after hostilities begin operate in a fundamentally different environment: they are punitive rather than coercive, they target a wartime economy rather than a peacetime one, and they interact with a completely different set of alliance commitments. Allied nations that might have supported pre-conflict sanctions may fracture over wartime ones. The refugee crisis unfolds with no prior institutional preparation.

These are not the same state. The terminal oil price may differ. The alliance structure will differ. The humanitarian consequences will differ. The probability distribution over outcomes — the entire stochastic landscape — depends on the path. Denote the sanctions operation as S and the war operation as W. We are asserting that S \cdot W != W \cdot S. The composition of geopolitical events does not commute. The risk factors form a non-abelian structure.

This is not a theoretical point. The difference between sanctions-then-war and war-then-sanctions in the case of Iraq in 1990-1991 versus Iraq in 2003 is a case study in non-commutativity. The 1990 sequence (invasion, then sanctions, then war) produced a fundamentally different geopolitical configuration than the 2003 sequence (sanctions regime over a decade, then war). The "same" events, in different orders, yielded different worlds.

The mathematical structure here is closer to a non-abelian group — or more precisely, a non-abelian monoid, since not all geopolitical events are "invertible" — than to the vector spaces of portfolio theory. The state of the world is not a point in R^n that you reach by adding up independent shocks. It is a point in a configuration space that you reach by composing operations in sequence, where the sequence matters.

## How Prediction Markets Hide Non-Commutativity

A prediction market contract is a binary instrument. It pays $1 if an event occurs and $0 if it does not. "Will oil exceed $150/barrel by December 2026?" You buy at $0.25, it resolves yes, you collect $1. Clean, simple, elegant.

And structurally incapable of encoding path dependence.

The contract pays the same dollar whether oil reached $150 via a gradual supply squeeze or via a sudden military disruption. The contract does not ask how the world arrived at the terminal state. It asks only whether the terminal state was reached. This is by design: binary contracts derive their liquidity and simplicity from this reduction. But the reduction is lossy. It projects a non-abelian path space onto a single boolean.

The price of the contract — $0.25 in our example — is supposed to encode the market's aggregated probability that oil reaches $150. But which probability? The probability conditional on what path? If sanctions precede conflict, the market might price the contract at $0.30. If conflict precedes sanctions, the market might price it at $0.55. But the contract itself is the same contract. The market price at any given moment is a single number — a scalar — that averages over all the paths the market collectively imagines.

This averaging is precisely the abelianization of a non-abelian structure. In group theory, every group G has an abelianization G/[G,G], obtained by quotienting out the commutator subgroup — the subgroup generated by all elements of the form a \cdot b \cdot a^{-1} \cdot b^{-1}. The abelianization collapses all the order-dependent information into an order-independent residue. A prediction market price is the abelianization of the market's collective path-dependent beliefs. It is the shadow of a higher-dimensional object projected onto a line.

This matters when the commutator subgroup is large — when the non-abelian structure carries significant information. If the geopolitical situation is one where path order genuinely changes terminal outcomes, then the scalar price of a binary contract is systematically discarding relevant information. Two traders with identical beliefs about the terminal probability but radically different beliefs about the likely path will express the same price. Their disagreement — which may be the most informative part of the market — is invisible.

## Restoring Non-Abelian Structure Through Milestone Signal Injection

This is the problem that SimpleFunctions' milestone signal architecture is designed to address. Rather than collapsing a thesis to a terminal probability, the system tracks intermediate milestones — causal nodes along the paths that lead to a terminal state. The ordering of these milestones, their conditional dependencies, and the structure of transitions between them are explicitly modeled.

Consider a thesis on the question "Will the US impose a full semiconductor export ban on Country X by Q4 2026?" The terminal event is binary: yes or no. But the paths to "yes" are structurally different and lead through different milestone sequences:

**Path A:** Diplomatic deterioration -> Congressional pressure -> Executive order -> Implementation.

**Path B:** Military provocation -> Emergency declaration -> Executive order -> Implementation.

These paths share a terminal state (the ban is imposed) but their milestone orderings differ, and the probabilities of subsequent milestones are conditional on which prior milestones have been reached, and in what order. The probability of Congressional pressure given diplomatic deterioration is not the same as the probability of Congressional pressure given a military provocation — in the latter case, Congress may rally in support or fracture in opposition, depending on the specifics.

The milestone signal injection framework tracks which milestones have been reached, which have been skipped, and which have occurred out of the "expected" order. It is, in algebraic terms, an attempt to preserve the non-abelian structure of the path space rather than projecting it onto a scalar.

When a new signal arrives — a diplomatic cable, a military deployment, an economic indicator — the system does not simply update a probability. It updates a position within a partially ordered set of milestones. This update is path-dependent: the same signal has different implications depending on which milestones have already been reached. A military deployment after diplomatic deterioration means something different than a military deployment before any diplomatic context.

Two SimpleFunctions theses can share the same endpoint and even the same current probability estimate but carry different confidence levels because their causal orderings differ. A thesis that has reached milestones A, B, C in the expected order carries higher structural confidence than one that has reached C, A — skipping B entirely — even if both currently estimate a 60% terminal probability. The ordering of causal nodes is itself informational. The non-abelian structure is the signal.

## When Abelian Becomes Non-Abelian: A Phase Transition as Trading Signal

There is a subtler point here, and it may be the most practically useful one: the transition from abelian to non-abelian structure is itself detectable and tradeable.

In stable geopolitical environments, many risk factors are approximately independent and approximately commutative. The order in which routine economic data releases arrive does not materially change their combined impact. Whether the jobs report comes before or after the CPI print, the market integrates both into a roughly similar equilibrium. The effective group structure is abelian. Linear models work. VaR is adequate. Prediction market prices are reasonable summaries.

But there are moments when this structure breaks. When events begin to develop order-dependence — when "A then B" starts producing materially different outcomes than "B then A" — the effective algebra of risk factors transitions from abelian to non-abelian. This transition is a phase change in the mathematical structure of the environment.

How do you detect it? One approach: track pairs of risk factors and measure the "commutator" — the difference in expected outcome between the two orderings. In a stable, abelian regime, this commutator is near zero for most pairs. As the environment becomes non-abelian, commutators grow. Specifically, monitor hypothetical conditional sequences: "If A has occurred, what is P(outcome | B next)?" versus "If B has occurred, what is P(outcome | A next)?" When these diverge significantly, the system has entered a non-abelian regime.

This detection has direct trading implications. In an abelian regime, binary prediction market contracts are reasonable instruments: the scalar price is a decent summary because path does not matter much. In a non-abelian regime, binary contracts become systematically mispriced — not because the market is irrational, but because the instrument itself cannot represent the relevant structure. The gap between the contract's abelian encoding and the environment's non-abelian reality is an arbitrage opportunity, but only for those who can model the path structure that the contract discards.

At SimpleFunctions, when the milestone tracking system detects growing commutators across geopolitical theses — when the ordering of signals starts to matter more — this is flagged as a regime indicator. Positions that depend on specific orderings are treated differently from positions that are order-insensitive. The confidence intervals widen not because uncertainty has increased in the scalar sense, but because the path space has become richer and the binary contract is now a worse projection of the underlying reality.

## The Limits of the Analogy and Why It Still Works

A mathematician will object, correctly, that geopolitical events do not literally form a group. There is no inverse of a war. You cannot "undo" a sanctions regime by applying an anti-sanctions element. The algebraic structure is more accurately a monoid (associative binary operation with identity, but no guaranteed inverses) or perhaps a category (where composition is defined but not every morphism is invertible).

This is a fair objection and an irrelevant one. The point of the abelian/non-abelian distinction is not to claim that geopolitics literally satisfies the group axioms. It is to identify the structural property — commutativity of risk factor composition — that traditional models assume and that reality violates. You do not need a full group structure to ask whether operations commute. You only need a set and a binary operation. If the operation is "apply geopolitical event E to world-state W," then asking whether E1 \circ E2 = E2 \circ E1 is perfectly well-defined without requiring that E1 have an inverse.

The analogy works because it identifies, with algebraic precision, the exact structural assumption that breaks: the assumption that risk factors can be superposed in any order without affecting the result. Every model that makes this assumption — and that includes most of quantitative finance — is, in non-abelian regimes, not merely imprecise but structurally wrong. It is answering a question about a commutative universe while operating in a non-commutative one.

## Practical Implications for March 2026

We are, as of this writing, in a geopolitical environment where several major risk factor pairs exhibit growing commutators. The sequencing of trade policy actions, military posture adjustments, and technology restrictions across the US-China axis is producing measurably order-dependent outcomes. The same applies to energy policy in Europe, where the ordering of renewable transition milestones relative to natural gas supply disruptions materially changes terminal energy price distributions.

Prediction market traders who model these situations as collections of independent binary events — who, in other words, implicitly assume an abelian structure — are systematically underestimating the complexity of the state space. They are pricing contracts as if the path does not matter. It does.

The traders who will outperform in this environment are those who model the path, not just the endpoint. Who track milestones, not just terminals. Who measure commutators and detect the transition from abelian to non-abelian regimes before the scalar price catches up.

The mathematics is old. The application is new. The edge is structural.

---

*SimpleFunctions builds analytical frameworks for prediction markets that go beyond terminal probabilities. Our milestone signal architecture is designed to capture the path-dependent, non-abelian structure of real-world geopolitical risk.*