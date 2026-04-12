# Cycle Clustering (CYC)

**Cycle Clustering is the process of grouping prediction-market contracts that belong to the same recurring event family using a fixed set of nine slug regex patterns. CYC turns the universe of 47,000 isolated contracts into ~2,500 connected event families, which is the prerequisite for computing yield curves, cross-sibling overround, and contagion velocity. Roughly 41.4% of markets get assigned to a family; the remaining 58.6% are events that do not fit any of the nine patterns and are handled separately.**


## Explanation

## Why Regex and Not LLM

The first instinct on a "group similar contracts" problem is to throw an embedding model at it and let semantic similarity do the work. That fails for prediction markets in a way that is worth understanding.

Prediction-market slugs encode their *family membership* in the slug itself, not in the prose description. `KXFEDDECISION-26MAY01` and `KXFEDDECISION-26JUN12` are the same family; the differences are the date suffixes. An embedding of the prose description ("Will the Fed cut rates at the May 2026 meeting?" vs "...June 2026 meeting?") gets you most of the way there, but it is non-deterministic, expensive on a 47K-market universe, and impossible to debug when it groups two contracts that look similar but are actually unrelated.

A regex grouper is the opposite. It looks at the slug, finds the date or sequence suffix, and produces a deterministic family ID by stripping the suffix. The whole module is in `src/lib/indicators/cyc-grouper.ts` and runs in milliseconds across the whole universe.

## The Nine Patterns

The patterns the grouper currently catches:

```
1. by-monthDay-year     KXFEDDECISION-26MAY01    →  KXFEDDECISION
2. iso-date             event-2026-05-01         →  event
3. quarter-year         spacex-ipo-q2-2026       →  spacex-ipo
4. year-only            recession-2026           →  recession
5. before-month         oil-50-by-may-2026       →  oil-50-by
6. by-end-of-year       supreme-court-by-eoy-26  →  supreme-court-by-eoy
7. month-year           ecb-rate-may-2026        →  ecb-rate
8. range-window         btc-100k-by-jun-2026     →  btc-100k-by
9. before-year          before-2027              →  before
```

These are not exhaustive. They catch the high-volume event families on Kalshi and Polymarket — Fed meetings, election cycles, IPO announcements, rate decisions, calendar-anchored "by year-end" events. They miss anything that does not have a date or sequence in the slug, which is most of the winner-pick events, unusual one-off events, and anything where the venue chose a slug format that does not encode the event marker in the suffix.

## The 41.4% Coverage Number

When the grouper runs against the full 47K-market universe, it assigns about 41.4% of contracts to a family ID. The remaining 58.6% are "uncatchable" — meaning the slug does not match any of the nine patterns and the contract is treated as its own family of one.

A family of one is not useful for yield curves or contagion analysis (you need at least two siblings for either). So the practical coverage is even lower than 41.4%: only the families with two or more members are usable, and those reduce to roughly 2,500 event families across the universe.

The 58.6% not covered is not a defect to fix. Most of those contracts genuinely have no siblings — a market about a specific person's specific action does not belong to a family because there is no other contract about the same kind of thing. The grouper is honest about this rather than hallucinating false relationships.


## Example

Five real-form Polymarket slugs run through the grouper:

| Slug | Pattern | Family ID |
|---|---|---|
| `will-fed-cut-rates-may-2026` | month-year | `will-fed-cut-rates` |
| `will-fed-cut-rates-june-2026` | month-year | `will-fed-cut-rates` |
| `will-fed-cut-rates-july-2026` | month-year | `will-fed-cut-rates` |
| `spacex-ipo-by-end-of-2026` | by-end-of-year | `spacex-ipo-by-end-of` |
| `will-trump-meet-with-xi-jinping` | none | (singleton) |

The first three contracts collapse into one family ID: `will-fed-cut-rates`. That family now has three members (May, June, July 2026) and can be plotted as a yield curve. The IY at each τ becomes a point on the curve, and the shape of the curve tells you which Fed meetings the market is treating as more uncertain than others.

The fourth contract has its own family of one — there is no other "by end of 2026" contract about SpaceX in the universe right now, so the family ID is unique. It is a singleton until another contract with the same family marker appears.

The fifth contract does not match any of the nine patterns. It becomes a singleton family of itself, and it does not contribute to any yield curve or sibling overround calculation. That is the correct behavior: there is no sibling to pair it with.

The composite output across the full universe is roughly 2,500 multi-member families and ~28,000 singletons. The 2,500 families are where the term-structure work happens. The singletons are the "isolated thesis" universe where contagion is null and yield curves do not apply.


## CLI

```bash
sf cycle list --min-members 2
```


## Related

[contagion-velocity-rate](contagion-velocity-rate.md), [yield-curve-prediction-markets](yield-curve-prediction-markets.md), [event-overround](event-overround.md), [event-contract](event-contract.md), [cross-venue](cross-venue.md), [null-as-signal](null-as-signal.md), [expected-edge](expected-edge.md), [thesis](thesis.md)
