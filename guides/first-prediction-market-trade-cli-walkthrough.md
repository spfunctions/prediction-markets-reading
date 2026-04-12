# Your First Prediction Market Trade: End-to-End CLI Walkthrough

> From npm install to your first filled order. Every command, every output, every decision point. The definitive zero-to-first-trade tutorial for prediction market trading with the SimpleFunctions CLI.

**Category:** guide | **Author:** SimpleFunctions | **Reading time:** 11 min

---
This is the tutorial I wish existed when I started trading prediction markets. No theory. No architecture diagrams. Just: open terminal, follow steps, end up with a real position on a real market.

Every command is real. Every output is what you'll actually see. Every decision point is explained so you understand *why*, not just *what*.

## Prerequisites

Before you start, you need:

1. **Node.js 18+** — run `node --version` to check
2. **A Kalshi account** — sign up at kalshi.com (free, takes 5 minutes)
3. **Kalshi API keys** — from your Kalshi dashboard: Settings → API Keys → Create
4. **An OpenRouter API key** — from openrouter.ai/keys (for the LLM that powers analysis)
5. **$100+ in your Kalshi account** — you need capital to trade

Total setup time: 10 minutes for accounts, 5 minutes for the walkthrough below.

## Step 1: Install the CLI

```bash
npm install -g @spfunctions/cli
```

Verify it works:

```
$ sf --version
@spfunctions/cli 1.2.4
```

If you see a version number, you're good. If you get "command not found," make sure your global npm bin directory is in your PATH.

## Step 2: Configure Your Keys

```bash
sf setup
```

The setup wizard walks you through each key:

```
$ sf setup

SimpleFunctions CLI Setup
─────────────────────────

? Kalshi API Key ID: ********-****-****-****-************
? Kalshi API Secret: ************************************
✓ Kalshi connection verified (account: your@email.com, balance: $500.00)

? OpenRouter API Key: sk-or-v1-************************************
✓ OpenRouter connection verified (model access: claude-3.5-haiku)

Configuration saved to ~/.sfrc
✓ Setup complete!
```

The CLI verifies each key works before saving. If Kalshi returns an auth error, double-check that you copied the full API secret (it's long).

**Where are keys stored?** In `~/.sfrc` on your machine. They never leave your computer. The CLI calls Kalshi and OpenRouter directly — there's no SimpleFunctions server in between.

## Step 3: Scan for Opportunities

Now the fun part. Let's search for recession-related contracts:

```bash
sf scan "recession 2026"
```

The scanner searches active markets on Kalshi, groups related contracts, and runs a quick analysis:

```
$ sf scan "recession 2026"

Scanning markets matching "recession 2026"...
Found 23 contracts across 4 events.

Analyzing for potential mispricing...

  #   Contract                    Exchange   Side    Price   Signal
  1   KXRECSSNBER-26              Kalshi     YES     35¢     Causal chain suggests 40-55% probability
  2   KXRECSSQ3-26                Kalshi     YES     28¢     Q3 timing may be underpriced given oil dynamics
  3   KXGDPNEG-26Q2               Kalshi     YES     22¢     GDP contraction linked to trade disruption
  4   KXUNEMPLOY5-26              Kalshi     YES     18¢     Unemployment lagging indicator, low confidence
  5   KXRECSSQ1-26                Kalshi     YES     12¢     Q1 almost expired, unlikely
  ...

Top candidate: KXRECSSNBER-26 (NBER-declared recession in 2026)
  Market says: 35% probability
  Quick analysis: Multiple causal pathways to recession (trade war, oil shock,
  credit tightening). Market may be underweighting compounding risks.
  Suggested next step: sf create "US enters recession by 2026"
```

**What you're seeing:**

- **23 contracts** across 4 events — Kalshi has multiple recession-related markets with different timing and definitions
- **The signal column** is a quick LLM assessment — not a trading recommendation, just a first-pass analysis
- **The top candidate** is the one with the most structural mispricing potential

**Decision point:** The scan says KXRECSSNBER-26 might be mispriced. Do you agree? Do you have a thesis about *why* recession probability should be higher than 35%? If yes, continue. If not, scan something else.

## Step 4: Create a Thesis

You've decided you think recession is underpriced. Now structure your reasoning:

```bash
sf create "US enters recession by 2026"
```

The CLI enters interactive mode and asks for your reasoning:

```
$ sf create "US enters recession by 2026"

Building thesis: "US enters recession by 2026"
───────────────────────────────────────────

? What's your reasoning? (Why do you think this is underpriced?)
> The Iran-US conflict is keeping the Strait of Hormuz blocked. Oil is
> above $100 and the SPR can't compensate. The Fed can't cut rates
> because inflation is supply-driven. Consumer spending will contract
> as energy costs eat into budgets.

Decomposing into causal tree...

Causal Tree: "US enters recession by 2026"
───────────────────────────────────────────

n1    Iran-US conflict persists                    88%
├── n1.1  Military strikes ongoing                  99%
├── n1.2  Iran maintains retaliation posture         85%
└── n1.3  No diplomatic resolution before Q3         82%

n2    Strait of Hormuz remains disrupted           97%
├── n2.1  Mines deployed (CENTCOM confirmed)         99%
└── n2.2  Minesweeping timeline > 3 months           87%

n3    Oil remains above $100/bbl                   64%
├── n3.1  SPR drawdown rate insufficient             95%
├── n3.2  OPEC+ doesn't increase production         68%
└── n3.3  No alternative routing at scale            71%

n4    Fed unable to ease monetary policy           58%
└── n4.1  Supply-driven inflation above 5%           72%

n5    Consumer spending contracts                  52%
n6    NBER declares recession                      45%

Thesis-implied probability: 45%
Current market price:       35¢
Preliminary edge:          +10¢

? Accept this tree? (Y/n) Y

Thesis "recession-2026" saved.
```

**What just happened:**

1. The LLM decomposed your reasoning into 12 testable sub-claims
2. Each sub-claim got a probability based on current evidence
3. The probabilities propagated through the tree to produce a thesis-implied probability of 45%
4. The market says 35%, so your preliminary edge is +10 cents

**Review the tree carefully.** Do you agree with the node probabilities? If n3.2 (OPEC doesn't compensate) feels too high at 68%, you can adjust:

```bash
sf adjust recession-2026 n3.2 --confidence 55
```

This will re-propagate and update your thesis-implied probability.

## Step 5: Check Your Edges

Now see where your thesis disagrees with the market across all related contracts:

```bash
sf edges
```

```
$ sf edges

Active thesis: recession-2026
Last updated: 2 minutes ago

Contract               Thesis   Market   Edge     Exec(100)   Liquidity
───────────────────────────────────────────────────────────────────────────
KXRECSSNBER-26 YES     45¢      35¢      +10¢    +8.7¢       HIGH
KXOIL100-26Q3 YES      64¢      41¢      +23¢    +21.2¢      HIGH
KXFEDCUT-26JUN NO      58¢      35¢      +23¢    +19.8¢      MEDIUM
KXIRANWAR-26 YES       88¢      84¢      +4¢     +2.1¢       LOW

Best risk-adjusted edge: KXRECSSNBER-26 (high liquidity, moderate edge)
Largest raw edge: KXOIL100-26Q3 (upstream node, fewer dependencies)
```

**Reading the output:**

- **Thesis**: What your causal model says the contract is worth
- **Market**: Current best ask on Kalshi
- **Edge**: Theoretical edge (thesis - market)
- **Exec(100)**: What your edge would actually be after slippage, buying 100 contracts
- **Liquidity**: HIGH/MEDIUM/LOW based on spread and depth

**Decision point:** KXOIL100-26Q3 has the largest edge (+23¢) and it's upstream in your causal chain (fewer dependencies = more reliable). KXRECSSNBER-26 has a smaller edge (+10¢) but it's the terminal contract — the one you actually care about. Which do you trade?

For your first trade, let's go with the recession contract. It's the one your thesis is directly about, the liquidity is HIGH, and the edge is meaningful.

## Step 6: Check Liquidity

Before placing any order, verify that the market can absorb your size:

```bash
sf liquidity --topic recession
```

```
$ sf liquidity --topic recession

KXRECSSNBER-26 (US Recession 2026 YES)
────────────────────────────────────────

  Spread:              1¢ (35¢ ask / 34¢ bid)
  Depth (ask, 5¢):     420 contracts
  Depth (bid, 5¢):     380 contracts
  Liquidity score:     HIGH

  Slippage estimates:
    50 contracts:      avg fill 35.4¢  (slippage: 0.4¢)
    100 contracts:     avg fill 36.5¢  (slippage: 1.5¢)
    200 contracts:     avg fill 37.8¢  (slippage: 2.8¢)
    500 contracts:     avg fill 39.2¢  (slippage: 4.2¢)

  Recommendation: Trade up to 200 contracts with acceptable slippage (<3¢)

KXRECSSQ3-26 (US Recession Q3 2026 YES)
────────────────────────────────────────

  Spread:              3¢ (28¢ ask / 25¢ bid)
  Depth (ask, 5¢):     180 contracts
  Depth (bid, 5¢):     120 contracts
  Liquidity score:     MEDIUM

  Recommendation: Trade up to 100 contracts. Wider spread eats more edge.
```

**What this tells you:** KXRECSSNBER-26 is liquid enough for a meaningful position. 100 contracts will cost you about 36.5¢ average (1.5¢ slippage), and your executable edge is still +8.5¢ after slippage. That's real edge.

## Step 7: Place Your First Order

You've done the analysis. The thesis says 45%. The market says 35%. The liquidity is there. Time to trade.

```bash
sf buy KXRECSSNBER-26 100 --price 32
```

**Why 32¢, not 35¢?** You're placing a limit order at 32¢, below the current ask of 35¢. This means your order sits in the book and waits for someone to sell to you at 32¢. You might not get filled immediately, but if you do, your edge is even better (45¢ thesis vs 32¢ fill = +13¢).

```
$ sf buy KXRECSSNBER-26 100 --price 32

Order Preview
──────────────────────────────

  Contract:    KXRECSSNBER-26 (US Recession 2026)
  Side:        YES
  Quantity:    100 contracts
  Limit price: 32¢
  Max cost:    $32.00
  Thesis edge: +13¢ (at limit price)

  ⚠  Limit price is 3¢ below best ask (35¢).
     Order will rest in the book until filled or cancelled.

? Confirm order? (Y/n) Y

Order placed: ORD-8f3a2b1c
  Status: RESTING
  Filled: 0/100

  Monitor: sf orders
  Cancel:  sf cancel ORD-8f3a2b1c
```

Alternatively, if you want to fill immediately at market:

```bash
sf buy KXRECSSNBER-26 100 --price 37
```

Setting the limit at 37¢ (above the best ask of 35¢) means your order will fill immediately, eating through the book up to 37¢ per contract. Based on the depth we checked, 100 contracts would fill at an average of ~36.5¢.

```
$ sf buy KXRECSSNBER-26 100 --price 37

Order Preview
──────────────────────────────

  Contract:    KXRECSSNBER-26 (US Recession 2026)
  Side:        YES
  Quantity:    100 contracts
  Limit price: 37¢
  Max cost:    $37.00
  Thesis edge: +8¢ (at limit price)

? Confirm order? (Y/n) Y

Order placed: ORD-9d4e5f2a
  Status: FILLED
  Filled: 100/100
  Average fill: 36.5¢
  Total cost: $36.50

  Position: 100 YES @ 36.5¢ avg
  If YES: profit = $63.50 (174% return)
  If NO:  loss = -$36.50 (100% loss)
  Thesis edge: +8.5¢
```

You now own 100 YES contracts on KXRECSSNBER-26 at an average cost of 36.5¢. If the NBER declares a recession in 2026, each contract pays $1.00 — you profit $63.50. If it doesn't, you lose your $36.50.

## Step 8: Monitor Your Position

Now set up monitoring so the system watches for you:

```bash
sf dashboard
```

```
$ sf dashboard

SimpleFunctions Dashboard
───────────────────────────────────────────────

Active Theses:
  recession-2026          45% confidence    Last eval: 12 min ago

Open Positions:
  KXRECSSNBER-26 YES      100 @ 36.5¢      Market: 35¢    P&L: -$1.50

Edge Monitor:
  KXRECSSNBER-26           Thesis: 45¢  Market: 35¢  Edge: +10¢    [HIGH liq]
  KXOIL100-26Q3            Thesis: 64¢  Market: 41¢  Edge: +23¢    [HIGH liq]

Recent Evaluations:
  03/19 14:30  n3.2 (OPEC) evaluated: 68% (unchanged)
  03/19 14:30  n4.1 (inflation) evaluated: 72% (unchanged)
  03/19 13:30  n1.3 (diplomacy) evaluated: 82% (unchanged)

Next evaluation: 14 minutes
```

The dashboard shows everything at a glance: your thesis confidence, open positions, current edges, and the most recent evaluations.

To start continuous background monitoring:

```bash
sf monitor --thesis recession-2026
```

This starts the heartbeat loop. Every 60 minutes (or more frequently during high volatility), the system scans for news, refreshes prices, re-evaluates your causal tree, and sends you a notification if anything meaningful changes.

## Step 9: Track Performance

After a few days or weeks, check how your trading is doing:

```bash
sf performance
```

```
$ sf performance

Performance Summary (last 30 days)
────────────────────────────────────────

  Account balance:     $536.50  (+$36.50 / +7.3%)
  Open positions:      1
  Closed positions:    0
  Unrealized P&L:      -$1.50

  Position detail:
    KXRECSSNBER-26 YES   100 @ 36.5¢   Market: 35¢   Unrealized: -$1.50

  Thesis accuracy:
    recession-2026       Active   45% confidence   Edge: +10¢
    Evaluations: 47      Signals: 2       Avg node delta: ±2.1%

  Monthly costs:
    LLM evaluations:    $4.80  (96 evals @ ~$0.05 avg)
    News searches:      $3.20  (320 searches @ ~$0.01)
    Total:              $8.00
```

Performance tracking shows you not just P&L but also how your thesis is behaving: how many evaluations have run, how many signals fired, how stable your node estimates are, and what the monitoring is costing you.

## What to Do Next

You've completed the full loop: scan → thesis → edges → liquidity → trade → monitor → performance.

Here's what comes next:

1. **Watch the evaluations.** Your thesis will update as news comes in. When a node changes significantly, you'll get a notification. Read the reasoning. Decide if you agree.

2. **Don't check prices compulsively.** The whole point of the monitoring system is to watch for you. If something changes, it'll tell you. Checking manually 10 times a day adds stress without adding information.

3. **Consider adding positions.** The oil contract (KXOIL100-26Q3) has a larger edge and is upstream in your causal chain. If you're confident in the thesis, diversifying across the causal chain reduces single-contract risk.

4. **Set exit conditions.** Run `sf strategy create --thesis recession-2026` to define mechanical exit rules. When should you sell? When your thesis confidence drops below 30%? When the edge narrows to less than 3¢? Define it now, not when you're panicking at 3am.

5. **Read the methodology.** Now that you've traded, the articles on [causal tree decomposition](/technicals/how-causal-tree-decomposition-works-prediction-market-trading) and [edge calculation](/technicals/edge-calculation-prediction-markets-theory-to-execution) will make much more sense. Theory is useful once you have practical context.

Your first trade is done. The system is watching. Now go do something else — the agent has it from here.