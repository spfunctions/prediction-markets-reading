# Adverse Selection on Prediction Markets: Why Your Counterparty Knows Something You Don't

> The fundamental market-maker problem in one sentence: the people who hit your bid know more than you. PMs have specific adverse-selection patterns by category, and the right defense is wider spreads, smaller sizes, or staying out entirely.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 11 min

---
When I post a maker quote on a Kalshi binary, the next person to hit it is doing so for a reason. Sometimes the reason is "I want exposure to this outcome and your price is fine." Most of the time the reason is something I do not yet know. That asymmetry is adverse selection, and it is the central problem of every market-maker who has ever existed.

The Kalshi or Polymarket version of the problem is not different in kind from the equity-options version, but it has its own texture, its own category-specific patterns, and its own defenses. This page is my working notes on each.

## The General Problem, Stated Cleanly

A market-maker quotes a two-sided price. A taker arrives and crosses one side of that price. From the maker's perspective, the question is: does the taker have information I do not have, or are they trading for some non-informational reason (rebalancing, hedging, retail noise)?

If the taker is uninformed, the maker collects the spread on average and the position mean-reverts. If the taker is informed, the maker collects the spread once and then takes a loss when the price moves against them as the information propagates. The maker's expected profit is roughly:

```
E[profit] = (1 − p_informed) × spread − p_informed × E[adverse_move]
```

Where p_informed is the probability the next taker is informed. The maker's defense is to widen the spread until E[profit] is positive at a defensible p_informed. The taker's strategy is to wait until the spread is tight enough that E[adverse_move] dominates, then strike.

The whole job of being a market-maker is *estimating p_informed* on each contract you camp, and adjusting your quotes until the math works. On prediction markets, the patterns of p_informed are remarkably category-specific.

## Adverse Selection by Category

**Sports.** The most adversely-selected category by far. Sharp sports bettors live and die by closing-line value, and when they identify a mispricing on a Kalshi or Polymarket sports contract, they have hours to grind it out before the game starts. The adverse selection is *concentrated in the last hour before tip-off*, when sharps are placing their final positions and any maker still on the book is donating to them. The defense: do not maker on sports contracts in the final two hours unless you also have an information edge.

**Election (US federal).** Heavy adverse selection, but the timing is different. Information leaks 24-48 hours before the resolution event — internal poll numbers, leaked Hill aide chatter, local results from early-counting precincts. The makers who get hurt are the ones still quoting tight in the run-up window. The defense: widen the spread by 2-4¢ in the 48 hours before resolution, or step back from the book entirely. The maker can come back in after the resolution event itself, when the post-event price is being sorted out by retail.

**Economic indicator releases (CPI, NFP, FOMC).** Adverse selection is sharp but lives in a tiny window — the 30 to 60 seconds *after* the release, when sophisticated traders have parsed the print and the makers have not. The Kalshi-side institutional MMs handle this by withdrawing their quotes ten seconds before the print and not coming back for two minutes. If you are quoting on these contracts and you have not built that withdrawal logic, you are paying for it. The defense: hard-coded API logic that pulls quotes around scheduled release times.

**Crypto daily (KXBTCD-style).** Adverse selection from crypto trading desks that are watching the spot price tick by tick. The desks know whether the contract is going to pin to YES or NO ten minutes before the makers do, because they are watching the underlying. The defense is the same as the economic indicator case — widen or step back when the underlying is approaching the strike, especially in the last hour of the daily window.

**Weather contracts.** *Almost no adverse selection*, because weather data is fully public. The same NWS forecast everyone sees is the same forecast the maker sees. Weather is the closest thing to a "fair" prediction market, and if you want to learn how to make markets on PMs without getting hurt, weather contracts are the best training ground. The defense is unnecessary; the spread is the genuine compensation for inventory risk and venue fees, not for adverse selection.

**Geopolitical / event contracts (war, election, regime change).** Adverse selection from people inside the affected country or with sources that ordinary news consumers do not have. This is the category I am most cautious about, because the information asymmetry can be enormous and you have no way to estimate p_informed in advance. The defense: do not make markets on geopolitical contracts unless you also have a defensible source of information yourself, or unless the spread is wide enough to absorb a 10-15¢ adverse move.

**SpaceX / private company markets.** Adverse selection from people who actually know the company. If KXIPOSPACEX-26MAY01 is the contract and someone at SpaceX leaks an internal slide deck, the leak hits a few accounts on Polymarket hours before it hits Twitter. The defense: stay out of single-company prediction markets unless the spread is at least 5¢ and you are not camping for more than a few hours at a time.

The asymmetry across categories is large enough that I think of "making markets on prediction markets" as five or six separate sub-strategies, one per category, with completely different parameters for each.

## How to Detect Adverse Selection in Real Time

You cannot directly observe p_informed. You can observe its effects.

The clearest signal is *trade tape asymmetry*. If your bid is being hit much more often than your ask (or vice versa) on a contract that should be roughly balanced, you are being picked off. The directional flow is the information. The fix is either to widen the side that is being hit, or to invert your quote so the side being hit becomes the side you *want* to be filled on.

A second signal is *quote refresh latency from the institutional makers*. If the Susquehanna-style camped maker on a Kalshi contract suddenly stops refreshing their quote, that is not because they are taking a coffee break. That is because they have a model that says the contract is about to be repriced. When the institutional maker steps back, the regime has flipped to taker-dominated and adverse selection is likely. The retail maker (you) should also step back.

A third signal is *the news API*. If a wire crosses about the underlying event in the τ window, treat it as an adverse-selection event for the next ten to thirty minutes regardless of which direction the news cuts. The information takes time to propagate, and during that propagation window the takers know more than the makers do. This is mechanical and can be automated — feed any news API into your maker logic and have it auto-widen or pull quotes when it fires.

You can also use the [contagion velocity rate](/learn/contagion-velocity-rate) as a slower-frequency proxy. CVR rising means the thesis is propagating across markets, which means information is flowing, which means anyone making markets in the affected family is at risk of being adversely selected. CVR is not real-time, but it is a useful early-warning signal at the watchlist level.

## The Defense: Three Tools

There are three defenses against adverse selection, and you should be applying all three depending on the contract.

**Widen the spread.** The simplest defense. If p_informed is high, widen until the math works. The cost is fewer fills (because the takers cross your wider spread less often), but the surviving fills are profitable on average. I run wider quotes on sports, geopolitical, and single-company contracts than I do on weather or scheduled-release contracts, and the difference is sometimes 4-5¢.

**Reduce the size.** Even at the same spread, smaller positions cap the damage of any single adverse fill. I size makers wider on contracts I cannot hedge — that means both wider in spread *and* smaller in notional. The two adjustments work together.

**Stay out.** The honest answer for some contracts is that the adverse selection is too concentrated to defend against. Single-stock binary contracts on companies you have no edge on are in this bucket. Geopolitical contracts with low news flow but real insider risk are in this bucket. Scheduled-release contracts where you cannot automate the quote-pulling logic are in this bucket. Recognize when a contract is outside your defensive envelope and skip it.

The error mode I see most often in new makers is *applying one defense to all contracts*. Either they widen everything (and miss the safe contracts where the spread is real compensation), or they tighten everything (and get killed on the adversely-selected ones). The right move is per-contract calibration, which is exactly what the category-by-category section above is for.

## A Worked Example

KXNFP-26MAY02 (May NFP report YES), Friday morning, 30 minutes before the 8:30 AM release. The bid-ask is 47¢ / 51¢ with $200 on each side. I am tempted to post a tighter inside quote at 48¢ / 50¢ to pick up the spread.

The right move is the opposite. The next ten minutes are exactly when the information arrives. Sophisticated rates traders are running their model on the consensus print and updating their priors. If I quote tighter, I am inviting them to hit my bid (or ask) with the model output, two minutes before the release that confirms it. The defense: widen, not tighten. I either pull my quote entirely or post way outside the displayed spread at 44¢ / 54¢, hoping for an uninformed trade and accepting that I will probably get no fills.

After the release, I come back at 8:32 AM. The market has repriced to wherever the print actually was. I now post a normal quote in the new neighborhood and collect spread on the post-release retail flow that is still trying to figure out what happened. The first ten minutes after the release are the *good* time to be a maker on these contracts; the last ten minutes before are the bad time.

That asymmetry — bad before, good after — is the entire structure of adverse selection on scheduled-release contracts. Once you see it, you stop trying to capture the pre-release spread and start sitting out for the two minutes that matter.

## Where This Frame Breaks

Two failure modes worth naming.

**The "informed" taker is wrong.** Sometimes the trader hitting your bid has a thesis they are very confident about, and the thesis is wrong. The contract resolves your way. From the outside it looks like you outsmarted them, but you did not — you got lucky. Do not learn the wrong lesson from this. The expected-value math is what matters, not the realized outcome on any one fill. If you are systematically losing on the contracts where the takers are informed *and the contracts where the takers are wrong*, you are misclassifying p_informed. Recalibrate.

**The category labels are too coarse.** Sports is a vast category and the adverse selection on a Tuesday-night NBA contract is different from the adverse selection on a Sunday-afternoon NFL contract. The same is true for crypto (BTC daily vs ETH weekly vs altcoin) and for election (federal vs state vs primary). The category-by-category guidance above is a starting point, not a finished classification. As you trade more, build sub-categories with sub-parameters.

A third issue: sometimes the *venue itself* is the adverse-selecting party. On Polymarket, the AMM curve is structured so that large taker orders move the price more than they would on a true CLOB. The "taker" you are quoting against is sometimes the curve itself, and the curve has no information — it just has an inventory function. This is not classical adverse selection but it has the same effect on your P&L. The defense is the same: widen on Polymarket relative to Kalshi for the same kind of contract.

## Practical Heuristics I Use Daily

I size makers wider on contracts I cannot hedge. That is the single most important rule and it covers 80% of the work. Hedgeable contracts (BTC daily, FX, rates) get tighter quotes because I can offset the adverse direction. Unhedgeable contracts (election, geopolitical, single-company) get wider quotes because the adverse fill is locked in.

I never make markets on a contract I have not classified by category in advance. The category is the prior on p_informed; without the prior, the spread I post is arbitrary.

I run `sf scan --by-news-flow desc` once an hour to find contracts where the news velocity has spiked, and I auto-widen my standing quotes on any of them by 2¢ until the velocity decays. The news API is the cheapest leading indicator of adverse selection that I have found.

I have a hard rule: never make markets in the 30 minutes before a scheduled release. Not even on contracts I think I have an edge on. The information asymmetry in that window is too sharp to defend against, even with wide quotes.

The longer I do this, the more I think of adverse selection as the *defining* problem of prediction-market making. Indicator-stack alpha is real but small. Liquidity-availability alpha is real but capped by depth. Adverse-selection avoidance is the difference between a maker who survives and a maker who funds the takers' P&L for six months and then quits.

For the companion question — *why* the regime flips when adverse selection arrives — see [maker-taker-regime-in-pms](/concepts/maker-taker-regime-in-pms). For the post-release dynamic, see [pin-risk-binary-settlements](/concepts/pin-risk-binary-settlements).