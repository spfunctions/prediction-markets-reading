# The Longshot Bias in Modern Prediction Markets — 80 Years of Evidence in One Number

> Pari-mutuel horse racing has had a documented longshot bias since the 1940s. Polymarket and Kalshi have a measurable version too, and the direction is not the same. What the calibration data actually says about cheap contracts.

**Category:** methodology | **Author:** Patrick Liu | **Reading time:** 10 min

---
In 1949, Richard Griffith published an analysis of horse race odds at American tracks and noticed something that has been replicated many times since: contracts at the long end of the odds distribution — the ones priced as longshots — paid out *less* often than their implied probability suggested they should. A horse priced at 50-1, implying a 1.96% chance of winning, would actually win something closer to 1.4% of the time. The favorite-priced horses were the opposite: a horse priced at 2-1, implying a 33% chance, would win closer to 38% of the time. The market was systematically overpaying favorites and underpaying longshots.

This is the "favorite-longshot bias" and it has been one of the most replicated empirical findings in sports betting research. Eighty years of data, multiple countries, different parimutuel systems — the bias holds. Bettors collectively are willing to pay more for the dream of a 50-1 score than the 50-1 odds justify, and they are correspondingly underpaying the boring favorites where the actual edge sits.

So the question for any modern prediction-market trader is: does the bias show up on Polymarket and Kalshi? And if it does, in what direction? The answer is more interesting than the obvious "yes, of course." The data on `/api/calibration` tells a story that does not look exactly like the horse-track story, and reading it carefully is the first step toward not getting picked off by the same bias in reverse.

## The Numbers from /api/calibration

I pulled the calibration data from the SimpleFunctions calibration endpoint, which buckets all resolved binary contracts by the YES price they were trading at and reports the actual fraction that resolved YES in each bucket. The data covers Kalshi and Polymarket combined, several years of resolved markets, several thousand contracts per bucket. The most interesting buckets are the extreme ones — the 0-10¢ bucket and the 90-100¢ bucket — because those are where the longshot vs favorite bias shows up most cleanly.

Here is the picture for the 0-10¢ bucket. Markets trading between 0 and 10 cents on the YES side, at the time the data was collected, resolved YES 11.46% of the time. The midpoint of the bucket is 5 cents, implying a "fair" resolution rate of 5%. The actual rate is more than double that. **Cheap contracts pay out more often than their price says they should**, by a meaningful margin.

Compare to horse racing's bias direction. At the track, a 50-1 longshot (2% implied) resolves at something like 1.4% — the bias is in the direction of *less* payout than implied. At Kalshi/Polymarket, the bias on cheap contracts goes the opposite way: more payout than implied, not less. The longshots pay better than they should.

The 90-100¢ bucket is the mirror. Markets trading between 90 and 100 cents on the YES side resolve YES around 88-89% of the time, against an implied rate of 95%. **Expensive contracts pay out *less* often than their price says they should**, again by a meaningful margin. The "favorite" contracts are overpriced.

So the modern prediction-market bias is *opposite in direction* from horse racing. Longshots are slightly underpriced; favorites are slightly overpriced. The pattern is real and it is consistent across years of data.

## Why the Direction Flipped

This is the part of the article that requires speculation, because nobody has run a full social-science study on it. But the pattern is suggestive enough to propose a mechanism.

The horse-track longshot bias is driven by retail bettors. The track audience pays for the lottery-ticket dream, and the parimutuel pool reflects that demand: longshots get more bets than their odds justify, the implied probability rises above the true probability, and the realized payout is below the implied. Retail demand for narrative payoffs distorts the prices.

Modern prediction markets have a different participant mix. Kalshi and Polymarket are not anonymous retail-only venues. They have a meaningful population of sophisticated traders running scans, providing liquidity, building causal models. The cheap-contract end of the price distribution is *exactly* where these traders look first, because cheap contracts have the highest implied yields and the most asymmetric payoffs. When a 5¢ contract starts to show information that suggests it should be at 8¢, the sharp money moves it before retail notices. The result is that cheap contracts spend more time correctly priced — and the bias, instead of tilting against the longshots, ends up tilting *for* them. Retail piles into the favorite end (where the price is "safer") and the favorite end ends up overpriced.

There is a second mechanism worth naming. Many cheap contracts are cheap because *the market thinks the event is unlikely*, not because anyone has done the work. A cheap Polymarket contract on "Will country X have a coup in the next 90 days" is priced low because coups are rare, but the *conditional* probability given specific signals (military movements, currency dislocations, embassy advisories) can spike well above the price. The traders who track those signals can buy the cheap contract at 4¢ when its conditional probability has moved to 12¢. They are the ones cashing in the longshot premium that the horse-track bettors were paying away.

The combined effect is that the modern prediction-market longshot bias exists, but it goes the *other* direction. Longshots are slightly underpriced; favorites are slightly overpriced. The directionality is the load-bearing fact: if you imported the horse-track intuition straight, you would do exactly the wrong thing. You would avoid cheap contracts (because "the bias says they pay less") when the data says they pay more.

## The Honest Tail Caveat

I want to flag something important about the calibration data. The 0-10¢ and 90-100¢ buckets are the ones I trust most, because they are large samples and the bias signal is unambiguous. The middle buckets — say 30-40¢ and 60-70¢ — show calibration patterns that are noisier and that I think reflect a model failure on the platform side as much as a market bias. The honest read is to focus on the tails and not over-interpret the middle.

This is partly why I am using the 11.46% number and not a more precise version. The signal in the tails survives reasonable changes to model assumptions; the signal in the middle does not. If you want to read the calibration data yourself, the [/calibration](/calibration) page on the site visualizes the byPriceBucket breakdown, and you can see the noise in the middle buckets directly.

## Three Trading Implications

**1. Be skeptical of buying contracts priced under 5¢ unless you have specific information.** The longshot bias works in your favor on average, but "on average" hides the specific failure mode: most cheap contracts resolve NO, and the average premium is small (2-3% better than implied). If you do not have a thesis, buying a basket of 5¢ contracts is a slow loss, not a positive-EV strategy. The bias is real but it is a thin edge that a thesis-aware approach captures and a blind buyer does not.

**2. Be skeptical of buying contracts priced over 90¢ even if "the contract should obviously resolve YES."** This is the symmetric implication and it is the one I had to learn the hard way. A contract at 96¢ feels like free money because the implied YES probability is 96% and the contract "obviously" should resolve. The data says contracts in this bucket resolve YES around 88-89%, which means there is roughly a 1-in-11 chance you are wrong. At 96¢ you get 4¢ if you win, lose 96¢ if you lose. Expected value is approximately (0.88)(4) − (0.12)(96) = 3.5 − 11.5 = −8 cents per contract on average. The favorites are *paying you to lose*. Avoid the 90+¢ bucket entirely unless your specific information is much stronger than the calibration data.

**3. The longshot bucket is the right place to put thesis-driven bets.** The combination of "the bucket is structurally underpriced on average" and "you can layer on specific information" is the most asymmetric setup in the market. A cheap contract where you have a specific reason to believe the conditional probability is twice the displayed probability is a 3-5x asymmetric bet, and the structural bias is on your side rather than against you. This is where I size most of my one-off conviction trades, not where I size my routine yield trades.

## Where This Reads Wrong

The longshot bias literature has some specific failure modes worth importing.

**Selection effects in the calibration data.** The contracts that *resolve* are not a random sample of all contracts. Some markets get cancelled, some get disputed, some get resolved by UMA in a way that creates ambiguity about whether the original price was "right." The calibration data on `/api/calibration` is computed only from cleanly resolved markets, but cleanliness itself is a filter, and the filter probably correlates with the kind of contracts that have crisp longshot pricing in the first place.

**Category-specific bias.** The aggregate calibration number masks big differences between categories. Sports binaries have one bias profile, election binaries have another, weather binaries are nearly perfectly calibrated (because the underlying physical process is well-modeled). The 11.46% number on the 0-10¢ bucket is an average across all categories — your specific category might be 6% or 18%. Always check the byCategory breakdown before betting on the bias.

**Sample size at the absolute extremes.** The 0-1¢ bucket and the 99-100¢ bucket are very thin samples. The calibration in those tiny extremes is not statistically meaningful. The "longshot bucket" I am referring to is really the 1-10¢ band, not the 0-1¢ band. Be careful with the very tip.

A fourth caveat: the bias is *small*. We are talking about a few percentage points of difference between implied and realized rates, not a 50% mispricing. Building a strategy that depends on systematically harvesting the longshot bias requires very low frictional costs and a large number of bets to overcome variance. The bias is enough to inform sizing decisions and to override naive intuition, but it is not large enough to be a standalone strategy.

## The sf CLI Read

The [implied yield](/learn/implied-yield) framing makes the longshot bias actionable. `sf scan --by-iy desc --max-price 0.10 --min-tau 14` returns the cheap-contract universe sorted by IY descending, with the τ filter removing the explosive near-expiry contracts. From there you are looking for contracts where you can layer on causal reasoning about why the cheap price is wrong. Without the causal layer, the bias alone is not enough to act on. With it, the cheap end of the price distribution is the most asymmetric place on the platform.

For the broader phenomenon catalog, see [reflexivity-loops](/concepts/reflexivity-loops) for the related question of how price itself becomes information, and [information-latency](/concepts/information-latency) for how fast new information actually reaches the price. The longshot bias is partially a story about *why* cheap contracts are slow to incorporate new information.

The horse-track longshot bias has been studied for eighty years. The modern prediction-market version is younger — maybe a decade of clean data — but the empirical pattern is real, the direction is opposite, and the mechanism is plausibly a story about who the marginal trader is. Read the calibration page yourself, look at the tail buckets, draw your own conclusions. The data is the part of the argument that does not depend on me.