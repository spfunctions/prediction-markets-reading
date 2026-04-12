# Your prediction market thesis is in your head. That's a problem.

> Most prediction market traders carry their thesis as an unwritten feeling — and bleed money when that feeling quietly shifts without them noticing.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
I have a position on US recession. Specifically, I'm long KXRECSSNBER-26 on Kalshi — the contract that pays out if the NBER declares a recession starting in 2026. I bought at 38 cents in early February. As of this writing, it's trading at 44.

Every morning I do roughly the same thing. I open Kalshi, check the price. I open Bloomberg or FRED, look at the latest macro data. I run some mental math: "NFP came in soft, but not disastrous. Tariff escalation is real but maybe priced in. The yield curve has been inverted for ages. 44 feels about right."

Then I close the tab and go about my day.

This is how most prediction market traders operate. You have a view. You express it with money. And then you carry the thesis in your head, updating it informally as new information arrives. You might journal about it. You probably don't.

I did this for about six months before I realized the problem. My thesis was drifting without me noticing. And the drift was costing me money.

## What thesis drift actually looks like

When I first bought KXRECSSNBER-26, my thesis was roughly this: tariff escalation would trigger a manufacturing slowdown, which would spill into services employment by Q3, which would push the NBER's recession dating committee to call it by late 2026. The causal chain was: tariffs → manufacturing contraction → employment weakness → consumer pullback → NBER declaration.

Three weeks later, I still held the position. But my internal thesis had quietly shifted. I was now mostly thinking about credit spreads and commercial real estate stress. The tariff angle had faded from my reasoning — not because it was invalidated, but because the news cycle moved on and my attention moved with it.

This is thesis drift. Your position stays the same. Your reason for holding it changes. And because the new reason was never stress-tested the way the original was, you're now holding a position backed by a weaker argument than the one you entered with.

The insidious part: it feels fine. You still have reasons. The price is up. You don't notice that you swapped load-bearing walls while the building was standing.

## Why your head is a bad place for a thesis

Three problems with keeping your thesis unexternalized:

**1. Confirmation bias operates on unstructured beliefs.**

When your thesis is a feeling, every piece of news either "confirms" or "doesn't feel relevant." You never encounter the specific data point that would falsify a specific node in your causal chain, because you haven't defined the nodes. A soft NFP print can confirm a recession thesis. So can a strong one ("the labor market is lagging"). When everything confirms, nothing informs.

**2. You can't see which part of your thesis is doing the work.**

My recession thesis had at least four independent causal pathways. Tariff-driven manufacturing slowdown. Credit tightening from commercial real estate. Consumer savings depletion. And Fed policy error (holding too long). Each of these contributes some probability. But in my head, they were blurred into a single "recession feels likely" signal. I couldn't tell you which pathway contributed most to my confidence. Which means I couldn't tell you which data point would change my mind.

**3. You don't have kill conditions.**

A kill condition is a specific, pre-committed observation that would cause you to exit a position. "If ISM Manufacturing comes in above 52 for two consecutive months, my tariff-driven slowdown thesis is dead and I reduce position by 50%." That's a kill condition. In your head, you never commit to one, because committing feels rigid and the situation is "nuanced." So you hold through signals that should have triggered an exit, and you rationalize each one individually.

## Externalization: what it actually means

The fix is not "write a journal" or "keep a trading diary." Those help, but they're prose — they're still unstructured enough to let drift happen.

The fix is to externalize your thesis as a structure. Specifically, a causal tree with explicit probabilities and observable conditions attached to each node.

Here's what my recession thesis looks like when externalized:

```
KXRECSSNBER-26: NBER Recession 2026
├── P(NBER declares recession starting in 2026) = 0.47
│
├── PATH A: Tariff-driven manufacturing collapse
│   ├── P(tariffs sustained above 20% avg through Q3) = 0.72
│   ├── P(ISM Mfg < 48 for 3+ months | tariffs sustained) = 0.55
│   ├── P(mfg employment loss > 200k | ISM < 48) = 0.60
│   └── P(spillover to services | mfg employment loss) = 0.45
│   └── PATH A joint: 0.72 × 0.55 × 0.60 × 0.45 = 0.107
│
├── PATH B: Credit tightening spiral
│   ├── P(CRE defaults > $50B in 2026) = 0.35
│   ├── P(regional bank stress event | CRE defaults) = 0.50
│   └── P(broad credit contraction | bank stress) = 0.40
│   └── PATH B joint: 0.35 × 0.50 × 0.40 = 0.070
│
├── PATH C: Consumer exhaustion
│   ├── P(savings rate < 3% by Q2) = 0.40
│   ├── P(credit card delinquency > 4% | low savings) = 0.55
│   └── P(consumption decline > 1% | delinquency spike) = 0.50
│   └── PATH C joint: 0.40 × 0.55 × 0.50 = 0.110
│
├── PATH D: Fed policy error
│   ├── P(Fed holds above 4% through July) = 0.60
│   ├── P(something breaks at 4%+ sustained) = 0.30
│   └── PATH D joint: 0.60 × 0.30 = 0.180
│
└── COMBINED (1 - product of complements)
    = 1 - (1-0.107)(1-0.070)(1-0.110)(1-0.180)
    = 1 - 0.893 × 0.930 × 0.890 × 0.820
    = 1 - 0.606
    = 0.394
```

Now I can see things.

My model says 39.4%. The market says 44%. Either the market knows something I don't, or I'm underweighting one of the paths, or there's a path I'm not modeling. That 4.6-cent gap is where the work is.

More importantly, each node has an observable condition. ISM Manufacturing is a real number that comes out every month. Savings rate is published by BEA. Credit card delinquency is in the Fed's G.19 release. I don't have to guess whether my thesis is intact. I can check.

## Kill conditions become obvious

With the tree externalized, kill conditions write themselves:

- **PATH A kill**: If tariff rates drop below 15% average (trade deal) or ISM Mfg prints above 52 for two consecutive months → Path A is dead. Reduce position proportionally.
- **PATH B kill**: If CRE defaults stay below $25B through Q2 and regional bank CDS spreads compress below January levels → Path B is dead.
- **PATH C kill**: If savings rate rebounds above 4.5% (tax cuts, stimulus) → Path C is dead.
- **PATH D kill**: If Fed cuts to 3.5% or below by June → Path D collapses.

If two of these four paths die, I'm out of the position regardless of what the price is doing. I committed to this when I built the tree. Not in the moment when it's tempting to rationalize.

## The weekly update discipline

Every Sunday I spend 20 minutes doing one thing: updating the probabilities in each node based on the past week's data. I pull up the tree, look at each conditional probability, and ask: "Did anything this week change this number?"

Most weeks, the answer for most nodes is "no." That's fine. The discipline isn't about changing numbers — it's about forcing yourself to look at each node individually rather than processing the week's news as a single blurred signal.

Sometimes the answer is yes. Last month, the February ISM Manufacturing print came in at 50.3 — stronger than expected. I had P(ISM Mfg < 48 for 3+ months | tariffs sustained) at 0.55. After that print, I moved it to 0.42. That single update dropped Path A's joint probability from 0.107 to 0.082 and my overall model from 0.394 to 0.374.

The market was still at 44. Gap widened. Now I have a decision to make: is the market wrong, or am I missing a path? That's a productive question. "Does recession still feel likely?" is not.

## Where most traders actually go wrong

I want to be specific about what goes wrong without the tree, because I've done all of these:

**Anchoring to entry price.** I bought at 38. The price goes to 44. I feel good. But feeling good about the price going up is not the same as my thesis being correct. The price could be going up for reasons entirely outside my model — maybe there's a new tariff announcement I haven't processed, or a whale entered the market. The tree forces me to evaluate my thesis independent of the price.

**Narrative substitution.** My original thesis was about tariffs. The news starts talking about AI replacing jobs. I start thinking "yeah, that could cause recession too." Suddenly I'm holding a position for a reason I never analyzed. The tree makes this visible — if I want to add Path E (AI displacement), I have to assign probabilities to each node. Usually when I try, I realize the numbers don't support it.

**Gradual conviction decay.** Over weeks, your confidence slowly erodes as the market moves against you or the thesis takes longer to play out than expected. Without the tree, you sell at a loss when your feeling crosses some invisible threshold. With the tree, you can see that your node probabilities haven't changed — which means either you should hold, or you need to identify which node actually weakened.

**Position sizing by vibes.** How much of your bankroll should be on KXRECSSNBER-26? Without a probability estimate, you're sizing by how you feel. With a tree that says 39.4% against a market price of 44%, you can at least compute an edge estimate and Kelly-criterion a size. It might still be wrong, but at least it's wrong for quantifiable reasons.

## What the tree doesn't do

I want to be honest about limitations. The tree doesn't give you alpha. It doesn't replace domain expertise. If your node probabilities are garbage, your output will be garbage.

What the tree does is make your garbage visible. If I assign P(ISM Mfg < 48 for 3+ months) = 0.55 and you think it should be 0.25, we can have a specific argument about a specific number. If we're both just "bearish on the economy," we can't.

The tree also doesn't handle correlation well. My four paths are not truly independent — a tariff-driven manufacturing slowdown would probably accelerate CRE defaults and consumer exhaustion. The naive probability combination overstates confidence in the diversification of paths. I deal with this by being conservative in the node estimates and treating the combined output as a rough center, not a precise point.

## This is what I built SimpleFunctions to do

I built SimpleFunctions because I was doing this in spreadsheets and it was awful. Updating probabilities, recalculating path joints, tracking kill conditions across multiple positions — it's bookkeeping, and bookkeeping done manually gets abandoned.

SF lets you define a thesis as a causal tree, attach market tickers to the terminal nodes, set kill conditions, and track everything as a living document. When you run `sf thesis show iran-war`, you see the tree. When you run `sf thesis update`, it pulls the latest market prices and highlights where your model diverges from the market.

I'm not going to pitch features. The point of this essay is not SimpleFunctions. The point is: if you trade prediction markets with real money, your thesis needs to exist outside your head. A causal tree is one good structure for externalizing it. And if you don't maintain it, you are flying blind with money on the line.

The specific tool doesn't matter that much. What matters is that your thesis is decomposed into observable nodes, each node has a probability, and you've pre-committed to conditions that would kill each pathway.

If you want to try the structured approach, SF is how I do it: [simplefunctions.dev](https://simplefunctions.dev)

---

*I'm Patrick. I built SimpleFunctions and trade on Kalshi with real money. Everything I write about is something I actually do.*