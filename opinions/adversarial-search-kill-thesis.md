# Adversarial search: how I try to kill my own thesis before trading on it

> The single most valuable feature in my trading system is the one that actively tries to prove me wrong every 15 minutes.

**Category:** essay | **Author:** Patrick Liu | **Reading time:** 10 min

---
I lost $2,400 on an Iran ceasefire I didn't see coming.

Not because the information wasn't available. It was. Reuters had it. Al Jazeera had it. A Swiss diplomatic back-channel had been active for two weeks. I just didn't look, because I was long on war continuation and everything I was reading confirmed that view.

That loss taught me more than any win. It taught me that the most dangerous state a trader can be in is *confident and correct so far*. Because that's when you stop looking for reasons you're wrong.

So I built a system that looks for me.

## The problem: you are not objective

Every trader thinks they're being objective. Nobody is.

This isn't a character flaw. It's how human cognition works. Once you form a thesis — "Iran war continues, oil stays elevated, recession follows" — your brain starts filtering information. You notice the headlines that confirm your view. You skim past the ones that don't. You weight a single hawkish Pentagon quote more heavily than three diplomatic signals pointing toward de-escalation.

Psychologists call this confirmation bias. Traders call it "doing your own research." The result is the same: you build an increasingly confident position on an increasingly narrow information diet.

I've watched myself do this. I've watched smart traders do this. I've watched quantitative systems do this, because the humans building them chose which signals to include and which to ignore.

The fix isn't willpower. You can't willpower your way out of a cognitive bias. The fix is a system that doesn't have the bias in the first place.

## The mechanism: adversarial search

Every 15 minutes, my system runs a heartbeat cycle. Most of what that cycle does is straightforward: refresh prices, check orderbooks, scan news for thesis-relevant signals. But there's one step that took me longer to build than everything else combined.

Adversarial search.

Here's how it works. For every active thesis, the system generates search queries specifically designed to *disprove* the thesis. Not confirm it. Disprove it.

If my thesis is "Iran war continues," the system doesn't search for "Iran military escalation" or "Hormuz strait tensions." Those would confirm my view. Instead, it generates queries like:

- `iran ceasefire deal progress`
- `us iran diplomatic back channel`
- `hormuz strait reopening timeline`
- `iran moderate faction gaining influence`
- `gulf state mediation iran deal`

These are the queries I would never run myself. Because I don't want to find a ceasefire deal. I'm long on war. A ceasefire deal is bad for my position. My brain literally does not want to look.

The system doesn't care about my position. It looks anyway.

## Real example: the ceasefire signal

Let me show you what this looks like in practice. Here's actual terminal output from an adversarial search cycle on one of my Iran war thesis nodes:

```
[heartbeat] thesis=iran-war cycle=4827 phase=adversarial-search
[adversarial] node=n1.3 "No diplomatic exit" (current: 82%)
[adversarial] generating counter-queries...
  q1: "iran ceasefire negotiations 2026"
  q2: "us iran diplomatic channel switzerland"
  q3: "iran moderate coalition government formation"
  q4: "gulf cooperation council iran mediation progress"
[adversarial] searching q1... 3 results
[adversarial] searching q2... 7 results
  >> SIGNAL DETECTED: "Swiss-mediated back-channel between US and Iranian
     officials has produced a preliminary framework for de-escalation,
     according to two diplomats briefed on the talks" (Reuters, 2h ago)
[adversarial] searching q3... 1 result
[adversarial] searching q4... 2 results
[adversarial] evaluating 13 results against node n1.3...
[eval] counter-evidence strength: MODERATE
[eval] source reliability: HIGH (Reuters, named diplomats)
[eval] node n1.3 adjustment: 82% → 68% (delta: -14)
[cascade] n1 "War persists": 88% → 79%
[cascade] n2 "Hormuz blocked": 97% → 89%
[cascade] n3 "Oil stays elevated": 64% → 52%
[cascade] n4 "Recession": 45% → 34%
[edges] recalculating...
  Recession 2026 YES: edge 37¢ → 22¢ (shrunk)
  WTI $150 YES: edge 37¢ → 19¢ (shrunk)
[alert] 2 edges dropped below 25¢ threshold
```

That's the system finding something I wouldn't have found myself. A Reuters report about Swiss-mediated back-channel talks. Two hours old. My confirmation-biased brain would have scrolled past it. The system flagged it, evaluated it, and adjusted my entire causal tree.

Node n1.3 ("No diplomatic exit") dropped from 82% to 68%. That's a meaningful shift. And because the causal tree is connected, everything downstream moved too. War persistence dropped. Hormuz blockade probability dropped. Oil and recession edges shrank.

Two of my edges fell below the minimum threshold I set. That's a signal to reduce position size or exit entirely.

## What happens when adversarial search finds something real

The cascade I showed above is the normal case: counter-evidence shifts probabilities, edges shrink, you adjust. But there's a more dramatic scenario.

Kill conditions.

Every thesis has nodes with kill conditions attached. For my Iran war thesis, one kill condition is: if n1.3 ("No diplomatic exit") drops below 40%, the thesis is structurally dead. War without diplomatic exit is the foundation. If diplomacy succeeds, the whole chain — Hormuz, oil, recession — collapses.

Here's what that looks like:

```
[heartbeat] thesis=iran-war cycle=5104 phase=adversarial-search
[adversarial] node=n1.3 "No diplomatic exit" (current: 51%)
  >> SIGNAL DETECTED: "Iran and US agree to 72-hour ceasefire
     effective immediately, with framework for permanent
     de-escalation" (AP, breaking)
[eval] counter-evidence strength: CRITICAL
[eval] node n1.3 adjustment: 51% → 22%
[KILL] node n1.3 breached kill threshold (40%)
[KILL] thesis iran-war marked DEAD
[KILL] all edges invalidated
[KILL] recommended action: EXIT ALL POSITIONS
```

The system doesn't just tell you your edge shrunk. It tells you your thesis is dead and you should get out. This is the kind of signal you need at 2 AM when breaking news hits and you're asleep. The system isn't asleep. It's running its heartbeat cycle, running adversarial search, and it caught the thing that kills your thesis before the market fully prices it in.

## The query generation problem

The hardest part of adversarial search isn't the searching. It's generating the right queries.

Bad adversarial queries are just negations of the thesis: "iran war not happening." That's useless. Nobody writes articles titled "Iran war not happening." The information that would disprove your thesis lives in articles about ceasefire progress, diplomatic back-channels, moderate political movements, economic pressure to de-escalate.

The system uses the causal tree structure to generate targeted counter-queries for each node. For node n1.2 ("Iran continues retaliation"), adversarial queries target Iranian domestic politics, military fatigue, economic strain from sanctions. For node n2.1 ("Mines deployed in Hormuz"), adversarial queries target minesweeping operations, alternative shipping routes, Hormuz transit resumption.

Each node gets its own set of adversarial queries because each node can be independently falsified. That's the power of having a causal tree instead of a monolithic thesis.

## Why not just monitor all news?

You might be thinking: why not just monitor everything and let relevance filtering handle it?

I tried that. It doesn't work. The problem is volume. There are thousands of articles per day about the Middle East. Most are noise. A general news monitor produces so many signals that you either (a) tune the sensitivity down until you miss the important stuff, or (b) drown in alerts.

Adversarial search is surgical. It generates 3-5 queries per node, per cycle. It's looking for *specific* types of information — the kind that would break specific links in your causal chain. It's not scanning the ocean. It's checking the five places where a crack would be fatal.

This focus is what makes it fast enough to run every 15 minutes. A full news scan would take hours and produce garbage. Adversarial search takes seconds and produces targeted counter-evidence.

## The psychological benefit

There's something that happens to your trading psychology when you know a system is actively trying to kill your thesis.

You relax.

Not in a complacent way. In a "I don't need to constantly worry about what I'm missing" way. Before I built adversarial search, I'd find myself compulsively checking news at 11 PM, looking for the thing that would blow up my position. Not because I enjoyed it, but because I was terrified of missing it.

Now I don't do that. The system checks every 15 minutes. It checks the specific things that would kill my thesis. If there's something to worry about, it tells me. If there isn't, I sleep.

This isn't a small thing. Trading decisions made at 11 PM in a state of anxiety are bad decisions. Decisions made at 9 AM after the system's overnight report says "adversarial search found nothing material" are better decisions.

## The meta-insight

Building a system that actively tries to prove you wrong is the single most valuable thing you can do as a trader.

Not because the system is smarter than you. It's not. It's an LLM evaluating search results against a causal tree. It makes mistakes. It sometimes overweights a weak source. It sometimes misses nuance in a diplomatic statement.

But it doesn't have confirmation bias. It will search for "iran ceasefire" with the same diligence it searches for "iran escalation." It will evaluate counter-evidence without flinching, because it has no position to protect.

The combination of human thesis construction and machine adversarial search is, in my experience, significantly better than either alone. I bring the domain knowledge, the causal reasoning, the ability to construct a thesis that maps to real-world dynamics. The system brings the discipline to continuously try to destroy that thesis.

When the thesis survives adversarial search cycle after cycle, my confidence in the trade is earned, not assumed. When adversarial search finds a crack, I find out before the market does.

## The numbers

I've been running adversarial search on my Iran war thesis for four months. In that time:

- 11,520 adversarial search cycles completed (4 months x 30 days x 96 cycles/day)
- 287 counter-signals detected and evaluated
- 23 resulted in node probability adjustments greater than 5%
- 3 triggered edge recalculations that changed my position sizing
- 1 would have triggered a kill condition (the ceasefire scare in February, which reversed within 48 hours)

Of those 287 counter-signals, roughly 240 were noise — the system correctly evaluated them as not material enough to shift probabilities. The remaining 47 were genuine signals that updated my model, mostly small adjustments of 2-5%.

The one that mattered most was a diplomatic back-channel report that dropped my "no diplomatic exit" node by 14 points. I reduced my position size by 30% that evening. Two days later, the talks stalled and the node recovered. But those two days of reduced exposure were the right call given what was known at the time.

That's the thing about adversarial search. Most of the time, it confirms that your thesis is intact. That confirmation has value too — it's earned confidence, not assumed confidence. And the one time it finds something real, it might save your account.

## How to set this up

If you want adversarial search running against your own thesis, the infrastructure is at [simplefunctions.dev](https://simplefunctions.dev). You write the thesis, decompose it into a causal tree, and the heartbeat engine handles the rest — including adversarial search on every cycle.

I trade on Kalshi with real money. This system manages my real positions. Everything I described above is what actually runs, not a hypothetical.

The thesis is yours. The adversarial search is the system's. That's the division of labor that works.