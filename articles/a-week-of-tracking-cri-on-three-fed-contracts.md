# A Week of Tracking CRI on Three Fed Contracts

> Daily log: three Kalshi Fed-decision contracts, seven mornings, what the cliff risk index told me each day and which mornings I should have listened to it.

**Category:** case-study | **Author:** Patrick Liu | **Published:** 2026-04-09

---
Last month I ran a small private experiment on myself. For one week, every morning at 7:30 a.m., I pulled the [cliff risk index](/learn/cliff-risk-index) on three Kalshi Fed-decision contracts and wrote down what I thought it was telling me. Then at the end of the week I went back and looked at what actually happened on each contract and asked myself, on a per-day basis, whether the CRI had been right and whether I had listened.

The three contracts were:

- **A** — KXFEDDECISION-26MAR01-Y, "Fed cuts at the March meeting," τ starting at about 14 days
- **B** — KXFEDDECISION-26MAY01-Y, "Fed cuts at the May meeting," τ starting at about 73 days
- **C** — KXFEDDECISION-26JUL01-Y, "Fed cuts at the July meeting," τ starting at about 134 days

I had small positions on all three going into the week. Modest. Six-figure-ish notional in aggregate. Nothing that would change my year, but enough that I cared about the answers.

This is the daily log, with the lesson at the end.

## Monday — Day 1

Mids that morning: A 0.51, B 0.39, C 0.31. Twenty-four-hour deltas: A +0.02, B +0.01, C 0.00. CRI values, eyeballed:

- A: |0.02| × 14 ≈ 0.28
- B: |0.01| × 73 ≈ 0.73
- C: |0.00| × 134 ≈ 0.00

The headline mover was A. The CRI winner was B. C was completely asleep.

What I wrote in my log: *"A is loud, B is structurally interesting, C is dead. Monday is a B day."*

What I actually did: nothing. I had convinced myself I was waiting for a Powell speech on Wednesday and "didn't want to chase the move."

## Tuesday — Day 2

Mids: A 0.50, B 0.40, C 0.31. Twenty-four-hour deltas were tiny. CRI values had collapsed from Monday — A was 0.14, B was 0.73 (still flagging because the Δp was the prior day's slow move, not a fresh one), C was unchanged at 0.00.

Notable: B had not moved, but the CRI was still flagging. This is one of the artifacts of using a 24-hour window — yesterday's signal is still in your window today. I wrote in the log: *"B's CRI is yesterday's news. C is still asleep. A is decaying back to sleep too."*

I did nothing. Again.

## Wednesday — Day 3

The Powell speech morning. Mids before the speech: A 0.49, B 0.39, C 0.30. CRI on all three was low because nothing had moved in 24 hours. The market was clearly waiting.

I wrote in the log: *"Coiled spring. CRI is the wrong tool today because the market is paused, not stuck. Watch the post-speech delta and recompute."*

This is one of the things I genuinely got right that week. CRI does not distinguish between "the contract is asleep because no one cares" and "the contract is asleep because everyone is waiting for the same scheduled event." A market that is *actively listening* will register zero CRI right up until the moment the catalyst hits. The metric is blind to anticipation. You have to know the calendar.

After the speech, the mids moved: A 0.55, B 0.41, C 0.30. C still hadn't budged.

Recomputed CRI on the post-speech snapshot, treating the move as a 6-hour delta and rescaling Δt:
- A: |0.06| / 0.25 days × 14 ≈ 3.36
- B: |0.02| / 0.25 days × 73 ≈ 5.84
- C: |0.00| × 134 ≈ 0.00

B was now flashing brighter than A by a wide margin. I wrote in the log: *"B is the live one now. A already moved. C is the silent one and I keep wondering what its silence means."*

What I did: I added to B. Not a lot — about 30% of my pre-existing B position size. I held A. I held C.

This was, in retrospect, the right move on B and the wrong non-move on C.

## Thursday — Day 4

Overnight, B drifted to 0.43 and A held at 0.55. C ticked up by half a cent to 0.305. CRI:

- A: ~0.0 (flat)
- B: |0.02| × 73 ≈ 1.46
- C: |0.005| × 134 ≈ 0.67

C was no longer dead. The CRI on C was now about half of B's, which was a major shift from "completely asleep" to "starting to wake up." I wrote in the log: *"C is whispering. B is still talking. Why is C waking up only after the speech?"*

This is the question I should have spent more time on. The answer was that the speech had moved the *whole curve* by reweighting near-term cut probability, and the slowest-moving end of the curve (the July contract) was the last to register the reweighting because of [thinner liquidity](/learn/liquidity). C was a lagging indicator of a structural reweighting that B and A had already absorbed.

I should have built into C on Thursday. I did not. I told myself I needed to "wait for confirmation."

## Friday — Day 5

Mids: A 0.56, B 0.46, C 0.32. CRI:

- A: |0.01| × 14 ≈ 0.14
- B: |0.03| × 73 ≈ 2.19
- C: |0.015| × 134 ≈ 2.01

C had now matched B in CRI for the first time all week. The whisper had become a normal speaking voice. B was still the leader but C was within striking distance.

I wrote in the log: *"C just confirmed. I should have moved on Thursday. Now Friday's price is already partway to where Thursday's CRI was forecasting."*

I added to C. About half the size I would have added on Thursday, because Thursday's price was 0.305 and Friday's was 0.32 and that 1.5-cent gap had eaten meaningfully into the [executable edge](/learn/executable-edge). The pattern of "wait for confirmation" had cost me about a third of the available move.

## Saturday and Sunday — No Trading

Markets closed. I used the weekend to read back through the week's log entries in one sitting and see what story they told.

The story was clear: B's CRI had been flashing on Monday and I had ignored it because I "wanted to wait for the speech." C's CRI had started flashing on Thursday and I had ignored it because I "wanted to wait for confirmation." In both cases the CRI was correct two days before I acted, and in both cases waiting cost me something concrete that I could measure.

The thing I had been doing all week was treating CRI as a *suggestion* and treating my own caution as the gating mechanism. The cliff risk index has no opinion about whether you are also waiting for a press conference or for confirmation or for your own gut to feel comfortable. It just tells you which contract is currently moving fast enough to deserve your attention.

## Monday Again — Day 8

A week later. Mids: A 0.61, B 0.53, C 0.36. The Fed had not even met yet — these were moves driven entirely by repricing expectations. All three contracts had moved meaningfully. The biggest gainer in absolute terms was B (+0.14), the biggest gainer in *percentage* terms over my entry was C (+0.04 from 0.305 if I had bought on Thursday), and A had been the most exhausted, drifting only +0.06 over the same window.

CRI had been right about B on Monday. CRI had been right about C on Thursday. I had been late on both.

## What I Take Away

The lesson is not "trust the CRI." That is the easy lesson and it is also wrong. The CRI is a noisy metric that flickers, lags 24 hours behind reality on a default window, and is genuinely blind to anticipation effects around scheduled events. It is not a magic oracle.

The lesson is more specific: **when the CRI is high but my position has not moved, I am not "patient" — I am missing the move.** This is the framing I should have applied to B on Monday and to C on Thursday. CRI was telling me both contracts were *active*. My position was not. The gap between "the market is moving" and "my position is moving" is the thing CRI is pointing at, and ignoring that gap is the most expensive thing I did all week.

I now have a rule: if any contract in my book has a CRI in the top quintile of my watchlist for two consecutive mornings *and* my position has not adjusted, I have to either size up or size out. Standing still is no longer one of the legal options. The standing-still failure mode is what cost me the moves on both B and C.

The other thing I now believe more strongly: **CRI on a single market is mostly noise; CRI as a relative ranking across a watchlist is signal.** The absolute value of CRI = 2.19 on B does not tell me much. The fact that B has been the highest-CRI contract in my watchlist for four consecutive mornings tells me a lot. I built my screener column to default-sort by CRI rank, not by raw CRI. That single change has been the highest-impact tweak to my morning routine since I added the [implied yield column](/blog/the-day-i-stopped-trusting-raw-probabilities) last fall.

There is a longer technical writeup of how the CRI is computed in the [indicator stack concept page](/concepts/pm-indicator-stack) and I will not repeat the math here. What I wanted to write down was the texture of using it on a real week with real positions and real costs of being wrong. The math is the easy part. Knowing when to move is the hard part. CRI helps with the second one if you let it.