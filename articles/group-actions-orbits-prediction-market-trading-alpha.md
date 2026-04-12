# Group Actions and Orbits: Why the Same Event Has Different Value for Different Traders

> The mathematics of symmetry explains why two rational traders can look at the same headline and reach opposite conclusions — and why both can be right about different things.

**Category:** insights | **Author:** SimpleFunctions Research | **Reading time:** 10 min | **Published:** 2026-03-26

---
# Group Actions and Orbits: Why the Same Event Has Different Value for Different Traders

*The mathematics of symmetry explains why two rational traders can look at the same headline and reach opposite conclusions — and why both can be right about different things.*

## The Algebra of Perspective

Every trader has a mental model. When a headline crosses the wire — "Iran seizes tanker in Strait of Hormuz" — some traders immediately think *oil supply disruption* and start pricing energy futures. Others think *escalation signal* and start pricing defense contracts and regional stability. Same event, completely different responses. This is not noise. This is structure, and it has a name in mathematics: **group actions on sets**.

A group action is one of the most powerful ideas in abstract algebra. A group *G* acts on a set *X* when each element of *G* defines a transformation of *X* that respects the group's structure. The action partitions *X* into **orbits** — subsets of elements that are interchangeable under the group's operations. Elements within the same orbit are, from the perspective of *G*, indistinguishable. Elements in different orbits are fundamentally different.

The claim of this article is direct: a trader's analytical framework — the set of transformations they apply to news, data, and scenarios — constitutes a group. Different frameworks constitute different groups. The same set of world events, acted upon by different groups, decomposes into different orbits. And the places where two traders' orbit decompositions disagree are exactly where trading edge lives.

## Group Actions: The Formal Setup

Let us fix notation. A **group** (*G*, ·) is a set with an associative binary operation, an identity element, and inverses for every element. A **group action** of *G* on a set *X* is a function *G* × *X* → *X*, written (*g*, *x*) ↦ *g* · *x*, satisfying two axioms:

1. **Identity:** *e* · *x* = *x* for all *x* ∈ *X*, where *e* is the identity element of *G*.
2. **Compatibility:** (*g* · *h*) · *x* = *g* · (*h* · *x*) for all *g*, *h* ∈ *G* and *x* ∈ *X*.

The **orbit** of an element *x* ∈ *X* under *G* is the set Orb(*x*) = {*g* · *x* : *g* ∈ *G*}. Orbits partition *X* — every element belongs to exactly one orbit, and two orbits are either identical or disjoint. The **stabilizer** of *x* is the subgroup Stab(*x*) = {*g* ∈ *G* : *g* · *x* = *x*}, the set of group elements that fix *x* in place.

The orbit-stabilizer theorem tells us that |Orb(*x*)| = |*G*| / |Stab(*x*)|. The bigger the stabilizer, the smaller the orbit. An element with a large stabilizer is one that many transformations leave unchanged — it is robust, symmetric, and in a sense, simple. An element with a small stabilizer is fragile under the group's operations — it is asymmetric and informationally rich.

Now translate this into markets.

## The Macro Trader's Group

Consider a macro trader — call her Alice. Alice's analytical framework consists of a set of transformations she applies to geopolitical and economic events. Her group *G_A* includes operations like:

- **Supply-demand substitution:** If event *x* reduces oil supply by 2 million barrels per day, it is equivalent to any other event that reduces supply by 2 million barrels per day.
- **Currency flow equivalence:** A tariff that redirects $50B in trade flows is equivalent to a sanctions regime that redirects $50B in trade flows.
- **Rate-sensitivity mapping:** Any event that moves the Fed's reaction function by 25 basis points is equivalent to any other such event.

Under Alice's group action, the set of world events *X* partitions into orbits defined by macroeconomic impact. Consider two specific events:

- *x₁*: "Iran blocks the Strait of Hormuz"
- *x₂*: "OPEC announces 2M bpd production cut"

Alice's group contains a transformation *g* such that *g* · *x₁* = *x₂*. Both events remove roughly the same volume from global supply. Both events raise Brent crude by similar magnitudes. Both events trigger the same second-order effects on petrochemical margins, shipping rates, and inflation expectations. Under *G_A*, these two events are in the **same orbit**. They are, for Alice's purposes, the same event wearing different clothes.

The stabilizer of *x₁* under Alice's group is relatively small — there are many macroeconomic transformations that map it somewhere else (a different supply shock of a different magnitude, for instance). But crucially, Alice cannot distinguish *x₁* from *x₂*. Her framework compresses them.

## The Geopolitical Analyst's Group

Now consider Bob, a geopolitical analyst. Bob's group *G_B* includes operations like:

- **Actor substitution:** Replace Iran with North Korea in a coercion scenario. If the strategic logic changes, the events are in different orbits.
- **Escalation ladder mapping:** Events are equivalent only if they sit at the same rung of an escalation ladder between the same actors.
- **Alliance signaling equivalence:** Events are equivalent if they send the same signal to the same set of allied and adversarial states.

Under *G_B*, the same two events land in **completely different orbits**:

- *x₁* ("Iran blocks Hormuz") is a military coercion move. It sits on an escalation ladder between Iran and the United States. It signals resolve to Gulf states, triggers alliance obligations, and has a nonzero probability of kinetic escalation. It belongs in an orbit with events like "Russia cuts gas to Europe" or "China blockades Taiwan" — acts of coercive denial between great powers.

- *x₂* ("OPEC cuts production") is an economic bargaining move among ostensible partners. It signals coordination among producers, adjusts fiscal breakeven calculations for Saudi Arabia, and is fundamentally reversible without loss of face. It belongs in an orbit with events like "OPEC+ extends existing cuts" or "Saudi Arabia raises OSPs" — routine commodity diplomacy.

There is no element *g* ∈ *G_B* such that *g* · *x₁* = *x₂*. Under Bob's group action, these events are as different as the integers 3 and 17 under the trivial group action. No symmetry connects them.

## The Same Set, Different Partitions

This is the core insight. Alice and Bob are looking at the same set *X* of world events. They are both rational. They both have internally consistent frameworks. But their groups *G_A* and *G_B* produce different orbit decompositions of *X*.

Where Alice sees one orbit (supply shocks), Bob sees two (military coercion and economic bargaining). Conversely, where Bob might see one orbit (escalation signals from state actors), Alice might see three (events that raise oil prices, events that strengthen the dollar, events that widen credit spreads) because the macroeconomic transmission channels diverge.

Neither decomposition is wrong. Each is **faithful to its group**. The question is not which is correct but which is more useful for a specific prediction market contract.

If the contract is "Will Brent crude exceed $120 by June 2026?", Alice's orbit decomposition may be more useful — she correctly identifies that the *source* of a supply disruption is less important than its *magnitude*. If the contract is "Will the US conduct a military strike on Iran by December 2026?", Bob's decomposition is more useful — he correctly identifies that the Hormuz blockade carries escalation risk that an OPEC cut does not.

## Your Thesis Is Your Group

This is where the SimpleFunctions thesis system enters the picture. When you write a thesis on SimpleFunctions — a structured causal argument about why a prediction market contract is mispriced — you are, in precise mathematical terms, defining a group action on the space of relevant events.

Your thesis specifies which events you consider equivalent (same orbit) and which you consider distinct (different orbits). It specifies your stabilizers — which transformations of the world leave your core argument unchanged, and which break it. A thesis that says "the probability of an Iran war is underpriced because of Hormuz escalation dynamics" implicitly groups all supply disruptions with military origins into one orbit and separates them from economically motivated supply adjustments.

The system then does something powerful: it computes, under your specific orbit decomposition, which events the market appears to be **incorrectly merging** and which it appears to be **incorrectly separating**.

**Incorrect merging** occurs when the market treats two events as equivalent — pricing them with the same impact coefficient — but your thesis places them in different orbits. The market sees one risk factor; you see two. If you are right, one of those risk factors is mispriced.

**Incorrect separation** occurs when the market treats two events as distinct but your thesis places them in the same orbit. The market is double-counting a risk factor. It is pricing the same underlying cause twice, once through each of its surface manifestations. If you are right, the risk is overpriced.

**These two errors — incorrect merging and incorrect separation — are the source of alpha.** Every systematic trading edge can be traced to a case where the market's implicit orbit decomposition disagrees with reality, and your thesis correctly identifies the disagreement.

## Burnside's Lemma and Independent Risk Factors

Burnside's lemma (sometimes called the Cauchy-Frobenius lemma) states that the number of distinct orbits under a group action is:

|*X* / *G*| = (1 / |*G*|) Σ_{g ∈ G} |*X^g*|

where *X^g* = {*x* ∈ *X* : *g* · *x* = *x*} is the set of elements fixed by *g*.

In the trading context, the number of distinct orbits equals the number of **truly independent risk factors** in your model. Burnside's lemma tells you how to count them: average the number of events left unchanged by each transformation in your group.

If your group is very large (you see many symmetries, many equivalences), Burnside's lemma yields a small number of orbits. You believe the world is driven by a few independent factors. Your portfolio will be concentrated. Your thesis is bold.

If your group is small (you see few symmetries, you treat most events as unique), the lemma yields many orbits — close to |*X*| itself. You believe the world is driven by many independent factors. Your portfolio will be diversified. Your thesis is cautious.

Neither is inherently better. But the **accuracy** of your orbit count — whether you have correctly identified the true number of independent risk factors — determines whether your portfolio construction is appropriate for the risks you actually face.

A macro trader who lumps all supply disruptions into one orbit may be underestimating her risk. If the Hormuz scenario and the OPEC scenario have different durations, different reversal probabilities, and different second-order geopolitical consequences, they are not the same risk factor, and Burnside's lemma — applied honestly — should yield a higher orbit count.

## Orbit Disagreements as Edge: A Practical Example

Consider two SimpleFunctions users, each building a thesis on the same prediction market contract: "Will the US and Iran engage in direct military conflict by December 2026?" The contract trades at 12%.

**Thesis A** is built by a macro-focused trader. Her causal tree looks like this:

- Root: US-Iran military conflict
  - Branch 1: Oil price spike above $130 triggers US intervention to secure supply
  - Branch 2: Iran nuclear breakout triggers Israeli strike, drawing in US
  - Branch 3: Proxy escalation in Iraq spills over

Under her group action, Branches 1 and 3 are in the same orbit — both are "escalation through economic pressure" scenarios. She assigns them a combined probability of 6% and Branch 2 a separate 8%, arriving at a total near 14%. She sees the contract as slightly underpriced.

**Thesis B** is built by a geopolitical analyst. His causal tree:

- Root: US-Iran military conflict
  - Branch 1: Hormuz blockade triggers US naval response (escalation ladder logic)
  - Branch 2: Iranian nuclear enrichment crosses US red line (nonproliferation logic)
  - Branch 3: IRGC attack on US base in Syria (proxy war spillover logic)
  - Branch 4: Domestic political incentives in US election year

Under his group action, all four branches are in **different orbits**. None of them are equivalent. The Hormuz scenario and the proxy war scenario, which Thesis A merges, are separated here because they involve different escalation dynamics, different decision-makers, and different off-ramp structures. He assigns them 5%, 4%, 3%, and 2% respectively. After accounting for correlation, he arrives at about 13%. He also sees a slight underpricing, but for different reasons.

Now here is where the mathematics becomes actionable. The edges that differ between Thesis A and Thesis B — the places where one thesis merges what the other separates, or vice versa — are exactly the **orbit disagreements**. These disagreements point to the events whose market pricing is most sensitive to the choice of analytical framework.

If you hold Thesis A, you should pay close attention to the events that Thesis B separates but you merge. If Thesis B is right that the Hormuz scenario and the proxy scenario are fundamentally different risks, you may be miscalculating your exposure. Conversely, if you hold Thesis B, you should examine whether your separation of these scenarios is justified or whether it is a distinction without a meaningful difference.

SimpleFunctions surfaces these disagreements explicitly. When multiple theses exist on the same contract, the system can identify which branches of which causal trees overlap and where they diverge. The overlaps are consensus. The divergences are where the real analytical work — and the real edge — lies.

## Stabilizers and Conviction

The stabilizer subgroup has its own trading interpretation. Recall that Stab(*x*) is the set of group elements that leave *x* fixed. A large stabilizer means many transformations of the world leave your assessment of event *x* unchanged. Translated: your conviction about *x* is robust to many alternative scenarios.

If your thesis on the Iran contract has a large stabilizer — if your probability estimate does not change whether oil is at $80 or $120, whether the US president is hawkish or dovish, whether Israel acts unilaterally or not — then your thesis is highly convicted but potentially brittle. It may be ignoring variables that matter.

If your stabilizer is small — if almost any change in circumstances alters your assessment — then your thesis is flexible but has low conviction. Your position size should reflect this.

The orbit-stabilizer theorem, |Orb(*x*)| · |Stab(*x*)| = |*G*|, enforces a tradeoff: the more robust your assessment (large stabilizer), the smaller the orbit of equivalent events (fewer things look the same to you), and vice versa. Conviction and generalization are in tension. The theorem does not resolve this tension — it quantifies it.

## Building Better Groups

The practical upshot of this framework is a discipline for self-examination. When you build a thesis on SimpleFunctions, you should ask:

**What is my group?** What transformations of the world do I consider irrelevant to my prediction? Every irrelevance assumption is an element of your group. Every element you add to your group merges orbits and reduces your count of independent risk factors.

**What are my orbits?** Which events do I consider equivalent? For each orbit, can I articulate *why* the events in it are interchangeable? If I cannot, the orbit may be too large — I may be incorrectly merging events that carry different information.

**What are my stabilizers?** For each key event, which changes in the world would *not* alter my assessment? If the list is long, my conviction is high but I should stress-test it. If the list is short, I should reduce my position size.

**Where do my orbits disagree with other theses?** The SimpleFunctions platform makes this comparison possible. The disagreements are not noise — they are the most valuable signals in the system.

The mathematics of group actions does not tell you which thesis is correct. It does something more useful: it gives you a precise language for describing how two rational analysts can disagree, where exactly their disagreements lie, and what empirical observations would resolve those disagreements. In prediction markets, where the price is a weighted average of all participants' orbit decompositions, understanding the structure of disagreement is understanding the structure of the market itself.

The next time a headline crosses your screen and you instinctively classify it — supply shock, escalation signal, political noise — notice that you are applying a group element to an event and placing it in an orbit. Notice that someone else, equally informed, is placing it in a different orbit. Then ask the only question that matters for trading: whose orbits are closer to the truth?