# Prediction Markets Need Fixed-Income Language

> Yield, spread, duration, convexity. The vocabulary of bond desks already exists, and prediction markets are mathematically the same instrument. Here is the dictionary that bridges them.

**Category:** insights | **Author:** Patrick Liu | **Published:** 2026-04-08

---
A binary prediction-market contract is, mathematically, a zero-coupon bond with a binary recovery rate. You buy it for some fraction of par. You hold it for a known duration. At maturity it pays par or it pays zero, conditional on a single observable event.

Bond traders have priced this exact instrument for fifty years. They call it a credit-risky zero. They have a vocabulary for it that handles every edge case anyone has ever thought of. And almost no prediction-market trader uses any of that vocabulary.

This essay is about why that is wrong, and what to import.

## The Vocabulary Mismatch

When a bond trader looks at a credit-risky zero, they ask three questions in sequence:

1. **What is the yield?**
2. **What is the spread?**
3. **What is the duration?**

When a prediction-market trader looks at the same instrument (a binary contract), they ask:

1. **What is the price in cents?**

That is the whole vocabulary. There is no second question. There is certainly no concept of duration. The market has been built by people who came from sports betting and crypto, neither of which has a developed yield-curve language, and so the entire framing has stayed flat — every contract is a price between $0.00 and $1.00 and you either think it should be higher or lower.

This is technically correct and analytically anemic. It is the difference between knowing the temperature outside and knowing the climate.

## What to Import, Term by Term

**Yield → Implied Yield (IY).** A contract trading at $0.40 with 60 days to expiry pays an annualized return of (1/0.40)^(365/60) − 1 ≈ 4710% if held to resolution. That is a yield, expressed in the same units as a treasury bill or a money-market fund. The number is enormous because binary contracts are inherently leveraged at short horizons, and that *is* the right thing to communicate. IY puts the leverage in the foreground where it belongs.

**Spread → Bid-Ask Spread, then Cross-Venue Spread.** Inside a single market, the bid-ask spread is the same concept it is in any orderbook. The interesting borrowing is *cross-venue* spreads: the same outcome trades at $0.42 on Kalshi and $0.39 on Polymarket, and the 3-cent difference is a spread you can either harvest as arbitrage or use as a measurement of liquidity asymmetry. Bond traders handle this kind of spread between bonds with the same maturity but different issuers; the math is identical.

**Duration → τ-Days.** Bonds have duration; prediction markets have τ-days, which is just "calendar days until resolution." The simpler version of duration in the prediction-market context is unweighted, because the payoff happens in one lump at maturity (you do not get coupons in the middle). But the role of duration — quantifying how exposed the position is to price moves over time — translates directly. A position with high τ is a long-duration position. It is more sensitive to news that arrives between now and resolution.

**Convexity → CRI (Cliff Risk Index).** Bonds have convexity, which captures how price changes accelerate as yields move. Prediction-market contracts have something analogous: the rate at which their YES price moves toward 0 or 1 as resolution approaches. The Cliff Risk Index — |Δp/Δt| × τ_remaining — is the simple version of this measurement. High CRI means the contract is at a steep part of its price-versus-time curve. That is convexity, in the only language prediction markets currently have for it.

**Curve → Indicator Stack across τ.** A treasury yield curve plots yield against maturity. A prediction-market "curve" plots IY against τ-days for a *family* of related contracts — for example, all Fed-decision contracts at every meeting on the calendar. The shape of that curve tells you which horizons the market is treating as more uncertain than others. A flat curve says all horizons price the same. A steep curve says near-term contracts are paying disproportionately well, which usually means the market is anxious about an imminent event.

This is the closest thing prediction markets have to a *term structure*, and almost nobody plots it. Once you start, you stop thinking of contracts as isolated bets and start thinking of them as points on a surface that has a shape, gradients, and inflection points.

## Why This Matters Beyond Vocabulary

Three reasons.

**One — it makes risk additive.** Once two prediction-market positions can be expressed in IY, you can compare them to each other and to your existing fixed-income positions in the same units. You can compute a portfolio-weighted yield. You can compute a portfolio-weighted duration. You can answer "what is my current rate exposure across both prediction markets and treasuries" with a single number, which is something no current prediction-market dashboard will tell you.

**Two — it makes the risk legible to institutions.** A pitch to a bond desk that says "we have a contract trading at 42 cents that we think should be at 60 cents" is not a pitch. A pitch that says "we have a position with 18-day duration paying 8.3 million percent annualized that you can hedge against an SOFR future" is a pitch. The difference is not in the underlying trade — it is in whether the language survives the first thirty seconds of an institutional conversation.

**Three — it gives you a curve to fit.** Once you can plot IY against τ for a family of contracts, you can do everything bond traders have been doing for decades: identify roll-down opportunities, find dislocations between adjacent maturities, build a forward curve, compute a butterfly. The math is already invented. Prediction markets just have not noticed yet that they live in the same room.

## What Prediction Markets Are Missing

There is one important asymmetry. A treasury bond has a known issuer with a continuously updating credit story; the yield reflects both the risk-free rate and the credit spread. A prediction-market contract has no issuer — only an event. There is no underlying credit to model. So some bond-desk machinery (rating agencies, default correlation, recovery models) translates badly or not at all.

The replacement is *causal reasoning about the event*. Where a credit analyst studies a company's balance sheet, a prediction-market analyst studies the causal tree of the event itself: who needs to do what, by when, for the contract to resolve in your favor. The Cliff Risk Index, the IY curve, the cross-venue spread — these are the *technical* layer. The causal tree is the *fundamental* layer. Both have to be present.

The technical vocabulary is borrowable. The fundamental work is original. Together they are what prediction markets become when they grow up.

## Where to Start

If you want one habit to build, build this one: every time you look at a Kalshi or Polymarket contract, compute the IY before you compute the edge. Force yourself to express the contract in yield units before you express it in probability units. The reframing alone will change how you size positions, how you compare them, and how you talk about them.

Then plot the curve. Then notice the shape. Then ask which point on the curve is dislocated from the rest. The bond desks have been doing this for fifty years. The math is sitting there. All prediction markets need is a vocabulary upgrade — and the upgrade is already written down, in the textbook of every fixed-income trader who ever looked at a zero-coupon bond.