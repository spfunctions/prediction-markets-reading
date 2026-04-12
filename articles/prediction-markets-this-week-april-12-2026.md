# Prediction Markets This Week: April 12, 2026

**Category:** markets | **Author:** Patrick Liu | **Reading time:** 5 min | **Published:** 2026-04-12

---
A weekly snapshot of the most active prediction markets across Kalshi and Polymarket, using live data from the SimpleFunctions indicator stack. Updated every Saturday.

## Biggest Movers (non-sports)

Data from [/api/public/screen?sort=volume](/api/public/screen?sort=volume&limit=20) — top markets by 24-hour volume, excluding sports and entertainment.

| Market | Venue | Price | Volume | IY |
|--------|-------|-------|--------|---------|
| California Governor: Mahajan | Kalshi | 14¢ | $254K | 393% |
| Hungary PM: Orbán | Polymarket | 27¢ | $1.9M | — |
| Hungary PM: Magyar | Polymarket | 74¢ | $1.6M | — |
| Bitcoin >$71,700 today | Kalshi | 64¢ | $164K | — |
| Iran conflict ends April 7 | Polymarket | 47¢ | $1.1M | — |
| Hormuz normal by end of April | Polymarket | 17¢ | $865K | — |
| California Governor: Steyer | Kalshi | 51¢ | $137K | 61% |
| Iranian regime falls by April 30 | Polymarket | 3¢ | $1.0M | — |

## Key Themes This Week

**Hungary goes to the polls.** The largest prediction market political event since the 2024 US election. $8M+ combined volume. TISZA at 77¢ to win parliament. Full analysis: [Hungary Election 2026](/blog/hungary-election-2026-prediction-markets-orban-magyar-tisza-fidesz).

**Hormuz shipping still disrupted.** 17¢ chance of normalization by end-April. Term structure slopes from 14¢ (Kalshi May 1) to 40¢ (June 1). Oil markets have not fully repriced. Full analysis: [Strait of Hormuz Crisis](/blog/strait-of-hormuz-prediction-markets-iran-oil-shipping-live-data).

**California governor market heats up.** $780K daily volume across 14 contracts, 19 months before the election. Primary-general spread on Shiller (76¢ primary, 6¢ general) is the structural trade. Full analysis: [California Governor 2026](/blog/california-governor-2026-prediction-market-odds-steyer-mahajan-porter).

**Crypto daily markets remain active.** Bitcoin daily contracts ($164K on the $71,700 strike alone) continue to be the most liquid non-political Kalshi category.

## Screener Highlights

**Highest Implied Yield (τ > 7 days):**
Run: [sf screen --iy-min 500 --tau-min 7 --limit 10](/api/public/screen?iy_min=500&tau_min_days=7&sort=iy&limit=10)

**Most Liquid (LAS < 0.05):**
Run: [sf screen --has-orderbook --las-max 0.05 --sort las](/api/public/screen?has_orderbook=true&las_max=0.05&sort=las&limit=10)

**Unloved Markets (no thesis, Polymarket, τ > 30):**
Run: [sf screen --without-thesis --venue polymarket --tau-min 30](/api/public/screen?no_thesis=true&venue=polymarket&tau_min_days=30&sort=iy&limit=10)

## How to Use This

Every link in this post hits a live API endpoint. The data updates every 15 minutes. Bookmark the [screener](/api/public/screen?sort=volume) or install the CLI:

```bash
npm install -g @spfunctions/cli
sf screen --sort volume --limit 20
```

Previous weeks: [What Is Going On in the World — April 2026](/blog/what-is-going-on-in-the-world-right-now-april-2026)

*Next update: April 19, 2026.*