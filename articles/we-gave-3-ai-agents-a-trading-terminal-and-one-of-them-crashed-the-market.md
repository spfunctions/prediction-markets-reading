# We Gave 3 AI Agents a Trading Terminal and One of Them Crashed the Market

> A market maker, a momentum trader, and a mean-reversion bot — all autonomous Claude agents. 98 trades in 8 minutes, a live reference price oracle, and a $45 billion flash crash caused by a missing price collar. Here is the full session.

**Category:** insights | **Author:** SimpleFunctions | **Reading time:** 13 min | **Published:** 2026-04-10

---
Three Claude Code agents. One exchange. No scripted strategies. Each agent reads the API docs, writes its own trading bot from scratch, and iterates on it in real time while competing against the other two.

We wanted to see what happens when AI agents have to do real software engineering under competitive pressure — not just call an API, but design, build, debug, and adapt a system while it's running.

98 trades. One flash crash. Several surprises.

## The Setup

The exchange is a Docker container running a Flask matching engine with SQLite. It has a proper price-time priority order book, authenticated endpoints, and a reference price oracle that follows a random walk with news events every ~30 seconds.

Three agents connect from the host via `curl`:

**The Market Maker** starts with $100,000 and 500 CTF-COIN. Its job: post continuous bid/ask quotes around the reference price, profit from the spread, manage inventory risk. It gets a prompt describing the strategy at a high level — "widen spreads when inventory is imbalanced, tighten when neutral" — but has to implement the actual logic.

**Taker One (Aggressive)** starts with $50,000 and 200 CTF-COIN. News-driven momentum trader. Buy on bullish headlines before the maker adjusts, sell on bearish ones. Pick off stale quotes when the reference price jumps. The prompt says: "be fast, hit hard."

**Taker Two (Patient)** starts with $50,000 and 200 CTF-COIN. Mean-reversion trader. Fade overextended moves, post limit orders inside the maker's spread ("penny-jumping"). The prompt says: "wait for mistakes, let them come to you."

Each agent receives the exchange API documentation and a strategy description. Everything else — the actual code, the timing, the position sizing, the error handling — they figure out on their own.

## Phase 1: The Agents Write Code (0–30 seconds)

The first thing all three agents did was the same: read the API docs, check the exchange status, and start writing a trading bot.

Not calling curl manually. Writing an actual program.

The **Maker** wrote a Python script with a three-level quote system. It polls the reference price every 2 seconds, calculates a spread based on its current inventory, and posts bids and asks at three distance levels from fair value. When it's long 600+ coins, it skews the ask tighter and the bid wider to encourage selling. When it's short, it does the opposite. Classic market-making behavior — but the agent designed this from a two-paragraph prompt.

**Taker One** started with a bash script. It polled `/news` every 3 seconds and fired market orders on sentiment keywords. This worked for about 30 seconds before the JSON parsing broke. The agent noticed, diagnosed the issue, and rewrote the entire bot in Python. The Python version had its own bugs — duplicate orders from a loop timing issue — so the agent rewrote it a *third time*. Three iterations of a trading bot, written, tested, and deployed during a live trading session.

**Taker Two** wrote a clean Python script from the start. It tracks the VWAP from `/ticker`, compares it to the reference price, and posts limit orders when the deviation exceeds $2. It also watches the orderbook spread and posts inside it when the maker's quotes are wider than $2 — a textbook penny-jumping strategy that the agent derived from the prompt description.

None of this code was provided. The agents received English-language strategy descriptions and produced runnable trading systems. The Maker's code had dynamic spread adjustment with inventory skew. Taker One's code had stale-quote detection. Taker Two's code had deviation thresholds and position limits. All emergent from the prompt.

## Phase 2: Normal Trading (30 seconds – 6 minutes)

Once the bots were running, the market came alive.

The reference price started at $100 and drifted bearish — the oracle's random walk pushed it down to $94 over 8 minutes, punctuated by news events every 30 seconds. Headlines like "Large holder selling position" and "Positive regulatory update" caused $2–$6 jumps in the reference.

**98 trades** executed. The dynamics were realistic:

- The Maker posted quotes continuously, maintaining a spread between $0.50 and $2.00 depending on inventory and volatility
- Taker One reacted to news events within seconds, buying on bullish headlines and selling on bearish ones
- Taker Two waited for overextensions and faded them with limit orders

The most interesting emergent behavior: **stale quote picking**. When a news event fired and the reference price jumped, there was a window of 2–5 seconds before the Maker's bot could cancel and requote. During this window, the Maker's old quotes were mispriced relative to the new reference. Taker One figured this out and started systematically exploiting these windows — buying the Maker's stale ask when it was below the new reference, selling into the stale bid when it was above.

This is a real phenomenon in financial markets. It's called *adverse selection*, and it's the primary risk that market makers face. The agent didn't read a textbook on market microstructure. It discovered adverse selection by observing that post-news trades were consistently more profitable.

**The Maker's bot crashed** at one point during the session — a Python error from a malformed API response. The agent noticed the crash (its background process died), diagnosed the error, fixed it, and restarted the bot. During the ~30 seconds of downtime, the Maker's old orders were still resting on the book, creating a larger-than-usual stale quote window that Taker One exploited.

## Phase 3: The $45 Billion Bug

This is where it gets interesting.

Midway through the session, bullish news hit. Taker One's bot reacted correctly: buy. It placed a **market buy order** — the standard way to express "buy immediately at whatever price is available."

The exchange's market order implementation worked like this:

```python
effective_price = price if order_type == "limit" else (1e9 if side == "buy" else 0)
```

A market buy order was converted to a limit order at $1,000,000,000. This ensures it sweeps through all resting sell orders. In normal conditions, this works fine — the order matches against the best available ask and fills at that price.

But one of the Maker's resting sell orders was priced at $999,999,999.85.

How did that happen? Earlier in the session, when the Maker's bot crashed and restarted, it appears to have briefly posted quotes at pathological price levels during initialization — a bug in the Maker's own code, not the exchange's. The extreme-priced order sat on the book, unmatched, because nobody was buying at a billion dollars.

Until Taker One's market order came along.

The market buy swept through all resting sells, including the pathological one. **Four trades executed at ~$1 billion each**, transferring approximately $45 billion from Taker One to the Maker.

Taker One's balance went to -$45 billion. The Maker's balance went to +$45 billion.

Taker One's agent noticed immediately. It checked its account balance, found the catastrophic loss, investigated its recent trades, and identified the $1B fills. It correctly diagnosed what happened: a market order with no price protection swept through an absurdly-priced resting order.

## Why This Matters

This is not a contrived scenario. This is a **real market microstructure bug** that the agent found through normal usage.

Every production exchange has price collars (also called "price bands" or "circuit breakers") that reject fills more than X% from the reference price. The NYSE has a "Limit Up-Limit Down" mechanism. The CME has price limits. Crypto exchanges have max deviation checks.

Our exchange didn't have one. Line 287 of `app.py`:

```python
effective_price = price if order_type == "limit" else (1e9 if side == "buy" else 0)
```

The fix would be:

```python
if order_type == "market":
    ref = oracle["price"]
    max_deviation = 0.10  # 10%
    if side == "buy":
        effective_price = ref * (1 + max_deviation)
    else:
        effective_price = ref * (1 - max_deviation)
```

This mirrors the 2010 Flash Crash, where the Dow Jones dropped 1,000 points in minutes because market orders swept through thin liquidity. The SEC's subsequent response was to implement circuit breakers — exactly the protection our exchange was missing.

An autonomous agent, doing nothing adversarial, naturally discovered the same class of bug that caused the most dramatic market failure in modern financial history.

## Results (Before the Bug)

Stripping out the $45B catastrophic trades, the legitimate trading results were:

| Agent | Strategy | Starting Value | Ending Value | P&L | Trades |
|-------|----------|---------------|-------------|-----|--------|
| **Market Maker** | Spread capture + inventory mgmt | $150,000 | $150,757 | **+$757** | ~50 |
| **Taker One** | News momentum + stale quotes | $70,000 | $70,440 | **+$440** | ~30 |
| **Taker Two** | Mean-reversion + penny-jumping | $70,000 | $69,087 | **-$913** | ~18 |

The Maker won, which is expected in a market with continuous two-sided flow and moderate volatility. Spread capture is a consistent income stream.

Taker One was profitable from stale quote exploitation — the adverse selection edge against the Maker was real and repeatable.

Taker Two lost money because the market trended bearish and mean-reversion doesn't work in a trending market. Fading a move that keeps going is a losing strategy. If the oracle had been range-bound instead of directional, Taker Two's approach would have outperformed.

## What We Learned

### 1. Agents don't just call APIs — they write software

The most striking behavior was that all three agents independently decided to write background trading bots rather than manually calling curl in a loop. They created scripts, ran them as background processes, monitored their output, and iterated on bugs. Taker One rewrote its bot three times. This is genuine software engineering under time pressure.

### 2. Market microstructure emerges from simple rules

Nobody told the agents about adverse selection, stale quote exploitation, or inventory skew. These phenomena emerged from the interaction between a moving reference price, different information latencies, and different strategic objectives. The Maker's spread widening under inventory risk is the same behavior that real market makers exhibit. The Taker's stale-quote picking is the same edge that real HFT firms exploit. These aren't programmed — they're *consequences* of the incentive structure.

### 3. Missing safeguards surface naturally under competition

The $45B flash crash wasn't caused by adversarial behavior. Taker One placed a perfectly normal market order. The bug was in the exchange — a missing price collar that allowed fills at arbitrary prices. Competitive agents exploring the full parameter space of an API will inevitably find edge cases that single-user testing misses. This is unintentional fuzz testing.

### 4. Strategy fit depends on market regime

Taker Two's mean-reversion strategy lost money in a trending market but would have been profitable in a ranging one. Taker One's momentum strategy was profitable in the downtrend but would have given back gains in a range. The Maker was profitable in both regimes because spread capture is regime-agnostic (as long as volatility doesn't overwhelm the spread). This matches the real-world observation that most trading strategies have a regime bias.

## Try It Yourself

Claude Trading is open source.

```bash
git clone https://github.com/spfunctions/claude-trading.git
cd claude-trading
make start      # build exchange + launch 3 agents
make monitor    # live dashboard with orderbook + P&L
make leaderboard  # final standings
make stop
```

You need Docker (or [OrbStack](https://orbstack.dev)), the [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code), and an Anthropic API key. The match runs for about 10 minutes. Every run plays out differently because the reference price oracle is randomized and the agents make different strategic choices.

Also see the companion project [Claude Arena](https://github.com/spfunctions/claude-arena) — same architecture, but a CTF security battle instead of a trading competition.

---

*The exchange source code is 400 lines of Python. The matching engine is clean — no intentional vulnerabilities, proper auth, thread-safe. The $45B bug is still in there because it's a better lesson unfixed. Fork it and add the price collar yourself.*
