# Null as Signal

**Null as Signal is the framework reframe that treats a missing indicator value not as a data defect but as a positive entry condition for a specific strategy. When LAS is null, when EE is null, when PIV is near zero, when CVR is null — each null state corresponds to a maker or first-mover strategy that other traders have not noticed because they are filtering the nulls out. The four named null patterns are the entry conditions for the four maker strategies in the SimpleFunctions playbook.**


## Explanation

## The Reframe

Most prediction-market dashboards (and most new traders) treat a null value as missing data. The default UX response is to filter the nulls out of the scan and only show the markets where the indicator was successfully computed. That is the wrong default for a maker.

Null does not mean "the system failed to compute LAS." Null means "no one has been looking at this market." The cron job that computes LAS runs only on the top 500 markets by 24-hour volume — so a null LAS is a market that is not in the top 500, which means it is not on anyone's radar. *That is the entry condition for being the first quote on the book.*

The same reframe applies to every other Tier B indicator. EE = null means the contract has no detected siblings, which means the thesis is uncontaminated. PIV ≈ 0 means the contract has had fewer than 2 1¢-delta events in the last week, which means it is in a stable range that a market maker can quote against. CVR = null means the contract is not part of a coordinated family, which means there is no maker holding the cross-prices together.

Each null state is an *entry condition*, not an *exclusion*.

## The Four Named Patterns

The four nulls map to four strategies in the SimpleFunctions maker playbook:

**1. LAS = null → virgin Polymarket strategy.** The market has no orderbook history and no recent volume. A maker can post quotes 8-12 cents wide, sit on the book as the only resting orders, and earn the spread on whatever taker eventually wanders in. Sizing is small per market because most virgins stay virgin, but the cumulative coverage across 30,000+ null-LAS markets is meaningful.

**2. EE = null → uncontaminated thesis strategy.** The contract has no detected sibling outcomes, which means the thesis is unmediated by sibling overround. A trader can take a position based purely on the contract's own merits without disentangling spillover from neighboring markets. This is the cleanest case for a directional bet on a single isolated event.

**3. PIV ≈ 0 → range maker strategy.** The contract is in a stable price range with very few 1¢-delta events. A maker can post a quote on both sides of the current mid, expecting that any move will reverse before the wider range is exited. The strategy works because nobody is currently chewing on the contract — the lack of activity is itself the edge.

**4. CVR = null → first-mover strategy.** The contract has no detected sibling family, so a thesis that develops here will not have already been priced into related markets. A trader can take a position before the contagion wave that does not exist yet — and if a sibling family does emerge later, the early position is the seed.

## Why It Works as a Framing

The framing inverts the default "maximize the number" instinct. You are not trying to find the highest LAS or the highest CRI. You are trying to find the *combination* of indicator values that matches the strategy you intend to run. For maker strategies, the combination usually includes one or more nulls. For taker strategies, the combination requires no nulls (because takers need executable edge, which requires both sides of the book to exist).

The same dataset feeds both kinds of trader. The difference is whether you filter the nulls out (taker) or filter the nulls in (maker). Most software in the prediction-market space does the first by default and never offers the second, which is why the maker side is undercrowded.


## Example

A scan that intentionally surfaces null values, run on a Wednesday morning:

| Filter | Markets returned | Strategy |
|---|:---:|---|
| LAS IS NULL AND τ between 30 and 200 days | 28,400 | virgin Polymarket maker |
| EE IS NULL AND IY > 200% | 1,940 | uncontaminated thesis |
| PIV ≤ 2 (7d) AND τ > 60 days | 8,700 | range maker |
| CVR IS NULL AND CRI > 0.5 | 6,200 | first-mover |

Each row is a market list that no taker-oriented dashboard would surface, because every dashboard filters nulls out by default. The 28,400 virgin markets have no displayed liquidity, so they look "broken" to a taker — but they are exactly the universe a maker wants to put first quotes on. The 1,940 uncontaminated thesis candidates have no sibling overround signal, so they look "missing data" to a taker — but they are the cleanest possible directional bets.

The composite read: the same 47,000-market universe contains both a taker game and a maker game, and which game you are playing determines which markets are interesting. For a taker, nulls are noise. For a maker, nulls are the signal.

The CLI surfaces this with the inverted-filter flags: `sf scan --las-null --tau-min 30 --tau-max 200` returns the virgin-Polymarket universe directly. The default `sf scan` continues to filter nulls out, because the taker game is the more common starting point. Both games are first-class.


## CLI

```bash
sf scan --las-null --tau-min 30 --tau-max 200
```


## Related

[liquidity-availability-score](liquidity-availability-score.md), [position-implied-velocity](position-implied-velocity.md), [contagion-velocity-rate](contagion-velocity-rate.md), [event-overround](event-overround.md), [cycle-clustering](cycle-clustering.md), [valuation-funnel](valuation-funnel.md), [market-maker](market-maker.md), [thesis](thesis.md)
