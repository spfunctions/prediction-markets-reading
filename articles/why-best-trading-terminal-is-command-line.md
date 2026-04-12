# Why the Best Trading Terminal Is a Command Line

> Bloomberg started as a keyboard. Robinhood ended as confetti. The next generation of trading infrastructure will be judged by how little there is to look at.

**Category:** markets | **Author:** Patrick Liu | **Reading time:** 9 min | **Published:** 2026-03-31

---
The Bloomberg Terminal started as a keyboard. Not a screen — a keyboard. When Mike Bloomberg built the first version in 1982, traders navigated it through typed commands: `EQUITY GO`, `DES`, `HP`. No mouse. No drag-and-drop. No chart you could pinch to zoom. Your hands never left the keyboard, and that was the point, because in markets, the distance between your hand and the execute button is measured in dollars.

Forty years later, every new trading product ships a dashboard. Real-time charts, color-coded heatmaps, streaming candlesticks, portfolio pie charts, notification badges. The modern retail trading interface is a spectacle — and Robinhood completed the arc by literally showering you with confetti when you made a trade, turning the act of putting money at risk into a celebration.

We've spent two decades turning traders into an audience. I think this was a wrong turn. And I think the next generation of trading infrastructure looks less like TradingView and more like a terminal.

---

## The Dashboard Tax

Here's a question nobody asks: what does a dashboard *cost*?

Not in dollars. In attention.

A real-time chart is not inert information. It's a stimulus. Every tick, every candle, every color shift from green to red is a micro-event that your visual cortex processes whether you want it to or not. This is not a metaphor — there's a well-documented phenomenon in behavioral finance called "myopic loss aversion," and the core finding is brutally simple: the more frequently you observe your portfolio, the worse your decisions get. Thaler, Tversky, and Kahneman showed this decades ago. Real-time dashboards are, in a precise technical sense, decision-degradation machines.

But it's worse than that. Dashboards don't just show you information — they frame it. A candlestick chart makes you think in terms of patterns. A portfolio percentage view makes you think in terms of allocation. A P&L ticker makes you think about what you've gained or lost. None of these frames are *wrong*, but they're all *chosen for you*, and each one excludes the frames you're not seeing. The dashboard decides what's salient before you do.

A trader staring at a dashboard is a driver staring at the dashboard instead of the road.

---

## What Bloomberg Got Right (And Then Forgot)

The original Bloomberg insight was that professionals need *speed*, not *spectacle*. A trader who can type `AAPL EQUITY HP` and get a price history in 800 milliseconds is faster than a trader who has to navigate a menu, click a search bar, type a ticker, select from an autocomplete dropdown, click "chart," then select a time range from a set of radio buttons.

The keyboard-first interface wasn't a technological limitation. It was a design philosophy: **the interface should be as thin as possible between intention and execution.**

Bloomberg kept the keyboard commands but layered a graphical interface on top. Then competitors came along and built the graphical-first experience. Then Robinhood came along and built the experience-first experience. Each generation made the interface thicker — more pixels between you and the trade, more visual processing before you reach a decision, more UI state to maintain in your head.

The progression: keyboard → GUI → experience → entertainment.

We need to go backwards.

---

## The Agent Runtime Changes Everything

Here's what's different now: you don't have to be the one watching the market.

For most of trading history, the human was in the loop at every step: observe market → form thesis → identify opportunity → evaluate risk → execute. The dashboard existed to serve step one: observe. It made sense to optimize the *observation* interface because the human was the observer.

But in an agent runtime architecture, steps one through four are delegable. An automated system can ingest price data, run it against a thesis, score the edge, check the spread and liquidity, and present you with a ranked list of executable opportunities — all before you look at anything.

This changes the role of the interface completely. You're no longer the observer. You're the **architect and the reviewer**. You define the thesis. You set the constraints. You review the agent's conclusions. You approve or reject. The interface doesn't need to help you *see* the market. It needs to help you *interrogate* the agent's reasoning and *authorize* its actions.

What kind of interface is optimized for review and authorization?

The same kind that software engineers have used for decades to review and authorize changes to complex systems: a command line.

---

## The Unix Philosophy Comes to Finance

`git` is instructive. Version control is at least as consequential as trading — a bad merge in a production codebase can cost millions — and yet the dominant interface is a CLI. There are GUIs for git (GitHub Desktop, GitKraken, SourceTree), and they're popular with beginners, but professionals overwhelmingly use the command line. Why?

Because the command line is **composable, scriptable, and auditable**.

Composable: `git log --oneline | grep "fix" | wc -l` — three tools piped together to answer a specific question. Try doing that in a GUI.

Scriptable: You can wrap a CLI command in a cron job, a CI pipeline, a pre-commit hook. A GUI action is a dead end — it happens when a human clicks, and then it's over.

Auditable: `git log` is a complete, immutable record of what happened. A series of dashboard clicks leaves no trace.

Now apply this to trading:

```
sf scan --thesis "iran-oil-recession" --min-edge 5 --max-spread 3
```

This is a complete, reproducible, auditable instruction. You can read it. You can modify it. You can run it again tomorrow. You can put it in a cron job that runs at 6 AM and 6 PM. You can pipe the output into another tool. You can diff yesterday's output against today's to see what changed.

Compare this to opening a dashboard, visually scanning a grid of cards, clicking into three or four that look interesting, mentally comparing their edges, and making a decision based on whichever one you happened to look at last.

The CLI version is not just faster. It's *structurally better* — because it makes the decision process explicit, repeatable, and inspectable.

---

## Three Commands, Not Three Screens

When we built SimpleFunctions, we deliberately chose three CLI entry points instead of three dashboard views:

**`sf scan`** runs a full market scan — ingests live prices from Kalshi and Polymarket, evaluates them against an active thesis, computes edge, checks spread and liquidity, and returns a ranked list of opportunities with execution readiness scores. The output is a structured table. Not a chart. Not a heatmap. A table with numbers you can sort, filter, and pipe.

**`sf opps`** is a continuous guardian daemon that monitors existing positions and market conditions, watching for changes that require action — edge decay, spread widening, thesis invalidation signals. It's a `tail -f` for your portfolio, not a portfolio view.

**`sf thesis monitor`** tracks the structural conditions underlying your macro thesis — not the market itself, but the *reasons* you're in the market. Is the Strait of Hormuz still a chokepoint? Has the Iran calculus shifted? This is the layer most dashboards don't even attempt, because it requires integrating news, geopolitical analysis, and market data into a single coherent signal.

Each of these could be a dashboard page. But a dashboard page invites *browsing*. A CLI command demands *specificity*. You have to say what you want. And the act of articulating what you want — choosing the flags, setting the thresholds — is itself a form of thinking that a point-and-click interface lets you skip.

Skipping that thinking is exactly how you lose money.

---

## "But I Need to See the Chart"

No, you need the information that the chart encodes. These are different things.

A candlestick chart encodes open, high, low, close, and volume over time. That's five numbers per period. If you're making a decision based on 30 candles, you're processing 150 numbers — but through a visual channel that's optimized for pattern recognition, not precise comparison.

The problem: pattern recognition is exactly the cognitive mode that generates the most false positives in financial data. You see a "head and shoulders" where there's noise. You see "support" at a round number because your brain likes round numbers. The chart *feels* informative in a way that a table of numbers doesn't, and that feeling is the danger.

A CLI doesn't prevent you from pulling up a chart when you genuinely need spatial/temporal visual analysis. It just doesn't make the chart the *default*. The default is structured data that your analytical mind processes, not visual stimulus that your pattern-matching mind reacts to.

---

## The Audience Problem

There's a market structure reason why every fintech ships a dashboard: **dashboards are what you show investors and what you screenshot for Twitter**. A CLI has no screenshot value. You can't put a terminal output in a pitch deck and make a VC feel something.

This creates a deep misalignment between what's good for the user and what's good for the company. The user needs a thin, fast, precise interface. The company needs an impressive, visual, shareable one. Every pixel of dashboard chrome is a compromise in favor of the company and against the user.

This is Guy Debord's *spectacle* applied to financial infrastructure: the image of trading has become more important than the act of trading. Robinhood's confetti was not a bug or a dark pattern — it was the logical endpoint of an industry that optimized for engagement instead of execution.

A CLI-first product is a bet that there exists a user base — maybe small, maybe growing — that optimizes for the opposite: execution quality over visual satisfaction, decision rigor over engagement, correctness over convenience.

---

## What Comes Next

The Bloomberg Terminal costs $24,000 a year and it's still keyboard-first for power users. That's not an accident or a legacy artifact. It's a signal that the professionals who make the most consequential decisions still prefer the thinnest possible interface.

As agent runtimes become the standard architecture for market interaction — not just in prediction markets, but across all programmatic trading — the interface question will resolve itself. You don't build a dashboard for an agent. You build a control plane. And a control plane looks like a terminal.

The next generation of trading infrastructure will be judged not by how good it looks, but by how little there is to look at.

---

*SimpleFunctions is a prediction market runtime. It has 16 tools, zero UI, and three CLI commands. [simplefunctions.dev](https://simplefunctions.dev)*